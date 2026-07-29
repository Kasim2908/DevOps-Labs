# 🐳 Day 03 - Docker Storage (Volumes & Bind Mounts)

## 📌 Objective

Learn how Docker manages data and how to persist container data using **Docker Volumes** and **Bind Mounts**. This lab demonstrates why container data is lost after removal and how Docker provides persistent storage solutions.

---

# 📚 Concepts Covered

- Docker Writable Layer
- Why Container Data is Lost
- Docker Volumes
- Named Volumes
- Volume Persistence
- Bind Mounts
- Volume vs Bind Mount
- Host ↔ Container File Synchronization
- Real-world Use Cases

---

# 🏗 Docker Storage Architecture

```
                Docker Host
+---------------------------------------+
|                                       |
|   ~/bind-mount-demo/                  |
|           │                           |
|           │ Bind Mount                |
|           ▼                           |
|      +-----------+                    |
|      | Container |                    |
|      |   /app    |                    |
|      +-----------+                    |
|                                       |
|                                       |
|   Docker Volume                       |
|      my-volume                        |
|           │                           |
|           ▼                           |
|      +-----------+                    |
|      |  /data    |                    |
|      +-----------+                    |
|                                       |
+---------------------------------------+
```

---

# 📖 Writable Layer

Every Docker container has a **Writable Layer** on top of the image layers.

```
Read-only Image Layers
-------------------------
Writable Layer
```

All changes made inside the container are stored in this writable layer.

If the container is removed, the writable layer is deleted.

---

# ❓ Why Container Data is Lost?

Example:

```bash
docker run -it ubuntu bash
```

Create a file:

```bash
echo "Hello Docker" > demo.txt
```

Exit and remove the container:

```bash
docker rm <container-id>
```

Run a new container:

```bash
docker run -it ubuntu bash
```

The file no longer exists because it was stored in the container's writable layer.

---

# 📦 Docker Volumes

Docker Volumes provide persistent storage managed by Docker.

A volume exists independently of containers.

```
Container A
      │
      ▼
+-------------+
| my-volume   |
+-------------+
      ▲
      │
Container B
```

Even if a container is deleted, the volume and its data remain.

---

# 🧪 Hands-on Lab: Named Volume

### Create a volume

```bash
docker volume create my-volume
```

---

### List volumes

```bash
docker volume ls
```

---

### Run a container using the volume

```bash
docker run -it -v my-volume:/data ubuntu:latest bash
```

---

### Create a file

```bash
echo "Docker Volume Demo" > /data/demo.txt
```

---

### Remove the container

```bash
docker rm <container-id>
```

---

### Start a new container

```bash
docker run -it -v my-volume:/data ubuntu:latest bash
```

---

### Verify

```bash
cat /data/demo.txt
```

Output:

```
Docker Volume Demo
```

✅ Data persisted because it was stored in the Docker Volume.

---

# 📁 Bind Mounts

A Bind Mount maps a directory from the **host machine** into a container.

```
Host Directory
/home/ubuntu/bind-mount-demo
          │
          │
          ▼
Container
/app
```

Both host and container access the same files.

---

# 🧪 Hands-on Lab: Bind Mount

### Host

```bash
mkdir ~/bind-mount-demo

echo "Hello from Host" > ~/bind-mount-demo/index.html
```

---

### Run container

```bash
docker run -it -v ~/bind-mount-demo:/app ubuntu:latest bash
```

---

### Verify inside container

```bash
cat /app/index.html
```

Output:

```
Hello from Host
```

---

### Modify inside container

```bash
echo "Changed from Container" > /app/index.html
```

---

### Verify on host

```bash
cat ~/bind-mount-demo/index.html
```

Output:

```
Changed from Container
```

---

### Modify on Host

```bash
echo "Changed from Host" > ~/bind-mount-demo/index.html
```

---

### Verify inside container

```bash
cat /app/index.html
```

Output:

```
Changed from Host
```

✅ Changes are immediately reflected because both environments share the same file.

---

# 🔄 Volume vs Bind Mount

| Docker Volume | Bind Mount |
|--------------|------------|
| Managed by Docker | Managed by the Host OS |
| Better for production | Better for development |
| Portable | Host path dependent |
| Ideal for databases | Ideal for source code |

---

# 🌍 Real-world Use Cases

### Docker Volumes

- MySQL
- PostgreSQL
- MongoDB
- Jenkins Home
- Prometheus Data
- Grafana Data

---

### Bind Mounts

- React Development
- Node.js Development
- Python Development
- Configuration Files
- Log Files

---

# 🧪 Practice Questions

## Practice 1: Volume Persistence

Create a volume named `project-data`, mount it inside `/app`, create a file named `notes.txt`, remove the container, start a new container with the same volume, and verify that the file still exists.

---

## Practice 2: Bind Mount Synchronization

Create a host directory named `docker-lab`, mount it to `/workspace` inside a container, create a file from the host, verify it in the container, then modify it inside the container and confirm the changes on the host.

---

## Practice 3: Compare Storage Types

Create one container using a Docker Volume and another using a Bind Mount. Observe how each stores data and note the differences in management and use cases.

---

# 🎯 Key Takeaways

- Every container has a writable layer.
- Data stored only in the writable layer is lost when the container is removed.
- Docker Volumes provide persistent storage.
- Bind Mounts directly connect host directories to containers.
- Volumes are recommended for production workloads.
- Bind Mounts are ideal during application development.

---

# 💡 Interview Questions

### 1. What is the Docker Writable Layer?

The writable layer is the top layer added to a running container where all runtime changes are stored.

---

### 2. Why is data lost when a container is removed?

Because the writable layer is deleted along with the container.

---

### 3. What is a Docker Volume?

A Docker-managed persistent storage mechanism that exists independently of containers.

---

### 4. What is a Bind Mount?

A mapping between a host directory and a directory inside a container, allowing both to access the same files.

---

### 5. Difference between Volume and Bind Mount?

Volumes are managed by Docker and are best suited for persistent application data, while Bind Mounts are managed by the host and are commonly used during development.

---

### 6. Which storage option is recommended for databases?

Docker Volumes, because they provide persistent and Docker-managed storage.

---

### 7. Which storage option is preferred for React or Node.js development?

Bind Mounts, because source code changes on the host are immediately visible inside the container.

---

# 📂 Project Structure

```
Day-03-Docker-Storage/
│── README.md
└── screenshots/
```

Add screenshots of:
- Volume creation
- Volume persistence demonstration
- Bind Mount setup
- Host ↔ Container synchronization

---


---

# 📚 Additional Resources

## 📖 Official Docker Documentation

- 📘 Docker Storage Overview  
  https://docs.docker.com/engine/storage/

- 📘 Docker Volumes  
  https://docs.docker.com/engine/storage/volumes/

- 📘 Bind Mounts  
  https://docs.docker.com/engine/storage/bind-mounts/

- 📘 Docker CLI Reference  
  https://docs.docker.com/reference/cli/docker/

- 📘 Docker Run Reference  
  https://docs.docker.com/reference/cli/docker/container/run/

---

## 🎥 Recommended Videos

- Docker Volumes Explained (TechWorld with Nana)
  https://www.youtube.com/results?search_query=TechWorld+with+Nana+Docker+Volumes

- Docker Storage Explained
  https://www.youtube.com/results?search_query=Docker+Volumes+and+Bind+Mounts

---

## 📝 Practice More

### Exercise 1
Create a Docker Volume named `mysql-data` and mount it to `/var/lib/mysql`.

---

### Exercise 2
Create a Bind Mount for a Node.js project and verify that file changes on the host are reflected inside the container.

---

### Exercise 3
Compare a Docker Volume and a Bind Mount by storing the same file in each and observing the differences after removing the container.

---

# 🏁 Conclusion

In this lab, you explored Docker's storage mechanisms and learned how to preserve data beyond a container's lifecycle. By working with Docker Volumes and Bind Mounts, you gained practical experience with persistent storage and file sharing—essential concepts for deploying stateful applications and supporting efficient development workflows.
