# Skill 05: User & Group Management

> Manage system users and groups - critical for security and access control

## 📚 Overview

Users and groups control access to system resources. Every process runs as a specific user, and permissions are assigned to users and groups.

---

## 🔧 Essential Commands

### User Management

```bash
# Create user
sudo useradd username
sudo useradd -m username              # Create with home directory
sudo useradd -m -s /bin/bash username # Specify shell
sudo useradd -m -G group1,group2 user # Add to groups

# Create user (interactive)
sudo adduser username

# Delete user
sudo userdel username
sudo userdel -r username              # Remove home directory too

# Modify user
sudo usermod -s /bin/bash username    # Change shell
sudo usermod -d /new/home username    # Change home directory
sudo usermod -aG group username       # Add to group (-a = append)
sudo usermod -G group1,group2 username # Set groups (removes from others)

# Set password
sudo passwd username

# Lock account
sudo usermod -L username
sudo passwd -l username

# Unlock account
sudo usermod -U username
sudo passwd -u username

# Check if user exists
id username
getent passwd username

# List all users
cat /etc/passwd | cut -d: -f1
getent passwd | awk -F: '{print $1}'

# View user info
id username
finger username
```

### Group Management

```bash
# Create group
sudo groupadd groupname
sudo groupadd -g 2000 groupname       # Specify GID

# Delete group
sudo groupdel groupname

# Add user to group
sudo usermod -aG groupname username

# Add multiple users to group
for user in user1 user2 user3; do
    sudo usermod -aG groupname $user
done

# Remove user from group
sudo deluser username groupname

# Check group membership
id username
groups username
getent group groupname

# List all groups
cat /etc/group | cut -d: -f1
getent group | awk -F: '{print $1}'

# View group info
getent group groupname
```

### User Configuration Files

```bash
# User information
cat /etc/passwd

# Format: username:x:UID:GID:comment:home:shell
# Example: john:x:1000:1000:John Doe:/home/john:/bin/bash

# User passwords (hashed)
sudo cat /etc/shadow

# Format: username:password_hash:lastchange:min:max:warn:inactive:expire:reserved
# x means no password, ! means locked

# Group information
cat /etc/group

# Format: groupname:x:GID:members
# Example: docker:x:999:john,jane

# Group passwords (rarely used)
sudo cat /etc/gshadow
```

### Sudo Access

```bash
# View sudoers file (READ ONLY - use visudo!)
sudo visudo

# Format:
# user  ALL=(ALL:ALL) ALL
# user  ALL=(root) NOPASSWD: /usr/bin/systemctl

# Add user to sudoers (one-liner)
echo "username ALL=(ALL:ALL) ALL" | sudo tee -a /etc/sudoers.d/username

# No password for specific command
echo "username ALL=(ALL) NOPASSWD: /usr/bin/reboot" | sudo tee -a /etc/sudoers.d/username

# Check if user is in sudoers
sudo -l -U username

# View what commands user can run
sudo -l
```

### Effective UID/GID

```bash
# Check current user/group
id
# uid=1000(user) gid=1000(user) groups=1000(user),4(adm),27(sudo)

# Run as different user
sudo -u other_user whoami

# Run as different group
newgrp groupname

# Check EUID
id -u  # Real UID
id -U  # Effective UID
```

---

## ✨ Best Practices

### 1. **Principle of Least Privilege**
```bash
# Only give sudo to necessary users
sudo usermod -aG sudo username

# Restrict sudo for specific commands
echo "user ALL=(root) /usr/bin/systemctl" | sudo tee -a /etc/sudoers.d/user
```

### 2. **Home Directory Security**
```bash
# Set proper permissions
sudo chmod 750 /home/username
sudo chown username:username /home/username

# Secure SSH directory
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### 3. **Group-based Access**
```bash
# Create group for project
sudo groupadd project

# Add developers
sudo usermod -aG project developer1
sudo usermod -aG project developer2

# Set group permissions
sudo chown -R :project /var/www/project
sudo chmod -R g+rw /var/www/project
```

### 4. **System Users**
```bash
# Create system user (no login shell)
sudo useradd -r -s /sbin/nologin username

# Good for daemon processes
# Example: nginx, mysql, docker users
```

---

## ⚠️ Common Pitfalls

### 1. **Accidental Sudoers Corruption**
```bash
# ❌ Bad: Direct edit might corrupt
sudo nano /etc/sudoers  # Can lock you out!

# ✅ Good: Use visudo for safe editing
sudo visudo
```

### 2. **Wrong GID in useradd**
```bash
# ❌ Might create wrong primary group
sudo useradd username

# ✅ Explicit group specification
sudo useradd -m -g staff username
```

### 3. **Home Directory Not Created**
```bash
# ❌ Missing home directory
sudo useradd username

# ✅ Create with home
sudo useradd -m username

# Or create manually
sudo mkdir -p /home/username
sudo chown username:username /home/username
sudo chmod 750 /home/username
```

---

## 📝 Exercises

### Exercise 1: User Creation

```bash
#!/bin/bash
# Create developer user with proper setup

username="developer"
homedir="/home/$username"

# Create user
sudo useradd -m -s /bin/bash -G sudo,docker $username

# Set password
echo "$username:secure_password" | sudo chpasswd

# Configure sudoers
echo "$username ALL=(ALL:ALL) NOPASSWD: /usr/bin/systemctl" | \
    sudo tee -a /etc/sudoers.d/$username

# Verify
id $username
groups $username
```

### Exercise 2: Group Management

```bash
#!/bin/bash
# Create project group and assign users

groupname="webdev"
users=("alice" "bob" "charlie")

# Create group
sudo groupadd $groupname

# Add users
for user in "${users[@]}"; do
    sudo usermod -aG $groupname $user
done

# Verify
getent group $groupname
```

### Exercise 3: Security Audit

```bash
#!/bin/bash
# Find security issues

# Users with empty passwords
awk -F: '($2=="") {print $1 ": Empty password!"}' /etc/shadow

# Users with UID 0 (besides root)
awk -F: '($3==0) {print $1}' /etc/passwd

# Sudo users
getent group sudo
getent group wheel

# Locked accounts
awk -F: '($2 ~ /^!/) {print $1 ": Locked"}' /etc/shadow
```

---

## 📊 Quick Reference

```bash
# User commands
sudo useradd -m username          # Create
sudo userdel -r username          # Delete
sudo usermod -aG group user       # Add to group
sudo passwd username              # Set password

# Group commands
sudo groupadd group               # Create
sudo groupdel group               # Delete
sudo usermod -aG group user       # Add user
sudo gpasswd -a user group        # Add user (alt)

# View info
id username                       # User info
groups username                   # Group membership
getent passwd                     # All users
getent group                      # All groups

# Sudo
sudo visudo                       # Edit sudoers safely
sudo -l                          # What can I run?
sudo -l -U username              # What can user run?

# Files
/etc/passwd                       # User info
/etc/shadow                       # Hashed passwords
/etc/group                        # Group info
/etc/sudoers                      # Sudo config
```

---

## 🎓 Completion Checklist

- [ ] Create and delete users
- [ ] Create and delete groups
- [ ] Modify user attributes (shell, home, groups)
- [ ] Set user passwords
- [ ] Lock/unlock accounts
- [ ] Understand /etc/passwd and /etc/shadow
- [ ] Configure sudo access safely
- [ ] Manage group membership
- [ ] Apply principle of least privilege
- [ ] Audit user accounts for security

**Next:** Skill 06 - Bash Scripting

---

**Skill Status:** ✅ Ready to Use | **Difficulty:** Intermediate | **Time:** 5 hours
