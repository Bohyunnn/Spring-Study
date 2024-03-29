1. 스프링 IoC 컨테이너와 빈 
- Ioc
: Inversion of Control
의존하는 관계를 직접생성하는 것이 아니라 생성자 또는 setter등을 사용해서 DI(주입) 해서 사용
내가 직접하는 일을 남이 해줌.
하는일 - 빈 인스턴스 생성. 의존관계 설정. 빈제공


- Beanfactory
:빈 설정 소스 빈정의 읽고 빈구성 및 제공
- bean
: 스프링 IoC 컨테이너가 관리하는 객체,의존성 관리 뛰어남. 

- ApplicationContext
: Beanfactory 상속받음. 오브젝트 생성, 관계설정, 후처리 등 여러가지 일을함

* Beanfactory와 ApplicationContext 차이점
: 인스턴스 생성시점이 다름.
ApplicationContext는 Pre-loading을 하고 즉, 즉시 인스턴스를 만들고 
BeanFactory는 lazy-loading을 하여 실제 요청 받는 시점에서 인스턴스를 만듬
정리하자면 두 인터페이스의 차이점은 인스턴스화 시점이 다름

- Component Scan
:특정 패키지 이하의 모든 클래스 중에 @Component 애노테이션을 사용한 클래스를
빈으로 자동으로 등록 해 줌.
즉 application 시작부터 해당패키지까지 컴포넌트를 읽어들임.
ex) @Component, @Controller, @Service, @Repository

- @Autowired
- required: 기본값은 true (따라서 못 찾으면 애플리케이션 구동 실패)
사용할 수 있는 위치
생성자 (스프링 4.3 부터는 생략 가능), 세터, 필드

- 같은 타입의 빈이 여러개 일때 한개만 지정해서받는법
: @Primary를 쓴다

- 같은 타입의 빈이 여러개 일때 여러개 모두 주입받는법
: @Qualifer 빈 이름으로 주입받는다.

- 빈의 scope
: 싱글톤, 프로토타입

- 프로토타입이 싱글톤을 참조하면 문제없음
but 프로토타입이 싱글톤 참조시 업데이트가 안됨. 

- proxy를 사용해서 싱글톤 빈이 프로토 타입을 업데이트 하게 한다
: scoped-proxy, obeject-provider

- Environment Capable 중 하나인 프로파일기능
: getEnvironment() 

- 프로파일
: bean들의 묶음, 환경설정

environment.getActiveProfile() : 디폴트로 아무것도 없음
environment.getDefaultProfile(): default 나옴 (아무것도 설정하지 않은상태)

- 프로파일 선언하는 방법(클래스)
@Configuration
@Profile("프로파일이름")

- 프로파일 선언하는 방법(메소드
@Bean
@Profile("프로파일이름")

- 프로파일 설정방법:
: IDE에서 active profile에서 사용
-Dspring.profile.actvie="test,A,B"

-테스트 프로파일
@ActvieProfiles

- 프로파일 표현식
:!(not), &(and), |(or)

- 프로퍼티
: IDE에서 VM option 으로 줄수도 있음 
@Properties("classpath:app.properties")
environment.getProperty("app.name")

- 프로퍼티 우선순위
:ServletConfig 매개변수
ServletContext 매개변수
JNDI(java/comp/env)

@Value("${app.name}"}

JVM 시스템 프로퍼티 (-Dkey="value")
JVM 시스템 환경변수 (운영체체 환경변수)


- MessageSource
:메세지 다국화 (i18n)
ApplicationContext  안에 messageSource 인터페이스
messageSource.getMessage()
스프링부트일경우 (messages.properties)
messages.ko_KR.propperties

messageSource.setBaseName
messageSource.setDefalutEncording("UTF-8)



- ApplicationEventPublisher
: 이벤트 프로그래밍에 필요한 인터페이스 제공
ApplicationContext extends ApplicationEventPublisher
: publishEvent(ApplicationEvent event)

- 이벤트 핸들러를 빈에 등록해줘야함
:@EventListener 라는 어노테이션을 추가해줘야한다

- pojo 기반의 프로그래밍
: 스프링 소스가 노출되지 않음

- 이벤트 핸들러를 여러개 사용할경우 기본적으로 순차적으로 실행함
:@Order 라는 어노테이션을 이용해서 순서를 정할수 있음
ex) @Order(Orderd.HIGHEST_PRECEDNCE)

- 비동기적으로 (순서없이) 사용하고 싶다면 @Async라는 어노테이션 사용
: 스프링부트 메인 클래스에 @EnableAsync라는 어노테이션을 해줘야 @Async라는 어노테이션이 동작함



* 스프링이 제공하는 기본 이벤트
- ContextRefreshdEvent: ApplicationContext를 초기화 했거나 리프래시 할때 발생
- ContextStartedEvent: ApplicationContext를 start() 하여 빈들이 시작신호를 받은 시점에 발생
- ContextStoppdEvent: ApplicationContext를 stop() 하여 빈들이 정지신호를 받은 시점에 발생
- ContextClosedEvent: ApplicationContext를 close()하여 싱글톤 빈 소멸되는 시점에 발생.
- RequestHandledEvent: Http요청을 처리했을 때 발생


- ResourceLoader
리소스를 읽어오는 기능을 제공하는 인터페이스
ApplicationContext extends ResourceLoader

@Autowired
ResourceLoader resourceLoader;

ResourceLoader.getResource(location:"classpath:resouces/test.txt");
ResourceLoader.getDescription();

