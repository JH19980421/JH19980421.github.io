---
layout: post
---
<br>

# Spring Framework

<br>
스프링 프레임워크란 자바 기반의 엔터프라이즈급 애플리케이션 개발을 위한 프레임워크이다. 애플리케이션을 효율적으로 개발하기 위해 다음과 같은 주요 모듈들을 제공한다.
<br>
- **Core Container:** 인스턴스를 생성하고 관리하는 **IoC(제어 역전)**, **DI(의존성 주입)**를 담당 (Beans, Core, Context, SpEL)
- **AOP (Aspect Oriented Programming):** 로깅, 트랜잭션, 보안 등 공통 관심사를 비즈니스 로직과 분리하여 관리
- **Data Access / Integration:** 데이터베이스 통신 및 데이터 통합 역할 (JDBC, ORM, OXM, JMS, Transactions)
- **Web:** 웹 애플리케이션 개발에 필요한 기능 제공 (Web, Servlet, WebSocket 등)

<br>
<br>

### IoC (Inversion of Control, 제어 역전)

<br>
기존에는 사용자가 `new` 연산자를 통해 직접 객체를 생성하고 관리해야 했다. 하지만 IoC를 통해 **객체의 제어권이 프레임워크(컨테이너)로 넘어가게 되었다.** 개발자는 객체 생성에 신경 쓰지 않고 비즈니스 로직 작성에만 집중할 수 있게 된다.
<br>
<br>

### DI (Dependency Injection, 의존성 주입)
<br>
<br>
DI는 IoC를 구현하는 구체적인 방법 중 하나로, 사용할 객체를 직접 생성하지 않고 **컨테이너가 생성한 객체를 주입받아 사용**하는 방식이다. 크게 세 가지 주입 방법이 있다.
<br>
<br>

1. **생성자 주입 (Constructor Injection) : [권장 방식]**
객체가 생성될 때 딱 한 번만 주입된다. **불변성(final)**을 보장할 수 있고, 생성 시점에 의존성이 모두 충족되어야 하므로 Null 에러를 방지할 수 있다.
    
    `@Service
    public class ShopService {
        private final ShopRepository shopRepository;
    
        // 생성자가 하나일 경우 @Autowired 생략 가능 (Spring 4.3+)
        public ShopService(ShopRepository shopRepository) {
            this.shopRepository = shopRepository;
        }
    }`

<br>

2. **필드 주입 (Field Injection) :**
변수에 바로 `@Autowired`를 붙이는 방식. 코드가 간결하지만 외부에서 변경이 불가능하여 테스트가 어렵고, 프레임워크에 의존적이라는 단점이 있다.
    
    ```java
    @Service
    public class ShopService {
        @Autowired
        private ShopRepository shopRepository;
    }
    ```
<br>

3. **Setter 주입 (Setter Injection) :**
주입받는 객체가 런타임에 변할 가능성이 있을 때 사용한다. 하지만 객체가 생성된 후에도 의존성이 변경될 수 있어 안정성이 떨어진다.
    
    ```java
    @Service
    public class ShopService {
        private ShopRepository shopRepository;
    
        @Autowired
        public void setShopRepository(ShopRepository shopRepository) {
            this.shopRepository = shopRepository;
        }
    }
    ```
    

---
<br>

# Spring Boot
<br>
<br>
스프링 프레임워크는 강력하지만 설정이 매우 복잡하다는 단점이 있다. 이를 보완하기 위해 **"설정보다 관습(Convention over Configuration)"**이라는 철학으로 복잡한 설정을 자동화하고, 서버까지 내장하여 즉시 실행 가능한 형태로 만든 것이 스프링 부트이다.

<br>
<br>

### AutoConfiguration (자동 설정)
<br>
스프링 부트의 핵심 기능으로, 클래스패스에 추가된 라이브러리를 바탕으로 필요한 환경 설정을 알아서 구성해준다.

<br>

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```
<br>

위의 `@SpringBootApplication` 내부를 살펴보면 핵심이 되는 두 어노테이션이 있다.

<br>

1. **`@ComponentScan`:** 프로젝트 패키지 내의 `@Component` 시리즈(`@Controller`, `@Service`, `@Repository`, `@Configuration` 등)를 찾아 Bean으로 등록한다.
2. **`@EnableAutoConfiguration`:** 프로젝트에 포함된 라이브러리(Jar)를 분석하여 필요한 빈들을 자동으로 등록한다. 이때 판단의 기준이 되는 것이 바로 **`spring.factories`** (또는 최신 버전의 **`.imports`**) 파일이다.
   
<br>

### spring.factories와 작동 원리
<br>
자동 설정 대상이 되는 클래스들은 라이브러리 내부의 특정 파일에 명시되어 있다.
<br>

- **Spring Boot 2.7 이전:** `META-INF/spring.factories`
- **Spring Boot 2.7 이후/3.x:** `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`

<br>

`# spring.factories 예시 중 일부
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration`

<br>

이 클래스들은 내부에 `@Conditional` 어노테이션(예: `@ConditionalOnClass`, `@ConditionalOnMissingBean`)을 가지고 있어, 특정 라이브러리가 존재하거나 사용자가 직접 등록한 빈이 없을 때만 활성화된다.

<br>

### 내장 WAS (Web Application Server)
<br>

스프링 부트는 별도의 외장 서버(Tomcat 등) 설치 없이 애플리케이션을 실행할 수 있도록 서버를 내장하고 있다. `spring-boot-starter-web` 의존성을 추가하면 톰캣이 자동으로 포함된다.

<br>

- **주요 의존성 구조:**
    - `org.springframework.boot:spring-boot-starter-web`
        - `spring-boot-starter-tomcat` (내장 톰캣)
        - `spring-boot-starter-json` (Jackson 라이브러리)
        - `spring-webmvc` (Spring MVC 프레임워크)

<br>
이처럼 내장 WAS를 사용함으로써 JAR 파일 하나만으로 어디서든 독립적으로 실행 가능한 애플리케이션이 완성된다.
