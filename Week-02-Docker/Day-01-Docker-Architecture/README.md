# 🐳 Day 01 - Docker Architecture

## 📌 Objective

Understand the fundamentals of Docker, its architecture, and how containers are created and managed. This lab covers Docker's core components and the lifecycle of running a container.

---

# 📚 Concepts Covered

- What is Docker?
- Why Docker?
- Virtual Machines vs Docker Containers
- Docker Architecture
- Docker Client
- Docker Daemon
- Docker Engine
- Docker Images
- Docker Containers
- Docker Registry
- containerd
- runc
- Read-only Layers
- Writable Layer
- Copy-on-Write (CoW)
- Docker Container Lifecycle

---

# 🏗 Docker Architecture

```
                +----------------------+
                |    Docker Client     |
                | (docker commands)    |
                +----------+-----------+
                           |
                           | REST API
                           |
                +----------v-----------+
                |    Docker Daemon     |
                |      (dockerd)       |
                +----------+-----------+
                           |
        -----------------------------------------
        |                  |                    |
        |                  |                    |
+-------v------+   +-------v------+   +--------v-------+
| Docker Image |   | Docker Volume|   | Docker Network |
+--------------+   +--------------+   +----------------+
                           |
                    +------v------+
                    | containerd  |
                    +------+------+
                           |
                    +------v------+
                    |    runc     |
                    +------+------+
                           |
                    +------v------+
                    | Container   |
                    +-------------+
```

---

# 📖 Docker Components

## Docker Client

The Docker Client is the command-line interface (CLI) used to communicate with the Docker Daemon.

Example:

```bash
docker run nginx
docker ps
docker images
```

---

## Docker Daemon (dockerd)

The Docker Daemon is the background service responsible for:

- Building images
- Running containers
- Managing volumes
- Managing networks
- Pulling images from registries

---

## Docker Image

A Docker Image is a read-only template used to create containers.

Examples:

- nginx
- ubuntu
- node
- mysql

List images:

```bash
docker images
```

---

## Docker Container

A container is a running instance of a Docker image.

Run a container:

```bash
docker run nginx
```

List running containers:

```bash
docker ps
```

List all containers:

```bash
docker ps -a
```

---

## Docker Registry

A Docker Registry stores Docker images.

Popular registries:

- Docker Hub
- GitHub Container Registry
- Amazon ECR
- Google Artifact Registry

Pull an image:

```bash
docker pull nginx
```

---

# ⚙ containerd

containerd is responsible for managing the complete lifecycle of containers.

Responsibilities:

- Image management
- Container lifecycle
- Snapshot management
- Storage

---

# ⚙ runc

runc is a lightweight runtime responsible for creating and running Linux containers.

It:

- Creates namespaces
- Creates cgroups
- Starts the container process

---

# 📦 Docker Image Layers

Docker images are built using multiple read-only layers.

Example:

```
Ubuntu Base Image
       ↓
Install Python
       ↓
Install Node.js
       ↓
Copy Application
```

Each instruction creates a new image layer.

---

# ✏ Writable Layer

When a container starts, Docker adds a writable layer above the image.

```
Read-only Image
-----------------------
Writable Layer
```

All changes made inside the container are stored in this writable layer.

If the container is removed, the writable layer is deleted.

---

# 🔄 Copy-on-Write (CoW)

Docker uses the Copy-on-Write mechanism.

If a file from a read-only layer is modified:

1. Docker copies the file to the writable layer.
2. The original image layer remains unchanged.

Benefits:

- Faster containers
- Less storage usage
- Efficient image sharing

---

# 🔁 Container Lifecycle

```
docker run
      │
      ▼
Created
      │
      ▼
Running
      │
 ┌────┴─────┐
 │          │
Stop      Restart
 │          │
 ▼          ▼
Exited -----
 │
 ▼
Removed
```

---

# 🧪 Hands-on Commands

### Pull an image

```bash
docker pull nginx
```

---

### Run a container

```bash
docker run -d nginx
```

---

### List running containers

```bash
docker ps
```

---

### List all containers

```bash
docker ps -a
```

---

### Stop a container

```bash
docker stop <container-id>
```

---

### Remove a container

```bash
docker rm <container-id>
```

---

### Remove an image

```bash
docker rmi nginx
```

---

# 🎯 Key Takeaways

- Docker packages applications with their dependencies.
- Docker uses images to create containers.
- Images are immutable and consist of read-only layers.
- Containers have an additional writable layer.
- containerd manages the container lifecycle.
- runc creates and starts containers.
- Docker Daemon performs all container management tasks.
- Copy-on-Write improves storage efficiency.

---

# 💡 Interview Questions

### 1. What is Docker?

Docker is a containerization platform that packages applications and their dependencies into lightweight, portable containers.

---

### 2. What is the difference between an Image and a Container?

- **Image:** Read-only blueprint.
- **Container:** Running instance of an image.

---

### 3. What is the role of the Docker Daemon?

It manages images, containers, networks, and volumes.

---

### 4. What is containerd?

containerd manages the lifecycle of containers and images.

---

### 5. What is runc?

runc is the low-level runtime that creates and starts Linux containers.

---

### 6. What is Copy-on-Write?

Copy-on-Write copies a file from a read-only layer into the writable layer only when it is modified.

---

### 7. Why are Docker containers lightweight?

Containers share the host operating system kernel instead of running a full guest operating system like virtual machines.

---

# 📂 Project Structure

```
Day-01-Docker-Architecture/
│── README.md
└── screenshots/
```

Add screenshots of your Docker commands and outputs inside the `screenshots/` directory.

---

# 🏁 Conclusion

This lab introduced Docker's architecture, core components, image layering, container lifecycle, and storage concepts. Understanding these fundamentals is essential before moving on to Dockerfiles, networking, volumes, and orchestration tools like Kubernetes.
