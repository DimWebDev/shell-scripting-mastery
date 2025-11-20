# Master Class: Shell Scripting for Linux and macOS

**Course Status:** Complete - All 10 Units Delivered
**Target Shells:** Bash (Linux Default/macOS Legacy), Zsh (macOS Default)
**Prerequisites:** None.

-----

## Table of Contents

1. [Unit 1: The Command Line Interface (CLI) Fundamentals](#unit-1-the-command-line-interface-cli-fundamentals) ✅
2. [Unit 2: Introduction to Shell Scripting](#unit-2-introduction-to-shell-scripting) ✅
3. [Unit 3: Control Structures and Flow Control](#unit-3-control-structures-and-flow-control) ✅
4. [Unit 4: Functions and Script Organization](#unit-4-functions-and-script-organization) ✅
5. [Unit 5: Input/Output Handling and Text Processing](#unit-5-inputoutput-handling-and-text-processing) ✅
6. [Unit 6: String Manipulation and Regex](#unit-6-string-manipulation-and-regex) ✅
7. [Unit 7: Advanced Scripting Topics](#unit-7-advanced-scripting-topics) ✅
8. [Unit 8: Package Managers and System Administration](#unit-8-package-managers-and-system-administration) ✅
9. [Unit 9: Practical Projects](#unit-9-practical-projects) ✅
10. [Unit 10: Best Practices](#unit-10-best-practices) ✅

-----

## Unit 1: The Command Line Interface (CLI) Fundamentals

### 1.1 The Environment: Terminal vs. Shell

It is crucial to distinguish between the window you see and the program running inside it.

  * **Terminal Emulator:** The graphical application (GUI) that accepts keyboard input.
      * *macOS:* Terminal.app, iTerm2.
      * *Linux:* GNOME Terminal, Alacritty, Terminator.
  * **The Shell:** The command-line interpreter that processes your commands.
      * **Bash (Bourne Again Shell):** The standard for most Linux distributions.
      * **Zsh (Z Shell):** The default for macOS (since Catalina). It is highly customizable and compatible with Bash.

### 1.2 Navigation & File Management

The file system is a tree. You are always at a specific "node" (directory).

| Command | Name | Description |
| :--- | :--- | :--- |
| `pwd` | Print Working Directory | Shows exactly where you are in the tree. |
| `cd` | Change Directory | Moves you to a new location. `cd ..` goes up one level; `cd ~` goes home. |
| `ls` | List | Lists files. Use `ls -la` to see hidden files (starting with `.`) and details. |

**Snippet: Efficient Navigation**

```bash
# Create a directory structure
mkdir -p project/src/assets

# Move files with verbose output to see what happens
mv -v file1.txt project/src/

# Copy a directory recursively
cp -r project project_backup

# Remove a directory (Use with CAUTION)
rm -rf project_backup
```

### 1.3 Permissions

Unix permissions manage who can Read (r), Write (w), and Execute (x) a file.

  * **User (u):** The owner.
  * **Group (g):** Other users in the owner's group.
  * **Others (o):** Everyone else.

**The `chmod` Command:**
Permissions are often represented by numbers (Octal):

  * Read = 4
  * Write = 2
  * Execute = 1

Example: `chmod 755 script.sh` means:

  * Owner: 4+2+1 = 7 (Read/Write/Execute)
  * Group: 4+0+1 = 5 (Read/Execute)
  * Others: 4+0+1 = 5 (Read/Execute)

-----

## Unit 2: Introduction to Shell Scripting

### 2.1 The Shebang

Every script must start with a "shebang" line. This tells the system which interpreter to use.

  * **Bash:** `#!/bin/bash`
  * **Zsh:** `#!/bin/zsh`
  * **Portable:** `#!/usr/bin/env bash` (Recommended for cross-platform compatibility).

### 2.2 Variables and Data Types

Shell variables are loosely typed (everything is treated as a string unless specified otherwise).

**Snippet: Your First Script**

```bash
#!/bin/bash

# DEFINING VARIABLES
# No spaces around the equal sign!
FIRST_NAME="John"
AGE=30

# READING INPUT
echo "Enter your favorite color:"
read COLOR

# OUTPUT
# We use '$' to access the variable's value
echo "Hello, $FIRST_NAME. You are $AGE years old and like $COLOR."

# SPECIAL VARIABLES
echo "Script Name: $0"
echo "First Argument: $1"
echo "Process ID: $$"
```

### 2.3 Execution

To run a script, you must make it executable.

1.  Save code as `myscript.sh`.
2.  `chmod +x myscript.sh`
3.  Run it: `./myscript.sh`

-----

## Unit 3: Control Structures and Flow Control

### 3.1 Conditional Logic (If/Else)

The core of automation is decision-making. In Bash/Zsh, spaces inside the brackets `[ ]` are mandatory.

[Image of flowchart logic for if-else statement]

**Snippet: File Checks and Logic**

```bash
#!/bin/bash

FILE="config.conf"

# Check if file exists (-f)
if [ -f "$FILE" ]; then
    echo "$FILE exists."
    
    # Check if file is writable (-w)
    if [ -w "$FILE" ]; then
        echo "We can write to it."
    else
        echo "Read-only file."
    fi

else
    echo "File not found. Creating it..."
    touch "$FILE"
fi
```

### 3.2 Loops

Loops allow you to repeat actions for items in a list or while a condition is true.

**Snippet: The `For` Loop (Batch Processing)**

```bash
# C-style syntax
for ((i=1; i<=5; i++)); do
    echo "Iteration number $i"
done

# Iterating over files (Globbing)
echo "Backing up .log files..."
for file in *.log; do
    # Ensure file exists to avoid errors if no match found
    [ -e "$file" ] || continue 
    cp "$file" "${file}.bak"
    echo "Backed up $file"
done
```

**Snippet: The `While` Loop**

```bash
COUNT=0
while [ $COUNT -lt 3 ]; do
    echo "Count is $COUNT"
    ((COUNT++)) # Arithmetic expansion
done
```

-----

## Unit 4: Functions and Script Organization

### 4.1 Defining Functions

Functions prevent code repetition. Variables inside functions should be declared `local` to avoid messing up global variables.

**Snippet: Modular Scripting**

```bash
#!/bin/bash

# Global variable
LOG_FILE="script.log"

log_message() {
    local timestamp=$(date "+%Y-%m-%d %H:%M:%S")
    local level=$1
    local message=$2
    
    # Append to log file
    echo "[$timestamp] [$level] $message" >> "$LOG_FILE"
}

check_disk_space() {
    # Returns 0 (success) or 1 (failure)
    local space=$(df / | awk 'NR==2 {print $5}' | tr -d '%')
    
    if [ "$space" -gt 90 ]; then
        return 1 # Error state
    else
        return 0 # Success state
    fi
}

# Main Logic
log_message "INFO" "Script started."

if check_disk_space; then
    log_message "INFO" "Disk space is healthy."
else
    log_message "CRITICAL" "Disk space is low!"
fi
```

-----

## Unit 5: Input/Output Handling and Text Processing

### 5.1 Streams and Redirection

Linux has three standard data streams:

1.  **stdin (0):** Input (Keyboard).
2.  **stdout (1):** Output (Screen).
3.  **stderr (2):** Error messages.

**Snippet: Redirection Mastery**

```bash
# Redirect output to file (Overwrite)
ls > file_list.txt

# Redirect output to file (Append)
date >> system.log

# Redirect Error only (2>)
ls /non_existent_folder 2> error.log

# Redirect BOTH output and error to same file
./deploy_script.sh > deploy.log 2>&1
```

### 5.2 The Power Trio: Grep, Sed, Awk

  * **Grep:** Search.
  * **Sed:** Edit.
  * **Awk:** Process structured data.

**Snippet: Text Processing Pipeline**

```bash
# 1. GREP: Find lines containing "Error"
grep "Error" application.log

# 2. SED: Replace "foo" with "bar" 
# Note: On macOS, use sed -i '' 's/foo/bar/g' file.txt
echo "Hello foo" | sed 's/foo/bar/'

# 3. AWK: Print the 2nd column of a CSV
echo "Name,Age,Role" | awk -F, '{print $2}'
```

-----

## Unit 6: String Manipulation and Regex

### 6.1 String Operations

Bash has built-in capabilities to manipulate strings without external tools.

**Snippet: String Slicing and Replacement**

```bash
FILENAME="image.png"

# Extract extension
EXTENSION="${FILENAME##*.}"
echo "Extension: $EXTENSION" # Output: png

# Extract filename without extension
NAME="${FILENAME%.*}"
echo "Name: $NAME" # Output: image

# Substring (Start at index 0, length 3)
echo "${FILENAME:0:3}" # Output: ima

# Replace substring (First match)
echo "${FILENAME/png/jpg}" # Output: image.jpg
```

### 6.2 Regular Expressions (Regex)

Used primarily with `grep -E` or within script conditionals `[[ ]]`.

  * `^`: Start of line.
  * `$`: End of line.
  * `[0-9]`: Any digit.
  * `.*`: Any character repeated any number of times.

**Snippet: Validating an Email Format**

```bash
EMAIL="user@example.com"
REGEX="^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"

if [[ "$EMAIL" =~ $REGEX ]]; then
    echo "Valid email."
else
    echo "Invalid email format."
fi
```

-----

## Unit 7: Advanced Scripting Topics

### 7.1 Arrays

Arrays store lists of data.

**Snippet: Iterating Arrays**

```bash
# Declare array
SERVERS=("web01" "db01" "cache01")

# Add element
SERVERS+=("web02")

# Loop through array
for server in "${SERVERS[@]}"; do
    echo "Pinging $server..."
    # ping -c 1 $server
done

# Associative Array (Dictionary) - Bash 4.0+ / Zsh
declare -A USER_MAP
USER_MAP[admin]="root"
USER_MAP[guest]="nobody"

echo "Admin maps to: ${USER_MAP[admin]}"
```

### 7.2 Error Handling and Traps

The `trap` command allows you to catch signals (like Ctrl+C) to perform cleanup before the script exits.

**Snippet: Robust Script with Cleanup**

```bash
#!/bin/bash

TEMP_FILE=$(mktemp)

# Function to run on exit
cleanup() {
    echo "Cleaning up temporary files..."
    rm -f "$TEMP_FILE"
    echo "Done."
}

# Trap EXIT signal (happens on successful end or error)
trap cleanup EXIT

echo "Working with $TEMP_FILE..."
sleep 2
# When script ends, 'cleanup' runs automatically
```

-----

## Unit 8: Package Managers and System Administration

### 8.1 Package Managers

  * **Debian/Ubuntu:** `apt` (Advanced Package Tool).
  * **RedHat/CentOS:** `yum` or `dnf`.
  * **macOS:** `brew` (Homebrew).

**Snippet: Cross-Platform Installation Script**

```bash
install_package() {
    PACKAGE=$1
    if [[ "$OSTYPE" == "linux-gnu"* ]]; then
        sudo apt update && sudo apt install -y "$PACKAGE"
    elif [[ "$OSTYPE" == "darwin"* ]]; then
        brew install "$PACKAGE"
    fi
}

install_package "wget"
```

### 8.2 Cron (Scheduling)

Cron schedules scripts to run automatically at specific times.

**Cron Syntax:** `* * * * * /path/to/script.sh`

  * Minute (0-59)
  * Hour (0-23)
  * Day of Month (1-31)
  * Month (1-12)
  * Day of Week (0-6)

**Example:** Run backup every day at 3 AM.
`0 3 * * * /home/user/scripts/backup.sh`

-----

## Unit 9: Practical Projects

### Project 9.1: The System Health Monitor

This script combines variables, conditionals, math, and mailing.

**Snippet: health\_check.sh**

```bash
#!/bin/bash

THRESHOLD_CPU=80
THRESHOLD_DISK=90
ADMIN_EMAIL="admin@example.com"

# Get CPU Usage (Idle time subtraction method)
CPU_IDLE=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print int($1)}')
CPU_USE=$(( 100 - CPU_IDLE ))

# Get Disk Usage
DISK_USE=$(df / | awk 'NR==2 {print $5}' | tr -d '%')

echo "CPU Load: $CPU_USE%"
echo "Disk Usage: $DISK_USE%"

if [ "$CPU_USE" -gt "$THRESHOLD_CPU" ]; then
    echo "High CPU load detected!" | mail -s "Alert: High CPU" "$ADMIN_EMAIL"
fi

if [ "$DISK_USE" -gt "$THRESHOLD_DISK" ]; then
    echo "Low Disk Space!" | mail -s "Alert: Low Disk" "$ADMIN_EMAIL"
fi
```

-----

## Unit 10: Best Practices

### 10.1 Formatting and Safety

1.  **Quote your variables:** Always use `"$VAR"` instead of `$VAR` to handle spaces in filenames correctly.
2.  **Exit on Error:** Put `set -e` at the top of your script. If any command fails, the script stops immediately (preventing snowballing errors).
3.  **Naming:** Use `UPPER_CASE` for global constants and `snake_case` for functions and local variables.

### 10.2 Debugging

  * Run with `bash -x script.sh` to print every command before it executes.
  * Use `ShellCheck`: A static analysis tool (available online or via CLI) that warns you about bad syntax and common bugs.

**Snippet: The Professional Header**

```bash
#!/bin/bash
#
# Script Name: backup_system.sh
# Description: Backs up /var/www to S3
# Author:      DevOps Team
# Date:        2025-10-25
#
#######################################

set -e          # Exit on error
set -u          # Exit on unset variables
set -o pipefail # Exit if any command in a pipe fails

# ... Code begins ...
```

-----

### **Next Step**

Would you like me to generate the **full code solution for the Capstone Project** mentioned in Unit 10 (a comprehensive scripting challenge that integrates loops, functions, error handling, and user input)?