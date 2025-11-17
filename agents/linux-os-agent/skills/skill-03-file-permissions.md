# Skill 03: File Permissions & Access Control

> Master Unix file permissions, ownership, and access control - critical for security and system administration

## 📚 Overview

File permissions are how Linux controls who can access, read, modify, or execute files and directories. This is fundamental to system security and user isolation.

### Learning Objectives
- Understand Unix permission model
- Use symbolic and numeric notation
- Change ownership and permissions
- Implement access control lists (ACLs)
- Manage default permissions (umask)
- Design secure permission schemes

---

## 🏗️ Core Concepts

### Permission Components

```
drwxr-xr-x  1  user  group  4096  Nov 17 10:00  mydir
│││││││││   │  │     │      │                   │
│││││││││   │  │     │      │                   File name
│││││││││   │  │     │      File size
│││││││││   │  │     Group name
│││││││││   │  User name
│││││││││   Link count
│││││││││
││││││││└─ Others: execute (x)
│││││││└── Others: write (w)
││││││└─── Others: read (r)
│││││└──── Group: execute (x)
││││└───── Group: write (w)
│││└────── Group: read (r)
││└─────── Owner: execute (x)
│└──────── Owner: write (w)
└───────── Owner: read (r) + file type (d=directory, -=file, l=link)
```

### Permission Types

| Symbol | Number | Meaning |
|--------|--------|---------|
| **r** | 4 | Read (view file content) |
| **w** | 2 | Write (modify file) |
| **x** | 1 | Execute (run as program) |

### Permission Targets

| Category | Target |
|----------|--------|
| **u** | User (owner) |
| **g** | Group |
| **o** | Others (everyone else) |
| **a** | All (user, group, others) |

### Directory Permissions

| Permission | Effect |
|-----------|--------|
| **r** (read) | Can list directory contents (ls) |
| **w** (write) | Can create/delete files in directory |
| **x** (execute) | Can enter directory (cd) |

---

## 🔧 Practical Implementation

### Essential Commands

#### 1. **chmod** - Change Permissions

```bash
# Symbolic notation (easier to read)
chmod u+x file.sh              # Owner: add execute
chmod g-w file.txt             # Group: remove write
chmod o=r file.txt             # Others: only read
chmod a-x file.txt             # All: remove execute
chmod u=rwx,g=rx,o= file.txt   # Set specific permissions

# Numeric notation
chmod 755 file.sh              # rwxr-xr-x (owner full, group/other read+exec)
chmod 644 file.txt             # rw-r--r-- (owner read/write, others read only)
chmod 700 private.txt          # rwx------ (owner only)
chmod 600 secret.key           # rw------- (owner read/write only)

# Recursive on directory
chmod -R 755 /var/www          # Apply to all files/dirs

# Only files or directories
chmod -R u+X /home/*           # Add execute only to directories (X = uppercase)

# Common combinations
chmod 755 /directory           # Standard directory
chmod 644 /file                # Standard file
chmod 700 /directory           # Private directory
chmod 600 /file                # Private file

# Add permissions without removing others
chmod +x script.sh             # Add execute to all who can read

# Remove all permissions
chmod 000 secret.txt           # Remove all access
```

**Numeric Calculation:**
```
755 = 7 (user rwx) + 5 (group rx) + 5 (other rx)
    = (4+2+1) + (4+1) + (4+1)

644 = 6 (user rw) + 4 (group r) + 4 (other r)
    = (4+2) + (4) + (4)

700 = 7 (user rwx) + 0 (group) + 0 (other)
    = (4+2+1) + 0 + 0
```

#### 2. **chown** - Change Ownership

```bash
# Change owner
sudo chown user file.txt

# Change owner and group
sudo chown user:group file.txt

# Change group only
sudo chown :group file.txt
# OR
sudo chgrp group file.txt

# Recursive on directory
sudo chown -R user:group /var/www

# Preserve links (don't follow symlinks)
sudo chown -P user:group link

# Change owner of symlink itself (not target)
sudo chown -h user:group symlink
```

#### 3. **umask** - Default Permissions

```bash
# View current umask
umask

# Example output: 0022
# Calculation: 0777 (default) - 0022 (umask) = 0755 (files) / 0777 (dirs)

# Change umask for current session
umask 0077      # More restrictive: new files 600, dirs 700

# Make permanent (add to ~/.bashrc)
echo "umask 0077" >> ~/.bashrc

# Common umask values
umask 0022      # Default (666-022=644 for files, 777-022=755 for dirs)
umask 0077      # Restrictive (666-077=600 for files, 777-077=700 for dirs)
umask 0002      # Group writable (666-002=664 for files, 777-002=775 for dirs)

# Umask calculation
# For files: max 666 (rw-rw-rw-)
# For dirs:  max 777 (rwxrwxrwx)
# Result = max - umask

umask 0022
# Files: 666 - 022 = 644 (rw-r--r--)
# Dirs:  777 - 022 = 755 (rwxr-xr-x)
```

#### 4. **ACLs** - Advanced Access Control

```bash
# Set ACL for specific user
sudo setfacl -m u:username:rwx /path/file

# Set ACL for specific group
sudo setfacl -m g:groupname:rx /path/file

# Remove ACL entry
sudo setfacl -x u:username /path/file

# Remove all ACLs
sudo setfacl -b /path/file

# View ACLs
getfacl /path/file

# Set default ACL (inherited by new files)
sudo setfacl -d -m u:username:rwx /directory

# Apply to directory recursively
sudo setfacl -R -m u:username:rwx /directory

# Copy ACL from one file to another
getfacl /source/file | sudo setfacl -f - /destination/file
```

### Special Permissions

```bash
# Setuid (Set User ID) - execute as file owner
chmod u+s file              # Binary 4000
chmod 4755 file

# Setgid (Set Group ID) - execute as file group
chmod g+s file              # Binary 2000
chmod 2755 file

# Sticky bit - only owner can delete files in directory
chmod +t /tmp               # Binary 1000
chmod 1777 /tmp

# Combined
chmod 7755 file             # setuid + setgid + sticky + rwxr-xr-x
```

### Security Best Practices

```bash
# Never world-writable files
chmod o-w secret.txt

# Private SSH keys (must be 600)
chmod 600 ~/.ssh/id_rsa
chmod 600 ~/.ssh/id_ed25519

# SSH directory permissions
chmod 700 ~/.ssh

# Web directories
chmod 755 /var/www/html        # Readable, but not writable
chmod 644 /var/www/html/*.html # Files not executable

# Configuration files
chmod 600 /etc/app.conf        # Owner only
chmod 640 /etc/app.conf        # Owner + group read

# Executable scripts
chmod 755 /usr/local/bin/myscript  # Everyone can execute
chmod 700 /usr/local/bin/myscript  # Owner only
```

---

## ✨ Best Practices

### 1. **Principle of Least Privilege**
```bash
# ❌ Bad: Too permissive
chmod 777 myfile.txt
chmod 777 -R /directory

# ✅ Good: Minimal needed permissions
chmod 644 file.txt         # Read/write for owner, read for others
chmod 755 directory        # Full access for owner, read+exec for others
chmod 600 private.key      # Owner only
```

### 2. **SSH Key Security**
```bash
# MUST be 600
chmod 600 ~/.ssh/id_rsa
chmod 600 ~/.ssh/id_ed25519

# Authorized keys
chmod 600 ~/.ssh/authorized_keys

# SSH directory
chmod 700 ~/.ssh
```

### 3. **Systemd Services**
```bash
# Service files
chmod 644 /etc/systemd/system/myservice.service

# Executable scripts
chmod 755 /usr/local/bin/myscript.sh

# Config files
chmod 600 /etc/myapp/config.yml
```

### 4. **Group-based Permissions**
```bash
# Create system group
sudo groupadd developers

# Add users to group
sudo usermod -aG developers user1
sudo usermod -aG developers user2

# Set group permissions on project
sudo chgrp -R developers /var/www/project
chmod -R g+rw /var/www/project
```

---

## ⚠️ Common Pitfalls

### 1. **Using 777 (World-Writable)**
```bash
# ❌ Dangerous: Everyone can modify
chmod 777 /var/www

# ✅ Safe: Only owner and group
chmod 755 /var/www
```

### 2. **Forgetting Executable Permission**
```bash
# ❌ File created but can't execute
echo "#!/bin/bash" > script.sh
./script.sh  # Permission denied

# ✅ Add execute permission
chmod +x script.sh
./script.sh  # Works
```

### 3. **Wrong SSH Key Permissions**
```bash
# ❌ Too permissive
chmod 644 ~/.ssh/id_rsa
ssh-add ~/.ssh/id_rsa  # Error: Permissions too open

# ✅ Correct permissions
chmod 600 ~/.ssh/id_rsa
ssh-add ~/.ssh/id_rsa  # Works
```

---

## 📝 Hands-On Exercises

### Exercise 1: Permission Inspection

```bash
# 1. List files with permissions
ls -l ~/

# 2. Identify permission components
# For: drwxr-xr-x  user  group  file.txt
# d = directory
# rwx = owner full access
# r-x = group read+execute
# r-x = others read+execute

# 3. Convert to numeric
# rwxr-xr-x = 755
# rw-r--r-- = 644
# rwx------ = 700

# 4. Find world-writable files (security risk)
find / -perm -o+w -type f 2>/dev/null | head -20
```

### Exercise 2: Permission Management

```bash
#!/bin/bash
# Create files with appropriate permissions

# Create script
cat > myapp.sh << 'EOF'
#!/bin/bash
echo "Hello from script"
EOF

# Set permissions
chmod 755 myapp.sh

# Verify
ls -l myapp.sh

# Create config
cat > config.txt << 'EOF'
API_KEY=secret123
EOF

# Restrict to owner only
chmod 600 config.txt

# Verify
ls -l config.txt
getfacl config.txt  # Check ACLs
```

### Exercise 3: ACL Configuration

```bash
# 1. Create test directory
mkdir /tmp/shared
cd /tmp/shared

# 2. Set ACL for specific user
sudo setfacl -m u:$(whoami):rwx /tmp/shared

# 3. View ACLs
getfacl /tmp/shared

# 4. Create default ACL for new files
sudo setfacl -d -m u:$(whoami):rw /tmp/shared

# 5. Test - create new file
touch /tmp/shared/testfile

# 6. Check permissions of new file
getfacl /tmp/shared/testfile
```

---

## 🔗 Further Learning

### Related Skills
- Skill 01: Process Management (process ownership)
- Skill 05: User & Group Management
- Skill 06: Bash Scripting (using permissions in scripts)

### Command Reference
- `man chmod` - Change mode
- `man chown` - Change owner
- `man umask` - User mask
- `man setfacl` - Set file ACLs
- `man getfacl` - Get file ACLs

---

## 📊 Quick Reference

```bash
# View permissions
ls -l file                 # Detailed permissions
stat file                  # All file information
getfacl file              # ACL information

# Change permissions (symbolic)
chmod u+x file            # Add execute for owner
chmod g-w file            # Remove write for group
chmod o= file             # Remove all for others
chmod a+r file            # Add read for all

# Change permissions (numeric)
chmod 755 file            # rwxr-xr-x
chmod 644 file            # rw-r--r--
chmod 700 file            # rwx------
chmod 600 file            # rw-------

# Change owner/group
chown user:group file     # Change both
chown user file           # Owner only
chgrp group file          # Group only

# Default permissions
umask 0022                # Set default mask
umask -S                  # Show symbolic mask

# ACLs
setfacl -m u:user:rwx file      # Set ACL
getfacl file                    # View ACLs
setfacl -b file                 # Remove all ACLs

# Find permissions
find / -perm 644                # Exact permissions
find / -perm -755               # At least 755
find / -perm /o+w               # World-writable (security issue)
```

---

## 🎓 Skill Completion Checklist

- [ ] Explain rwx permission model
- [ ] Interpret permission strings (drwxr-xr-x)
- [ ] Convert between symbolic and numeric notation
- [ ] Use `chmod` to change file permissions
- [ ] Use `chown` to change ownership
- [ ] Explain and set umask
- [ ] Configure ACLs with setfacl/getfacl
- [ ] Implement least privilege principle
- [ ] Secure sensitive files (SSH keys, configs)
- [ ] Audit and fix permission issues

**Next:** Skill 04 - Package Management

---

**Skill Status:** ✅ Ready to Use
**Last Updated:** November 2024
**Difficulty:** Beginner
**Time to Complete:** 5 hours
