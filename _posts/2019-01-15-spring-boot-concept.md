---
layout: post
title: "스프링 부트 개념과 활용(백기선님 인프런 강좌)"
author: academey
categories: Spring
---

# 1.스프링 부트 시작하기

메이븐으로 프로젝트 만들고
스프링 부트 독스에서 pom.xml 에다 부모 프로젝트랑 디펜던시 넣어줌.
https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#getting-started-maven-installation
이렇게 하는 방법이 있고, spring-initializer 로 하는 방법이 있고, https://start.spring.io/ 에서 웹 디펜던시 넣어서 다운 받는 방법이 있다.
원하는 대로 하자.

IDE에 psvm 치면 public static void main 만들어줌.
@SpringBootApplication 가 있는 패키지 아래에 어노테이션을 검색해서 빈으로 다 등록시켜준다.

# 2.스프링 부트 원리

1. 의존성 관리
   Parent pom 설정을 써서 부트에서 의존성 관리를 안해줘도 되는 거임.
   <parent> 태그 따라가다 보면 다 설정 미리 되어 있음.
   얘네들의 변수값만 바꿔주면 버젼 변경이 가능함. 아래 느낌으로
   <properties>
   <java.version>1.8</java.version>
   <spring.version>5.0.6.RELEASE</spring.version>
   </properties>

근데 기존에 안 적혀있는 패키지들은 새로 넣어줘야함. 이 때는 Inteliji 옆에 따라가는 동그라미가 없으니 참고.
Inteliji에서 디펜던시 새로운 거 필요하면 auto import 해서 가져다 줌.

2. 자동 설정 이해
   어떻게 톰캣이 자동으로 뜨는지 궁금하지?

@SpringBootConfiguration에는 세개의 어노테이션이 있는데,
@SpringBootConfiguration, @ComponentScan, @EnableAutoConfiguration 이다.

여기서 빈을 읽어올 때, @ComponentScan 으로 한 번 빈으로 등록하고, 다음에 더 세팅된 빈들을 @EnableAutoConfiguration로 긁어 온다.
@ComponentScan : @Component, @Configuration, @Repository, @Service, @Controller, @RestController … 등
@EnableAutoConfiguration : spring.factories, @Configuration # spring.factories 는 spring-auto-configure 패키지에 직접 명시되어 있는 configuration 파일들이 명시되어 있다. 저 파일 모두가 @Configuration 이 붙어있고, 불러오는 시점을 정해놨다.

@ConditionalOnWebApplication(type = TYPE.SERVLET) 은 웹 어플리케이션 설정에 따라 해당 Configuration 파일을 호출한다는 의미이다.
이것처럼, AutoConfiguration 을 통해서 수많은 설정들이 특정 조건에 따라서 불러와지게 되는 것이다.

# 3. 자동 설정 만들기 : Starter 와 AutoConfiguration

이제 새 클래스를 외부에서 자동으로 관리해주고 설정해주는 프로젝트를 만들어보자. hyunjoon-springboot-starter 뭐 이런식으로 만든 다음에, 클래스를 만들고 Configuration에서 빈을 생성한 다음에 spring.factories 파일에 해당 Configuration 파일을 적어둔다. 이후, 메인 프로젝트에 와서 해당하는 프로젝트의 아티팩트 정보를 디펜던시에 박아 넣으면 임포트가 된다.
그렇게 가져온 해당 클래스를 자동으로 실행시켜주기 위한 HolomanRunner 를 ApplicationRunner 를 상속받아서 구현시키자. 그리고 Autowired 로 홀로맨을 받으면??? 이 프로젝트에서는 홀로맨을 빈으로 등록한 적이 없는데, 가져온다.

그런데, 메인 프로젝트에서 따로 동일한 빈을 등록하면 가져온 빈이 등록한 빈을 override 해버린다. 컴포넌트 스캔으로 빈을 가져온 다음에 AutoConfiguration에서 빈을 가져오니까 원래 등록한 빈이 오버라이드 되는 것이다.

- Xxx-Spring-Boot-Autoconfigure 모듈: 자동 설정
- Xxx-Spring-Boot-Starter 모듈: 필요한 의존성 정의
- 그냥 하나로 만들고 싶을 때는? \* Xxx-Spring-Boot-Starter
  구현 방법

1. 의존성 추가
   <dependencies>
   <dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-autoconfigure</artifactId>
   </dependency>
   <dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-autoconfigure-processor</artifactId>
   <optional>true</optional>
   </dependency>
   </dependencies>

<dependencyManagement>
  <dependencies>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-dependencies</artifactId>
          <version>2.0.3.RELEASE</version>
          <type>pom</type>
          <scope>import</scope>
      </dependency>
  </dependencies>
</dependencyManagement>
2. @Configuration 파일 작성
내부에 설정할 빈 만들기.
3. src/main/resource/META-INF에 spring.factories 파일 만들기

4. spring.factories 안에 자동 설정 파일 추가
   org.springframework.boot.autoconfigure.EnableAutoConfiguration=클래스명쓰기
5. mvn install
   AutoConfiguration이 가져온 빈이 오버라이드 안 하려면 조건을 설정해주면 된다.
   외부 빈에 @ConditionalOnMissingBean 이 붙어있어야 한다. 그러면 내가 만든 빈이 더 강하다!
   그런데 나는 굳이 빈을 만들고 싶지 않다. 특정 파일에 해당 빈의 속성만 적어두면, 그 빈이 읽어서 만들어주면 안 되냐?
   이제 그 설정을 해보자. ConfigurationProperties 를 쓸때가 됐다.

- 덮어쓰기 방지하기
  - @ConditionalOnMissingBean
- 빈 재정의 수고 덜기
  _ @ConfigurationProperties(“holoman”) # 앞에 prefix 정의
  _ @EnableConfigurationProperties(HolomanProperties) # 요거는 Configuration 파일 위에 선언해줘야 함. 이 컨피규 가져갈 때 나는 요 Properties 파일 쓰겠다! \* 프로퍼티 키값 자동 완성
  <dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-configuration-processor</artifactId>
  <optional>true</optional>
  </dependency>

# 4. 내장 웹 서버 이해

라이브러리 보면, 톰캣이 설치되어 있다. 그 톰캣을 자바 코드로 부를 수 있는데, 테스트 해보자.
서버를 톰캣 부르고 서블릿 매핑해서 뭐 설정해서 서버 시작할 수 있다. 이거를 부트에서는 auto-configure 의 spring.factories 기억나지? 거기서 서블릿 웹서버 자동 설정으로 사용해준다.
ServletWebServerFactoryAutoConfiguration 에서 서블릿 웹 서버 생성하고, TomcatServletWebServerFactoryCustomizer 에서 서버 커스터마이징이 가능하다.

스프링 부트는 서버가 아니다. 톰캣 객체 생성 포트 설정 톰캣에 컨텍스트 추가 서블릿 만들기 톰캣에 서블릿 추가 컨텍스트에 서블릿 맵핑 톰캣 실행 및 대기 이 모든 과정을 보다 상세히 또 유연하고 설정하고 실행해주는게 바로 스프링 부트의 자동 설정. ServletWebServerFactoryAutoConfiguration (서블릿 웹 서버 생성) TomcatServletWebServerFactoryCustomizer (서버 커스터마이징) DispatcherServletAutoConfiguration 서블릿 만들고 등록

만약 서블릿 컨테이너로 톰캣을 쓰지 않고 싶다!
그러면 spring-boot-starter-web 에 톰캣이 있으니, 거기에서 먼저 빼야 한다.
<exclusion> <groupId>org.springframework.boot</groupid><artifactId>spring-boot-starter-tomcat</artifactId></exclusion>

그리고 undertoe 라는 서블릿 컨테이너를 디펜던시에 다시 넣는다. 뭐 다른 거 쓸 수도 있다.
제티를 넣으려면 <dependency> <groupId>org.springframework.boot</groupId> <artifactId>spring-boot-starter-jetty</artifactId> </dependency>

요렇게 넣자.
이제 스프링 웹 애플리케이션 구동 시 타입을 프로퍼티 값으로 지정하자. application.properties 에 spring.main.web-application-type=none 하면 된다.

서버 포트는 server.port = 로 설정할 수 있는데, 이걸 자바 단으로 가져오려면아래와같은클래스의빈
ApplicationListener<ServletWebServerInitializedEvent>을만들어서관리하는게좋다.
PortListener 에서 onApplicationEvent 메소드를 구현해서 해당 이벤트로 ServletWebServerApplciationContext 로 웹 서버를 가져오고 포트를 조회할 수 있다.
자세한 건 문서 참조. https://docs.spring.io/spring-boot/docs/current/reference/html/howto-embedded-web-servers.html#howto-discover-the-http-port-at-runtime

# 5. HTTPS와 HTTP2

인증서를 야매로 만들고,(https://gist.github.com/keesun/f93f0b83d7232137283450e08a53c4fd) properties 에서 아래와 같이 등록하면,
server.ssl.key-store=keystore.p12
server.ssl.key-store-type=PKCS12
server.ssl.key-store-password=**\***
server.ssl.key-alias=spring
서버 실행할 때 톰캣의 커넥터에 SSL 설정이 된다.
커넥터가 하나여서, https를 설정하면 http 는 아예 막힌다. 두 개 다 연결해주려면, 따로 커넥터를 만들어서 tomcat에 추가적으로 붙여줘야 한다.

http2 같은 경우는 서블릿 컨테이너 중 톰캣은 자바 버젼과 호환성 문제가 있으니, undertow라는 걸 써서 했다. 이건 좀 뒤로 미루자

# 6. 독립적으로 실행 가능한 JAR 파일

이제 웹 프로젝트를 빌드해서 내려보자.
mvn clean package 를 치면 지우고 다시 jar 파일 내려준다. target 폴더 들어가면 jar 있음.
해당 파일 unzip 해보면, BOOT-INF와 META-INF에 프로젝트에 필요한 라이브러리 다 꽂아놨음.
해당하는 jarFile을 읽어들이고 실행하는 로더와 자르 런쳐가 다 존재한다.
MANIFEST 파일의 Main-Class부터 시작한다.

# 7. SpringApplication

vmoptions 에 -dDebug 옵션 주면 로깅 디버그 레벨까지 뿌려줌. 여기서 빈 꽂는 거 어떤 조건 떄문에 주입이 안 되었는지 확인할 수 있음.

- 기본 로그 레벨 INFO 뒤에 로깅 수업때 자세히 살펴볼 예정
- FailureAnalyzer
- 배너
  - banner.txt | gif | jpg | png
  - classpath 또는 spring.banner.location
  - \${spring-boot.version} 등의 변수를 사용할 수 있음. Banner 클래스 구현하고
  - SpringApplication.setBanner()로 설정 가능.
  - 배너 끄는 방법
- SpringApplicationBuilder로 빌더 패턴 사용 가능
  - new SpringApplicationBuilder().sources.(SpringApplication.class).banner(new Banner() { @Override public void printBanner(Environment environment, Class<?> sourceClass, PrintStream out) { out.println("test"); } }).run(args);
- 스프링 어플리케이션에 발생하는 많은 이벤트들이 있다. 앱 가동시, 에러 발생시, 컨텍스트 로드시 .. 등등 의 이벤트를 받아보자.
- SampleListener implements ApplicationListener<ApplicationStartingEvent> (어떤 이벤트 받을지 알려줘야 함. 그리고 해당 클래스를 빈으로 등록 시켜줘야 함. 그런데, 애플리케이션 컨텍스트가 만들어진 이후 이벤트들은 빈을 자동 감지해서 해당 리스너를 호출해줄 수 있는데, 그 전에 발생하는 이벤트들은 어떻게 해야하는가?
- 직접 빈에 등록해줘야 한다. SpringApplication app = new SpringApplication(SpringApplication.class); app.addListeners(new SampleListener());
- 아래와 같은 arguments 를 받아 출력해주는 걸 확인해보면, vmoptions(-D??) 에 들어간 인자는 없고, program arguments(--??) 에 들어간 인자만 있다.
- 이제는 arguments 를 스프링으로부터 주입 받아보자. 스프링에서 컴포넌트 등록할 때 constructor 파라미터의 타입이 빈으로 등록되어 있으면 자동으로 주입해주던거 기억나지? ApplicationArguments 가 바로 등록된 빈이다. 따라서 아래와 같은 컨스트럭터를 가진 클래스를 컴포넌트로 등록하면 자동으로 ApplicationArguments를 가져온다~ 이말이야
- public ArgumentsDebugger(ApplicationArguments arguments) {
- 이걸 인터페이스 해둔 게 ApplicationRunner 이니까, 여기서 run 메소드를 오버라이드 하면 된다. CommandLineRunner는 다중인자로 받아진다. ApplicationRunner 써라. 그리고 @Order를 통해 우선순위를 줄 수도 있음.

# 8. 외부 설정 1부 (우선순위)

- application.properties 에다 외부 설정값을 넣어줄 수 있는데, 이걸 스프링 단에서 호출하려면 @Value("\${hyunjoon.name}”) private String name; 이런 식으로 불러준다.
- 사용할 수 있는 외부 설정 중 properties의 우선순위는 되게 낮다. 즉, 외부 설정으로부터 넘어오는 값의 우선순위가 있다.
- 프로퍼티들은 Environment를 Autowired 해서 가져올 수 있음. 테스트 용 프로퍼티를 설정하기 위해서 Environment 를 설정하자. environment.getProperty(“keen.name”) 을 가져오자.
- 그런데 중요한 에러가 있을 수 있다. 소스를 빌드 시에, 앱 밑에 있는 걸 빌드하고 테스트 폴더 아래에 있는 걸 빌드한다. 즉 각각 애플리케이션 프로퍼티를 다른 걸 가져가게 된다. 따라서 소스 상에 쓰는 프로퍼티를 테스트 프로퍼티 파일에 안 넣어두면 에러가 발생하니 주의 하자. 그래서, 다 똑같이 가져갈 심산 아니면 Test 파일 위에 프로퍼티 지정해주면 된다. 근데 그게 또 ㄴ너무 길다! ㅡㄱ러면 test.properties 파일을 만든 다음에 @TestPropertySource(locations=“classpath:test.properties”) 로 적는다.
- 그렇게 되면, 테스트를 빌드할 때만 test.prproerties 가져와서 우선순위 높은 놈으로 오버라이드 시켜준다.
- 엥? 그러면 application.properties 는 항상 엎어써버리는 거냐? 그건 아님. 맨 상위의 application.properties 가 더 강하다는 걸 알 수 있음.
- 프로퍼티 우선 순위
- 유저 홈 디렉토리에 있는 spring-boot-dev-tools.properties
- 테스트에 있는 @TestPropertySource
- @SpringBootTest 애노테이션의 properties 애트리뷰트
- 커맨드 라인 아규먼트
- (몰라도됨)
- SPRING_APPLICATION_JSON (환경 변수 또는 시스템 프로티) 에 들어있는 프로퍼티
- ServletConfig 파라미터
- ServletContext 파라미터
- java:comp/env JNDI 애트리뷰트
- System.getProperties() 자바 시스템 프로퍼티
- OS 환경 변수
- (/몰라도됨)
- RandomValuePropertySource
- JAR 밖에 있는 특정 프로파일용 application properties
- JAR 안에 있는 특정 프로파일용 application properties
- JAR 밖에 있는 application properties
- JAR 안에 있는 application properties(굉장히 낮다)
- @PropertySource
- 기본 프로퍼티 (SpringApplication.setDefaultProperties)
- application.properties 우선 순위 (높은게 낮은걸 덮어 씁니다.)
- file:./config/
- file:./
- classpath:/config/
- classpath:/

# 9. 외부 설정 2부 (묶어주기)

- 이제 해당하는 Configuration Properties 를 하나로 묶어서 관리하자. 스프링에도 컴포넌트로 등록하고!
- 관리하고 싶은 정보를 클래스로 만들어서 아래와 같이 어노테이션을 붙이자.
- @Component
- @ConfigurationProperties("keesun")
- 그러면 @EnableConfigurationProperties
- 를 붙여야 하는데, 요건 기본 스프링 프레임워크에 붙여져 있음. 프로퍼티가 이제 타입 세이프하게 되는 것이다.
- 그리고 바인딩할 때, context-path (케밥) \_(언더스코어) 카멜케이스, 다 대문자 모두 다 융통성 있게 바인딩해준다.
- 프로퍼티의 속성에 맞게 타입 컨버젼도 지원한다.
- 시간 정보를 받을 수도 있음. @DurationUnit(ChronoUnit.SECONDS) # 초로 단위 설정한 거임
- 아니면 이 애노테이션 없이도 s 만 붙여주면 초로 인식해주기도 함.

# 10. 프로파일

- 프로파일에 따라 다른 빈 설정을 사용해보자.
- config 폴더 아래에 다음과 같은 클래스를 만들고, application.properties 에 spring.profiles.active=prod 를 작성해보자.
- @Profile("prod")
- @Configuration
- public class BeanConfiguration {
-     @Bean
-     public String hello() {
-         return "hello";
-     }
- }
-
- 즉, 프로파일은 프로퍼티다. 콘솔에서 실행할 때 파라미터에 prod 를 넣으면, 프로덕션 프로파일로 실행된다는 것이다.
- 프로파일에 따른 프로퍼티 파일도 따로 만들 수 있다. application-prod.properties 라는 파일이 있으면 applciation.properties 를 오버라이드 한다.

# 11. 로깅

- 로깅 구현체 윗단의 추상 레이어를 로깅 Facade라고 한다. 그 로깅 퍼사드의 종류가 SLF4J 혹은 Commons Logging이고, interface 덩어리라고 보면 된다. 기본 스프링 부트는 Commons Logging 에 Logback 이라는 구현체를 붙인다고 알기만 하면 될듯.
- spring-boot-starter-web 에 들어가 있으니, 로깅을 제외하고 싶으면 exclusions 에 spring-boot-starter-logger 를 제외해야 한다.
- 로깅 퍼사드 VS 로거
- Commons Logging, SLF4j
- JUL, Log4J2, Logback
- 스프링 5에 로거 관련 변경 사항
- https://docs.spring.io/spring/docs/5.0.0.RC3/spring-framework-reference/overview.html#overview-logging
- Spring-JCL
  - Commons Logging -> SLF4j or Log4j2
  - pom.xml에 exclusion 안해도 됨.
- 스프링 부트 로깅
- 기본 포맷
- –debug (일부 핵심 라이브러리만 디버깅 모드로)
- –trace (전부 다 디버깅 모드로)
- 컬러 출력: spring.output.ansi.enabled
- 파일 출력: logging.file 또는 logging.path
- 로그 레벨 조정: logging.level.패지키 = 로그 레벨(DEBUG, INFO 등등)
- 커스텀 로그 설정 파일 사용하기
- Logback: logback-spring.xml
- Log4J2: log4j2-spring.xml
- JUL (비추): logging.properties
- Logback extension
  - 프로파일 <springProfile name=”프로파일”>
  - Environment 프로퍼티 <springProperty>
- 로거를 Log4j2로 변경하기

# 12. 테스트

- 테스트에 필요한 건 spring-boot-starter-test로 주입받자. 우선 테스트할 컨트롤러와 서비스를 만든 다음에, 테스트로 넘어 가자.
- MVC 를 테스트 하기 위해서는 MockMvc 가 필요한데, 그렇게 하기 위해서는 위에 웹 환경을 MOCK 이라고 설정해주면 autowired가 된다. MockMvc 에 요청을 날려서 반환하는 값들이 일치하는지 체크하자. MOCK 은 내장 톰캣 구현 안 한다.
- @RunWith(SpringRunner.class)
- @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
- public class SampleControllerTest {
-     @Autowired
-     MockMvc mockMvc;
-
- }
- @RunWith(SpringRunner.class) @SpringBootTest(webEnvironment= SpringBootTest.WebEnvironment.MOCK) @AutoConfigureMockMvcmock 를 테스트 위에 달아줘야함.
- Mvc.perform(get(“/hello”).andExpect(status().isOk()) 이런 느낌으로 체크
- 랜덤 포트를 쓰면 테스트레스트템플릿을 따로 주입받아서 써야 한다. 내장 톰켓 서버에 아예 요청을 보내는 방식인데, TestRestTemplate 이란 것을 autowired 받아서 testRestTemplate.getForObject(“/hello”, String.class); 처럼 요청한 리소스가 뭔지 알아낸다.
- 근데 나는 서비스를 테스트하고 싶지 않고 컨트롤러만 테스트하고 싶다. 그러면 서비스를 MockBean으로 갈아끼우자. 테스트 스텁이 되는 것이다. 기존에 등록해둔 서비스가 아닌 새로운 빈으로 꽂자. @MockBean SampleService mockSampleService; 를 변수로 만들자. 모든 테스트 메소드들 마다 새로 리셋이 된다. 따
- 그리고 특정 메소드의 리턴 값을 모킹해주자. when(MockSampleService.getName()).thenReturn(“whiteship”);
- 그리고 async한 구조인 web-flux 를 사용해서 Reactive 웹으로 쓰면서 WebTestClient를 가져와서 api 테스트 가능하다.
- 그런데, 이렇게 계속 모킹해야할 서비스들이 많아지는데, 귀찮지 않느냐. 나는 잘라서 테스트하고 싶은 것만 하고 싶으면 슬라이스를 써야한다. @WebMvcTest(SampleController.class)를 어노테이션으로 붙이면 웹 관련된 애들만 빈으로 등록이 된다. Service, Repository 는 하나도 안 가져옴. 웹 계층 아래 있는 애들은 싹 다 끊긴다. 따라서 mockBean 으로 다 채워줘야 함. @JsonTest, @WebMvcTest 등의 슬라이스 테스트 어노테이션에 의해 레이어 별로 등록이 됨. 근데 뭐 서비스 공통적인 건 하나의 빈으로 쓰게 되니까, @AutoConfigureMockMvc 쓰자.

# 13. 테스트 유틸

- Unit 의 OutputCapture 라는 테스트 로깅용 클래스가 있다.
- 이건 빈이 아니라 JUnit Annotation인데, @Rule public OutputCapture outputCapture = new OutputCapture(); 로 받는다.
- 콘솔과 로깅, output 모두 캡쳐할 수 있다. outputCapture.toString() 에 다 담겨져있음.

# 14. Spring-Boot-Devtools

- 스프링은 클래스 로더를 두개 쓰는데, 의존성 클래스 로더랑 애플리케이션 로더 중 애플리케이션만 리로드하니까 개발시에 빠름. 의존성 추가한 다음에 build 해주면 자동으로 리로드해준다.
- 캐시 설정을 개발 환경에 맞게 변경.
- 클래스패스에 있는 파일이 변경 될 때마다 자동으로 재시작.
  - 직접 껐다 켜는거 (cold starts)보다 빠른다. 왜?
  - 릴로딩 보다는 느리다. (JRebel 같은건 아님)
  - 리스타트 하고 싶지 않은 리소스는? spring.devtools.restart.exclude
  - 리스타트 기능 끄려면? spring.devtools.restart.enabled = false
- 라이브 릴로드? 리스타트 했을 때 브라우저 자동 리프레시 하는 기능
  - 브라우저 플러그인 설치해야 함.
  - 라이브 릴로드 서버 끄려면? spring.devtools.liveload.enabled = false
- 글로벌 설정
  - ~/.spring-boot-devtools.properties <- 요게 가장 우선순위가 높음.
- 리모트 애플리케이션

# 15. 웹 MVC

1. 기본 설정

- @RestController, @GetMapping 등의 어노테이션을 auto-configure 에서 가져온다. 거기에 WebMvcAutoConfiguration 클래스가 있어서 가능한것이다.
- 스프링 MVC 설정을 확장할며면 @Configuration 을 붙인 WebMvcConfigurer 를 구현한 클래스를 따로 만들어야 하는데, 우리는 우선 기본 설정만 먼저 쭉 따라가보자
- 스프링 웹 MVC
  - https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#spring-web
- 스프링 부트 MVC
  - 자동 설정으로 제공하는 여러 기본 기능 (앞으로 살펴볼 예정)
- 스프링 MVC 확장
  - @Configuration + WebMvcConfigurer
- 스프링 MVC 재정의
  - @Configuration + @EnableWebMvc

2. HttpMessageConverters

- HttpMessageConverters Request Body 에 객체가 들어올 때, JSON 형식이면 JSONConverter 를 써서 객체로 변환시켜 준다. 그리고 해당 객체를 또 HTTP 에 JSON 으로 내려줄 때 변환해준다.
- @RestController 가 붙어 있으면, @ResponseBody 가 항상 붙어있다. 그래서 메시지 컨버터가 작동한다. 그냥 컨트롤러였으면 뷰 네임 resolver 를 찾으려고 했겠지
- User에 setter 가 있어야 오브젝트 매핑이 됨.

3. ViewResolver

- ContentNegotiationViewResolver : 뷰 리졸버 중 하나로 리퀘스트의 Accept Header 에 따라 반환하는 값을 다르게 내려줌. 레일즈에서 비슷하게 한 구조인듯. acceptHeader 를 받지 못하는 경우도 있어서, format이라는 파라미터를 받아서 pdf, html, xml 등등 요청할 수도 있음. 요건 스프링 부트에서 자동으로 등록되어 있다.
- 따라서 acceptHeader를 XML로 요청을 보내면 되는데 이 떄 406(MediaTypeNotAcceptableException) 이 뜬다. 처리해줄 HTTPMessageConverter 가 없는 것이다. 이건 HttpMessageConvertersAutoConfiguration을 뒤져보면, JacksonHttpMessageConvertersConfiguration 이라는 클래스가 있는데, 거기를 가보면 JsonConverter 는 있는데, XmlMapper 라는 클래스가 없어서 컨버팅이 안 된다.
- 그 의존성을 넣어주려면 아래 코드 추가하자.
- <dependency>
- <groupId>com.fasterxml.jackson.dataformat</groupId>
- <artifactId>jackson-dataformat-xml</artifactId>
- <version>2.9.6</version>
- </dependency>
- JSON 은 요즘은 뭐 다 쓰니까 안 추가해도 잘 됨.

4. 정적 리소스 지원

- Hello.html 요청이 오면, /static/hello.html 을 찾아서 보내준다.
- 기본 리소스 위치는 아래와 같은데, 따로 설정할 수도 있다. Localhost:3333/hello.html 에 즉시 접근이 가능하다. 이걸 ResourceHttpRequestHandler 에서 처리한다. 리소스가 변경되지 않았을 때를 체크해서 내려준다. 즉, lastModified 를 체크해서 304를 내려준다. Request 에 lf-modifed-at 을 보내고, response의 lf-modified-at 이 변경되었다면 새로 받아서 오는 거고, 아니면 브라우져 단의 리소스를 가져다 쓴다.
- 리소스 위치를 변경하려면, 프로퍼티 설정하자. spring.mvc.static-path-pattern=/static/\*\* 를 하면 앞에 붙여서 써야 함. 기본 위치를 변경하는 건 별로 안 좋은 것 같음.
- 그거 보다 WebConfig implements WebMvcConfigurer를 만들어서 관리하자 기존 리소스 위치는 유지하되 따로 추가하는 거임.
- 기본 리소스 위치
  - classpath:/static
  - classpath:/public
  - classpath:/resources/
  - classpath:/META-INF/resources
  - 예) “/hello.html” => /static/hello.html
  - spring.mvc.static-path-pattern: 맵핑 설정 변경 가능
  - spring.mvc.static-locations: 리소스 찾을 위치 변경 가능
- Last-Modified 헤더를 보고 304 응답을 보냄.
- ResourceHttpRequestHandler가 처리함.
  - WebMvcConfigurer의 addRersourceHandlers로 커스터마이징 할 수 있음

5. 웹 JAR

- 클라이언트용 javascript 라이브러리를 jar 파일로 의존성 관리를 할 수 있다. 그게 webjar 임. 이 webjar가 maven 에도 올라와있기 때문에 jquery 를 넣어보자.
- 이 두개를 추가하면, jquery 가 들어가고, 버져닝 없이 알아서 찾아준다.
- <dependency>
-     <groupId>org.webjars</groupId>
-     <artifactId>webjars-locator-core</artifactId>
-     <version>0.36</version>
- </dependency>
- <dependency>
-     <groupId>org.webjars</groupId>
-     <artifactId>jquery</artifactId>
-     <version>3.3.1-1</version>
- </dependency>

6. Index 페이지와 파비콘

- 저 기본 리소스 위치 중 아무곳에나 index.html 이 있으면 그걸 내려준다.
- 파비콘도 마찬가지로 리소스 위치 중 favicon.ico 있으면 됨. favicon.io 사이트 들어가서 만들면 좋음

6. Index 페이지와 파비콘

- 저 기본 리소스 위치 중 아무곳에나 index.html 이 있으면 그걸 내려준다.
- 파비콘도 마찬가지로 리소스 위치 중 favicon.ico 있으면 됨. favicon.io 사이트 들어가서 만들면 좋음

7. Thymeleaf

- 스프링 부트가 자동 설정을 지원하는 템플릿 엔진 : FreeMarker, Groovy, Thymeleaf, Mustache
- JSP를 권장하지 않는 이유 : JAR 패키징할 때 동작 안 하고, WAR 패키징해야하고, Undertow 라는 서블릿 컨테이너가 지원하지 않음.
- Thymeleaf 의존성 추가하면서 자동설정 적용되면, resources/templates 의 뷰에서 찾게 된다. 이제 RESTController 가 아니라 그냥 컨트롤러 만들어야 함. 컨트롤러에서 바로 테스트 생성하려면 시프트 + fn + T 누르면 된다.
- 테스트에서 andDo(print())가 된다. 예전에는 서블릿 엔진에서 JSP 스펙을 렌더링해야지만 뷰단이 출력이 되었었는데, thymeleaf는 독자적으로 서블릿 컨테이너 없이 뷰단의 최종 결과물을 출력해줄 수 있다.
- Thymeleaf 사용하기
- https://www.thymeleaf.org/
- https://www.thymeleaf.org/doc/articles/standarddialect5minutes.html
- 의존성 추가: spring-boot-starter-thymeleaf
- 템플릿 파일 위치: /src/main/resources/template/
- 예제: https://github.com/thymeleaf/thymeleafexamples-stsm/blob/3.0-master/src/main/webapp/WEB-INF/templates/seedstartermng.html

8. HTML Unit

- HTML 템플릿 뷰 테스트를 보다 전문적으로 하자. HTML 단위 테스트 툴이다.
- 웹 클라이언트를 만들어서 요청을 보낸고 그 결과를 HTMLPage 로 담아서 본문을 체크할 수 있다.
- 모델, 뷰 파일의 이름이 아니라 html 더 특화된 테스트들을 많이 해볼 수 있다. WebClient 를 주입받아서 HtmlPage page = webClient.getPage(“/hellow”); 같이 받아온다.

9. ExceptionHandler

- 스프링은 자동으로 에러 핸들러가 달려져 있다. 없는 페이지에 요청 날리면 브라우져에서는 에러 페이지, curl 에서는 JSON 형식으로 404를 반환한다. BasicErrorControllerd에서 보면, MediaType.TEXT_HTML_VALUE 면 html 로 만들어서 리턴하고, 아닌 경우는 @ResponseBody 로 내려준다.
- 이제 예시를 만들어보자. 특정 Exception을 던지면 그 예외를 처리해주는 메소드를 만들자. @ExceptionHandler(SampleException.class) public @Responsbody AppError sampleError(SampleException s) { return appError;}
- 이런 것처럼, BasicErrorController 에서는 특정한 에러들에 대해서 미리 인터셉터 하는 로직을 다 만들어놨다. ExceptionHandler 를 쓰지는 않았지만, 특정한 error 들에 대한 경로를 받아온다.
- 400 번대, 500번대 에러 페이지들을 만들고 싶으면 resources/static/error 밑에 4xx.html 혹은 404.html 처럼 정적 페이지 만들어서 내려줄 수 있다.
- 스프링 @MVC 예외 처리 방법
  - @ControllerAdvice
  - @ExceptionHandler
- 스프링 부트가 제공하는 기본 예외 처리기
- BasicErrorController HTML과 JSON 응답 지원 커스터마이징 방법 ErrorController 구현 커스텀 에러 페이지 상태 코드 값에 따라 에러 페이지 보여주기 src/main/resources/static|template/error/ 404.html 5xx.html ErrorViewResolver 구현

10. HATEOS(Hypermedia As The Engine Of Application State)

-
- 서버 : 현재 리소스와 연관된 링크 정보를 클라이언트에게 제공한다.
- 클라이언트 : 연관된 링크 정보를 바탕으로 리소스에 접근한다.
- spring-boot-starter-hateoas 의존성 추가한 다음에 쓰자.
- https://spring.io/understanding/HATEOAS https://spring.io/guides/gs/rest-hateoas/ 이 문서 읽으면 이해됨.
- Resource<Hateos> hateousResource = new Resource<Hateos); hateosResource.add(linkTo(methodsOn(SampleController.class).hateos()).withSelfRel());

11. CORS

- SOP(Single origin policy) : 같은 origin에만 요청을 보낼 수 있다.
- Cross Origin Resource Sharing : 다른 도메인끼리 리소스 공유할 수 있다.
- Origin 은 아래의 3가지를 조합한 것이다. 즉, https://naver.com 과 https://daum.net은 다른 origin이므로 리소르를 SOP가 적용되어 있으면 못 가져 온다. CORS 를 달아주면 가능.
  - URI 스키마 (http, https)
  - Hostname (localhost, naver)
  - 포트 (8080, 443, 80)
- 스프링에서는 CORS를 쓰려면 빈 설정을 해줘야 하는데, 부트에서는 제공하는 기능은 달아주기만 하면 쉽게 할 수 있다.
- 서버에서는 헤더에 Access-Controll-Allow-Origin 값을 설정해줘야 한다.
  - 그 첫번쨰 방법은 CORS 를 열고자 하는 메소드에 어노테이션을 붙인다. @CrossOrigin(origins=“https://naver.com”)
  - 위의 어노테이션을 컨트롤러에 붙이는 방법이 있다.
  - 전역으로 열어주려면, WebConfig implements WebMvcConfigurer 로 기존 스프링 MVC 설정을 ‘확장’ 하면 된다. WebConfig 파일을 만들고, @Configuration을 붙인다음에 @Override public void addCorsMapping(CorsRegistry registry) { registry.addMaping(“/\*\*”).allowedOrigins(“http://localhost:18080”);} 하면 된다.
- CORS SOP과 CORS Single-Origin Policy Cross-Origin Resource Sharing Origin? URI 스키마 (http, https) hostname (whiteship.me, localhost) 포트 (8080, 18080) 스프링 MVC @CrossOrigin https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-cors @Controller나 @RequestMapping에 추가하거나 WebMvcConfigurer 사용해서 글로벌 설정

# 16. 스프링 데이터

1. 인메모리 DB

- H2(콘솔 때문에 편함), HSQL, Derby
- Spring-jdbc 가 클래스패스에 있으면 자동 설정이 필요한 빈을 설정해줍니다.
  - DataSource
  - JdbcTemplate
- 프로젝트 만들때 Jdbc, H2 넣어서 만들어보자.
- 디비 설정이 따로 없으면 h2 인메모리 test 디비로 넣어준다.
- ApplicationRunner 를 만들어서 H2Runner implements ApplicationRunner 를 만들어서, Connection connection = dataSource.getConnection(); 한 다음에 SQL 쳐보자. connection.createStatement(); statement.executeUpdate(sql);
- H2 콘솔을 써보자.
  - Spring-boot-dev-tools 를 추가 하거나
  - Spring.h2.console.enabled=true 를 넣으면 됨.
  - /h2-console 에 접속하자.

2. MySQL

- DBCP(Database connecting pool) : DB 접속하는 지점을 미리 만들어놓고, 관리하는 곳이다. 몇 개 만들어 둘 것이냐 등등 조건들이 성능에 굉장히 민감하다.
  - HikariCP(기본) : 부트 디폴트. auto commit 설정이 true이고, connectionTimeout 은 기본이 30초이다. maximumPoolSize 는 커넥션 개체의 최대 개수인데, 기본이 10개이다. 동시에 일을 할 수 있는 커넥션 개수는 CPU 코어와 같기 때문에, 그 이상으로 시킬 필요가 없다. https://github.com/brettwooldridge/HikariCP#frequently-used
  - Tomcat
  - Commons
- 이제 Mysql 의존성을 추가하자. Mysql-connector-java 를 추가
- 도커를 이용해서 mysql 을 설치하자.
  - docker run -p 3306:3306 --name mysql_boot -e MYSQL_ROOT_PASSWORD=1 -e MYSQL_DATABASE=springboot -e MYSQL_USER=keesun -e MYSQL_PASSWORD=pass -d mysql // 이 스크립트를 치면 3306 포트에 mysql 띄워준다.
    - spring.datasource.url=jdbc:mysql://localhost:3306/springboot
    - spring.datasource.username=keesun
    - spring.datasource.password=pass
  - docker exec -i -t mysql_boot bash // 도커에 들어가서 배쉬를 실행한다.
    - mysql -u keesun -p 을 통해 디비 로그인
- 그런데, MySQL은 라이센스 & 공개 의무가 있어서 불편하니까 postgre를 보는 게 나을듯.

3. PostgreSQL

- org.postgresql 의존성을 넣자.
- 마찬가지로 도커로 postgresql 설치. docker run -p 5432:5432 -e POSTGRES_PASSWORD=pass -e POSTGRES_USER=keesun -e POSTGRES_DB=springboot --name postgres_boot -d postgres
  - docker exec -i -t postgres_boot bash
  - su - postgres
  - psql -U keesun springboot
    - 이 콘솔에서 \l 을 치면 databases 목록이 나옴. 테이블 조회는 \dt

4. 스프링 Data JPA

- ORM(Object Relational Mapping) 과 JPA(Java Persistence API)
  - 객체와 릴레이션을 맵핑할 때 발생하는 개념적 불일치를 해결하는 프레임워크
  - http://hibernate.org/orm/what-is-an-orm/
  - 테이블에서는 로우와 칼럼, id 값으로 Entity를 식별하는데, 객체에서는 상속계층이 있고, id 값을 hashcode이다. 어떤 값이 같아야 객체가 같은 것인가? 등등의 테이블 <-> 객체의 문제를 표준화 시킨 것이다.
  - JPA : ORM을 위한 자바 표준
- 스프링 데이터 JPA
  - JPA 표준 스펙을 쉽게 스프링 데이터로 추상화 시켜놓은 것. ORM 을 쉽게 쓸 수 있음. 그 구현체는 hibernate
  - Spring Data JPA -> JPA -> Hibernate -> DataSource 를 사용.
- 스프링 데이터 JPA 의존성 추가
  - Spring-boot-starter-data-jpa
  - 원래 @EnableJpaRepositories 어노테이션 붙여야 쓸 수 있는데, 부트에서 자동으로 붙어 있음.
  - @Entity Account 라는 클래스를 만들고, @Id @GeneratedValue private Long id; 를 만든다. 그리고 기타 프로퍼티와 getter, setter, equals, hashcode 등을 만든다. 이 때 롬복을 쓸 수도 있지만, 그건 나중에 보자.
  - Public interfcae AccountRepository extends JpaRepository<Account, Long> 을 만들자. 처음 건 Entitiy Class, 뒤에 건 id의 타입
  - 이제 이 Repository 를 테스트해보자. 테스트를 만들기에 앞서, 슬라이스 테스트를 할 건데, 그러기 위해서는 인메모리 데이터가 필요하다. H2 를 받아오자.
  - 그리고 MainApplication 돌리면 마찬가지로 등록된 디비가 없어서 문제 생긴다. Postgres SQL 의존성 추가하고 도커에 띄운 다음 프로퍼티에 datasource.url 설정하고 띄워보자.
  - 워닝이 뜨는데 특정 메소드 지원 안해서 뜨는 거니까 무시하고 프로퍼티에spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true 추가해주자.
- 테스트 해보자. Repository에는 기본적으로 JPA 에서 지원하는 기능들이 모두 들어가 있다. 인터페이스만 만들면 알아서 구현이 되어있다는 소리다. JDBC 를 직접 쓰지 않고 그 추상화 버젼인 JPA 에서 하는 것이다.
  - 테스트 코드에서 @Autowired AccountRepository; 를 받은 다음에, accountRepository.save(account); 와 accountRepository.findByUsername(newAccount.getUsername()); 이 동작하는지 테스트해보자.
  - findByUserName 은 기본 인터페이스에 생성이 안 되어 있으므로, 인터페이스에 Account findByUserName(String username); 으로 만들자. 만들면 JPA 가 알아서 구현해준다.
  - 이 때, 유저가 없을 수도 있으므로 Optional<Account> findByUserName 으로 구현할 수도 있다. 그렇게 되면 Optional 을 받아서 isNotEmpty로 변경하자.
  - 기본 설정 외에 다르게 구현하고 싶다면, Query 를 직접 줄 수 있는데, JPQL 라는 JPA 의 쿼리를 써야하지만 @Query(NativeQuery = true, value = “select \* from account;”) 처럼 쿼리 자체를 박아 넣을 수도 있다.

5. 데이터베이스 초기화

- JPA를 이용해서 초기 스키마를 만드는 방법은 다음과 같다.
  - spring.jpa.hibernate.ddl-auto=update // 운영 DB 에서는 validate 하면 됨. 업데이트 안 시키고 검증만 하는 거임.
  - spring.jpa.generate-ddl=true
  - spring.jpa.show-sql=true // 이건 그냥 스키마 생성 로그 찍히는거임
- SQL 스크립트를 직접 박아서 데이터베이스 초기화
  - Schema.sql 또는 schema-\${platform}.sql 으로 초기화. 내부에 쿼리 박아두면 됨.
  - data.sql data-\${platform}.sql 에 초기 목 데이터 넣을 수 있음.
  - properties에 spring.datasource.platform=postgresql 이렇게 적어서 쓰는 거임.

6. 데이터베이스 마이그레이션 툴

- Flyway와 Liquilbase가 대표적. 여기서는 Flyway를 써보겠다. org.flywaydb:flyway-core 의존성을 추가하자.
- 이제 데이터 스키마를 모두 버전 관리를 해보자. Properties 에서 ddl-auto 는 validate로 변경해서 스키마와 JPA Entity 가 일치하는지만 파악하면 된다.
- 마이그레이션 폴더
  - db/migration 또는 db/migration/{vendor}
  - spring.flyway.locations로 변경 가능
- 마이그레이션 파일 이름
  - V숫자\_\_이름.sql V는 꼭 대문자로.
  - 숫자는 순차적으로 (타임스탬프 권장)
  - 숫자와 이름 사이에 언더바 두 개.
  - 이름은 가능한 서술적으로
- 여기서 만약 isActive 라는 멤버변수를 추가하면, validate에서 에러가 난다. 따라서 두번째 마이그레이션 파일을 만들고 거기에 칼럼을 추가를 시켜줘야 한다.

6. Redis

- 의존성 추가 : spring-boot-starter-data-redis
- Redist 설치
  - docker run -p 6379:6379 –name redis_boot -d redis
  - docker exec -i -t redis_boot redis-cli
- 스프링 데이터 Redis
  - https://projects.spring.io/spring-data-redis/
  - StringRedisTemplate 또는 RedisTemplate // 가져온 의존성에 의해 StringRedisTemplate 빈이 떠 있으니까 autowired 로 가져온 다음 ValueOperations<String, String> values = redisTemplate.opsForValue(); 를 받고 values.set(“”,””); 을. 하면 키 밸류 값을 넣을 수 있음.
  - extends CrudRepository // JPARepository 처럼 인터페이스 상속받으면 자동으로 구현됨. @RedisHash(“accounts”) public class RedisAccount 라는 클래스를 만들고 CrudRepository 로 해당 클래스를 받아서 save 하면 redis 키에 객체의 해쉬코드와 등록이 된다. 얘네는 hash get, get 으로 가져와서 필드도 넣어줘야 가저온다.
  -
- Redis 주요 커맨드
  - https://redis.io/commands
  - keys \*
  - get {key}
  - hgetall {key}
  - hget {key} {column}
- 커스터마이징 spring.redis.\*
  - 스프링 레디스 기본 포트가 6379 이고 url 이 로컬호스트여서 프로퍼티 설정 없이 가능했는데, 만약 변경된다면 설정을 해줘야 한다.

7. MongoDB

- 의존성 추가
  - spring-boot-starter-data-mongodb
  - 의존성 추가하기만 하면 기본이 27017 포트라서 연결이 된다.
- MongoDB 설치 및 실행
  - 커맨드 구글링하면 나온다! Docker mongodb
  - docker run -p 27017:27017 –name mongo_boot -d mongo
  - docker exec -i -t mongo_boot bash
    - mongo
      - db
      - use test
      - db.accounts.find()
- 스프링 데이터 몽고DB
  - MongoTemplate 을 MongoRunner 에서 Autowired로 가져온다. 그리고 mongoTemplate.insert(mongoAccount); 로 넣어주자.
  - MongoRepository 를 상속받으면, 다른 리포지토리와 동일하게 동작한다. MongoRunner 에서 MongoRepository 주입받아 save 하자.
  - 이제 테스트를 만들어보자. 따로 테스트 몽고 띄우기가 번거로우니까 내장형 MongoDB (테스트용) 를 써서 만들 수 있다. 의존성을 추가하자.
  - de.flapdoodle.embed:de.flapdoodle.embed.mongo <scope>test</scope>
  - 이제 슬라이스 테스트해보자. @RunWith(SpringRunner.class) @DataMongoTest public class MongoAccountRepositoryTest {} 를 붙이면, 몽고 관련된 빈들이 붙어온다.
  - mongoAccountRepository.findById(); 는 기본으로 할 수 있고, mongoAccountRepository.findByEmail(); 은 인터페이스만 붙여줘도 자동으로 구현해준다.

8. Neo4j
   Neo4j 는 노드간의 연관 관계를 영속화하는데 유리한 그래프 데이터베이스입니다.

- 의존성 추가
  - org.springframework.boot:spring-boot-starter-data-neo4j
- Neo4j 설치 및 실행
  - docker run -p 7474:7474 -p 7687:7687 -d --name neo4j_boot neo4j
  - 브라우저에 콘솔이 뜬다! Localhost:7474/browser (7474는 http 용 포트 뚫은거)
  - SessionFactory 를 Autowired 받아올 수 있고, 여기서 Session session = sessionFactory.openSession(); 을 통해 세션 가져올 수 있음.
  - 만드려는 Entity 클래스에는 @NodeEntity 어노테이션 붙여줘야 함.
  - 그래프 데이터베이스에서 관계를 넣으려면 멤버 변수에 @Relationship(type=“has”) private Set<Neo4jRole> roles; 처럼 넣어준다.
  - 이제 또 똑같이 Neo4jAccountRepository extends Neo4jRepository<Neo4jAccount, Long> 으로 만들자.

9. 정리

- SQL 은 JDBCTemplate 만 가지고 있으면 바로 쓸 수 있어서 편했고, datasource를 properties에 박아서 사용해주면 된다.
- Embeded DB는 H2를 주로 설명해줬고 의존성만 추가해주면 테스트에서도 쓸 수 있어서 편했다. 웹콘솔도 있다.
- Production 과 연결하려면 히카리 DBCP 썼고, mariadb, postgresql등을 docker에 띄우고 url 세팅해줬다
- Entity Mapping
- JPA Repository 상속받아서 인터페이스로 자동 구현하기.
- hibernate.ddl-auto 로 자동으로 스키마 만들어주기
- NoSQL DB : Mongo, Redis. 설정을 바꾸려면 프로퍼티에 박아주거나, 설정해주는 빈을 만들어서 커스터마이징하면 된다.

# 17. 스프링 시큐리티

1. spring-boot-starter-security
   우선 의존성에 Thymeleaf 만들고 컨트롤러 대충 만들어보자. /hello 하면 /hello 리턴해주는 컨트롤런데, 이런 거는 그냥 @Configuration public class WebConfig implements WebMvcConfigurer { @Override public void addViewControllers(ViewControllerRegistry registry) { } } 에서 대충 해도 됨. 물론 @Model 넘기고 그런 거 할거면 컨트롤러로 하는 게 낫긴 하지..

아무튼! My 페이지에 대해서 스프링 시큐리티를 추가해보자.

- 그렇게 되면, 기존에 만들었던 메소드들이 401을 내려준다. 모든 요청들이 Security 에 대해 인증을 요구 한다.
- Basic Authentication 이랑 폼 인증이 둘다 적용된다. 다음은 Basic 인증에서 막혔음을 보여준다 .Headers = {WWW-Authenticate=[Basic realm="Realm"]
- Accept header 에 따라 이 베이직 인증이 발동한다. 즉, 요청시 원하는 응답의 형태에 따라 다르다.
- mockMvc.perform(get(“/hello”))accept(MediaType.TEXT_HTML)) 을 하면, 401이 이 아니라 302를 띄우고 로그인으로 보내준다. 스프링 부트에서 폼 로그인 기능을 자동으로 넣어주는데, 인증 정보가 없기 때문에 그 페이지로 리다이렉트 된 것이다.
- 테스트에서 깨지는 걸 해결하려면 <dependency> <groupId>org.springframework.security</groupId> <artifactId>spring-security-test</artifactId> <version>\${spring-security.version}</version><scope>test</scope> </dependency> 를 추가해주자. 그리고 @WithMockUser 어노테이션을 테스트 클래스 위에 붙이면 인증을 통과한다.

스프링 부트 시큐리티 자동 설정

- SecurityAutoConfigurtaion
  - AuthenticationEventPublisher 를 받아서 빈으로 등록해준다.
  - 요거를 대체하려면, 정말 간단하다. @Configuration public class WebSecurityConfig extends WebSecurityConfgiurerAdapter 로 Basic 인증이 구현된다. 밑에 있는 UserDetailService 도 거의 모든 서비스가 따로 구현해야 된다. 따라서 스프링 부트 시큐리티는 딱히 쓸 일 없다.
- 모든 요청에 인증이 필요함
- UserDetailsServiceAutoConfiguration
  - 기본 사용자 생성
  - Username: user
  - Password: 랜덤값 생성, 콘솔에 출력
  - Spring-security.user.name 과 password 프로퍼티에 설정 가능.

2. 시큐리티 설정 커스터마이징
   기본 설정이 모든 접근에 authorized 체크를 하는 거니까, 우리가 다시 만들어보자.
   Config 폴더 밑에 @Configuration public class WebSecurityConfig extends WebSecurityConfgiurerAdapter 의 configure 메소드를 오버라이드 해주자.

원래 메소드를 보면, 모든 요청에 대해 폼로그인과 basic http 날리는 걸로 되어 있었는데, 우리는 hello에 대해서만 formlogin 과 httpBasic 을 걸어두자.
@Override
protected void configure(HttpSecurity http) throws Exception {
http.authorizeRequests().antMatchers("/", "/hello").permitAll()
.anyRequest().authenticated()
.and()
.formLogin()
.and()
.httpBasic();
}

자, 이제 유저 관리를 해줘야 한다.
인메모리로 쓰게 h2 꽂아주고, Account에 @Entity 달아주고 리포지토리를 JPA 상속받아 만들고 AccountService 에 createAccount 메소드를 만들어서 return accountRepository.save(account); 해주자

자, 이제 UserDetailsService 를 구현해줘야 하는데, 보통 유저를 관리하는 서비스 계층에 구현하니까, AccountService implements UserDetailsService 를 하면 유저를 우리가 관리하게 된다.

그러면 loadUserByUsername 이라는 메소드를 구현해야 하는데, Optional<Account> byUsername = accountRepository.findByUsername(username); 으로 가져온 유저네임으로, 없으면 예외를 던져주자. Account account = byUsername.orElseThrow(() -> new UsernameNotFoundException(username);

그리고 UserDetails 라는 인터페잇흐를 반환해줘야 하는데, 스프링이 기본으로 구현해둔 구현체인 User 라는 클래스를 써서 return new User(account.getUsername(), account.getPassword(), authorities() ); 를 줘야 한다. Authorities 에는 권한 설정한 걸 넘겨줘야 해서,
private Collection<? extends GrantedAuthority> authorities() {
return Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"));
}

다음과 같이 주면 된다.

그리고 어플리케이션 러너로 아이디를 등록한 다음, 로그인해보면 실패한다!
Passsword Encoder 를 반드시 쓰도록 스프링이 정해놔서 그렇다. 그러므로 SecurityConfig 에 https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#core-services-password-encoding 를 참고해서 PasswordEncoder 를 등록하자. 그리고 service 에 유저 만들때 항상 encode 된 값으로 저장하자. 로그에 확인해보면, 잘 찍히는 걸 알 수 있따

많이 남았다. 폼 인증, OAuth 등등.. 스프링 시큐리티 관련해서 공부를 해봐라.

# 18. 스프링 Rest Client

1. 스프링부트 Rest Client
   스프링 부트는 레스트 클라이언트를 쉽게 쓰라고 빈을 등록해준다.
   RestTemplate과 WebClient 를 등록해주는 건 아니고, 빌더를 등록해준다.
   필요할 때마다 등록해서 사용하면된다.

RestTemplate

- Blocking I/O 기반의 Synchronous API (동기적임)
- RestTemplateAutoConfiguration
- 프로젝트에 spring-web 모듈이 있으면 RestTemplateBuilder 를 빈으로 등록해준다.
- 테스트를 해보기 위해 RestController 를 만들고, 그 안에 ApplicationRunner 를 컴포넌트로 등록해보자. 그리고 RestTemplateBuilder 를 주입받자. 뭘 하기 전에 우선 StopWatch stopWatch = new StopWatch(); 를 해서 시간을 재보자

Webclient

- Non-Blocking I/O 기반의 Asynchronous API
- WebClientAutoConfiguration
- 프로젝트에 spring-webflux 모듈이 있다면 WebClient.builder 를 빈으로 등록해줍니다.

  2.커스터마이징
  빌더를 설정할 때, 쿠키, url 등등 다양한 걸 미리 설정해놓을 수 있다. Builder.baseUrl(“http://localhost:8080”) 요런느낌으로, 전역으로 하려면 아예 빈으로 등록해버리자.
  @Bean public WebClientCustomizer webClientCustomizer() { return webClientBuilder -> webClientBuilder.baseUrl("http://localhost"); }

# 19. 그 외

캐시 메시징 Validation 이메일 전송 JTA 스프링 인티그레이션 스프링 세션 JMX 웹소켓 코틀린 …

다 비슷한 패턴이다. 의존성 추가하고, 프로퍼티 변경하고, 빈을 재정의하고.
관심있는 게 있으면 꼭 문서를 찾아보자.

# 20. 스프링 부트 운영 : Actuator

1. Actuator
   의존성 추가

- spring-boot-starter-actuator
  애플리케이션의 각종 정보를 확인할 수 있는 Endpoints
- 다양한 Endpoint 제공.
  - 그 정보는 여기에 있다. https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints
  - 빈을 보거나, 캐시를 보거나, 상태를 보거나 http 정보를 보거나 헬스체크, 애플리케이션 전반적인 정보, 리퀘스트 매핑 정보, 스케쥴된 태스크들, 세션 등등이 있다.
- JMX 또는 HTTP를 통해 접근 가능함.
- shutdown을 제외한 모든 Endpoint는 기본적으로 활성화 되어 있음. http 는 헬스랑 info 뿐.
- 프로퍼티에 설정을 통해 활성화 옵션 조절 가능.
  - management.endpoints.enabled-by-default=false
  - management.endpoint.info.enabled=true

2. JMX 와 HTTP
   JConsole 사용하기

- 콘솔에, jconsole 이라고 쳐보자. 그러면 java monitoring & management 콘솔이 뜨는데, 우리가 만든 앱에 로그인 해보자.
- 그러면 메모리, 쓰레드 등등 볼 수 있는데, Mbeans 에 가서 springframework.boot 에서 WebApplication 을 조작할 수 있는 게 몇가지 있다. 거기의 Endpoints 에는 http 에 지원하는 것보다 많은데, 빈을 볼수도 있고, 환경 설정, 등등을 조회할 수 있다.

근데 보기 별로 안좋다.
VisualVM 사용하기

- 검색해서 다운받고 설치해라.
- Extensions 에서 mbeans 추가해서 똑같이 사용가능. 조금 더 이쁠뿐이다..
  HTTP 사용하기
- /actuator health와 info를 제외한 대부분의 Endpoint가 기본적으로 비공개 상태
- 다음과 같이 설정하면 endpoint 늘어난다.
- 공개 옵션 조정
  - management.endpoints.web.exposure.include=\*
  - management.endpoints.web.exposure.exclude=env,beans

3. 스프링 부트 어드민
   스프링 부트 Actuator UI 제공.

- 어드민 서버 설정

  - 어드민 서버 용 프로젝트가 필요하다. 프로젝트 만들고de.codecentric:spring-boot-admin-starter-server:2.0.1를 추가하자.
  - 그리고 application 위에 @EnableAdminServer 어노테이션 추가하자.

- 클라이언트 설정
  - spring-boot-admin-starter-client 를 추가하자.
  - 프로퍼티에 접속할 어드민 서버 주소 적어주자. spring.boot.admin.client.url=http://localhost:8080
  - management.endpoints.web.exposure.include=\*

이제 localhost:8080 들어가보면, 어드민이 떠있다. 들어가면 모니터링 부터 로그 레벨 설정까지 할 수 있다. 여기서 꼭 Security 걸어줘야 한다!!!! 중요한 정보들이 많으니까 주의하자.

# 21. 끝

스프링 부트 원리

- 의존성 관리
- 자동 설정 내장
- 웹 서버
- JAR 패키징

스프링 부트 활용

- 스프링 부트 핵심 기능
- 다양한 기술 연동

스프링 부트 운영

- Actuator
- 스프링 부트 어드민
