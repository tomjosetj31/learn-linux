# Day 6: System Administration

## Package Management

### APT — Debian/Ubuntu

```bash
# Update package index (always do this first)
sudo apt update

# Upgrade all installed packages
sudo apt upgrade
sudo apt full-upgrade      # may remove packages (smarter)
sudo apt dist-upgrade      # handle changing dependencies

# Install package
sudo apt install nginx
sudo apt install -y nginx   # auto-confirm (-y)
sudo apt install nginx=1.18.0  # specific version

# Remove package
sudo apt remove nginx           # remove but keep config files
sudo apt purge nginx            # remove including config files
sudo apt autoremove             # remove unused dependencies

# Search for packages
apt search nginx
apt-cache search nginx

# Show package info
apt show nginx
apt-cache showpkg nginx

# List installed packages
dpkg -l
dpkg -l | grep nginx
apt list --installed
apt list --installed | grep nginx

# Check which package provides a file
dpkg -S /usr/bin/python3
apt-file search python3

# Download package without installing
apt download nginx

# Inspect a .deb package
dpkg -c nginx.deb    # list contents
dpkg -I nginx.deb    # show info

# Install a .deb file
sudo dpkg -i package.deb
sudo apt install -f    # fix missing dependencies after dpkg install

# Hold package (prevent upgrades)
sudo apt-mark hold nginx
sudo apt-mark unhold nginx
apt-mark showhold     # list held packages

# Clean cache
sudo apt clean        # remove all cached packages
sudo apt autoclean    # remove only outdated cached packages
```

### DNF/YUM — RHEL/CentOS/Rocky

```bash
# Update package index and upgrade
sudo dnf update
sudo dnf upgrade

# Install package
sudo dnf install nginx
sudo dnf install -y nginx

# Remove package
sudo dnf remove nginx
sudo dnf autoremove        # remove orphaned packages

# Search
dnf search nginx
dnf info nginx

# List installed packages
dnf list installed
rpm -qa                     # list all installed RPMs
rpm -qa | grep nginx

# Query which package provides a file
dnf provides /usr/sbin/nginx
rpm -qf /usr/sbin/nginx     # query installed package owning file

# List package files
rpm -ql nginx

# Install local RPM
sudo dnf install ./package.rpm
sudo rpm -ivh package.rpm

# Enable/disable repositories
sudo dnf config-manager --enable epel
sudo dnf config-manager --disable epel

# List available repositories
dnf repolist
dnf repolist all

# Clean cache
sudo dnf clean all
```

---

## systemd — Service Management

### Understanding systemd

`systemd` is the init system used by most modern Linux distros. It manages:
- **Services** (units of type `.service`)
- **Mounts** (`.mount`)
- **Timers** (`.timer` — cron alternative)
- **Sockets** (`.socket`)
- **Targets** (`.target` — like runlevels)

### Managing Services with `systemctl`

```bash
# Start/stop/restart a service
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx

# Reload config without full restart (if supported)
sudo systemctl reload nginx
sudo systemctl reload-or-restart nginx

# Enable/disable (start at boot)
sudo systemctl enable nginx
sudo systemctl disable nginx

# Enable and start in one command
sudo systemctl enable --now nginx
sudo systemctl disable --now nginx

# Check service status
systemctl status nginx
systemctl status nginx --no-pager    # no pagination

# Check if service is active/enabled
systemctl is-active nginx
systemctl is-enabled nginx
systemctl is-failed nginx

# View all services
systemctl list-units --type=service
systemctl list-units --type=service --state=failed
systemctl list-unit-files --type=service

# Mask a service (prevent it from ever being started)
sudo systemctl mask postfix
sudo systemctl unmask postfix

# Reload systemd after adding/changing unit files
sudo systemctl daemon-reload
```

### systemd Unit Files

Unit files live in `/etc/systemd/system/` (custom) or `/lib/systemd/system/` (package-installed).

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
Documentation=https://myapp.example.com/docs
After=network.target postgresql.service
Requires=postgresql.service

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/server --port 8080
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5
StandardOutput=journal
StandardError=journal
Environment=NODE_ENV=production
EnvironmentFile=/etc/myapp/config.env

# Resource limits
LimitNOFILE=65536
MemoryMax=512M
CPUQuota=50%

[Install]
WantedBy=multi-user.target
```

```bash
# Enable and start custom service
sudo systemctl daemon-reload
sudo systemctl enable myapp.service
sudo systemctl start myapp.service
sudo systemctl status myapp.service
```

### systemd Targets (Runlevels)

```bash
# List targets
systemctl list-units --type=target

# Common targets:
# poweroff.target   = shutdown (runlevel 0)
# rescue.target     = single user mode (runlevel 1)
# multi-user.target = multi-user no GUI (runlevel 3)
# graphical.target  = multi-user with GUI (runlevel 5)
# reboot.target     = reboot (runlevel 6)

# Get current default target
systemctl get-default

# Set default target
sudo systemctl set-default multi-user.target

# Switch to a target without reboot
sudo systemctl isolate rescue.target
```

---

## Logging with journald and syslog

### `journalctl` — systemd Journal

```bash
# View all logs
journalctl

# Follow live (like tail -f)
journalctl -f

# Logs for a specific service
journalctl -u nginx
journalctl -u nginx -f         # follow nginx logs
journalctl -u nginx --since today
journalctl -u nginx --since "2024-03-01" --until "2024-03-02"
journalctl -u nginx -n 50      # last 50 lines
journalctl -u nginx --no-pager # no pager

# Filter by priority
journalctl -p err              # errors and above
journalctl -p warning          # warnings and above
# Priority levels: debug, info, notice, warning, err, crit, alert, emerg

# Show only kernel messages
journalctl -k

# Since last boot
journalctl -b
journalctl -b -1               # previous boot
journalctl --list-boots        # list all boots

# Filter by time
journalctl --since "10 minutes ago"
journalctl --since "2024-03-10 09:00:00" --until "2024-03-10 09:30:00"

# Show disk usage
journalctl --disk-usage

# Vacuum old logs
sudo journalctl --vacuum-time=7d    # keep last 7 days
sudo journalctl --vacuum-size=500M  # keep last 500MB
```

### Traditional Log Files

```bash
# Key log files in /var/log/
/var/log/syslog          # system messages (Debian/Ubuntu)
/var/log/messages        # system messages (RHEL/CentOS)
/var/log/auth.log        # authentication logs (Debian/Ubuntu)
/var/log/secure          # authentication logs (RHEL/CentOS)
/var/log/kern.log        # kernel messages
/var/log/dmesg           # kernel ring buffer
/var/log/nginx/access.log
/var/log/nginx/error.log
/var/log/apache2/access.log
/var/log/mysql/error.log
/var/log/postgresql/postgresql-*.log

# View logs
tail -f /var/log/syslog
grep "error" /var/log/nginx/error.log
grep -i "failed" /var/log/auth.log
grep "sshd" /var/log/auth.log | grep "Failed"

# View kernel messages
dmesg
dmesg | grep -i error
dmesg -T             # with human-readable timestamps
dmesg -H             # with pager
```

### Log Rotation with `logrotate`

```bash
# Configuration
cat /etc/logrotate.conf          # global defaults
ls /etc/logrotate.d/             # per-application configs

# Example custom config: /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily                        # rotate daily
    missingok                    # don't error if log missing
    rotate 14                    # keep 14 rotations
    compress                     # gzip old logs
    delaycompress                # compress on next rotation
    notifempty                   # don't rotate empty logs
    create 0640 myapp myapp      # create new file with perms
    sharedscripts
    postrotate
        systemctl reload myapp   # reload app after rotation
    endscript
}

# Test logrotate config
sudo logrotate -d /etc/logrotate.d/myapp    # dry run (debug)
sudo logrotate -f /etc/logrotate.d/myapp    # force rotate now
```

---

## Disk Management

### Disk and Partition Info

```bash
# List block devices
lsblk
lsblk -f                 # show filesystem info
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT

# List disks
fdisk -l                 # list all partitions
sudo fdisk -l /dev/sda   # specific disk

# Filesystem usage
df -h                    # human-readable
df -h --type=ext4        # specific filesystem type
df -ih                   # inode usage

# Identify disk by UUID or label
blkid
blkid /dev/sda1

# View /etc/fstab (filesystems to mount at boot)
cat /etc/fstab
# Format: device  mountpoint  fstype  options  dump  pass
```

### Partitioning

```bash
# Create/modify partitions with fdisk (MBR)
sudo fdisk /dev/sdb
# Commands: n=new, d=delete, p=print, w=write, q=quit

# GPT partitioning with gdisk
sudo gdisk /dev/sdb

# Modern: parted
sudo parted /dev/sdb
(parted) mklabel gpt
(parted) mkpart primary ext4 0% 100%
(parted) print

# Non-interactive parted
sudo parted -s /dev/sdb mklabel gpt
sudo parted -s /dev/sdb mkpart primary ext4 0% 100%
```

### Filesystem Operations

```bash
# Create filesystems
sudo mkfs.ext4 /dev/sdb1                 # ext4
sudo mkfs.xfs /dev/sdb1                  # XFS (default on RHEL 7+)
sudo mkfs.ext4 -L "mydata" /dev/sdb1     # with label

# Mount filesystem
sudo mount /dev/sdb1 /mnt/data
sudo mount -t ext4 /dev/sdb1 /mnt/data

# Mount options
sudo mount -o ro /dev/sdb1 /mnt/data     # read-only
sudo mount -o remount,rw /mnt/data       # remount read-write

# Unmount
sudo umount /mnt/data
sudo umount /dev/sdb1

# Make persistent (add to /etc/fstab)
# UUID=abc123 /mnt/data ext4 defaults 0 2
echo "UUID=$(blkid -s UUID -o value /dev/sdb1) /mnt/data ext4 defaults 0 2" | sudo tee -a /etc/fstab

# Check filesystem integrity (unmount first)
sudo umount /dev/sdb1
sudo fsck /dev/sdb1
sudo fsck -p /dev/sdb1    # auto-repair

# Resize filesystem (ext4 — after resizing partition)
sudo resize2fs /dev/sdb1          # grow to partition size
sudo resize2fs /dev/sdb1 50G      # shrink to 50G (unmount first)
```

### LVM — Logical Volume Manager

```bash
# LVM concepts: PV (Physical Volume) → VG (Volume Group) → LV (Logical Volume)

# Initialize physical volume
sudo pvcreate /dev/sdb /dev/sdc
pvdisplay
pvs

# Create volume group
sudo vgcreate myvg /dev/sdb /dev/sdc
vgdisplay
vgs

# Create logical volume
sudo lvcreate -L 50G -n mydata myvg      # 50GB LV
sudo lvcreate -l 100%FREE -n mydata myvg  # use all free space
lvdisplay
lvs

# Create filesystem and mount
sudo mkfs.ext4 /dev/myvg/mydata
sudo mount /dev/myvg/mydata /mnt/data

# Extend LV (add 20GB)
sudo lvextend -L +20G /dev/myvg/mydata
sudo resize2fs /dev/myvg/mydata           # ext4: resize filesystem to fill
sudo xfs_growfs /mnt/data                 # XFS: resize mounted filesystem

# Add disk to VG
sudo pvcreate /dev/sdd
sudo vgextend myvg /dev/sdd

# Snapshots (great for backups)
sudo lvcreate -L 5G -s -n mydata-snap /dev/myvg/mydata
```

### Disk Space Troubleshooting

```bash
# Find what's eating disk space
du -sh /* 2>/dev/null | sort -rh | head -20         # top-level directories
du -sh /var/* 2>/dev/null | sort -rh | head -10
du -sh /var/log/* | sort -rh | head -10

# Find large files
find / -type f -size +100M 2>/dev/null
find /var -type f -size +50M -exec ls -lh {} \;

# Find files not accessed recently (candidates for cleanup)
find /tmp -type f -atime +30 -delete             # delete old tmp files
find /var/log -name "*.gz" -mtime +30 -delete    # old compressed logs

# Check inode usage (sometimes disk "full" = inodes exhausted)
df -i
df -i | grep -v "100%\|Filesystem"

# Which process has a deleted file open (disk space not freed)
sudo lsof | grep deleted
# Restart that process or kill it to free space
```

---

## Performance Monitoring

```bash
# System overview
top
htop
glances         # apt install glances — comprehensive overview

# CPU stats
mpstat          # apt install sysstat
mpstat -P ALL   # per-CPU stats
sar -u 1 5      # CPU utilization every 1s, 5 times

# Memory stats
vmstat 1 5      # virtual memory stats every 1s
free -h -s 2    # refresh every 2 seconds

# Disk I/O
iostat -xh 1 5  # extended disk stats
iotop           # per-process I/O (like top for disk)

# Network stats
nethogs         # per-process network usage
iftop -i eth0   # bandwidth per connection
nload eth0      # total bandwidth

# All-in-one performance snapshot
sar -A          # all available stats

# System calls (debugging)
strace -p PID           # trace system calls of a process
strace -c command       # summarize system calls

# Open files per process
lsof -p PID
lsof -u username
```

---

## System Startup and Boot

```bash
# View boot messages
dmesg
dmesg -T | grep -i error
journalctl -b             # this boot
journalctl -b -1          # previous boot

# Analyze boot time
systemd-analyze
systemd-analyze blame              # time per unit
systemd-analyze critical-chain     # critical path

# System shutdown/reboot
sudo shutdown -h now          # shutdown immediately
sudo shutdown -r now          # reboot immediately
sudo shutdown -h +10          # shutdown in 10 minutes
sudo shutdown -r +5 "Rebooting for kernel update"  # with message
sudo shutdown -c              # cancel scheduled shutdown
sudo reboot                   # reboot (same as shutdown -r now)
sudo poweroff                 # power off
sudo halt                     # halt (may or may not power off)
```

---

## Day 6 Practice Exercises

1. Install `nginx` using your distro's package manager. Start, enable, and verify it's running with `systemctl`.
2. Create a custom systemd service that runs a simple shell script as a service.
3. Use `journalctl -u nginx -n 50` to view the last 50 log lines for nginx.
4. Create a logrotate config for `/var/log/myapp.log` that rotates weekly, keeps 4 backups, and compresses old logs.
5. Use `lsblk` and `df -h` to list all block devices and their usage.
6. Use `du -sh /var/*` to find the largest directories under `/var`. Sort the output.
7. Find all files larger than 10MB in `/var/log` and list them with their sizes.
8. Use `systemd-analyze blame` to see which services are slowest to start.
9. View the last 100 lines of `/var/log/auth.log` and identify any failed login attempts.
10. Use `sar -u 1 5` to capture CPU stats for 5 seconds. Which CPU state has the most idle time?

---

## Summary

| Concept | Key Commands |
|---------|-------------|
| APT packages | `apt update`, `apt install`, `apt remove`, `apt upgrade` |
| DNF packages | `dnf install`, `dnf remove`, `dnf update` |
| Service control | `systemctl start/stop/restart/enable/status` |
| Custom service | Create `/etc/systemd/system/app.service`, `daemon-reload` |
| Journal logs | `journalctl -u service`, `journalctl -f`, `journalctl -p err` |
| Log files | `/var/log/syslog`, `tail -f`, `grep` |
| Log rotation | `/etc/logrotate.d/`, `logrotate -d` (dry run) |
| Disk info | `lsblk`, `df -h`, `du -sh`, `blkid` |
| Partitions | `fdisk`, `parted`, `mkfs.ext4` |
| LVM | `pvcreate`, `vgcreate`, `lvcreate`, `lvextend` |
| Performance | `top`, `vmstat`, `iostat`, `iotop` |
| Boot | `systemd-analyze`, `journalctl -b`, `dmesg` |

---

[Previous: Day 5](../day-5/README.md) | [Next: Day 7 - DevOps on Linux](../day-7/README.md)
