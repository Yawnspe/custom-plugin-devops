# Skill 01: Process Management & Monitoring

> Master understanding, monitoring, and managing processes in Linux - the foundation of system administration

## 📚 Overview

Process management is **the most fundamental skill** in Linux system administration. Every application, service, and system function runs as one or more processes. Understanding how to view, monitor, and manage these processes is essential for any DevOps engineer.

### What You'll Learn
- Understanding process hierarchy and lifecycle
- Viewing and analyzing running processes
- Monitoring resource usage (CPU, memory)
- Managing process priority
- Killing and restarting processes
- Understanding process states
- Real-time monitoring

### Why It Matters
- **Troubleshooting:** Identify stuck or misbehaving processes
- **Performance:** Find resource hogs consuming CPU/memory
- **Automation:** Stop/start services programmatically
- **Optimization:** Adjust priorities for critical tasks
- **Reliability:** Ensure critical processes keep running

### Prerequisites
- Basic Linux familiarity
- Terminal access
- Understanding of basic commands (ls, cd, pwd)

---

## 🏗️ Foundational Concepts

### What is a Process?

A **process** is an instance of a program in execution. It's not the same as the program itself - you can run the same program multiple times, creating multiple processes.

**Key Characteristics:**
- **PID (Process ID):** Unique identifier for the process (integer)
- **PPID (Parent PID):** The process that created this process
- **User:** The user running the process
- **State:** Running, sleeping, stopped, zombie, etc.
- **Priority:** How much CPU time the process gets (nice value)

### Process Lifecycle

```
                    fork()
                      |
                      ↓
            ┌─────────────────────┐
            │    Runnable/Running │◄────┐
            └─────┬───────────────┘     │
                  │                     │
       wait for I/O│                    │ scheduled
                  │                     │
                  ↓                     │
        ┌──────────────────┐            │
        │  Waiting/Blocked │────────────┘
        │  (for I/O, lock) │
        └──────────────────┘

        ┌─────────────────────┐
        │   exit()/killed     │
        │   (Zombie state)    │
        └──────────┬──────────┘
                   │
                   ↓
            [Process Terminated]
```

### Process States

| State | Symbol | Meaning |
|-------|--------|---------|
| **Running** | R | Currently executing on CPU |
| **Sleeping** | S | Waiting for an event (interruptible) |
| **Disk Sleep** | D | Waiting for I/O (uninterruptible) |
| **Stopped** | T | Stopped by signal (SIGSTOP) |
| **Zombie** | Z | Terminated but not reaped by parent |
| **Traced** | t | Being traced by debugger |

### Process Hierarchy

Every process (except init) has a **parent process**. Processes form a tree structure:

```
        systemd (PID 1)
         │
         ├─ sshd (PID 1234)
         │  └─ bash (PID 5678)
         │     └─ vim (PID 9012)
         │
         ├─ dockerd (PID 2345)
         │
         └─ cron (PID 3456)
            └─ backup.sh (PID 7890)
```

---

## 🔧 Practical Implementation

### Essential Commands

#### 1. **ps** - List Processes

```bash
# Simple list of current user's processes
ps

# All processes in Unix format (detailed)
ps aux

# Show process tree
ps -ef
ps --forest

# Search for specific process
ps aux | grep "nginx"

# Show all processes with full details
ps auxww

# Custom columns output
ps -eo pid,user,cmd,etime
# pid: Process ID
# user: User running process
# cmd: Command line
# etime: Elapsed time

# Show parent-child relationships
ps -aux --forest

# Processes of specific user
ps -u username

# CPU and memory info
ps aux | sort -k3 -rn | head  # Top CPU users
ps aux | sort -k4 -rn | head  # Top memory users
```

**Output Explanation:**
```
USER    PID  %CPU %MEM    VSZ   RSS TTY STAT START   TIME COMMAND
root      1  0.0  0.1  19356  1544 ?   Ss  08:00   0:01 /lib/systemd/systemd
root    100  0.2  0.5  28456  5124 ?   Ss  08:01   0:05 /usr/sbin/sshd -D
user   1234  0.1  0.3  21456  3456 ?   S   08:02   0:02 bash

USER: User running the process
PID: Process ID
%CPU: CPU percentage
%MEM: Memory percentage
VSZ: Virtual memory size (KB)
RSS: Resident set size (actual memory, KB)
TTY: Terminal (? = no terminal)
STAT: Process state (S=sleeping, R=running, Z=zombie, Ss=session leader, S+=foreground)
START: When process started
TIME: CPU time used
COMMAND: The command that started the process
```

#### 2. **top** - Real-time Process Monitor

```bash
# Interactive monitoring (q to quit)
top

# Non-interactive (batch mode, update every 2 seconds, 5 iterations)
top -b -d 2 -n 5

# Sort by CPU (default)
top -o %CPU

# Sort by memory
top -o %MEM

# Monitor specific process
top -p 1234

# Show specific user
top -u username

# Update interval (seconds)
top -d 5

# Number of processes to show
top -n 30
```

**Interactive Commands in top:**
- `q` - Quit
- `space` - Refresh immediately
- `Shift+P` - Sort by CPU
- `Shift+M` - Sort by memory
- `Shift+N` - Sort by PID
- `k` - Kill process (press k, enter PID)
- `r` - Renice process (change priority)
- `f` - Choose which fields to display

#### 3. **htop** - Enhanced Process Monitor

```bash
# Interactive monitoring (more user-friendly than top)
htop

# Monitor specific process
htop -p 1234

# Show processes of user
htop -u username

# Non-interactive (batch mode)
htop -b -n 1

# Sort by CPU
htop -s PERCENT_CPU

# Sort by memory
htop -s PERCENT_MEM
```

**Advantages over top:**
- Color-coded display
- Easier navigation
- Tree view (F5)
- Better help (F1)
- Can select multiple processes (space to mark)
- Easier killing processes (F9)

#### 4. **pgrep** & **pkill** - Search and Kill Processes

```bash
# Find PID by process name
pgrep nginx
# Output: 1234
#         5678

# Find with full command match
pgrep -f "python script.py"

# Count matching processes
pgrep -c nginx
# Output: 2

# Find newest process
pgrep -n nginx
# Output: 5678

# Find oldest process
pgrep -o nginx
# Output: 1234

# Kill all processes matching name (dangerous!)
pkill nginx

# Kill with SIGTERM (graceful shutdown)
pkill -TERM nginx

# Kill only processes of user
pkill -u username nginx

# Kill with confirmation
pkill -i nginx  # -i for case-insensitive
```

#### 5. **kill** - Send Signals to Processes

```bash
# Terminate process gracefully (SIGTERM - signal 15)
kill 1234

# Force kill (SIGKILL - signal 9)
kill -9 1234

# SIGTERM (signal 15) - graceful shutdown
kill -TERM 1234
kill -15 1234

# SIGSTOP (signal 19) - pause process
kill -STOP 1234

# SIGCONT (signal 18) - resume process
kill -CONT 1234

# SIGHUP (signal 1) - hangup (often used to reload config)
kill -HUP 1234

# List all signals
kill -l

# Kill background process by job number
kill %1
```

**Signal Guide:**
| Signal | Number | Meaning |
|--------|--------|---------|
| SIGHUP | 1 | Reload configuration |
| SIGINT | 2 | Interrupt (Ctrl+C) |
| SIGTERM | 15 | Graceful termination |
| SIGKILL | 9 | Force kill (can't be caught) |
| SIGSTOP | 19 | Pause (can't be caught) |
| SIGCONT | 18 | Continue/Resume |

#### 6. **bg** & **fg** - Background/Foreground Control

```bash
# Run command in background (append &)
sleep 1000 &
# Output: [1] 5678

# List background jobs
jobs

# List all jobs with details
jobs -l

# Bring job to foreground
fg %1        # Job number 1
fg %sleep    # By name

# Send foreground job to background (Ctrl+Z, then bg)
# Ctrl+Z to pause
bg %1        # Resume in background

# Run long command in background
./long_script.sh > output.log 2>&1 &

# Disconnect background process from terminal
nohup ./long_task.sh > output.log 2>&1 &
```

#### 7. **wait** - Wait for Process Completion

```bash
# Wait for specific PID
wait 1234

# Wait for all background jobs
wait

# Wait with timeout (bash)
timeout 30 wait 1234

# In scripts
process_pid=$!
wait $process_pid
exit_code=$?
```

### Process Priority (Nice Values)

**Nice value:** Controls CPU scheduling priority (-20 to 19)
- **-20:** Highest priority (only root)
- **0:** Default
- **19:** Lowest priority

```bash
# Run command with specific nice value
nice -n 10 ./my_script.sh    # Lower priority
nice -n -10 ./my_script.sh   # Higher priority (needs root)

# Change priority of running process
renice 10 1234               # Make PID 1234 lower priority
renice -5 -u username        # Increase priority of user's processes

# View nice value
ps -o pid,user,nice,comm

# In Batch mode
ps auxN | head -20
```

### System-Wide Process Information

```bash
# Number of running processes
ps aux | wc -l

# Processes by user
ps aux | awk '{print $1}' | sort | uniq -c

# Memory usage by process
ps aux --sort=-%mem | head -20

# CPU usage by process
ps aux --sort=-%cpu | head -20

# Find idle processes (no CPU in 1 hour)
ps axe -o etime,pid,comm | grep [0-9]-

# Zombie processes (zombie detection)
ps aux | grep "[Z]"

# Show load average
uptime
# Load average: 0.5, 0.4, 0.3 (1min, 5min, 15min averages)

# Detailed process info
cat /proc/[PID]/status
cat /proc/[PID]/environ
cat /proc/[PID]/cmdline
```

---

## ✨ Best Practices

### 1. **Process Naming Conventions**
```bash
# Good: Clear, descriptive command lines
/usr/bin/python3 /opt/app/worker.py --config=/etc/app.conf

# Bad: Unclear short names
./worker.py
```

### 2. **Signal Handling**
```bash
# Always use graceful shutdown first
kill -TERM $PID    # Try to stop gracefully
sleep 5
kill -9 $PID       # Force kill if needed
```

### 3. **Process Monitoring**
```bash
# Monitor in background
while true; do
    ps aux | grep "critical_app" | grep -v grep
    sleep 60
done &
```

### 4. **Avoiding Zombie Processes**
```bash
# Bad: Parent doesn't wait for child
#!/bin/bash
/long/running/task &

# Good: Parent waits for child
#!/bin/bash
/long/running/task &
wait $!
exit $?
```

### 5. **Resource Limits**
```bash
# View limits
ulimit -a

# Set max processes for user
ulimit -u 1000

# Set max open files
ulimit -n 4096

# In systemd services
[Service]
LimitNOFILE=65536
LimitNPROC=4096
```

---

## ⚠️ Common Pitfalls

### 1. **Using `kill -9` Immediately**
```bash
# ❌ Bad: Force kill without grace period
kill -9 $PID

# ✅ Good: Allow graceful shutdown
kill $PID
sleep 5
kill -9 $PID  # Only if necessary
```

### 2. **Not Checking if Process Exists**
```bash
# ❌ Bad: Assuming process exists
kill $PID

# ✅ Good: Check first
if ps -p $PID > /dev/null; then
    kill $PID
else
    echo "Process not found"
fi
```

### 3. **Orphaned Child Processes**
```bash
# ❌ Bad: Starting background process without proper handling
./script.sh &

# ✅ Good: Use nohup and handle signals
nohup ./script.sh > /tmp/log.txt 2>&1 &
disown $!  # Disconnect from shell
```

### 4. **Resource Exhaustion**
```bash
# ❌ Bad: No resource limits
while true; do
    some_task &
done

# ✅ Good: Implement limits
max_jobs=10
while true; do
    while [ $(jobs -r | wc -l) -ge $max_jobs ]; do
        sleep 1
    done
    some_task &
done
```

---

## 📝 Hands-On Exercises

### Exercise 1: Process Exploration (Beginner)

**Goal:** Understand the process tree of your system

```bash
# 1. List all processes in tree format
ps -ef --forest

# 2. Find the PID of your shell
echo $$

# 3. Look at your shell's parent process
ps -p $$
ps -p $PPID  # Parent's PID

# 4. Count total processes
ps aux | wc -l

# 5. Find init process
ps -p 1

# Questions to answer:
# - What is the parent of your shell?
# - How many processes are running?
# - What is the PID of systemd?
```

### Exercise 2: Monitoring Processes (Beginner-Intermediate)

**Goal:** Learn to monitor system performance

```bash
#!/bin/bash
# Script to monitor top processes by CPU and memory

echo "=== Top 5 CPU Users ==="
ps aux | sort -k3 -rn | head -6

echo -e "\n=== Top 5 Memory Users ==="
ps aux | sort -k4 -rn | head -6

echo -e "\n=== Load Average ==="
uptime

echo -e "\n=== Process Count ==="
echo "Total: $(ps aux | wc -l)"
echo "Running: $(ps aux | grep -c ' R ')"
echo "Sleeping: $(ps aux | grep -c ' S ')"

# Additional questions:
# - What process uses the most memory?
# - What's the load average?
# - How many processes are running vs sleeping?
```

### Exercise 3: Process Management (Intermediate)

**Goal:** Control and manage processes programmatically

```bash
#!/bin/bash
# Practical process management script

# Start a long-running process
sleep 3600 &
PID=$!
echo "Started process with PID: $PID"

# Monitor the process
while kill -0 $PID 2>/dev/null; do
    echo "Process $PID is still running at $(date)"
    sleep 5
done

echo "Process $PID has terminated"

# Variation: Force kill after timeout
start_timeout_process() {
    "$@" &
    PID=$!
    timeout 10 wait $PID || {
        echo "Process exceeded timeout, killing..."
        kill -9 $PID
    }
}

# Usage: start_timeout_process sleep 3600
```

---

## 🔗 Further Learning

### Related Skills
- **Skill 02:** Filesystem & Storage Management
- **Skill 07:** System Monitoring & Diagnostics
- **Skill 08:** Service Management (systemd)

### Command Reference
- `man ps` - Full ps manual
- `man top` - Full top manual
- `man kill` - Signal handling
- `man proc` - /proc filesystem documentation
- `man 7 signal` - All signal types

### External Resources
- Linux Kernel Documentation
- Linux Process Management - Linux Foundation
- System V IPC - Process communication
- Process management forums and communities

### Advanced Topics
- Process tracing (`strace`, `ltrace`)
- Process profiling
- cgroups (control groups)
- namespaces
- Container process isolation

---

## 📊 Quick Reference Cheat Sheet

```bash
# View processes
ps aux                          # All processes detailed
ps -ef --forest                 # Tree view
pgrep process_name              # Find PID by name
ps -p PID                       # Specific process

# Real-time monitoring
top                             # CPU/memory usage
htop                            # Enhanced top
watch -n 1 'ps aux | grep ...'  # Custom monitoring

# Control processes
kill PID                        # Terminate gracefully
kill -9 PID                     # Force kill
pkill process_name              # Kill by name
bg %1                           # Background job
fg %1                           # Foreground job
jobs                            # List jobs

# Priority
nice -n 10 command              # Run with lower priority
renice 10 PID                   # Change priority
ps -o pid,nice,comm             # Show priorities

# Information
uptime                          # Load average
ps aux | grep -c nginx          # Count matching processes
cat /proc/PID/status            # Detailed process info
```

---

## 🎓 Skill Completion Checklist

Before moving to the next skill, verify you can:

- [ ] List all running processes with `ps`
- [ ] Use `top` and `htop` to monitor system resources
- [ ] Find a process by name using `pgrep`
- [ ] Understand process states (R, S, D, Z, T)
- [ ] Send signals to processes with `kill`
- [ ] Manage process priorities with `nice` and `renice`
- [ ] Control foreground/background processes
- [ ] Explain process hierarchy and parent-child relationships
- [ ] Monitor CPU and memory usage by process
- [ ] Troubleshoot high CPU/memory usage scenarios

**When you can do all of these, move to Skill 02: Filesystem & Storage Management**

---

**Skill Status:** ✅ Ready to Use
**Last Updated:** November 2024
**Difficulty:** Beginner
**Time to Complete:** 6 hours
