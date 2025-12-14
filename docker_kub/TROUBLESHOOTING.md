# Docker & Kubernetes Troubleshooting Guide

## Table of Contents
1. [Docker Troubleshooting](#docker-troubleshooting)
2. [Kubernetes Troubleshooting](#kubernetes-troubleshooting)
3. [Common Issues](#common-issues)
4. [Debugging Techniques](#debugging-techniques)
5. [Performance Issues](#performance-issues)
6. [Network Issues](#network-issues)
7. [Storage Issues](#storage-issues)
8. [Quick Reference](#quick-reference)

---

## Docker Troubleshooting

### Container Won't Start

```bash
# Check logs
docker logs container-name

# Check previous logs (if crashed)
docker logs container-name --previous

# Inspect container
docker inspect container-name

# Check events
docker events
```

### Container Keeps Restarting

```bash
# Check restart policy
docker inspect container-name | grep RestartPolicy

# Check exit code
docker inspect container-name | grep ExitCode

# Check logs for errors
docker logs container-name
```

### Image Pull Errors

```bash
# Check authentication
docker login

# Check image exists
docker pull image-name:tag

# Check network connectivity
docker pull image-name:tag --debug
```

### Out of Disk Space

```bash
# Check disk usage
docker system df

# Clean up
docker system prune -a

# Remove unused volumes
docker volume prune

# Remove unused images
docker image prune -a
```

### Permission Denied

```bash
# Check if user is in docker group
groups

# Add user to docker group
sudo usermod -aG docker $USER
# Log out and log back in

# Or use sudo
sudo docker command
```

---

## Kubernetes Troubleshooting

### Pod Not Starting

```bash
# Describe pod (shows events and errors)
kubectl describe pod pod-name

# Check pod status
kubectl get pod pod-name -o yaml

# Check events
kubectl get events --sort-by='.lastTimestamp'

# Check logs
kubectl logs pod-name

# Check previous container logs
kubectl logs pod-name --previous
```

### Pod CrashLoopBackOff

```bash
# Check logs
kubectl logs pod-name

# Check previous logs
kubectl logs pod-name --previous

# Describe pod
kubectl describe pod pod-name

# Common causes:
# - Application error
# - Resource limits exceeded
# - Health check failing
# - Configuration error
```

### Pod Pending

```bash
# Describe pod to see why
kubectl describe pod pod-name

# Common causes:
# - Insufficient resources
# - Node selector not matching
# - Taints/tolerations
# - PVC not bound

# Check node resources
kubectl top nodes

# Check node conditions
kubectl describe node node-name
```

### ImagePullBackOff

```bash
# Check image name
kubectl describe pod pod-name | grep Image

# Check image pull secrets
kubectl get secrets

# Create image pull secret
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=pass

# Add to pod spec
# imagePullSecrets:
# - name: regcred
```

### Service Not Accessible

```bash
# Check service endpoints
kubectl get endpoints service-name

# Describe service
kubectl describe service service-name

# Check pod labels match service selector
kubectl get pods --show-labels
kubectl get service service-name -o yaml | grep selector

# Test connectivity
kubectl run -it --rm debug --image=busybox --restart=Never -- \
  wget -O- http://service-name:port
```

---

## Common Issues

### Issue 1: High Memory Usage

```bash
# Check pod memory usage
kubectl top pods

# Check node memory
kubectl top nodes

# Describe pod for OOMKilled
kubectl describe pod pod-name | grep -i oom

# Solution: Increase memory limits
kubectl set resources deployment deployment-name \
  --limits=memory=2Gi
```

### Issue 2: High CPU Usage

```bash
# Check CPU usage
kubectl top pods
kubectl top nodes

# Check HPA status
kubectl get hpa

# Solution: Scale up or increase CPU limits
kubectl scale deployment deployment-name --replicas=5
```

### Issue 3: Network Connectivity Issues

```bash
# Test DNS
kubectl run -it --rm debug --image=busybox --restart=Never -- \
  nslookup service-name.namespace.svc.cluster.local

# Test connectivity
kubectl run -it --rm debug --image=busybox --restart=Never -- \
  wget -O- http://service-name:port

# Check network policies
kubectl get networkpolicies

# Check service endpoints
kubectl get endpoints service-name
```

### Issue 4: Storage Issues

```bash
# Check PVC status
kubectl get pvc

# Describe PVC
kubectl describe pvc pvc-name

# Check PV
kubectl get pv

# Common issues:
# - Storage class not found
# - Insufficient storage
# - Access mode mismatch
```

### Issue 5: Configuration Issues

```bash
# Check ConfigMap
kubectl get configmap configmap-name -o yaml

# Check Secret
kubectl get secret secret-name -o yaml

# Verify in pod
kubectl exec pod-name -- env | grep CONFIG_VAR
```

---

## Debugging Techniques

### Interactive Debugging

```bash
# Run debug pod
kubectl run -it --rm debug --image=busybox --restart=Never -- sh

# Execute command in pod
kubectl exec -it pod-name -- /bin/bash

# Execute command as root
kubectl exec -it pod-name -- /bin/bash -c "whoami"
```

### Port Forwarding

```bash
# Forward port to local machine
kubectl port-forward pod/pod-name 8080:8080

# Forward service port
kubectl port-forward service/service-name 8080:80

# Forward deployment port
kubectl port-forward deployment/deployment-name 8080:8080
```

### Log Collection

```bash
# Follow logs
kubectl logs -f pod-name

# Logs from all containers
kubectl logs pod-name --all-containers=true

# Logs with timestamps
kubectl logs pod-name --timestamps

# Last N lines
kubectl logs pod-name --tail=100

# Since timestamp
kubectl logs pod-name --since=1h
```

### Resource Inspection

```bash
# Get resource details
kubectl get pod pod-name -o yaml
kubectl get deployment deployment-name -o yaml
kubectl get service service-name -o yaml

# Export configuration
kubectl get deployment deployment-name -o yaml > deployment-backup.yaml
```

---

## Performance Issues

### Check Resource Usage

```bash
# Pod resources
kubectl top pods

# Node resources
kubectl top nodes

# Detailed metrics
kubectl describe pod pod-name | grep -A 5 "Limits\|Requests"
```

### Check HPA Status

```bash
# Get HPA
kubectl get hpa

# Describe HPA
kubectl describe hpa hpa-name

# Check if scaling is working
kubectl get pods -l app=myapp
```

### Check Pod Distribution

```bash
# Check pod distribution across nodes
kubectl get pods -o wide

# Check node capacity
kubectl describe nodes | grep -A 5 "Capacity\|Allocatable"
```

### Database Performance

```bash
# Connect to database pod
kubectl exec -it postgres-pod -- psql -U postgres

# Check connections
SELECT count(*) FROM pg_stat_activity;

# Check slow queries
SELECT * FROM pg_stat_statements ORDER BY total_time DESC LIMIT 10;
```

---

## Network Issues

### DNS Issues

```bash
# Test DNS resolution
kubectl run -it --rm debug --image=busybox --restart=Never -- \
  nslookup kubernetes.default

# Check CoreDNS
kubectl get pods -n kube-system | grep coredns
kubectl logs -n kube-system coredns-pod-name
```

### Service Discovery

```bash
# List services
kubectl get services

# Get service details
kubectl get service service-name -o yaml

# Check endpoints
kubectl get endpoints service-name

# Test service from pod
kubectl run -it --rm debug --image=busybox --restart=Never -- \
  wget -O- http://service-name.namespace.svc.cluster.local:port
```

### Ingress Issues

```bash
# Check ingress
kubectl get ingress

# Describe ingress
kubectl describe ingress ingress-name

# Check ingress controller
kubectl get pods -n ingress-nginx

# Check ingress controller logs
kubectl logs -n ingress-nginx ingress-nginx-controller-pod-name
```

---

## Storage Issues

### PVC Not Binding

```bash
# Check PVC status
kubectl get pvc

# Describe PVC
kubectl describe pvc pvc-name

# Check storage classes
kubectl get storageclass

# Check PV
kubectl get pv
```

### Volume Mount Issues

```bash
# Check pod volume mounts
kubectl describe pod pod-name | grep -A 10 "Volumes\|Mounts"

# Verify mount in pod
kubectl exec pod-name -- ls -la /mount/path

# Check permissions
kubectl exec pod-name -- ls -la /mount/path
```

---

## Quick Reference

### Common Commands

```bash
# Get all resources
kubectl get all

# Describe resource
kubectl describe <resource> <name>

# Get logs
kubectl logs <pod-name>

# Execute command
kubectl exec -it <pod-name> -- <command>

# Port forward
kubectl port-forward <resource>/<name> <local-port>:<remote-port>

# Get events
kubectl get events --sort-by='.lastTimestamp'

# Check resource usage
kubectl top pods
kubectl top nodes
```

### Debug Pods

```bash
# Busybox debug pod
kubectl run -it --rm debug --image=busybox --restart=Never -- sh

# Curl debug pod
kubectl run -it --rm debug --image=curlimages/curl --restart=Never -- sh

# Network debug pod
kubectl run -it --rm debug --image=nicolaka/netshoot --restart=Never -- sh
```

### Useful One-Liners

```bash
# Delete all pods in namespace
kubectl delete pods --all -n namespace

# Get all resources in namespace
kubectl get all -n namespace

# Watch pods
kubectl get pods -w

# Get pod logs with timestamps
kubectl logs pod-name --timestamps | tail -100

# Check pod resource usage
kubectl top pod pod-name --containers

# Get pod IP
kubectl get pod pod-name -o jsonpath='{.status.podIP}'

# Get service endpoints
kubectl get endpoints service-name -o jsonpath='{.subsets[*].addresses[*].ip}'
```

### Troubleshooting Checklist

1. **Pod Issues:**
   - [ ] Check pod status: `kubectl get pods`
   - [ ] Describe pod: `kubectl describe pod pod-name`
   - [ ] Check logs: `kubectl logs pod-name`
   - [ ] Check events: `kubectl get events`

2. **Service Issues:**
   - [ ] Check service: `kubectl get svc`
   - [ ] Check endpoints: `kubectl get endpoints`
   - [ ] Test connectivity: `kubectl run debug --image=busybox`

3. **Network Issues:**
   - [ ] Check DNS: `nslookup service-name`
   - [ ] Check ingress: `kubectl get ingress`
   - [ ] Check network policies: `kubectl get networkpolicies`

4. **Storage Issues:**
   - [ ] Check PVC: `kubectl get pvc`
   - [ ] Check PV: `kubectl get pv`
   - [ ] Check storage class: `kubectl get storageclass`

5. **Resource Issues:**
   - [ ] Check resource usage: `kubectl top pods`
   - [ ] Check limits: `kubectl describe pod pod-name`
   - [ ] Check HPA: `kubectl get hpa`

---

## Summary

Troubleshooting steps:
1. ✅ Check resource status (`kubectl get`)
2. ✅ Describe resource (`kubectl describe`)
3. ✅ Check logs (`kubectl logs`)
4. ✅ Check events (`kubectl get events`)
5. ✅ Use debug pods for testing
6. ✅ Port forward for local testing
7. ✅ Check resource usage
8. ✅ Verify configurations

For more details, see:
- [Docker Fundamentals](./DOCKER_FUNDAMENTALS.md)
- [Kubernetes Fundamentals](./KUBERNETES_FUNDAMENTALS.md)
- [Production Deployment](./PRODUCTION_DEPLOYMENT.md)

