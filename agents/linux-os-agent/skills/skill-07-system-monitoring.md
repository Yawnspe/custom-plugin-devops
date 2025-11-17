# Skill 07: System Monitoring & Diagnostics

> Monitor system resources, identify bottlenecks, and diagnose performance issues

## 📚 Overview

System monitoring is essential for identifying resource constraints, diagnosing performance issues, and ensuring system health.

---

## 🔧 Essential Monitoring Commands

### CPU Monitoring

```bash
# Real-time CPU usage
top                         # Interactive
htop                        # Enhanced
watch -n 1 'top -bn1 | head'  # Non-interactive

# CPU usage per core
mpstat -P ALL 1

# Load average
uptime
# Load: 1.5, 1.2, 0.8 (1min, 5min, 15min avg)

# Process CPU usage
ps aux | sort -k3 -rn | head -10  # Top by CPU

# CPU info
nproc                       # Number of CPUs
lscpu                       # CPU details
```

### Memory Monitoring

```bash
# Free memory
free -h                     # Human readable
free -m                     # In megabytes
free -b                     # In bytes

# Detailed memory info
cat /proc/meminfo

# Memory usage by process
ps aux | sort -k4 -rn | head -10  # Top by memory
ps aux --sort=-%mem

# Virtual memory statistics
vmstat 1 5                  # Show 5 samples, 1 sec apart

# Memory in MB for specific process
ps -o pid,rss,cmd -p PID | awk '{print $2/1024 " MB"}'
```

### Disk I/O Monitoring

```bash
# I/O statistics
iostat 1 5                  # 5 samples, 1 sec apart
iostat -x                   # Extended stats

# Disk usage by device
lsblk
df -h
df -h --total

# I/O per process (if available)
iotop -b -n 1              # Batch mode

# I/O wait
iostat -c 1                # Just CPU stats
# wa = I/O wait percentage
```

### Network Monitoring

```bash
# Network interface stats
ifstat -i eth0 1            # Interface specific
ifstat 1 5                  # All interfaces

# Connection statistics
netstat -s
ss -s

# Active connections
netstat -tulnp
ss -tulnp

# Network throughput
iftop -i eth0              # Top connections
nethogs eth0               # App-level bandwidth

# Packet statistics
ip -s link
tcpdump -i eth0 -nn host 192.168.1.1
```

### Log Analysis

```bash
# View logs
tail -f /var/log/syslog         # Follow log
tail -100 /var/log/syslog       # Last 100 lines
grep "ERROR" /var/log/app.log   # Search

# System journal (systemd)
journalctl                      # All logs
journalctl -f                   # Follow
journalctl -u service_name      # Service logs
journalctl -n 50                # Last 50 lines
journalctl --since "1 hour ago" # Time filter
journalctl -p err               # Error level

# Kernel logs
dmesg
dmesg | tail -20
```

### System Health

```bash
# Overall system status
systemctl status                # Service status
systemctl list-units --failed   # Failed services

# Check disk space
df -h /                         # Root filesystem
du -sh /*                       # Directory sizes

# Temperature (if available)
sensors                         # CPU temp
watch -n 1 'sensors | grep -A 1 "Core"'

# Uptime
uptime
cat /proc/uptime

# System load
cat /proc/loadavg
```

---

## 📝 Monitoring Scripts

### System Health Check

```bash
#!/bin/bash

echo "=== System Health Check ==="
echo "Uptime: $(uptime | cut -d',' -f1)"
echo ""

echo "=== CPU ==="
echo "Load Average: $(uptime | tail -c 14)"
echo "CPU Count: $(nproc)"
echo ""

echo "=== Memory ==="
free -h | grep Mem
echo ""

echo "=== Disk ==="
df -h / | tail -1
echo ""

echo "=== Top Processes (CPU) ==="
ps aux | sort -k3 -rn | head -5 | tail -4
echo ""

echo "=== Top Processes (Memory) ==="
ps aux | sort -k4 -rn | head -5 | tail -4
```

### Alerting Script

```bash
#!/bin/bash

CPU_THRESHOLD=80
MEM_THRESHOLD=80
DISK_THRESHOLD=90

check_cpu() {
    usage=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}')
    usage=${usage%.*}  # Integer
    if [ $usage -gt $CPU_THRESHOLD ]; then
        echo "ALERT: CPU usage is ${usage}%"
    fi
}

check_memory() {
    usage=$(free | grep Mem | awk '{printf("%.0f", 100 * $3/$2)}')
    if [ $usage -gt $MEM_THRESHOLD ]; then
        echo "ALERT: Memory usage is ${usage}%"
    fi
}

check_disk() {
    usage=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')
    if [ $usage -gt $DISK_THRESHOLD ]; then
        echo "ALERT: Disk usage is ${usage}%"
    fi
}

check_cpu
check_memory
check_disk
```

---

## ✨ Best Practices

### 1. **Baseline Monitoring**
```bash
# Create baseline of normal system behavior
# Run during normal operations
ps aux > /var/baseline/processes.txt
free -h > /var/baseline/memory.txt
df -h > /var/baseline/disk.txt
```

### 2. **Continuous Monitoring**
```bash
#!/bin/bash
# Monitor and alert

while true; do
    # Your checks here
    # Alert if issues
    sleep 300  # Every 5 minutes
done
```

### 3. **Log Rotation**
```bash
# Configure in /etc/logrotate.d/myapp
/var/log/myapp.log {
    daily
    rotate 7
    compress
    delaycompress
    notifempty
    create 0644 root root
}
```

---

## 🎓 Completion Checklist

- [ ] Monitor CPU usage and load
- [ ] Monitor memory usage
- [ ] Monitor disk space and I/O
- [ ] Monitor network activity
- [ ] Read and interpret system logs
- [ ] Use top/htop effectively
- [ ] Create monitoring scripts
- [ ] Set up alerts
- [ ] Analyze trends
- [ ] Diagnose performance issues

**Next:** Skill 08 - Service Management (systemd)

---

**Skill Status:** ✅ Ready to Use | **Difficulty:** Intermediate | **Time:** 6 hours
