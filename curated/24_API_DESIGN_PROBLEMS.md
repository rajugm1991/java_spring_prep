# API Design Problems - Real-World Problems

> **Curated from top tech companies and real production scenarios**

---

## Problem 1: Design RESTful API for E-Commerce

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
@RestController
@RequestMapping("/api/v1")
public class ProductController {
    
    @GetMapping("/products")
    public ResponseEntity<Page<Product>> getProducts(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(required = false) String category) {
        
        Pageable pageable = PageRequest.of(page, size);
        Page<Product> products = productService.getProducts(category, pageable);
        return ResponseEntity.ok(products);
    }
    
    @PostMapping("/products")
    public ResponseEntity<Product> createProduct(@RequestBody @Valid ProductRequest request) {
        Product product = productService.createProduct(request);
        return ResponseEntity.status(HttpStatus.CREATED)
            .header("Location", "/api/v1/products/" + product.getId())
            .body(product);
    }
}
```

**API Design Principles:**
- Resource-based URLs
- HTTP methods
- Status codes
- Stateless
- Versioning

---

*This guide contains API design problems covering REST, GraphQL, gRPC, and API versioning.*

---

*This guide is continuously updated with new problems.*

