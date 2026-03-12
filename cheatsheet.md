# Linux for DevOps — Cheatsheet

## Navigation
```bash
pwd                    # current directory
ls -lah                # list all files with details
cd /path               # absolute path
cd ../..               # go up two levels
cd -                   # previous directory
```

## File Operations
```bash
touch file.txt         # create file
mkdir -p a/b/c         # create nested dirs
cp -r src/ dst/        # copy recursively
mv old new             # move / rename
rm -rf dir/            # delete dir (careful!)
ln -s target link      # symlink
```

## Viewing Files
```bash
cat file               # print file
less file              # page through file
head -20 file          # first 20 lines
tail -f file           # follow live
grep -r "term" dir/    # recursive search
```

## Permissions
```bash
chmod 755 file         # rwxr-xr-x
chmod 644 file         # rw-r--r--
chmod 600 file         # rw------- (private)
chmod +x script.sh     # make executable
chown user:group file  # change ownership
chown -R user:grp dir/ # recursive
```

## Users & Groups
```bash
whoami                 # current user
id                     # uid, gid, groups
adduser tom            # create user (interactive)
useradd -m -s /bin/bash tom
passwd tom             # set password
usermod -aG docker tom # add to group
userdel -r tom         # delete user + home
groupadd devops        # create group
sudo -l                # list sudo privileges
su - tom               # switch user
```

## Processes
```bash
ps aux                 # all processes
ps aux | grep nginx    # filter by name
top / htop             # interactive monitor
kill PID               # graceful kill (SIGTERM)
kill -9 PID            # force kill (SIGKILL)
pkill nginx            # kill by name
pgrep nginx            # find PIDs by name
command &              # run in background
jobs                   # list background jobs
fg / bg                # foreground/background
nohup cmd &            # survive logout
```

## Networking
```bash
ip addr show           # interfaces and IPs
ip route show          # routing table
ping -c 4 host         # test reachability
traceroute host        # trace hops
ss -tulnp              # listening ports
dig google.com         # DNS lookup
nslookup google.com    # DNS lookup
curl -I https://url    # HTTP headers
wget -O file.tar.gz url # download file
nc -zv host port       # test port
```

## SSH
```bash
ssh user@host          # connect
ssh -p 2222 user@host  # custom port
ssh-keygen -t ed25519  # generate key pair
ssh-copy-id user@host  # copy public key
scp file user@host:/path   # copy to remote
scp user@host:/file ./     # copy from remote
rsync -avz src/ user@host:/dst/  # sync
```

## Firewall (UFW)
```bash
ufw enable             # enable firewall
ufw allow 22           # allow SSH
ufw allow 80/tcp       # allow HTTP
ufw deny 23            # deny telnet
ufw status verbose     # view rules
ufw delete allow 8080  # remove rule
```

## Package Management
```bash
# APT (Debian/Ubuntu)
apt update             # refresh index
apt install nginx      # install
apt remove nginx       # remove
apt upgrade            # upgrade all
apt search nginx       # search
apt show nginx         # info

# DNF (RHEL/CentOS)
dnf install nginx      # install
dnf remove nginx       # remove
dnf update             # upgrade all
dnf search nginx       # search
```

## systemd
```bash
systemctl start nginx          # start
systemctl stop nginx           # stop
systemctl restart nginx        # restart
systemctl reload nginx         # reload config
systemctl enable nginx         # start at boot
systemctl disable nginx        # disable autostart
systemctl status nginx         # check status
systemctl list-units --failed  # failed services
systemctl daemon-reload        # reload unit files
```

## Logs
```bash
journalctl -f                  # follow system log
journalctl -u nginx            # service logs
journalctl -u nginx -n 50      # last 50 lines
journalctl -p err              # errors only
journalctl -b                  # this boot
journalctl --since "1 hour ago"
tail -f /var/log/syslog        # traditional log
grep "error" /var/log/nginx/error.log
```

## Disk
```bash
df -h                  # filesystem usage
du -sh /var/log/       # directory size
du -sh /var/* | sort -rh | head  # largest dirs
lsblk                  # block devices
fdisk -l               # partition info
mount /dev/sdb1 /mnt   # mount
umount /mnt            # unmount
mkfs.ext4 /dev/sdb1    # create filesystem
```

## Text Processing
```bash
grep -i "error" file         # case-insensitive
grep -r "todo" dir/          # recursive
grep -n "pattern" file       # with line numbers
grep -v "debug" file         # invert match
awk '{print $1}' file        # first column
awk -F: '{print $1}' /etc/passwd   # custom delimiter
sed 's/old/new/g' file       # replace all
cut -d: -f1 /etc/passwd      # cut field 1
sort -n numbers.txt          # numeric sort
sort -rh | head -10          # largest first
wc -l file                   # line count
```

## Archives
```bash
tar -czvf out.tar.gz dir/    # create gzip archive
tar -xzvf archive.tar.gz     # extract gzip
tar -xzvf archive.tar.gz -C /opt/  # extract to dir
tar -tvf archive.tar.gz      # list contents
zip -r out.zip dir/          # create zip
unzip out.zip -d /opt/       # extract zip
gzip file                    # compress file
gunzip file.gz               # decompress
```

## Find
```bash
find / -name "nginx.conf"         # by name
find / -type f -size +100M        # large files
find /var/log -mtime -1           # modified today
find / -perm /u+s                 # SUID files
find /tmp -mtime +7 -delete       # delete old files
find . -name "*.log" -exec gzip {} \;
```

## Shell Scripting Quick Reference
```bash
#!/usr/bin/env bash
set -euo pipefail

VAR="value"                    # variable
echo "${VAR}"                  # use variable
RESULT=$(command)              # command substitution

if [[ "$VAR" == "test" ]]; then
    echo "match"
fi

for item in a b c; do
    echo "$item"
done

while [[ condition ]]; do
    command
done

func() {
    local arg=$1
    echo "$arg"
}

[ -f file ]   # file exists
[ -d dir ]    # directory exists
[ -z "$str" ] # string is empty
[ $a -gt $b ] # numeric greater than
```

## Environment Variables
```bash
export MY_VAR="value"          # set variable
echo $MY_VAR                   # use variable
unset MY_VAR                   # delete variable
env                            # list all
export PATH="$PATH:/new/dir"   # add to PATH
source .env                    # load .env file
export $(grep -v ^# .env | xargs)  # load .env
```

## Cron
```bash
crontab -e                     # edit crontab
crontab -l                     # list crontab

# Syntax: MIN HOUR DOM MON DOW command
0 2 * * *    /opt/backup.sh    # daily at 2am
*/5 * * * *  /opt/check.sh     # every 5 min
0 9 * * 1-5  /opt/report.sh    # weekdays 9am
0 0 1 * *    /opt/monthly.sh   # 1st of month
```

## Performance
```bash
top / htop             # CPU, memory, processes
free -h                # memory usage
vmstat 1 5             # virtual memory stats
iostat -h              # disk I/O stats
iotop                  # per-process I/O
uptime                 # load average
sar -u 1 5             # CPU stats (sysstat)
lscpu                  # CPU info
```

## Security
```bash
# SSH hardening (/etc/ssh/sshd_config)
PermitRootLogin no
PasswordAuthentication no
MaxAuthTries 3

# find risky files
find / -perm /u+s 2>/dev/null          # SUID files
find / -type f -perm -o+w 2>/dev/null  # world-writable

# fail2ban
fail2ban-client status sshd
fail2ban-client set sshd unbanip IP

# sysctl hardening
sysctl -w net.ipv4.tcp_syncookies=1
sysctl -p   # apply /etc/sysctl.conf
```

## Useful Shortcuts
| Shortcut | Action |
|----------|--------|
| `Ctrl+C` | Kill current command |
| `Ctrl+Z` | Suspend command |
| `Ctrl+L` | Clear screen |
| `Ctrl+A` | Beginning of line |
| `Ctrl+E` | End of line |
| `Ctrl+R` | Search history |
| `Ctrl+U` | Clear line |
| `Ctrl+W` | Delete word |
| `Tab` | Auto-complete |
| `!!` | Repeat last command |
| `sudo !!` | Repeat last as root |
| `!$` | Last argument |
