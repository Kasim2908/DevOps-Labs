# 💻 DevOps Networking Commands

This document contains commonly used Linux networking commands that every DevOps Engineer should know.

---

# 1. Check IP Address

```bash
ip addr
```

**Purpose:** Displays all network interfaces and their IP addresses.

---

# 2. Show Routing Table

```bash
ip route
```

**Purpose:** Displays the routing table of the system.

---

# 3. Display Network Interfaces

```bash
ip link
```

**Purpose:** Lists all available network interfaces.

---

# 4. Display Hostname

```bash
hostname
```

**Purpose:** Shows the system hostname.

---

# 5. Show System IP

```bash
hostname -I
```

**Purpose:** Displays the assigned IP address.

---

# 6. Test Connectivity

```bash
ping google.com
```

**Purpose:** Checks whether a remote host is reachable.

---

# 7. Trace Route

```bash
traceroute google.com
```

**Purpose:** Shows the path packets take to reach a destination.

---

# 8. DNS Lookup

```bash
nslookup google.com
```

**Purpose:** Resolves a domain name into an IP address.

---

# 9. Download Web Content

```bash
curl https://google.com
```

**Purpose:** Sends HTTP requests to web servers.

---

# 10. Display Listening Ports

```bash
ss -tulnp
```

**Purpose:** Shows all listening TCP/UDP ports.

---

# 11. Display Active Connections

```bash
netstat -tulnp
```

**Purpose:** Lists active network connections.

---

# 12. Show Open Files & Ports

```bash
lsof -i
```

**Purpose:** Displays processes using network ports.

---

# 13. Check ARP Table

```bash
arp -a
```

**Purpose:** Displays IP-to-MAC address mappings.

---

# 14. Show Network Configuration

```bash
ifconfig
```

**Purpose:** Displays network interface configuration (legacy command).

---

# 15. Packet Capture

```bash
tcpdump -i eth0
```

**Purpose:** Captures network packets for troubleshooting.

---

# 16. Check Firewall Status

```bash
sudo ufw status
```

**Purpose:** Displays UFW firewall status.

---

# 17. Allow SSH Port

```bash
sudo ufw allow 22
```

**Purpose:** Allows SSH traffic through the firewall.

---

# 18. Check Internet Connectivity

```bash
ping 8.8.8.8
```

**Purpose:** Tests internet connectivity without relying on DNS.

---

# 19. Test SSH Connection

```bash
ssh user@server-ip
```

**Purpose:** Connects securely to a remote Linux server.

---

# 20. Check Network Statistics

```bash
ip -s link
```

**Purpose:** Displays network interface statistics.

---

# 🌐 Official References

- Linux Networking: https://www.kernel.org/doc/html/latest/networking/index.html
- ip Command: https://man7.org/linux/man-pages/man8/ip.8.html
- curl Documentation: https://curl.se/docs/
- OpenSSH: https://www.openssh.com/manual.html
- tcpdump: https://www.tcpdump.org/manpages/

---

# 🚀 Most Used Commands for DevOps

```bash
ip addr
ip route
ping
curl
ssh
ss -tulnp
netstat -tulnp
hostname -I
nslookup
traceroute
tcpdump
lsof -i
ufw status
```
