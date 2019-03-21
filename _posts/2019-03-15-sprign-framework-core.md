---
layout: post
title: "스프링 프레임워크 핵심 기술 정리 (백기선님 인프런 강좌)"
author: academey
categories: Spring
---

# 1. IoC 컨테이너 1부 : 스프링 IoC 컨테이너와 빈

Inversion of Control : 의존 관계 주입(Dependency Injection) 이라고도 하며, 어떤 객체가 사용하는 의존 객체를 직접 만들어 사용하는 게 아니라, 주입 받아 사용하는 방법을 말 함.

BookService -> new BookRepository (X)
BookService(bookRepository) (O)

이렇게 하지 않고, IoC 컨테이너를 사용하는 이유는 의존 받기 쉬운 형태로 되어 있기 때문이다.

초기에는 IoC 컨테이너, 빈을 넣어두는 곳, 에 빈을 모두 다 정의해놓고 가져오는 방식이었다. Xml에 넣다가 이제는 어노테이션을 쓰기 시작했다.

스프링 IoC 컨테이너

- BeanFactory
  - getBean
- 애플리케이션 컴포넌트의 중앙 저장소
- 빈 설정 소스에서 빈 정의를 읽은 다음 빈을 구성하고 제공한다.

빈

- 스프링 IoC 컨테이너가 관리하는 객체
- 의존성 관리
- 스코프
  - 싱글톤 : 하나 (디폴트) // 무거운 것들을 하나로만 관리.
  - 프로포토타입 : 매번 다른 객체
- 라이프사이클 인터페이스
  - @PostConstruct
- 테스트 시 주입하는 빈을 갈아챌 수 있으니 더 용이함. when(bookRepository.save(book)).thenReturn(book);

ApplicationContext

- BeanFactory
- 메시지 소스 처리 기능 (i18n)
- 이벤트 발행 기능
- 리로스 로딩 기능

빈으로 관리하려면 @Repository 로 등록을 해줘야 한다.
의존성 주입을 해야 하면 빈으로 등록해줘야 하며, 빈의 스코프 때문이기도 하다.
북서비스라는 인스턴스는 오직 하나만 필요하다.

# 2. IoC 컨테이너 2부: ApplicationContext와 다양한 빈 설정 방법

고전적으로 Application.xml 만들어서 빈 주입해보자. <bean id=“bookService”><property name=“bookRepository”> </bean> ….

요렇게 일일이 정의하고 주입하기 너무 번거롭다! 그래서 나온 게 컴포넌트 스캔이다.
특정 패키지 밑에 있는 어노테이션을 찾아서 다 빈으로 등록시키자는 것이다

그런데 이 방법 말고, 빈 설정 파일 자체를 xml 말고 자바로 쓰고 싶어서 @Configuration ApplicationConfig 을 사용하기 시작했다. 이 설정을 ApplicationContext context = new AnnotationConfigApplicationContext(ApplicationContext.class) 처럼 가져와서 컨텍스트를 사용했다.

그리고 아예 의존성 주입을 설정 하지 않고, Autowired 를 사용하면 된다. 이 방법은 Setter 가 항상 필요하다. 더 쉽게 하려면 그냥 @ComponentScan을 이용해서 알아서 빈을 등록시킨다.

# 3. IoC 컨테이너 3부: @Autowire

필요한 의존 객체의 “타입”에 해당하는 빈을 찾아 주입한다.

@Autowired

- Required : 기본값이 true, 없으면 구동 실패

사용할 수 있는 위치

- 생성자
- 세터 : BookService 는 만들 수 있지만, Autowired 가 붙어 있으므로 주입하려고 시도 함. 주입 실패하면 빌드 실패.
- 필드

경우의 수

- 해당 타입의 빈이 없는 경우 -> 에러
- 해당 타입의 빈이 한 개인 경우 -> 정상
- 해당 타입의 빈이 여러 개인 경우
  - 빈 이름으로 정의
  - 쓰고 싶은 클래스에 @Primary 붙이기
  - 해당 타입의 빈 모두 주입 받기 List<BookRepository>
  - @Qualifier(Bean 이름 넣기) keesunBookRepository

동작 원리

- 빈 라이프사이클 중 BeanPostProcessor 라는 게 있는데, 빈이 생성되거나 생성되기 전에 작업을 넣을 수 있다. @PostConstruct 등을 사용한다던가.
- 이 중 AutowiredAnnotationBeanPostProceesor 가 initalization 이전에 빈을 주입 시킨다. 그리고 @PostConstruct 에서 불러온 빈들을 사용할 수 있게 되는 것이다.
- 빈 팩토리의 동작 순서는 여기서 확인하자. https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/BeanFactory.html

# 4. IoC 컨테이너 4부: @Component 와 컴포넌트 스캔

컴포넌트 스캔 주요 기능

- 스캔 위치 설정 : 컴포넌트 스캔 하위의 패키지들임.
- 필터 : 어떤 어노테이션을 스캔할지 안 할지
  - AutoConfiguration 같은 건 제외하고 있다. 스캔 시 포함 안 시킬 것들을 거르자.

컴포넌트 어노테이션을 들고 있으면 빈으로 등록이 되는데, 싱글톤 스코프 빈은 구동 시에 등록함. 그래서 빈 많으면 오래 걸리지만, 구동시간에만 걸리기 때문에 다른 방법으로 하고자 한다면, 펑션을 사용해서 빈을 등록할 수 있다.

App.addInitializers((ApicationContextInitailizer<GenericApplicationContext>) cox -> {
If () { Cox.registerBean(MyService.class);}
});

애플리케이션 구동 시 성능 상 이점이 있으며, 조건문에 따라 빈을 등록할 수 있는 둥 여러 방법이 있음.

@Component (다 컴포넌트임)

- @Repository
- @Service
- @Controller
- @Configuration

동작 원리

- ConfigurationClassPostProcessor 와 BeanFactoryPostProcessor 에 의해 스캐닝 됨

# 5. IoC 컨테이너 5부: 빈의 스코프

앵간하면 싱글톤으로 다 쓰는데, 요청마다 다른 인스턴스를 사용하고 싶을 수 있다. 그러면 @Component @Scope(“prototype”) 를 붙이면 된다. 그러면 요청을 날릴 때마다 새로운 인스턴스가 간다.

그런데 문제가 있다. 만약 싱글톤이 프로토타입을 참조하고 있다면,
싱글톤 객체를 생성할 때 프로토타입의 객체를 박아놓는다. 이 때 프로퍼티가 변경되지 않는다. 따라서 옛날 프로토 객체를 계속 보고 있는 것이다.

이걸 간단하게 쓰려면, @Scope(value=“prototype”, proxyMode = ScopedProxyMode.TARGET_CLASS) 을 씌우면 된다. 직접 참조하면 안 되고, 프록시를 거쳐야지만 프로토를 새롭게 할당해준다. 이 프록시 빈을 주입시켜주는 것이다.

이 방법이 싫다면,
ObjectProvider<Proto> proto; 와 같이 정의해주면 된다.

싱글톤은 항상 쓰레드 세이프가 아니다. A 쓰레드와 B 쓰레드 모두 같은 자원(객체)를 공유하기 때문에 싱글톤의 프로퍼티는 변경될 가능성이 농후하다. 따라서 쓰레드 세이프 한 방식을 고민해야 한다.

스코프

- 싱글톤 (디폴트)
- 프로토타입
  - Request
  - Session
  - WebSocket
  - ...

프로토타입 빈이 싱글톤 빈을 참조하면?

- 아무 문제 없음

싱글톤 빈이 프로토타입 빈을 참조하면?

- 프로토타입 빈이 업데이트가 안되네?
- 업데이트 하려면
  - scoped-proxy
  - Object-Provider
  - Provider ( 표준)

싱글톤 객체 사용시 주의할점

- 프로퍼티가 공유
- ApplicationContext 초기 구동시 인스턴스 생성

# 6. Environment 1부. 프로파일

프로파일과 프로퍼티를 다루는 인터페이스

applicationContext는 빈 팩토리 기능만 하는 것이 아니다. EnvironmentCapable 이라는 인터페이스가 있는데, 그 중 하나가 프로파일이다.

ctx.getEnvironment(); 를 통해 가져온 다음, environment.getActiveProfiles() 를 통해 현재의 프로파일을 볼 수 있다. 현재는 default 프로파일.
빈으로 등록한 객체들은 다 명시해주지 않으면 default 프로파일에 들어가 있다.

@Configuration @Profile(“test”) public class TestConfiguration { @Bean public TestBookRepository {} } 이런 식으로 등록하면, 저 빈은 테스트 프로파일에만 등록이 된 것이다. 그래서 일반 앱에서 TestBookRepository 를 주입 받으면 에러난다.
IDE에서 특정 프로파일을 지정해주려면, Active profiles 설정에서 test 를 적어주면 된다. 참고로, 디폴트는 항상 적용되는 것이고, 테스트가 오버라이드 하는 방식이다. 혹은 VM options 에 -Dspring.profiles.actdive=“test” 처럼 프로퍼티를 설정해주면 된다.

@Profile(“!prod”) 이런 방식으로 쓸 수도 있는데, 그러면 prod 가 아닌 프로파일들에는 등록이 되므로 test 에 해당 빈이 있다. ! 혹은 & 혹은 | 이 사용가능하다. prod | test 이런 느낌으로..

프로파일

- 빈들의 그룹
- Environment 의 역할은 활성화할 프로파일 확인 및 설정

프로파일 유즈 케이스

- 테스트 환경에서는 A라는 빈을, 배포 환경에서는 B라는 빈을 사용하고 싶다.
- 이 빈은 모니터링 용도니까 테스트할 때는 필요 없고 배포할 때만 등록되면 좋겠다.

프로파일 정의하기

- 클래스에 정의
  - @Configuration @Profile(“test”)
  - @Component @Profile(“test”)
- 메소드에 정의
  - @Bean @Profile(“test”)

6. Environment 2부. 프로퍼티
   애플리케이션에 설정한 프로퍼티 값.
   위의 저 VMOptions 에서 넘긴 것 처럼 -Dapp.name=spring5 이렇게 넘길 수 있다.
   이 프로퍼티에 접근하려면 ctx.getEnvironemnt().getProperty(“app.name”) 을 통해 접근할 수 있다.

조금 더 체계적으로 프로퍼티를 사용하려면 파일을 쓸수 있다.
app.properties 를 만들고, @PropertySource(“classpath:/app.properties”) 를 해서 넣어주거나,
@Value(“\${app.name}”) 을 붙여서 프로퍼티의 값을 붙이거나 등등 기능이 있으니, 이건 스프링 부트에서 참고하자.

# 7. IoC 컨테이너 7부 : MessageSource

국제화 (i18n) 기능을 제공하는 인터페이스. 메시지를 다국화시키자! 이걸 ApplicationContext 에서 제어할 수 있다.
ApplicationContext의 인터페이스 중 MessageSource 가 있으니 바로 주입받은 다음 기능을 사용할 수 있다.
그 전에, 메시지를 만들어보자.

Resources/messages.properties 와 messages_ko_KR.properties 를 만들어보자.
그리고 greeting=Hello, {0} 와 greeting=안녕, {0} 을 넣어주자.

그리고 messageSource.getMessage(“greeting”, new String[]{“keesun”}, Locale.KOREA);
messageSource.getMessage(“greeting”, new String[]{“keesun”}, Locale.getDefault());
를 실행시켜보면, 메시지를 뱉어준다. 메시지 소스는 이미 스프링 부트에 빈으로 등록이 되어있다.

조금 더 재미있는 기능은 릴로딩 기능이 있다. 아래와 같이 해놓고, 계속 어플리케이션 러너에서 찍는다.
이 상태에서 프로퍼티를 변경해도 알아서 업데이트 시킨다. 빌드하면 계속 가져간다.
@Bean
Public MessageSource messageSource() {
Var messageSource = new ReloadalbeResourceBundleMessageSource();
messageSource.setBasename(“classpath:/messages”);
messageSource.setDefaultEncoding(“UTF-8”);
messageSource.setCacheSeconds(3);
}

# 8. IoC 컨테이너 8부: ApplicationEventPublisher

또 다른 인터페이스, ApplicationEventPublisher 를 봐보자. 옵져버 패턴의 구현체로 이벤트 기반의 프로그래밍 시 유용하다.

ApplicationContext extends ApplicationEventPublisher

- publishEvent(ApplicationEvent event)

이벤트 만들기

- ApplicationEvent 상속
- 스프링 4.2 부터는 이 클래스를 상속받지 않아도 이벤트로 사용 가능하다.

이벤트 발생

- ApplicationEventPublisher.publishEvent();

이벤트 처리 방법

- ApplicationListener<이벤트> 를 구현한 클래스를 만들어서 빈으로 등록하기.
- @EventListener 를 사용해서 빈의 메소드에 등록해 사용할 수도 있다.
- 기본적으로는 동일한 쓰레드에서 Synchronized로 실행된다. 실행 순서는 정해져있지 않다.
- 순서를 정하고 싶다면 @Order 를 사용한다.
- 비동기로 실행하고 싶으면 @Async 를 붙이자. 대신, 이 경우 @EnableAsync 를 메인앱에 붙여줘야 동작한다. (별도의 쓰레드로 돈다)

스프링이 제공하는 기본 이벤트

- ContextRefreshedEvent : ApplicationContext를 초기화 했거나 리프레시 했을 때 발생.
- ContextStartedEvent : ApplicationContext 를 start()하여 라이프사이클 빈들이 시작 신호를 받은 시점에 발생
- ContextStoppedEvent : ApplicationContext를 stop() 하여 라이프사이크 빈들이 정지 신호를 받은 시점에 발생
- ContextClosedEvent : ApplicationContext를 close()하여 싱글톤 빈 소멸되는 시점에 발생
- RequestHandledEvent : HTTP 요청을 처리했을 때 발생

ApplicationEvent 를 상속받는 이벤트 클래스를 만들어보자. 내가 넘기고 싶은 데이터를 설정할 수도 있다.
Public class MyEvent extends ApplicationEvent {
Public MyEvent(Object Source, int data) {
super(source);
this.data = data;
}  
}

그리고 @Autowired ApplicationEventPublihser publisheEvent; 를 가지고 와서 publishEvent.publishEvent(new MyEvent(this, 100);) 요렇게 넘기면,
이벤트가 던져진 것인데, 그러면 이 이벤트를 수신하는 클래스도 만들어야 한다.

@Component public class MyEventHandler implements ApplicationListener<MyEvent> {
@Override
Public void onApplicationEvent(MyEvent event) {}
}

로 구현해야 하는데, 스프링 4.2 이후부터는 Event 에 ApplicationEvent 를 상속받지 않아도 된다. 비 침투성을 가장 살린 것이다. 스프링의 문법이 안 들어가려고 노력 한다.
핸들러도 마찬가지로 implements 하지 않아도 된다. 대신 빈으로 항상 등록이 되어야 한다. 그리고 이벤트를 받는 메소드 위에 @EventListener 를 붙여줘야 한다.
메소드 이름은 맘대로 바꿔도 된다.@EventListener public void(MyEvent event) 처럼 말이다.

만약 다른 클래스의 다른 메소드도 @EventListenre public void(MyEvent event) 가 구현되어 있다면? Publish 시에 두 메소드 다 실행이 된다.
기본적으로는 순차적으로 실행이 된다. 즉, 동일한 쓰레드에서 동기적으로 A가 끝나고 B가 실행이 된다. 순서는 안 정해져있다.

순서를 정하고 싶다면 @Order 라는 어노테이션을 붙여서 사용하자. 비 동기를 쓰고 싶으면 @Async 를 붙이자. 대신 매인 엡에 @EnableAsync 붙여줘야 함.

# 9. IoC 컨테이너 9부: ResourceLoader

리소스를 읽어오는 기능을 제공하는 인터페이스

ApplicationContext extends ResourceLoader

리소스 읽어오기

- 파일 시스템에서 읽어오기
- 클래스패스에서 읽어오기
- URL로 읽어오기
- 상대/절대 경로로 읽어오기

@Autowired ResourceLoader resourceLoader;
resourceLoader.getResource(“classpath:test.txt”);
resource.exists(); // false

# 10. Resource 추상화

org.springframewokr.core.io.Resource

특징

- java.net.URL 을 Resource 라는 클래스로 추상화 시켰다.
- 스프링 내부에서 많이 사용하는 인터페이스.

추상화 한 이유

- 클래스패스 기준으로 리소스 읽어오는 기능 부재
- ServletContext를 기준으로 상대 경로를 읽어오는 기능 부재
- 새로운 핸들러를 등록하여 특별한 URL 접미사를 만들어 사용할 수는 있지만 구현이 복잡하고 편의성 메소드가 부조갛다.

인터페이스

- 상속 받은 인터페이스
- 주요 메소드
  - getInputStream()
  - exist()
  - isOpen()
  - getDescription : 전체 경로 포함한 파일 이름 또는 실제 URL

구현체

- UrlResource : java.net.URL
- ClassPathResource : 지원하는 접두어 classpath
- FileSystemResource
- ServletContextResource : 웹 어플리케이션 루트에서 상대 경로로 리소스 찾는다.

리소스 읽어오기

- Resource 타입은 location 문자열과 ApplicationContext 의 타입에 따라 결정 된다.
  - FileSystemXmlApplicationContext 면, FileSystemResource
  - ClassPathXmlApplicationContext 면, ClassPathResource
  - WebApplcationContext -> ServletContextResource 로 가져온다.
- ApplicationContext의 타입에 상관없이 강제하려면 java.net.URL 접두어(+classpath:)중 하나를 사용할 수 있다.
  - Classpath:me/whiteship/config.xml -> ClassPathResource
  - File:///some/resource -> FileSystemResource

거의 다 WebApplicationContext 라서 서블릿컨텍스트로 찾겠지만, 다들 잘 모르니까 그냥 항상 접두어를 써서 명시적으로 프로그래밍하자.
가져온 리소스로더들의 클래스를 보면, WebServerApplicationContext고, 프리픽스를 써서 불러온 리소스의 클래스를 보면 ClassPathResource다.
만약 프리픽스를 쓰지 않으면 기본 classpath 는 설정이 안 되어 있기 때문에 위치를 찾지 못해 에러가 발생한다.

# 11. Validation 추상화

org.springframewokr.validation.Validator
https://beanvalidation.org/

애플리케이션에서 사용하는 객체 검증용 인터페이스
@NotNull, @Email, @Size …. 등 빈의 데이터를 검증시킬 수 있음.

Validator 를 구현하려면 supports(지원하는 클래스인지) 와 validate(에러가 없는지)라는 함수를 구현해야 한다.

Event 라는 클래스가 있다고 하면, 해당 객체를 검증하는 EventValidator implements Validator 클래스를 만들어보자.
그리고 ValidationUtils 를 이용해 검증하면 된다. ValidatonUtils.rejectIfEmptyOrWthiespace(errors, {필드명}, {에러코드}, {디폴트메시지});
에러코드는 위에서 썼던 메시지 소스를 이용해서 보낼 수도 있다. 에러코드를 키값으로 찾는다. 디폴트 메시지는 에러 코드로 못찾으면 보여주는 메시지다.

그런데, 스프링이 알아서 더 넣어준다.
만약 ValidatonUtils.rejectIfEmptyOrWthiespace(errors, “title", "notempty", {디폴트메시지});
로 했다면 추가적으로 “notempty.title", “nonempty.event.title” 등이 추가된다.

혹은 에러를 직접 if 문으로 찾아서 errors.reject 함수를 써서 담을 수도 있다.

이런식의 validation 은 옛날 방식이다. 요즘은 LocalValidatorFactoryBean이라는 게 자동으로 등록이 되어있다.
따라서 @Autowired Validator validator; 를 받고, EventValidator 를 만들 필요가 없으며 validator.validate(event, errors); 를 하면 된다.

그런데, 뭐가 에러인지 어떻게 아느냐? 그건 Event 클래스에 검증하는 어노테이션을 박아주면 된다. @NotEmpty @Size(min = 0) String title;

# 12. 데이터 바인딩 추상화 : PropertyEditor

데이터 바인딩

- 기술적 관점 : 프로퍼티의 값을 타겟 객체에 설정하는 것을 의미한다.
- 사용자 관점 : 사용자 입력 값을 애플리케이션 도메인 모델에 동적으로 변환해 넣어주는 기능
- 주로 문자열을 입력할텐데, 그 데이터를 객체의 인트, 데이트, 불린 등등으로 변환해서 넣어주는 기능을 의미한다. 심지어 Event, Book 같은 도메인 타입으로 변환해서 넣어주기도 한다.

org.springframework.validation.DataBinder 라는 인터페이스가 있다. 요걸 써서 해보자.

이벤트라는 클래스가 있고, 컨트롤러에서 받아서 바인딩해본다고 하자.
@GetMapping(“/event/{event}”)
public String getEvent(@PathVariable Event event) {
Return event.getId().toString();
}

이렇게 하고 /event/1 로 요청을 보내면, 에러가 난다. String to Event 맵핑이 안되었으니까!
따라서 class EventEditor implements PropertyEditorSupport 를 만든 다음에 getAsText와 setAsText를 구현해보자.

- getAsText : 객체 -> 텍스트
  - Return (Event)getValue().getId().toString();
- setAsText : 텍스트 -> 객체
  - 들어온 문자열을 Event 로 만들면 된다.
  - 따라서 setValue(new Event(Integer.parseInt(text)); 를 해주면 된다.

그런데, 얘네들의 getValue 와 setValue 같은 멤버변수들이 쓰레드 세이프하지 않다. 상태를 저장하고 있다. 따라서 여러 쓰레드에 공유해서(빈으로) 쓰면 망한다.
PropertyEditor 는 슬라이드 스코프의 빈으로(한 쓰레드에서만 쓰는) 써야만 한다.

빈으로 쓰지도 못하고, 구현도 귀찮고, Object와 스트링으로만 변환할 수 있고 등등 얘는 안쓰는게 좋다.

# 13. 데이터 바인딩 추상화 : Converter 와 Formatter

문자열 -> 객체 로밖에 바인딩이 안 되며, 상태 정보를 들고 있는 단점을 없앤 클래스다.

StringToEventConverter implements Converter<String, Event> 라는 클래스를 만들고 convert 를 오버라이드 한다.

이제 얘네의 상태값을 싱글톤으로 유지하기 위해서는 ConvertRegistry 에 등록해줘야 하는데, 이걸 직접 등록하지는 않고, WebConfig implements WebMvcConfigurer 에다가 등록을 해주면 된다. addFormatters 라는 메소드가 있으니 요거 오버라이드 한 다음에, registry.addConverter 를 해주면된다.

Converter 보다 조금 더 웹 인터페이스에 맞게 만들어진 클래스는 Formatter이다.

Public class EventFormatter implements Formatter<Event> 을 구현하면 String <-> object 변환이 가능한데 Locale 기반으로도 가능하다. 따라서 i18n 을 쓰기도 편하다. Event parse(String text, Locale locale) 을 구현해주면 된다. 이것도 마찬가지로 WebConfig 의 registry 에 등록해주자.

이제 이 컨버터와 포매터를 ConversionService 에서 쓴다. 우리는 이걸 쓰고 있는거시다.
DefaultFormattingConversionService 라는 클래스를 많이 쓰는데,
FormatterRegistry 와 ConversionService 모두 구현해놨다. 부트는 이걸 WebConversionService (extends DefaultFormattingConversionService) 를 등록해준다. 이 클래스는 조금 더 기능이 많다.

그런데, 위에 처럼 WebConfig 에 등록해줄 필요가 없다!!!
@Component 로 등록해두면, 스프링 부트가 알아서 등록한다!!!!!

Converter

- S 타입을 T 타입ㅂ으로 변환할 수 있는 일반적인 변환기
- 상태 정보 없음 == Stateless == 쓰레드 세이프
- ConvertRegistry 에 등록해서 사용

Converter<S, T> (Source to Target)이다.

컨버터는 일방향이고, 포매터는 양방향이다. 포매터 만들고 빈으로 등록해서 쓰자~
그런데 이건 JPA 를 쓰면, Entity의 컨버터가 있으니까 그거 쓰면 된다.
그리고 등록된 컨버터들을 보려면 conversionService 를 출력하면 모든 컨버터가 뜬다.

# 14. SpEL (스프링 Expression Language)

JSP 중에. <c:if test=“\${sessionScope.cart.numberOfItems > 0}”> … </c:if>
요런 걸 스프링이 개발했다. EL 이 필요하고, 메소드와 문자열 템플릿까지 제공하는 Expression Language 가 필요했다. 여러 군데에서 쓰인다 뭐 시큐리티, 데이터, @Value 등등..

@Value(“#{1 + 1}”) int value;
@Value(“#{‘hello ‘ + ‘world’}”) String greeting;
@Value(“#{1 eq 1}”) boolean trueOrFalse;

SpEL 을 통해 스트링을 파싱해서 값을 넣어준다.

$를 쓰면, 프로퍼티를 참조하게 된다. #은 표현식을 쓰는 방법이다.
application.properties에 프로퍼티를 넣어두는데, my.value = 100 을 넣어놨다고 치면
@Value(“${my.value}”) 를 통해서 값을 주입받을 수 있다.

표현식 안에는 프로퍼티를 쓸 수 있지만, 프로퍼티 안에서는 표현식 못 쓴다.
“#{${my.value} eq 100}” (O)
“${my.value#{hihihi}}” (X)

배열, 리스트, 맵, 인덱서 모두 지원한다.
{1,2,3,4}
+, >, and or not 등등 레퍼런스 찾아보면 될 것 같다.

메소드 호출하는 기능도 있다. 빈의 정보도 참고할 수 있다. 한 번 해보자.

@Value(“#{sample.data}”) 이런 식으로, 샘플이라는 빈의 data라는 프로퍼티를 뽑을 수 있다. 또한 메소드를 호출할 수 있는데, 그건 나중에 보자.

@ConditionalOnExpression 에서도 지원하니까, 그 때 익스프레션을 기반으로 선별적으로 빈을 등록할 수 있다.

스프링 시큐리티에서도 또한 xml 설정에 hasRole(‘admin’) 뭐 이런 메소드들을 호출 할 수 있다.
스프링 데이터의 쿼리 어노테이션에도 당연히 넣을 수 있다.
@Query(“select ~~~ :#{#customer.firstname}”)
List<User> findUserByCustomerFirstName(@Param(“customer”) Customer customer)

ExpressionParser 를 이용해서 Expression 을 뽑을 수도 있다.
ExpressionParser parser = new SpleExpressionParser();
Expression expression = parser.parseExpression(“2 + 100”);
expression.getValue(Integer.class); 등을 이용하면 컨버젼되는 것이다.
위에서 배웠던 컨버젼 서비스를 여기서도 쓴다!

# 15. 스프링 AOP: 개념 소개

AspectJ 와 연동할 수 있으며, 스프링 AOP기능을 연결해주며 캐시와 스프링 트랜잭션등 다른 기능들이 적용되고 있다.

Aspect Oriented Programming 은 OOP를 보완하는 수단으로 흩어진 Aspect를 모듈화 할 수 있는 기법이다.

여러 클래스에 흩어져있는 비슷한 코드들, 가령 트랜잭션을 예로 들어보자. Class A, B, C 모두 auto commit 을 false로 만들고 커밋과 롤백을 넣고 중간에 서비스 로직까지 넣어야 한다. 혹은 메소드의 시간을 측정하기 위한 기능을 넣고 싶을 때도 코드가 중복된다.

각각의 concern 을 Aspect로 빼서 관리하자. 흩어진 관심사를 한 곳에 모은다. A에서 X를 쓴다고 해서 A에 넣지 말고, X를 어디에 쓰는지 정의하는 느낌이다.

해야 되는 일들을 묶어 놓은 것이 AOP이다.
AOP 주요 개념

- Aspect 와 Target
  - Asepect 는 모듈을 묶어 놓은 것.
  - Target : Advice 를 해줘야 하는 대상
  - Advice :해야 하는 일
  - PointCut : 어디에 적용되어야 하는지
  - Join Point : 메소드 실행 시점 (끼워드는 시점)
    - 필드가 업데이트 되었을 때, 무슨 메소드가 호출되었을 때, 객체가 생성될 때 등등 언제 끼어들 것인가.
- Aspect: 묶은 모듈 (Advice, PointCut) 포함
- Advice: 해야할 일들
- Pointcut: 어디에 적용해야 하는 지 정보(A, B, C)
- Target: 적용이 되는 대상(Class A, B, C)
- Join Point: 합류 지점
- AOP 구현체
  - 자바
    - AspectJ
- AOP 적용 방법
  _ 컴파일
  _ 컴파일할 때 소스 코드를 끼워넣으면서 적용시키는 방법.
  _ 로드 타임
  _ 클래스 파일을 로딩하는 시점에 클래스 코드를 변경한다.
  _ 런타임
  _ 스프링은 A라는 빈을 쓸 때 필요한 에스펙트 들을 걸 알고 있음. 그리고 A가 호출될 때 A라는 빈을 감싼 프록시 빈을 만듦. 그리고 그 프록시 빈에서 애스펙트를 호출하고 넘겨줌. 이걸 가장 많이 하긴 하는데,ㅡ
  흩어진 Aspect를 모듈화 할 수 있는 프로그래밍 기법이다.

# 16. 스프링 AOP: 프록시 기반 AOP

AspectJ 와 연동할 수 있으며, 스프링 AOP기능을 연결해주며 캐시와 스프링 트랜잭션등 다른 기능들이 적용되고 있다.

스프링 AOP 특징

- 프록시 기반의 AOP 구현체
- 스프링 빈에만 AOP를 적용할 수 있다.
- 모든 AOP를 제공하는 것이 목적이 아니라, 스프링 IoC와 연동하여 엔터프라이즈 애플리케이션의 문제에 대한 해결책을 제공하는 것이 목적.

프록시 패턴

- 인터페이스를 통해 접근 제어 또는 부가 기능 추가.

EventService 라는 인터페이스를 만들고, SimpleEvent 에 구현한다. 그리고 SimpleEvent 를 서비스로 등록해주자.
등록한 다음 eventService 인터페이스를 Autowired 로 받아서 createEvent와 publishEvent를 호출한다.

이제, 클라이언트 코드인 AppRunner 와 구현체인 SimpleEvent를 건들지 않고 기능을 추가해보자.
EventService 에서 시간을 측정해보자. 이걸 SimpleEvent에 넣어서 코드 앞 뒤로 long begin = System.currentTimeMills();를 측정한다고 해보자.
그러면 얼마나 관심도가 갑자기 분산되냐. 그걸 CrossCutting Concern 이라고 한다.

그래서 이 코드를 없애고 싶어서 프록시 패턴을 이용한다.
Public class ProxySimpleEventService implements EventService { } 를 만들고 @Primary 빈으로 등록하자. 이렇게 되면 같은 인터페이스를 구현했더라도 SimpleEvent 보다 먼저 불려와진다.
그리고 나서 simpleEventService를 가져오자. @Autowired EventService simpleEventService; 즉, RealSubject를 불러오자.
그리고, RealSubject 가 하는 일을 위임해서 실행해주자.
Public void createEvent {
simpleEventService.createEvent();
}

그리고 위 아래에 시간 측정하는 로직을 넣는다. 이렇게 프록시 패턴을 구현할 수 있다.

문제점

- 매번 프록시 클래스를 작성해야 하면?
- 여러 클래스 여러 메소드에 적용하려면?
- 객체들 관계도 너무 복잡하다.

뭐 이걸 해결하려면 다이내믹 프록시를 달고, 그걸 달면 어떻게 어떻게 되고… 할 수 있지만 넘 복잡했다.
런타임으로 프록시 객체를 만드는 방법이 있다. 그 방법을 기반으로 스프링에 달아서 해결한다.

스프링 AOP

- 스프링 IoC 컨테이너가 제공하는 기반시설과 다이나믹 프록시를 사용해 여러 복잡한 문제를 해결
- 동적 프록시 : 동적으로 프록시 객체 생성하는 방법
  - 자바가 제공하는 방법은 인터페이스 기반 프록시 생성
  - CGlib은 클래스 기반 프록시도 지원
- 스프링 IoC : 기존 빈을 대체하는 동적 프록시 빈을 만들어 등록 시켜준다.
  - 클라이언트 코드 변경 없음.
  - AbstractAutoProxyCreator 에서 만들어줌. BeanPostProcessor 의 구현체이다.

# 17. 스프링 AOP: @AOP

애노테이션 기반의 스프링 @AOP

일단 어노테이션을 추가하고, 애스펙트(포인트 컷, 어드바이스가 묶여진 모듈들) 만들어보자.
@Component @Aspect public class PerfAspecr

해야 할 일이 Advice고, 언제 해야 하는가는 PointCut 이다. 이 두가지를 애스펙트에 정의해보자.

Public Object logPerf(ProceddingJoinPoint pjp) { //pjp 는 어드바이스가 적용되는 대상이다.
Object retVal = php.proceed();
return retVal;
}

요거는 그냥 타겟을 받아서 진행시킨 것이다. 아무런 일도 일어나지 않는다.
여기에 앞 뒤에 long begin System.currentTimeMillis(); 를 받고 sout 해주면 그게 프록시 기능을 넣은 것이다.

이제, 이 어드바이스(할일)을 어떻게 적용할 것인가? 이 어드바이스를 호출을 언제 할 것인가? 메소드의 에러가 났을 때? 메소드 실행하는 중간에? 등등을 설정할 수 있는 게 @Around 다.

@Around를 쓰면 해당 어드바이스를 어느 시점에, 어느곳에 적용하겠다. 를 명시할 수 있다.
@Around(“execution(_ me.whiteship.._.EventService.\*(..))”) // 실행할 때 적용해라. 뭘? me.whiteship밑의 eventService 밑의 모든 메소드가 실행될 때 해당 어드바이스를 실행해라.

그런데 이렇게 하면 문제가 있다. deleteAction 때는 안 하고 싶다. 이럴 때는 execution 보다는 annotation으로 쓰는 게 좋다.
@PerLogging 이라는 어노테이션을 만들자.
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.CLASS) # 클래스 파일까지 어노테이션을 유지하겠다. 라는 의미다. 기본값이 CLASS 다. 만약 SOURCE 라고 쓰면 컴파일하고 나서 사라진다.
public @interface PerLogging

이제 이 @PerLogging을 달아보자. @PerLogging public void publishEvent 하면 끝난다.

이제 위의 @Around를 변경하자.
@Around(“@annotation(PerLogging)”)
이라고 쓰면, PerLogging 어노테이션이 붙은 곳에 해당 어드바이스를 적용해라. 라는 의미이다.
뭐 @Around(“bean(simpleEventService)”) 같이 빈을 대상으로 적용할 수도 있다~~

@Around 보다 조금 간단하게, 메소드가 실행되기 이전에 어드바이스를 적용하고 싶으면
@Before(“bean(simpleEventService)”) 이런 식으로 적용할 수 있다.

의존성 추가

- org.springframework.boot:spring-boot-starter-aop

애스팩트 정의

- @Aspect
- 빈으로 등록해야 하니까 @Component 도 추가.

포인트 컷 정의

- @Pointcut(표현식)
- 주요 표현식
  - execution
  - @annotation
  - bean
- 포인트컷 조합
  - &&, ||

어드바이스 정의

- @Before
- @AfterReturning
- @AfterThrowing
- @Around

# 18. Null-safety

스프링 프레임워크에 추가된 NULL 관련 어노테이션이다.

NULL을 허용하느냐 마느냐 미연에 방지하는 툴이다.

컴파일 타임에 NULL 포인트 익셉션을 미연에 방지할 수 있다.

이름이 무조건 있어야 한다.
@NonNull String name

리턴값이 있어야 한다.
@NonNull
Public String createEvent()

그러면 eventService.createEvent(null); 을 했을 때 에러가 나야 하지 않느냐?
에러가 나지 않는다. 그래서 찾아보니, inteliji Compiler 옵션에 Configure Annotations 에 Add Runtime 어노테이션 중 스프링 NonNull과 Nullable을 추가해줘야 인식한다.
요러면 쪼끄만하게 에러가 뜬다.

그리고 기본 값으로 모두 NonNullable 시키는 @NonNullAPI 뭐 이런 게 있는데, 많이 쓰지는 않을듯...
