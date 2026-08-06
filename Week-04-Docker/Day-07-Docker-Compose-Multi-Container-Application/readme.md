## 🎯 Objective

Build and run a real-world multi-container backend application using **Docker Compose**, **Node.js**, **Express**, and **MongoDB** while learning how Docker orchestrates multiple services together.

---

# 📚 Topics Covered

- Building a Node.js backend
- Creating a Dockerfile
- Using `.dockerignore`
- Environment Variables (`.env`)
- Docker Image Creation
- Docker Compose
- Multi-Container Applications
- MongoDB Integration
- Named Volumes
- Custom Networks
- Health Checks
- `depends_on`
- Docker DNS
- Container Logs
- Testing REST APIs

---

# 📂 Project Structure

```text
backend/
├── Dockerfile
├── .dockerignore
├── .env
├── package.json
├── package-lock.json
├── server.js
└── node_modules/
```

---

# 1️⃣ Initialize Node.js Project

Initialize the project.

```bash
npm init -y
```

Install required packages.

```bash
npm install express mongoose dotenv
```

Install development dependency.

```bash
npm install --save-dev nodemon
```

---

# 2️⃣ Configure Environment Variables

Create `.env`

```env
PORT=5000

MONGO_URI=mongodb://mongodb:27017/opsstack
```

### Why use Environment Variables?

- Avoid hardcoding values
- Improve security
- Easy configuration for different environments
- Better production practices

---

# 3️⃣ Create Dockerfile

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 5000

CMD ["npm","start"]
```

---

## Dockerfile Explanation

### Base Image

```dockerfile
FROM node:22-alpine
```

Uses a lightweight Node.js image.

---

### Working Directory

```dockerfile
WORKDIR /app
```

Sets the working directory inside the container.

---

### Copy Dependency Files

```dockerfile
COPY package*.json ./
```

Copies dependency files first.

---

### Install Dependencies

```dockerfile
RUN npm install
```

Installs required Node.js packages.

---

### Copy Application Source

```dockerfile
COPY . .
```

Copies remaining application files.

---

### Expose Port

```dockerfile
EXPOSE 5000
```

Documents that the application listens on port **5000**.

---

### Start Application

```dockerfile
CMD ["npm","start"]
```

Runs the application when the container starts.

---

# 4️⃣ Create .dockerignore

```text
node_modules
.git
.gitignore
.env
npm-debug.log
```

## Benefits

- Smaller image size
- Faster builds
- Prevents copying secret files
- Reduces Docker build context

---

# 5️⃣ Build Docker Image

Build image

```bash
docker build -t opsstack-backend:v1 .
```

Verify

```bash
docker images
```

---

# 6️⃣ Docker Compose

Created a multi-container application using Docker Compose.

Services included

- Backend
- MongoDB

Main features used

- Build
- Environment Variables
- Named Volumes
- Health Checks
- Restart Policies
- Custom Networks
- depends_on

---

# 7️⃣ Health Check

```yaml
healthcheck:
  test: ["CMD","mongosh","--eval","db.adminCommand('ping')"]
  interval: 10s
  timeout: 5s
  retries: 5
```

### Purpose

- Wait until MongoDB is ready
- Improve application reliability
- Prevent startup failures

---

# 8️⃣ depends_on

```yaml
depends_on:
  mongodb:
    condition: service_healthy
```

### Startup Flow

```text
MongoDB
     │
     ▼
Health Check
     │
     ▼
Healthy
     │
     ▼
Backend Starts
```

---

# 9️⃣ Named Volume

```yaml
volumes:
  - mongodb-data:/data/db
```

### Benefits

- Persistent database storage
- Data survives container deletion
- Easy backups

---

# 🔟 Custom Network

```yaml
networks:
  - backend-network
```

### Benefits

- Secure communication
- Docker DNS
- No need for container IP addresses

Backend communicates with MongoDB using

```text
mongodb
```

instead of

```text
172.x.x.x
```

---

# 1️⃣1️⃣ Run Application

Start containers

```bash
docker compose up -d
```

Rebuild application

```bash
docker compose up -d --build
```

Stop application

```bash
docker compose down
```

---

# 1️⃣2️⃣ Verify Running Containers

```bash
docker compose ps
```

Example

```text
mongodb

opsstack-backend
```

---

# 1️⃣3️⃣ View Logs

```bash
docker compose logs

docker compose logs backend

docker compose logs -f
```

Observed

```text
Server running on port 5000

Connected to MongoDB
```

---

# 1️⃣4️⃣ Test Backend

```bash
curl http://localhost:5000
```

Output

```text
🚀 Welcome to OpsStack Backend
```

---

# 🛠 Commands Practiced

```bash
npm init -y

npm install express mongoose dotenv

npm install --save-dev nodemon

docker build -t opsstack-backend:v1 .

docker compose up -d

docker compose up -d --build

docker compose down

docker compose ps

docker compose logs

docker images

curl http://localhost:5000
```

---

# 🧠 Key Learnings

- Built a Docker image for a Node.js backend.
- Used Docker Compose to run multiple containers.
- Connected Node.js with MongoDB using Docker networking.
- Used environment variables for application configuration.
- Implemented health checks for MongoDB.
- Used `depends_on` to control startup order.
- Used named volumes for persistent storage.
- Learned Docker DNS for service-to-service communication.
- Tested the backend using `curl`.

---

# 🏗 Architecture

```text
          Client
             │
             ▼
      Express Backend
             │
             ▼
          MongoDB
             │
       Named Volume
```

---

# 🎯 Learning Outcome

After completing today's hands-on lab, I can:

- Build Docker images for Node.js applications.
- Create optimized Dockerfiles.
- Use `.dockerignore` effectively.
- Manage application configuration using `.env`.
- Build and run multi-container applications with Docker Compose.
- Connect multiple services using Docker networks.
- Configure health checks and startup dependencies.
- Persist database data using Docker volumes.
- Debug and verify running containers using Docker Compose commands.

---

## 🚀 Next Topic

**Week 03 – Git & GitHub**

- Git Fundamentals
- Git Workflow
- Branching
- Merging
- Rebasing
- Merge Conflicts
- GitHub Collaboration
- Pull Requests
- Tags & Releases
