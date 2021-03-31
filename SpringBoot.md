# SpringBoot

@RestController : @Controller , @ResponseBody

@SpringBootApplication : @Configuration, @EnableAutoConfiguration, @ComponentScan



**Spring 구성요소**

//TODO



**Spring 주요 키워드**

- IOC(Inversion of Control)
  - DL(Dependency Lookup)
  - DI(Dependency Injection)
- POJO(Plain Old Java Object)
- AOP(Aspect Oriented Programming)



**Spring의 주요 모듈**

- Spring JDBC
- Spring MVC
- Spring AOP
- Spring ORM
- Spring JMS
- Spring TEST
- Spring Expression language (SPEL)



## RESTful Web Service

https://spring.io/guides/gs/rest-service/

RESTful 웹서비스는 스프링관점에서, controller가 HTTP requests를 처리한다. controller은 ```@RestController```어노테이션으로 식별가능하다. 예를 들면

~~~java
@RestController
public class GreetingController {

	private static final String template = "Hello, %s!";
	private final AtomicLong counter = new AtomicLong();

	@GetMapping("/greeting")
	public Greeting greeting(@RequestParam(value = "name", defaultValue = "World") String name) {
		return new Greeting(counter.incrementAndGet(), String.format(template, name));
	}
}
~~~

```@GetMapping```에 의해 /greeeting로 온 HTTP GET requests는 greeting method에 매핑된다. 마찬가지로 ```@PostMapping```이 존재하고, ```@RequestMapping(Method=GET or POST)```로 표현할 수 있다.



전통적인 MVC컨트롤러와 RESTful web service 컨트롤러의 중요 차이점은 HTTP response body를 생성하는 방법에 있다. Greeting object를 리턴하면 object data로 JSON형태의 HTTP response가 자동으로 만들어진다. 이는 Jackson2가 클래스경로에 있어서 스프링의 ```@MappingJackson2HttpMessageConverter```가 자동으로 Instance를 JSON으로 변환해주기 때문이다.





## Dependency Injection (DI) and Inversion of Control (IOC)

https://dzone.com/articles/spring-vs-spring-boot

~~~java
@restcontroller
public class mycontroller {
    private myservice service = new myservice();

    @requestmapping("/welcome")
    public string welcome() {
        return service.retrievewelcomemessage();
    }
}
~~~

위의 예시코드를 보면 mycontroller는 myservice에 의존적이고 강하게 결합돼있다. 

의존성이 문제를 가지는 이유는 (https://gmlwjd9405.github.io/2018/11/09/dependency-injection.html)

- myservice 객체를 변경하게 되면 mycontroller도 수정되어야 한다.
- 하나의 모듈이 바뀌면 의존한 다른 모듈까지 변경되어야 한다.
- 두 객체 사이의 의존성 때문에 Unit Test 작성이 어렵다.

Spring에서는 위의 문제를 해결하기 위해 DI기능을 기본적으로 제공한다. DI를 위해 사용되는 2가지 annotation이 있다.

- ```@component``` 

  @component로 등록하면 beanfactory에 관리해야 할 bean으로 등록된다.

- ```@autowired```

  특정 타입의 올바른 매칭과 자동연결(autowire)에 쓰인다.

~~~java
@component
public class myservice {
    public string retrievewelcomemessage(){
       return "welcome to innovationm";
    }
}
~~~

~~~java
@restcontroller
public class mycontroller {

    @autowired
    private myservice service;

    @requestmapping("/welcome")
    public string welcome() {
        return service.retrievewelcomemessage();
    }
}
~~~

위처럼 사용하면, spring framework에서 myservice bean을 만들고, mycontroller에서 myservice를 자동으로 연결한다.



