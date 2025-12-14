# Amazon Interview Problems - Real Questions & Solutions

> **Curated from actual Amazon staff engineer interviews**

This guide contains real system design and low-level design problems asked in Amazon interviews.

---

## System Design Problems

### Problem 1: Design Amazon's Shopping Cart

**Problem Statement:**
Design a shopping cart system that can handle millions of users adding/removing items.

**Requirements:**
- 100M users
- 10M active carts
- Add/remove items
- Save for later
- Cart expiration (30 days)
- Real-time updates

**Solution:**

```java
// Cart Service
@Service
public class CartService {
    @Autowired
    private CartRepository cartRepository;
    @Autowired
    private RedisTemplate<String, Cart> redisCache;
    
    public Cart getCart(String userId) {
        // Check cache first
        Cart cart = redisCache.opsForValue().get("cart:" + userId);
        if (cart != null) {
            return cart;
        }
        
        // Load from database
        cart = cartRepository.findByUserId(userId);
        if (cart == null) {
            cart = new Cart(userId);
            cartRepository.save(cart);
        }
        
        // Cache for 30 days
        redisCache.opsForValue().set("cart:" + userId, cart, 30, TimeUnit.DAYS);
        return cart;
    }
    
    public void addItem(String userId, CartItem item) {
        Cart cart = getCart(userId);
        cart.addItem(item);
        saveCart(userId, cart);
    }
    
    private void saveCart(String userId, Cart cart) {
        cartRepository.save(cart);
        redisCache.opsForValue().set("cart:" + userId, cart, 30, TimeUnit.DAYS);
    }
}
```

**Architecture:**
- Redis for hot cart data
- Database for persistence
- TTL for cart expiration
- Event-driven updates

---

### Problem 2: Design Amazon's Recommendation System

**Problem Statement:**
Design a recommendation system that suggests products to users.

**Requirements:**
- Real-time recommendations
- Multiple algorithms (collaborative, content-based)
- Handle cold start
- Personalization

**Solution:**

```java
// Recommendation Service
@Service
public class RecommendationService {
    @Autowired
    private CollaborativeFiltering collaborativeFiltering;
    @Autowired
    private ContentBasedFiltering contentBasedFiltering;
    @Autowired
    private PopularItemsService popularItemsService;
    
    public List<Product> getRecommendations(String userId) {
        // Try collaborative filtering first
        List<Product> recommendations = collaborativeFiltering.recommend(userId);
        
        // If not enough, use content-based
        if (recommendations.size() < 10) {
            recommendations.addAll(contentBasedFiltering.recommend(userId));
        }
        
        // Cold start: Use popular items
        if (recommendations.isEmpty()) {
            recommendations = popularItemsService.getPopularItems();
        }
        
        return recommendations.stream()
            .limit(20)
            .collect(Collectors.toList());
    }
}
```

---

## Low-Level Design Problems

### Problem 3: Design Amazon's Order Processing System

**Problem Statement:**
Design an order processing system with multiple steps and failure handling.

**Requirements:**
- Order creation
- Inventory check
- Payment processing
- Shipping
- Failure handling

**Solution:**

```java
// Order Processor with Saga Pattern
@Service
public class OrderProcessor {
    @Autowired
    private InventoryService inventoryService;
    @Autowired
    private PaymentService paymentService;
    @Autowired
    private ShippingService shippingService;
    
    public OrderResult processOrder(OrderRequest request) {
        List<OrderStep> steps = new ArrayList<>();
        
        try {
            // Step 1: Reserve inventory
            InventoryResult inventory = inventoryService.reserve(request.getItems());
            steps.add(new OrderStep("inventory", inventory));
            
            if (!inventory.isReserved()) {
                return OrderResult.failed("Inventory unavailable");
            }
            
            // Step 2: Process payment
            PaymentResult payment = paymentService.process(request.getPayment());
            steps.add(new OrderStep("payment", payment));
            
            if (!payment.isSuccess()) {
                compensate(steps);
                return OrderResult.failed("Payment failed");
            }
            
            // Step 3: Create shipping
            ShippingResult shipping = shippingService.create(request);
            steps.add(new OrderStep("shipping", shipping));
            
            return OrderResult.success();
            
        } catch (Exception e) {
            compensate(steps);
            throw e;
        }
    }
    
    private void compensate(List<OrderStep> steps) {
        Collections.reverse(steps);
        for (OrderStep step : steps) {
            switch (step.getType()) {
                case "inventory":
                    inventoryService.release(step.getResult());
                    break;
                case "payment":
                    paymentService.refund(step.getResult());
                    break;
            }
        }
    }
}
```

---

## Summary

Amazon interview problems focus on:
- ✅ E-commerce systems
- ✅ High availability
- ✅ Scalability
- ✅ Distributed systems
- ✅ Real-world scenarios

**Key Takeaways:**
- Focus on scalability
- Handle failures gracefully
- Use caching effectively
- Design for high availability

---

*This guide is continuously updated with new Amazon interview problems.*

