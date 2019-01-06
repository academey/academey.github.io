---
layout: post
title:  "토비의 스프링 3.1"
author: academey
categories: spring
cover: "/assets/toby-spring/cover.jpg"
---
# 0. 들어가며
토비의 스프링 책을 보며 정리한 글입니다. 원문 내용이 인용된 부분이 많아 문제가 있다면 언제든 연락주시면 지우겠습니다. 누군가의 스프링 학습에 도움이 되길 바라며 공유합니다.

# 1장. 오브젝트와 의존 관계
유저라는 데이터를 다뤄야 한다. 이 때, UserDao 라는 DB 연결 및 데이터 가져오는 클래스를 만들고 싶다.
어찌저찌 구현은 했으나 Naver 의 DB, Daum의 DB 등 다양한 디비와 연결하고 싶으면 어떡해야 하냐.

* 상속을 통해 추상 메소드를 서브 클래스에서 필요에 맞게 구현하는 방법 -> 템플릿 메소드 패턴 (abstract method인 getConnection을 자녀 클래스에서 필요에 맞게 구현한다)
* 서브클래스에서 사용하는 클래스의 구체적인 오브젝트 생성 방법 결정 -> 팩토리 메소드 패턴 (NUserDao 에서 UserDao 를 상속받고, Connection 을 네이버 것으로 맞춘다.)
* 그러나, 상속에서는 해결하지 못하는 문제가 있다. 다중 상속을 허용하지 않으며, 커넥션 객체를 가져오는 방법을 위해 UserDao 를 분리했다면, 트랜잭션을 넣고 싶으면 또 다른 상속을 써야 하고 다르게 분리하면 NUserDaoWithTransaction, …. 등등 무수히 많은 문제가 생긴다.
* 아예 화끈하게 분리해보자. 커넥션을 만드는 쪽과 커넥션을 사용하는 부분으로 UserDao 와 SimpleConnectionMaker 로. 이 때 클래스를 하게 되면 종속되므로 인터페이스를 통해 해결할 수 있다.
* A가 B를 써야 하는 클라 -> 서비스 관계일 때, A에서 B를 설정하지 않고, A를 사용하는 곳에서 어떤 B를 쓸지 결정한다. 즉, 의존성을 클라 단에 최대한 배치함으로써 오브젝트와 오브젝트를 동적으로 연결해준다. 즉, 클래스와 클래스의 연관성 없이 이용하게 되는 것이다.
* 다른 관심사가 엮여있으면, 확장성이 떨어진다.
* OCP 가 여기서 적용된 것이다. UserDao는 DB 연결의 확장은 오픈되어 있으나, 내부의 핵심 기능 코드들은 변화에 영향을 받지 않는다.
* 높은 응집도, 낮은 결합력
* 전략 패턴 : 자신의 기능 맥락(context)에서 독립적인 책임으로 분리 가능한 기능을 인터페이스를 통해 통째로 외부로 분리시키고, 클라이언트의 필요에 따라 바꿔서 사용할 수 있는 디자인 패턴.
* UserDao : 전략 패턴의 컨텍스트
* DB 연결 방식 : 독립적인 책임으로 분리 가능한 기능
* 인터페이스 : ConnectionMaker
* UserDaoTest : 클라이언트
* 팩토리 패턴 : 객체의 생성 방법을 결정하고 오브젝트를 돌려주는 오브젝트
* 즉, UserDao 에서 결정하지 말고, DaoFactory에서 오브젝트 결정시켜야 함. UserDao Dao = new DaoFactory().userDao();
* 애플리케이션 컴포넌트와 애플리케이션 구조를 결정하는 오브젝트를 분리하자.
* 제어의 역전 : 오브젝트가 사용될 오브젝트를 정하고, 그 오브젝트가 오브젝트를 부르는 구조가 모든 종류의 작업을 사용하는 쪽에서 제어하는 구조이다. 이 구조를 뒤집자. 제어의 역전에서는 오브젝트가 자기가 사용할 오브젝트를 선택하지 않고, 생성하지 않는다. 모든 오브젝트는 이 권한을 위임받은 특별한 오브젝트에 의해 결정되고 만들어진다.
* 애플리케이션 코드가 라이브러리를 이용하지만,
* 프레임워크는 프레임워크가 애플리케이션 코드를 이용한다. 프레임워크가 흐름을 주도하기 때문.

이제 위의 개념에 스프링을 적용해보자.

* 빈 : 스프링이 제어권을 가지고 만들고 관계를 부여하는 오브젝트를 빈이라고 한다. 스프링 빈은 스프링 컨테이너가 생성과 관계설정, 사용 등을 제어해주는 IoC가 적용된 오브젝트다.
* 빈팩토리 : 빈의 제어를 담당하는 IoC 오브젝트를 빈팩토리라고 부른다. 스프링은 그걸 확장한 Application context를 이용한다. 두 가지가 비슷하다. 애플리케이션 컨텍스트는 앱 전반에 걸쳐 모든 구성요소의 제어 작업을 담당한다는 의미가 좀 더 부각된다.
* IoC 컨테이너 : IoC 방식으로 빈을 관리한다는 의미에서 애플리케이션 컨텍스트 혹은 빈 팩토리를 뜻한다. 해당 컨테이너는 하나의 앱에서 여러개가 만들어져 사용되기도 한다.
* 설정 정보 : Applcation Context 또는 빈 팩토리가 IoC를 적용하기 위해 사용하는 메타정보.
* @Configuration : 애플리케이션 컨텍스트가 사용할 설정정보이다.
* @Bean : userDao 메소드는 UserDao 오브젝트를 생성하고 초기화해서 돌려주는 것이니, Application context가 관리하는 빈이다.
* 오브젝트 팩토리에 대응하는 것이 스프링의 ApplcationContext == IoC 컨테이너 == 스프링컨테이너 == 빈 팩토리
* DaoFactory를 오브젝트 팩토리로 직접 사용하는 것보다 ApplcationContext를 사용하는 이유
    * 클라단에서 구체적인 팩토리 클래스를 알 필요가 없다.
    * 오브젝트 생성과 관계설정만이 아니라, 만들어지는 방식과 시점, 인터셉팅 등 다양한 기능 설정 가능
    * 빈의 이름을 이용해 찾거나 빈의 타입으로 찾거나 특별한 에노테이션 있는 걸 찾을 수도 있음.
    * 가장 중요한 건, 오브젝트 팩토리에서 생성하는 것은 동일한 오브젝트가 아니다. 매 번 다른 녀석들인데, ApplcationContext에서 빈을 가져올 경우, 동일한 오브젝트를 돌려준다.
    * 즉, ApplcationContext는 싱글톤을 저장하고 관리하는 싱글톤 레지스트리다.
        * 각 로직을 담당한 오브젝트를 새로 만들면 부하가 걸리기 때문이다. 그래서 서비스 오브젝트를 만들어서 싱글톤으로 관리한다. 여러 스레드에서 하나의 오브젝트를 공유해서 동시에 사용한다.
        * 자바 자체의 싱글톤은 많은 단점이 있다. 상속 및 테스트가 안 되어서 이점을 다 포기하는 경우. 따라서 스프링은 직접 싱글톤 형태의 오브젝트를 만들고 관리하는 기능을 제공한다. 그게 바로 싱글톤 레지스트리다.
* Bean으로 등록 이후에 public 생성자로 인스턴스를 사용할 수도 있고, 이로 인해 손쉬운 테스트도 가능하다.
* 객체 지향과 여러 디자인 패턴에 적용할 수 있는 구조가 되는 것이다.
* 싱글톤은 기본적으로 Write가 없는 Stateless 상태여야 한다. 필드의 값을 변경하고 유지하는 상태유지, Stateful한 방식으로 만들면 안된다. 여러 사용자가 읽고 쓰면 난리난다.
* 스프링이 관리하는 빈이라면 싱글톤 클래스의 인스턴스 변수로 사용해도 좋다.
* 결국, DI를 원하는 오브젝트라면 자기 자신이 컨테이너가 관리하는 빈이 되어야 한다.

의존 관계 주입(DI)
* 의존관계란 A -> B, B의 메소드가 변하면 A가 영향을 받는 관계를 의미한다.
* 인터페이스를 의존했으니 결합력이 줄어들어서 올바른 설계이다. 그런데 이 때, 런타임에 연결되는 클래스와의 연결관계는 그럼 어떻게 되는 것인가?
* 런타임 시에 의존관계를 맺는 대상, 실제 사용대상인 오브젝트를 의존 오브젝트라고 한다.
* 즉, 의존관계 주입은 이렇게 구체적인 의존 오브젝트와 이를 사용하는 주체, 클라이언트 오브젝트와 런타임 시에 연결해주는 작업을 의미한다.
* 클래스 모델에 런타임 시점의 의존관계가 드러나지 않는다. 즉, 인터페이스에만 의존한다.
* 런타임 시점의 의존 관계는 컨테이너나 팩토리 같은 제 3자가 결정. (DaoFactory, 애플리케이션 컨텍스트, 빈 팩토리, IoC 컨테이너 등)
* 의존관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 주입해줌으로써 결정된다.
* 의존관계 검색과 주입: 자신이 필요로 하는 의존 오브젝트를 능동적으로 찾는다.
* 의존관계 주입 vs 의존관계 검색
* 의존관계 검색은 ApplicationContext등 코드가 난잡해진다. 그런데 결국 의존성 주입을 받기 위해서는 어느 순간에는 필요하다.
* 또, 의존관계 주입 시에는 반드시 자신이 스프링의 빈이어야 하는데(자신이 호출될 때, IoC가 적용되기 위해 스프링에 등록되어야 하기 때문이다 컨테이너가 UserDao에 ConnectionMaker를 주입해주기 위해선 해당 클래스들에 대해 초기화와 생성 권한을 가져야 하기 때문이다.)
* 의존관계 검색 시에는 그럴 필요가 없다. 어딘가에서 new UserDao()한 다음에 ConnectMaker 빈만 찾아서 꽂아주면 된다.
* 결국, UserDao 의 Constructor에 파라미터로 넘겨온 ConnectionMaker를 넣을 것이냐(의존관계 주입) ApplicationContext에서 찾아온 ConnectionMaker Bean을 넣을 것이냐(의존관계 검색) 의 차이이다.
* 전자로 하면 테스트 코드 못 짜고, 싱글톤 유지해야 되고 등등 문제가 많음.
부가적인 기능 추가
* DI 가 쌓여진 클래스에 부가적인 기능을 추가하고 싶다면, 해당 클래스를 주입하는 클래스를 만들어서 호출하는 인터페이스의 메소드 위에 해당하는 로직을 추가한다.
* public class CountingConnectionMaker implements ConnectionMaker { int counter = 0; prviate ConnectionMake realConnectionMaker; } 느낌으로다가.
* DI를 주입하기 위해 클라이언트 단에서 다른 빈을 불러오기만(DL 의존성 로드) 함으로써 런타임 의존관계는 이전 상태로 복구된다.
XML DI 설정
* <beans> 가 @Configuration, <bean>이 @Bean을 대응한다.
* @Bean 메소드를 통해 빈의 이름, 빈의 클래스, 빈의 의존 오브젝트를 넣어줄 수 있다.
* 의존관계가 있는 빈을 만들 때 프로퍼티를 넣어줘야 한다. 이 때, set을 제외한 나머지 이름(프로퍼티의 이름)이 name이 되고, ref는 오브젝트의 빈 이름이다.
* <bean id =“userDao” class=“springbok.dao.UserDao”> <property name(프로퍼티 이름)=“connectionMaker” ref(주입할 빈의 ID)=“connectionMaker”> </bean>
* 이제는 AnnotationConfigApplicationContext (@Configuration) 이 아니라 GenericXmlApplicationContext를 이용해 애플리케이션 컨택스트를 생성한다. 그리고 XML 설정 파일을 불러올 수 있는 ClassPathXmlApplicationContext를 이용해 만들수도 있다. springbook.user.dao 패키지 안의 설정 파일을 사용하기위해서이다.
프로퍼티 값의 주입
* Datasource 에 프로퍼티 값을 넣고 싶다면 <property name=“driverClass” value=“com.mysql.jdbc.Driver” /> 처럼 넣는다.
* 그런데 이 때, 해당 값이 클래스로 어떻게 변환되는가? 스프링이 수정자 메소드 파라미터 타입을 참고해 적절한 형태로 변환시켜 넣어준다. Class driveClass = Class.forName(“com.mysql.jdbc.Driver”); 을 지정해주는 것이다.


# 2장. 테스트

단위테스트
* 다른 코드들은 신경 쓰지 않고, 오직 하나의 관심에만 집중하자.
* 일단은 정상동작하는 코드를 만들자. 그리고 확신을 가지고 코드를 변경하자. 점진적인 개발이 좋다.

JUnit
* Main 메소드는 제어권을 가진다는 의미인데, 테스트 코드가 거기 있는건 옳지 않다. 이제는 프레임워크의 손에 맡겨보자.
* public으로 선언하기
* @Test 어노테이션 붙이기
* assertThat(user.getName(), is(user2.getName()));
* 첫 번째 파라미터의 값을 뒤에 나오는 matcher와 비교해 일치하면 넘어간다.
* Matcher 의 종류는 다양하다. not(user2); userArray, not(hasItem(user2));
* assertThat(test, is(true)); 또는 assetTrue(isBoolean); 또는 assertThat(value, either(is(nullValue()).or(is(this.context));
* Exception Throw를 테스트하고 싶다면? 예상되는 예외 클래스를 지정해준다.
* @Test(expected=EmptyResultDataAccessException.class) 따라서 실제 UserDao 코드에 if(user==null) throw new EmptyResultDataAcessException(); 을 던져줘야 한다.
* 네거티브 테스트 먼저 작성해라.
* 테스트는 수도 코드를 작성하는 것과 같다. 어떤 조건을 가지고 뭘 할 때 어떤 결과가 나온다. 잘 작성된 하나의 기능 정의서가 된다.
TDD
* 테스트를 작성하고 이를 성공시키는 코드를 만드는 작업의 주기를 최대한 짧게 가져가야 한다.
* 코드에 대한 피드백과 확신을 가져 자신감과 여유를 줄 수 있다.
* 간신히 찾아낸 버그의 추억 -> 진작에 충분한 테스트를 했다면 쉽게 찾아냈을 삽질
* @Before 을 써서 JUnit 테스트 셋업을 하자. @Before public void setUp() { ApplicationContext context = new GenericXmlApplicationContext(“applicationContext.xml”); this.dao = context.getBean(“userDao”, userDao.class); -> 여기서 dao를 인스턴스 변수로 만들었다.
* 주의할 것은, JUnit에서는 매 테스트 메소드를 실행할 때마다 클래스 오브젝트를 각각 만든다. 즉, 테스트 메소드가 2개 있으면, 2개 만들고, 2번 @Before 실행하고, 함수 실행하고, 2번 @After 실행한다.
* ??? 과부하 걸리지 않을까요?
* -> 테스트는 가능한 독립적인 오브젝트가 원칙이다. 그러나 너무 많은 시간과 자원이 소모될 경우 테스트 전체가 공유하는 오브젝트를 만들기도 한다. 이 때도 일관성을 유지해야 한다. 즉, 빈은 싱글톤으로 만들었기 때문에 상태를 가지지 않는다. UserDao 빈을 가져다가 add, get 메소드 등을 사용한다고 해서 UserDao 빈의 상태가 바뀌지 않으므로 애플리케이션 컨텍스트는 한 번만 만들고 여러 테스트가 공유해서 사용해도 된다.
* 이렇게 참조할 애플리케이션 컨텍스트들을 저장해두자. -> 테스트를 위한 애플리케이션 컨텍스트 관리
픽스쳐
* Fixture : 테스트를 수행하는데 필요한 정보나 오브젝트
* 여러 테스트에서 반복적으로 사용되기 때문에 @Before에서 생성해놓는 것이 좋다. dao가 대표적인 픽스쳐다.
* 심지어 모든 곳에서 사용되지 않는 변수들이라도 한 곳에 모아서 초기화 시킨 다음 공통으로 사용하는 게 관리하기 좋다. 따라서 user1, user2, user3도 인스턴스화 시킨 후 setUp에서 초기화시킨다.

테스트를 위한 애플리케이션 컨텍스트 관리
* 스프링은 JUnit 용 Test Context Framework 를 지원한다. 몇개의 어노테이션을 쓰면 알아서 해준다.
* @RunWith(SpringJUnit4ClassRunner.class) @ContextConfiguration(locations=“/applicationContext.xml”) public class UserDaoTest { @Autowired private ApplicationContext context; }
    * 참고로 ApplicationContext 자체는 자동으로 빈으로 등록해놓기 때문에 ApplicationContext를 가져올 수 있는 것이다.
* @RunWith(SpringJUnit4ClassRunner.class) => 스프링의 테스트 컨텍스트 프레임워크의 JUnit 확장기능 지정
* @ContextConfiguration(locations=“/applicationContext.xml”) -> 테스트 컨텍스트가 자동으로 만들어줄 애플리케이션 컨텍스트의 위치 지정
* @Autowired 를 통해 테스트 오브젝트가 만들어지면 스프링 테스트 컨텍스트에 의해 자동으로 주입된다.
* 다른 클래스도 해당 컨텍스트를 사용한다면, 한 번만 로드해서 알아서 꽂아준다.
* @Autowired 가 붙은 인스턴스 변수가 있으면, 테스트 컨택스트 프레임워크는 변수 타입과 일치하는 컨텍스트 내의 빈을 찾는다. 일반적으로 수정자가 필요한데, 이 경우는 없어도 주입 된다. 또 별도의 DI설정 없이 필드의 타입 정보를 이용해 빈을 자동으로 가져올 수 있는데, 이런 방법을 타입에 의한 자동 와이어링이라고 한다.
* 그런데 앞의 경우는, applicationContext.xml에 정의된 빈이 아니라 ApplicationContext라는 타입의 변수인데, 어떻게 어플리케이션 컨텍스트가 DI된 것인가? 바로, 스프링 애플리케이션 컨택스트는 초기화할 때 자기 자신도 빈으로 등록한다. 즉, ApplicationContext 타입의 빈이 존재하고 DI도 가능하다.
* 따라서, @Autowired UserDao dao;  라고 하면, 어플리케이션 컨텍스트를 DI 받아서 DL 방식으로 UserDao를 가져올 때보다 테스트코드가 깔끔해졌다. @Autowired를 지정하기만 하면 어떤 빈이든 등록만 되어 있고 타입만 같으면 가져올 수 있다. 타입이 같은 게 두개 있다면, 속성 이름과 bean id가 같은 걸로 찾아온다
* 여기서 의문이 든다. SimpleDriverDataSource 쓸 건데, 굳이 Datasource 인터페이스를 써야 하는가?
* 절대 바뀌지 않는 건 없다. DI를 통해 주입하는 게 클래스 내부의 함수를 변경하는 것보다 훨씬 단순하고 쉽다.
* 클래스의 구현 방식이 바뀌지 않다고 해도 인터페이스를 두면 중간 서비스를 끼울 수 있다.
* 효율적인 테스트를 위해 주입 받는 게 훨씬 좋다. 하나의 빈만 구현해서 사용하기 때문이다.
테스트 코드에 의한 DI
* 테스트용 DB에 연결하기 위해서 어떻게 할 것인가?
* DataSource 인터페이스에 SingleConnectionDatasource를 구현한다. 커넥션이 하나만 있어서 다중 사용자는 안되지만, 테스트는 순차적으로 진행되기 때문에 문제 없다.
* 대신, @Before 에서 해당 datasource 를 직접 구현해주고, 기존에 등록된 애플리케이션 컨텍스트가 변경된다는 것을 테스트 컨텍스트 프레임워크에 알려줘야 한다.
* 따라서 테스트 클래스 위에 @DirtiesContext 를 붙이고, 빈에서 가져온 userDao 오브젝트에 setDataSource 를 시킨다. 이렇게 @DirtiesContext가 붙어 있으면 다른 테스트 클래스에서 해당 애플리케이션 컨텍스트를 공유하지 않는다.
* -> 그냥 테스트 전용 applicationContext.xml 을 만들자.
* 그리고 테스트 클래스의 @ContextConfiguration(locations=“test_applicationContext.xml”) 로 변경한다.

# 3장. 템플릿

확장에는 열려있고, 변경에는 닫혀 있는 형태. 개방폐쇄원칙(OCP) 가 다른 목적에 의해 독립적으로 변경할 수 있는 효율적인 구조를 만들어준다.
템플릿이란 이렇게 바뀌는 성질이 다른 코드 중에서 변경이 거의 일어나지 않는 일정한 패턴의 부분들을 독립시켜서 효과적으로 활용하는 방법이다.

예외처리 기능을 갖춘 DAO
* Db connection 시 예외가 발생하면, getConnection을 닫지 않아서 커넥션 풀이 다 사용이 되고 리소스가 모자라진다.
* 따라서 반드시 반환을 하도록 구조가 짜여있어야 한다. finally는 무조건 실행되는 블록이므로 거기서 닫자.
* Close 메소드를 실행할 때도 예외가 발생할 수 있으니 요것도 try를 해줘야 한다.
* 그럼 이 더러운 try catch finally 문을 모든 걸 다 하고 있을거냐?? 안된다.

분리와 재사용을 위한 디자인 패턴 적용
* 메소드 추출 : try { c= dataSource.getConnection; ps = makeStatement(c); -> 변하는 부분을 메소드로 추출하자.} 분리시킨 메소드를 공통으로 사용하는 게 아니라, 이건 반대로 분리된 걸 확장될 때마다 새롭게 만들어야 한다.
* 이걸 상속을 통해 해결하면 makeStatement를 abstract protected 로 추상 메소드화 시킨 후 overWrite 한다.
* 그러나, 이렇게 하면 커넥션을 이용하려는 메소드마다 모두 상속해서 클래스를 만들어야 한다. 또한 클래스 레벨에서 관계가 결정된다는 점에서 너무 유연하지 않다.
* 오브젝트를 아예 분리하고, 인터페이스를 통해 의존하는 전략 패턴을 사용하자.
* 컨텍스트가 전략 인터페이스를 연결받고, 전략 인터페이스를 통해 독립된 전략 클래스에 위임하는 것이다.
* 즉, 컨텍스트는 DB 커넥션 가져오고, PreparedStatement를 만들어줄 외부 기능 호출하고, 전달받은 PreparedStatement 실행하고, 예외 발생하면 던지고, 커넥션을 닫아주는 역할을 한다.
* PreparedStatement를 만들어주는 외부 기능이 바로 전략 인터페이스다. 이걸 인터페이스로 만들고, 그 메소드에서 PreparedStatement 를 생성할 때 이 컨텍스트 내에서 만들어준 DB 커넥션을 전달해야 한다. 커넥션이 없으면 PreparedStatement도 못만드니까 말이다.
* 따라서 컨텍스트가 만든 Connection을 전달받아서 PreparedStatement 를 만들고 만들어진 PreparedStatement를 오브젝트에 돌려준다.
* Public interface StatementStrategy { PreparedStatement makePreparedStatement(Connection c) throws SQLException; }
* Class DeleteAllStatement implements StatementStrategy { } 하고, StatementStrategy strategy = new DeleteAllStatement(); ps = strategy.makePreparedStatemnet(); ps.excuteUpdate(); 잠시만! 이러면 클래스를 고정하고 왜 클라단에서 업데이트 치냐? 관심사가 분리되어야 하는 OCP전략에 들어 맞지 않다.
* 결국, 클라이언트(UserDaoTest) 단에서 어떤 전략을 사용할지 결정해야 한다.  이렇게 전략 오브젝트 생성과 컨텍스트로 전달을 담당하는 책임을 분리시킨 것이 ObjectFactory이며, 이걸 일반화 시킨 것이 DI(의존관계 주입)이다.
* 컨텍스트에 해당하는 부분을 별도의 메소드로 독립시키자. 클라이언트는 DeleteAllStatement 같은 확고한 전략 클래스 오브젝트를 컨텍스트의 메소드를 호출할 때 전달시켜줘야 한다. 즉, void jdbcContextWithStatementStrategy(StatementStrategy stmt) throws SQLException {} 과 같은 정의를 해줘야 한다.
* 이제 StatementStrategy st = new DeleteAllStatement(); jdbcContextWithStatementStrategy(st); 로 책임을 분리하자.
* 그런데 아직 아쉬운 점이 있다.
* DAO 메소드마다 새로운 StatementStrategy 구현 클래스를 만들어야 한다.(쿼리를 다 담고 있어야 하니까) 그러면 기존 UserDao 보다 클래스 파일의 개수가 많아진다. 런타임 시에 다이나믹한 DI를 제외하면 나은 게 없다.
* 또한, DAO 메소드에서 전달하는 User와 같은 부가정보가 있을 경우, 이를 위해 오브젝트를 전달받는 생성자와 인스턴스 변수를 만들어줘야 한다. 번거롭다.
* 이 두가지를 해결하자.
로컬 클래스
* StatetmentStrategy 전략 클래스를 매번 독립된 파일로 만들지 말고 UserDao 안에 내부 클래스로 정의해버리는 것이다.
* Public void add(User user) { class AddStatement implements StatementStrategy {} StatementStrategy st = new AddStatement(user); } 클래스 파일이 줄고, 생성 로직을 이해하기도 좋다. 또한 User 에도 접근이 가능하다.
* 그럼 이제, 이름도 없는 익명 내부 클래스를 박아주는 건 어떨까. 상속할 클래스나 인터페이스를 생성자 대신 사용해 다음과 같이 사용한다. New 인터페이스이름() { 클래스 본문 }; 우리가 쓰는 건 클래스 재사용 안 하니까 여기서 쓰기 좋다
* StatementStrategy st = new StatementStrategy() { public PreparedStatemnet makePreparedStatement(Connection c) {}};
* 이제 더 줄이자. 어차피 다시 안 쓸 거니까 그냥 익명 클래스로 바로 담아 버리자. jdbcContextWithStatementStrategy( new StatementStrategy ……. );
JdbcContext 의 분리
* 이제 jdbcContextWithStatementStrategy 를 따로 클래스로 분리하자. 다른 Dao 에서도 사용해야지.
* JdbcContext 라는 클래스를 만들고 DataSource 주입 받는 걸 옮기고, workWithStatementStrategy 로 메소드 명 변경하자.
* 그리고 UserDao 에 private JdbcContext 라는 것에 설정자도 넣어둬서 DI 를 해두자.
* 그런데 기존에 인터페이스를 거쳐 런타임 시에 오브젝트를 주입하는 방식이 아닌, 클래스 레벨에서 의존관계가 결정된다. 이렇게 해도 될까?
* 꼭, 그럴 필요는 없다. DI의 개념을 충실히 따르면 인터페이스를 두고 의존관계가 고정되지 않게 해야 한다. 그러나 스프링의 DI는 객체를 생성하고 IoC 개념을 포괄한다. 그런 의미에서 스프링으로 의존성을 주입했다는 점에서 충분하다.
* 싱글톤 레지스트리의 싱글톤 빈이 되고,
* DI를 통해 다른 빈에 의존하고 있기 때문이다.
* 인터페이스를 사용하지 않은 이유는, UserDao와 JdbcContext가 매우 응집도가 강하기 때문이다. 항상 UserDao는 JdbcContext를 써야 한다. DataSource와 달리 JdbcContext는 바뀔일이 없으므로 인터페이스를 두지 않고 강력한 결합 관계를 만드는 것이다.
코드를 이용한 수동 DI
* 만약, JdbcContext를 빈으로 만들어서 주입하지 않고, 수동 DI를 적용하려면?
* 싱글톤을 포기해야 한다. Dao 마다 인스턴스 하나씩 생성해줘야 하는데, 내부 상태값 저장하는 게 없어서 메모리 별로 안 먹는다. 그러면 안에 의존하고 있던 DataSource 타입 빈을 어떻게 넣어주냐?
* 이 초기화를 UserDao 에서 책임져야 한다. 이제는 DI Container 역할을 해야 한다. 즉, UserDao 빈에 DataSource 빈을 주입 받은 후, 수동으로 JdbcContext를 설정하면서 넣어준다.
* setDataSource(DataSource dataSource) { this.jdbcContext = new JdbcContext(); this.jdbcContext.setDataSource(dataSource); -> DI가 일어났다. this.dataSource = dataSource; }
* 두 경우 각각 장단점이 있다.
* 인터페이스를 사용하지 않아 실제 의존관계가 설정파일에 명확하게 드러난다는 장점이 있다. 하지만 DI 원칙에 부합하지 않는다.
* 반면 수동으로 DI하면 JdbcContext가 아닌 UserDao 내부에서 의존성이 만들어지고 관계를 드러내지 않는다. 다만, 싱글톤이 아니고 부가적인 코드가 더 필요하다.
템플릿과 콜백
* 복잡하지만 바뀌지 않는 일정한 패턴을 가지고 일부분만 자주 바꿔야할 경우, 전략 패턴의 기본 구조에 익명 내부 클래스를 활용했다.
* 이런 방식을 스프링에서 템플릿/콜백 패턴이라고 한다.
* 전략 패턴(동일한 Interface)을 템플릿, 익명 내부 클래스로 만들어지는 오브젝트, 전략을 콜백이라고 부른다.
콜백의 분리와 재활용
* 그런데 익명 내부 클래스는 아직도 불편하다. 재사용 가능한 코드만 꺼내보자. new StatementStrategy() { public PreaparedStatement makePreparedStatement(Connection c) throws SQLException { return c.prepareStatement(“delete from users”); }
* 변하는 건 저것밖에 없다. 따라서 분리하자.
* public void executeSql(final String query) throws Exception { this.jdbcContext.workWithStatementStrategy( new StatementStrategy() { public PreparedStatement makePreparedStatement(Connection c) throws SQLException { return c.prepareStatement(query)}}); }
* public void deleteAll() throws SQLException { executeSql(“delete from users”); } 결국 한 문장으로 압축이 된다.
* 그런데, 요 메소드를 UserDao만 사용하기 아깝다. 템플릿 클래스에서 사용해도 될 것 같다. executeSql 메소드를 옮기자.
* 그러면  public void deleteAll() throws SQLException { this.jdbcContext.executeSql(“delete from users”); } 이 된다. 결국 관심사가 분리된 모습이다.
* 중복된 코드는 메소드로 분리하자. 그 중 일부는 필요에 따라 바꿔야하면 인터페이스를 사이에 두고 분리해서 전략 패턴을 적용하고 DI로 의존관계를 관리한다. 그런데 바뀌는 부분이 한 애플리케이션 내에 동시에 여러 종류가 만들어질 수 있다면, 템플릿/콜백 패턴을 적용하는 것을 고려하자.
* Try/catch/finally 가 많으면 템플릿/콜백을 사용하자.
중복의 제거와 템플릿/콜백 설계
* 파일을 읽어서 연산하는 작업이 필요하다면?
* 열고, 읽고, 닫고 계속 try/catch/finally 문 쓸거냐? 절대 안된다…...
* 읽어서 연산하는 작업을 인터페이스로 뽑아서, 콜백으로 넘겨주자.
* 콜백 인터페이스는 Public interface BufferReaderCallback { Integer doSomethingWithReader(BufferReader br) throws IOException; }
* 템플릿 메소드는 public Integer fileReadTemplate(String filePath, BufferReaderCallback callback) { BufferReader br = null; try { br = new BufferReader(new FileReader(filePath; int ret = callback.doSomething(); }
* 그렇게 되면 클라이언트 쪽 코드는 fileReadTemplate(filepath, new BufferedReaderCallback() { public Inetger doSomethingWithReader(BufferReader br) { Integer multiply = 1; string line = null; while((line = br.readline()) != null){ multiply *= Integer.valueOf(line); }
* 결국, 저 안에만 뽑아내면 된다.
* 인터페이스를 따로 빼보자. 초기값과 라인에 대해 수행하는 메소드만 있으면 된다. Public interface LineCallback { Integer doSomethingWithLine(String line, Integer value); }
* 저 인터페이스를 사용하는 템플릿을 만들자. Public Integer lineReadTemplate(String filepath,  LineCallback callback, int initVal) { 파일 읽고 Integer res = initVal; while( ) { res = callback.doSomethingWithLine(line, res);  내부적으로 연산함.} return res; }
* 이제는, 연산하는 부분만 넣어주면 된다. lineReadTemplate(filepath, new LineCallback() { public Integer doSomethingWithLine(String line, Integer value) { return value * Integer.valueOf(line); } });
* 관심사가 철저하게 분리되는 모습이다.
* 타입 형을 다양하게 가져가고 싶으면, 제너릭스를 이용하자.
* 정의 할 때 Interface LineCallback<T> { T doSomethingWithLine(String line, T value); } public <T> T lineReadTemplate(String filepath, LineCallback<T> callback, T initVal);
* 선언할 때 lineReadTemplate(filePath, new LineCallback<String>() { public String doSomethingWithLine }
스프링의 JdbcTemplate
* 우리가 썼던 StatementStrategy 인터페이스의 makePreparedStatement() 메소드를 사용했던 콜백과 JdbcTemplate의 PreparedStatementCreator 인터페이스의 createPreparedStatement() 는 구조가 일치한다.
* 요렇게 만든걸 JdbcTemplate.update 메소드에 콜백으로 넘길 수도 있고, SQL 문장만 update에 넘기는 메소드도 있다. 또한, 저 콜백에서 파라미터 바인딩하는 게 아니라, update 메소드 뒤에 가변 인수들을 붙이면서 바인딩 할 수 있다. this.jdbcTemplate.update(“insert into users(id) values(?)”, user.getId()); 요런 식으로 말이다.
* 다음은, 쿼리를 실행하고 결과값을 ResultSet 에 담아 가져오자.
* 이런 작업 흐름에서 필요한 건 PreparedStatementCreator 콜백과 ResultSetExtractor를 콜백으로 받는 query() 메소드다.
* PreparedStatementCreator : 템플릿으로부터 Connection을 받고 PreparedStatement를 돌려준다.
* ResultSetExtractor : 템플릿으로부터 ResultSet을 받고 거기서 추출한 결과를 돌려준다.
* this.jdbcTemplate.query(new PreparedStatementCreator(){}, new ResultSetExtractor<Integer>(public Integer extractData(ResultSet rs) { rs.next(); return rs.getInt(1); }) {}); 이런 식이다.
* 클라 -> 템플릿 -> 콜백의 구조이다.
* 이렇게 Integer 결과값을 가져올 수 있는 콜백을 내장하는 queryForInt() 라는 메소드가 있다. this.jdbcTemplate.queryForInt(“select count(*) from users;”); 정말 간단하게 한 줄로 변한다.
queryForObject
* ResultSet의 결과를 User 오브젝트로 만들어서 넣어줘야 한다. 이를 위해 RowMapper 콜백을 사용한다. RowMapper와 ResultSetExtractor 둘 다 템플릿으로부터 ResultSet을 받고 필요한 정보를 추출해서 전달한다.
* this.jdbcTemplate.queryForObject(“select * from users wheree id=“?”, new Object[] {id}, New RowMapper<User>() { public User mapRow(ResultSet rs, int rowNum) { User user = new User();
* SQL의 ? 에 바인딩할 id 값을 전달한다.SQL에 바인딩할 파라미터 값이 id이다. 가변인자 대신 배열을 이용해서 전달했다.
Query()
* 여러개의 로우가 결과로 나오는 경우. query의 리턴타입은 List<T>이다.
* List<User> getAll() { return this.jdbcTemplate.query(“select * from users order by id”, new RowMapper<User>() { public User mapRow(ResultSet rs, int rowNum) throws SQLException { User user = new User(rs.getString(“id”), rs.getString(“name”)); return user;}
* RowMapper 가 중복되니까 UserMapper 라는 콜백 오브젝트로 인스턴스 변수에 저장하자.
최후의 개선
* 요걸 더 이쁘게 만들고 싶다면, UserMapper를 독립된 빈으로 만들고 XML 설정에 필드 이름과 오브젝트 프로퍼티를 가지게 함으로써 UserMapper 를 분리해 코트 수정 없이 변경할 수 있다.
* 또한 SQL 쿼리를 최적화하기 위해 쿼리를 외부 리소스에 담아서 조회하도록 만든다.

# 4장. 예외
* 익셉션 받은 다음에 프린트만 하는거, 아무짓도 안하는 거 다 나중에 물밀려 온다. 절대 그러지마라. 차라리 시스템을 꺼버려라.
* Throws Exception을 모든 메소드에 붙여서 날리는 기계적인 개발은 정말 무책임하다.
* 자바에서 throw를 통해 발생시킬 수 있는 예외는 3가지가 있다.
* Error: 시스템에 비정상적인 상황 발생. OutOfMemmory 혹인 ThreadDeath
* Checked Exception(체크 예외) : RuntimeException을 상속하지 않음. 사용할 메소드가 체크 예외를 던지면 반드시 catch문으로 잡든지 throw로 다시 던져야 한다. 안그러면 컴파일 에러남.
* UnChecked Exception(체크되지 않은 예외) (RuntimeException이라고도 불림) : RuntimeException을 상속함. 명시적인 예외처리를 강제하지 않기 때문에 언체크 예외라고 불린다. 물론 명시적으로 잡아줘도 되긴 한다. 런타임 예외는 주로 프로그래밍 오류가 있을 때 발생하도록 의도된 것들이다. 눌포인터에러나 IllegalArgumentException 들이 있다. 피할 수 있지만 개발자가 부주의해서 발생하는 경우에 발생하도록 만든 것이다.
예외 처리 방법
* 예외 복구 : 예외상황을 파악하고 문제를 해결해서 정상 상태로 돌려 놓는 것. SQL Injection이 실패해서 여러번 시도하거나, 파일이 없어서 유저에게 다른 파일을 넣으라고 알려주는 것까지 지시하는 것.
* 예외처리 회피 : 로그를 남기고 예외를 다시 던지는 것.
* 콜백 오브젝트는 SQLException에 대한 예외를 회피하고 템플릿 레벨에서 처리하도록 던져준다.
* 책임을 분명하게 지게 하거나, 자신을 사용하는 쪽에서 예외를 다루는 게 최선이라는 확신이 있어야 한다.
* 예외 전환 : 예외를 전환해서 던진다.
* 첫번째는 의미를 분명하게 해줄 수 있는 예외로 바꿔준다. 유저 생성시에 발생한 건지, 인서트에 발생한건지 서비스 계층에서 모르기때문에 DAO에서 정보를 해석해서 바꿔 던져준다. 보통 전환하는 예외에 원래 발생한 예외를 담아서 중첩 예외로 만드는 것이 좋다. getCause 메소드로 원래 예외가 뭔지 확인할 수 있다. catch(SQLException e) { throw DuplicateUserIdException(e); 혹은 throw DuplicationUserIdException.initCause(e);} 로 던져준다.
* 두 번째 전환 방법은 예외를 처리하기 쉽게 포장하는(wrap) 것이다. 중첩 예외를 이용해 새로운 예외에 담아서 보내는 것은 맞지만, 의미를 명확하게 하려고 함이 아니다. 주로 예외처리를 강제하는 체크 예외를 언체크 예외인 런타임 예외로 바꾸는 경우에 사용한다. 대표적으로 EJBException 이 있다. EJB 에서 발생하는 에러들은 대부분 복구 가능한 예외가 아니다. 따라서 요런 건 런타임 예외로 포장해서 던지는 게 낫다. 예를 들면 catch(NamingException ne) { throws new EJBException(ne) };
* EJBException 은 런타임 예외이므로 EJB는 시스템 익셉션으로 인식하고 트랜잭션을 자동으로 롤백해준다. 잡아도 복구할 방법이 없는 API이니까 일일이 예외를 잡거나 다시 던지는 걸 할 필요가 없다.
* 체크 예외를 throws를 사용해서 넘기는 건 무의미하다. DAO에서 발생한게 컨트롤러까지 전달된다고 해서 무슨 소용이 있는가? 어차피 복구가 불가능하면 가능한 한 빨리 런타임 예외로 던져서 다른 계층의 메소드에 throws를 안 쓰는 게 더 좋다.
* 로직 상에 예외조건이 발생하면, 체크 예외를 사용하는 게 적절하다. 비지니스의 예외는 적절한 복구 작업이 필요하기 때문이다.

예외 처리 전략 - 런타임 예외의 보편화 (낙관적인 예외처리)
* 체크 예외는 일반적인 예외, 언체크 예외는 프로그램 오류가 있을 때 사용한다. 처음 자바는 통제 불가능한 시스템 예외일지라도 앱이 멈추지 않도록 복구해줬어야 했다. 그런데 서버로 넘어오면서 요청에 문제가 있으면 해당 작업만 중단시키면 그만이다. 서버를 일시 중지하고 예외 상황을 복구할 수 없다.따라서 예외상황을 미리 파악하고 예외가 발생하지 않도록 해야 한다.
* 예외 나면 통신을 끊으니까 체크 예외의 활용도와 가치는 점점 떨어지고 있다. 따라서 대응이 불가능한 체크 예외면 런타임 예외로 전환해서 던지는 게 맞다. 복구 불가능하거나 복구할 생각이 없으니까 아예 낮은 수준에서 언체크 예외로 던지도록 만드는 것이다.
* 예를 들어 add를 했을 때, 원인이 ID 중복이면 DuplicatedUserIdException으로 던져줘서 add()를 쓰는 클라이언트에서 대응을 해주면 된다. 그런데 그냥 SQLException은 대부분 복구 불가능하므로 계속 throw를 타고 다닐 뿐이다. 따라서 런타임 예외로 포장해서 던져버려 그 밖의 메소드가 신경쓰지 않게 하는 편이 낫다.
* Public class DuplicatedUserIdException extends RuntimeException { public DuplcationUserIdException(Throwable cause) { super(cause); } } RuntimeException을 상속해서 언체크 예외가 되었다. 따라서 메소드 선언의 throws에 포함시키지 않아도 된다. 반면에 add()를 쓰는 클라 쪽에서 아이디 중복 예외를 처리하고 싶으면 활용하도록 알려주기 위해 DuplicatedUserIdException을 add 메소드의 throws 선언에 포함시킨다.
* 이제 이 add 메소드는 처리하지 못 할 SQLException을 처리하기 위해 불필요한 throws 를 선언하지 않아도 되고, 쓸 생각이 있다면 DuplicatedUserIdException을 catch해서 복구를 해줄 수도 있다.
* 이렇게 런타임을 일반화시켜 사용하면 코딩의 자유도를 높일 수 있지만, 예외상황을 충분히 고려하지 않았을 경우 문제가 많이 커질 수 있으니 확실히 인지하고 쓰자.
* 런타임 예외 중심의 전략 -> 못 잡으면 어차피 어쩔 수 없는 에러임. 근데 잡고 싶은 건 따로 잡을 수 있음. / 체크 예외 -> 처리할 수 없는 게 대부분이더라도 일단 잡고 보자.

애플리케이션 예외
* 시스템 또는 외부 상황이 원인이 아니라 애플리케이션 자체의 로직에 의해 의도적으로 발생시키고 반드시 catch해서 무엇인가 조치를 취하도록 요구하는 예외
* 잔고의 금액 보다 더 많이 출금을 시도하면 중단시키고, 경고를 보내야 한다.
* 첫 번째 해결 방법은, 정상적인 출금처리를 했을 경우와 아닌 경우에 다른 종류의 리턴 값을 보내준다. 따라서 클라 쪽에서는 반드시 리턴 값을 확인한다. 그 값의 확인에 따라 로직이 분기된다.
* 요렇게 하면, 리턴 값을 명확하게 코드화하고 잘 관리하지 않으면 혼란이 생긴다. 정상적인 처리가 안 됐을 때 전달하는 값의 표준이 없으므로 의사소통에 따라 문제가 생길 수 있으며, 결과 값을 확인해야 하는 조건문 때문에 지저분해진다.
* 두 번째는 정상적인 흐름은 그대로 두고, 잔고 부족과 같은 예외 상황은 비지니스적 의미를 띤 예외로 던진다. InsuffiecientBalanceException을 던진다. 예외상황을 처리하는 catch 블록을 메소드 호출 직후에 두지 않고 추후에 모아서 정리하면 이해하기도 편하다.
* 이 때 사용하는 예외는 의도적으로 체크 예외로 만든다. 그래서 개발자가 잊지 않고 해당 로직에 대해 구현하도록 강재해야 한다. throws Exception 처럼 무책임하게 쓰면 놓칠수도 있지만 말이다.
* 또한 InsuffiecientBalanceException에 예외 상황에 대한 정보도 담아서 내리는 것도 좋다. 얼마나 인출 가능할지에 대해 포함 시켜 내리는것이다. 예외의 대응을 위한 정보까지 예외에서 알려주는 것이다.

비표준 SQL
* 특정 DB에 맞춰진 SQL 혹은 최적화된 쿼리들을 사용하다보면 DB 종속적이게 된다. 따라서 외부에 독립시키거나 DAO를 DB별로 파야 한다.
* 요 방법은 7장에서 보자.
호환성 없는 SQLException의 DB 에러 정보
* DB를 사용하다 생기는 에러는 다양하다. SQL의 문법 오류, 커넥션 에러, 필드나 테이블 존재 안하거나 데드락 혹은 락. 심지어 디비마다 원인이 제각각이다.
* 그래서 SQLException은 예외가 발생했을 때의 디비 상태를 담은 SQL 상태정보를 부가적으로 제공한다. 이 상태정보로 DB 별로 달라지는 에러 코드를 대신할 수 있다.
* 스펙에 정의된 SQL 상태 코드를 통해 DB에 독립적인 에러 정보를 얻을 수 있다. 그러나 JDBC 드라이버에서 정확하게 안 넣어주므로 호환성 없는 에러 코드와 표준을 잘 따르지 않는 SQLException만으로 DB 독립적인 코드를 못 짠다.
DB 에러 코드 매핑을 통한 전환
* 따라서, 요런 것들을 해결할 수 없으니 각 DB별 에러 코드를 통해 해결한다.
* SQLErrorCodes 의 빈을 만들고 각 예외 클래스에 디비 마다 발생시키는 에러 코드를 매핑한다. <bean id=“Oracle” class=“org.springframewokr.jdbc.suppor.SQLErrorCodes”> <property name=“duplicateKeyCodes” value=“1”></bean> 같은 식.
* 이렇게 되면 JdbcTemplate에서 런타임 예외인 SQLException 을 중복 키로 인해 발생하는 DuplicatieKeyException으로 매핑해서 던져 준다.

DAO 인터페이스와 구현의 분리
* DAO를 굳이 따로 만드는 이유는, 데이터 액세스 로직을 다른 코드와 분리해놓기 위함이다. 또한 DAO는 전략 패턴을 적용해 구현 방법을 변경해서 사용할 수 있다. DAO를 사용하는 쪽에서 어떤 데이터 액세스 기능을 사용하는지 몰라도 된다. 따라서 구체적인 클래스 정보와 구현 방법을 감추고 DI를 통해 제공되는 것이 좋다.
* 그런데 메소드 선언에 나타나는 예외 정보가 문제될 수 있다. 인터페이스에 Throws SQLException; 을 해야 하는데, JDBC가 아닌 다른 데이터 액세스 기능으로 사용하면 JdoException;이라든가 다른 걸 써야 한다. 가장 단순하게는 throws Exception; 하면 되지만 다른 방법이 좋을 것 같다. 다른 커넥션들은 런타임 예외를 던진다. 따라서 JDBC API도 런타임 예외로 포장해서 던져줄 수 있다. 그러므로 throws SQLException; 이 필요 없어졌다. 이렇게 DAO에서 쓰는 데이터 액세스 패키지와 독립적인 인터페이스 선언이 되었다.
* 그러나, 대부분의 데이터 액세스 예외는 복구 불가능하지만 중복 키 에러처럼 비지니스 로직에서 복구해줄 수 있는 예외도 있다. 문제는 데이터 액세스에 따라 다른 예외가 던져진 다는 것이다. 따라서 DAO가 의존적이 된다. 이걸 어떻게 해결하냐
스프링의 데이터 엑세스  지원 기술을 이용해 DAO를 만들면 사용 기술에 독립적인 일관성 예외를 던질 수 있다.

기술에 독립적인 UserDao 만들기
* Interfcae UserDao 만든 다음에 필요한 프로퍼티를 넣자. 대신, setDataSource 같은 메소드는 UserDao 의 구현 방법에 따라 변경될 수 있으며, 클라이언트 쪽에서 알 필요가 없으니 포함시키지 않는다.

# 5장. 서비스 추상화
DAO에 트랜잭션을 적용하면서 어떻게 비슷한 종류의 기술을 추상화하고 일관되게 사용할 수 있는지 살펴보자.
유저의 레벨을 만들어보자.
DB에 archer 타입으로 BASIC, SILVER, GOLD 를 넣는 거 보다는 레벨을 코드화해서 숫자로 넣는 게 좋다.
그러면 자바에서도 숫자로 넣는 게 좋을까? 안 좋다. 의미 없는 숫자를 프로퍼티에 사용하면 타입이 안전하지 않다. 따라서 Enum을 쓰자.

* 직접 쿼리 되는 거에 대해 테스트를 만들어야 디플로이 전에 알아차릴 수 있다. 반드시 테스트 만들자.
* 사용자 수정 시, 테스트에서 user1이라는 텍스쳐 픽스쳐는 마음대로 써도 된다. 어차피 @Test 실행될 때마다 UserDaoTest 오브젝트는 새로 만들어지고, setUp도 새로 불려지기 때문이다.
* 이제 사용자 레벨 업그레이드 로직을 담는 UserService 를 만들고, UserServiceTest 에서 해당 로직을 테스트 하자.
* UserServiceTest 에서 관심있는 건,  유저들을 활동에 따라 레벨 업그레이드를 잘 시켜주냐는 것이다.
UserService.add()
* 맨 처음에 User의 level 필드를 Level.Basic 로 어디서 설정하는 게 좋을까? UserDaoJdbc 의 add()는 아닌 것 같다. 주어진 User 오브젝트를 넣는 데에만 집중해야지 비지니스 로직을 섞으면 안 된다.
* 그렇다면 User Class에서 초기 값을 박는 것은? 그러나 처음 가입할 때를 제외하면 무의미해서 의미가 없는 것 같다.
* 사용자 관리에 대한 비지니스 로직을 담는 UserService 에서 관리하자. UserService add() 를 만드는 것이다.
upgradeLevels()의 문제점
* If, elseif, else 가 보기 불편하다. 레벨을 체크하고 & 조건을 파악한 다음에 업그레이드 시킨다. 업그레이드하는 로직까지 따로 들어가있다. 이걸 분리하자. 조건 체크와, 업그레이드를
* 간단하게, if(canUpgradeLevel(user)) { upgradeLevel(user) } 로 간단하게 표현이 가능하다.
* 마찬가지로 upgradeLevel도 간소화시켜보자. Enum Level 에서 NextLevel 을 리턴하도록 하면, 불필요한 if 문을 타지 않아도 된다. 즉, Level 의 관심사를 그 쪽으로 책임을 돌리자.
* 그리고 upgradeLevel 도 UserService에 있어야 하는가? 레벨의 클라이언트는 User이므로 User에 메소드를 붙이자. Public void upgradeLevel() { Level nextLevel = this.level.nextLevel(); this.level = nextLevel; }
* 이렇게 되면 UserService의 upgradeLevel은 불필요한 if 문이 제외되고 훨씬 간결해진다. user.upgradeLevel(); userDao.update(user);
* 내가 오브젝트에게 데이터를 요청하고 내가 가공하는 것이 아니라, 내가 오브젝트에게 작업을 요청한다! 알아서 가공해오라고.
* UserService 는 USer에게 레벨 업그레이드 해라. 하고 User는 Level에게 다음 레벨이 뭔지 알려달라. 가 바람직하다는 의미이다.
* 이제 트랜잭션이 되는지 보자. UserService 에서 저장시에 에러를 뱉도록 만들어야 하는데, UserService 를 상속한 클래스를 UserServiceTest 내부에 만든다. (어차피 여기서밖에 안쓰니까) 그리고 upgradeLevel()과 user id받고 뭐 에러 발생시켜라.
* 보니까 에러가 나도 rollback이 안된다. JDbc의 autoCommit을 꺼줘야 한다. 그리고 트랜잭션의 경계를 설정하자. 어디서부터 어디까지 할꺼냐. SQL은 매번 커밋이 되고 있다.
* 결국 JdbcTemplate 에서도 매번 커넥션 부르고 닫는다. 결국 JdbcTemplate의 메소드는 각 메소드마다 하나씩의 독립적인 트랜잭션으로 실행된다. 데이터 액세스 코드를 DAO로 만들어서 분리하면 이처럼 메소드 하나마다 하나의 트랜잭션이 만들어진다.
* 트랜잭션을 하려면, 그 작업이 진행되는 동안 DB 커넥션도 하나만 사용해야 한다. 이걸 어떻게 해결해야 할까.
* DAO 내부에 upgradeLevels 메소드를 옮긴다. 하나의 커넥션으로 여러 사용자들을 불러서 한번에 업데이트 친다. -> 비지니스 로직과 데이터 로직을 묶어버리는 최악의 결과.
* 결국 UserService의 upgradeLevels 에서 트랜잭션의 시작과 끝을 관리하므로 거기서 해줘야 한다. 그러면 DB 커넥션도 이 메소드 안에서 만들고 종료시켜야 한다.
* 그러면 그 커넥션을 DAO의 메소드를 통해서 업데이트 시키는 것이다. 이 때, 기존 템플릿 처럼 새로 커넥션 만들면 안된다. 별개의 트랜잭션이니까!! 따라서 커넥션을 DAO 메소드에 전달해줘야 한다. 그렇게 되면 UserDao 의 인터페이스는 add(Connection c, User user); 로 변경되어야 한다.
* 그런데 upgradeLevels 의 upgradeLevel에서 update를 호출한다. 따라서 user 에서 DAO update 구문을 호출하니까 거기까지 connection을 파라미터로 전달해줘야 한다. UserService::upgradeLevels -> UserService::upgradeLevel -> UserDao::update 이렇게 커넥션을 건너 건너 줘야 한다.

UserService 트랜잭션 경계설정의 문제점
* 해결은 됐는데, 문제다.
* 커넥션을 비롯한 깔끔한 처리가 됐던 JdbcTemplate를 못 쓴다. 결국 초기 방식대로 넘어가서 트라이 캐치 문으로 점철되어 있다.
* 또 DAO의 메소드와 비지니스 로직을 담은 UserService 에 Connection들을 막 넘기고 있다. upgradeLevels 에서 어딘가 DAO를 필요로 한다면 계속 넘겨줘야 한다.
* UserDao 인터페이스에 Connection을 넘기면서, 데이터 액세스 기술에 독립적이지 않다. JPA나 하이버네이트로 변경할며ㅕㄴ Connection 대신 Session 오브젝트를 넘겨야 한다. 지금까지 DI가 물거품이 된다. 또, 이제 호출하는 모든 곳에서 커넥션을 넘겨줘야 한다.
* 트랜잭션 경계설정은 해야 하는데, 파라미터로 Connection을 전달하다 DAO를 호출할 때 사용하는 건 피하고 싶다. 이를 위해 트랜잭션 동기화 방식을 사용하자.
* 트랜잭션 동기화 : UserService에서 트랜잭션을 시작하기 위해 만든 Connection 오브젝트를 특별한 저장소에 저장해놓고 호출되는 DAO 메소드에서 저장된 Connection을 가져다 쓰는 방식
* UserService에서 트랜잭션 시작하고 트랜잭션 동기화 저장소에 커넥션 오브젝트를 저장한 다음 DAO에서 커넥션을 가져다 쓰다가 끝나면 제거해준다.
* UserService에서 커넥션을 써야하니까 DataSource를 DI 해주고, TransactionSynchronizationManager.initSynchronization(); Connection c = DataSourceUtils.getConnection(dataSource); // DB 커넥션 생성과 동기화(저장소에 바인딩해줌)를 함꼐 해주는 유틸리티 메소드.
* c.setAutoCommit(false); 하고 try{} catch {c.rollback();} finally { DataSourceUtils.releaseConnection(c, dataSource); TransactionSynchronizationManager.unbindResource(this.dataSource); TransactionSynchronizationManager.clearSynchronization();

JDbcTemplate 과 트랜잭션 동기화
* JDbcTemplate은 트랜잭션 동기화 저장소에 등록된 DB 커넥션이나 트랜잭션이 없으면 직접 커넥션을 만들고 트랜잭션을 시작하고, 동기화가 시작되어있다면 그 커낵션을 가져와서 사용한다. JDbc 코드의 try catch finally 작업 흐름 지원, SQLException 예외 변환과 함꼐 제공하는 유용한 기능이다.
* 이제 다 해결된 것 같다. 커넥션도 안 넘겨도 되고, UserDao 코드 안 바꿔도 되고, 비지니스 로직 레벨에서 트랜잭션을 적용했고… 그런데 이제 부터 본격적인 고민의 시작이다.

트랜잭션 서비스 추상화
* 문제가 생겼다. UserService 모듈을 사간 고객이 하나의 DB 트랜잭션이 아닌 여러 DB에 연결하고 싶다고 한다. 그런데 로컬 트랜잭션으로 불가능하다. 이는 하나의 DB에 종속되기 때문이다. 따라서 별도의 트랜잭션 관리자를 통해 트랜잭션을 관리하는 글로벌 트랜잭션을 써야 한다.
* 자바는 JDBC 외에 이런 글로벌 트랜잭션을 지원하는 JTA(Java Transaction API)를 제공한다. 여러 개의 DB 또는 메시징 서버에 대한 트랜잭션을 관리한다.
* 애플리케이션에서는 DB는 JDBC, 메시징은 JMS같은 API를 쓰는데 트랜잭션은 JDBC가 아닌 JTA를 이용해서 관리하는 것이다. 트랜잭션 매니저는 DB와 메시지 서버를 제어하고 관리하는 리소스 매니저와 연결된다. 이를 통해 JTA가 모든 트랜잭션을 통합해서 관리한다.
* InitialContext cox = new InitialContext(); UserTransaction tx = (UserTransaction)ctx.lookup(USER_TX_JNDI_NAME); 을 통해 처리해주는데,
* 이렇게 되면 글로벌 트랜잭션을 써야되면 JTA를 쓰고, 로컬 트랜잭션만 필요한 고객들은 JDBC 트랜잭션을 쓰는 코드가 되었다.
* 또 다른 DB 쓰면 세션으로 뭐 하면서 변경해야 한다.

트랜잭션 API의 의존관계 문제와 해결책
* UserService 에서 트랜잭션 경계설정을 해야하기 때문에 특정 데이터 엑세스 기술에 종속되고 말았다. UserService -> UserDao, UserDaoJdbc 가 되어 버렸다.
* DI가 허사가 되었다. UserService가 DB에 의존적이 되지 않으려면 트랜잭션 경계설정을 제거해야 하는데, 그건 불가능하다. 하지만 특정 기술에 의존적인 Connection, UserTransaction 등에 종속되지 않게 하는 방법은 있다.
* DB에서 제공하는 클라와 API는 서로 호환이 안되지만 SQL을 사용하는 공통점이 있다. 이 공통점을 뽑아내 추상화한 것이 JDBC이다. 그 추상화 기술 때문에 DB 종류와 관계 없이 개발할 수 있었다.
* 마찬가지로 트랜잭션도 추상화를 도입해볼 수 있을 것 같다. 트랜잭션 경계설정 방법의 공통점이 있을 것같다. 애플리케이션 코드에서는 추상 계층이 제공하는 API를 이용해 트랜잭션을 쓰면 기술에 종속되지 않을 수 있다.
* 그렇게 되면, 결국 스프링이 제공하는 트랜잭션 경계설정을 위한 추상 인터페이스인 PlatformTransactionManager 를 이용해서 어떤 TransactionManager쓸지 DI 받아서 설정시켜준다.
* <bean id=“userService”> <property name=“userDao”/> <property name=“transactionManager”> </bean> <bean id=“transactionManager”> <property name=“dataSource”> </bean>

서비스 추상화와 단일 책임 원칙
* UserDao와 UserService는 각각 담당하는 코드가 관심에 따라 분리되고, 독자적으로 확장된 것이다. 그런데 트랜잭션의 추상화는 이와 좀 다르다. 비지니스 로직과 로우레벨의 트랜잭션이라는 기술, 아예 다른 특성을 갖는 코드를 분리한 것이다.
* UserDao 와 UserService는 인터페이스와 DI를 통해 연결됨으로써 결합도가 낮아졌다. 데이터 엑세스 로직이 바뀌거나, 데이터 엑세스 기술이 바뀌어도 영향을 주지 않으며, 독립적으로 확장이 가능하다.
* 또 UserDao는 DB연결 생성하는 방법에 대해 독립적이다. DB가 어떻게 생기든 상관 없고,트랜잭션 기술도 구체적인 코드와 독립적이다. DI가 관심과 책임 성격이 다른 코드들을 깔끔하게 분리해버린다.
* DI를 쓰지 않았을 때는 UserService가 바뀌는 이유가 두가지 였다. 트랜잭션과 유저 레벨 관리 방법. 트랜잭션 서비스의 추상화를 도입하고 책임이 분리되어 버렸다. 따라서 수정 대상이 명확해진다.

메일 서비스 추상화
* UserServcie -> JavaMail -> 테스트용 메일 서버 로 테스트하자. 메일 실제로 보내야 할 이유는 없다. DB 업데이트 되는 것만 보고 완료.
* 그런데 굳이 JavaMail도 테스트할 필요 없다. JavaMail이 동작하면 외부의 메일 서버와 연동하고 전송하는 부하가 큰 작업이 일어나기 때문에, UserService -> 테스트용 JavaMail로 변경하자.
* 실제 메일을 전송하는 JavaMail 대신에 같은 인터페이스를 이용하는 오브젝트를 만들자. 그런데 내부적으로 JavaMail에서는 Session 오브젝트를 만들어야 하며, 상속이 불가능한 클래스에 private 생성자여서 스태틱 팩토리로 만들어야 하다. 즉, 인터페이스로 확장 혹은 지원이 불가능하게 만들어놓은 API 클래스이다. 이걸 트랜잭션을 적용하면서 살펴봤던 서비스 추상화를 이용하자. Public interface MailSender { void send(SimpleMailMessage simpleMessage) }
* 요 인터페이스를 구현한 JavaMailSenderImpl 클래스를 이용해서 JavaMailSenderImpl mailSender = new JavaMailSenderImpl(); .setHost(“mail.server.com”); 이렇게 되면 일단 지저분한 try catch 문은 사라진다. DI를 적용해서 이제 MailSender 인터페이스만 남기고 구체적인 메일 전송 구현의 클래스 정보는 모두 제거한다.
* 그러면 UserService의 sendUpgradeEmail(User user) { SimpleMailMessage mailMessage = new SimpleMailMessage(); this.mailSender.send(mailMessage); }
* 스프링 빈으로 등록된 MailSender 의 구현체들은 싱글톤으로 사용 가능하다. 이제 테스트 파일의 bean configuration에서는 bean의 클래스를 MailSender 인터페이스를 구현한 DummayMailSender 라는 아무것도 하지 않는 클래스를 만들어서 달아주면 된다.
* 테스트 파일에서는 @Autowired MailSender mailSender; testUserService.setMailSender(mailSender); 해주자.
* 즉, MailSender 를 통해 DummyMailService 와 JavaMailServiceImpl을 추상화 시킴으로써 인터페이스를 연결하는 것이다.

테스트 대역의 종류와 특징
* 테스트 대상인 오브젝트의 의존 오브젝트가 되는 것들, UserDao의 DataSource라던가 UserService의 MailSender등을 구현한 테스트 용 오브젝트들을 통틀어서 테스트 대역이라고 부른다.
    * 테스트 스텁 : 테스트 대상 오브젝트의 의존객체로서 테스트 동안 코드가 정상적으로 수행할 수 있도록 도움. 테스트 코드 내부에서 만들어진다. 의존 오브젝트를 DI 대신 사용하는 구현체. DummyMailSender가 그 예시.
    * 그런데 어떤 스텁은 특정 값을 돌려줘야 하기도 하고, 예외를 발생시키기도 해야 한다. 의존 오브젝트로서 간접적인 도움을 주는 것만이 아니라 매우 적극적으로 참여할수도 있다.
    * 테스트 대상 오브젝트의 메소드가 값을 돌려주는 것 외에 간접적으로 의존 오브젝트에 넘기는 값과 그 행위 자체에 대해서 검증하고 싶으면 어떻게 해야할까? 이 경우는 단순히 가장 밖에 있는 값을 대조하는 assertThat으로 불가능하다.
    * 테스트 대상의 간접적인 출력 결과를 검증하고, 테스트 오브젝트와 의존 오브젝트 사이에 일어나는 일을 검증하도록 설계된 목 오브젝트를 사용해야 한다. 커뮤니케이션 내용을 저장해뒀다가 테스트 결과를 검증할 때 사용한다.
    * 테스트 오브젝트는 테스트에서만 값을 넘겨주는 게 아니다. 의존 오브젝트와도 값을 주고받는다. 테스트 대상이 의존 오브젝트로부터 입력받는 값이 필요하다. 따라서 스텁 오브젝트가 메소드 호출 시 특정 값을 리턴받도록 만들어야 한다. 혹은 반대로 출력하는 값을.
    * 이 경우들을 테스트에서는 정확하게 알 수가 없으니, 해당 정보들을 보존해두는 기능을 가진 목 오브젝트를 만들어서 사용하는 것이 좋다. 그걸로 테스트 검증하자.
        * 직접 테스트 구간 : 테스트 -입력-> 테스트 대상 오브젝트 -출력-> 테스트
        * 간접 테스트 구간 : 테스트 대상 오브젝트 -출력-> 의존/협력 오브젝트 -출력-> 테스트 대상 오브젝트

목 오브젝트를 이용한 테스트
UserServiceTest에서 upgradeAllOrNothing 테스트에서 DummyMailSender는 충분하다. 어차피 중간에 에러를 발생하고 롤백되어야 하기 때문에 메일이 전송됐는지 여부는 관심의 대상이 아니기 때문이다.
반면 upgradeLevels 테스트의 경우는 메일을 보내야 하기 때문에 메일 전송 자체를 검증해야 한다.
* UserService의 코드가 정상적으로 수행되며 출력 값을 보관하는 기능을 추가한 클래스를 만들자. UserServiceTest에서만 사용할 것이므로 스태틱 멤버 클래스로 정의하자.
* Static class MockMailSender implements MailSender { private List<String> requests = new ArrayList<String>(); public List<String> getRequests(){}; void send(SimpleMailMessage mailMessage) { requests.add(mailMessage.getTo()[0]); }
* 전송할 때마다 클래스 멤버 변수에 저장해둔다. 그리고 테스트에서 아래와 같이 작성한다.
* userService.setMailSender(mockMailSender); userService.upgradeLevels(); List<String> request = mockMailSender.getRequests(); assertThat(request.size(), is(2));
* 이런 식으로 테스트 오브젝트와 의존 오브젝트간의 인터랙션까지 테스트할 수 있다.


# 6장. AOP
트랜잭션의 경계설정 기능을 AOP를 이용해 바꿔보자.

트랜잭션 경계설정과 비지니스 로직은 서로 독립적인데 앞뒤로 존재하므로 분리 가능하다.
직접적으로 정보도 안 주고 받는데, 뺄 수 없을까?

DI 적용을 이용한 트랜잭션 분리
* 트랜잭션을 빼버리면, UserService를 사용하는 직접적으로 클라이언트들은 다 트랜잭션이 없이 사용해야 한다. 그러면, 간접적으로 사용하자. DI의 기본 아이디어는 실제 사용할 오브젝트를 감춘 채 인터페이스로 간접 접근을 할 수 있는 장점이 있으므로 얼마든지 구현 클래스를 변경할 수 있다.
* UserService 인터페이스를 만들고, 비지니스 로직과 트랜잭션 경계설정을 분리해서 담아보자. UserService를 구현한 또 다른 구현 클래스는 트랜잭션의 경계설정만 담당하고 비지니스 로직은 UserServiceImpl에 위임한다.
* Public interface UserService { void add(User user); void upgradeLevels(); } public class UserServiceImpl implements UserService { public void upgradeLevels(); } public class UsersServiceTx implements UserService { UserService userService; PlatformTransactionManager transactionMananger; public upgradeLevels() { TransactionStatus status ….. try{ userService.upgradeLevels(); } catch {} }
* 이렇게 되면, Client -> UserServiceTx -> UserServiceImpl 과 같은 의존관계가 만들어진다.
* 이제 테스트를 해보자. 원래 테스트에서 UserService를 @Autowired 로 가져왔었는데, 둘 다 UserService 인터페이슨데 어떻게 하냐?? 그럴 때는 필드 이름과 id 값이 같은 걸 가져오니까 UserServiceTx 가 UserService id 로 등록되어 있어야 하는 점을 주목하자.
* UserServiceImpl 클래스를 가져와서 앞 테스트 단에서 만들었던 MockMailSender 도 DI 해줘야 한다. 따라서 @Autowired UserServiceImpl userServiceImpl; 로 받아온다음에 mockMailSender 를 DI해주자. 그런데 문제가 있다. 원래 만들었던 upgradeOrNothing 클래스에서 테스트하고자 했던 부분은 사용자 관리 로직이 아니다. 트랜잭션 기술을 테스트하고 싶었기 때문에 구조를 변경해야 한다.
* 이제 TestUserService 가 트랜잭션 기능이 빠진 UserServiceImpl을 상속하고 있다. 따라서 UserServiceTx를 수동 DI 시킨 후에 트랜잭션 테스트를 수행해야 한다.
* UserServiceTx txUserService = new UserServiceTx(); txUserService.setTransactionManager(transactionManager); txUserService.setUserService(testUserService); 다시, UserServiceTx <- testUserService <- UserServiceImpl 이 구조인 것이다.
* 이렇게 트랜잭션을 분리하니 장점이 생겼다.
    * 비지니스 로직을 담당할 때 트랜잭션 같은 기술 쪽을 신경쓰지 않아도 된다. 로우 레벨의 API는 물론 스프링의 트랜잭션 추상화 API도 필요 없다. 비지니스 로직만 신경쓰면 된다.
    * 비지니스 로직에 대한 테스트를 쉽게 만들 수 있다. -> 다음에서 살펴보자.

고립된 단위 테스트
테스트는 무조건 작아야 좋다. 원인이 찾기 쉬우며 의도와 내용이 분명해지며 만들기 쉬워진다.
* UserServiceTest의 테스트 대상인 UserService는 사용자 정보를 관리하는 비지니스 로직의 구현 코드다.
* 하지만 UserService는 UserDao, TransactionManager, MailSender 라는 세가지 의존관계를 가지고 있다. 더 큰 문제는 JDBC 부터 DB 네트워크 통신 및 DB 서버, 트랜잭션 매니져….. 그 뒤에 숨겨진 훨씬 더 많은 오브젝트와 환경, 서비스, 서버, 네트워크까지 테스트한다.
* 그래서 UserService라는 테스트 대상이 테스트 단위가 아닌 모든 것들이 합쳐진 테스트 대상이 된다. 잘못된 비지니스 로직보다 다른 오류 때문에 시간을 훨씬 낭비할 가능성이 크다. 배보다 배꼽이 훨씬 크다.
* 그래서 테스트의 대상이 환경이나 외부서버, 다른 클래스의 코드에 영향을 받지 않도록 고립시킬 필요가 있다. 테스트를 위한 대역을 사용하는 것이다.
* MailSender는 이미 DummyMailSender 라는 테스트 스텁을 사용했다. 또 테스트 대역이 테스트 검증에도 참여하도록 MockMailSender라는 목 오브젝트도 사용해봤다.
* 같은 방법을 UserDao 에도 적용할 수 있다. 하지만 PlatformTransactionManager 에는 이런 수고를 할 필요가 없다. 트랜잭션 코드를 독립시켰기 때문에 UserServiceImpl은 PlatformTransactionManager에 더이상 의존하지 않기 때문이다.
* 즉, 이렇게 고립된 테스트가 가능하도록 USerSErvice 를 재구성해보면, UserServiceImpl은 MockUserDao와 MockMailSender에만 의존하고 UserServiceTx는 그 외의 JDbc, TransactionManager, DataSource, JavaMailSender, DB Server 등을 의존한다.
* UserDao는 단지 테스트 대상의 코드가 정상적으로 수행되도록 만드는 스텁이 아니라 목 오브젝트로 만들어야 한다. 그 이유는 고립된 환경에서 동작하는 upgradeLevels의 테스트 결과를 검증할 방법이 없다. 잘 돌아갔는지 돌아간 걸로만 보면 잘 몰라!
* 그래서 DAO를 통해 반영된 DB를 확인해왔는데, 의존 오브젝트나 외부 서비스에 의존하지 않는 고립된 테스트 방식으로 만든 UserServiceImpl은 DB랑 연결도 안 되어 있으니 작업 결과를 검증할 방법이 없다. 그래서 이럴 땐 테스트 대상인 UserServiceImpl이 UserDao에게 어떤 요청을 했는지 확인하는 작업이 필요하다. DB에는 결과가 반영되지 않았지만 update()를 호출했다면 결국 DB에 반영될 것이라고 결론을 내릴 수 있기 때문이다.

고립된 단위 테스트 활용
* 기존 테스트 구성은 다음과 같다.
    * 테스트용 정보를 DB에 넣는다. UserDao 는 DB에서 정보를 가져오기 때문에 최후의 의존 대상인 DB에 직접 넣어줘야 한다.
    * 메일 발송 여부를 확인하기 위해 MailSender 목 오브젝를 DI 해준다.
    * 실제 테스트 대상인 userService 메소드를 실행한다.
    * 결과가 DB에 반영되었는지 DB에서 데이터를 가져와 확인한다
    * 목 오브젝트를 통해 UserService에 메일 발송이 있었는지 확인한다.
* 이제 실제 UserDao와 DB까지 직접 의존하고 있는 첫 번쨰와 네번째 테스트 방식도 목 오브젝트를 만들어서 적용해보겠다. upgradeLevels()가 실행되는 중에 UserDao 와 어떤 정보를 주고 받을지 입출력 내용을 확인할 필요가 있다.
    * userDao를 사용하는 경우는 두가지 인데, 우선 userDao.getAll(); -> 업그레이드 할 후보 사용자 목록을 가져온다.
    * userDao.update(user); -> 수정된 사용자 정보를 DB에 반영한다.
* getAll을 지원하기 위해 유저 목록을 제공해줘야 한다. 그런데 update의 경우는 리턴 값이 없다. 따라서 getAll()에 대해서는 스텁으로서, update()에 대해서는 목 오브젝트로서 동작하는 UserDao 타입의 테스트 대역이 필요하다.
* 마찬가지로 테스트에서만 필요하니까 Static Class MockUserDao implements UserDao { private List<User> getAll() { return this.users } // 스텁 기능 제공 public void update(User user) { updated.add(user); } // 목 오브젝트 기능 제공
* 조금 귀찮지만, 그 외의 안 쓰는 메소드들도 인터페이스가 있으니 구현은 해줘야 한다. 따라서 에러를 던져두던지 return null로 하자.
* 이제 다시 테스트로 돌아가보자.
* 기존 테스트는 @autowired된 UserService 빈을 가져왔는데, 이제는 많은 의존 오브젝트와 서비스, 외부 환경에 의존하지 않는다. 완전히 고립되어서 테스트만을 위해 독립적으로 동작하도록 했으니 컨테이너에서 빈을 가져올 필요가 없다. 따라서 UserServiceImpl을 생성하고 MockUserDao를 수동 DI 해주면 된다. 그리고 검증하는 것은 굉장히 간단해진다. DB에 직접 넣고 조회할 필요 없이 목 오브젝트에 해당 리스트들이 추가되었는지만 확인해주면 된다.

단위테스트와 통합테스트
* 단위 테스트 : 테스트 대상 클래스를 목 오브젝트 등의 테스트 대역을 이용해 의존 오브젝트를 이용하지 않도록 고립 시켜서 테스트하는 것
* 통합 테스트 : 두 개 이상의 계층이 다른 오브젝트가 연동하도록 만들어 테스트하는 것. 스프링의 테스트 컨텍스트 프레임워크를 사용하는 것도 포함한다.
* 단위 테스트를 미리미리 만들자. 그런데 단위테스트를 위해 스텁이나 목 오브젝트가 필수적이다. 그런데 메소드별로 다른 기능 검증해야한다면, 같은 의존 인터페이스를 구현한 여러 목 클래스를 선언해줘야 한다.
* 이 귀찮은 목 오브젝트 작성을 편리하게 해주는 목 오브젝트 프레임워크가 있다.

Mockito 프레임워크
* 목 클래스를 일일이 준비하지 않고 메소드 호출만으로 다이내믹하게 특정 인터페이스를 구현한 테스트용 목 오브젝트를 만들 수 있다.
* UserDao 인텊에시를 구현한 테스트용 오브젝트는 다음과 같이 Mockito 스태틱 메소드 한 번 호출하면 만들어진다. 스태틱 임포트를 사용해 로컬 메소트처럼 호출하자.
* UserDao mockUserDao = mock(UserDao.class); 이렇게 만들어진 목 오브젝트는 아무런 기능이 없다. 따라서 getAll 을 호출할 때 사용자 목록을 리턴하도록 스텁 기능을 추가해보자.
* When(mockUserDao.getAll()).thenReturn(this.users); 이 구문 하나면 getAll을 호출했을 때 users를 리턴해준다.
* 다음은 update가 호출됐는지 확인하기 위해서 다음과 같은 검증코드면 충분하다. verify(mockUserDao, times(2)).update(any(User.class));
* Mockito 목 오브젝트는 다음과 같은 네단계를 거쳐 사용하면 된다.
    * 인터페이스를 이용해 목 오브젝트를 만든다.
    * 목 오브젝트가 리턴할 값이 있으면 지정해준다. 예외를 던지게 만들 수도 있다.
    * 테스트 대상 오브젝트에 DI해서 목 오브젝트가 테스트 중에 사용되게 만든다.
    * 테스트 대상 오브젝트를 사용한 후에 목 오브젝트의 특정 메소드가 호출되었는지, 어떤 값을 가지고 몇 번 호출됐는지 검증한다.
* MailSender mockMailSender = mock(MailSender.class); userServiceImpl.setMailSender(mockMailsender); # 리턴 값이 없는 메소드를 가진 목 오브젝트니까 그냥 설정만 해주면 된다.
* 함수를 호출하고 어떤 파라미터로 불리는지는 테스트할 수 있지만 레벨의 변화는 체크해봐야 한다.MailSender의 경우 파라미터를 정밀하게 체크하기 위해 SimpleMailMessage를 캡쳐할 수 있는 오브젝트를 만든다.

프록시와 프록시 패턴, 데코레이터 패턴
* 트랜잭션 경계 설정을 비즈니스 로직 에서 분리해낼때, 부가기능 클래스를 만들고 그 외 모든 기능을 핵심 기능을 가진 클래스로 위임해줬다. 그리고 부가기능이 핵심기능 인터페이스의 탈을 쓴 채 핵심기능을 사용하는 형태가 되었다.
* 클라이언트가 사용하려는 실제 대상인 척 처럼 위장해서 요청을 받아주는 것을 프록시라고 한다. 그리고 프록시를 통해 요청을 위임받아 타겟, 실제 오브젝트에 연결한다. 프록시의 목적은 두가지다.
    * 클라이언트가 타겟에 접근하는 방법을 제어하거나
    * 타겟에 부가적인 기능을 부여해주기 위해서이다.
* 데코레이터 패턴은 타깃에 부가 기능을 런타임시에 동적으로 부여해주기 위해 프록시를 사용하는 패턴이다. 프록시로서 동작하는 각 데코레이터는 위임하는 대상에도 인터페이스로 접근하기 때문에 자신이 최종 타깃으로 위임하는지, 아니면 다음 단계의 데코레이터 프록시로 위임하는지 알지 못한다. 그래서 다음 위임 대상은 인터페이스로 선언하고 생성자나 수정자 메소드를 통해 위임 대상을 외부에서 런타임시에 주입받을 수 있게 만들어야 한다.
* InputStream이라는 인터페이스를 구현한 타깃 FileInputStream에 버퍼 읽기 기능을 제공해주는 BufferedInputStream 이라는 데코레이터를 적용한 예시이다.
* InputStream is = new BufferedStream(new FileInputStream(“a.txt”));
* UserServiceImpl에 UserServiceTx를 추가한 것도 데코레이터 패턴을 적용한 것이라고 볼 수 있다. 이 경우는 수정자 메소드를 이용해 UserSErviceTx에 위임할 타깃인 USerServiceImpl을 주입해줬다.
* 다이내믹한 구성 방법은 스프링의 DI를 이용하면 아주 편하다. UserServiceTx 클래스로 선언 뙨 userService 빈은 데코레이터이다. 또한 UserService 라는 인터페이스에 위임하지 UserServiceImpl 이라는 구체적인 클래스가 정해져있지 않다.
* 즉, 데코레이터 패턴은 타깃의 코드를 손대지않고 클라이언트가 호출하는 방법도 변경하지 않은 채로 새로운 기능을 추가할 때 유용하다.

프록시 패턴
* 일반적으로 사용하는 프록시 : 클라이언트와 사용 대상 사이에 대리 역할을 맡은 오브젝트
* 디자인 패턴에서 말하는 프록시 패턴 : 프록시를 사용하는 방법 중에 타깃에 대한 접근 방법을 제어하려는 목적의 경우
* 프록시 패턴의 프록시는 타깃의 기능을 확장하거나 추가하지 않는다. 대신 클라이언트가 타깃에 접근하는 방식을 변경해준다.
* 예를 들어서, 타깃 오브젝트를 생성하기 복잡하거나 당장 필요하지 않을 때, 레퍼런스가 필요하다면 프록시를 넘겨준다. 그리고 프록시의 메소드를 통해 타깃을 사용하려고 하면 그 때 가서야 타깃 오브젝트를 생성하고 요청을 위임해준다.
* 또한 원격 오브젝트를 사용할 때도 로컬에 존재하는 것처럼 사용하는 것도 된다. 요청을 받으면 네트워크를 통해 원격 오브젝트를 실행하고 결과를 받아서 돌려준다.
* 또는 특별한 상황에서 타깃에 대한 접근 권한을 제어하기 위해 쓸 수도 있다. 수정 가능한 오브젝트가 있는데 특정 레이어에서는 읽기전용으로 강제하도록 하고, 특정 메소드를 호출하면 예외를 발생시키면 된다.
* 이렇게 타깃에 기능에는 관여를 하지 않으면서 접근하는 방법을 제어한다.
* 타깃과 동일한 인터페이스를 구현하고 클라이언트와 타깃 사이에 존재하면서 기능의 부가 또는 접근 제어를 담당하는 오브젝트를 모두 프록시라고 부르자. 하지만 그떄마다 사용의 목적이 기능이 부가인지(데코레이터), 접근 제어인지(프록시) 구분해보면 어떤 패턴인지 알 수 있다.
* 클라이언트 -> 접근 제어 프록시 -> 페이징 데코레이터 -> 소스코드 출력 기능(타겟)

다이내믹 프록시
* 그런데 타깃 코드를 고치는 것보다 프록시를 귀찮게 느끼는 사람들이 많다. 일일이 클래스를 정의해야 하고, 인터페이스의 구현할 코드가 많으면 모든 메소드를 위임해야 해서 귀찮기 때문이다.
* 이걸 프록시를 편하게 만들어주도록 지원해주는 클래스들이 있다. 일일이 프록시 클래스를 정의하지 않아도 몇 가지 API를 이용해 프록시처럼 동작하는 오브젝트를 다이내믹하게 생성하는 것이다.
* 프록시는 두가지 기능으로 구성된다.
    * 타깃과 같은 메소드를 구현하고 있다가 메소드가 호출되면 타깃 오브젝트로 위임한다.
    * 지정된 요청에 대해서는 부가기능을 수행한다. (혹은 권한 관리)
* 그런데 이걸 코드상으로 작성해주면 몹시 번거롭다.
    * 인터페이스를 구현하고 위임하는 코드를 작성하기가 번거롭다.
    * 부가기능 코드가 중복될 가능성이 높다. 트랜잭션은 DB를 사용하는 대부분 로직에 적용될 수 있는데, 해당 트랜잭션 경계설정 부가기능을 다른 프록시 패턴에서도 써야할 것이다.
* 첫 번째 문제를 해결하는데 유용한 것은 JDK의 다이내믹 프록시로 해결할 수 있다.
* 다이내미 프록시는 리플렉션 기능을 이용해 프록시를 만들어준다. 리플렉션은 자바의 코드 자체를 추상화해서 접근하도록 만든 것이다. 특정 클래스이름.class를 쓰면 클래스 자체의 구성정보를 담은 Class 타입의 오브젝트를 갖고 있다. 이 오브젝트를 통해 클래스의 이름과 속성, 상속과 타입 등을 알 수 있다. 이 중 메소드에 대한 정의를 담은 Method 인터페이스를 이용해 메소드를 호출해보자. String.class 혹은 name.getClass() 를 통해 클래스 오브젝트를 가져오고 length라는 메소드에 대한 정보를 가져와보려면 다음과 같이 하자.
* Method lengthMethod = String.class.getMethod(“length”); 이 메소드의 정보를 가져올 뿐만 아니라 해당 메소드를 실행시킬수도 있다. Method 인터페이스의 invoke 메소드를 사용하면 된다. Invoke 메소드는 메소드를 실행시킬 대상 오브젝트와 파라미터 목록을 받아서 메소드로 호출한 뒤에 그 결과를 Object 타입으로 돌려준다. Public Object invoke(Object obj, Object …args)
* Int length = lenghtMethod.invoke(name): 이런식으로. 쓸 수 있다.

프록시 클래스
* 다이내믹 프록시를 이용한 프록시를 만들어보자. 구현할 인터페이스는 다음과 같이 정의한다. Interface HelloTarget implements Hello { public String sayHello(String name) {} }
* 프록시를 적용할 타깃 클래스는 다음과 같이 정의한다. Public Class HelloTarget implements Hello { }
* 데코레이터 패턴을 적용한 프록시 클래스를 만들어보자. Public class HelloUpperCase implements Hello { Hello hello; // 위임할 타깃 오브젝트. 여기서는 타깃 클래스의 오브젝트인 것을 알지만 다른 프록시가 낄 수도 있으니 인터페이스로 접근한다. }
* 이제 프록시를 적용할 수 있다. Hello proxiedHello = new HelloUpperCase(new HelloTarget()); 그런데 문제가 있다. 부가기능인 리턴 값을 대문자로 바꾸는 기능이 모든 메소드에 중복되고, 인터페이스의 모든 메소드가 위임하도록 코드를 만들어야 한다.
다이나믹 프록시 적용
* 다이내믹 프록시는 프록시 팩토리에 의해 런타임 시 다이내믹하게 만들어지는 오브젝트다. 다이내믹 프록시 오브젝트는 타깃의 인터페이스와 같은 타입으로 만들어진다. 클라이언트는 다이내믹 프록시 오브젝트를 타깃 인터페이스를 통해 사용할 수 있다. 프록시 팩토리에게 인터페이스만 제공해주면 해당 인터페이스를 구현한 클래스의 오브젝트를 자동으로 만들어주기 때문이다. 다이내믹 프록시가 인터페이스 구현 클래스의 오브젝트는 만들어주지만, 프록시로서 필요한 부가기능 제공 코드는 직접 작성해야 한다. 부가기능은 프록시 오브젝트와 독립적으로 InvocationHandler를 구현한 오브젝트에 담는다. InvocationHandler 인터페이스는 다음과 같은 메소드 한 개만 가진 간단한 인터페이스이다.
* Public Object invoke(Object proxy, Method method, Object[] args)
* Invoke 메소드는 리플렉션(클래스 오브젝트)의 Method 인터페이스를 파라미터로 받는다. 메소드를 호출할 때 전달되는 파라미터 args도 받는다. 다이내믹 프록시 오브젝트는 클라이언트의 모든 요청을 리플렉션 정보로 변환해서 InvocationHandler 구현 오브젝트의 invoke() 메소드로 넘기는 것이다. 타깃 인터페이스의 모든 메소드 요청이 하나의 메소드로 집중되기 때문에 중복되는 기능을 효과적으로 제공할 수 있다.
* Invoke 메소드는 리플렉션의 Method 인터페이스를 받는다. 다이내믹 프록시 오브젝트는 모든 요청을 리플렉션 정보로 변환해서 InvocationHandler 구현 오브젝트의 invoke 메소드로 넘기는 것이다. 타깃 인터페이스의 모든 메소드 요청이 하나의 메소드로 집중되기 때문에 중복되는 기능을 효과적으로 제공할 수 있다.
* 프록시 팩토리한테 Hello 인터페이스를 제공하면서 다이내믹 프록시를 만들어달라고 요청하면 Hello 인터페이스의 모든 메소드를 구현한 오브젝트를 생성해준다. InvocationHandler 인터페이스를 구현한 오브젝트를 제공하면 다이내믹 프록시가 받는 모든 요청은 InvocationHandler의 invoke 메소드 하나로 처리할 수 있다.
* 다이내믹 프록시로부터 메소드 호출 정보를 받아서 처리하는 InvocationHandler를 만들어보자.
* Public class UppercaseHandler implements InvocationHandler { Hello target; public Object invoke(Object proxy, Method method, Object[] args) { String ret = (String)method.invoke(target, args); -> 타깃으로 위임. 인터페이스의 모든 메소드 호출에 적용된다. return ref.toUppercase(); -> 부가기능 제공 }
* 다이내믹 프록시가 클라이언트로 받는 모든 요청은 invoke 메소드로 전달된다. 다이내믹 프록시 (sayHello) -> InvocationHandler (invoke(Method)) -> HelloTarget (sayHello) 이런 느낌인 것이다.
* 이제 이 InvocationHandler를 사용하고 Hello 인터페이스를 구현하는 프록시를 만들어보자. 다이나믹 프록시의 생성은 Proxy 클래스의 newProxyInstance() 스태틱 팩토리 메소드를 이용하면 된다.
* Hello proxiedHello = (Hello)Proxy.newProxyInstance(getClass().getClassLoader(), new Class[] { Hello.class }, new UppercaseHandler(new HelloTarget()));  # 생성된 다이내믹 프록시 오브젝트는 Hello 인터페이스를 구현하고 있으므로 Hello 타입으로 캐스팅해도 된다. # 동적으로 생성되는 다이내믹 프록시 클래스의 로딩에 사용할 클래스 로더, 구현할 인터페이스, 부가기능과 위임 코드를 담은 InvocationHandler 이다.
* 이런 식의 과정들이 과연 좋은 것인가? 리턴 타입이 스트링이 아니면 에러가 발생하니까 확인해서 넘기도록 변경해야 하는 단점이 있다. 반면 메소드가 많으면 확장성이 열려있어서 편하긴 하다. 게다가 타깃의 종류에 상관없이 적용이 가능하다.
* 따라서 class UppercaseHandler { Object target; public Object invoke(Object proxy, Method method, Object[] args) { Object ret= method.invoke(target, args); if ( ref instanceOf String) { }} 로 변경하면 된다.

다이내믹 프록시를 이용한 트랜잭션 부가 기능
* 이제 UserServiceTX를 다이내믹 프록시 방식으로 변경해보자. 트랜잭션 기능을 부가해주는 InvocationHandler 하나만 정의하면 될 것 같다.
* Public class TransactionHandler implements InvocationHandler { private Object target; private PlatformTransactionManager transactionManager; private String pattern; 패턴 받아서 해당 패턴을 가진 메소드들만 invokeTransaction 시킨다. 기존 인터페이스로 만든 클래스와 다른 점은 Method.invoke 를 사용하기 때문에 예외가 InvocationTargetException이 던져진다.
* 이제 클라이언트, UserServiceTest를 변경해주자. TransactionHandler txHandler = new TransactionHandler; txHandler.setTarget(testUserService); txHandler.setPattern(“upgradeLevels”); txHandler.setTransactionManager(transactionManager); UserService txUserService = (UserService)Proxy.newProxyInstance(getClass().getClassLoader(), new Class[] { UserService.class }, txHandler);
* UserServiceTx 오브젝트 대신 TransactionHandler를 만들고 타깃 오브젝트와 트랜잭션 매니져, 메소드 패턴을 주입해준다. 요걸로 다이내믹 프록시를 생성하면 모든 필요한 작업이 끝이다.

다이내믹 프록시를 위한 팩토리 빈
* TransactionHandler와 다이내믹 프록시를 스프링의 DI를 통해 사용할 수 있도록 해보자. 그런데 문제는 DI 대상이되는 다이내믹 프록시 오브젝트는 일반적인 스프링의 빈으로 등록할 방법이 없다. 스프링의 빈은 기본적으로 클래스 이름과 프로퍼티로 정의된다. 스프링은 지정된 클래스 이름을 가지고 리플렉션을 이용해 해당 클래스의 오브젝트를 만든다.클래스의 이름을 가지고 있다면 Date now = (Date) Class.forName(“java.util.Date”).newInstance(); 처럼 지 정된 클래스 이름을 가지고 리플렉션을 이용해 오브젝트를 생성한다.
* 문제는 다이내믹 프록시 오브젝트는 이런 식으로 프록시 오브젝트가 생성되지 않는다. 사실 다이내믹 프록시 오브젝트의 클래스가 어떤 것인지 알 수도 없다. 내부적으로 다이나믹하게 정의하기 때문이다. 따라서 미리 클래스 정보를 알아서 스프링의 빈에 등록할 방법이 없다.
* 스프링은 클래스 정보를 가지고 오브젝트를 만드는 방법 외에 빈을 만드는 방법을 제공한다. 팩토리 빈이라는 방법이 있는데, 스프링을 대신해서 오브젝트의 생성 로직을 담당하도록 만들어진 빈이다. 스프링의 팩토리 빈 인터페이스를 구현해보면 될 것 같다. 다음과 같다.
* Public interface FactoryBean<T> { T getObject() throws Exception; # 빈 오브젝트를 생성해서 돌려준다. Class<? Extends T> getObjectType(); # 생성되는 오브젝트 타입을 알려준다. Boolean isSignleton(); # getObject 가 돌려주는 오브젝트가 싱글톤 오브젝트인지 알려준다.
* FactoryBean 인터페이스로 구현한 클래스를 스프링의 빈으로 등록하면 팩토리 빈으로 동작한다. 동작원리를 확인하기 위해 테스트용 클래스를 정의해보자.
* Public class Message { private Message() { } # 생성자가 private 이라 스태틱 팩토리 메소드로만 만들 수 있음. Public static Message newMessage(String text) { return new Message(text);} }
* 스태틱 메소드를 사용해야만 오브젝트를 만들 수 있으므로 다음과 같이 스프링 빈으로 등록해서 사용할 수 없다. <bean id =“message” class=“~~.Message”> # 생성자를 통해 만들 수가 없기 때문이다. 요걸 해결하기 위해 팩토리 빈 클래스를 만들자.
* Public class MessageFactoryBean implements FactoryBean<Message> { public Message getObject() throws Exception { # 실제 빈으로 사용될 오브젝트를 직접 생성한다. Public boolean isSingleton() { return false } # 이 팩토리는 매번 새로운 오브젝트를 만들므로 false를 반환한다.
* 이제 스프링 컨테이너에서 MessageFactoryBean 에서 빈을 가져올 때 getObject로 Message를 빈 오브젝트로 등록한다. 따라서 <bean id=“message” class=“~~.MessageFactoryBean”> 으로 등록했을 때, message라는 아이디의 빈 오브젝트 타입은 Message이다. getObjectType 메소드가 돌려주는 타입으로 빈의 타입이 결정된다. 드물지만 팩토리 빈 자체를 가져오고 싶으면 &를 이름앞에 붙이면 된다. &message로 가져올 수 있다고 한다.

다이내믹 프록시를 만들어주는 팩토리 빈
* Proxy의 newProxyInstance() 메소드를 통해서만 생성이 가능한 다이내믹 프록시 오브젝트는 일반적인 방법으로 스프링의 빈 등록이 불가능하다. 대신 팩토리 빈으로 넣어줄 수 있다.
* 스프링 빈에는 팩토리 빈과 UserServiceImpl 만 빈으로 등록한다. 팩토리 빈은 다이내믹 프록시가 위임할 타깃 오브젝트인 UserServiceImpl에 대한 레퍼런스를 프로퍼티를 통해 DI 받아둬야 한다. 다이내믹. 프록시와 함꼐 생성할 TransactionHandler에게 타깃 오브젝트를 전달해줘야 하기 때문이다. 결국 팩토리 빈은 타깃 오브젝트를 DI 받은 TransactionHandler가 구현된 다이내믹 프록시 오브젝트를 만드는것이다.
* 트랜잭션 프록시 팩토리 빈은 다음과 같다. public class TxProxyFactoryBean implements FactoryBean<Object> { Object target; PlatformTransactionManager transactionManager; String pattern; Class<?> serviceInterface; public Object getObject() { TransactionHandler txHandler = new TransactionHandler(); txHandler.setTarget(this.target); txHandler.setTransactionManager(); txHandler.setPattern(); return Proxy.newProxyInstance(getClass().getClassLoader(), new Class []{serviceInterface}, txHandler); }
* 이제 <bean id =“userService” class=“TxProxyFactoryBean”> <property “target” ref=“userServiceImpl” /> <“transactionManager” “transactionManager”> <“serviceInterface” value=“UserService”> # properrty에서 value의 타입이 클래스면 해당하는 이름을 가진 클래스로 넣어준다. 그리고 UserServiceTx 는 없애자.
* UserServiceTest 를 살펴보자. @autowired 로 가져온 userService 빈을 사용하기 때문에 TxProxyFactoryBean 이 생성한 다이내믹 프록시를 통해 UserService를 사용하게 될 것이다. 그런데 문제는 upgradeAllOrNothing() 테스트가 문제다. 비지니스 로직을 수정한 TestUserService 오브젝트를 타깃 오브젝트로 대신 사용해야 한다. 설정에는 정상적인 UserServiceImpl 오브젝트로 지정되어 있지만 테스트 메소드에서 TestUserService 오브젝트가 동작하도록 해야 한다. TransactionHandler 와 다이내믹 프록시 오브젝트를 직접 만들어서 테스트했을 때는 타깃 오브젝트를 변경하기 쉬웠는데, 이제는 스프링 빈에서 호출한 팩토리 빈으로 프록시 오브젝트를 만들기 때문에 쉽지 않다. 가장 문제는 타깃 ㅇ오브젝트에 대한 레퍼런스는 TransactionHAnlder 오브젝트가 가지고 있는데, TransactionHandler가 TxProxyFactoryBean 내부에서 만들어져 생성에 사용될 뿐 참조할 방법이 없다. 따라서 이미 스프링 빈으로 만들어진 트랜잭션 프록시 오브젝트의 타깃을 변경하기 어렵다. 어떻게 할까
    * TestUserService를 사용하는 테스트용 설정을 별도로 만든다거나
    * 프록시 팩토리 빈 코드를 확장한다거나
    * TxProxyFactoryBean의 트랜잭션을 지원하는 프록시를 바르게 만들어주는지 확인하면되니까, TxProxyFactoryBean 을 직접 가져와서 프록시를 만들어보자.
* 앞에서 팩토리 빈은 내부에서 생성하는 오브젝트가 빈 오브젝트로 사용되지만, 우리는 팩토리 빈을 가져와서 target 프로퍼티를 재구성해준다음에 프록시 오브젝트를 생성하자. 이렇게 하면 컨텍스트의 설ㅇ저을 변경해버리므로 @DirtiesContext를 등록해주자.
* @Autowired ApplicationContext context; # 팩토리 빈을 가져오려면 필요하다. TxProxyFactoryBean txProxyFactoryBean = context.getBean(“&userService”, TxProxyFactoryBean.class); txProxyFactoryBean.setTarget(testUserService); UserService txUserService = (UserService) txProxyFactoryBean.getObject();

프록시 팩토리 빈 방식의 장점과 한계
* TransactionHandler 를 이용하는 다이내믹 프록시를 생성해주는 TxProxyFactoryBean 은 코드의 수정 없이도 다양한 클래스에 적용할 수 있다. 타깃 오브젝트에 맞는 프로퍼티만 설정해서 빈으로 등록해주면 된다. 꽂아주는 것만으로 모든 클래스에 트랜잭션을 쓸 수 있다.
* 앞에서 데코레이터 패턴이 적용된 프록시를 이용하면 장점이 있지만 적극적으로 활용되지 못하는 이유는 두가지가 있다고 했다.
    * 프록시를 적용할 대상이 구현하고 있는 인터페이스를 구현하는 프록시 클래스를 일일이 만들어야 하고
    * 코드 위임 및 중복되는 코드가 생겨 중복의 문제가 있다.
* 이 두 문제를 프록시 팩토리 빈이 해결해준다. 스프링의 DI가 있었기에 문제점을 해결할 수 있었다.
* 그런데, 중복 없는 최적화된 코드를 하려고 하면 한계에 부딪힌다. 프록시를 통해 타깃에 부가기능을 제공하는 것은 메소드 단위로 일어난다. 하나의 클래스 안에 존재하는 여러 개의 메소드에 부가기능을 한 번에 제공하는 것은가능했으나, 한 번에 여러개의 클래스에 부가기능을 제공하는 일은 지금까지 방법으로는 불가능하다. 하나의 타깃 오브젝트에만 부여되는 부가기능은 상관 없지만, 트랜잭션과 같이 비지니스 로직을 담은 많은 클래스의 메소드에 적용해야 한다면, 중복이 졸라 생긴다.
* 하나의 타겟에 여러 개의 프록시를 달 때도 문제다. 적용 대상인 서비스 클래스가 200개되면 부가기능이 5개면 몇천줄로 늘어나고 DI xml 은 사람 손으로 편집할 수 없다. 게다가, TransactionHandler 오브젝트가 프록시 팩토리 빈 개수만큼 만들어진다. 동일한 코드임에도 불구하고 타깃 오브젝트가 달라지면 새로운 TransactionHandler 가 생긴다. 싱글톤 빈으로 사용할수는 없을까? 등등의 고민들부터 어마어마한 문제가 생기는데, 스프링의 DI 가 있다면 이것도 해결할 수 있다.

스프링의 프록시 팩토리 빈
* 스프링은 트랜잭션 기술과 메일 발송 기술에 적용했던 서비스 추상화를 프록시 기술에도 적용하고 있다. 자바에는 JDK에서 제공하는 다이내믹 프록시 외에도 프록시를 제공하는 다양한 기술이 존재한다. 따라서 스프링은 일관된 방법으로 프록시를 만들 수 있게 도와주는 추상 레이어를 제공한다. 생성된 프록시는 스프링의 빈으로 등록되어야 한다. 스프링은 프록시 오브젝트를 생성해주는 기술을 추상화한 팩토리 빈을 제공해준다.
* 스프링의 ProxyFactoryBean은 프록시를 생성해서 빈 오브젝트로 등록하게 해주는 팩토리 빈이다. 기존에 만들었던 TxProxyFactory 빈과 달리 ProxyFactoryBean은 순수하게 프록시를 생성하는 작업만을 담당하고 프록시를 통해 제공해줄 부가기능은 별도의 빈에 둘 수 있다. ProxyFactoryBean이 생성하는 프록시에서 사용할 부가기능은 MethodInterceptor 인터페이스를 구현해서 만든다. InvocationHandler 와 비슷하지만 한 가지 다른 점이 있다. InvocationHandler의 invoke() 메소드는 타깃 오브젝트에 대한 정보를 제공하지 않는다. 따라서 타깃은 InvocationHandler를 구현한 클래스가 직접 알고 있어야 한다. 반면에 MethodInterceptro의 invoke 메소드는 ProxyFactoryBean으로부터 타깃 오브젝트에 대한 정보까지도 함께 제공받는다. 그래서 타깃 오브젝트와 상관 없이 독립적으로 만들어질 수 있다. 따라서 타깃이 다른 여러 프록시에서 함꼐 쓸 수도 있고 싱글톤 빈으로 등록 가능하다.


