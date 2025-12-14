# CI/CD Integration Guide

## Table of Contents
1. [Overview](#overview)
2. [GitHub Actions](#github-actions)
3. [GitLab CI](#gitlab-ci)
4. [Jenkins](#jenkins)
5. [ArgoCD](#argocd)
6. [Best Practices](#best-practices)
7. [Security](#security)

---

## Overview

CI/CD (Continuous Integration/Continuous Deployment) automates building, testing, and deploying applications to Kubernetes.

### CI/CD Pipeline Stages

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│  Build  │ -> │  Test   │ -> │  Deploy │ -> │ Monitor │
└─────────┘    └─────────┘    └─────────┘    └─────────┘
```

1. **Build:** Compile code and build Docker images
2. **Test:** Run unit tests, integration tests
3. **Deploy:** Deploy to Kubernetes
4. **Monitor:** Monitor deployment health

---

## GitHub Actions

### Basic Workflow

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up JDK
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
    
    - name: Build application
      run: mvn clean package -DskipTests
    
    - name: Run tests
      run: mvn test
    
    - name: Build Docker image
      run: docker build -t $IMAGE_NAME:${{ github.sha }} .
    
    - name: Push Docker image
      run: |
        echo "${{ secrets.GITHUB_TOKEN }}" | docker login $REGISTRY -u ${{ github.actor }} --password-stdin
        docker push $IMAGE_NAME:${{ github.sha }}
        docker tag $IMAGE_NAME:${{ github.sha }} $IMAGE_NAME:latest
        docker push $IMAGE_NAME:latest
```

### Deploy to Kubernetes

```yaml
# .github/workflows/deploy.yml
name: Deploy to Kubernetes

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Configure kubectl
      run: |
        echo "${{ secrets.KUBECONFIG }}" | base64 -d > kubeconfig
        export KUBECONFIG=kubeconfig
    
    - name: Deploy to Kubernetes
      run: |
        kubectl apply -f k8s/namespace.yaml
        kubectl apply -f k8s/configmaps.yaml
        kubectl apply -f k8s/secrets.yaml
        kubectl set image deployment/product-service \
          product-service=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
        kubectl rollout status deployment/product-service -n ecommerce
```

### Multi-Stage Deployment

```yaml
# .github/workflows/deploy-stages.yml
name: Deploy to Stages

on:
  push:
    branches: [main, develop]

jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Deploy to Staging
      run: ./deploy.sh staging
  
  deploy-production:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    needs: [deploy-staging]
    steps:
    - uses: actions/checkout@v3
    - name: Deploy to Production
      run: ./deploy.sh production
```

---

## GitLab CI

### Basic Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"
  REGISTRY: registry.gitlab.com
  IMAGE_NAME: $CI_REGISTRY_IMAGE

build:
  stage: build
  script:
    - mvn clean package -DskipTests
    - docker build -t $IMAGE_NAME:$CI_COMMIT_SHA .
    - docker push $IMAGE_NAME:$CI_COMMIT_SHA
  only:
    - main
    - develop

test:
  stage: test
  script:
    - mvn test
  only:
    - merge_requests

deploy-staging:
  stage: deploy
  script:
    - kubectl set image deployment/product-service \
        product-service=$IMAGE_NAME:$CI_COMMIT_SHA -n staging
    - kubectl rollout status deployment/product-service -n staging
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

deploy-production:
  stage: deploy
  script:
    - kubectl set image deployment/product-service \
        product-service=$IMAGE_NAME:$CI_COMMIT_SHA -n production
    - kubectl rollout status deployment/product-service -n production
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - main
```

---

## Jenkins

### Jenkinsfile

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        REGISTRY = 'registry.example.com'
        IMAGE_NAME = 'product-service'
        KUBECONFIG = credentials('kubeconfig')
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "${env.REGISTRY}/${env.IMAGE_NAME}:${env.BUILD_NUMBER}"
                    sh "docker build -t ${imageTag} ."
                    sh "docker tag ${imageTag} ${env.REGISTRY}/${env.IMAGE_NAME}:latest"
                }
            }
        }
        
        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh "echo ${PASSWORD} | docker login ${REGISTRY} -u ${USERNAME} --password-stdin"
                    sh "docker push ${env.REGISTRY}/${env.IMAGE_NAME}:${env.BUILD_NUMBER}"
                    sh "docker push ${env.REGISTRY}/${env.IMAGE_NAME}:latest"
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                sh "kubectl set image deployment/product-service product-service=${env.REGISTRY}/${env.IMAGE_NAME}:${env.BUILD_NUMBER} -n staging"
                sh "kubectl rollout status deployment/product-service -n staging"
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                sh "kubectl set image deployment/product-service product-service=${env.REGISTRY}/${env.IMAGE_NAME}:${env.BUILD_NUMBER} -n production"
                sh "kubectl rollout status deployment/product-service -n production"
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            cleanWs()
        }
    }
}
```

---

## ArgoCD

### Application Definition

```yaml
# argocd-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: product-service
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/k8s-manifests
    targetRevision: main
    path: product-service
  destination:
    server: https://kubernetes.default.svc
    namespace: ecommerce
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

### Sync Strategy

```yaml
spec:
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
    - CreateNamespace=true
    - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

---

## Best Practices

### 1. Use Semantic Versioning

```bash
# Tag images with semantic versions
docker tag myapp:$SHA myapp:v1.2.3
docker tag myapp:$SHA myapp:v1.2
docker tag myapp:$SHA myapp:latest
```

### 2. Separate Build and Deploy

```yaml
# Build job
build:
  runs-on: ubuntu-latest
  outputs:
    image-tag: ${{ github.sha }}
  steps:
    - name: Build and push
      run: |
        docker build -t myapp:${{ github.sha }} .
        docker push myapp:${{ github.sha }}

# Deploy job
deploy:
  needs: build
  runs-on: ubuntu-latest
  steps:
    - name: Deploy
      run: |
        kubectl set image deployment/myapp myapp=myapp:${{ needs.build.outputs.image-tag }}
```

### 3. Use Secrets Management

```yaml
# GitHub Actions
- name: Deploy
  env:
    KUBECONFIG: ${{ secrets.KUBECONFIG }}
    DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
  run: ./deploy.sh
```

### 4. Implement Rollback

```bash
#!/bin/bash
# rollback.sh

DEPLOYMENT=$1
NAMESPACE=$2

# Get current revision
CURRENT_REVISION=$(kubectl rollout history deployment/$DEPLOYMENT -n $NAMESPACE | tail -1 | awk '{print $1}')

# Rollback to previous revision
kubectl rollout undo deployment/$DEPLOYMENT -n $NAMESPACE

# Wait for rollout
kubectl rollout status deployment/$DEPLOYMENT -n $NAMESPACE
```

### 5. Health Checks

```yaml
# Wait for deployment to be healthy
- name: Wait for deployment
  run: |
    kubectl rollout status deployment/product-service -n ecommerce --timeout=5m
    
    # Verify health endpoint
    kubectl wait --for=condition=ready pod \
      -l app=product-service -n ecommerce --timeout=5m
```

---

## Security

### Scan Images

```yaml
# GitHub Actions
- name: Scan image
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: myapp:${{ github.sha }}
    format: 'sarif'
    output: 'trivy-results.sarif'
```

### Use Image Pull Secrets

```yaml
# Kubernetes deployment
spec:
  imagePullSecrets:
  - name: registry-secret
  containers:
  - name: app
    image: registry.example.com/app:1.0
```

### RBAC for CI/CD

```yaml
# Service account for CI/CD
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cicd-sa
  namespace: ecommerce
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cicd-role
  namespace: ecommerce
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "update", "patch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cicd-rolebinding
  namespace: ecommerce
subjects:
- kind: ServiceAccount
  name: cicd-sa
  namespace: ecommerce
roleRef:
  kind: Role
  name: cicd-role
  apiGroup: rbac.authorization.k8s.io
```

---

## Summary

CI/CD Best Practices:
- ✅ Automate build, test, and deploy
- ✅ Use semantic versioning
- ✅ Separate build and deploy stages
- ✅ Implement health checks
- ✅ Use secrets management
- ✅ Scan images for vulnerabilities
- ✅ Implement rollback strategy
- ✅ Use RBAC for security

Next: [Production Deployment](./PRODUCTION_DEPLOYMENT.md)

