This is a comprehensive technical manual for shell scripting syntax, core commands, and package management. It covers the structural rules of Bash/Zsh and provides a deep reference for the tools you will use daily.

-----

# Part 1: The Syntax of Shell Scripting

Shell scripting is not just about running commands one after another; it is a full programming language with its own syntax rules. Understanding these rules is critical to writing scripts that do not break when they encounter unexpected spaces or empty variables.

## 1\. The Interpreter (The Shebang)

Every script must begin with a "Shebang" (`#!`). This tells the operating system which program to use to interpret the text file.

  * **Format:** `#!<path_to_interpreter>`
  * **Standard Linux:** `#!/bin/bash`
  * **Portable (Recommended):** `#!/usr/bin/env bash` (Finds bash wherever it is installed).

## 2\. Comments

  * **Single Line:** Any line starting with `#` is ignored by the shell.
  * **Inline:** You can place `#` after a command.

<!-- end list -->

```bash
# This is a comment
echo "Hello" # This prints Hello
```

## 3\. Variables and Quoting Rules

This is the most common source of errors.

### Defining Variables

  * **No Spaces:** You cannot have spaces around the `=` sign.
  * **Case Sensitivity:** `VAR` is different from `var`.

<!-- end list -->

```bash
NAME="Alice"       # Correct
NAME = "Alice"     # Incorrect (Shell tries to run command 'NAME')
```

### Accessing Variables

Use `$` to retrieve the value.

  * `$NAME` works, but `${NAME}` is safer and cleaner, especially when concatenating text.

<!-- end list -->

```bash
FRUIT="apple"
echo "I have 5 $FRUITs"   # ERROR: Looks for variable $FRUITs
echo "I have 5 ${FRUIT}s" # CORRECT: Prints "I have 5 apples"
```

### Quoting (The Golden Rules)

  * **Single Quotes (`' '`):** Literal string. No variable expansion happens.
      * `echo '$NAME'` output: `$NAME`
  * **Double Quotes (`" "`):** Weak quoting. Variables are expanded, but spaces are preserved.
      * `echo "$NAME"` output: `Alice`
  * **No Quotes:** Dangerous. If the variable contains spaces, the shell treats the parts as separate arguments.

**Example of the "No Quote" Bug:**

```bash
FILE="My Document.txt"
rm $FILE  # DANGER: This executes 'rm My' and 'rm Document.txt'
rm "$FILE" # SAFE: This executes 'rm "My Document.txt"'
```

## 4\. Command Substitution

You often want to save the *result* of a command into a variable.

  * **Syntax:** `$(command)`
  * **Legacy:** `` `command` `` (Avoid this; it's hard to nest).

<!-- end list -->

```bash
CURRENT_DIR=$(pwd)
echo "We are in $CURRENT_DIR"
```

## 5\. Exit Codes

Every command returns a number when it finishes, called the **Exit Status**.

  * `0` = Success.
  * `1-255` = Error.
  * variable `$?` holds the exit code of the *last* command.

<!-- end list -->

```bash
ls /non_existent_folder
echo $? # Prints 2 (or 1, depending on OS) indicating failure
```

-----

# Part 2: Core Commands & Arguments Reference

This section details the commands you will use 90% of the time.

## 1\. File System Manipulation

### `ls` - List Directory Contents

  * `ls`: Simple list.
  * `ls -l`: Long format (permissions, owner, size, date).
  * `ls -a`: All files (includes hidden dotfiles like `.bashrc`).
  * `ls -h`: Human-readable sizes (KB, MB instead of bytes).
  * **Combo:** `ls -lah` (The standard for checking directories).

### `cd` - Change Directory

  * `cd foldername`: Go to folder.
  * `cd ..`: Go up one level.
  * `cd ~`: Go to user home directory.
  * `cd -`: Go to the *previous* directory you were in (very useful toggle).

### `mkdir` - Make Directory

  * `mkdir folder`: Creates a folder.
  * `mkdir -p path/to/folder`: Create parent directories if they don't exist.
      * *Without `-p`, `mkdir a/b` fails if `a` doesn't exist.*

### `cp` - Copy

  * `cp source destination`: Copy file.
  * `cp -r source_folder dest_folder`: Recursive. Copies folder and contents.
  * `cp -v`: Verbose. Shows you what is being copied.

### `mv` - Move or Rename

  * `mv oldname newname`: Rename.
  * `mv file folder/`: Move file into folder.
  * `mv -f`: Force (overwrite without asking).

### `rm` - Remove (Delete)

  * `rm file`: Delete file.
  * `rm -r folder`: Recursive (delete folder and contents).
  * `rm -f`: Force (no confirmation).
  * **Danger Zone:** `rm -rf /` (Deletes everything on the computer).

-----

## 2\. File Inspection and Searching

### `cat` - Concatenate

Prints the whole file to the screen.

  * `cat file.txt`: View file.
  * `cat -n file.txt`: View with line numbers.

### `less` - Pager

Allows scrolling through large files.

  * `less huge_log.txt`
      * Use arrows to scroll.
      * Press `/` to search.
      * Press `q` to quit.

### `grep` - Global Regular Expression Print

Finds text inside files.

  * `grep "text" file`: Search for "text" in file.
  * `grep -i "text" file`: Case-insensitive search.
  * `grep -r "text" folder`: Recursive search (look in all files in a folder).
  * `grep -v "text"`: Invert match (show lines that DO NOT contain "text").

### `find` - Search for Files

Searches based on file metadata (name, size, time), not content.

  * `find . -name "*.txt"`: Find all text files in current directory (`.`) and subdirectories.
  * `find /var/log -type f -mtime +7`: Find files (`-type f`) in logs older than 7 days (`-mtime +7`).

-----

## 3\. Permissions and Ownership

### `chmod` - Change Mode

  * `chmod +x script.sh`: Make executable.
  * `chmod 777 file`: Read/Write/Execute for everyone (Insecure).
  * `chmod 644 file`: Owner can write, everyone else can only read (Standard for documents).
  * `chmod 755 file`: Owner can write/execute, others can read/execute (Standard for scripts).

### `chown` - Change Owner

  * `chown user:group file`: Changes ownership.
  * `chown -R user:group folder`: Recursive ownership change.

-----

## 4\. Text Processing (The Heavy Lifters)

### `head` & `tail`

  * `head -n 5 file`: Show first 5 lines.
  * `tail -n 5 file`: Show last 5 lines.
  * `tail -f logfile`: **Follow.** Keeps the stream open and prints new lines as they are added to the file (essential for monitoring logs).

### `cut`

Extract columns.

  * `cut -d "," -f 1 file.csv`: Use delimiter comma (`-d ","`) and extract field 1 (`-f 1`).

### `wc` - Word Count

  * `wc -l`: Count lines.
  * `wc -w`: Count words.

**Example:** Count how many files are in a directory.

```bash
ls | wc -l
```

-----

# Part 3: Control Flow Syntax (Logic)

## 1\. The `if` Statement

The spacing inside `[ ]` is mandatory.

**Syntax:**

```bash
if [ condition ]; then
    # commands
elif [ condition ]; then
    # commands
else
    # commands
fi
```

**Common Conditions:**

  * `[ "$A" == "$B" ]`: String equality.
  * `[ "$A" != "$B" ]`: String inequality.
  * `[ -z "$A" ]`: True if string is empty.
  * `[ -n "$A" ]`: True if string is NOT empty.
  * `[ $A -eq $B ]`: Number equal (`-ne` not equal, `-gt` greater than, `-lt` less than).
  * `[ -f "file.txt" ]`: True if file exists.
  * `[ -d "folder" ]`: True if directory exists.

## 2\. Loops

### The `for` Loop

Iterate over a list.

```bash
# Iterate over a string list
for name in Alice Bob Charlie; do
    echo "Hello $name"
done

# Iterate over files (Globbing)
for file in *.jpg; do
    echo "Processing $file"
done
```

### The `while` Loop

Run while a condition is true.

```bash
COUNT=1
while [ $COUNT -le 5 ]; do
    echo "Attempt $COUNT"
    ((COUNT++))
done
```

-----

# Part 4: Package Management (Installing Software)

How you install software depends on your Operating System.

## 1\. Linux (Debian/Ubuntu/WSL2) using `apt`

The standard package manager for Debian-based systems is **APT** (Advanced Package Tool). Because installing software modifies system files, you almost always need `sudo`.

**Update Repositories**
Before installing, always update your local list of available packages.

```bash
sudo apt update
```

**Install a Package**

```bash
sudo apt install package_name
# Example:
sudo apt install python3 git curl
```

  * `-y`: Automatically answer "yes" to prompts (useful in scripts).
      * `sudo apt install -y git`

**Upgrade All Packages**
Updates all installed software to the newest version found in the repositories.

```bash
sudo apt upgrade
```

**Remove a Package**

```bash
sudo apt remove package_name
```

**Search for a Package**

```bash
apt search keyword
```

## 2\. macOS using Homebrew (`brew`)

macOS does not come with a package manager. The industry standard is **Homebrew**.

**Installation (One-time setup)**
Run this command in your macOS terminal (from the official brew.sh):

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**Install a Package**
Unlike Linux, you generally do **not** use `sudo` with brew.

```bash
brew install package_name
# Example:
brew install wget node python
```

**Install GUI Applications (Casks)**
Brew can install standard Mac apps (like Chrome or VS Code).

```bash
brew install --cask google-chrome
brew install --cask visual-studio-code
```

**Update and Upgrade**

```bash
brew update      # Updates the list of packages (formulae)
brew upgrade     # Upgrades the actual installed software
```

**Cleanup**
Brew caches downloads. To free up space:

```bash
brew cleanup
```

-----

# Part 5: Putting it together (Example Script)

Here is a script that uses variables, logic, loops, and package management checks.

```bash
#!/bin/bash

# Script: setup_env.sh
# Purpose: Checks for git and installs it if missing. Backs up config files.

# 1. Define Variables
BACKUP_DIR="$HOME/configs_backup"
DATE_STAMP=$(date +%Y%m%d)

# 2. Check OS type
if [[ "$OSTYPE" == "linux-gnu"* ]]; then
    OS="linux"
elif [[ "$OSTYPE" == "darwin"* ]]; then
    OS="mac"
else
    echo "Unsupported OS"
    exit 1
fi

echo "Detected OS: $OS"

# 3. Function to check command existence
check_git() {
    if command -v git >/dev/null 2>&1; then
        echo "Git is already installed."
    else
        echo "Git not found. Installing..."
        if [ "$OS" == "linux" ]; then
            sudo apt update && sudo apt install -y git
        elif [ "$OS" == "mac" ]; then
            brew install git
        fi
    fi
}

# 4. Run Check
check_git

# 5. Create Backup Directory
if [ ! -d "$BACKUP_DIR" ]; then
    mkdir -p "$BACKUP_DIR"
    echo "Created backup directory at $BACKUP_DIR"
fi

# 6. Loop through config files and backup
# We use a simulated array of files for demonstration
CONFIG_FILES=(".bashrc" ".vimrc" ".zshrc")

echo "Starting backup..."
for file in "${CONFIG_FILES[@]}"; do
    TARGET="$HOME/$file"
    
    if [ -f "$TARGET" ]; then
        cp "$TARGET" "$BACKUP_DIR/${file}_$DATE_STAMP"
        echo "Backed up $file"
    else
        echo "Warning: $file not found, skipping."
    fi
done

echo "Setup complete."
```

### **Next Step**

Would you like a **diagram visualizing the standard Linux directory structure** (e.g., what goes in `/bin`, `/etc`, `/var`) so you know exactly where to look for files when scripting?