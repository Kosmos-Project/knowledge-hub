# 스프링 클라우드 게이트웨이

스프링 클라우드 게이트 웨이는 스프링 생태계 위에서 만들어진 API 게이트웨이다. 간단하고 효과적으로 API를 라우팅하며 보안, 모니터링, 탄력성에 대한 횡단 관심사를 제공한다.<br/>

Spring cloud는 비동기 기반으로 돌아가기 때문에, Spring webflux, project reactor에 대한 지식이 필요하다.(mono, flux 개념을 알아야함) 이 프로젝트들은 netty위에서 실행되며, 전통적인 서블릿 컨테이너에서 동작하지 않음을 알고있어야 한다.<br/>

스프링 클라우드는 `route`, `predicate`, `filter`로 구성되며 이들의 역할은 다음과 같다.

- route : 게이트 웨이의 기본적인 빌딩블록. ID, URI, predicate에 관한 집합, filter에 관한 집합에의해 정의된다. 만약 `aggregate predicate`가 참이면, 해당 경로로 라우팅된다.<br/>
- predicate : java8에서 제공하는 `function predicate`이며, 입력 값의 타입은 `ServerWebExchange`이다. `predicate`는 HTTP 요청의 모든 항목과 매칭할 수 있다.<br/>
- filter : 전송중인 요청 전후로 요청과 응답을 변형할 수 있다.
<br/>

스프링 클라우드 게이트웨이가 작동하는 방식은 다음과 같다.
1. 게이트 웨이 클라이언트가 요청을 만든다.
2. 게이트 웨이 핸들러 매핑이 요청을 어디로 라우팅할지 결정한다.
3. 게이트 웨이 웹 핸들러 필터 체인을 통해 요청을 실행한다.
<br/>

3번의 경우, 필터 내부의 로직은 프록시 요청이 전송되는 전후로 실행될 수 있다. 

route, predicate, filter를 설정하는 방법은 property에서 작성하면된다. 아래는 공식문서의 예제를 가져왔다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}
        filters:
        - AddRequestHeader=X-Request-Red, Blue-{segment}

```

<br/>
CORS를 게이트 웨이에서 설정하는 방법은 아래와 같다.

```yaml
spring:
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins: "https://docs.spring.io"
            allowedMethods:
            - GET
```

<br/>
