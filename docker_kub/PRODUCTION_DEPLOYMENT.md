# Production Deployment Guide

## Table of Contents
1. [Pre-Production Checklist](#pre-production-checklist)
2. [Security Hardening](#security-hardening)
3. [High Availability](#high-availability)
4. [Disaster Recovery](#disaster-recovery)
5. [Performance Optimization](#performance-optimization)
6. [Monitoring & Alerting](#monitoring--alerting)
7. [Backup & Restore](#backup--restore)
8. [Rolling Updates](#rolling-updates)
9. [Blue-Green Deployment](#blue-green-deployment)
10. [Canary Deployment](#canary-deployment)

---

## Pre-Production Checklist

### Infrastructure
- [ ] Kubernetes cluster configured (3+ nodes)
- [ ] Load balancer configured
- [ ] DNS configured
- [ ] SSL/TLS certificates configured
- [ ] Network policies configured
- [ ] Resource quotas set
- [ ] Storage classes configured

### Application
- [ ] All services containerized
- [ ] Images pushed to registry
- [ ] Health checks implemented
- [ ] Logging configured
- [ ] Metrics exposed
- [ ] Error handling implemented
- [ ] Graceful shutdown implemented

### Security
- [ ] Secrets management configured
- [ ] RBAC configured
- [ ] Network policies applied
- [ ] Pod security policies applied
- [ ] Image scanning enabled
- [ ] TLS everywhere
- [ ] Non-root users configured

### Monitoring
- [ ] Prometheus configured
- [ ] Grafana dashboards created
- [ ] Alerting rules configured
- [ ] Log aggregation set up
- [ ] Distributed tracing configured

---

## Security Hardening

### Pod Security Standards

```yaml
# k8s/psp.yaml
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

### Network Policies

```yaml
# k8s/network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-gateway-policy
  namespace: ecommerce
spec:
  podSelector:
    matchLabels:
      app: api-gateway
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: user-service
    ports:
    - protocol: TCP
      port: 8081
  - to:
    - podSelector:
        matchLabels:
          app: product-service
    ports:
    - protocol: TCP
      port: 8082
```

### RBAC

```yaml
# k8s/rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: ecommerce
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: ecommerce
rules:
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: ecommerce
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: ecommerce
roleRef:
  kind: Role
  name: app-role
  apiGroup: rbac.authorization.k8s.io
```

### Image Security

```yaml
# Use image pull secrets
apiVersion: v1
kind: Pod
spec:
  imagePullSecrets:
  - name: registry-secret
  containers:
  - name: app
    image: registry.example.com/app:1.0
```

---

## High Availability

### Multi-Zone Deployment

```yaml
# k8s/deployment-ha.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service
spec:
  replicas: 6
  selector:
    matchLabels:
      app: product-service
  template:
    metadata:
      labels:
        app: product-service
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - product-service
              topologyKey: kubernetes.io/hostname
      containers:
      - name: product-service
        image: registry.example.com/product-service:1.0
```

### Pod Disruption Budget

```yaml
# k8s/pdb.yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: product-service-pdb
  namespace: ecommerce
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: product-service
```

### Database High Availability

```yaml
# k8s/postgres-ha.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 3
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
        - name: POSTGRES_REPLICATION_MODE
          value: "master"
```

---

## Disaster Recovery

### Backup Strategy

```bash
#!/bin/bash
# backup.sh

# Backup PostgreSQL
kubectl exec -n ecommerce postgres-0 -- \
  pg_dump -U postgres ecommerce > backup_$(date +%Y%m%d).sql

# Backup ConfigMaps
kubectl get configmaps -n ecommerce -o yaml > configmaps_backup.yaml

# Backup Secrets (encrypted)
kubectl get secrets -n ecommerce -o yaml > secrets_backup.yaml
```

### Restore Strategy

```bash
#!/bin/bash
# restore.sh

# Restore PostgreSQL
kubectl exec -i -n ecommerce postgres-0 -- \
  psql -U postgres ecommerce < backup_20230101.sql

# Restore ConfigMaps
kubectl apply -f configmaps_backup.yaml

# Restore Secrets
kubectl apply -f secrets_backup.yaml
```

### CronJob for Automated Backups

```yaml
# k8s/backup-cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: database-backup
  namespace: ecommerce
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: postgres:15
            command:
            - /bin/bash
            - -c
            - |
              pg_dump -h postgres-service -U postgres ecommerce > /backup/backup_$(date +%Y%m%d).sql
              # Upload to S3
              aws s3 cp /backup/backup_*.sql s3://backups/
            env:
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: db_password
            volumeMounts:
            - name: backup-volume
              mountPath: /backup
          volumes:
          - name: backup-volume
            persistentVolumeClaim:
              claimName: backup-pvc
          restartPolicy: OnFailure
```

---

## Performance Optimization

### Resource Requests and Limits

```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      containers:
      - name: app
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
```

### Horizontal Pod Autoscaler

```yaml
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: product-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: product-service
  minReplicas: 3
  maxReplicas: 20
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
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      - type: Pods
        value: 4
        periodSeconds: 15
      selectPolicy: Max
```

### Vertical Pod Autoscaler

```yaml
# k8s/vpa.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: product-service-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: product-service
  updatePolicy:
    updateMode: "Auto"
```

---

## Monitoring & Alerting

### Prometheus Rules

```yaml
# k8s/prometheus-rules.yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: ecommerce-alerts
spec:
  groups:
  - name: ecommerce
    rules:
    - alert: HighCPUUsage
      expr: rate(container_cpu_usage_seconds_total[5m]) > 0.8
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High CPU usage detected"
    
    - alert: HighMemoryUsage
      expr: container_memory_usage_bytes / container_spec_memory_limit_bytes > 0.9
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "High memory usage detected"
    
    - alert: PodCrashLooping
      expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Pod is crash looping"
    
    - alert: HighErrorRate
      expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High error rate detected"
```

### Grafana Dashboard

```json
{
  "dashboard": {
    "title": "E-Commerce Services",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])"
          }
        ]
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m])"
          }
        ]
      },
      {
        "title": "Response Time",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))"
          }
        ]
      }
    ]
  }
}
```

---

## Rolling Updates

### Deployment Strategy

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
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
        image: registry.example.com/product-service:2.0
        readinessProbe:
          httpGet:
            path: /actuator/health
            port: 8082
          initialDelaySeconds: 10
          periodSeconds: 5
          successThreshold: 1
          failureThreshold: 3
```

### Rollout Commands

```bash
# Update image
kubectl set image deployment/product-service \
  product-service=registry.example.com/product-service:2.0

# Monitor rollout
kubectl rollout status deployment/product-service

# Pause rollout
kubectl rollout pause deployment/product-service

# Resume rollout
kubectl rollout resume deployment/product-service

# Rollback
kubectl rollout undo deployment/product-service

# Rollback to specific revision
kubectl rollout undo deployment/product-service --to-revision=2

# View history
kubectl rollout history deployment/product-service
```

---

## Blue-Green Deployment

### Strategy

```yaml
# Blue deployment (current)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service-blue
spec:
  replicas: 5
  selector:
    matchLabels:
      app: product-service
      version: blue
  template:
    metadata:
      labels:
        app: product-service
        version: blue
    spec:
      containers:
      - name: product-service
        image: registry.example.com/product-service:1.0

# Green deployment (new)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service-green
spec:
  replicas: 5
  selector:
    matchLabels:
      app: product-service
      version: green
  template:
    metadata:
      labels:
        app: product-service
        version: green
    spec:
      containers:
      - name: product-service
        image: registry.example.com/product-service:2.0

# Service (switch between blue/green)
apiVersion: v1
kind: Service
metadata:
  name: product-service
spec:
  selector:
    app: product-service
    version: blue  # Switch to 'green' for new version
  ports:
  - port: 8082
```

### Switch Script

```bash
#!/bin/bash
# blue-green-switch.sh

CURRENT_VERSION=$(kubectl get svc product-service -o jsonpath='{.spec.selector.version}')

if [ "$CURRENT_VERSION" == "blue" ]; then
  NEW_VERSION="green"
else
  NEW_VERSION="blue"
fi

echo "Switching from $CURRENT_VERSION to $NEW_VERSION"

kubectl patch service product-service -p \
  "{\"spec\":{\"selector\":{\"version\":\"$NEW_VERSION\"}}}"

# Wait and verify
sleep 10
kubectl get pods -l version=$NEW_VERSION
```

---

## Canary Deployment

### Canary Deployment

```yaml
# Stable deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service-stable
spec:
  replicas: 9
  selector:
    matchLabels:
      app: product-service
      version: stable
  template:
    metadata:
      labels:
        app: product-service
        version: stable
    spec:
      containers:
      - name: product-service
        image: registry.example.com/product-service:1.0

# Canary deployment (10%)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: product-service
      version: canary
  template:
    metadata:
      labels:
        app: product-service
        version: canary
    spec:
      containers:
      - name: product-service
        image: registry.example.com/product-service:2.0

# Service (routes to both)
apiVersion: v1
kind: Service
metadata:
  name: product-service
spec:
  selector:
    app: product-service
  ports:
  - port: 8082
```

### Using Istio for Traffic Splitting

```yaml
# VirtualService for canary
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: product-service
spec:
  hosts:
  - product-service
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: product-service
        subset: canary
      weight: 100
  - route:
    - destination:
        host: product-service
        subset: stable
      weight: 90
    - destination:
        host: product-service
        subset: canary
      weight: 10
```

---

## Summary

Production deployment requires:
- ✅ Security hardening (RBAC, Network Policies, PSP)
- ✅ High availability (multi-zone, PDB)
- ✅ Disaster recovery (backups, restore)
- ✅ Performance optimization (HPA, VPA, resources)
- ✅ Monitoring and alerting
- ✅ Rolling updates strategy
- ✅ Blue-green or canary deployments

Next: [CI/CD Integration](./CICD_INTEGRATION.md)

