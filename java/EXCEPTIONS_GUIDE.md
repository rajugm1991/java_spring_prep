# Java Exceptions - Complete Advanced Guide 🚨

> Comprehensive guide to Java exceptions for senior engineers - Interview preparation with advanced topics, visual diagrams, and real-world examples

---

## Table of Contents
1. [Exception Hierarchy](#exception-hierarchy)
2. [Checked vs Unchecked Exceptions](#checked-vs-unchecked-exceptions)
3. [Exception Handling Mechanisms](#exception-handling-mechanisms)
4. [Advanced Exception Concepts](#advanced-exception-concepts)
5. [Best Practices](#best-practices)
6. [Common Patterns](#common-patterns)
7. [Performance Considerations](#performance-considerations)
8. [Interview Questions](#interview-questions)

---

## Exception Hierarchy

### Visual Hierarchy Diagram

```
                    Throwable
                    /       \
              Error      Exception
              /            /      \
    OutOfMemoryError  RuntimeException  Checked Exceptions
    StackOverflowError    /              \
    ...              NullPointerException  IOException
                      IllegalArgumentException  SQLException
                      IllegalStateException     FileNotFoundException
                      ...
```

### Key Classes

#### Throwable (Root Class)

**All exceptions and errors extend from Throwable**

```java
public class Throwable implements Serializable {
    private String message;
    private Throwable cause;
    private StackTraceElement[] stackTrace;
    
    // Constructors
    public Throwable()
    public Throwable(String message)
    public Throwable(String message, Throwable cause)
    public Throwable(Throwable cause)
    
    // Methods
    public String getMessage()
    public Throwable getCause()
    public StackTraceElement[] getStackTrace()
    public void printStackTrace()
    public Throwable initCause(Throwable cause)
}
```

**Key Points:**
- Root of exception hierarchy
- Contains message, cause, and stack trace
- Can be serialized
- Both Error and Exception extend Throwable

#### Error (Unchecked)

**Serious problems that applications should not catch**

```java
// Common Error Types
OutOfMemoryError        // JVM runs out of memory
StackOverflowError      // Stack overflow
NoClassDefFoundError    // Class not found at runtime
LinkageError           // Linkage problems
VirtualMachineError    // JVM errors
```

**Example:**
```java
public class ErrorExample {
    public static void main(String[] args) {
        // This will cause OutOfMemoryError
        List<byte[]> list = new ArrayList<>();
        while (true) {
            list.add(new byte[1024 * 1024]); // 1MB each
        }
    }
}
```

**Characteristics:**
- ❌ Should NOT be caught
- ❌ Indicates serious JVM problems
- ❌ Usually cannot be recovered
- ✅ Unchecked (no compile-time checking)

#### Exception (Base Class)

**Problems that applications might want to catch**

```java
// Two main categories:
1. RuntimeException (Unchecked)
2. Checked Exceptions
```

---

## Checked vs Unchecked Exceptions

### Visual Comparison

```
┌─────────────────────────────────────────────────────────┐
│              EXCEPTION TYPE COMPARISON                   │
└─────────────────────────────────────────────────────────┘

Checked Exceptions              Unchecked Exceptions
─────────────────              ────────────────────
✅ Compile-time checking       ❌ No compile-time checking
✅ Must be handled/caught      ❌ Optional handling
✅ Must be declared            ❌ No declaration needed
✅ Extend Exception            ✅ Extend RuntimeException
✅ Recovery possible           ⚠️ Usually programming errors
✅ Examples:                   ✅ Examples:
   - IOException                  - NullPointerException
   - SQLException                 - IllegalArgumentException
   - FileNotFoundException        - IllegalStateException
   - ClassNotFoundException       - IndexOutOfBoundsException
```

### Checked Exceptions

**Must be handled at compile time**

```java
// Example: IOException
public void readFile(String filename) throws IOException {
    FileReader file = new FileReader(filename);
    // Compiler forces you to handle IOException
}

// Must either:
// 1. Declare with throws
public void method() throws IOException { }

// 2. Handle with try-catch
public void method() {
    try {
        readFile("test.txt");
    } catch (IOException e) {
        // Handle exception
    }
}
```

**Common Checked Exceptions:**

| Exception | When Thrown |
|-----------|-------------|
| `IOException` | I/O operations fail |
| `SQLException` | Database operations fail |
| `FileNotFoundException` | File not found |
| `ClassNotFoundException` | Class not found at runtime |
| `InterruptedException` | Thread interrupted |
| `ParseException` | Parsing fails |

### Unchecked Exceptions (RuntimeException)

**No compile-time checking required**

```java
// Example: NullPointerException
public void process(String str) {
    int length = str.length(); // No compile-time check
    // If str is null, throws NullPointerException at runtime
}

// No need to declare or catch
public void method() {
    process(null); // Compiles fine, throws at runtime
}
```

**Common Unchecked Exceptions:**

| Exception | When Thrown |
|-----------|-------------|
| `NullPointerException` | Null reference accessed |
| `IllegalArgumentException` | Invalid argument passed |
| `IllegalStateException` | Invalid state |
| `IndexOutOfBoundsException` | Array/Collection index invalid |
| `ArithmeticException` | Arithmetic error (e.g., divide by zero) |
| `ClassCastException` | Invalid cast |
| `ConcurrentModificationException` | Collection modified during iteration |

### Decision Tree: When to Use Which?

```
Need to create custom exception?
    │
    ├─ Can caller recover? ──Yes──→ Checked Exception
    │                              (IOException, SQLException)
    │
    ├─ Programming error? ──Yes──→ Unchecked Exception
    │                            (IllegalArgumentException)
    │
    └─ JVM/system error? ──Yes──→ Error
                                  (OutOfMemoryError)
```

---

## Real-World Examples from E-Kart E-Commerce Application

### Exception Structure in E-Kart

The E-Kart application follows a structured approach to exception handling with custom exceptions for different scenarios.

#### Exception Hierarchy in E-Kart

```
Throwable
├── Error (System-level issues - not caught)
│   └── OutOfMemoryError (JVM memory exhausted)
│
└── Exception
    ├── Checked Exceptions (Recoverable)
    │   ├── IOException (File/Network operations)
    │   ├── SQLException (Database operations)
    │   └── InterruptedException (Thread interruption)
    │
    └── RuntimeException (Unchecked - Programming errors)
        ├── IllegalArgumentException (Invalid input)
        ├── IllegalStateException (Invalid state)
        ├── NullPointerException (Null reference)
        └── Custom Business Exceptions
            ├── PaymentProcessingException
            ├── RefundException
            ├── LockAcquisitionException
            └── EventPublishException
```

### Example 1: Payment Processing Exception (Unchecked)

**Location:** `product-service/src/main/java/com/ekart/product_service/saga/step/ProcessPaymentStep.java`

**Why Unchecked?**
- Payment failures are usually programming/configuration errors
- Caller typically cannot recover at runtime
- Should fail fast and be handled at a higher level

**Implementation:**

```java
@Component
public class ProcessPaymentStep implements SagaStep {
    
    private static final Logger logger = LoggerFactory.getLogger(ProcessPaymentStep.class);
    
    @Override
    public void execute(SagaContext context) throws Exception {
        logger.info("Executing ProcessPaymentStep for saga: {}, orderId: {}", 
                   context.getSagaId(), context.getOrderId());
        
        // Calculate total amount
        BigDecimal totalAmount = context.getRequest().getCartItems().stream()
                .map(item -> {
                    BigDecimal unitPrice = BigDecimal.valueOf(1000); // Mock price
                    return unitPrice.multiply(BigDecimal.valueOf(item.getQuantity()));
                })
                .reduce(BigDecimal.ZERO, BigDecimal::add);
        
        // Process payment
        boolean paymentSuccess = simulatePaymentProcessing(context.getOrderId(), totalAmount);
        
        if (!paymentSuccess) {
            // Throw unchecked exception - payment failure is a business error
            throw new PaymentProcessingException("Payment processing failed for order: " + context.getOrderId());
        }
        
        context.setPaymentProcessed(true);
        logger.info("Payment processed for order: {}, amount: {}", context.getOrderId(), totalAmount);
    }
    
    @Override
    public void compensate(SagaContext context) throws Exception {
        logger.info("Compensating ProcessPaymentStep for saga: {}, orderId: {}", 
                   context.getSagaId(), context.getOrderId());
        
        if (context.isPaymentProcessed()) {
            // Refund payment
            boolean refundSuccess = simulateRefund(context.getOrderId());
            
            if (!refundSuccess) {
                // Throw unchecked exception - refund failure needs attention
                throw new RefundException("Failed to refund payment for order: " + context.getOrderId());
            }
            
            context.setPaymentProcessed(false);
            logger.info("Payment refunded for order: {}", context.getOrderId());
        }
    }
    
    // Custom Unchecked Exception for Payment Processing
    public static class PaymentProcessingException extends RuntimeException {
        public PaymentProcessingException(String message) {
            super(message);
        }
        
        public PaymentProcessingException(String message, Throwable cause) {
            super(message, cause);
        }
    }
    
    // Custom Unchecked Exception for Refund
    public static class RefundException extends RuntimeException {
        public RefundException(String message) {
            super(message);
        }
        
        public RefundException(String message, Throwable cause) {
            super(message, cause);
        }
    }
}
```

**Key Points:**
- ✅ Extends `RuntimeException` (unchecked)
- ✅ Meaningful error messages with context (orderId)
- ✅ Can include cause for exception chaining
- ✅ Used in Saga pattern for distributed transactions

### Example 2: Distributed Lock Exception (Unchecked)

**Location:** `product-service/src/main/java/com/ekart/product_service/lock/DistributedLockService.java`

**Why Unchecked?**
- Lock acquisition failure is usually a system/configuration issue
- Indicates resource contention or system problems
- Should be handled at a higher level with retry logic

**Implementation:**

```java
@Service
public class DistributedLockService {
    
    private static final Logger logger = LoggerFactory.getLogger(DistributedLockService.class);
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    private static final String LOCK_PREFIX = "lock:";
    private static final Duration DEFAULT_LOCK_TIMEOUT = Duration.ofSeconds(30);
    
    /**
     * Execute a critical section with distributed lock
     * 
     * @param lockKey The lock key
     * @param timeout Lock timeout
     * @param criticalSection The code to execute
     * @return Result of the critical section
     * @throws LockAcquisitionException if lock cannot be acquired
     */
    public <T> T executeWithLock(String lockKey, Duration timeout, 
                                LockedOperation<T> criticalSection) {
        LockToken lockToken = acquireLock(lockKey, timeout);
        if (lockToken == null) {
            // Throw unchecked exception - lock acquisition failure
            throw new LockAcquisitionException("Could not acquire lock: " + lockKey);
        }
        
        try {
            return criticalSection.execute();
        } finally {
            releaseLock(lockToken);
        }
    }
    
    /**
     * Exception thrown when lock acquisition fails
     * 
     * This is an unchecked exception because:
     * - Lock failures indicate system/resource issues
     * - Usually cannot be recovered at the call site
     * - Should be handled with retry logic at higher level
     */
    public static class LockAcquisitionException extends RuntimeException {
        public LockAcquisitionException(String message) {
            super(message);
        }
        
        public LockAcquisitionException(String message, Throwable cause) {
            super(message, cause);
        }
    }
}
```

**Usage Example:**

```java
@Service
public class InventoryService {
    
    @Autowired
    private DistributedLockService lockService;
    
    @Retryable(value = {LockAcquisitionException.class}, maxAttempts = 3)
    public void updateInventory(Long skuId, Integer quantity) {
        String lockKey = "inventory:" + skuId;
        
        // Will throw LockAcquisitionException if lock cannot be acquired
        lockService.executeWithLock(lockKey, Duration.ofSeconds(30), () -> {
            // Critical section - update inventory
            Inventory inventory = inventoryRepository.findBySkuId(skuId)
                .orElseThrow(() -> new InventoryNotFoundException(skuId));
            
            if (inventory.getAvailableQuantity() >= quantity) {
                inventory.setAvailableQuantity(
                    inventory.getAvailableQuantity() - quantity
                );
                inventoryRepository.save(inventory);
            } else {
                throw new InsufficientInventoryException(skuId, quantity);
            }
            
            return null;
        });
    }
}
```

### Example 3: Event Publishing Exception (Unchecked)

**Location:** `product-service/src/main/java/com/ekart/product_service/event/OrderEventPublisher.java`

**Why Unchecked?**
- Event publishing failures are usually infrastructure issues
- Should not block main business logic
- Can be handled with retry or dead-letter queue

**Implementation:**

```java
@Component
public class OrderEventPublisher {
    
    private static final Logger logger = LoggerFactory.getLogger(OrderEventPublisher.class);
    
    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;
    
    /**
     * Publish OrderCreated event
     * 
     * @param orderId Order ID
     * @param buyerId Buyer ID
     * @param orderData Order data
     * @throws EventPublishException if event publishing fails
     */
    public void publishOrderCreated(String orderId, Long buyerId, Object orderData) {
        try {
            OrderCreatedEvent event = new OrderCreatedEvent(orderId, buyerId, orderData);
            kafkaTemplate.send("order-created", orderId, event);
            logger.info("OrderCreated event published: orderId={}", orderId);
        } catch (Exception e) {
            // Wrap in unchecked exception with cause
            logger.error("Failed to publish OrderCreated event: orderId={}", orderId, e);
            throw new EventPublishException("Failed to publish OrderCreated event", e);
        }
    }
    
    /**
     * Custom Unchecked Exception for Event Publishing
     * 
     * Why unchecked?
     * - Event publishing failures are infrastructure issues
     * - Should not block main business logic
     * - Can be handled with retry mechanism or dead-letter queue
     */
    public static class EventPublishException extends RuntimeException {
        public EventPublishException(String message) {
            super(message);
        }
        
        public EventPublishException(String message, Throwable cause) {
            super(message, cause);
        }
    }
}
```

### Example 4: Saga Pattern Exception Handling (Checked)

**Location:** `product-service/src/main/java/com/ekart/product_service/saga/OrderSagaOrchestrator.java`

**Why Checked Exception in Interface?**
- Saga steps can fail for recoverable reasons
- Caller (orchestrator) needs to handle failures
- Allows compensation logic to be executed

**Implementation:**

```java
/**
 * Saga Step Interface
 * 
 * Uses checked Exception because:
 * - Steps can fail for recoverable reasons (network issues, temporary failures)
 * - Orchestrator needs to handle failures and execute compensation
 * - Allows proper error handling and retry logic
 */
public interface SagaStep {
    /**
     * Execute the saga step
     * 
     * @param context Saga context
     * @throws Exception if step fails (checked - caller must handle)
     */
    void execute(SagaContext context) throws Exception;
    
    /**
     * Compensate (undo) the saga step
     * 
     * @param context Saga context
     * @throws Exception if compensation fails (checked - caller must handle)
     */
    void compensate(SagaContext context) throws Exception;
}

/**
 * Saga Orchestrator - Handles exception and compensation
 */
@Component
public class OrderSagaOrchestrator {
    
    private static final Logger logger = LoggerFactory.getLogger(OrderSagaOrchestrator.class);
    
    @Autowired
    private List<SagaStep> sagaSteps;
    
    @Autowired
    private OrderEventPublisher eventPublisher;
    
    /**
     * Process order using Saga pattern
     * 
     * Handles checked exceptions from saga steps and executes compensation
     */
    public SagaResult processOrder(Long buyerId, CheckoutRequest request) {
        String sagaId = UUID.randomUUID().toString();
        SagaContext context = new SagaContext(sagaId, buyerId, request);
        List<SagaStep> completedSteps = new ArrayList<>();
        
        try {
            // Execute saga steps sequentially
            for (SagaStep step : sagaSteps) {
                logger.info("Executing saga step: {}", step.getClass().getSimpleName());
                
                // Step can throw checked Exception
                step.execute(context);
                completedSteps.add(step);
            }
            
            // All steps completed successfully
            eventPublisher.publishOrderCreated(context.getOrderId(), buyerId, context);
            return new SagaResult(true, "Order processed successfully", context);
            
        } catch (Exception e) {
            // Handle checked exception from saga steps
            logger.error("Order saga failed: sagaId={}, error={}", sagaId, e.getMessage(), e);
            
            // Compensate completed steps in reverse order
            compensate(completedSteps, context, e);
            
            // Publish OrderCancelled event
            eventPublisher.publishOrderCancelled(context.getOrderId(), buyerId, e.getMessage());
            
            return new SagaResult(false, "Order processing failed: " + e.getMessage(), context);
        }
    }
    
    /**
     * Compensate completed steps in reverse order
     * 
     * Handles exceptions during compensation - continues even if one fails
     */
    private void compensate(List<SagaStep> completedSteps, SagaContext context, Exception error) {
        logger.info("Compensating {} completed steps", completedSteps.size());
        
        // Compensate in reverse order
        for (int i = completedSteps.size() - 1; i >= 0; i--) {
            SagaStep step = completedSteps.get(i);
            try {
                logger.info("Compensating step: {}", step.getClass().getSimpleName());
                // Compensation can also throw checked Exception
                step.compensate(context);
            } catch (Exception e) {
                // Log but continue compensating other steps
                logger.error("Error compensating step: {}", step.getClass().getSimpleName(), e);
            }
        }
    }
}
```

### Example 5: Service Layer Exception Handling

**Location:** `product-service/src/main/java/com/ekart/product_service/service/OrderService.java`

**Proper Exception Structure:**

```java
@Service
public class OrderService {
    
    private static final Logger logger = LoggerFactory.getLogger(OrderService.class);
    
    @Autowired
    private ProductService productService;
    
    @Autowired
    private OrderSagaOrchestrator sagaOrchestrator;
    
    /**
     * Process checkout
     * 
     * Returns ApiResponse instead of throwing exceptions for expected failures
     * Throws exceptions only for unexpected errors
     */
    public ApiResponse<Map<String, Object>> checkout(Long buyerId, CheckoutRequest request) {
        try {
            logger.info("Processing checkout for buyer: {}", buyerId);
            
            // Validate cart items - return error response instead of exception
            for (CheckoutRequest.CartItemRequest item : request.getCartItems()) {
                ApiResponse<ProductDTO> productResponse = productService.getProductBySkuId(item.getSkuId());
                
                // Expected failure - return error response (not exception)
                if (!productResponse.isSuccess() || productResponse.getData() == null) {
                    return new ApiResponse<>(false, 
                        "Product not found: SKU " + item.getSkuId(), 
                        null);
                }
                
                ProductDTO product = productResponse.getData();
                
                // Expected failure - return error response (not exception)
                if (product.getAvailableQuantity() < item.getQuantity()) {
                    return new ApiResponse<>(false, 
                        String.format("Insufficient inventory for %s. Available: %d", 
                            product.getProductName(), 
                            product.getAvailableQuantity()), 
                        null);
                }
            }
            
            // Process order using Saga pattern
            // This can throw unchecked exceptions for unexpected errors
            SagaResult sagaResult = sagaOrchestrator.processOrder(buyerId, request);
            
            if (sagaResult.isSuccess()) {
                Map<String, Object> result = new HashMap<>();
                result.put("orderId", sagaResult.getContext().getOrderId());
                result.put("sagaId", sagaResult.getContext().getSagaId());
                return new ApiResponse<>(true, "Order processed successfully", result);
            } else {
                // Saga failed - return error response
                return new ApiResponse<>(false, sagaResult.getMessage(), null);
            }
            
        } catch (Exception e) {
            // Unexpected error - log and return error response
            logger.error("Unexpected error during checkout: buyerId={}", buyerId, e);
            return new ApiResponse<>(false, 
                "An unexpected error occurred. Please try again later.", 
                null);
        }
    }
}
```

### Example 6: Resilience Pattern Exception Handling

**Location:** `product-service/src/main/java/com/ekart/product_service/service/ResilientProductService.java`

**Handling Exceptions with Resilience Patterns:**

```java
@Service
public class ResilientProductService {
    
    private static final Logger logger = LoggerFactory.getLogger(ResilientProductService.class);
    
    @Autowired
    private RestTemplate restTemplate;
    
    @Value("${seller.service.url:http://localhost:8083}")
    private String sellerServiceUrl;
    
    /**
     * Fetch products with resilience patterns
     * 
     * Uses Circuit Breaker, Retry, Timeout, and Bulkhead
     * Handles various exceptions appropriately
     */
    @CircuitBreaker(name = "sellerService", fallbackMethod = "getProductsFallback")
    @Retry(name = "sellerService")
    @TimeLimiter(name = "sellerService")
    @Bulkhead(name = "sellerService")
    public CompletableFuture<List<ProductDTO>> getProducts(Long sellerId) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                String url = sellerServiceUrl + "/api/sellers/" + sellerId + "/products";
                ResponseEntity<List<ProductDTO>> response = restTemplate.exchange(
                    url, 
                    HttpMethod.GET, 
                    new HttpEntity<>(new HttpHeaders()), 
                    new ParameterizedTypeReference<List<ProductDTO>>() {}
                );
                
                return response.getBody();
            } catch (HttpClientErrorException e) {
                // Client error (4xx) - don't retry
                logger.warn("Client error fetching products: sellerId={}, status={}", 
                    sellerId, e.getStatusCode());
                throw new CompletionException("Client error: " + e.getMessage(), e);
            } catch (HttpServerErrorException e) {
                // Server error (5xx) - retry will handle
                logger.error("Server error fetching products: sellerId={}, status={}", 
                    sellerId, e.getStatusCode());
                throw new CompletionException("Server error: " + e.getMessage(), e);
            } catch (ResourceAccessException e) {
                // Network/timeout error - retry will handle
                logger.error("Network error fetching products: sellerId={}", sellerId, e);
                throw new CompletionException("Network error: " + e.getMessage(), e);
            } catch (Exception e) {
                // Unexpected error
                logger.error("Unexpected error fetching products: sellerId={}", sellerId, e);
                throw new CompletionException("Unexpected error: " + e.getMessage(), e);
            }
        });
    }
    
    /**
     * Fallback method when circuit breaker is open
     * 
     * Returns empty list instead of throwing exception
     */
    public CompletableFuture<List<ProductDTO>> getProductsFallback(Long sellerId, Exception e) {
        logger.warn("Circuit breaker open - using fallback: sellerId={}, error={}", 
            sellerId, e.getMessage());
        return CompletableFuture.completedFuture(new ArrayList<>());
    }
}
```

### Proper Exception Structure Guidelines

#### 1. Custom Exception Best Practices

```java
/**
 * ✅ GOOD: Well-structured custom exception
 */
public class ProductNotFoundException extends RuntimeException {
    private final Long productId;
    
    public ProductNotFoundException(Long productId) {
        super("Product not found: " + productId);
        this.productId = productId;
    }
    
    public ProductNotFoundException(Long productId, Throwable cause) {
        super("Product not found: " + productId, cause);
        this.productId = productId;
    }
    
    public Long getProductId() {
        return productId;
    }
}

/**
 * ❌ BAD: Poor exception structure
 */
public class BadException extends RuntimeException {
    // No context information
    // No meaningful message
    // No cause support
}
```

#### 2. Exception Hierarchy for E-Kart

```java
/**
 * Base exception for all business exceptions
 */
public abstract class EKartException extends RuntimeException {
    private final String errorCode;
    private final Map<String, Object> context;
    
    protected EKartException(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
        this.context = new HashMap<>();
    }
    
    protected EKartException(String errorCode, String message, Throwable cause) {
        super(message, cause);
        this.errorCode = errorCode;
        this.context = new HashMap<>();
    }
    
    public String getErrorCode() {
        return errorCode;
    }
    
    public EKartException withContext(String key, Object value) {
        this.context.put(key, value);
        return this;
    }
    
    public Map<String, Object> getContext() {
        return new HashMap<>(context);
    }
}

/**
 * Product-related exceptions
 */
public class ProductNotFoundException extends EKartException {
    public ProductNotFoundException(Long productId) {
        super("PRODUCT_NOT_FOUND", "Product not found: " + productId);
        withContext("productId", productId);
    }
}

public class InsufficientInventoryException extends EKartException {
    public InsufficientInventoryException(Long skuId, Integer requested, Integer available) {
        super("INSUFFICIENT_INVENTORY", 
            String.format("Insufficient inventory for SKU %d. Requested: %d, Available: %d", 
                skuId, requested, available));
        withContext("skuId", skuId);
        withContext("requested", requested);
        withContext("available", available);
    }
}

/**
 * Order-related exceptions
 */
public class OrderNotFoundException extends EKartException {
    public OrderNotFoundException(String orderId) {
        super("ORDER_NOT_FOUND", "Order not found: " + orderId);
        withContext("orderId", orderId);
    }
}

public class OrderProcessingException extends EKartException {
    public OrderProcessingException(String orderId, String reason) {
        super("ORDER_PROCESSING_FAILED", 
            "Order processing failed: " + orderId + ". Reason: " + reason);
        withContext("orderId", orderId);
        withContext("reason", reason);
    }
}

/**
 * Payment-related exceptions
 */
public class PaymentProcessingException extends EKartException {
    public PaymentProcessingException(String orderId, String reason) {
        super("PAYMENT_PROCESSING_FAILED", 
            "Payment processing failed for order: " + orderId + ". Reason: " + reason);
        withContext("orderId", orderId);
        withContext("reason", reason);
    }
}
```

### When to Use Checked vs Unchecked Exceptions in E-Kart

#### Use Checked Exceptions When:

1. **Recoverable Errors**
   ```java
   // Database connection can be retried
   public void connectToDatabase() throws SQLException {
       // Connection might fail but can be retried
   }
   ```

2. **External Dependencies**
   ```java
   // File operations can fail but are recoverable
   public void readConfiguration() throws IOException {
       // File might not exist, but can be handled
   }
   ```

3. **Saga Steps**
   ```java
   // Saga steps can fail and need compensation
   public interface SagaStep {
       void execute(SagaContext context) throws Exception;
   }
   ```

#### Use Unchecked Exceptions When:

1. **Programming Errors**
   ```java
   // Invalid input is a programming error
   public void validateProduct(Long productId) {
       if (productId == null || productId <= 0) {
           throw new IllegalArgumentException("Invalid product ID: " + productId);
       }
   }
   ```

2. **Business Logic Errors**
   ```java
   // Payment failure is a business error
   public void processPayment(Order order) {
       if (!paymentGateway.process(order)) {
           throw new PaymentProcessingException("Payment failed for order: " + order.getId());
       }
   }
   ```

3. **System/Infrastructure Issues**
   ```java
   // Lock acquisition failure is a system issue
   public void acquireLock(String key) {
       if (!lockService.tryLock(key)) {
           throw new LockAcquisitionException("Could not acquire lock: " + key);
       }
   }
   ```

4. **Event Publishing Failures**
   ```java
   // Event publishing failure shouldn't block business logic
   public void publishEvent(Event event) {
       try {
           kafkaTemplate.send("topic", event);
       } catch (Exception e) {
           throw new EventPublishException("Failed to publish event", e);
       }
   }
   ```

### Exception Handling Patterns in E-Kart

#### Pattern 1: Return Error Response (Expected Failures)

```java
// ✅ GOOD: Return error response for expected failures
public ApiResponse<ProductDTO> getProduct(Long skuId) {
    Product product = productRepository.findById(skuId).orElse(null);
    
    if (product == null) {
        // Expected failure - return error response
        return new ApiResponse<>(false, "Product not found", null);
    }
    
    return new ApiResponse<>(true, "Product found", convertToDTO(product));
}
```

#### Pattern 2: Throw Exception (Unexpected Errors)

```java
// ✅ GOOD: Throw exception for unexpected errors
public ProductDTO getProduct(Long skuId) {
    try {
        Product product = productRepository.findById(skuId)
            .orElseThrow(() -> new ProductNotFoundException(skuId));
        return convertToDTO(product);
    } catch (DataAccessException e) {
        // Unexpected database error
        logger.error("Database error fetching product: skuId={}", skuId, e);
        throw new ProductServiceException("Failed to fetch product", e);
    }
}
```

#### Pattern 3: Exception Chaining

```java
// ✅ GOOD: Chain exceptions to preserve context
public void processOrder(Order order) {
    try {
        processPayment(order);
    } catch (PaymentProcessingException e) {
        // Chain with order context
        throw new OrderProcessingException(order.getId(), e.getMessage(), e);
    }
}
```

#### Pattern 4: Global Exception Handler

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);
    
    @ExceptionHandler(ProductNotFoundException.class)
    public ResponseEntity<ApiResponse<?>> handleProductNotFound(ProductNotFoundException e) {
        logger.warn("Product not found: {}", e.getProductId());
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ApiResponse<>(false, e.getMessage(), null));
    }
    
    @ExceptionHandler(PaymentProcessingException.class)
    public ResponseEntity<ApiResponse<?>> handlePaymentError(PaymentProcessingException e) {
        logger.error("Payment processing failed: {}", e.getMessage(), e);
        return ResponseEntity.status(HttpStatus.PAYMENT_REQUIRED)
            .body(new ApiResponse<>(false, e.getMessage(), null));
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiResponse<?>> handleGenericException(Exception e) {
        logger.error("Unexpected error: {}", e.getMessage(), e);
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(new ApiResponse<>(false, "An unexpected error occurred", null));
    }
}
```

---

## Exception Handling Mechanisms

### 1. Try-Catch Block

**Basic Structure:**

```java
try {
    // Code that might throw exception
    riskyOperation();
} catch (ExceptionType e) {
    // Handle exception
    handleException(e);
} finally {
    // Always executes
    cleanup();
}
```

**Visual Flow:**

```
┌─────────────────────────────────────────┐
│           TRY-CATCH-FINALLY FLOW        │
└─────────────────────────────────────────┘

try {
    operation()
    │
    ├─ Success? ──Yes──→ finally → Continue
    │
    └─ Exception? ──Yes──→ catch → finally → Continue/Throw
}
```

**Example:**
```java
public void processFile(String filename) {
    FileReader reader = null;
    try {
        reader = new FileReader(filename);
        // Read file
        int data = reader.read();
    } catch (FileNotFoundException e) {
        System.err.println("File not found: " + filename);
        // Log or handle
    } catch (IOException e) {
        System.err.println("IO Error: " + e.getMessage());
    } finally {
        // Always executes, even if exception occurs
        if (reader != null) {
            try {
                reader.close();
            } catch (IOException e) {
                // Handle close exception
            }
        }
    }
}
```

### 2. Multiple Catch Blocks

**Order matters: Most specific to least specific**

```java
try {
    // Code
} catch (FileNotFoundException e) {
    // Most specific - handles file not found
} catch (IOException e) {
    // More general - handles all IO exceptions
} catch (Exception e) {
    // Most general - handles all exceptions
}
```

**⚠️ Important:** Order from specific to general!

```java
// ❌ WRONG - Won't compile
try {
    // Code
} catch (Exception e) {  // Too general first
} catch (IOException e) {  // Compiler error!
}

// ✅ CORRECT
try {
    // Code
} catch (IOException e) {  // Specific first
} catch (Exception e) {    // General last
}
```

### 3. Try-With-Resources (Java 7+)

**Automatic resource management**

```java
// Old way (manual cleanup)
FileReader reader = null;
try {
    reader = new FileReader("file.txt");
    // Use reader
} finally {
    if (reader != null) {
        reader.close();
    }
}

// New way (automatic cleanup)
try (FileReader reader = new FileReader("file.txt")) {
    // Use reader
    // Automatically closed, even if exception occurs
}
```

**Multiple Resources:**

```java
try (
    FileReader reader = new FileReader("input.txt");
    FileWriter writer = new FileWriter("output.txt");
    Connection conn = dataSource.getConnection()
) {
    // Use resources
    // All automatically closed in reverse order
}
```

**Custom Resources (AutoCloseable):**

```java
public class DatabaseConnection implements AutoCloseable {
    private Connection connection;
    
    public DatabaseConnection(String url) throws SQLException {
        this.connection = DriverManager.getConnection(url);
    }
    
    @Override
    public void close() throws SQLException {
        if (connection != null && !connection.isClosed()) {
            connection.close();
            System.out.println("Connection closed");
        }
    }
    
    public Connection getConnection() {
        return connection;
    }
}

// Usage
try (DatabaseConnection db = new DatabaseConnection("jdbc:...")) {
    Connection conn = db.getConnection();
    // Use connection
    // Automatically closed
}
```

**Suppressed Exceptions:**

```java
try (Resource1 r1 = new Resource1();
     Resource2 r2 = new Resource2()) {
    // If exception occurs here AND in close()
    // Original exception is primary
    // Close exceptions are suppressed
} catch (Exception e) {
    // Primary exception
    Throwable[] suppressed = e.getSuppressed();
    // Access suppressed exceptions
}
```

### 4. Throws Declaration

**Declare exceptions that method might throw**

```java
public void riskyMethod() throws IOException, SQLException {
    // Method declares it might throw these exceptions
    // Caller must handle them
}

// Multiple exceptions
public void complexMethod() 
    throws IOException, 
           SQLException, 
           ClassNotFoundException {
    // Multiple checked exceptions
}
```

**Throws vs Throw:**

```java
// throws - Declaration (method signature)
public void method() throws IOException {
    // Method might throw IOException
}

// throw - Statement (actually throw exception)
public void method() {
    throw new IOException("Error occurred");
}
```

---

## Advanced Exception Concepts

### 1. Exception Chaining

**Preserve original exception information**

```java
try {
    // Some operation
} catch (SQLException e) {
    // Wrap in higher-level exception
    throw new DataAccessException("Database error", e);
    // Original exception is preserved as cause
}
```

**Example:**

```java
public class ExceptionChaining {
    public void processData() throws BusinessException {
        try {
            readFromDatabase();
        } catch (SQLException e) {
            // Chain exceptions
            throw new BusinessException(
                "Failed to process data", 
                e  // Original exception as cause
            );
        }
    }
    
    private void readFromDatabase() throws SQLException {
        throw new SQLException("Connection failed");
    }
}

// Usage
try {
    processor.processData();
} catch (BusinessException e) {
    // Get root cause
    Throwable cause = e.getCause();
    if (cause instanceof SQLException) {
        SQLException sqlEx = (SQLException) cause;
        // Handle SQL exception
    }
}
```

**Visual Representation:**

```
BusinessException
    └─ cause: SQLException
           └─ cause: IOException
                  └─ cause: null
```

### 2. Custom Exceptions

**Creating your own exception classes**

```java
// Checked Exception
public class InsufficientFundsException extends Exception {
    private double balance;
    private double amount;
    
    public InsufficientFundsException(double balance, double amount) {
        super(String.format(
            "Insufficient funds. Balance: %.2f, Required: %.2f",
            balance, amount
        ));
        this.balance = balance;
        this.amount = amount;
    }
    
    public double getBalance() {
        return balance;
    }
    
    public double getAmount() {
        return amount;
    }
}

// Unchecked Exception
public class InvalidOrderException extends RuntimeException {
    private String orderId;
    
    public InvalidOrderException(String orderId, String message) {
        super(message);
        this.orderId = orderId;
    }
    
    public String getOrderId() {
        return orderId;
    }
}

// Usage
public void withdraw(double amount) throws InsufficientFundsException {
    if (amount > balance) {
        throw new InsufficientFundsException(balance, amount);
    }
    balance -= amount;
}
```

**Best Practices for Custom Exceptions:**

```java
// ✅ DO:
// 1. Provide meaningful messages
public class UserNotFoundException extends Exception {
    public UserNotFoundException(Long userId) {
        super("User not found with ID: " + userId);
    }
}

// 2. Include context information
public class ValidationException extends Exception {
    private String field;
    private Object value;
    
    public ValidationException(String field, Object value, String message) {
        super(message);
        this.field = field;
        this.value = value;
    }
}

// 3. Preserve cause
public class ServiceException extends Exception {
    public ServiceException(String message, Throwable cause) {
        super(message, cause);
    }
}

// ❌ DON'T:
// 1. Empty exception classes
public class MyException extends Exception {
    // No additional information
}

// 2. Generic names
public class Error extends Exception { }  // Too generic

// 3. Expose sensitive information
public class DatabaseException extends Exception {
    public DatabaseException(String password) {  // ❌ Don't expose passwords
        super("Database error: " + password);
    }
}
```

### 3. Exception Translation

**Convert low-level exceptions to high-level**

```java
public class DataAccessLayer {
    // Low-level SQLException
    public User findUser(Long id) throws SQLException {
        // Database operation
    }
    
    // Translate to business exception
    public User getUser(Long id) throws UserNotFoundException {
        try {
            return findUser(id);
        } catch (SQLException e) {
            // Translate to business exception
            throw new UserNotFoundException(id, e);
        }
    }
}

// Business layer doesn't know about SQLException
public class UserService {
    public User getUser(Long id) throws UserNotFoundException {
        return dataAccess.getUser(id);
        // Only deals with business exceptions
    }
}
```

### 4. Exception Suppression

**Suppress exceptions in finally block**

```java
// Problem: Exception in finally can mask original exception
public void problematic() {
    FileReader reader = null;
    try {
        reader = new FileReader("file.txt");
        throw new IOException("Read error");
    } finally {
        if (reader != null) {
            reader.close(); // If this throws, original exception is lost!
        }
    }
}

// Solution 1: Try-with-resources (automatic suppression)
public void better() {
    try (FileReader reader = new FileReader("file.txt")) {
        throw new IOException("Read error");
        // Close exception is suppressed
    }
}

// Solution 2: Manual suppression
public void manualSuppression() {
    FileReader reader = null;
    IOException readException = null;
    try {
        reader = new FileReader("file.txt");
        throw new IOException("Read error");
    } catch (IOException e) {
        readException = e;
        throw e;
    } finally {
        if (reader != null) {
            try {
                reader.close();
            } catch (IOException closeException) {
                if (readException != null) {
                    readException.addSuppressed(closeException);
                } else {
                    throw closeException;
                }
            }
        }
    }
}
```

---

## Best Practices

### 1. Exception Handling Best Practices

**✅ DO:**

```java
// 1. Catch specific exceptions
try {
    // Code
} catch (FileNotFoundException e) {
    // Handle specific case
} catch (IOException e) {
    // Handle general case
}

// 2. Log exceptions properly
catch (Exception e) {
    logger.error("Error processing file: {}", filename, e);
    // Include context and exception
}

// 3. Preserve stack trace
catch (Exception e) {
    throw new BusinessException("Error occurred", e);
    // Don't lose original exception
}

// 4. Use try-with-resources
try (Resource r = new Resource()) {
    // Use resource
}

// 5. Clean up in finally
try {
    // Code
} finally {
    cleanup(); // Always executes
}
```

**❌ DON'T:**

```java
// 1. Don't catch and ignore
try {
    riskyOperation();
} catch (Exception e) {
    // Empty catch - bad!
}

// 2. Don't catch generic Exception
try {
    // Code
} catch (Exception e) {
    // Too broad - catches everything
}

// 3. Don't expose sensitive information
catch (SQLException e) {
    throw new RuntimeException(e.getMessage());
    // Might expose database structure
}

// 4. Don't use exceptions for control flow
// ❌ BAD
try {
    while (true) {
        array[i++];
    }
} catch (ArrayIndexOutOfBoundsException e) {
    // Using exception for normal flow
}

// ✅ GOOD
for (int i = 0; i < array.length; i++) {
    // Normal loop
}

// 5. Don't throw RuntimeException without reason
public void method() {
    throw new RuntimeException("Error");
    // Use checked exception or specific unchecked
}
```

### 2. Exception Design Principles

**Principle 1: Fail Fast**

```java
// ✅ Fail fast - detect errors early
public void processOrder(Order order) {
    if (order == null) {
        throw new IllegalArgumentException("Order cannot be null");
    }
    if (order.getItems().isEmpty()) {
        throw new IllegalStateException("Order must have items");
    }
    // Process order
}
```

**Principle 2: Fail Explicitly**

```java
// ✅ Clear exception messages
throw new InsufficientFundsException(
    balance, 
    amount,
    "Cannot withdraw " + amount + ". Available: " + balance
);

// ❌ Vague messages
throw new Exception("Error");
```

**Principle 3: Fail Safely**

```java
// ✅ Handle exceptions gracefully
public List<String> readLines(String filename) {
    try {
        return Files.readAllLines(Paths.get(filename));
    } catch (IOException e) {
        logger.warn("Failed to read file: {}", filename, e);
        return Collections.emptyList(); // Return safe default
    }
}
```

---

## Common Patterns

### Pattern 1: Retry Pattern

```java
public class RetryPattern {
    public <T> T executeWithRetry(
            Callable<T> operation,
            int maxRetries,
            long delayMs) throws Exception {
        
        Exception lastException = null;
        
        for (int attempt = 0; attempt < maxRetries; attempt++) {
            try {
                return operation.call();
            } catch (Exception e) {
                lastException = e;
                if (attempt < maxRetries - 1) {
                    Thread.sleep(delayMs);
                    logger.warn("Retry attempt {} after error", attempt + 1, e);
                }
            }
        }
        
        throw new RetryException("Failed after " + maxRetries + " attempts", lastException);
    }
}

// Usage
String result = retryPattern.executeWithRetry(
    () -> httpClient.get(url),
    3,  // max retries
    1000  // delay between retries
);
```

### Pattern 2: Circuit Breaker Pattern

```java
public class CircuitBreaker {
    private enum State { CLOSED, OPEN, HALF_OPEN }
    
    private State state = State.CLOSED;
    private int failureCount = 0;
    private long lastFailureTime = 0;
    private final int threshold = 5;
    private final long timeout = 60000; // 1 minute
    
    public <T> T execute(Callable<T> operation) throws Exception {
        if (state == State.OPEN) {
            if (System.currentTimeMillis() - lastFailureTime > timeout) {
                state = State.HALF_OPEN;
            } else {
                throw new CircuitBreakerOpenException("Circuit breaker is OPEN");
            }
        }
        
        try {
            T result = operation.call();
            onSuccess();
            return result;
        } catch (Exception e) {
            onFailure();
            throw e;
        }
    }
    
    private void onSuccess() {
        failureCount = 0;
        state = State.CLOSED;
    }
    
    private void onFailure() {
        failureCount++;
        lastFailureTime = System.currentTimeMillis();
        if (failureCount >= threshold) {
            state = State.OPEN;
        }
    }
}
```

### Pattern 3: Exception Handler Chain

```java
public abstract class ExceptionHandler {
    protected ExceptionHandler next;
    
    public void setNext(ExceptionHandler next) {
        this.next = next;
    }
    
    public abstract void handle(Exception e);
}

public class SQLExceptionHandler extends ExceptionHandler {
    @Override
    public void handle(Exception e) {
        if (e instanceof SQLException) {
            // Handle SQL exception
            logger.error("SQL Error", e);
            // Retry or fallback
        } else if (next != null) {
            next.handle(e);
        }
    }
}

public class IOExceptionHandler extends ExceptionHandler {
    @Override
    public void handle(Exception e) {
        if (e instanceof IOException) {
            // Handle IO exception
            logger.error("IO Error", e);
        } else if (next != null) {
            next.handle(e);
        }
    }
}

// Usage
ExceptionHandler chain = new SQLExceptionHandler();
chain.setNext(new IOExceptionHandler());
chain.setNext(new DefaultExceptionHandler());

try {
    // Operation
} catch (Exception e) {
    chain.handle(e);
}
```

---

## Performance Considerations

### 1. Exception Overhead

**Exceptions are expensive!**

```java
// ❌ BAD - Using exceptions for control flow
public int findIndex(int[] array, int value) {
    try {
        int i = 0;
        while (true) {
            if (array[i] == value) return i;
            i++;
        }
    } catch (ArrayIndexOutOfBoundsException e) {
        return -1;
    }
}

// ✅ GOOD - Normal control flow
public int findIndex(int[] array, int value) {
    for (int i = 0; i < array.length; i++) {
        if (array[i] == value) return i;
    }
    return -1;
}
```

**Performance Comparison:**

```
Normal return:     ~1-2 nanoseconds
Throwing exception: ~1-5 microseconds (1000x slower!)
```

### 2. Stack Trace Generation

**Stack traces are expensive to generate**

```java
// ❌ Creates full stack trace immediately
throw new Exception("Error");

// ✅ Lazy stack trace (Java 7+)
throw new Exception("Error") {
    @Override
    public synchronized Throwable fillInStackTrace() {
        return this; // Skip stack trace generation
    }
};

// Or use static exceptions for known cases
public class KnownException extends Exception {
    @Override
    public synchronized Throwable fillInStackTrace() {
        return this; // No stack trace needed
    }
}
```

### 3. Exception Pooling

**Reuse exception objects for known cases**

```java
public class ExceptionPool {
    private static final Exception KNOWN_EXCEPTION = 
        new KnownException("Known error") {
            @Override
            public synchronized Throwable fillInStackTrace() {
                return this;
            }
        };
    
    public static Exception getKnownException() {
        return KNOWN_EXCEPTION;
    }
}
```

---

## Interview Questions

### Q1: What is the difference between Error and Exception?

**Answer:**
"Error and Exception both extend Throwable, but serve different purposes:

**Error:**
- Represents serious problems that applications should NOT catch
- Indicates JVM/system-level problems
- Examples: OutOfMemoryError, StackOverflowError
- Usually cannot be recovered from
- Unchecked (no compile-time checking)

**Exception:**
- Represents problems that applications MIGHT want to catch
- Indicates application-level problems
- Can be checked (must handle) or unchecked (RuntimeException)
- Usually can be recovered from
- Examples: IOException, NullPointerException

In practice, I never catch Error subclasses as they indicate serious JVM problems that typically require system-level intervention."

### Q2: Explain checked vs unchecked exceptions.

**Answer:**
"Checked exceptions are verified at compile-time, while unchecked exceptions are not.

**Checked Exceptions:**
- Must be declared in method signature with `throws`
- Must be handled with try-catch or re-thrown
- Extend Exception (but not RuntimeException)
- Examples: IOException, SQLException
- Indicate recoverable conditions

**Unchecked Exceptions:**
- No compile-time checking
- Optional to handle
- Extend RuntimeException
- Examples: NullPointerException, IllegalArgumentException
- Usually indicate programming errors

I use checked exceptions for recoverable conditions (like file I/O) and unchecked exceptions for programming errors (like invalid arguments)."

### Q3: What happens if an exception is thrown in finally block?

**Answer:**
"If an exception is thrown in a finally block, it can mask the original exception. However, with try-with-resources (Java 7+), exceptions thrown during resource closing are automatically suppressed and added to the primary exception's suppressed exceptions list.

**Example:**
```java
try {
    throw new IOException("Original");
} finally {
    throw new SQLException("Finally"); // This masks original!
}
// Only SQLException is propagated

// With try-with-resources:
try (Resource r = new Resource()) {
    throw new IOException("Original");
}
// IOException is primary, close() exceptions are suppressed
```

I always use try-with-resources for automatic resource management to avoid this issue."

### Q4: Explain exception chaining.

**Answer:**
"Exception chaining preserves the original exception when wrapping it in a higher-level exception. This is done using the `cause` parameter in exception constructors.

**Example:**
```java
try {
    databaseOperation();
} catch (SQLException e) {
    throw new BusinessException("Data access failed", e);
    // Original SQLException is preserved as cause
}

// Later, can access root cause:
BusinessException be = ...;
Throwable cause = be.getCause(); // SQLException
```

This is crucial for debugging as it maintains the full exception chain. I always preserve the original exception when translating between layers (e.g., SQLException → BusinessException)."

### Q5: When should you create custom exceptions?

**Answer:**
"I create custom exceptions when:
1. **Domain-specific errors**: Business logic errors (InsufficientFundsException)
2. **Better context**: Need additional information beyond standard exceptions
3. **Layer abstraction**: Hide implementation details (SQLException → DataAccessException)
4. **API clarity**: Make API contracts explicit

**Example:**
```java
public class OrderProcessingException extends Exception {
    private String orderId;
    private OrderStatus currentStatus;
    
    public OrderProcessingException(String orderId, OrderStatus status, String message) {
        super(message);
        this.orderId = orderId;
        this.currentStatus = status;
    }
}
```

I follow these principles:
- Meaningful names ending with 'Exception'
- Include relevant context (IDs, state, etc.)
- Preserve cause when wrapping
- Use checked for recoverable, unchecked for programming errors"

### Q6: What is try-with-resources and how does it work?

**Answer:**
"Try-with-resources (Java 7+) automatically manages resources that implement AutoCloseable. Resources are automatically closed, even if exceptions occur.

**Key points:**
- Resources must implement AutoCloseable interface
- Resources are closed in reverse order of declaration
- Exceptions in close() are suppressed if primary exception exists
- More concise than manual try-finally

**Example:**
```java
try (FileReader reader = new FileReader("file.txt");
     FileWriter writer = new FileWriter("output.txt")) {
    // Use resources
    // Both automatically closed
}
```

I use try-with-resources for all resource management (files, connections, streams) as it's safer and cleaner than manual cleanup."

### Q7: Explain the exception handling hierarchy in catch blocks.

**Answer:**
"Catch blocks must be ordered from most specific to least specific. The compiler enforces this to ensure specific exceptions are caught before general ones.

**Rules:**
1. More specific exceptions must come first
2. Cannot catch a subclass after catching its superclass
3. Compiler error if order is wrong

**Example:**
```java
// ✅ Correct order
try {
    // Code
} catch (FileNotFoundException e) {  // Specific
    // Handle file not found
} catch (IOException e) {            // More general
    // Handle other IO errors
} catch (Exception e) {              // Most general
    // Handle any other exception
}

// ❌ Wrong - won't compile
try {
    // Code
} catch (Exception e) {              // Too general first
} catch (IOException e) {            // Compiler error!
}
```

I always order catch blocks from specific to general to handle exceptions appropriately."

### Q8: What are suppressed exceptions?

**Answer:**
"Suppressed exceptions are exceptions that occur during resource cleanup (in close() method) when using try-with-resources. They're automatically added to the primary exception's suppressed exceptions list.

**Example:**
```java
try (Resource r = new Resource()) {
    throw new IOException("Primary exception");
    // If close() also throws SQLException, it's suppressed
} catch (IOException e) {
    Throwable[] suppressed = e.getSuppressed();
    // Access suppressed exceptions
}
```

This ensures the primary exception isn't lost. I can access suppressed exceptions using `getSuppressed()` method when needed for logging or debugging."

### Q9: How do you handle exceptions in a multi-threaded environment?

**Answer:**
"In multi-threaded environments, exceptions in one thread don't affect others. However, I need to handle exceptions properly:

**Key considerations:**
1. **Uncaught exceptions**: Use UncaughtExceptionHandler
2. **ExecutorService**: Exceptions in tasks are wrapped in ExecutionException
3. **Thread pools**: Handle exceptions in submitted tasks
4. **Thread interruption**: Handle InterruptedException properly

**Example:**
```java
// Uncaught exception handler
Thread.setDefaultUncaughtExceptionHandler((t, e) -> {
    logger.error("Uncaught exception in thread: {}", t.getName(), e);
});

// ExecutorService
ExecutorService executor = Executors.newFixedThreadPool(10);
Future<String> future = executor.submit(() -> {
    // Might throw exception
    return process();
});

try {
    String result = future.get();
} catch (ExecutionException e) {
    Throwable cause = e.getCause(); // Original exception
    // Handle
}
```

I always set uncaught exception handlers and properly handle ExecutionException from futures."

### Q10: What is the performance impact of exceptions?

**Answer:**
"Exceptions have significant performance overhead:

**Costs:**
1. **Stack trace generation**: Most expensive part (~1-5 microseconds)
2. **Exception object creation**: Memory allocation
3. **Unwinding stack**: Finding appropriate catch block

**Best practices:**
1. Don't use exceptions for control flow
2. Use lazy stack traces for known exceptions
3. Prefer return codes for expected conditions
4. Cache exception objects for known cases

**Example:**
```java
// ❌ Bad - exception for control flow
try {
    while (true) array[i++];
} catch (IndexOutOfBoundsException e) { }

// ✅ Good - normal control flow
for (int i = 0; i < array.length; i++) { }
```

I only use exceptions for exceptional conditions, not for normal program flow."

---

## Quick Reference

### Exception Hierarchy
```
Throwable
├── Error (Unchecked)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── ...
└── Exception
    ├── RuntimeException (Unchecked)
    │   ├── NullPointerException
    │   ├── IllegalArgumentException
    │   └── ...
    └── Checked Exceptions
        ├── IOException
        ├── SQLException
        └── ...
```

### Key Methods
```java
// Throwable methods
getMessage()           // Exception message
getCause()             // Original exception
getStackTrace()        // Stack trace array
printStackTrace()      // Print stack trace
initCause(Throwable)   // Set cause
getSuppressed()        // Suppressed exceptions (Java 7+)
```

### Best Practices Checklist
- ✅ Catch specific exceptions first
- ✅ Use try-with-resources
- ✅ Preserve exception chain
- ✅ Log with context
- ✅ Don't ignore exceptions
- ✅ Don't use exceptions for control flow
- ✅ Create meaningful custom exceptions
- ✅ Fail fast and explicitly

---

## Advanced Exception Scenarios

### 1. Exception in Constructor

**Problem:** What happens if exception is thrown in constructor?

```java
public class ResourceManager {
    private FileWriter writer;
    
    public ResourceManager(String filename) throws IOException {
        this.writer = new FileWriter(filename);
        // If exception here, object is not fully constructed
        // But partially constructed object exists
    }
}

// Solution 1: Factory method with validation
public class ResourceManager {
    private FileWriter writer;
    
    private ResourceManager(FileWriter writer) {
        this.writer = writer;
    }
    
    public static ResourceManager create(String filename) throws IOException {
        FileWriter writer = new FileWriter(filename);
        try {
            // Validate
            return new ResourceManager(writer);
        } catch (Exception e) {
            writer.close(); // Cleanup
            throw e;
        }
    }
}

// Solution 2: Try-with-resources in factory
public static ResourceManager createSafe(String filename) throws IOException {
    FileWriter writer = new FileWriter(filename);
    try {
        return new ResourceManager(writer);
    } catch (Exception e) {
        try (FileWriter w = writer) {
            // Auto-close on exception
        }
        throw e;
    }
}
```

### 2. Exception in Static Initializer

**Static initializers can throw exceptions**

```java
public class ConfigLoader {
    static {
        try {
            loadConfiguration();
        } catch (IOException e) {
            // Static initializer can only throw ExceptionInInitializerError
            throw new ExceptionInInitializerError(e);
        }
    }
    
    private static void loadConfiguration() throws IOException {
        // Load config
    }
}

// Catching ExceptionInInitializerError
try {
    ConfigLoader loader = new ConfigLoader();
} catch (ExceptionInInitializerError e) {
    Throwable cause = e.getCause(); // Original exception
    // Handle
}
```

### 3. Exception in Finalizer

**Finalizers can throw exceptions (but they're ignored!)**

```java
public class Resource {
    @Override
    protected void finalize() throws Throwable {
        try {
            cleanup();
        } catch (Exception e) {
            // Exceptions in finalizer are IGNORED!
            // They don't propagate
            logger.error("Error in finalizer", e);
        } finally {
            super.finalize();
        }
    }
}

// ⚠️ Important: Don't rely on finalizers for cleanup!
// Use try-with-resources instead
```

### 4. Exception in Lambda Expressions

**Handling exceptions in lambdas**

```java
// Problem: Checked exceptions in lambda
List<String> files = Arrays.asList("file1.txt", "file2.txt");
files.forEach(file -> {
    try {
        Files.readAllLines(Paths.get(file)); // IOException
    } catch (IOException e) {
        throw new RuntimeException(e); // Wrap unchecked
    }
});

// Solution: Extract to method
files.forEach(this::processFile);

private void processFile(String file) {
    try {
        Files.readAllLines(Paths.get(file));
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}

// Solution: Custom functional interface
@FunctionalInterface
public interface ThrowingConsumer<T, E extends Exception> {
    void accept(T t) throws E;
}

public static <T> Consumer<T> unchecked(ThrowingConsumer<T, Exception> consumer) {
    return t -> {
        try {
            consumer.accept(t);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    };
}

// Usage
files.forEach(unchecked(file -> Files.readAllLines(Paths.get(file))));
```

### 5. Exception in Stream Operations

**Exception handling in streams**

```java
// Problem: Checked exceptions in stream
List<String> results = files.stream()
    .map(file -> Files.readAllLines(Paths.get(file))) // IOException
    .collect(Collectors.toList());

// Solution 1: Wrap in method
List<String> results = files.stream()
    .map(this::readFileSafely)
    .filter(Objects::nonNull)
    .collect(Collectors.toList());

private List<String> readFileSafely(String file) {
    try {
        return Files.readAllLines(Paths.get(file));
    } catch (IOException e) {
        logger.error("Failed to read: {}", file, e);
        return null;
    }
}

// Solution 2: Custom collector with exception handling
public static <T, R> Function<T, Optional<R>> safe(Function<T, R> function) {
    return t -> {
        try {
            return Optional.of(function.apply(t));
        } catch (Exception e) {
            logger.error("Error processing: {}", t, e);
            return Optional.empty();
        }
    };
}

// Usage
List<String> results = files.stream()
    .map(safe(file -> Files.readAllLines(Paths.get(file))))
    .filter(Optional::isPresent)
    .map(Optional::get)
    .flatMap(List::stream)
    .collect(Collectors.toList());
```

### 6. Exception Propagation in Threads

**How exceptions propagate in multi-threading**

```java
// Uncaught exceptions in threads
Thread thread = new Thread(() -> {
    throw new RuntimeException("Thread exception");
});

// Default: Prints to console and thread dies
thread.start();

// Custom handler
Thread.setDefaultUncaughtExceptionHandler((t, e) -> {
    logger.error("Uncaught exception in thread: {}", t.getName(), e);
    // Send alert, log, etc.
});

// ExecutorService exceptions
ExecutorService executor = Executors.newFixedThreadPool(10);

// submit() - exceptions wrapped in ExecutionException
Future<String> future = executor.submit(() -> {
    throw new IOException("Task failed");
});

try {
    future.get(); // Throws ExecutionException
} catch (ExecutionException e) {
    Throwable cause = e.getCause(); // Original IOException
}

// execute() - exceptions go to UncaughtExceptionHandler
executor.execute(() -> {
    throw new RuntimeException("Task failed");
    // Goes to UncaughtExceptionHandler
});
```

### 7. Exception in CompletableFuture

**Exception handling in async operations**

```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> {
        if (random.nextBoolean()) {
            throw new RuntimeException("Random failure");
        }
        return "Success";
    })
    .exceptionally(throwable -> {
        // Handle exception, return default value
        logger.error("Async operation failed", throwable);
        return "Default value";
    })
    .thenApply(result -> "Processed: " + result);

// Multiple exception handlers
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> operation())
    .handle((result, throwable) -> {
        if (throwable != null) {
            // Handle exception
            return "Error occurred";
        }
        return result;
    });

// Exception composition
CompletableFuture<String> future1 = CompletableFuture
    .supplyAsync(() -> "Result 1");

CompletableFuture<String> future2 = CompletableFuture
    .supplyAsync(() -> {
        throw new RuntimeException("Failure");
    });

CompletableFuture<String> combined = future1
    .thenCombine(future2, (r1, r2) -> r1 + r2)
    .exceptionally(throwable -> {
        // Handles exception from either future
        return "Fallback";
    });
```

---

## Exception Handling in Frameworks

### 1. Spring Framework Exception Handling

**@ControllerAdvice for global exception handling**

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(
            ResourceNotFoundException e) {
        ErrorResponse error = new ErrorResponse(
            "NOT_FOUND",
            e.getMessage(),
            System.currentTimeMillis()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
            ValidationException e) {
        ErrorResponse error = new ErrorResponse(
            "VALIDATION_ERROR",
            e.getMessage(),
            e.getField(),
            System.currentTimeMillis()
        );
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception e) {
        logger.error("Unexpected error", e);
        ErrorResponse error = new ErrorResponse(
            "INTERNAL_ERROR",
            "An unexpected error occurred",
            System.currentTimeMillis()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(error);
    }
}

// Controller
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        if (user == null) {
            throw new ResourceNotFoundException("User not found: " + id);
        }
        return ResponseEntity.ok(user);
    }
}
```

### 2. Transaction Exception Handling

**Exception handling in transactional methods**

```java
@Service
@Transactional
public class OrderService {
    
    // RuntimeException rolls back transaction
    public void createOrder(Order order) {
        // If RuntimeException thrown, transaction rolls back
        if (order == null) {
            throw new IllegalArgumentException("Order cannot be null");
        }
        orderRepository.save(order);
    }
    
    // Checked exception does NOT roll back by default
    public void processOrder(Order order) throws IOException {
        // IOException does NOT roll back transaction
        // Transaction commits even if IOException thrown
    }
    
    // Configure rollback behavior
    @Transactional(rollbackFor = IOException.class)
    public void processOrderWithRollback(Order order) throws IOException {
        // Now IOException will roll back transaction
    }
    
    @Transactional(noRollbackFor = BusinessException.class)
    public void processOrderNoRollback(Order order) {
        // BusinessException will NOT roll back
        throw new BusinessException("Business rule violation");
    }
}
```

### 3. JPA Exception Handling

**Converting JPA exceptions to domain exceptions**

```java
@Repository
public class UserRepository {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    public User findById(Long id) {
        try {
            return entityManager.find(User.class, id);
        } catch (EntityNotFoundException e) {
            throw new UserNotFoundException(id, e);
        } catch (PersistenceException e) {
            // Handle database errors
            if (e.getCause() instanceof SQLException) {
                SQLException sqlEx = (SQLException) e.getCause();
                throw new DataAccessException("Database error", sqlEx);
            }
            throw new DataAccessException("Persistence error", e);
        }
    }
    
    public User save(User user) {
        try {
            if (user.getId() == null) {
                entityManager.persist(user);
            } else {
                return entityManager.merge(user);
            }
            return user;
        } catch (EntityExistsException e) {
            throw new DuplicateUserException(user.getEmail(), e);
        } catch (OptimisticLockException e) {
            throw new ConcurrentModificationException(
                "User was modified by another transaction", e);
        }
    }
}
```

---

## Exception Testing

### 1. Testing Exceptions with JUnit

```java
@Test
public void testExceptionThrown() {
    // JUnit 4
    try {
        service.riskyOperation();
        fail("Should have thrown exception");
    } catch (ExpectedException e) {
        assertEquals("Expected message", e.getMessage());
    }
    
    // JUnit 4 with @Test(expected)
    @Test(expected = IllegalArgumentException.class)
    public void testExceptionExpected() {
        service.riskyOperation();
    }
    
    // JUnit 5
    @Test
    void testExceptionThrown() {
        IllegalArgumentException exception = assertThrows(
            IllegalArgumentException.class,
            () -> service.riskyOperation()
        );
        assertEquals("Expected message", exception.getMessage());
    }
    
    // JUnit 5 with assertThrows
    @Test
    void testExceptionWithMessage() {
        Exception exception = assertThrows(
            IllegalArgumentException.class,
            () -> service.riskyOperation(),
            "Should throw IllegalArgumentException"
        );
        assertTrue(exception.getMessage().contains("Expected"));
    }
}
```

### 2. Testing Exception Scenarios

```java
public class ExceptionTestScenarios {
    
    @Test
    void testExceptionInTryFinally() {
        List<String> operations = new ArrayList<>();
        
        try {
            operations.add("try-start");
            throw new RuntimeException("Error");
        } catch (RuntimeException e) {
            operations.add("catch");
            throw e; // Re-throw
        } finally {
            operations.add("finally");
        }
        
        // Finally always executes
        assertTrue(operations.contains("finally"));
    }
    
    @Test
    void testExceptionSuppression() {
        try (AutoCloseableResource resource = new AutoCloseableResource()) {
            throw new IOException("Primary exception");
        } catch (IOException e) {
            // Check suppressed exceptions
            Throwable[] suppressed = e.getSuppressed();
            assertEquals(1, suppressed.length);
            assertTrue(suppressed[0] instanceof SQLException);
        }
    }
    
    @Test
    void testExceptionChaining() {
        try {
            service.processData();
        } catch (BusinessException e) {
            // Check cause
            Throwable cause = e.getCause();
            assertTrue(cause instanceof SQLException);
            
            // Check message
            assertTrue(e.getMessage().contains("Failed"));
        }
    }
}
```

---

## Exception Monitoring and Logging

### 1. Structured Exception Logging

```java
public class ExceptionLogger {
    private static final Logger logger = LoggerFactory.getLogger(ExceptionLogger.class);
    
    public void logException(Exception e, Map<String, Object> context) {
        MDC.put("exception.type", e.getClass().getSimpleName());
        MDC.put("exception.message", e.getMessage());
        
        if (context != null) {
            context.forEach((key, value) -> MDC.put("context." + key, String.valueOf(value)));
        }
        
        logger.error("Exception occurred: {}", e.getMessage(), e);
        
        // Clear MDC
        MDC.clear();
    }
    
    public void logExceptionWithMetrics(Exception e, String operation) {
        // Log exception
        logger.error("Operation failed: {}", operation, e);
        
        // Record metrics
        meterRegistry.counter("exception.count", 
            "type", e.getClass().getSimpleName(),
            "operation", operation
        ).increment();
    }
}
```

### 2. Exception Alerting

```java
@Component
public class ExceptionAlertService {
    
    @Value("${alert.threshold:10}")
    private int alertThreshold;
    
    private final Map<String, AtomicInteger> exceptionCounts = new ConcurrentHashMap<>();
    
    public void handleException(Exception e, String context) {
        String key = e.getClass().getSimpleName() + ":" + context;
        int count = exceptionCounts.computeIfAbsent(
            key, k -> new AtomicInteger(0)
        ).incrementAndGet();
        
        if (count >= alertThreshold) {
            sendAlert(e, context, count);
            exceptionCounts.put(key, new AtomicInteger(0)); // Reset
        }
    }
    
    private void sendAlert(Exception e, String context, int count) {
        Alert alert = Alert.builder()
            .severity("HIGH")
            .message(String.format(
                "Exception %s occurred %d times in context %s",
                e.getClass().getSimpleName(), count, context
            ))
            .exception(e)
            .timestamp(Instant.now())
            .build();
        
        alertService.send(alert);
    }
}
```

---

## Real-World Exception Patterns

### Pattern 1: Result Wrapper Pattern

**Avoid exceptions for expected failures**

```java
public class Result<T> {
    private final T value;
    private final Exception error;
    private final boolean success;
    
    private Result(T value, Exception error, boolean success) {
        this.value = value;
        this.error = error;
        this.success = success;
    }
    
    public static <T> Result<T> success(T value) {
        return new Result<>(value, null, true);
    }
    
    public static <T> Result<T> failure(Exception error) {
        return new Result<>(null, error, false);
    }
    
    public boolean isSuccess() {
        return success;
    }
    
    public T getValue() {
        if (!success) {
            throw new IllegalStateException("Cannot get value from failed result");
        }
        return value;
    }
    
    public Exception getError() {
        return error;
    }
    
    public <R> Result<R> map(Function<T, R> mapper) {
        if (success) {
            try {
                return Result.success(mapper.apply(value));
            } catch (Exception e) {
                return Result.failure(e);
            }
        }
        return Result.failure(error);
    }
}

// Usage
Result<String> result = service.processData();
if (result.isSuccess()) {
    String data = result.getValue();
} else {
    Exception error = result.getError();
    // Handle error
}
```

### Pattern 2: Either Pattern (Functional Style)

```java
public class Either<L, R> {
    private final L left;
    private final R right;
    private final boolean isRight;
    
    private Either(L left, R right, boolean isRight) {
        this.left = left;
        this.right = right;
        this.isRight = isRight;
    }
    
    public static <L, R> Either<L, R> left(L value) {
        return new Either<>(value, null, false);
    }
    
    public static <L, R> Either<L, R> right(R value) {
        return new Either<>(null, value, true);
    }
    
    public boolean isRight() {
        return isRight;
    }
    
    public R getRight() {
        if (!isRight) {
            throw new IllegalStateException("Not a right value");
        }
        return right;
    }
    
    public L getLeft() {
        if (isRight) {
            throw new IllegalStateException("Not a left value");
        }
        return left;
    }
    
    public <T> Either<L, T> map(Function<R, T> mapper) {
        if (isRight) {
            return Either.right(mapper.apply(right));
        }
        return Either.left(left);
    }
}

// Usage: Either<Exception, String>
Either<Exception, String> result = service.process();
if (result.isRight()) {
    String value = result.getRight();
} else {
    Exception error = result.getLeft();
}
```

---

## Quick Reference Summary

### Exception Types Summary

| Type | Checked? | When to Use | Examples |
|------|----------|-------------|----------|
| **Error** | No | JVM problems | OutOfMemoryError |
| **RuntimeException** | No | Programming errors | NullPointerException |
| **Checked Exception** | Yes | Recoverable errors | IOException |

### Exception Handling Checklist

- ✅ Catch specific exceptions first
- ✅ Use try-with-resources
- ✅ Preserve exception chain
- ✅ Log with context
- ✅ Don't ignore exceptions
- ✅ Don't use for control flow
- ✅ Create meaningful custom exceptions
- ✅ Handle in appropriate layer

### Performance Tips

- ⚡ Exceptions are expensive (~1-5 microseconds)
- ⚡ Don't use for control flow
- ⚡ Use lazy stack traces when possible
- ⚡ Cache exception objects for known cases

---

**Happy Coding! 🚨**


