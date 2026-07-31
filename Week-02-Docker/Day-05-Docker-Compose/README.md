
# 🐳 Day 05 - Docker Compose

## 📌 Introduction

Managing multiple Docker containers using individual `docker run` commands quickly becomes difficult as applications grow. Docker Compose solves this problem by allowing you to define and manage multi-container applications using a single YAML configuration file.

In this session, I learned how to create, manage, and orchestrate multiple containers with Docker Compose, use environment variables, create persistent volumes, and understand automatic networking between services.

---

# 🎯 Learning Objectives

- Understand Docker Compose and its benefits
- Create and manage multi-container applications
- Learn the structure of a `compose.yaml` file
- Use Docker Volumes for persistent storage
- Configure Environment Variables using `.env`
- Understand Docker Compose networking
- Troubleshoot common Docker Compose issues

---

# 📚 What is Docker Compose?

Docker Compose is a tool used to define and run multi-container Docker applications.

Instead of running multiple long `docker run` commands, Docker Compose allows us to describe the entire application inside a single YAML file.

Example:

```yaml
services:
  web:
    image: nginx

  database:
    image: mongo
```

Run everything with:

```bash
docker compose up
```

Stop everything with:

```bash
docker compose down
```

---

# 🏗 Why Docker Compose?

Without Docker Compose:

- Multiple docker run commands
- Manual networking
- Manual volume creation
- Manual environment variables
- Difficult to manage

With Docker Compose:

- One YAML configuration
- Automatic networking
- Automatic DNS
- Easy scaling
- Infrastructure as Code

---

# 📂 Project Structure

```
Day-05-Docker-Compose/
│
├── README.md
├── compose-nginx/
│   └── compose.yaml
│
├── compose-multi/
│   └── compose.yaml
│
└── compose-env/
    ├── compose.yaml
    └── .env.example
```

---

# 🧪 Lab 1 - Single Container (Nginx)

compose.yaml

```yaml
services:
  web:
    image: nginx:latest
    container_name: nginx-compose

    ports:
      - "8080:80"

    restart: unless-stopped
```

Run

```bash
docker compose up -d
```

Check

```bash
docker compose ps
```

Stop

```bash
docker compose down
```

---

# 🧪 Lab 2 - Multi Container Application

compose.yaml

```yaml
services:
  web:
    image: nginx:latest

  database:
    image: mongo:latest
```

Docker Compose automatically:

- Creates a network
- Connects both containers
- Provides DNS
- Starts services together

---

# 🌐 Docker Compose Networking

Docker Compose automatically creates a bridge network.

Example

```
compose-multi_default
```

Both containers communicate using service names.

Example

```
web
 ↓
database
```

Connection string

```
mongodb://database:27017
```

No IP address is required.

---

# 📦 Docker Volumes

Containers are temporary.

Without volumes:

```
Container Deleted
        ↓
Data Lost
```

With volumes:

```
Container Deleted
        ↓
Volume Exists
        ↓
Data Preserved
```

Example

```yaml
volumes:
  - mongodb-data:/data/db

volumes:
  mongodb-data:
```

Useful Commands

```bash
docker volume ls

docker volume inspect compose-multi_mongodb-data
```

---

# 🌱 Environment Variables

Instead of hardcoding values

```yaml
environment:
  MONGO_INITDB_ROOT_USERNAME: admin
```

Use

`.env`

```env
MONGO_USER=admin
MONGO_PASSWORD=admin123
MONGO_PORT=27017
```

compose.yaml

```yaml
environment:
  MONGO_INITDB_ROOT_USERNAME: ${MONGO_USER}
  MONGO_INITDB_ROOT_PASSWORD: ${MONGO_PASSWORD}
```

Validate

```bash
docker compose config
```

---

# 🔧 Useful Docker Compose Commands

Start containers

```bash
docker compose up
```

Run in background

```bash
docker compose up -d
```

View containers

```bash
docker compose ps
```

View logs

```bash
docker compose logs
```

Stop application

```bash
docker compose down
```

Stop and remove volumes

```bash
docker compose down -v
```

Restart

```bash
docker compose restart
```

List Docker volumes

```bash
docker volume ls
```

Inspect volume

```bash
docker volume inspect <volume-name>
```

Validate compose file

```bash
docker compose config
```

---

# 🧠 Interview Questions

### What is Docker Compose?

Docker Compose is a tool used to define and manage multi-container Docker applications using a YAML configuration file.

---

### What is compose.yaml?

A configuration file that defines services, networks, volumes, ports, and environment variables.

---

### Difference between Docker Run and Docker Compose?

| Docker Run | Docker Compose |
|------------|----------------|
| One container | Multiple containers |
| Long commands | YAML configuration |
| Manual networking | Automatic networking |
| Difficult to manage | Easy to manage |

---

### What is a Docker Volume?

A Docker Volume stores data outside the container lifecycle, ensuring data persists even after containers are removed.

---

### Difference between docker compose down and docker compose down -v?

`docker compose down`

- Removes containers
- Removes networks
- Keeps volumes

`docker compose down -v`

- Removes containers
- Removes networks
- Removes volumes

---

### Why use a .env file?

- Better security
- Easier configuration
- Environment-specific values
- Cleaner compose files

---

### Why did "Port is already allocated" occur?

Because another container was already using the same host port.

Example

```
MongoDB Container A
↓

Host Port 27017

MongoDB Container B
↓

Host Port 27017 ❌
```

Use a different host port or stop the existing container.

---

# 📖 Best Practices

- Use Docker Compose for multi-container applications.
- Store configuration in `.env` files.
- Never commit real secrets to GitHub.
- Use named volumes for databases.
- Validate compose files before deployment.
- Use meaningful service names.
- Keep compose files readable and organized.

---

# 🎯 Key Takeaways

- Learned Docker Compose fundamentals.
- Created single and multi-container applications.
- Used Docker Volumes for persistent storage.
- Configured environment variables.
- Understood Docker networking and service discovery.
- Troubleshot port conflicts.
- Practiced production-ready Docker Compose workflows.

---

# 📚 Resources

- https://docs.docker.com/compose/
- https://docs.docker.com/engine/storage/volumes/
- https://docs.docker.com/reference/compose-file/

---

- Build vs Image
- Custom Networks
- Multi-Service Application (Node.js + MongoDB)
