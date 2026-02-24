# 🚀 MEAN Stack CRUD Application — DevOps CI/CD Deployment
### Discover Dollar | DevOps Engineer Internship Assignment

A fully containerized MEAN stack (MongoDB, Express, Angular 15, Node.js) application deployed on AWS EC2 with Nginx reverse proxy and automated CI/CD pipeline using Jenkins.

---

# 🏗️ Architecture Overview

```
Internet
    ↓
AWS EC2 - App Server <EC2-APP>
    ↓
Nginx (Port 80)
    ↓
  /        →  Frontend Container (Angular - Port 80)
  /api/    →  Backend Container (Express - Port 8080)
                    ↓
              MongoDB Container (27017)
                    ↓
               Docker Volume (mongo-data)

Jenkins EC2 (Separate Server)
    ↓
CI/CD Pipeline
    ↓
Docker Hub → App EC2
```

---

# 🔁 CI/CD Workflow Explained

This project implements a complete Continuous Integration and Continuous Deployment pipeline.

## Continuous Integration (CI)
- Code pushed to GitHub (main branch)
- Jenkins clones repository
- Backend Docker image built
- Frontend Docker image built (multi-stage)
- Images pushed to Docker Hub

## Continuous Deployment (CD)
- Jenkins connects securely to App EC2 via SSH
- Pulls latest Docker images
- Stops running containers
- Recreates containers using Docker Compose
- Application updated automatically

---

# 🛠️ Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | Angular 15 |
| Backend | Node.js + Express |
| Database | MongoDB 6 |
| Reverse Proxy | Nginx |
| Containerization | Docker + Docker Compose |
| CI/CD | Jenkins |
| Cloud | AWS EC2 (Ubuntu 22.04) |
| Registry | Docker Hub |

---

# 📁 Repository Structure

```
crud-dd-task-mean-app/
├── backend/
│   ├── app/
│   │   ├── config/
│   │   ├── controllers/
│   │   ├── models/
│   │   └── routes/
│   ├── Dockerfile
│   ├── server.js
│   └── package.json
├── frontend/
│   ├── src/
│   ├── Dockerfile
│   └── package.json
├── docker-compose.yml
├── nginx.conf
├── Jenkinsfile
└── README.md
```

---

# 🐳 Docker Configuration

## Backend Dockerfile

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 8080
CMD ["node", "server.js"]
```

---

## Frontend Dockerfile (Multi-Stage Build)

```dockerfile
# Stage 1 – Build Angular App
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2 – Production Image
FROM nginx:alpine
COPY --from=build /app/dist/angular-15-crud /usr/share/nginx/html
EXPOSE 80
```

---

# 📦 Why Multi-Stage Build?

The Angular frontend uses a multi-stage Docker build:

1. First stage builds production-ready Angular files using Node.js.
2. Second stage copies only compiled static files into lightweight Nginx image.

### Benefits:
- Smaller image size
- Better security (Node not present in production)
- Faster deployment
- Reduced attack surface

---

# 🐙 Docker Compose Configuration

```yaml


services:
  mongo:
    image: mongo:6
    container_name: dd-mongo
    restart: always
    volumes:
      - mongo-data:/data/db
    networks:
      - mean-network

  backend:
    image: jaikiran333/dd-backend:latest
    container_name: dd-backend
    restart: always
    expose:
      - "8080"
    environment:
      - MONGO_URI=mongodb://mongo:27017/dddb
    depends_on:
      - mongo
    networks:
      - mean-network

  frontend:
    image: jaikiran333/dd-frontend:latest
    container_name: dd-frontend
    restart: always
    expose:
      - "80"
    depends_on:
      - backend
    networks:
      - mean-network

  nginx:
    image: nginx:alpine
    container_name: dd-nginx
    restart: always
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - frontend
      - backend
    networks:
      - mean-network

volumes:
  mongo-data:

networks:
  mean-network:
    driver: bridge
```

> Only Nginx exposes port 80 to the outside world. Backend and frontend are internally accessible via Docker network.

---

# 🔀 Nginx Reverse Proxy

```nginx
events {}

http {
  server {
    listen 80;

    location / {
      proxy_pass http://frontend:80;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
    }

    location /api/ {
      proxy_pass http://backend:8080;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
    }
  }
}
```

---

# ⚙️ Jenkins Pipeline (Declarative)

Pipeline Stages:

1. Clone Repository
2. Build Backend Image
3. Build Frontend Image
4. Push Images to Docker Hub
5. Deploy to App Server via SSH

---

# 🔧 Jenkins Plugins Used

- Pipeline
- Git
- Credentials Binding
- SSH Agent

---

# 🔐 Jenkins Credentials

| ID | Type | Purpose |
|----|------|----------|
| dockerhub-creds | Username/Password | Docker Hub login |
| app-server-ssh-key | SSH Private Key | SSH access to App EC2 |
| github-creds | Username/Password | GitHub access |

---

# 🚀 Deployment Steps (Manual)

```bash
docker-compose up -d
```

Access Application:

```
http://<EC2-PUBLIC-IP>
```

---

# 🌐 Infrastructure Details

## EC2 Instances

| Server | Purpose | Instance Type | Port |
|--------|---------|--------------|------|
| App Server | Runs MEAN stack | t2.micro | 80 |
| Jenkins Server | CI/CD | t2.medium | 8080 |

---

# 🔮 Future Improvements

- Add HTTPS using Let's Encrypt
- Use AWS Load Balancer
- Implement Auto Scaling
- Move MongoDB to MongoDB Atlas
- Add Jenkins webhook trigger
- Add monitoring (Prometheus + Grafana)

---

# 🎯 Project Outcome

This project demonstrates:

- Complete Docker-based containerization
- Multi-stage production builds
- Secure credential management
- Automated CI/CD deployment
- Reverse proxy configuration
- Cloud deployment using AWS
- Zero manual deployment process

The implementation follows industry-standard DevOps practices.

---

# 📸 Screenshots

(Add your screenshots here)

- Jenkins successful build
- Docker Hub images
- Running containers
- Application UI

---

# 📌 Author

**Jaikiran Jayaraman**  
DevOps Engineer Internship Assignment
