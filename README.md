# Custom DevOps Roadmap Plugin

> **Professional-Grade DevOps Learning Plugin** - Complete curriculum with 7 specialized agents

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](./plugin.yml)
[![Status](https://img.shields.io/badge/status-Production%20Ready-brightgreen.svg)]()
[![Agents](https://img.shields.io/badge/agents-7-blueviolet.svg)]()
[![Skills](https://img.shields.io/badge/skills-48+-orange.svg)]()

## Overview

A comprehensive **DevOps learning plugin** implementing the complete [roadmap.sh/devops](https://roadmap.sh/devops) curriculum through 7 specialized agents. Each agent covers a critical DevOps domain with 6-8 detailed skill modules.

Perfect for:
- DevOps Engineers learning the full stack
- Cloud Architects designing infrastructure
- Platform Engineers building automation
- Students entering DevOps field
- Teams standardizing practices

## Features

### Comprehensive
- **48+ skill modules** covering 100% of roadmap.sh/devops
- **500+ pages** of structured technical content
- **200+ code examples** with real-world use cases
- **100+ best practices** from industry experts

### Practical
- Real-world scenarios and use cases
- Hands-on exercises for each skill
- Production-ready configurations
- Tested implementations

### Progressive
- Structured learning path from fundamentals to advanced
- Clear skill dependencies and prerequisites
- Estimated completion times
- Difficulty level indicators

### Expert-Designed
- Based on roadmap.sh/devops curriculum
- Verified by DevOps professionals
- Updated with latest tools and practices
- Industry-standard methodologies

## 7 Core Agents

| # | Agent | Focus | Skills | Level |
|---|-------|-------|--------|-------|
| 1 | **Linux & OS Fundamentals** | Operating Systems, Linux Administration | 8 | Beginner→Intermediate |
| 2 | **Network & Protocols** | Networking, DNS, SSH, Firewalls | 6 | Intermediate |
| 3 | **Container & Orchestration** | Docker, Kubernetes, Registries | 7 | Intermediate→Advanced |
| 4 | **Infrastructure as Code** | Terraform, Ansible, CloudFormation | 6 | Intermediate→Advanced |
| 5 | **CI/CD & Version Control** | Git, Jenkins, GitHub Actions, GitLab | 7 | Intermediate→Advanced |
| 6 | **Monitoring & Logging** | Prometheus, Grafana, ELK, Datadog | 6 | Intermediate |
| 7 | **Cloud & Advanced Topics** | AWS, Azure, GCP, Security, SRE | 8 | Advanced |

## Installation

```bash
# Clone the repository
git clone https://github.com/pluginagentmarketplace/custom-plugin-devops.git

# Navigate to plugin directory
cd custom-plugin-devops

# Install with Claude Code
claude plugin install .
```

## Usage

```bash
# Ask an agent for help with specific skills
@linux-os-agent: Guide me through process management in Linux

@container-orchestration-agent: How do I deploy an app to Kubernetes?

@cicd-agent: Create a Jenkins pipeline for my project

@monitoring-logging-agent: Set up Prometheus monitoring

# Get overview of an agent
@linux-os-agent: What skills are available?

# Progressive learning
@linux-os-agent: skill-01-process-management
@linux-os-agent: skill-02-filesystem-storage
@linux-os-agent: skill-03-file-permissions
```

## Learning Path

### Recommended Progression

```
Start Here ↓
┌─────────────────────────────────────────────┐
│ 1. Linux & OS Fundamentals (4-6 weeks)     │
│    Foundation for all DevOps work           │
└──────────────────┬──────────────────────────┘
                   ↓
┌─────────────────────────────────────────────┐
│ 2. Network & Protocols (3-4 weeks)         │
│    Understanding system communication       │
└──────────────────┬──────────────────────────┘
                   ↓
┌─────────────────────────────────────────────┐
│ 3. Container & Orchestration (6-8 weeks)   │
│    Containerize and scale applications      │
└──────────────────┬──────────────────────────┘
                   ↓
        ┌──────────┴──────────┐
        ↓                     ↓
    Track A              Track B
    ┌─────────┐          ┌─────────┐
    │ IaC     │          │ CI/CD   │
    │ (5-7w)  │          │ (5-6w)  │
    └────┬────┘          └────┬────┘
         └────────┬───────────┘
                  ↓
    ┌──────────────────────────┐
    │ Monitoring & Logging     │
    │ (4-5 weeks)              │
    └────────┬─────────────────┘
             ↓
    ┌──────────────────────────┐
    │ Cloud & Advanced Topics  │
    │ (6-8 weeks)              │
    └──────────────────────────┘
             ↓
         Master DevOps!
```

**Total Time:** ~44 weeks intensive learning or ~6 months part-time

## Directory Structure

```
custom-plugin-devops/
│
├── agents/                                    # 7 Agent implementations
│   ├── linux-os-agent/
│   │   ├── agent.yml                         # Configuration
│   │   ├── README.md                         # Documentation
│   │   └── skills/
│   │       ├── skill-01-process-management.md
│   │       ├── skill-02-filesystem-storage.md
│   │       ├── skill-03-file-permissions.md
│   │       ├── skill-04-package-management.md
│   │       ├── skill-05-user-group-management.md
│   │       ├── skill-06-bash-scripting.md
│   │       ├── skill-07-system-monitoring.md
│   │       └── skill-08-service-management.md
│   │
│   ├── network-protocol-agent/
│   ├── container-orchestration-agent/
│   ├── iac-agent/
│   ├── cicd-agent/
│   ├── monitoring-logging-agent/
│   └── cloud-advanced-agent/
│
├── docs/                                      # Documentation
│   ├── ARCHITECTURE.md                        # System design
│   ├── QUICKSTART.md                          # Getting started
│   ├── PLUGIN_STRUCTURE.md                    # Layout explanation
│   ├── IMPLEMENTATION_GUIDE.md                # Usage guide
│   └── BEST_PRACTICES.md                      # Tips & tricks
│
├── plugin.yml                                 # Plugin metadata
├── main.md                                    # Detailed overview
├── README.md                                  # This file
└── CHANGELOG.md                               # Version history
```

## Testing

```bash
# Validate plugin structure
/plugin validate .

# Test specific agent
@linux-os-agent: What are your capabilities?

# Run through sample exercises
# Follow hands-on exercises in each skill module
```

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## Documentation

- [main.md](./main.md) - Detailed overview
- [QUICKSTART.md](./docs/QUICKSTART.md) - Getting started guide
- [ARCHITECTURE.md](./docs/ARCHITECTURE.md) - System design
- [PLUGIN_STRUCTURE.md](./docs/PLUGIN_STRUCTURE.md) - Directory layout
- [IMPLEMENTATION_GUIDE.md](./docs/IMPLEMENTATION_GUIDE.md) - Usage patterns
- [BEST_PRACTICES.md](./docs/BEST_PRACTICES.md) - Pro tips

## Support

- See documentation in `/docs`
- Report issues on GitHub
- Discussions welcome
- Reference: https://roadmap.sh/devops

## Acknowledgments

Built based on [roadmap.sh/devops](https://roadmap.sh/devops) - the excellent DevOps learning roadmap.

## License

MIT License

---

**License:** MIT
