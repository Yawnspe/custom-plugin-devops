# Skill 01: TCP/IP & Networking Fundamentals

> Master the foundational networking concepts that power the internet and all modern infrastructure

## 📚 Overview

TCP/IP networking is the **foundation of all internet communication** and modern infrastructure. Understanding how data flows through networks, how devices communicate, and how protocols work is absolutely essential for any DevOps engineer. Without this knowledge, troubleshooting network issues becomes guesswork.

### What You'll Learn
- OSI and TCP/IP networking models
- IP addressing and subnetting
- TCP vs UDP protocols
- Routing and network layers
- Network interfaces and configuration
- Common networking commands
- Network architecture basics

### Why It Matters
- **Troubleshooting:** Debug connectivity and performance issues
- **Architecture:** Design scalable network topologies
- **Security:** Understand attack vectors and protections
- **Performance:** Optimize data flow and reduce latency
- **Cloud:** Work effectively with VPCs, subnets, and load balancers

### Prerequisites
- Linux command line basics
- Basic understanding of computers and internet

---

## 🏗️ Foundational Concepts

### The OSI Model

The **OSI (Open Systems Interconnection)** model is a conceptual framework with 7 layers:

```
┌─────────────────────────────────────────┐
│  7. Application  │ HTTP, FTP, SSH, DNS  │
├──────────────────┼──────────────────────┤
│  6. Presentation │ SSL/TLS, Encryption  │
├──────────────────┼──────────────────────┤
│  5. Session      │ Session Management   │
├──────────────────┼──────────────────────┤
│  4. Transport    │ TCP, UDP            │
├──────────────────┼──────────────────────┤
│  3. Network      │ IP, ICMP, Routing   │
├──────────────────┼──────────────────────┤
│  2. Data Link    │ Ethernet, MAC, ARP  │
├──────────────────┼──────────────────────┤
│  1. Physical     │ Cables, Signals     │
└──────────────────┴──────────────────────┘
```

**Key Points:**
- Each layer serves the layer above it
- Each layer uses services from the layer below
- Encapsulation: data is wrapped at each layer
- Decapsulation: data is unwrapped at each layer

### The TCP/IP Model

The **TCP/IP model** is a simpler, practical 4-layer model:

```
┌────────────────────────────────────────┐
│  4. Application   │ HTTP, SSH, DNS     │
├───────────────────┼────────────────────┤
│  3. Transport     │ TCP, UDP           │
├───────────────────┼────────────────────┤
│  2. Internet      │ IP, ICMP, ARP      │
├───────────────────┼────────────────────┤
│  1. Link          │ Ethernet, WiFi     │
└───────────────────┴────────────────────┘
```

**Comparison:**
- OSI: Theoretical, educational model (7 layers)
- TCP/IP: Practical implementation (4 layers)
- Most professionals reference both

### IP Addressing

**IPv4 Address:** 32-bit address (e.g., 192.168.1.100)

```
Structure: 4 octets (bytes) separated by dots
Binary:    11000000.10101000.00000001.01100100
Decimal:   192.168.1.100

Total addresses: 2^32 = 4,294,967,296 (~4.3 billion)
```

**Address Classes (Historical):**

| Class | Range | Default Mask | Use Case |
|-------|-------|--------------|----------|
| A | 1.0.0.0 - 126.255.255.255 | /8 (255.0.0.0) | Large networks |
| B | 128.0.0.0 - 191.255.255.255 | /16 (255.255.0.0) | Medium networks |
| C | 192.0.0.0 - 223.255.255.255 | /24 (255.255.255.0) | Small networks |
| D | 224.0.0.0 - 239.255.255.255 | N/A | Multicast |
| E | 240.0.0.0 - 255.255.255.255 | N/A | Reserved |

**Private IP Ranges (RFC 1918):**
```
10.0.0.0/8        (10.0.0.0 - 10.255.255.255)     - 16 million IPs
172.16.0.0/12     (172.16.0.0 - 172.31.255.255)   - 1 million IPs
192.168.0.0/16    (192.168.0.0 - 192.168.255.255) - 65,536 IPs
```

**Special Addresses:**
- `127.0.0.1` - Localhost (loopback)
- `0.0.0.0` - Default route / all interfaces
- `255.255.255.255` - Broadcast address

### Subnetting

**Subnet Mask:** Divides IP address into network and host portions

```
IP Address:    192.168.1.100
Subnet Mask:   255.255.255.0  (/24 in CIDR notation)

Network:       192.168.1.0    (first address)
Broadcast:     192.168.1.255  (last address)
Usable IPs:    192.168.1.1 - 192.168.1.254 (254 hosts)
```

**CIDR Notation:**
```
/24 = 255.255.255.0   = 256 addresses (254 usable)
/25 = 255.255.255.128 = 128 addresses (126 usable)
/26 = 255.255.255.192 = 64 addresses (62 usable)
/27 = 255.255.255.224 = 32 addresses (30 usable)
/28 = 255.255.255.240 = 16 addresses (14 usable)
/30 = 255.255.255.252 = 4 addresses (2 usable) - point-to-point links
```

**Subnetting Example:**
```
Network: 192.168.1.0/24

Split into 4 subnets (/26):
  Subnet 1: 192.168.1.0/26   (192.168.1.0 - 192.168.1.63)
  Subnet 2: 192.168.1.64/26  (192.168.1.64 - 192.168.1.127)
  Subnet 3: 192.168.1.128/26 (192.168.1.128 - 192.168.1.191)
  Subnet 4: 192.168.1.192/26 (192.168.1.192 - 192.168.1.255)
```

### TCP vs UDP

**TCP (Transmission Control Protocol):**
- **Connection-oriented:** Three-way handshake
- **Reliable:** Guaranteed delivery, retransmission
- **Ordered:** Packets arrive in order
- **Flow control:** Prevents overwhelming receiver
- **Use cases:** HTTP, SSH, FTP, email

```
Three-Way Handshake:
Client          Server
  |   SYN  -->    |
  |  <-- SYN-ACK  |
  |   ACK  -->    |
  [Connection Established]
```

**UDP (User Datagram Protocol):**
- **Connectionless:** No handshake
- **Unreliable:** No delivery guarantee
- **Unordered:** Packets may arrive out of order
- **Fast:** Low overhead
- **Use cases:** DNS, DHCP, streaming, gaming

**Comparison:**

| Feature | TCP | UDP |
|---------|-----|-----|
| Speed | Slower | Faster |
| Reliability | High | Low |
| Overhead | High | Low |
| Order | Guaranteed | Not guaranteed |
| Use case | Accuracy critical | Speed critical |

### Ports and Sockets

**Port Numbers:** Identify applications/services (0-65535)

```
Well-Known Ports (0-1023):
  20/21  - FTP
  22     - SSH
  23     - Telnet
  25     - SMTP (email)
  53     - DNS
  80     - HTTP
  443    - HTTPS
  3306   - MySQL
  5432   - PostgreSQL
  6379   - Redis
  27017  - MongoDB

Registered Ports (1024-49151):
  3000   - Node.js (common dev port)
  8080   - HTTP alternate
  8443   - HTTPS alternate

Dynamic/Private Ports (49152-65535):
  Used for client connections
```

**Socket:** IP address + port number
```
192.168.1.100:80       (web server)
192.168.1.100:22       (SSH server)
10.0.0.50:443          (HTTPS server)
```

---

## 🔧 Practical Implementation

### Network Interface Configuration

#### View Network Interfaces

```bash
# Modern method (ip command)
ip addr show
ip a  # Short form

# Show specific interface
ip addr show eth0

# Old method (deprecated but still common)
ifconfig
ifconfig eth0

# List all interfaces (brief)
ip link show

# Show interface statistics
ip -s link show eth0
```

**Sample Output:**
```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
    link/ether 00:0c:29:3f:47:c2
    inet 192.168.1.100/24 brd 192.168.1.255 scope global eth0
    inet6 fe80::20c:29ff:fe3f:47c2/64 scope link
```

#### Configure Network Interface

```bash
# Bring interface up
sudo ip link set eth0 up

# Bring interface down
sudo ip link set eth0 down

# Assign IP address
sudo ip addr add 192.168.1.100/24 dev eth0

# Remove IP address
sudo ip addr del 192.168.1.100/24 dev eth0

# Set MTU (Maximum Transmission Unit)
sudo ip link set eth0 mtu 1450

# Temporary configuration (lost on reboot)
sudo ip addr add 10.0.0.5/24 dev eth0
sudo ip link set eth0 up
```

#### Persistent Configuration (Ubuntu/Debian)

Edit `/etc/netplan/01-netcfg.yaml`:
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

Apply configuration:
```bash
sudo netplan apply
```

#### Persistent Configuration (CentOS/RHEL)

Edit `/etc/sysconfig/network-scripts/ifcfg-eth0`:
```
TYPE=Ethernet
BOOTPROTO=static
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=8.8.4.4
```

Restart networking:
```bash
sudo systemctl restart NetworkManager
```

### Routing

#### View Routing Table

```bash
# Show routing table
ip route show
route -n  # Old method

# Show default gateway
ip route show default

# Show route to specific IP
ip route get 8.8.8.8
```

**Sample Output:**
```
default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.100
```

#### Manage Routes

```bash
# Add default gateway
sudo ip route add default via 192.168.1.1

# Add static route
sudo ip route add 10.0.0.0/24 via 192.168.1.254

# Delete route
sudo ip route del 10.0.0.0/24

# Add route to specific network via gateway
sudo ip route add 172.16.0.0/16 via 192.168.1.250 dev eth0
```

### Network Testing Commands

#### ping - Test Connectivity

```bash
# Basic ping
ping google.com
ping 8.8.8.8

# Specific count
ping -c 4 google.com

# Specific interval (0.2 seconds)
ping -i 0.2 google.com

# Specific packet size (1000 bytes)
ping -s 1000 google.com

# Set timeout (5 seconds)
ping -W 5 google.com

# Flood ping (requires root, use carefully)
sudo ping -f google.com

# IPv6 ping
ping6 google.com
```

#### traceroute - Trace Network Path

```bash
# Trace route to destination
traceroute google.com

# Use ICMP instead of UDP
traceroute -I google.com

# Max hops
traceroute -m 20 google.com

# Don't resolve hostnames (faster)
traceroute -n google.com

# Specify interface
traceroute -i eth0 google.com
```

#### netstat - Network Statistics

```bash
# All listening ports
netstat -tuln

# All connections
netstat -tun

# With process information (requires root)
sudo netstat -tulnp

# Routing table
netstat -r

# Interface statistics
netstat -i

# TCP connections
netstat -t

# UDP connections
netstat -u
```

**Flags:**
- `-t` - TCP
- `-u` - UDP
- `-l` - Listening
- `-n` - Numeric (don't resolve names)
- `-p` - Process information

#### ss - Socket Statistics (modern netstat replacement)

```bash
# All TCP sockets
ss -ta

# All listening TCP sockets
ss -tl

# All UDP sockets
ss -ua

# With process information
sudo ss -tlnp

# Show summary statistics
ss -s

# Filter by port
ss -tan 'sport = :80'

# Filter established connections
ss -o state established

# Show memory usage
ss -tm
```

### ARP (Address Resolution Protocol)

```bash
# Show ARP cache
ip neigh show
arp -n  # Old method

# Add static ARP entry
sudo ip neigh add 192.168.1.50 lladdr 00:11:22:33:44:55 dev eth0

# Delete ARP entry
sudo ip neigh del 192.168.1.50 dev eth0

# Flush ARP cache
sudo ip neigh flush all
```

---

## ✨ Best Practices

### 1. **IP Addressing Strategy**

```bash
# Good: Organized subnetting
10.0.0.0/16    - Production VPC
  10.0.1.0/24  - Web tier
  10.0.2.0/24  - App tier
  10.0.3.0/24  - Database tier
  10.0.10.0/24 - Management/bastion

# Document your IP allocation
# Use IPAM (IP Address Management) tools
```

### 2. **Network Segmentation**

```bash
# Separate environments
10.1.0.0/16  - Development
10.2.0.0/16  - Staging
10.3.0.0/16  - Production

# Separate tiers within environment
10.3.1.0/24  - Public subnet (DMZ)
10.3.2.0/24  - Application subnet
10.3.3.0/24  - Database subnet (private)
```

### 3. **Use Static IPs for Infrastructure**

```yaml
# Critical infrastructure should have static IPs
DNS Servers:     10.0.0.10, 10.0.0.11
Load Balancers:  10.0.1.10, 10.0.1.11
Databases:       10.0.3.10, 10.0.3.11
Monitoring:      10.0.4.10
```

### 4. **Monitor Network Health**

```bash
#!/bin/bash
# Basic network health check script

check_interface() {
    if ip link show eth0 | grep -q "UP"; then
        echo "✓ eth0 is UP"
    else
        echo "✗ eth0 is DOWN"
        return 1
    fi
}

check_gateway() {
    gateway=$(ip route show default | awk '{print $3}')
    if ping -c 1 -W 2 $gateway &>/dev/null; then
        echo "✓ Gateway $gateway is reachable"
    else
        echo "✗ Gateway $gateway is unreachable"
        return 1
    fi
}

check_dns() {
    if ping -c 1 -W 2 google.com &>/dev/null; then
        echo "✓ DNS resolution working"
    else
        echo "✗ DNS resolution failed"
        return 1
    fi
}

check_interface
check_gateway
check_dns
```

---

## ⚠️ Common Pitfalls

### 1. **Subnet Mask Misconfiguration**

```bash
# ❌ Bad: Wrong subnet mask
IP: 192.168.1.100
Mask: 255.255.0.0  # Too broad, wrong network

# ✅ Good: Correct subnet mask
IP: 192.168.1.100
Mask: 255.255.255.0  # Matches network design
```

### 2. **Incorrect Default Gateway**

```bash
# ❌ Bad: Gateway not in same subnet
IP:      192.168.1.100/24
Gateway: 192.168.2.1  # Different subnet!

# ✅ Good: Gateway in same subnet
IP:      192.168.1.100/24
Gateway: 192.168.1.1  # Same subnet
```

### 3. **IP Conflicts**

```bash
# Always check for conflicts before assigning static IP
ping -c 2 192.168.1.100

# If ping succeeds, IP is already in use!
# Choose different IP
```

### 4. **Not Testing After Changes**

```bash
# After any network change, test connectivity
ping -c 4 8.8.8.8           # Internet connectivity
ping -c 4 192.168.1.1       # Gateway
ip addr show                 # Verify IP assignment
ip route show                # Verify routes
```

---

## 📝 Hands-On Exercises

### Exercise 1: IP Addressing & Subnetting (Beginner)

**Goal:** Practice IP addressing calculations

```bash
# Calculate the following:
# 1. Given network 192.168.10.0/24
#    - Network address: ?
#    - Broadcast address: ?
#    - First usable IP: ?
#    - Last usable IP: ?
#    - Total usable IPs: ?

# Answers:
# Network: 192.168.10.0
# Broadcast: 192.168.10.255
# First usable: 192.168.10.1
# Last usable: 192.168.10.254
# Usable IPs: 254

# 2. Subnet 172.16.0.0/16 into 4 equal subnets
# What subnet mask do you need? (/18)
# List all 4 subnet ranges
```

### Exercise 2: Network Configuration (Intermediate)

**Goal:** Configure network interface and routing

```bash
# 1. Check current network configuration
ip addr show
ip route show

# 2. Add a secondary IP address
sudo ip addr add 192.168.1.200/24 dev eth0

# 3. Verify the change
ip addr show eth0

# 4. Add a route to 10.0.0.0/24 via 192.168.1.254
sudo ip route add 10.0.0.0/24 via 192.168.1.254

# 5. Test the route
ip route get 10.0.0.50

# 6. Clean up (remove configurations)
sudo ip addr del 192.168.1.200/24 dev eth0
sudo ip route del 10.0.0.0/24
```

### Exercise 3: Network Diagnostics (Intermediate)

**Goal:** Diagnose network connectivity issues

```bash
#!/bin/bash
# Network diagnostic script

echo "=== Network Diagnostic Report ==="
echo ""

echo "1. Network Interfaces:"
ip -br addr show
echo ""

echo "2. Default Gateway:"
ip route show default
echo ""

echo "3. Gateway Reachability:"
gateway=$(ip route show default | awk '{print $3}')
if ping -c 2 $gateway &>/dev/null; then
    echo "✓ Gateway $gateway is reachable"
else
    echo "✗ Gateway $gateway is unreachable"
fi
echo ""

echo "4. DNS Resolution:"
if nslookup google.com &>/dev/null; then
    echo "✓ DNS is working"
else
    echo "✗ DNS is not working"
fi
echo ""

echo "5. Internet Connectivity:"
if ping -c 2 8.8.8.8 &>/dev/null; then
    echo "✓ Internet is reachable"
else
    echo "✗ Internet is unreachable"
fi
echo ""

echo "6. Active Connections:"
ss -s
```

---

## 🔗 Further Learning

### Related Skills
- **Skill 02:** DNS & Domain Resolution
- **Skill 06:** Network Troubleshooting & Diagnostics
- **Cloud Agent:** VPC and cloud networking

### Command Reference
- `man ip` - IP configuration manual
- `man ping` - Ping utility
- `man traceroute` - Traceroute manual
- `man ss` - Socket statistics

### External Resources
- TCP/IP Illustrated by W. Richard Stevens
- RFC 791 - Internet Protocol
- RFC 793 - Transmission Control Protocol
- Subnet Calculator: https://www.subnet-calculator.com/
- IPv4 Exhaustion and IPv6: https://www.internetsociety.org/

### Advanced Topics
- IPv6 addressing and configuration
- VLAN (Virtual LAN) configuration
- BGP (Border Gateway Protocol)
- MPLS (Multiprotocol Label Switching)
- Software-Defined Networking (SDN)

---

## 🎓 Skill Completion Checklist

Before moving to the next skill, verify you can:

- [ ] Explain the OSI and TCP/IP models
- [ ] Calculate subnet masks and IP ranges
- [ ] Configure network interfaces with `ip` command
- [ ] Understand the difference between TCP and UDP
- [ ] View and modify routing tables
- [ ] Test network connectivity with ping
- [ ] Use netstat/ss to view connections
- [ ] Understand private vs public IP addresses
- [ ] Configure static IP addresses
- [ ] Troubleshoot basic network connectivity issues

**When you can do all of these, move to Skill 02: DNS & Domain Resolution**

---

**Skill Status:** ✅ Ready to Use
**Last Updated:** November 2024
**Difficulty:** Beginner
**Time to Complete:** 7 hours
