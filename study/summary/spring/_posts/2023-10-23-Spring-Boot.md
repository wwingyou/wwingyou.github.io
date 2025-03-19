---
layout: post
tags: Spring
title: Spring Boot
---

**Spring**의 복잡한 설정을 간단하게 해주는 여러가지 기능들을 제공한다.

# 스타터 라이브러리

프로젝트를 시작하기 위한 여러 라이브러리들을 하나로 묶어서 제공하는 라이브러리이다.

`spring-boot-starter-*` 형식의 이름을 가진다.
예) `spring-boot-starter-web`

비공식 스타터는 `*-spring-boot-starter` 형식의 이름을 가진다.
예) `mybatis-spring-boot-starter`

### 라이브러리 버전 변경

스프링 부트가 설정한 버전을 변경해야할 경우에는 아래와 같은 설정을 `build.gradle`에 추가해준다.

```
ext['tomcat.version'] = '10.1.4'
```

# 라이브러리 버전 관리

라이브러리간에 호환이 되는 버전을 자동으로 선택해준다.

### 설정

`io.spring.dependency-management` 플러그인을 추가해준다.

**build.gradle**
```groovy
plugins {
	id 'io.spring.dependency-management' version '1.1.0'
}
```

이제 `dependencies`에서 버전을 명시해주지 않아도 자동으로 알맞은 버전을 가져온다.

```groovy
dependencies {
	implementation 'org.springframework:spring-webmvc' // no version specified
}
```

# 자동 구성

`@AutoConfiguration` 어노테이션을 이용하면 따로 빈 등록을 하지 않아도 라이브러리를 프로젝트에 포함시키는 것만으로도 자동으로 빈 등록을 해주는 구성 클래스를 만들수 있다. 

스프링부트 스타터 라이브러리들은 이 기능을 이용해 자동으로 컨테이너를 구성해준다.

**자동 구성 내부 원리**

다른 구성 클래스를 포함시키는데 사용되는 `@Import` 어노테이션은 임포트할 구성 클래스를 정적으로 결정한다. 하지만 `ImportSelector`를 이용하면 구성 클래스를 동적으로 선택할 수 있는데, 이를 이용해 `@AutoConfiguration` 어노테이션이 붙은 클래스들을 찾아 구성 클래스로 등록한다. 

### 조건부 구성

`@Conditional` 어노테이션을 이용하면 조건에 맞을 때만 구성이 되도록 설정할 수 있다.

`@ConditionalOn*`형태의 간단한 어노테이션들이 많이 있다. 
예) `@ConditionalOnProperty`: 특정 환경 변수가 설정되어 있을 때 구성된다.

# 외부 설정

스프링 부트 프로젝트에서는 외부 설정값을 가져올 수 있는 대표적인 소스가 4가지 있다.

1. 외부 설정 파일 (`properties` 파일)
2. 운영체제 환경 변수
3. JVM 환경 변수
4. 실행 인수 (옵션 인수)

스프링 부트는 여러 외부 설정 소스를 통합해서 사용할 수 있는 추상화 계층을 제공한다.

각각의 소스를 활용하여 외부 설정을 구성하는 방법은 [[스프링 부트 외부 설정 구성]]을 참고하자.
