# Day 3: Users, Groups & Process Management

## User Management

### Understanding Linux Users

Linux is a multi-user system. Every file, process, and action is tied to a user.

```bash
# View current user
whoami

# View user ID and group memberships
id
id tom

# View all logged-in users
who
w              # more detailed: uptime, what command they're running

# View login history
last
last tom       # login history for specific user
lastb          # failed login attempts (as root)
```

### User Account Files

```bash
# /etc/passwd — user account info (NOT passwords)
cat /etc/passwd
# Format: username:x:UID:GID:comment:home:shell
# Example: tom:x:1000:1000:Tom DevOps:/home/tom:/bin/bash

# /etc/shadow — encrypted passwords (readable by root only)
sudo cat /etc/shadow
# Format: username:$6$hash...:lastchange:min:max:warn:inactive:expire

# /etc/group — group definitions
cat /etc/group
# Format: groupname:x:GID:member1,member2
```

### Creating and Managing Users

```bash
# Create a new user (Debian/Ubuntu)
sudo adduser tom              # interactive, creates home dir, sets password

# Create user with options (lower-level)
sudo useradd -m -s /bin/bash -c "Tom DevOps" tom
# -m: create home directory
# -s: set login shell
# -c: comment/description

# Create system user (no home dir, for services)
sudo useradd --system --no-create-home --shell /usr/sbin/nologin nginx

# Set or change password
sudo passwd tom
sudo passwd root

# Change password as yourself
passwd

# Lock a user account
sudo passwd -l tom
sudo usermod -L tom     # same thing

# Unlock a user account
sudo passwd -u tom
sudo usermod -U tom

# Expire a user account (force password reset on next login)
sudo passwd -e tom

# Delete a user
sudo userdel tom             # keep home directory
sudo userdel -r tom          # delete home directory too

# Modify user properties
sudo usermod -s /bin/bash tom            # change shell
sudo usermod -d /new/home tom            # change home directory
sudo usermod -c "New Comment" tom        # update comment
sudo usermod -aG docker tom              # add to group (IMPORTANT: use -a!)
sudo usermod -l newname oldname          # rename user
```

### Creating and Managing Groups

```bash
# Create a group
sudo groupadd devops
sudo groupadd -g 2000 devops   # specify GID

# Delete a group
sudo groupdel devops

# Add user to group
sudo gpasswd -a tom devops
sudo usermod -aG devops tom   # -a = append (don't remove existing groups!)

# Remove user from group
sudo gpasswd -d tom devops

# View group members
getent group devops
grep "devops" /etc/group

# See your groups
groups
groups tom

# Change your active group for this session
newgrp docker    # switch primary group to docker
```

---

## sudo — Privilege Escalation

### Using sudo

```bash
# Run a single command as root
sudo apt update
sudo systemctl restart nginx

# Open a root shell (use sparingly!)
sudo -i        # login shell (loads root environment)
sudo -s        # current shell (inherits environment)
sudo su -      # switch to root user

# Run command as a different user
sudo -u postgres psql
sudo -u www-data ls /var/www/

# Check your sudo privileges
sudo -l

# Run last command with sudo (common pattern)
sudo !!
```

### Configuring sudo (`/etc/sudoers`)

```bash
# Always edit sudoers with visudo (prevents syntax errors)
sudo visudo

# Basic syntax:
# user  host=(runas)  command
# %group  host=(runas)  command

# Allow tom to run all commands
tom ALL=(ALL:ALL) ALL

# Allow tom to run without password
tom ALL=(ALL) NOPASSWD: ALL

# Allow devops group to run systemctl
%devops ALL=(ALL) /usr/bin/systemctl

# Allow specific commands only
tom ALL=(ALL) /usr/bin/apt, /usr/bin/systemctl restart nginx

# Drop-in sudoers files (preferred — safer than editing main file)
sudo visudo -f /etc/sudoers.d/devops-team
```

---

## Process Management

### Understanding Processes

Every running program is a process. Each has:
- **PID** — Process ID (unique number)
- **PPID** — Parent Process ID
- **UID** — User who owns/started it
- **State** — R(running), S(sleeping), Z(zombie), D(uninterruptible), T(stopped)

```bash
# View your own processes
ps

# View all processes (detailed)
ps aux
# a = all users, u = user-readable format, x = include processes without terminal

# Column meaning in ps aux:
# USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND
# VSZ = virtual memory size, RSS = resident (physical) memory

# View process tree
ps auxf
pstree
pstree -p   # with PIDs

# Filter by name
ps aux | grep nginx
ps aux | grep -v grep | grep nginx   # exclude the grep itself

# Find process by name
pgrep nginx
pgrep -l nginx          # include process name
pgrep -u tom bash       # processes by user
```

### `top` and `htop` — Interactive Process Viewers

```bash
# top: built-in process monitor
top

# Key commands in top:
# q     — quit
# k     — kill a process (enter PID)
# r     — renice (change priority)
# f     — select fields to display
# P     — sort by CPU
# M     — sort by Memory
# 1     — toggle per-CPU stats
# h     — help

# htop: more user-friendly (install first)
sudo apt install htop   # Ubuntu
sudo dnf install htop   # RHEL

htop
# Use arrow keys, F2=setup, F3=search, F9=kill, F10=quit
```

### Killing Processes

```bash
# Send signal to process (default is SIGTERM = 15)
kill PID
kill 1234

# Force kill (SIGKILL = 9 — cannot be caught or ignored)
kill -9 1234
kill -SIGKILL 1234

# Kill by name
killall nginx
pkill nginx             # same, more flexible
pkill -9 nginx
pkill -u tom bash       # kill all bash sessions by user

# Graceful reload (SIGHUP = 1 — many services reload config)
kill -HUP 1234
kill -1 1234
killall -HUP nginx

# Common signals
# SIGTERM (15) — graceful termination (default)
# SIGKILL (9)  — force kill (uncatchable)
# SIGHUP  (1)  — hangup / reload config
# SIGINT  (2)  — interrupt (same as Ctrl+C)
# SIGSTOP (19) — pause process
# SIGCONT (18) — continue paused process

# List all signals
kill -l
```

### Process Priority (Nice)

```bash
# Run command with lower priority (niceness: -20 highest, 19 lowest)
nice -n 10 ./long-running-script.sh

# Change priority of running process
renice 10 -p 1234          # by PID
renice 5 -u tom            # all processes by user
renice -5 -p 1234          # higher priority (needs root)

# View priority in top/ps
ps -o pid,ni,comm -p 1234   # ni = niceness
```

---

## Job Control

### Background and Foreground Jobs

```bash
# Run command in background with &
./long-script.sh &
sleep 300 &

# View background jobs
jobs
jobs -l    # with PIDs

# Bring job to foreground
fg
fg %1      # bring job 1 to foreground
fg %2      # bring job 2

# Send foreground job to background
# (first press Ctrl+Z to suspend, then bg)
Ctrl+Z     # suspend current job
bg         # resume in background
bg %1      # resume job 1 in background

# Disown a job (keep running after you logout)
./script.sh &
disown %1
disown -a  # disown all jobs
```

### `nohup` — Run After Logout

```bash
# Run command immune to hangup signal (survives logout)
nohup ./deploy.sh &
nohup ./deploy.sh > deploy.log 2>&1 &

# Output goes to nohup.out by default (redirect to capture)
nohup python3 server.py > server.log 2>&1 &

# Check it's running
ps aux | grep server.py
```

### `screen` and `tmux` — Terminal Multiplexers

```bash
# screen — run persistent sessions
screen                      # start new session
screen -S mysession         # named session
screen -ls                  # list sessions
screen -r                   # reattach
screen -r mysession         # reattach by name
# Inside screen: Ctrl+A then D to detach

# tmux — modern, recommended
tmux                        # start new session
tmux new -s deploy          # named session
tmux ls                     # list sessions
tmux attach -t deploy       # reattach
tmux kill-session -t deploy # kill session
# Inside tmux: Ctrl+B then D to detach
```

---

## System Resource Monitoring

### Memory

```bash
# Human-readable memory usage
free -h

# Example output:
#               total   used    free    shared  buff/cache  available
# Mem:          7.7Gi   1.8Gi   3.2Gi   256Mi   2.7Gi       5.4Gi
# Swap:         2.0Gi   0B      2.0Gi

# Key: "available" is what actually can be used (not just "free")
# buff/cache can be freed if needed

# Memory info details
cat /proc/meminfo
```

### CPU

```bash
# CPU info
lscpu                       # CPU architecture info
cat /proc/cpuinfo           # detailed per-core info
nproc                       # number of processing units

# Load average
uptime
# Load average: 0.05, 0.10, 0.15  (1min, 5min, 15min)
# Healthy: values < number of CPU cores
# High load: values consistently > number of CPU cores
```

### Disk I/O

```bash
# Disk usage
df -h                       # filesystem space usage
df -h /var/log              # specific path

# Directory disk usage
du -sh /var/log/            # total size of directory
du -sh /var/log/*           # size of each item
du -h --max-depth=1 /var/   # one level deep

# I/O stats (requires sysstat)
sudo apt install sysstat
iostat -h
iostat -h 2 5    # every 2 seconds, 5 times

# Real-time disk I/O
iotop             # interactive (like top for I/O)
```

---

## Scheduling Tasks with `cron`

### Crontab Syntax

```
* * * * * command
│ │ │ │ │
│ │ │ │ └── Day of week (0-7, 0 and 7 = Sunday)
│ │ │ └──── Month (1-12)
│ │ └────── Day of month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)
```

Special characters:
- `*` = any value
- `,` = list of values: `1,3,5`
- `-` = range: `1-5`
- `/` = step: `*/5` (every 5)

### Using Crontab

```bash
# Edit your crontab
crontab -e

# List your crontab
crontab -l

# Remove your crontab
crontab -r

# Edit another user's crontab (as root)
sudo crontab -u tom -e

# Common examples:
# Run at midnight every day
0 0 * * * /opt/scripts/backup.sh

# Run every 5 minutes
*/5 * * * * /opt/scripts/healthcheck.sh

# Run at 2 AM on Sundays
0 2 * * 0 /opt/scripts/weekly-cleanup.sh

# Run at 9 AM on weekdays (Mon-Fri)
0 9 * * 1-5 /opt/scripts/report.sh

# Run on the 1st of every month at midnight
0 0 1 * * /opt/scripts/monthly-report.sh

# Best practice: redirect output for debugging
*/5 * * * * /opt/scripts/healthcheck.sh >> /var/log/healthcheck.log 2>&1
```

### System-wide Cron

```bash
# System cron files
ls /etc/cron.d/         # drop-in cron files
ls /etc/cron.daily/     # scripts run daily
ls /etc/cron.weekly/    # scripts run weekly
ls /etc/cron.monthly/   # scripts run monthly
cat /etc/crontab        # system crontab (includes user field)

# Drop-in format (/etc/cron.d/) — includes user field:
# * * * * * root /usr/local/bin/cleanup.sh
```

---

## Day 3 Practice Exercises

1. Create a user `devuser` with home directory and bash shell. Set a password.
2. Create a group `devteam` and add `devuser` to it.
3. Grant `devuser` passwordless sudo access to run `systemctl` only.
4. Run `sleep 600` in the background. Find its PID with `pgrep`. Then kill it.
5. Use `ps aux` to find the top 5 memory-consuming processes.
6. Start `top`, identify the highest CPU process, then quit.
7. Use `nohup` to run a script that writes the date to a file every 10 seconds.
8. Create a crontab entry that logs disk usage to `/tmp/disk.log` every minute.
9. Use `du -sh` to find the 5 largest directories under `/var`.
10. Lock the `devuser` account, try to su to it, then unlock it.

---

## Summary

| Concept | Key Commands |
|---------|-------------|
| User info | `whoami`, `id`, `who`, `w`, `last` |
| Create users | `adduser`, `useradd -m -s /bin/bash` |
| Modify users | `usermod -aG group user`, `passwd`, `userdel -r` |
| Groups | `groupadd`, `gpasswd -a user group` |
| sudo | `sudo command`, `sudo -l`, `visudo` |
| View processes | `ps aux`, `top`, `htop`, `pstree` |
| Kill processes | `kill PID`, `kill -9`, `pkill name` |
| Job control | `command &`, `jobs`, `fg`, `bg`, `nohup` |
| Resource monitoring | `free -h`, `uptime`, `df -h`, `du -sh` |
| Cron | `crontab -e`, `crontab -l`, `/etc/cron.d/` |

---

[Previous: Day 2](../day-2/README.md) | [Next: Day 4 - Networking](../day-4/README.md)
