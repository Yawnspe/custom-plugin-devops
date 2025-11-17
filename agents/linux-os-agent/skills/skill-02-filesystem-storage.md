# Skill 02: Filesystem & Storage Management

> Master Linux filesystems, partitions, mounting, and disk space management

## 📚 Overview

Filesystems are how Linux organizes and stores data. Understanding partitions, filesystems, mounting, and disk management is critical for system administration and troubleshooting.

### Key Learning Objectives
- Understand Linux filesystem hierarchy (FHS)
- Work with partitions and block devices
- Mount and unmount filesystems
- Monitor disk space usage
- Manage storage effectively
- Understand different filesystem types

---

## 🏗️ Core Concepts

### Linux Directory Structure (FHS)

```
/                              Root directory
├── /bin                       Essential user binaries (ls, cp, bash)
├── /sbin                      System binaries (fdisk, iptables)
├── /etc                       System configuration files
├── /home                      User home directories
├── /root                      Root user's home directory
├── /var                       Variable data (logs, temp files)
│   └── /var/log              System and application logs
├── /tmp                       Temporary files (cleared on reboot)
├── /usr                       User programs and data
│   ├── /usr/bin              User binaries
│   ├── /usr/sbin             System binaries (less critical)
│   ├── /usr/lib              Libraries
│   └── /usr/share            Shared data
├── /opt                       Optional/third-party software
├── /lib                       Essential libraries
├── /dev                       Device files
├── /proc                      Kernel information (virtual filesystem)
├── /sys                       System information (virtual filesystem)
├── /boot                      Boot loader files
├── /media                     Removable media mount points
├── /mnt                       Temporary mount points
└── /run                       Runtime data (temporary)
```

### Common Filesystem Types

| Type | Purpose | Journaling | Use Case |
|------|---------|-----------|----------|
| **ext4** | Most common Linux filesystem | Yes | General purpose |
| **XFS** | High-performance filesystem | Yes | Large files, enterprise |
| **Btrfs** | Modern with snapshots | Yes | Advanced features |
| **ZFS** | Advanced data management | N/A | Enterprise storage |

### Inode Structure

```
Inode (Index Node) contains:
├── File type (regular file, directory, symlink)
├── Permissions
├── Owner (UID) and group (GID)
├── File size
├── Timestamps (accessed, modified, created)
├── Link count
└── Pointers to data blocks on disk
```

---

## 🔧 Practical Implementation

### Essential Commands

#### 1. **df** - Disk Space Usage

```bash
# Show disk space in human-readable format
df -h

# Show all filesystems including virtual ones
df -a

# Show filesystem type
df -T

# Show disk usage by inode
df -i

# Show only specific partition
df -h /home

# Find filesystems filling up
df -h | grep -E '8[0-9]%|9[0-9]%'
```

#### 2. **du** - Directory Disk Usage

```bash
# Show size of directory and subdirectories
du -sh /home

# Human-readable with all directories
du -h /var/log

# Sort by size (largest first)
du -sh /var/* | sort -rh | head -10

# Depth limit (only first level)
du -h -d 1 /

# Find directories over 100MB
du -h /var | grep -E '^[0-9]*[MG]'

# Show files only
du -h --max-depth=1 /var/log | grep -v '/$'
```

#### 3. **lsblk** - Block Devices

```bash
# List all block devices
lsblk

# Show filesystem info
lsblk -f

# Show full paths
lsblk -p

# Show device tree
lsblk -t
```

#### 4. **fdisk** - Partition Management (Dangerous!)

```bash
# List partitions
sudo fdisk -l

# Edit partition table (DANGEROUS - use with caution!)
sudo fdisk /dev/sda

# Within fdisk:
#   p - Print partition table
#   n - New partition
#   d - Delete partition
#   w - Write changes
#   q - Quit without saving

# Create partition interactively
sudo parted /dev/sda
```

#### 5. **mount** & **umount** - Filesystem Mounting

```bash
# List mounted filesystems
mount

# Mount filesystem
sudo mount /dev/sda1 /mnt/data

# Mount with read-only
sudo mount -o ro /dev/sda1 /mnt/data

# Mount with custom options
sudo mount -o defaults,noatime /dev/sda1 /mnt/data

# Unmount filesystem
sudo umount /mnt/data

# Force unmount (if busy)
sudo umount -l /mnt/data

# Check before unmounting
lsof /mnt/data  # See what's using the mount

# Remount with different options
sudo mount -o remount,ro /
```

#### 6. **mkfs** - Create Filesystem

```bash
# Create ext4 filesystem (DESTRUCTIVE!)
sudo mkfs.ext4 /dev/sda1

# Create with label
sudo mkfs.ext4 -L "mydata" /dev/sda1

# Create XFS filesystem
sudo mkfs.xfs /dev/sda1

# Confirm before operation
sudo mkfs -t ext4 -F /dev/sda1  # -F forces confirmation
```

### Fstab Configuration

The `/etc/fstab` file controls automatic mounting:

```bash
# View current fstab
cat /etc/fstab

# Format:
# /dev/sda1      /mnt/data  ext4  defaults  0  2
# |              |          |      |         |  |
# Device         Mountpoint Type   Options   |  |
#                                            |  |
#                           Dump flag      FS check order
#                           0=no dump
#                           1=dump          0=skip check
#                                          1=check first
#                                          2=check after root

# Add new entry
echo "/dev/sda1  /mnt/data  ext4  defaults,nofail  0  2" | sudo tee -a /etc/fstab

# Test fstab changes (before reboot)
sudo mount -a

# Mount all entries in fstab
sudo mount -a
```

### Disk Usage Analysis

```bash
# Find large files
find / -type f -size +100M -exec ls -lh {} \;

# Find most recently modified files
find / -type f -mtime -1  # Modified in last 1 day

# Find empty files
find / -type f -empty

# Check disk usage with tree
tree -h -L 2 /var

# Disk usage per user
du -sh /home/*

# Cache impact on disk
# Show actual disk usage (excluding cache)
du -sh --apparent-size /var
```

---

## ✨ Best Practices

### 1. **Regular Monitoring**
```bash
# Monitor disk usage hourly
#!/bin/bash
while true; do
    df -h >> /var/log/disk_usage.log
    sleep 3600
done
```

### 2. **Cleanup Old Files**
```bash
# Delete logs older than 30 days
find /var/log -type f -name "*.log" -mtime +30 -delete

# Archive old logs
tar czf /archive/logs_2024.tar.gz /var/log/*.log.*
```

### 3. **Mount Options**
```bash
# Safe defaults for production
mount -o defaults,relatime,errors=remount-ro /dev/sda1 /data

# Performance tuning
mount -o defaults,noatime,nodiratime /dev/sda1 /data
```

### 4. **Backup Before Changes**
```bash
# Backup fstab before changes
sudo cp /etc/fstab /etc/fstab.backup

# Test mount points
sudo mount -a --verbose
```

---

## ⚠️ Common Pitfalls

### 1. **Unmounting While in Use**
```bash
# ❌ Bad: Unmounting while directory is mounted
cd /mnt/data
umount /mnt/data  # Will fail or cause corruption

# ✅ Good: Unmount after leaving directory
cd /
umount /mnt/data
```

### 2. **Running Out of Inodes**
```bash
# ❌ Bad: No inode monitoring
# Can fill inode table with many small files

# ✅ Good: Monitor inodes
df -i
# If >85%: consolidate files or migrate

# Find which directory has many files
find /dir -type f | wc -l
```

### 3. **Incorrect fstab Entry**
```bash
# ❌ Bad: Typo in mount point
/dev/sda1  /mnt/dataa  ext4  defaults  0  2

# ✅ Good: Test with mount -a first
sudo mount -a --verbose
```

---

## 📝 Hands-On Exercises

### Exercise 1: Disk Space Investigation

```bash
# 1. Check total disk space
df -h /

# 2. Find top space consumers
du -sh /var/* | sort -rh | head -5
du -sh /home/* | sort -rh | head -5

# 3. Check inode usage
df -i /

# 4. Find large log files
find /var/log -type f -size +100M

# 5. What's using the most space?
# Answer: Likely /var/log if system is old

# Solutions:
du -sh /var/log/*         # Find which log
sudo rm /var/log/old.*   # Clean old logs
```

### Exercise 2: Mounting Practice

```bash
# 1. Create a filesystem on a file (loopback)
dd if=/dev/zero of=/tmp/test.img bs=1M count=100
mkfs.ext4 /tmp/test.img

# 2. Create mount point
mkdir -p /tmp/testmount

# 3. Mount it
sudo mount -o loop /tmp/test.img /tmp/testmount

# 4. Verify
mount | grep testmount

# 5. Unmount
sudo umount /tmp/testmount

# 6. Cleanup
rm /tmp/test.img
rmdir /tmp/testmount
```

### Exercise 3: Filesystem Monitoring Script

```bash
#!/bin/bash
# Monitor disk usage and alert when >80%

THRESHOLD=80

df -h | tail -n +2 | while read line; do
    usage=$(echo $line | awk '{print $5}' | sed 's/%//g')
    filesystem=$(echo $line | awk '{print $6}')

    if [ $usage -gt $THRESHOLD ]; then
        echo "ALERT: $filesystem is $usage% full!"
        # Send email, pagerduty, etc
    fi
done
```

---

## 🔗 Further Learning

### Related Skills
- **Skill 01:** Process Management
- **Skill 04:** Package Management
- **Skill 07:** System Monitoring

### Command Reference
- `man df` - Disk free
- `man du` - Disk usage
- `man mount` - Mounting filesystems
- `man fstab` - Filesystem table
- `man mkfs` - Create filesystems

---

## 📊 Quick Reference

```bash
# Disk space
df -h                              # Show disk usage
du -sh /path/*                     # Directory sizes
lsblk                              # Block devices

# Mounting
mount                              # Show mounts
sudo mount /dev/sda1 /mnt          # Mount
sudo umount /mnt                   # Unmount
mount -a                           # Mount all from fstab

# Filesystem info
blkid                              # Show UUIDs
tune2fs -l /dev/sda1              # ext4 info
xfs_admin -l /dev/sda1            # XFS info

# Finding space usage
find /path -size +100M             # Find large files
du -h /path | sort -rh | head      # Sorted directory sizes
```

---

## 🎓 Skill Completion Checklist

- [ ] Explain Linux filesystem hierarchy (FHS)
- [ ] Monitor disk space with `df` and `du`
- [ ] List block devices with `lsblk`
- [ ] Mount and unmount filesystems
- [ ] Configure automatic mounting with fstab
- [ ] Find large files and directories
- [ ] Understand inode usage
- [ ] Explain different filesystem types
- [ ] Clean up disk space safely
- [ ] Monitor disk usage trends

**Next:** Skill 03 - File Permissions & Access Control

---

**Skill Status:** ✅ Ready to Use
**Last Updated:** November 2024
**Difficulty:** Beginner
**Time to Complete:** 6 hours
