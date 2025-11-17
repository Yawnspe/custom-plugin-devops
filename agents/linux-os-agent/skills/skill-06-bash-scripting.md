# Skill 06: Bash Scripting & Automation

> Write production-grade Bash scripts to automate tasks and system administration

## 📚 Overview

Bash scripting enables automation of repetitive tasks, system administration, and infrastructure management. Essential for DevOps engineers.

---

## 🏗️ Core Concepts

### Script Basics

```bash
#!/bin/bash
# Shebang tells system to use /bin/bash
# First line of every script

# Comments start with #

# Variable assignment
name="John"
age=25
path="/usr/local/bin"

# Variables with spaces (use quotes)
message="Hello, World!"
long_value="This is a long string"

# Variable expansion
echo "Hello, $name"          # String interpolation
echo "Hello, ${name}!"       # Explicit delimiters (safer)
echo "$age years old"
echo ${age}

# Command substitution
current_date=$(date +%Y-%m-%d)
file_count=$(ls -1 | wc -l)
echo "Today is $current_date"

# Command substitution (legacy)
current_date=`date +%Y-%m-%d`

# Arithmetic
count=$((10 + 5))          # count = 15
count=$((count * 2))       # count = 30
result=$((10 / 3))         # result = 3 (integer division)

# String operations
string="Hello World"
echo ${string:0:5}         # Hello (substring)
echo ${string#Hello}       # World (remove prefix)
echo ${string%World}       # Hello (remove suffix)
echo ${#string}            # 11 (length)
```

### Control Structures

```bash
# If statements
if [ $age -eq 18 ]; then
    echo "Adult"
elif [ $age -gt 18 ]; then
    echo "Older adult"
else
    echo "Minor"
fi

# Comparison operators
[ $a -eq $b ]    # Equal
[ $a -ne $b ]    # Not equal
[ $a -lt $b ]    # Less than
[ $a -le $b ]    # Less than or equal
[ $a -gt $b ]    # Greater than
[ $a -ge $b ]    # Greater than or equal

# String comparisons
[ "$str1" = "$str2" ]      # Equal
[ "$str1" != "$str2" ]     # Not equal
[ -z "$str" ]              # Empty string
[ -n "$str" ]              # Not empty string

# File tests
[ -f /path/file ]          # Regular file exists
[ -d /path/dir ]           # Directory exists
[ -e /path ]               # Path exists
[ -r /path ]               # Readable
[ -w /path ]               # Writable
[ -x /path ]               # Executable
[ -s /path ]               # File not empty

# Logical operators
[ $a -gt 0 ] && echo "Positive"      # AND
[ $a -lt 0 ] || echo "Not negative"  # OR
[ ! $a -eq 0 ]                        # NOT

# For loops
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

for file in /home/*; do
    echo "File: $file"
done

for ((i=1; i<=10; i++)); do
    echo "Count: $i"
done

# While loops
count=1
while [ $count -le 10 ]; do
    echo "Count: $count"
    count=$((count + 1))
done

# Until loops
count=1
until [ $count -gt 5 ]; do
    echo "Count: $count"
    count=$((count + 1))
done

# Case statements
case "$1" in
    start)
        echo "Starting service"
        ;;
    stop)
        echo "Stopping service"
        ;;
    restart)
        echo "Restarting service"
        ;;
    *)
        echo "Unknown command"
        ;;
esac
```

### Functions

```bash
# Define function
function greet {
    echo "Hello, $1!"
}

# Alternative syntax
greet() {
    local name=$1
    local age=$2
    echo "Hi $name, age $age"
}

# Call function
greet "Alice"
greet "Bob" "25"

# Return values
add() {
    echo $(($1 + $2))
}

result=$(add 5 3)
echo "Result: $result"

# Exit codes
check_file() {
    if [ -f "$1" ]; then
        return 0        # Success
    else
        return 1        # Failure
    fi
}

if check_file "/etc/passwd"; then
    echo "File exists"
else
    echo "File not found"
fi
```

### Error Handling

```bash
#!/bin/bash
set -e              # Exit on any error
set -u              # Error on undefined variables
set -o pipefail     # Pipe failures cause exit

# Trap errors
trap 'echo "Error occurred!"; exit 1' ERR

# Check command success
if ! mkdir /tmp/testdir; then
    echo "Failed to create directory"
    exit 1
fi

# Check exit code explicitly
ls /nonexistent
if [ $? -ne 0 ]; then
    echo "ls failed with code: $?"
fi

# Or statement for fallback
mkdir /tmp/testdir || mkdir /tmp/testdir2 || exit 1
```

### Input/Output

```bash
# Read input
read name
read -p "Enter name: " name

# Read multiple variables
read first last age
# Input: John Doe 25

# Read from file
while IFS= read -r line; do
    echo "Line: $line"
done < /path/file

# Write output
echo "Simple message"
echo -e "Line 1\nLine 2"    # Enable escape sequences
echo -n "No newline"        # No newline at end

# Redirect output
echo "Message" > file.txt           # Write (overwrite)
echo "Message" >> file.txt          # Append
echo "Error" >&2                    # Stderr

# Here documents
cat << EOF
Line 1
Line 2
EOF

# Here strings
while read line; do
    echo ">>$line"
done <<< "string"
```

### Arrays

```bash
# Create array
fruits=("apple" "banana" "orange")

# Access elements
echo ${fruits[0]}          # apple
echo ${fruits[1]}          # banana
echo ${fruits[@]}          # All elements

# Array length
echo ${#fruits[@]}         # 3

# Add to array
fruits+=("grape")

# Loop through array
for fruit in "${fruits[@]}"; do
    echo "Fruit: $fruit"
done

# Associative arrays
declare -A person
person[name]="John"
person[age]="25"
echo ${person[name]}       # John
```

---

## ✨ Best Practices

### 1. **Error Handling**
```bash
#!/bin/bash
set -euo pipefail

trap cleanup EXIT

cleanup() {
    # Cleanup code
    rm -f /tmp/tempfile
}

main() {
    # Script logic
    echo "Running..."
}

main "$@"
```

### 2. **Input Validation**
```bash
#!/bin/bash

if [ $# -ne 2 ]; then
    echo "Usage: $0 <source> <destination>"
    exit 1
fi

source=$1
destination=$2

[ -f "$source" ] || { echo "Source not found"; exit 1; }
[ -d "$destination" ] || { echo "Destination not found"; exit 1; }

cp "$source" "$destination"
```

### 3. **Logging**
```bash
#!/bin/bash

log_file="/var/log/myscript.log"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$log_file"
}

log "Script started"
# Do work
log "Script completed"
```

### 4. **Modularization**
```bash
#!/bin/bash

# Source common functions
source /usr/local/lib/common.sh

# Use functions from common.sh
check_root
log_info "Running as root"
```

---

## 📝 Real-World Examples

### System Backup Script

```bash
#!/bin/bash
set -euo pipefail

BACKUP_DIR="/backups"
SOURCE_DIR="/data"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/backup_$DATE.tar.gz"

[ -d "$SOURCE_DIR" ] || { echo "Source not found"; exit 1; }

tar -czf "$BACKUP_FILE" "$SOURCE_DIR"
echo "Backup created: $BACKUP_FILE"
```

### Log Rotation Script

```bash
#!/bin/bash

LOG_DIR="/var/log/app"
MAX_SIZE=$((10 * 1024 * 1024))  # 10MB

for logfile in $LOG_DIR/*.log; do
    size=$(stat -f%z "$logfile" 2>/dev/null || stat -c%s "$logfile" 2>/dev/null)

    if [ $size -gt $MAX_SIZE ]; then
        mv "$logfile" "$logfile.$(date +%s)"
        gzip "$logfile".* 2>/dev/null || true
    fi
done
```

---

## 🎓 Completion Checklist

- [ ] Write simple Bash scripts with variables
- [ ] Use if/else and case statements
- [ ] Create and use functions
- [ ] Implement loops (for, while, until)
- [ ] Handle input and output properly
- [ ] Error handling with set -e and trap
- [ ] Read from files and command output
- [ ] Use arrays and associative arrays
- [ ] Create production-ready scripts
- [ ] Debug scripts with set -x

**Next:** Skill 07 - System Monitoring & Diagnostics

---

**Skill Status:** ✅ Ready to Use | **Difficulty:** Intermediate | **Time:** 8 hours
