# Service Discovery and API Gateway - Complete Guide

## Table of Contents
1. [Service Discovery Overview](#service-discovery-overview)
2. [Eureka Service Discovery Implementation](#eureka-service-discovery-implementation)
3. [API Gateway Overview](#api-gateway-overview)
4. [Spring Cloud Gateway Implementation](#spring-cloud-gateway-implementation)
5. [When to Use Service Discovery](#when-to-use-service-discovery)
6. [When to Use API Gateway](#when-to-use-api-gateway)
7. [Best Practices](#best-practices)
8. [Real-Time Examples from Codebase](#real-time-examples-from-codebase)
9. [Advanced Patterns and Strategies](#advanced-patterns-and-strategies)
10. [Advanced Interview Questions](#advanced-interview-questions)

---

## Service Discovery Overview

### What is Service Discovery?

**Service Discovery** is a pattern that allows microservices to find and communicate with each other without hardcoding service locations (IP addresses and ports).

### The Problem Without Service Discovery

**❌ Without Service Discovery:**
```java
// Hardcoded service URLs - BAD!
String userServiceUrl = "http://localhost:8081/api/users";
String productServiceUrl = "http://localhost:8082/api/products";

// Problems:
// 1. Services can't scale (multiple instances)
// 2. Services can't move (IP/port changes)
// 3. Manual configuration required
// 4. No health checking
// 5. No load balancing
```

**✅ With Service Discovery:**
```java
// Dynamic service discovery - GOOD!
String userServiceUrl = "http://user-service/api/users";
// Eureka automatically:
// 1. Finds available instances
// 2. Load balances requests
// 3. Handles failures
// 4. Updates when instances change
```

### Service Discovery Patterns

#### 1. Client-Side Discovery
- Client queries service registry
- Client selects service instance
- Client makes request directly
- **Example:** Netflix Eureka with Ribbon

#### 2. Server-Side Discovery
- Client makes request to load balancer
- Load balancer queries service registry
- Load balancer routes to service instance
- **Example:** AWS ELB, Kubernetes Service

### Service Discovery Components

1. **Service Registry**: Database of available service instances
2. **Service Registration**: Services register themselves
3. **Service Discovery**: Clients discover service instances
4. **Health Checks**: Monitor service availability
5. **Load Balancing**: Distribute requests across instances

---

## Eureka Service Discovery Implementation

### What is Eureka?

**Netflix Eureka** is a REST-based service registry for:
- Service registration and discovery
- Health monitoring
- Load balancing
- High availability

### Eureka Architecture

```
┌─────────────────┐
│  Eureka Server  │  ← Service Registry
│   (Port 8761)   │
└────────┬────────┘
         │
    ┌────┴────┬──────────┬──────────┐
    │         │          │          │
┌───▼───┐ ┌──▼───┐  ┌───▼───┐  ┌───▼───┐
│User   │ │Product│  │Seller │  │Gateway│
│Service│ │Service│  │Service│  │       │
└───────┘ └───────┘  └───────┘  └───────┘
    │         │          │          │
    └─────────┴──────────┴──────────┘
              │
         Register & Discover
```

### Eureka Server Setup

**File:** `service-registry/src/main/java/com/ekart/service_registry/ServiceRegistryApplication.java`

```java
package com.ekart.service_registry;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer  // Enable Eureka Server
public class ServiceRegistryApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServiceRegistryApplication.class, args);
    }
}
```

**Configuration:** `service-registry/src/main/resources/application.yaml`

```yaml
server:
  port: 8761  # Eureka Server Port

spring:
  application:
    name: service-registry

eureka:
  instance:
    hostname: localhost
  client:
    # Don't register this server with itself
    registerWithEureka: false
    # Don't fetch registry (it's the server)
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka
```

**Dependencies:** `pom.xml`

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

### Eureka Client Setup

**File:** `product-service/src/main/resources/application.yaml`

```yaml
spring:
  application:
    name: product-service  # Service name (used for discovery)

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka  # Eureka Server URL
    # Registration settings
    register-with-eureka: true   # Register with Eureka
    fetch-registry: true         # Fetch service registry
    # Heartbeat settings
    health-check:
      enabled: true              # Enable health checks
  instance:
    prefer-ip-address: true      # Use IP instead of hostname
    lease-renewal-interval-in-seconds: 30    # Heartbeat every 30s
    lease-expiration-duration-in-seconds: 90  # Expire after 90s
    # Metadata
    metadata-map:
      zone: us-east-1
      instance-type: standard
```

**Dependencies:** `pom.xml`

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

**Application Class:**

```java
@SpringBootApplication
@EnableEurekaClient  // Enable Eureka Client (optional in newer versions)
public class ProductServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProductServiceApplication.class, args);
    }
}
```

### Service Registration Flow

```
1. Service Starts
   ↓
2. Service Registers with Eureka
   POST /eureka/apps/{service-name}
   {
     "instance": {
       "hostName": "localhost",
       "ipAddr": "192.168.1.100",
       "port": 8082,
       "status": "UP"
     }
   }
   ↓
3. Eureka Stores Registration
   ↓
4. Service Sends Heartbeat (every 30s)
   PUT /eureka/apps/{service-name}/{instance-id}
   ↓
5. Eureka Updates Last Heartbeat Time
   ↓
6. If No Heartbeat for 90s → Mark as DOWN
```

### Service Discovery Flow

```
1. Client Needs Service
   ↓
2. Client Queries Eureka
   GET /eureka/apps/{service-name}
   ↓
3. Eureka Returns Available Instances
   {
     "application": {
       "name": "product-service",
       "instance": [
         {
           "hostName": "localhost",
           "ipAddr": "192.168.1.100",
           "port": 8082,
           "status": "UP"
         },
         {
           "hostName": "localhost",
           "ipAddr": "192.168.1.101",
           "port": 8082,
           "status": "UP"
         }
       ]
     }
   }
   ↓
4. Client Selects Instance (Load Balancing)
   ↓
5. Client Makes Request to Selected Instance
```

### Using Service Discovery in Code

**Option 1: Using RestTemplate with Eureka**

```java
@Service
public class ProductServiceClient {
    
    @Autowired
    private RestTemplate restTemplate;
    
    @Autowired
    private DiscoveryClient discoveryClient;
    
    public ProductDTO getProduct(Long id) {
        // Get service instances
        List<ServiceInstance> instances = discoveryClient
            .getInstances("product-service");
        
        if (instances.isEmpty()) {
            throw new ServiceUnavailableException("Product service not available");
        }
        
        // Select instance (round-robin, random, etc.)
        ServiceInstance instance = instances.get(0);
        String url = "http://" + instance.getHost() + ":" + instance.getPort() 
                   + "/api/products/" + id;
        
        return restTemplate.getForObject(url, ProductDTO.class);
    }
}
```

**Option 2: Using LoadBalanced RestTemplate**

```java
@Configuration
public class RestTemplateConfig {
    
    @Bean
    @LoadBalanced  // Enables Eureka service discovery
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

@Service
public class ProductServiceClient {
    
    @Autowired
    private RestTemplate restTemplate;  // LoadBalanced
    
    public ProductDTO getProduct(Long id) {
        // Eureka automatically resolves service name
        String url = "http://product-service/api/products/" + id;
        return restTemplate.getForObject(url, ProductDTO.class);
    }
}
```

**Option 3: Using Feign Client**

```java
@FeignClient(name = "product-service")
public interface ProductServiceClient {
    
    @GetMapping("/api/products/{id}")
    ProductDTO getProduct(@PathVariable Long id);
    
    @GetMapping("/api/products")
    List<ProductDTO> getAllProducts();
}

@Service
public class OrderService {
    
    @Autowired
    private ProductServiceClient productClient;
    
    public Order createOrder(OrderRequest request) {
        // Feign automatically uses Eureka for service discovery
        ProductDTO product = productClient.getProduct(request.getProductId());
        // ... create order
    }
}
```

### Eureka Dashboard

**Access:** http://localhost:8761

**Features:**
- View registered services
- View service instances
- View service status
- View metadata
- View last heartbeat time

---

## API Gateway Overview

### What is API Gateway?

**API Gateway** is a single entry point for all client requests that:
- Routes requests to appropriate microservices
- Handles cross-cutting concerns (auth, rate limiting, logging)
- Provides unified API interface
- Hides internal service structure

### The Problem Without API Gateway

**❌ Without API Gateway:**
```
Client → User Service (Port 8081)
Client → Product Service (Port 8082)
Client → Seller Service (Port 8083)
Client → Order Service (Port 8084)

Problems:
1. Clients need to know all service URLs
2. CORS issues (multiple origins)
3. Authentication duplicated in each service
4. Rate limiting duplicated
5. No unified error handling
6. Service coupling (clients depend on services)
```

**✅ With API Gateway:**
```
Client → API Gateway (Port 8080) → Services

Benefits:
1. Single entry point
2. Centralized authentication
3. Centralized rate limiting
4. Unified error handling
5. Service abstraction
6. Request/response transformation
```

### API Gateway Responsibilities

1. **Routing**: Route requests to appropriate services
2. **Authentication**: Verify JWT tokens, API keys
3. **Authorization**: Check permissions
4. **Rate Limiting**: Control request rates
5. **Load Balancing**: Distribute requests
6. **Caching**: Cache responses
7. **Request/Response Transformation**: Modify requests/responses
8. **Monitoring**: Logging, metrics, tracing
9. **Circuit Breaking**: Handle service failures
10. **CORS**: Handle cross-origin requests

---

## Spring Cloud Gateway Implementation

### What is Spring Cloud Gateway?

**Spring Cloud Gateway** is a reactive API gateway built on Spring WebFlux that provides:
- Reactive, non-blocking architecture
- Route definitions (YAML or Java)
- Built-in filters
- Integration with service discovery
- High performance

### Spring Cloud Gateway Setup

**File:** `api-gateway/src/main/java/com/ekart/api_gateway/ApiGatewayApplication.java`

```java
package com.ekart.api_gateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient  // Enable Eureka client for service discovery
public class ApiGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
}
```

**Configuration:** `api-gateway/src/main/resources/application.yaml`

```yaml
server:
  port: 8080

spring:
  application:
    name: api-gateway
  main:
    web-application-type: reactive  # Enable reactive (WebFlux)
  cloud:
    gateway:
      routes:
        # User Service Route
        - id: user-service
          uri: lb://user-service  # lb:// = load balanced via Eureka
          predicates:
            - Path=/api/users/**
          filters:
            - StripPrefix=0  # Don't strip /api/users prefix
        
        # Product Service Route
        - id: product-service
          uri: lb://product-service
          predicates:
            - Path=/api/products/**
          filters:
            - StripPrefix=0
        
        # Seller Service Route
        - id: seller-service
          uri: lb://seller-service
          predicates:
            - Path=/api/sellers/**
          filters:
            - StripPrefix=0
      
      # Global CORS Configuration
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOriginPatterns: "*"
            allowedMethods:
              - GET
              - POST
              - PUT
              - DELETE
              - OPTIONS
              - PATCH
            allowedHeaders: "*"
            allowCredentials: true
            maxAge: 3600

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
  instance:
    prefer-ip-address: true
```

**Dependencies:** `pom.xml`

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### Route Configuration

#### Route Components

1. **ID**: Unique route identifier
2. **URI**: Target service URI (`lb://service-name` for Eureka)
3. **Predicates**: Conditions for matching requests
4. **Filters**: Request/response modifications

#### Common Predicates

```yaml
routes:
  - id: example-route
    uri: lb://service-name
    predicates:
      # Path matching
      - Path=/api/products/**
      - Path=/api/products/{id}
      
      # Method matching
      - Method=GET,POST
      
      # Header matching
      - Header=X-Request-Id, \d+
      
      # Query parameter matching
      - Query=category,electronics
      
      # Cookie matching
      - Cookie=sessionId,.*
      
      # Host matching
      - Host=api.example.com
      
      # Time-based
      - After=2024-01-01T00:00:00Z
      - Before=2024-12-31T23:59:59Z
      - Between=2024-01-01T00:00:00Z,2024-12-31T23:59:59Z
```

#### Common Filters

```yaml
routes:
  - id: example-route
    uri: lb://service-name
    predicates:
      - Path=/api/products/**
    filters:
      # Strip prefix
      - StripPrefix=1  # /api/products/123 → /products/123
      
      # Add prefix
      - PrefixPath=/v1  # /api/products → /v1/api/products
      
      # Add request header
      - AddRequestHeader=X-Request-Id,${random.uuid}
      
      # Add response header
      - AddResponseHeader=X-Response-Time,${T(java.lang.System).currentTimeMillis()}
      
      # Remove request header
      - RemoveRequestHeader=X-Secret-Key
      
      # Rewrite path
      - RewritePath=/api/products/(?<segment>.*), /products/${segment}
      
      # Circuit breaker
      - name: CircuitBreaker
        args:
          name: productServiceCircuitBreaker
          fallbackUri: forward:/fallback/products
      
      # Retry
      - name: Retry
        args:
          retries: 3
          statuses: BAD_GATEWAY,INTERNAL_SERVER_ERROR
          methods: GET,POST
```

### Custom Filters

#### Global Filter Example: JWT Authentication

**File:** `api-gateway/src/main/java/com/ekart/api_gateway/filter/JwtAuthenticationFilter.java`

```java
@Component
public class JwtAuthenticationFilter implements GlobalFilter, Ordered {
    
    @Autowired
    private JwtUtil jwtUtil;
    
    private static final String[] PUBLIC_PATHS = {
        "/api/users/register",
        "/api/users/login",
        "/api/sellers/register",
        "/api/sellers/login"
    };
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        String path = request.getURI().getPath();
        
        // Skip authentication for public paths
        if (isPublicPath(path)) {
            return chain.filter(exchange);
        }
        
        // Extract token from Authorization header
        String authHeader = request.getHeaders().getFirst("Authorization");
        
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            return onError(exchange, "Missing or invalid Authorization header", 
                         HttpStatus.UNAUTHORIZED);
        }
        
        String token = authHeader.substring(7);
        
        try {
            // Validate token
            if (!jwtUtil.validateAccessToken(token)) {
                return onError(exchange, "Invalid or expired token", 
                              HttpStatus.UNAUTHORIZED);
            }
            
            // Extract user information
            String email = jwtUtil.extractEmail(token);
            Long userId = jwtUtil.extractUserId(token);
            String role = jwtUtil.extractRole(token);
            
            // Add user info to request headers for downstream services
            ServerHttpRequest.Builder requestBuilder = request.mutate();
            requestBuilder.header("X-User-Email", email);
            requestBuilder.header("X-User-Id", String.valueOf(userId));
            requestBuilder.header("X-User-Role", role);
            
            return chain.filter(exchange.mutate().request(requestBuilder.build()).build());
            
        } catch (Exception e) {
            return onError(exchange, "Token validation failed: " + e.getMessage(), 
                          HttpStatus.UNAUTHORIZED);
        }
    }
    
    private boolean isPublicPath(String path) {
        return Arrays.stream(PUBLIC_PATHS)
            .anyMatch(path::startsWith);
    }
    
    private Mono<Void> onError(ServerWebExchange exchange, String message, 
                              HttpStatus status) {
        ServerHttpResponse response = exchange.getResponse();
        response.setStatusCode(status);
        response.getHeaders().add("Content-Type", "application/json");
        
        String errorBody = String.format("{\"success\":false,\"message\":\"%s\"}", message);
        return response.writeWith(Mono.just(
            response.bufferFactory().wrap(errorBody.getBytes())));
    }
    
    @Override
    public int getOrder() {
        return -100; // High priority - run early
    }
}
```

#### Global Filter Example: Rate Limiting

**File:** `api-gateway/src/main/java/com/ekart/api_gateway/filter/RateLimitingFilter.java`

```java
@Component
public class RateLimitingFilter implements GlobalFilter, Ordered {
    
    private final Map<String, Bucket> cache = new ConcurrentHashMap<>();
    
    @Value("${rate.limit.requests.per-minute:60}")
    private int requestsPerMinute;
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        String clientIp = getClientIp(request);
        String userId = request.getHeaders().getFirst("X-User-Id");
        
        // Generate rate limit key
        String key = userId != null ? "user:" + userId : "ip:" + clientIp;
        
        // Get or create bucket
        Bucket bucket = cache.computeIfAbsent(key, k -> createBucket());
        
        // Try to consume token
        if (bucket.tryConsume(1)) {
            // Add rate limit headers
            ServerHttpResponse response = exchange.getResponse();
            response.getHeaders().add("X-RateLimit-Limit", 
                                     String.valueOf(requestsPerMinute));
            response.getHeaders().add("X-RateLimit-Remaining", 
                                     String.valueOf(bucket.getAvailableTokens()));
            
            return chain.filter(exchange);
        } else {
            // Rate limit exceeded
            return onRateLimitExceeded(exchange);
        }
    }
    
    private Bucket createBucket() {
        return Bucket4j.builder()
            .addLimit(Bandwidth.classic(requestsPerMinute, 
                       Refill.intervally(requestsPerMinute, Duration.ofMinutes(1))))
            .build();
    }
    
    private String getClientIp(ServerHttpRequest request) {
        String xForwardedFor = request.getHeaders().getFirst("X-Forwarded-For");
        if (xForwardedFor != null) {
            return xForwardedFor.split(",")[0].trim();
        }
        return request.getRemoteAddress() != null 
            ? request.getRemoteAddress().getAddress().getHostAddress() 
            : "unknown";
    }
    
    private Mono<Void> onRateLimitExceeded(ServerWebExchange exchange) {
        ServerHttpResponse response = exchange.getResponse();
        response.setStatusCode(HttpStatus.TOO_MANY_REQUESTS);
        response.getHeaders().add("Content-Type", "application/json");
        response.getHeaders().add("Retry-After", "60");
        
        String errorBody = "{\"success\":false,\"message\":\"Rate limit exceeded\"}";
        return response.writeWith(Mono.just(
            response.bufferFactory().wrap(errorBody.getBytes())));
    }
    
    @Override
    public int getOrder() {
        return -200; // Run before authentication
    }
}
```

### Filter Order

Filters execute in order based on `getOrder()` return value:
- Lower values execute first
- Negative values run before route filters
- Positive values run after route filters

**Recommended Order:**
1. Rate Limiting: -200
2. Authentication: -100
3. Logging: 0
4. Response Transformation: 100

---

## When to Use Service Discovery

### ✅ Use Service Discovery When:

1. **Microservices Architecture**: Multiple services need to communicate
2. **Dynamic Scaling**: Services scale up/down frequently
3. **Container Orchestration**: Services run in containers (Docker, Kubernetes)
4. **Cloud Deployment**: Services deployed in cloud (AWS, Azure, GCP)
5. **High Availability**: Need automatic failover
6. **Load Balancing**: Need to distribute requests across instances
7. **Service Mesh**: Using service mesh (Istio, Linkerd)

### ❌ Don't Use Service Discovery When:

1. **Monolithic Application**: Single application
2. **Static Infrastructure**: Services have fixed IPs/ports
3. **Simple Applications**: Few services, simple architecture
4. **Direct Database Access**: Services access database directly
5. **Message Queue Only**: Services communicate only via message queue

---

## When to Use API Gateway

### ✅ Use API Gateway When:

1. **Multiple Microservices**: Need single entry point
2. **Cross-Cutting Concerns**: Need centralized auth, rate limiting, logging
3. **Client Diversity**: Multiple client types (web, mobile, third-party)
4. **API Versioning**: Need to manage API versions
5. **Request Transformation**: Need to transform requests/responses
6. **Protocol Translation**: Need to translate protocols (REST, GraphQL, gRPC)
7. **Security**: Need centralized security policies
8. **Monitoring**: Need centralized monitoring and analytics

### ❌ Don't Use API Gateway When:

1. **Simple Application**: Single service, simple architecture
2. **Internal Services Only**: Services only communicate internally
3. **Direct Service Access**: Clients can access services directly
4. **Performance Critical**: Gateway adds latency (though minimal)
5. **Tight Coupling**: Services tightly coupled (use message queue instead)

---

## Best Practices

### Service Discovery Best Practices

**✅ DO:**
- Use health checks to remove unhealthy instances
- Configure appropriate heartbeat intervals
- Use metadata for service versioning
- Implement retry logic for service discovery failures
- Use circuit breakers for service calls
- Monitor service registry health
- Use multiple Eureka servers for high availability

**❌ DON'T:**
- Hardcode service URLs
- Ignore service health status
- Use long heartbeat intervals (causes slow failure detection)
- Register services without health checks
- Expose service registry publicly

### API Gateway Best Practices

**✅ DO:**
- Implement authentication/authorization at gateway
- Use rate limiting to protect services
- Implement circuit breakers for resilience
- Log all requests for monitoring
- Use request/response transformation sparingly
- Cache responses when appropriate
- Monitor gateway performance
- Use HTTPS for all external traffic

**❌ DON'T:**
- Put business logic in gateway
- Block requests unnecessarily
- Expose internal service details
- Skip authentication for sensitive endpoints
- Ignore gateway performance
- Use gateway for data aggregation (use BFF pattern instead)

---

## Real-Time Examples from Codebase

### Example 1: Service Registration

**Service:** `product-service`

```yaml
# application.yaml
spring:
  application:
    name: product-service  # Service name in Eureka

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
    register-with-eureka: true
    fetch-registry: true
  instance:
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 30
    lease-expiration-duration-in-seconds: 90
```

**Result:** Service appears in Eureka dashboard at http://localhost:8761

### Example 2: Service Discovery in Gateway

**Gateway Route Configuration:**

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: product-service
          uri: lb://product-service  # lb:// = load balanced via Eureka
          predicates:
            - Path=/api/products/**
```

**Flow:**
1. Request: `GET http://localhost:8080/api/products/123`
2. Gateway matches route: `product-service`
3. Gateway queries Eureka: "Where is product-service?"
4. Eureka returns: `http://192.168.1.100:8082`
5. Gateway forwards: `GET http://192.168.1.100:8082/api/products/123`
6. Gateway returns response to client

### Example 3: JWT Authentication Filter

**Request Flow:**
```
1. Client Request:
   GET /api/products/123
   Authorization: Bearer <token>

2. Gateway JWT Filter:
   - Extracts token
   - Validates token
   - Extracts user info
   - Adds headers: X-User-Id, X-User-Email

3. Gateway Forwards:
   GET /api/products/123
   X-User-Id: 1001
   X-User-Email: user@example.com

4. Product Service:
   - Receives request with user context
   - Processes request
   - Returns response
```

### Example 4: Rate Limiting

**Configuration:**

```yaml
rate:
  limit:
    requests-per-minute: 60
    premium:
      requests-per-minute: 120
```

**Behavior:**
- Free users: 60 requests/minute
- Premium users: 120 requests/minute
- Exceeded: Returns 429 Too Many Requests

---

## Advanced Patterns and Strategies

### 1. Multi-Region Service Discovery

**Scenario:** Services deployed in multiple regions

```yaml
# Region 1
eureka:
  client:
    service-url:
      defaultZone: http://eureka-us-east-1:8761/eureka
  instance:
    metadata-map:
      zone: us-east-1

# Region 2
eureka:
  client:
    service-url:
      defaultZone: http://eureka-us-west-2:8761/eureka
  instance:
    metadata-map:
      zone: us-west-2
```

**Gateway Route with Zone Preference:**

```java
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("product-service", r -> r
            .path("/api/products/**")
            .uri("lb://product-service")
            .filters(f -> f
                .filter(new ZonePreferenceFilter())  // Prefer same zone
            ))
        .build();
}
```

### 2. Canary Deployment with Gateway

**Scenario:** Gradually roll out new service version

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: product-service-v1
          uri: lb://product-service
          predicates:
            - Path=/api/products/**
            - Header=X-Version, v1
          filters:
            - StripPrefix=0
        
        - id: product-service-v2
          uri: lb://product-service-v2
          predicates:
            - Path=/api/products/**
            - Header=X-Version, v2
          filters:
            - StripPrefix=0
        
        - id: product-service-default
          uri: lb://product-service
          predicates:
            - Path=/api/products/**
            - Weight=group1, 90  # 90% to v1
          filters:
            - StripPrefix=0
```

### 3. API Versioning Strategy

**Option 1: Path-Based Versioning**

```yaml
routes:
  - id: product-service-v1
    uri: lb://product-service-v1
    predicates:
      - Path=/api/v1/products/**
  
  - id: product-service-v2
    uri: lb://product-service-v2
    predicates:
      - Path=/api/v2/products/**
```

**Option 2: Header-Based Versioning**

```yaml
routes:
  - id: product-service-v1
    uri: lb://product-service-v1
    predicates:
      - Path=/api/products/**
      - Header=X-API-Version, 1
  
  - id: product-service-v2
    uri: lb://product-service-v2
    predicates:
      - Path=/api/products/**
      - Header=X-API-Version, 2
```

### 4. Circuit Breaker Pattern

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: product-service
          uri: lb://product-service
          predicates:
            - Path=/api/products/**
          filters:
            - name: CircuitBreaker
              args:
                name: productServiceCircuitBreaker
                fallbackUri: forward:/fallback/products
            - name: Retry
              args:
                retries: 3
                statuses: BAD_GATEWAY,INTERNAL_SERVER_ERROR
```

**Fallback Handler:**

```java
@RestController
public class FallbackController {
    
    @GetMapping("/fallback/products")
    public ResponseEntity<Map<String, Object>> productFallback() {
        Map<String, Object> response = new HashMap<>();
        response.put("success", false);
        response.put("message", "Product service is temporarily unavailable");
        response.put("fallback", true);
        return ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE).body(response);
    }
}
```

### 5. Request/Response Transformation

```java
@Component
public class RequestTransformationFilter implements GatewayFilter, Ordered {
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        
        // Transform request body
        if (request.getHeaders().getContentType() != null 
            && request.getHeaders().getContentType().includes(MediaType.APPLICATION_JSON)) {
            
            return DataBufferUtils.join(request.getBody())
                .flatMap(dataBuffer -> {
                    byte[] bytes = new byte[dataBuffer.readableByteCount()];
                    dataBuffer.read(bytes);
                    DataBufferUtils.release(dataBuffer);
                    
                    // Transform JSON
                    String body = new String(bytes);
                    String transformed = transformJson(body);
                    
                    // Create new request with transformed body
                    ServerHttpRequest newRequest = request.mutate()
                        .body(transformed.getBytes())
                        .build();
                    
                    return chain.filter(exchange.mutate().request(newRequest).build());
                });
        }
        
        return chain.filter(exchange);
    }
    
    private String transformJson(String json) {
        // Transform logic
        return json;
    }
    
    @Override
    public int getOrder() {
        return 0;
    }
}
```

---

## Advanced Interview Questions

### Q1: Explain the difference between client-side and server-side service discovery.

**Answer:**
"Client-side discovery means the client is responsible for querying the service registry and selecting a service instance. The client then makes a direct request to the selected instance. Netflix Eureka with Ribbon is an example.

Server-side discovery means the client makes a request to a load balancer, which queries the service registry and routes the request to an appropriate instance. AWS ELB and Kubernetes Service are examples.

Client-side gives more control but adds complexity to clients. Server-side is simpler for clients but requires a load balancer. I use client-side with Eureka for fine-grained control and server-side with Kubernetes for simplicity."

### Q2: How does Eureka handle service failures and network partitions?

**Answer:**
"Eureka uses several mechanisms:

1. **Heartbeats**: Services send heartbeats every 30 seconds. If no heartbeat for 90 seconds, Eureka marks the instance as DOWN.

2. **Self-Preservation Mode**: If >15% of instances fail to renew within 90 seconds, Eureka enters self-preservation mode and doesn't evict instances. This prevents network partitions from removing all instances.

3. **Peer Replication**: Multiple Eureka servers replicate registrations. If one server fails, others continue serving.

4. **Client Caching**: Eureka clients cache the registry locally and refresh every 30 seconds. If Eureka is down, clients use cached data.

5. **Zone Awareness**: Clients prefer instances in the same zone, reducing cross-zone latency and partition impact.

In our system, I configure 30-second heartbeats, 90-second expiration, and enable self-preservation to handle network issues gracefully."

### Q3: What is the difference between Spring Cloud Gateway and Zuul?

**Answer:**
"Spring Cloud Gateway is built on Spring WebFlux (reactive, non-blocking) while Zuul 1.x is built on Servlet API (blocking). Gateway uses Netty for better performance and scalability.

Key differences:
- **Architecture**: Gateway is reactive, Zuul is blocking
- **Performance**: Gateway handles more concurrent requests
- **Filter Model**: Gateway uses WebFlux filters, Zuul uses Servlet filters
- **Maintenance**: Zuul 1.x is in maintenance mode, Gateway is actively developed
- **Features**: Gateway has better integration with Spring Cloud ecosystem

I use Spring Cloud Gateway for new projects because of its reactive architecture and better performance. For existing Zuul projects, I'd migrate gradually."

### Q4: How do you implement distributed rate limiting in Spring Cloud Gateway?

**Answer:**
"For distributed rate limiting, I use Redis with Spring Cloud Gateway's built-in Redis rate limiter:

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: product-service
          uri: lb://product-service
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
                key-resolver: "#{@ipKeyResolver}"
```

The key-resolver determines the rate limit key (IP, user ID, etc.). Redis stores the token bucket state, allowing rate limiting across multiple gateway instances.

For per-user rate limiting, I use a custom key resolver that extracts user ID from JWT token. For premium users, I configure higher limits.

I also implement fallback behavior: if Redis is down, I allow requests but log the failure for monitoring."

### Q5: Explain how API Gateway handles authentication and authorization.

**Answer:**
"I implement authentication at the gateway level using a global filter:

1. **Token Extraction**: Extract JWT token from Authorization header
2. **Token Validation**: Validate token signature and expiration
3. **User Context**: Extract user information (ID, email, role)
4. **Header Injection**: Add user context to request headers for downstream services
5. **Public Paths**: Skip authentication for public endpoints (login, register)

For authorization, I check user roles and permissions:
- Extract role from JWT token
- Check if role has permission for requested resource
- Reject unauthorized requests with 403 Forbidden

Downstream services trust the gateway and use the injected headers (X-User-Id, X-User-Role) for authorization. This avoids duplicate authentication logic in each service.

I also implement token refresh: if access token expires, the gateway can refresh it using the refresh token before forwarding the request."

### Q6: How do you handle service versioning with API Gateway?

**Answer:**
"I use multiple strategies:

1. **Path-Based Versioning**: `/api/v1/products`, `/api/v2/products`
   - Simple, explicit
   - Gateway routes to different service versions

2. **Header-Based Versioning**: `X-API-Version: 2`
   - Clean URLs
   - Gateway routes based on header

3. **Query Parameter**: `?version=2`
   - Easy for testing
   - Gateway routes based on parameter

4. **Canary Deployment**: Gradually route traffic to new version
   - Use weight-based routing: 90% to v1, 10% to v2
   - Monitor metrics, gradually increase v2 traffic

I prefer path-based for public APIs (explicit) and header-based for internal APIs (clean URLs). For canary deployments, I use weight-based routing with monitoring to ensure new version performs well."

### Q7: What is the difference between API Gateway and Service Mesh?

**Answer:**
"API Gateway is a north-south traffic controller (client to services) while Service Mesh is an east-west traffic controller (service to service).

API Gateway:
- Single entry point for external clients
- Handles authentication, rate limiting, routing
- Provides unified API interface
- Examples: Spring Cloud Gateway, Kong, AWS API Gateway

Service Mesh:
- Handles inter-service communication
- Provides service discovery, load balancing, security
- Transparent to services (sidecar pattern)
- Examples: Istio, Linkerd, Consul Connect

I use both: API Gateway for external traffic and Service Mesh for internal service communication. They complement each other - Gateway handles client concerns, Service Mesh handles service-to-service concerns."

### Q8: How do you implement circuit breaker pattern in Spring Cloud Gateway?

**Answer:**
"I use Resilience4j Circuit Breaker with Spring Cloud Gateway:

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: product-service
          uri: lb://product-service
          filters:
            - name: CircuitBreaker
              args:
                name: productServiceCircuitBreaker
                fallbackUri: forward:/fallback/products
```

Configuration:
```yaml
resilience4j:
  circuitbreaker:
    instances:
      productServiceCircuitBreaker:
        failureRateThreshold: 50
        waitDurationInOpenState: 10s
        slidingWindowSize: 10
        minimumNumberOfCalls: 5
```

When circuit opens (too many failures), requests go to fallback handler instead of service. After wait duration, circuit enters half-open state to test if service recovered.

I also implement retry with exponential backoff for transient failures before circuit opens."

### Q9: Explain how you handle CORS in API Gateway.

**Answer:**
"I configure CORS at the gateway level to handle cross-origin requests:

```yaml
spring:
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOriginPatterns: 
              - "https://*.example.com"
              - "http://localhost:3000"
            allowedMethods:
              - GET
              - POST
              - PUT
              - DELETE
            allowedHeaders: "*"
            allowCredentials: true
            maxAge: 3600
```

For production, I:
- Whitelist specific origins (not *)
- Use `allowedOriginPatterns` for subdomain matching
- Set `allowCredentials: true` only when needed
- Configure appropriate `maxAge` for preflight caching

I also handle preflight OPTIONS requests automatically. For fine-grained control, I create a custom CORS filter that checks origin against a database whitelist."

### Q10: How do you monitor and troubleshoot API Gateway performance?

**Answer:**
"I use multiple monitoring approaches:

1. **Actuator Endpoints**: Expose gateway metrics
   ```yaml
   management:
     endpoints:
       web:
         exposure:
           include: health,info,gateway,metrics
   ```

2. **Micrometer Metrics**: Track request rates, latencies, errors
   - Request count by route
   - Response time percentiles
   - Error rates
   - Circuit breaker states

3. **Distributed Tracing**: Use Sleuth/Zipkin for request tracing
   - Trace requests across gateway and services
   - Identify bottlenecks
   - Debug slow requests

4. **Logging**: Structured logging with correlation IDs
   - Log all requests with timing
   - Log filter execution
   - Log errors with stack traces

5. **Health Checks**: Monitor gateway and downstream services
   - Gateway health endpoint
   - Service health aggregation

6. **Performance Testing**: Load test gateway
   - Measure throughput
   - Identify bottlenecks
   - Tune thread pools and connection pools

I also set up alerts for high error rates, slow responses, and circuit breaker openings."

### Q11: How do you handle API Gateway high availability?

**Answer:**
"I implement high availability using:

1. **Multiple Gateway Instances**: Deploy multiple gateway instances behind a load balancer
   - Load balancer distributes requests
   - If one gateway fails, others handle traffic

2. **Stateless Design**: Gateway instances are stateless
   - No session state
   - Can scale horizontally
   - Any instance can handle any request

3. **Shared State**: Use Redis for shared state
   - Rate limiting state
   - Session data (if needed)
   - Cache (if needed)

4. **Health Checks**: Load balancer health checks
   - Remove unhealthy instances
   - Automatic failover

5. **Service Discovery**: Gateway uses Eureka for service discovery
   - If Eureka is down, use cached registry
   - Multiple Eureka servers for redundancy

6. **Circuit Breakers**: Protect gateway from downstream failures
   - Fail fast on service failures
   - Prevent cascade failures

In production, I deploy at least 3 gateway instances across multiple availability zones for redundancy."

### Q12: Explain the difference between gateway filters and global filters.

**Answer:**
"Gateway filters are applied to specific routes, while global filters are applied to all routes.

**Gateway Filters:**
- Defined per route in configuration
- Applied only to matching routes
- Examples: StripPrefix, AddRequestHeader, CircuitBreaker

**Global Filters:**
- Implemented as Spring beans
- Applied to all routes automatically
- Examples: Authentication, Logging, Rate Limiting

I use gateway filters for route-specific logic (path rewriting, headers) and global filters for cross-cutting concerns (authentication, logging, rate limiting).

Filter order matters: global filters can set order using `Ordered` interface. Lower order values execute first. I typically order: Rate Limiting (-200), Authentication (-100), Logging (0), Response Transformation (100)."

---

## Summary

**Service Discovery (Eureka):**
- Enables dynamic service location
- Handles service registration and discovery
- Provides health monitoring
- Supports load balancing
- Essential for microservices architecture

**API Gateway (Spring Cloud Gateway):**
- Single entry point for clients
- Centralized cross-cutting concerns
- Service abstraction
- Request routing and transformation
- High performance (reactive architecture)

**Key Takeaways:**
- Use service discovery for microservices communication
- Use API gateway for client-facing APIs
- Implement authentication at gateway level
- Use circuit breakers for resilience
- Monitor and log everything
- Design for high availability

**Best Practices:**
- Keep gateway stateless
- Use health checks
- Implement rate limiting
- Use circuit breakers
- Monitor performance
- Design for failure

