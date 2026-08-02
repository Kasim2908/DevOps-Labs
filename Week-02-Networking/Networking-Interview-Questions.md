# ❓ Networking Interview Questions & Answers

This document contains some of the most commonly asked networking interview questions for DevOps, Cloud, Linux, and System Administration roles.

---

# 1. What is a Computer Network?

### Answer

A computer network is a group of interconnected devices that communicate and share resources such as files, printers, internet connections, and applications using communication protocols.

---

# 2. What is an IP Address?

### Answer

An IP (Internet Protocol) Address is a unique identifier assigned to every device connected to a network.

Example:

```
192.168.1.10
```

It helps devices identify and communicate with each other.

---

# 3. What is the difference between Public IP and Private IP?

### Answer

| Public IP | Private IP |
|-----------|------------|
| Accessible over the Internet | Used inside local networks |
| Assigned by ISP | Assigned by Router |
| Globally Unique | Can be reused |

Private IP ranges:

```
10.0.0.0/8

172.16.0.0 – 172.31.255.255

192.168.0.0/16
```

---

# 4. What is DNS?

### Answer

DNS (Domain Name System) translates human-readable domain names into IP addresses.

Example:

```
google.com

↓

142.250.xxx.xxx
```

Without DNS, users would have to remember IP addresses instead of domain names.

---

# 5. What is DHCP?

### Answer

DHCP (Dynamic Host Configuration Protocol) automatically assigns IP addresses and other network settings to devices joining a network.

Benefits:

- Automatic IP assignment
- Reduces manual configuration
- Prevents IP conflicts

---

# 6. What is the difference between TCP and UDP?

### Answer

| TCP | UDP |
|-----|-----|
| Connection-Oriented | Connectionless |
| Reliable | Faster |
| Error Checking | No Error Recovery |
| Ordered Delivery | No Order Guarantee |

Examples:

TCP

- HTTP
- HTTPS
- SSH

UDP

- DNS
- Video Streaming
- Online Gaming

---

# 7. What is the OSI Model?

### Answer

The OSI (Open Systems Interconnection) Model is a seven-layer networking model that explains how data moves between devices.

Layers:

1. Physical
2. Data Link
3. Network
4. Transport
5. Session
6. Presentation
7. Application

---

# 8. What is the TCP/IP Model?

### Answer

The TCP/IP Model consists of four layers:

1. Network Access
2. Internet
3. Transport
4. Application

It is the practical networking model used on the Internet.

---

# 9. What is a Port?

### Answer

A port is a logical communication endpoint used by applications.

Examples:

- 22 → SSH
- 80 → HTTP
- 443 → HTTPS
- 53 → DNS
- 3306 → MySQL
- 27017 → MongoDB

---

# 10. What is a MAC Address?

### Answer

A MAC (Media Access Control) Address is the unique physical address assigned to a network interface card (NIC).

Example:

```
00:1A:2B:3C:4D:5E
```

Unlike IP addresses, MAC addresses are hardware-based.

---

# 11. What is a Subnet Mask?

### Answer

A subnet mask separates the network portion from the host portion of an IP address.

Example:

```
IP Address

192.168.1.10

Subnet Mask

255.255.255.0
```

It helps determine whether devices are on the same network.

---

# 12. What is CIDR?

### Answer

CIDR (Classless Inter-Domain Routing) is a notation used to represent IP addresses and subnet masks efficiently.

Example:

```
192.168.1.0/24
```

Here:

- 24 bits represent the network.
- 8 bits represent the host.

---

# 13. What is a Gateway?

### Answer

A Gateway is the device (usually a router) that connects one network to another.

Example:

```
Laptop

↓

Router (Gateway)

↓

Internet
```

Without a gateway, devices cannot communicate outside their local network.

---

# 14. What is NAT?

### Answer

NAT (Network Address Translation) converts private IP addresses into public IP addresses, allowing multiple devices to share a single public IP.

Benefits:

- Conserves public IP addresses
- Improves security
- Enables internet access for private networks

---

# 15. What is HTTP and HTTPS?

### Answer

HTTP (HyperText Transfer Protocol) is used for transferring web pages.

HTTPS is the secure version of HTTP using SSL/TLS encryption.

| HTTP | HTTPS |
|------|-------|
| Port 80 | Port 443 |
| Not Encrypted | Encrypted |
| Less Secure | More Secure |

---

# 16. What is SSH?

### Answer

SSH (Secure Shell) is a secure protocol used to remotely access Linux servers.

Example:

```bash
ssh ubuntu@192.168.1.10
```

SSH encrypts all communication between the client and server.

---

# 17. What happens when you type "google.com" in your browser?

### Answer

1. Browser checks local cache.
2. DNS resolves the domain to an IP address.
3. Browser establishes a TCP connection.
4. HTTPS handshake takes place.
5. HTTP request is sent.
6. Google server processes the request.
7. Response is returned.
8. Browser renders the webpage.

---

# 18. What is the difference between a Hub, Switch, and Router?

### Answer

| Hub | Switch | Router |
|-----|--------|--------|
| Broadcasts to all devices | Sends data to specific device | Connects different networks |
| Layer 1 | Layer 2 | Layer 3 |
| Slow | Faster | Intelligent Routing |

---

# 19. What is a Firewall?

### Answer

A Firewall is a security system that monitors and controls incoming and outgoing network traffic based on predefined security rules.

Examples:

- UFW
- iptables
- AWS Security Groups

---

# 20. Why is Networking Important for a DevOps Engineer?

### Answer

Networking is essential because DevOps Engineers work with:

- Linux Servers
- Docker Containers
- Kubernetes Clusters
- Cloud Platforms (AWS, Azure, GCP)
- Load Balancers
- Reverse Proxies
- CI/CD Pipelines
- Monitoring Tools

Understanding networking helps troubleshoot connectivity issues, configure infrastructure, secure applications, and ensure reliable communication between services.

---

# 🎯 Interview Tip

For networking interviews:

- Understand concepts instead of memorizing definitions.
- Explain answers using real-world examples.
- Be familiar with common ports and protocols.
- Practice Linux networking commands.
- Relate networking concepts to Docker, Kubernetes, and Cloud environments.
