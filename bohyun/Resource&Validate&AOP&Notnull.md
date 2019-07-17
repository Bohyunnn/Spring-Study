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



**Validate**

: Application에서 사용하는 객체 검증용 인터페이스



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
