# Unit 4: Functions and Script Organization

**Learning Objectives:**
- Define and call functions to reuse code
- Understand variable scope (global vs. local)
- Pass arguments to functions
- Handle return values and exit codes
- Organize larger scripts for maintainability

**Estimated Time:** 2-3 hours

-----

## Introduction: The DRY Principle

As your scripts grow from 10 lines to 100 lines, you'll notice a problem: you're copying and pasting the same code over and over.

- Need to check if a user exists? Copy-paste the check logic.
- Need to log a message with a timestamp? Copy-paste the `echo` command.

This violates the **DRY Principle** (Don't Repeat Yourself). If you find a bug in your logic, you have to fix it in five different places.

**Functions** solve this. A function is a block of code with a name. You define it once, and you can "call" it as many times as you want. It's like creating your own custom command.

-----

## 4.1 Defining and Calling Functions

There are two common ways to define functions in shell scripting.

### Syntax Style 1 (Recommended)
This is the standard POSIX syntax, compatible with almost all shells (Bash, Zsh, Dash, Sh).

```bash
my_function() {
    # Commands go here
    echo "Hello from inside the function!"
}
```

### Syntax Style 2 (Bash Specific)
This works in Bash but might not work in other shells.

```bash
function my_function {
    echo "Hello from inside the function!"
}
```

### Calling a Function
To run the function, simply type its name—**without parentheses**.

```bash
#!/bin/bash

greet() {
    echo "Hello, World!"
}

echo "Script starting..."
greet   # Call the function
echo "Script ending..."
```

**Output:**
```
Script starting...
Hello, World!
Script ending...
```

-----

## 4.2 Passing Arguments to Functions

Functions in shell scripts don't declare arguments in parentheses like Python or JavaScript `def greet(name):`. Instead, they receive arguments exactly like a script does—using `$1`, `$2`, etc.

```bash
#!/bin/bash

greet_user() {
    # $1 is the first argument passed TO THE FUNCTION
    # It is NOT the same as the script's $1
    echo "Hello, $1!"
}

greet_user "Alice"
greet_user "Bob"
```

### Handling Multiple Arguments

```bash
#!/bin/bash

create_user() {
    USERNAME=$1
    ROLE=$2
    
    echo "Creating user: $USERNAME"
    echo "Assigning role: $ROLE"
}

create_user "john_doe" "admin"
```

**Important:** The variables `$1`, `$2`, `$@`, `$#` inside a function refer to the function's arguments, not the script's command-line arguments.

-----

## 4.3 Variable Scope: Global vs. Local

This is a common source of bugs. By default, **all variables in shell scripts are global**.

### The Problem with Global Variables

```bash
#!/bin/bash

set_name() {
    NAME="Alice"  # This changes the global variable!
}

NAME="Bob"
echo "Before: $NAME"  # Prints Bob

set_name
echo "After: $NAME"   # Prints Alice (Oops!)
```

### The Solution: `local`

Use the `local` keyword to keep variables contained within the function.

```bash
#!/bin/bash

set_name() {
    local NAME="Alice"  # Only exists inside this function
    echo "Inside function: $NAME"
}

NAME="Bob"
set_name              # Prints: Inside function: Alice
echo "Outside: $NAME" # Prints: Outside: Bob (Safe!)
```

**Best Practice:** Always use `local` for variables inside functions unless you specifically intend to modify a global variable.

-----

## 4.4 Return Values

In most programming languages, functions `return` data (like a string or number). In shell scripting, `return` only sends an **exit code** (0-255), usually to indicate success or failure.

### Returning Success/Failure

```bash
#!/bin/bash

check_file() {
    local FILE=$1
    if [ -f "$FILE" ]; then
        return 0  # Success
    else
        return 1  # Failure
    fi
}

if check_file "config.txt"; then
    echo "File exists!"
else
    echo "File missing!"
fi
```

### "Returning" Data (The Workaround)

To get data out of a function, you usually `echo` it and capture the output using command substitution `$()`.

```bash
#!/bin/bash

get_uppercase() {
    local STR=$1
    echo "$STR" | tr '[:lower:]' '[:upper:]'
    # We are NOT using 'return' here
}

# Capture the output
RESULT=$(get_uppercase "hello world")
echo "Converted: $RESULT"
```

-----

## 4.5 Script Organization Patterns

As scripts get larger, organization becomes critical.

### The "Main" Function Pattern

A professional script structure often looks like this:

```bash
#!/bin/bash

# 1. Global Constants
LOG_FILE="/var/log/myscript.log"
DEBUG=true

# 2. Function Definitions
log_msg() {
    local MSG=$1
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $MSG"
}

setup_env() {
    log_msg "Setting up environment..."
    # setup logic
}

cleanup() {
    log_msg "Cleaning up..."
    # cleanup logic
}

# 3. Main Logic
main() {
    log_msg "Script started."
    
    setup_env
    
    # Do work...
    
    cleanup
    log_msg "Script finished."
}

# 4. Execute Main
main "$@"
```

This structure makes your code readable, testable, and easy to follow.

-----

## 4.6 Function Documentation Best Practices

Before we dive into the practical example, let's establish a standard for documenting functions:

### Professional Function Header Template

```bash
# ═══════════════════════════════════════════════════════════════
# Function: function_name
# Purpose: One-line description of what this function does
# Parameters:
#   $1 - parameter_name: Description of first parameter
#   $2 - parameter_name: Description of second parameter
# Returns:
#   0 - Success
#   1 - Failure (with specific error condition)
# Side Effects: What global state changes or external actions occur
# Example:
#   function_name "arg1" "arg2"
#   if function_name "$file" "$dest"; then
#       echo "Success"
#   fi
# ═══════════════════════════════════════════════════════════════
function_name() {
    local param1=$1
    local param2=$2

    # Implementation here
}
```

**Example with real documentation:**

```bash
# ═══════════════════════════════════════════════════════════════
# Function: backup_file
# Purpose: Create a timestamped backup copy of a file
# Parameters:
#   $1 - source_file: Path to file to backup (must exist and be readable)
#   $2 - backup_dir: Directory where backup will be stored (must be writable)
# Returns:
#   0 - Backup created successfully
#   1 - Source file doesn't exist or isn't readable
#   2 - Backup directory doesn't exist or isn't writable
# Side Effects: Creates a new file in backup_dir
# Example:
#   backup_file "/etc/nginx/nginx.conf" "/backups"
#   # Creates: /backups/nginx.conf_20240115_143022.bak
# ═══════════════════════════════════════════════════════════════
backup_file() {
    local source_file=$1
    local backup_dir=$2
    local timestamp=$(date +%Y%m%d_%H%M%S)

    # Validate source file
    if [[ ! -f "$source_file" ]] || [[ ! -r "$source_file" ]]; then
        echo "ERROR: Source file doesn't exist or isn't readable: $source_file" >&2
        return 1
    fi

    # Validate backup directory
    if [[ ! -d "$backup_dir" ]] || [[ ! -w "$backup_dir" ]]; then
        echo "ERROR: Backup directory doesn't exist or isn't writable: $backup_dir" >&2
        return 2
    fi

    # Create backup filename
    local filename=$(basename "$source_file")
    local backup_file="${backup_dir}/${filename}_${timestamp}.bak"

    # Perform backup
    if cp "$source_file" "$backup_file"; then
        echo "Backup created: $backup_file"
        return 0
    else
        echo "ERROR: Failed to create backup" >&2
        return 1
    fi
}
```

**💡 Key Benefits of This Documentation Style:**
1. **Self-documenting** - Anyone can understand the function without reading implementation
2. **Clear contract** - Caller knows exactly what inputs are needed and what outputs to expect
3. **Error handling** - Documents all failure modes and their return codes
4. **Usage examples** - Shows how to call the function correctly

-----

## 4.7 Practical Example: User Management Script

Let's build a modular script using everything we've learned.

```bash
#!/bin/bash

# Global configuration
USER_FILE="users.txt"
LOG_FILE="user_management.log"

# Logging function
log() {
    local LEVEL=$1
    local MSG=$2
    local TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")
    echo "[$TIMESTAMP] [$LEVEL] $MSG" >> "$LOG_FILE"
    
    # Also print errors to screen
    if [ "$LEVEL" = "ERROR" ]; then
        echo "Error: $MSG" >&2
    fi
}

# Check if user exists
user_exists() {
    local USER=$1
    if id "$USER" &>/dev/null; then
        return 0 # True
    else
        return 1 # False
    fi
}

# Create a single user
create_user() {
    local USER=$1
    
    if user_exists "$USER"; then
        log "WARN" "User $USER already exists. Skipping."
        return 0
    fi
    
    # Simulate user creation (replace echo with useradd in real script)
    echo "Creating user: $USER"
    log "INFO" "Created user: $USER"
}

# Main function
main() {
    # Check if input file exists
    if [ ! -f "$USER_FILE" ]; then
        log "ERROR" "Input file $USER_FILE not found!"
        exit 1
    fi
    
    log "INFO" "Starting batch user creation..."
    
    # Loop through file
    while read -r USERNAME; do
        # Skip empty lines
        [ -z "$USERNAME" ] && continue
        
        create_user "$USERNAME"
    done < "$USER_FILE"
    
    log "INFO" "Batch processing complete."
}

# Start the script
main
```

-----

## Common Mistakes

1. **Using parentheses to call functions:**
   ```bash
   my_func()  # Wrong!
   my_func    # Correct
   ```

2. **Forgetting `local`:**
   Variables leak into the global scope and cause hard-to-find bugs.

3. **Trying to `return` strings:**
   ```bash
   get_name() {
       return "Alice"  # Error! return only takes numbers 0-255
   }
   ```

-----

## Practice Exercises

### Exercise 1: Math Functions
Create a script with functions `add`, `subtract`, and `multiply`.
- Each function takes two arguments.
- Print the result.
- Call them from the main part of the script.

### Exercise 2: File Checker
Write a function `check_required_files` that:
- Takes a list of filenames as arguments.
- Checks if each exists.
- Returns 0 if ALL exist, 1 if ANY are missing.

### Exercise 3: Logger
Create a reusable logging function that:
- Takes a message and a "level" (INFO, WARN, ERROR).
- Prints colored output to the screen (Green for INFO, Yellow for WARN, Red for ERROR).

-----

## Summary

You now have the power of modularity:

✅ **Functions** - Reusable blocks of code  
✅ **Arguments** - Passing data using `$1`, `$2`  
✅ **Scope** - Protecting variables with `local`  
✅ **Return Values** - Using exit codes for logic and `echo` for data  
✅ **Organization** - Structuring scripts with a `main` function  

**Next Up:** In Unit 5, we'll dive into Input/Output handling and the "Power Trio" of text processing tools: Grep, Sed, and Awk.
