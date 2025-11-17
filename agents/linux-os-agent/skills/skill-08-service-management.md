# Skill 08: Service Management (systemd)

> Master systemd for managing system services, daemons, and processes

## 📚 Overview

systemd is the modern init system in Linux, managing services, timers, and system resources. Essential for DevOps engineers.

---

## 🔧 Essential Commands

### Service Management

```bash
# Check service status
sudo systemctl status nginx           # Current status
sudo systemctl is-active nginx        # Yes/no answer
sudo systemctl is-enabled nginx       # Enabled at boot?

# Start/stop/restart
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx           # Reload config (no restart)

# Enable/disable at boot
sudo systemctl enable nginx           # Enable at boot
sudo systemctl disable nginx          # Disable at boot
sudo systemctl enable --now nginx     # Enable and start

# List services
systemctl list-units --type=service   # All services
systemctl list-units --type=service --all  # Include inactive
systemctl list-unit-files --type=service   # Installed services
```

### Service Information

```bash
# View service file
systemctl cat nginx                   # Show service definition
cat /lib/systemd/system/nginx.service # Direct path

# Dependencies
systemctl list-dependencies nginx     # What depends on it
systemctl list-dependencies nginx --reverse  # What it depends on

# Service properties
systemctl show nginx                  # All properties
systemctl show nginx -p ExecStart     # Specific property

# View logs
journalctl -u nginx                   # Service logs
journalctl -u nginx -f                # Follow logs
journalctl -u nginx -n 50             # Last 50 lines
journalctl -u nginx --since "1 hour ago"
```

### Creating Custom Services

```bash
# Create service file
cat > /etc/systemd/system/myapp.service << 'EOF'
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 /opt/myapp/app.py
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

# Create oneshot service (runs once then stops)
[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh

# Simple HTTP service
[Service]
Type=simple
ExecStart=/usr/bin/python3 -m http.server 8000

# Reload and enable
sudo systemctl daemon-reload
sudo systemctl enable myapp.service
sudo systemctl start myapp.service
```

### Service File Structure

```ini
[Unit]
Description=Service description
Documentation=man:nginx(8)
After=syslog.target network-online.target
Wants=network-online.target

[Service]
Type=notify                          # Type: simple, forking, oneshot, notify, idle
ExecStartPre=/usr/sbin/nginx -t      # Pre-start checks
ExecStart=/usr/sbin/nginx            # Main process
ExecReload=/bin/kill -s HUP $MAINPID # Reload command
ExecStop=/bin/kill -s QUIT $MAINPID  # Stop command
Restart=on-failure                   # Restart policy
RestartSec=10                        # Wait 10s before restart
StandardOutput=journal               # Logging
StandardError=journal
User=www-data                        # Run as user
Group=www-data                       # Run as group
ProtectSystem=strict                 # Sandbox filesystem
ProtectHome=yes                      # Sandbox home

[Install]
WantedBy=multi-user.target           # Target to add to
```

### Timers (Cron Alternative)

```bash
# Create timer
cat > /etc/systemd/system/backup.service << 'EOF'
[Unit]
Description=Daily Backup
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
EOF

# Create corresponding timer
cat > /etc/systemd/system/backup.timer << 'EOF'
[Unit]
Description=Daily Backup Timer
Requires=backup.service

[Timer]
OnCalendar=daily            # Daily
OnCalendar=*-*-01 02:00     # 1st of month at 2 AM
OnCalendar=Mon,Wed,Fri 10:00  # Mon/Wed/Fri at 10 AM
OnBootSec=10min             # 10 mins after boot
OnUnitActiveSec=1d          # Every day from last run
Persistent=true             # Catch up missed runs

[Install]
WantedBy=timers.target
EOF

# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer

# Check timer
systemctl list-timers
systemctl status backup.timer
```

### Resource Limits

```bash
# Configure limits in service
[Service]
MemoryLimit=512M            # Max memory
MemoryAccounting=yes        # Track memory
CPUQuota=50%                # CPU limit
CPUAccounting=yes           # Track CPU
TasksMax=100                # Max processes

# Check resource usage
systemctl status myapp
# Memory: 256.0M

# Runtime info
cat /sys/fs/cgroup/memory/system.slice/myapp.service/memory.limit_in_bytes
```

### Advanced Service Control

```bash
# Mask service (prevent all operations)
sudo systemctl mask unwanted-service
sudo systemctl unmask service

# Isolate target (like runlevel)
sudo systemctl isolate multi-user.target  # Text mode
sudo systemctl isolate graphical.target   # GUI mode

# List targets
systemctl list-units --type=target

# Default target
systemctl get-default
sudo systemctl set-default multi-user.target

# View failure state
systemctl list-units --failed
systemctl reset-failed
```

---

## 📝 Common Service Examples

### Python Web App Service

```ini
[Unit]
Description=Python Web Application
After=network.target

[Service]
Type=notify
User=appuser
WorkingDirectory=/app
Environment="PYTHONUNBUFFERED=1"
ExecStart=/usr/bin/python3 -m gunicorn --bind 0.0.0.0:8000 app:app
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

### Docker Container Service

```ini
[Unit]
Description=My Docker Container
After=docker.service
Requires=docker.service

[Service]
Type=simple
Restart=always
ExecStart=/usr/bin/docker run --name myapp myapp:latest
ExecStop=/usr/bin/docker stop myapp
ExecStopPost=/usr/bin/docker rm myapp

[Install]
WantedBy=multi-user.target
```

---

## ✨ Best Practices

### 1. **Proper Logging**
```ini
[Service]
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp
```

### 2. **Security**
```ini
[Service]
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=yes
ReadWritePaths=/var/myapp
```

### 3. **Graceful Shutdown**
```ini
[Service]
ExecStop=/bin/kill -TERM $MAINPID
TimeoutStopSec=30
KillMode=process
```

---

## 🎓 Completion Checklist

- [ ] Check service status with systemctl
- [ ] Start, stop, restart services
- [ ] Enable/disable services at boot
- [ ] View service logs with journalctl
- [ ] Create custom service files
- [ ] Understand service file structure
- [ ] Configure service restart policies
- [ ] Create and manage timers
- [ ] Set resource limits
- [ ] Troubleshoot service issues

**Completed:** Linux & OS Fundamentals Agent (All 8 Skills) ✅

---

**Skill Status:** ✅ Ready to Use | **Difficulty:** Intermediate | **Time:** 5 hours

**Agent Complete!** You've mastered Linux fundamentals. Ready for Agent 2: Network & Protocols
