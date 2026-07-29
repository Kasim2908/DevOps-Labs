# 🐳 Day 02 - Dockerfile

## 📌 Objective

Learn how to build custom Docker images using a **Dockerfile**. This lab covers Dockerfile instructions, image building, Docker layer caching, and best practices.

---

# 📚 Concepts Covered

- What is a Dockerfile?
- Docker Build Process
- FROM
- RUN
- WORKDIR
- COPY
- ADD
- ENV
- EXPOSE
- CMD
- ENTRYPOINT
- CMD vs ENTRYPOINT
- Shell Form vs Exec Form
- Docker Layer Caching
- Cache Invalidation
- Build Context

---

# 🏗 Docker Image Build Workflow

```
Dockerfile
     │
     ▼
docker build
     │
     ▼
Docker Image
     │
     ▼
docker run
     │
     ▼
Docker Container
```

---

# 📖 What is a Dockerfile?

A **Dockerfile** is a text file containing instructions to automatically build a Docker image.

Instead of manually installing software every time, Docker executes each instruction step-by-step to create a reusable image.

---

# 🧱 Common Dockerfile Instructions

## FROM

Specifies the base image.

```dockerfile
FROM ubuntu:24.04
```

Every Dockerfile starts with a `FROM` instruction.

---

## RUN

Executes commands while building the image.

```dockerfile
RUN apt update && apt install -y curl
```

Each `RUN` creates a new image layer.

---

## WORKDIR

Sets the working directory inside the container.

```dockerfile
WORKDIR /app
```

Equivalent to:

```bash
cd /app
```

---

## COPY

Copies files from the host to the image.

```dockerfile
COPY . .
```

Example:

```
Host
 ├── app.js
 └── package.json

↓

Container (/app)
 ├── app.js
 └── package.json
```

---

## ADD

Copies files like `COPY` but can also:

- Extract local tar files
- Download remote URLs

Example:

```dockerfile
ADD project.tar.gz /app
```

> **Best Practice:** Prefer `COPY` unless you specifically need `ADD` features.

---

## ENV

Defines environment variables.

```dockerfile
ENV APP_ENV=production
```

Access inside container:

```bash
echo $APP_ENV
```

---

## EXPOSE

Documents the port used by the application.

```dockerfile
EXPOSE 8080
```

It does **not** publish the port automatically.

---

## CMD

Specifies the default command executed when the container starts.

```dockerfile
CMD ["bash"]
```

---

## ENTRYPOINT

Defines the main executable for the container.

```dockerfile
ENTRYPOINT ["echo"]
```

---

# CMD vs ENTRYPOINT

Dockerfile:

```dockerfile
ENTRYPOINT ["echo"]
CMD ["Hello Docker"]
```

Run:

```bash
docker run demo
```

Output:

```
Hello Docker
```

Run:

```bash
docker run demo DevOps
```

Output:

```
DevOps
```

`CMD` provides default arguments, while `ENTRYPOINT` defines the executable.

---

# Shell Form vs Exec Form

## Shell Form

```dockerfile
CMD echo "Hello Docker"
```

Runs using:

```
/bin/sh -c
```

---

## Exec Form

```dockerfile
CMD ["echo", "Hello Docker"]
```

Recommended because:

- Better signal handling
- No shell process
- Predictable behavior

---

# Docker Layer Caching

Every Dockerfile instruction creates a layer.

```
FROM Ubuntu
      │
RUN apt install curl
      │
COPY .
      │
ENV APP_ENV
      │
CMD
```

If only the last instruction changes, Docker reuses all previous cached layers, making builds much faster.

---

# Cache Invalidation

Changing one instruction invalidates that layer and all layers below it.

Example:

```
FROM
RUN
COPY   ← changed
ENV
CMD
```

Docker rebuilds:

- COPY
- ENV
- CMD

Previous layers remain cached.

---

# Build Context

When running:

```bash
docker build .
```

The `.` represents the **build context**.

Docker sends all files in the current directory to the Docker daemon.

```
Project Folder
│
├── Dockerfile
├── app.js
├── package.json
└── README.md
```

Everything inside the folder is available for `COPY` or `ADD`.

---

# 🧪 Sample Dockerfiles

## 1️⃣ Ubuntu Lab

```dockerfile
FROM ubuntu:24.04

RUN apt update && apt install -y curl

WORKDIR /app

COPY . .

ENV APP_ENV=development

EXPOSE 8080

CMD ["bash"]
```

Build:

```bash
docker build -t ubuntu-lab .
```

Run:

```bash
docker run -it ubuntu-lab
```

---

## 2️⃣ Python Flask App

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

Build:

```bash
docker build -t flask-app .
```

Run:

```bash
docker run -p 5000:5000 flask-app
```

---

## 3️⃣ Node.js Express App

```dockerfile
FROM node:22-alpine

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

Build:

```bash
docker build -t node-app .
```

Run:

```bash
docker run -p 3000:3000 node-app
```

---

# 🧪 Hands-on Commands

Build an image:

```bash
docker build -t my-image:v1 .
```

List images:

```bash
docker images
```

Run container:

```bash
docker run -it my-image:v1
```

Run in detached mode:

```bash
docker run -d my-image:v1
```

Inspect image:

```bash
docker image inspect my-image:v1
```

View image history:

```bash
docker history my-image:v1
```

---

# ✅ Best Practices

- Use lightweight base images (e.g., Alpine) when appropriate.
- Keep images small.
- Use `.dockerignore` to exclude unnecessary files.
- Prefer `COPY` over `ADD` unless extra functionality is required.
- Combine related `RUN` commands to reduce image layers.
- Use the Exec Form for `CMD` and `ENTRYPOINT`.

---

# 🎯 Key Takeaways

- Dockerfiles automate image creation.
- Every instruction creates a new layer.
- Docker uses layer caching to speed up builds.
- `CMD` defines the default command.
- `ENTRYPOINT` defines the executable.
- `COPY` transfers files from the host to the image.
- `WORKDIR` sets the working directory.
- `ENV` defines environment variables.
- `EXPOSE` documents the application port.

---

# 💡 Interview Questions

### 1. What is a Dockerfile?

A Dockerfile is a text file containing instructions to build a Docker image automatically.

---

### 2. What is the difference between COPY and ADD?

- **COPY:** Copies files from the host into the image.
- **ADD:** Can also extract local archives and download remote URLs.

---

### 3. What is Docker layer caching?

Docker caches image layers to avoid rebuilding unchanged instructions, improving build performance.

---

### 4. What is Build Context?

The build context is the directory sent to the Docker daemon during `docker build`, containing files available to the Dockerfile.

---

### 5. Difference between CMD and ENTRYPOINT?

- **CMD:** Specifies the default command or arguments and can be overridden.
- **ENTRYPOINT:** Defines the main executable and is intended to remain fixed.

---

### 6. Why is Exec Form preferred?

It avoids running through a shell, provides better signal handling, and is more predictable for containerized applications.

---

---

# 📚 Additional Resources

## 📖 Official Docker Documentation

- 📘 Dockerfile Overview  
  https://docs.docker.com/build/concepts/dockerfile/

- 📘 Dockerfile Reference (All Instructions)  
  https://docs.docker.com/reference/dockerfile/

- 📘 Docker Build Documentation  
  https://docs.docker.com/build/

- 📘 Docker Build Context  
  https://docs.docker.com/build/concepts/context/

- 📘 Docker Image Best Practices  
  https://docs.docker.com/build/building/best-practices/

- 📘 Docker CLI Reference  
  https://docs.docker.com/reference/cli/docker/

- 📘 docker build Command  
  https://docs.docker.com/reference/cli/docker/buildx/build/

---

## 🎥 Recommended Videos

### Dockerfile Basics
- TechWorld with Nana – Docker Tutorial for Beginners
  https://www.youtube.com/results?search_query=TechWorld+with+Nana+Docker+Tutorial

### Dockerfile Deep Dive
- Dockerfile Tutorial
  https://www.youtube.com/results?search_query=Dockerfile+Tutorial

### Docker Image Layers & Caching
- Docker Layer Caching Explained
  https://www.youtube.com/results?search_query=Docker+Layer+Caching

---

## 📖 Recommended Reading

- Docker Best Practices for Building Images
  https://docs.docker.com/build/building/best-practices/

- Understanding Docker Image Layers
  https://docs.docker.com/get-started/docker-concepts/building-images/understanding-image-layers/

---

## 📝 Practice Exercises

### Exercise 1
Create a Dockerfile that:
- Uses `ubuntu:24.04`
- Installs `git`
- Sets the working directory to `/project`
- Starts with `bash`

---

### Exercise 2
Create a Dockerfile that:
- Uses `ubuntu:24.04`
- Sets an environment variable named `APP_NAME`
- Prints its value after running the container

---

### Exercise 3
Create a Dockerfile using:
- `ENTRYPOINT ["echo"]`
- `CMD ["Docker"]`

Run the container:
- Without arguments
- With your name as an argument

Observe how `CMD` is overridden.

---

### Exercise 4
Modify the Dockerfile by changing only the `ENV` instruction and rebuild the image.

Observe which layers are cached and which layers are rebuilt.

---

### Exercise 5
Experiment with both Shell Form and Exec Form of `CMD` and compare their behavior.

---

# 📂 Project Structure

```
Day-02-Dockerfile/
│── README.md
└── screenshots/
```

Add screenshots of your image builds, Dockerfile execution, and container outputs inside the `screenshots/` directory.

---

# 🏁 Conclusion

In this lab, you learned how to create custom Docker images using Dockerfiles, optimize builds with layer caching, and use essential Dockerfile instructions. These skills form the foundation for containerizing applications and building efficient, production-ready Docker images.
