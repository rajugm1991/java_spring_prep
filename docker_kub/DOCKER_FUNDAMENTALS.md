# Docker Fundamentals - Complete Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Core Concepts](#core-concepts)
4. [Dockerfile Deep Dive](#dockerfile-deep-dive)
5. [Image Management](#image-management)
6. [Container Management](#container-management)
7. [Networking](#networking)
8. [Volumes and Data Management](#volumes-and-data-management)
9. [Docker Registry](#docker-registry)
10. [Practical Examples](#practical-examples)

---

## Introduction

### What is Docker?

Docker is a platform that uses containerization to package applications with all their dependencies into lightweight, portable containers that run consistently across different environments.

### Why Docker?

**Before Docker:**
- "It works on my machine" problem
- Complex dependency management
- Inconsistent environments
- Difficult deployment processes

**With Docker:**
- Consistent environments (dev, staging, prod)
- Easy dependency management
- Fast deployment
- Resource efficiency
- Scalability

### Container vs Virtual Machine

```
Virtual Machine:
┌─────────────────────────────────────┐
│  Application A                     │
│  ┌─────────────────────────────┐  │
│  │ Guest OS (Ubuntu)           │  │
│  │ Libraries & Dependencies    │  │
│  └─────────────────────────────┘  │
│  ┌─────────────────────────────┐  │
│  │ Hypervisor                  │  │
│  └─────────────────────────────┘  │
│  ┌─────────────────────────────┐  │
│  │ Host OS                      │  │
│  └─────────────────────────────┘  │
│  ┌─────────────────────────────┐  │
│  │ Hardware                     │  │
│  └─────────────────────────────┘  │
└─────────────────────────────────────┘
Size: GBs | Boot Time: Minutes | Overhead: High

Container:
┌─────────────────────────────────────┐
│  Application A    Application B     │
│  ┌─────────────┐ ┌─────────────┐  │
│  │ Libraries   │ │ Libraries   │  │
│  └─────────────┘ └─────────────┘  │
│  ┌─────────────────────────────┐  │
│  │ Docker Engine                │  │
│  └─────────────────────────────┘  │
│  ┌─────────────────────────────┐  │
│  │ Host OS                      │  │
│  └─────────────────────────────┘  │
│  ┌─────────────────────────────┐  │
│  │ Hardware                     │  │
│  └─────────────────────────────┘  │
└─────────────────────────────────────┘
Size: MBs | Boot Time: Seconds | Overhead: Low
```

---

## Installation

### Linux (Ubuntu/Debian)

```bash
# Update package index
sudo apt-get update

# Install prerequisites
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Add Docker's official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set up repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Start Docker
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group (optional, to run without sudo)
sudo usermod -aG docker $USER
```

### macOS

```bash
# Using Homebrew
brew install --cask docker

# Or download Docker Desktop from:
# https://www.docker.com/products/docker-desktop
```

### Windows

1. Download Docker Desktop from https://www.docker.com/products/docker-desktop
2. Install and restart
3. Enable WSL 2 backend (recommended)

### Verify Installation

```bash
docker --version
docker-compose --version
docker run hello-world
```

---

## Core Concepts

### 1. Image

An image is a read-only template used to create containers. It contains:
- Application code
- Runtime environment
- Dependencies
- Configuration files

**Image Layers:**
```
┌─────────────────────────┐
│   Application Layer      │  ← Your changes
├─────────────────────────┤
│   Dependency Layer      │  ← npm install
├─────────────────────────┤
│   Runtime Layer         │  ← Node.js
├─────────────────────────┤
│   Base OS Layer          │  ← Ubuntu/Alpine
└─────────────────────────┘
```

### 2. Container

A container is a running instance of an image. Multiple containers can run from the same image.

### 3. Dockerfile

A Dockerfile contains instructions to build an image.

### 4. Registry

A registry stores Docker images (Docker Hub, AWS ECR, Google GCR, Azure ACR).

---

## Dockerfile Deep Dive

### Basic Structure

```dockerfile
# Base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Set environment variable
ENV NODE_ENV=production

# Run command
CMD ["node", "server.js"]
```

### Common Instructions

#### FROM
```dockerfile
# Use specific version (recommended)
FROM node:18-alpine

# Avoid 'latest' tag
# FROM node:latest  ❌
```

#### WORKDIR
```dockerfile
# Sets working directory and creates it if doesn't exist
WORKDIR /app
# Equivalent to: RUN mkdir -p /app && cd /app
```

#### COPY vs ADD
```dockerfile
# COPY - Preferred (simpler)
COPY package.json .
COPY src ./src

# ADD - Has extra features (URL, tar extraction)
ADD https://example.com/file.tar.gz /tmp/  # Can download
ADD file.tar.gz /tmp/  # Can extract automatically
```

#### RUN
```dockerfile
# Chain commands to reduce layers
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*

# Use && to ensure previous command succeeds
```

#### CMD vs ENTRYPOINT
```dockerfile
# CMD - Default command (can be overridden)
CMD ["node", "server.js"]
# docker run myimage → runs: node server.js
# docker run myimage npm start → overrides: npm start

# ENTRYPOINT - Always executed (cannot be overridden)
ENTRYPOINT ["node"]
CMD ["server.js"]
# docker run myimage → runs: node server.js
# docker run myimage app.js → runs: node app.js
```

#### ENV
```dockerfile
# Set environment variable
ENV NODE_ENV=production
ENV PORT=3000

# Or use ARG for build-time variables
ARG BUILD_VERSION
ENV VERSION=$BUILD_VERSION
```

#### EXPOSE
```dockerfile
# Document which port the container listens on
EXPOSE 3000
# Note: Doesn't actually publish the port
# Use -p flag when running: docker run -p 3000:3000
```

### Multi-Stage Builds

```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /build
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Stage 2: Runtime
FROM node:18-alpine
WORKDIR /app
# Copy only built files from builder stage
COPY --from=builder /build/dist ./dist
COPY --from=builder /build/node_modules ./node_modules
COPY package*.json ./
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

**Benefits:**
- Smaller final image (no build tools)
- Better security (fewer dependencies)
- Faster builds (cached layers)

### Best Practices

```dockerfile
# 1. Use specific base image
FROM node:18-alpine  # ✅
# FROM node:latest    # ❌

# 2. Use .dockerignore
# .dockerignore
node_modules
.git
*.log
.env

# 3. Order instructions by change frequency
# Dependencies change less → copy first
COPY package*.json ./
RUN npm install
# Code changes more → copy last
COPY . .

# 4. Use non-root user
RUN addgroup -g 1001 -S appuser && \
    adduser -S appuser -u 1001
USER appuser

# 5. Add health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1

# 6. Use exec form for CMD
CMD ["node", "server.js"]  # ✅
# CMD node server.js       # ❌ (shell form)
```

---

## Image Management

### Building Images

```bash
# Basic build
docker build -t myapp:1.0 .

# Build with tag
docker build -t myapp:latest -t myapp:1.0 .

# Build with build args
docker build --build-arg VERSION=1.0 -t myapp:1.0 .

# Build from different Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# Build without cache
docker build --no-cache -t myapp:1.0 .
```

### Tagging Images

```bash
# Tag image
docker tag myapp:1.0 myregistry.com/myapp:1.0

# Tag with multiple tags
docker tag myapp:1.0 myapp:latest
docker tag myapp:1.0 myapp:v1.0.0
```

### Listing Images

```bash
# List all images
docker images

# List with filters
docker images | grep myapp
docker images --filter "dangling=true"

# Show image size
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

### Removing Images

```bash
# Remove image
docker rmi myapp:1.0

# Force remove (even if in use)
docker rmi -f myapp:1.0

# Remove dangling images
docker image prune

# Remove all unused images
docker image prune -a
```

### Inspecting Images

```bash
# Inspect image
docker inspect myapp:1.0

# Show image history
docker history myapp:1.0

# Show image layers
docker image inspect myapp:1.0 --format='{{.RootFS.Layers}}'
```

---

## Container Management

### Running Containers

```bash
# Run container
docker run nginx

# Run in detached mode
docker run -d nginx

# Run with name
docker run -d --name mynginx nginx

# Run with port mapping
docker run -d -p 8080:80 nginx

# Run with environment variables
docker run -d -e NODE_ENV=production myapp

# Run with volume
docker run -d -v /host/path:/container/path myapp

# Run with resource limits
docker run -d --memory="512m" --cpus="1.0" myapp

# Run and remove on exit
docker run --rm myapp

# Run with custom command
docker run nginx nginx -v
```

### Container Lifecycle

```bash
# Start container
docker start mycontainer

# Stop container
docker stop mycontainer

# Restart container
docker restart mycontainer

# Pause container
docker pause mycontainer

# Unpause container
docker unpause mycontainer

# Remove container
docker rm mycontainer

# Remove running container (force)
docker rm -f mycontainer
```

### Listing Containers

```bash
# List running containers
docker ps

# List all containers
docker ps -a

# List with format
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"

# List container IDs only
docker ps -q
```

### Container Logs

```bash
# View logs
docker logs mycontainer

# Follow logs
docker logs -f mycontainer

# Last N lines
docker logs --tail 100 mycontainer

# Since timestamp
docker logs --since 2023-01-01T00:00:00 mycontainer

# Until timestamp
docker logs --until 2023-01-01T12:00:00 mycontainer
```

### Executing Commands

```bash
# Execute command in running container
docker exec mycontainer ls

# Interactive shell
docker exec -it mycontainer /bin/bash

# Execute as specific user
docker exec -u root mycontainer whoami
```

### Container Stats

```bash
# Real-time stats
docker stats

# Stats for specific container
docker stats mycontainer

# No-stream (one snapshot)
docker stats --no-stream
```

---

## Networking

### Default Networks

Docker creates three networks by default:

```bash
# List networks
docker network ls

# Inspect network
docker network inspect bridge
```

1. **bridge** - Default network for containers
2. **host** - Uses host's network directly
3. **none** - No networking

### Creating Networks

```bash
# Create network
docker network create mynetwork

# Create with driver
docker network create --driver bridge mynetwork

# Create with subnet
docker network create --subnet=172.20.0.0/16 mynetwork
```

### Connecting Containers

```bash
# Connect container to network
docker network connect mynetwork mycontainer

# Disconnect container
docker network disconnect mynetwork mycontainer

# Run container on specific network
docker run -d --network=mynetwork myapp
```

### Container Communication

```bash
# Containers on same network can communicate by name
docker run -d --name app1 --network=mynetwork myapp
docker run -d --name app2 --network=mynetwork myapp
# app2 can reach app1 at: http://app1:3000
```

### Port Mapping

```bash
# Map port
docker run -p 8080:80 nginx
# Host:Container

# Map all ports
docker run -P nginx
# Maps all EXPOSE ports randomly

# Bind to specific IP
docker run -p 127.0.0.1:8080:80 nginx

# Map UDP port
docker run -p 8080:80/udp nginx
```

---

## Volumes and Data Management

### Types of Storage

1. **Bind Mount** - Mount host directory
2. **Volume** - Managed by Docker
3. **tmpfs** - In-memory storage

### Bind Mounts

```bash
# Mount host directory
docker run -v /host/path:/container/path myapp

# Mount with read-only
docker run -v /host/path:/container/path:ro myapp
```

### Volumes

```bash
# Create volume
docker volume create myvolume

# List volumes
docker volume ls

# Inspect volume
docker volume inspect myvolume

# Use volume
docker run -v myvolume:/data myapp

# Remove volume
docker volume rm myvolume

# Remove unused volumes
docker volume prune
```

### tmpfs

```bash
# In-memory storage
docker run --tmpfs /tmp myapp

# With options
docker run --tmpfs /tmp:rw,noexec,nosuid,size=100m myapp
```

### Data Persistence Example

```bash
# Database with persistent volume
docker run -d \
  --name postgres \
  -v postgres_data:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=password \
  postgres:15

# Backup volume
docker run --rm \
  -v postgres_data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/postgres_backup.tar.gz /data

# Restore volume
docker run --rm \
  -v postgres_data:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/postgres_backup.tar.gz -C /
```

---

## Docker Registry

### Docker Hub

```bash
# Login
docker login

# Push image
docker push username/myapp:1.0

# Pull image
docker pull username/myapp:1.0
```

### Private Registry

```bash
# Run local registry
docker run -d -p 5000:5000 --name registry registry:2

# Tag for local registry
docker tag myapp:1.0 localhost:5000/myapp:1.0

# Push to local registry
docker push localhost:5000/myapp:1.0

# Pull from local registry
docker pull localhost:5000/myapp:1.0
```

### AWS ECR

```bash
# Login to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com

# Tag image
docker tag myapp:1.0 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:1.0

# Push to ECR
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:1.0
```

---

## Practical Examples

### Example 1: Spring Boot Application

```dockerfile
# Dockerfile
FROM openjdk:17-jdk-slim AS builder
WORKDIR /build
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

FROM openjdk:17-jre-slim
WORKDIR /app
COPY --from=builder /build/target/myapp.jar app.jar
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:8080/actuator/health || exit 1
CMD ["java", "-jar", "app.jar"]
```

```bash
# Build
docker build -t myapp:1.0 .

# Run
docker run -d -p 8080:8080 --name myapp myapp:1.0
```

### Example 2: Node.js Application

```dockerfile
# Dockerfile
FROM node:18-alpine AS builder
WORKDIR /build
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /build/node_modules ./node_modules
COPY . .
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs
EXPOSE 3000
CMD ["node", "server.js"]
```

### Example 3: Python Application

```dockerfile
# Dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]
```

---

## Summary

Key Takeaways:
- ✅ Use specific base images (not `latest`)
- ✅ Leverage layer caching (copy dependencies first)
- ✅ Use multi-stage builds for smaller images
- ✅ Run as non-root user
- ✅ Add health checks
- ✅ Use volumes for persistent data
- ✅ Use networks for container communication
- ✅ Tag images properly for versioning

Next: [Docker Best Practices](./DOCKER_BEST_PRACTICES.md)

