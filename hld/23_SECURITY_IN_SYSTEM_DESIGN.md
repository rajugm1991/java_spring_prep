# Security in System Design

## Table of Contents
1. [Introduction](#introduction)
2. [Authentication](#authentication)
3. [Authorization](#authorization)
4. [Encryption](#encryption)
5. [DDoS Protection](#ddos-protection)
6. [API Security](#api-security)
7. [Data Privacy](#data-privacy)
8. [Real-World Examples](#real-world-examples)

---

## Introduction

**Security** is crucial for protecting systems, data, and users from threats.

### Security Layers

```
Layer 1: Network Security (Firewall, DDoS)
Layer 2: Application Security (Auth, Authz)
Layer 3: Data Security (Encryption, Masking)
```

---

## Authentication

**Authentication:** Verifying who the user is.

### Methods

#### 1. **JWT (JSON Web Token)**

**How it works:**
```
1. User logs in
2. Server generates JWT
3. Client stores JWT
4. Client sends JWT with requests
5. Server validates JWT
```

**JWT Structure:**
```
Header.Payload.Signature

Header: Algorithm, type
Payload: User ID, roles, expiration
Signature: HMAC(header + payload, secret)
```

**Example:**
```java
String token = Jwts.builder()
    .setSubject(userId)
    .setExpiration(new Date(System.currentTimeMillis() + 3600000))
    .signWith(SignatureAlgorithm.HS256, secret)
    .compact();
```

---

#### 2. **OAuth 2.0**

**Flow:**
```
1. User → Authorization Server: Request access
2. Authorization Server → User: Login page
3. User → Authorization Server: Credentials
4. Authorization Server → Client: Authorization code
5. Client → Authorization Server: Exchange code for token
6. Authorization Server → Client: Access token
```

**Use Cases:**
- Third-party login (Google, Facebook)
- API access
- Mobile apps

---

#### 3. **SSO (Single Sign-On)**

**One login for multiple services:**
```
User logs in once → Access all services
```

**Example:**
```
Google Account → Gmail, YouTube, Drive
```

---

## Authorization

**Authorization:** Determining what user can do.

### RBAC (Role-Based Access Control)

**Roles and Permissions:**
```
Admin: Can do everything
User: Can read/write own data
Guest: Can only read
```

**Example:**
```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(String userId) {
    userService.delete(userId);
}
```

---

### ABAC (Attribute-Based Access Control)

**Policies based on attributes:**
```
If user.department == "Finance" AND resource.type == "Financial"
Then allow access
```

---

## Encryption

### Encryption at Rest

**Encrypt data stored in database:**
```
Plain text → Encrypt → Store in database
```

**Example:**
```java
String encrypted = AES.encrypt(data, key);
database.save(encrypted);
```

---

### Encryption in Transit

**Encrypt data over network:**
```
TLS/SSL: Encrypts HTTP traffic
```

**Example:**
```
HTTPS: HTTP over TLS
All data encrypted between client and server
```

---

### Key Management

**Store encryption keys securely:**
```
AWS KMS: Key Management Service
Azure Key Vault: Key storage
HashiCorp Vault: Secrets management
```

---

## DDoS Protection

**DDoS (Distributed Denial of Service):** Overwhelm system with traffic.

### Protection Strategies

#### 1. **Rate Limiting**

**Limit requests per IP:**
```
IP: 192.168.1.1
Limit: 100 requests/minute
If exceeded → Block
```

---

#### 2. **CDN**

**Distribute traffic:**
```
DDoS attack → CDN absorbs → Origin protected
```

---

#### 3. **DDoS Protection Services**

**CloudFlare, AWS Shield:**
```
Traffic → DDoS Protection → Filter malicious → Forward legitimate
```

---

## API Security

### API Key

**Simple authentication:**
```
Request Header: X-API-Key: abc123
```

---

### OAuth 2.0

**Token-based:**
```
Request Header: Authorization: Bearer <token>
```

---

### Rate Limiting

**Prevent abuse:**
```
100 requests/hour per API key
```

---

## Data Privacy

### GDPR Compliance

**Requirements:**
- User data encryption
- Right to be forgotten
- Data portability
- Consent management

**Implementation:**
```java
public void deleteUserData(String userId) {
    // Delete from all systems
    userRepository.delete(userId);
    orderRepository.deleteByUserId(userId);
    analyticsRepository.deleteByUserId(userId);
}
```

---

### Data Masking

**Hide sensitive data:**
```
Credit Card: 1234-5678-9012-3456
Masked: ****-****-****-3456
```

---

## Real-World Examples

### Example 1: Banking System

**Security Measures:**
- Multi-factor authentication
- Encryption at rest and in transit
- Fraud detection
- Audit logging
- PCI-DSS compliance

---

### Example 2: Healthcare System

**Security Measures:**
- HIPAA compliance
- Role-based access
- Data encryption
- Audit trails
- Patient data privacy

---

### Example 3: E-Commerce Platform

**Security Measures:**
- JWT authentication
- HTTPS/TLS
- PCI-DSS for payments
- Rate limiting
- DDoS protection

---

## Summary

**Key Security Measures:**
1. ✅ **Authentication:** JWT, OAuth, SSO
2. ✅ **Authorization:** RBAC, ABAC
3. ✅ **Encryption:** At rest, in transit
4. ✅ **DDoS Protection:** Rate limiting, CDN
5. ✅ **API Security:** API keys, OAuth
6. ✅ **Data Privacy:** GDPR, masking

**Next Steps:**
- Learn [Cost Optimization](./24_COST_OPTIMIZATION.md)
- Understand [System Monitoring](./22_SYSTEM_MONITORING.md)
- Practice [Real-World Designs](./13_DESIGNING_URL_SHORTENER.md)

---

**References:**
- [Security Best Practices](https://algomaster.io/learn/system-design)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
