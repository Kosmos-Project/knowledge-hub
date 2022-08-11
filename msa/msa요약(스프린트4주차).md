# MSA 요약(스프린트 4주차)

## 1. What is MSA?

- application 개발을 위한 방법론 중 하나
- application은 각 담당 영역을 가지는 소규모의 독립적인 구성요소(component)로 이루어진다.
     - 여기서 담당 영역은 서비스라고 생각하자.<br/>

- MSA 방식으로 개발된 application의 경우, 단일 사용자가 application에 요청할 때, 내부적으로 component들끼리 필요한 데이터를 주고받은 후, 최종적으로 그 사용자에게 응답을 반환할 수 있다.
- Google cloud에서는 MSA를 아래와 같이 정의한다.<br/>
> 마이크로서비스 아키텍처는 애플리케이션이 서비스 모음으로 개발되는 애플리케이션 아키텍처의 한 유형입니다. 또한 마이크로서비스 아키텍처 다이어그램과 서비스를 독립적으로 개발, 배포, 유지관리할 수 있는 프레임워크를 제공합니다.
<br/>
- MSA로 개발된 application은 보통 cloud 환경에서 실행된다.
  - 독립적인 구성요소가 많다는 것은 그만큼 신경써야할 부분이 많다.(e.g. 각 구성요소에 대해 build, deploy, health check를 신경써야함)
  - 위의 문제를 cloud에서 제공되는 서비스(AWS의 CodeStar)를 활용하면 생산성을 높일 수 있다.
<br/>

참고
- https://cloud.google.com/learn/what-is-microservices-architecture

<br/>

## 2. Why MSA?
MSA를 활용하면 다음의 이점을 갖고 있다.
- 서비스 단위의 독립적 배포
 - 서비스 단위로 component가 분리되기 때문에, 다른 component에 영향을 받지않고 독립적으로 배포가 가능하다. <br/>
- 장애 허용
  - 이 부분은 cloud 환경에서 실행될 때, 실현할 수 있다. 예를들어 pod가 죽어도, auto scaling이 되기 때문에 pod의 수가 보장된다. 다시말해, 장애가 나도 서비스 제공에는 지장이 없다.<br/>
- 자유로운 기술스택
    - 어떤 component는 spring으로 다른 component는 fast api로 개발 할 수 있다. 다시말해, 다양한 기술스택을 고를 수 있다.<br/>

Azure문서에 따르면 MSA는 주로 도메인이 복잡하고, 업데이트가 빈번하게 발생할 때, 사용된다고한다. 여기서 업데이트는 데이터에 대한 스키마 업데이트도 포함한다.

참고
- https://www.logicmonitor.com/blog/what-are-microservices#what
- http://www.iteyes.co.kr/index.php/msa-micro-service-architecture/

## 3. MSA Consideration
MSA로 application을 개발할 때, 고려사항이 존재한다.
- 복잡성
    - 다수의 component들이 존재하기 때문에 workflow가 복잡하여, 전체적으로 어려운 느낌을 준다.<br/>
- 개발 및 테스트
    - component가 다른 component에 의존한다면(e.g. API 호출), component를 어떻게 테스트할지 고민을 해야한다.
    - 다른 경우로 component API spec이 달라지면, 해당 API를 호출하고 있는 component들도 변경해야한다. 따라서 설계단계에서 확장성을 고려하여 API design을 진행해야한다.<br/>
- 유지보수
    - 자유로운 기술스택 선정에 따른 부작용이다. 만약 fast api로 개발한 담당자가 1명인데, 그 담당자가 퇴사한다면..?ㅠㅠ<br/>
- 네트워크
    - component들사이에서 통신이 빈번하게 일어나기 때문에 네트워크 상태는 매우 중요하다. component들이 연쇄적으로 의존하는 것을 막아야한다.(e.g. c1 -> c2 -> c3)
    