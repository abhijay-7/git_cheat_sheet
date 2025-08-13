# Docker Compose In-Depth Cheat Sheet

## Table of Contents
1. [Basic Commands](#basic-commands)
2. [Compose File Structure](#compose-file-structure)
3. [Services Configuration](#services-configuration)
4. [Networks](#networks)
5. [Volumes](#volumes)
6. [Environment Variables](#environment-variables)
7. [Profiles](#profiles)
8. [Extensions and Anchors](#extensions-and-anchors)
9. [Multi-Stage and Override Files](#multi-stage-and-override-files)
10. [Advanced Patterns](#advanced-patterns)
11. [Troubleshooting](#troubleshooting)

## Basic Commands

### Essential Commands
```bash
# Start services
docker-compose up
docker-compose up -d                    # Detached mode
docker-compose up --build               # Force rebuild
docker-compose up --scale app=3         # Scale specific service
docker-compose up service1 service2     # Start specific services

# Stop and remove
docker-compose down                     # Stop and remove containers
docker-compose down -v                  # Remove volumes too
docker-compose down --rmi all           # Remove images too
docker-compose down --remove-orphans    # Remove orphaned containers

# Build
docker-compose build                    # Build all services
docker-compose build --no-cache app     # Build specific service without cache
docker-compose build --parallel         # Build in parallel

# Service management
docker-compose start                    # Start existing containers
docker-compose stop                     # Stop running containers
docker-compose restart                  # Restart containers
docker-compose pause                    # Pause containers
docker-compose unpause                  # Unpause containers

# Logs and monitoring
docker-compose logs                     # View logs for all services
docker-compose logs -f app              # Follow logs for specific service
docker-compose logs --tail=100 app      # Last 100 lines
docker-compose logs -t app              # Show timestamps

# Execute commands
docker-compose exec app bash            # Execute command in running container
docker-compose run --rm app npm test    # Run one-off command
docker-compose run --rm --no-deps app bash  # Run without dependencies

# Status and inspection
docker-compose ps                       # List containers
docker-compose ps -q                    # Quiet mode (IDs only)
docker-compose top                      # Display running processes
docker-compose port app 3000            # Show port mapping

# Configuration
docker-compose config                   # Validate and view config
docker-compose config --services        # List services
docker-compose config --volumes         # List volumes

# Images and cleanup
docker-compose images                   # List images
docker-compose pull                     # Pull latest images
docker-compose push                     # Push images to registry
```

### File Management
```bash
# Specify compose file
docker-compose -f docker-compose.prod.yml up
docker-compose -f compose.yml -f compose.override.yml up

# Multiple files
docker-compose -f base.yml -f override.yml -f local.yml up

# Project name
docker-compose -p myproject up          # Custom project name
COMPOSE_PROJECT_NAME=myapp docker-compose up
```

## Compose File Structure

### Version and Basic Structure
```yaml
version: '3.8'  # Compose file format version

services:
  # Service definitions
  
networks:
  # Network definitions (optional)
  
volumes:
  # Volume definitions (optional)

configs:
  # Config definitions (optional)

secrets:
  # Secret definitions (optional)
```

### Complete Example Structure
```yaml
version: '3.8'

x-common-variables: &common-variables
  NODE_ENV: production
  LOG_LEVEL: info

x-app-common: &app-common
  restart: unless-stopped
  networks:
    - app-network
  depends_on:
    - db
    - redis

services:
  app:
    <<: *app-common
    build:
      context: .
      dockerfile: Dockerfile.prod
      args:
        NODE_VERSION: 18
    ports:
      - "3000:3000"
    environment:
      <<: *common-variables
      DATABASE_URL: postgresql://user:pass@db:5432/mydb
    volumes:
      - ./app:/app:ro
      - node_modules:/app/node_modules
      - app-logs:/app/logs
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  worker:
    <<: *app-common
    build:
      context: .
      dockerfile: Dockerfile.worker
    command: ["npm", "run", "worker"]
    environment:
      <<: *common-variables
      WORKER_CONCURRENCY: 4
    volumes:
      - ./app:/app:ro
      - node_modules:/app/node_modules
    deploy:
      replicas: 2
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    ports:
      - "5432:5432"
    networks:
      - app-network
    restart: unless-stopped
    secrets:
      - db_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d mydb"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes --requirepass mypassword
    volumes:
      - redis_data:/data
      - ./redis.conf:/usr/local/etc/redis/redis.conf:ro
    networks:
      - app-network
    restart: unless-stopped
    sysctls:
      - net.core.somaxconn=65535

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
      - static_files:/var/www/static:ro
    networks:
      - app-network
    depends_on:
      - app
    restart: unless-stopped

volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local
  node_modules:
    driver: local
  app-logs:
    driver: local
  static_files:
    driver: local

networks:
  app-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16

secrets:
  db_password:
    file: ./secrets/db_password.txt

configs:
  nginx_config:
    file: ./nginx.conf
```

## Services Configuration

### Build Options
```yaml
services:
  app:
    # Simple build
    build: .
    
    # Advanced build
    build:
      context: .                        # Build context
      dockerfile: Dockerfile.prod       # Custom dockerfile
      args:                            # Build arguments
        NODE_VERSION: 18
        BUILD_ENV: production
      target: production               # Multi-stage target
      cache_from:                      # Cache sources
        - myregistry/myapp:cache
      extra_hosts:                     # Add host entries
        - "somehost:162.242.195.82"
      isolation: process               # Windows isolation
      network: host                    # Network mode during build
      shm_size: 64M                   # Shared memory size
      labels:                          # Build labels
        - "com.example.version=1.0"
```

### Image and Container Options
```yaml
services:
  app:
    image: node:18-alpine
    container_name: my-app-container
    hostname: app-server
    domainname: example.com
    
    # Command and entrypoint
    command: ["npm", "start"]
    entrypoint: ["/docker-entrypoint.sh"]
    
    # Working directory and user
    working_dir: /app
    user: "1000:1000"
    
    # Platform specification
    platform: linux/amd64
    
    # Init process
    init: true
    
    # Privileges and capabilities
    privileged: true
    cap_add:
      - SYS_ADMIN
    cap_drop:
      - NET_ADMIN
    
    # Security options
    security_opt:
      - seccomp:unconfined
      - apparmor:unconfined
```

### Resource Management
```yaml
services:
  app:
    # Memory and CPU limits
    mem_limit: 512M
    mem_reservation: 256M
    memswap_limit: 1G
    mem_swappiness: 60
    
    cpus: 1.5
    cpu_shares: 1024
    cpu_quota: 50000
    cpu_period: 100000
    
    # Device mapping
    devices:
      - "/dev/sda:/dev/xvda:rwm"
    
    # Ulimits
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
      memlock: -1
    
    # Process limits
    pids_limit: 100
    
    # OOM killer
    oom_kill_disable: true
    oom_score_adj: -500
```

### Health Checks
```yaml
services:
  app:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      # Alternative formats:
      # test: curl -f http://localhost:3000/health
      # test: ["CMD-SHELL", "curl -f http://localhost:3000/health || exit 1"]
      
      interval: 30s                    # Check interval
      timeout: 10s                     # Timeout for each check
      retries: 3                       # Failed retries before unhealthy
      start_period: 40s                # Grace period
      
    # Disable healthcheck
    # healthcheck:
    #   disable: true
```

### Dependencies
```yaml
services:
  web:
    depends_on:
      - db
      - redis
    
    # Long form with conditions (Compose 3.8+)
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
```

### Restart Policies
```yaml
services:
  app:
    restart: "no"                      # Never restart
    restart: always                    # Always restart
    restart: on-failure               # Restart on failure
    restart: on-failure:3             # Max 3 restart attempts
    restart: unless-stopped           # Restart unless manually stopped
```

## Networks

### Network Types
```yaml
networks:
  # Default bridge network
  frontend:
    driver: bridge
    
  # Custom bridge with options
  backend:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: custom-bridge
    ipam:
      config:
        - subnet: 172.28.0.0/16
          ip_range: 172.28.5.0/24
          gateway: 172.28.5.254
    labels:
      com.example.description: "Backend network"
    
  # Overlay network for swarm
  overlay-net:
    driver: overlay
    attachable: true
    
  # Host network
  host-net:
    driver: host
    
  # External network
  external-net:
    external: true
    name: my-existing-network

services:
  app:
    networks:
      - frontend
      - backend
      
  db:
    networks:
      backend:
        aliases:                       # Network aliases
          - database
          - db-server
        ipv4_address: 172.28.1.10     # Static IP
```

### Network Configuration
```yaml
services:
  app:
    # Network mode
    network_mode: "bridge"             # bridge, host, none, service:name
    network_mode: "host"
    network_mode: "service:db"
    
    # DNS configuration
    dns:
      - 8.8.8.8
      - 1.1.1.1
    dns_search:
      - example.com
    dns_opt:
      - use-vc
      - no-tld-query
      
    # Extra hosts
    extra_hosts:
      - "host.docker.internal:host-gateway"
      - "api.example.com:192.168.1.100"
    
    # External links (deprecated)
    external_links:
      - redis_1
      - project_db_1:database
      
    # Links (deprecated)
    links:
      - db
      - db:database
```

## Volumes

### Volume Types
```yaml
volumes:
  # Named volume with default driver
  postgres_data:
  
  # Named volume with custom driver
  nfs_data:
    driver: local
    driver_opts:
      type: nfs
      o: addr=192.168.1.100,rw
      device: ":/path/to/dir"
  
  # External volume
  external_data:
    external: true
    name: my-existing-volume

services:
  app:
    volumes:
      # Named volume
      - postgres_data:/var/lib/postgresql/data
      
      # Bind mount
      - ./app:/app                     # Read-write
      - ./config:/etc/config:ro        # Read-only
      
      # Anonymous volume
      - /app/node_modules
      
      # Tmpfs mount
      - type: tmpfs
        target: /tmp
        tmpfs:
          size: 100M
          mode: 1777
      
      # Volume with options
      - type: volume
        source: postgres_data
        target: /var/lib/postgresql/data
        volume:
          nocopy: true
      
      # Bind mount with options
      - type: bind
        source: ./app
        target: /app
        bind:
          propagation: rprivate
        read_only: true
        
      # Named pipe (Windows)
      - type: npipe
        source: \\.\pipe\docker_engine
        target: \\.\pipe\docker_engine
```

### Volume Options
```yaml
services:
  app:
    volumes:
      - ./data:/data:rw,z              # SELinux label
      - ./logs:/logs:ro,Z              # SELinux private label
      - cache:/cache:delegated         # macOS performance
      - ./src:/src:cached              # macOS performance
      - ./build:/build:consistent      # Default consistency
```

## Environment Variables

### Variable Definition
```yaml
services:
  app:
    # Simple list
    environment:
      - NODE_ENV=production
      - DEBUG=true
      - PORT=3000
    
    # Map format
    environment:
      NODE_ENV: production
      DEBUG: "true"                    # Quoted to ensure string
      PORT: 3000
      EMPTY_VAR:                      # Empty value
    
    # From file
    env_file:
      - .env
      - .env.local
      - ./config/.env.production
    
    # Multiple files (latter override former)
    env_file:
      - .env.base
      - .env.override
```

### Environment File Formats
```bash
# .env file format
NODE_ENV=production
PORT=3000
DEBUG=true

# Comments are supported
# DATABASE_URL=postgresql://localhost:5432/db

# Quotes are optional but recommended for complex values
SECRET_KEY="my-secret-key-with-special-chars!@#"
JSON_CONFIG='{"key": "value", "number": 123}'

# Multiline values (not supported in compose)
# Use environment section instead
```

### Variable Interpolation
```yaml
services:
  app:
    image: "webapp:${TAG:-latest}"     # Default value
    ports:
      - "${PORT:-3000}:3000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - API_KEY=${API_KEY:?error}      # Required variable
      - LOG_LEVEL=${LOG_LEVEL:-info}
    volumes:
      - "${PWD}/data:/data"
    
    # Using variables in build args
    build:
      context: .
      args:
        NODE_VERSION: ${NODE_VERSION:-18}
```

### .env File for Compose
```bash
# .env file in same directory as docker-compose.yml
COMPOSE_PROJECT_NAME=myapp
COMPOSE_FILE=docker-compose.yml:docker-compose.override.yml
COMPOSE_PATH_SEPARATOR=:
COMPOSE_HTTP_TIMEOUT=60
COMPOSE_TLS_VERSION=TLSv1_2

# Application variables
TAG=latest
PORT=3000
DATABASE_URL=postgresql://user:pass@db:5432/mydb
```

## Profiles

### Profile Definition
```yaml
version: '3.8'

services:
  # Always started
  app:
    image: myapp:latest
    ports:
      - "3000:3000"
  
  # Development profile
  db-dev:
    image: postgres:13
    profiles:
      - development
      - test
    environment:
      POSTGRES_DB: myapp_dev
  
  # Production profile
  db-prod:
    image: postgres:13
    profiles:
      - production
    environment:
      POSTGRES_DB: myapp_prod
      POSTGRES_REPLICATION: async
  
  # Debug tools
  adminer:
    image: adminer
    profiles:
      - debug
      - development
    ports:
      - "8080:8080"
  
  # Testing services
  test-runner:
    image: myapp:test
    profiles:
      - test
    command: npm test
    depends_on:
      - db-dev
```

### Using Profiles
```bash
# Start with specific profiles
docker-compose --profile development up
docker-compose --profile production up

# Multiple profiles
docker-compose --profile development --profile debug up

# Environment variable
COMPOSE_PROFILES=development,debug docker-compose up

# All services (including those without profiles)
docker-compose --profile "*" up
```

## Extensions and Anchors

### YAML Anchors and Aliases
```yaml
version: '3.8'

# Define reusable configurations
x-app-common: &app-common
  restart: unless-stopped
  networks:
    - app-network
  environment: &common-env
    NODE_ENV: production
    LOG_LEVEL: info
    TZ: UTC

x-healthcheck-common: &healthcheck-common
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 40s

x-postgres-common: &postgres-common
  image: postgres:15-alpine
  restart: unless-stopped
  networks:
    - app-network
  volumes:
    - type: volume
      source: postgres_data
      target: /var/lib/postgresql/data

services:
  app:
    <<: *app-common                    # Merge app-common
    build: .
    ports:
      - "3000:3000"
    environment:
      <<: *common-env                  # Merge common environment
      PORT: 3000
      DATABASE_URL: postgresql://user:pass@db:5432/mydb
    healthcheck:
      <<: *healthcheck-common
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
    depends_on:
      - db
      - redis

  worker:
    <<: *app-common
    build:
      context: .
      dockerfile: Dockerfile.worker
    environment:
      <<: *common-env
      WORKER_CONCURRENCY: 4
    depends_on:
      - db
      - redis

  db:
    <<: *postgres-common
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password

volumes:
  postgres_data:

networks:
  app-network:
```

### Extension Fields
```yaml
version: '3.8'

# Extension fields (ignored by Compose but useful for organization)
x-logging: &default-logging
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"

x-deploy: &default-deploy
  restart_policy:
    condition: on-failure
    delay: 5s
    max_attempts: 3
  resources:
    limits:
      memory: 512M
    reservations:
      memory: 256M

services:
  app:
    image: myapp:latest
    logging: *default-logging
    deploy: *default-deploy
    
  worker:
    image: myapp:worker
    logging: *default-logging
    deploy:
      <<: *default-deploy
      replicas: 3                      # Override replicas
```

## Multi-Stage and Override Files

### Base Compose File (docker-compose.yml)
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: development
    volumes:
      - .:/app
      - /app/node_modules
    depends_on:
      - db

  db:
    image: postgres:13
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Development Override (docker-compose.override.yml)
```yaml
# This file is automatically loaded with docker-compose.yml
version: '3.8'

services:
  app:
    build:
      target: development              # Override build target
    environment:
      DEBUG: "true"
    volumes:
      - .:/app:cached                  # Override for macOS performance
    ports:
      - "9229:9229"                   # Debug port

  # Additional development services
  adminer:
    image: adminer
    ports:
      - "8080:8080"
    depends_on:
      - db

  redis:
    image: redis:alpine
```

### Production Override (docker-compose.prod.yml)
```yaml
version: '3.8'

services:
  app:
    build:
      target: production
    environment:
      NODE_ENV: production
    volumes: []                        # Remove development volumes
    restart: unless-stopped
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M

  db:
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    volumes:
      - postgres_data:/var/lib/postgresql/data:Z  # SELinux
    secrets:
      - db_password
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.prod.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - app
    restart: unless-stopped

secrets:
  db_password:
    external: true
```

### Testing Override (docker-compose.test.yml)
```yaml
version: '3.8'

services:
  app:
    build:
      target: test
    environment:
      NODE_ENV: test
      CI: "true"
    command: npm test
    depends_on:
      - db-test

  db-test:
    image: postgres:13-alpine
    environment:
      POSTGRES_DB: myapp_test
      POSTGRES_USER: test_user
      POSTGRES_PASSWORD: test_password
    tmpfs:
      - /var/lib/postgresql/data       # Use tmpfs for faster tests
```

### Using Multiple Files
```bash
# Development (default)
docker-compose up

# Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up

# Testing
docker-compose -f docker-compose.yml -f docker-compose.test.yml up

# Complex setup
docker-compose \
  -f docker-compose.yml \
  -f docker-compose.override.yml \
  -f docker-compose.local.yml \
  up
```

## Advanced Patterns

### Service Discovery and Load Balancing
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app

  app:
    build: .
    deploy:
      replicas: 3
    environment:
      NODE_ENV: production
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

# nginx.conf for load balancing
# upstream app_servers {
#     server app:3000;
# }
```

### Database Replication
```yaml
version: '3.8'

services:
  db-master:
    image: postgres:13
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_REPLICATION_USER: replicator
      POSTGRES_REPLICATION_PASSWORD: replicator_password
    volumes:
      - db_master_data:/var/lib/postgresql/data
      - ./postgresql.conf:/etc/postgresql/postgresql.conf
    command: postgres -c config_file=/etc/postgresql/postgresql.conf

  db-replica:
    image: postgres:13
    environment:
      PGUSER: user
      POSTGRES_PASSWORD: password
      POSTGRES_MASTER_SERVICE: db-master
      POSTGRES_MASTER_PORT: 5432
      POSTGRES_REPLICATION_USER: replicator
      POSTGRES_REPLICATION_PASSWORD: replicator_password
    volumes:
      - db_replica_data:/var/lib/postgresql/data
    depends_on:
      - db-master

volumes:
  db_master_data:
  db_replica_data:
```

### Microservices with API Gateway
```yaml
version: '3.8'

services:
  api-gateway:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./gateway.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - user-service
      - product-service
      - order-service

  user-service:
    build: ./services/user
    environment:
      SERVICE_NAME: user-service
      DATABASE_URL: postgresql://user:pass@db:5432/users
    networks:
      - app-network
      - user-db-network

  product-service:
    build: ./services/product
    environment:
      SERVICE_NAME: product-service
      DATABASE_URL: postgresql://user:pass@db:5432/products
    networks:
      - app-network
      - product-db-network

  order-service:
    build: ./services/order
    environment:
      SERVICE_NAME: order-service
      DATABASE_URL: postgresql://user:pass@db:5432/orders
    networks:
      - app-network
      - order-db-network

  db:
    image: postgres:13
    environment:
      POSTGRES_MULTIPLE_DATABASES: users,products,orders
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./create-multiple-databases.sh:/docker-entrypoint-initdb.d/create-multiple-databases.sh
    networks:
      - user-db-network
      - product-db-network
      - order-db-network

volumes:
  postgres_data:

networks:
  app-network:
  user-db-network:
  product-db-network:
  order-db-network:
```

### Development Tools Integration
```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      target: development
    volumes:
      - .:/app:cached
      - /app/node_modules
    environment:
      NODE_ENV: development
      DEBUG: "*"
    ports:
      - "3000:3000"
      - "9229:9229"                   # Node.js debug port
    command: npm run dev

  # Hot reloading proxy
  webpack-dev-server:
    build:
      context: .
      dockerfile: Dockerfile.webpack
    volumes:
      - ./src:/app/src:cached
      - ./public:/app/public:cached
    ports:
      - "8080:8080"
    environment:
      WEBPACK_DEV_SERVER: "true"

  # Database GUI
  pgadmin:
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: admin
    ports:
      - "5050:80"
    depends_on:
      - db

  # Redis GUI
  redis-commander:
    image: rediscommander/redis-commander
    environment:
      REDIS_HOSTS: local:redis:6379
    ports:
      - "8081:8081"
    depends_on:
      - redis

  # Log aggregation
  elasticsearch:
    image: elasticsearch:7.9.3
    environment:
      discovery.type: single-node
      ES_JAVA_OPTS: "-Xmx256m -Xms256m"
    volumes:
      - es_data:/usr/share/elasticsearch/data

  logstash:
    image: logstash:7.9.3
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf:ro
    depends_on:
      - elasticsearch

  kibana:
    image: kibana:7.9.3
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

volumes:
  es_data:
```

## Troubleshooting

### Debugging Commands
```bash
# Configuration validation
docker-compose config                   # Validate syntax
docker-compose config --quiet          # Just validate, no output
docker-compose config --services        # List services
docker-compose config --volumes         # List volumes
docker-compose config --resolve-image-digests  # Show image digests

# Service inspection
docker-compose ps                       # Running containers
docker-compose ps -a                    # All containers
docker-compose ps --services           # Service names only
docker-compose top                      # Running processes
docker-compose port web 80              # Port mapping

# Logs and debugging
docker-compose logs                     # All service logs
docker-compose logs -f web              # Follow specific service
docker-compose logs --tail=50 web       # Last 50 lines
docker-compose logs -t web              # With timestamps
docker-compose logs --since="2023-01-01T00:00:00Z" web

# Container access
docker-compose exec web bash            # Execute command
docker-compose run --rm web bash        # One-off container
docker-compose run --rm --no-deps web npm test  # Without dependencies

# Network debugging
docker-compose exec web ping db          # Test connectivity
docker-compose exec web nslookup db      # DNS resolution
docker-compose exec web netstat -an     # Network connections
```

### Common Issues and Solutions

#### Service Won't Start
```bash
# Check logs
docker-compose logs service_name

# Check configuration
docker-compose config

# Rebuild without cache
docker-compose build --no-cache service_name

# Start with verbose output
docker-compose up --verbose
```

#### Port Conflicts
```bash
# Check what's using the port
lsof -i :8080
netstat -tulpn | grep 8080

# Use different port mapping
ports:
  - "8081:8080"  # Use different host port

# Or let Docker assign random port
ports:
  - "8080"
```

#### Volume Permission Issues
```bash
# Check container user
docker-compose exec service_name id

# Fix ownership (Linux/macOS)
sudo chown -R $USER:$USER ./data

# Use proper user in compose file
services:
  app:
    user: "${UID:-1000}:${GID:-1000}"

# Or set in Dockerfile
FROM node:18-alpine
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001
USER nextjs
```

#### Network Issues
```bash
# List networks
docker network ls

# Inspect network
docker network inspect $(docker-compose ps -q | head -1 | xargs docker inspect --format='{{range .NetworkSettings.Networks}}{{.NetworkID}}{{end}}')

# Recreate networks
docker-compose down
docker-compose up
```

#### Environment Variable Issues
```yaml
# Debug environment variables
services:
  app:
    command: ["sh", "-c", "env | sort && sleep 3600"]
    
# Or check in running container
docker-compose exec app env
```

#### Performance Issues
```yaml
# Use cached/delegated volumes on macOS
volumes:
  - ./src:/app/src:cached
  - ./build:/app/build:delegated

# Reduce context size with .dockerignore
# node_modules
# .git
# *.log

# Use multi-stage builds
FROM node:18-alpine AS dependencies
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine AS runtime
WORKDIR /app
COPY --from=dependencies /app/node_modules ./node_modules
COPY . .
CMD ["npm", "start"]
```

### Advanced Debugging

#### Health Check Debugging
```yaml
services:
  app:
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:3000/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    
# Check health status
docker-compose ps
docker inspect $(docker-compose ps -q app) | jq '.[0].State.Health'
```

#### Dependency Issues
```yaml
# Wait for service to be ready
services:
  app:
    depends_on:
      db:
        condition: service_healthy
  
  db:
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $POSTGRES_USER -d $POSTGRES_DB"]
      interval: 10s
      timeout: 5s
      retries: 5
```

#### Resource Monitoring
```bash
# Monitor resource usage
docker stats $(docker-compose ps -q)

# Check container processes
docker-compose exec app ps aux

# Check disk usage
docker system df
docker-compose images
```

## Performance Optimization

### Build Optimization
```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.optimized
      cache_from:
        - myregistry/myapp:cache
        - myregistry/myapp:latest
      target: production
```

```dockerfile
# Dockerfile.optimized
FROM node:18-alpine AS base
WORKDIR /app

# Dependencies stage
FROM base AS dependencies
COPY package*.json ./
RUN npm ci --only=production --frozen-lockfile

# Build stage  
FROM base AS build
COPY package*.json ./
RUN npm ci --frozen-lockfile
COPY . .
RUN npm run build

# Runtime stage
FROM base AS production
COPY --from=dependencies /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY package*.json ./
USER node
EXPOSE 3000
CMD ["npm", "start"]
```

### Resource Limits
```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
```

### Scaling Strategies
```yaml
services:
  app:
    image: myapp:latest
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
        order: start-first
      rollback_config:
        parallelism: 1
        delay: 5s
        failure_action: pause
      placement:
        constraints:
          - node.role == worker
```

## Security Best Practices

### Secrets Management
```yaml
version: '3.8'

services:
  app:
    image: myapp:latest
    environment:
      DATABASE_PASSWORD_FILE: /run/secrets/db_password
      API_KEY_FILE: /run/secrets/api_key
    secrets:
      - db_password
      - api_key
    user: "1001:1001"

secrets:
  db_password:
    file: ./secrets/db_password.txt
  api_key:
    external: true
    name: app_api_key
```

### Network Security
```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    networks:
      - frontend
    ports:
      - "443:443"

  app:
    build: .
    networks:
      - frontend
      - backend
    # No external ports exposed

  db:
    image: postgres:13
    networks:
      - backend
    # Only accessible from backend network

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No external access
```

### Security Configurations
```yaml
services:
  app:
    image: myapp:latest
    user: "1001:1001"                 # Non-root user
    read_only: true                   # Read-only filesystem
    tmpfs:
      - /tmp:noexec,nosuid,size=100m
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
```

## Production Deployment

### Production Compose File
```yaml
version: '3.8'

x-common: &common
  restart: unless-stopped
  logging:
    driver: "json-file"
    options:
      max-size: "10m"
      max-file: "3"

services:
  nginx:
    <<: *common
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.prod.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
      - static_files:/var/www/static:ro
    depends_on:
      - app

  app:
    <<: *common
    image: myregistry.com/myapp:${TAG}
    environment:
      NODE_ENV: production
      DATABASE_URL_FILE: /run/secrets/database_url
    secrets:
      - database_url
    volumes:
      - static_files:/app/public
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback

  db:
    <<: *common
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: app_user
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./backup:/backup:ro
    secrets:
      - db_password

  redis:
    <<: *common
    image: redis:7-alpine
    command: redis-server --requirepass-file /run/secrets/redis_password
    volumes:
      - redis_data:/data
    secrets:
      - redis_password

  backup:
    image: postgres:15-alpine
    environment:
      PGPASSWORD_FILE: /run/secrets/db_password
    volumes:
      - ./backups:/backups
      - backup_scripts:/scripts:ro
    secrets:
      - db_password
    profiles:
      - backup
    command: >
      sh -c "
        while true; do
          pg_dump -h db -U app_user -d myapp > /backups/backup_$(date +%Y%m%d_%H%M%S).sql
          find /backups -name '*.sql' -mtime +7 -delete
          sleep 86400
        done
      "

volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local
  static_files:
    driver: local
  backup_scripts:
    driver: local

secrets:
  database_url:
    external: true
  db_password:
    external: true
  redis_password:
    external: true
```

### CI/CD Integration
```yaml
# .github/workflows/deploy.yml example
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Build and push Docker image
        run: |
          docker build -t myregistry.com/myapp:${{ github.sha }} .
          docker push myregistry.com/myapp:${{ github.sha }}
      
      - name: Deploy to production
        run: |
          export TAG=${{ github.sha }}
          docker-compose -f docker-compose.prod.yml pull
          docker-compose -f docker-compose.prod.yml up -d
```
