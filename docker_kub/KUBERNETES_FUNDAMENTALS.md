# Kubernetes Fundamentals - Complete Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Architecture](#architecture)
3. [Core Concepts](#core-concepts)
4. [Pods](#pods)
5. [Deployments](#deployments)
6. [Services](#services)
7. [ConfigMaps and Secrets](#configmaps-and-secrets)
8. [Namespaces](#namespaces)
9. [Basic Operations](#basic-operations)
10. [Common Patterns](#common-patterns)

---

## Introduction

### What is Kubernetes?

Kubernetes (K8s) is an open-source container orchestration platform that automates:
- **Deployment:** Deploy containers across clusters
- **Scaling:** Scale applications up/down
- **Management:** Manage container lifecycle
- **Networking:** Service discovery and load balancing
- **Storage:** Persistent storage management

### Why Kubernetes?

**Without Kubernetes:**
- Manual container management
- Difficult scaling
- No self-healing
- Complex networking
- No rolling updates

**With Kubernetes:**
- Automated orchestration
- Easy scaling
- Self-healing
- Service discovery
- Rolling updates/rollbacks

---

## Architecture

### Cluster Components

```
┌─────────────────────────────────────────────────┐
│              Kubernetes Cluster                 │
│                                                 │
│  ┌──────────────────────────────────────────┐  │
│  │         Control Plane                    │  │
│  │  ┌──────────┐  ┌──────────┐           │  │
│  │  │ API      │  │ etcd      │           │  │
│  │  │ Server   │  │ (Storage) │           │  │
│  │  └──────────┘  └──────────┘           │  │
│  │  ┌──────────┐  ┌──────────┐           │  │
│  │  │Scheduler │  │Controller│           │  │
│  │  │          │  │ Manager  │           │  │
│  │  └──────────┘  └──────────┘           │  │
│  └──────────────────────────────────────────┘  │
│                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │  Node 1  │  │  Node 2  │  │  Node 3  │    │
│  │ ┌──────┐ │  │ ┌──────┐ │  │ ┌──────┐ │    │
│  │ │Kubelet│ │  │ │Kubelet│ │  │ │Kubelet│ │    │
│  │ └──────┘ │  │ └──────┘ │  │ └──────┘ │    │
│  │ ┌──────┐ │  │ ┌──────┐ │  │ ┌──────┐ │    │
│  │ │Proxy │ │  │ │Proxy │ │  │ │Proxy │ │    │
│  │ └──────┘ │  │ └──────┘ │  │ └──────┘ │    │
│  │ ┌──────┐ │  │ ┌──────┐ │  │ ┌──────┐ │    │
│  │ │ Pods │ │  │ │ Pods │ │  │ │ Pods │ │    │
│  │ └──────┘ │  │ └──────┘ │  │ └──────┘ │    │
│  └──────────┘  └──────────┘  └──────────┘    │
└─────────────────────────────────────────────────┘
```

### Control Plane Components

1. **API Server:** Entry point for all operations
2. **etcd:** Distributed key-value store (cluster state)
3. **Scheduler:** Assigns pods to nodes
4. **Controller Manager:** Runs controllers (replicas, endpoints, etc.)

### Node Components

1. **Kubelet:** Agent that runs on each node
2. **kube-proxy:** Network proxy for service networking
3. **Container Runtime:** Docker, containerd, CRI-O

---

## Core Concepts

### 1. Pod

Smallest deployable unit. Contains one or more containers.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    ports:
    - containerPort: 8080
```

### 2. Deployment

Manages Pod replicas and provides rolling updates.

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

### 3. Service

Exposes Pods as network service.

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

### 4. Namespace

Logical separation of resources.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

---

## Pods

### Basic Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: myapp
    image: nginx:latest
    ports:
    - containerPort: 80
```

### Pod with Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    env:
    - name: DATABASE_URL
      value: "postgresql://db:5432/mydb"
    - name: NODE_ENV
      value: "production"
```

### Pod with Resource Limits

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
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

### Pod with Volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    volumeMounts:
    - name: config
      mountPath: /app/config
  volumes:
  - name: config
    configMap:
      name: app-config
```

### Multi-Container Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: myapp
    image: myapp:1.0
  - name: sidecar
    image: sidecar:1.0
```

---

## Deployments

### Basic Deployment

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

### Deployment with Rolling Update

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
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
```

### Deployment with Health Checks

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
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Deployment Commands

```bash
# Create deployment
kubectl apply -f deployment.yaml

# Get deployments
kubectl get deployments

# Describe deployment
kubectl describe deployment myapp-deployment

# Scale deployment
kubectl scale deployment myapp-deployment --replicas=5

# Update image
kubectl set image deployment/myapp-deployment myapp=myapp:2.0

# Rollout status
kubectl rollout status deployment/myapp-deployment

# Rollback
kubectl rollout undo deployment/myapp-deployment

# Rollback to specific revision
kubectl rollout undo deployment/myapp-deployment --to-revision=2

# View rollout history
kubectl rollout history deployment/myapp-deployment
```

---

## Services

### Service Types

1. **ClusterIP** (default) - Internal only
2. **NodePort** - Exposes on node IP
3. **LoadBalancer** - External load balancer
4. **ExternalName** - Maps to external DNS

### ClusterIP Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```

### NodePort Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080
```

### LoadBalancer Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```

### Service Commands

```bash
# Create service
kubectl apply -f service.yaml

# Get services
kubectl get services

# Describe service
kubectl describe service myapp-service

# Get endpoints
kubectl get endpoints myapp-service

# Port forward
kubectl port-forward service/myapp-service 8080:80
```

---

## ConfigMaps and Secrets

### ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgresql://db:5432/mydb"
  log_level: "INFO"
  max_connections: "100"
```

### Using ConfigMap in Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    envFrom:
    - configMapRef:
        name: app-config
    # Or individual keys
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    volumeMounts:
    - name: config
      mountPath: /app/config
  volumes:
  - name: config
    configMap:
      name: app-config
```

### Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  password: cGFzc3dvcmQ=  # base64 encoded
  api_key: YXBpX2tleQ==
```

### Create Secret from File

```bash
# Create from file
kubectl create secret generic app-secret \
  --from-file=password=./password.txt

# Create from literal
kubectl create secret generic app-secret \
  --from-literal=password=secret123
```

### Using Secret in Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    env:
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: password
    volumeMounts:
    - name: secret
      mountPath: /app/secrets
      readOnly: true
  volumes:
  - name: secret
    secret:
      secretName: app-secret
```

---

## Namespaces

### Create Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

### Using Namespace

```bash
# Create in namespace
kubectl apply -f deployment.yaml -n production

# Get resources in namespace
kubectl get pods -n production

# Set default namespace
kubectl config set-context --current --namespace=production
```

### Resource Quota

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
    pods: "50"
```

---

## Basic Operations

### Common kubectl Commands

```bash
# Get resources
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get configmaps
kubectl get secrets

# Describe resource
kubectl describe pod myapp-pod

# View logs
kubectl logs myapp-pod
kubectl logs -f myapp-pod  # follow

# Execute command
kubectl exec -it myapp-pod -- bash

# Delete resource
kubectl delete pod myapp-pod
kubectl delete deployment myapp-deployment

# Apply configuration
kubectl apply -f deployment.yaml

# Edit resource
kubectl edit deployment myapp-deployment

# Port forward
kubectl port-forward pod/myapp-pod 8080:8080
```

### Debugging

```bash
# Get events
kubectl get events --sort-by='.lastTimestamp'

# Describe pod (shows events)
kubectl describe pod myapp-pod

# Check pod logs
kubectl logs myapp-pod

# Previous container logs (if crashed)
kubectl logs myapp-pod --previous

# Interactive shell
kubectl exec -it myapp-pod -- /bin/bash
```

---

## Common Patterns

### Pattern 1: Complete Application

```yaml
# deployment.yaml
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
        env:
        - name: DATABASE_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: database_url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
---
# service.yaml
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
---
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgresql://db:5432/mydb"
```

### Pattern 2: Database with Persistent Volume

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
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
        image: postgres:15
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
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

---

## Summary

Key Concepts:
- ✅ **Pods:** Smallest deployable unit
- ✅ **Deployments:** Manage Pod replicas
- ✅ **Services:** Expose Pods as network service
- ✅ **ConfigMaps:** Configuration data
- ✅ **Secrets:** Sensitive data
- ✅ **Namespaces:** Logical separation

Next: [Kubernetes Advanced](./KUBERNETES_ADVANCED.md)

