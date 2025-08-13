# Docker Comprehensive Cheat Sheet

## Docker CLI Commands

### Image Management
```bash
# List images
docker images
docker image ls

# Pull an image
docker pull <image_name>:<tag>
docker pull ubuntu:22.04

# Build an image
docker build -t <image_name>:<tag> .
docker build -t myapp:v1.0 .

# Remove images
docker rmi <image_id>
docker image rm <image_name>:<tag>

# Remove all unused images
docker image prune -a

# Tag an image
docker tag <source_image> <target_image>:<tag>

# Push image to registry
docker push <image_name>:<tag>

# Search for images
docker search <image_name>
```

### Container Management
```bash
# Run a container
docker run <image_name>
docker run -d -p 8080:80 --name mycontainer nginx

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Start/Stop containers
docker start <container_id>
docker stop <container_id>
docker restart <container_id>

# Remove containers
docker rm <container_id>
docker container rm <container_name>

# Remove all stopped containers
docker container prune

# Execute commands in running container
docker exec -it <container_id> /bin/bash
docker exec -it <container_id> sh

# View container logs
docker logs <container_id>
docker logs -f <container_id>  # Follow logs

# Copy files between host and container
docker cp <container_id>:/path/to/file /host/path
docker cp /host/path <container_id>:/path/to/file

# Inspect container details
docker inspect <container_id>

# View container resource usage
docker stats
docker stats <container_id>
```

### Docker Run Options
```bash
# Common run flags
-d, --detach          # Run in background
-it                   # Interactive with TTY
-p, --publish         # Port mapping (host:container)
-P, --publish-all     # Publish all exposed ports
-v, --volume          # Volume mounting
--name                # Assign container name
--rm                  # Remove container when it exits
-e, --env             # Set environment variables
--network             # Connect to network
--restart             # Restart policy
--memory              # Memory limit
--cpus                # CPU limit

# Examples
docker run -d -p 3000:3000 -v /host/data:/app/data --name myapp node:18
docker run -it --rm -e NODE_ENV=production ubuntu:22.04 /bin/bash
```

### Volume Management
```bash
# List volumes
docker volume ls

# Create volume
docker volume create <volume_name>

# Inspect volume
docker volume inspect <volume_name>

# Remove volume
docker volume rm <volume_name>

# Remove unused volumes
docker volume prune

# Mount volume
docker run -v <volume_name>:/path/in/container <image>
docker run -v /host/path:/container/path <image>  # Bind mount
```

### Network Management
```bash
# List networks
docker network ls

# Create network
docker network create <network_name>
docker network create --driver bridge mynetwork

# Connect container to network
docker network connect <network_name> <container_name>

# Disconnect from network
docker network disconnect <network_name> <container_name>

# Inspect network
docker network inspect <network_name>

# Remove network
docker network rm <network_name>

# Remove unused networks
docker network prune
```

### System Commands
```bash
# System information
docker info
docker version

# Clean up everything
docker system prune -a  # Remove all unused containers, networks, images

# View disk usage
docker system df

# Remove everything (use with caution!)
docker system prune -a --volumes
```

## Dockerfile Guide

### Basic Structure
```dockerfile
# Use official base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001
USER nextjs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Start command
CMD ["npm", "start"]
```

### Dockerfile Instructions

#### Essential Instructions
```dockerfile
# Base image
FROM <image>:<tag>
FROM ubuntu:22.04
FROM node:18-alpine AS builder  # Multi-stage build

# Working directory
WORKDIR /path/to/directory

# Copy files
COPY <src> <dest>
COPY . .
COPY --from=builder /app/dist ./dist  # Multi-stage copy

# Add files (with extraction for archives)
ADD <src> <dest>

# Run commands
RUN apt-get update && apt-get install -y curl
RUN npm install

# Environment variables
ENV NODE_ENV=production
ENV PORT=3000

# Arguments (build-time variables)
ARG BUILD_VERSION=latest
ARG NODE_VERSION=18

# Expose ports
EXPOSE 3000
EXPOSE 8080 8443

# Volume mount points
VOLUME ["/data", "/logs"]

# User specification
USER nodejs
USER 1001:1001

# Labels
LABEL maintainer="your-email@example.com"
LABEL version="1.0.0"
LABEL description="My application"

# Entry point (always executed)
ENTRYPOINT ["docker-entrypoint.sh"]

# Default command
CMD ["npm", "start"]
CMD ["node", "server.js"]
```

#### Advanced Instructions
```dockerfile
# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/ || exit 1

# Signal handling
STOPSIGNAL SIGTERM

# Shell form vs Exec form
RUN echo "Shell form"
RUN ["echo", "Exec form"]

# Multi-line RUN with proper cleanup
RUN apt-get update && \
    apt-get install -y \
        curl \
        wget \
        vim && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### Multi-Stage Build Example
```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force
COPY --from=builder /app/dist ./dist
USER node
EXPOSE 3000
CMD ["npm", "start"]
```

## .dockerignore Guide

### Basic .dockerignore
```dockerignore
# Git
.git
.gitignore

# Dependencies
node_modules
npm-debug.log*

# Environment files
.env
.env.local
.env.*.local

# Build outputs
dist/
build/
.next/
out/

# Logs
*.log
logs/

# Runtime data
pids/
*.pid
*.seed
*.pid.lock

# Coverage directory used by tools like istanbul
coverage/

# IDE files
.vscode/
.idea/
*.swp
*.swo

# OS files
.DS_Store
Thumbs.db

# Test files
test/
tests/
*.test.js
*.spec.js

# Documentation
README.md
docs/

# Docker files (unless needed)
Dockerfile*
docker-compose*.yml

# Temporary files
tmp/
temp/
```

### Advanced .dockerignore Patterns
```dockerignore
# Use ! to include files after excluding parent directory
node_modules/
!node_modules/important-package/

# Wildcard patterns
**/*.tmp
**/.cache
**/temp*

# Comments are supported
# This is a comment

# Multiple extensions
*.{log,tmp,cache}
```

## Docker Compose Essentials

### Basic docker-compose.yml
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
    volumes:
      - ./app:/app
      - node_modules:/app/node_modules
    depends_on:
      - db
      - redis

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
  node_modules:
```

### Docker Compose Commands
```bash
# Start services
docker-compose up
docker-compose up -d  # Detached mode

# Stop services
docker-compose down
docker-compose down -v  # Remove volumes too

# Build services
docker-compose build
docker-compose build --no-cache

# View logs
docker-compose logs
docker-compose logs -f app  # Follow logs for specific service

# Execute commands
docker-compose exec app /bin/bash

# Scale services
docker-compose up --scale app=3

# View running services
docker-compose ps
```

## Best Practices

### Security
```dockerfile
# Use non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup
USER appuser

# Use specific image tags
FROM node:18.17.0-alpine  # Not 'latest'

# Minimize attack surface
FROM distroless/nodejs18-debian11  # Minimal base image

# Scan for vulnerabilities
# Use docker scan <image_name>
```

### Performance Optimization
```dockerfile
# Order layers by frequency of change (least to most)
FROM node:18-alpine
WORKDIR /app

# Dependencies first (cached if unchanged)
COPY package*.json ./
RUN npm ci --only=production

# Code last (changes most frequently)
COPY . .

# Use .dockerignore to reduce context size
# Multi-stage builds to reduce final image size

# Combine RUN commands to reduce layers
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### Image Size Optimization
```dockerfile
# Use Alpine Linux
FROM node:18-alpine

# Multi-stage builds
FROM node:18 AS builder
# ... build steps ...
FROM node:18-alpine AS production
COPY --from=builder /app/dist ./

# Remove unnecessary files
RUN npm ci --only=production && \
    npm cache clean --force

# Use distroless for runtime
FROM gcr.io/distroless/nodejs18-debian11
```

## Troubleshooting Commands

### Debugging
```bash
# Check container processes
docker exec <container> ps aux

# Check network connectivity
docker exec <container> ping <target>

# View container filesystem changes
docker diff <container>

# Export container as tar
docker export <container> > container.tar

# Import image from tar
docker import container.tar myimage:tag

# View image history
docker history <image>

# Check port mappings
docker port <container>
```

### Common Issues
```bash
# Container exits immediately
docker logs <container>  # Check logs
docker run -it <image> /bin/sh  # Debug interactively

# Port already in use
docker ps  # Check running containers
lsof -i :8080  # Check what's using the port

# Permission denied
# Check user permissions in Dockerfile
# Use correct user/group IDs

# Out of space
docker system df  # Check disk usage
docker system prune -a  # Clean up
```

## Environment Variables

### In Dockerfile
```dockerfile
# Set default values
ENV NODE_ENV=production
ENV PORT=3000
ENV DB_HOST=localhost

# Use ARG for build-time variables
ARG BUILD_VERSION=latest
ARG NODE_VERSION=18
```

### At Runtime
```bash
# Single variable
docker run -e NODE_ENV=production myapp

# Multiple variables
docker run -e NODE_ENV=production -e PORT=3000 myapp

# From file
docker run --env-file .env myapp
```

### .env File Format
```env
NODE_ENV=production
PORT=3000
DATABASE_URL=postgresql://user:pass@localhost:5432/db
API_KEY=your-secret-key
```

This cheat sheet covers the most important Docker concepts and commands you'll use in daily development. Keep it handy for quick reference!