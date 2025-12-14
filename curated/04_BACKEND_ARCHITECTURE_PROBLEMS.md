# Backend Architecture Problems - 30+ Real-World Problems

> **Curated from top tech companies and real production scenarios**

This guide contains 30+ real-world backend architecture problems with comprehensive solutions, suitable for staff engineer interviews.

---

## Table of Contents

1. [Microservices Architecture Problems](#microservices-architecture-problems)
2. [API Design Problems](#api-design-problems)
3. [Database Design Problems](#database-design-problems)
4. [Caching Problems](#caching-problems)
5. [Performance Problems](#performance-problems)

---

## Microservices Architecture Problems

### Problem 1: Design Microservices Communication

**Problem Statement:**
Design communication patterns for 20+ microservices handling 1M requests per day.

**Requirements:**
- Synchronous and asynchronous communication
- Service discovery
- Load balancing
- Circuit breaker
- Retry mechanism

**Solution:**

```java
// Synchronous: REST API with Service Discovery
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    @Autowired
    private RestTemplate restTemplate;
    
    @Autowired
    private CircuitBreaker circuitBreaker;
    
    @PostMapping
    public ResponseEntity<Order> createOrder(@RequestBody OrderRequest request) {
        // Call inventory service
        InventoryResponse inventory = circuitBreaker.executeSupplier(() ->
            restTemplate.postForObject(
                "http://inventory-service/api/inventory/reserve",
                request.getItems(),
                InventoryResponse.class
            )
        );
        
        if (!inventory.isReserved()) {
            return ResponseEntity.badRequest().build();
        }
        
        // Create order
        Order order = orderService.createOrder(request);
        return ResponseEntity.ok(order);
    }
}

// Asynchronous: Event-Driven with Kafka
@Service
public class OrderService {
    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;
    
    public void createOrder(Order order) {
        orderRepository.save(order);
        
        // Publish event
        OrderCreatedEvent event = new OrderCreatedEvent(order);
        kafkaTemplate.send("order-created", order.getId(), event);
    }
}

// Service Discovery with Eureka
@SpringBootApplication
@EnableEurekaClient
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}

// Circuit Breaker Configuration
@Configuration
public class ResilienceConfig {
    @Bean
    public CircuitBreaker circuitBreaker() {
        return CircuitBreaker.of("inventoryService", CircuitBreakerConfig.custom()
            .failureRateThreshold(50)
            .waitDurationInOpenState(Duration.ofSeconds(10))
            .slidingWindowSize(10)
            .build());
    }
}
```

**Communication Patterns:**
- **Synchronous**: REST for request-response
- **Asynchronous**: Kafka for event-driven
- **Service Discovery**: Eureka/Consul
- **Circuit Breaker**: Resilience4j
- **API Gateway**: Single entry point

---

### Problem 2: Design Distributed Transaction Handling

**Problem Statement:**
Handle transactions across multiple microservices (order, inventory, payment) ensuring consistency.

**Requirements:**
- Distributed transactions
- Consistency guarantees
- Failure handling
- Rollback mechanism

**Solution:**

```java
// Saga Pattern: Orchestration-based
@Service
public class OrderSagaOrchestrator {
    @Autowired
    private OrderService orderService;
    @Autowired
    private InventoryService inventoryService;
    @Autowired
    private PaymentService paymentService;
    
    @Transactional
    public OrderResult processOrder(OrderRequest request) {
        List<SagaStep> steps = new ArrayList<>();
        
        try {
            // Step 1: Create order
            Order order = orderService.createOrder(request);
            steps.add(new SagaStep("createOrder", order.getId(), null));
            
            // Step 2: Reserve inventory
            InventoryResult inventory = inventoryService.reserveInventory(
                order.getId(), request.getItems()
            );
            steps.add(new SagaStep("reserveInventory", order.getId(), inventory));
            
            if (!inventory.isReserved()) {
                compensate(steps);
                return OrderResult.failed("Inventory unavailable");
            }
            
            // Step 3: Process payment
            PaymentResult payment = paymentService.processPayment(
                order.getId(), request.getPaymentInfo()
            );
            steps.add(new SagaStep("processPayment", order.getId(), payment));
            
            if (!payment.isSuccess()) {
                compensate(steps);
                return OrderResult.failed("Payment failed");
            }
            
            // All steps successful
            return OrderResult.success(order);
            
        } catch (Exception e) {
            compensate(steps);
            throw e;
        }
    }
    
    private void compensate(List<SagaStep> steps) {
        // Compensate in reverse order
        Collections.reverse(steps);
        for (SagaStep step : steps) {
            switch (step.getAction()) {
                case "processPayment":
                    paymentService.refund(step.getOrderId());
                    break;
                case "reserveInventory":
                    inventoryService.releaseInventory(step.getOrderId());
                    break;
                case "createOrder":
                    orderService.cancelOrder(step.getOrderId());
                    break;
            }
        }
    }
}

// Saga Step
public class SagaStep {
    private String action;
    private String orderId;
    private Object result;
    
    // Constructor, getters
}
```

**Saga Pattern:**
- **Orchestration**: Central coordinator
- **Choreography**: Event-driven
- **Compensation**: Rollback mechanism
- **Eventual Consistency**: Acceptable for most cases

---

## API Design Problems

### Problem 3: Design RESTful API for E-Commerce

**Problem Statement:**
Design RESTful API for e-commerce platform with products, orders, users, and payments.

**Requirements:**
- RESTful principles
- Versioning
- Pagination
- Filtering and sorting
- Error handling
- Rate limiting

**Solution:**

```java
// RESTful API Design
@RestController
@RequestMapping("/api/v1")
public class ProductController {
    
    // GET /api/v1/products - List products
    @GetMapping("/products")
    public ResponseEntity<Page<Product>> getProducts(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(required = false) String category,
            @RequestParam(required = false) String sortBy,
            @RequestParam(required = false) String order) {
        
        Pageable pageable = PageRequest.of(page, size);
        if (sortBy != null) {
            Sort sort = order.equals("desc") 
                ? Sort.by(sortBy).descending() 
                : Sort.by(sortBy).ascending();
            pageable = PageRequest.of(page, size, sort);
        }
        
        Page<Product> products = productService.getProducts(category, pageable);
        return ResponseEntity.ok(products);
    }
    
    // GET /api/v1/products/{id} - Get product
    @GetMapping("/products/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable Long id) {
        Product product = productService.getProduct(id);
        if (product == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(product);
    }
    
    // POST /api/v1/products - Create product
    @PostMapping("/products")
    public ResponseEntity<Product> createProduct(@RequestBody @Valid ProductRequest request) {
        Product product = productService.createProduct(request);
        return ResponseEntity.status(HttpStatus.CREATED)
            .header("Location", "/api/v1/products/" + product.getId())
            .body(product);
    }
    
    // PUT /api/v1/products/{id} - Update product
    @PutMapping("/products/{id}")
    public ResponseEntity<Product> updateProduct(
            @PathVariable Long id,
            @RequestBody @Valid ProductRequest request) {
        Product product = productService.updateProduct(id, request);
        return ResponseEntity.ok(product);
    }
    
    // DELETE /api/v1/products/{id} - Delete product
    @DeleteMapping("/products/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        productService.deleteProduct(id);
        return ResponseEntity.noContent().build();
    }
}

// Error Handling
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException e) {
        ErrorResponse error = new ErrorResponse(
            "NOT_FOUND",
            e.getMessage(),
            HttpStatus.NOT_FOUND.value()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(ValidationException e) {
        ErrorResponse error = new ErrorResponse(
            "VALIDATION_ERROR",
            e.getMessage(),
            HttpStatus.BAD_REQUEST.value()
        );
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
}

// Rate Limiting
@Component
public class RateLimitingFilter implements Filter {
    @Autowired
    private RateLimiter rateLimiter;
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                        FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String clientId = getClientId(httpRequest);
        
        if (!rateLimiter.allowRequest(clientId)) {
            HttpServletResponse httpResponse = (HttpServletResponse) response;
            httpResponse.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            return;
        }
        
        chain.doFilter(request, response);
    }
}
```

**RESTful Principles:**
- Resource-based URLs
- HTTP methods (GET, POST, PUT, DELETE)
- Status codes
- Stateless
- HATEOAS (optional)

---

## Database Design Problems

### Problem 4: Design Database for High-Traffic Application

**Problem Statement:**
Design database schema for e-commerce platform handling 10M users and 1M orders per day.

**Requirements:**
- Normalized schema
- Indexing strategy
- Sharding strategy
- Read replicas
- Caching strategy

**Solution:**

```sql
-- Normalized Schema
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_email (email),
    INDEX idx_created_at (created_at)
);

CREATE TABLE products (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    category_id BIGINT,
    seller_id BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_category (category_id),
    INDEX idx_seller (seller_id),
    INDEX idx_created_at (created_at),
    FULLTEXT INDEX idx_search (name, description)
);

CREATE TABLE orders (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    status VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_user (user_id),
    INDEX idx_status (status),
    INDEX idx_created_at (created_at),
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE order_items (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    order_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(id),
    FOREIGN KEY (product_id) REFERENCES products(id),
    INDEX idx_order (order_id)
);

-- Sharding Strategy: Shard by user_id
-- Shard 1: user_id % 4 = 0
-- Shard 2: user_id % 4 = 1
-- Shard 3: user_id % 4 = 2
-- Shard 4: user_id % 4 = 3

-- Read Replicas
-- Master: Handles writes
-- Replica 1, 2, 3: Handle reads

-- Caching Strategy
-- Hot data: User profiles, popular products
-- Cache TTL: 5 minutes for products, 1 hour for user profiles
```

**Database Design Principles:**
- Normalization (3NF)
- Proper indexing
- Sharding for scale
- Read replicas for read scalability
- Caching for hot data

---

## Caching Problems

### Problem 5: Design Multi-Level Caching Strategy

**Problem Statement:**
Design caching strategy for application with 1M requests per day, reducing database load by 80%.

**Requirements:**
- Multiple cache levels
- Cache invalidation
- Cache warming
- Distributed caching

**Solution:**

```java
// Multi-Level Caching
@Service
public class ProductService {
    @Autowired
    private ProductRepository productRepository;
    
    @Autowired
    private RedisTemplate<String, Product> redisCache; // L2 Cache
    
    private final Cache<String, Product> localCache = Caffeine.newBuilder()
        .maximumSize(10000)
        .expireAfterWrite(5, TimeUnit.MINUTES)
        .build(); // L1 Cache
    
    public Product getProduct(Long id) {
        String cacheKey = "product:" + id;
        
        // L1: Local cache
        Product product = localCache.getIfPresent(cacheKey);
        if (product != null) {
            return product;
        }
        
        // L2: Redis cache
        product = redisCache.opsForValue().get(cacheKey);
        if (product != null) {
            localCache.put(cacheKey, product);
            return product;
        }
        
        // L3: Database
        product = productRepository.findById(id).orElse(null);
        if (product != null) {
            redisCache.opsForValue().set(cacheKey, product, 10, TimeUnit.MINUTES);
            localCache.put(cacheKey, product);
        }
        
        return product;
    }
    
    public void updateProduct(Long id, Product product) {
        // Update database
        productRepository.save(product);
        
        // Invalidate caches
        String cacheKey = "product:" + id;
        localCache.invalidate(cacheKey);
        redisCache.delete(cacheKey);
        
        // Publish invalidation event
        kafkaTemplate.send("cache-invalidation", cacheKey);
    }
}

// Cache Warming
@Component
public class CacheWarmer {
    @Autowired
    private ProductService productService;
    
    @Scheduled(fixedRate = 3600000) // Every hour
    public void warmCache() {
        // Get popular products
        List<Long> popularProductIds = getPopularProductIds();
        
        // Pre-load into cache
        popularProductIds.parallelStream()
            .forEach(id -> productService.getProduct(id));
    }
}
```

**Caching Strategy:**
- **L1**: Local cache (Caffeine) - Fastest, limited size
- **L2**: Distributed cache (Redis) - Shared across instances
- **L3**: Database - Source of truth
- **Invalidation**: Event-based + TTL
- **Warming**: Pre-load popular data

---

## Summary

These problems cover:
- ✅ Microservices architecture
- ✅ API design
- ✅ Database design
- ✅ Caching strategies
- ✅ Distributed transactions
- ✅ Performance optimization

**Key Principles:**
- Service-oriented architecture
- RESTful API design
- Database optimization
- Multi-level caching
- Saga pattern for distributed transactions

---

*This guide is continuously updated with new problems and solutions.*

