# MicroService vs Monolithic
| Architecture     | MSA(MicroService Architecture)                                                                             | MA(Monolithic Architecture)                                                                                                                                                       |
|------------------|------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 방식             | 독립된 여러 서비스                                                                                         | 하나의 애플리케이션                                                                                                                                                               |
| 특징             | 1. 결합도가 낮고 확장성이 높다. <br>2. 비용이 많이 든다. <br>3. 통합 테스트가 까다롭다. <br>4. 개발자의 자율성이 높다. | 1. 결합도가 높고 확장성이 낮다. <br>2. 구조가 간단하다. <br>3. 비용이 적게 든다.                                                                                                          |
| 형상 관리        | 배포한 각 서비스에 대해 다른 서비스와 형상관리가 필수적이다.                                               | 하나의 어플리케이션으로 관리되기 때문에 형상 관리가 쉽다.                                                                                                                         |
| 배포 및 유지보수 | 1. 독립적인 서비스로 배포가 빠르다. <br>2. 서비스 별로 유지보수가 가능하다.                                    | 1. 전체를 다시 빌드하고 배포해 프로젝트의 규모가 커질수록 어플리케이션의 구동 시간, 빌드, 배포 시간이 많이 소요된다. <br>2. 한 프로젝트에 많은 양의 코드가 있어 유지 보수가 까다롭다. |
| 조직구조         | 대규모 조직에 적합하다. 하나의 독립된 팀에서 하나의 서비스를 개발한다.                                       |                                                                                                                                                                                   |

# API Gateway


API Gateway란 API서버 앞단에서 모든 API 서버들의 Endpoint를 단일화해 묶어주고 API에 대한 인증/인가/로깅/모니터링 외 메시지 기반으로 여러 서버로 라우팅한다.

## API Gateway 종류

### 1. *Spring Cloud Gateway (SCG)*

- Asynchronous, Non-blocking API, WebSockets, SSE를 지원하며, API Gateway 패턴을 구현함
- SCG는 스프링 기반으로 만들어졌기 때문에 Spring 서비스(Spring 5, Spring Boot 2, Project Reactor)와의 호환성이 좋음
- API 라우팅 및 보안, 모니터링/메트릭 등의 기능을 제공
- Spring Cloud Gateway는 Netty 런타임 기반으로 동작
→ **Servlet Container나 WAR로 빌드된 경우 동작하지 않음**

### 2. *Spring Cloud Zuul (Zuul)*

- Spring Cloud + Zuul 의 형태
- Spring Cloud 초창기 버전에 Netflix OSS에 포함된 컴포넌트 중 하나로 API gateway 패턴을 구현하는 기능
- Servlet Container 기반으로 만들어져 synchronous, blocking 방식으로 서비스를 처리
- Spring boot에서 사용되었으나 2.4버전 이후로 [zuul, hystrix, ribbon](https://spring.io/blog/2018/12/12/spring-cloud-greenwich-rc1-available-now#spring-cloud-netflix-projects-entering-maintenance-mode)
를 제공하지 않으며 SCG 사용을 권장

### 3. *Zuul2*

- Zuul에 없는 Asynchronous, Non-blocking 방식
- SCG와 동일하게 websocket, limit rate를 지원
- Zuul2도 Spring boot 통합 기능 제공하지 않으므로 SCG 사용 권장

## **SCG 동작 원리 및 구조**

<img src="https://velog.velcdn.com/images%2F_koiil%2Fpost%2Febf2211d-bd55-4f17-bf97-8c7dfa59ffd7%2Fimage.png" width="600" />
출처 : [https://www.slideshare.net/ifkakao/msa-api-gateway](https://www.slideshare.net/ifkakao/msa-api-gateway]

## API Gateway 역할

- **인증/인가** : 부적절한 요청을 차단하여 Backend service를 보호
- **L/B & Routing** : Client의 요청에 따라 적절한 backend service를 로드밸런싱(L/B: Load Balancing)하고 연결(라우팅)
- **Logging** : 유통되는 트래픽 로깅
- **[Circuit Break](https://bcho.tistory.com/1247)** : Backend service 장애 감지 및 대처

모든 Frondend의 요청을 라우팅하므로 다음의 유즈케이스에도 활용된다.

- 점진적으로 레거시 시스템을 신규 시스템으로 교체 : Strangler pattern 적용
- 트래픽 일부만 새로운 서비스로 라우팅 : Canari 배포(특정 user에게 배포해보고 이후에 전체 배포하는 타입) 가능

## API Gateway 적용시 고려할 사항

1. MSA 모델에서 API gateway를 사용할 경우 API Gateway 적용의 가장 큰 단점은 API Gateway를 내부 마이크로서비스와 결합해 기존 **SOA(*Service Oriented Architecture*)**의 **ESB(*Enterprise Service Bus*)**에서 발생했던 문제점이 다시 발생할 수 있다.
2. API Gateway의 Scale-out 적용이 유연하게 일어나지 않을 경우, API Gateway가 병목지점이 되어 어플리케이션의 성능저하가 일어날 수 있다.
3. API Gateway라는 추가적인 계층이 만들어지는 것이기 때문에, 그만큼 네트워크 latency가 증가한다.

---

***SOA(*Service Oriented Architecture*)***

**SOA**란 기능 단위로 묶어서 표준화된 호출 인터페이스를 통해서 서비스라는 소프트웨어 컴포넌트 단위로 재조합하고 분산 시스템 아키텍처에 대한 접근 방식으로 느슨한 결합 방식을 이용하며 이 서비스들을 서로 조합(Orchestration)하여 업무 기능을 구현한 애플리케이션을 만들어내는 소프트웨어 아키텍처이다.

***ESB(*Enterprise Service Bus*)***

**ESB**란 SOA에서 사용되는 개념으로, 서비스화로 통합되어 복잡해진 상태에서 서비스의 내용을 변경하거나 보강할 때 사용된다. 모든 서비스 중앙에 하나의 버스를 만들어 관리하며 복잡도를 해소하고 Intermediary 서비스를 만들어 서비스의 내용이 변경되었을 때 개선가능하다.

### **SOA vs MSA**

SOA와 MSA 모두 서비스 지향 설계 방식으로 MSA가 SOA에 속한다는 얘기도 있다. MSA는 큰 서비스를 잘게 쪼개어 개발/운영 하는 아키텍처이다.


## 기타 공부

**AOP(*Aspect Oriented Programming*)**

흩어진 관심사를 Aspect로 모듈화하고 핵심적인 비즈니스 로직에서 분리하여 재사용하는 것

* 소스 코드상에서 다른 부분에 계속 반복해서 쓰는 코드들을 흩어진 관심사(Crosscutting Concerns)라고 한다.

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FuTIy9%2FbtrH0cxHpnL%2Fr6vtVkE2K6RRK8X9VPqELk%2Fimg.png" width="550" height="280" />

동일한 기능을 여러 서비스에서 이용할 때 AOP는 기존 OOP(*Object Oriented Programming*)에서 바라보던 관점을 다르게 하여 부가 기능적인 측면에서 봤을 때 공통 요소를 추출하는 역할을 한다.
|                OOP               |                    AOP                    |
|:--------------------------------:|:-----------------------------------------:|
|      비즈니스 로직의 모듈화      |       인프라 혹은 부가 기능의 모듈화      |
| 모듈화 핵심 단위는 비즈니스 로직 | 각 모듈들의 주 목적 외 필요한 부가 기능들 |
