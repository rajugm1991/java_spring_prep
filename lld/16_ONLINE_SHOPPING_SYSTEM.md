# Online Shopping System

## Table of Contents
1. [Problem Statement](#problem-statement)
2. [Requirements Analysis](#requirements-analysis)
3. [Class Design](#class-design)
4. [Complete Implementation](#complete-implementation)
5. [Design Patterns Used](#design-patterns-used)

---

## Problem Statement

Design an online shopping system (e-commerce) that can:
- Browse products
- Add to cart
- Place orders
- Process payments
- Manage inventory
- Handle shipping

---

## Requirements Analysis

### Functional Requirements

1. **Product Management**
   - Browse products
   - Search products
   - View product details

2. **Shopping Cart**
   - Add items
   - Remove items
   - Update quantities
   - Calculate total

3. **Order Management**
   - Place order
   - Track order status
   - Cancel order

4. **Payment**
   - Multiple payment methods
   - Process payment
   - Refund

---

## Class Design

### Core Classes

```java
// Product class
public class Product {
    private final Long productId;
    private String name;
    private String description;
    private double price;
    private int stock;
    private Category category;
    
    public Product(Long productId, String name, double price) {
        this.productId = productId;
        this.name = name;
        this.price = price;
        this.stock = 0;
    }
    
    public boolean isAvailable() {
        return stock > 0;
    }
    
    public void reduceStock(int quantity) {
        if (stock < quantity) {
            throw new InsufficientStockException("Not enough stock");
        }
        stock -= quantity;
    }
}

// Cart Item
public class CartItem {
    private Product product;
    private int quantity;
    
    public CartItem(Product product, int quantity) {
        this.product = product;
        this.quantity = quantity;
    }
    
    public double getTotal() {
        return product.getPrice() * quantity;
    }
}

// Shopping Cart
public class ShoppingCart {
    private Long userId;
    private List<CartItem> items;
    
    public ShoppingCart(Long userId) {
        this.userId = userId;
        this.items = new ArrayList<>();
    }
    
    public void addItem(Product product, int quantity) {
        CartItem existingItem = findItem(product);
        if (existingItem != null) {
            existingItem.setQuantity(existingItem.getQuantity() + quantity);
        } else {
            items.add(new CartItem(product, quantity));
        }
    }
    
    public void removeItem(Product product) {
        items.removeIf(item -> item.getProduct().equals(product));
    }
    
    public double calculateTotal() {
        return items.stream()
            .mapToDouble(CartItem::getTotal)
            .sum();
    }
    
    private CartItem findItem(Product product) {
        return items.stream()
            .filter(item -> item.getProduct().equals(product))
            .findFirst()
            .orElse(null);
    }
}

// Order
public class Order {
    private final String orderId;
    private final Long userId;
    private List<OrderItem> items;
    private OrderStatus status;
    private double total;
    private LocalDateTime createdAt;
    
    public Order(String orderId, Long userId, List<OrderItem> items) {
        this.orderId = orderId;
        this.userId = userId;
        this.items = items;
        this.status = OrderStatus.PENDING;
        this.total = calculateTotal();
        this.createdAt = LocalDateTime.now();
    }
    
    private double calculateTotal() {
        return items.stream()
            .mapToDouble(item -> item.getPrice() * item.getQuantity())
            .sum();
    }
    
    public void confirm() {
        if (status != OrderStatus.PENDING) {
            throw new IllegalStateException("Order already processed");
        }
        this.status = OrderStatus.CONFIRMED;
    }
    
    public void cancel() {
        if (status == OrderStatus.SHIPPED || status == OrderStatus.DELIVERED) {
            throw new IllegalStateException("Cannot cancel shipped order");
        }
        this.status = OrderStatus.CANCELLED;
    }
}

// Payment Strategy
public interface PaymentStrategy {
    PaymentResult pay(double amount);
}

public class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;
    
    public PaymentResult pay(double amount) {
        // Process credit card payment
        return new PaymentResult(true, "Payment successful");
    }
}

public class PayPalPayment implements PaymentStrategy {
    private String email;
    
    public PaymentResult pay(double amount) {
        // Process PayPal payment
        return new PaymentResult(true, "Payment successful");
    }
}

// Order Service
public class OrderService {
    private ProductService productService;
    private PaymentService paymentService;
    private InventoryService inventoryService;
    
    public Order placeOrder(ShoppingCart cart, PaymentStrategy paymentStrategy) {
        // Validate cart
        validateCart(cart);
        
        // Create order
        List<OrderItem> orderItems = convertToOrderItems(cart);
        Order order = new Order(generateOrderId(), cart.getUserId(), orderItems);
        
        // Process payment
        PaymentResult payment = paymentService.processPayment(
            order.getTotal(), 
            paymentStrategy
        );
        
        if (payment.isSuccess()) {
            // Update inventory
            inventoryService.reserveItems(order);
            order.confirm();
        } else {
            throw new PaymentException("Payment failed");
        }
        
        return order;
    }
    
    private void validateCart(ShoppingCart cart) {
        if (cart.getItems().isEmpty()) {
            throw new IllegalArgumentException("Cart is empty");
        }
        
        for (CartItem item : cart.getItems()) {
            if (!item.getProduct().isAvailable()) {
                throw new InsufficientStockException("Product not available");
            }
        }
    }
}
```

---

## Design Patterns Used

### 1. Strategy Pattern (Payment)

```java
// Different payment strategies
PaymentStrategy creditCard = new CreditCardPayment();
PaymentStrategy paypal = new PayPalPayment();

// Use any strategy
paymentService.processPayment(amount, creditCard);
```

### 2. Factory Pattern (Order Creation)

```java
public class OrderFactory {
    public static Order createOrder(ShoppingCart cart) {
        // Factory logic to create order
    }
}
```

---

## Summary

**Key Components:**
1. ✅ **Product:** Product information
2. ✅ **ShoppingCart:** Cart management
3. ✅ **Order:** Order processing
4. ✅ **PaymentStrategy:** Multiple payment methods
5. ✅ **OrderService:** Business logic

**Next Steps:**
- Practice [Vending Machine](./17_VENDING_MACHINE.md)
- Learn [Design Patterns](./08_CREATIONAL_PATTERNS.md)
- Review [Strategy Pattern](./10_BEHAVIORAL_PATTERNS.md)

---

**References:**
- [Strategy Pattern](./10_BEHAVIORAL_PATTERNS.md)
- [Factory Pattern](./08_CREATIONAL_PATTERNS.md)

