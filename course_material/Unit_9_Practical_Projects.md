# Unit 9: Practical Projects

**Learning Objectives:**
- Build complete, production-ready shell scripts
- Combine all concepts: variables, functions, loops, conditionals, I/O, text processing
- Handle real-world scenarios: error handling, logging, user input validation
- Create modular, maintainable code
- Apply best practices from the start

**Estimated Time:** 4-6 hours

**Prerequisites:** Units 1-8

-----

## Introduction: From Theory to Practice

You've learned the building blocks. Now it's time to build something real. Each project in this unit combines multiple concepts and teaches you how to structure professional scripts.

**Project Structure Pattern:**
```bash
#!/bin/bash
#
# Script Name: project_name.sh
# Description: What it does
# Author: Your Name
# Date: YYYY-MM-DD
#

set -euo pipefail  # Safety first!

# Configuration (constants)
readonly SCRIPT_NAME="${0##*/}"
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# Global variables
LOG_FILE=""
VERBOSE=false

# Functions (organized by purpose)
setup() { ... }
validate_input() { ... }
main() { ... }

# Main execution
main "$@"
```

-----

## Project 1: Advanced File Backup System

**Combines:** Functions, loops, conditionals, file operations, logging, error handling

**Requirements:**
- Backup multiple directories
- Support compression (tar.gz)
- Rotation (keep only N recent backups)
- Logging with timestamps
- Dry-run mode
- Progress indicators

```bash
#!/bin/bash
#
# Script Name: backup_system.sh
# Description: Advanced backup system with rotation and compression
# Author: Shell Scripting Course
# Date: 2024-01-15
#

set -euo pipefail

# Configuration
readonly SCRIPT_NAME="${0##*/}"
readonly BACKUP_ROOT="/backups"
readonly MAX_BACKUPS=7
readonly COMPRESSION_LEVEL=9

# Global variables
LOG_FILE=""
VERBOSE=false
DRY_RUN=false

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# ═══════════════════════════════════════════════════════════════
# LOGGING FUNCTIONS
# ═══════════════════════════════════════════════════════════════

# Function: setup_logging
# Purpose: Initialize logging system and create log file
# Parameters: None
# Returns: 0 (always succeeds)
# Side effects: Creates BACKUP_ROOT directory and LOG_FILE
setup_logging() {
    local timestamp=$(date +%Y%m%d_%H%M%S)
    LOG_FILE="${BACKUP_ROOT}/backup_${timestamp}.log"

    # Create backup directory if it doesn't exist
    mkdir -p "$BACKUP_ROOT"  # -p = no error if exists, create parents

    log "INFO" "Backup system started"
    log "INFO" "Log file: $LOG_FILE"
}

# Function: log
# Purpose: Write timestamped messages to log file and optionally console
# Parameters:
#   $1 - level: Log level (INFO, WARN, ERROR)
#   $2 - message: The message to log
# Returns: 0 (always succeeds)
# Behavior:
#   - Always logs to file
#   - Shows on console if VERBOSE=true or level=ERROR
#   - Uses color-coding for console output
log() {
    local level=$1
    local message=$2
    local timestamp=$(date "+%Y-%m-%d %H:%M:%S")

    # Write to log file (append)
    echo "[$timestamp] [$level] $message" >> "$LOG_FILE"

    # Conditionally show on console
    if [[ "$VERBOSE" == true || "$level" == "ERROR" ]]; then
        case $level in
            "INFO")  echo -e "${GREEN}[$level]${NC} $message" ;;  # Green
            "WARN")  echo -e "${YELLOW}[$level]${NC} $message" ;; # Yellow
            "ERROR") echo -e "${RED}[$level]${NC} $message" ;;    # Red
        esac
    fi
}

# ═══════════════════════════════════════════════════════════════
# VALIDATION FUNCTIONS
# ═══════════════════════════════════════════════════════════════

# Function: validate_directory
# Purpose: Check if directory exists and is readable
# Parameters:
#   $1 - dir: Directory path to validate
# Returns: 0 if valid, 1 if invalid
# Example: validate_directory "/home/user/docs" || exit 1
validate_directory() {
    local dir=$1

    # Check existence
    if [[ ! -d "$dir" ]]; then
        log "ERROR" "Directory does not exist: $dir"
        return 1
    fi

    # Check readability
    if [[ ! -r "$dir" ]]; then
        log "ERROR" "Directory not readable: $dir"
        return 1
    fi

    return 0  # All checks passed
}

# Function: validate_backup_destination
# Purpose: Ensure backup destination is writable
# Parameters: None (uses global BACKUP_ROOT)
# Returns: 0 if writable, 1 if not
validate_backup_destination() {
    if [[ ! -w "$BACKUP_ROOT" ]]; then
        log "ERROR" "Backup destination not writable: $BACKUP_ROOT"
        return 1
    fi

    return 0
}

# ═══════════════════════════════════════════════════════════════
# BACKUP FUNCTIONS
# ═══════════════════════════════════════════════════════════════

# Function: create_backup
# Purpose: Create compressed tar.gz backup of a directory
# Parameters:
#   $1 - source_dir: Directory to backup (e.g., /home/user/documents)
#   $2 - backup_name: Base name for backup file (e.g., "documents")
# Returns: 0 on success, 1 on failure
# Output: Prints backup file path on success
# Example: create_backup "/home/user/docs" "docs_backup"
#          Creates: /backups/docs_backup_20240115_143022.tar.gz
create_backup() {
    local source_dir=$1
    local backup_name=$2
    local timestamp=$(date +%Y%m%d_%H%M%S)
    local backup_file="${BACKUP_ROOT}/${backup_name}_${timestamp}.tar.gz"

    log "INFO" "Creating backup: $backup_file"

    # Handle dry-run mode
    if [[ "$DRY_RUN" == true ]]; then
        log "INFO" "[DRY RUN] Would create: $backup_file"
        return 0
    fi

    # ⚡ CREATE COMPRESSED ARCHIVE
    # tar: -c (create) -z (gzip) -f (file)
    # -C changes to parent directory, then archives just the folder name
    if tar -czf "$backup_file" -C "$(dirname "$source_dir")" "$(basename "$source_dir")"; then
        log "INFO" "Backup created successfully: $(basename "$backup_file")"
        echo "$backup_file"  # Return path for caller
        return 0
    else
        log "ERROR" "Failed to create backup: $backup_file"
        return 1
    fi
}

# Function: rotate_backups
# Purpose: Keep only the N most recent backups, delete older ones
# Parameters:
#   $1 - backup_pattern: Base name pattern to match (e.g., "documents")
# Returns: 0 (always succeeds)
# Algorithm:
#   1. Find all backups matching pattern
#   2. Sort by modification time (newest first)
#   3. If count > MAX_BACKUPS, delete the oldest ones
# Example: rotate_backups "documents"
#          Keeps: documents_20240115_*.tar.gz (7 most recent)
#          Deletes: older backups beyond limit
rotate_backups() {
    local backup_pattern=$1

    log "INFO" "Rotating backups for pattern: $backup_pattern"

    # Handle dry-run mode
    if [[ "$DRY_RUN" == true ]]; then
        log "INFO" "[DRY RUN] Would rotate backups"
        return 0
    fi

    # ⚡ FIND AND SORT BACKUPS (newest first)
    # find: locates matching files
    # printf '%T@ %p\n': prints timestamp + path
    # sort -nr: numeric reverse sort (newest first)
    # cut: extracts just the path
    local backups=($(find "$BACKUP_ROOT" -name "${backup_pattern}_*.tar.gz" -type f -printf '%T@ %p\n' | sort -nr | cut -d' ' -f2-))

    # Count how many backups exist
    local count=${#backups[@]}

    # ⚡ CLEANUP LOGIC: Remove old backups if we exceed the limit
    if [[ $count -gt $MAX_BACKUPS ]]; then
        local to_remove=$((count - MAX_BACKUPS))
        log "INFO" "Found $count backups, removing $to_remove old ones"

        # Loop starting from index MAX_BACKUPS (keeping 0 through MAX_BACKUPS-1)
        for ((i = MAX_BACKUPS; i < count; i++)); do
            log "INFO" "Removing old backup: $(basename "${backups[i]}")"
            rm -f "${backups[i]}"
        done
    else
        log "INFO" "Only $count backups found (limit: $MAX_BACKUPS), no rotation needed"
    fi
}

# Progress and summary
show_progress() {
    local current=$1
    local total=$2
    local item=$3
    
    if [[ "$VERBOSE" == true ]]; then
        local percentage=$((current * 100 / total))
        echo -ne "\rProgress: $current/$total ($percentage%) - $item"
    fi
}

show_summary() {
    local backups_created=$1
    local total_size=$2
    local start_time=$3
    
    local end_time=$(date +%s)
    local duration=$((end_time - start_time))
    
    log "INFO" "Backup session completed"
    log "INFO" "Backups created: $backups_created"
    log "INFO" "Total backup size: $(numfmt --to=iec-i --suffix=B $total_size)"
    log "INFO" "Duration: ${duration}s"
    
    if [[ "$VERBOSE" == true ]]; then
        echo
        echo "Summary:"
        echo "  Backups created: $backups_created"
        echo "  Total size: $(numfmt --to=iec-i --suffix=B $total_size)"
        echo "  Duration: ${duration}s"
    fi
}

# Main backup function
perform_backup() {
    local dirs=("$@")
    local start_time=$(date +%s)
    local backups_created=0
    local total_size=0
    
    log "INFO" "Starting backup of ${#dirs[@]} directories"
    
    for i in "${!dirs[@]}"; do
        local dir=${dirs[i]}
        local backup_name=$(basename "$dir" | tr '/' '_')
        
        show_progress $((i + 1)) ${#dirs[@]} "$backup_name"
        
        if create_backup "$dir" "$backup_name"; then
            ((backups_created++))
            
            # Get file size if backup was created
            if [[ "$DRY_RUN" == false ]]; then
                local backup_file=$(ls -t "${BACKUP_ROOT}/${backup_name}"_*.tar.gz | head -1)
                local size=$(stat -f%z "$backup_file" 2>/dev/null || stat -c%s "$backup_file" 2>/dev/null || echo 0)
                ((total_size += size))
            fi
            
            rotate_backups "$backup_name"
        fi
    done
    
    if [[ "$VERBOSE" == true ]]; then
        echo # New line after progress
    fi
    
    show_summary $backups_created $total_size $start_time
}

# Usage and help
usage() {
    cat << EOF
Usage: $SCRIPT_NAME [OPTIONS] DIRECTORY [DIRECTORY ...]

Advanced backup system with compression and rotation.

OPTIONS:
    -d, --dry-run     Show what would be done without making changes
    -v, --verbose     Enable verbose output
    -h, --help        Show this help message

EXAMPLES:
    $SCRIPT_NAME /home/user/documents
    $SCRIPT_NAME -v /etc /var/log /home/user
    $SCRIPT_NAME --dry-run /home/user/projects

EOF
}

# Parse command line arguments
parse_args() {
    while [[ $# -gt 0 ]]; do
        case $1 in
            -d|--dry-run)
                DRY_RUN=true
                shift
                ;;
            -v|--verbose)
                VERBOSE=true
                shift
                ;;
            -h|--help)
                usage
                exit 0
                ;;
            -*)
                log "ERROR" "Unknown option: $1"
                usage
                exit 1
                ;;
            *)
                # Collect directories
                DIRECTORIES+=("$1")
                shift
                ;;
        esac
    done
}

# Main function
main() {
    local DIRECTORIES=()
    
    # Parse arguments
    parse_args "$@"
    
    # Setup
    setup_logging
    
    # Validate
    if [[ ${#DIRECTORIES[@]} -eq 0 ]]; then
        log "ERROR" "No directories specified"
        usage
        exit 1
    fi
    
    validate_backup_destination
    
    for dir in "${DIRECTORIES[@]}"; do
        validate_directory "$dir" || exit 1
    done
    
    # Perform backup
    perform_backup "${DIRECTORIES[@]}"
    
    log "INFO" "Backup system finished successfully"
}

# Run main function with all arguments
main "$@"
```

**How to use:**
```bash
# Backup single directory
./backup_system.sh /home/user/documents

# Backup multiple directories with verbose output
./backup_system.sh -v /etc /var/log /home/user

# Dry run to see what would happen
./backup_system.sh --dry-run /home/user/projects
```

**Testing the script:**
```bash
# Create test directories
mkdir -p test_backup/source1 test_backup/source2
echo "test file 1" > test_backup/source1/file1.txt
echo "test file 2" > test_backup/source2/file2.txt

# Run backup
./backup_system.sh -v test_backup/source1 test_backup/source2

# Check results
ls -la backups/
```

-----

## Project 2: Interactive Menu-Driven System Monitor

**Combines:** Functions, loops, case statements, user input, system commands, formatting

**Requirements:**
- Interactive menu system
- Real-time system monitoring
- Process management
- Service control
- Colorized output
- Input validation

```bash
#!/bin/bash
#
# Script Name: system_monitor.sh
# Description: Interactive system monitoring and management tool
# Author: Shell Scripting Course
# Date: 2024-01-15
#

set -euo pipefail

# Configuration
readonly SCRIPT_NAME="${0##*/}"
readonly REFRESH_INTERVAL=2

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
MAGENTA='\033[0;35m'
CYAN='\033[0;36m'
WHITE='\033[1;37m'
NC='\033[0m' # No Color

# Global variables
LAST_UPDATE=0
SYSTEM_INFO=""

# Utility functions
print_header() {
    clear
    echo -e "${CYAN}================================${NC}"
    echo -e "${CYAN}    System Monitor v1.0${NC}"
    echo -e "${CYAN}================================${NC}"
    echo
}

print_menu() {
    echo -e "${WHITE}Choose an option:${NC}"
    echo "1) System Overview"
    echo "2) Process Monitor"
    echo "3) Memory Usage"
    echo "4) Disk Usage"
    echo "5) Network Status"
    echo "6) Service Management"
    echo "7) System Logs"
    echo "8) Performance Graphs"
    echo "9) Exit"
    echo
}

get_system_info() {
    SYSTEM_INFO=$(uname -a)
}

# Monitoring functions
show_system_overview() {
    print_header
    echo -e "${GREEN}System Overview${NC}"
    echo "=================="
    
    echo -e "${YELLOW}OS:${NC} $(uname -s) $(uname -r)"
    echo -e "${YELLOW}Hostname:${NC} $(hostname)"
    echo -e "${YELLOW}Uptime:${NC} $(uptime | awk -F'up ' '{print $2}' | awk -F',' '{print $1}')"
    
    # CPU Info
    if [[ "$OSTYPE" == "linux-gnu"* ]]; then
        echo -e "${YELLOW}CPU:${NC} $(nproc) cores - $(lscpu | grep "Model name" | cut -d: -f2 | xargs)"
    elif [[ "$OSTYPE" == "darwin"* ]]; then
        echo -e "${YELLOW}CPU:${NC} $(sysctl -n hw.ncpu) cores - $(sysctl -n machdep.cpu.brand_string)"
    fi
    
    # Memory
    if [[ "$OSTYPE" == "linux-gnu"* ]]; then
        local mem_total=$(free -h | awk 'NR==2{print $2}')
        local mem_used=$(free -h | awk 'NR==2{print $3}')
        echo -e "${YELLOW}Memory:${NC} $mem_used / $mem_total"
    elif [[ "$OSTYPE" == "darwin"* ]]; then
        local mem_total=$(echo "scale=2; $(sysctl -n hw.memsize) / 1024 / 1024 / 1024" | bc)
        echo -e "${YELLOW}Memory:${NC} ${mem_total} GB total"
    fi
    
    # Disk
    local disk_usage=$(df -h / | awk 'NR==2{print $3"/"$2" ("$5" used)"}')
    echo -e "${YELLOW}Disk (/) :${NC} $disk_usage"
    
    # Load average
    local load=$(uptime | awk -F'load average:' '{print $2}' | xargs)
    echo -e "${YELLOW}Load Average:${NC} $load"
    
    echo
    read -p "Press Enter to return to menu..."
}

show_process_monitor() {
    print_header
    echo -e "${GREEN}Process Monitor${NC}"
    echo "==============="
    
    echo "Top 10 processes by CPU usage:"
    echo
    
    if [[ "$OSTYPE" == "linux-gnu"* ]]; then
        ps aux --sort=-%cpu | head -11 | awk 'NR==1{print "\033[1;33m"$0"\033[0m"} NR>1{print}'
    elif [[ "$OSTYPE" == "darwin"* ]]; then
        ps aux -r | head -11 | awk 'NR==1{print "\033[1;33m"$0"\033[0m"} NR>1{print}'
    fi
    
    echo
    echo "Process management options:"
    echo "p) Kill a process by PID"
    echo "n) Kill a process by name"
    echo "r) Return to menu"
    echo
    
    local choice
    read -p "Choose option: " choice
    
    case $choice in
        p|P)
            read -p "Enter PID to kill: " pid
            if [[ "$pid" =~ ^[0-9]+$ ]]; then
                kill "$pid" && echo "Process $pid killed" || echo "Failed to kill process $pid"
            else
                echo "Invalid PID"
            fi
            sleep 2
            ;;
        n|N)
            read -p "Enter process name to kill: " pname
            pkill "$pname" && echo "Processes named '$pname' killed" || echo "Failed to kill processes"
            sleep 2
            ;;
    esac
}

show_memory_usage() {
    print_header
    echo -e "${GREEN}Memory Usage${NC}"
    echo "============="
    
    if [[ "$OSTYPE" == "linux-gnu"* ]]; then
        echo "Memory Information:"
        free -h | awk 'NR==1{print "\033[1;33m"$0"\033[0m"} NR==2{print}'
        echo
        echo "Top 10 memory-consuming processes:"
        ps aux --sort=-%mem | head -11 | awk 'NR==1{print "\033[1;33m"$0"\033[0m"} NR>1{print}'
    elif [[ "$OSTYPE" == "darwin"* ]]; then
        echo "Memory Information:"
        vm_stat | awk '/Pages free/ {free=$3} /Pages active/ {active=$3} /Pages inactive/ {inactive=$3} /Pages wired/ {wired=$3} END {total=free+active+inactive+wired; print "Free: " free*4096/1024/1024 " MB"; print "Active: " active*4096/1024/1024 " MB"; print "Inactive: " inactive*4096/1024/1024 " MB"; print "Wired: " wired*4096/1024/1024 " MB"; print "Total: " total*4096/1024/1024 " MB"}'
        echo
        echo "Top 10 memory-consuming processes:"
        ps aux -m | head -11 | awk 'NR==1{print "\033[1;33m"$0"\033[0m"} NR>1{print}'
    fi
    
    echo
    read -p "Press Enter to return to menu..."
}

show_disk_usage() {
    print_header
    echo -e "${GREEN}Disk Usage${NC}"
    echo "==========="
    
    echo "Filesystem usage:"
    df -h | awk 'NR==1{print "\033[1;33m"$0"\033[0m"} NR>1{print}'
    echo
    
    echo "Largest directories in /home:"
    if [[ -d /home ]]; then
        du -sh /home/* 2>/dev/null | sort -hr | head -10
    fi
    
    echo
    read -p "Press Enter to return to menu..."
}

show_network_status() {
    print_header
    echo -e "${GREEN}Network Status${NC}"
    echo "==============="
    
    echo "Network interfaces:"
    if [[ "$OSTYPE" == "linux-gnu"* ]]; then
        ip addr show | grep -E "^[0-9]+:" | cut -d: -f2 | xargs | awk '{print "\033[1;33mInterface:\033[0m " $0}'
        echo
        echo "IP addresses:"
        ip addr show | grep "inet " | awk '{print $2}' | grep -v "127.0.0.1"
    elif [[ "$OSTYPE" == "darwin"* ]]; then
        ifconfig | grep -E "^[a-z0-9]+:" | awk -F: '{print "\033[1;33mInterface:\033[0m " $1}'
        echo
        echo "IP addresses:"
        ifconfig | grep "inet " | awk '{print $2}' | grep -v "127.0.0.1"
    fi
    
    echo
    echo "Network connectivity test:"
    if ping -c 1 -W 2 8.8.8.8 &>/dev/null; then
        echo -e "${GREEN}✓ Internet connection OK${NC}"
    else
        echo -e "${RED}✗ No internet connection${NC}"
    fi
    
    echo
    read -p "Press Enter to return to menu..."
}

show_service_management() {
    print_header
    echo -e "${GREEN}Service Management${NC}"
    echo "==================="
    
    if [[ "$OSTYPE" == "linux-gnu"* ]]; then
        echo "Common services status:"
        for service in sshd cron apache2 mysql; do
            if systemctl is-active --quiet "$service" 2>/dev/null; then
                echo -e "${GREEN}✓ $service${NC} is running"
            else
                echo -e "${RED}✗ $service${NC} is stopped"
            fi
        done
        
        echo
        echo "Service management options:"
        echo "s) Start a service"
        echo "p) Stop a service"
        echo "r) Restart a service"
        echo "t) Check service status"
        echo "m) Return to menu"
        echo
        
        local choice
        read -p "Choose option: " choice
        
        case $choice in
            s|S)
                read -p "Enter service name to start: " service
                sudo systemctl start "$service" && echo "Service $service started" || echo "Failed to start $service"
                ;;
            p|P)
                read -p "Enter service name to stop: " service
                sudo systemctl stop "$service" && echo "Service $service stopped" || echo "Failed to stop $service"
                ;;
            r|R)
                read -p "Enter service name to restart: " service
                sudo systemctl restart "$service" && echo "Service $service restarted" || echo "Failed to restart $service"
                ;;
            t|T)
                read -p "Enter service name to check: " service
                systemctl status "$service" | head -10
                ;;
        esac
        
        if [[ "$choice" != "m" && "$choice" != "M" ]]; then
            sleep 2
        fi
    else
        echo "Service management is primarily for Linux systems with systemd"
        echo
        read -p "Press Enter to return to menu..."
    fi
}

show_system_logs() {
    print_header
    echo -e "${GREEN}System Logs${NC}"
    echo "============"
    
    if [[ "$OSTYPE" == "linux-gnu"* ]]; then
        echo "Recent system messages:"
        journalctl -n 20 --no-pager | tail -10
    elif [[ "$OSTYPE" == "darwin"* ]]; then
        echo "Recent system logs:"
        log show --last 1h | tail -10
    fi
    
    echo
    read -p "Press Enter to return to menu..."
}

show_performance_graphs() {
    print_header
    echo -e "${GREEN}Performance Graphs${NC}"
    echo "==================="
    
    echo "Simple ASCII graphs (run for a few seconds to see changes):"
    echo
    
    # Simple CPU usage graph
    echo "CPU Usage (last 10 seconds):"
    for i in {1..10}; do
        local cpu_usage=0
        
        if [[ "$OSTYPE" == "linux-gnu"* ]]; then
            cpu_usage=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}' | awk '{print int($1)}')
        elif [[ "$OSTYPE" == "darwin"* ]]; then
            cpu_usage=$(ps -A -o %cpu | awk '{s+=$1} END {print int(s)}')
        fi
        
        local bars=$((cpu_usage / 5))
        printf "%3d%% [" "$cpu_usage"
        for ((j=0; j<bars; j++)); do printf "█"; done
        for ((j=bars; j<20; j++)); do printf " "; done
        printf "]\n"
        
        sleep 1
    done
    
    echo
    read -p "Press Enter to return to menu..."
}

# Main menu loop
main() {
    get_system_info
    
    while true; do
        print_header
        print_menu
        
        local choice
        read -p "Enter your choice (1-9): " choice
        
        case $choice in
            1) show_system_overview ;;
            2) show_process_monitor ;;
            3) show_memory_usage ;;
            4) show_disk_usage ;;
            5) show_network_status ;;
            6) show_service_management ;;
            7) show_system_logs ;;
            8) show_performance_graphs ;;
            9) 
                echo "Goodbye!"
                exit 0
                ;;
            *)
                echo -e "${RED}Invalid option. Please choose 1-9.${NC}"
                sleep 2
                ;;
        esac
    done
}

# Run the main function
main "$@"
```

**How to use:**
```bash
# Make executable and run
chmod +x system_monitor.sh
./system_monitor.sh
```

**Features demonstrated:**
- Interactive menu system
- Colorized output
- Cross-platform compatibility
- Real-time monitoring
- Process management
- Service control
- Input validation

-----

## Project 3: Automated Deployment Script

**Combines:** Functions, error handling, logging, user input, file operations, external commands

**Requirements:**
- Deploy web application from git
- Install dependencies
- Configure environment
- Backup current version
- Rollback capability
- Email notifications

```bash
#!/bin/bash
#
# Script Name: deploy_app.sh
# Description: Automated web application deployment
# Author: Shell Scripting Course
# Date: 2024-01-15
#

set -euo pipefail

# Configuration
readonly SCRIPT_NAME="${0##*/}"
readonly APP_NAME="mywebapp"
readonly APP_DIR="/var/www/$APP_NAME"
readonly BACKUP_DIR="/var/backups/$APP_NAME"
readonly GIT_REPO="https://github.com/user/mywebapp.git"
readonly ADMIN_EMAIL="admin@example.com"

# Global variables
LOG_FILE=""
DEPLOYMENT_ID=""
START_TIME=""
ROLLBACK_NEEDED=false

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

# Logging functions
setup_logging() {
    DEPLOYMENT_ID=$(date +%Y%m%d_%H%M%S)
    LOG_FILE="/var/log/deploy_${APP_NAME}_${DEPLOYMENT_ID}.log"
    
    log "INFO" "Deployment started - ID: $DEPLOYMENT_ID"
    START_TIME=$(date +%s)
}

log() {
    local level=$1
    local message=$2
    local timestamp=$(date "+%Y-%m-%d %H:%M:%S")
    
    echo "[$timestamp] [$level] $message" >> "$LOG_FILE"
    echo -e "[$level] $message"
}

error_exit() {
    local message=$1
    log "ERROR" "$message"
    ROLLBACK_NEEDED=true
    rollback
    send_notification "DEPLOYMENT FAILED" "$message"
    exit 1
}

# Validation functions
validate_environment() {
    log "INFO" "Validating deployment environment"
    
    # Check if running as root or sudo
    if [[ $EUID -ne 0 ]]; then
        error_exit "This script must be run as root or with sudo"
    fi
    
    # Check required commands
    local required_commands=("git" "npm" "pm2")
    for cmd in "${required_commands[@]}"; do
        if ! command -v "$cmd" &> /dev/null; then
            error_exit "Required command not found: $cmd"
        fi
    done
    
    # Check disk space
    local available_space=$(df "$APP_DIR" | awk 'NR==2 {print $4}')
    if [[ $available_space -lt 1048576 ]]; then # 1GB in KB
        error_exit "Insufficient disk space. Need at least 1GB free"
    fi
    
    log "INFO" "Environment validation passed"
}

# Backup functions
create_backup() {
    log "INFO" "Creating backup of current deployment"
    
    mkdir -p "$BACKUP_DIR"
    local backup_file="${BACKUP_DIR}/backup_${DEPLOYMENT_ID}.tar.gz"
    
    if [[ -d "$APP_DIR" ]]; then
        if tar -czf "$backup_file" -C "$(dirname "$APP_DIR")" "$(basename "$APP_DIR")"; then
            log "INFO" "Backup created: $backup_file"
            echo "$backup_file"
        else
            error_exit "Failed to create backup"
        fi
    else
        log "INFO" "No existing deployment to backup"
        echo ""
    fi
}

rollback() {
    if [[ "$ROLLBACK_NEEDED" == true ]]; then
        log "WARN" "Performing rollback"
        
        # Find the most recent backup
        local latest_backup=$(find "$BACKUP_DIR" -name "backup_*.tar.gz" -type f -printf '%T@ %p\n' | sort -nr | head -1 | cut -d' ' -f2-)
        
        if [[ -n "$latest_backup" ]]; then
            log "INFO" "Rolling back to: $(basename "$latest_backup")"
            
            # Stop the application
            pm2 stop "$APP_NAME" || true
            pm2 delete "$APP_NAME" || true
            
            # Restore from backup
            rm -rf "$APP_DIR"
            mkdir -p "$(dirname "$APP_DIR")"
            tar -xzf "$latest_backup" -C "$(dirname "$APP_DIR")"
            
            # Restart the application
            cd "$APP_DIR"
            npm install
            pm2 start app.js --name "$APP_NAME"
            
            log "INFO" "Rollback completed"
        else
            log "ERROR" "No backup found for rollback"
        fi
    fi
}

# Deployment functions
clone_repository() {
    log "INFO" "Cloning repository"
    
    # Create temporary directory for new deployment
    local temp_dir=$(mktemp -d)
    
    if git clone "$GIT_REPO" "$temp_dir"; then
        log "INFO" "Repository cloned successfully"
        echo "$temp_dir"
    else
        error_exit "Failed to clone repository"
    fi
}

install_dependencies() {
    local deploy_dir=$1
    
    log "INFO" "Installing dependencies"
    cd "$deploy_dir"
    
    if [[ -f "package.json" ]]; then
        if npm install; then
            log "INFO" "Dependencies installed successfully"
        else
            error_exit "Failed to install dependencies"
        fi
    else
        log "WARN" "No package.json found, skipping dependency installation"
    fi
}

run_tests() {
    local deploy_dir=$1
    
    log "INFO" "Running tests"
    cd "$deploy_dir"
    
    if [[ -f "package.json" ]] && grep -q '"test"' package.json; then
        if npm test; then
            log "INFO" "Tests passed"
        else
            error_exit "Tests failed"
        fi
    else
        log "WARN" "No tests defined, skipping"
    fi
}

deploy_application() {
    local deploy_dir=$1
    
    log "INFO" "Deploying application"
    
    # Stop current application
    pm2 stop "$APP_NAME" || true
    pm2 delete "$APP_NAME" || true
    
    # Move new deployment to production
    rm -rf "$APP_DIR"
    mv "$deploy_dir" "$APP_DIR"
    
    # Start new application
    cd "$APP_DIR"
    pm2 start app.js --name "$APP_NAME"
    pm2 save
    
    log "INFO" "Application deployed successfully"
}

# Notification functions
send_notification() {
    local subject=$1
    local message=$2
    
    if command -v mail &> /dev/null; then
        echo "$message" | mail -s "[$APP_NAME] $subject" "$ADMIN_EMAIL"
    fi
}

# Main deployment function
perform_deployment() {
    log "INFO" "Starting deployment process"
    
    # Validate environment
    validate_environment
    
    # Create backup
    local backup_file=$(create_backup)
    
    # Clone and prepare new version
    local deploy_dir=$(clone_repository)
    
    # Install dependencies
    install_dependencies "$deploy_dir"
    
    # Run tests
    run_tests "$deploy_dir"
    
    # Deploy
    deploy_application "$deploy_dir"
    
    # Calculate deployment time
    local end_time=$(date +%s)
    local duration=$((end_time - START_TIME))
    
    log "INFO" "Deployment completed successfully in ${duration}s"
    
    # Send success notification
    send_notification "DEPLOYMENT SUCCESSFUL" "Deployment completed in ${duration}s. Backup: $(basename "$backup_file")"
}

# Usage function
usage() {
    cat << EOF
Usage: $SCRIPT_NAME [OPTIONS]

Automated deployment script for $APP_NAME.

OPTIONS:
    -h, --help     Show this help message
    --no-tests     Skip running tests
    --force        Force deployment even if tests fail

EXAMPLES:
    $SCRIPT_NAME
    $SCRIPT_NAME --no-tests

EOF
}

# Parse command line arguments
parse_args() {
    SKIP_TESTS=false
    FORCE_DEPLOY=false
    
    while [[ $# -gt 0 ]]; do
        case $1 in
            -h|--help)
                usage
                exit 0
                ;;
            --no-tests)
                SKIP_TESTS=true
                shift
                ;;
            --force)
                FORCE_DEPLOY=true
                shift
                ;;
            *)
                echo "Unknown option: $1"
                usage
                exit 1
                ;;
        esac
    done
}

# Main function
main() {
    parse_args "$@"
    setup_logging
    
    # Trap for cleanup
    trap 'ROLLBACK_NEEDED=true; rollback' ERR
    
    perform_deployment
}

# Run main function
main "$@"
```

**How to use:**
```bash
# Basic deployment
sudo ./deploy_app.sh

# Skip tests
sudo ./deploy_app.sh --no-tests

# Force deployment
sudo ./deploy_app.sh --force
```

**Features demonstrated:**
- Comprehensive error handling with rollback
- Logging and notifications
- Environment validation
- Backup and recovery
- External command integration
- Command-line argument parsing

-----

## Project 4: Log Analysis and Reporting Tool

**Combines:** Text processing, arrays, file operations, math, reporting

**Requirements:**
- Parse various log formats
- Generate statistics and reports
- Filter and search logs
- Export to different formats
- Handle large log files efficiently

```bash
#!/bin/bash
#
# Script Name: log_analyzer.sh
# Description: Advanced log analysis and reporting tool
# Author: Shell Scripting Course
# Date: 2024-01-15
#

set -euo pipefail

# Configuration
readonly SCRIPT_NAME="${0##*/}"
readonly MAX_FILE_SIZE="100M"
readonly DEFAULT_TIME_FORMAT="%Y-%m-%d %H:%M:%S"

# Global variables
declare -A STATS
declare -a ERRORS
declare -a WARNINGS
REPORT_FILE=""

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

# Utility functions
print_header() {
    echo -e "${BLUE}================================${NC}"
    echo -e "${BLUE}     Log Analysis Tool${NC}"
    echo -e "${BLUE}================================${NC}"
    echo
}

log() {
    local level=$1
    local message=$2
    
    case $level in
        "INFO")  echo -e "${GREEN}[$level]${NC} $message" ;;
        "WARN")  echo -e "${YELLOW}[$level]${NC} $message" ;;
        "ERROR") echo -e "${RED}[$level]${NC} $message" ;;
    esac
}

# Validation functions
validate_log_file() {
    local file=$1
    
    if [[ ! -f "$file" ]]; then
        log "ERROR" "Log file does not exist: $file"
        return 1
    fi
    
    if [[ ! -r "$file" ]]; then
        log "ERROR" "Log file not readable: $file"
        return 1
    fi
    
    # Check file size
    local file_size=$(stat -f%z "$file" 2>/dev/null || stat -c%s "$file" 2>/dev/null)
    if [[ $file_size -gt $(numfmt --from=iec "$MAX_FILE_SIZE") ]]; then
        log "WARN" "Large log file detected: $(numfmt --to=iec $file_size)"
    fi
    
    return 0
}

# Analysis functions
analyze_apache_logs() {
    local log_file=$1
    
    log "INFO" "Analyzing Apache access logs"
    
    # Total requests
    STATS["total_requests"]=$(wc -l < "$log_file")
    
    # Unique IP addresses
    STATS["unique_ips"]=$(awk '{print $1}' "$log_file" | sort | uniq | wc -l)
    
    # HTTP status codes
    while read -r code count; do
        STATS["status_$code"]=$count
    done < <(awk '{print $9}' "$log_file" | sort | uniq -c | sort -nr)
    
    # Top requesting IPs
    while read -r ip count; do
        STATS["ip_${ip//./_}"]=$count
    done < <(awk '{print $1}' "$log_file" | sort | uniq -c | sort -nr | head -10)
    
    # Requested URLs
    while read -r url count; do
        STATS["url_${url//[^a-zA-Z0-9]/_}"]=$count
    done < <(awk '{print $7}' "$log_file" | sort | uniq -c | sort -nr | head -10)
    
    # Errors (4xx and 5xx)
    STATS["error_requests"]=$(awk '$9 ~ /^4|^5/ {print $9}' "$log_file" | wc -l)
    
    # Bandwidth (approximate)
    STATS["bandwidth"]=$(awk '{sum += $10} END {print sum}' "$log_file")
}

analyze_syslog() {
    local log_file=$1
    
    log "INFO" "Analyzing system logs"
    
    # Total log entries
    STATS["total_entries"]=$(wc -l < "$log_file")
    
    # Error messages
    ERRORS=($(grep -i "error\|fail\|critical" "$log_file" | head -20))
    STATS["error_count"]=$(grep -c -i "error\|fail\|critical" "$log_file")
    
    # Warning messages
    WARNINGS=($(grep -i "warn\|warning" "$log_file" | head -20))
    STATS["warning_count"]=$(grep -c -i "warn\|warning" "$log_file")
    
    # Service mentions
    while read -r service count; do
        STATS["service_${service}"]=$count
    done < <(awk '{print $5}' "$log_file" | tr -d '[]:' | sort | uniq -c | sort -nr | head -10)
    
    # Time distribution
    STATS["entries_today"]=$(grep "$(date +%b\ %e)" "$log_file" | wc -l)
    STATS["entries_hour"]=$(grep "$(date +%b\ %e\ %H)" "$log_file" | wc -l)
}

analyze_application_logs() {
    local log_file=$1
    
    log "INFO" "Analyzing application logs"
    
    # Detect log format
    if head -5 "$log_file" | grep -q "^\["; then
        log "INFO" "Detected timestamped log format"
        
        # Extract timestamps and calculate time range
        local first_time=$(head -1 "$log_file" | grep -o '\[[0-9-]\+\s[0-9:]\+\]' | tr -d '[]' || echo "")
        local last_time=$(tail -1 "$log_file" | grep -o '\[[0-9-]\+\s[0-9:]\+\]' | tr -d '[]' || echo "")
        
        if [[ -n "$first_time" && -n "$last_time" ]]; then
            STATS["log_start"]=$first_time
            STATS["log_end"]=$last_time
        fi
    fi
    
    # Common patterns
    STATS["exceptions"]=$(grep -c -i "exception\|error\|fail" "$log_file")
    STATS["debug_messages"]=$(grep -c -i "debug" "$log_file")
    STATS["info_messages"]=$(grep -c -i "info" "$log_file")
    
    # Performance metrics (if present)
    if grep -q "response.time\|duration\|latency" "$log_file"; then
        STATS["has_performance_data"]=1
    fi
}

# Reporting functions
generate_report() {
    local log_file=$1
    local report_format=${2:-"text"}
    
    REPORT_FILE="log_analysis_$(basename "$log_file" .log)_$(date +%Y%m%d_%H%M%S)"
    
    case $report_format in
        "text") generate_text_report ;;
        "json") generate_json_report ;;
        "html") generate_html_report ;;
        *) 
            log "ERROR" "Unknown report format: $report_format"
            return 1
            ;;
    esac
    
    log "INFO" "Report generated: $REPORT_FILE"
}

generate_text_report() {
    {
        echo "========================================"
        echo "    LOG ANALYSIS REPORT"
        echo "========================================"
        echo "File: $(basename "$log_file")"
        echo "Analysis Date: $(date)"
        echo "----------------------------------------"
        echo
        
        echo "SUMMARY STATISTICS:"
        echo "==================="
        for key in "${!STATS[@]}"; do
            echo "$key: ${STATS[$key]}"
        done
        echo
        
        if [[ ${#ERRORS[@]} -gt 0 ]]; then
            echo "RECENT ERRORS:"
            echo "==============="
            printf '%s\n' "${ERRORS[@]}"
            echo
        fi
        
        if [[ ${#WARNINGS[@]} -gt 0 ]]; then
            echo "RECENT WARNINGS:"
            echo "================="
            printf '%s\n' "${WARNINGS[@]}"
            echo
        fi
        
    } > "$REPORT_FILE.txt"
}

generate_json_report() {
    {
        echo "{"
        echo '  "file": "'$(basename "$log_file")'",'
        echo '  "analysis_date": "'$(date)'",'
        echo '  "statistics": {'
        
        local first=true
        for key in "${!STATS[@]}"; do
            if [[ $first == true ]]; then
                first=false
            else
                echo ","
            fi
            echo -n '    "'$key'": "'${STATS[$key]}'"'
        done
        echo
        echo "  },"
        
        echo '  "errors": ['
        local first=true
        for error in "${ERRORS[@]}"; do
            if [[ $first == true ]]; then
                first=false
            else
                echo ","
            fi
            echo -n '    "'$error'"'
        done
        echo
        echo "  ],"
        
        echo '  "warnings": ['
        first=true
        for warning in "${WARNINGS[@]}"; do
            if [[ $first == true ]]; then
                first=false
            else
                echo ","
            fi
            echo -n '    "'$warning'"'
        done
        echo
        echo "  ]"
        echo "}"
        
    } > "$REPORT_FILE.json"
}

generate_html_report() {
    {
        echo "<!DOCTYPE html>"
        echo "<html><head><title>Log Analysis Report</title>"
        echo "<style>body{font-family:Arial,sans-serif;margin:20px}h1{color:#333}table{border-collapse:collapse;width:100%}th,td{border:1px solid #ddd;padding:8px;text-align:left}th{background-color:#f2f2f2}.error{color:red}.warning{color:orange}</style>"
        echo "</head><body>"
        echo "<h1>Log Analysis Report</h1>"
        echo "<p><strong>File:</strong> $(basename "$log_file")</p>"
        echo "<p><strong>Analysis Date:</strong> $(date)</p>"
        echo "<h2>Summary Statistics</h2>"
        echo "<table><tr><th>Metric</th><th>Value</th></tr>"
        
        for key in "${!STATS[@]}"; do
            echo "<tr><td>$key</td><td>${STATS[$key]}</td></tr>"
        done
        
        echo "</table>"
        
        if [[ ${#ERRORS[@]} -gt 0 ]]; then
            echo "<h2>Recent Errors</h2><ul>"
            for error in "${ERRORS[@]}"; do
                echo "<li class='error'>$error</li>"
            done
            echo "</ul>"
        fi
        
        if [[ ${#WARNINGS[@]} -gt 0 ]]; then
            echo "<h2>Recent Warnings</h2><ul>"
            for warning in "${WARNINGS[@]}"; do
                echo "<li class='warning'>$warning</li>"
            done
            echo "</ul>"
        fi
        
        echo "</body></html>"
        
    } > "$REPORT_FILE.html"
}

# Auto-detect log type
detect_log_type() {
    local log_file=$1
    
    # Check first few lines to determine log type
    local first_lines=$(head -10 "$log_file")
    
    if echo "$first_lines" | grep -q "GET\|POST\|HTTP"; then
        echo "apache"
    elif echo "$first_lines" | grep -q "kernel\|systemd\|syslog"; then
        echo "syslog"
    else
        echo "application"
    fi
}

# Main analysis function
analyze_log() {
    local log_file=$1
    local report_format=${2:-"text"}
    
    log "INFO" "Starting log analysis for: $(basename "$log_file")"
    
    # Validate input
    validate_log_file "$log_file" || return 1
    
    # Detect log type
    local log_type=$(detect_log_type "$log_file")
    log "INFO" "Detected log type: $log_type"
    
    # Clear previous stats
    STATS=()
    ERRORS=()
    WARNINGS=()
    
    # Analyze based on type
    case $log_type in
        "apache") analyze_apache_logs "$log_file" ;;
        "syslog") analyze_syslog "$log_file" ;;
        "application") analyze_application_logs "$log_file" ;;
        *) 
            log "WARN" "Unknown log type, using generic analysis"
            analyze_application_logs "$log_file"
            ;;
    esac
    
    # Generate report
    generate_report "$log_file" "$report_format"
    
    log "INFO" "Analysis completed successfully"
}

# Usage function
usage() {
    cat << EOF
Usage: $SCRIPT_NAME LOG_FILE [REPORT_FORMAT]

Advanced log analysis and reporting tool.

REPORT_FORMAT:
    text    Plain text report (default)
    json    JSON format report
    html    HTML format report

EXAMPLES:
    $SCRIPT_NAME /var/log/apache2/access.log
    $SCRIPT_NAME /var/log/syslog json
    $SCRIPT_NAME app.log html

EOF
}

# Main function
main() {
    if [[ $# -lt 1 ]]; then
        usage
        exit 1
    fi
    
    local log_file=$1
    local report_format=${2:-"text"}
    
    print_header
    analyze_log "$log_file" "$report_format"
}

# Run main function
main "$@"
```

**How to use:**
```bash
# Analyze Apache logs
./log_analyzer.sh /var/log/apache2/access.log

# Generate JSON report
./log_analyzer.sh /var/log/syslog json

# Create HTML report
./log_analyzer.sh app.log html
```

**Features demonstrated:**
- Log format auto-detection
- Multiple output formats
- Statistical analysis
- Error and warning extraction
- Large file handling
- Modular analysis functions

-----

## Summary

These projects demonstrate professional shell scripting:

✅ **Modular Design** - Functions, error handling, logging  
✅ **User Experience** - Input validation, help messages, progress indicators  
✅ **Robustness** - Backup/recovery, cross-platform compatibility  
✅ **Real-world Application** - Complete solutions, not just examples  

**Next Up:** Unit 10 covers best practices and advanced techniques to make your scripts production-ready.

**Exercises:**
1. Extend the backup system to support cloud storage (AWS S3, Google Cloud)
2. Add email notifications to the system monitor
3. Create a web interface for the deployment script status
4. Add machine learning-based anomaly detection to the log analyzer
