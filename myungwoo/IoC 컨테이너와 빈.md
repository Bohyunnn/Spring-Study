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
