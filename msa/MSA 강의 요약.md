# Spring Cloud를 통한 MSA 구현
## 개요
강의 내용을 요약하고, 대략적인 정보 수집으로 알게 된 내용을 의식의 흐름에 따라 정리해 보았습니다.
개인적인 목적으로 요약해두었던 강의 노트를 게시한 것이므로, 평어체나 보그체(한영혼용체), 각종 은어 등이 포함되어 있는데, 양해해 주시면 감사하겠습니다.

## MSA의 일반론적인 등장 배경과 장단점
예전의 모놀리식 아키텍쳐에서는 모든 처리 로직을 하나의 어플리케이션에서 처리하였다.
예를 들어 로그인은 유저 서비스 클래스에서, 게시판은 BBS 서비스 클래스에서, 결제는 결제 서비스 클래스에서 처리하는 식이고, 이 모든 것이 하나의 데몬에서 돌아갔다.

그러다 보니, 특정 분야에 특화된 기술 스택을 적용하기 어렵고, 코드의 규모가 커지다 보니 수정이나 리팩토링, 스케일 아웃 등 유지 보수를 하기 어렵다는 단점이 있다는 것이 발견되었다.

이런 문제를 해결하기 위하여 마이크로서비스 아키텍처라는 설계방식이 생겨나기 시작했다.
각각의 Aspect별로, 기존에는 폴더나 패키지를 쪼개서 관리했던 서비스들을, 서로 다른 각각의 인스턴스 개체(데몬)으로 분리하여 운영하는 것이다.
이럴 경우, 각각의 Aspect에 최적화된 기술스택을 적용함과 동시에, 장애 대응이나 접속자수/부하/트래픽에 따라 유연하게 각 서비스별로 운영 규모를 달리 할 수 있다는 장점을 갖는다.
또한, 특정 서비스에서 오류가 발생하더라도, 전체 서비스가 다운되지 않는 견고함을 가진다는 장점도 있다.

MSA의 단점도 있는데, 기술스택이 다양한 만큼 특정 기술스택을 개발하던 사람이 퇴사하거나 특정 기술스택이 Outdated되어 시장에서 사장된다면(Classic PHP와 같이),
해당 서비스의 유지보수가 어려워진다는 문제가 발생한다.
-> 다만 강의 외적으로 다른 개발자 커뮤니티나 여러 블로거들의 의견을 참조해 본 결과, MSA의 경우 이런 일이 모놀리식 대비 자주 발생한다는 문제가 있지만,
  서비스별로 잘게 나뉘어 있다 보니 대응하기도 쉽다는 정보를 찾을 수 있었다. 해당 서비스만 일시 중지시키고, 신버전을 개발하여 해당 서비스만 대체하는 식으로 하면,
  서비스를 잘게 나눠놓은 정도에 따라 다르긴 하지만 생각보다 금방 교체가 가능하다고 한다.

## MSA를 구현하기 위한 Spring Cloud의 구성 및 소개
마이크로 서비스 아키텍처를 지원하기 위한 프레임워크 - 스프링 클라우드

스프링 클라우드는 다양한 하위 서비스를 제공

빠르게 어플리케이션을 개발할 수 있도록 분산 처리 등의 다양한 기능을 제공

스프링클라우드는 사용중인 스프링부트의 버전과 맞춰서 설치하여 개발을 하여야 된다.

스프링클라우드의 공식적으로 지원하는 서비스 중 이번 강의에서 다음의 기능들을 실습해볼 것이다

SC(Spring Cloud) Config, SC Netflix, SC Security, SC sleuth, SC Starters, SC Gateway, SC OpenFeign

 

환경 설정 관리를 위해서는 SC Config Server가 필요
게이트웨이 ip나 토큰 정보 등 여러 서비스에서의 설정 정보를 Git 등 한곳에 모아두고 사용.

서비스 등록과 위치정보 확인을 위해서는 Netfilx Eureks를 사용

서비스 부하 분산을 위해 Ribbon이나 Netflix Zouls, SC Gateway를 사용. 최신 버전에서는 SC게이트웨이를 사용하는 것을 강력히 추천함.

마이크로서비스간 REST통신을 원활하고 쉽게 해주기 위해서 FeignClient를 사용

외부 모니터링 또는 노드 추적을 위해 Zipkin Distributed Tracing이나 Netflix API Gateway를 사용

장애 진단 및 추적을 위해 Netflix의 Hystrix를 사용

## Service Discovery와 유레카
### 유레카 서버의 셋업 및 
MS가 1개의 서비스를 3개의 인스턴스로 로드밸런싱 해서 개발하는 예제

한대의 PC에서 할 경우에는 포트로 분산실행하고, 여러대라면 ip주소나 도메인을 다르게 해서 분산 운영이 가능하다.

Spring Cloud Netflix Eureka에 MS들을 저장해 놓고 개발할 것이다.

Spring Cloud Netflix Eureka는 Service discovery 역할을 한다.

각각의 마이크로서비스가 자신들의 위치정보를 Eureka서버에 등록을 먼저하여야 한다. 클라이언트는 어떠한 요청을 로드벨런스나 API게이트웨이 서버에 날리면 그 정보가 SD에 전달이 되고 SD를 통해서 Req/Res가 전달된다.

EcommerceDiscoveryService의 샘플 예제

스프링 프로젝트이고, 따라서 인텔리J의 Spring Initalizer를 통해서 프로젝트를 오픈한다.

Group은 개발팀 또는 개발사명의 도메인을 뒤집은 것
Artifact는 제품(프로젝트) 이름
메이븐이나 그래들은 취향대로 하면되고 여기서는 메이븐 쓸꺼
자바버전은 최소 8이상이어야되고 11이나 14를 권장
Description은 그냥 프로젝트 간략한 설명

알고있지만 다시 정리해보았다.

 

이 강의에서는 SB2.4.1을 쓸거고, Spring Colud Discovery > Eureka Server를 디펜던시로 추가한다.

 

SB버전별로 SCD의 버전을 다르게 사용하여야한다. 자세한 것은 홈페이지나 이전 강의 참조.

인텔리J와 SB 프렘웤 프로젝트 구조는 많이써봤으니까 생략..

EnableEurekaServer라는 어노테이션을 @SpringBootApplication 아랫줄에 삽입하여야함.

application.properties파일이나 yml을 써야되는데.. yml도 한번 써보는걸 권장한다고 한다.

port번호 지정. 강의자는 8761번을 예시로 사용함

어플리케이션 ID를 지정해야 되는데

그걸 discoveryservice로 했다.

eureka client 설정 - 서버인데 왜 이걸 지정해야 하냐면, register-with-eureka와 fetch-registry를 false로 옵션을 줘야 유레카서버 자기자신이 자기자신에게 등록하는 것을 막을 수 있기 때문이다.

실행 방법은 그냥 흔하디 흔한 InteliJ로 SB개발하는것과 똑같고

유레카 서버에 접속하면 대쉬보드 화면을 보여준다.

시스템 상태와 등록된 서비스 인스턴스에 대한 정보가 나타난다.

### 유레카에 등록할 서비스를 생성하는 예시
유레카 서버에 등록할 첫번째 MS인 유저서비스를 생성해보는 과정이다. 이건 그냥 일반적인 SB 프레임워크 생성하듯 하면 되고, 별 게 없다.

물론, eureka-client 라이브러리는 추가하여야 된다.

@EnableDiscoveryClient라는 어노테이션을 메인에 추가해 주어야 된다. @EnableEurekaClient를 추가해도 된다고 한다.

application.yml (또는 properties)의 경우
포트 설정에 신경쓰고(강의자는 9001를 사용하였다.)
어플리케이션 네임은 편한대로 하고
eureka.client.register-with-eureka와 fetch-registry를 true로 하여야 한다. 전자는 유레카에 등록하겠다는 거고 후자는 갱신되는 인스턴스들의 정보를 받아오겠다는 뜻이다.

service-url에 defaultZone이라는 걸 설정해줘야하는데, 이것은 유레카 서버의 주소로 하면 된다. 예시의 경우 http://127.0.0.1:8761/eureka가 되겠다.

실행 방법은... 별 거 없다. 일반적인 SB프로젝트와 동일.
당연하겠지만, 유레카에 등록 및 연동해서 작동하기 위해서는 유레카가 같이 켜져있어야된다. 유레카 대시보드에 잠깐 뜨는 경고문은 무시해도 된다. 곧 있으면 사라진다.

서비스 이름은 대소문자 구분이 되지 아니하며 유레카 대시보드상에서는 전부다 대문자로 표출된다.

이로써 User Service를 Eureka 라이브러리에 등록해 보았다.

### 여러 인스턴스 띄우기
유레카 서비스는 기존대로 띄워놓고, 유저 서비스를 여러 개 실행시켜 본다.

첫번째 유저 서비스는 9001번 포트로 정상 작동.

두번째 유저 서비스를 구동시키기 위해서는, Edit Configurations를 통해서 실행 설정을 해주어야 된다. 실행정보 목록 위의 아이콘 중 Copy 아이콘을 눌러서 하나 더 추가해준다. 그렇게 새로 만들어진 프로파일로 돌려서 눌러 두번째 유저 서비스를 실행시킨다.

단, 포트 충돌이 날것인데, 포트번호만 9002로 돌려서 실행시킨다.

application yml의 서버포트값을 바꾸면 매번 빌드와 패키징을 해야하므로, yml을 여러개 하거나..

아니면 VM-option에다 -Dserver.port=9002 옵션을 추가하여 설정을 override 할 수 있다. 강의자는 이 방법을 알려줌. 다만 이렇게 하느니 기존에 내가 하던데로 그냥 yml을 환경별로 여러개 만들어서 default properties를 실행 옵션별로 다르게 지정해서 돌려쓰는게 나을 것 같다. 다만 포트세팅 하나만 딱 바꿀거라면 강의자가 말하는 방법도 그렇게 나쁘지는 않은 듯.

아무튼 이렇게 2개 실행시키고 나면 유레카에서 인스턴스가 2개 됨을 알 수 있다.

 

마지막으로, 빌드해서 CLI로 돌리는 방법이 있는데, 이것은 그냥 수동 배포할때랑 똑같이 JAR파일 떨어트려서 하면 된다. 터미널을 쓰든 Gradle/Maven 패널을 쓰든 상관은 없다. 어쨌든 빌드/패키징 해서 JAR파일을 떨어트리면 된다.

당연하겠지만 시스템에 JDK는 설치가 되어 있어야 할 것이고..

아무튼 JAR파일을 그냥 시작시키게되면 또 포트충돌이 날 것이므로 run -Dspring-boot.run.jvmArguments='-Dserver.port=9003'과 같이 CLI 실행 옵션으로 포트번호를 달리 해 준 상태로 기동하면 된다.

그렇게 하고 나면 Eureka에서 인스턴스가 3개 떠있는 것을 확인해 볼 수 있다.

강의자는 4번째 인스턴스도 시작했는데...
별다른 차이는 없고 그냥 java -jar [jar파일경로] -(포트변경 옵션)으로 실행하는 방법을 알려줌.

이 작업을 편리하게 할 수 있는 방법은 다음 강의에서

### 여러 인스턴스 편하게 띄우기
매번 포트 번호를 지정하는 것은 곤란한 일이므로, 랜덤 포트 번호를 사용해 보도록 한다.

0번을 입력하면 랜덤 포트가 된다.
사실 도커로 돌리면 이 모든 귀차니즘이 덜어지긴 하지만... 그건 뒤에서 다루겠지요.

다만 0번으로 돌리게되면 기본값으로는 server.port에 명시적으로 입력된 값만 뜨기 때문에 유레카 대시보드 상에는 1개밖에 표출이 안된다.

따라서 이걸 해결하기 위해 instance-id 옵션을 주어야 하는데..

eureka.instance.instance-id: ${spring.cloud.client.hostname}:${spring.application.instance.id:${random.value}}

로 옵션을 추가한다. 그렇게 설정하면 유레카 상에서 정상적으로 정보가 표출된다.

스프링 부트와 스프링 클라우드를 통해서 이렇게 간단하게 MSA를 설정할 수 있다.

## API 게이트웨이와 구현 방법
3개의 MS로 구성된 MSA를 예시로 들면, 클라이언트에서 세 가지의 서비스 주소와 접속 정보, 엔드포인트에 대한 정보를 가지고 있어야 하고, 서비스가 추가되거나 접속 정보가 바뀔 경우 매번 수정되어야 하는 비효율성과 불편함이 있다.

따라서 단일 진입점을 유지하기 위해 API Gateway를 둔다. 이렇게 할 경우 직접적으로 서비스를 호출하지 않고 게이트웨이를 통해서만 소통하게 되며 이럴 경우 불편함이 해소되고 다음과 같은 상당한 장점이 있다.

인증 및 권한 부여를 통합할 수 있고, 검색 역시 통합적으로 서비스할 수 있게 된다. 응답 캐싱도 가능하며, 장애 대응을 위한 정책, 회로 차단기 및 QoS(속도 제한 및 부하 분산)을 적용할 수도 있다. 또한, MSA의 경우 서비스들이 서로 관계된 것들끼리 복잡하게 상호 호출하는 경우가 많기 때문에, 로깅 및 상관관계 추적 등의 측면에서도 장점이 된다. 이를 위한 로깅 관리 제품으로 ELK등이 있다. 또한, 헤더 및 쿼리 스트링의 변환도 가능하고, IP 화이트리스트 또는 블랙리스트를 적용하여 접근 제어도 가능하다.

Spring Cloud에서의 MSA간 통신을 위해서 가능한 구현 방법은 두 가지가 있다.

1) RestTemplate
RestTemplate restTemplate = new RestTemplate();
restTemplate.getForObject("http://서비스주소:8080/", User.class, 200); (POST로 호출도 가능)

2) Feign Client
@FeignClient("stores")
public interface StoreClient {
@RequestMapping(method=RequestMethod.GET, value = "/stores")
List<Store> getStores();
} (외부 서비스명을 등록해 사용한다. 직접적인 주소 입력 필요 ㄴㄴ)

클라이언트 사이드에서의 로드 밸런싱을 위해서 Netflix Ribbon이라는 툴이 생겨났다. 다만 요즘에는 비동기 지원이 잘 되지 않아서 사용하지 않는 경우도 있다. 리액트와 같은 프레임워크와는 잘 맞지 않다. 클라이언트 사이드에서 사용할 수 있는 로드 밸런서로써, 클라이언트에서 여러 서비스들의 접속 정보를 모두 저장하고 관리하는 것은 동일하나, 호출때마다 접속정보를 입력하지 않아도 된다. API Gateway를 클라이언트 측에 탑재한 것이라고 보면 된다. 다만 이 툴은 Spring 프레임워크 생태계에서 메인터넌스 상태로 전환되어 신규 업데이트는 되지 않는다.

Netflix Zuul이라는 제품도 나와 있다. 이것 또한 API Gateway의 기능을 한다. 클라이언트에 통합된 것은 아니고, API Gateway가 별도의 데몬으로 작동하는 방식이다. 다만, 이것도 메인터넌스 상태로 전환되었고 최근 버전에서는 사용할 수 없다.

### 넷플릭스 줄을 이용
NetFlix Zuul을 통해 구현해보는 회차이다.

스프링부트 2.3.8 및 그 이전 버전에서 실습해야 한다.

디펜던시로는 롬복, 스프링 웹, 유레카 디스커버리 클라이언트를 추가하였다.

First Service와 Second Service라는 2개의 스프링부트 프로젝트 및 서비스를 만든다. 각각의 기능은 그저 welcome이라는 엔드포인트 하나만 가지고 있으며, 각각 welcome to the first service라는 문구와 welcome to the second service라는 문구를 응답해 줄 뿐이다.

마지막으로, Zuul 어플리케이션을 생성한다. 롬복, 스프링 웹, Zuul을 디펜던시로 선택한다.

@EnableZuulProxy
public class ZuulServiceApplication {
public ~~ (메인함수)

}
와 같은 식으로 애플리케이션 메인 함수가 작성된다.

컨피그에
zuul:
routes:
first-service:
path: /first-service/**
url: http://localhost:8081
second-service:
path: /second-service/**
url: http://localhost:8082
위와 같은 식으로 작성하여 각각의 서비스 이름과 엔드포인트, 포워딩해 줄 url을 지정한다.

Zuul Filter라는 것을 추가해서 각각의 서비스가 호출될 때 사전처리, 사후처리를 적용할 수 있다.

사전처리에는 인증 기능 등을 추가할 수 있고, 사후처리에는 로깅 등의 기능을 추가할 수 있다.

Object run 이라는 메소드에 처리할 내용을 작성한다.
예제에서는 호출시에 로그를 프린트하도록 하였다.
filterType 메소드에 return "pre"로 반환하면 사전필터고, "post"로 반환하면 사후필터이다.

단일 서비스에 여러 서비스 네임의 중복 지정도 가능한 것으로 확인됨.

Zuul 필터를 적용해 보는 실습을 하는 과정이다.

ZuulLoggingFilter라는 클래스를 하나 추가하였다.

ZuulFilter를 구현하기 위해서는 ZuulFilter라는 객체를 상속받아야 하고, @Component라는 스프링 빈 스테레오타입을 지정한다.

추상 메소드를 제공(재정의)하여야 하는데...
네 가지의 메소드를 정의하여야 한다.
filterType은 사전필터인지 사후필터인지 적용하는 메소드
filterOrder는 필터 적용 순서를 지정하는 메소드
shouldFilter는 필터의 적용 여부를 지정하는 메소드이다.
마지막으로, run이라는 메소드를 작성하는데, 필터가 적용될 때마다 이 메소드의 내용이 작동된다.

이 예시에서는
public Object run() throws ZuulException {
logger.info("******* printing logs: ");

RequestContext ctx = RequestContext.getCurrentContext();
HttpServletRequest request = null;
logger.info("******* printing logs: " + request.getRequestURL());
return null;
}

와 같은 코드를 적용하여, 사용자가 어떤 URL을 호출했는지 로깅하는 기능의 코드를 추가하였다.

변환작업 등에 응용할 수 있다.

### Spring Cloud Gatway를 사용
이번에는, Spring Cloud Gateway를 사용하여 앞서 Zuul과 비슷한 예제를 구현해보었다.
Spring Cloud Routing이라는 카테고리에 Gateway라는 의존성 항목을 선택하여 프로젝트를 생성한다.

spring:
application:
name: gateway-service
cloud:
gateway:
routes:
- id: first-service
uri: http://localhost:8081/
predicates:
- Path=/first-service/**
- id: second-service
uri: http://localhost:8082/
predicates:
- Path=/second-service/**
위와 같은 식으로 설정파일을 작성한다. 서비스네임이 id로 바뀌었을 뿐 줄과 설정상 큰 차이는 없다. 줄과의 기능적인 차이점은, 비동기 지원 면에서 월등히 뛰어나다는 것이다.

@RestController
@RequestMapping("/first-service")
public class FirstServiceController{}
와 같은 식으로 스프링의 컨트롤러에서 그저 기존의 엔드포인트에서 특정한 섹션을 지정하듯이 service-id를 엔드포인트 uri로써 지정해주면 해당 서비스로 연결이 된다.

앞선 회차에서 설명한 Spring Cloud Gateway를 실제로 예제를 통해 구현해 보는 과정이다. 별다른 특이사항은 없다.

단, 주의사항 하나. 각각의 MS를 구성하는 서비스의 Controller에 RequestMapping시에 Service-ID를 받도록 해야한다.
ID가 first-service일 때는
@RequestMapping("/first-service") 와 같이 하면 된다.
그대로 주소값이 포워딩 되어서 넘어가기 때문에 이러한 주의점이 발생한다.

Spring Cloud Gateway 툴을 사용하면서 필터를 적용하는 방법에 대해서 알아본다.

Gateway Handler Mapping - 클라이언트에서 들어온 요청의 정보
Predicate - 요청의 조건
PreFilter, PostFilter - 사전 처리 필터, 사후 처리 필터

JAVA Code로 작성도 가능하며, Properties로도 적용이 가능하다.

자바코드로 FilterConfig라는 클래스를 만들고, 스테레오타입으로는 @Configuration을 지정한다. 이럴 경우 기동 처음에 메모리에 우선 등록하게 된다. @Bean의 네임은 RouteLocator gatewayRoute로 한다.

.route(r -> r.path("/first-service/**")
.filters(f -> f.addRequestHeader("first-request", "first-request-header").addResponseHeader("first-response", "first-response-header")
.build();

식의 코드를 작성하여 필터를 작성할 수 있다. 람다식이 아니라 일반 코드로도 작성이 가능하지만 람다식(익명 메소드)를 사용하는 것이 요즈음의 추세이다. .route에 명시된 지점으로 요청이 들어오면 .filter에 명시된 작업을 실행하겠다는 것이다. 여기의 예제에서는 단순히 헤더에 특정 정보를 추가하는 코드를 작성하였다.

Filter Config을 작성하는 예제를 실습하는 내용이다.
route(r -> r.path("/first-service/**)
.filter(f -> f.addRequestHeader("", "").addResponseHeader("", ""))
.uri("http://localhost:8081))
.build();

와 같은 내용을 작성하여서 리퀘스트 헤더와 리스폰스 헤더에 특정 문자열을 추가하는 코드를 작성하였다.

Applcation YAML파일에서 할 것을 위와 같이 JAVA Code로써 작성하는 것도 가능하다.

서비스 측에서는 public String message(@RequestHeader("first-request") String header) {
log.info(header);
return "Hello World in First Service"
}
와 같은 코드를 작성하여 헤더값을 로그로 기록하고, 헤더 내용으로 헬로우 월드를 넣어주었다. 세컨 서비스 역시 문구만 다소 다를뿐 같은 기능을 하는 코드를 넣어주었다.

이를 통해 application YAML파일에 작성되는 라우팅 정보를 코드로 구현하는 방법과 Req/Res를 가로채서 특정 정보를 추가하는 방법을 실습하였다.

스테레오타입과 @Bean 어노테이션만 막아줘도 해당 클래스를 스프링부트 등록에서 배제하는 것이 가능하다.

filters:
- AddRequestHeader=first-request, first-request-header2
- AddResponseHeader=first-response, first-response-header2
와 같이 간단한 기능은 spring boot 프로퍼티스에서도 필터로써 추가해 줄 수 있다.
--하지만 난 필터 로직 작성은 코드로 하는 게 더 깔끔하다고 생각한다--

사용자 정의 필터를 적용해보는 실습 과정이다.

public class CustomFilter extends AbstractGatewayFilterFactory<CustomFilter.Config> {
public CustomFilter() {
super(Config.class);
}

@Override
public GatewayFIlter apply(Config config) {
return (exchange, chain) -> {
ServerHttpRequest request = exchange.getREquest();
ServerHttpResponse response = exchange.getResponse();

log.info("Custom PRE filter: request uri -> {}", request.getId());
return chain.filter(exchange).then(Mono.fromRunnable() -> {
log.info("Custom POST filter: response code -> {}", response.getStatusCode();
}));
}
}
}
와 같은 코드를 통해 사전 필터로 요청한 uri를 로깅하고, 사후 필터로 응답 코드를 로깅하는 예제를 만들어 보았다.

실무적으로는, 여기에다 로그인 인증 토큰 관리 코드 등을 삽입하여서 활용한다.

마지막으로, 위처럼 구현한 커스텀필터를 적용하기 위해 YAML파일에 - CustomFilter를 추가해 준다.

테스트를 위해서는, 퍼스트 서비스와 세컨드 서비스에 엔드포인트를 하나 추가하여 특정한 스트링을 하나 반환해 준다.

커스텀필터를 마저 구현해본다.

AbstractGatewayFilterFactory라는 걸 상속받아서, <CustomFilter.Config> 형식을 제너릭에다 넣어준다.

구현시켜줘야 할 추상 메소드들을 추가하고, apply에서 적용하고자 하는 프리 필터와 포스트 필터를 구현해 준다. 프리 필터를 먼저 반환하고, 그 반환값 안에서 포스트 필터의 구현 코드를 반환한다.

글로벌 필터의 구현 방식을 알아본다.

코드의 구현 방식은 똑같다.

다만, AbstractGatewayFilterFactory의 제네릭에 <GlobalFilter.Config>을 넣어준다.

기존의 커스텀필터의 경우 프로퍼티스의 라우팅정보에 사용할 커스텀필터의 클래스 네임을 명시해서 지정해주어야 해당 라우팅에 적용이 되는데, 글로벌필터는 프로퍼티스에 지정해줄 필요 없이 전역적으로 적용이 된다.

필터를 구현할 때 필터 클래스에 임포트하는 AbstractGatewayFilterFactory 클래스의 추상 메소드에서 제공되는 Config에 몇 가지 변수를 선언하여 활용할 수도 있다. 저렇게 선언된 변수들은 application 프로퍼티스에서
default-filters:
- name: GlobalFilter
args:라는 설정값에서 컨피그처럼 변수의 사용이 가능하다. (컨피그에 값을 세팅하면 그 값이 변수로 들어옴)

예제에서는, 컨피그 파일에 입력해둔 기본 메시지를 추가하는 기능과 컨피그에서 설정한 true/false값에 따라 프리필터/포스트필터 각각의 적용여부를 컨트롤할 수 있는 기능의 코드를 작성하였다.

로깅 필터를 추가로 만들어보는 실습 예제이다.

필터의 적용 순서는 먼저 글로벌 필터가 적용되고, 커스텀필터의 경우 컨피그에 정의된 라우트에 해당 필터명을 명시한 순서대로 적용된다.

추가적인 파라미터를 넣어줄거라면 무조건 해당 라우트에 적용되는 모든 필터에 대해 -name:을 붙여주어야 한다.

팩토리 추상메소드 구현시 두번째 인자값으로 Ordered.HIGHEST_PRECEDENCE 를 넣어주면 해당 필터의 우선순위를 최고우선순위로 오버라이드 할 수 있다. 즉, 기본적으로는 컨피그파일에서의 명시 순서대로 작동하지만 두번째 인자값을 통해 수동으로 필터의 적용 우선순위를 오버라이드 할 수 있다는 것이다.

### 유레카와 SPG의 연동
유레카와 Spring Cloud Gateway를 연동해보는 과정이다.

작동 순서는

클라이언트의 요청

유레카로 가서 해당 마이크로서비스의 위치 검색 및 발견

APIGateway로 가서 Spring Cloud Gateway에 명시된 필터 등의 기능이 적용되고 설정된 대로 포워딩된다.

스프링 게이트웨이 어플리케이션 프로퍼티스에 uri의 값이 상당히 달라진다. 유레카 없이 작동할 때는 uri에 직접 해당 서비스의 주소를 입력하여 작동했는데, 유레카와 연동하여 작동할 때는 lb://MY-FIRST-SERVICE와 같은 식으로 uri를 작성해준다. lb://를 붙여주고, 그 뒤에 유레카에 등록시 지정해둔 어플리케이션 서비스 네임을 적어주는 것이다.

당연하겠지만, 디펜던시에 유레카 클라이언트를 추가해주고..
프로퍼티스를 통해서 register-with-eureka와 fetch-registry를 트루로 해주고, 유레카 서버 url을 지정해 준 다음에 서버들을 전부 기동하면... 적용된다.

참고: 유레카 서버가 실행되기 전 실행된 서비스들은 재기동시켜주어야 원활하게 설정 적용이 된다.

이번에는, 서비스의 다중 인스턴스 구동을 활용해본다.

First Service와 Second Service를 포트 변경을 통해 각각 2개의 인스턴스를 구동한다.

랜덤포트로 하는 예시도 보여주는데.. 이때는 예전의 유레카 클라이언트 강의에서 나왔듯 추가 설정이 필요하다.

API Gateway Service의 라우트 컨피그서는 포트 번호를 명시하지 않는다.

이렇게 하게 되면 라운드 로빈 방식으로 호출할때마다 번갈아가면서 여러 인스턴스에 고르게 요청이 분산되어 작동하게 된다.

## MSA 예시 프로젝트 설계
MSA 자체를 만드는 것에 포커스를 맞추기 위해 간단하게 만든다

상품을 조회할 수 있는 카탈로그 서비스와, 사용자를 조회하고 주문을 확인할 수 있는 유저 서비스가 있고, 상품 주문 정보가 저장되는 오더 서비스가 있는 형태의 구성을 하게 된다.

이런 경우에 서비스 간의 상호 호출이 발생한다.

첫 번째 예시는, 유저 서비스에서 사용자가 주문 확인을 해야 할 때 오더 서비스의 정보를 가져오는 상황이다. 서비스간 통신을 하는 방법 중 첫 번째 방법은 유저 서비스에서 오더 서비스의 메소드를 직접 호출하는 식으로 구현이 가능하다. 다음 파트에서 이에 대한 예시를 실습해 본다.

두 번째 예시는, 상품이 주문 되었을 때 오더 서비스가 상품 수량을 업데이트를 하는 상황을 가정해 보자. 이때는 카탈로그 서비스에 요청을 해야 하는데, 서비스간 요청이 아니라 카프카라는 메시지 서비스를 사용할 수도 있다.
오더 서비스에서 카프카에 데이터를 프로듀스 해 두면, subscriber로 등록이 되어있는 카달로그 서비스에 해당 수치가 전달이 된다. 그러면 카달로그 서비스에서 이를 갱신하게 된다. 마찬가지로, 다음 파트에서 이에 대한 예시를 실습해 본다.

프로젝트의 전체 밑그림을 그려보는 단계이다.

유레카 서버를 하나 만들고, 세 가지 서비스를 유레카에 등록한다. 그리고 세 가지 서비스는 카프카의 메시징 채널을 통해 상호간 소통이 가능하고, 부하 분산과 서비스 라우팅을 위해 API게이트웨이를 물려서 프론트의 입력을 받게끔 한다.

구현 단계가 끝나고 운영 파트의 강의에서는, Configuration 서버를 추가하고 모니터링 기능을 추가 해본다

이 강의에서 생략된 부분까지 살려서 조금 더 충실하게 구성해보자면, 배포할 때 쿠버네티스를 사용하는 방법과 각 서비스를 도커 이미지화 하여 도커 컨테이너로 돌리는 방법을 적용하고, 변경사항을 CI/CD 파이프라인을 통해서 도커 레지스트리를 활용해 실시간 배포가 이뤄지게 하는 방법도 가능하다.
또한 API 게이트웨이 앞단에 NGINX 인그레스라는 툴을 물려서 클라이언트에서 요청이 들어오면 자동으로 배포를 하게 만들 수도 있다.
모니터링 역시 프로메테우스와 같은 서비스를 도커를 활용하여 연동하면 조금 더 충실하게 가능하다. (다만 이 강의에서는 분량상 생략)

하여, 이 강의에서 실습하는 전체 애플리케이션의 구성요소는 다음과 같다.

Git 저장소를 이용해서 코드 관리를 하고

Configuration 서버를 이용해서 Git 저장소에 들어가던 프로파일 정보와 설정 정보를 안전하게 별도 관리하고,

유레카 서버를 통해서 마이크로서비스를 등록 및 검색하고

API 게이트웨이 서버를 통해서 부하분산과 서비스 라우팅을 제공하고

마이크로서비스간 메시지 발행과 구독을 위해 카프카를 사용한다.

마이크로서비스는 카탈로그 서비스, 유저 서비스, 오더 서비스로 구성된다.

카탈로그 서비스

상품 목록 조회 기능

유저 서비스

사용자 정보 등록

전체 사용자 등록

사용자 정보, 주문 내역 조회

오더 서비스

사용자 주문 등록

사용자 주문 확인

## User Microservice 구현 예시
### 간단한 서비스 기능 소개
프론트는 생략

비즈니스 로직으로 RestAPI만 구현 (UI단 없다)

API 게이트웨이의 추가를 통해 /user-service/라는 접두어 없이 접근이 가능하다.

사용자 정보 등록 /users POST
전체 사용자 조회 /users GET
개별 사용자의 상세 정보, 주문 내역 /users/{user_id} GET
작동 상태 확인 /users/health_check GET
환영 메시지 /users/welcome GET

과 같은 형식으로 엔드포인트를 구성할 수 있다.

### H2DB 사용 및 연동
데브툴, 롬복, 웹, 유레카 디스커버리 클라이언트를 디펜던시로 사용

메인메소드 위에 @EnableDiscoveryClient 어노테이션을 넣어줘야 작동이 가능하다.

서버 포트는 랜덤포트(0)번으로 하고, 인스턴스 아이디를 ${spring.application.name}:${spring.application.instance.id}:${random.value}}와 같은 식으로 넣어서 다중 인스턴스 실행에 대응할 수 있다.
defaultZone에는 유레카 서버의 주소를 넣어 주어야 한다.

포트가 분리될 경우에는 따로 접두어 없이 /만 있으면 된다.

텍스트를 따로 yml파일로 분리하기 위해, @Value("${application.yml에서 가져오고 싶은 키값}")형식의 어노테이션을 변수 위에 작성하면, 굳이 하드코딩 하지 않아도 된다.

데이터베이스는 H2DB를 간단히 사용. JPA 연동
H2DB를 사용하기 위해서는 디펜던시가 있어야 한다.

application.yml에서 h2설정을 추가한다.

h2:
console:
enabled: true
settings:
web-allow-others: true
path: h2-console //포트넘버 뒤에 이 키워드를 붙이면 h2 웹 UI로 접근이 가능하다.

### 간단한 팁(이미알고있는것도 있지만)
인텔리제이에서 CTRL+N키를 누르면 스프링 프레임워크 관련 메소드 오버라이드, 의존성 주입 등의 작업을 편안하게 처리할 수 있다

스프링에서 객체를 등록하려면 반드시 @Component나 다른 기타 스테레오타입 어노테이션을 넣어줘야 한다.

롬복을 사용하는 경우 @Data를 넣으면 게터와 세터들이 자동으로 추가가 되고, @AllArgsConstructor를 넣으면 모든 인자가 있는 생성자를 추가한 것과 같으며, @NoArgConstructor를 넣으면 인자가 없는 생성자를 추가한 것과 같게 된다.

@Value의 용법은 바로 앞선 강의에서 설명했으니 예외

### Spring Security 연동
스프링 시큐리티

spring security를 의존성 추가
SecurityConfiguration 클래스 생성
SecurityConfiguration 클래스에 @EnableWebSecurity 추가
Authentication -> configure(AuthenticationManagerBuilder auth) 메소드 재정의
Password encode를 위한 BCryptPasswordEncoder 빈정의
Authorization -> configure(HttpSecurity http)메서드를 재정의

BCryptPasswordEncoder 타입이 없다는 오류가 있다면, 오토와이어를 걸어주는데, 없을 게 뻔하므로, 메인에다가 빈을 하나 생성해서 새로운 인스턴스를 리턴하도록 코드를 넣어준다

## Spring Cloud Config
### 개요
Spring Cloud Config이란 분산 시스템에서 서버, 클라이언트 구성에 필요한 설정 정보(app.yml 등)을 외부 시스템에서 관리하는 것이다.

다음과 같은 여러 장점이 있다.
중앙화 된 저장소에서 구성요소 관리 가능하다는 것과,
각 서비스를 다시 빌드하지 않고 바로 적용 가능하다는 것,
애플리케이션 배포 파이프라인을 통해 개발-업데이트(스테이징)-운영 환경에 맞는 구성 정보 사용이 가능하다

프라이빗 또는 퍼블릭 깃 리포에다 저장이 된다
설정에 따라서는 암호화된 저장소나 암호화된 로컬 파일에도 저장할 수 있다.

같은 설정 정보라 하더라도 application-dev.yml과 application.yml로 개발환경과 운영환경을 구분하여 환경에 따라 다른 설정 파일이 배포되도록 할 수 있다.
