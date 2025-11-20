# Unit 1: The Command Line Interface (CLI) Fundamentals

**Learning Objectives:**
- Understand the difference between a terminal and a shell
- Navigate the file system confidently using command-line tools
- Manage files and directories efficiently
- Understand and modify file permissions

**Estimated Time:** 2-3 hours

-----

## Introduction: Why Learn the Command Line?

The command line might seem intimidating at first—no colorful buttons, no mouse clicks, just a blinking cursor waiting for your input. But here's the truth: mastering the command line is like gaining superpowers for your computer. Tasks that take dozens of clicks in a graphical interface can be completed with a single command. Even better, you can automate repetitive work, manage remote servers, and understand how professional developers actually work.

Think of it this way: graphical interfaces (like Finder on Mac or File Explorer on Windows) are like driving an automatic car. The command line is like driving a manual transmission—it requires more initial learning, but gives you complete control and understanding of what's happening under the hood.

-----

## 1.1 The Environment: Terminal vs. Shell

Let's clear up a common confusion right away. When you open that black window on your computer, two different programs are actually working together. Understanding the difference is important.

### The Terminal Emulator

The **terminal emulator** is the window you see—the actual application with a graphical interface. It's the "container" that displays text and accepts your keyboard input. Think of it as the messenger between you and the actual shell program.

**On macOS, you have several options:**
- **Terminal.app** - The built-in terminal that comes with every Mac
- **iTerm2** - A popular third-party terminal with extra features like split panes and better customization

**On Linux, common choices include:**
- **GNOME Terminal** - The default for many Linux distributions
- **Alacritty** - A modern, fast terminal emulator
- **Terminator** - Great for managing multiple terminal sessions

### The Shell

The **shell** is the actual program running inside the terminal. It interprets your commands and executes them. It's the "brain" that understands what you type and makes things happen.

**The two main shells you'll encounter:**

1. **Bash (Bourne Again Shell)**
   - The standard shell on most Linux systems
   - Been around since 1989—rock solid and reliable
   - What you'll use on most servers and cloud systems

2. **Zsh (Z Shell)**
   - The default shell on macOS since Catalina (2019)
   - Highly customizable with themes and plugins
   - Compatible with Bash, meaning most Bash commands work in Zsh

**How to check which shell you're using:**
```bash
echo $SHELL
```

This will output something like `/bin/bash` or `/bin/zsh`.

**A real-world analogy:** If the terminal is a phone, the shell is the person you're talking to. You can use different phones (terminals) to talk to the same person (shell), or different people (shells) can answer on the same phone.

-----

## 1.2 Navigation & File Management

Everything in Linux and macOS is organized as a tree structure. Imagine a real tree turned upside down—the root at the top, branches spreading downward, with files as the leaves at the end.

### Understanding Your Location

You're always standing somewhere in this tree. The directory you're currently in is called the **working directory**.

#### `pwd` - Print Working Directory

This command tells you exactly where you are in the file system.

```bash
pwd
```

**Output example:**
```
/Users/alice/Documents
```

This means you're inside the `Documents` folder, which is inside the `alice` folder, which is inside the `Users` folder, which is at the root (`/`) of your system.

**Real-world use case:** You're following a tutorial that says "make sure you're in the project folder." Run `pwd` to confirm you're in the right place before running commands.

### Moving Around

#### `cd` - Change Directory

This is how you move from one folder to another.

**Basic navigation:**
```bash
# Go into a folder called 'projects'
cd projects

# Go up one level (to the parent directory)
cd ..

# Go to your home directory (your personal user folder)
cd ~

# Go to the previous directory you were in (like a back button)
cd -

# Go directly to a specific path
cd /Users/alice/Documents/Work
```

**Practice exercise:**
```bash
# Start from your home directory
cd ~

# Check where you are
pwd

# Create a test folder structure (we'll learn mkdir soon)
mkdir -p test/subfolder/deep

# Navigate into it
cd test
pwd

# Go deeper
cd subfolder/deep
pwd

# Jump back to home
cd ~
pwd
```

### Viewing Directory Contents

#### `ls` - List

This shows you what files and folders are in your current location.

**Basic usage:**
```bash
ls
```

**Common options (flags) that make it more useful:**
```bash
# Long format - shows permissions, owner, size, and date
ls -l

# Show all files, including hidden ones (files starting with .)
ls -a

# Human-readable sizes (shows "1.2M" instead of "1234567")
ls -h

# Combine flags - this is the most common way to use ls
ls -lah
```

**Example output of `ls -lah`:**
```
drwxr-xr-x  4 alice  staff   128B Nov 20 10:30 .
drwxr-xr-x  8 alice  staff   256B Nov 19 15:22 ..
-rw-r--r--  1 alice  staff   2.3K Nov 20 09:15 report.txt
-rwxr-xr-x  1 alice  staff   1.1K Nov 18 14:00 backup.sh
```

**Breaking down what you see:**
- `drwxr-xr-x` - Permissions (we'll explain this soon)
- `alice` - The owner of the file
- `staff` - The group
- `2.3K` - File size
- `Nov 20 09:15` - Last modified date
- `report.txt` - File name

**Hidden files:** On Unix systems, any file starting with a dot (`.`) is hidden. These are usually configuration files. For example, `.bashrc` or `.zshrc` configure your shell environment.

### Creating Directories

#### `mkdir` - Make Directory

```bash
# Create a single folder
mkdir my_project

# Create multiple folders at once
mkdir folder1 folder2 folder3

# Create nested folders (parent folders that don't exist yet)
mkdir -p projects/python/web_app
```

**The `-p` flag is incredibly useful.** Without it, if you try `mkdir a/b/c` and folder `a` doesn't exist, you'll get an error. With `-p`, it creates all necessary parent directories automatically.

**Real-world example:**
```bash
# Setting up a project structure in one command
mkdir -p my_website/{css,js,images,pages}

# This creates:
# my_website/
#   ├── css/
#   ├── js/
#   ├── images/
#   └── pages/
```

### Copying Files and Directories

#### `cp` - Copy

```bash
# Copy a file
cp source.txt destination.txt

# Copy a file into a directory
cp report.pdf Documents/

# Copy an entire directory and its contents (recursive)
cp -r project_folder project_backup

# Copy with verbose output (shows what's being copied)
cp -v important.txt backup/
```

**The `-r` flag means "recursive"** - it tells `cp` to copy the folder and everything inside it, including subfolders.

**Common mistake:** Forgetting `-r` when copying folders. Without it, you'll get an error saying `cp: -r not specified; omitting directory`.

### Moving and Renaming

#### `mv` - Move

Here's something neat: `mv` does double duty. It both moves files AND renames them (which is really just moving to a new name).

```bash
# Rename a file
mv old_name.txt new_name.txt

# Move a file into a directory
mv report.pdf Documents/

# Move and rename at the same time
mv old_report.txt Documents/final_report.txt

# Move multiple files into a directory
mv file1.txt file2.txt file3.txt archive/

# Force overwrite without asking (use carefully!)
mv -f source.txt destination.txt
```

**Real-world example:**
```bash
# You downloaded a file to Downloads but want it in your project
mv ~/Downloads/data.csv ~/projects/analysis/data.csv
```

### Deleting Files and Directories

#### `rm` - Remove

**⚠️ WARNING:** There's no "Recycle Bin" or "Trash" with `rm`. When you delete something, it's gone forever. Be extremely careful, especially with the `-r` and `-f` flags.

```bash
# Delete a single file
rm unwanted.txt

# Delete multiple files
rm file1.txt file2.txt file3.txt

# Delete a directory and everything inside it (recursive)
rm -r old_project/

# Force deletion without confirmation prompts
rm -f stubborn_file.txt

# The most dangerous command (NEVER RUN THIS)
# rm -rf /
# This would try to delete everything on your computer
```

**Safer alternatives:**
```bash
# Use -i for interactive mode - it asks before deleting each file
rm -i important_file.txt

# Or move files to a trash folder instead of deleting
mv unwanted_file.txt ~/.trash/
```

**Practice exercise - putting it all together:**
```bash
# Create a practice area
mkdir -p practice/test
cd practice

# Create some test files
touch file1.txt file2.txt file3.txt

# Copy a file
cp file1.txt file1_backup.txt

# Rename a file
mv file2.txt renamed.txt

# Move a file into the subfolder
mv file3.txt test/

# Check your work
ls -la
ls test/

# Clean up when done
cd ..
rm -r practice
```

-----

## 1.3 Permissions

Unix-based systems (Linux and macOS) have a sophisticated permission system. Every file and directory has rules about who can read it, write to it, or execute it.

### The Three Permission Types

1. **Read (r)** - Can view the file's contents or list a directory's contents
2. **Write (w)** - Can modify the file or add/delete files in a directory
3. **Execute (x)** - Can run the file as a program or enter a directory

### The Three User Categories

1. **User (u)** - The owner of the file
2. **Group (g)** - Other users in the same group as the owner
3. **Others (o)** - Everyone else on the system

### Reading Permission Strings

When you run `ls -l`, you see permission strings like `-rwxr-xr-x`. Let's decode this:

```
-rwxr-xr-x
│└┬┘└┬┘└┬┘
│ │  │  └─── Others: read + execute
│ │  └────── Group: read + execute
│ └───────── User/Owner: read + write + execute
└─────────── File type (- for file, d for directory)
```

**Examples:**
- `-rw-r--r--` - A file the owner can edit, but others can only read
- `drwxr-xr-x` - A directory the owner can fully control, others can enter and read
- `-rwx------` - A private file only the owner can access
- `-rw-rw-rw-` - A file anyone can edit (usually not a good idea!)

### The `chmod` Command - Change Mode

This is how you change permissions. There are two ways to use it: symbolic mode and numeric (octal) mode.

#### Symbolic Mode (Beginner-Friendly)

```bash
# Make a script executable
chmod +x script.sh

# Remove write permission for everyone
chmod -w file.txt

# Give group members write permission
chmod g+w shared_file.txt

# Remove execute permission for others
chmod o-x private_script.sh
```

#### Numeric Mode (More Common in Practice)

Permissions are represented by numbers:
- **Read = 4**
- **Write = 2**
- **Execute = 1**

You add these numbers together for each user category.

**Common permission codes:**

```bash
# 644 - Standard for regular files
# Owner: 4+2 = 6 (read+write)
# Group: 4 = 4 (read only)
# Others: 4 = 4 (read only)
chmod 644 document.txt

# 755 - Standard for scripts and executables
# Owner: 4+2+1 = 7 (read+write+execute)
# Group: 4+1 = 5 (read+execute)
# Others: 4+1 = 5 (read+execute)
chmod 755 script.sh

# 700 - Private file only you can access
# Owner: 7 (full control)
# Group: 0 (no access)
# Others: 0 (no access)
chmod 700 private_key.txt

# 777 - Everything for everyone (usually a bad idea!)
chmod 777 file.txt
```

**Real-world example:**
```bash
# You wrote a backup script and want to run it
# First, check current permissions
ls -l backup.sh
# Output: -rw-r--r--  (not executable)

# Make it executable for the owner
chmod u+x backup.sh
# Or use numeric: chmod 755 backup.sh

# Now you can run it
./backup.sh
```

**Quick reference table:**

| Number | Permissions | Symbolic |
|--------|-------------|----------|
| 0      | ---         | No access |
| 1      | --x         | Execute only |
| 2      | -w-         | Write only |
| 3      | -wx         | Write + Execute |
| 4      | r--         | Read only |
| 5      | r-x         | Read + Execute |
| 6      | rw-         | Read + Write |
| 7      | rwx         | Full access |

-----

## Practical Navigation Scenario

Let's walk through a realistic workflow:

```bash
# You're starting a new Python project
cd ~
mkdir -p projects/python/todo_app
cd projects/python/todo_app

# Create project structure
mkdir -p src tests docs data

# Create some initial files
touch src/main.py src/utils.py
touch tests/test_main.py
touch README.md .gitignore

# Make a script executable
chmod +x src/main.py

# Check your work
ls -la
tree  # If you have tree installed, it shows a nice visual

# Output might look like:
# .
# ├── README.md
# ├── data/
# ├── docs/
# ├── src/
# │   ├── main.py
# │   └── utils.py
# └── tests/
#     └── test_main.py
```

-----

## Common Beginner Mistakes and How to Avoid Them

1. **Using `rm -rf` carelessly**
   - Always double-check what directory you're in with `pwd`
   - Use `ls` first to see what you're about to delete
   - Consider using trash commands instead

2. **Forgetting where you are**
   - Get in the habit of running `pwd` frequently
   - Configure your shell prompt to show your current directory

3. **Not using tab completion**
   - Start typing a filename and press Tab—the shell will complete it
   - This prevents typos and saves time

4. **Ignoring permissions errors**
   - If you get "Permission denied," check with `ls -l`
   - Don't reflexively use `sudo` for everything

5. **Not backing up before experimenting**
   - Make copies: `cp important.txt important.txt.backup`
   - Or use version control (we'll get to Git later)

-----

## Summary

You now understand the foundational skills for command-line work:

✅ The difference between terminals and shells  
✅ How to navigate the file system confidently  
✅ Creating, copying, moving, and deleting files  
✅ Understanding and modifying file permissions  

These commands might seem simple, but they're the building blocks for everything else. Professional developers use these commands hundreds of times per day. The more you practice, the more natural they'll become—like typing on a keyboard.

**Practice Challenge:**

Try this exercise without looking at the notes:
1. Create a folder structure for a website project
2. Create some placeholder files
3. Set appropriate permissions
4. Navigate around the structure
5. Clean up when done

```bash
# Solution (try it yourself first!)
mkdir -p my_site/{css,js,images}
touch my_site/index.html my_site/css/style.css
chmod 644 my_site/index.html
chmod 755 my_site
ls -laR my_site
rm -r my_site
```

**Next Steps:** In Unit 2, we'll start writing actual shell scripts that combine these commands to automate tasks. You'll learn about variables, user input, and making your scripts reusable.
