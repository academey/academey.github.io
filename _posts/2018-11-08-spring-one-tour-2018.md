---
layout: post
title:  "Spring One Tour 2018"
author: academey
categories: conference
cover: "/assets/spring-one-tour/cover.jpeg"
---
## 000. 느낀점
처음으로 간 스프링 컨퍼런스. 아는 개념보다 모르는 개념이 훨씬 많아 와! 이런 것도 있구나 하면서 들었다. 컨퍼런스는 스스로에게 키워드를 던져줄 수 있는 기회인 것 같다.

이번 컨퍼런스에서 남은 것은 Reactive, Web Flux, Event driven architrcture 이다.

## 0. Introduction

XP는 개발자, 기획자 등등이 소프트웨어의 전체적인 자율권을 가지고 개발하는 방법론.
이외의 것들을 하는 방법론들의 명칭을 만들어가고 있다.
기존의 레거시 프로젝트를 현대화 하는 프로젝트도 하고 있다.
아무쪼록 플랫폼과 툴에 대해 설명하고 오픈 스택에 대해 발표할텐테, 좋은 시간 되고 많은 영감이 되길 바랍니다.


## 1. Reactive Spring with Spring Boot 2.0

### 1. Full Stack Reactive Kotlin!
코틀린은 자바의 다음 단계인 것 같다. 새로운 것을 더욱 빠르게 쓸 수 있다.

Reactive Programming?

`non-blocking, event-driven applications 이다. 몇 개 안되는 쓰레드로 BackPressure 를 주요 재료로 소비자를 압박하지 않도록 하는 것이다.`

자바에서 개별 커넥션들을 보면 쓰레드가 생긴다. 커넥션이 많아지면 쓰레드가 많아지는데 문제는 아무것도 안한다. 쓰레드는 항상 대기 상태가 많아 비효율적이다. 따라서 이걸 뒤집어서 소수의 쓰레드를 사용한다. 예를 들어 노드처럼 하나의 쓰레드를 사용하는데, 문제는 하나 조지면 클난다. 따라서 여러 쓰레드로 대체한다. 소수의 리퀘스트는 동기화가 불가능하다. 소수이기 때문에 빠르게 응답해줘야 한다. 따라서 Reactive에서는 적은 리퀘스트가 안 좋다. 그러나 확장성에 대해서는 미친듯이 열려있다. 결국, 리액티브 프로그래밍은 많은 커넥션에 대해 효율적이게 반응 할 수 있다.

비동기 프로그래밍 들어보신분? 콜백을 사용하는 방법으로 쓰레드를 줄일 수 있었는데, 특정 액션을 통해 콜백을 호출해서 하는 방식이 있었는데 굉장히 복잡해져 콜백 지옥이 생긴다. 따라서 이 부분을 해결하는 실행 모델을 만들고자 한다. 요청에 대해서 Subscribe가 느리다는 것이 문제되지 않게 클라우드에서 처리하는 것을 만들고자 한다. 모든 디바이스를 처리할 수 없기 때문이다.

이게 Reactive Streams 에서 구현이 되고 있다. 우리가 보는 확장성을 보면 서비스와 디비, 디비와 디비 즉 인터페이스간의 인터랙션이 중요하다. 결국 기본적인 specification에 대해서는 합의를 보자 라는 건데, 결론은 간단하다. API가 4개가 있다. Publisher 는 값을 만들고, Subscriber 는 이 값을 소모하고, Subscription 은 서로의 인터랙션, Processor는 그 값을 중간에서 처리한다. 이건 모두 그냥 인터페이스다. 요걸 구현해줘야 한다. 구현하는 것보다 개발된 것들이 있으니 써도 된다. 이런 인터페이스를 어떻게 사용하고 접근할 건지 호환성에 대해 고민하고 Compatibility 킷을 통해 호환성을 관리한다.

RestController RequestMapping 을 보면, 자바에서 메소드 콜링을 할 때 타입드 오브젝트를 돌려 보내고 반복적으로 리터닝이 가능하다. 따라서 다양한 오브젝트를 보내는 게 가능하다. 즉, 리액트 프로그래밍에서는 0~100 을 돌려보낼 수 있다. Publisher 인터페이스를 구현할 때 0,1 또는 플럭스라는 리액티브 컬렉션을 보내고는 한다. 또, Non Blocking IPC 에 모두 적용이 가능하다.

Netflix의 IT Crowd 봐라. 재밌따 ㅋㅋ

자 이제 실습해보자. Spring Initializer 에서 Gradle project에 kotlin으로 시작하겠다. 리엑티브 웹, 몽고디비 사용하겠다. 코틀린은 압축된 자바라고 하면 된다. 세미콜론도 쓰지 않고 등등 뭐가 있다.  코틀린에서는 클래스 컨스트럭터를 class 명에 붙이고, 몽고에 등록하니까 @Document 붙여주고, data class Coffee(@Id val id: String? = null, val name: String = "Any old joe") 뭐 이런식으로 한다.
운영상 얼마나 커피가 소모되는지 알고 싶다. 따라서 상위에 관리하는 클래스를 두자.
class CoffeOrder(val coffeId: String, val whenOrdered: Instant) 이런식으로, 스프링 데이터 안에서 찾기 위해서 스프링 부트에 AutoConfiguration을 해준다. 인터페이스만 있다면 데이터 드라이버 클래스와의 인터랙션을 위해 만들어준다. 따라서 interface CoffeRepo: ReactiveCrudRepository<Coffee, String>을 만들자.
이게 레포 여할을 한다.
쉽게 보기 위해, 컴포넌트를 만들어서 확인하자. 참고로 코틀린에선 메소드가 없고 모든게 function이다. CoffeeRepo 빈을 넣고 싶기 때문에 @Component class DataLoader(private val repo: CoffeRepo) {
  @PostConstruct
  private fun load() = Flux.just("latte", "cappuccino", "espresso").map{ Coffee(name = it) }.flatMap {repo.save(it) };
}

Reactive Application에서는 아무것도 일어나지 않는다. 퍼블리셔가 없으면 아무것도 하지 않는다. 따라서 스프링같은 터미널 앱이 필요하다. Subscribe를 해줘야 한다. 따라서 맨 뒤에 .subscribe { println(it) }을 해야 한다. 그리고 repo.deleteAll()을 쳐줘야 한다. 새로운 레코드가 들어가기 전에 딜리트 된다는 보장이 없다. 그런데 nonBlocking 코드인데 블록킹하면 안되지 않는가? thenMany로 처리하자. 뭔가 비동기 이후 then 같은 과정?

이제 Internal API를 서비스를 사용해서 매핑해보자. @Service class CoffeService(private val repo: CoffeRepo) {
  fun getAllCoffees() = repo.findAll()

  fun getCoffeeById(id: String) = repo.findById(id)
}

그런데, 아직 고객이 없다. 오더가 들어오는 걸 시뮬레이션 해보자. Flux.generate를 사용해보자.
fun getOrdersForCoffee(coffeeId: String) = Flux.interval(Duration.ofSeconds()).onBackPressureDrop.map{CoffeOrder(coffeeId, Instant.now())} 그리고 백프레셔가 있으면 버퍼 사이즈를 지정해서 넣어준다. 나는 subscribe가 원하지 않는 값은 드랍해버리겠다

이제 테스트 해보자. 전체 플럭스는 필요 없고, CoffeeService의 WebFlux만 가져오자.

@RunWith(SpringRUnner::class)
@WebFluxTest(CoffeeService::class)
한 다음, @mockBean로 CoffeeRepo 만들고, coffee를 만들자. 그리고 Mockito.`when`.(repo.findAll()).thenReturn(Flux.just(coffe1, coffee2)) // 참고로 느낌 표 2개는 null 체크 안하고 넘기는 거다. 프로덕션에서는 쓰면 안 된다.

Interactive환경(여러 리퀘스트에 반응) 에서는 테스트 하기 편하다. 하나 가고 하나 뱉으면 되니까, 그런데 리액티브에서는 어떻게 해야되는가. StepVerifier.create(service.getAllCoffees()).expectNext(coffee1).expectNext(coffee2).verifyComplete()

이제 시간을 Compression을 해보자. 버츄얼 타임을 이용해서 파파팍 보내보자.
StepVerifier.withVurtualTime { service.getOrdersForCoffee(coffee2.id!!).take(10)}.thenAwait(Duration.ofHours(10))
Flux가 데이터를 10개씩만 받고, 10시간 기다린다.

DB Access 와 Connection을 리액티브하게 실습해봤다. CoffeeController의 RequestMapping 하고 Service를 인젝트해보자. 이런 방법으로 블록킹 되지 않고 Reactive 코딩이 가능하다.

https://github.com/mkheck/FSR 들어가서 좀 더 봐보면 좋겠다.

## 2. Cloud-Native Spring
교육 비디오 많다. Josh Long -> Cloud Native Java

Microservice는 분산되어 있어서 불안정하다.

리액티브 프로그래밍은 소프트웨어를 async I/O를 쓰고 쓰레드를 재활용해서 가져가고자 한다. 그래서 쓰레드를 재사용 I/O를 목적으로 하자. 그러나 피보나치, 크립토 등등은 처리를 해야 해서 불가능하고, 오늘 우리가 할 것은 소프트웨어를 써보자.

### 1. R2DB
우리는 R2DB 를 써보자. r2dbc-postgresql 씀. 데이터 클래스 다 만들어보자. @AllArgsController @NoArgsConstructor @Data class Reservation { @Id private Integer id; }

interface ReservationRepository extends ReactiveCrud

스프링엔 AutoConfiguration이 없어서 직접 써야 한다.
일반적인 Configuration 만들고 붙여서 컴포넌트 만들어서 넣는다.

@Component
@Log4j2
class SampleDataInitializer{
  private final ReservationRepository reservationRepository; SampleDataInitializer(ReservationRepository reservationREpository) {
  }

  @EventListner(ApplicationREadyEvent.class)
  public void go() throws Exception {
    this.reservationRepository.findAll().flatMap( r->this.reservationRepository.delete(r)).thenMany(Flux.just("Josh")).map(name -> new Reservation(null, name))
  }
})}
키를 제한해서 리퀘스트 수를 조정할 수 있다. 이후에는 Spring Security 를 보면 된다. authentication을 위한 서비스를 붙이자.

@Bean MapReactiveUserDetailService authentication(){
  UserDetails jlong = User.username("jlong").password("pw").roles("USER").build();
  return new MapREactiveUserDetailService(jlong);
}
@Bean SecutiryWebFilterChain authorizetion(ServerHttpSecurity http {
  http.httpBasic();
  http.authorizeExchange()
  .pathMatchers
})

컨텍스트 오브젝트에 파이프라인에 연결 시켜서 ~~한다.
이걸 무제한 리퀘스트 날려보면, 5~7회 때는 리퀘스트를 막게 된다.
스트림에서, 퍼블리셔에 접근하기 전까지는 아무런 일도 일어나지 않는다.

## 3. Spring Cloud Gateway
### 1. What is API gateway?
MVC 기반의 애플리케이션과 마이크로서비스가 서로 연결할 때, 직접 요청하거나 도메인을 붙여 한 경우가 많았다. 그런데 문제점이 많았다. 클라이언트가 다양해짐에 따라 요청을 하나로 받아서 분리해주는 Gateway의 필요성이 생겼다. 리버스 프록싱의 도구.

시대가 마이크로서비스를 부르면서, 단순한 기능 외에 분산기법을 사용해야 하는데, 모든 서비스를 왔다갔다하는 도구인 만큼, 여러 기능을 소화해야 한다. 버전 관리성, 레이턴시 등등 LB의 역할을 한다. SAAS

결국 이 게이트웨이 패턴을 앱 앞 쪽에 넣고, 또 하나의 폼을 추가하지 않고 디비를 작은 서비스를 두고 쓴다던지, 백엔드가 잘 깨질 것 같으면 쓰는 패턴이다.

앱으로의 직접 접근이 안되게 하고싶고, 인증을 게이트웨이에서 하게 한다든지 또 마이크로서비스로 전환하고 싶다면 요청을 분산해서 보내는 경우이다.

### 2. Cloud event driven architrcture

Event storming 이란? 협업 기법이다. 비지니스 문제를 발견하는 현업의 이벤트와 관련. 비지니스 도메인을 이해하는 것.

도메인 전문가와 개발자들을 모아서 도메인에서 일어나는 주요 이벤트들을 꺼낸다. 그러면, 이 이벤트들의 트리거는 무엇인가?
실행하려는 커맨드(사용자 혹은 시스템에 의해), 지연시간(정기적으로 실행), 다른 이벤트에 의해(인증서 발급 기능)

이 과정 중에 체크해야하는게 있다. 잔고가 남았는지, 한도가 얼마나 남았는지 등의 제약사항들을 정하며 시스템을 구축한다. 이 과정을 하며 규칙 통일이 진행되며 확립된다. 또한 이벤트 세션을 통해 각각의 다른 관심 분야들의 사람들의 전문가들이 가지고 있던 다른 정의가 서로 합쳐지고, 차이를 인식하게된다. 결국 하나의 단어가 각 컨텍스트마다 사용되는 개념을 정의한다. Loosely coupled model이 된다.
세번째는 이벤트와 변수들이 연관 관계를 가지게 된다. 이런 것들을 한 곳에 취합해놈으로써 일관성을 가지게 된다.

결국 개발에서는 메시지가 중요하다.

Command -> Aggregate, external sysyem -> domain event 를 통해 리드 모델을 만들고 UI를 만든다.

결국, 기술적 요소를 제외하고 필요한 프로세스를 모두 모아 협업이 가능하다.

이걸 이제 모델에 적용해보자.
메소드가 커맨드 - 블루노트
분기처리 invariants - 옐로노트 라고 하면 메소드 이후 상태변화를 도메인 이벤트를 의미한다.

이제 파리미터를 이벤트로 보낸다. 해당 이벤트의 정보를 저장하고 넘기고 싶기 때문이다.

모든 이벤트를 저장한다.

로딩 하는 부분은 좀 더 어렵다. 그런데 왜 이벤트를 저장하느냐는 질문에 대해 대답해주겠다.

우선 로드 구현할 때, 마구 만들어왔던 이벤트들을 모듀 불러서 각각 크레딕카드를 해당하는 uuid로 만든 다음 생성된 이벤트의 인스턴스 케이스에 따라 핸들링을 해준다.

즉, 크레딧 카드를 저장하는 게 아니라 크레딧 카드의 상태를 저장함으로써 로직를 모아둔다.

왜? 그냥 따로 써도 되지 않느냐. 왜 이벤트 소싱을 하느냐
1. Audit이 가능하다. 즉 단일 사실이 모여지기 때문에 한 곳에서 관리할 수 있음

2. 익셉션이 어디에서 일어나는지 스테이트가 옮겨가있어서 모른다. 그래서 다시 옛날의 스테이트로 찾아가서 확인할 수 있다.

3. 이벤트의 다양한 결과를 얻을 수 있다. 해당 크레딧 카드의 아키텍쳐 컴퍼넌트가 사기를 감지하는 놈만 끼워서 근접한 타임스탬프 이벤트에 대해 조사할 수 있다. 즉, 메소드의 실행 정보를 모아서 확인할 수 있다.
이와 같은 특정 스테이트를 보면서 디버깅 할 수 있다.
물론 단점도 있다. 너무 많은 스테이트가 있으면서 문제, 잘못 만든 이벤트가 있으면 변경이 안되므로 문제, compensation 부분 환불 등의 이벤트 픽스를 염두해야하고, 써야되는 코드와 사고의 전환이 필요하다.

결국 변경하고 싶은 부분이 있으면, 채널과
플럭스를 써서 두개이상의 값이 있으면 보내고, withdrawal 로 subscribe하는 브라우저마다 messanger를 핸들링해서 도매인 이벤트로 컨버트해준다음, 메시지를 보낸다.
이벤트 타입에 따라 브라우저가 다음 이벤트를 실행시키고, 다음것을 돌리기 위해 unsubscribe 하고 등등 한다.

Github ddd-by-examples 드가라

## 4.Spring, Functions, Serverless and You

지금까지 인프라의 많은 변화들이 있다.

Iaas 부터 시작하자. 인프라로 서비스하는 것. 효율을 위해 여러 앱을 키워놓는다. 그런데 하나의 앱이 터지면 다 터진다. 변경사항을 조율하는 것이 힘들다.
이제는 역전이 되었다. 인프라가 넘친다. 인프라를 가축으로 취급하자. 인간은 반복하기 힘들다. 인프라에서 일관성이 필요하다. 자동화를 통해 consistency를 유지해야한다. 스트립트를 통해 리드 타임을 줄일 수 있다. 서버 올리는데 며칠 만에 수많은 인스턴스 올릴 수 있다.
우리가 컨테이너를 사용하는데, 원하는 모든 걸 넣고 쉽게 옮긴다. 운영에 필요한 세팅 등등 모두 다. 이 접근법의 장점은 환경을 분리해서 구현이 가능하다 그러나 도커에 올리는 건 굉장히 책임이 높다. 개인정보를 다 올려서 조지는 경우도 많다.
개발자로서 해야하는것들이 뭔지 알아야한다. 컨테이너, 플랫 폼, 서버리스에 올리다보면 개발자가
컨테이너, 애플리케이션, functions 을 올려줘야 한다.

만약 레이턴시가 중요하다면 서버리스를 쓰면 안 된다.
Function as a service 콜드 스타트를 막기 위해 핑을 날리는 건 애초에 목적에 저해된다. 많은 프로바이더들이 다이나믹 로딩을 한다. 그러므로 결국 트레이드오프를 해야한다.

## 5. 스프링 부트

개발자는 운영에 시간을 많이 보낸다. 따라서 데브옵스를 잘해야한다.

개발 기획 디자인 팀 다 나누는 거 말고, 클라우드 플랫폼 팀을 만들어서 빌드 툴을 만들다던지 배포 파일 파이프라인을 만들면서 도구를 만들어나갔다.

왜 로그인 써버 망가진다고 스트리밍이 꺼져?

이제, 클라우드의 장점을 이용하자. 카오스 엔지니어링이다. 그냥 아무거나 끄고, 서킷을 죽인다. 디비를 꺼버리는데도 문제가 없는지를 파악하는 기술이다. Api gateway로 모니터링하면서 무슨 일이 일어나는지 보면서 확인해야한다.

## 6. CD with spinnaker & Kibernates

Devops 하면서 이 쪽 분야 온 사람이다.

팟에다가 컨테이너를 올려서 관리하자. 쉘 스크립트에서 단순하게 앱 컨테이너 관리 가능하다.

서비스로 네트워크를 오픈하고 싶다. 쿱커틀 익스포즈 하면 k get all 했을 때 해당 서비스가 생기고 아이피가 없다. 아직 만들어지고 있다. 쿠버니티스의 서비스는 메타 데이터로 팟을 연결하고 서비스 디스커버리를 발견하고 만든다. 스태틱 포트를 뽑아서 어플리캐이션에 접근할 수 있다.

인그레스로 경로에 대해 제어할 수 있다. 또한 권한과 lb도 추가하고 등등..
