# 🌐 Day 04 - Docker Networking

> Part of my **DevOps Learning Journey** 🚀

## 📌 Objective

The goal of this session was to understand how Docker enables communication between containers, the host machine, and external users. I explored Docker's networking architecture, created custom networks, tested container-to-container communication, learned Docker DNS, implemented port mapping, and understood production networking best practices.

---

# 📚 Topics Covered

- Docker Networking Overview
- Docker Network Drivers
- Bridge Network
- Host Network
- None Network
- Docker Network Commands
- Inspecting Networks
- Docker Bridge (`docker0`)
- IPAM (IP Address Management)
- Custom Bridge Network
- Docker DNS
- Container-to-Container Communication
- Port Mapping (`-p`)
- EXPOSE vs `-p`
- Publish All Ports (`-P`)
- Docker Network Connect
- Docker Network Disconnect
- Multi-Network Containers
- Production Network Architecture
- Troubleshooting Docker Networking
- Interview Questions

---

# 🌐 What is Docker Networking?

Docker Networking allows containers to communicate with:

- Other containers
- The Docker host
- External systems (Internet)
- Applications running outside Docker

Without networking, containers would be isolated and unable to communicate.

---

# 🏗 Docker Network Architecture

```
                 Internet
                      │
                      ▼
                Docker Host
                      │
          +-----------------------+
          |      docker0 Bridge   |
          +-----------------------+
            │        │        │
            ▼        ▼        ▼
        Container  Container  Container
```

---

# Docker Network Drivers

Docker provides several network drivers.

## 1. Bridge Network (Default)

The Bridge Network is automatically created during Docker installation.

Characteristics:

- Default network
- Containers receive private IP addresses
- Supports Internet access
- Uses Linux bridge (`docker0`)
- Suitable for standalone containers

Example:

```bash
docker run nginx
```

Docker automatically connects the container to the **bridge** network.

---

## 2. Host Network

In Host mode, the container shares the host's networking stack.

Characteristics:

- No separate container IP
- No NAT
- Better performance
- Suitable for networking-intensive applications

Example:

```bash
docker run --network host nginx
```

---

## 3. None Network

Completely disables networking.

Characteristics:

- No Internet
- No communication
- Fully isolated container

Example:

```bash
docker run --network none ubuntu
```

---

# Docker Network Commands

List networks

```bash
docker network ls
```

Inspect network

```bash
docker network inspect bridge
```

Create network

```bash
docker network create devops-net
```

Remove network

```bash
docker network rm devops-net
```

---

# Docker Bridge (`docker0`)

Docker creates a virtual Linux bridge called:

```
docker0
```

Verify:

```bash
ip addr show docker0
```

The bridge connects containers on the default bridge network.

---

# IPAM (IP Address Management)

Docker automatically assigns IP addresses.

Example:

```
Subnet:
172.17.0.0/16

Gateway:
172.17.0.1
```

Every container receives a unique IP from the subnet.

---

# Custom Bridge Network

Production applications rarely use the default bridge.

Create:

```bash
docker network create devops-net
```

Run container:

```bash
docker run -d --network devops-net nginx
```

Advantages:

- Better isolation
- Automatic DNS
- Easy container communication
- Used by Docker Compose

---

# Docker DNS

Containers on a custom bridge network can communicate using container names.

Example:

```
web1
web2
```

Instead of:

```
172.18.0.2
```

Docker automatically resolves:

```
web1
↓

172.18.0.2
```

---

# Container-to-Container Communication

Example:

```bash
docker exec -it web2 bash
```

Install ping:

```bash
apt update
apt install -y iputils-ping
```

Ping another container:

```bash
ping web1
```

Docker DNS resolves the container name automatically.

---

# Port Mapping

Containers are isolated.

Users cannot access container ports directly.

Publish ports using:

```bash
docker run -p 8080:80 nginx
```

Meaning:

```
Host Port
8080
    │
    ▼
Container Port
80
```

Access:

```
http://EC2-Public-IP:8080
```

---

# EXPOSE vs -p

## EXPOSE

Dockerfile instruction.

Example:

```dockerfile
EXPOSE 80
```

Purpose:

- Documentation
- Indicates the application listens on port 80

It **does not** publish the port.

---

## -p

Publishes container ports to the host.

Example:

```bash
docker run -p 8080:80 nginx
```

Makes the application accessible from outside Docker.

---

# Publish All Ports (-P)

Docker automatically publishes all exposed ports.

Example:

```bash
docker run -P nginx
```

Example output:

```
32768 -> 80
```

Useful for testing.

---

# Docker Network Connect

Attach a running container to another network.

```bash
docker network connect devops-net nginx1
```

One container can belong to multiple networks.

---

# Docker Network Disconnect

Remove a container from a network.

```bash
docker network disconnect devops-net nginx1
```

---

# Multi-Network Architecture

```
             frontend-net
                    │
                    │
             Frontend App
                    │
                    │
               Backend API
              /           \
             /             \
 database-net          cache-net
      │                    │
 PostgreSQL             Redis
```

Benefits:

- Better security
- Isolation
- Scalability
- Easier management

---

# AWS EC2 Port Mapping

When running Docker on EC2:

Container:

```
-p 8080:80
```

Also allow the port in the **EC2 Security Group**.

Inbound Rule:

```
Type:
Custom TCP

Port:
8080

Source:
0.0.0.0/0
```

Without opening the port in AWS Security Groups, the application cannot be accessed externally.

---

# Common Troubleshooting

## Container not accessible

Check:

```bash
docker ps
```

Verify port mapping.

---

Check locally:

```bash
curl localhost:8080
```

---

Inspect network:

```bash
docker network inspect bridge
```

---

Verify Security Group:

- Port 8080 open
- Public IPv4 used
- Container running

---

# Important Commands

```bash
docker network ls

docker network inspect bridge

docker network create devops-net

docker network rm devops-net

docker network connect devops-net nginx1

docker network disconnect devops-net nginx1

docker run -d --network devops-net nginx

docker run -d -p 8080:80 nginx

docker run -P nginx

docker ps

docker inspect nginx1

docker exec -it web2 bash
```

---

# Interview Questions

### What are Docker's default networks?

- Bridge
- Host
- None

---

### What is docker0?

A Linux bridge created by Docker that connects containers on the default bridge network.

---

### What is IPAM?

IP Address Management. Docker automatically assigns IP addresses to containers.

---

### Why use a custom bridge network?

- Automatic DNS
- Better isolation
- Easier communication
- Used by Docker Compose

---

### Difference between EXPOSE and -p?

| EXPOSE | -p |
|---------|----|
| Documentation | Publishes ports |
| Dockerfile | docker run |
| Doesn't expose externally | Makes application accessible |

---

### Can a container belong to multiple networks?

Yes.

A container can connect to multiple Docker networks simultaneously.

---

### Why use container names instead of IP addresses?

Docker automatically updates DNS records if container IPs change.

Container names remain constant.

---

# Key Takeaways

- Docker networking enables communication between containers and external systems.
- The Bridge network is the default Docker network.
- Custom bridge networks provide automatic DNS resolution.
- Docker DNS allows communication using container names.
- Port mapping exposes container services to external users.
- `EXPOSE` documents ports; `-p` publishes them.
- A container can connect to multiple Docker networks.
- AWS Security Groups must allow published ports when using Docker on EC2.

---

# 📖 Extra Resources

## Official Documentation

- Docker Networking Overview  
  https://docs.docker.com/engine/network/

- Bridge Network Driver  
  https://docs.docker.com/network/drivers/bridge/

- Host Network Driver  
  https://docs.docker.com/network/drivers/host/

- None Network Driver  
  https://docs.docker.com/network/drivers/none/

- Docker CLI Network Commands  
  https://docs.docker.com/reference/cli/docker/network/

- Docker Port Publishing  
  https://docs.docker.com/get-started/docker-concepts/running-containers/publishing-ports/

---

## Learn More

- Docker Getting Started  
  https://docs.docker.com/get-started/

- Docker Compose Documentation  
  https://docs.docker.com/compose/

- Play with Docker (Free Playground)  
  https://labs.play-with-docker.com/

---



⭐ If you found this repository useful, consider giving it a **Star**!
