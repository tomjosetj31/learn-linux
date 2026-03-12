# Day 2: File Management & Permissions

## File Types in Linux

Everything in Linux is a file. Linux uses special characters to indicate file types:

```bash
# List with type indicator
ls -la

# File type is shown as the first character in permissions:
# -  regular file
# d  directory
# l  symbolic link
# c  character device
# b  block device
# p  named pipe (FIFO)
# s  socket
```

```bash
# Identify file type
file myfile.txt
file /bin/bash
file /dev/sda

# Example output:
# myfile.txt:  ASCII text
# /bin/bash:   ELF 64-bit LSB pie executable
# /dev/sda:    block special (8/0)
```

---

## Working with Files

### Creating Files

```bash
# Create empty file (or update timestamp if exists)
touch file.txt
touch file1.txt file2.txt file3.txt   # multiple at once

# Create file with content using echo
echo "Hello World" > hello.txt

# Create file with multiple lines using heredoc
cat > config.txt << EOF
server=localhost
port=8080
debug=true
EOF

# Create using a text editor
nano myfile.txt
vi myfile.txt
vim myfile.txt
```

### Viewing Files

```bash
# Print entire file
cat file.txt

# Print with line numbers
cat -n file.txt

# View file page by page (recommended for large files)
less /var/log/syslog    # q=quit, /=search, n=next match, Space=next page, b=back

# View first N lines (default 10)
head file.txt
head -n 20 file.txt
head -20 file.txt

# View last N lines (default 10)
tail file.txt
tail -n 50 file.txt

# Follow file in real time (great for logs)
tail -f /var/log/nginx/access.log
tail -f -n 100 /var/log/syslog   # show last 100 lines then follow
```

### Copying, Moving, Renaming

```bash
# Copy file
cp source.txt destination.txt

# Copy and preserve attributes (timestamps, permissions)
cp -p source.txt destination.txt

# Copy directory recursively
cp -r sourcedir/ destdir/

# Copy with verbose output
cp -v file.txt /tmp/

# Move (also renames)
mv oldname.txt newname.txt
mv file.txt /var/tmp/
mv dir1/ /opt/newlocation/

# Move multiple files to a directory
mv file1.txt file2.txt file3.txt /tmp/

# Interactive (ask before overwriting)
mv -i file.txt /tmp/
cp -i file.txt /tmp/
```

### Deleting Files

```bash
# Delete a file
rm file.txt

# Interactive deletion (asks confirmation)
rm -i file.txt

# Delete a directory and its contents
rm -r mydir/

# Force delete without prompts (DANGEROUS — double-check before running)
rm -rf mydir/

# Delete files matching a pattern
rm *.log
rm /tmp/*.tmp

# Safe deletion tip: list first, then delete
ls /tmp/*.log      # review
rm /tmp/*.log      # then delete
```

---

## File Permissions

### Understanding Permission Strings

```bash
ls -l

# Output example:
# -rw-r--r-- 1 tom devops 1024 Mar 10 09:00 file.txt
# drwxr-xr-x 2 tom devops 4096 Mar 10 09:00 mydir/
```

Breaking down `-rw-r--r--`:

```
- rw- r-- r--
│ │   │   │
│ │   │   └── Other permissions (everyone else)
│ │   └────── Group permissions
│ └────────── Owner (user) permissions
└──────────── File type (- = file, d = dir, l = link)
```

| Character | Permission | Numeric value |
|-----------|-----------|--------------|
| `r` | Read | 4 |
| `w` | Write | 2 |
| `x` | Execute | 1 |
| `-` | No permission | 0 |

### Numeric (Octal) Permissions

```
rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
--- = 0+0+0 = 0
```

Common permission sets:

| Octal | Symbolic | Meaning |
|-------|----------|---------|
| `755` | `rwxr-xr-x` | Owner: all; Group/Other: read+execute (directories, executables) |
| `644` | `rw-r--r--` | Owner: read+write; Group/Other: read-only (regular files) |
| `600` | `rw-------` | Owner only, read+write (SSH keys, sensitive configs) |
| `700` | `rwx------` | Owner only, all permissions (private scripts) |
| `777` | `rwxrwxrwx` | Everyone full access (avoid in production!) |
| `000` | `----------` | No permissions |

### Changing Permissions with `chmod`

```bash
# Symbolic mode
chmod u+x script.sh      # add execute for owner (user)
chmod g+w file.txt       # add write for group
chmod o-r file.txt       # remove read from others
chmod a+x script.sh      # add execute for all (a = all)
chmod u+x,g-w file.txt   # multiple changes

# Numeric mode
chmod 755 script.sh      # rwxr-xr-x
chmod 644 config.txt     # rw-r--r--
chmod 600 ~/.ssh/id_rsa  # rw------- (required for SSH keys!)

# Recursive (apply to all files in directory)
chmod -R 755 /var/www/html/

# Tip: make a script executable
chmod +x deploy.sh
./deploy.sh
```

### Changing Ownership with `chown`

```bash
# Change file owner
chown tom file.txt

# Change owner and group
chown tom:devops file.txt

# Change group only
chown :devops file.txt
chgrp devops file.txt    # same thing

# Recursive
chown -R tom:devops /var/www/html/
chown -R nginx:nginx /etc/nginx/

# Check ownership
ls -l file.txt
stat file.txt
```

### Special Permissions

```bash
# Setuid (SUID) — run as file owner, not current user
chmod u+s /usr/bin/passwd   # passwd always runs as root
ls -l /usr/bin/passwd       # shows: -rwsr-xr-x

# Setgid (SGID) — files inherit directory group
chmod g+s /shared/teamdir/
# new files in this dir get the dir's group, not creator's group

# Sticky bit — only owner can delete files (used on /tmp)
chmod +t /shared/public/
ls -ld /tmp/  # shows: drwxrwxrwt
```

---

## File Search

### `find` — Search by Properties

```bash
# Find by name
find /home -name "*.txt"
find / -name "nginx.conf"

# Case-insensitive name search
find /etc -iname "*.conf"

# Find by type
find /var -type f     # files only
find /var -type d     # directories only
find /var -type l     # symlinks only

# Find by size
find / -size +100M    # larger than 100MB
find / -size -1k      # smaller than 1KB
find /tmp -size +0c -size -10k   # between 0 and 10KB

# Find by modification time
find /var/log -mtime -1    # modified in last 1 day
find /tmp -mtime +7        # older than 7 days
find /home -mmin -60       # modified in last 60 minutes

# Find by permissions
find /home -perm 777       # exactly 777
find / -perm /u+s          # has SUID bit

# Find by owner
find /home -user tom
find /var -group www-data

# Execute command on results
find /tmp -name "*.tmp" -delete                    # delete found files
find /var/log -name "*.log" -exec gzip {} \;      # gzip all log files
find /home -type f -exec chmod 644 {} \;           # fix permissions

# Combine conditions
find /var/log -name "*.log" -size +10M -mtime -7
```

### `grep` — Search File Contents

```bash
# Basic search
grep "error" /var/log/syslog
grep "nginx" /etc/passwd

# Case-insensitive
grep -i "error" /var/log/syslog

# Show line numbers
grep -n "error" /var/log/syslog

# Invert match (lines NOT containing pattern)
grep -v "DEBUG" app.log

# Recursive search in directory
grep -r "TODO" /home/tom/projects/
grep -r "password" /etc/ 2>/dev/null

# Count matches
grep -c "ERROR" app.log

# Show only filenames with matches
grep -l "error" /var/log/*.log

# Context: lines before/after match
grep -A 3 "ERROR" app.log   # 3 lines after
grep -B 2 "ERROR" app.log   # 2 lines before
grep -C 3 "ERROR" app.log   # 3 lines before and after

# Extended regex (use | for OR, + for one-or-more)
grep -E "error|warning|critical" app.log
grep -E "^[0-9]{4}-[0-9]{2}" app.log   # lines starting with date

# Fixed string (faster, no regex interpretation)
grep -F "1.2.3.4" /var/log/nginx/access.log
```

---

## Text Processing

### `cut` — Extract Fields

```bash
# Cut by delimiter (field 1 from colon-separated)
cut -d: -f1 /etc/passwd        # usernames only
cut -d: -f1,3 /etc/passwd      # username and UID
cut -d, -f2 data.csv           # second CSV column

# Cut by character position
cut -c1-10 file.txt            # first 10 characters
```

### `sort` — Sort Lines

```bash
# Sort alphabetically
sort names.txt

# Sort in reverse
sort -r names.txt

# Sort numerically
sort -n numbers.txt

# Sort by specific field (column 3, colon-separated)
sort -t: -k3 -n /etc/passwd    # sort by UID

# Remove duplicates
sort -u names.txt

# Sort by size (ls output)
ls -l | sort -k5 -n            # sort by file size
```

### `awk` — Pattern Processing & Field Extraction

```bash
# Print specific column ($1 = first field)
awk '{print $1}' file.txt
awk '{print $1, $3}' file.txt

# Custom delimiter
awk -F: '{print $1}' /etc/passwd          # usernames
awk -F: '{print $1, $3}' /etc/passwd      # username UID

# Filter rows matching pattern
awk '/error/ {print}' app.log
awk '$3 > 1000 {print $1, $3}' /etc/passwd  # UIDs > 1000

# Print specific line range
awk 'NR==5,NR==10 {print}' file.txt

# Calculate sum
awk '{sum += $1} END {print sum}' numbers.txt
```

### `sed` — Stream Editor

```bash
# Substitute (replace first occurrence per line)
sed 's/old/new/' file.txt

# Substitute all occurrences (global)
sed 's/old/new/g' file.txt

# Edit file in-place
sed -i 's/localhost/0.0.0.0/g' config.txt

# Delete lines matching pattern
sed '/^#/d' config.txt        # delete comment lines
sed '/^$/d' config.txt        # delete empty lines

# Print specific line number
sed -n '5p' file.txt          # print line 5
sed -n '5,10p' file.txt       # print lines 5-10

# Insert line after match
sed '/^server/a port=8080' config.txt

# Real DevOps use case: update config in deployment
sed -i "s/DB_HOST=.*/DB_HOST=${DB_HOST}/" .env
```

---

## Links

### Hard Links

```bash
# Hard link: another name for the same inode (same file data)
ln original.txt hardlink.txt

# Both point to same data; deleting one doesn't remove data
# Hard links cannot span filesystems or link to directories

ls -li original.txt hardlink.txt   # same inode number
```

### Symbolic (Soft) Links

```bash
# Symlink: a pointer/shortcut to another file or directory
ln -s /opt/myapp/current /usr/local/bin/myapp
ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/mysite

# List symlinks
ls -la   # shows: myapp -> /opt/myapp/current

# Remove a symlink (do NOT use rm -rf on directory symlinks!)
rm mylink
unlink mylink

# Update a symlink
ln -sf /opt/myapp/v2.0 /usr/local/bin/myapp   # -f = force overwrite

# Real DevOps use: version pinning
ln -s /usr/bin/python3 /usr/local/bin/python
```

---

## Archives and Compression

### `tar` — Archive Files

```bash
# Create archive
tar -cvf archive.tar /path/to/dir/     # c=create, v=verbose, f=file

# Create compressed archive (gzip)
tar -czvf archive.tar.gz /path/to/dir/

# Create compressed archive (bzip2 — slower but smaller)
tar -cjvf archive.tar.bz2 /path/to/dir/

# Create compressed archive (xz — even smaller)
tar -cJvf archive.tar.xz /path/to/dir/

# Extract archive
tar -xvf archive.tar
tar -xzvf archive.tar.gz
tar -xjvf archive.tar.bz2

# Extract to specific directory
tar -xzvf archive.tar.gz -C /opt/

# List archive contents without extracting
tar -tvf archive.tar.gz

# Extract single file from archive
tar -xzvf archive.tar.gz path/to/specific/file.txt
```

### Compression Tools

```bash
# gzip — compress (replaces original file)
gzip largefile.log          # creates largefile.log.gz
gzip -k largefile.log       # keep original (-k = keep)
gzip -d file.gz             # decompress
gunzip file.gz              # same as gzip -d

# bzip2
bzip2 file.txt              # creates file.txt.bz2
bunzip2 file.txt.bz2

# xz (best compression ratio)
xz file.txt
unxz file.txt.xz

# zip/unzip (Windows compatible)
zip -r archive.zip mydir/
unzip archive.zip
unzip archive.zip -d /tmp/  # extract to /tmp

# View compressed file without extracting
zcat file.gz
zless file.gz
bzcat file.bz2
```

---

## `umask` — Default Permissions

```bash
# View current umask
umask         # e.g., 0022

# umask is subtracted from default permissions:
# Files: 666 - 022 = 644 (rw-r--r--)
# Dirs:  777 - 022 = 755 (rwxr-xr-x)

# Set umask for current session
umask 027    # group: rx only, others: none
# Files: 666 - 027 = 640 (rw-r-----)
# Dirs:  777 - 027 = 750 (rwxr-x---)

# Set permanently: add to ~/.bashrc
echo "umask 027" >> ~/.bashrc
```

---

## Day 2 Practice Exercises

1. Create a directory `~/lab/day2/` with subdirectories `scripts/`, `configs/`, `logs/`.
2. Create a file `~/lab/day2/configs/app.conf` with three key=value pairs.
3. Set the permissions so only you can read/write `app.conf` (mode 600).
4. Create a script `~/lab/day2/scripts/hello.sh` with `echo "hello"`, make it executable, and run it.
5. Create a symlink `/tmp/myconfig` pointing to `~/lab/day2/configs/app.conf`.
6. In `/etc/passwd`, find all users with UID >= 1000 using `awk`.
7. Archive your entire `~/lab/day2/` directory into `~/day2-backup.tar.gz`.
8. Use `find` to locate all `.conf` files in `/etc` modified in the last 7 days.
9. Use `grep` with context to find "root" in `/etc/passwd` and show 2 lines after each match.
10. Use `sed` to replace all occurrences of `localhost` with `127.0.0.1` in a copy of `/etc/hosts`.

---

## Summary

| Concept | Key Commands |
|---------|-------------|
| View files | `cat`, `less`, `head`, `tail`, `tail -f` |
| Copy/Move/Delete | `cp`, `mv`, `rm -rf` |
| Permissions | `chmod 755`, `chmod u+x`, `chown user:group` |
| Search files | `find / -name`, `find -type f -mtime` |
| Search content | `grep -r`, `grep -i`, `grep -n` |
| Text processing | `cut -d: -f1`, `sort -n`, `awk '{print $1}'`, `sed 's/old/new/g'` |
| Links | `ln` (hard), `ln -s` (symlink) |
| Archives | `tar -czvf`, `tar -xzvf`, `gzip`, `zip` |

---

[Previous: Day 1](../day-1/README.md) | [Next: Day 3 - Users, Groups & Processes](../day-3/README.md)
