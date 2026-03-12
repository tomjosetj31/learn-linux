# Day 4: Networking

## Networking Fundamentals

### Key Concepts

| Concept | Description |
|---------|-------------|
| IP Address | Unique identifier for a device on a network (IPv4: `192.168.1.10`) |
| Subnet Mask | Defines the network portion vs host portion (`/24` = `255.255.255.0`) |
| Gateway | Router IP that connects to other networks / the internet |
| DNS | Domain Name System — translates names to IPs (`google.com` → `142.250.80.46`) |
| Port | Number identifying a service on a host (HTTP=80, HTTPS=443, SSH=22) |
| CIDR | Classless Inter-Domain Routing: `192.168.1.0/24` = 256 addresses |

### Common Port Numbers (Memorize These)

| Port | Protocol | Service |
|------|----------|---------|
| 22 | TCP | SSH |
| 80 | TCP | HTTP |
| 443 | TCP | HTTPS |
| 21 | TCP | FTP |
| 25 | TCP | SMTP (email) |
| 53 | UDP/TCP | DNS |
| 3306 | TCP | MySQL |
| 5432 | TCP | PostgreSQL |
| 6379 | TCP | Redis |
| 27017 | TCP | MongoDB |
| 8080 | TCP | HTTP (alternate) |
| 9200 | TCP | Elasticsearch |

---

## Network Interface Management

### Viewing Network Interfaces

```bash
# Show all network interfaces and IP addresses
ip addr show
ip a          # shorthand

# Show specific interface
ip addr show eth0
ip addr show lo      # loopback interface

# Legacy command (still works, but deprecated)
ifconfig
ifconfig eth0

# Show routing table
ip route show
ip r          # shorthand
route -n      # legacy

# Show default gateway
ip route | grep default

# Show network interfaces with stats
ip -s link show
```

### Configuring Interfaces (Temporary)

```bash
# Assign IP address to interface
sudo ip addr add 192.168.1.100/24 dev eth0

# Remove IP address
sudo ip addr del 192.168.1.100/24 dev eth0

# Bring interface up/down
sudo ip link set eth0 up
sudo ip link set eth0 down

# Add a route
sudo ip route add 10.0.0.0/8 via 192.168.1.1
sudo ip route add default via 192.168.1.1   # default gateway

# Delete a route
sudo ip route del 10.0.0.0/8
```

### Persistent Network Configuration

**Ubuntu (Netplan — modern)**
```yaml
# /etc/netplan/00-installer-config.yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true

# Static IP example:
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

```bash
# Apply netplan changes
sudo netplan apply
sudo netplan try      # apply with auto-revert if you lose connection
```

**RHEL/CentOS (NetworkManager)**
```bash
# List connections
nmcli connection show

# Show details
nmcli device show eth0

# Set static IP
sudo nmcli con mod eth0 ipv4.addresses 192.168.1.100/24
sudo nmcli con mod eth0 ipv4.gateway 192.168.1.1
sudo nmcli con mod eth0 ipv4.dns "8.8.8.8 8.8.4.4"
sudo nmcli con mod eth0 ipv4.method manual
sudo nmcli con up eth0
```

---

## DNS

### DNS Lookup Tools

```bash
# Basic lookup
nslookup google.com
nslookup google.com 8.8.8.8    # use specific DNS server

# Detailed DNS query
dig google.com
dig google.com A               # A record (IPv4)
dig google.com AAAA            # IPv6 address
dig google.com MX              # mail exchange records
dig google.com NS              # name servers
dig google.com TXT             # text records
dig @8.8.8.8 google.com        # query specific DNS server
dig +short google.com          # just the IP

# Reverse DNS lookup (IP to hostname)
dig -x 8.8.8.8
nslookup 8.8.8.8

# Simple lookup
host google.com
host 8.8.8.8
```

### DNS Configuration Files

```bash
# DNS resolver config (which DNS server to use)
cat /etc/resolv.conf
# nameserver 8.8.8.8
# nameserver 8.8.4.4
# search mydomain.local

# Local hostname resolution (checked before DNS)
cat /etc/hosts
# 127.0.0.1   localhost
# 192.168.1.10  myserver.local myserver

# NSSwitch — controls resolution order
cat /etc/nsswitch.conf
# hosts: files dns   (check /etc/hosts first, then DNS)

# Add a local hostname override
echo "192.168.1.50  staging.myapp.com" | sudo tee -a /etc/hosts
```

---

## SSH — Secure Shell

### Connecting via SSH

```bash
# Basic connection
ssh user@hostname
ssh tom@192.168.1.10
ssh tom@server.example.com

# Specify port (default is 22)
ssh -p 2222 tom@server.example.com

# Connect with verbose output (debug connection issues)
ssh -v tom@server.example.com
ssh -vvv tom@server.example.com   # very verbose

# Run a command on remote host without interactive shell
ssh tom@server.example.com "ls -la /var/log"
ssh tom@server.example.com "sudo systemctl status nginx"

# Forward X11 (GUI apps)
ssh -X tom@server.example.com
```

### SSH Key Management

```bash
# Generate SSH key pair
ssh-keygen -t ed25519 -C "tom@mycompany.com"
ssh-keygen -t rsa -b 4096 -C "tom@mycompany.com"    # RSA (older, but widely supported)
# Keys saved to: ~/.ssh/id_ed25519 (private) and ~/.ssh/id_ed25519.pub (public)

# Copy public key to remote server (enables passwordless login)
ssh-copy-id tom@server.example.com
ssh-copy-id -i ~/.ssh/id_ed25519.pub tom@server.example.com

# Manual method (if ssh-copy-id unavailable)
cat ~/.ssh/id_ed25519.pub | ssh tom@server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Set correct permissions (REQUIRED for SSH keys to work)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519          # private key
chmod 644 ~/.ssh/id_ed25519.pub      # public key
chmod 600 ~/.ssh/authorized_keys

# Test key-based login
ssh tom@server.example.com

# List loaded keys in SSH agent
ssh-add -l

# Add key to SSH agent (avoids typing passphrase repeatedly)
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

### SSH Config File (`~/.ssh/config`)

Create shortcuts and per-host settings:

```bash
# ~/.ssh/config
Host dev-server
    HostName 192.168.1.10
    User tom
    Port 22
    IdentityFile ~/.ssh/id_ed25519

Host prod-server
    HostName prod.example.com
    User deploy
    Port 2222
    IdentityFile ~/.ssh/id_rsa_prod
    ServerAliveInterval 60
    ServerAliveCountMax 3

Host bastion
    HostName bastion.example.com
    User tom
    IdentityFile ~/.ssh/id_ed25519

Host private-server
    HostName 10.0.0.5
    User tom
    ProxyJump bastion    # connect through bastion host
```

```bash
# Now you can simply:
ssh dev-server
ssh prod-server
ssh private-server    # automatically uses bastion as jump host
```

### SSH Server Configuration (`/etc/ssh/sshd_config`)

```bash
# Key security settings (edit as root)
sudo nano /etc/ssh/sshd_config

# Recommended security settings:
Port 2222                          # change default port
PermitRootLogin no                 # never allow root SSH
PasswordAuthentication no          # key-only authentication
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers tom deploy              # whitelist users

# Apply changes
sudo systemctl reload sshd
sudo systemctl restart sshd

# Test config before applying
sudo sshd -t    # test config syntax
```

### SCP and SFTP — Secure File Transfer

```bash
# Copy file to remote
scp file.txt tom@server:/home/tom/
scp -P 2222 file.txt tom@server:/tmp/   # non-standard port

# Copy file from remote
scp tom@server:/var/log/app.log ./

# Copy directory recursively
scp -r mydir/ tom@server:/home/tom/

# Use SSH config aliases
scp file.txt dev-server:/home/tom/

# SFTP interactive session
sftp tom@server.example.com
sftp> ls
sftp> get remotefile.txt
sftp> put localfile.txt
sftp> mkdir newdir
sftp> exit

# rsync — smarter file sync (only transfers changes)
rsync -avz localdir/ tom@server:/remote/dir/
rsync -avz --delete localdir/ tom@server:/remote/dir/  # mirror (delete remote extras)
rsync -avz -e "ssh -p 2222" localdir/ tom@server:/remote/dir/  # custom port
```

---

## Firewall Management

### UFW (Uncomplicated Firewall) — Ubuntu

```bash
# Check status
sudo ufw status
sudo ufw status verbose

# Enable/disable
sudo ufw enable
sudo ufw disable

# Allow ports
sudo ufw allow 22           # allow SSH
sudo ufw allow ssh          # same (uses /etc/services)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 8080
sudo ufw allow from 192.168.1.0/24 to any port 5432   # PostgreSQL from subnet only

# Deny ports
sudo ufw deny 23            # deny telnet
sudo ufw deny from 10.0.0.5

# Delete rules
sudo ufw delete allow 8080
sudo ufw delete deny 23

# Reset to defaults
sudo ufw reset

# Logging
sudo ufw logging on
sudo ufw logging medium
```

### firewalld — RHEL/CentOS

```bash
# Check status
sudo firewall-cmd --state
sudo firewall-cmd --list-all

# Add service permanently
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-service=ssh

# Add port
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --permanent --add-port=3000-3010/tcp

# Remove
sudo firewall-cmd --permanent --remove-service=http
sudo firewall-cmd --permanent --remove-port=8080/tcp

# Reload after changes
sudo firewall-cmd --reload

# Allow from specific IP
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.10" port port="5432" protocol="tcp" accept'
```

### iptables (Low-level)

```bash
# List rules
sudo iptables -L -n -v
sudo iptables -L INPUT -n -v    # INPUT chain only

# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Drop everything else
sudo iptables -A INPUT -j DROP

# Save rules (Ubuntu)
sudo iptables-save > /etc/iptables/rules.v4

# Save rules (RHEL)
sudo service iptables save
```

---

## Network Troubleshooting

### Connectivity Testing

```bash
# ICMP ping
ping google.com
ping -c 4 google.com         # send 4 packets only
ping -i 0.2 google.com       # ping every 0.2 seconds
ping 8.8.8.8

# Trace route to destination
traceroute google.com
traceroute -n google.com     # don't resolve hostnames (faster)
mtr google.com               # real-time traceroute (install: apt install mtr)

# Test TCP port connectivity
telnet server.example.com 80
nc -zv server.example.com 80       # netcat port test
nc -zv server.example.com 80-100   # range of ports

# Test HTTP response
curl -I https://google.com         # headers only
curl -v https://google.com         # verbose
curl -o /dev/null -s -w "%{http_code}" https://google.com  # just status code
```

### Viewing Connections and Ports

```bash
# View listening ports and active connections
ss -tulnp        # t=TCP, u=UDP, l=listening, n=numeric, p=process
ss -tulnp | grep :80
ss -s            # summary statistics

# Legacy netstat (deprecated but still available)
netstat -tulnp
netstat -an | grep :80

# Check what's using a port
sudo lsof -i :80
sudo lsof -i :443
sudo lsof -i TCP:22

# Find process using port 8080
sudo ss -tulnp | grep 8080
sudo fuser 8080/tcp         # show PID using port

# View all open files/connections for a process
sudo lsof -p 1234
sudo lsof -u nginx
```

### Network Statistics

```bash
# Network interface statistics
ip -s link show eth0
cat /proc/net/dev

# Bandwidth monitoring
nload eth0                   # install: apt install nload
iftop -i eth0                # install: apt install iftop
nethogs                      # per-process bandwidth

# Socket statistics
ss -s                        # summary
ss -tp                       # TCP with process info
```

### Common Troubleshooting Workflow

```bash
# 1. Can I reach the gateway?
ping 192.168.1.1

# 2. Can I resolve DNS?
nslookup google.com
cat /etc/resolv.conf

# 3. Can I reach the internet?
ping 8.8.8.8        # if this works but DNS fails, it's a DNS issue

# 4. Is the service listening on the right port?
sudo ss -tulnp | grep nginx
curl http://localhost

# 5. Is the firewall blocking the port?
sudo ufw status
sudo iptables -L -n -v

# 6. Is the route correct?
ip route show
traceroute destination

# 7. Are there errors on the interface?
ip -s link show eth0      # look for errors/drops
dmesg | grep -i eth0      # kernel messages about interface
```

---

## HTTP Tools

```bash
# curl — Swiss army knife for HTTP
curl http://example.com                        # GET request
curl -O http://example.com/file.tar.gz         # download file (keep name)
curl -o output.tar.gz http://example.com/file  # download with custom name
curl -X POST http://api.example.com/data \
     -H "Content-Type: application/json" \
     -d '{"key":"value"}'                      # POST with JSON
curl -H "Authorization: Bearer TOKEN" http://api.example.com  # with auth
curl -L http://example.com                     # follow redirects
curl -k https://self-signed.example.com        # ignore SSL errors
curl --retry 3 --retry-delay 5 http://api.example.com  # retry

# wget — download files
wget http://example.com/file.tar.gz
wget -O /tmp/output.tar.gz http://example.com/file    # custom output name
wget -r -np http://example.com/docs/                   # recursive download
wget -c http://example.com/largefile.tar.gz            # resume partial download
wget -q --spider http://example.com           # check if URL exists (quiet)
```

---

## Day 4 Practice Exercises

1. Use `ip addr show` to find your machine's IP and MAC address.
2. Use `dig` to look up the A, MX, and NS records for `github.com`.
3. Generate an SSH key pair and copy the public key to a test server (or localhost).
4. Create an `~/.ssh/config` entry for a server with a custom port and user.
5. Use `ss -tulnp` to list all listening services. Identify what's running on port 22.
6. Enable UFW, allow SSH and HTTP, then verify with `sudo ufw status`.
7. Use `curl -I` to check the HTTP headers of `https://httpbin.org/get`.
8. Use `traceroute` to trace the path to `8.8.8.8`. How many hops?
9. Use `nc -zv` to test if port 443 is open on `google.com`.
10. Configure `/etc/hosts` to resolve `myapp.local` to `127.0.0.1`, then ping it.

---

## Summary

| Concept | Key Commands |
|---------|-------------|
| View interfaces | `ip addr show`, `ip route show` |
| DNS lookup | `dig`, `nslookup`, `host` |
| SSH connect | `ssh user@host`, `ssh -p port user@host` |
| SSH keys | `ssh-keygen`, `ssh-copy-id`, `~/.ssh/config` |
| Firewall (Ubuntu) | `ufw allow 80`, `ufw enable`, `ufw status` |
| Firewall (RHEL) | `firewall-cmd --add-service=http --permanent` |
| Test connectivity | `ping`, `traceroute`, `nc -zv host port` |
| View ports | `ss -tulnp`, `lsof -i :80` |
| HTTP | `curl -X GET/POST`, `wget -O` |
| File transfer | `scp`, `rsync -avz` |

---

[Previous: Day 3](../day-3/README.md) | [Next: Day 5 - Shell Scripting](../day-5/README.md)
