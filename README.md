# 🐚 Bash Scripting Lab

> From beginner basics to real sysadmin automation scripts — running on Ubuntu (WSL2)

![Bash](https://img.shields.io/badge/Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat&logo=linux&logoColor=black)
![Ubuntu](https://img.shields.io/badge/Ubuntu_22.04-E95420?style=flat&logo=ubuntu&logoColor=white)
![WSL2](https://img.shields.io/badge/WSL2-0078D4?style=flat&logo=windows&logoColor=white)

---

## 📋 Project Overview

This lab covers Bash scripting from the ground up — starting with core concepts like variables, loops, and conditionals, then building up to real sysadmin automation scripts used in IT environments. All scripts were written and tested on Ubuntu 22.04 running inside WSL2 on Windows 11.

| Detail | Value |
|---|---|
| **Environment** | Ubuntu 22.04 LTS (WSL2) |
| **Shell** | Bash (GNU bash 5.x) |
| **Editor** | nano |
| **Scripts written** | 8 |
| **Topics covered** | Variables, loops, conditionals, functions, user management, backups, disk monitoring, log parsing |

---

## 📁 Scripts in This Lab

| Script | What it does |
|---|---|
| `hello.sh` | Hello world — variables and echo |
| `input.sh` | User input and conditionals |
| `loops.sh` | For and while loops |
| `functions.sh` | Functions and return values |
| `create_users.sh` | Bulk user creation from a list |
| `backup.sh` | Automated directory backup with timestamps |
| `disk_alert.sh` | Disk usage monitor with threshold alerts |
| `log_parser.sh` | Parse system logs and summarize errors |

---

## 🔰 Part 1 — Bash Basics

### Getting Started

Open your WSL Ubuntu terminal. Create a folder for all your scripts:
```bash
mkdir ~/bash-lab
cd ~/bash-lab
```

Every bash script starts with a **shebang line** that tells the system which interpreter to use:
```bash
#!/bin/bash
```

To make a script executable:
```bash
chmod +x scriptname.sh
```

To run a script:
```bash
./scriptname.sh
```

---

### Script 1 — Variables and Echo (`hello.sh`)

```bash
nano hello.sh
```

```bash
#!/bin/bash

# Variables — no spaces around the = sign
NAME="Woori"
DATE=$(date)           # Command substitution — runs a command and stores output
OS=$(uname -o)

echo "Hello, $NAME!"
echo "Today is: $DATE"
echo "Operating system: $OS"

# Double quotes preserve spaces, single quotes treat everything literally
echo "My name is $NAME"    # prints: My name is Woori
echo 'My name is $NAME'    # prints: My name is $NAME
```

```bash
chmod +x hello.sh
./hello.sh
```

**Key concepts:** Variables, `echo`, command substitution with `$()`, quoting rules

---

### Script 2 — User Input and Conditionals (`input.sh`)

```bash
nano input.sh
```

```bash
#!/bin/bash

# Read user input
echo "Enter your name:"
read NAME

echo "Enter your age:"
read AGE

# if / elif / else
if [ $AGE -lt 18 ]; then
    echo "Hello $NAME, you are a minor."
elif [ $AGE -ge 18 ] && [ $AGE -lt 65 ]; then
    echo "Hello $NAME, you are an adult."
else
    echo "Hello $NAME, you are a senior."
fi

# String comparison
echo "Enter yes or no:"
read ANSWER

if [ "$ANSWER" == "yes" ]; then
    echo "You said yes!"
elif [ "$ANSWER" == "no" ]; then
    echo "You said no!"
else
    echo "Invalid input."
fi
```

```bash
chmod +x input.sh
./input.sh
```

**Comparison operators:**
| Operator | Meaning |
|---|---|
| `-eq` | Equal to (numbers) |
| `-ne` | Not equal to (numbers) |
| `-lt` | Less than |
| `-gt` | Greater than |
| `-le` | Less than or equal |
| `-ge` | Greater than or equal |
| `==` | Equal to (strings) |
| `!=` | Not equal to (strings) |

---

### Script 3 — Loops (`loops.sh`)

```bash
nano loops.sh
```

```bash
#!/bin/bash

# For loop — iterate over a list
echo "=== For Loop ==="
for FRUIT in apple banana cherry mango; do
    echo "Fruit: $FRUIT"
done

# For loop — numeric range
echo ""
echo "=== Counting 1 to 5 ==="
for i in {1..5}; do
    echo "Number: $i"
done

# While loop
echo ""
echo "=== While Loop ==="
COUNT=1
while [ $COUNT -le 5 ]; do
    echo "Count: $COUNT"
    COUNT=$((COUNT + 1))   # arithmetic expression
done

# Loop through files in a directory
echo ""
echo "=== Files in current directory ==="
for FILE in ~/bash-lab/*; do
    echo "Found file: $FILE"
done
```

```bash
chmod +x loops.sh
./loops.sh
```

---

### Script 4 — Functions (`functions.sh`)

```bash
nano functions.sh
```

```bash
#!/bin/bash

# Define a function
greet() {
    local NAME=$1    # $1 is the first argument passed to the function
    echo "Hello, $NAME! Welcome to the bash lab."
}

# Function with return value
add_numbers() {
    local NUM1=$1
    local NUM2=$2
    local RESULT=$((NUM1 + NUM2))
    echo $RESULT     # echo is used to "return" values in bash
}

# Function to check if a file exists
check_file() {
    local FILE=$1
    if [ -f "$FILE" ]; then
        echo "File $FILE exists."
    else
        echo "File $FILE does not exist."
    fi
}

# Call the functions
greet "Woori"
greet "Admin"

TOTAL=$(add_numbers 15 27)
echo "15 + 27 = $TOTAL"

check_file "hello.sh"
check_file "fakefile.txt"
```

```bash
chmod +x functions.sh
./functions.sh
```

**File test operators:**
| Operator | Meaning |
|---|---|
| `-f file` | True if file exists and is a regular file |
| `-d dir` | True if directory exists |
| `-e path` | True if path exists (file or directory) |
| `-r file` | True if file is readable |
| `-w file` | True if file is writable |
| `-s file` | True if file is not empty |

---

## ⚙️ Part 2 — Sysadmin Automation Scripts

---

### Script 5 — Bulk User Creation (`create_users.sh`)

This script reads a list of usernames from a file and creates them automatically — exactly what you'd do in a real IT environment when onboarding multiple employees at once.

**First create a file with usernames:**
```bash
nano users.txt
```
```
jsmith
mjohnson
swilliams
abrown
```

**Now create the script:**
```bash
nano create_users.sh
```

```bash
#!/bin/bash

# Must be run as root
if [ "$EUID" -ne 0 ]; then
    echo "ERROR: Please run this script as root (use sudo)"
    exit 1
fi

# Check if users file was provided
if [ ! -f "$1" ]; then
    echo "ERROR: Users file not found."
    echo "Usage: sudo ./create_users.sh users.txt"
    exit 1
fi

USERFILE=$1
CREATED=0
SKIPPED=0
LOGFILE="/var/log/user_creation.log"

echo "==============================="
echo " Bulk User Creation Script"
echo " $(date)"
echo "==============================="

# Read each line from the file
while IFS= read -r USERNAME; do

    # Skip empty lines
    [ -z "$USERNAME" ] && continue

    # Check if user already exists
    if id "$USERNAME" &>/dev/null; then
        echo "[SKIP] User '$USERNAME' already exists."
        echo "$(date) - SKIPPED: $USERNAME already exists" >> $LOGFILE
        SKIPPED=$((SKIPPED + 1))
    else
        # Create the user with a home directory
        useradd -m -s /bin/bash "$USERNAME"

        # Set a default password (username as password — force change on login)
        echo "$USERNAME:ChangeMe123!" | chpasswd
        passwd -e "$USERNAME"   # force password change on first login

        echo "[OK] Created user '$USERNAME' with home directory."
        echo "$(date) - CREATED: $USERNAME" >> $LOGFILE
        CREATED=$((CREATED + 1))
    fi

done < "$USERFILE"

echo ""
echo "==============================="
echo " Summary"
echo " Created : $CREATED users"
echo " Skipped : $SKIPPED users"
echo " Log     : $LOGFILE"
echo "==============================="
```

```bash
chmod +x create_users.sh
sudo ./create_users.sh users.txt
```

**Verify users were created:**
```bash
cat /etc/passwd | grep -E "jsmith|mjohnson|swilliams|abrown"
ls /home/
```

---

### Script 6 — Automated Backup (`backup.sh`)

This script backs up a directory, adds a timestamp to the filename, and removes backups older than 7 days — a common task in IT operations.

```bash
nano backup.sh
```

```bash
#!/bin/bash

# ── Configuration ──────────────────────────────
SOURCE_DIR="$HOME/bash-lab"          # Directory to back up
BACKUP_DIR="$HOME/backups"           # Where to store backups
RETENTION_DAYS=7                     # Delete backups older than this
LOGFILE="$HOME/backups/backup.log"
# ───────────────────────────────────────────────

# Create backup directory if it doesn't exist
mkdir -p "$BACKUP_DIR"

# Generate timestamped filename
TIMESTAMP=$(date +"%Y-%m-%d_%H-%M-%S")
BACKUP_FILE="$BACKUP_DIR/bash-lab-backup-$TIMESTAMP.tar.gz"

echo "==============================="
echo " Automated Backup Script"
echo " $(date)"
echo "==============================="

# Check if source directory exists
if [ ! -d "$SOURCE_DIR" ]; then
    echo "ERROR: Source directory $SOURCE_DIR not found."
    echo "$(date) - ERROR: Source not found" >> "$LOGFILE"
    exit 1
fi

# Create the backup
echo "Backing up: $SOURCE_DIR"
echo "Destination: $BACKUP_FILE"

tar -czf "$BACKUP_FILE" "$SOURCE_DIR" 2>/dev/null

# Check if backup succeeded
if [ $? -eq 0 ]; then
    FILESIZE=$(du -sh "$BACKUP_FILE" | cut -f1)
    echo "[OK] Backup created successfully. Size: $FILESIZE"
    echo "$(date) - SUCCESS: $BACKUP_FILE ($FILESIZE)" >> "$LOGFILE"
else
    echo "[FAIL] Backup failed!"
    echo "$(date) - FAILED: $BACKUP_FILE" >> "$LOGFILE"
    exit 1
fi

# Remove backups older than retention period
echo ""
echo "Cleaning up backups older than $RETENTION_DAYS days..."
DELETED=$(find "$BACKUP_DIR" -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete -print | wc -l)
echo "[OK] Removed $DELETED old backup(s)."
echo "$(date) - CLEANUP: Removed $DELETED old backups" >> "$LOGFILE"

# Show all current backups
echo ""
echo "Current backups:"
ls -lh "$BACKUP_DIR"/*.tar.gz 2>/dev/null || echo "No backups found."

echo ""
echo "==============================="
echo " Backup complete."
echo "==============================="
```

```bash
chmod +x backup.sh
./backup.sh
```

**Automate it with cron to run every day at 2 AM:**
```bash
crontab -e
```
Add this line:
```
0 2 * * * /home/your-username/bash-lab/backup.sh
```

---

### Script 7 — Disk Usage Monitor (`disk_alert.sh`)

This script checks disk usage on all mounted drives and sends an alert if any partition exceeds a defined threshold — commonly used in IT monitoring.

```bash
nano disk_alert.sh
```

```bash
#!/bin/bash

# ── Configuration ──────────────────────────────
THRESHOLD=80          # Alert if disk usage exceeds this percentage
LOGFILE="$HOME/disk_alerts.log"
# ───────────────────────────────────────────────

echo "==============================="
echo " Disk Usage Monitor"
echo " $(date)"
echo " Alert threshold: ${THRESHOLD}%"
echo "==============================="

ALERT_COUNT=0

# Loop through all mounted filesystems
while IFS= read -r LINE; do

    # Extract usage percentage and mount point
    USAGE=$(echo "$LINE" | awk '{print $5}' | tr -d '%')
    MOUNT=$(echo "$LINE" | awk '{print $6}')
    PARTITION=$(echo "$LINE" | awk '{print $1}')

    # Skip header line
    [ "$USAGE" = "Use%" ] && continue

    # Check if usage exceeds threshold
    if [ "$USAGE" -ge "$THRESHOLD" ] 2>/dev/null; then
        echo "[ALERT] $MOUNT is at ${USAGE}% usage! (Partition: $PARTITION)"
        echo "$(date) - ALERT: $MOUNT at ${USAGE}%" >> "$LOGFILE"
        ALERT_COUNT=$((ALERT_COUNT + 1))
    else
        echo "[OK]    $MOUNT is at ${USAGE}% usage."
    fi

done <<< "$(df -h | awk 'NR>1')"

echo ""
echo "==============================="
if [ $ALERT_COUNT -gt 0 ]; then
    echo " ⚠️  $ALERT_COUNT partition(s) above threshold!"
    echo " Check log: $LOGFILE"
else
    echo " ✅ All partitions within normal range."
fi
echo "==============================="
```

```bash
chmod +x disk_alert.sh
./disk_alert.sh
```

**Test with a lower threshold to trigger an alert:**
```bash
# Temporarily set threshold to 1% to see alert output
THRESHOLD=1 ./disk_alert.sh
```

---

### Script 8 — Log Parser (`log_parser.sh`)

This script scans system log files, counts errors and warnings, and produces a summary report — a task done daily in IT operations and NOC environments.

```bash
nano log_parser.sh
```

```bash
#!/bin/bash

# ── Configuration ──────────────────────────────
LOGFILE="/var/log/syslog"      # Log file to parse
REPORT="$HOME/log_report.txt"  # Output report file
# ───────────────────────────────────────────────

echo "===============================" | tee "$REPORT"
echo " System Log Parser" | tee -a "$REPORT"
echo " $(date)" | tee -a "$REPORT"
echo " Parsing: $LOGFILE" | tee -a "$REPORT"
echo "===============================" | tee -a "$REPORT"

# Check if log file exists
if [ ! -f "$LOGFILE" ]; then
    echo "ERROR: Log file $LOGFILE not found."
    exit 1
fi

# Count total lines
TOTAL=$(wc -l < "$LOGFILE")
echo "" | tee -a "$REPORT"
echo "Total log entries : $TOTAL" | tee -a "$REPORT"

# Count errors
ERRORS=$(grep -ci "error" "$LOGFILE")
echo "Errors found      : $ERRORS" | tee -a "$REPORT"

# Count warnings
WARNINGS=$(grep -ci "warning" "$LOGFILE")
echo "Warnings found    : $WARNINGS" | tee -a "$REPORT"

# Count failed attempts
FAILED=$(grep -ci "failed" "$LOGFILE")
echo "Failed events     : $FAILED" | tee -a "$REPORT"

# Show last 5 errors
echo "" | tee -a "$REPORT"
echo "--- Last 5 Errors ---" | tee -a "$REPORT"
grep -i "error" "$LOGFILE" | tail -5 | tee -a "$REPORT"

# Show last 5 warnings
echo "" | tee -a "$REPORT"
echo "--- Last 5 Warnings ---" | tee -a "$REPORT"
grep -i "warning" "$LOGFILE" | tail -5 | tee -a "$REPORT"

# Top 5 most frequent log sources
echo "" | tee -a "$REPORT"
echo "--- Top 5 Log Sources ---" | tee -a "$REPORT"
awk '{print $5}' "$LOGFILE" | sort | uniq -c | sort -rn | head -5 | tee -a "$REPORT"

echo "" | tee -a "$REPORT"
echo "===============================" | tee -a "$REPORT"
echo " Report saved to: $REPORT" | tee -a "$REPORT"
echo "===============================" | tee -a "$REPORT"
```

```bash
chmod +x log_parser.sh
./log_parser.sh
```

---

## 📟 Bash Quick Reference

### Special Variables
| Variable | Meaning |
|---|---|
| `$0` | Name of the script |
| `$1, $2 ...` | Arguments passed to the script |
| `$#` | Number of arguments |
| `$@` | All arguments |
| `$?` | Exit code of last command (0 = success) |
| `$$` | PID of current script |
| `$USER` | Current username |
| `$HOME` | Home directory path |

### Useful Commands Used in Scripts
| Command | What it does |
|---|---|
| `echo` | Print text to terminal |
| `read` | Read input from user |
| `chmod +x` | Make script executable |
| `grep` | Search for patterns in text |
| `awk` | Extract fields from text |
| `sed` | Find and replace in text |
| `cut` | Extract columns from text |
| `wc -l` | Count lines |
| `tee` | Print to screen AND write to file |
| `date` | Get current date/time |
| `df -h` | Show disk usage |
| `tar -czf` | Create compressed archive |
| `find` | Search for files |
| `crontab -e` | Edit scheduled tasks |

---

## 🧠 Skills Demonstrated

**Bash / Scripting**
- Variables, user input, command substitution
- Conditionals (if/elif/else) with string and numeric comparisons
- For loops and while loops
- Functions with arguments and return values
- File and directory existence checks
- Error handling and exit codes
- Reading files line by line
- Logging script output to files

**Sysadmin / IT Operations**
- Bulk user creation and management
- Automated backups with timestamping and retention policies
- Disk usage monitoring with threshold alerting
- System log parsing and report generation
- Cron job scheduling for automation
- Running scripts with elevated privileges (sudo)

---

## 🔜 Next Steps

- Add email alerts to the disk monitor using `mail` or `sendmail`
- Expand the backup script to support remote backups via `rsync` over SSH
- Build a menu-driven system administration tool combining all scripts
- Write a script that monitors services and restarts them if they go down
- Explore `cron` more deeply — schedule all scripts to run automatically

---

*Built as part of an IT home lab portfolio — 2026*
