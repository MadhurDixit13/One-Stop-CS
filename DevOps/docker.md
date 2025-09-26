# Docker 🐳

[![Docker](https://img.shields.io/badge/Docker-Containerization-blue)](https://docker.com/)
[![Go](https://img.shields.io/badge/Go-1.19+-blue)](https://golang.org/)
[![License](https://img.shields.io/badge/License-Apache%202.0-green.svg)](https://opensource.org/licenses/Apache-2.0)

> **Containerization Platform** - Docker is an open platform for developing, shipping, and running applications using containerization technology that packages applications and their dependencies into lightweight, portable containers.

## 🎯 Overview

Docker is a powerful containerization platform that:
- **Packages Applications** - Bundle apps with all dependencies
- **Ensures Consistency** - Same environment across development, testing, and production
- **Enables Portability** - Run anywhere Docker is supported
- **Simplifies Deployment** - Easy application deployment and scaling
- **Improves Efficiency** - Resource optimization and faster startup times
- **Facilitates DevOps** - Streamlines development and operations workflows

## 🚀 Key Features

### Core Capabilities
- **Containerization** - Isolated application environments
- **Image Management** - Build, store, and distribute container images
- **Orchestration** - Manage multiple containers
- **Networking** - Connect containers and services
- **Volume Management** - Persistent data storage
- **Multi-Platform Support** - Linux, Windows, macOS

### Container Benefits
- **Lightweight** - Shared OS kernel, minimal overhead
- **Fast Startup** - Quick container initialization
- **Scalable** - Easy horizontal scaling
- **Secure** - Process isolation and resource limits
- **Version Control** - Image versioning and rollbacks
- **Microservices Ready** - Perfect for microservices architecture

## 🏗️ Architecture

### Docker Components
```
Docker Architecture
├── 🐳 Docker Engine
│   ├── Docker Daemon
│   ├── REST API
│   └── CLI Interface
├── 📦 Docker Images
│   ├── Base Images
│   ├── Application Images
│   └── Multi-stage Builds
├── 🏃 Docker Containers
│   ├── Running Instances
│   ├── Process Isolation
│   └── Resource Management
├── 🌐 Docker Network
│   ├── Bridge Networks
│   ├── Overlay Networks
│   └── Host Networks
└── 💾 Docker Volumes
    ├── Named Volumes
    ├── Bind Mounts
    └── tmpfs Mounts
```

## 🛠️ Installation & Setup

### Linux Installation
```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# CentOS/RHEL
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl enable docker
```

### macOS Installation
```bash
# Using Homebrew
brew install --cask docker

# Or download Docker Desktop from docker.com
```

### Windows Installation
```powershell
# Using Chocolatey
choco install docker-desktop

# Or download Docker Desktop from docker.com
```

### Post-Installation Setup
```bash
# Add user to docker group (Linux)
sudo usermod -aG docker $USER
newgrp docker

# Verify installation
docker --version
docker run hello-world

# Check Docker info
docker info
```

## 📦 Basic Docker Commands

### Image Management
```bash
# Pull an image
docker pull nginx:latest
docker pull ubuntu:20.04

# List images
docker images
docker image ls

# Remove images
docker rmi nginx:latest
docker image rm ubuntu:20.04

# Build image from Dockerfile
docker build -t myapp:latest .

# Tag an image
docker tag myapp:latest myapp:v1.0

# Push to registry
docker push myapp:latest
```

### Container Management
```bash
# Run a container
docker run -d --name web-server nginx:latest
docker run -it ubuntu:20.04 /bin/bash

# List containers
docker ps
docker ps -a

# Stop/start containers
docker stop web-server
docker start web-server
docker restart web-server

# Remove containers
docker rm web-server
docker rm -f web-server

# Execute commands in running container
docker exec -it web-server /bin/bash

# View container logs
docker logs web-server
docker logs -f web-server
```

### Container Configuration
```bash
# Run with port mapping
docker run -d -p 8080:80 --name web nginx

# Run with volume mounting
docker run -d -v /host/path:/container/path --name app myapp

# Run with environment variables
docker run -d -e MYSQL_ROOT_PASSWORD=secret --name db mysql

# Run with resource limits
docker run -d --memory=512m --cpus=1 --name app myapp

# Run with network
docker run -d --network mynetwork --name app myapp
```

## 📝 Dockerfile

### Basic Dockerfile
```dockerfile
# Use official Python runtime as base image
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Copy requirements first for better caching
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose port
EXPOSE 8000

# Set environment variables
ENV PYTHONPATH=/app
ENV FLASK_ENV=production

# Create non-root user
RUN useradd --create-home --shell /bin/bash app
USER app

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

# Run application
CMD ["python", "app.py"]
```

### Multi-stage Build
```dockerfile
# Build stage
FROM node:16-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Production stage
FROM node:16-alpine AS production
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
USER node
CMD ["npm", "start"]
```

### Advanced Dockerfile
```dockerfile
# Multi-architecture build
FROM --platform=$BUILDPLATFORM golang:1.19-alpine AS builder
ARG TARGETPLATFORM
ARG BUILDPLATFORM
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o app .

# Final stage
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/app .
EXPOSE 8080
CMD ["./app"]
```

## 🐳 Docker Compose

### Basic Compose File
```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
    depends_on:
      - db
    volumes:
      - ./logs:/app/logs

  db:
    image: postgres:13
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:6-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

### Advanced Compose File
```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.prod
    ports:
      - "80:8000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
    depends_on:
      - db
      - redis
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
        reservations:
          memory: 256M
          cpus: '0.25'
    networks:
      - frontend
      - backend

  db:
    image: postgres:13
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped
    networks:
      - backend

  redis:
    image: redis:6-alpine
    restart: unless-stopped
    networks:
      - backend

  nginx:
    image: nginx:alpine
    ports:
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - web
    restart: unless-stopped
    networks:
      - frontend

volumes:
  postgres_data:

networks:
  frontend:
  backend:
```

### Compose Commands
```bash
# Start services
docker-compose up -d

# Build and start
docker-compose up --build

# Stop services
docker-compose down

# View logs
docker-compose logs -f web

# Scale services
docker-compose up --scale web=3

# Use specific compose file
docker-compose -f docker-compose.prod.yml up -d
```

## 🌐 Docker Networking

### Network Types
```bash
# List networks
docker network ls

# Create custom network
docker network create mynetwork

# Run container on specific network
docker run -d --network mynetwork --name app myapp

# Connect container to network
docker network connect mynetwork existing-container

# Disconnect from network
docker network disconnect mynetwork existing-container

# Inspect network
docker network inspect mynetwork
```

### Network Configuration
```yaml
# docker-compose.yml with custom networks
version: '3.8'

services:
  web:
    image: nginx
    networks:
      - frontend
      - backend

  api:
    image: myapi
    networks:
      - backend

  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true
```

## 💾 Docker Volumes

### Volume Management
```bash
# List volumes
docker volume ls

# Create volume
docker volume create myvolume

# Inspect volume
docker volume inspect myvolume

# Remove volume
docker volume rm myvolume

# Run with named volume
docker run -d -v myvolume:/data --name app myapp

# Run with bind mount
docker run -d -v /host/path:/container/path --name app myapp
```

### Volume Configuration
```yaml
# docker-compose.yml with volumes
version: '3.8'

services:
  web:
    image: nginx
    volumes:
      - web_data:/var/www/html
      - ./config:/etc/nginx/conf.d
      - /host/logs:/var/log/nginx

  db:
    image: postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql

volumes:
  web_data:
  postgres_data:
    driver: local
```

## 🔧 Best Practices

### Dockerfile Best Practices
1. **Use Multi-stage Builds** - Reduce final image size
2. **Leverage Layer Caching** - Order commands by frequency of change
3. **Use .dockerignore** - Exclude unnecessary files
4. **Run as Non-root** - Improve security
5. **Use Specific Tags** - Avoid latest tag in production
6. **Minimize Layers** - Combine RUN commands when possible

### Security Best Practices
1. **Scan Images** - Use vulnerability scanners
2. **Use Official Images** - Prefer official base images
3. **Keep Images Updated** - Regular security updates
4. **Limit Resources** - Set memory and CPU limits
5. **Use Secrets** - Don't hardcode sensitive data
6. **Network Isolation** - Use custom networks

### Performance Best Practices
1. **Optimize Image Size** - Use alpine or distroless images
2. **Use Health Checks** - Monitor container health
3. **Resource Limits** - Set appropriate limits
4. **Volume Optimization** - Use appropriate volume types
5. **Network Optimization** - Use host networking when appropriate

## 🚀 Advanced Features

### Docker Swarm
```bash
# Initialize swarm
docker swarm init

# Join swarm
docker swarm join --token SWMTKN-1-xxx manager-ip:2377

# Deploy stack
docker stack deploy -c docker-compose.yml myapp

# List services
docker service ls

# Scale service
docker service scale myapp_web=3

# Update service
docker service update --image myapp:v2.0 myapp_web
```

### Docker Buildx
```bash
# Create builder instance
docker buildx create --name mybuilder --use

# Build multi-platform image
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest .

# Build and push
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest --push .
```

### Docker Secrets
```bash
# Create secret
echo "mysecret" | docker secret create db_password -

# Use secret in service
docker service create --secret db_password --name web nginx
```

## 🔍 Monitoring and Debugging

### Container Monitoring
```bash
# View container stats
docker stats

# View specific container stats
docker stats web-server

# View container processes
docker top web-server

# Inspect container
docker inspect web-server

# View container logs
docker logs web-server
docker logs --tail 100 -f web-server
```

### Debugging Commands
```bash
# Execute shell in container
docker exec -it web-server /bin/bash

# Copy files to/from container
docker cp file.txt web-server:/app/
docker cp web-server:/app/file.txt ./

# View container filesystem
docker diff web-server

# Pause/unpause container
docker pause web-server
docker unpause web-server
```

## 🐛 Troubleshooting

### Common Issues
1. **Container Won't Start** - Check logs and configuration
2. **Port Conflicts** - Verify port mappings
3. **Volume Mount Issues** - Check permissions and paths
4. **Network Connectivity** - Verify network configuration
5. **Resource Limits** - Check memory and CPU limits

### Debugging Tools
```bash
# Check Docker daemon status
sudo systemctl status docker

# View Docker daemon logs
sudo journalctl -u docker.service

# Check disk usage
docker system df

# Clean up unused resources
docker system prune -a

# Check for issues
docker system events
```

## 📚 Learning Resources

### Official Documentation
- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Reference](https://docs.docker.com/compose/)
- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)

### Community Resources
- [Docker Hub](https://hub.docker.com/)
- [GitHub Repository](https://github.com/docker/docker)
- [Docker Community](https://www.docker.com/community/)

## 🔄 Updates and Roadmap

### Recent Updates
- **v24.0.0** - Enhanced security features and performance improvements
- **v23.0.0** - New buildx features and multi-platform support
- **v22.0.0** - Improved compose integration and networking

### Upcoming Features
- **v25.0.0** - Advanced security scanning
- **v26.0.0** - Enhanced orchestration capabilities
- **v27.0.0** - Improved developer experience

## 🤝 Contributing

### How to Contribute
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests and documentation
5. Submit a pull request

### Contribution Guidelines
- Follow Go coding standards
- Write comprehensive tests
- Update documentation
- Ensure backward compatibility

## 📄 License

This project is licensed under the Apache License 2.0 - see the [LICENSE](https://github.com/docker/docker/blob/main/LICENSE) file for details.

## 🙏 Acknowledgments

- **Docker Inc.** for creating and maintaining Docker
- **Open Source Community** for contributions and feedback
- **Users** for providing real-world use cases and improvements
- **Contributors** for advancing containerization technology

---

<div align="center">

**🌟 Star this repository if you find it helpful!**

[⬆ Back to Top](#docker-)

</div>
