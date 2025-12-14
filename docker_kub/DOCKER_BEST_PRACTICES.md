# Docker Best Practices for Production

## Table of Contents
1. [Image Optimization](#image-optimization)
2. [Security Best Practices](#security-best-practices)
3. [Performance Optimization](#performance-optimization)
4. [Multi-Stage Builds](#multi-stage-builds)
5. [Layer Caching](#layer-caching)
6. [Health Checks](#health-checks)
7. [Resource Management](#resource-management)
8. [Logging Best Practices](#logging-best-practices)
9. [.dockerignore](#dockerignore)
10. [Production Checklist](#production-checklist)

---

## Image Optimization

### 1. Use Minimal Base Images

```dockerfile
# ❌ Bad: Large base image (~700MB)
FROM ubuntu:latest
RUN apt-get update && apt-get install -y python3

# ✅ Good: Minimal base image (~50MB)
FROM python:3.11-slim

# ✅ Better: Alpine-based (~15MB)
FROM python:3.11-alpine
```

**Size Comparison:**
- `ubuntu:latest`: ~700MB
- `python:3.11-slim`: ~50MB
- `python:3.11-alpine`: ~15MB

### 2. Clean Up Package Managers

```dockerfile
# ❌ Bad: Leaves cache
RUN apt-get update && apt-get install -y curl

# ✅ Good: Removes cache
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

### 3. Combine RUN Commands

```dockerfile
# ❌ Bad: Multiple layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y wget

# ✅ Good: Single layer
RUN apt-get update && \
    apt-get install -y curl wget && \
    rm -rf /var/lib/apt/lists/*
```

### 4. Use .dockerignore

```dockerignore
# .dockerignore
node_modules
.git
.gitignore
*.md
.env
.idea
target
*.log
.DS_Store
coverage
.vscode
```

**Impact:** Reduces build context size and build time.

---

## Security Best Practices

### 1. Don't Run as Root

```dockerfile
# ❌ Bad: Runs as root
FROM node:18-alpine
COPY . .
CMD ["node", "server.js"]

# ✅ Good: Non-root user
FROM node:18-alpine
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs
COPY --chown=nodejs:nodejs . .
CMD ["node", "server.js"]
```

### 2. Use Specific Image Tags

```dockerfile
# ❌ Bad: Latest tag (unpredictable)
FROM node:latest

# ✅ Good: Specific version
FROM node:18-alpine

# ✅ Better: Digest (immutable)
FROM node@sha256:abc123...
```

### 3. Scan Images for Vulnerabilities

```bash
# Docker Scout (built-in)
docker scout cves myimage:1.0

# Trivy
trivy image myimage:1.0

# Snyk
snyk test --docker myimage:1.0
```

### 4. Don't Store Secrets in Images

```dockerfile
# ❌ Bad: Hardcoded secret
ENV API_KEY=secret123

# ✅ Good: Use secrets at runtime
# docker run -e API_KEY=secret123 myimage
```

### 5. Use Multi-Stage Builds

```dockerfile
# Stage 1: Build (can have build tools)
FROM node:18-alpine AS builder
WORKDIR /build
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Runtime (minimal dependencies)
FROM node:18-alpine
WORKDIR /app
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs
COPY --from=builder --chown=nodejs:nodejs /build/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /build/node_modules ./node_modules
CMD ["node", "dist/server.js"]
```

### 6. Limit Capabilities

```bash
# Run with limited capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp
```

---

## Performance Optimization

### 1. Leverage Layer Caching

```dockerfile
# ✅ Good: Dependencies cached separately
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# If code changes, npm install is cached
# If only package.json changes, only npm install runs
```

### 2. Use BuildKit

```bash
# Enable BuildKit
export DOCKER_BUILDKIT=1

# Or use buildx
docker buildx build -t myapp:1.0 .
```

**Benefits:**
- Faster builds
- Better caching
- Parallel builds
- Secret management

### 3. Use Build Cache Mounts

```dockerfile
# Cache npm dependencies
RUN --mount=type=cache,target=/root/.npm \
    npm install
```

### 4. Minimize Layers

```dockerfile
# ❌ Bad: Many layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y wget
RUN apt-get clean

# ✅ Good: Single layer
RUN apt-get update && \
    apt-get install -y curl wget && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

---

## Multi-Stage Builds

### Example: Java Application

```dockerfile
# Stage 1: Build
FROM maven:3.8-openjdk-17 AS builder
WORKDIR /build
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Runtime
FROM openjdk:17-jre-slim
WORKDIR /app
COPY --from=builder /build/target/myapp.jar app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

**Benefits:**
- Final image: ~150MB (vs ~800MB with build tools)
- No build tools in production
- Better security

### Example: Node.js Application

```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /build
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Dependencies
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Stage 3: Runtime
FROM node:18-alpine
WORKDIR /app
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
COPY --from=deps --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /build/dist ./dist
USER nodejs
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

---

## Layer Caching

### Order Matters

```dockerfile
# ✅ Good: Dependencies first (change less frequently)
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# ❌ Bad: Code first (changes frequently)
COPY . .
RUN npm install
RUN npm run build
```

### Cache Invalidation

```dockerfile
# This invalidates cache from this point
ARG CACHE_BUST=1
RUN echo "Cache bust: $CACHE_BUST"

# Use for forcing rebuild
# docker build --build-arg CACHE_BUST=$(date +%s)
```

---

## Health Checks

### HTTP Health Check

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD curl -f http://localhost:8080/health || exit 1
```

### Custom Health Check Script

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD /app/healthcheck.sh || exit 1
```

### Check Health Status

```bash
# Check health status
docker ps
# HEALTHY / UNHEALTHY / STARTING

# Inspect health
docker inspect --format='{{.State.Health.Status}}' mycontainer
```

---

## Resource Management

### Memory Limits

```bash
# Set memory limit
docker run -m 512m myapp

# Memory + swap
docker run -m 512m --memory-swap=1g myapp
```

### CPU Limits

```bash
# Limit CPU
docker run --cpus="1.5" myapp

# CPU shares (relative)
docker run --cpu-shares=512 myapp
```

### In Dockerfile

```dockerfile
# Document resource requirements
# Note: Dockerfile doesn't enforce limits
# Use docker run flags or docker-compose
```

### In docker-compose.yml

```yaml
services:
  web:
    image: myapp:1.0
    deploy:
      resources:
        limits:
          cpus: '1.5'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
```

---

## Logging Best Practices

### 1. Use JSON Logging

```dockerfile
# Configure application to log to stdout/stderr
# Docker captures these automatically
```

### 2. Log Rotation

```bash
# Configure log rotation
docker run --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  myapp
```

### 3. Structured Logging

```json
{
  "timestamp": "2023-01-01T00:00:00Z",
  "level": "INFO",
  "message": "Request processed",
  "requestId": "abc123"
}
```

### 4. Centralized Logging

```yaml
# docker-compose.yml
services:
  web:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

---

## .dockerignore

### Complete .dockerignore Example

```dockerignore
# Version control
.git
.gitignore
.gitattributes

# IDE
.idea
.vscode
*.swp
*.swo
*~

# Dependencies
node_modules
vendor
target
dist
build

# Environment files
.env
.env.local
.env.*.local

# Documentation
*.md
docs/
README*

# Tests
coverage
.nyc_output
*.test.js
*.spec.js

# Logs
*.log
logs/

# OS files
.DS_Store
Thumbs.db

# Temporary files
tmp/
temp/
*.tmp
```

---

## Production Checklist

### Image Security
- [ ] Use specific image tags (not `latest`)
- [ ] Scan images for vulnerabilities
- [ ] Use minimal base images
- [ ] Run as non-root user
- [ ] No secrets in images
- [ ] Use multi-stage builds

### Image Optimization
- [ ] Use .dockerignore
- [ ] Optimize layer caching
- [ ] Combine RUN commands
- [ ] Clean up package managers
- [ ] Use BuildKit

### Container Configuration
- [ ] Set resource limits
- [ ] Add health checks
- [ ] Configure logging
- [ ] Use proper networking
- [ ] Use volumes for persistent data

### Monitoring
- [ ] Health check endpoint
- [ ] Structured logging
- [ ] Metrics endpoint
- [ ] Log aggregation

### Deployment
- [ ] Tag images properly
- [ ] Use private registry
- [ ] Implement CI/CD
- [ ] Rollback strategy
- [ ] Monitoring and alerting

---

## Example: Production-Ready Dockerfile

```dockerfile
# Multi-stage build for Spring Boot
FROM maven:3.8-openjdk-17 AS builder
WORKDIR /build

# Cache dependencies
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Build application
COPY src ./src
RUN mvn clean package -DskipTests -B

# Runtime stage
FROM openjdk:17-jre-slim
WORKDIR /app

# Create non-root user
RUN groupadd -r appuser && \
    useradd -r -g appuser -u 1001 appuser

# Copy application
COPY --from=builder --chown=appuser:appuser /build/target/myapp.jar app.jar

# Switch to non-root user
USER appuser

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

# Run application
CMD ["java", "-jar", "app.jar"]
```

---

## Summary

Key Best Practices:
1. ✅ Use minimal base images
2. ✅ Leverage layer caching
3. ✅ Use multi-stage builds
4. ✅ Run as non-root user
5. ✅ Add health checks
6. ✅ Set resource limits
7. ✅ Use .dockerignore
8. ✅ Scan for vulnerabilities
9. ✅ Configure logging
10. ✅ Document everything

Next: [Docker Compose Guide](./DOCKER_COMPOSE_GUIDE.md)

