# Day 1: Linux Fundamentals

## What is Linux?

Linux is an open-source operating system kernel created by **Linus Torvalds** in 1991. It powers everything from personal computers to servers, smartphones (Android), supercomputers, and cloud infrastructure. In the DevOps world, Linux is the dominant OS — virtually all servers, containers, and cloud instances run Linux.

### Why Linux for DevOps?

- **Free and open-source** — no licensing costs, full transparency
- **Stable and secure** — production servers often run for years without reboots
- **Powerful CLI** — automate anything from the command line
- **Ubiquitous in the cloud** — AWS, GCP, Azure all default to Linux
- **Container-native** — Docker, Kubernetes, and most DevOps tools are Linux-first

---

## Linux Distributions

A **distribution** (distro) packages the Linux kernel with software, tools, and a package manager to create a complete OS.

| Family | Distros | Package Manager | Common Use |
|--------|---------|----------------|------------|
| Debian | Ubuntu, Debian, Linux Mint | `apt` | Cloud servers, containers |
| Red Hat | RHEL, CentOS, Rocky, AlmaLinux, Fedora | `dnf` / `yum` | Enterprise servers |
| Arch | Arch Linux, Manjaro | `pacman` | Advanced users, desktop |
| SUSE | openSUSE, SLES | `zypper` | Enterprise, SAP |

> For DevOps, you'll most commonly encounter **Ubuntu** and **RHEL/CentOS-family** distros.

---

## The Shell

The **shell** is a program that takes your commands and passes them to the OS kernel. The most common shell is **Bash** (Bourne Again SHell).

```bash
# Check your current shell
echo $SHELL

# Check available shells
cat /etc/shells

# Check bash version
bash --version
```

### The Prompt

```
tom@ubuntu-server:~$
```

| Part | Meaning |
|------|---------|
| `tom` | Current username |
| `ubuntu-server` | Hostname |
| `~` | Current directory (`~` = home directory) |
| `$` | Normal user prompt (`#` = root user) |

---

## Filesystem Hierarchy Standard (FHS)

Linux organizes files in a single tree starting at `/` (root). Understanding this is essential.

```
/
├── bin/        # Essential user binaries (ls, cp, mv, bash)
├── sbin/       # Essential system binaries (fdisk, ifconfig) — for root
├── etc/        # Configuration files (nginx.conf, hosts, passwd)
├── home/       # User home directories (/home/tom)
├── root/       # Root user's home directory
├── var/        # Variable data: logs, databases, mail (/var/log)
├── tmp/        # Temporary files (cleared on reboot)
├── usr/        # User programs and data
│   ├── bin/    # Non-essential binaries (python3, git)
│   ├── lib/    # Libraries
│   └── local/  # Locally compiled software
├── lib/        # Shared libraries needed by /bin and /sbin
├── opt/        # Optional/third-party software
├── proc/       # Virtual FS — info about running processes (/proc/cpuinfo)
├── sys/        # Virtual FS — info about devices and kernel
├── dev/        # Device files (disks, terminals)
├── mnt/        # Temporary mount points
├── media/      # Removable media mount points (USB, CD)
└── boot/       # Boot files (kernel, GRUB bootloader)
```

### Key Directories for DevOps

```bash
/etc/          # All configuration files live here
/var/log/      # Application and system logs
/var/lib/      # Application state data (e.g., /var/lib/docker)
/tmp/          # Temporary files — safe to delete
/proc/         # Live system info (no actual files on disk)
/usr/local/bin # Where you install custom scripts/binaries
```

---

## Essential Navigation Commands

### Knowing Where You Are

```bash
# Print working directory (current location)
pwd

# Example output:
# /home/tom
```

### Listing Files and Directories

```bash
# List files in current directory
ls

# List with details (permissions, owner, size, date)
ls -l

# List all files including hidden (starting with .)
ls -a

# List all with details, human-readable sizes
ls -lah

# List a specific directory
ls -l /var/log

# Sort by modification time (newest first)
ls -lt

# List with colors and file type indicators
ls --color=auto -F
```

### Navigating Directories

```bash
# Change to a directory
cd /etc

# Go to home directory (all three are equivalent)
cd
cd ~
cd $HOME

# Go up one level (parent directory)
cd ..

# Go up two levels
cd ../..

# Go back to the previous directory
cd -

# Use tab completion — type part of a name and press Tab
cd /var/lo<TAB>   # completes to /var/log/
```

### Absolute vs Relative Paths

```bash
# Absolute path: starts from / (root)
cd /home/tom/projects

# Relative path: relative to your current location
# (if you're already in /home/tom)
cd projects
cd ./projects   # same thing, ./ means current directory
```

---

## Getting Help

```bash
# Manual page for a command (press q to quit)
man ls
man cp
man bash

# Search man pages for a keyword
man -k "copy file"
apropos "copy file"

# Short help for most commands
ls --help
cp --help

# Describe what a command does
whatis ls

# Find where a command lives
which python3
type ls

# Show command history
history
history 20   # last 20 commands
```

---

## Working with Files and Directories (Basics)

You'll go deep on this in Day 2, but here are the foundational commands:

```bash
# Create a directory
mkdir mydir

# Create nested directories
mkdir -p projects/devops/ansible

# Create an empty file
touch myfile.txt

# View file contents
cat myfile.txt

# View long files one page at a time (q to quit, space/b for pages)
less /var/log/syslog
more /var/log/syslog

# View first 10 lines
head myfile.txt

# View last 10 lines
tail myfile.txt

# Follow a log file in real time (Ctrl+C to stop)
tail -f /var/log/syslog

# Copy a file
cp source.txt destination.txt

# Move or rename a file
mv oldname.txt newname.txt
mv file.txt /tmp/

# Remove a file
rm file.txt

# Remove a directory and its contents (be careful!)
rm -rf mydir/

# Print text to terminal
echo "Hello, DevOps!"

# Write text to a file
echo "Hello" > file.txt      # overwrites
echo "World" >> file.txt     # appends
```

---

## Keyboard Shortcuts (Terminal)

These will save you enormous amounts of time:

| Shortcut | Action |
|----------|--------|
| `Ctrl + C` | Kill/interrupt current command |
| `Ctrl + Z` | Suspend current command (bg/fg to manage) |
| `Ctrl + D` | Exit shell / send EOF |
| `Ctrl + L` | Clear the screen (same as `clear`) |
| `Ctrl + A` | Move cursor to beginning of line |
| `Ctrl + E` | Move cursor to end of line |
| `Ctrl + U` | Delete from cursor to beginning of line |
| `Ctrl + K` | Delete from cursor to end of line |
| `Ctrl + W` | Delete the word before the cursor |
| `Ctrl + R` | Reverse search command history |
| `Tab` | Auto-complete commands, files, and paths |
| `↑` / `↓` | Navigate command history |
| `!!` | Repeat last command |
| `!$` | Last argument of previous command |

```bash
# Practical examples
sudo !!           # re-run last command with sudo
cat /etc/hosts
vi !$             # opens /etc/hosts (last argument)
```

---

## Output Redirection & Pipes

One of Linux's most powerful features: connecting commands together.

```bash
# Redirect output to a file (overwrite)
ls -l > filelist.txt

# Redirect output to a file (append)
echo "new line" >> filelist.txt

# Redirect errors to a file
ls /nonexistent 2> errors.txt

# Redirect both stdout and stderr
command > output.txt 2>&1
command &> output.txt   # shorthand

# Pipe: send output of one command as input to another
ls -l | grep ".txt"
cat /etc/passwd | grep "tom"
ps aux | grep nginx

# Count lines in output
ls | wc -l
cat /etc/passwd | wc -l

# Chain with multiple pipes
cat /var/log/syslog | grep "ERROR" | tail -20
```

---

## Basic System Information Commands

```bash
# System and kernel info
uname -a           # all info
uname -r           # kernel version only

# OS release info
cat /etc/os-release
lsb_release -a

# System uptime
uptime

# Current date and time
date
date "+%Y-%m-%d %H:%M:%S"

# Logged-in users
who
w

# CPU info
cat /proc/cpuinfo
lscpu

# Memory info
free -h
cat /proc/meminfo

# Disk usage
df -h              # disk free (all filesystems)
du -sh /var/log/   # disk usage of a directory

# Hostname
hostname
hostname -I        # show IP addresses
```

---

## The `man` Page Deep Dive

Man pages have sections. Understanding them helps:

| Section | Content |
|---------|---------|
| 1 | User commands (ls, cp, grep) |
| 2 | System calls (open, read, write) |
| 3 | Library functions (printf, malloc) |
| 5 | File formats (passwd, fstab) |
| 8 | System administration commands (mount, sysctl) |

```bash
# Specify section number
man 5 passwd    # passwd file format
man 1 passwd    # passwd command

# Search within man page: /search_term then n for next match
man ls
# then type: /color   (to find color-related options)
```

---

## Day 1 Practice Exercises

1. Open a terminal and run `uname -a`. What kernel version are you running?
2. Navigate to `/etc` and list its contents sorted by modification time.
3. Create a directory structure: `~/devops-lab/day1/notes/`
4. Create a file `notes.txt` inside it with the text "Linux is awesome".
5. Use `cat`, `head`, and `tail` to view `/etc/passwd`. How many lines does it have? (hint: `wc -l`)
6. Use a pipe to find all `.conf` files in `/etc` that contain the word "timeout".
7. View the man page for `ls`. Find what the `-R` flag does.
8. Use `Ctrl+R` to search your history for the `uname` command you ran earlier.

---

## Summary

| Concept | Key Takeaway |
|---------|-------------|
| Linux | Open-source OS kernel, dominant in DevOps/cloud |
| Distros | Ubuntu (Debian) and RHEL-family most common |
| Shell | Bash is the standard; `$` = user, `#` = root |
| FHS | Single tree from `/`; learn `/etc`, `/var`, `/home` |
| Navigation | `pwd`, `ls`, `cd`, absolute vs relative paths |
| Help | `man`, `--help`, `whatis`, `apropos` |
| Pipes & Redirection | `|`, `>`, `>>`, `2>`, `&>` — connect and redirect |

---

[Next: Day 2 - File Management & Permissions](../day-2/README.md)
