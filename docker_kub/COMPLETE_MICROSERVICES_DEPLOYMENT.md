# Complete Microservices Deployment Guide

## Table of Contents
1. [Overview](#overview)
2. [Project Structure](#project-structure)
3. [Docker Setup](#docker-setup)
4. [Kubernetes Deployment](#kubernetes-deployment)
5. [Service Configuration](#service-configuration)
6. [Networking](#networking)
7. [Storage](#storage)
8. [Monitoring](#monitoring)
9. [CI/CD Pipeline](#cicd-pipeline)
10. [Production Checklist](#production-checklist)

---

## Overview

This guide demonstrates deploying our e-commerce microservices application to Kubernetes with Docker.

### Architecture

```
┌─────────────────────────────────────────────────┐
│              Kubernetes Cluster                  │
│                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │   API    │  │  User    │  │ Product  │    │
│  │ Gateway  │  │ Service  │  │ Service  │    │
│  └──────────┘  └──────────┘  └──────────┘    │
│                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │   Eureka │  │  Config  │  │  Seller  │    │
│  │ Registry│  │  Server  │  │ Service  │    │
│  └──────────┘  └──────────┘  └──────────┘    │
│                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │  Redis   │  │  Kafka   │  │ Postgres │    │
│  └──────────┘  └──────────┘  └──────────┘    │
└─────────────────────────────────────────────────┘
```

---

## Project Structure

```
sb_ecommerce/
├── api-gateway/
│   ├── Dockerfile
│   └── k8s/
│       ├── deployment.yaml
│       └── service.yaml
├── user-service/
│   ├── Dockerfile
│   └── k8s/
│       ├── deployment.yaml
│       ├── service.yaml
│       └── configmap.yaml
├── product-service/
│   ├── Dockerfile
│   └── k8s/
│       ├── deployment.yaml
│       ├── service.yaml
│       └── configmap.yaml
├── k8s/
│   ├── namespace.yaml
│   ├── configmaps.yaml
│   ├── secrets.yaml
│   └── ingress.yaml
└── docker-compose.yml
```

---

## Docker Setup

### Dockerfile for Spring Boot Service

```dockerfile
# api-gateway/Dockerfile
FROM maven:3.8-openjdk-17 AS builder
WORKDIR /build
COPY pom.xml .
RUN mvn dependency:go-offline -B
COPY src ./src
RUN mvn clean package -DskipTests -B

FROM openjdk:17-jre-slim
WORKDIR /app
RUN groupadd -r appuser && \
    useradd -r -g appuser -u 1001 appuser
COPY --from=builder --chown=appuser:appuser /build/target/api-gateway.jar app.jar
USER appuser
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD curl -f http://localhost:8080/actuator/health || exit 1
CMD ["java", "-jar", "app.jar"]
```

### Build Script

```bash
#!/bin/bash
# build-all.sh

services=("api-gateway" "user-service" "product-service" "seller-service")

for service in "${services[@]}"; do
    echo "Building $service..."
    cd "$service"
    docker build -t "$service:latest" .
    docker tag "$service:latest" "registry.example.com/$service:latest"
    cd ..
done

echo "Pushing images..."
for service in "${services[@]}"; do
    docker push "registry.example.com/$service:latest"
done
```

---

## Kubernetes Deployment

### Namespace

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ecommerce
```

### API Gateway Deployment

```yaml
# api-gateway/k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
  namespace: ecommerce
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api-gateway
  template:
    metadata:
      labels:
        app: api-gateway
    spec:
      containers:
      - name: api-gateway
        image: registry.example.com/api-gateway:latest
        ports:
        - containerPort: 8080
        env:
        - name: EUREKA_CLIENT_SERVICEURL_DEFAULTZONE
          value: "http://eureka-service:8761/eureka"
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: api-gateway-service
  namespace: ecommerce
spec:
  selector:
    app: api-gateway
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
```

### User Service Deployment

```yaml
# user-service/k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  namespace: ecommerce
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
      - name: user-service
        image: registry.example.com/user-service:latest
        ports:
        - containerPort: 8081
        env:
        - name: SPRING_DATASOURCE_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: database_url
        - name: SPRING_DATASOURCE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: db_password
        - name: EUREKA_CLIENT_SERVICEURL_DEFAULTZONE
          value: "http://eureka-service:8761/eureka"
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8081
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health
            port: 8081
          initialDelaySeconds: 30
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: user-service
  namespace: ecommerce
spec:
  selector:
    app: user-service
  ports:
  - port: 8081
    targetPort: 8081
```

### Product Service Deployment

```yaml
# product-service/k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service
  namespace: ecommerce
spec:
  replicas: 3
  selector:
    matchLabels:
      app: product-service
  template:
    metadata:
      labels:
        app: product-service
    spec:
      containers:
      - name: product-service
        image: registry.example.com/product-service:latest
        ports:
        - containerPort: 8082
        env:
        - name: SPRING_DATA_REDIS_HOST
          value: "redis-service"
        - name: SPRING_KAFKA_BOOTSTRAP_SERVERS
          value: "kafka-service:9092"
        - name: EUREKA_CLIENT_SERVICEURL_DEFAULTZONE
          value: "http://eureka-service:8761/eureka"
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8082
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health
            port: 8082
          initialDelaySeconds: 30
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: product-service
  namespace: ecommerce
spec:
  selector:
    app: product-service
  ports:
  - port: 8082
    targetPort: 8082
```

---

## Service Configuration

### ConfigMap

```yaml
# k8s/configmaps.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: ecommerce
data:
  database_url: "jdbc:postgresql://postgres-service:5432/ecommerce"
  redis_host: "redis-service"
  kafka_bootstrap_servers: "kafka-service:9092"
  eureka_url: "http://eureka-service:8761/eureka"
```

### Secrets

```yaml
# k8s/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
  namespace: ecommerce
type: Opaque
data:
  db_password: cGFzc3dvcmQ=  # base64 encoded
  api_key: YXBpX2tleQ==
```

**Create secrets:**
```bash
kubectl create secret generic app-secrets \
  --from-literal=db_password=password \
  --from-literal=api_key=apikey \
  -n ecommerce
```

---

## Networking

### Ingress

```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
  namespace: ecommerce
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - api.example.com
    secretName: ecommerce-tls
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-gateway-service
            port:
              number: 80
```

---

## Storage

### PostgreSQL StatefulSet

```yaml
# k8s/postgres-statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: ecommerce
spec:
  serviceName: postgres-service
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15-alpine
        env:
        - name: POSTGRES_DB
          value: ecommerce
        - name: POSTGRES_USER
          value: postgres
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: db_password
        volumeMounts:
        - name: postgres-data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: postgres-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 20Gi
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
  namespace: ecommerce
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
```

### Redis Deployment

```yaml
# k8s/redis-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: ecommerce
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7-alpine
        ports:
        - containerPort: 6379
        volumeMounts:
        - name: redis-data
          mountPath: /data
  volumes:
  - name: redis-data
    persistentVolumeClaim:
      claimName: redis-pvc
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: redis-pvc
  namespace: ecommerce
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
---
apiVersion: v1
kind: Service
metadata:
  name: redis-service
  namespace: ecommerce
spec:
  selector:
    app: redis
  ports:
  - port: 6379
    targetPort: 6379
```

---

## Monitoring

### Horizontal Pod Autoscaler

```yaml
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: product-service-hpa
  namespace: ecommerce
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: product-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### ServiceMonitor for Prometheus

```yaml
# k8s/servicemonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: ecommerce-services
  namespace: ecommerce
spec:
  selector:
    matchLabels:
      app: product-service
  endpoints:
  - port: http
    path: /actuator/prometheus
    interval: 30s
```

---

## CI/CD Pipeline

### GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy to Kubernetes

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up JDK
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
    
    - name: Build and push Docker images
      run: |
        docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}
        ./build-all.sh
    
    - name: Configure kubectl
      run: |
        echo "${{ secrets.KUBECONFIG }}" | base64 -d > kubeconfig
        export KUBECONFIG=kubeconfig
    
    - name: Deploy to Kubernetes
      run: |
        kubectl apply -f k8s/namespace.yaml
        kubectl apply -f k8s/configmaps.yaml
        kubectl apply -f k8s/secrets.yaml
        kubectl apply -f k8s/
        kubectl rollout status deployment/product-service -n ecommerce
```

---

## Production Checklist

### Pre-Deployment
- [ ] All Docker images built and pushed
- [ ] ConfigMaps and Secrets created
- [ ] Resource limits set
- [ ] Health checks configured
- [ ] Ingress configured
- [ ] TLS certificates configured
- [ ] Monitoring set up
- [ ] Logging configured
- [ ] Backup strategy in place

### Deployment
- [ ] Namespace created
- [ ] ConfigMaps applied
- [ ] Secrets applied
- [ ] Database deployed
- [ ] Redis deployed
- [ ] Kafka deployed
- [ ] Services deployed
- [ ] Ingress configured
- [ ] HPA configured

### Post-Deployment
- [ ] Health checks passing
- [ ] Services accessible
- [ ] Monitoring working
- [ ] Logs accessible
- [ ] Performance metrics normal
- [ ] Rollback plan ready

---

## Deployment Script

```bash
#!/bin/bash
# deploy.sh

set -e

NAMESPACE="ecommerce"

echo "Creating namespace..."
kubectl apply -f k8s/namespace.yaml

echo "Creating ConfigMaps..."
kubectl apply -f k8s/configmaps.yaml

echo "Creating Secrets..."
kubectl apply -f k8s/secrets.yaml

echo "Deploying infrastructure..."
kubectl apply -f k8s/postgres-statefulset.yaml
kubectl apply -f k8s/redis-deployment.yaml

echo "Waiting for infrastructure..."
kubectl wait --for=condition=ready pod -l app=postgres -n $NAMESPACE --timeout=300s
kubectl wait --for=condition=ready pod -l app=redis -n $NAMESPACE --timeout=300s

echo "Deploying services..."
kubectl apply -f user-service/k8s/
kubectl apply -f product-service/k8s/
kubectl apply -f api-gateway/k8s/

echo "Waiting for deployments..."
kubectl rollout status deployment/user-service -n $NAMESPACE
kubectl rollout status deployment/product-service -n $NAMESPACE
kubectl rollout status deployment/api-gateway -n $NAMESPACE

echo "Deploying Ingress..."
kubectl apply -f k8s/ingress.yaml

echo "Deployment complete!"
kubectl get all -n $NAMESPACE
```

---

## Summary

This guide provides a complete example of deploying microservices to Kubernetes:
- ✅ Docker images for all services
- ✅ Kubernetes deployments with health checks
- ✅ ConfigMaps and Secrets for configuration
- ✅ StatefulSets for databases
- ✅ Ingress for external access
- ✅ HPA for auto-scaling
- ✅ CI/CD pipeline
- ✅ Monitoring setup

Next: [Production Deployment](./PRODUCTION_DEPLOYMENT.md)

