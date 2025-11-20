# Unit 5: Input/Output Handling and Text Processing

**Learning Objectives:**
- Master the three standard data streams (STDIN, STDOUT, STDERR)
- Redirect input and output to files and other commands
- Use pipes to build powerful processing pipelines
- Master the "Power Trio" of text processing: Grep, Sed, and Awk

**Estimated Time:** 3-4 hours

-----

## Introduction: Everything is a Stream

In the Unix philosophy, "everything is a file," and text is the universal interface. Programs are designed to accept text as input and produce text as output. This allows you to chain simple tools together to perform complex tasks—like Lego blocks.

Understanding how to control these streams of text is what separates a casual user from a power user.

-----

## 5.1 The Three Standard Streams

Every time you run a command, the system opens three data channels (File Descriptors):

1. **STDIN (0) - Standard Input**: Where the command gets its data (default: keyboard).
2. **STDOUT (1) - Standard Output**: Where the command sends successful results (default: screen).
3. **STDERR (2) - Standard Error**: Where the command sends error messages (default: screen).

Why separate Output and Error? So you can save the results to a file without saving the error messages, or vice versa.

-----

## 5.2 Redirection: Controlling the Flow

You can change where these streams go using redirection operators.

### Output Redirection (`>` and `>>`)

```bash
# > Overwrite: Saves output to file, replacing existing content
echo "Hello World" > message.txt

# >> Append: Adds output to the end of the file
echo "Another line" >> message.txt
```

### Input Redirection (`<`)

Feed a file into a command instead of typing it:

```bash
# Count lines in a file
wc -l < large_file.txt
```

### Error Redirection (`2>` and `2>>`)

Redirect only the error messages (Stream 2):

```bash
# Save errors to a log file, show normal output on screen
ls /nonexistent_folder 2> errors.log
```

### Redirecting Both (`&>`)

Save everything (Output + Errors) to the same place:

```bash
# Useful for background scripts or cron jobs
./backup_script.sh &> full_log.txt
```

### The "Black Hole" (`/dev/null`)

Sometimes you want to silence output completely. `/dev/null` is a special system file that discards everything written to it.

```bash
# Run command silently (hide output and errors)
./noisy_script.sh &> /dev/null
```

-----

## 5.3 Pipes: Connecting Commands

The pipe operator `|` takes the **STDOUT** of the first command and feeds it into the **STDIN** of the second command.

```bash
# ⚡ PIPELINE BREAKDOWN: List files -> Filter -> Count
ls -la | grep ".txt" | wc -l

# Step-by-step execution flow:
# 1. ls -la          → Lists all files with details
# 2. | (pipe)        → Takes output from ls, feeds it to grep
# 3. grep ".txt"     → Filters only lines containing ".txt"
# 4. | (pipe)        → Takes filtered output, feeds it to wc
# 5. wc -l           → Counts the number of lines (= number of .txt files)
#
# Output example: 15 (means 15 text files found)
```

This is the superpower of the command line. You don't need a specialized tool to "count text files created in November"; you just combine `ls`, `grep`, and `wc`.

-----

## 5.4 The Power Trio: Grep, Sed, Awk

These three tools are the heavy lifters of text processing.

### 1. Grep (Global Regular Expression Print)
**Purpose:** Search and filter text.

```bash
# Basic search
grep "error" app.log

# Case insensitive search (-i)
grep -i "error" app.log

# Recursive search in directory (-r)
grep -r "TODO" ./src/

# Invert match (show lines NOT containing pattern) (-v)
grep -v "debug" app.log

# Count matches (-c)
grep -c "failed" auth.log
```

**Real-world example:** Find all active user accounts (lines not starting with #) in config:
```bash
grep -v "^#" /etc/ssh/sshd_config
```

### 2. Sed (Stream Editor)
**Purpose:** Modify and transform text.

The most common use is substitution: `s/find/replace/flags`

```bash
# Replace "foo" with "bar" (first occurrence per line)
echo "foo bar foo" | sed 's/foo/bar/'
# Output: bar bar foo

# Replace ALL occurrences (global flag 'g')
echo "foo bar foo" | sed 's/foo/bar/g'
# Output: bar bar bar

# Delete lines containing "temp"
sed '/temp/d' file.txt

# Print only lines 5 through 10
sed -n '5,10p' file.txt
```

**In-place editing (`-i`):**
Be careful! This modifies the file directly.
```bash
# Linux (GNU sed)
sed -i 's/http:/https:/g' config.xml

# macOS (BSD sed) - requires empty string for backup extension
sed -i '' 's/http:/https:/g' config.xml
```

### 3. Awk (Aho, Weinberger, Kernighan)
**Purpose:** Process structured data (columns, CSVs, logs).

Awk treats text as records (lines) and fields (columns). By default, fields are separated by whitespace.

- `$0` = The whole line
- `$1` = First column
- `$2` = Second column, etc.
- `NF` = Number of Fields (columns)
- `NR` = Number of Records (current line number)

```bash
# Print the first column (Usernames from 'who')
who | awk '{print $1}'

# Print specific columns
echo "Alice 25 Engineer" | awk '{print $1 " is a " $3}'
# Output: Alice is a Engineer

# Filter rows (like grep, but smarter)
# Print rows where the 3rd column is greater than 50
awk '$3 > 50 {print $0}' data.txt

# Change delimiter (e.g., for CSV)
awk -F, '{print $2}' data.csv
```

-----

## 5.5 Practical Examples: Putting It Together

### Example 1: Log Analysis
Find the top 5 IP addresses hitting your web server.

```bash
# ⚡ ADVANCED PIPELINE: Analyze web server access patterns

# Sample log line format:
# 192.168.1.100 - - [01/Jan/2024:10:15:30] "GET /index.html HTTP/1.1" 200

cat access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -n 5
# │              │                  │      │          │          └─ Step 6: Take top 5 results
# │              │                  │      │          └─ Step 5: Sort by count (numeric, reverse order)
# │              │                  │      └─ Step 4: Count unique IPs (-c = count)
# │              │                  └─ Step 3: Sort IPs alphabetically (required for uniq to work)
# │              └─ Step 2: Extract first column ($1 = IP address)
# └─ Step 1: Read the access log file

# 💡 TIP: uniq only works on sorted data, that's why we sort first!
#
# Output example:
#     245 192.168.1.100
#     189 192.168.1.105
#     156 192.168.1.102
#      89 192.168.1.200
#      67 192.168.1.150
#
# Reading: "192.168.1.100 made 245 requests"
```

### Example 2: Bulk Rename
Rename all `.htm` files to `.html`.

```bash
# ⚡ LOOP + SED: Batch file renaming

ls *.htm | while read FILE; do
    # Loop through each .htm file found by ls

    # Use sed to transform the filename
    NEW_NAME=$(echo "$FILE" | sed 's/\.htm$/.html/')
    #          │              │   │ │  │ └─ Replacement: .html
    #          │              │   │ │  └─ Matches end of line ($)
    #          │              │   │ └─ Escaped dot (literal .)
    #          │              │   └─ Pattern: \.htm$
    #          │              └─ s/ = substitution command
    #          └─ Current filename (e.g., "index.htm")

    # Rename the file
    mv "$FILE" "$NEW_NAME"
    # Example: mv "index.htm" "index.html"

    # 💡 TIP: Always quote "$FILE" to handle filenames with spaces!
done

# Before:  page1.htm, page2.htm, page3.htm
# After:   page1.html, page2.html, page3.html
```

### Example 3: Extracting Data from Config
Get the value of a specific setting from a config file `key=value`.

```bash
# Sample config.ini file:
# port=8080
# host=localhost
# debug=true

# ⚡ METHOD 1: Using grep + cut

grep "^port=" config.ini | cut -d= -f2
# │    │                  │   │   └─ Field 2 (= second part after split)
# │    │                  │   └─ Delimiter: split on '=' character
# │    │                  └─ Cut command: extract specific field
# │    └─ Pattern: lines starting with "port="
# └─ Search in config.ini
#
# Flow:  "port=8080" → grep finds it → cut splits on '=' → returns "8080"
# Output: 8080


# ⚡ METHOD 2: Using awk (more powerful)

awk -F= '/^port/ {print $2}' config.ini
# │   │   │        │         └─ Input file
# │   │   │        └─ Action: print second field
# │   │   └─ Pattern: lines starting with "port"
# │   └─ Field separator: '=' character
# └─ Awk command
#
# Flow:  awk splits each line on '=' → finds matching line → prints second part
# Output: 8080

# 💡 TIP: awk is better for complex config parsing, cut is faster for simple cases
```

-----

## Common Mistakes

1. **Overwriting files accidentally:**
   `cat file.txt > file.txt` will erase the file before reading it! Use a temporary file or `sed -i`.

2. **Forgetting to quote variables in grep:**
   `grep $PATTERN file` breaks if pattern has spaces. Use `grep "$PATTERN" file`.

3. **Confusing `>` and `>>`:**
   Using `>` on a log file will wipe your history. Use `>>` to append.

-----

## Practice Exercises

### Exercise 1: The Cleaner
Write a command that removes all empty lines from a file.
*Hint: `grep` or `sed` can do this.*

### Exercise 2: The Summarizer
Given a CSV file `employees.csv` (Name,Role,Salary), write a command to calculate the average salary.
*Hint: `awk` allows math: `{sum+=$3} END {print sum/NR}`*

### Exercise 3: The Error Hunter
Write a script that scans a log file for the word "ERROR", extracts the timestamp (first 2 columns), and saves it to `incident_report.txt`.

-----

## Summary

You now wield the power of text processing:

✅ **Streams** - Input, Output, Error  
✅ **Redirection** - `>`, `>>`, `2>`, `|`  
✅ **Grep** - Finding the needle in the haystack  
✅ **Sed** - Changing text on the fly  
✅ **Awk** - Analyzing structured data and columns  

**Next Up:** In Unit 6, we'll refine these skills with advanced String Manipulation and Regular Expressions—the language of pattern matching.
