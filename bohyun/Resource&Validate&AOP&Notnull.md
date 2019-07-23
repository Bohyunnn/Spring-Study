ApplicationContext는 Bean Factory Resouce publisher, Message Source, Environment Capable 기능


**Resource 추상화**

java.net.URL을 추상화 한 것
스프링 내부에서 많이 사용하는 인터페이스
[추상화 한 이유]
- classpath를 통해 가져오는 기능이 없음
- ServletContext를 기준으로 상대 경로를 읽어오는 기능이 없음
- 새로운 핸들러를 등록하여 특별한 URL 접미사를 만들어 사용할 수 있지만 구현이 복잡하고 편의성 메소드가 부족하다.
[https://blog.outsider.ne.kr/794](https://blog.outsider.ne.kr/794)

[구현체]
- UrlResource
: java.net.URL, http, https, ftp, file, jar 프로토콜 지원
-  ClassPathResource
: classPath에서 얻어와야하는 Resource
- FileSystemResource
: java.io.File  핸들에 대한 Resource 구현
- ServletContextResource
: 상대 경로를 인터프리팅하는 ServletContext 리소르에 대한 Resource 구현
```
ex1) Resource resource = resourceLoader.getResouce("classpath:test.txt");
-> classPathResouce(ClassPathXmlApplicationContext) 참조

ex2) Resource resource = resourceLoader.getResouce("file://test.txt");
-> fileSystemResource(FileSystemXmlApplicationContext) 참조

ex2) Resource resource = resourceLoader.getResouce("test.txt");
-> ServletContextResource(WebApplicationContext) 참조
```

-------------------------------------


**Validate**

: Application에서 사용하는 객체 검증용 인터페이스

1. **PropertyEditor**

[org.springframework.validation.DataBinder](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/validation/DataBinder.html)

[](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/validation/DataBinder.html)

-   기술적 관점: 프로퍼티 값을 타겟 객체에 설정하는 기능이다.
-   사용자 관점: 사용자 입력값을 애플리케이션 도메인 모델에 동적으로 변환해 넣어주는 기능이다. 즉, 입력값은 대부분 문자열인데 그 값을 객체가 가지고 있는 int, long, Boolean 등 알맞은 도메인 타입으로 변환하여 넣어주는 기능이다.
```
public class DemoEditor extends PropertyEditorSupport {
	@Override
	public String getAsText() {
		Demo demo = (Demo)getValue();
		return demo.getId().toString();
	}
	@Override
	public void setAsText(String text) throws IllegalArgumentException {
		setValue(new Demo(Integer.parseInt(text)));
	}
}
```

1. PropertyEditorSupport를 상속받으면 원하는 것만 구현할 수 있다.

2. String 타입 text를 Integer 타입으로 변환하여 생성자에 전달해주고 setValue에 넣어준다.

3. getValue, setValue는 PropertyEditor가 가지고 있는 값이기 때문에 서로 다른 스레드에게 공유가 되기 때문에 state full하다. (thread-safe 하지 않음) -> Bean으로 등록해서 사용하면 안된다.

4. String과 Object 간의 변환만 가능하기 때문에 범위가 제한적이다.

  
```
@RestController
public class DemoContoller {
	@InitBinder			
	public void init(WebDataBinder webDataBinder) {
		webDataBinder.registerCustomEditor(Demo.class, new DemoEditor());
	}
	@GetMapping(value="/demo/{demo}") 
	public String getDemo(@PathVariable Demo demo) {
		System.out.println(demo);
		return demo.getId().toString();
	}
}
```

1. @InitBinder: WebDataBinder에 추가한 커스텀 propertyEditor는 메소드가 있는 Controller 클래스 안에서만 동작한다. Demo 클래스 타입을 처리할 propertyEditor로 DemoEditor를 사용하도록 설정할 수 있다.

2. getDemo 함수에서 도메인 네임으로 "/demo/{demo}"을 사용하고 사용자의 입력값 {demo}는 Demo 타입으로 변환해서 넣어줘야한다.

 ``` 
public class Demo {
	private Integer id;
	private String title;
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public String getTitle() {
		return title;
	}
	public void setTitle(String title) {
		this.title = title;
	}
	@Override
	public String toString() {
		return "Demo [id=" + id + ", title=" + title + "]";
	}
	public Demo(Integer id) {
		this.id = id;
	}
}
```

=> 스프링 3.0 이전까지 DataBinder시 사용했던 인터페이스이다. 스프링 3.0 이후부터는 PropertyEditor를 대신할 수 있는 새로운 타입 변환 API인 Converter와 Formatter가 도입됐다.



2. **Converter와 Fommater**

**"Converter"**

-   S타입을 T타입으로 변환할 수 있는 매우 일반적인 변환기
-   Thread-safe(stateless) 하다.
```
public class DemoConverter {
	public static class StringToEventConverter implements Converter<String, Demo>{
		@Override
		public Demo convert(String source) {
			return new Demo(Integer.parseInt(source));
		}
	}
	public static class ObjectToStrintConverter implements Converter<Demo, String> {
		@Override
		public String convert(Demo source) {
			return source.getId().toString();
		}
	}
}
```

1. Thread-safe 하기 때문에 Bean으로 등록할 수 있다.

2. Converter<변환하기전 타입 오브젝트, 변환할 타입 오브젝트>

  

![](https://k.kakaocdn.net/dn/d8kmxX/btqwXIfVOj1/iamvq1dmMacaant48bx3ik/img.png)

- coneverter 인터페이스-
```
@Configuration
public class WebConfig implements WebMvcConfigurer {
	@Override
	public void addFormatters(FormatterRegistry registry) {
		registry.addConverter(new DemoConverter.StringToEventConverter());
	}
}
```
1. Spring Framework라면 WebConfig에 Converter를 등록해야한다.

**"Fommater"**

-   Object와 String 간의 변환을 담당한다.
-   문자열을 Locale에 따라 다국화하는 기능도 제공한다.(optional)
-   String 타입의 폼 필드 정보와 컨트롤러 메소드의 파라미터 사이에 양방향으로 적용할 수 있도록 두 개의 변환 메소드를  갖고 있다.

**ConversionService**
-   실제 타입을 변환하는 작업은 ConversionService를 사용함
-   thread-safe하게 사용할 수 있음
-   Spring MVC, Bean 설정, SPEL에서 사용
-   DefaultFormattingConversionService(FormatterRegistry, ConverterRegistry, 여러 기본 컨버터와 포메터 등록해줌)

![](https://k.kakaocdn.net/dn/d5yd3C/btqwVS5o0e2/K10tsiQ0FqtobM8KLtafPK/img.png)

-DefaultFormattingConversionService-

**Spring Boot**

-   웹 애플리케이션인 경우에 DefaultFormattingConversionService를 상속하여 만든 WebConverterService를 빈으로 등록해 준다.
-   Converter와 Formatter 빈을 찾아 자동으로 등록해준다.

public class DemoConverter {
	
    @Component
	public static class StringToEventConverter implements Converter<String, Demo>{
		@Override
		public Demo convert(String source) {
			return new Demo(Integer.parseInt(source));
		}
	}
	
    @Component
	public static class ObjectToStrintConverter implements Converter<Demo, String> {
		@Override
		public String convert(Demo source) {
			return source.getId().toString();
		}
	}

}

1. Converter를 @Component를 이용해 Bean으로 등록할 수 있다.


-------------------------------------


**SPEL(Expression Language)**

객체 그래프를 조회하고 조작하는 기능을 제공

Unified EL과 비슷하지만 메소드 호출을 지원하며 문자열 템플릿 기능도 제공한다.

OGNL, MVEL, JBOss, EL 등 자바에서 사용할 수 있는 여러 EL이 있지만, SpEL은 모든 스프링 프로젝트 전반에 걸쳐 사용할 EL로 만들었다.

스프링 3.0부터 지원


[문법]

#(“표현식”)

$(“프로퍼티”)

#($(my.date) +1) 표현식은 프로퍼티를 가질 수 있지만, 반대는 안됨

Cf) ApplicationRunner 인터페이스는 SpringBootApplication이 실행된 후 바로 실행됨
```
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;

@Component
public  class AppRunner implements ApplicationRunner {

	@Value("#{1+1}")
	int  value;

	@Value("#{'hello' + 'world'}")
	String greeting;

	@Value("#{ 1 eq 1}")
	boolean  trueOfFalse;

	@Override
	public  void run(ApplicationArguments args) **throws** Exception {
		System._out_.println("---------");
		System._out_.println(value);
		System._out_.println(greeting);
		System._out_.println(trueOfFalse);
		System._out_.println("---------");
	}
}
```

사용하는곳)
- @Value 애노테이션
- @ConditionOnExpression 애노테이션
- 스프링 시큐리티
- 메소드 시큐리티 @PreAuthorize, @PostAuthorize, @PreFilter, @PostFilter
- XML 인터셉터 URL 실행
- 스프링 데이터
- Query 애노테이션
- Thymleaf

-------------------------------------

**Spring AOP(Aspect-oriendted Programming)**

: OOP를 보완하는 수단으로, 흩어진 Aspect를 모듈화 할 수 있는 프로그래밍 기법

: 로깅, 트랜잭션 처리할 때 공통 관심사항을 묶어서 처리


AOP 주요개념

- Aspect와 Target

- Advice(해야할일)

- Join Point와 Pointcut(어디에 적용할건지)


AOP 적용방법

- 컴파일 (AspectJ)

- 로딩 타임 (AspectJ)

- 런타임 (Spring AOP)
