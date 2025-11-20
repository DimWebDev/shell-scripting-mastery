# Unit 8: Package Managers and System Administration

**Learning Objectives:**
- Master package managers for different operating systems (apt, yum/dnf, brew)
- Create cross-platform installation scripts
- Understand cron for automated task scheduling
- Monitor system health and resources
- Automate system administration tasks

**Estimated Time:** 2-3 hours

-----

## Introduction: Automating System Management

Up to this point, we've focused on the shell scripting language itself. Now we apply those skills to real system administration tasks. Professional sysadmins don't manually install software or check system status—they write scripts to automate it.

This unit covers the tools and patterns you'll use to manage servers, workstations, and development environments automatically.

-----

## 8.1 Package Managers: Installing Software Automatically

Different operating systems use different package managers. A package manager is like an "app store" for the command line—it handles downloading, installing, updating, and removing software.

### Linux: APT (Advanced Package Tool)

Used by Debian, Ubuntu, and derivatives. Requires `sudo` for most operations.

**Basic Commands:**
```bash
# Update package list (always do this first)
sudo apt update

# Install a package
sudo apt install package_name

# Install multiple packages
sudo apt install git curl wget

# Install without prompting (-y = yes to all)
sudo apt install -y python3

# Search for packages
apt search keyword

# Show package information
apt show package_name

# Remove a package
sudo apt remove package_name

# Remove package and config files
sudo apt purge package_name

# Clean up (remove downloaded packages)
sudo apt autoclean
sudo apt autoremove
```

**Real-world example: Setting up a web server**
```bash
#!/bin/bash

# Update system
sudo apt update && sudo apt upgrade -y

# Install Apache, PHP, MySQL
sudo apt install -y apache2 php mysql-server

# Enable and start services
sudo systemctl enable apache2
sudo systemctl start apache2

echo "Web server setup complete!"
```

### Linux: YUM/DNF (Red Hat/CentOS/Fedora)

Used by Red Hat-based distributions.

**YUM (older systems):**
```bash
sudo yum update
sudo yum install package_name
sudo yum remove package_name
```

**DNF (newer systems, Fedora 22+):**
```bash
sudo dnf update
sudo dnf install package_name
sudo dnf remove package_name
```

### macOS: Homebrew

macOS doesn't come with a package manager, but Homebrew fills that gap. No `sudo` needed.

**Installation (one-time setup):**
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**Basic Commands:**
```bash
# Update Homebrew itself
brew update

# Install a package
brew install package_name

# Install GUI applications (Casks)
brew install --cask google-chrome
brew install --cask visual-studio-code

# Search for packages
brew search keyword

# List installed packages
brew list

# Remove a package
brew uninstall package_name

# Clean up old versions
brew cleanup
```

**Real-world example: Setting up a development environment**
```bash
#!/bin/bash

# Install development tools
brew install git node python3

# Install GUI apps
brew install --cask iterm2 visual-studio-code

# Install fonts (for better terminal experience)
brew tap homebrew/cask-fonts
brew install --cask font-fira-code

echo "Development environment ready!"
```

### Cross-Platform Installation Script

Combining everything into a reusable function:

```bash
#!/bin/bash

# Function to install packages across different OS
install_package() {
    local PACKAGE=$1
    
    if [[ "$OSTYPE" == "linux-gnu"* ]]; then
        # Linux - try apt first, then yum/dnf
        if command -v apt &> /dev/null; then
            sudo apt update && sudo apt install -y "$PACKAGE"
        elif command -v dnf &> /dev/null; then
            sudo dnf install -y "$PACKAGE"
        elif command -v yum &> /dev/null; then
            sudo yum install -y "$PACKAGE"
        else
            echo "ERROR: No supported package manager found"
            return 1
        fi
    elif [[ "$OSTYPE" == "darwin"* ]]; then
        # macOS - use Homebrew
        if ! command -v brew &> /dev/null; then
            echo "ERROR: Homebrew not installed. Install from https://brew.sh/"
            return 1
        fi
        brew install "$PACKAGE"
    else
        echo "ERROR: Unsupported OS: $OSTYPE"
        return 1
    fi
}

# Usage examples
install_package "git"
install_package "curl"
install_package "python3"
```

-----

## 8.2 Cron: Automated Task Scheduling

Cron is the Unix scheduler. It runs commands at specified times, making it perfect for backups, maintenance, and monitoring.

### Understanding Cron Syntax

A cron job is defined by five time fields followed by the command:

```
* * * * * /path/to/command
│ │ │ │ │
│ │ │ │ └─ Day of week (0-6, Sunday=0)
│ │ │ └─── Month (1-12)
│ │ └───── Day of month (1-31)
│ └─────── Hour (0-23)
└───────── Minute (0-59)
```

**Special characters:**
- `*` = Every (wildcard)
- `,` = List (e.g., `1,3,5`)
- `-` = Range (e.g., `1-5`)
- `/` = Step (e.g., `*/15` = every 15 units)

### Common Cron Schedules

```bash
# Every minute
* * * * * /path/to/script.sh

# Every 15 minutes
*/15 * * * * /path/to/script.sh

# Every hour at minute 0
0 * * * * /path/to/script.sh

# Every day at 3:00 AM
0 3 * * * /path/to/script.sh

# Every Monday at 9:00 AM
0 9 * * 1 /path/to/script.sh

# Every weekday (Monday-Friday) at 6:00 PM
0 18 * * 1-5 /path/to/script.sh

# First day of every month at midnight
0 0 1 * * /path/to/script.sh
```

### Managing Cron Jobs

**View current cron jobs:**
```bash
crontab -l
```

**Edit cron jobs:**
```bash
crontab -e
```

**Remove all cron jobs:**
```bash
crontab -r
```

**Example cron file:**
```bash
# Backup database every day at 2 AM
0 2 * * * /home/user/scripts/backup_db.sh

# Check disk space every hour
0 * * * * /home/user/scripts/check_disk.sh

# Rotate logs weekly (Sunday at 3 AM)
0 3 * * 0 /home/user/scripts/rotate_logs.sh

# Send system report every Monday at 8 AM
0 8 * * 1 /home/user/scripts/system_report.sh | mail -s "Weekly Report" admin@example.com
```

### Cron Best Practices

1. **Use absolute paths** - Cron doesn't have your PATH
2. **Redirect output** - Capture logs for debugging
3. **Test manually first** - Run the command before scheduling
4. **Handle errors** - Scripts should be robust (use `set -e`)

**Example: Robust cron script**
```bash
#!/bin/bash
set -euo pipefail

# Log file for this cron job
LOG_FILE="/var/log/my_backup.log"

# Function to log messages
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $*" >> "$LOG_FILE"
}

log "Starting backup..."

# Your backup logic here
tar -czf "/backups/backup_$(date +%Y%m%d).tar.gz" /important/data

log "Backup completed successfully"
```

**Cron entry:**
```bash
# Run backup every day at 3 AM, log output
0 3 * * * /home/user/scripts/backup.sh >> /var/log/cron_backup.log 2>&1
```

-----

## 8.3 System Monitoring and Health Checks

Combining our scripting skills with system commands to monitor server health.

### CPU and Memory Monitoring

```bash
#!/bin/bash

# Get CPU usage (percentage)
cpu_usage() {
    # Linux
    if [[ "$OSTYPE" == "linux-gnu"* ]]; then
        top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}'
    # macOS
    elif [[ "$OSTYPE" == "darwin"* ]]; then
        ps -A -o %cpu | awk '{s+=$1} END {print s}'
    fi
}

# Get memory usage (percentage)
memory_usage() {
    if [[ "$OSTYPE" == "linux-gnu"* ]]; then
        free | awk 'NR==2{printf "%.0f", $3*100/$2 }'
    elif [[ "$OSTYPE" == "darwin"* ]]; then
        vm_stat | awk '/Pages active/ {print $3}' | tr -d '.' | awk '{print int($1 * 4096 / 1024 / 1024 / 1024 * 100)}'
    fi
}

echo "CPU Usage: $(cpu_usage)%"
echo "Memory Usage: $(memory_usage)%"
```

### Disk Space Monitoring

```bash
#!/bin/bash

THRESHOLD=90

# Check all mounted filesystems
df -h | awk 'NR>1 {print $1, $5}' | while read FS USAGE; do
    # Remove % sign
    PERCENT=$(echo "$USAGE" | tr -d '%')
    
    if [ "$PERCENT" -gt "$THRESHOLD" ]; then
        echo "WARNING: $FS is ${PERCENT}% full"
    fi
done
```

### Process Monitoring

```bash
#!/bin/bash

# Check if a process is running
is_process_running() {
    local PROCESS=$1
    pgrep -f "$PROCESS" > /dev/null
}

# Example: Ensure web server is running
if ! is_process_running "apache2"; then
    echo "Apache is not running. Starting..."
    sudo systemctl start apache2
fi

if ! is_process_running "mysql"; then
    echo "MySQL is not running. Starting..."
    sudo systemctl start mysql
fi
```

### Network Connectivity Check

```bash
#!/bin/bash

# Test internet connectivity
test_connectivity() {
    if ping -c 1 -W 2 8.8.8.8 &> /dev/null; then
        echo "✓ Internet connection OK"
        return 0
    else
        echo "✗ No internet connection"
        return 1
    fi
}

# Test specific service
test_service() {
    local HOST=$1
    local PORT=$2
    
    if nc -z -w5 "$HOST" "$PORT" &> /dev/null; then
        echo "✓ $HOST:$PORT is accessible"
        return 0
    else
        echo "✗ $HOST:$PORT is not accessible"
        return 1
    fi
}

# Usage
test_connectivity
test_service "google.com" 80
test_service "localhost" 3306  # MySQL
```

-----

## 8.4 Complete System Administration Script

Putting it all together: A comprehensive server health check script.

```bash
#!/bin/bash
set -euo pipefail

# Configuration
THRESHOLD_CPU=80
THRESHOLD_MEMORY=85
THRESHOLD_DISK=90
ADMIN_EMAIL="admin@example.com"
LOG_FILE="/var/log/system_health.log"

# Logging function
log() {
    local LEVEL=$1
    local MSG=$2
    local TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")
    echo "[$TIMESTAMP] [$LEVEL] $MSG" >> "$LOG_FILE"
    
    # Also print warnings/errors to console
    if [[ "$LEVEL" == "WARN" || "$LEVEL" == "ERROR" ]]; then
        echo "$LEVEL: $MSG"
    fi
}

# System monitoring functions
get_cpu_usage() {
    if [[ "$OSTYPE" == "linux-gnu"* ]]; then
        top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}' | awk '{print int($1)}'
    elif [[ "$OSTYPE" == "darwin"* ]]; then
        ps -A -o %cpu | awk '{s+=$1} END {print int(s)}'
    fi
}

get_memory_usage() {
    if [[ "$OSTYPE" == "linux-gnu"* ]]; then
        free | awk 'NR==2{printf "%.0f", $3*100/$2 }'
    elif [[ "$OSTYPE" == "darwin"* ]]; then
        vm_stat | awk '/Pages active/ {print $3}' | tr -d '.' | awk '{print int($1 * 4096 / 1024 / 1024 / 1024 * 100)}'
    fi
}

check_disk_space() {
    local ISSUES=0
    
    df -h | awk 'NR>1 {print $1, $5}' | while read FS USAGE; do
        PERCENT=$(echo "$USAGE" | tr -d '%')
        
        if [ "$PERCENT" -gt "$THRESHOLD_DISK" ]; then
            log "WARN" "Disk $FS is ${PERCENT}% full"
            ISSUES=$((ISSUES + 1))
        fi
    done
    
    return $ISSUES
}

check_services() {
    local SERVICES=("sshd" "cron")
    local ISSUES=0
    
    for SERVICE in "${SERVICES[@]}"; do
        if ! systemctl is-active --quiet "$SERVICE" 2>/dev/null; then
            log "WARN" "Service $SERVICE is not running"
            ISSUES=$((ISSUES + 1))
        fi
    done
    
    return $ISSUES
}

# Main health check
main() {
    log "INFO" "Starting system health check"
    
    local ALERTS=0
    
    # CPU Check
    CPU=$(get_cpu_usage)
    if [ "$CPU" -gt "$THRESHOLD_CPU" ]; then
        log "WARN" "High CPU usage: ${CPU}%"
        ALERTS=$((ALERTS + 1))
    fi
    
    # Memory Check
    MEMORY=$(get_memory_usage)
    if [ "$MEMORY" -gt "$THRESHOLD_MEMORY" ]; then
        log "WARN" "High memory usage: ${MEMORY}%"
        ALERTS=$((ALERTS + 1))
    fi
    
    # Disk Check
    if ! check_disk_space; then
        ALERTS=$((ALERTS + 1))
    fi
    
    # Service Check
    if ! check_services; then
        ALERTS=$((ALERTS + 1))
    fi
    
    # Summary
    if [ "$ALERTS" -gt 0 ]; then
        log "WARN" "Health check completed with $ALERTS alerts"
        
        # Send email alert (if mail is configured)
        if command -v mail &> /dev/null; then
            tail -20 "$LOG_FILE" | mail -s "System Health Alert" "$ADMIN_EMAIL"
        fi
    else
        log "INFO" "All systems healthy"
    fi
}

# Run the health check
main
```

**To schedule this with cron:**
```bash
# Check system health every 15 minutes
*/15 * * * * /path/to/health_check.sh
```

-----

## Common System Administration Tasks

### User Management
```bash
#!/bin/bash

create_user() {
    local USERNAME=$1
    local PASSWORD=$2
    
    # Create user
    sudo useradd -m "$USERNAME"
    
    # Set password
    echo "$USERNAME:$PASSWORD" | sudo chpasswd
    
    # Add to sudo group (optional)
    sudo usermod -aG sudo "$USERNAME"
    
    echo "User $USERNAME created successfully"
}

# Usage
create_user "john_doe" "secure_password123"
```

### Log Rotation
```bash
#!/bin/bash

rotate_logs() {
    local LOG_DIR=${1:-"/var/log"}
    local MAX_SIZE=${2:-"100M"}
    
    find "$LOG_DIR" -name "*.log" -size +"$MAX_SIZE" -exec gzip {} \;
    find "$LOG_DIR" -name "*.log.gz" -mtime +30 -delete
}

rotate_logs
```

### Backup Automation
```bash
#!/bin/bash

backup_directory() {
    local SOURCE=$1
    local DEST=$2
    local TIMESTAMP=$(date +%Y%m%d_%H%M%S)
    
    tar -czf "${DEST}/backup_${TIMESTAMP}.tar.gz" "$SOURCE"
    
    # Keep only last 7 backups
    ls -t "${DEST}"/backup_*.tar.gz | tail -n +8 | xargs rm -f
}

backup_directory "/home/user" "/backups"
```

-----

## Practice Exercises

### Exercise 1: Package Installer
Create a script that reads a list of packages from a file and installs them using the appropriate package manager for the current OS.

### Exercise 2: Cron Job Manager
Write a script that can add, remove, and list cron jobs programmatically (without using `crontab -e`).

### Exercise 3: System Report Generator
Create a script that generates a comprehensive system report including:
- OS version
- CPU and memory usage
- Disk space
- Running processes
- Network interfaces

**Save the report to a timestamped file and optionally email it.**

-----

## Summary

You now have the tools to automate system administration:

✅ **Package Managers** - Install software across different OS  
✅ **Cron Scheduling** - Automate tasks at specific times  
✅ **System Monitoring** - Track CPU, memory, disk, and services  
✅ **Health Checks** - Proactive system maintenance  

**Next Up:** In Unit 9, we'll build complete practical projects that combine everything we've learned into real-world applications.
