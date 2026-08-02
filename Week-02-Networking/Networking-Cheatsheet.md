# 📄 Networking Cheat Sheet

A quick revision guide for important networking concepts, protocols, ports, and Linux networking commands.

---

# 🌐 OSI Model

| Layer | Name | Examples |
|--------|------|----------|
| 7 | Application | HTTP, HTTPS, FTP, DNS |
| 6 | Presentation | SSL/TLS, Encryption |
| 5 | Session | NetBIOS, RPC |
| 4 | Transport | TCP, UDP |
| 3 | Network | IP, ICMP |
| 2 | Data Link | Ethernet, MAC |
| 1 | Physical | Cable, Fiber, Wi-Fi |

---

# 🌐 TCP/IP Model

| Layer | Protocols |
|--------|-----------|
| Application | HTTP, HTTPS, FTP, SSH, DNS |
| Transport | TCP, UDP |
| Internet | IP, ICMP |
| Network Access | Ethernet, Wi-Fi |

---

# 🌍 Private IP Address Ranges

| Class | Range |
|--------|-------|
| Class A | 10.0.0.0 – 10.255.255.255 |
| Class B | 172.16.0.0 – 172.31.255.255 |
| Class C | 192.168.0.0 – 192.168.255.255 |

---

# 📡 Common Ports

| Port | Service |
|------|---------|
| 20/21 | FTP |
| 22 | SSH |
| 23 | Telnet |
| 25 | SMTP |
| 53 | DNS |
| 67/68 | DHCP |
| 80 | HTTP |
| 110 | POP3 |
| 143 | IMAP |
| 443 | HTTPS |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 8080 | HTTP Alternate |
| 27017 | MongoDB |

---

# 🔐 Common Protocols

| Protocol | Purpose |
|-----------|---------|
| HTTP | Web Communication |
| HTTPS | Secure Web Communication |
| SSH | Remote Login |
| FTP | File Transfer |
| DNS | Domain Name Resolution |
| DHCP | Automatic IP Assignment |
| TCP | Reliable Communication |
| UDP | Fast Communication |
| SMTP | Send Emails |
| IMAP | Receive Emails |
| ICMP | Network Diagnostics |

---

# ⚖️ TCP vs UDP

| TCP | UDP |
|-----|-----|
| Reliable | Faster |
| Connection-Oriented | Connectionless |
| Error Checking | No Error Checking |
| Ordered Delivery | No Delivery Guarantee |
| Used for HTTP, HTTPS, SSH | Used for DNS, Streaming, Gaming |

---

# 🌍 HTTP vs HTTPS

| HTTP | HTTPS |
|------|-------|
| Port 80 | Port 443 |
| Unencrypted | Encrypted |
| Less Secure | More Secure |
| No SSL/TLS | Uses SSL/TLS |

---

# 🧠 Important Networking Terms

- IP Address
- MAC Address
- DNS
- DHCP
- NAT
- Gateway
- Router
- Switch
- Hub
- Firewall
- CIDR
- Subnet Mask
- Packet
- Port
- Protocol

---

# 💻 Essential Linux Networking Commands

| Command | Purpose |
|----------|---------|
| `ip addr` | Show IP Address |
| `ip route` | Show Routing Table |
| `ip link` | Show Network Interfaces |
| `hostname` | Display Hostname |
| `hostname -I` | Show IP Address |
| `ping` | Test Connectivity |
| `curl` | Test HTTP Requests |
| `wget` | Download Files |
| `nslookup` | DNS Lookup |
| `dig` | Advanced DNS Lookup |
| `traceroute` | Trace Network Path |
| `ss -tulnp` | Show Listening Ports |
| `netstat -tulnp` | Display Active Connections |
| `lsof -i` | Show Open Ports |
| `arp -a` | Display ARP Table |
| `tcpdump` | Capture Network Packets |
| `ifconfig` | Show Network Configuration (Legacy) |

---

# 🔥 UFW Firewall Commands

```bash
sudo ufw status

sudo ufw enable

sudo ufw disable

sudo ufw allow 22

sudo ufw allow 80

sudo ufw allow 443
```

---

# 🛠 Frequently Used DevOps Commands

```bash
ip addr

ip route

hostname -I

ping google.com

curl https://google.com

ssh ubuntu@server-ip

ss -tulnp

netstat -tulnp

nslookup google.com

dig google.com

traceroute google.com

tcpdump -i eth0
```

---

# 📌 Remember These

### Private IP Ranges

```
10.0.0.0/8

172.16.0.0/12

192.168.0.0/16
```

---

### Secure Ports

```
22  → SSH

443 → HTTPS
```

---

### Common Database Ports

```
3306  → MySQL

5432  → PostgreSQL

27017 → MongoDB

6379  → Redis
```

---

### Common Web Ports

```
80  → HTTP

443 → HTTPS

8080 → HTTP Alternate
```

---

# 🎯 DevOps Tip

A DevOps Engineer should be comfortable with:

- Linux Networking Commands
- DNS Troubleshooting
- SSH Connectivity
- HTTP/HTTPS Requests
- Docker Networking
- Kubernetes Networking
- Firewall Configuration
- Routing Basics
- Port Management
- Cloud Networking (AWS, Azure, GCP)
