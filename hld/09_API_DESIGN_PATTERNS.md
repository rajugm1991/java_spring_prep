# API Design Patterns

## Table of Contents
1. [Introduction](#introduction)
2. [REST API Design](#rest-api-design)
3. [GraphQL Architecture](#graphql-architecture)
4. [gRPC for Microservices](#grpc-for-microservices)
5. [API Gateway Patterns](#api-gateway-patterns)
6. [API Versioning](#api-versioning)
7. [Real-World Examples](#real-world-examples)

---

## Introduction

**API Design** defines how clients interact with your services. Good API design is crucial for:
- Developer experience
- System maintainability
- Performance
- Security

---

## REST API Design

**REST (Representational State Transfer)** is an architectural style for designing web services.

### REST Principles

1. **Stateless:** Each request contains all information needed
2. **Resource-Based:** URLs represent resources
3. **HTTP Methods:** Use standard HTTP verbs
4. **JSON/XML:** Standard data formats

### REST API Design

#### Resource Naming

```
Good:
GET    /api/users/123
POST   /api/users
PUT    /api/users/123
DELETE /api/users/123

Bad:
GET    /api/getUser?id=123
POST   /api/createUser
POST   /api/updateUser
POST   /api/deleteUser
```

#### HTTP Methods

| Method | Purpose | Idempotent |
|--------|---------|------------|
| GET | Read resource | Yes |
| POST | Create resource | No |
| PUT | Update resource (full) | Yes |
| PATCH | Update resource (partial) | No |
| DELETE | Delete resource | Yes |

#### Status Codes

```
2xx Success:
200 OK - Request succeeded
201 Created - Resource created
204 No Content - Success, no content

4xx Client Error:
400 Bad Request - Invalid request
401 Unauthorized - Authentication required
403 Forbidden - Not authorized
404 Not Found - Resource not found
429 Too Many Requests - Rate limit exceeded

5xx Server Error:
500 Internal Server Error - Server error
503 Service Unavailable - Service down
```

### Example: E-Commerce API

```java
@RestController
@RequestMapping("/api/v1/products")
public class ProductController {
    
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable String id) {
        Product product = productService.getProduct(id);
        return ResponseEntity.ok(product);
    }
    
    @PostMapping
    public ResponseEntity<Product> createProduct(@RequestBody Product product) {
        Product created = productService.createProduct(product);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<Product> updateProduct(
            @PathVariable String id, 
            @RequestBody Product product) {
        Product updated = productService.updateProduct(id, product);
        return ResponseEntity.ok(updated);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable String id) {
        productService.deleteProduct(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

## GraphQL Architecture

**GraphQL** is a query language and runtime for APIs.

### GraphQL vs REST

**REST:**
```
GET /api/users/123
GET /api/users/123/orders
GET /api/users/123/profile
(3 requests for user data)
```

**GraphQL:**
```
Query {
  user(id: 123) {
    name
    email
    orders {
      id
      total
    }
    profile {
      address
    }
  }
}
(1 request for all data)
```

### GraphQL Architecture

```
┌──────────┐
│  Client  │
└────┬─────┘
     │ GraphQL Query
     ▼
┌──────────────┐
│ GraphQL      │
│ Server       │
└──────┬───────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│User │ │Order│
│Svc  │ │Svc  │
└─────┘ └─────┘
```

### GraphQL Example

```graphql
# Query
query {
  product(id: "123") {
    name
    price
    seller {
      name
      rating
    }
  }
}

# Mutation
mutation {
  createOrder(input: {
    userId: "456"
    items: [
      {productId: "123", quantity: 2}
    ]
  }) {
    id
    total
    status
  }
}
```

### When to Use GraphQL

- ✅ Complex data relationships
- ✅ Mobile apps (reduce bandwidth)
- ✅ Multiple clients with different needs
- ❌ Simple CRUD operations
- ❌ Real-time requirements (use WebSocket)

---

## gRPC for Microservices

**gRPC** is a high-performance RPC framework.

### gRPC vs REST

| Aspect | REST | gRPC |
|--------|------|------|
| **Protocol** | HTTP/1.1 | HTTP/2 |
| **Format** | JSON | Protocol Buffers |
| **Performance** | Slower | Faster |
| **Browser Support** | Yes | Limited |
| **Streaming** | No | Yes |

### gRPC Architecture

```
┌──────────┐
│  Client  │
└────┬─────┘
     │ gRPC (HTTP/2)
     ▼
┌──────────────┐
│   gRPC       │
│   Server     │
└──────┬───────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│User │ │Order│
│Svc  │ │Svc  │
└─────┘ └─────┘
```

### gRPC Example

**Proto Definition:**
```protobuf
service ProductService {
  rpc GetProduct(ProductRequest) returns (Product);
  rpc CreateProduct(Product) returns (Product);
}

message Product {
  string id = 1;
  string name = 2;
  double price = 3;
}
```

**Java Implementation:**
```java
@Service
public class ProductServiceImpl extends ProductServiceGrpc.ProductServiceImplBase {
    
    @Override
    public void getProduct(ProductRequest request, 
                          StreamObserver<Product> responseObserver) {
        Product product = productService.getProduct(request.getId());
        responseObserver.onNext(product);
        responseObserver.onCompleted();
    }
}
```

### When to Use gRPC

- ✅ Microservices communication
- ✅ High performance needed
- ✅ Streaming support needed
- ✅ Strong typing required
- ❌ Browser clients (use REST/GraphQL)

---

## API Gateway Patterns

**API Gateway** is a single entry point for all client requests.

### API Gateway Functions

1. **Routing:** Route to appropriate service
2. **Authentication:** Verify JWT tokens
3. **Rate Limiting:** Prevent abuse
4. **Load Balancing:** Distribute traffic
5. **Request/Response Transformation:** Modify data
6. **Monitoring:** Log requests, metrics

### API Gateway Architecture

```
┌──────────┐
│  Client  │
└────┬─────┘
     │
     ▼
┌──────────────┐
│ API Gateway  │
│ - Auth       │
│ - Rate Limit │
│ - Routing    │
└──────┬───────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│User │ │Order│
│Svc  │ │Svc  │
└─────┘ └─────┘
```

### API Gateway Example: Spring Cloud Gateway

```java
@Configuration
public class GatewayConfig {
    
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("user-service", r -> r
                .path("/api/users/**")
                .filters(f -> f
                    .addRequestHeader("X-User-Id", "123")
                    .circuitBreaker(c -> c.setName("userService")))
                .uri("lb://user-service"))
            .route("order-service", r -> r
                .path("/api/orders/**")
                .uri("lb://order-service"))
            .build();
    }
}
```

---

## API Versioning

### Versioning Strategies

#### 1. **URL Versioning**

```
/api/v1/users
/api/v2/users
```

**Pros:** Simple, clear
**Cons:** URL pollution

---

#### 2. **Header Versioning**

```
GET /api/users
Header: API-Version: 2
```

**Pros:** Clean URLs
**Cons:** Less discoverable

---

#### 3. **Query Parameter Versioning**

```
/api/users?version=2
```

**Pros:** Optional
**Cons:** Can be forgotten

---

### Versioning Best Practices

```java
@RestController
@RequestMapping("/api/v1/products")
public class ProductControllerV1 {
    // V1 implementation
}

@RestController
@RequestMapping("/api/v2/products")
public class ProductControllerV2 {
    // V2 implementation with new features
}
```

---

## Real-World Examples

### Example 1: Stripe API

**Design:**
- RESTful
- Clear resource naming
- Comprehensive documentation
- Versioning in URL

```
POST /v1/charges
GET  /v1/charges/{id}
GET  /v1/customers/{id}/charges
```

---

### Example 2: GitHub API

**Design:**
- RESTful
- GraphQL option
- Rate limiting
- Webhooks

```
GET  /api/v3/user
POST /api/v3/repos
GET  /api/v3/repos/{owner}/{repo}/issues
```

---

### Example 3: Netflix API Gateway

**Design:**
- Single entry point
- Service routing
- Authentication
- Rate limiting
- Monitoring

---

## Summary

**Key Patterns:**
1. ✅ **REST:** Standard, simple, widely supported
2. ✅ **GraphQL:** Flexible queries, reduce over-fetching
3. ✅ **gRPC:** High performance, microservices
4. ✅ **API Gateway:** Single entry point, cross-cutting concerns
5. ✅ **Versioning:** Plan for evolution

**Next Steps:**
- Learn [Database Design](./10_DATABASE_DESIGN.md)
- Understand [Microservices](./07_MONOLITHIC_VS_MICROSERVICES.md)
- Practice [Real-World Designs](./13_DESIGNING_URL_SHORTENER.md)

---

**References:**
- [REST API Design](https://restfulapi.net/)
- [GraphQL Documentation](https://graphql.org/)
- [gRPC Documentation](https://grpc.io/)
