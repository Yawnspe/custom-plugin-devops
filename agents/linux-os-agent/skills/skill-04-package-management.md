# Skill 04: Package Management (APT, YUM)

> Master software installation and management using apt (Debian/Ubuntu) and yum (RHEL/CentOS)

## 📚 Overview

Package managers automate software installation, updates, and dependency management. Debian-based systems use APT while RHEL-based systems use YUM/DNF.

### Learning Objectives
- Install, update, and remove packages
- Search for packages
- Manage repositories
- Handle dependencies
- Use both APT and YUM tools

---

## 🏗️ Core Concepts

### Debian/Ubuntu (APT)

**Distribution:** Ubuntu, Debian, Linux Mint, etc.
**Package Format:** .deb
**Package Manager:** apt (modern), apt-get, dpkg (low-level)

### RHEL/CentOS (YUM)

**Distribution:** CentOS, RHEL, Fedora, AlmaLinux
**Package Format:** .rpm
**Package Manager:** yum, dnf (modern), rpm (low-level)

---

## 🔧 Essential Commands

### APT (Debian/Ubuntu)

```bash
# Update package lists
sudo apt update

# Upgrade installed packages
sudo apt upgrade                    # Safe upgrade
sudo apt full-upgrade              # More aggressive

# Install package
sudo apt install package_name
sudo apt install pkg1 pkg2 pkg3    # Multiple packages

# Remove package (keep config)
sudo apt remove package_name

# Remove package and config
sudo apt purge package_name

# Remove unused dependencies
sudo apt autoremove

# Search for package
apt search keyword
apt-cache search keyword

# Show package info
apt show package_name
apt-cache show package_name

# List installed packages
apt list --installed | grep pattern
dpkg -l | grep pattern

# Check if package is installed
dpkg -l | grep ^ii | grep package_name

# Find which package provides a command
apt-file search /path/to/command

# Autoremove and autoclean
sudo apt autoclean               # Remove partial packages
sudo apt autoremove              # Remove unused dependencies
```

### YUM (RHEL/CentOS)

```bash
# Update all packages
sudo yum update

# Or use dnf (newer)
sudo dnf update

# Install package
sudo yum install package_name
sudo dnf install package_name

# Remove package
sudo yum remove package_name
sudo dnf remove package_name

# Search for package
yum search keyword
dnf search keyword

# Show package info
yum info package_name
dnf info package_name

# List installed packages
yum list installed
dnf list installed

# List available packages
yum list available
dnf list available

# Find which package provides a file
yum provides /path/to/file
dnf provides /path/to/file

# Update specific package
yum update package_name
dnf upgrade package_name

# Check for updates
yum check-update
dnf check-update

# Downgrade package
yum downgrade package_name
dnf downgrade package_name
```

### Repository Management

```bash
# APT - Add repository
sudo add-apt-repository ppa:user/ppa-name
sudo apt update

# APT - Remove repository
sudo add-apt-repository --remove ppa:user/ppa-name

# APT - List repositories
cat /etc/apt/sources.list
ls -la /etc/apt/sources.list.d/

# YUM - List enabled repositories
yum repolist
dnf repolist

# YUM - Enable/disable repository
yum-config-manager --enable repository-name
yum-config-manager --disable repository-name

# YUM - Add custom repository
sudo yum-config-manager --add-repo=https://example.com/repo.repo
```

### Package Information

```bash
# APT - Dependencies
apt depends package_name
apt-cache depends package_name

# APT - Reverse dependencies (what needs this package)
apt-cache rdepends package_name

# YUM - Dependencies
yum deplist package_name
dnf repoquery --requires package_name

# APT - Package size
apt-cache show package_name | grep Size

# YUM - Package size
yum info package_name | grep Size

# APT - Installation size
apt-cache show package_name | grep Installed-Size
```

### Managing Multiple Versions

```bash
# APT - Install specific version
sudo apt install package_name=1.2.3

# APT - Hold package (prevent updates)
sudo apt-mark hold package_name
sudo apt-mark unhold package_name

# YUM - Install specific version
sudo yum install package_name-1.2.3

# YUM - Lock package
sudo yum versionlock package_name
```

---

## ✨ Best Practices

### 1. **Keep System Updated**
```bash
# Regular updates (weekly)
sudo apt update && sudo apt upgrade

# On RHEL/CentOS
sudo yum update

# Automatic updates
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

### 2. **Clean Up**
```bash
# Remove unnecessary packages
sudo apt autoremove

# Clean package cache
sudo apt clean           # Remove all cached packages
sudo apt autoclean       # Remove old cached packages

# On RHEL/CentOS
sudo yum autoremove
sudo yum clean all
```

### 3. **Repository Security**
```bash
# Verify APT keys
apt-key list

# Add key for custom repository
curl https://example.com/key.gpg | sudo apt-key add -

# YUM keys
rpm --import https://example.com/key.gpg
```

### 4. **Scripted Installation**
```bash
#!/bin/bash
# Install multiple packages safely

packages="nginx mysql-server php curl wget"

sudo apt update
for package in $packages; do
    if dpkg -l | grep -q "^ii.*$package"; then
        echo "$package already installed"
    else
        sudo apt install -y $package
    fi
done
```

---

## ⚠️ Common Pitfalls

### 1. **Forgetting apt update**
```bash
# ❌ Bad: Package lists outdated
sudo apt install nginx

# ✅ Good: Update first
sudo apt update
sudo apt install nginx
```

### 2. **Breaking Dependencies**
```bash
# ❌ Bad: Force remove without considering dependencies
sudo apt remove --force-yes package

# ✅ Good: Let apt handle dependencies
sudo apt remove package
sudo apt autoremove  # Clean up unused deps
```

### 3. **Mixing Package Managers**
```bash
# ❌ Bad: Inconsistent state
apt install nginx
yum install nginx  # Different distro

# ✅ Good: Use appropriate manager for distro
# Ubuntu: apt
# CentOS: yum
```

---

## 📝 Exercises

### Exercise 1: Install and Manage Packages

```bash
# 1. Search for web server packages
apt search "web server"
# or
yum search "web server"

# 2. Install nginx
sudo apt install nginx
# or
sudo yum install nginx

# 3. Verify installation
which nginx
nginx -v

# 4. Remove it
sudo apt remove nginx
# or
sudo yum remove nginx
```

### Exercise 2: Repository Management

```bash
# 1. List current repositories
cat /etc/apt/sources.list

# 2. Add a PPA (Ubuntu)
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update

# 3. Install from new repository
sudo apt install python3.11

# 4. Remove repository
sudo add-apt-repository --remove ppa:deadsnakes/ppa
```

---

## 📊 Quick Reference

```bash
# APT
sudo apt update && sudo apt upgrade    # Update all
sudo apt install package               # Install
sudo apt remove package                # Remove
sudo apt autoremove                    # Clean deps
apt search keyword                     # Search
apt show package                       # Info
apt-cache depends package              # Dependencies

# YUM/DNF
sudo yum update                        # Update all
sudo yum install package               # Install
sudo yum remove package                # Remove
yum search keyword                     # Search
yum info package                       # Info
yum deplist package                    # Dependencies

# APT file search
sudo apt install apt-file
apt-file update
apt-file search /usr/bin/command       # Find package providing command
```

---

## 🎓 Completion Checklist

- [ ] Update package lists with apt/yum
- [ ] Install packages and handle dependencies
- [ ] Remove packages cleanly
- [ ] Search for and find packages
- [ ] Manage system repositories
- [ ] Understand apt vs yum differences
- [ ] Install from custom repositories
- [ ] Manage package versions
- [ ] Clean up unused packages
- [ ] Handle broken dependencies

**Next:** Skill 05 - User & Group Management

---

**Skill Status:** ✅ Ready to Use | **Difficulty:** Beginner | **Time:** 5 hours
