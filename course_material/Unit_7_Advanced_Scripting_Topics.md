# Unit 7: Advanced Scripting Topics

**Learning Objectives:**
- Master arrays for storing and manipulating lists of data
- Use associative arrays (dictionaries) for key-value storage
- Implement robust error handling with traps
- Handle signals and cleanup operations gracefully
- Build fault-tolerant scripts that recover from failures

**Estimated Time:** 3-4 hours

-----

## Introduction: Taking Scripts to the Next Level

You've mastered the basics: variables, functions, loops, and conditionals. Now we'll explore advanced features that make your scripts more powerful and resilient.

**What makes a script "advanced"?**
- Handling complex data structures (not just simple variables)
- Gracefully recovering from errors
- Cleaning up resources even when things go wrong
- Building scripts that can run unattended for hours or days

These topics separate quick hacks from production-ready automation tools.

-----

## 7.1 Arrays: Working with Lists of Data

### What is an Array?

An **array** is a variable that holds multiple values. Instead of storing one piece of data, it stores a list.

**Without arrays:**
```bash
SERVER1="web01"
SERVER2="db01"
SERVER3="cache01"
# This doesn't scale well!
```

**With arrays:**
```bash
SERVERS=("web01" "db01" "cache01")
# Much cleaner and scalable
```

### Creating and Accessing Arrays

#### Syntax 1: Declare with Values

```bash
#!/bin/bash

# Create an array
FRUITS=("apple" "banana" "cherry")

# Access individual elements (0-indexed)
echo "${FRUITS[0]}"  # Prints: apple
echo "${FRUITS[1]}"  # Prints: banana
echo "${FRUITS[2]}"  # Prints: cherry
```

**Important:** Always use `${ARRAY[index]}` with curly braces!

#### Syntax 2: Build Array Element by Element

```bash
#!/bin/bash

# Declare empty array
COLORS=()

# Add elements
COLORS[0]="red"
COLORS[1]="green"
COLORS[2]="blue"

echo "${COLORS[1]}"  # Prints: green
```

### Array Operations

#### Getting Array Length

```bash
#!/bin/bash

SERVERS=("web01" "db01" "cache01")

# Get number of elements
echo "Number of servers: ${#SERVERS[@]}"  # Prints: 3
```

#### Adding Elements

```bash
#!/bin/bash

TASKS=("backup" "update" "cleanup")

# Append to end of array
TASKS+=("monitor")
TASKS+=("restart")

echo "${TASKS[@]}"  # Prints: backup update cleanup monitor restart
```

#### Accessing All Elements

```bash
#!/bin/bash

SERVERS=("web01" "db01" "cache01")

# Get all elements as separate words
echo "${SERVERS[@]}"  # Prints: web01 db01 cache01

# Get all elements as a single string
echo "${SERVERS[*]}"  # Prints: web01 db01 cache01
```

**The difference between `@` and `*`:**
- `"${ARRAY[@]}"` - Each element is a separate word (use in loops!)
- `"${ARRAY[*]}"` - All elements joined into one string

### Looping Through Arrays

This is the most common use case:

```bash
#!/bin/bash

SERVERS=("web01" "db01" "cache01" "lb01")

# Loop through each element
for SERVER in "${SERVERS[@]}"; do
    echo "Pinging $SERVER..."
    # In real script: ping -c 1 "$SERVER"
done
```

**Real-world example: Backup multiple databases**

```bash
#!/bin/bash

# ⚡ REAL-WORLD EXAMPLE: Automated database backup system

# List of databases to backup
DATABASES=("users" "products" "orders" "analytics")
# ↑ Array holds all database names we need to back up

BACKUP_DIR="/var/backups/mysql"  # Where backups will be stored
TIMESTAMP=$(date +%Y%m%d_%H%M%S)  # Creates: 20240115_143022

# Create backup directory if it doesn't exist
mkdir -p "$BACKUP_DIR"  # -p = create parent directories, don't error if exists

# Backup each database
for DB in "${DATABASES[@]}"; do
    # "${DATABASES[@]}" expands to: "users" "products" "orders" "analytics"
    # Loop runs 4 times, once for each database

    echo "Backing up database: $DB"  # Progress indicator

    # Create unique backup filename: users_20240115_143022.sql.gz
    BACKUP_FILE="$BACKUP_DIR/${DB}_${TIMESTAMP}.sql.gz"

    # 💡 PIPELINE: Dump database → Compress → Save to file
    mysqldump "$DB" | gzip > "$BACKUP_FILE"
    #         │        │      └─ Step 3: Save compressed data to file
    #         │        └─ Step 2: Compress SQL data with gzip
    #         └─ Step 1: Export database as SQL

    # Check if backup succeeded
    if [ $? -eq 0 ]; then
        # $? = exit code of last command (mysqldump | gzip)
        # 0 = success
        echo "  ✓ Success: $BACKUP_FILE"
    else
        echo "  ✗ Failed: $DB"
    fi
done

echo "Backup complete!"

# Output example:
# Backing up database: users
#   ✓ Success: /var/backups/mysql/users_20240115_143022.sql.gz
# Backing up database: products
#   ✓ Success: /var/backups/mysql/products_20240115_143022.sql.gz
# ...
# Backup complete!
```

### Looping with Indices

Sometimes you need both the index and the value:

```bash
#!/bin/bash

ITEMS=("first" "second" "third")

# Loop using indices
for i in "${!ITEMS[@]}"; do
    echo "Index $i: ${ITEMS[$i]}"
done
```

**Output:**
```
Index 0: first
Index 1: second
Index 2: third
```

### Slicing Arrays

Extract a portion of an array:

```bash
#!/bin/bash

NUMBERS=(10 20 30 40 50 60)

# Get elements starting at index 2, length 3
# Syntax: ${ARRAY[@]:start:length}
SUBSET=("${NUMBERS[@]:2:3}")

echo "${SUBSET[@]}"  # Prints: 30 40 50
```

### Real-World Example: Process Log Files

```bash
#!/bin/bash

# Find all log files from today
LOG_FILES=($(find /var/log -name "*.log" -mtime 0))

echo "Found ${#LOG_FILES[@]} log files from today"
echo "========================================"

# Process each log file
for LOG in "${LOG_FILES[@]}"; do
    echo "Analyzing: $LOG"

    # Count errors
    ERROR_COUNT=$(grep -c "ERROR" "$LOG" 2>/dev/null || echo "0")

    # Count warnings
    WARN_COUNT=$(grep -c "WARN" "$LOG" 2>/dev/null || echo "0")

    echo "  Errors: $ERROR_COUNT"
    echo "  Warnings: $WARN_COUNT"
    echo ""
done
```

-----

## 7.2 Associative Arrays: Key-Value Storage

### What is an Associative Array?

An **associative array** (also called a hash, map, or dictionary) stores data as key-value pairs. Think of it like a real dictionary: you look up a word (key) to get its definition (value).

**Available in:** Bash 4.0+ and Zsh

### Creating Associative Arrays

```bash
#!/bin/bash

# MUST use declare -A to create associative array
declare -A USER_ROLES

# Assign values
USER_ROLES["alice"]="admin"
USER_ROLES["bob"]="developer"
USER_ROLES["charlie"]="guest"

# Access values by key
echo "${USER_ROLES["alice"]}"  # Prints: admin
```

### Real-World Example: Configuration Storage

```bash
#!/bin/bash

# Store application configuration
declare -A CONFIG

CONFIG["app_name"]="MyApp"
CONFIG["version"]="2.1.0"
CONFIG["port"]="8080"
CONFIG["debug"]="false"
CONFIG["db_host"]="localhost"
CONFIG["db_port"]="5432"

# Display configuration
echo "Application: ${CONFIG["app_name"]} v${CONFIG["version"]}"
echo "Running on port: ${CONFIG["port"]}"
echo "Database: ${CONFIG["db_host"]}:${CONFIG["db_port"]}"
```

### Looping Through Associative Arrays

```bash
#!/bin/bash

declare -A SERVER_IPS

SERVER_IPS["web"]="192.168.1.10"
SERVER_IPS["db"]="192.168.1.20"
SERVER_IPS["cache"]="192.168.1.30"

# Loop through keys
for SERVER in "${!SERVER_IPS[@]}"; do
    IP="${SERVER_IPS[$SERVER]}"
    echo "$SERVER: $IP"
done
```

**Output:**
```
web: 192.168.1.10
db: 192.168.1.20
cache: 192.168.1.30
```

### Checking if a Key Exists

```bash
#!/bin/bash

declare -A USERS
USERS["alice"]="admin"
USERS["bob"]="user"

# Check if key exists
if [[ -v USERS["alice"] ]]; then
    echo "Alice exists: ${USERS["alice"]}"
else
    echo "Alice not found"
fi

if [[ -v USERS["charlie"] ]]; then
    echo "Charlie exists"
else
    echo "Charlie not found"  # This will execute
fi
```

### Real-World Example: User Management

```bash
#!/bin/bash

# Store user permissions
declare -A USER_PERMISSIONS

USER_PERMISSIONS["alice"]="read,write,delete"
USER_PERMISSIONS["bob"]="read,write"
USER_PERMISSIONS["charlie"]="read"

# Function to check if user has permission
has_permission() {
    local username=$1
    local permission=$2

    # Check if user exists
    if [[ ! -v USER_PERMISSIONS["$username"] ]]; then
        return 1  # User doesn't exist
    fi

    # Check if user has the permission
    if [[ "${USER_PERMISSIONS[$username]}" == *"$permission"* ]]; then
        return 0  # Has permission
    else
        return 1  # Doesn't have permission
    fi
}

# Test permissions
if has_permission "alice" "delete"; then
    echo "Alice can delete files"
else
    echo "Alice cannot delete files"
fi

if has_permission "charlie" "delete"; then
    echo "Charlie can delete files"
else
    echo "Charlie cannot delete files"  # This executes
fi
```

### Practical Example: HTTP Status Codes

```bash
#!/bin/bash

# Map HTTP status codes to descriptions
declare -A HTTP_CODES

HTTP_CODES[200]="OK"
HTTP_CODES[201]="Created"
HTTP_CODES[400]="Bad Request"
HTTP_CODES[401]="Unauthorized"
HTTP_CODES[403]="Forbidden"
HTTP_CODES[404]="Not Found"
HTTP_CODES[500]="Internal Server Error"

# Function to explain status code
explain_status() {
    local code=$1

    if [[ -v HTTP_CODES[$code] ]]; then
        echo "$code: ${HTTP_CODES[$code]}"
    else
        echo "$code: Unknown status code"
    fi
}

# Test it
explain_status 200   # Prints: 200: OK
explain_status 404   # Prints: 404: Not Found
explain_status 999   # Prints: 999: Unknown status code
```

-----

## 7.3 Error Handling with Traps

### What is a Trap?

A **trap** catches signals (like Ctrl+C or script errors) and runs cleanup code before the script exits. This ensures resources are properly released even when things go wrong.

**Common signals:**
- `EXIT` - Script exits (normal or error)
- `INT` - User presses Ctrl+C
- `TERM` - Termination signal (from kill command)
- `ERR` - Command fails (when using `set -e`)

### Basic Trap Syntax

```bash
trap 'commands' SIGNAL
```

### Example 1: Simple Cleanup on Exit

```bash
#!/bin/bash

# ⚡ EXECUTION FLOW: Automatic cleanup with trap

# Create temporary file
TEMP_FILE=$(mktemp)
# mktemp creates: /tmp/tmp.aBc123XyZ
# ⚠️ Without cleanup, this temp file stays on disk forever!

# Define cleanup function
cleanup() {
    echo "Cleaning up temporary files..."
    rm -f "$TEMP_FILE"  # -f = force, don't error if file doesn't exist
    echo "Cleanup complete."
}

# ⚡ KEY LINE: Register the cleanup function to run on EXIT
trap cleanup EXIT
# Now, whenever the script exits (normal finish, error, or Ctrl+C), cleanup runs!

# Do some work with the temp file
echo "Working with temporary file: $TEMP_FILE"
echo "Some data" > "$TEMP_FILE"  # Write to temp file
sleep 2  # Simulate work

# When script ends (normally or with error), cleanup runs automatically
echo "Script finished."

# ═══════════════════════════════════════════════════════════════
# EXECUTION TIMELINE:
# ═══════════════════════════════════════════════════════════════
# 1. Script creates: /tmp/tmp.aBc123XyZ
# 2. Registers: "When I exit, run cleanup()"
# 3. Does work...
# 4. Script ends
# 5. ⚡ AUTOMATICALLY: cleanup() runs
# 6. Temp file deleted
#
# This works even if:
# - User presses Ctrl+C
# - Script encounters an error
# - Script completes normally
```

**What happens:**
1. Script creates temp file
2. Registers cleanup function with trap
3. Does work
4. When script exits (for ANY reason), cleanup function runs
5. Temp file is deleted

### Example 2: Handling Ctrl+C Gracefully

```bash
#!/bin/bash

# Flag to track if we should exit
SHOULD_EXIT=false

# Trap Ctrl+C (SIGINT)
interrupt_handler() {
    echo ""
    echo "Interrupt received! Cleaning up..."
    SHOULD_EXIT=true
}

trap interrupt_handler INT

# Long-running task
echo "Processing files... (Press Ctrl+C to stop gracefully)"

for i in {1..100}; do
    # Check if we should exit
    if [ "$SHOULD_EXIT" = true ]; then
        echo "Exiting gracefully..."
        break
    fi

    echo "Processing item $i..."
    sleep 1
done

echo "Task completed!"
```

### Example 3: Comprehensive Error Handling

```bash
#!/bin/bash

# ⚡ PRODUCTION-READY: Complete error handling with cleanup

# ⚠️ Exit on error - any command that fails will stop the script
set -e

# Create temporary resources
TEMP_DIR=$(mktemp -d)          # Creates: /tmp/tmp.XyZ123/
LOCK_FILE="/tmp/myscript.lock"  # Prevents multiple instances

# ═══════════════════════════════════════════════════════════════
# CLEANUP FUNCTION - Runs no matter how script exits
# ═══════════════════════════════════════════════════════════════
cleanup() {
    local exit_code=$?
    # $? captures the exit code that triggered this cleanup
    # 0 = success, non-zero = error

    echo "Running cleanup..."

    # Remove temporary directory (if it exists)
    if [ -d "$TEMP_DIR" ]; then
        rm -rf "$TEMP_DIR"
        echo "  Removed temp directory: $TEMP_DIR"
    fi

    # Remove lock file (prevents script from being stuck)
    if [ -f "$LOCK_FILE" ]; then
        rm -f "$LOCK_FILE"
        echo "  Removed lock file"
    fi

    # Exit with the same code as what triggered cleanup
    # This preserves the original error code
    exit $exit_code
}

# ⚡ REGISTER CLEANUP for multiple signals
trap cleanup EXIT INT TERM ERR
#              │    │   │    └─ Script encountered an error (with set -e)
#              │    │   └─ Received termination signal (kill command)
#              │    └─ User pressed Ctrl+C
#              └─ Script exits (any reason)

# ═══════════════════════════════════════════════════════════════
# PREVENT MULTIPLE INSTANCES (Lock file pattern)
# ═══════════════════════════════════════════════════════════════
if [ -f "$LOCK_FILE" ]; then
    echo "ERROR: Script is already running!"
    exit 1  # cleanup() will run and remove our temp files
fi

# Create lock file - signals "I'm running"
touch "$LOCK_FILE"

# ═══════════════════════════════════════════════════════════════
# ACTUAL WORK
# ═══════════════════════════════════════════════════════════════
echo "Starting work in: $TEMP_DIR"
cd "$TEMP_DIR"  # Work in temp directory

echo "Creating files..."
touch file1.txt file2.txt file3.txt

echo "Processing..."
sleep 3  # Simulate work

# 💡 TIP: Uncomment next line to test error handling
# cat /nonexistent/file  # This would trigger cleanup with error code

echo "Work complete!"

# cleanup will run automatically via trap

# ═══════════════════════════════════════════════════════════════
# WHAT HAPPENS IN DIFFERENT SCENARIOS:
# ═══════════════════════════════════════════════════════════════
# Scenario 1: Normal completion
#   → cleanup() runs → temp files deleted → lock removed → exit 0
#
# Scenario 2: User presses Ctrl+C
#   → cleanup() runs immediately → resources freed → exit 130
#
# Scenario 3: Command fails (e.g., cat /nonexistent/file)
#   → set -e catches error → cleanup() runs → exit with error code
#
# Scenario 4: Someone tries to run script again
#   → Lock file exists → script exits → cleanup() runs → exit 1
```

### Real-World Example: Database Backup with Rollback

```bash
#!/bin/bash

set -e

# Configuration
DB_NAME="production"
BACKUP_DIR="/var/backups/db"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/${DB_NAME}_${TIMESTAMP}.sql"
TEMP_BACKUP="${BACKUP_FILE}.tmp"

# Track if backup succeeded
BACKUP_SUCCESS=false

# Cleanup and rollback on failure
cleanup() {
    if [ "$BACKUP_SUCCESS" = false ]; then
        echo "Backup failed! Rolling back..."

        # Remove incomplete backup file
        if [ -f "$TEMP_BACKUP" ]; then
            rm -f "$TEMP_BACKUP"
            echo "  Removed incomplete backup"
        fi

        # Send alert (in real script)
        echo "  Would send alert to admin"

        exit 1
    else
        echo "Backup completed successfully"
    fi
}

trap cleanup EXIT

# Create backup directory
mkdir -p "$BACKUP_DIR"

echo "Starting database backup..."
echo "Database: $DB_NAME"
echo "Destination: $BACKUP_FILE"

# Perform backup (example - adjust for your database)
echo "Dumping database..."
# mysqldump "$DB_NAME" > "$TEMP_BACKUP"

# For demo, just create a file
echo "Simulated database dump" > "$TEMP_BACKUP"
sleep 2

# Verify backup
if [ ! -s "$TEMP_BACKUP" ]; then
    echo "ERROR: Backup file is empty!"
    exit 1
fi

# Move temp file to final location
mv "$TEMP_BACKUP" "$BACKUP_FILE"

# Compress
echo "Compressing backup..."
gzip "$BACKUP_FILE"

# Mark success
BACKUP_SUCCESS=true

echo "Backup saved: ${BACKUP_FILE}.gz"

# cleanup will run, but won't rollback since BACKUP_SUCCESS=true
```

### Example 4: Logging All Signals

```bash
#!/bin/bash

# Log file
LOG="script_signals.log"

# Function to log signals
log_signal() {
    local signal=$1
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] Received signal: $signal" >> "$LOG"
}

# Trap multiple signals
trap 'log_signal EXIT' EXIT
trap 'log_signal INT; exit' INT
trap 'log_signal TERM; exit' TERM

echo "Script running... (Check $LOG for signal logs)"
echo "Press Ctrl+C to interrupt"

# Simulate work
for i in {1..10}; do
    echo "Working... $i/10"
    sleep 1
done

echo "Script completed normally"
```

### Advanced: Trap with Error Line Numbers

```bash
#!/bin/bash

# Exit on error
set -e

# Error handler with line number
error_handler() {
    local line_number=$1
    local error_code=$2
    local command="$3"

    echo "ERROR on line $line_number:"
    echo "  Command: $command"
    echo "  Exit code: $error_code"

    # Add additional error handling here
}

# Trap errors with line number information
trap 'error_handler ${LINENO} $? "$BASH_COMMAND"' ERR

echo "Starting script..."

# This will trigger the error handler
some_nonexistent_command

echo "This won't execute"
```

-----

## 7.4 Combining Arrays and Traps: Production Example

Here's a complete example using both concepts:

```bash
#!/bin/bash

set -e

# Services to monitor
SERVICES=("nginx" "mysql" "redis" "mongodb")

# Temporary files
declare -A TEMP_FILES

# Cleanup function
cleanup() {
    echo "Cleaning up temporary files..."

    for SERVICE in "${!TEMP_FILES[@]}"; do
        local temp="${TEMP_FILES[$SERVICE]}"
        if [ -f "$temp" ]; then
            rm -f "$temp"
            echo "  Removed: $temp"
        fi
    done
}

# Register cleanup
trap cleanup EXIT INT TERM

# Check each service
echo "Checking service health..."
echo "=========================="

for SERVICE in "${SERVICES[@]}"; do
    # Create temp file for this service
    TEMP_FILE=$(mktemp)
    TEMP_FILES["$SERVICE"]="$TEMP_FILE"

    echo "Checking: $SERVICE"

    # Simulate health check (replace with actual check)
    # systemctl status $SERVICE > "$TEMP_FILE" 2>&1
    echo "Service $SERVICE is running" > "$TEMP_FILE"

    # Analyze results
    if grep -q "running" "$TEMP_FILE"; then
        echo "  ✓ Status: Healthy"
    else
        echo "  ✗ Status: Failed"
    fi
done

echo "=========================="
echo "Health check complete"

# Cleanup runs automatically via trap
```

-----

## Common Mistakes and Best Practices

### Arrays

1. **Forgetting quotes in loops:**
   ```bash
   for item in ${ARRAY[@]}; do    # ❌ Breaks with spaces
   for item in "${ARRAY[@]}"; do  # ✅ Safe
   ```

2. **Not using curly braces:**
   ```bash
   echo $ARRAY[0]       # ❌ Wrong!
   echo ${ARRAY[0]}     # ✅ Correct
   ```

3. **Forgetting `declare -A` for associative arrays:**
   ```bash
   MYMAP["key"]="value"           # ❌ Creates indexed array!
   declare -A MYMAP
   MYMAP["key"]="value"           # ✅ Correct
   ```

### Traps

1. **Not handling cleanup:**
   Always trap EXIT to ensure cleanup happens

2. **Forgetting to remove locks:**
   Use traps to remove lock files and prevent deadlocks

3. **Not testing error paths:**
   Deliberately trigger errors to ensure your traps work

-----

## Practice Exercises

### Exercise 1: Server Inventory
Create a script that:
- Uses an associative array to store server names and IPs
- Loops through and pings each server
- Reports which servers are up/down

### Exercise 2: Log Rotator
Write a script that:
- Uses an array to list log files to rotate
- Compresses old logs
- Uses a trap to ensure incomplete operations are cleaned up

### Exercise 3: Batch File Processor
Build a script that:
- Stores a list of files to process in an array
- Processes each file
- Uses trap to ensure temp files are cleaned up if user cancels (Ctrl+C)

**Solution hint for Exercise 3:**
```bash
#!/bin/bash

FILES=("file1.txt" "file2.txt" "file3.txt")
TEMP_DIR=$(mktemp -d)

cleanup() {
    rm -rf "$TEMP_DIR"
    echo "Cleaned up temporary directory"
}

trap cleanup EXIT INT

for FILE in "${FILES[@]}"; do
    # Process each file
    echo "Processing $FILE..."
    # Your processing logic here
done
```

-----

## Summary

You now have powerful tools for advanced scripting:

✅ **Arrays** - Store and manipulate lists of data efficiently
✅ **Associative Arrays** - Use key-value pairs for complex data
✅ **Traps** - Handle signals and ensure cleanup
✅ **Error Recovery** - Build fault-tolerant scripts
✅ **Resource Management** - Properly clean up temp files and locks

These advanced features let you build production-ready scripts that handle edge cases gracefully.

**Next Up:** In Unit 8, we'll explore package managers and system administration tasks, learning how to write portable scripts that work across different Linux distributions and macOS.
