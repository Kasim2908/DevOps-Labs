1. Introduction to Networking
2. Types of Networks
3. OSI Model
4. TCP/IP Model
5. IP Address
6. IPv4 vs IPv6
7. Public vs Private IP
8. MAC Address
9. Subnet Mask
10. CIDR Notation
11. Default Gateway
12. DNS
13. DHCP
14. Ports
15. Protocols
16. HTTP vs HTTPS
17. TCP vs UDP
18. SSH
19. Firewall
20. NAT
21. Routing

---

# 1. Introduction to Networking

## What is Networking?

A **computer network** is a collection of two or more devices connected together to communicate and share resources such as files, printers, internet access, and applications.

A network allows computers to exchange information using a common set of communication rules called **protocols**.

---

## Why is Networking Important?

Networking is one of the most important skills for a DevOps Engineer because almost every modern application communicates over a network.

Examples include:

- Accessing websites
- Connecting to cloud servers
- Communicating between Docker containers
- Kubernetes cluster communication
- CI/CD pipeline deployments
- Database connections
- API communication

Without networking, applications cannot communicate with each other.

---

## Real World Example

Suppose you open your browser and visit:

```
https://github.com
```

Several networking components work together behind the scenes:

```
Your Laptop
      │
      ▼
Wi-Fi Router
      │
      ▼
Internet
      │
      ▼
DNS Server
      │
      ▼
GitHub Server
      │
      ▼
Response Returned
```

This entire communication happens within a few milliseconds.

---

## Components of a Network

A computer network consists of several components:

- Client
- Server
- Router
- Switch
- Hub
- Modem
- Firewall
- Network Interface Card (NIC)
- Transmission Media (Ethernet Cable, Fiber Optics, Wi-Fi)

---

## Client

A client is a device that requests a service.

Examples:

- Laptop
- Mobile Phone
- Desktop
- Tablet

Example:

Your browser requesting a web page from GitHub.

---

## Server

A server is a computer that provides services or resources to clients.

Examples:

- Web Server
- Database Server
- Mail Server
- DNS Server
- File Server

Example:

GitHub's servers send the requested web pages to your browser.

---

## Communication Process

A basic network communication follows these steps:

```
Client

↓

Request

↓

Network

↓

Server

↓

Response

↓

Client
```

Example:

```
Browser

↓

HTTP Request

↓

Google Server

↓

HTML Response

↓

Browser Displays Website
```

---

## Network Communication Rules

Devices communicate using predefined rules known as **Protocols**.

Examples of protocols include:

- HTTP
- HTTPS
- SSH
- FTP
- DNS
- TCP
- UDP

These protocols define how data is transmitted, received, and interpreted.

---

## Key Takeaways

- A network connects multiple devices.
- Devices communicate using protocols.
- Clients request services from servers.
- Networking enables internet communication.
- Networking is a fundamental skill for Docker, Kubernetes, AWS, and DevOps.