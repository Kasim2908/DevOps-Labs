# 🐳 Day 06 - Advanced Docker Compose

## 📌 Introduction

Docker Compose is a powerful tool for defining and managing multi-container Docker applications. In Day 05, we learned the fundamentals of Docker Compose, including creating services, networking, volumes, and environment variables.

In this session, we explored advanced Docker Compose concepts that are commonly used in real-world DevOps projects. We learned how to build custom Docker images, control service startup order, perform health checks, and use bind mounts for efficient development workflows.

---

# 🎯 Learning Objectives

- Understand the difference between `build` and `image`
- Learn how `depends_on` works
- Understand Docker Health Checks
- Use Bind Mounts with Docker Compose
- Learn production-ready Docker Compose practices
- Troubleshoot common Docker Compose issues

---

# 📚 Topics Covered

- Build vs Image
- depends_on
- Health Checks
- Bind Mounts
- Restart Policies
- Docker Compose Best Practices

---

# 🏗 Project Structure

```
Day-06-Advanced-Docker-Compose/
│
├── README.md
├── commands.md
│
├── compose-build/
│   ├── Dockerfile
│   ├── compose.yaml
│   └── index.html
│
├── compose-depends/
│   └── compose.yaml
│
├── compose-health/
│   └── compose.yaml
│
└── compose-bind/
    ├── compose.yaml
    └── index.html
```

---

# 1️⃣ Build vs Image

Docker Compose supports two methods for creating containers.

## Using `image`

The `image` keyword tells Docker Compose to use an existing Docker image from the local system or Docker Hub.

Example

```yaml
services:
  web:
    image: nginx:latest
```

Docker simply pulls the image (if it does not already exist) and starts the container.

---

## Using `build`

The `build` keyword tells Docker Compose to build a Docker image from a local Dockerfile before starting the container.

Example

```yaml
services:
  web:
    build: .
```

Docker Compose performs the following steps:

```
Dockerfile
      │
      ▼
Build Image
      │
      ▼
Create Container
      │
      ▼
Run Container
```

---

## Build vs Image

| Build | Image |
|--------|-------|
| Uses Dockerfile | Uses existing image |
| Creates custom image | Pulls image from Docker Hub |
| Used for custom applications | Used for official images |
| Example: Node.js, Python Apps | Example: Nginx, Redis, MongoDB |

---

# 2️⃣ depends_on

In multi-container applications, some services depend on other services.

Example

```
Browser
    │
    ▼
Node.js Application
    │
    ▼
MongoDB
```

The application depends on MongoDB.

Without `depends_on`, Docker starts both containers simultaneously.

```
MongoDB   ⏳ Starting

Node.js   ⏳ Starting
```

Node.js may try to connect before MongoDB is ready.

Result:

```
Connection Refused

MongoServerSelectionError
```

---

## Using depends_on

```yaml
services:
  app:
    image: nginx

    depends_on:
      - database

  database:
    image: mongo
```

Now Docker starts:

```
Database

↓

Application
```

---

## Important Note

`depends_on`

✅ Controls startup order.

❌ Does NOT wait until the database is fully ready.

---

# 3️⃣ Docker Health Checks

Health Checks determine whether the application inside a container is actually healthy.

Instead of asking

```
Did the container start?
```

Docker asks

```
Is the application running correctly?
```

Example

```yaml
healthcheck:
  test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]

  interval: 10s

  timeout: 5s

  retries: 5
```

---

## Health Check Parameters

### test

Command Docker executes.

Example

```bash
mongosh --eval "db.adminCommand('ping')"
```

---

### interval

Time between health checks.

Example

```
Every 10 seconds
```

---

### timeout

Maximum waiting time before Docker considers the health check failed.

---

### retries

Number of failed attempts before Docker marks the container as unhealthy.

---

## Combining depends_on with Health Checks

```yaml
services:

  database:

    image: mongo

    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]

  app:

    image: nginx

    depends_on:

      database:

        condition: service_healthy
```

Startup flow

```
Start MongoDB

↓

Run Health Check

↓

MongoDB Healthy

↓

Start Application
```

This provides a safer startup sequence for dependent services.

---

# 4️⃣ Bind Mounts

Bind Mounts map files or directories from the host machine directly into a running container.

Example

```yaml
services:
  web:
    image: nginx

    volumes:
      - ./index.html:/usr/share/nginx/html/index.html
```

---

## How Bind Mounts Work

```
Host Machine

index.html
      │
      │
      ▼
Container

/usr/share/nginx/html/index.html
```

Both point to the same file.

---

## Benefits

- Live code changes
- Faster development
- No image rebuild required
- No container restart required when editing mounted files

---

## Bind Mount vs Named Volume

| Bind Mount | Named Volume |
|------------|--------------|
| Maps host files | Managed by Docker |
| Ideal for source code | Ideal for databases |
| Easy to edit | Persistent storage |
| Development | Production |

---

# 5️⃣ Restart Policies

Docker Compose supports multiple restart policies.

## no

Never restart the container.

```yaml
restart: "no"
```

---

## always

Always restart the container.

```yaml
restart: always
```

---

## unless-stopped

Restart unless manually stopped.

```yaml
restart: unless-stopped
```

---

## on-failure

Restart only if the container exits with an error.

```yaml
restart: on-failure
```

---

# 📋 Useful Commands

Build Image

```bash
docker compose build
```

Run Containers

```bash
docker compose up -d
```

Stop Containers

```bash
docker compose down
```

Force Recreate

```bash
docker compose up -d --force-recreate
```

Show Running Containers

```bash
docker compose ps
```

View Logs

```bash
docker compose logs
```

Follow Logs

```bash
docker compose logs -f
```

Execute Commands

```bash
docker compose exec web bash
```

Validate Compose File

```bash
docker compose config
```

---

# 🧠 Interview Questions

### What is the difference between build and image?

`build` creates a Docker image from a Dockerfile.

`image` uses an existing Docker image.

---

### What does depends_on do?

It controls the startup order of services.

---

### Does depends_on wait until a service is ready?

No.

It only waits until the container has started.

---

### Why do we need Health Checks?

Health Checks verify that the application inside the container is healthy before dependent services start using it.

---

### What is a Bind Mount?

A Bind Mount maps a file or directory from the host machine directly into the container.

---

### Why are Bind Mounts useful?

They allow developers to modify files on the host machine and immediately see changes inside the running container.

---

### Difference between Bind Mount and Named Volume?

Bind Mounts are mainly used during development.

Named Volumes are mainly used for persistent application data.

---

### What does docker compose build do?

Builds Docker images using the Dockerfile specified in the Compose configuration.

---

### What is restart: unless-stopped?

The container automatically restarts unless it has been manually stopped by the user.

---

# 📝 Best Practices

- Use `build` for custom applications.
- Use `image` for official Docker images.
- Combine `depends_on` with Health Checks for better startup reliability.
- Use Bind Mounts during development.
- Use Named Volumes for databases.
- Store configuration using `.env` files.
- Never commit sensitive credentials to GitHub.
- Validate Compose files before deployment.
- Use meaningful service names.

---

# 🚀 Key Takeaways

- Learned the difference between `build` and `image`.
- Controlled service startup using `depends_on`.
- Implemented Health Checks for reliable application startup.
- Used Bind Mounts for live code synchronization.
- Explored Docker restart policies.
- Practiced production-ready Docker Compose workflows.

---

# 📚 Official Documentation

- Docker Compose Overview  
  https://docs.docker.com/compose/

- Compose File Reference  
  https://docs.docker.com/reference/compose-file/

- Docker Health Checks  
  https://docs.docker.com/reference/dockerfile/#healthcheck

- Docker Volumes  
  https://docs.docker.com/engine/storage/volumes/

- Bind Mounts  
  https://docs.docker.com/engine/storage/bind-mounts/

---

# 🎯 Next Topic

➡️ Docker Compose Advanced Project

- Custom Networks
- Multi-Service Architecture
- Node.js + MongoDB
- Health Checks
- Production Docker Compose
- Reverse Proxy
- Best Practices

---

⭐ If you found this helpful, don't forget to star the repository and continue your DevOps learning journey!
