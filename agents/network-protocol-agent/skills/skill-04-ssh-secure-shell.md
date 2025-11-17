# Skill 04: SSH & Secure Remote Access

> Master SSH - the secure protocol for remote server access, file transfers, and tunneling

## 📚 Overview

SSH (Secure Shell) is the **standard protocol for secure remote access** to servers. As a DevOps engineer, you'll use SSH daily to manage servers, deploy applications, transfer files, and create secure tunnels. Mastering SSH is essential for security and productivity.

### What You'll Learn
- SSH protocol and architecture
- Key-based authentication vs passwords
- SSH configuration and customization
- Port forwarding and tunneling
- Secure file transfers (SCP, SFTP)
- SSH agent and key management
- Security best practices
- Troubleshooting SSH issues

### Why It Matters
- **Security:** Encrypted remote access
- **Automation:** Password-less authentication for scripts
- **Debugging:** Access production servers safely
- **Tunneling:** Secure connections through firewalls
- **DevOps:** Essential for CI/CD and deployments

### Prerequisites
- TCP/IP fundamentals (Skill 01)
- Linux command line basics
- Basic understanding of encryption

---

## 🏗️ Foundational Concepts

### What is SSH?

**SSH (Secure Shell)** provides:
- **Encryption:** All traffic is encrypted
- **Authentication:** Verify identity of server and client
- **Integrity:** Data cannot be tampered with
- **Confidentiality:** Data cannot be read by eavesdroppers

**SSH vs Telnet:**
```
Telnet (Port 23):
  - Unencrypted plaintext
  - Passwords visible on network
  - INSECURE - Never use!

SSH (Port 22):
  - Encrypted communication
  - Secure authentication
  - Industry standard
```

### SSH Architecture

```
SSH Client                                SSH Server
    |                                         |
    | ------ Connection Request -------->     |
    |                                         |
    | <----- Server Public Key ---------     |
    |                                         |
    | ------ Client Auth (key/pass) ---->    |
    |                                         |
    | <----- Authenticated -------------     |
    |                                         |
    [Encrypted Session Established]
```

### Authentication Methods

**1. Password Authentication:**
```
User provides username and password
- Simple but less secure
- Vulnerable to brute force
- Not ideal for automation
```

**2. Public Key Authentication (Recommended):**
```
User has key pair (public + private)
- Private key stays on client (never shared)
- Public key on server (~/.ssh/authorized_keys)
- More secure
- Perfect for automation
```

**3. Certificate-Based Authentication:**
```
SSH certificates signed by CA
- Enterprise environments
- Centralized management
```

### SSH Keys

**Key Pair Generation:**
```
ssh-keygen generates two files:
  - id_rsa (private key) - Keep secret!
  - id_rsa.pub (public key) - Share this

Private key = Password to your account
Public key = Lock that only your private key can open
```

**Key Types:**
- **RSA:** Traditional, 2048-4096 bits (most compatible)
- **Ed25519:** Modern, faster, more secure (recommended)
- **ECDSA:** Elliptic curve (good security)
- **DSA:** Deprecated (insecure, don't use)

---

## 🔧 Practical Implementation

### SSH Client Usage

#### Basic SSH Connection

```bash
# Connect to remote server
ssh username@hostname
ssh user@192.168.1.100
ssh user@example.com

# Specify port
ssh -p 2222 user@hostname

# Use specific private key
ssh -i ~/.ssh/mykey user@hostname

# Execute command on remote server
ssh user@hostname 'uptime'
ssh user@hostname 'df -h'

# Run command and exit
ssh user@hostname 'sudo systemctl restart nginx'

# Interactive with command
ssh -t user@hostname 'sudo vim /etc/nginx/nginx.conf'
```

#### SSH Key Generation

```bash
# Generate RSA key (2048 bit)
ssh-keygen -t rsa -b 2048

# Generate RSA key (4096 bit, more secure)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Generate Ed25519 key (recommended)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Generate key with custom filename
ssh-keygen -t ed25519 -f ~/.ssh/myserver_key

# Generate key without passphrase (automation)
ssh-keygen -t ed25519 -f ~/.ssh/deploy_key -N ""

# Change passphrase of existing key
ssh-keygen -p -f ~/.ssh/id_rsa
```

**Interactive prompts:**
```bash
$ ssh-keygen -t ed25519
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/user/.ssh/id_ed25519): [Press Enter]
Enter passphrase (empty for no passphrase): [Type passphrase]
Enter same passphrase again: [Type passphrase]
Your identification has been saved in /home/user/.ssh/id_ed25519
Your public key has been saved in /home/user/.ssh/id_ed25519.pub
```

#### Copy Public Key to Server

```bash
# Use ssh-copy-id (easiest method)
ssh-copy-id user@hostname
ssh-copy-id -i ~/.ssh/mykey.pub user@hostname

# Specify port
ssh-copy-id -p 2222 user@hostname

# Manual method
cat ~/.ssh/id_rsa.pub | ssh user@hostname 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'

# Or copy paste manually
cat ~/.ssh/id_rsa.pub
# Then on remote server:
mkdir -p ~/.ssh
echo "ssh-rsa AAAAB3Nza..." >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### SSH Configuration

#### Client Configuration (~/.ssh/config)

```bash
# Create/edit config file
vim ~/.ssh/config
```

**Basic config:**
```
# Default settings for all hosts
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    TCPKeepAlive yes

# Specific server configuration
Host myserver
    HostName 192.168.1.100
    User ubuntu
    Port 22
    IdentityFile ~/.ssh/myserver_key

# Production server
Host prod
    HostName prod.example.com
    User deploy
    Port 2222
    IdentityFile ~/.ssh/prod_key
    ForwardAgent yes

# Jump host / Bastion
Host bastion
    HostName bastion.example.com
    User admin
    IdentityFile ~/.ssh/bastion_key

# Server behind bastion
Host internal
    HostName 10.0.1.50
    User ubuntu
    ProxyJump bastion
    IdentityFile ~/.ssh/internal_key

# GitHub
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_key
```

**Usage:**
```bash
# Instead of: ssh -p 2222 -i ~/.ssh/prod_key deploy@prod.example.com
# Just use:
ssh prod

# Connect through bastion automatically
ssh internal  # Automatically jumps through bastion
```

#### Server Configuration (/etc/ssh/sshd_config)

```bash
# Edit server config (requires root)
sudo vim /etc/ssh/sshd_config
```

**Recommended settings:**
```
# Change default port (security through obscurity)
Port 2222

# Listen on specific IP
ListenAddress 0.0.0.0
ListenAddress ::

# Use protocol 2 only
Protocol 2

# Authentication settings
PermitRootLogin no                    # Disable root login
PasswordAuthentication no             # Disable password auth
PubkeyAuthentication yes              # Enable key-based auth
AuthorizedKeysFile .ssh/authorized_keys

# Security settings
PermitEmptyPasswords no
ChallengeResponseAuthentication no
UsePAM yes

# Limit users/groups
AllowUsers user1 user2
AllowGroups ssh-users

# Connection settings
MaxAuthTries 3
MaxSessions 10
LoginGraceTime 60
ClientAliveInterval 300
ClientAliveCountMax 2

# Disable X11 forwarding if not needed
X11Forwarding no

# Logging
SyslogFacility AUTH
LogLevel VERBOSE

# SFTP subsystem
Subsystem sftp /usr/lib/openssh/sftp-server
```

**Restart SSH after changes:**
```bash
# Test configuration first
sudo sshd -t

# If test passes, restart
sudo systemctl restart sshd
sudo systemctl restart ssh  # On Debian/Ubuntu
```

### SSH Agent

```bash
# Start SSH agent
eval "$(ssh-agent -s)"

# Add key to agent
ssh-add ~/.ssh/id_rsa
ssh-add ~/.ssh/id_ed25519

# Add key with specific lifetime (1 hour)
ssh-add -t 3600 ~/.ssh/id_rsa

# List loaded keys
ssh-add -l

# Remove all keys from agent
ssh-add -D

# Remove specific key
ssh-add -d ~/.ssh/id_rsa

# Forward agent to remote server
ssh -A user@hostname
```

### Port Forwarding & Tunneling

#### Local Port Forwarding

```bash
# Forward local port to remote host
ssh -L local_port:remote_host:remote_port user@ssh_server

# Example: Access remote database
ssh -L 3306:localhost:3306 user@dbserver
# Now: mysql -h 127.0.0.1 -P 3306

# Access service through SSH tunnel
ssh -L 8080:internal-server:80 user@bastion
# Open http://localhost:8080 in browser

# Multiple forwards
ssh -L 3306:db:3306 -L 6379:redis:6379 user@bastion
```

#### Remote Port Forwarding

```bash
# Forward remote port to local machine
ssh -R remote_port:local_host:local_port user@remote_server

# Example: Expose local web server to remote
ssh -R 8080:localhost:3000 user@remote
# Remote users can access localhost:8080

# Reverse SSH for remote support
ssh -R 2222:localhost:22 user@public-server
# From public-server: ssh -p 2222 localhost
```

#### Dynamic Port Forwarding (SOCKS Proxy)

```bash
# Create SOCKS proxy
ssh -D 1080 user@remote

# Configure browser/app to use SOCKS5 proxy:
# Host: localhost
# Port: 1080

# Use with curl
curl --socks5 localhost:1080 https://example.com

# Full tunnel for all traffic
ssh -D 1080 -C -N user@remote
# -C: Compression
# -N: No remote command (tunnel only)
```

### File Transfer

#### SCP (Secure Copy)

```bash
# Copy file to remote
scp file.txt user@remote:/path/to/destination/
scp local.txt user@192.168.1.100:/home/user/

# Copy file from remote
scp user@remote:/path/to/file.txt ./local.txt

# Copy directory recursively
scp -r /local/directory user@remote:/remote/path/

# Specify port
scp -P 2222 file.txt user@remote:/path/

# Use specific key
scp -i ~/.ssh/mykey file.txt user@remote:/path/

# Copy between two remote hosts
scp user1@host1:/file.txt user2@host2:/path/

# Show progress
scp -v file.txt user@remote:/path/

# Limit bandwidth (KB/s)
scp -l 1000 largefile.zip user@remote:/path/
```

#### SFTP (SSH File Transfer Protocol)

```bash
# Connect to SFTP server
sftp user@hostname

# SFTP commands:
sftp> ls                    # List remote files
sftp> lls                   # List local files
sftp> pwd                   # Remote working directory
sftp> lpwd                  # Local working directory
sftp> cd /path             # Change remote directory
sftp> lcd /path            # Change local directory
sftp> get file.txt         # Download file
sftp> put file.txt         # Upload file
sftp> get -r directory     # Download directory
sftp> put -r directory     # Upload directory
sftp> mkdir newdir         # Create remote directory
sftp> rm file.txt          # Delete remote file
sftp> bye                   # Exit

# Non-interactive SFTP
sftp user@hostname <<EOF
get remote.txt
put local.txt
bye
EOF

# Batch mode
sftp -b commands.txt user@hostname
```

### SSH Tunneling Use Cases

#### Database Access Through Bastion

```bash
# SSH tunnel to database
ssh -L 5432:database.internal:5432 user@bastion -N -f
# -N: No command execution
# -f: Background mode

# Connect to database through tunnel
psql -h localhost -p 5432 -U dbuser database
```

#### Access Internal Web Application

```bash
# Tunnel to internal web app
ssh -L 8080:webapp.internal:80 user@bastion -N

# Open browser: http://localhost:8080
```

---

## ✨ Best Practices

### 1. **Use Key-Based Authentication**

```bash
# ✅ Good: Generate strong keys
ssh-keygen -t ed25519 -C "work@example.com"

# ✅ Good: Use different keys for different purposes
~/.ssh/id_ed25519_work    # Work servers
~/.ssh/id_ed25519_personal # Personal projects
~/.ssh/id_ed25519_github   # GitHub
```

### 2. **Protect Your Private Keys**

```bash
# Set correct permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/authorized_keys

# Use passphrase for keys
ssh-keygen -t ed25519 -C "email@example.com"
# Enter strong passphrase when prompted
```

### 3. **Use SSH Config File**

```bash
# ~/.ssh/config
Host prod
    HostName prod.example.com
    User deploy
    IdentityFile ~/.ssh/prod_key
    Port 2222

# Usage:
ssh prod  # Instead of full command
```

### 4. **Harden SSH Server**

```bash
# /etc/ssh/sshd_config
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
Port 2222  # Non-standard port
MaxAuthTries 3
AllowUsers deploy admin
```

### 5. **Use SSH Agent for Multiple Connections**

```bash
# Start agent and add keys
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Agent forwards authentication
ssh -A user@bastion
# From bastion, connect to other servers without re-entering password
```

### 6. **Monitor SSH Access**

```bash
# View SSH logs
sudo tail -f /var/log/auth.log  # Debian/Ubuntu
sudo tail -f /var/log/secure    # CentOS/RHEL

# Failed login attempts
sudo grep "Failed password" /var/log/auth.log

# Successful logins
sudo grep "Accepted" /var/log/auth.log
```

---

## ⚠️ Common Pitfalls

### 1. **Wrong File Permissions**

```bash
# ❌ Bad: Incorrect permissions
chmod 777 ~/.ssh
chmod 644 ~/.ssh/id_rsa  # Private key readable by others!

# ✅ Good: Correct permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
```

### 2. **Using Root Account**

```bash
# ❌ Bad: Direct root access
ssh root@server

# ✅ Good: Use regular user with sudo
ssh user@server
sudo command
```

### 3. **Forgetting to Test Config Changes**

```bash
# ❌ Bad: Restart without testing
sudo systemctl restart sshd  # May lock you out!

# ✅ Good: Test first, keep session open
sudo sshd -t  # Test configuration
# Open new terminal, test connection
# If successful, then restart in original session
```

### 4. **Not Using Jump Hosts Properly**

```bash
# ❌ Bad: Manual two-step process
ssh bastion
ssh internal-server

# ✅ Good: Use ProxyJump
ssh -J bastion internal-server
# Or in ~/.ssh/config:
Host internal
    ProxyJump bastion
```

---

## 📝 Hands-On Exercises

### Exercise 1: SSH Key Setup (Beginner)

```bash
# 1. Generate SSH key pair
ssh-keygen -t ed25519 -C "lab@example.com" -f ~/.ssh/lab_key

# 2. View public key
cat ~/.ssh/lab_key.pub

# 3. Copy to remote server (if you have access)
ssh-copy-id -i ~/.ssh/lab_key.pub user@hostname

# 4. Test connection
ssh -i ~/.ssh/lab_key user@hostname

# 5. Add to SSH config
cat >> ~/.ssh/config <<EOF
Host lab
    HostName hostname
    User user
    IdentityFile ~/.ssh/lab_key
EOF

# 6. Test simplified connection
ssh lab
```

### Exercise 2: Port Forwarding (Intermediate)

```bash
# Scenario: Access remote MySQL database through SSH tunnel

# 1. Create SSH tunnel
ssh -L 3306:localhost:3306 user@dbserver -N -f

# 2. Verify tunnel is running
ps aux | grep "ssh -L"
netstat -tuln | grep 3306

# 3. Connect to database through tunnel
mysql -h 127.0.0.1 -P 3306 -u dbuser -p

# 4. Kill tunnel when done
pkill -f "ssh -L 3306"
```

### Exercise 3: SSH Security Audit (Advanced)

```bash
#!/bin/bash
# ssh_security_audit.sh

echo "SSH Security Audit"
echo "=================="
echo ""

# Check SSH version
echo "1. SSH Version:"
ssh -V 2>&1
echo ""

# Check running SSH daemon
echo "2. SSH Daemon Status:"
systemctl status sshd | grep Active
echo ""

# Check SSH port
echo "3. SSH Port:"
sudo grep "^Port" /etc/ssh/sshd_config || echo "Default port 22"
echo ""

# Check root login setting
echo "4. Root Login:"
sudo grep "^PermitRootLogin" /etc/ssh/sshd_config
echo ""

# Check password authentication
echo "5. Password Authentication:"
sudo grep "^PasswordAuthentication" /etc/ssh/sshd_config
echo ""

# Recent SSH login attempts
echo "6. Recent SSH Activity (last 20):"
sudo grep "sshd" /var/log/auth.log | tail -20
echo ""

# Active SSH connections
echo "7. Active SSH Connections:"
who
echo ""

# Check for suspicious authorized_keys
echo "8. Authorized Keys Check:"
if [ -f ~/.ssh/authorized_keys ]; then
    echo "Found authorized_keys file"
    wc -l ~/.ssh/authorized_keys
else
    echo "No authorized_keys file"
fi
```

---

## 🔗 Further Learning

### Related Skills
- **Skill 01:** TCP/IP Fundamentals
- **Skill 05:** Firewall & Network Security
- **CI/CD Agent:** SSH in deployment pipelines

### Command Reference
- `man ssh` - SSH client manual
- `man sshd_config` - SSH daemon configuration
- `man ssh_config` - SSH client configuration
- `man ssh-keygen` - Key generation tool

### External Resources
- OpenSSH Official Documentation
- RFC 4251-4254: SSH Protocol
- SSH Academy: https://www.ssh.com/academy/
- Hardening SSH: Mozilla's OpenSSH Guidelines

### Advanced Topics
- SSH certificates
- Multi-factor authentication (MFA) with SSH
- SSH bastion/jump hosts
- SSH key management at scale
- SSH honeypots
- SSH session recording

---

## 🎓 Skill Completion Checklist

Before moving to the next skill, verify you can:

- [ ] Generate SSH key pairs (RSA, Ed25519)
- [ ] Copy public keys to remote servers
- [ ] Configure SSH client (~/.ssh/config)
- [ ] Use SSH agent for key management
- [ ] Connect to servers with key-based auth
- [ ] Set up port forwarding (local, remote, dynamic)
- [ ] Transfer files with SCP and SFTP
- [ ] Configure SSH server (sshd_config)
- [ ] Use jump hosts / ProxyJump
- [ ] Troubleshoot SSH connection issues

**When you can do all of these, move to Skill 05: Firewall & Network Security**

---

**Skill Status:** ✅ Ready to Use
**Last Updated:** November 2024
**Difficulty:** Intermediate
**Time to Complete:** 5 hours
