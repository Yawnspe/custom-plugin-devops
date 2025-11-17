# Linux & OS Fundamentals Agent

> Master the foundation of all DevOps work - Operating Systems, Linux Administration, and System Management

## 🎯 Agent Overview

This agent covers the **essential foundation** for DevOps engineers: **Linux and Operating System fundamentals**. Without solid Linux knowledge, all advanced DevOps tools become significantly harder to understand and debug.

## 📚 What You'll Learn

This agent teaches you to:

- ✅ Understand OS concepts (processes, memory, I/O, filesystems)
- ✅ Navigate and manage Linux filesystems efficiently
- ✅ Monitor system processes and resources in real-time
- ✅ Configure file permissions and access control
- ✅ Manage system packages and software
- ✅ Create and manage users and groups
- ✅ Write production-grade Bash scripts
- ✅ Manage system services with systemd
- ✅ Troubleshoot system issues
- ✅ Optimize system performance

## 🎓 Skill Modules (8)

| # | Skill | Duration | Level | Status |
|---|-------|----------|-------|--------|
| 1 | Process Management & Monitoring | 6h | Beginner | 📖 |
| 2 | Filesystem & Storage Management | 6h | Beginner | 📖 |
| 3 | File Permissions & Access Control | 5h | Beginner | 📖 |
| 4 | Package Management (APT, YUM) | 5h | Beginner | 📖 |
| 5 | User & Group Management | 5h | Intermediate | 📖 |
| 6 | Bash Scripting & Automation | 8h | Intermediate | 📖 |
| 7 | System Monitoring & Diagnostics | 6h | Intermediate | 📖 |
| 8 | Service Management (systemd) | 5h | Intermediate | 📖 |

**Total Estimated Time:** 46-48 hours

## 🏗️ Agent Structure

```
linux-os-agent/
├── agent.yml                                    # Configuration
├── README.md                                    # This file
└── skills/
    ├── skill-01-process-management.md          # Processes, threads, scheduling
    ├── skill-02-filesystem-storage.md          # Filesystems, mounting, partitions
    ├── skill-03-file-permissions.md            # chmod, chown, ACLs
    ├── skill-04-package-management.md          # apt, yum, package management
    ├── skill-05-user-group-management.md       # Users, groups, permissions
    ├── skill-06-bash-scripting.md              # Scripts, automation, functions
    ├── skill-07-system-monitoring.md           # htop, iostat, vmstat, monitoring
    └── skill-08-service-management.md          # systemd, service management
```

## 🚀 Quick Start

### 1. Start with Process Management
```bash
@linux-os-agent: skill-01-process-management
```

### 2. Continue with Filesystem Management
```bash
@linux-os-agent: skill-02-filesystem-storage
```

### 3. Progress Through All Skills
Follow the learning path in the recommended order for optimal understanding.

## 🎯 Learning Outcomes

Upon completing all 8 skills, you will be able to:

### Process Management
- View and understand running processes
- Monitor CPU and memory usage
- Manage process priority
- Kill and restart processes
- Understand process states and transitions

### Filesystem Management
- Navigate the Linux directory structure
- Understand different filesystem types
- Mount and unmount filesystems
- Analyze disk usage
- Manage disk partitions

### Permissions & Access Control
- Understand Unix permission model
- Change file ownership and permissions
- Use symbolic and numeric notation
- Implement access control lists (ACLs)
- Manage default permissions with umask

### Package Management
- Install and remove packages
- Update system packages
- Search for packages
- Manage software repositories
- Work with dependencies

### User & Group Management
- Create and delete users
- Create and manage groups
- Change user passwords
- Modify user group membership
- Configure sudo access

### Bash Scripting
- Write production-grade Bash scripts
- Use variables and arrays
- Implement control structures
- Create reusable functions
- Handle errors and edge cases
- Automate repetitive tasks

### System Monitoring
- Monitor system resources in real-time
- Analyze I/O performance
- Track network usage
- Identify bottlenecks
- Set up performance baselines

### Service Management
- Manage services with systemd
- Enable/disable services at boot
- View service logs
- Create custom systemd services
- Troubleshoot service issues

## 📊 Key Statistics

- **8 Skill Modules** covering all critical Linux topics
- **48+ Hours** of comprehensive content
- **200+ Commands** with practical examples
- **50+ Exercises** for hands-on practice
- **20+ Projects** for real-world application

## 🛠️ Tools & Technologies

- **Linux Distributions:** Ubuntu, CentOS, RHEL, Debian
- **Shells:** Bash, sh
- **Package Managers:** apt, apt-get, yum, dnf
- **Monitoring Tools:** top, htop, ps, systemctl
- **Editors:** nano, vim, vi
- **Scripting:** Bash/shell scripting

## 📖 Skill Module Format

Each skill module includes:

```
📌 Overview & Objectives
📚 Foundational Concepts
🔧 Practical Implementation
  ├── CLI Commands
  ├── Configuration Files
  └── Code Examples
✨ Best Practices
⚠️ Common Pitfalls
📝 Hands-On Exercises
🔗 Further Learning
```

## 🎯 Prerequisites

- **Minimal:** Basic computer literacy
- **Recommended:** Basic command-line familiarity
- **Optional:** Linux installed on your system

## 💡 Tips for Success

1. **Practice Hands-On:** Don't just read - execute every command
2. **Use Virtual Machines:** Set up VMs for experimentation
3. **Follow Best Practices:** Learn industry standards from day one
4. **Troubleshoot Problems:** Try to solve errors yourself first
5. **Review Regularly:** Revisit concepts until they're muscle memory
6. **Read Man Pages:** Use `man <command>` for deeper knowledge
7. **Create a Lab:** Build your own Linux lab environment

## 🔗 Related Skills

- **After completing this agent:**
  - Network & Protocols Agent (networking concepts)
  - CI/CD Agent (automation and scripting)

## 📚 External Resources

- Linux Foundation: https://www.linuxfoundation.org/
- Linux Academy: Linux courses and training
- GNU/Linux Documentation Project
- Manual Pages: `man <command>`

## ❓ Common Questions

**Q: Do I need to use Linux as my daily OS?**
A: Not required, but highly recommended. You can use a virtual machine or WSL2 on Windows.

**Q: What Linux distribution should I learn?**
A: Start with Ubuntu (most user-friendly). RedHat/CentOS is also important for enterprise environments.

**Q: How long does this agent take to complete?**
A: 46-48 hours total, or about 1 week at 40+ hours/week, or 3-4 weeks part-time.

**Q: What's the difference between skill-01 and skill-02?**
A: skill-01 focuses on processes and system resources, while skill-02 focuses on disk storage and filesystems.

## 🎓 Progression Path

```
Beginner (Skills 1-4)
    ↓ [Master basic concepts]
Intermediate (Skills 5-8)
    ↓ [Ready for network & containers]
Next Agent: Network & Protocols
```

## 🏆 Mastery Criteria

You've mastered this agent when you can:

✅ Write complex Bash scripts without reference
✅ Troubleshoot system performance issues
✅ Manage users and permissions effectively
✅ Monitor system resources in real-time
✅ Understand OS concepts deeply
✅ Comfortable with CLI for daily work
✅ Know 100+ common Linux commands
✅ Can explain Linux architecture to others

## 📞 Support

- 📖 Check the skill modules first
- 💬 Review the practical examples
- 🔍 Use `man` pages for command help
- 📚 Refer to external resources
- 🐧 Join Linux communities

## 📝 License

MIT License - Free to use and share

---

**Version:** 1.0.0
**Last Updated:** November 2024
**Status:** ✅ Production Ready
**Difficulty:** Beginner → Intermediate
**Time to Complete:** 46-48 hours
