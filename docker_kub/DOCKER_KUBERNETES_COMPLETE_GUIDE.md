# Docker & Kubernetes Complete Guide for Staff Engineers

## Table of Contents
1. [Overview](#overview)
2. [Docker Fundamentals](#docker-fundamentals)
3. [Docker Best Practices](#docker-best-practices)
4. [Docker Compose](#docker-compose)
5. [Kubernetes Fundamentals](#kubernetes-fundamentals)
6. [Kubernetes Advanced Topics](#kubernetes-advanced-topics)
7. [Production Deployment](#production-deployment)
8. [CI/CD Integration](#cicd-integration)
9. [Monitoring & Observability](#monitoring--observability)
10. [Troubleshooting](#troubleshooting)
11. [Real-World Examples](#real-world-examples)

---

## Overview

This guide provides comprehensive coverage of Docker and Kubernetes for staff engineers working on full-stack applications. It covers everything from basics to production-grade deployments.

### Prerequisites
- Basic understanding of Linux/Unix commands
- Familiarity with microservices architecture
- Knowledge of containerization concepts

### Quick Links
- [Docker Fundamentals](./DOCKER_FUNDAMENTALS.md)
- [Docker Best Practices](./DOCKER_BEST_PRACTICES.md)
- [Docker Compose Guide](./DOCKER_COMPOSE_GUIDE.md)
- [Kubernetes Fundamentals](./KUBERNETES_FUNDAMENTALS.md)
- [Kubernetes Advanced](./KUBERNETES_ADVANCED.md)
- [Production Deployment](./PRODUCTION_DEPLOYMENT.md)
- [CI/CD Integration](./CICD_INTEGRATION.md)
- [Troubleshooting](./TROUBLESHOOTING.md)

---

## Docker Fundamentals

### What is Docker?

Docker is a platform for developing, shipping, and running applications using containerization. Containers package an application with all its dependencies, ensuring it runs consistently across different environments.

### Key Concepts

#### 1. Container vs Virtual Machine

```
Virtual Machine:
┌─────────────────────────────────────┐
│         Application                 │
│         Guest OS                    │
│         Hypervisor                  │
│         Host OS                     │
│         Hardware                    │
└─────────────────────────────────────┘
(Heavy, Slow, More Resources)

Container:
┌─────────────────────────────────────┐
│         Application                 │
│         Docker Engine                │
│         Host OS                     │
│         Hardware                    │
└─────────────────────────────────────┘
(Lightweight, Fast, Less Resources)
```

#### 2. Docker Components

- **Image:** Read-only template for creating containers
- **Container:** Running instance of an image
- **Dockerfile:** Instructions to build an image
- **Registry:** Repository for storing images (Docker Hub, ECR, GCR)
- **Docker Engine:** Runtime that runs containers

### Basic Docker Commands

```bash
# Build an image
docker build -t myapp:1.0 .

# Run a container
docker run -d -p 8080:8080 --name myapp myapp:1.0

# List running containers
docker ps

# List all containers
docker ps -a

# Stop a container
docker stop myapp

# Remove a container
docker rm myapp

# View logs
docker logs myapp

# Execute command in running container
docker exec -it myapp bash

# Remove an image
docker rmi myapp:1.0

# List images
docker images

# Pull an image
docker pull nginx:latest

# Push an image
docker push myregistry/myapp:1.0
```

### Dockerfile Best Practices

```dockerfile
# Use specific base image (not latest)
FROM openjdk:17-jdk-slim

# Set working directory
WORKDIR /app

# Copy only necessary files
COPY target/myapp.jar app.jar

# Use non-root user
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

# Use exec form for CMD
CMD ["java", "-jar", "app.jar"]
```

### Multi-Stage Builds

```dockerfile
# Stage 1: Build
FROM maven:3.8-openjdk-17 AS builder
WORKDIR /build
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Runtime
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY --from=builder /build/target/myapp.jar app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

**Benefits:**
- Smaller final image (no build tools)
- Faster builds (cached layers)
- Better security (fewer dependencies)

---

## Docker Best Practices

### 1. Image Size Optimization

```dockerfile
# Bad: Large image
FROM ubuntu:latest
RUN apt-get update && apt-get install -y \
    python3 python3-pip git curl wget vim

# Good: Minimal image
FROM python:3.11-slim
RUN pip install --no-cache-dir flask
```

### 2. Layer Caching

```dockerfile
# Bad: Changes invalidate cache
COPY . .
RUN npm install
RUN npm run build

# Good: Dependencies cached separately
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
```

### 3. Security Best Practices

```dockerfile
# Use specific versions
FROM node:18-alpine

# Don't run as root
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

# Scan for vulnerabilities
# docker scan myimage

# Use secrets management
# Don't hardcode secrets in Dockerfile
```

### 4. Health Checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD curl -f http://localhost:8080/health || exit 1
```

### 5. .dockerignore

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
```

---

## Docker Compose

Docker Compose allows you to define and run multi-container Docker applications.

### Basic docker-compose.yml

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=dev
    depends_on:
      - db
      - redis

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

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

  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

volumes:
  postgres_data:
```

### Compose Commands

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f web

# Scale services
docker-compose up -d --scale web=3

# Build and start
docker-compose up -d --build

# Stop and remove volumes
docker-compose down -v
```

### Environment Variables

```yaml
# docker-compose.yml
services:
  web:
    env_file:
      - .env
    environment:
      - DATABASE_URL=${DATABASE_URL}
```

```bash
# .env
DATABASE_URL=postgresql://user:pass@db:5432/mydb
```

---

## Kubernetes Fundamentals

### What is Kubernetes?

Kubernetes (K8s) is an open-source container orchestration platform that automates deployment, scaling, and management of containerized applications.

### Key Concepts

#### 1. Cluster Architecture

```
┌─────────────────────────────────────────┐
│         Kubernetes Cluster              │
│                                         │
│  ┌──────────────┐                      │
│  │ Control Plane│                      │
│  │ - API Server │                      │
│  │ - etcd       │                      │
│  │ - Scheduler  │                      │
│  │ - Controller │                      │
│  └──────────────┘                      │
│                                         │
│  ┌──────────┐  ┌──────────┐           │
│  │   Node   │  │   Node   │           │
│  │ - Kubelet│  │ - Kubelet│           │
│  │ - Pods   │  │ - Pods   │           │
│  └──────────┘  └──────────┘           │
└─────────────────────────────────────────┘
```

#### 2. Core Objects

**Pod:** Smallest deployable unit (contains one or more containers)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    ports:
    - containerPort: 8080
```

**Deployment:** Manages Pod replicas

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.0
        ports:
        - containerPort: 8080
```

**Service:** Exposes Pods as network service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
```

**ConfigMap:** Stores configuration data

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgresql://db:5432/mydb"
  log_level: "INFO"
```

**Secret:** Stores sensitive data

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  password: cGFzc3dvcmQ=  # base64 encoded
```

### Basic kubectl Commands

```bash
# Apply configuration
kubectl apply -f deployment.yaml

# Get resources
kubectl get pods
kubectl get deployments
kubectl get services

# Describe resource
kubectl describe pod myapp-pod

# View logs
kubectl logs myapp-pod

# Execute command in pod
kubectl exec -it myapp-pod -- bash

# Delete resource
kubectl delete pod myapp-pod

# Scale deployment
kubectl scale deployment myapp-deployment --replicas=5

# Port forward
kubectl port-forward myapp-pod 8080:8080
```

---

## Kubernetes Advanced Topics

### 1. Rolling Updates

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:2.0
        readinessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
```

### 2. Resource Limits

```yaml
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```

### 3. Horizontal Pod Autoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### 4. Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
```

### 5. StatefulSets

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  serviceName: database
  replicas: 3
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
      - name: database
        image: postgres:15
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

### 6. Jobs and CronJobs

```yaml
# Job
apiVersion: batch/v1
kind: Job
metadata:
  name: backup-job
spec:
  template:
    spec:
      containers:
      - name: backup
        image: backup:1.0
      restartPolicy: Never

# CronJob
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-cronjob
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup:1.0
          restartPolicy: OnFailure
```

---

## Production Deployment

### 1. Namespace Strategy

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
---
apiVersion: v1
kind: Namespace
metadata:
  name: staging
```

### 2. Resource Quotas

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    persistentvolumeclaims: "10"
```

### 3. Network Policies

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: myapp-network-policy
spec:
  podSelector:
    matchLabels:
      app: myapp
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
```

### 4. Pod Security Policies

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'projected'
    - 'secret'
    - 'downwardAPI'
    - 'persistentVolumeClaim'
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
```

---

## CI/CD Integration

### GitHub Actions Example

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
    
    - name: Build Docker image
      run: |
        docker build -t myapp:${{ github.sha }} .
        docker tag myapp:${{ github.sha }} myapp:latest
    
    - name: Push to registry
      run: |
        echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
        docker push myapp:${{ github.sha }}
        docker push myapp:latest
    
    - name: Deploy to Kubernetes
      run: |
        echo "${{ secrets.KUBECONFIG }}" | base64 -d > kubeconfig
        export KUBECONFIG=kubeconfig
        kubectl set image deployment/myapp-deployment myapp=myapp:${{ github.sha }}
        kubectl rollout status deployment/myapp-deployment
```

### GitLab CI Example

```yaml
# .gitlab-ci.yml
stages:
  - build
  - deploy

build:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

deploy:
  stage: deploy
  script:
    - kubectl set image deployment/myapp-deployment myapp=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  only:
    - main
```

---

## Monitoring & Observability

### 1. Prometheus Metrics

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
    prometheus.io/path: "/actuator/prometheus"
spec:
  selector:
    app: myapp
  ports:
  - port: 8080
```

### 2. Logging with Fluentd

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
data:
  fluent.conf: |
    <source>
      @type tail
      path /var/log/containers/*.log
      pos_file /var/log/fluentd-containers.log.pos
      tag kubernetes.*
      read_from_head true
    </source>
    <match kubernetes.**>
      @type elasticsearch
      host elasticsearch.logging.svc.cluster.local
      port 9200
      logstash_format true
    </match>
```

### 3. Distributed Tracing

```yaml
# Jaeger deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jaeger
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jaeger
  template:
    metadata:
      labels:
        app: jaeger
    spec:
      containers:
      - name: jaeger
        image: jaegertracing/all-in-one:latest
        ports:
        - containerPort: 16686
        env:
        - name: COLLECTOR_ZIPKIN_HTTP_PORT
          value: "9411"
```

---

## Troubleshooting

### Common Issues

#### 1. Pod Not Starting

```bash
# Check pod status
kubectl describe pod myapp-pod

# Check events
kubectl get events --sort-by='.lastTimestamp'

# Check logs
kubectl logs myapp-pod
```

#### 2. Image Pull Errors

```bash
# Check image pull secrets
kubectl get secrets

# Create image pull secret
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=pass
```

#### 3. Resource Constraints

```bash
# Check node resources
kubectl top nodes

# Check pod resources
kubectl top pods

# Describe node
kubectl describe node node-name
```

#### 4. Network Issues

```bash
# Test connectivity
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -O- http://myapp-service:80

# Check service endpoints
kubectl get endpoints myapp-service
```

---

## Real-World Examples

### Complete Microservices Deployment

See [COMPLETE_MICROSERVICES_DEPLOYMENT.md](./COMPLETE_MICROSERVICES_DEPLOYMENT.md) for a full example of deploying our e-commerce microservices to Kubernetes.

### Key Files:
- [Docker Fundamentals](./DOCKER_FUNDAMENTALS.md)
- [Docker Best Practices](./DOCKER_BEST_PRACTICES.md)
- [Docker Compose Guide](./DOCKER_COMPOSE_GUIDE.md)
- [Kubernetes Fundamentals](./KUBERNETES_FUNDAMENTALS.md)
- [Kubernetes Advanced](./KUBERNETES_ADVANCED.md)
- [Production Deployment](./PRODUCTION_DEPLOYMENT.md)
- [CI/CD Integration](./CICD_INTEGRATION.md)
- [Troubleshooting](./TROUBLESHOOTING.md)

---

## Next Steps

1. Review [Docker Fundamentals](./DOCKER_FUNDAMENTALS.md)
2. Learn [Kubernetes Fundamentals](./KUBERNETES_FUNDAMENTALS.md)
3. Practice with [Real-World Examples](./COMPLETE_MICROSERVICES_DEPLOYMENT.md)
4. Set up [CI/CD Pipeline](./CICD_INTEGRATION.md)
5. Implement [Monitoring](./MONITORING.md)

