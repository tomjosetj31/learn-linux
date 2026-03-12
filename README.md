# Learn Linux for DevOps

A comprehensive beginner-to-intermediate guide for learning Linux as a DevOps engineer.

---

## Course Contents

| Day | Topic | Description |
|-----|-------|-------------|
| [Day 1](./day-1/README.md) | **Linux Fundamentals** | History, distros, filesystem hierarchy, shell basics, essential navigation commands |
| [Day 2](./day-2/README.md) | **File Management & Permissions** | Files, directories, permissions, ownership, links, archives, text processing |
| [Day 3](./day-3/README.md) | **Users, Groups & Process Management** | User/group management, sudo, processes, signals, jobs, scheduling |
| [Day 4](./day-4/README.md) | **Networking** | IP addressing, DNS, SSH, firewall, network tools, troubleshooting |
| [Day 5](./day-5/README.md) | **Shell Scripting** | Bash scripting, variables, conditionals, loops, functions, best practices |
| [Day 6](./day-6/README.md) | **System Administration** | Package management, systemd, logging, disk management, performance monitoring |
| [Day 7](./day-7/README.md) | **DevOps on Linux** | Security hardening, cron jobs, environment variables, Linux in CI/CD, Docker basics |

---

## Quick Start

### Prerequisites

- A Linux machine, VM, WSL (Windows Subsystem for Linux), or cloud instance (EC2, GCP, etc.)
- Basic computer literacy
- Curiosity and willingness to use the terminal

### Recommended Distros for Learning

```bash
# Ubuntu (most beginner-friendly, widely used in DevOps)
# Download from: https://ubuntu.com/download/server

# CentOS Stream / Rocky Linux / AlmaLinux (RHEL-compatible, common in enterprise)
# Download from: https://rockylinux.org/

# Debian (stable, widely used in servers and containers)
# Download from: https://www.debian.org/
```

### Your First Command

```bash
# Check your Linux version
uname -a

# Check your distro
cat /etc/os-release

# See who you are
whoami

# See where you are
pwd
```

---

## Repository Structure

```
learn-linux/
├── README.md              # This file
├── cheatsheet.md          # Quick reference for all commands
├── day-1/
│   └── README.md          # Linux Fundamentals
├── day-2/
│   └── README.md          # File Management & Permissions
├── day-3/
│   └── README.md          # Users, Groups & Process Management
├── day-4/
│   └── README.md          # Networking
├── day-5/
│   └── README.md          # Shell Scripting
├── day-6/
│   └── README.md          # System Administration
└── day-7/
    └── README.md          # DevOps on Linux
```

---

## Learning Path

### Beginner (Days 1-2)
- Understand Linux and its role in DevOps
- Navigate the filesystem confidently
- Manage files, permissions, and ownership

### Intermediate (Days 3-4)
- Manage users, processes, and system jobs
- Configure networking and SSH for remote work
- Troubleshoot connectivity issues

### Advanced (Days 5-7)
- Write Bash scripts to automate tasks
- Administer services, logs, and disk storage
- Harden servers and integrate Linux in CI/CD pipelines

---

## Tips for Success

1. **Use the terminal daily** - Muscle memory is key; avoid GUIs when possible
2. **Read man pages** - `man <command>` is your most powerful reference
3. **Break things in a safe lab** - Use VMs or containers so you can reset
4. **Type commands yourself** - Don't copy-paste when learning; typing builds retention
5. **Keep notes** - Build your own cheatsheet as you go

---

## Useful Resources

- [Linux man pages online](https://man7.org/linux/man-pages/)
- [The Linux Command Line (free book)](https://linuxcommand.org/tlcl.php)
- [OverTheWire: Bandit](https://overthewire.org/wargames/bandit/) - Learn Linux via challenges
- [explainshell.com](https://explainshell.com) - Paste any command and get it explained
- [tldr pages](https://tldr.sh/) - Simplified man pages with examples

---

## License

This learning material is free to use and share.

---

Happy Hacking!
