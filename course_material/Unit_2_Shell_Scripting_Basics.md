# Unit 2: Introduction to Shell Scripting

**Learning Objectives:**
- Understand what the shebang is and why it matters
- Declare and use variables correctly
- Master quoting rules to avoid common bugs
- Capture command output with command substitution
- Understand exit codes and script success/failure

**Estimated Time:** 2-3 hours

-----

## Introduction: From Commands to Scripts

In Unit 1, you learned to run commands one at a time in the terminal. But what if you need to run the same sequence of 10 commands every morning? Or 100 times with different inputs? This is where shell scripting comes in.

A **shell script** is simply a text file containing a series of commands that the shell executes in order. Think of it as a recipe—instead of cooking the same meal from memory each time, you write down the steps once and follow them whenever needed.

The beauty of shell scripts:
- **Automation** - Run complex tasks with a single command
- **Consistency** - The same steps execute the same way every time
- **Reusability** - Share your scripts with teammates
- **Time-saving** - A 5-minute script can save hours of repetitive work

Let's build your first scripts from the ground up.

-----

## 2.1 The Shebang: Telling the System Which Shell to Use

### What is a Shebang?

Every shell script should start with a special line called the **shebang** (pronounced "hash-bang" or "sharp-bang"). It looks like this:

```bash
#!/bin/bash
```

This line tells the operating system, "Hey, use the Bash shell to run this script." Without it, the system might guess wrong or use a different shell, leading to unpredictable behavior.

### Anatomy of the Shebang

```bash
#!/bin/bash
│││
│││
││└─ The path to the interpreter program
│└── The bang (exclamation mark)
└─── The hash symbol
```

The `#!` must be the very first two characters in your file—no spaces, no blank lines above it.

### Common Shebang Lines

```bash
#!/bin/bash
# Use Bash (most common for Linux)

#!/bin/zsh
# Use Zsh (default on modern macOS)

#!/usr/bin/env bash
# Portable - finds bash wherever it's installed (RECOMMENDED)

#!/usr/bin/env python3
# Not just for shells! This would run the file as a Python script
```

### Why Use `#!/usr/bin/env bash`?

The `/usr/bin/env` approach is more portable because different systems install programs in different locations:
- On some systems, bash might be in `/bin/bash`
- On others, it might be in `/usr/local/bin/bash`
- `env` searches your system's PATH and finds bash wherever it lives

**Real-world example:**
```bash
#!/usr/bin/env bash
# This script will work on Linux, macOS, and most Unix systems

echo "Hello from a portable script!"
```

### Testing Your Understanding

Create a file called `hello.sh`:

```bash
#!/bin/bash

echo "Hello, World!"
echo "This is my first shell script."
```

Make it executable and run it:

```bash
chmod +x hello.sh
./hello.sh
```

**What happens if you forget the shebang?** The script might still work if you run it with `bash hello.sh`, but if you try to run it directly with `./hello.sh`, you might get an error or it might run with the wrong shell.

-----

## 2.2 Variables: Storing and Using Data

Variables are containers for storing data. In shell scripting, they're essential for making your scripts flexible and reusable.

### Declaring Variables

The syntax is simple, but **the spacing is critical**:

```bash
NAME="Alice"          # CORRECT
AGE=25                # CORRECT

NAME = "Alice"        # WRONG! Spaces around = confuse the shell
AGE= 25               # WRONG!
```

**Rules for variable names:**
- Can contain letters, numbers, and underscores
- Must start with a letter or underscore (not a number)
- Convention: Use UPPERCASE for constants, lowercase for regular variables
- No spaces around the `=` sign

```bash
# Good variable names
USER_NAME="john_doe"
MAX_RETRIES=3
_temp_file="/tmp/data"

# Bad variable names
2nd_user="error"      # Can't start with a number
my-var="error"        # Hyphens not allowed
```

### Accessing Variables

To get the value stored in a variable, prefix it with a dollar sign `$`:

```bash
NAME="Alice"
echo $NAME           # Prints: Alice
echo "Hello, $NAME"  # Prints: Hello, Alice
```

### The ${} Syntax (Best Practice)

While `$NAME` works, `${NAME}` is safer and clearer, especially when concatenating:

```bash
FRUIT="apple"

# This doesn't work - shell looks for variable $FRUITs
echo "I have 5 $FRUITs"

# This works - clearly separates variable from surrounding text
echo "I have 5 ${FRUIT}s"  # Prints: I have 5 apples
```

**Another example:**
```bash
FILE="report"
EXT="pdf"

# Create a filename
FULL_NAME="${FILE}.${EXT}"
echo $FULL_NAME  # Prints: report.pdf
```

### Reading User Input

The `read` command captures input from the user and stores it in a variable:

```bash
#!/bin/bash

echo "What's your name?"
read USERNAME

echo "What's your age?"
read AGE

echo "Hello, $USERNAME! You are $AGE years old."
```

**Try it yourself:**
```bash
#!/bin/bash

echo "Enter your favorite programming language:"
read LANGUAGE

echo "Great choice! $LANGUAGE is awesome."
```

### Special Built-in Variables

The shell provides several special variables automatically:

```bash
#!/bin/bash

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "All arguments: $@"
echo "Number of arguments: $#"
echo "Script's process ID: $$"
echo "Last command's exit code: $?"
```

**Real-world example:**
```bash
#!/bin/bash
# Script: greet.sh
# Usage: ./greet.sh Alice 25

NAME=$1  # First command-line argument
AGE=$2   # Second command-line argument

echo "Hello, $NAME!"
echo "You are $AGE years old."
```

Run it like this:
```bash
./greet.sh Alice 25
# Output:
# Hello, Alice!
# You are 25 years old.
```

-----

## 2.3 Quoting Rules: The Source of Most Bugs

Quoting in shell scripts is where beginners make the most mistakes. Understanding these rules will save you hours of debugging.

### Three Types of Quoting

1. **No Quotes** - The shell interprets everything (dangerous!)
2. **Double Quotes `" "`** - The shell expands variables but preserves spaces
3. **Single Quotes `' '`** - Everything is treated as literal text

### No Quotes: Maximum Interpretation

```bash
FILE=my document.txt
rm $FILE  # DANGER! This tries to delete 'my' and 'document.txt' separately
```

The shell sees the space and thinks you're giving it two separate arguments.

### Double Quotes: Safe Variable Expansion

```bash
FILE="my document.txt"
rm "$FILE"  # SAFE: Treats the whole filename as one argument
```

**Key point:** Always quote variables that might contain spaces:

```bash
#!/bin/bash

FULL_NAME="Alice Smith"

# Wrong - treats Alice and Smith as separate arguments
echo Hello $FULL_NAME  # Might display oddly

# Right - keeps the name together
echo "Hello $FULL_NAME"  # Displays: Hello Alice Smith
```

### Single Quotes: No Variable Expansion

```bash
NAME="Alice"

echo "Hello, $NAME"    # Prints: Hello, Alice
echo 'Hello, $NAME'    # Prints: Hello, $NAME (literal)
```

Use single quotes when you want the exact text, without any variable substitution.

### Real-World Bug Examples

**Bug #1: Filenames with spaces**
```bash
#!/bin/bash

# User has a file called "my photo.jpg"
FILE="my photo.jpg"

# This fails (tries to copy "my" and "photo.jpg" separately)
cp $FILE backup/

# This works
cp "$FILE" backup/
```

**Bug #2: Empty variables**
```bash
#!/bin/bash

# If USERNAME is empty, this becomes: [ = "admin" ]
# which is a syntax error
if [ $USERNAME = "admin" ]; then
    echo "Welcome, admin"
fi

# This works even if USERNAME is empty: [ "" = "admin" ]
if [ "$USERNAME" = "admin" ]; then
    echo "Welcome, admin"
fi
```

### The Golden Rule

**Always quote your variables unless you have a specific reason not to:**

```bash
"$VARIABLE"    # ✅ Safe default
'$VARIABLE'    # ✅ When you want literal text
$VARIABLE      # ⚠️  Only if you're 100% sure it's safe
```

-----

## 2.4 Command Substitution: Capturing Output

Often you want to save the output of a command into a variable. This is called **command substitution**.

### Modern Syntax: $()

```bash
CURRENT_DIR=$(pwd)
echo "You are in: $CURRENT_DIR"

TODAY=$(date +%Y-%m-%d)
echo "Today's date: $TODAY"
```

### How It Works

The shell:
1. Runs the command inside `$()`
2. Captures its output
3. Substitutes that output into the variable

```bash
# Step-by-step example
USER_COUNT=$(who | wc -l)

# What happens:
# 1. `who` lists logged-in users
# 2. `wc -l` counts the lines
# 3. Result (e.g., "3") is stored in USER_COUNT

echo "There are $USER_COUNT users logged in."
```

### Legacy Syntax: Backticks (Avoid)

You might see older scripts use backticks:

```bash
HOSTNAME=`hostname`  # Old way
HOSTNAME=$(hostname)  # Modern way (preferred)
```

The `$()` syntax is better because:
- Easier to read
- Can be nested
- Less confusing than backticks

### Practical Examples

**Example 1: Create timestamped backups**
```bash
#!/bin/bash

TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_${TIMESTAMP}.tar.gz"

tar -czf "$BACKUP_FILE" /path/to/important/files
echo "Backup created: $BACKUP_FILE"
```

**Example 2: Count files in a directory**
```bash
#!/bin/bash

FILE_COUNT=$(ls -1 | wc -l)
echo "This directory contains $FILE_COUNT files."
```

**Example 3: Get system information**
```bash
#!/bin/bash

HOSTNAME=$(hostname)
KERNEL=$(uname -r)
MEMORY=$(free -h | awk 'NR==2 {print $2}')

echo "System Information:"
echo "Hostname: $HOSTNAME"
echo "Kernel: $KERNEL"
echo "Total Memory: $MEMORY"
```

-----

## 2.5 Exit Codes: Success and Failure

Every command in Unix returns a number when it finishes, called the **exit code** (or exit status).

### The Convention

- **0** = Success (everything worked)
- **1-255** = Error (something went wrong)

Different error codes can mean different things, but 0 always means success.

### Checking the Last Exit Code

The special variable `$?` holds the exit code of the most recently executed command:

```bash
ls /home
echo $?  # Prints: 0 (success)

ls /nonexistent
echo $?  # Prints: 2 (error - directory doesn't exist)
```

### Why Exit Codes Matter

Scripts often chain commands together. Exit codes let you check if each step succeeded before moving to the next:

```bash
#!/bin/bash

# Try to download a file
wget https://example.com/data.csv

# Check if the download succeeded
if [ $? -eq 0 ]; then
    echo "Download successful!"
    # Process the file
    python analyze.py data.csv
else
    echo "Download failed!"
    exit 1  # Exit the script with an error code
fi
```

### Setting Your Script's Exit Code

Use the `exit` command to end your script with a specific code:

```bash
#!/bin/bash

if [ ! -f "config.txt" ]; then
    echo "ERROR: config.txt not found"
    exit 1  # Exit with error
fi

# Continue with normal processing...
echo "Config loaded successfully"
exit 0  # Exit with success
```

### Real-World Pattern: Error Checking

```bash
#!/bin/bash

# Function to check if previous command succeeded
check_error() {
    if [ $? -ne 0 ]; then
        echo "ERROR: $1"
        exit 1
    fi
}

# Try to create a directory
mkdir /var/app/data
check_error "Failed to create directory"

# Try to copy files
cp config.ini /var/app/data/
check_error "Failed to copy config file"

echo "Setup completed successfully!"
```

### Using exit codes with && and ||

Bash provides shortcuts for checking success/failure:

```bash
# && means "run the next command only if the previous succeeded"
mkdir test_dir && cd test_dir && touch file.txt

# || means "run the next command only if the previous failed"
wget https://example.com/file.zip || echo "Download failed"
```

-----

## 2.6 Putting It All Together: Your First Practical Script

Let's create a script that uses everything we've learned:

```bash
#!/usr/bin/env bash
#
# Script: backup_script.sh
# Purpose: Creates a timestamped backup of a directory
# Usage: ./backup_script.sh /path/to/source

# Check if user provided a directory
if [ $# -eq 0 ]; then
    echo "Usage: $0 <directory>"
    exit 1
fi

SOURCE_DIR="$1"

# Check if the directory exists
if [ ! -d "$SOURCE_DIR" ]; then
    echo "ERROR: Directory '$SOURCE_DIR' does not exist."
    exit 1
fi

# Create backup filename with timestamp
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_${TIMESTAMP}.tar.gz"

echo "Starting backup of $SOURCE_DIR..."

# Create the backup
tar -czf "$BACKUP_FILE" "$SOURCE_DIR"

# Check if backup succeeded
if [ $? -eq 0 ]; then
    SIZE=$(du -h "$BACKUP_FILE" | cut -f1)
    echo "✓ Backup successful: $BACKUP_FILE ($SIZE)"
    exit 0
else
    echo "✗ Backup failed!"
    exit 1
fi
```

**To use this script:**
```bash
chmod +x backup_script.sh
./backup_script.sh ~/Documents
```

-----

## Common Beginner Mistakes

### 1. Spaces Around the = Sign

```bash
NAME = "Alice"  # ❌ Shell thinks NAME is a command
NAME="Alice"    # ✅ Correct
```

### 2. Not Quoting Variables

```bash
FILE="my document.txt"
rm $FILE        # ❌ Tries to delete "my" and "document.txt"
rm "$FILE"      # ✅ Deletes "my document.txt"
```

### 3. Forgetting the $

```bash
NAME="Alice"
echo NAME       # ❌ Prints: NAME
echo $NAME      # ✅ Prints: Alice
```

### 4. Using = Instead of -eq for Numbers

```bash
if [ $AGE = 18 ]; then     # ❌ String comparison
if [ $AGE -eq 18 ]; then   # ✅ Numeric comparison
```

### 5. Not Making Scripts Executable

```bash
./script.sh     # ❌ "Permission denied"
chmod +x script.sh
./script.sh     # ✅ Works
```

-----

## Practice Exercises

### Exercise 1: Personal Greeter
Create a script that asks for your name and age, then greets you:
```bash
# Expected output:
# What's your name? Alice
# How old are you? 25
# Hello, Alice! You are 25 years old.
```

### Exercise 2: System Info Reporter
Write a script that displays:
- Current username
- Current directory
- Today's date
- Number of files in the current directory

### Exercise 3: File Backup
Create a script that:
- Takes a filename as an argument
- Creates a backup with `.bak` extension
- Reports success or failure

**Solution hint for Exercise 3:**
```bash
#!/bin/bash
FILE="$1"
if [ -f "$FILE" ]; then
    cp "$FILE" "${FILE}.bak"
    echo "Backup created: ${FILE}.bak"
else
    echo "File not found: $FILE"
    exit 1
fi
```

-----

## Summary

You've now mastered the fundamentals of shell scripting:

✅ **Shebang** - Always start with `#!/usr/bin/env bash`  
✅ **Variables** - No spaces around `=`, always quote them  
✅ **Quoting** - Use double quotes `" "` for variables, single quotes `' '` for literal text  
✅ **Command substitution** - Use `$(command)` to capture output  
✅ **Exit codes** - Check `$?` for success/failure, use `exit` to set your script's code  

These building blocks will be used in every script you write. Practice them until they become second nature.

**Next Up:** In Unit 3, we'll learn about control structures—making decisions with `if` statements and repeating tasks with loops. This is where your scripts become truly powerful and intelligent.
