---
layout: post
title: "OAS와 Swagger Codegen으로 API 문서 및 코드 자동 생성하기"
tags: Swagger Spring 굴비잇기
product: goolbitg
---

## 적절한 문서화 도구의 필요성

굴비잇기 프로젝트를 진행하기 전 API 문서를 어떻게 관리할 것인지에 대한 고민이
많았다. 많은 사람들이 Swagger를 사용해 코드에서 자동으로 문서를 만드는 방식을
사용하고 있고 나 역시도 그렇게 해왔다. 하지만 이 방식에는 여러가지 단점이
있었다.

1. 서버 개발 전 설계 단계에서는 **Notion**과 같은 다른 도구를 사용해야 한다.
2. 코드에 **Swagger** 관련 어노테이션이 침투하여 가독성을 해친다.
3. API 문서를 코드와 분리해서 관리할 수 없다.
4. 서버가 완전히 개발되기 전까지는 클라이언트의 API 호출 로직을 구현하기가
   힘들다.

이런 단점들을 해결할 수 있는 방법이 뭐가 있을지 생각하던 중, [Swagger](https://swagger.io/) 사에서 다양한 API 제작 관련 도구들을 제공한다는 것을 알고 관심을
가지고 찾아보았다. 그 과정에서 **Swagger**가 코드에서 API 문서를 자동 생성하는
방법에 대해 자세하게 알게 되었는데, 그 핵심에는 **OAS(OpenAPI
Specification)**이 있다는 것을 알게되었다.

## OAS (OpenAPI Specification)

[OpenAPI Specification](https://spec.openapis.org/oas/v3.0.3.html)은 *다양한 프로그래밍 언어에 적용되는 HTTP API를 만들기 위한 정규화된 문서 규격*이다.
사람 또는 컴퓨터가 서버에 직접 접근하지 않고도 어떤 기능을 하는 API인지 알 수
있게 해준다.

아래는 **OAS** 문서의 예시이다:
```yaml
openapi: 3.0.3
info:
  title: Simple User API
  description: A simple API to manage users
  version: 1.0.0
servers:
  - url: https://api.example.com/v1
paths:
  /users:
    get:
      summary: Get a list of users
      responses:
        '200':
          description: A list of users
          content:
            application/json:
              schema:
                type: array
                items:
                  type: object
                  properties:
                    id:
                      type: integer
                      example: 1
                    name:
                      type: string
                      example: John Doe
```

위 스펙을 **Swagger Editor**를 이용해 **Swagger UI**로 만들면 아래와 같은 결과를
얻을 수 있다.

![swagger editor](/assets/images/swagger-editor.png)

위 예시에서 알 수 있듯, **Swagger UI**는 **OAS**로부터 테스트 가능한 API
문서를 만들어내는 도구이다. 그래서 일반적으로 **Spring Boot** 코드로부터
**Swagger UI** 문서를 만들 때는 [Swagger
Core](https://github.com/swagger-api/swagger-core) 라이브러리를 이용해
코드로부터 **OAS**를 먼저 생성한 후, 이것을 이용해 **Swagger UI**를 만든다.

그런데 이 방식은 이미 만들어진 서버의 API 문서를 만들 때는 좋은 방법이지만, API
설계부터 시작해야 하는 경우에는 앞서 말한 것처럼 별로 어울리지 않는다.
순서상으로 본다면 **OAS**를 먼저 작성하는 것이 더 적절하다.

## Swagger Codegen

앞서 말했듯 개발 시작 단계부터 **Swagger UI**로 API 문서를 제공하기로 결정했다면
코드에서 스펙을 생성하는 것보다 애초에 스펙부터 작성하는 것이 더 적절하다.
하지만 이 경우 코드와 문서가 일치하게 되는 기존의 방식의 장점이 사라진다.
그리고 **OAS**가 단지 API 문서를 만드는 용도로만 사용되기 때문에 다른 도구를
사용하는 것에 비해 가지는 강점이 매우 적다.

이때 사용하기 좋은 것이 바로 [Swagger Codegen](https://swagger.io/tools/swagger-codegen/)이다.
**Swagger Codegen**은 **OAS** 스펙으로부터 **Spring Boot** 코드를 생성해준다! 

**Swagger Codegen**은 여러가지 방식으로 사용할 수 있지만, 이번 프로젝트에서는 
Gradle 플러그인을 이용하여 빌드시 자동으로 코드 생성과 Swagger UI 생성을
해 주도록 설정했다. 플러그인 설정은 [스프링 6와 스프링 부트 3로 배우는 모던 API 개발](https://www.yes24.com/Product/Goods/139756583)을 참고했다.

**src/main/resources/api/config.json**
```json
{
  "library": "spring-boot",
  "dateLibrary": "java8",
  "hideGenerationTimestamp": true,
  "modelPackage": "com.goolbitg.api.model",
  "apiPackage": "com.goolbitg.api",
  "invokerPackage": "com.goolbitg.api",
  "serializableModel": true,
  "useTags": true,
  "useGzipFeature" : true,
  "unhandledException": true,
  "useSpringBoot3": true,
  "useSwaggerUI": true,
  "useBeanValidation": true,
}
```

**src/main/resources/api/openapi.yaml**
```yaml
openapi: 3.0.3

info:
  title: 🎣 굴비잇기 API 서버
  description: >
    굴비잇기 프로젝트의 API 서버입니다.
    ...
  version: 1.0.3

paths:
  /api/v1/auth/login:
    post:
      summary: 로그인
      operationId: login
      description: >
        외부 인증서버로부터 얻은 id 토큰을 이용해 로그인합니다.
    # ... 나머지 스펙들
```

**build.gradle**
```gradle
plugins {
    // ...
    id 'org.hidetake.swagger.generator' version '2.19.2' // 핵심 플러그인
}

group = 'com.goolbitg'
version = '0.0.1-SNAPSHOT'

// Swagger 플러그인 세팅
swaggerSources {
    def typeMappings = 'URI=URI'
    def importMappings = 'URI=java.net.URI'
    goolbitg {
        def apiYaml = "${rootDir}/src/main/resources/api/openapi.yaml"
        def configJson = "${rootDir}/src/main/resources/api/config.json"
        inputFile = file(apiYaml)
        def ignoreFile = file("${rootDir}/src/main/resources/api/.openapi-generator-ignore")
        code {
            language = 'spring'
            configFile = file(configJson)
            rawOptions = ['--ignore-file-override', ignoreFile, '--type-mappings',
                typeMappings, '--import-mappings', importMappings] as List<String>
            components = [models: true, apis: true, supportingFiles: 'ApiUtil.java']
            dependsOn validation
        }
    }
}

compileJava.dependsOn swaggerSources.goolbitg.code
sourceSets.main.java.srcDir "${swaggerSources.goolbitg.code.outputDir}/src/main/java"
sourceSets.main.resources.srcDir "${swaggerSources.goolbitg.code.outputDir}/src/main/resources"

dependencies {
    // ...
    // Swagger Codegen dependency
    swaggerCodegen 'org.openapitools:openapi-generator-cli:6.2.1'
    // Swagger UI dependency
    swaggerUI 'org.webjars:swagger-ui:3.52.5'
}

build.dependsOn generateSwaggerUI

processResources {
    dependsOn generateSwaggerCode 
    dependsOn generateSwaggerUI
    // Swagger UI 정적 파일을 jar에 포함
    from("${swaggerSources.goolbitg.ui.outputDir}") {
        into 'static/swagger-ui'
    }
}
```

이렇게 설정 후 gradle 빌드 태스크를 수행하면 `openapi.yaml`에 정의해 둔 데이터
모델과 API 인터페이스가 자동으로 생성된다.

아래는 생성된 API 인터페이스 예시이다.

**AuthApi.java**
```java
/**
 * NOTE: This class is auto generated by OpenAPI Generator (https://openapi-generator.tech) (6.2.1).
 * https://openapi-generator.tech
 * Do not edit the class manually.
 */
package com.goolbitg.api;

// imports

@Generated(value = "org.openapitools.codegen.languages.SpringCodegen")
@Validated
@Tag(name = "Auth", description = "인증")
public interface AuthApi {

    default Optional<NativeWebRequest> getRequest() {
        return Optional.empty();
    }

    /**
     * POST /api/v1/auth/login : 로그인
     * 외부 인증서버로부터 얻은 id 토큰을 이용해 로그인합니다. ### 인증 타입 - &#x60;KAKAO&#x60;: 카카오 로그인 - &#x60;APPLE&#x60;: 애플 로그인 
     *
     * @param authRequestDto  (required)
     * @return 로그인 성공 (status code 200)
     *         or 로그인 실패 (status code 422)
     */
    @Operation(
        operationId = "login",
        summary = "로그인",
        tags = { "Auth" },
        responses = {
            @ApiResponse(responseCode = "200", description = "로그인 성공", content = {
                @Content(mediaType = "application/json", schema = @Schema(implementation = AuthResponseDto.class))
            }),
            @ApiResponse(responseCode = "422", description = "로그인 실패", content = {
                @Content(mediaType = "application/json", schema = @Schema(implementation = ErrorDto.class))
            })
        },
        security = {
            @SecurityRequirement(name = "bearerAuth")
        }
    )
    @RequestMapping(
        method = RequestMethod.POST,
        value = "/api/v1/auth/login",
        produces = { "application/json" },
        consumes = { "application/json" }
    )
    default ResponseEntity<AuthResponseDto> login(
        @Parameter(name = "AuthRequestDto", description = "", required = true) @Valid @RequestBody AuthRequestDto authRequestDto
    ) throws Exception {
        getRequest().ifPresent(request -> {
            for (MediaType mediaType: MediaType.parseMediaTypes(request.getHeader("Accept"))) {
                if (mediaType.isCompatibleWith(MediaType.valueOf("application/json"))) {
                    String exampleString = "{ \"accessToken\" : \"accessToken\", \"refreshToken\" : \"refreshToken\" }";
                    ApiUtil.setExampleResponse(request, "application/json", exampleString);
                    break;
                }
            }
        });
        return new ResponseEntity<>(HttpStatus.NOT_IMPLEMENTED);

    }

    // 나머지 API pathes...
}
```

보다시피 Swagger UI 관련 어노테이션이 인터페이스 메소드에 자동 생성으로 추가된
것을 볼 수 있다! 이제 이 인터페이스를 구현하는 컨트롤러를 작성하면 Swagger 관련
어노테이션이 없는 깔끔한 컨트롤러 코드를 유지할 수 있다. 물론 이 프로젝트에서는
Swagger UI를 OAS로부터 직접 생성하는 방식을 사용했기 때문에 당장은 이
어노테이션들이 큰 의미가 없지만, **Swagger Core** 라이브러리에 의존하는 다른
도구를 사용할 때는 충분히 의미가 있다.

코드를 자세히 살펴보면 `getRequest()` 메소드가 유효한 요청 객체를 반환할 경우
`exampleString`을 반환해 주는 것을 볼 수 있다. 이는 **Swagger Codegen**이 스펙에
적힌 예시 데이터를 보고 자동으로 생성해 준 것인데, 덕분에 해당 메소드를 구현하지
않은 상태에서는 `502 Not Implemented` 상태 코드와 함께 예시 데이터를 응답한다.

하지만 마찬가지로 코드에서 볼 수 있듯이 `getRequest()` 메소드가 빈 `Optional`
객체를 반환하고 있으므로 예시 데이터는 제공되지 않는다. 이를 정정하려면
컨트롤러를 정의하고 `getRequest()` 메소드를 오버라이드 해주면 된다.

**AuthController.java**
```java
@RestController
public class AuthController implements AuthApi {

    @Override
    public Optional<NativeWebRequest> getRequest() {
        return ControllerUtils.getRequest();
    }

}
```

**ControllerUtils.java**
```java
public class ControllerUtils {

    public static Optional<NativeWebRequest> getRequest() {
        ServletRequestAttributes attributes =
    (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();

        if (attributes != null) {
            HttpServletRequest servletRequest = attributes.getRequest();
            HttpServletResponse servletResponse = attributes.getResponse();
            return Optional.of(new ServletWebRequest(servletRequest, servletResponse));
        }
        return Optional.empty();
    }

}
```

이제 프로젝트를 빌드하고 실행시키면 각 엔드포인트가 예시 데이터를 제공하는 스텁
API 서버가 실행된다! `http://localhost:8080/swagger-ui/index.html`을 요청하면
Gradle Plugin에 의해 자동 생성된 Swagger UI 페이지가 우리를 반겨준다. 그리고
API 테스트를 실행하면 스펙에 적어둔 예시 데이터가 응답으로 오는 것을 확인할 수
있다. 야호!

이제 이 서버를 미리 배포해두고 프론트엔드 개발자들이 열심히 API 호출 코드를
작성하도록 독려하면 된다. 더 이상 서버가 없어 개발을 못 한다는 말은 나올 수
없다!

## 문제점들

모든게 아름다워 보였던 Swagger 워크플로우였지만, 단점 또한 아주 명확했다.
지금부터는 이 워크플로우를 적용하며 어떤 문제점들이 있었고, 또 어떻게 극복했는지
설명해보려고 한다.

### 1. DTO 클래스를 수정할 수가 없다

이 워크플로우에서는 DTO 클래스가 자동 생성되므로 코드를 직접 수정할 수 없다.
그래서 DTO에 정적 빌더 메소드 등을 정의하는 일 등을 할 수 없었다.

하지만 이 부분은 따로 매퍼 메소드를 정의하면 되니 크게 문제되지는 않았다.

### 2. 생각보다 어려운 OpenAPI Specification 작성

굴비잇기 프로젝트에서는 대략 50개 정도의 API 엔드포인트가 필요했는데, 이 스펙을
표현하는데 무려 2,567 라인(!)이나 필요했다. 중복 코드를 컴포넌트로 잘
정리하고, IDE의 코드 폴딩 기능 등을 사용해야 그럭저럭 스펙 작성을 용이하게 할 수
있다.

### 3. 제네릭 타입 DTO는 어떻게?

공통 API 응답이나 페이지 응답과 같은 제네릭 타입 DTO를 표현할 방법이 마땅치
않다.
가장 적절한 방법은 `allOf` 키워드를 사용하는 것인데 아래와 같다.
```yaml
components:
  schemas:
    GenericResponse:
      type: object
      properties:
        status:
          type: string
        data:
          $ref: "#/components/schemas/DataPlaceholder"

    DataPlaceholder:
      type: object
      properties:
        value:
          type: string

    UserResponse:
      allOf:
        - $ref: "#/components/schemas/GenericResponse"
        - type: object
          properties:
            data:
              $ref: "#/components/schemas/User"

    OrderResponse:
      allOf:
        - $ref: "#/components/schemas/GenericResponse"
        - type: object
          properties:
            data:
              $ref: "#/components/schemas/Order"
```

하지만 이 경우 `GenericResponse<User>`과 같이 코드가 생성되는 것이 아니라
`UserResponse`와 같이 새로운 이름의 클래스가 생겨버린다. 상속 또한 적용되지
않아서 동일한 코드를 클래스마다 여러번 짜 줘야 하는 문제가 있다.

이 부분은 적절한 해결책을 찾지 못했다. 아마 이 부분이 **Swagger Codegen**을
이용하는 워크플로우를 사용하지 않을 가장 큰 이유가 될 것이다.

## 마무리

이상 Swagger Codegen을 사용하여 워크플로우를 구축한 기록이다. Swagger Codegen을
이용한 코드 자동 생성은 장점이 많지만 제네릭 DTO를 만들 수가 없다는 점 때문에 큰
프로젝트에서는 사용을 좀더 고려해봐야 할 것 같다. 이번 프로젝트처럼 단기간에
프론트엔드 개발자들과 협업해야 하는 경우에는 좋은 선택이 될 것 같다.

