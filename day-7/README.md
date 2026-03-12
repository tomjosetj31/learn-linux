# Day 7: DevOps on Linux

## Environment Variables

### What Are Environment Variables?

Environment variables are key-value pairs available to all processes running in a shell session. They configure software behavior and pass configuration between scripts and applications.

```bash
# View all environment variables
env
printenv

# View a specific variable
echo $HOME
echo $PATH
echo $USER
printenv HOME

# Common built-in variables
echo $HOME        # current user's home directory
echo $USER        # current username
echo $SHELL       # current shell
echo $PATH        # directories searched for executables
echo $PWD         # current directory
echo $HOSTNAME    # machine hostname
echo $LANG        # locale/language
echo $TERM        # terminal type
echo $EDITOR      # default text editor
echo $PS1         # shell prompt string
```

### Setting Environment Variables

```bash
# Set for current shell session only
export MY_VAR="hello"
export DB_HOST="localhost"
export DB_PORT=5432

# Set temporarily for a single command
DB_HOST=prod.db.example.com ./deploy.sh
NODE_ENV=production node app.js

# Make permanent (persist across sessions)
# Add to ~/.bashrc (interactive shells)
echo 'export MY_VAR="hello"' >> ~/.bashrc
source ~/.bashrc     # reload immediately

# ~/.bash_profile: login shells (SSH sessions)
# ~/.bashrc: interactive non-login shells
# /etc/environment: system-wide (all users, no export needed)
# /etc/profile.d/*.sh: system-wide scripts

# Unset a variable
unset MY_VAR
```

### The PATH Variable

```bash
# View PATH
echo $PATH
# /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# Add directory to PATH
export PATH="$PATH:/opt/myapp/bin"
export PATH="/usr/local/go/bin:$PATH"    # prepend (search this first)

# Make permanent
echo 'export PATH="$PATH:/opt/myapp/bin"' >> ~/.bashrc

# See which binary will be used
which python3
type python3

# View all locations of a command
whereis python3
```

### `.env` Files (12-Factor App Pattern)

```bash
# .env file — store environment-specific config
# .env
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp
DB_USER=myapp
DB_PASS=secretpassword
API_KEY=abc123
NODE_ENV=production

# Load .env in bash
set -a           # auto-export all variables
source .env
set +a

# Or with a one-liner
export $(grep -v '^#' .env | xargs)

# NEVER commit .env to git!
echo ".env" >> .gitignore
echo ".env.*" >> .gitignore

# Use .env.example as a template (safe to commit)
cp .env.example .env
```

---

## Security Hardening

### User and Access Security

```bash
# Audit users with no password
sudo awk -F: '($2 == "" ) {print $1}' /etc/shadow

# Find users with UID 0 (should only be root)
awk -F: '($3 == 0) {print $1}' /etc/passwd

# Lock root account (rely on sudo instead)
sudo passwd -l root

# Check for users with login shells who shouldn't have them
grep -v "nologin\|false" /etc/passwd | grep -v "^root"

# Ensure SSH directory permissions are correct
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Audit sudoers
sudo cat /etc/sudoers
ls -la /etc/sudoers.d/

# Set password aging policies
sudo chage -l tom              # view current policy
sudo chage -M 90 tom           # max password age: 90 days
sudo chage -W 7 tom            # warn 7 days before expiry
sudo chage -m 7 tom            # minimum 7 days between changes
```

### File and System Security

```bash
# Find world-writable files (security risk)
find / -type f -perm -o+w 2>/dev/null | grep -v proc

# Find SUID/SGID files (potential privilege escalation)
find / -type f \( -perm -4000 -o -perm -2000 \) 2>/dev/null

# Find files with no owner
find / -nouser -o -nogroup 2>/dev/null

# Check for open world-writable directories
find / -type d -perm -o+w 2>/dev/null | grep -v proc | grep -v sys

# Disable unnecessary SUID bits
sudo chmod u-s /usr/bin/some_program

# Verify important file checksums
sha256sum /usr/bin/sudo
sha256sum /bin/bash

# Check for recently modified system files
find /bin /sbin /usr/bin /usr/sbin -newer /var/log/dpkg.log 2>/dev/null

# Restrict core dumps (can expose sensitive data)
echo "* hard core 0" | sudo tee -a /etc/security/limits.conf
echo "fs.suid_dumpable = 0" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### SSH Hardening (Recap + More)

```bash
# /etc/ssh/sshd_config hardening checklist:
Port 2222                    # Non-standard port
PermitRootLogin no           # No root login
PasswordAuthentication no    # Key-only
MaxAuthTries 3               # Limit attempts
MaxSessions 5                # Limit sessions
AllowUsers tom deploy        # Whitelist users
Protocol 2                   # SSH protocol 2 only
X11Forwarding no             # Disable X11 if not needed
PrintLastLog yes             # Show last login
Banner /etc/ssh/banner.txt   # Legal warning banner

# Create login banner
sudo bash -c 'cat > /etc/ssh/banner.txt << EOF
*****************************************************
* Authorized access only. Activity is monitored.   *
* Unauthorized access will be prosecuted.           *
*****************************************************
EOF'

# Apply and test
sudo sshd -t                        # test config
sudo systemctl reload sshd
```

### fail2ban — Brute Force Protection

```bash
# Install
sudo apt install fail2ban           # Debian/Ubuntu
sudo dnf install fail2ban           # RHEL

# Configure
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

# Key settings in jail.local:
[DEFAULT]
bantime  = 3600      # ban for 1 hour
findtime = 600       # within 10 minutes
maxretry = 5         # after 5 failures

[sshd]
enabled = true
port    = ssh
logpath = %(sshd_log)s

# Enable and start
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# View banned IPs
sudo fail2ban-client status
sudo fail2ban-client status sshd

# Unban an IP
sudo fail2ban-client set sshd unbanip 192.168.1.100
```

### Kernel Security Parameters (sysctl)

```bash
# View all kernel parameters
sysctl -a

# View specific parameter
sysctl net.ipv4.ip_forward

# Set temporarily
sudo sysctl -w net.ipv4.ip_forward=1

# Persist in /etc/sysctl.conf or /etc/sysctl.d/
sudo cat >> /etc/sysctl.d/99-security.conf << EOF
# Disable IP forwarding (unless this is a router)
net.ipv4.ip_forward = 0

# Disable ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0

# Enable SYN cookies (SYN flood protection)
net.ipv4.tcp_syncookies = 1

# Disable source routing
net.ipv4.conf.all.accept_source_route = 0

# Ignore ping broadcasts
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Randomize address space (ASLR)
kernel.randomize_va_space = 2
EOF

sudo sysctl -p /etc/sysctl.d/99-security.conf
```

---

## Linux in CI/CD Pipelines

### Common DevOps Tasks in Linux

```bash
# Health check endpoint (used in pipelines)
check_health() {
    local url=$1
    local max_attempts=${2:-30}
    local attempt=0

    echo "Waiting for $url to be healthy..."
    until curl -sf "$url/health" | grep -q '"status":"ok"'; do
        ((attempt++))
        if [ $attempt -ge $max_attempts ]; then
            echo "Health check failed after $max_attempts attempts"
            return 1
        fi
        echo "Attempt $attempt/$max_attempts — retrying in 5s..."
        sleep 5
    done
    echo "Service is healthy!"
}

# Blue-green deployment symlink switch
DEPLOY_DIR="/opt/myapp"
NEW_VERSION="/opt/releases/myapp-1.2.3"
CURRENT_LINK="$DEPLOY_DIR/current"

ln -sfn "$NEW_VERSION" "$CURRENT_LINK"
systemctl restart myapp

# Rollback
PREVIOUS_VERSION="/opt/releases/myapp-1.2.2"
ln -sfn "$PREVIOUS_VERSION" "$CURRENT_LINK"
systemctl restart myapp
```

### Deployment Script Template

```bash
#!/usr/bin/env bash
set -euo pipefail

# Configuration
APP_NAME="myapp"
DEPLOY_USER="deploy"
DEPLOY_DIR="/opt/${APP_NAME}"
RELEASES_DIR="${DEPLOY_DIR}/releases"
CURRENT_LINK="${DEPLOY_DIR}/current"
KEEP_RELEASES=5
GIT_REPO="https://github.com/example/myapp.git"

log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*"; }
error() { log "ERROR: $*" >&2; exit 1; }

# Validate we are the deploy user
[[ "$(whoami)" == "$DEPLOY_USER" ]] || error "Must run as $DEPLOY_USER"

# Create release directory
RELEASE_DIR="${RELEASES_DIR}/$(date +%Y%m%d%H%M%S)"
mkdir -p "$RELEASE_DIR"

# Pull code
log "Cloning repository..."
git clone --depth 1 "$GIT_REPO" "$RELEASE_DIR"

# Install dependencies (example: Node.js)
log "Installing dependencies..."
cd "$RELEASE_DIR"
npm ci --production

# Run database migrations
log "Running migrations..."
npm run migrate

# Symlink shared files (e.g., .env, uploads)
ln -s "${DEPLOY_DIR}/shared/.env" "${RELEASE_DIR}/.env"
ln -s "${DEPLOY_DIR}/shared/uploads" "${RELEASE_DIR}/uploads"

# Switch to new release
log "Switching to new release..."
ln -sfn "$RELEASE_DIR" "$CURRENT_LINK"

# Restart service
log "Restarting application..."
sudo systemctl restart "$APP_NAME"

# Health check
log "Running health check..."
sleep 5
curl -sf "http://localhost:8080/health" | grep -q '"status":"ok"' || error "Health check failed!"

# Cleanup old releases
log "Cleaning up old releases..."
ls -dt "${RELEASES_DIR}"/* | tail -n +$((KEEP_RELEASES + 1)) | xargs rm -rf

log "Deployment complete! Running: ${RELEASE_DIR}"
```

---

## Linux and Docker Basics

### Docker on Linux

```bash
# Install Docker (Ubuntu)
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker

# Add user to docker group (avoid sudo)
sudo usermod -aG docker $USER
newgrp docker   # apply without logging out

# Key Linux-Docker interactions
docker run -d --name nginx -p 80:80 nginx

# Containers use Linux namespaces and cgroups under the hood
# View container processes on the host
ps aux | grep containerd
ls /proc/$(pgrep containerd)/ns/   # namespaces

# Docker uses iptables rules
sudo iptables -L -n | grep DOCKER

# Container logs go to journald
sudo journalctl -u docker
docker logs nginx

# Resource limits (cgroups)
docker run -d --cpus 0.5 --memory 256m nginx

# Volumes are just directories on the host
docker volume inspect myvolume
ls /var/lib/docker/volumes/
```

### Container Troubleshooting on Linux

```bash
# Check container resource usage
docker stats

# Inspect a container's network
docker inspect nginx | grep -A 20 '"NetworkSettings"'

# Enter a running container
docker exec -it nginx /bin/bash
docker exec -it nginx sh    # if bash not available

# Check which ports are in use
sudo ss -tulnp | grep docker
sudo ss -tulnp | grep :80

# Docker disk usage
docker system df
docker system prune -a    # CAREFUL: removes unused images, containers, volumes

# Logs with timestamps
docker logs --timestamps --tail 100 nginx
docker logs -f nginx   # follow
```

---

## System Auditing and Compliance

```bash
# auditd — Linux Audit Framework
sudo apt install auditd     # Debian/Ubuntu
sudo dnf install audit      # RHEL

# Start and enable
sudo systemctl enable --now auditd

# Audit rules
sudo auditctl -l             # list current rules
sudo auditctl -w /etc/passwd -p wa -k user-modify  # watch passwd for write/attr
sudo auditctl -w /etc/sudoers -p wa -k sudo-changes
sudo auditctl -a always,exit -F arch=b64 -S execve -k exec-track  # log all exec

# View audit logs
sudo ausearch -k user-modify
sudo ausearch -k user-modify --start today
sudo aureport --summary      # summary report
sudo aureport --auth         # authentication report

# Persistent rules (survive reboot)
# Add to /etc/audit/rules.d/audit.rules:
-w /etc/passwd -p wa -k user-modify
-w /etc/sudoers -p wa -k sudo-changes
-w /var/log/auth.log -p wa -k auth-log
```

### CIS Benchmark Checks (Quick Manual)

```bash
# Check for world-readable /etc/shadow (should show 640 or 000)
stat /etc/shadow

# Ensure /tmp is mounted with noexec
mount | grep /tmp

# Check for unneeded services
systemctl list-units --type=service --state=active

# Ensure SSH root login disabled
grep PermitRootLogin /etc/ssh/sshd_config

# Check password policies
grep PASS_MAX_DAYS /etc/login.defs
grep PASS_MIN_DAYS /etc/login.defs
grep PASS_WARN_AGE /etc/login.defs

# List listening services (minimize attack surface)
sudo ss -tulnp

# Use lynis for automated auditing
sudo apt install lynis
sudo lynis audit system
```

---

## Useful Linux Aliases and Shell Customization

### Power Aliases for DevOps

```bash
# Add to ~/.bashrc

# Navigation
alias ..='cd ..'
alias ...='cd ../..'
alias ll='ls -lah --color=auto'
alias la='ls -a --color=auto'

# Safety nets
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# System
alias df='df -h'
alias du='du -h'
alias free='free -h'
alias top='htop'

# Networking
alias ports='ss -tulnp'
alias myip='curl -s https://api.ipify.org && echo'
alias localip='hostname -I | awk "{print \$1}"'

# Logs
alias logs='sudo journalctl -f'
alias syslog='sudo tail -f /var/log/syslog'

# Docker
alias dps='docker ps'
alias dpsa='docker ps -a'
alias dlog='docker logs -f'
alias dexec='docker exec -it'

# Git
alias gs='git status'
alias gl='git log --oneline -10'
alias gp='git pull'
alias gd='git diff'

# Reload bashrc
alias reload='source ~/.bashrc && echo "bashrc reloaded"'

# Process
alias psmem='ps aux --sort=-%mem | head'
alias pscpu='ps aux --sort=-%cpu | head'
```

### Useful Shell Functions

```bash
# Add to ~/.bashrc

# Extract any archive
extract() {
    case "$1" in
        *.tar.gz|*.tgz)    tar -xzf "$1" ;;
        *.tar.bz2|*.tbz2)  tar -xjf "$1" ;;
        *.tar.xz)           tar -xJf "$1" ;;
        *.tar)              tar -xf "$1"  ;;
        *.gz)               gunzip "$1"   ;;
        *.bz2)              bunzip2 "$1"  ;;
        *.zip)              unzip "$1"    ;;
        *.rar)              unrar x "$1"  ;;
        *)                  echo "Cannot extract: $1" ;;
    esac
}

# cd into directory and list
cdl() {
    cd "$1" && ls -la
}

# Create directory and cd into it
mkcd() {
    mkdir -p "$1" && cd "$1"
}

# Check if a port is open
port_check() {
    nc -zv "$1" "$2" 2>&1 | grep -q succeeded && echo "Port $2 OPEN" || echo "Port $2 CLOSED"
}

# Quick backup of a file
backup() {
    cp "$1" "${1}.bak.$(date +%Y%m%d%H%M%S)"
}

# Repeat a command N times
repeat() {
    n=$1; shift
    for _ in $(seq "$n"); do "$@"; done
}

# Wait for a service to be ready
wait_for() {
    local host=$1 port=$2 timeout=${3:-30}
    local start=$SECONDS
    until nc -z "$host" "$port" 2>/dev/null; do
        [[ $((SECONDS - start)) -gt $timeout ]] && { echo "Timeout!"; return 1; }
        sleep 1
    done
    echo "$host:$port is ready"
}
```

---

## Linux Performance Tuning for DevOps

### Open File Limits

```bash
# Check current limits
ulimit -a
ulimit -n        # open file limit (important for high-traffic servers)

# Increase for current session
ulimit -n 65536

# Permanent increase for a user
echo "tom soft nofile 65536" | sudo tee -a /etc/security/limits.conf
echo "tom hard nofile 65536" | sudo tee -a /etc/security/limits.conf

# For services (systemd)
# Add to [Service] section:
LimitNOFILE=65536

# Check system-wide limit
cat /proc/sys/fs/file-max
sudo sysctl -w fs.file-max=2097152
```

### TCP Tuning (Web Servers)

```bash
# /etc/sysctl.d/99-tuning.conf
# Increase connection queue
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535

# Reuse TIME_WAIT sockets
net.ipv4.tcp_tw_reuse = 1

# Reduce TIME_WAIT duration
net.ipv4.tcp_fin_timeout = 15

# Increase port range
net.ipv4.ip_local_port_range = 1024 65535

# Increase TCP buffer sizes for high throughput
net.core.rmem_max = 134217728
net.core.wmem_max = 134217728
net.ipv4.tcp_rmem = 4096 87380 134217728
net.ipv4.tcp_wmem = 4096 65536 134217728

sudo sysctl -p /etc/sysctl.d/99-tuning.conf
```

---

## Day 7 Practice Exercises

1. Create a `.env` file with 5 variables. Write a script that loads them and prints each with its value.
2. Audit your system for SUID files. Review the list — are any unexpected?
3. Install and configure `fail2ban` to protect SSH. Simulate failed logins and verify the ban.
4. Write a deployment script that: clones a repo, creates a symlink to `current`, and restarts a service.
5. Add 10 useful aliases to `~/.bashrc`. Reload and test them.
6. Write a `wait_for` function that waits for `localhost:80` to be available before continuing.
7. Set open file limits to 65536 for your user both temporarily and permanently.
8. Use `auditctl` to monitor `/etc/passwd` for changes. Make a change, then check `ausearch`.
9. Harden your SSH config: disable password auth, disable root login, change port. Test it still works.
10. Write a complete health check script that checks: disk usage < 80%, memory usage < 90%, and 3 services are running. Exit 1 with a descriptive message on any failure.

---

## Week 1 Complete — What's Next?

You've completed the 7-day Linux for DevOps curriculum. Here's what to explore next:

### Recommended Next Steps

| Topic | Why |
|-------|-----|
| **Docker** | Containers run on Linux; deep knowledge unlocks debugging |
| **Kubernetes** | Orchestrates Linux containers at scale |
| **Ansible** | Automates Linux configuration at scale |
| **Terraform** | Provision Linux infrastructure on cloud |
| **Prometheus + Grafana** | Monitor Linux systems and services |
| **Nginx / Apache** | Web servers running on Linux |
| **PostgreSQL / MySQL** | Databases on Linux |

### Linux Skills Checklist

- [ ] Navigate filesystem and manipulate files confidently
- [ ] Set and understand file permissions and ownership
- [ ] Manage users, groups, and sudo access
- [ ] Monitor and manage processes
- [ ] Diagnose and fix network connectivity issues
- [ ] Write Bash scripts with error handling
- [ ] Manage services with systemd
- [ ] Read and analyze logs with journalctl and grep
- [ ] Manage disk space and partitions
- [ ] Harden a Linux server for production use

---

## Summary

| Concept | Key Takeaway |
|---------|-------------|
| Environment variables | `export VAR=value`, `.env` files, `~/.bashrc` |
| PATH | Controls where executables are found; add dirs with `$PATH:newdir` |
| Security hardening | Disable root SSH, use keys, fail2ban, audit SUID files |
| sysctl | Kernel parameters for security and performance tuning |
| CI/CD on Linux | Deployment scripts, blue-green via symlinks, health checks |
| Docker on Linux | Namespaces, cgroups, iptables — it's all Linux under the hood |
| Aliases & functions | Customize shell for productivity; store in `~/.bashrc` |
| Audit | `auditd` / `auditctl` for compliance and intrusion detection |
| Limits | `ulimit`, `/etc/security/limits.conf`, systemd `LimitNOFILE` |

---

[Previous: Day 6](../day-6/README.md) | [Back to Start](../README.md)
