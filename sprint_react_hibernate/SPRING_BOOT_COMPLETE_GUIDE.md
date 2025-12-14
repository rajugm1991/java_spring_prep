# Spring Boot - Complete Guide from Basics to Advanced 🚀

> Comprehensive guide covering Spring Boot fundamentals to advanced topics for staff engineer interview preparation with real-world examples, best practices, and architectural patterns

---

## Table of Contents
1. [Spring Boot Fundamentals](#spring-boot-fundamentals)
2. [Core Concepts](#core-concepts)
3. [Dependency Injection and IoC](#dependency-injection-and-ioc)
4. [Spring Boot Auto-Configuration](#spring-boot-auto-configuration)
5. [Spring Boot Starters](#spring-boot-starters)
6. [Application Properties and Profiles](#application-properties-and-profiles)
7. [RESTful Web Services](#restful-web-services)
8. [Data Access Layer](#data-access-layer)
9. [Security](#security)
10. [Testing](#testing)
11. [Actuator and Monitoring](#actuator-and-monitoring)
12. [Advanced Topics](#advanced-topics)
13. [Microservices Architecture](#microservices-architecture)
14. [Performance Optimization](#performance-optimization)
15. [Best Practices](#best-practices)
16. [Staff Engineer Interview Questions](#staff-engineer-interview-questions)

---

## Spring Boot Fundamentals

### What is Spring Boot?

**Spring Boot** is an opinionated framework built on top of Spring Framework that simplifies the development of production-ready applications.

**Key Features:**
- **Auto-configuration**: Automatically configures Spring based on classpath
- **Standalone**: Embedded servers (Tomcat, Jetty, Undertow)
- **Production-ready**: Actuator, metrics, health checks
- **No XML**: Convention over configuration
- **Starter dependencies**: Pre-configured dependency sets

### Spring Boot vs Spring Framework

| Aspect | Spring Framework | Spring Boot |
|--------|------------------|-------------|
| **Configuration** | Manual (XML/Java) | Auto-configuration |
| **Deployment** | External server | Embedded server |
| **Dependencies** | Manual management | Starter dependencies |
| **Setup Time** | High | Low |
| **Boilerplate** | High | Minimal |
| **Learning Curve** | Steep | Gentle |

### Spring Boot Architecture

```
┌─────────────────────────────────────────────────────────┐
│              SPRING BOOT ARCHITECTURE                    │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│           Application Layer (Your Code)               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────┐  │
│  │ Controllers  │  │  Services   │  │ Repos    │  │
│  └──────────────┘  └──────────────┘  └──────────┘  │
└─────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────┐
│         Spring Boot Auto-Configuration               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────┐  │
│  │   Web MVC    │  │   Data JPA   │  │ Security │  │
│  └──────────────┘  └──────────────┘  └──────────┘  │
└─────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────┐
│            Spring Framework Core                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────┐  │
│  │      IoC     │  │      AOP     │  │   DI    │  │
│  └──────────────┘  └──────────────┘  └──────────┘  │
└─────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────┐
│         Embedded Server (Tomcat/Jetty)               │
└─────────────────────────────────────────────────────┘
```

### Creating Your First Spring Boot Application

#### Method 1: Spring Initializr

```bash
# Using Spring Initializr website
# https://start.spring.io/

# Or using CLI
spring init --dependencies=web,data-jpa,h2 --groupId=com.example --artifactId=myapp
```

#### Method 2: Manual Setup

**1. Maven Project Structure:**

```
my-spring-boot-app/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/myapp/
│   │   │       └── MyApplication.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
│       └── java/
└── pom.xml
```

**2. pom.xml:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/>
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>my-spring-boot-app</artifactId>
    <version>1.0.0</version>
    <name>My Spring Boot App</name>
    
    <properties>
        <java.version>17</java.version>
    </properties>
    
    <dependencies>
        <!-- Spring Boot Web Starter -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- Spring Boot Data JPA -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        
        <!-- H2 Database -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Spring Boot Test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

**3. Main Application Class:**

```java
package com.example.myapp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

**4. Simple REST Controller:**

```java
package com.example.myapp.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    
    @GetMapping("/hello")
    public String hello() {
        return "Hello, Spring Boot!";
    }
}
```

**5. application.properties:**

```properties
# Server Configuration
server.port=8080
server.servlet.context-path=/api

# Application Name
spring.application.name=my-spring-boot-app

# Logging
logging.level.com.example.myapp=INFO
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} - %msg%n
```

**6. Run the Application:**

```bash
# Using Maven
mvn spring-boot:run

# Or build and run JAR
mvn clean package
java -jar target/my-spring-boot-app-1.0.0.jar
```

---

## Core Concepts

### @SpringBootApplication Annotation

The `@SpringBootApplication` annotation is a convenience annotation that combines:

```java
@SpringBootApplication
public class MyApplication {
    // Equivalent to:
    // @Configuration
    // @EnableAutoConfiguration
    // @ComponentScan(basePackages = "com.example.myapp")
}
```

**What it does:**
1. **@Configuration**: Marks class as configuration source
2. **@EnableAutoConfiguration**: Enables Spring Boot auto-configuration
3. **@ComponentScan**: Scans for components in the package

### Component Scanning

**Default Behavior:**
- Scans package of `@SpringBootApplication` class and sub-packages
- Finds `@Component`, `@Service`, `@Repository`, `@Controller`, `@RestController`

**Custom Component Scan:**

```java
@SpringBootApplication
@ComponentScan(basePackages = {
    "com.example.myapp",
    "com.example.common"
})
public class MyApplication {
    // ...
}
```

### Bean Lifecycle

```
┌─────────────────────────────────────────────────────────┐
│              BEAN LIFECYCLE                              │
└─────────────────────────────────────────────────────────┘

1. Instantiation
   └─ Create bean instance
   
2. Populate Properties
   └─ Inject dependencies (@Autowired)
   
3. BeanNameAware.setBeanName()
   └─ Set bean name
   
4. BeanFactoryAware.setBeanFactory()
   └─ Set bean factory
   
5. ApplicationContextAware.setApplicationContext()
   └─ Set application context
   
6. @PostConstruct
   └─ Custom initialization
   
7. InitializingBean.afterPropertiesSet()
   └─ After properties set
   
8. Bean Ready (In Use)
   
9. @PreDestroy
   └─ Before destruction
   
10. DisposableBean.destroy()
    └─ Cleanup
```

**Example:**

```java
@Component
public class MyBean implements BeanNameAware, InitializingBean, DisposableBean {
    
    private String beanName;
    
    @PostConstruct
    public void init() {
        System.out.println("PostConstruct: Initializing bean");
    }
    
    @Override
    public void setBeanName(String name) {
        this.beanName = name;
        System.out.println("BeanNameAware: Bean name = " + name);
    }
    
    @Override
    public void afterPropertiesSet() {
        System.out.println("InitializingBean: After properties set");
    }
    
    @PreDestroy
    public void cleanup() {
        System.out.println("PreDestroy: Cleaning up bean");
    }
    
    @Override
    public void destroy() {
        System.out.println("DisposableBean: Destroying bean");
    }
}
```

---

## Dependency Injection and IoC

### Inversion of Control (IoC)

**Traditional Approach:**
```java
// ❌ BAD: Tight coupling
public class OrderService {
    private PaymentService paymentService = new PaymentService();
    private EmailService emailService = new EmailService();
}
```

**IoC Approach:**
```java
// ✅ GOOD: Loose coupling
@Service
public class OrderService {
    private final PaymentService paymentService;
    private final EmailService emailService;
    
    // Constructor injection (recommended)
    public OrderService(PaymentService paymentService, EmailService emailService) {
        this.paymentService = paymentService;
        this.emailService = emailService;
    }
}
```

### Dependency Injection Types

#### 1. Constructor Injection (Recommended)

```java
@Service
public class OrderService {
    private final PaymentService paymentService;
    private final EmailService emailService;
    
    // Constructor injection - immutable, testable
    public OrderService(PaymentService paymentService, EmailService emailService) {
        this.paymentService = paymentService;
        this.emailService = emailService;
    }
}
```

**Benefits:**
- ✅ Immutable dependencies
- ✅ Required dependencies (fail-fast)
- ✅ Easy to test
- ✅ No circular dependency issues

#### 2. Field Injection

```java
@Service
public class OrderService {
    @Autowired
    private PaymentService paymentService;
    
    @Autowired
    private EmailService emailService;
}
```

**Drawbacks:**
- ❌ Not immutable
- ❌ Hard to test (need reflection)
- ❌ Can hide dependencies

#### 3. Setter Injection

```java
@Service
public class OrderService {
    private PaymentService paymentService;
    private EmailService emailService;
    
    @Autowired
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
    
    @Autowired
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

**Use When:**
- Optional dependencies
- Need to change dependencies at runtime

### @Autowired Annotation

**Constructor Injection (No @Autowired needed in Spring 4.3+):**

```java
@Service
public class OrderService {
    private final PaymentService paymentService;
    
    // @Autowired optional for single constructor
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

**Multiple Constructors:**

```java
@Service
public class OrderService {
    private final PaymentService paymentService;
    
    @Autowired
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
    
    // Another constructor
    public OrderService() {
        this.paymentService = new DefaultPaymentService();
    }
}
```

**Optional Dependencies:**

```java
@Service
public class OrderService {
    @Autowired(required = false)
    private OptionalService optionalService;
    
    // Or use Optional
    private final Optional<OptionalService> optionalService;
    
    public OrderService(Optional<OptionalService> optionalService) {
        this.optionalService = optionalService;
    }
}
```

### @Qualifier Annotation

**When multiple beans of same type:**

```java
// Interface
public interface PaymentService {
    void processPayment();
}

// Implementation 1
@Service
@Qualifier("creditCard")
public class CreditCardPaymentService implements PaymentService {
    // ...
}

// Implementation 2
@Service
@Qualifier("paypal")
public class PayPalPaymentService implements PaymentService {
    // ...
}

// Usage
@Service
public class OrderService {
    private final PaymentService paymentService;
    
    public OrderService(@Qualifier("creditCard") PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

**Using @Primary:**

```java
@Service
@Primary
public class CreditCardPaymentService implements PaymentService {
    // This will be injected by default
}

@Service
public class PayPalPaymentService implements PaymentService {
    // Requires @Qualifier to inject
}
```

### @Value Annotation

**Injecting Properties:**

```java
@Component
public class AppConfig {
    
    // Simple value
    @Value("${app.name}")
    private String appName;
    
    // Default value
    @Value("${app.version:1.0.0}")
    private String appVersion;
    
    // System property
    @Value("${java.home}")
    private String javaHome;
    
    // SpEL (Spring Expression Language)
    @Value("#{systemProperties['user.name']}")
    private String userName;
    
    // SpEL with method call
    @Value("#{T(java.lang.Math).random() * 100}")
    private double randomNumber;
    
    // List
    @Value("${app.servers:localhost:8080,localhost:8081}")
    private List<String> servers;
}
```

**application.properties:**

```properties
app.name=My Application
app.version=2.0.0
app.servers=server1:8080,server2:8080,server3:8080
```

---

## Spring Boot Auto-Configuration

### How Auto-Configuration Works

**Spring Boot Auto-Configuration Process:**

```
1. Application Starts
   │
   ├─> @SpringBootApplication
   │   └─> @EnableAutoConfiguration
   │       └─> AutoConfigurationImportSelector
   │
2. Load spring.factories
   │   └─> META-INF/spring.factories
   │       └─> org.springframework.boot.autoconfigure.EnableAutoConfiguration
   │
3. Filter Auto-Configuration Classes
   │   ├─> @ConditionalOnClass
   │   ├─> @ConditionalOnBean
   │   ├─> @ConditionalOnMissingBean
   │   └─> @ConditionalOnProperty
   │
4. Apply Auto-Configuration
   └─> Create beans automatically
```

### Conditional Annotations

**Common Conditional Annotations:**

```java
// Enable only if class is on classpath
@ConditionalOnClass(DataSource.class)

// Enable only if bean exists
@ConditionalOnBean(DataSource.class)

// Enable only if bean doesn't exist
@ConditionalOnMissingBean(DataSource.class)

// Enable based on property
@ConditionalOnProperty(name = "app.feature.enabled", havingValue = "true")

// Enable based on classpath resource
@ConditionalOnResource(resources = "classpath:config.properties")

// Enable based on web application type
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)

// Enable based on expression
@ConditionalOnExpression("${app.feature.enabled:false}")
```

**Example Auto-Configuration:**

```java
@Configuration
@ConditionalOnClass(DataSource.class)
@ConditionalOnProperty(name = "spring.datasource.url")
public class DataSourceAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public DataSource dataSource(DataSourceProperties properties) {
        return properties.initializeDataSourceBuilder().build();
    }
}
```

### Custom Auto-Configuration

**1. Create Auto-Configuration Class:**

```java
@Configuration
@ConditionalOnClass(MyService.class)
@EnableConfigurationProperties(MyServiceProperties.class)
public class MyServiceAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public MyService myService(MyServiceProperties properties) {
        return new MyService(properties);
    }
}
```

**2. Create Properties Class:**

```java
@ConfigurationProperties(prefix = "my.service")
public class MyServiceProperties {
    private String url = "http://localhost:8080";
    private int timeout = 5000;
    
    // Getters and setters
    public String getUrl() { return url; }
    public void setUrl(String url) { this.url = url; }
    
    public int getTimeout() { return timeout; }
    public void setTimeout(int timeout) { this.timeout = timeout; }
}
```

**3. Register in spring.factories:**

```properties
# META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.autoconfigure.MyServiceAutoConfiguration
```

**4. Use in application.properties:**

```properties
my.service.url=http://api.example.com
my.service.timeout=10000
```

### Debugging Auto-Configuration

**Enable Debug Logging:**

```properties
# application.properties
debug=true

# Or specific
logging.level.org.springframework.boot.autoconfigure=DEBUG
```

**Output shows:**
- Positive matches (applied)
- Negative matches (not applied)
- Exclusions

---

## Spring Boot Starters

### What are Starters?

**Starters** are dependency descriptors that include transitive dependencies needed for a specific feature.

**Benefits:**
- Simplify dependency management
- Version compatibility guaranteed
- Pre-configured dependencies

### Common Starters

#### Web Starter

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

**Includes:**
- Spring MVC
- Embedded Tomcat
- Jackson (JSON)
- Validation

#### Data JPA Starter

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

**Includes:**
- Spring Data JPA
- Hibernate
- Transaction management

#### Security Starter

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

**Includes:**
- Spring Security
- Authentication/Authorization
- CSRF protection

#### Test Starter

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

**Includes:**
- JUnit 5
- Mockito
- AssertJ
- Spring Test

### Custom Starter

**1. Create Starter Module:**

```
my-custom-starter/
├── src/
│   └── main/
│       ├── java/
│       │   └── com/example/starter/
│       │       ├── MyServiceAutoConfiguration.java
│       │       └── MyService.java
│       └── resources/
│           └── META-INF/
│               └── spring.factories
└── pom.xml
```

**2. Auto-Configuration:**

```java
@Configuration
@ConditionalOnClass(MyService.class)
@EnableConfigurationProperties(MyServiceProperties.class)
public class MyServiceAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public MyService myService(MyServiceProperties properties) {
        return new MyService(properties);
    }
}
```

**3. Register:**

```properties
# META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.starter.MyServiceAutoConfiguration
```

**4. Use in Another Project:**

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>my-custom-starter</artifactId>
    <version>1.0.0</version>
</dependency>
```

---

## Application Properties and Profiles

### application.properties vs application.yml

**application.properties:**

```properties
server.port=8080
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update
```

**application.yml:**

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: password
  jpa:
    hibernate:
      ddl-auto: update
```

### Property Sources Priority

```
1. Command line arguments
2. SPRING_APPLICATION_JSON (environment variable)
3. ServletConfig init parameters
4. ServletContext init parameters
5. java:comp/env JNDI attributes
6. System properties
7. OS environment variables
8. RandomValuePropertySource
9. application-{profile}.properties/yml
10. application.properties/yml
11. @PropertySource
12. Default properties
```

### Profiles

**Defining Profiles:**

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(MyApplication.class);
        app.setAdditionalProfiles("dev");
        app.run(args);
    }
}
```

**Using Profiles:**

```properties
# application.properties
spring.profiles.active=dev

# application-dev.properties
server.port=8080
spring.datasource.url=jdbc:h2:mem:devdb

# application-prod.properties
server.port=8443
spring.datasource.url=jdbc:mysql://prod-server:3306/proddb
```

**Profile-Specific Beans:**

```java
@Configuration
public class DatabaseConfig {
    
    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        return new HikariDataSource(); // H2 for dev
    }
    
    @Bean
    @Profile("prod")
    public DataSource prodDataSource() {
        return new HikariDataSource(); // MySQL for prod
    }
}
```

**Activating Profiles:**

```bash
# Command line
java -jar app.jar --spring.profiles.active=prod

# Environment variable
export SPRING_PROFILES_ACTIVE=prod

# application.properties
spring.profiles.active=prod,logging
```

### @ConfigurationProperties

**Binding Properties to POJO:**

```java
@ConfigurationProperties(prefix = "app")
@Component
public class AppProperties {
    private String name;
    private String version;
    private Server server = new Server();
    private List<String> features = new ArrayList<>();
    
    // Getters and setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public Server getServer() { return server; }
    public void setServer(Server server) { this.server = server; }
    
    public static class Server {
        private String host;
        private int port;
        
        // Getters and setters
        public String getHost() { return host; }
        public void setHost(String host) { this.host = host; }
        
        public int getPort() { return port; }
        public void setPort(int port) { this.port = port; }
    }
}
```

**application.properties:**

```properties
app.name=My Application
app.version=1.0.0
app.server.host=localhost
app.server.port=8080
app.features[0]=feature1
app.features[1]=feature2
```

**Usage:**

```java
@Service
public class MyService {
    private final AppProperties appProperties;
    
    public MyService(AppProperties appProperties) {
        this.appProperties = appProperties;
    }
    
    public void doSomething() {
        System.out.println("App: " + appProperties.getName());
        System.out.println("Server: " + appProperties.getServer().getHost());
    }
}
```

### Property Validation

```java
@ConfigurationProperties(prefix = "app")
@Validated
public class AppProperties {
    
    @NotBlank
    private String name;
    
    @Min(1)
    @Max(65535)
    private int port;
    
    @Email
    private String email;
    
    @NotNull
    @Valid
    private Server server;
    
    // Getters and setters
}
```

---

## RESTful Web Services

### REST Controller

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers() {
        List<User> users = userService.findAll();
        return ResponseEntity.ok(users);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        return userService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody @Valid User user) {
        User created = userService.save(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(@PathVariable Long id, @RequestBody @Valid User user) {
        return userService.update(id, user)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        if (userService.delete(id)) {
            return ResponseEntity.noContent().build();
        }
        return ResponseEntity.notFound().build();
    }
}
```

### Request Mapping

**HTTP Methods:**

```java
@GetMapping("/users")      // GET /users
@PostMapping("/users")     // POST /users
@PutMapping("/users/{id}") // PUT /users/{id}
@DeleteMapping("/users/{id}") // DELETE /users/{id}
@PatchMapping("/users/{id}")  // PATCH /users/{id}
```

**Path Variables:**

```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    // GET /users/123 -> id = 123
}

@GetMapping("/users/{userId}/orders/{orderId}")
public Order getOrder(
    @PathVariable Long userId,
    @PathVariable Long orderId
) {
    // GET /users/123/orders/456
}
```

**Request Parameters:**

```java
@GetMapping("/users")
public List<User> getUsers(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size,
    @RequestParam(required = false) String name
) {
    // GET /users?page=0&size=10&name=John
}
```

**Request Body:**

```java
@PostMapping("/users")
public User createUser(@RequestBody @Valid User user) {
    // POST /users with JSON body
    return userService.save(user);
}
```

### Response Entity

```java
@GetMapping("/users/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    return userService.findById(id)
        .map(user -> ResponseEntity.ok()
            .header("Custom-Header", "value")
            .body(user))
        .orElse(ResponseEntity.notFound().build());
}

@PostMapping("/users")
public ResponseEntity<User> createUser(@RequestBody User user) {
    User created = userService.save(user);
    URI location = ServletUriComponentsBuilder
        .fromCurrentRequest()
        .path("/{id}")
        .buildAndExpand(created.getId())
        .toUri();
    
    return ResponseEntity.created(location).body(created);
}
```

### Exception Handling

**Global Exception Handler:**

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            Instant.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
        );
        
        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Validation failed",
            Instant.now(),
            errors
        );
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An unexpected error occurred",
            Instant.now()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

### Content Negotiation

```java
@GetMapping(value = "/users/{id}", produces = {
    MediaType.APPLICATION_JSON_VALUE,
    MediaType.APPLICATION_XML_VALUE
})
public ResponseEntity<User> getUser(@PathVariable Long id) {
    // Returns JSON or XML based on Accept header
    return ResponseEntity.ok(userService.findById(id).orElseThrow());
}
```

---

## Data Access Layer

### Spring Data JPA

**Entity:**

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String email;
    
    @Column(nullable = false)
    private String name;
    
    @CreatedDate
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    private LocalDateTime updatedAt;
    
    // Getters and setters
}
```

**Repository:**

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Query methods
    Optional<User> findByEmail(String email);
    List<User> findByNameContaining(String name);
    List<User> findByEmailAndName(String email, String name);
    
    // Custom query
    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findByEmailCustom(@Param("email") String email);
    
    // Native query
    @Query(value = "SELECT * FROM users WHERE email = ?1", nativeQuery = true)
    Optional<User> findByEmailNative(String email);
    
    // Pagination
    Page<User> findByName(String name, Pageable pageable);
    
    // Sorting
    List<User> findByOrderByNameAsc();
}
```

**Service:**

```java
@Service
@Transactional
public class UserService {
    
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public User createUser(User user) {
        if (userRepository.findByEmail(user.getEmail()).isPresent()) {
            throw new DuplicateEmailException("Email already exists");
        }
        return userRepository.save(user);
    }
    
    public Optional<User> getUser(Long id) {
        return userRepository.findById(id);
    }
    
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }
    
    public User updateUser(Long id, User user) {
        return userRepository.findById(id)
            .map(existing -> {
                existing.setName(user.getName());
                existing.setEmail(user.getEmail());
                return userRepository.save(existing);
            })
            .orElseThrow(() -> new ResourceNotFoundException("User not found: " + id));
    }
    
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```

### Transaction Management

**Declarative Transactions:**

```java
@Service
@Transactional
public class OrderService {
    
    @Transactional(readOnly = true)
    public Order getOrder(Long id) {
        // Read-only transaction
        return orderRepository.findById(id).orElseThrow();
    }
    
    @Transactional(rollbackFor = Exception.class)
    public Order createOrder(Order order) {
        // Transaction with rollback on any exception
        return orderRepository.save(order);
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logOrder(Order order) {
        // New transaction (independent)
        orderLogRepository.save(new OrderLog(order));
    }
}
```

**Programmatic Transactions:**

```java
@Service
public class OrderService {
    
    private final TransactionTemplate transactionTemplate;
    
    public OrderService(PlatformTransactionManager transactionManager) {
        this.transactionTemplate = new TransactionTemplate(transactionManager);
    }
    
    public Order createOrder(Order order) {
        return transactionTemplate.execute(status -> {
            // Transactional code
            Order saved = orderRepository.save(order);
            inventoryService.reserve(saved.getItems());
            return saved;
        });
    }
}
```

---

## Security

### Spring Security Basics

**Dependency:**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

**Basic Configuration:**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults())
            .csrf(csrf -> csrf.disable()); // For stateless APIs
        
        return http.build();
    }
    
    @Bean
    public UserDetailsService userDetailsService() {
        UserDetails user = User.withDefaultPasswordEncoder()
            .username("user")
            .password("password")
            .roles("USER")
            .build();
        
        UserDetails admin = User.withDefaultPasswordEncoder()
            .username("admin")
            .password("admin")
            .roles("ADMIN")
            .build();
        
        return new InMemoryUserDetailsManager(user, admin);
    }
}
```

### JWT Authentication

**JWT Service:**

```java
@Service
public class JwtService {
    
    @Value("${jwt.secret}")
    private String secret;
    
    @Value("${jwt.expiration}")
    private Long expiration;
    
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        return createToken(claims, userDetails.getUsername());
    }
    
    private String createToken(Map<String, Object> claims, String subject) {
        return Jwts.builder()
            .setClaims(claims)
            .setSubject(subject)
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + expiration))
            .signWith(SignatureAlgorithm.HS256, secret)
            .compact();
    }
    
    public Boolean validateToken(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));
    }
    
    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }
    
    private Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }
    
    private <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }
    
    private Claims extractAllClaims(String token) {
        return Jwts.parser().setSigningKey(secret).parseClaimsJws(token).getBody();
    }
    
    private Boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }
}
```

---

## Testing

### Unit Testing

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void shouldCreateUser() {
        // Given
        User user = new User("john@example.com", "John Doe");
        User savedUser = new User(1L, "john@example.com", "John Doe");
        
        when(userRepository.findByEmail("john@example.com")).thenReturn(Optional.empty());
        when(userRepository.save(user)).thenReturn(savedUser);
        
        // When
        User result = userService.createUser(user);
        
        // Then
        assertThat(result.getId()).isNotNull();
        assertThat(result.getEmail()).isEqualTo("john@example.com");
        verify(userRepository).save(user);
    }
}
```

### Integration Testing

```java
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerIntegrationTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void shouldCreateUser() throws Exception {
        // Given
        String userJson = """
            {
                "email": "john@example.com",
                "name": "John Doe"
            }
            """;
        
        // When & Then
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(userJson))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.email").value("john@example.com"));
    }
}
```

---

## Actuator and Monitoring

### Spring Boot Actuator

**Dependency:**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

**Configuration:**

```properties
# Enable all endpoints
management.endpoints.web.exposure.include=*

# Or specific endpoints
management.endpoints.web.exposure.include=health,info,metrics

# Health endpoint details
management.endpoint.health.show-details=always

# Custom health indicator
management.health.custom.enabled=true
```

**Endpoints:**

- `/actuator/health` - Application health
- `/actuator/info` - Application information
- `/actuator/metrics` - Application metrics
- `/actuator/env` - Environment variables
- `/actuator/loggers` - Logging configuration

---

## Advanced Topics

### Custom Starter Development

**Creating a Production-Ready Starter:**

```java
// 1. Auto-Configuration Class
@Configuration
@ConditionalOnClass(MyService.class)
@EnableConfigurationProperties(MyServiceProperties.class)
@AutoConfigureAfter(DataSourceAutoConfiguration.class)
public class MyServiceAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public MyService myService(MyServiceProperties properties) {
        return new MyService(properties);
    }
    
    @Bean
    @ConditionalOnProperty(prefix = "my.service", name = "metrics.enabled", havingValue = "true")
    public MyServiceMetrics myServiceMetrics(MyService myService) {
        return new MyServiceMetrics(myService);
    }
}

// 2. Properties Class
@ConfigurationProperties(prefix = "my.service")
@Validated
public class MyServiceProperties {
    @NotBlank
    private String url = "http://localhost:8080";
    
    @Min(1000)
    @Max(60000)
    private int timeout = 5000;
    
    private Metrics metrics = new Metrics();
    
    // Getters and setters
    
    public static class Metrics {
        private boolean enabled = false;
        private String prefix = "myservice";
        
        // Getters and setters
    }
}

// 3. Register in spring.factories
# META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.starter.MyServiceAutoConfiguration
```

### Actuator Customization

**Custom Health Indicator:**

```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {
    
    private final DataSource dataSource;
    
    public DatabaseHealthIndicator(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    
    @Override
    public Health health() {
        try (Connection connection = dataSource.getConnection()) {
            if (connection.isValid(1)) {
                return Health.up()
                    .withDetail("database", "Available")
                    .withDetail("validationQuery", "isValid()")
                    .build();
            }
        } catch (SQLException e) {
            return Health.down()
                .withDetail("database", "Unavailable")
                .withDetail("error", e.getMessage())
                .build();
        }
        return Health.down().build();
    }
}
```

**Custom Metrics:**

```java
@Component
public class CustomMetrics {
    
    private final MeterRegistry meterRegistry;
    private final Counter requestCounter;
    private final Timer requestTimer;
    
    public CustomMetrics(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        this.requestCounter = Counter.builder("custom.requests")
            .description("Total number of requests")
            .register(meterRegistry);
        this.requestTimer = Timer.builder("custom.request.duration")
            .description("Request duration")
            .register(meterRegistry);
    }
    
    public void recordRequest() {
        requestCounter.increment();
    }
    
    public void recordDuration(Duration duration) {
        requestTimer.record(duration);
    }
}
```

### Caching

**Spring Cache Abstraction:**

```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        RedisCacheManager.Builder builder = RedisCacheManager
            .RedisCacheManagerBuilder
            .fromConnectionFactory(redisConnectionFactory())
            .cacheDefaults(cacheConfiguration());
        
        return builder.build();
    }
    
    private RedisCacheConfiguration cacheConfiguration() {
        return RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(5))
            .disableCachingNullValues()
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new GenericJackson2JsonRedisSerializer()));
    }
}

// Usage
@Service
public class ProductService {
    
    @Cacheable(value = "products", key = "#id")
    public Product getProduct(Long id) {
        return productRepository.findById(id).orElseThrow();
    }
    
    @CacheEvict(value = "products", key = "#product.id")
    public Product updateProduct(Product product) {
        return productRepository.save(product);
    }
    
    @CacheEvict(value = "products", allEntries = true)
    public void clearCache() {
        // Clear all cache entries
    }
}
```

### Async Processing

**Async Configuration:**

```java
@Configuration
@EnableAsync
public class AsyncConfig {
    
    @Bean(name = "taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

// Usage
@Service
public class OrderService {
    
    @Async("taskExecutor")
    public CompletableFuture<Order> processOrderAsync(Order order) {
        // Async processing
        Order processed = processOrder(order);
        return CompletableFuture.completedFuture(processed);
    }
    
    @Async
    public void sendEmailAsync(String email, String message) {
        emailService.send(email, message);
    }
}
```

### WebFlux (Reactive Programming)

**Reactive Controller:**

```java
@RestController
@RequestMapping("/api/reactive")
public class ReactiveController {
    
    private final ReactiveUserService userService;
    
    public ReactiveController(ReactiveUserService userService) {
        this.userService = userService;
    }
    
    @GetMapping("/users")
    public Flux<User> getAllUsers() {
        return userService.findAll();
    }
    
    @GetMapping("/users/{id}")
    public Mono<User> getUser(@PathVariable Long id) {
        return userService.findById(id)
            .switchIfEmpty(Mono.error(new ResourceNotFoundException()));
    }
    
    @PostMapping("/users")
    public Mono<ResponseEntity<User>> createUser(@RequestBody User user) {
        return userService.save(user)
            .map(saved -> ResponseEntity.status(HttpStatus.CREATED).body(saved));
    }
}
```

**Reactive Service:**

```java
@Service
public class ReactiveUserService {
    
    private final ReactiveUserRepository userRepository;
    
    public ReactiveUserService(ReactiveUserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public Flux<User> findAll() {
        return userRepository.findAll();
    }
    
    public Mono<User> findById(Long id) {
        return userRepository.findById(id);
    }
    
    public Mono<User> save(User user) {
        return userRepository.save(user);
    }
    
    public Mono<Void> delete(Long id) {
        return userRepository.deleteById(id);
    }
}
```

---

## Microservices Architecture

### Service Discovery with Eureka

**Eureka Server:**

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

**Eureka Client:**

```java
@SpringBootApplication
@EnableEurekaClient
public class ProductServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProductServiceApplication.class, args);
    }
}
```

**application.yml:**

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
  instance:
    prefer-ip-address: true
```

### API Gateway with Spring Cloud Gateway

**Gateway Configuration:**

```java
@Configuration
public class GatewayConfig {
    
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("product-service", r -> r
                .path("/api/products/**")
                .uri("lb://product-service"))
            .route("order-service", r -> r
                .path("/api/orders/**")
                .uri("lb://order-service"))
            .build();
    }
    
    @Bean
    public GlobalFilter customGlobalFilter() {
        return (exchange, chain) -> {
            ServerHttpRequest request = exchange.getRequest();
            // Add custom headers
            ServerHttpRequest modifiedRequest = request.mutate()
                .header("X-Gateway-Request", "true")
                .build();
            return chain.filter(exchange.mutate().request(modifiedRequest).build());
        };
    }
}
```

### Distributed Tracing with Sleuth

**Dependency:**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

**Automatic Trace IDs:**

```java
@Service
public class OrderService {
    
    private static final Logger logger = LoggerFactory.getLogger(OrderService.class);
    
    public Order createOrder(Order order) {
        // Trace ID automatically added to logs
        logger.info("Creating order: {}", order.getId());
        return orderRepository.save(order);
    }
}
```

### Circuit Breaker with Resilience4j

**Configuration:**

```java
@Configuration
public class ResilienceConfig {
    
    @Bean
    public CircuitBreakerRegistry circuitBreakerRegistry() {
        return CircuitBreakerRegistry.ofDefaults();
    }
    
    @Bean
    public CircuitBreaker productServiceCircuitBreaker() {
        CircuitBreakerConfig config = CircuitBreakerConfig.custom()
            .failureRateThreshold(50)
            .waitDurationInOpenState(Duration.ofMillis(1000))
            .slidingWindowSize(10)
            .build();
        
        return CircuitBreakerRegistry.of(config).circuitBreaker("productService");
    }
}

// Usage
@Service
public class ProductService {
    
    private final CircuitBreaker circuitBreaker;
    private final RestTemplate restTemplate;
    
    public ProductService(CircuitBreaker circuitBreaker, RestTemplate restTemplate) {
        this.circuitBreaker = circuitBreaker;
        this.restTemplate = restTemplate;
    }
    
    public Product getProduct(Long id) {
        return circuitBreaker.executeSupplier(() -> {
            return restTemplate.getForObject("/products/" + id, Product.class);
        });
    }
}
```

---

## Performance Optimization

### Connection Pooling

**HikariCP Configuration:**

```properties
# HikariCP (default in Spring Boot 2.x+)
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.idle-timeout=600000
spring.datasource.hikari.max-lifetime=1800000
spring.datasource.hikari.leak-detection-threshold=60000
```

**Custom DataSource:**

```java
@Configuration
public class DataSourceConfig {
    
    @Bean
    @ConfigurationProperties("spring.datasource.hikari")
    public HikariDataSource dataSource() {
        return DataSourceBuilder.create()
            .type(HikariDataSource.class)
            .build();
    }
}
```

### Caching Strategies

**Multi-Level Caching:**

```java
@Service
public class ProductService {
    
    @Cacheable(value = "products", key = "#id")
    public Product getProduct(Long id) {
        // L1: Local cache (Caffeine)
        return productRepository.findById(id).orElseThrow();
    }
    
    @Cacheable(value = "product-list", key = "#page + '_' + #size")
    public List<Product> getProducts(int page, int size) {
        // L2: Distributed cache (Redis)
        return productRepository.findAll(PageRequest.of(page, size));
    }
}
```

### Database Query Optimization

**Batch Operations:**

```java
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    
    @Modifying
    @Query("UPDATE Product p SET p.price = :price WHERE p.id IN :ids")
    void updatePrices(@Param("ids") List<Long> ids, @Param("price") BigDecimal price);
    
    @Modifying
    @Query(value = "INSERT INTO products (name, price) VALUES (:name, :price)", nativeQuery = true)
    void bulkInsert(@Param("name") String name, @Param("price") BigDecimal price);
}
```

**Query Optimization:**

```java
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    
    // Use projections to fetch only needed fields
    @Query("SELECT p.id, p.name, p.price FROM Product p WHERE p.category = :category")
    List<ProductSummary> findByCategory(@Param("category") String category);
    
    // Use entity graph to avoid N+1 problem
    @EntityGraph(attributePaths = {"category", "reviews"})
    @Query("SELECT p FROM Product p WHERE p.id = :id")
    Optional<Product> findByIdWithDetails(@Param("id") Long id);
}
```

---

## Best Practices

### Project Structure

```
src/
├── main/
│   ├── java/
│   │   └── com/example/app/
│   │       ├── Application.java
│   │       ├── config/
│   │       │   ├── SecurityConfig.java
│   │       │   ├── CacheConfig.java
│   │       │   └── WebConfig.java
│   │       ├── controller/
│   │       │   ├── UserController.java
│   │       │   └── ProductController.java
│   │       ├── service/
│   │       │   ├── UserService.java
│   │       │   └── ProductService.java
│   │       ├── repository/
│   │       │   ├── UserRepository.java
│   │       │   └── ProductRepository.java
│   │       ├── model/
│   │       │   ├── User.java
│   │       │   └── Product.java
│   │       ├── dto/
│   │       │   ├── UserDTO.java
│   │       │   └── ProductDTO.java
│   │       ├── exception/
│   │       │   ├── GlobalExceptionHandler.java
│   │       │   └── ResourceNotFoundException.java
│   │       └── util/
│   │           └── Mapper.java
│   └── resources/
│       ├── application.properties
│       ├── application-dev.properties
│       ├── application-prod.properties
│       └── db/
│           └── migration/
│               └── V1__Initial_schema.sql
└── test/
    └── java/
        └── com/example/app/
            ├── controller/
            ├── service/
            └── repository/
```

### Naming Conventions

**✅ GOOD:**
- Controllers: `UserController`, `ProductController`
- Services: `UserService`, `ProductService`
- Repositories: `UserRepository`, `ProductRepository`
- DTOs: `UserDTO`, `ProductDTO`
- Exceptions: `UserNotFoundException`, `ValidationException`

**❌ BAD:**
- Controllers: `UserCtrl`, `ProductCtrl`
- Services: `UserSvc`, `ProductSvc`
- Repositories: `UserRepo`, `ProductRepo`

### Error Handling Best Practices

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        logger.warn("Resource not found: {}", ex.getMessage());
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            Instant.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(ValidationException ex) {
        logger.warn("Validation failed: {}", ex.getMessage());
        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            ex.getMessage(),
            Instant.now(),
            ex.getErrors()
        );
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        logger.error("Unexpected error", ex);
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An unexpected error occurred",
            Instant.now()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

### Logging Best Practices

```java
@Service
public class OrderService {
    
    private static final Logger logger = LoggerFactory.getLogger(OrderService.class);
    
    public Order createOrder(Order order) {
        logger.info("Creating order: userId={}, items={}", 
            order.getUserId(), order.getItems().size());
        
        try {
            Order created = orderRepository.save(order);
            logger.info("Order created successfully: orderId={}", created.getId());
            return created;
        } catch (Exception e) {
            logger.error("Failed to create order: userId={}", order.getUserId(), e);
            throw new OrderCreationException("Failed to create order", e);
        }
    }
}
```

**Logging Configuration:**

```properties
# Logging levels
logging.level.com.example.app=INFO
logging.level.com.example.app.service=DEBUG
logging.level.org.springframework.web=DEBUG

# Logging pattern
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n
logging.pattern.file=%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n

# File logging
logging.file.name=logs/application.log
logging.file.max-size=10MB
logging.file.max-history=30
```

---

## Staff Engineer Interview Questions

### Q1: Explain Spring Boot Auto-Configuration in Detail

**Answer:**
"Spring Boot auto-configuration automatically configures Spring beans based on:
1. **Classpath dependencies**: Detects libraries on classpath (e.g., DataSource, JPA)
2. **Existing bean definitions**: Checks if beans already exist
3. **Property configurations**: Reads from application.properties

The process:
- Spring Boot scans `META-INF/spring.factories` for auto-configuration classes
- Uses conditional annotations (@ConditionalOnClass, @ConditionalOnBean, @ConditionalOnMissingBean) to determine applicability
- Only applies configurations when conditions are met
- Allows customization through properties or custom bean definitions

Example: If H2 is on classpath and no DataSource bean exists, Spring Boot auto-configures an in-memory H2 DataSource."

### Q2: How does Spring Boot's Embedded Server Work?

**Answer:**
"Spring Boot includes embedded servers (Tomcat, Jetty, Undertow) that start automatically:
1. **Embedded Tomcat**: Default servlet container, starts on port 8080
2. **Configuration**: Controlled via `server.port`, `server.servlet.context-path`
3. **Customization**: Can replace with Jetty/Undertow or configure Tomcat programmatically
4. **Benefits**: No external server needed, easier deployment, faster startup

The embedded server is configured through `ServletWebServerFactory` beans and can be customized via `WebServerFactoryCustomizer`."

### Q3: Explain Spring Boot's Starter Dependencies

**Answer:**
"Starters are dependency descriptors that:
1. **Bundle dependencies**: Include transitive dependencies needed for a feature
2. **Version management**: Spring Boot manages compatible versions
3. **Auto-configuration**: Trigger relevant auto-configurations
4. **Convention over configuration**: Provide sensible defaults

Example: `spring-boot-starter-web` includes:
- Spring MVC
- Embedded Tomcat
- Jackson (JSON)
- Validation
- All compatible versions

This eliminates dependency conflicts and version management overhead."

### Q4: How Does Spring Boot Handle Application Properties?

**Answer:**
"Spring Boot loads properties in this order (highest to lowest priority):
1. Command line arguments
2. SPRING_APPLICATION_JSON environment variable
3. System properties
4. OS environment variables
5. application-{profile}.properties/yml
6. application.properties/yml
7. @PropertySource annotations
8. Default properties

Properties can be:
- Type-safe binding via @ConfigurationProperties
- Validated using Bean Validation
- Profile-specific via application-{profile}.properties
- Externalized for different environments"

### Q5: Explain Spring Boot's Component Scanning

**Answer:**
"Component scanning:
1. **Default behavior**: Scans package of @SpringBootApplication class and sub-packages
2. **Annotations detected**: @Component, @Service, @Repository, @Controller, @RestController
3. **Customization**: Can specify base packages via @ComponentScan
4. **Performance**: Uses classpath scanning, can be optimized with @Indexed

Best practice: Keep @SpringBootApplication at root package to scan entire application."

### Q6: How Does Spring Boot Handle Database Migrations?

**Answer:**
"Spring Boot supports Flyway and Liquibase:
1. **Flyway**: SQL-based migrations, versioned files (V1__Initial.sql)
2. **Liquibase**: XML/YAML/SQL migrations, more flexible
3. **Auto-execution**: Runs on startup if enabled
4. **Version control**: Tracks applied migrations
5. **Rollback**: Supports rollback strategies

Configuration:
```properties
spring.flyway.enabled=true
spring.flyway.locations=classpath:db/migration
spring.flyway.baseline-on-migrate=true
```"

### Q7: Explain Spring Boot Actuator Endpoints

**Answer:**
"Actuator provides production-ready features:
1. **Health**: Application health status (/actuator/health)
2. **Metrics**: Application metrics (/actuator/metrics)
3. **Info**: Application information (/actuator/info)
4. **Environment**: Environment variables (/actuator/env)
5. **Loggers**: Logging configuration (/actuator/loggers)
6. **Beans**: Spring bean definitions (/actuator/beans)

Security: Endpoints should be secured in production, exposed selectively via `management.endpoints.web.exposure.include`."

### Q8: How Does Spring Boot Handle Transactions?

**Answer:**
"Spring Boot auto-configures transaction management:
1. **PlatformTransactionManager**: Auto-configured based on DataSource
2. **@Transactional**: Declarative transaction management
3. **Propagation**: Controls transaction scope (REQUIRED, REQUIRES_NEW, etc.)
4. **Isolation**: Controls transaction isolation level
5. **Rollback**: Configurable rollback rules

Best practices:
- Use @Transactional at service layer
- Keep transactions short
- Use read-only transactions for queries
- Handle exceptions properly"

### Q9: Explain Spring Boot's Testing Support

**Answer:**
"Spring Boot provides comprehensive testing support:
1. **@SpringBootTest**: Integration tests with full application context
2. **@WebMvcTest**: Web layer tests with mocked dependencies
3. **@DataJpaTest**: Repository tests with in-memory database
4. **@MockBean**: Mock Spring beans
5. **TestContainers**: Integration with Docker containers
6. **Test slices**: Test specific layers in isolation

Example:
```java
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerTest {
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    void shouldCreateUser() throws Exception {
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(userJson))
            .andExpect(status().isCreated());
    }
}
```"

### Q10: How Does Spring Boot Handle Caching?

**Answer:**
"Spring Boot supports multiple cache providers:
1. **Simple**: In-memory cache (default)
2. **Caffeine**: High-performance Java cache
3. **Redis**: Distributed cache
4. **EhCache**: Mature Java cache
5. **Hazelcast**: Distributed in-memory cache

Configuration:
```java
@EnableCaching
@Configuration
public class CacheConfig {
    @Bean
    public CacheManager cacheManager() {
        return new CaffeineCacheManager("products", "users");
    }
}
```

Usage:
```java
@Cacheable("products")
public Product getProduct(Long id) { }

@CacheEvict("products")
public void updateProduct(Product product) { }
```"

### Q11: Explain Spring Boot's Profile System

**Answer:**
"Profiles allow environment-specific configurations:
1. **Profile-specific files**: application-{profile}.properties
2. **@Profile annotation**: Conditionally create beans
3. **Active profiles**: Set via spring.profiles.active
4. **Default profile**: 'default' profile if none specified
5. **Multiple profiles**: Can activate multiple profiles

Use cases:
- Development: application-dev.properties
- Testing: application-test.properties
- Production: application-prod.properties

Best practice: Externalize sensitive properties, use profiles for environment differences."

### Q12: How Does Spring Boot Handle Security?

**Answer:**
"Spring Boot integrates Spring Security:
1. **Auto-configuration**: Default security configuration
2. **Customization**: Override SecurityFilterChain
3. **OAuth2**: Support for OAuth2/OIDC
4. **JWT**: Token-based authentication
5. **Method security**: @PreAuthorize, @Secured

Configuration:
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) {
        http.authorizeHttpRequests(auth -> auth
            .requestMatchers("/public/**").permitAll()
            .anyRequest().authenticated()
        );
        return http.build();
    }
}
```"

### Q13: Explain Spring Boot's Async Support

**Answer:**
"Spring Boot supports asynchronous processing:
1. **@EnableAsync**: Enables async method execution
2. **@Async**: Marks methods for async execution
3. **CompletableFuture**: Returns CompletableFuture for async results
4. **TaskExecutor**: Configurable thread pool
5. **Exception handling**: @AsyncUncaughtExceptionHandler

Configuration:
```java
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        return executor;
    }
}
```"

### Q14: How Does Spring Boot Handle Database Connections?

**Answer:**
"Spring Boot auto-configures database connections:
1. **DataSource**: Auto-configured based on classpath
2. **Connection pooling**: HikariCP (default), can use others
3. **JPA/Hibernate**: Auto-configured with sensible defaults
4. **Transaction management**: Auto-configured PlatformTransactionManager
5. **Multiple datasources**: Supported with custom configuration

Configuration:
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.hikari.maximum-pool-size=20
spring.jpa.hibernate.ddl-auto=update
```"

### Q15: Explain Spring Boot's Externalized Configuration

**Answer:**
"Spring Boot supports externalized configuration:
1. **Properties files**: application.properties, application.yml
2. **Environment variables**: SPRING_APPLICATION_JSON
3. **Command line**: --spring.profiles.active=prod
4. **@PropertySource**: Custom property sources
5. **@ConfigurationProperties**: Type-safe property binding

Priority (highest to lowest):
1. Command line arguments
2. Environment variables
3. application-{profile}.properties
4. application.properties

Best practice: Use profiles for environment-specific configs, externalize sensitive data."

### Q16: How Does Spring Boot Handle REST APIs?

**Answer:**
"Spring Boot provides REST support via Spring MVC:
1. **@RestController**: Combines @Controller + @ResponseBody
2. **@RequestMapping**: URL mapping
3. **HTTP methods**: @GetMapping, @PostMapping, etc.
4. **Content negotiation**: JSON/XML based on Accept header
5. **Exception handling**: @RestControllerAdvice for global handling
6. **Validation**: @Valid with Bean Validation

Example:
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(userService.findById(id));
    }
}
```"

### Q17: Explain Spring Boot's Health Checks

**Answer:**
"Spring Boot Actuator provides health checks:
1. **Default health**: Basic application health
2. **Custom health indicators**: Implement HealthIndicator
3. **Health groups**: Group multiple checks
4. **Database health**: Auto-configured if DataSource present
5. **Disk space health**: Checks disk space

Configuration:
```properties
management.endpoint.health.show-details=always
management.health.diskspace.enabled=true
```

Custom health:
```java
@Component
public class CustomHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        // Check custom condition
        return Health.up().withDetail("custom", "OK").build();
    }
}
```"

### Q18: How Does Spring Boot Handle Microservices?

**Answer:**
"Spring Boot supports microservices via Spring Cloud:
1. **Service discovery**: Eureka, Consul, Zookeeper
2. **API Gateway**: Spring Cloud Gateway
3. **Load balancing**: Ribbon, Spring Cloud LoadBalancer
4. **Circuit breaker**: Resilience4j, Hystrix
5. **Distributed tracing**: Sleuth, Zipkin
6. **Configuration server**: Spring Cloud Config

Architecture:
- Each service is independent Spring Boot application
- Services register with discovery server
- API Gateway routes requests to services
- Circuit breakers prevent cascading failures"

### Q19: Explain Spring Boot's Performance Optimization

**Answer:**
"Spring Boot performance optimization:
1. **Connection pooling**: HikariCP with optimal pool size
2. **Caching**: Multi-level caching (local + distributed)
3. **Async processing**: Non-blocking operations
4. **Lazy initialization**: @Lazy for expensive beans
5. **Component scanning**: Limit scan scope
6. **JVM tuning**: Heap size, GC settings
7. **Database optimization**: Query optimization, indexing
8. **Compression**: Gzip compression for responses

Best practices:
- Use connection pooling
- Implement caching strategies
- Use async for I/O operations
- Optimize database queries
- Monitor with Actuator metrics"

### Q20: How Does Spring Boot Handle Production Deployment?

**Answer:**
"Spring Boot production deployment:
1. **JAR packaging**: Executable JAR with embedded server
2. **WAR packaging**: Deploy to external servlet container
3. **Docker**: Containerize application
4. **Kubernetes**: Deploy to K8s clusters
5. **Cloud platforms**: AWS, Azure, GCP support
6. **Monitoring**: Actuator endpoints for health/metrics
7. **Logging**: Structured logging (JSON format)
8. **Configuration**: Externalize via Config Server

Best practices:
- Use profiles for environment configs
- Enable Actuator endpoints (secured)
- Implement health checks
- Use structured logging
- Monitor metrics and alerts
- Implement graceful shutdown"

---

## Summary

### Key Takeaways

1. **Auto-Configuration**: Spring Boot automatically configures based on classpath
2. **Starters**: Simplify dependency management
3. **Embedded Server**: No external server needed
4. **Profiles**: Environment-specific configurations
5. **Actuator**: Production-ready monitoring
6. **Testing**: Comprehensive testing support
7. **Microservices**: Full support via Spring Cloud
8. **Performance**: Built-in optimization features

### Best Practices Checklist

- ✅ Use constructor injection
- ✅ Follow naming conventions
- ✅ Implement proper error handling
- ✅ Use profiles for environments
- ✅ Secure Actuator endpoints
- ✅ Implement health checks
- ✅ Use connection pooling
- ✅ Implement caching strategies
- ✅ Write comprehensive tests
- ✅ Monitor with Actuator

### Common Pitfalls

- ❌ Not using profiles for environment configs
- ❌ Exposing Actuator endpoints without security
- ❌ Not implementing proper error handling
- ❌ Using field injection instead of constructor injection
- ❌ Not optimizing database queries
- ❌ Ignoring connection pool configuration
- ❌ Not implementing health checks
- ❌ Not using async for I/O operations

---

**Happy Coding with Spring Boot! 🚀**


