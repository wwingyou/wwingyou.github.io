---
layout: post
tags: Spring
title: Spring Data Jpa
---

[[JPA]]를 위한 [[Spring Data]] 프레임워크이다.

`JpaRepository` 인터페이스를 이용해서 기본적인 [[CRUD]] 기능을 제공하는 리포지토리를 자동 생성해준다.

![spring-data-jpa-image](/assets/images/spring-data-jpa.png)

# 설정

`spring-boot-starter-data-jpa` 라이브러리를 추가해주면 된다.

**build.gradle**
```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
```

# 사용

`JpaRepository`를 상속받는 인터페이스를 만들면 스프링이 자동으로 구현체를 생성해준다.

```java
public interface ItemRepository extends JpaRepository<Item, Long> {
}
```

제네릭 타입으로는 엔터티와 엔터티의 ID 타입을 지정해주면 된다.

### 메소드 자동 생성

스프링 데이터 JPA는 인터페이스의 메소드 이름을 분석해 [[JPQL]]을 생성하고 수행하는 코드를 대신 만들어준다. 

```java
List<Item> findByItemNameLikeAndPriceLessThanEqual(String itemName, Integer price);
```

정확한 메소드 이름 규칙은 공식문서를 참고하자.

[https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)

### 리포지토리 사용

Spring Data JPA는 리포지토리 구현체를 만든 후 자동으로 컨테이너에 등록해준다. 그러므로 의존성 주입으로 가져와 바로 사용하면 된다.

```java
@RequiredArgsConstructor
public class ItemService {

	private final ItemRepository repository; // injected
}
```
