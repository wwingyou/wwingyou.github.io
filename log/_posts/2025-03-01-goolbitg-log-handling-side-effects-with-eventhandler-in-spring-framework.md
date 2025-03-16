---
layout: post
title: "복잡한 비즈니스 로직을 'EventHandler'로 정리하기"
tags: Spring 굴비잇기
product: goolbitg
---

## 시작은 간단했다

작심삼일 챌린지는 굴비잇기 앱의 핵심 기능 중 하나이다. 사용자는 원하는 챌린지를
등록할 수 있고, 3일간 챌린지 체크를 진행할 수 있으며, 원할 때 챌린지를 취소할 수
있다.

비즈니스 로직 구현은 간단했다. 특정 유저의 특정 날짜 챌린지 기록을 저장할 수
있는 `ChallengeRecord` 엔터티를 만들고, 이 엔터티를 적절하게 생성, 수정 삭제해 
주면 되었다. 챌린지를 등록하면 등록일로부터 3개의 기록을 생성, 체크하면 해당
날짜 기록을 변경, 취소하면 기록들을 삭제해 주었다.

여기까지는 모든것이 순조로웠다. 그러나...


## 새로운 요구사항의 등장

하지만 연속 챌린지 성공 회수와 최장 연속 회수를 확인할 수 있어야 한다는 새로운
요구사항이 생겼다. 이를 구현하는 가장 간단한 방법은 해당 통계 수치가 요청될 시에
과거 기록들을 모두 확인하여 통계 수치를 계산하는 것이다. 하지만 챌린지 기록은
매일 매일 새로 추가되었기 때문에 이렇게 구현한다면 통계 수치를 구하는 시간은
서비스가 지속될수록 점점 더 느려지게 될 것이다. 이는 장기적으로 별로 좋은 방법이
아니었다.

그래서 통계 수치를 저장할 수 있도록 `ChallengeStat`이라는 새로운 엔터티를
정의했다. 그리고 챌린지 기록이 업데이트될 때마다 `ChallengeStat`에 값을 계산해서
저장해주도록 구현하였다. 이렇게 하면 이전에 저장된 연속 성공 회수를 이용해 모든
기록을 순회하지 않고도 새로운 값을 계산할 수 있어 속도면에서 훨씬 유리했다.

하지만 이것이 끝이 아니었다! 이제는 유저마다 그날 총 몇개의 챌린지를 수행해야
하고 몇개를 이미 수행했는지 확인할 수 있어야 한다는 새로운 요구사항이 등장했다.
이를 위해 새로운 엔터티 `DailyRecord`가 정의되었고, 챌린지 기록 업데이트시마다
이 엔터티 역시 함께 업데이트되어야 했다.

업데이트할 엔터티의 수가 많아지자 자연스럽게 메소드의 길이는 길어졌다. 제공해야
할 통계 수치는 이게 끝이 아니었으므로 결국 챌린지 등록 메소드는 아래와 같이
엄청나게 길고 읽기 힘들어졌다.

```java
    @Override
    @Transactional
    public void enrollChallenge(String userId, Long challengeId, LocalDate date) {
        DailyRecordId dailyRecordId = new DailyRecordId(userId, date);
        ChallengeStatId id = new ChallengeStatId(challengeId, userId);
        ChallengeRecordId recordId = new ChallengeRecordId(challengeId, userId, date);

        Challenge challenge = challengeRepository.findById(challengeId)
                .orElseThrow(() -> ChallengeException.challengeNotExist(challengeId));
        User user = userRepository.findById(userId)
                .orElseThrow(() -> UserException.userNotExist(userId));
        UserStat userStat = user.getStat();
        DailyRecord dailyRecord = findOrCreateDailyRecord(userId, date, dailyRecordId);
        ChallengeStat challengeStat = challengeStatRepository.findById(id)
                .orElse(ChallengeStat.getDefault(challengeId, userId));
        Optional<ChallengeRecord> recordResult = challengeRecordRepository.findById(recordId);

        int startDay = 0;
        if (recordResult.isPresent()) {
            if (recordResult.get().getStatus() == ChallengeRecordStatus.SUCCESS &&
                recordResult.get().getLocation() == 3) {
                startDay = 1;
            } else if (!recordResult.get().getStatus().equals(ChallengeRecordStatus.FAIL)) {
                throw ChallengeException.alreadyEnrolled(challengeId);
            }
        }

        // ** ChallengeStat update... **
        Integer prevEnrollCount = challengeStat.getEnrollCount();
        if (prevEnrollCount == 0) {
            // New Enrollment
            userStat.increaseChallengeCount();
        }
        challengeStat.enroll();

        for (int i = 0; i < 3; i++) {
            ChallengeRecord record = ChallengeRecord.builder()
                    .challengeId(challengeId)
                    .userId(userId)
                    .date(date.plusDays(i + startDay))
                    .status(ChallengeRecordStatus.WAIT)
                    .location(i + 1)
                    .build();
            challengeRecordRepository.save(record);
        }


        // ** DailyRecord update... **
        if (startDay == 0) dailyRecord.enroll();
        challenge.enroll();

        dailyRecordRepository.save(dailyRecord);
    }
```

## 리팩토링이 필요한 시점

이 방식은 하나의 메소드가 너무 커져 읽기가 힘들었을 뿐만 아니라 나중에 새로운
통계 수치가 추가되었을 때 추가로 메소드를 수정해줘야 한다는 문제 또한 가지고
있었다. 이는 OCP 원칙을 위반하는 것이었다. 좀더 유지보수가 용이한 코드를 위해 
챌린지 기록 업데이트 로직과 통계 기록 업데이트 로직을 분리할 필요가 있었다.

다행히도 **Spring Framework**의 `ApplicationEvent`와 `EventHandler`를 사용하면 이런 문제를
해결할 수 있었다. 서비스 레이어의 메소드에서는 핵심적인 기록 수정 로직만
담당하고 변경된 기록에 대한 정보를 담은 이벤트를 발생, `EventHandler`에서
파생되는 통계 업데이트 로직을 수행할 수 있게 해 주는 것이다.

**ChallengeEnrollEvent.java**
```java
@Getter
public class ChallengeEnrollEvent extends ApplicationEvent {

    private String userId;
    private long challengeId;
    private LocalDate date;
    private boolean carriedOver;

    public ChallengeEnrollEvent(
        Object source,
        String userId,
        long challengeId,
        LocalDate date,
        boolean carriedOver
    ) {
        super(source);
        this.userId = userId;
        this.challengeId = challengeId;
        this.date = date;
        this.carriedOver = carriedOver;
    }

}
```

**ChallengeEventSubscriber.java**
```java
@Component
public class ChallengeEventSubscriber {

    @Autowired
    private ChallengeStatRepository challengeStatRepository;
    @Autowired
    private DailyRecordRepository dailyRecordRepository;
    @Autowired
    private UserRepository userRepository;

    @EventListener
    public void updateStatOnEnroll(ChallengeEnrollEvent event) {
        long challengeId = event.getChallengeId();
        String userId = event.getUserId();
        ChallengeStatId challengeStatId = new ChallengeStatId(challengeId, userId);
        ChallengeStat challengeStat = challengeStatRepository.findById(challengeStatId)
                .orElse(ChallengeStat.getDefault(challengeId, userId));
        UserStat userStat = userRepository.findById(userId)
                .orElseThrow(() -> UserException.userNotExist(userId))
                .getStat();

        if (challengeStat.getEnrollCount() == 0) {
            userStat.increaseChallengeCount();
        }
        challengeStat.enroll();

        challengeStatRepository.save(challengeStat);
    }

    @EventListener
    public void updateDailyRecordOnEnroll(ChallengeEnrollEvent event) {
        if (!event.isCarriedOver()) {
            String userId = event.getUserId();
            LocalDate date = event.getDate();
            DailyRecordId dailyRecordId = new DailyRecordId(userId, date);
            DailyRecord dailyRecord = dailyRecordRepository.findById(dailyRecordId)
                    .orElseGet(() -> DailyRecord.getDefault(userId, date));
            dailyRecord.enroll();
            dailyRecordRepository.save(dailyRecord);
        }
    }
}
```

**ChallengeServiceImpl.java**
```java
@Service
public class ChallengeServiceImpl implements ChallengeService {

    // ... other methods

    @Override
    @Transactional
    public void enrollChallenge(String userId, Long challengeId, LocalDate date) {
        ChallengeRecordId recordId = new ChallengeRecordId(challengeId, userId, date);
        Challenge challenge = challengeRepository.findById(challengeId)
                .orElseThrow(() -> ChallengeException.challengeNotExist(challengeId));
        Optional<ChallengeRecord> recordResult = challengeRecordRepository.findById(recordId);

        int startDay = 0;
        if (recordResult.isPresent()) {
            if (recordResult.get().getStatus() == ChallengeRecordStatus.SUCCESS &&
                recordResult.get().getLocation() == 3) {
                startDay = 1;
            } else if (!recordResult.get().getStatus().equals(ChallengeRecordStatus.FAIL)) {
                throw ChallengeException.alreadyEnrolled(challengeId);
            }
        }

        for (int i = 0; i < 3; i++) {
            ChallengeRecord record = ChallengeRecord.builder()
                    .challengeId(challengeId)
                    .userId(userId)
                    .date(date.plusDays(i + startDay))
                    .status(ChallengeRecordStatus.WAIT)
                    .location(i + 1)
                    .build();
            challengeRecordRepository.save(record);
        }

        challenge.enroll();

        applicationEventPublisher.publishEvent(
            new ChallengeEnrollEvent(this, userId, challengeId, date, startDay == 1)
        );
    }

    // ... other methods

}
```

이렇게 함으로써 복잡했던 서비스 레이어의 메소드를 간단하게 만들 수 있었고, 이후
새로운 통계 수치가 추가되더라도 기존 코드를 수정하는것이 아닌 새로운
`EventHandler`를 추가하는 것으로 대응할 수 있게 되었다.
