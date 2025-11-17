# Skill 05: Firewall & Network Security

> Master firewall configuration and network security to protect your infrastructure

## 📚 Overview

Firewalls are your **first line of defense** against network attacks. Understanding how to configure and manage firewalls is critical for securing servers, applications, and infrastructure. This skill covers iptables, ufw, firewalld, and network security principles.

### What You'll Learn
- Firewall concepts and types
- iptables rules and chains
- ufw (Uncomplicated Firewall)
- firewalld zones and services
- Port security and filtering
- NAT and port forwarding
- Security best practices
- Monitoring and logging

### Why It Matters
- **Security:** Protect against unauthorized access
- **Compliance:** Meet security requirements
- **Control:** Manage network traffic flow
- **Audit:** Track and log access attempts
- **Defense:** Prevent attacks and intrusions

### Prerequisites
- TCP/IP fundamentals (Skill 01)
- SSH basics (Skill 04)
- Linux system administration

---

## 🏗️ Foundational Concepts

### What is a Firewall?

A **firewall** filters network traffic based on rules:
- **Allow:** Permitted traffic passes through
- **Deny/Drop:** Blocked traffic is rejected or silently dropped
- **Log:** Traffic is logged for audit

```
Internet → Firewall → Server
            ↓
         [Rules]
         Allow: 80, 443
         Deny: All others
```

### Firewall Types

**Packet Filtering:**
- Examines packet headers (IP, port, protocol)
- Fast but basic
- Examples: iptables, pf

**Stateful Inspection:**
- Tracks connection state
- Understands TCP sessions
- More intelligent filtering

**Application Layer:**
- Deep packet inspection
- Application-aware
- Examples: WAF (Web Application Firewall)

### Linux Firewall Tools

```
iptables     - Low-level, powerful, complex
ufw          - Simple, user-friendly (Ubuntu)
firewalld    - Dynamic, zone-based (RHEL/CentOS)
nftables     - Modern replacement for iptables
```

### iptables Chains

```
INPUT    - Incoming traffic to local system
OUTPUT   - Outgoing traffic from local system
FORWARD  - Traffic being routed through system

     ┌──────────────┐
     │   ROUTING    │
     │   DECISION   │
     └──────┬───────┘
            │
      ┌─────┴─────┐
      │           │
   INPUT       OUTPUT
      │           │
   Local      From Local
   Process    Process
      │           │
   FORWARD ───────┘
      │
   To Another
   Interface
```

### Policy Types

```
ACCEPT  - Allow packet
DROP    - Silently discard packet
REJECT  - Discard and send error message
LOG     - Log packet (then continue to next rule)
```

---

## 🔧 Practical Implementation

### iptables Basics

#### View Rules

```bash
# List all rules
sudo iptables -L
sudo iptables -L -v  # Verbose with packet counts
sudo iptables -L -n  # Numeric (don't resolve names)
sudo iptables -L -v -n --line-numbers

# List specific chain
sudo iptables -L INPUT
sudo iptables -L OUTPUT
sudo iptables -L FORWARD

# List NAT rules
sudo iptables -t nat -L -v -n
```

#### Basic Rules

```bash
# Allow SSH (port 22)
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP (port 80)
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Allow HTTPS (port 443)
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow from specific IP
sudo iptables -A INPUT -s 192.168.1.100 -j ACCEPT

# Allow from subnet
sudo iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT

# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT
```

#### Delete Rules

```bash
# Delete specific rule by number
sudo iptables -D INPUT 3

# Delete specific rule by specification
sudo iptables -D INPUT -p tcp --dport 8080 -j ACCEPT

# Flush all rules in chain
sudo iptables -F INPUT

# Flush all rules in all chains
sudo iptables -F
```

#### Set Default Policies

```bash
# View current policies
sudo iptables -L | grep policy

# Set default policy to DROP (strict)
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Set default policy to ACCEPT (permissive)
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
```

#### Complete Firewall Script

```bash
#!/bin/bash
# firewall_setup.sh

# Flush existing rules
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X

# Default policies
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH (from specific subnet only)
iptables -A INPUT -p tcp -s 192.168.1.0/24 --dport 22 -j ACCEPT

# Allow HTTP and HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow ping (ICMP)
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# Log dropped packets
iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables-dropped: " --log-level 7

# Drop everything else
iptables -A INPUT -j DROP

# Save rules
iptables-save > /etc/iptables/rules.v4
```

### ufw (Uncomplicated Firewall)

#### Basic Commands

```bash
# Install ufw
sudo apt install ufw  # Debian/Ubuntu

# Check status
sudo ufw status
sudo ufw status verbose
sudo ufw status numbered

# Enable/disable firewall
sudo ufw enable
sudo ufw disable

# Reset to defaults
sudo ufw reset
```

#### Allow/Deny Rules

```bash
# Allow port
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443

# Allow service by name
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https

# Allow from specific IP
sudo ufw allow from 192.168.1.100

# Allow from subnet
sudo ufw allow from 192.168.1.0/24

# Allow from IP to specific port
sudo ufw allow from 192.168.1.100 to any port 22

# Allow specific protocol
sudo ufw allow 53/udp
sudo ufw allow 53/tcp

# Deny port
sudo ufw deny 23  # Deny telnet

# Allow port range
sudo ufw allow 6000:6007/tcp
```

#### Delete Rules

```bash
# Delete by rule number
sudo ufw status numbered
sudo ufw delete 2

# Delete by specification
sudo ufw delete allow 80
sudo ufw delete allow from 192.168.1.100
```

#### Common Configurations

```bash
# Web server
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS

# Database server (MySQL)
sudo ufw allow from 192.168.1.0/24 to any port 3306

# Allow SSH from specific IP only
sudo ufw default deny incoming
sudo ufw allow from 203.0.113.0/24 to any port 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

### firewalld (CentOS/RHEL)

#### Basic Commands

```bash
# Install firewalld
sudo yum install firewalld  # CentOS/RHEL

# Start and enable
sudo systemctl start firewalld
sudo systemctl enable firewalld

# Check status
sudo firewall-cmd --state
sudo firewall-cmd --list-all

# List zones
sudo firewall-cmd --get-zones
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --get-default-zone
```

#### Zone Management

```bash
# Set default zone
sudo firewall-cmd --set-default-zone=public

# Add interface to zone
sudo firewall-cmd --zone=public --add-interface=eth0 --permanent

# List zone details
sudo firewall-cmd --zone=public --list-all
```

#### Service and Port Management

```bash
# Add service
sudo firewall-cmd --zone=public --add-service=http
sudo firewall-cmd --zone=public --add-service=https
sudo firewall-cmd --zone=public --add-service=ssh

# Make permanent
sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --reload

# Add port
sudo firewall-cmd --zone=public --add-port=8080/tcp
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent

# Remove service/port
sudo firewall-cmd --zone=public --remove-service=http
sudo firewall-cmd --zone=public --remove-port=8080/tcp

# List services and ports
sudo firewall-cmd --zone=public --list-services
sudo firewall-cmd --zone=public --list-ports
```

#### Rich Rules

```bash
# Allow SSH from specific subnet
sudo firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name="ssh" accept'

# Block specific IP
sudo firewall-cmd --add-rich-rule='rule family="ipv4" source address="10.0.0.50" reject'

# Rate limit SSH
sudo firewall-cmd --add-rich-rule='rule service name="ssh" limit value="3/m" accept'
```

### NAT and Port Forwarding

#### iptables NAT

```bash
# Enable IP forwarding
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
# Make permanent:
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# SNAT (Source NAT) - Masquerade
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Port forwarding (DNAT)
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 80

# Forward external port to internal server
sudo iptables -t nat -A PREROUTING -p tcp -d 203.0.113.1 --dport 80 -j DNAT --to-destination 192.168.1.10:80
```

#### ufw NAT

```bash
# Edit /etc/ufw/before.rules
sudo vim /etc/ufw/before.rules

# Add before *filter:
*nat
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -s 192.168.1.0/24 -o eth0 -j MASQUERADE
COMMIT

# Enable forwarding
sudo vim /etc/default/ufw
# Set: DEFAULT_FORWARD_POLICY="ACCEPT"

# Apply changes
sudo ufw disable && sudo ufw enable
```

---

## ✨ Best Practices

### 1. **Default Deny Policy**

```bash
# Start with deny all, then whitelist
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Then explicitly allow needed services
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

### 2. **Limit SSH Access**

```bash
# Allow SSH only from management network
sudo ufw allow from 192.168.1.0/24 to any port 22

# Rate limit SSH attempts
sudo ufw limit ssh

# Or with iptables:
sudo iptables -A INPUT -p tcp --dport 22 -m recent --name ssh --set
sudo iptables -A INPUT -p tcp --dport 22 -m recent --name ssh --update --seconds 60 --hitcount 4 -j DROP
```

### 3. **Log Suspicious Activity**

```bash
# Log dropped packets
sudo iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables-DROP: "

# With ufw
sudo ufw logging on
sudo ufw logging medium  # or low, high, full
```

### 4. **Protect Against Common Attacks**

```bash
# Block invalid packets
sudo iptables -A INPUT -m state --state INVALID -j DROP

# Block fragmented packets
sudo iptables -A INPUT -f -j DROP

# Block XMAS packets
sudo iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP

# Block NULL packets
sudo iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP

# SYN flood protection
sudo iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j ACCEPT
sudo iptables -A INPUT -p tcp --syn -j DROP
```

### 5. **Backup and Restore Rules**

```bash
# Save iptables rules
sudo iptables-save > /tmp/iptables-backup.rules

# Restore iptables rules
sudo iptables-restore < /tmp/iptables-backup.rules

# Make rules persistent (Ubuntu)
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

---

## ⚠️ Common Pitfalls

### 1. **Locking Yourself Out**

```bash
# ❌ Bad: Enabling firewall without allowing SSH
sudo ufw enable  # Might lock you out!

# ✅ Good: Allow SSH first
sudo ufw allow 22
sudo ufw enable

# Always keep a backup session open when testing
```

### 2. **Forgetting Established Connections**

```bash
# ❌ Bad: Only allowing new connections
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# ✅ Good: Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

### 3. **Not Testing Rules**

```bash
# ✅ Good: Test before making permanent
sudo iptables -A INPUT -p tcp --dport 8080 -j ACCEPT
# Test the service
# If working, then save:
sudo iptables-save > /etc/iptables/rules.v4
```

---

## 📝 Hands-On Exercises

### Exercise 1: Basic Firewall Setup (Beginner)

```bash
# 1. Check current firewall status
sudo ufw status

# 2. Enable ufw with default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# 3. Allow essential services
sudo ufw allow 22/tcp   # SSH
sudo ufw allow 80/tcp   # HTTP
sudo ufw allow 443/tcp  # HTTPS

# 4. Enable firewall
sudo ufw enable

# 5. Verify rules
sudo ufw status verbose
```

### Exercise 2: Advanced iptables Configuration (Intermediate)

```bash
#!/bin/bash
# advanced_firewall.sh

# Flush rules
iptables -F

# Default policies
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Loopback
iptables -A INPUT -i lo -j ACCEPT

# Established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# SSH with rate limiting
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Web services
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# ICMP (ping)
iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/s -j ACCEPT

# Log and drop
iptables -A INPUT -m limit --limit 3/min -j LOG --log-prefix "FW-DROP: "
iptables -A INPUT -j DROP

echo "Firewall configured!"
```

### Exercise 3: Firewall Audit Script (Advanced)

```bash
#!/bin/bash
# firewall_audit.sh

echo "Firewall Security Audit"
echo "======================"
echo ""

# Check if firewall is active
if command -v ufw &> /dev/null; then
    echo "1. UFW Status:"
    sudo ufw status
    echo ""
fi

if command -v firewall-cmd &> /dev/null; then
    echo "1. firewalld Status:"
    sudo firewall-cmd --state
    sudo firewall-cmd --list-all
    echo ""
fi

# Check iptables rules
echo "2. iptables Rules:"
sudo iptables -L -n -v | head -20
echo ""

# Check for common vulnerabilities
echo "3. Security Checks:"

# Check if SSH is accessible from anywhere
if sudo iptables -L INPUT -n | grep -q "dpt:22.*0.0.0.0/0"; then
    echo "⚠  WARNING: SSH accessible from anywhere"
else
    echo "✓ SSH properly restricted"
fi

# Check default policies
policies=$(sudo iptables -L | grep "policy")
echo "Default policies: $policies"
echo ""

# Check open ports
echo "4. Listening Ports:"
sudo netstat -tuln | grep LISTEN | head -10
```

---

## 🔗 Further Learning

### Related Skills
- **Skill 01:** TCP/IP Fundamentals
- **Skill 04:** SSH & Secure Remote Access
- **Cloud Agent:** Security groups and NACLs

### Command Reference
- `man iptables` - iptables manual
- `man ufw` - ufw manual
- `man firewall-cmd` - firewalld manual

### External Resources
- Netfilter Documentation
- iptables Tutorial by Frozentux
- CIS Benchmarks for Linux
- NIST Cybersecurity Framework

### Advanced Topics
- nftables (iptables replacement)
- Application firewalls (ModSecurity, WAF)
- IDS/IPS (Intrusion Detection/Prevention)
- DDoS protection
- Network segmentation
- Zero Trust architecture

---

## 🎓 Skill Completion Checklist

Before moving to the next skill, verify you can:

- [ ] Understand firewall concepts and types
- [ ] Configure iptables rules
- [ ] Use ufw for simple firewall management
- [ ] Configure firewalld zones and services
- [ ] Implement NAT and port forwarding
- [ ] Set up rate limiting
- [ ] Log firewall events
- [ ] Protect against common attacks
- [ ] Backup and restore firewall rules
- [ ] Audit firewall configuration

**When you can do all of these, move to Skill 06: Network Troubleshooting & Diagnostics**

---

**Skill Status:** ✅ Ready to Use
**Last Updated:** November 2024
**Difficulty:** Intermediate
**Time to Complete:** 6 hours
