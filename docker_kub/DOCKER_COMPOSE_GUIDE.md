# Docker Compose Complete Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Basic Usage](#basic-usage)
3. [Service Configuration](#service-configuration)
4. [Networking](#networking)
5. [Volumes](#volumes)
6. [Environment Variables](#environment-variables)
7. [Health Checks](#health-checks)
8. [Scaling](#scaling)
9. [Production Considerations](#production-considerations)
10. [Real-World Examples](#real-world-examples)

---

## Introduction

Docker Compose is a tool for defining and running multi-container Docker applications. You define services in a YAML file and Compose handles starting/stopping them.

### Why Docker Compose?

- **Simplified Multi-Container Setup:** Define all services in one file
- **Service Dependencies:** Automatically start dependencies
- **Networking:** Automatic service discovery
- **Volume Management:** Shared volumes between services
- **Environment Management:** Centralized environment variables

---

## Basic Usage

### Basic docker-compose.yml

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
```

### Commands

```bash
# Start services
docker-compose up

# Start in detached mode
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs

# Follow logs
docker-compose logs -f

# View running services
docker-compose ps

# Execute command in service
docker-compose exec web ls

# Restart service
docker-compose restart web

# Scale services
docker-compose up -d --scale web=3
```

---

## Service Configuration

### Build from Dockerfile

```yaml
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
```

### Build with Arguments

```yaml
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        VERSION: 1.0
    ports:
      - "8080:8080"
```

### Image vs Build

```yaml
version: '3.8'

services:
  # Use pre-built image
  redis:
    image: redis:7-alpine
  
  # Build from Dockerfile
  web:
    build: .
```

### Container Name

```yaml
version: '3.8'

services:
  web:
    image: nginx
    container_name: my-nginx
```

### Restart Policy

```yaml
version: '3.8'

services:
  web:
    image: nginx
    restart: always  # always, unless-stopped, on-failure, no
```

---

## Networking

### Default Network

All services are on the same network and can communicate by service name.

```yaml
version: '3.8'

services:
  web:
    image: nginx
    ports:
      - "8080:80"
  
  api:
    image: myapp:1.0
    # Can reach web at: http://web:80
```

### Custom Networks

```yaml
version: '3.8'

services:
  web:
    image: nginx
    networks:
      - frontend
  
  api:
    image: myapp:1.0
    networks:
      - frontend
      - backend
  
  db:
    image: postgres:15
    networks:
      - backend

networks:
  frontend:
  backend:
```

### External Networks

```yaml
version: '3.8'

services:
  web:
    image: nginx
    networks:
      - external-network

networks:
  external-network:
    external: true
    name: my-existing-network
```

---

## Volumes

### Named Volumes

```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Bind Mounts

```yaml
version: '3.8'

services:
  web:
    image: nginx
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./html:/usr/share/nginx/html
```

### Volume Options

```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data
        # volume driver options
        driver: local
        driver_opts:
          type: none
          o: bind
          device: /path/on/host

volumes:
  postgres_data:
```

---

## Environment Variables

### In docker-compose.yml

```yaml
version: '3.8'

services:
  web:
    image: myapp:1.0
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://db:5432/mydb
      - API_KEY=secret123
```

### From .env File

```yaml
version: '3.8'

services:
  web:
    image: myapp:1.0
    env_file:
      - .env
      - .env.production
```

### From Shell Environment

```yaml
version: '3.8'

services:
  web:
    image: myapp:1.0
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - API_KEY=${API_KEY}
```

### .env File Example

```bash
# .env
DATABASE_URL=postgresql://user:pass@db:5432/mydb
API_KEY=secret123
NODE_ENV=production
```

---

## Health Checks

### Health Check in Service

```yaml
version: '3.8'

services:
  web:
    image: myapp:1.0
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### Depends On with Health Check

```yaml
version: '3.8'

services:
  web:
    image: myapp:1.0
    depends_on:
      db:
        condition: service_healthy
  
  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

---

## Scaling

### Scale Services

```bash
# Scale web service to 3 instances
docker-compose up -d --scale web=3
```

### Load Balancing

```yaml
version: '3.8'

services:
  nginx:
    image: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - web
  
  web:
    image: myapp:1.0
    # Scale this service
```

```nginx
# nginx.conf
upstream web {
    server web:8080;
}

server {
    listen 80;
    location / {
        proxy_pass http://web;
    }
}
```

---

## Production Considerations

### Resource Limits

```yaml
version: '3.8'

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

**Note:** `deploy` section requires Docker Swarm mode or Docker Compose v3+

### Logging Configuration

```yaml
version: '3.8'

services:
  web:
    image: myapp:1.0
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

### Security

```yaml
version: '3.8'

services:
  web:
    image: myapp:1.0
    read_only: true
    tmpfs:
      - /tmp
    security_opt:
      - no-new-privileges:true
```

---

## Real-World Examples

### Example 1: Full-Stack Application

```yaml
version: '3.8'

services:
  # Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://api:8080
    depends_on:
      - api
  
  # Backend API
  api:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgresql://db:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
  
  # Database
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5
  
  # Redis
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
  
  # Kafka
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
  
  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
    ports:
      - "9092:9092"

volumes:
  postgres_data:
  redis_data:
```

### Example 2: Microservices

```yaml
version: '3.8'

services:
  # API Gateway
  gateway:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./gateway/nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - user-service
      - product-service
  
  # User Service
  user-service:
    build:
      context: ./user-service
    environment:
      - DATABASE_URL=postgresql://user-db:5432/userdb
    depends_on:
      - user-db
  
  user-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: userdb
    volumes:
      - user_db_data:/var/lib/postgresql/data
  
  # Product Service
  product-service:
    build:
      context: ./product-service
    environment:
      - DATABASE_URL=postgresql://product-db:5432/productdb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - product-db
      - redis
  
  product-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: productdb
    volumes:
      - product_db_data:/var/lib/postgresql/data
  
  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  user_db_data:
  product_db_data:
  redis_data:
```

### Example 3: Development with Hot Reload

```yaml
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - .:/app
      - /app/node_modules  # Prevent overwriting
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
    command: npm run dev
```

```dockerfile
# Dockerfile.dev
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
CMD ["npm", "run", "dev"]
```

---

## Advanced Features

### Profiles

```yaml
version: '3.8'

services:
  web:
    image: nginx
    profiles:
      - production
  
  dev-tools:
    image: dev-tools:latest
    profiles:
      - development
```

```bash
# Start with profile
docker-compose --profile production up
```

### Extends

```yaml
# docker-compose.base.yml
version: '3.8'

services:
  web:
    build: .
    environment:
      - DATABASE_URL=postgresql://db:5432/mydb

# docker-compose.yml
version: '3.8'

services:
  web:
    extends:
      file: docker-compose.base.yml
      service: web
    ports:
      - "8080:8080"
```

### Multiple Compose Files

```bash
# Override with production settings
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
```

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  web:
    restart: always
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
```

---

## Summary

Key Points:
- ✅ Define all services in one file
- ✅ Use depends_on for service ordering
- ✅ Use volumes for persistent data
- ✅ Use environment variables for configuration
- ✅ Add health checks for reliability
- ✅ Use networks for service isolation
- ✅ Set resource limits for production

Next: [Kubernetes Fundamentals](./KUBERNETES_FUNDAMENTALS.md)

