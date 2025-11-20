# Unit 3: Control Structures and Flow Control

**Learning Objectives:**
- Make decisions in scripts using if/else statements
- Test conditions for files, strings, and numbers
- Repeat tasks efficiently with for loops
- Create conditional loops with while
- Combine control structures for complex automation

**Estimated Time:** 2-3 hours

-----

## Introduction: Making Scripts Intelligent

So far, your scripts execute commands in a straight line—command 1, then command 2, then command 3. But real automation needs intelligence. Your scripts should be able to:

- **Make decisions**: "If the file exists, process it; otherwise, create it."
- **Repeat actions**: "For each file in this directory, rename it."
- **Wait for conditions**: "Keep trying to connect until it works."

This is called **control flow**—controlling which code runs and when. It's what transforms a simple list of commands into a powerful automation tool.

Think of it like a choose-your-own-adventure book. Instead of reading page 1, 2, 3 in order, you read page 1, then jump to page 15 if you choose option A, or page 42 if you choose option B. Your script follows a similar pattern based on conditions you define.

-----

## 3.1 Conditional Logic: The if Statement

The `if` statement lets your script make decisions. The basic idea: "If this condition is true, do these commands."

### Basic Syntax

```bash
if [ condition ]; then
    # Commands to run if condition is true
fi
```

**Important syntax notes:**
- Spaces inside the brackets `[ ]` are **mandatory**
- The semicolon `;` before `then` is required (or put `then` on the next line)
- Always close with `fi` ("if" spelled backwards)

### Your First if Statement

```bash
#!/bin/bash

AGE=25

if [ $AGE -ge 18 ]; then
    echo "You are an adult."
fi
```

**Breaking this down:**
- `[ $AGE -ge 18 ]` - Test if AGE is greater than or equal to 18
- `-ge` means "greater than or equal" (numeric comparison)
- If true, print the message
- If false, skip the echo command

### Adding else: Two Paths

```bash
#!/bin/bash

AGE=15

if [ $AGE -ge 18 ]; then
    echo "You are an adult."
else
    echo "You are a minor."
fi
```

Now the script always does something—it takes one path or the other.

### Adding elif: Multiple Conditions

```bash
#!/bin/bash

SCORE=85

if [ $SCORE -ge 90 ]; then
    echo "Grade: A"
elif [ $SCORE -ge 80 ]; then
    echo "Grade: B"
elif [ $SCORE -ge 70 ]; then
    echo "Grade: C"
else
    echo "Grade: F"
fi
```

The script checks each condition in order and executes the first match it finds.

-----

## 3.2 Test Conditions: The Heart of Decision Making

The magic happens inside the brackets `[ ]`. This is where you write your conditions.

### Numeric Comparisons

Used for comparing numbers:

```bash
[ $A -eq $B ]  # Equal to
[ $A -ne $B ]  # Not equal to
[ $A -gt $B ]  # Greater than
[ $A -ge $B ]  # Greater than or equal
[ $A -lt $B ]  # Less than
[ $A -le $B ]  # Less than or equal
```

**Example:**
```bash
#!/bin/bash

DISK_USAGE=85

if [ $DISK_USAGE -gt 90 ]; then
    echo "WARNING: Disk almost full!"
elif [ $DISK_USAGE -gt 75 ]; then
    echo "Disk usage getting high."
else
    echo "Disk usage normal."
fi
```

### String Comparisons

Used for comparing text:

```bash
[ "$A" = "$B" ]   # Strings are equal
[ "$A" != "$B" ]  # Strings are not equal
[ -z "$A" ]       # String is empty (zero length)
[ -n "$A" ]       # String is not empty
```

**Always quote your variables in string comparisons!**

```bash
#!/bin/bash

echo "Enter your username:"
read USERNAME

if [ -z "$USERNAME" ]; then
    echo "ERROR: Username cannot be empty!"
    exit 1
fi

if [ "$USERNAME" = "admin" ]; then
    echo "Welcome, administrator!"
else
    echo "Welcome, $USERNAME!"
fi
```

### File Tests

Shell scripting provides powerful file checking capabilities:

```bash
[ -f "$FILE" ]  # File exists and is a regular file
[ -d "$DIR" ]   # Directory exists
[ -e "$PATH" ]  # Exists (file or directory)
[ -r "$FILE" ]  # File is readable
[ -w "$FILE" ]  # File is writable
[ -x "$FILE" ]  # File is executable
[ -s "$FILE" ]  # File exists and is not empty
```

**Practical example:**
```bash
#!/bin/bash

CONFIG_FILE="config.ini"

if [ -f "$CONFIG_FILE" ]; then
    echo "Loading configuration..."
    
    if [ -r "$CONFIG_FILE" ]; then
        # Process the file
        cat "$CONFIG_FILE"
    else
        echo "ERROR: Cannot read $CONFIG_FILE"
        exit 1
    fi
else
    echo "Config file not found. Creating default..."
    echo "# Default configuration" > "$CONFIG_FILE"
fi
```

### Combining Conditions

Use `&&` (AND) and `||` (OR) to combine multiple conditions:

```bash
# AND - both conditions must be true
if [ -f "$FILE" ] && [ -r "$FILE" ]; then
    echo "File exists and is readable"
fi

# OR - at least one condition must be true
if [ "$USER" = "admin" ] || [ "$USER" = "root" ]; then
    echo "Privileged user detected"
fi
```

**Example with complex logic:**
```bash
#!/bin/bash

FILE="report.pdf"
MIN_SIZE=1000  # bytes

if [ -f "$FILE" ] && [ -s "$FILE" ]; then
    SIZE=$(stat -f%z "$FILE" 2>/dev/null || stat -c%s "$FILE")
    
    if [ $SIZE -gt $MIN_SIZE ]; then
        echo "✓ File is valid and sufficient size"
    else
        echo "✗ File is too small"
    fi
else
    echo "✗ File doesn't exist or is empty"
fi
```

### Negating Conditions

Use `!` to negate (reverse) a condition:

```bash
if [ ! -f "$FILE" ]; then
    echo "File does NOT exist"
fi

# Equivalent to:
if ! [ -f "$FILE" ]; then
    echo "File does NOT exist"
fi
```

-----

## 3.3 The for Loop: Repeating for Each Item

The `for` loop repeats commands for each item in a list. It's perfect for batch processing.

### Basic for Loop Syntax

```bash
for variable in list; do
    # Commands that use $variable
done
```

### Example 1: Loop Over a Simple List

```bash
#!/bin/bash

for NAME in Alice Bob Charlie; do
    echo "Hello, $NAME!"
done
```

**Output:**
```
Hello, Alice!
Hello, Bob!
Hello, Charlie!
```

### Example 2: Loop Over Files (Globbing)

One of the most powerful uses—processing all files matching a pattern:

```bash
#!/bin/bash

for FILE in *.txt; do
    echo "Processing $FILE..."
    # Count lines in each text file
    wc -l "$FILE"
done
```

**Important:** Always quote `"$FILE"` to handle filenames with spaces!

### Example 3: Loop Over Numbers

```bash
#!/bin/bash

# C-style for loop
for ((i=1; i<=5; i++)); do
    echo "Iteration $i"
done
```

**Output:**
```
Iteration 1
Iteration 2
Iteration 3
Iteration 4
Iteration 5
```

### Real-World Example: Batch Rename Files

```bash
#!/bin/bash
# Add a prefix to all .jpg files

PREFIX="vacation_"

for FILE in *.jpg; do
    # Check if any .jpg files exist
    [ -e "$FILE" ] || continue
    
    # Create new name
    NEW_NAME="${PREFIX}${FILE}"
    
    # Rename the file
    mv "$FILE" "$NEW_NAME"
    echo "Renamed: $FILE → $NEW_NAME"
done

echo "Batch rename complete!"
```

### Real-World Example: Backup Multiple Files

```bash
#!/bin/bash
# Create timestamped backups of important files

TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="backups_${TIMESTAMP}"

# Create backup directory
mkdir -p "$BACKUP_DIR"

# List of files to backup
FILES="config.ini database.db app.log"

for FILE in $FILES; do
    if [ -f "$FILE" ]; then
        cp "$FILE" "$BACKUP_DIR/"
        echo "✓ Backed up: $FILE"
    else
        echo "✗ Not found: $FILE"
    fi
done

echo "Backup saved to: $BACKUP_DIR"
```

### Looping Over Command Output

You can loop over the output of any command:

```bash
#!/bin/bash

# Process each user currently logged in
for USER in $(who | awk '{print $1}' | sort -u); do
    echo "User logged in: $USER"
done
```

### The continue and break Commands

Control loop execution:

```bash
#!/bin/bash

for NUM in 1 2 3 4 5; do
    # Skip even numbers
    if [ $((NUM % 2)) -eq 0 ]; then
        continue  # Skip to next iteration
    fi
    
    echo "Processing odd number: $NUM"
    
    # Stop if we reach 5
    if [ $NUM -eq 5 ]; then
        break  # Exit the loop entirely
    fi
done
```

-----

## 3.4 The while Loop: Repeating While a Condition is True

The `while` loop continues executing as long as a condition remains true. It's useful when you don't know in advance how many times to loop.

### Basic while Loop Syntax

```bash
while [ condition ]; do
    # Commands
done
```

### Example 1: Counting Loop

```bash
#!/bin/bash

COUNT=1

while [ $COUNT -le 5 ]; do
    echo "Count: $COUNT"
    COUNT=$((COUNT + 1))  # Increment counter
done

echo "Done!"
```

**Output:**
```
Count: 1
Count: 2
Count: 3
Count: 4
Count: 5
Done!
```

### Example 2: Read File Line by Line

This is one of the most common uses of `while`:

```bash
#!/bin/bash

FILE="names.txt"

while read LINE; do
    echo "Processing: $LINE"
done < "$FILE"
```

**How it works:**
- `read LINE` reads one line from input
- `< "$FILE"` redirects the file into the loop
- Loop continues until all lines are read

### Example 3: Menu System

```bash
#!/bin/bash

while true; do
    echo ""
    echo "===== MENU ====="
    echo "1. Show date"
    echo "2. Show current directory"
    echo "3. List files"
    echo "4. Exit"
    echo "================"
    
    read -p "Choose an option: " CHOICE
    
    case $CHOICE in
        1)
            date
            ;;
        2)
            pwd
            ;;
        3)
            ls -la
            ;;
        4)
            echo "Goodbye!"
            break
            ;;
        *)
            echo "Invalid option!"
            ;;
    esac
done
```

### Real-World Example: Retry Logic

```bash
#!/bin/bash

MAX_ATTEMPTS=5
ATTEMPT=1

while [ $ATTEMPT -le $MAX_ATTEMPTS ]; do
    echo "Attempting to connect... (Attempt $ATTEMPT/$MAX_ATTEMPTS)"
    
    # Try to ping a server
    if ping -c 1 google.com &> /dev/null; then
        echo "✓ Connection successful!"
        break
    else
        echo "✗ Connection failed."
        
        if [ $ATTEMPT -eq $MAX_ATTEMPTS ]; then
            echo "Maximum attempts reached. Giving up."
            exit 1
        fi
        
        ATTEMPT=$((ATTEMPT + 1))
        sleep 2  # Wait 2 seconds before retrying
    fi
done
```

### Real-World Example: Monitor Disk Space

```bash
#!/bin/bash

THRESHOLD=90

while true; do
    # Get current disk usage percentage
    USAGE=$(df / | awk 'NR==2 {print $5}' | tr -d '%')
    
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] Disk usage: ${USAGE}%"
    
    if [ $USAGE -gt $THRESHOLD ]; then
        echo "⚠️  WARNING: Disk usage above ${THRESHOLD}%!"
        # Could send email or notification here
    fi
    
    # Check every 5 minutes
    sleep 300
done
```

-----

## 3.5 Combining Control Structures

The real power comes from nesting and combining if statements and loops:

### Example: Process Only Valid Files

```bash
#!/bin/bash

for FILE in *.log; do
    # Skip if no files match
    [ -e "$FILE" ] || continue
    
    # Check if file is readable and not empty
    if [ -r "$FILE" ] && [ -s "$FILE" ]; then
        echo "Processing: $FILE"
        
        # Count error lines
        ERROR_COUNT=$(grep -i "error" "$FILE" | wc -l)
        
        if [ $ERROR_COUNT -gt 0 ]; then
            echo "  Found $ERROR_COUNT errors"
        else
            echo "  No errors found"
        fi
    else
        echo "Skipping: $FILE (unreadable or empty)"
    fi
done
```

### Example: Nested Loops for Directory Tree

```bash
#!/bin/bash

# Process files in multiple directories
for DIR in projects documents downloads; do
    if [ -d "$HOME/$DIR" ]; then
        echo "Checking directory: $DIR"
        
        for FILE in "$HOME/$DIR"/*.txt; do
            [ -e "$FILE" ] || continue
            
            SIZE=$(stat -f%z "$FILE" 2>/dev/null || stat -c%s "$FILE")
            
            if [ $SIZE -gt 10000 ]; then
                echo "  Large file: $(basename "$FILE") ($SIZE bytes)"
            fi
        done
    fi
done
```

-----

## 3.6 Complete Practical Example: File Organizer

Let's build a script that uses everything from Units 2 and 3:

```bash
#!/usr/bin/env bash
#
# Script: organize_files.sh
# Purpose: Organize files by extension into subdirectories
# Usage: ./organize_files.sh [directory]

# Use current directory if none specified
TARGET_DIR="${1:-.}"

# Check if directory exists
if [ ! -d "$TARGET_DIR" ]; then
    echo "ERROR: Directory '$TARGET_DIR' not found."
    exit 1
fi

echo "Organizing files in: $TARGET_DIR"
echo "=================================="

# Counter variables
MOVED_COUNT=0
ERROR_COUNT=0

# Loop through all files
for FILE in "$TARGET_DIR"/*; do
    # Skip if not a regular file
    [ -f "$FILE" ] || continue
    
    # Get filename and extension
    BASENAME=$(basename "$FILE")
    EXTENSION="${BASENAME##*.}"
    
    # Skip files without extension
    if [ "$EXTENSION" = "$BASENAME" ]; then
        echo "Skipping: $BASENAME (no extension)"
        continue
    fi
    
    # Create directory for this extension
    EXT_DIR="$TARGET_DIR/$EXTENSION"
    
    if [ ! -d "$EXT_DIR" ]; then
        mkdir "$EXT_DIR"
        echo "Created directory: $EXTENSION/"
    fi
    
    # Move the file
    if mv "$FILE" "$EXT_DIR/"; then
        echo "Moved: $BASENAME → $EXTENSION/"
        MOVED_COUNT=$((MOVED_COUNT + 1))
    else
        echo "ERROR: Failed to move $BASENAME"
        ERROR_COUNT=$((ERROR_COUNT + 1))
    fi
done

# Summary
echo "=================================="
echo "Organization complete!"
echo "Files moved: $MOVED_COUNT"
echo "Errors: $ERROR_COUNT"

if [ $ERROR_COUNT -eq 0 ]; then
    exit 0
else
    exit 1
fi
```

**Test it:**
```bash
# Create test environment
mkdir test_organize
cd test_organize
touch report.pdf document.docx photo.jpg script.sh notes.txt

# Run the organizer
../organize_files.sh .

# Check results
ls -la
```

-----

## Common Mistakes and How to Avoid Them

### 1. Forgetting Spaces in Conditions

```bash
if [$AGE -gt 18]; then    # ❌ No spaces around brackets
if [ $AGE -gt 18 ]; then  # ✅ Correct
```

### 2. Using = for Number Comparison

```bash
if [ $AGE = 18 ]; then     # ⚠️  String comparison (works but not intended)
if [ $AGE -eq 18 ]; then   # ✅ Numeric comparison
```

### 3. Not Quoting Variables

```bash
if [ $FILE = "test.txt" ]; then    # ❌ Breaks if FILE has spaces or is empty
if [ "$FILE" = "test.txt" ]; then  # ✅ Safe
```

### 4. Forgetting to Check for Empty Loops

```bash
for FILE in *.txt; do
    rm "$FILE"  # ❌ Tries to delete "*.txt" if no files match
done

for FILE in *.txt; do
    [ -e "$FILE" ] || continue  # ✅ Skip if no match
    rm "$FILE"
done
```

### 5. Infinite Loops

```bash
COUNT=1
while [ $COUNT -le 10 ]; do
    echo $COUNT
    # ❌ Forgot to increment! Runs forever
done

# ✅ Remember to update your condition variable
while [ $COUNT -le 10 ]; do
    echo $COUNT
    COUNT=$((COUNT + 1))
done
```

-----

## Practice Exercises

### Exercise 1: Number Guessing Game
Create a script where:
- The script picks a random number 1-10
- User guesses until correct
- Show "too high" or "too low" hints

### Exercise 2: File Type Reporter
Write a script that:
- Loops through all files in current directory
- Counts how many are: directories, text files, executable files
- Displays totals at the end

### Exercise 3: Backup Validator
Create a script that:
- Takes a directory path as argument
- For each file, check if a `.bak` version exists
- If not, create one
- Report all actions taken

**Solution hint for Exercise 3:**
```bash
#!/bin/bash
DIR="${1:-.}"
for FILE in "$DIR"/*; do
    [ -f "$FILE" ] || continue
    BACKUP="${FILE}.bak"
    if [ ! -f "$BACKUP" ]; then
        cp "$FILE" "$BACKUP"
        echo "Created backup: $(basename "$BACKUP")"
    fi
done
```

-----

## Summary

You now have the tools to create intelligent, powerful scripts:

✅ **if statements** - Make decisions based on conditions  
✅ **Test conditions** - Check files, strings, numbers  
✅ **for loops** - Process lists and files efficiently  
✅ **while loops** - Repeat until a condition changes  
✅ **Combining structures** - Build complex automation logic  

These control structures are the foundation of scripting. Every automation task you'll ever write uses these patterns.

**Next Up:** In Unit 4, we'll learn about functions—how to organize your code into reusable blocks, making your scripts cleaner and more maintainable. You'll also learn about script organization and professional coding practices.
