## IoC컨테이너

: 어떤 객체, 어떤 클래스가 의존 객체를 직접 만들어서 쓰는게 아니라 어떤 장치를 통해 의존성을 주입받는 방법

-> 스프링 애플리케이션에서 객체 생성, 관계 설정, 사용, 제거를 코드 대신 독립된 컨테이너가 담당

1. BeanFactory (Bean을 담고 있는 Container, 객체 생성과 객체 사이의 런타임 관계 설정하는 DI 관점)

2. 애플리케이션 컴포넌트의 중앙 저장소

3. 빈 설정 소스로부터 빈 정의를 읽어들이고 빈을 구성하고 제공

```

@Service

public class BookService {

/** 객체를 직접 만들어서 내부에서 주입받는 방식  **/

/**
 private BookRepository bookRepository = new BookRepository();

 private BookeService bookService = new BookService(bookRepository);
 
 **/
 
 
 /** IoC 컨테이너에 의해 만들어진 Bean을 의존성 주입 **/
private BookRepository bookRepository;

pubic BookService(BookRepository bookRepository) {

        This.bookRepository = bookRepository;

}

(생략)

}

 ```
 

## Bean

: 스프링 IoC 컨테이너가 관리하는 객체

[장점]

1. 의존성 관리

2. 스코프 – 싱글톤(하나) / 프로토타입(매번 다른 객체)

3. 라이프사이클 인터페이스 지원

 

## ApplicationContext

: 가장 많이 사용하는 빈 / BeanFactory를 상속한 서브 인터페이스

1. 메시지 소스 처리 기능

2. 이벤트 발행 기능

3. 리소스 로딩 기능

   이 외에도 여러가지 기능이 존재한다.


Bean을 설정하는 방법 3가지
: Xml, Java, Annotation

1. Xml로 Bean 설정
```
<bean id = “bookService” class=”~~.bookService”>

           <property name=”bookRepository” ref=”bookRepository”/>

</bean>

<bean id = “bookRepository” class=”~~.bookRepository”></bean>

<context:component-scan base-package=”~~”/>

main 함수>>

ApplicationContect context = new ClassPathXmlApplicationContect(‘’applicaion.xml”);

String[] beanDefinitionNames = context.getBeanDefinitionNames();

Sysout(Arrays.toString(beanDefinitionNames));

BookService bookService = (BookService) context.getBean(“bookService”);
```

 

2. Annotation으로 Bean 설정

@Component : Bean 주입

@Service

@Repository

@Autowired : 의존성 주입

 

3. Java로 Bean 설정

```
@Configuraion

Public class ApplicationConfig {

	@Bean

	Public BookRepository bookRepository() {

		return new BookRepository;

	}

	@Bean

	public BookService bookService() {

		BookService bookService = new BookService();

		bookService.setBookRepository(bookRepository());

		return bookService;

	}

}


main 함수>>

ApplicationContext context = new ApplicationConfigApplicationContext(ApplicationConfig.class);

String[] beanDefinitionNames = context.getBeanDefinitionNames();

Sysout(Arrays.toString(beanDefinitionNames));

BookService bookService = (BookService) context.getBean(“bookService”);
```

@Autowired    required: 기본값은 true (false이면 해당하는 bean이 없어도 구동 됨)

사용할 수 있는 위치 -> 생성자, 세터, 필드

해당 타입의 빈이 없는 경우 `@AutoWired(required=false)`

해당 타입의 빈이 하나인 경우 `@AutoWired(required=true)`

해당 타입의 빈이 여러 개인 경우

    @Repository
    
    public class bohyunRepository implements bookRepository { …. }
    
    public interface BookRepository {...}
    
    @Repository
    
    public class bookRepository { …. }

Repository로 등록된 bean이 2가지 이상이면 에러남

           빈 이름으로 시도, 같은 이름의 빈 찾으면 해당 빈 사용 / 같은 이름 못찾으면 실패

** 해결방법:

1.   @Primary: 같은 빈이 있다면 우선순위로 등록함.

    @Repository @Primary
    
    public class bohyunRepository implements bookRepository { …. }

2.   @Qualifier(“빈이름”) : 같은 빈이 있다면 빈으로 등록하고 싶은 클래스의 이름을 쓰면됨.

    @Qualifier(“bohyunRepository”)
    
    BookRepository bookRepository

3.    해당 타입의 빈 모두 주입 받기

    @Autowired
    
    List<BookRepository> bookRepository;
    
    @Repository
    
    public class bohyunRepository implements bookRepository { …. }
    
    @Repository
    
    public bookRepository { …. }

cf) 필드 이름이 같아도 의존성 주입이 됨.

    @Repository
    
    public class bohyunRepository implement Repository{ … }
    
    @Autowired
    
    BookRepository bohyunRepository;

 
BeanPostProcessor
: 새로 만든 빈 인스턴스를 수정할 수 있는 라이프 사이클 인터페이스
  Bean의 인스턴스를 만들고 InitializingBean

  ```  AutowiredAnnotationBeanPostProcessor extends BeanPostProcessor```

: 스프링이 제공하는 @Autowired와 @Value 애노테이션 그리고 JSR-330의 @Inject 애노테이션을 지원하는 애노테이션 처리기

Cf) BeanFactory가 자신에게 등록되있는 Bean을 찾으면서 BeanPostProcessor을 찾아서 다른 일반적인 Bean들에게 BeanPostProcessor의 로직을 적용시킴

 
Ex)

    public class BookService implements I
    @PostConstruct
    Public void setUp() { … }


**@Component**
-    @Repository
-    @Service
-    @Controller
-    @Configuration

@ComponentScan
: 스캔할 패키지와 애노테이션에 대한 정보

실제 스캐닝은 ConfiturationClassPostProcessor라는 BeanFacotoryProcessor에 의해 처리됨

    main(String[] args) {
    	SpringApplication.run(BohyunApplication.class, args);~~
    }
    
     main(String[] args) {
    
	    Var app = new SpringApplication(bohyunApplication.class);
    
    	App.addInitializers(new ApplicaionContextInitializer<...> {
    		@Override
    		public void initializer(~~~);
    	});
    	App.run(args);
    }


 **빈의 스코프**
**Environment**
프로파일과 프로퍼티를 다루는 인터페이스

프로파일
빈들의 그룹
Environment의역할을 활성화할 프로파일 확인 및 설정

프로파일 유즈케이스
테스트 환경에서는 A라는 빈을 사용하고 배포환경에서는 B라는 빈을 쓰고 싶다.
이 빈은 모니터링 용도니깐 테스트할 때는 필요가 없고 배포할 때만 등록이 되면 좋겠다.

 
프로파일 정리하기

클래스에 정의
@Configuration @Profile(“test”)

메소드에 정의
@Bean @Profile(“test”)

프로파일 설정하기
-Dspring.profiles.avive = “testA,B...”
@ActiveProfiles(테스트용)


**프로퍼티**

다양한 방법으로 정의할 수 있는 설정값

Environment의 역할은 프로퍼티 소스 설정 및 프로퍼티 값 가져오기
프로퍼티에는 우선 순위가 잇다
StandartServiceEnvironment의 우선순위
ServletConfig 매개변수
ServletContext매개변수
JNDI(java.comp/eve/)
JVM 시스템 프로퍼티 (-Dkey=”value”)
JVM 시스템 환경 변수 (운영체제 환경 변수)

@PropertySource
Environment를 통해 프로퍼티 추가하는 방법
스프링 부트의 외부 설정 참고
기본 프로퍼티 소스 지원(application.properties)
프로파일을 고려한 계층형 프로퍼티 우선 순위 제공
 

**MessageSource**

국제화(i18n)기능을 제공하는 인터페이스

    ApplicationContext extends MessageSource
    getMessage(String code, Object[] args. String defai;t, Local loc)

스프링 부트를 사용한다면 별다른 설정 필요없이 messages properties 사용할 수 있음
messages.properties
messages_ko_kr.properties

ApplicationEventPublisher

: 이벤트 프로그래밍에 필요한 인터페이스 제공, 옵저버 패턴 구현체

   ``` 
   ApplicationContext extends ApplicationEventPublisher
   publishEvent(ApplicationEvent event)
   ```

이벤트 만들기
- ApplicationEvent 상속
-  스프링 4.2부터는 이 클래스를 상속받지 않아도 이벤트로 사용할 수 있다.

이벤트 발생시키는 방법

    ApplicationEventPublisher.publishEvent();
    
이벤트 처리하는 방법
- ApplicationListener<이벤트> 구현한 클래스 만들어서 빈으로 등록하기
- 스프링 4.2 부터 @EventListener을 사용해서 빈의 메소드에 사용할 수 있다.
- 기본적으로 Synchronized
-  순서를 정하고 싶다면 @Order와 함께 사용
-  비동기적으로 실행되고 싶다면 @Async와 함께 사용

 

스프링이 제공하는 기본 이벤트
- ContextRefreshedEvent: ApplicationContext를 초기화 했거나 리프래시 했을 때 발생
-  ContextStartedEvent: ApplicationContext를 start()하여 라이프사이클 빈들이 시작신호를 받은 시점에 발생
- ContextStoppedEvent: ApplicaionContext를 stop()하여 라이프사이클 빈들이 정지 신호를 받은 시점에 발생
- RequestHandlerEvent: HTTP 요청을 처리했을 떄 발생


ResourceLoader
: 리소스를 읽어오는 기능을 제공하는 인터페이스

    ApplicationContext extends ResourceLoader

리소스 읽어오기
- 파일 시스템에서 읽어오기
- 클래스패스에서 읽어오기
- URL로 읽어오기
- 상대/절대 경로로 읽어오기

    resource getResource(java.lang.String location)

 
