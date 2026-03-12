# Day 5: Shell Scripting

## What is Shell Scripting?

A shell script is a file containing a sequence of shell commands, executed by the shell (Bash) as if you'd typed them yourself. Shell scripts are the backbone of DevOps automation: deployments, backups, log rotation, health checks, and CI/CD pipelines all rely on them.

---

## Script Basics

### Your First Script

```bash
#!/bin/bash
# This is a comment
echo "Hello, DevOps!"
```

```bash
# Save as hello.sh, make executable, run
chmod +x hello.sh
./hello.sh

# Or run without making executable
bash hello.sh
```

### The Shebang (`#!`)

The shebang tells the OS which interpreter to use:

```bash
#!/bin/bash        # Bash (most common)
#!/bin/sh          # POSIX sh (more portable)
#!/usr/bin/env bash # Safer: finds bash in PATH (use this in scripts shared across systems)
#!/usr/bin/env python3
```

### Script Best Practices

```bash
#!/usr/bin/env bash
set -e          # exit immediately on error
set -u          # treat unset variables as errors
set -o pipefail # pipe fails if any command in pipe fails
# Combined (recommended):
set -euo pipefail

# Enable debug mode (print each command before executing)
set -x

# Script template
#!/usr/bin/env bash
set -euo pipefail

# Constants
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"

# Functions
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*"
}

error() {
    echo "[ERROR] $*" >&2
    exit 1
}

# Main
main() {
    log "Script started"
    # your code here
    log "Script completed"
}

main "$@"
```

---

## Variables

### Declaring and Using Variables

```bash
# Assign (no spaces around =)
name="Tom"
age=25
pi=3.14
is_devops=true

# Use (prefix with $)
echo "Hello, $name"
echo "You are $age years old"

# Curly braces (recommended for clarity and concatenation)
echo "Hello, ${name}!"
file="backup"
echo "${file}_2024.tar.gz"    # outputs: backup_2024.tar.gz

# Read-only (constant)
readonly MAX_RETRIES=3

# Unset a variable
unset name
```

### Variable Types and Special Variables

```bash
# String
greeting="Hello World"

# Integer (can use arithmetic)
count=0
count=$((count + 1))

# Arrays
fruits=("apple" "banana" "cherry")
echo "${fruits[0]}"           # apple
echo "${fruits[@]}"           # all elements
echo "${#fruits[@]}"          # number of elements
fruits+=("date")              # append element

# Associative arrays (dictionaries) — Bash 4+
declare -A config
config["host"]="localhost"
config["port"]="5432"
echo "${config["host"]}"

# Special variables
echo $0    # script name
echo $1    # first argument
echo $2    # second argument
echo $@    # all arguments (array)
echo $*    # all arguments (string)
echo $#    # number of arguments
echo $?    # exit code of last command (0 = success)
echo $$    # PID of current script
echo $!    # PID of last background command
echo $LINENO  # current line number
```

### Command Substitution

```bash
# Capture command output into variable
current_date=$(date +%Y-%m-%d)
hostname=$(hostname -f)
user_count=$(cat /etc/passwd | wc -l)
free_disk=$(df -h / | awk 'NR==2{print $4}')

echo "Date: $current_date"
echo "Hostname: $hostname"
echo "Free disk: $free_disk"

# Nested substitution
log_dir="/var/log/$(hostname)"
backup_file="backup-$(date +%Y%m%d-%H%M%S).tar.gz"
```

### String Operations

```bash
str="Hello, World!"

# Length
echo ${#str}                    # 13

# Substring: ${var:start:length}
echo ${str:0:5}                 # Hello
echo ${str:7}                   # World!

# Replace first occurrence
echo ${str/World/DevOps}        # Hello, DevOps!

# Replace all occurrences
echo ${str//l/L}                # HeLLo, WorLd!

# Remove prefix
path="/home/tom/scripts/deploy.sh"
echo ${path##*/}                # deploy.sh (remove longest prefix match)
echo ${path#*/}                 # home/tom/scripts/deploy.sh (remove shortest)

# Remove suffix
echo ${path%.*}                 # /home/tom/scripts/deploy (remove extension)
echo ${path%%/*}                # (empty — remove longest suffix match from right)

# Default value: use default if var is unset/empty
echo ${name:-"anonymous"}       # use "anonymous" if name unset
echo ${port:-8080}              # use 8080 if port unset

# Uppercase/lowercase (Bash 4+)
echo ${str^^}                   # HELLO, WORLD!
echo ${str,,}                   # hello, world!
```

---

## User Input

```bash
# Read from stdin
read -p "Enter your name: " username
echo "Hello, $username!"

# Read with timeout
read -t 5 -p "Enter name (5s timeout): " username

# Read silently (passwords)
read -s -p "Password: " password
echo ""   # newline after silent input

# Read with default value
read -p "Enter port [8080]: " port
port=${port:-8080}

# Read multiple values
read -p "Enter first and last name: " first last
echo "First: $first, Last: $last"

# Read into array
read -a colors <<< "red green blue"
echo ${colors[0]}   # red
```

---

## Conditionals

### `if` Statements

```bash
# Basic if
if [ condition ]; then
    commands
elif [ other_condition ]; then
    commands
else
    commands
fi

# Example
age=25
if [ $age -ge 18 ]; then
    echo "Adult"
elif [ $age -ge 13 ]; then
    echo "Teenager"
else
    echo "Child"
fi
```

### Test Conditions

```bash
# Numeric comparisons
[ $a -eq $b ]    # equal
[ $a -ne $b ]    # not equal
[ $a -lt $b ]    # less than
[ $a -le $b ]    # less than or equal
[ $a -gt $b ]    # greater than
[ $a -ge $b ]    # greater than or equal

# String comparisons
[ "$a" = "$b" ]  # equal (use = not == in POSIX sh)
[ "$a" == "$b" ] # equal (Bash)
[ "$a" != "$b" ] # not equal
[ -z "$a" ]      # empty string (zero length)
[ -n "$a" ]      # non-empty string

# File tests
[ -e file ]      # exists
[ -f file ]      # is a regular file
[ -d dir ]       # is a directory
[ -L link ]      # is a symlink
[ -r file ]      # readable
[ -w file ]      # writable
[ -x file ]      # executable
[ -s file ]      # exists and not empty (non-zero size)
[ file1 -nt file2 ]  # file1 newer than file2
[ file1 -ot file2 ]  # file1 older than file2

# Logical operators
[ $a -gt 0 ] && [ $b -gt 0 ]   # AND
[ $a -gt 0 ] || [ $b -gt 0 ]   # OR
[ ! -f file ]                   # NOT

# Double brackets [[ ]] (Bash only — preferred)
[[ "$str" == "hello" ]]         # string comparison
[[ "$str" =~ ^[0-9]+$ ]]        # regex match
[[ -f file && -r file ]]        # AND inside [[ ]]
[[ -d dir || -f file ]]         # OR inside [[ ]]
```

### `case` Statement

```bash
# case — cleaner than multiple elif
read -p "Enter environment: " env

case "$env" in
    production|prod)
        echo "Connecting to production"
        DB_HOST="prod-db.example.com"
        ;;
    staging|stage)
        echo "Connecting to staging"
        DB_HOST="staging-db.example.com"
        ;;
    development|dev)
        echo "Connecting to development"
        DB_HOST="localhost"
        ;;
    *)
        echo "Unknown environment: $env"
        exit 1
        ;;
esac
```

---

## Loops

### `for` Loop

```bash
# Loop over a list
for fruit in apple banana cherry; do
    echo "Fruit: $fruit"
done

# Loop over array
servers=("web1" "web2" "db1")
for server in "${servers[@]}"; do
    echo "Checking $server..."
    ping -c 1 "$server" && echo "$server is up" || echo "$server is DOWN"
done

# Loop with range (C-style)
for ((i=1; i<=5; i++)); do
    echo "Iteration $i"
done

# Loop over files
for file in /var/log/*.log; do
    echo "Processing: $file"
    gzip "$file"
done

# Loop over command output
for user in $(awk -F: '$3 >= 1000 {print $1}' /etc/passwd); do
    echo "Regular user: $user"
done

# Loop over lines in a file
while IFS= read -r line; do
    echo "Line: $line"
done < servers.txt
```

### `while` Loop

```bash
# Basic while
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    ((count++))
done

# While with a command
while ping -c 1 server.example.com &>/dev/null; do
    echo "Server is up, waiting..."
    sleep 5
done
echo "Server is down!"

# Infinite loop with break
while true; do
    read -p "Enter command (quit to exit): " cmd
    if [[ "$cmd" == "quit" ]]; then
        break
    fi
    eval "$cmd"
done

# Read file line by line
while IFS= read -r line; do
    echo "Processing: $line"
done < /etc/hosts

# Process command output line by line
ps aux | while IFS= read -r line; do
    echo "$line"
done
```

### `until` Loop

```bash
# until: loops UNTIL condition is true (opposite of while)
retries=0
until curl -s http://myapp.example.com/health | grep -q '"status":"ok"'; do
    ((retries++))
    if [ $retries -ge 10 ]; then
        echo "App failed to start after $retries attempts"
        exit 1
    fi
    echo "Waiting for app to start... attempt $retries"
    sleep 5
done
echo "App is healthy!"
```

### Loop Control

```bash
# break — exit loop
for i in 1 2 3 4 5; do
    if [ $i -eq 3 ]; then
        break    # exit the loop
    fi
    echo $i
done

# continue — skip rest of iteration
for i in 1 2 3 4 5; do
    if [ $i -eq 3 ]; then
        continue    # skip this iteration
    fi
    echo $i
done

# break/continue with nested loops (specify level)
for i in 1 2 3; do
    for j in a b c; do
        if [[ "$j" == "b" ]]; then
            break 2    # break out of both loops
        fi
        echo "$i-$j"
    done
done
```

---

## Functions

```bash
# Define a function
greet() {
    echo "Hello, $1!"
}

# Call it
greet "Tom"
greet "DevOps"

# Function with multiple params
create_user() {
    local username=$1
    local group=$2
    local shell=${3:-/bin/bash}    # default shell

    sudo useradd -m -s "$shell" -G "$group" "$username"
    echo "Created user: $username in group: $group"
}

create_user "alice" "devops"
create_user "bob" "devops" "/bin/sh"

# Return values
# Bash functions return exit codes (0-255), not values
# Use echo to "return" a value
get_free_disk() {
    local path=${1:-/}
    df -h "$path" | awk 'NR==2{print $4}'
}

free=$(get_free_disk /var)
echo "Free space in /var: $free"

# Check function return code
is_service_running() {
    systemctl is-active --quiet "$1"
}

if is_service_running nginx; then
    echo "nginx is running"
else
    echo "nginx is NOT running"
fi

# local variables (avoid polluting global scope)
my_function() {
    local local_var="I am local"
    global_var="I am global"
}

# Recursive function
factorial() {
    local n=$1
    if [ $n -le 1 ]; then
        echo 1
    else
        local prev=$(factorial $((n - 1)))
        echo $((n * prev))
    fi
}

echo "5! = $(factorial 5)"
```

---

## Error Handling

```bash
# Check exit codes
cp file.txt /backup/
if [ $? -ne 0 ]; then
    echo "Backup failed!" >&2
    exit 1
fi

# Shorter pattern
cp file.txt /backup/ || { echo "Backup failed!" >&2; exit 1; }

# Trap: execute on exit or signals
cleanup() {
    echo "Cleaning up temp files..."
    rm -f /tmp/script-$$-*
}
trap cleanup EXIT          # always run on exit
trap cleanup EXIT INT TERM # also on Ctrl+C and kill

# Trap errors (with set -e)
set -e
trap 'echo "Error on line $LINENO"; exit 1' ERR

# Example: robust script with cleanup
#!/usr/bin/env bash
set -euo pipefail

TEMP_FILE=$(mktemp)
LOCK_FILE="/var/run/myapp.lock"

cleanup() {
    rm -f "$TEMP_FILE" "$LOCK_FILE"
    echo "Cleanup done"
}
trap cleanup EXIT

# Check if already running (locking)
if [ -f "$LOCK_FILE" ]; then
    echo "Script already running!" >&2
    exit 1
fi
touch "$LOCK_FILE"

# ... script logic ...
```

---

## Script Arguments and Argument Parsing

```bash
#!/usr/bin/env bash
# Usage: ./deploy.sh -e production -v 1.2.3

usage() {
    echo "Usage: $0 -e <environment> -v <version> [-d]"
    echo "  -e  Environment (dev/staging/prod)"
    echo "  -v  Version to deploy"
    echo "  -d  Dry run"
    exit 1
}

DRY_RUN=false

while getopts "e:v:dh" opt; do
    case $opt in
        e) ENVIRONMENT="$OPTARG" ;;
        v) VERSION="$OPTARG" ;;
        d) DRY_RUN=true ;;
        h) usage ;;
        *) usage ;;
    esac
done

# Validate required args
[[ -z "${ENVIRONMENT:-}" ]] && { echo "Error: -e required"; usage; }
[[ -z "${VERSION:-}" ]] && { echo "Error: -v required"; usage; }

echo "Deploying version $VERSION to $ENVIRONMENT"
echo "Dry run: $DRY_RUN"
```

---

## Practical Script Examples

### Health Check Script

```bash
#!/usr/bin/env bash
set -euo pipefail

SERVICES=("nginx" "postgresql" "redis")
ALERT_EMAIL="admin@example.com"
LOG_FILE="/var/log/healthcheck.log"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

check_service() {
    local service=$1
    if systemctl is-active --quiet "$service"; then
        log "OK: $service is running"
        return 0
    else
        log "FAIL: $service is NOT running"
        return 1
    fi
}

failed_services=()

for service in "${SERVICES[@]}"; do
    check_service "$service" || failed_services+=("$service")
done

if [ ${#failed_services[@]} -gt 0 ]; then
    log "ALERT: Failed services: ${failed_services[*]}"
    echo "Services down: ${failed_services[*]}" | mail -s "Service Alert" "$ALERT_EMAIL"
    exit 1
fi

log "All services healthy"
```

### Backup Script

```bash
#!/usr/bin/env bash
set -euo pipefail

BACKUP_DIR="/backups"
SOURCE_DIR="/var/www/html"
RETENTION_DAYS=7
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/web-backup-${TIMESTAMP}.tar.gz"

log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*"; }

# Create backup directory if needed
mkdir -p "$BACKUP_DIR"

# Create backup
log "Starting backup of $SOURCE_DIR"
tar -czf "$BACKUP_FILE" "$SOURCE_DIR"
log "Backup created: $BACKUP_FILE ($(du -sh "$BACKUP_FILE" | cut -f1))"

# Cleanup old backups
log "Removing backups older than $RETENTION_DAYS days"
find "$BACKUP_DIR" -name "web-backup-*.tar.gz" -mtime +$RETENTION_DAYS -delete

log "Backup complete. Current backups:"
ls -lh "$BACKUP_DIR"/web-backup-*.tar.gz
```

---

## Day 5 Practice Exercises

1. Write a script that accepts a filename as argument, checks it exists, and prints its line count, word count, and size.
2. Write a script that loops through a list of servers and pings each one, reporting which are up or down.
3. Create a function `check_disk()` that accepts a path and a threshold (%), and exits with error if usage exceeds the threshold.
4. Write a script using `getopts` that accepts `-h host`, `-p port`, and `-t timeout` flags for an SSH connectivity check.
5. Write a backup script that archives a directory with a timestamp, and deletes backups older than 14 days.
6. Write a while loop that keeps retrying an HTTP health endpoint every 5 seconds until it returns 200, or fails after 10 attempts.
7. Create a script with a `trap` that cleans up a temp file and prints "Interrupted!" when you press Ctrl+C.
8. Write a script that reads `/etc/passwd` and prints only users with `/home` directories and bash as their shell.

---

## Summary

| Concept | Syntax |
|---------|--------|
| Shebang | `#!/usr/bin/env bash` |
| Set flags | `set -euo pipefail` |
| Variable | `name="value"`, `echo "$name"` |
| Command sub | `result=$(command)` |
| Conditional | `if [[ condition ]]; then ... fi` |
| Case | `case "$var" in pattern) ;; esac` |
| For loop | `for item in list; do ... done` |
| While loop | `while [[ condition ]]; do ... done` |
| Function | `func() { local var=$1; ... }` |
| Arguments | `$1`, `$@`, `$#`, `getopts` |
| Error handling | `command || exit 1`, `trap cleanup EXIT` |
| Debug | `set -x`, `bash -x script.sh` |

---

[Previous: Day 4](../day-4/README.md) | [Next: Day 6 - System Administration](../day-6/README.md)
