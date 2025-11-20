# Unit 6: String Manipulation and Regular Expressions

**Learning Objectives:**
- Manipulate strings natively in Bash (slicing, replacement, length)
- Understand the difference between Globbing (wildcards) and Regex
- Master basic Regular Expression syntax
- Use Regex with `grep` and conditional statements
- Validate user input (emails, phone numbers, dates)

**Estimated Time:** 3-4 hours

-----

## Introduction: Precision Editing

In Unit 5, we learned to process text streams. Now we zoom in. How do you check if a user's input is a valid email address? How do you change a filename from `image_001.png` to `img_001.jpg` without launching a heavy external program?

This unit covers two distinct but related skills:
1. **Native Bash String Manipulation:** Fast, built-in ways to modify variables.
2. **Regular Expressions (Regex):** A universal language for describing text patterns.

-----

## 6.1 Native Bash String Manipulation

Many beginners use `sed` or `awk` just to change a file extension or lowercase a string. This is slow because it starts a new process. Bash can do this natively—and instantly.

### 1. String Length
To count characters in a variable, use `${#VAR}`.

```bash
MESSAGE="Hello World"
echo "Length: ${#MESSAGE}"
# Output: 11
```

### 2. Slicing (Substring Extraction)
Extract parts of a string based on offset (start) and length.
**Syntax:** `${VAR:offset:length}`

```bash
DATE="2025-10-25"

YEAR=${DATE:0:4}   # Start at 0, take 4 -> "2025"
MONTH=${DATE:5:2}  # Start at 5, take 2 -> "10"
DAY=${DATE:8:2}    # Start at 8, take 2 -> "25"

# Negative offset (from end) - Note the space before -2
LAST_TWO=${DATE: -2}
```

### 3. Substring Removal (Trimming)
Useful for removing file extensions or paths.

- `#` removes from the **beginning** (shortest match)
- `##` removes from the **beginning** (longest match)
- `%` removes from the **end** (shortest match)
- `%%` removes from the **end** (longest match)

**Mnemonic:** `#` is on the left of `$`, `%` is on the right of `$`.

```bash
FILE="path/to/image.png"

# Remove path (delete everything up to last /)
FILENAME=${FILE##*/}
echo $FILENAME  # Output: image.png

# Remove extension (delete from last . to end)
NAME=${FILENAME%.*}
echo $NAME      # Output: image

# Get extension (delete everything up to last .)
EXT=${FILE##*.}
echo $EXT       # Output: png
```

### 4. Search and Replace
**Syntax:** `${VAR/pattern/replacement}`

```bash
TEXT="I love apples and apples"

# Replace FIRST occurrence
echo ${TEXT/apples/bananas}
# Output: I love bananas and apples

# Replace ALL occurrences (start with /)
echo ${TEXT//apples/bananas}
# Output: I love bananas and bananas

# Delete substring (replace with nothing)
echo ${TEXT// apples/}
# Output: I love and
```

### 5. Case Conversion (Bash 4.0+)

```bash
VAR="Hello World"

# To Uppercase (^)
echo ${VAR^^}   # HELLO WORLD

# To Lowercase (,)
echo ${VAR,,}   # hello world
```

-----

## 6.2 The Great Confusion: Globbing vs. Regex

Before diving into Regex, we must fix the #1 mistake beginners make.

You have used `*` before to list files: `ls *.txt`.
This is **NOT** Regular Expression. This is **Globbing** (Wildcards).

| Feature | **Globbing (Wildcards)** | **Regular Expressions (Regex)** |
| :--- | :--- | :--- |
| **Used by** | The Shell (`ls`, `cp`, `mv`, `for` loops) | Text Tools (`grep`, `sed`, `awk`, `vim`) |
| **Purpose** | Matching **Filenames** | Matching **Text Content** inside files/variables |
| **`*` means** | "Match any sequence of characters" | "Match the *previous* character 0 or more times" |
| **`?` means** | "Match exactly one character" | "Match the *previous* character 0 or 1 time" |

**Rule:** If you are working with filenames on the disk, use Globbing. If you are working with text inside a file or variable, use Regex.

-----

## 6.3 Regular Expressions (Regex) Fundamentals

Regex is a pattern-matching language used by `grep`, `sed`, `awk`, and many programming languages.

### Basic Anchors and Quantifiers

| Symbol | Meaning | Example | Matches |
| :--- | :--- | :--- | :--- |
| `^` | Start of line | `^Error` | Lines starting with "Error" |
| `$` | End of line | `done$` | Lines ending with "done" |
| `.` | Any single character | `b.t` | bat, bet, bit, bot |
| `*` | 0 or more of previous | `ab*c` | ac, abc, abbc, abbbc |
| `+` | 1 or more of previous | `ab+c` | abc, abbc (NOT ac) |
| `?` | 0 or 1 of previous | `colou?r` | color, colour |
| `[]` | Character set | `[aeiou]` | Any vowel |
| `[^]` | Negated set | `[^0-9]` | Any non-digit |

### Character Classes (Shortcuts)

- `[0-9]` or `\d` : Any digit
- `[a-z]` : Any lowercase letter
- `[A-Z]` : Any uppercase letter
- `[a-zA-Z0-9]` : Alphanumeric

### Grouping and Alternation

- `(a|b)` : Match 'a' OR 'b'
- `(abc)+` : Match "abc" one or more times (e.g., "abcabc")

-----

## 6.4 Using Regex in Scripts

### 1. With `grep`
Use `grep -E` (Extended Regex) for modern syntax like `+`, `?`, `|`.

```bash
# Find lines that look like phone numbers (XXX-XXX-XXXX)
grep -E "[0-9]{3}-[0-9]{3}-[0-9]{4}" contacts.txt

# Find lines starting with "Error" or "Warning"
grep -E "^(Error|Warning)" app.log
```

### 2. With Conditional Statements `[[ ]]`
Bash's advanced test `[[ ]]` supports Regex matching using the `=~` operator.

**Syntax:** `[[ "$STRING" =~ REGEX ]]`

**Example: Validate an Integer**
```bash
#!/bin/bash

read -p "Enter a number: " NUM

# Regex: ^ starts line, [0-9]+ means one or more digits, $ ends line
if [[ "$NUM" =~ ^[0-9]+$ ]]; then
    echo "Valid integer."
else
    echo "Not an integer."
fi
```

**Example: Validate a Date (YYYY-MM-DD)**
```bash
#!/bin/bash

DATE="2025-10-25"
REGEX="^[0-9]{4}-[0-9]{2}-[0-9]{2}$"

if [[ "$DATE" =~ $REGEX ]]; then
    echo "Valid date format."
else
    echo "Invalid format. Use YYYY-MM-DD."
fi
```

**Example: Validate an Email (Simple)**
```bash
EMAIL="user@example.com"
# Simple regex: chars @ chars . chars
REGEX="^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"

if [[ "$EMAIL" =~ $REGEX ]]; then
    echo "Valid email."
else
    echo "Invalid email."
fi
```

-----

## 6.5 Practical Example: The Data Validator

Let's build a script that validates a user registration form.

```bash
#!/bin/bash

validate_input() {
    local INPUT=$1
    local TYPE=$2
    
    case $TYPE in
        "email")
            # Simple email regex
            [[ "$INPUT" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]
            ;;
        "phone")
            # Matches 123-456-7890
            [[ "$INPUT" =~ ^[0-9]{3}-[0-9]{3}-[0-9]{4}$ ]]
            ;;
        "username")
            # Alphanumeric, 3-16 chars
            [[ "$INPUT" =~ ^[a-zA-Z0-9]{3,16}$ ]]
            ;;
        *)
            return 1
            ;;
    esac
}

echo "--- Registration ---"

read -p "Username (3-16 chars, no symbols): " USER
if ! validate_input "$USER" "username"; then
    echo "Invalid username."
    exit 1
fi

read -p "Email: " EMAIL
if ! validate_input "$EMAIL" "email"; then
    echo "Invalid email."
    exit 1
fi

read -p "Phone (XXX-XXX-XXXX): " PHONE
if ! validate_input "$PHONE" "phone"; then
    echo "Invalid phone format."
    exit 1
fi

echo "Registration successful for $USER!"
```

-----

## Common Mistakes

1. **Quoting the Regex variable:**
   In `[[ ]]`, do **NOT** quote the regex variable on the right side.
   ```bash
   # Wrong
   [[ "$VAR" =~ "$REGEX" ]]
   
   # Right
   [[ "$VAR" =~ $REGEX ]]
   ```

2. **Confusing Globbing and Regex:**
   Trying to use `*.txt` inside `grep` or `sed`. Remember: `*` in regex means "repeat previous char", not "any file".

3. **Forgetting Anchors (`^` and `$`)**:
   Without anchors, `[0-9]+` matches "abc123xyz". If you want *only* numbers, use `^[0-9]+$`.

-----

## Practice Exercises

### Exercise 1: The Renamer
Write a script that takes a filename like `My Photo.jpg` and converts it to `my_photo.jpg` (lowercase, spaces to underscores) using native Bash string manipulation.

### Exercise 2: Log Filter
Write a command using `grep -E` to find all lines in a log file that contain an IP address (format: `X.X.X.X` where X is 1-3 digits).

### Exercise 3: Input Sanitizer
Write a script that asks for a "Product Code" (Format: 2 letters, dash, 4 numbers, e.g., `AB-1234`). Loop until the user enters a valid code.

-----

## Summary

You now have precision control over text:

✅ **Native Manipulation** - `${VAR#...}`, `${VAR%...}`, `${VAR/...}`  
✅ **Slicing** - `${VAR:0:5}`  
✅ **Regex vs Globbing** - Knowing when to use which  
✅ **Regex Syntax** - `^`, `$`, `*`, `+`, `[]`  
✅ **Validation** - Using `[[ =~ ]]` to check patterns  

**Next Up:** In Unit 7, we'll tackle Advanced Scripting Topics like Arrays, Error Handling, and Traps to make your scripts robust and professional.
