# 🐳 Docker Cheatsheet

## 🚀 Complete Docker Guide for Beginners to Advanced Users 📦 Learn Docker Installation, Images, Containers, Networking, Security, Monitoring & Best Practices

---

## 📚 Table of Contents
1. Introduction to Docker
2. Installing Docker
3. Basic Docker Operations
4. Building Docker Images
5. Docker Compose
6. Docker Volumes and Storage
7. Docker Networking
8. Docker Security
9. Advanced Docker Features
10. Monitoring and Logging
11. Docker Best Practices
12. Troubleshooting Docker
13. References

---

## 🚀 1. Introduction to Docker

### 🔹 What is Docker?
Docker is an open-source platform that automates the deployment, scaling, and management of applications using containerization technology.

Containers are:
* Lightweight
* Portable
* Consistent environments

They include everything needed to run an application:
* Code
* Runtime
* System tools
* Libraries
* Dependencies

### 🔑 Key Concepts

| Component | Description |
| :--- | :--- |
| **Docker Engine** | Core component responsible for running containers |
| **Image** | Lightweight executable package containing app + dependencies |
| **Container** | Running instance of a Docker image |
| **Dockerfile** | Script used to build Docker images |
| **Registry** | Storage system for Docker images (Docker Hub) |
| **Docker Compose** | Tool to manage multi-container applications |

---

## 🛠 2. Installing Docker

### 🐧 Install Docker on Linux

#### Install Docker Engine
```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

#### Start Docker Service
```bash
sudo systemctl start docker
sudo systemctl enable docker
```

### 🍎 Install Docker on macOS
Install Docker Desktop from Docker official website.

### 🪟 Install Docker on Windows
Install Docker Desktop from Docker official website.

---

## ⚙️ 3. Basic Docker Operations

### 📦 Working with Docker Images

#### Search for an Image
```bash
docker search nginx
```

#### Pull an Image
```bash
docker pull nginx
```

#### List Images
```bash
docker images
```

#### Remove an Image
```bash
docker rmi nginx
```

### 📦 Working with Docker Containers

#### Run a Container
```bash
docker run -d -p 80:80 --name mynginx nginx
```

#### List Running Containers
```bash
docker ps
```

#### List All Containers
```bash
docker ps -a
```

#### Stop a Container
```bash
docker stop mynginx
```

#### Remove a Container
```bash
docker rm mynginx
```

### 🌐 Docker Networks

#### List Networks
```bash
docker network ls
```

#### Create Network
```bash
docker network create mynetwork
```

#### Connect Container to Network
```bash
docker network connect mynetwork mynginx
```

#### Disconnect Container
```bash
docker network disconnect mynetwork mynginx
```

---

## 🏗 4. Building Docker Images

### 📄 Dockerfile Basics

#### Sample Dockerfile
```dockerfile
# Use Node.js image
FROM node:14

# Set working directory
WORKDIR /app

# Copy files
COPY . /app

# Install dependencies
RUN npm install

# Expose port
EXPOSE 8080

# Environment variable
ENV NODE_ENV production

# Start application
CMD ["node", "app.js"]
```

#### Build Docker Image
```bash
docker build -t mynodeapp .
```

#### Tag an Image
```bash
docker tag mynodeapp myrepo/mynodeapp:v1.0
```

#### Push Image to Docker Hub
```bash
docker push myrepo/mynodeapp:v1.0
```

---

## 🧩 5. Docker Compose

### 🔹 What is Docker Compose?
Docker Compose is used to define and manage multi-container applications using a YAML configuration file.

### 📄 Sample docker-compose.yml
```yaml
version: '3'

services:
  web:
    image: nginx
    ports:
      - "8080:80"

  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: example
```

### 🚀 Docker Compose Commands

#### Start Services
```bash
docker-compose up
```

#### Stop Services
```bash
docker-compose down
```

#### Scale Containers
```bash
docker-compose up --scale web=3
```

### 📦 Volumes in Docker Compose
```yaml
services:
  web:
    image: nginx
    volumes:
      - ./webdata:/usr/share/nginx/html
```

---

## 💾 6. Docker Volumes and Storage

### 🔹 Understanding Volumes
Volumes are used to persist container data permanently.

### 📦 Volume Commands

#### Create Volume
```bash
docker volume create myvolume
```

#### List Volumes
```bash
docker volume ls
```

#### Inspect Volume
```bash
docker volume inspect myvolume
```

#### Remove Volume
```bash
docker volume rm myvolume
```

### 🔗 Mount Volume to Container
```bash
docker run -d -p 80:80 --name mynginx -v myvolume:/usr/share/nginx/html nginx
```

### 📁 Bind Mount Example
```bash
docker run -d -p 80:80 --name mynginx -v /path/to/local/dir:/usr/share/nginx/html nginx
```

---

## 🌐 7. Docker Networking

### 🔹 Networking Modes

| Mode | Description |
| :--- | :--- |
| **Bridge** | Default network driver |
| **Host** | Shares host network |
| **Overlay** | Multi-host networking |

#### Create Custom Network
```bash
docker network create mynetwork
```

#### Run Container in Network
```bash
docker run -d --name mynginx --network=mynetwork nginx
```

#### Inspect Network
```bash
docker network inspect mynetwork
```

### 🔹 Docker DNS
Docker containers can resolve each other's names using embedded DNS.

---

## 🔒 8. Docker Security

### 🔹 Run Containers as Non-root User
```dockerfile
FROM nginx
USER www-data
```

### 🔹 Docker Content Trust
Enable Docker Content Trust:
```bash
export DOCKER_CONTENT_TRUST=1
```

### 🔹 Docker Secrets

#### Create Secret
```bash
echo "mysecretpassword" | docker secret create my_secret -
```

#### Use Secret
```bash
docker service create --name myservice --secret my_secret nginx
```

### 🔹 Limit Resources

#### Limit Memory
```bash
docker run -d --name mynginx --memory="256m" nginx
```

#### Limit CPU
```bash
docker run -d --name mynginx --cpus="1.0" nginx
```

---

## 🚀 9. Advanced Docker Features

### 🔹 Docker Swarm

#### Initialize Swarm
```bash
docker swarm init
```

#### Join Swarm
```bash
docker swarm join --token SWMTKN-1-xxxx
```

#### Deploy Stack
```bash
docker stack deploy -c docker-compose.yml mystack
```

### 🔹 Multi-Stage Builds
```dockerfile
# First Stage
FROM golang:1.16 as builder

WORKDIR /app

COPY . .

RUN go build -o myapp

# Second Stage
FROM alpine:latest

WORKDIR /app

COPY --from=builder /app/myapp .

CMD ["./myapp"]
```

### 🔹 Docker Plugins

#### List Plugins
```bash
docker plugin ls
```

#### Install Plugin
```bash
docker plugin install vieux/sshfs
```

### 🔹 Docker Daemon Configuration

#### daemon.json Example
```json
{
  "log-driver": "json-file",
  "log-level": "warn",
  "storage-driver": "overlay2"
}
```

#### Reload Docker Daemon
```bash
sudo systemctl reload docker
```

---

## 📊 10. Monitoring and Logging

### 🔹 Docker Logs

#### View Logs
```bash
docker logs mynginx
```

#### Follow Logs
```bash
docker logs -f mynginx
```

### 🔹 Monitor Containers

#### Resource Usage
```bash
docker stats mynginx
```

#### Docker Events
```bash
docker events
```

### 🔹 Monitoring with cAdvisor
```bash
docker run -d \
--name=cadvisor \
--volume=/:/rootfs:ro \
--volume=/var/run:/var/run:ro \
--volume=/sys:/sys:ro \
--volume=/var/lib/docker/:/var/lib/docker:ro \
--volume=/dev/disk/:/dev/disk:ro \
--publish=8080:8080 \
google/cadvisor:latest
```

---

## ✅ 11. Docker Best Practices

### 🔹 Dockerfile Best Practices
* Use multi-stage builds
* Minimize image size
* Use .dockerignore
* Leverage build cache

### 🔹 Container Best Practices
* Keep containers stateless
* Treat containers as immutable
