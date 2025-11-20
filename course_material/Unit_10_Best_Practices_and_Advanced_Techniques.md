# Unit 10: Best Practices and Advanced Techniques

**Learning Objectives:**
- Write production-ready, maintainable shell scripts
- Implement security best practices
- Optimize script performance
- Test and debug effectively
- Follow coding standards and conventions
- Handle edge cases and error scenarios
- Create reusable, modular code

**Estimated Time:** 3-4 hours

**Prerequisites:** Units 1-9

-----

## Introduction: From Good to Great

You've learned the syntax and built complete projects. Now we'll focus on the "soft skills" that separate amateur scripts from professional tools. These practices will make your scripts reliable, secure, and maintainable.

**The Three Pillars of Professional Scripting:**
1. **Reliability** - Scripts that work consistently
2. **Security** - Scripts that don't create vulnerabilities  
3. **Maintainability** - Scripts that can be understood and modified

-----

## 10.1 Script Structure and Organization

### The Professional Script Template

Every production script should follow this structure:

```bash
#!/bin/bash
#
# Script Name: script_name.sh
# Description: What the script does
# Author: Your Name
# Date: YYYY-MM-DD
# Version: 1.0.0
# License: MIT/GPL/etc.
#
# Dependencies: list of required commands/tools
# Usage: ./script_name.sh [options] [arguments]
#
# Examples:
#   ./script_name.sh --help
#   ./script_name.sh --verbose input.txt
#
# Changelog:
#   v1.0.0 - Initial release
#   v1.1.0 - Added feature X
#

# ⚡ SAFETY OPTIONS - Make script fail fast and predictably
set -euo pipefail
# │││
# ││└─ pipefail: Exit if ANY command in a pipeline fails
# ││             Example: "cat missing.txt | grep foo" fails even if grep succeeds
# ││             ⚠️ Without this, only the last command's exit code is checked
# ││
# │└── -u: Treat unset variables as errors
# │         Example: echo "$UNDEFINED_VAR" → script exits with error
# │         ⚠️ Without this, it silently prints empty string (very dangerous!)
# │
# └─── -e: Exit immediately if any command fails (exit code != 0)
#          Example: cp /nonexistent /tmp → script stops here
#          ⚠️ Without this, script continues after errors (creates cascading failures)
#
# 💡 WHY THIS MATTERS:
#    These options turn bash into a "fail-fast" system like Python or Java
#    Without them, bash silently continues after errors, leading to:
#    - Data corruption (writing to wrong files)
#    - Security issues (using empty variables as file paths)
#    - Hard-to-debug failures hours later
#
# Example WITHOUT set -euo pipefail:
#   USER=""  # Oops, forgot to set this
#   rm -rf /home/$USER/*  # Becomes: rm -rf /home/*  ⚠️ DISASTER!
#
# Example WITH set -euo pipefail:
#   USER=""  # Script exits here: "USER: unbound variable"
#   rm -rf /home/$USER/*  # Never executes

# =============================================================================
# CONFIGURATION
# =============================================================================

# Constants (readonly - cannot be modified after assignment)
readonly SCRIPT_NAME="${0##*/}"  # Extract filename from path: /path/to/script.sh → script.sh
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"  # Absolute path to script directory
readonly SCRIPT_VERSION="1.0.0"  # Semantic versioning

# Default configuration (can be overridden by environment variables)
# Pattern: VAR="${VAR:-default}" means "use VAR if set, otherwise use default"
LOG_LEVEL="${LOG_LEVEL:-INFO}"  # Can override: LOG_LEVEL=DEBUG ./script.sh
LOG_FILE="${LOG_FILE:-/var/log/${SCRIPT_NAME%.sh}.log}"  # ${VAR%.sh} removes .sh extension
BACKUP_DIR="${BACKUP_DIR:-/var/backups}"

# Color codes
readonly RED='\033[0;31m'
readonly GREEN='\033[0;32m'
readonly YELLOW='\033[1;33m'
readonly BLUE='\033[0;34m'
readonly NC='\033[0m'  # No Color

# =============================================================================
# FUNCTIONS
# =============================================================================

# Logging functions
setup_logging() {
    # Create log directory if it doesn't exist
    mkdir -p "$(dirname "$LOG_FILE")"
    
    # Rotate log if it gets too big (>10MB)
    if [[ -f "$LOG_FILE" ]] && [[ $(stat -f%z "$LOG_FILE" 2>/dev/null || stat -c%s "$LOG_FILE" 2>/dev/null) -gt 10485760 ]]; then
        mv "$LOG_FILE" "${LOG_FILE}.old"
    fi
    
    log "INFO" "Script started: $SCRIPT_NAME v$SCRIPT_VERSION"
}

log() {
    local level=$1
    local message=$2
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    
    # Log to file
    echo "[$timestamp] [$level] $message" >> "$LOG_FILE"
    
    # Log to console based on log level
    case $LOG_LEVEL in
        "DEBUG") 
            echo -e "${BLUE}[$level]${NC} $message" >&2 ;;
        "INFO") 
            [[ $level == "INFO" ]] && echo -e "${GREEN}[$level]${NC} $message" >&2 ;;
        "WARN") 
            [[ $level =~ ^(WARN|ERROR)$ ]] && echo -e "${YELLOW}[$level]${NC} $message" >&2 ;;
        "ERROR") 
            [[ $level == "ERROR" ]] && echo -e "${RED}[$level]${NC} $message" >&2 ;;
    esac
}

# Error handling
error_exit() {
    local message=$1
    local code=${2:-1}
    
    log "ERROR" "$message"
    echo -e "${RED}ERROR:${NC} $message" >&2
    
    # Call cleanup if it exists
    if declare -f cleanup > /dev/null; then
        cleanup
    fi
    
    exit "$code"
}

# Validation functions
validate_dependencies() {
    local deps=("$@")
    local missing=()
    
    for dep in "${deps[@]}"; do
        if ! command -v "$dep" &> /dev/null; then
            missing+=("$dep")
        fi
    done
    
    if [[ ${#missing[@]} -gt 0 ]]; then
        error_exit "Missing required dependencies: ${missing[*]}"
    fi
}

validate_file() {
    local file=$1
    local permissions=${2:-"r"}
    
    if [[ ! -f "$file" ]]; then
        error_exit "File does not exist: $file"
    fi
    
    case $permissions in
        "r") [[ ! -r "$file" ]] && error_exit "File not readable: $file" ;;
        "w") [[ ! -w "$file" ]] && error_exit "File not writable: $file" ;;
        "x") [[ ! -x "$file" ]] && error_exit "File not executable: $file" ;;
        "rw") [[ ! -r "$file" || ! -w "$file" ]] && error_exit "File not readable/writable: $file" ;;
    esac
}

# Utility functions
show_version() {
    echo "$SCRIPT_NAME version $SCRIPT_VERSION"
}

show_help() {
    cat << EOF
$SCRIPT_NAME - Description of what the script does

USAGE:
    $SCRIPT_NAME [OPTIONS] [ARGUMENTS]

OPTIONS:
    -h, --help          Show this help message
    -v, --version       Show version information
    -d, --debug         Enable debug logging
    -q, --quiet         Suppress non-error output
    --log-file FILE     Specify log file location
    --dry-run           Show what would be done without doing it

EXAMPLES:
    $SCRIPT_NAME --help
    $SCRIPT_NAME --debug input.txt
    $SCRIPT_NAME --dry-run --log-file /tmp/my.log

DEPENDENCIES:
    curl, jq, awk

For more information, see the README.md file.
EOF
}

# Argument parsing
parse_args() {
    DRY_RUN=false
    
    while [[ $# -gt 0 ]]; do
        case $1 in
            -h|--help)
                show_help
                exit 0
                ;;
            -v|--version)
                show_version
                exit 0
                ;;
            -d|--debug)
                LOG_LEVEL="DEBUG"
                shift
                ;;
            -q|--quiet)
                LOG_LEVEL="ERROR"
                shift
                ;;
            --log-file)
                LOG_FILE="$2"
                shift 2
                ;;
            --dry-run)
                DRY_RUN=true
                shift
                ;;
            -*)
                error_exit "Unknown option: $1"
                ;;
            *)
                # Positional arguments
                POSITIONAL_ARGS+=("$1")
                shift
                ;;
        esac
    done
    
    # Restore positional arguments
    set -- "${POSITIONAL_ARGS[@]}"
}

# Cleanup function (override in main script)
cleanup() {
    log "INFO" "Performing cleanup"
    # Add cleanup logic here
}

# =============================================================================
# MAIN LOGIC
# =============================================================================

main() {
    # Parse command line arguments
    parse_args "$@"
    
    # Setup
    setup_logging
    
    # Validate dependencies
    validate_dependencies "curl" "jq"
    
    # Validate arguments
    if [[ $# -eq 0 ]]; then
        error_exit "No input file specified. Use --help for usage information."
    fi
    
    validate_file "$1" "r"
    
    # Main logic here
    local input_file=$1
    
    log "INFO" "Processing file: $input_file"
    
    if [[ "$DRY_RUN" == true ]]; then
        log "INFO" "[DRY RUN] Would process $input_file"
        return 0
    fi
    
    # Your script logic here
    process_file "$input_file"
    
    log "INFO" "Script completed successfully"
}

# Trap signals
trap 'error_exit "Script interrupted by user"' INT TERM
trap 'cleanup' EXIT

# Run main function
main "$@"
```

### Function Organization Principles

1. **Single Responsibility** - Each function does one thing
2. **Descriptive Names** - `process_user_data()` not `do_stuff()`
3. **Consistent Parameters** - Input first, then options
4. **Return Values** - Use exit codes for status, echo for data
5. **Documentation** - Comment complex logic

**Good Example:**
```bash
# Function to validate email format
# Returns: 0 if valid, 1 if invalid
validate_email() {
    local email=$1
    
    # Basic regex check
    if [[ $email =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
        return 0
    else
        return 1
    fi
}

# Usage
if validate_email "$user_email"; then
    echo "Email is valid"
else
    echo "Email is invalid"
fi
```

-----

## 10.2 Security Best Practices

### Input Validation and Sanitization

**Never trust user input!**

```bash
#!/bin/bash

# UNSAFE - Command injection vulnerability
# echo "User input: $user_input" | mail $user_email

# SAFE - Validate and sanitize
validate_email() {
    local email=$1
    
    # Check format
    if [[ ! $email =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
        return 1
    fi
    
    # Check length
    if [[ ${#email} -gt 254 ]]; then
        return 1
    fi
    
    return 0
}

send_secure_email() {
    local recipient=$1
    local subject=$2
    local message=$3
    
    # Validate recipient
    if ! validate_email "$recipient"; then
        error_exit "Invalid email address: $recipient"
    fi
    
    # Sanitize subject (remove dangerous characters)
    subject=$(echo "$subject" | tr -d '\n\r')
    
    # Use safe mail command
    echo "$message" | mail -s "$subject" -r "noreply@yourdomain.com" "$recipient"
}
```

### Secure File Operations

```bash
#!/bin/bash

# Create temporary files securely
create_temp_file() {
    local template=${1:-"temp_XXXXXX"}
    
    # Use mktemp for secure temporary files
    if ! mktemp "$template"; then
        error_exit "Failed to create temporary file"
    fi
}

# Secure file permissions
set_secure_permissions() {
    local file=$1
    local owner=${2:-"root"}
    local group=${3:-"root"}
    local perms=${4:-"600"}
    
    chown "$owner:$group" "$file"
    chmod "$perms" "$file"
}

# Example usage
temp_file=$(create_temp_file "my_script_XXXXXX")
echo "Sensitive data" > "$temp_file"
set_secure_permissions "$temp_file"

# Cleanup
trap 'rm -f "$temp_file"' EXIT
```

### Password and Secret Handling

```bash
#!/bin/bash

# NEVER store passwords in scripts
# BAD: PASSWORD="secret123"

# Use environment variables or secure prompts
get_password() {
    local prompt=${1:-"Enter password:"}
    
    # Disable echo for password input
    stty -echo
    echo -n "$prompt " >&2
    read -r password
    echo >&2
    stty echo
    
    echo "$password"
}

# Use password securely (don't store in variables longer than needed)
setup_database() {
    local db_password
    
    # Get password securely
    db_password=$(get_password "Database password:")
    
    # Use immediately and clear from memory
    mysql -u admin -p"$db_password" -e "CREATE DATABASE myapp;"
    
    # Clear password from memory
    unset db_password
}

# Use keyring or secure storage for persistent secrets
store_secret() {
    local key=$1
    local value=$2
    
    # Use system keyring if available
    if command -v secret-tool &> /dev/null; then
        echo -n "$value" | secret-tool store --label="$key" application "$SCRIPT_NAME"
    else
        # Fallback to encrypted file
        echo "$value" | openssl enc -aes-256-cbc -salt -out "/etc/${SCRIPT_NAME}/${key}.enc" -k "$(get_master_password)"
    fi
}
```

### Privilege Management

```bash
#!/bin/bash

# Check if running with appropriate privileges
check_privileges() {
    local required_user=${1:-"root"}
    
    if [[ $EUID -ne 0 && "$required_user" == "root" ]]; then
        error_exit "This script must be run as root (use sudo)"
    fi
    
    if [[ $(whoami) != "$required_user" && "$required_user" != "root" ]]; then
        error_exit "This script must be run as $required_user"
    fi
}

# Drop privileges when no longer needed
drop_privileges() {
    local target_user=${1:-"nobody"}
    
    if [[ $EUID -eq 0 ]]; then
        log "INFO" "Dropping privileges to $target_user"
        
        # Change to non-privileged user
        if ! su -c "$0" "$target_user"; then
            error_exit "Failed to drop privileges"
        fi
    fi
}

# Run specific commands with elevated privileges
run_with_sudo() {
    local command=$1
    
    if [[ $EUID -eq 0 ]]; then
        # Already root, run directly
        eval "$command"
    else
        # Use sudo
        if ! sudo --non-interactive "$command"; then
            error_exit "Command failed (may need sudo access): $command"
        fi
    fi
}
```

-----

## 10.3 Performance Optimization

### Efficient Text Processing

```bash
#!/bin/bash

# INEFFICIENT - Multiple passes through file
# lines=$(wc -l < file.txt)
# words=$(wc -w < file.txt)
# chars=$(wc -c < file.txt)

# EFFICIENT - Single pass with awk
analyze_file() {
    local file=$1
    
    awk '
    BEGIN { lines=0; words=0; chars=0 }
    {
        lines++
        words += NF
        chars += length($0) + 1  # +1 for newline
    }
    END {
        print "Lines:", lines
        print "Words:", words
        print "Characters:", chars
    }
    ' "$file"
}

# Use sed for simple substitutions (faster than multiple grep calls)
# INEFFICIENT
# content=$(cat file.txt)
# content=$(echo "$content" | sed 's/old/new/g')
# content=$(echo "$content" | sed 's/foo/bar/g')

# EFFICIENT
process_content() {
    local file=$1
    
    sed -e 's/old/new/g' \
        -e 's/foo/bar/g' \
        "$file"
}
```

### Memory-Efficient Processing

```bash
#!/bin/bash

# Process large files line by line (not all at once)
process_large_file() {
    local file=$1
    
    while IFS= read -r line || [[ -n $line ]]; do
        # Process one line at a time
        process_line "$line"
    done < "$file"
}

# Use temporary files for large datasets instead of variables
sort_large_dataset() {
    local input_file=$1
    local output_file=$2
    
    # Use sort command instead of loading into array
    sort "$input_file" > "$output_file"
}

# Parallel processing for CPU-intensive tasks
parallel_processing() {
    local items=("$@")
    local max_jobs=4
    
    for item in "${items[@]}"; do
        # Limit concurrent jobs
        while [[ $(jobs -r | wc -l) -ge $max_jobs ]]; do
            sleep 0.1
        done
        
        process_item "$item" &
    done
    
    # Wait for all background jobs to complete
    wait
}
```

### Caching and Memoization

```bash
#!/bin/bash

# Simple caching with associative arrays
declare -A CACHE

cached_command() {
    local key=$1
    local command=$2
    
    # Check cache first
    if [[ -n "${CACHE[$key]:-}" ]]; then
        echo "${CACHE[$key]}"
        return 0
    fi
    
    # Execute command and cache result
    local result
    result=$(eval "$command")
    CACHE[$key]="$result"
    
    echo "$result"
}

# Usage
get_hostname() {
    cached_command "hostname" "hostname"
}

get_os_version() {
    cached_command "os_version" "uname -s -r"
}

# File-based caching for expensive operations
cache_file_result() {
    local cache_file=$1
    local ttl_seconds=${2:-3600}  # 1 hour default
    local command=$3
    
    # Check if cache is still valid
    if [[ -f "$cache_file" ]] && [[ $(($(date +%s) - $(stat -f%M "$cache_file" 2>/dev/null || stat -c%Y "$cache_file" 2>/dev/null))) -lt $ttl_seconds ]]; then
        cat "$cache_file"
        return 0
    fi
    
    # Generate new result
    local result
    result=$(eval "$command")
    
    # Cache result
    echo "$result" > "$cache_file"
    
    echo "$result"
}
```

-----

## 10.4 Testing and Debugging

### Unit Testing Framework

```bash
#!/bin/bash
#
# Script Name: test_framework.sh
# Description: Simple testing framework for shell scripts
#

# Test framework functions
TESTS_RUN=0
TESTS_PASSED=0
TESTS_FAILED=0

assert_equals() {
    local expected=$1
    local actual=$2
    local message=${3:-"Assertion failed"}
    
    ((TESTS_RUN++))
    
    if [[ "$expected" == "$actual" ]]; then
        ((TESTS_PASSED++))
        echo -e "${GREEN}✓ PASS${NC} $message"
    else
        ((TESTS_FAILED++))
        echo -e "${RED}✗ FAIL${NC} $message"
        echo "  Expected: '$expected'"
        echo "  Actual:   '$actual'"
    fi
}

assert_true() {
    local condition=$1
    local message=${2:-"Condition should be true"}
    
    ((TESTS_RUN++))
    
    if eval "$condition"; then
        ((TESTS_PASSED++))
        echo -e "${GREEN}✓ PASS${NC} $message"
    else
        ((TESTS_FAILED++))
        echo -e "${RED}✗ FAIL${NC} $message"
    fi
}

assert_file_exists() {
    local file=$1
    local message=${2:-"File should exist: $file"}
    
    assert_true "[[ -f '$file' ]]" "$message"
}

run_tests() {
    local test_function=$1
    
    echo "Running tests for $test_function..."
    echo
    
    # Run the test function
    $test_function
    
    echo
    echo "Results: $TESTS_PASSED passed, $TESTS_FAILED failed, $TESTS_RUN total"
    
    if [[ $TESTS_FAILED -gt 0 ]]; then
        return 1
    else
        return 0
    fi
}

# Example test suite
test_validate_email() {
    # Source the function to test
    source email_validator.sh
    
    # Test cases
    assert_true "validate_email 'user@example.com'" "Valid email should pass"
    assert_true "validate_email 'test.email+tag@domain.co.uk'" "Complex valid email should pass"
    
    assert_equals 1 "$(validate_email 'invalid-email' ; echo $?)" "Invalid email should fail"
    assert_equals 1 "$(validate_email '@example.com' ; echo $?)" "Email without local part should fail"
    assert_equals 1 "$(validate_email 'user@' ; echo $?)" "Email without domain should fail"
}

# Usage
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    run_tests "test_validate_email"
fi
```

### Debugging Techniques

```bash
#!/bin/bash

# Enable debugging
set -x  # Print each command before execution
set -v  # Print shell input lines as they are read

# Or enable conditionally
DEBUG=${DEBUG:-false}
if [[ "$DEBUG" == true ]]; then
    set -x
fi

# Debug function with call stack
debug() {
    local message=$1
    
    if [[ "$DEBUG" == true ]]; then
        echo "DEBUG: $message" >&2
        echo "  Called from: ${FUNCNAME[1]} at line ${BASH_LINENO[0]}" >&2
    fi
}

# Trace function calls
trace_function() {
    local func_name=$1
    
    debug "Entering $func_name"
    
    # Call original function
    "$@"
    
    local exit_code=$?
    debug "Exiting $func_name with code $exit_code"
    
    return $exit_code
}

# Profile script performance
profile_script() {
    local start_time=$(date +%s.%N)
    
    # Run the script
    "$@"
    
    local end_time=$(date +%s.%N)
    local duration=$(echo "$end_time - $start_time" | bc)
    
    echo "Script execution time: ${duration}s" >&2
}

# Memory usage tracking
track_memory() {
    local pid=$$
    
    while kill -0 $pid 2>/dev/null; do
        ps -o pid,ppid,pmem,rss,vsz,comm -p $pid
        sleep 1
    done
}

# Usage examples
debug "Starting script execution"

if [[ "$PROFILE" == true ]]; then
    profile_script main "$@"
else
    main "$@"
fi
```

### Integration Testing

```bash
#!/bin/bash
#
# Script Name: integration_test.sh
# Description: Integration tests for deployment script
#

setup_test_environment() {
    # Create temporary directories
    TEST_DIR=$(mktemp -d)
    TEST_APP_DIR="$TEST_DIR/app"
    TEST_BACKUP_DIR="$TEST_DIR/backups"
    
    # Create mock application
    mkdir -p "$TEST_APP_DIR"
    echo 'console.log("Hello World");' > "$TEST_APP_DIR/app.js"
    echo '{"name": "test-app", "version": "1.0.0"}' > "$TEST_APP_DIR/package.json"
    
    # Set environment variables
    export APP_DIR="$TEST_APP_DIR"
    export BACKUP_DIR="$TEST_BACKUP_DIR"
}

cleanup_test_environment() {
    rm -rf "$TEST_DIR"
}

test_deployment_success() {
    setup_test_environment
    
    # Run deployment
    if ./deploy_app.sh --dry-run; then
        assert_true "[[ -d '$TEST_BACKUP_DIR' ]]" "Backup directory should be created"
        assert_file_exists "$TEST_APP_DIR/app.js" "Application should exist"
        
        echo "✓ Deployment test passed"
    else
        echo "✗ Deployment test failed"
        return 1
    fi
    
    cleanup_test_environment
}

test_deployment_failure() {
    setup_test_environment
    
    # Remove package.json to cause failure
    rm "$TEST_APP_DIR/package.json"
    
    # Run deployment (should fail)
    if ./deploy_app.sh --dry-run 2>/dev/null; then
        echo "✗ Deployment should have failed"
        cleanup_test_environment
        return 1
    else
        echo "✓ Deployment correctly failed on missing package.json"
    fi
    
    cleanup_test_environment
}

run_integration_tests() {
    echo "Running integration tests..."
    
    test_deployment_success
    test_deployment_failure
    
    echo "Integration tests completed"
}
```

-----

## 10.5 Code Quality and Standards

### Linting and Static Analysis

```bash
#!/bin/bash
#
# Script Name: lint_script.sh
# Description: Lint and validate shell scripts
#

lint_script() {
    local script_file=$1
    
    echo "Linting $script_file..."
    
    # Check with shellcheck if available
    if command -v shellcheck &> /dev/null; then
        if shellcheck "$script_file"; then
            echo "✓ ShellCheck passed"
        else
            echo "✗ ShellCheck found issues"
            return 1
        fi
    else
        echo "! ShellCheck not available, install for better linting"
    fi
    
    # Basic syntax check
    if bash -n "$script_file"; then
        echo "✓ Syntax check passed"
    else
        echo "✗ Syntax errors found"
        return 1
    fi
    
    # Check for common issues
    local issues=0
    
    # Check for unset variables
    if grep -n '\$[a-zA-Z_][a-zA-Z0-9_]*' "$script_file" | grep -v '^\s*#'; then
        echo "! Found variable references that might not be quoted"
    fi
    
    # Check for proper error handling
    if ! grep -q 'set -e' "$script_file"; then
        echo "! Consider adding 'set -e' for better error handling"
        ((issues++))
    fi
    
    # Check for functions without documentation
    while IFS= read -r line; do
        local func_name=$(echo "$line" | sed 's/^\([a-zA-Z_][a-zA-Z0-9_]*\)().*/\1/')
        if [[ -n "$func_name" ]] && ! grep -B1 "^$func_name()" "$script_file" | grep -q '^#'; then
            echo "! Function $func_name lacks documentation"
            ((issues++))
        fi
    done < <(grep '^[a-zA-Z_][a-zA-Z0-9_]*()' "$script_file")
    
    if [[ $issues -gt 0 ]]; then
        echo "! Found $issues potential issues"
        return 1
    else
        echo "✓ All checks passed"
        return 0
    fi
}

# Usage
if [[ $# -eq 0 ]]; then
    echo "Usage: $0 <script_file>"
    exit 1
fi

lint_script "$1"
```

### Code Formatting Standards

```bash
#!/bin/bash
#
# Script Name: format_script.sh
# Description: Format shell scripts according to standards
#

format_script() {
    local script_file=$1
    
    echo "Formatting $script_file..."
    
    # Create backup
    cp "$script_file" "${script_file}.bak"
    
    # Apply formatting rules
    {
        # Remove trailing whitespace
        sed 's/[[:space:]]*$//' "$script_file" |
        
        # Fix indentation (basic)
        awk '
        BEGIN { indent = 0 }
        /{$/ { print; indent++; next }
        /}$/ { indent--; print; next }
        { for (i = 0; i < indent; i++) printf "    "; print }
        ' |
        
        # Add blank lines around functions
        sed '/^[a-zA-Z_][a-zA-Z0-9_]*() {$/,/}$/ {
            /^[a-zA-Z_][a-zA-Z0-9_]*() {$/ i \
            
            /}$/ a \
            
        }'
        
    } > "${script_file}.formatted"
    
    mv "${script_file}.formatted" "$script_file"
    
    echo "✓ Script formatted"
}

# Check if shfmt is available for better formatting
format_with_shfmt() {
    local script_file=$1
    
    if command -v shfmt &> /dev/null; then
        shfmt -i 4 -w "$script_file"
        echo "✓ Script formatted with shfmt"
    else
        format_script "$script_file"
    fi
}

# Usage
if [[ $# -eq 0 ]]; then
    echo "Usage: $0 <script_file>"
    exit 1
fi

format_with_shfmt "$1"
```

### Documentation Standards

```bash
#!/bin/bash
#
# Script Name: generate_docs.sh
# Description: Generate documentation from script comments
#

generate_docs() {
    local script_file=$1
    local output_file="${script_file%.*}.md"
    
    echo "Generating documentation for $script_file..."
    
    {
        # Extract script header
        sed -n '/^#!/,/^$/p' "$script_file" | grep '^#' | sed 's/^#//'
        echo
        
        # Extract function documentation
        echo "## Functions"
        echo
        
        local in_function=false
        local func_name=""
        
        while IFS= read -r line; do
            if [[ $line =~ ^[a-zA-Z_][a-zA-Z0-9_]*\(\) ]]; then
                # Function definition
                func_name=$(echo "$line" | sed 's/().*//')
                in_function=true
                echo "### \`$func_name()\`"
                echo
            elif [[ $in_function == true && $line =~ ^[[:space:]]*# ]]; then
                # Function comment
                echo "$line" | sed 's/^[[:space:]]*#//'
            elif [[ $in_function == true && $line =~ ^[[:space:]]*$ ]]; then
                # End of function comments
                echo
                in_function=false
            fi
        done < "$script_file"
        
    } > "$output_file"
    
    echo "✓ Documentation generated: $output_file"
}

# Usage
if [[ $# -eq 0 ]]; then
    echo "Usage: $0 <script_file>"
    exit 1
fi

generate_docs "$1"
```

-----

## 10.6 Advanced Error Handling and Recovery

### Comprehensive Error Handling

```bash
#!/bin/bash

# Error codes
readonly ERR_INVALID_ARGUMENT=1
readonly ERR_FILE_NOT_FOUND=2
readonly ERR_PERMISSION_DENIED=3
readonly ERR_NETWORK_ERROR=4
readonly ERR_DEPENDENCY_MISSING=5

# Error messages
declare -A ERROR_MESSAGES=(
    [$ERR_INVALID_ARGUMENT]="Invalid argument provided"
    [$ERR_FILE_NOT_FOUND]="File not found"
    [$ERR_PERMISSION_DENIED]="Permission denied"
    [$ERR_NETWORK_ERROR]="Network error occurred"
    [$ERR_DEPENDENCY_MISSING]="Required dependency missing"
)

# Enhanced error exit with cleanup
error_exit() {
    local code=$1
    local message=${2:-${ERROR_MESSAGES[$code]:-"Unknown error"}}
    local show_usage=${3:-false}
    
    # Log error
    log "ERROR" "[$code] $message"
    
    # Show error to user
    echo -e "${RED}ERROR [$code]:${NC} $message" >&2
    
    # Show usage if requested
    if [[ "$show_usage" == true ]]; then
        echo >&2
        show_help >&2
    fi
    
    # Perform cleanup
    cleanup_on_error "$code"
    
    exit "$code"
}

# Context-aware cleanup
cleanup_on_error() {
    local error_code=$1
    
    case $error_code in
        $ERR_NETWORK_ERROR)
            # Clean up network connections
            kill_network_processes
            ;;
        $ERR_FILE_NOT_FOUND)
            # Clean up temporary files
            cleanup_temp_files
            ;;
        *)
            # General cleanup
            cleanup
            ;;
    esac
}

# Retry mechanism with exponential backoff
retry_command() {
    local command=$1
    local max_attempts=${2:-3}
    local base_delay=${3:-1}
    local attempt=1
    
    while [[ $attempt -le $max_attempts ]]; do
        log "INFO" "Attempt $attempt/$max_attempts: $command"
        
        if eval "$command"; then
            log "INFO" "Command succeeded on attempt $attempt"
            return 0
        fi
        
        if [[ $attempt -eq $max_attempts ]]; then
            log "ERROR" "Command failed after $max_attempts attempts"
            return 1
        fi
        
        local delay=$((base_delay * (2 ** (attempt - 1))))
        log "WARN" "Command failed, retrying in ${delay}s..."
        sleep "$delay"
        
        ((attempt++))
    done
}

# Circuit breaker pattern
declare -A CIRCUIT_BREAKERS

check_circuit_breaker() {
    local service=$1
    local failure_threshold=${2:-5}
    local reset_timeout=${3:-60}
    
    local now=$(date +%s)
    
    # Initialize if not exists
    if [[ -z "${CIRCUIT_BREAKERS[$service]:-}" ]]; then
        CIRCUIT_BREAKERS[$service]="open:0:$now"
    fi
    
    local state=$(echo "${CIRCUIT_BREAKERS[$service]}" | cut -d: -f1)
    local failures=$(echo "${CIRCUIT_BREAKERS[$service]}" | cut -d: -f2)
    local last_failure=$(echo "${CIRCUIT_BREAKERS[$service]}" | cut -d: -f3)
    
    case $state in
        "open")
            # Allow request
            return 0
            ;;
        "half-open")
            # Test request
            if [[ $((now - last_failure)) -gt $reset_timeout ]]; then
                CIRCUIT_BREAKERS[$service]="half-open:0:$now"
                return 0
            else
                return 1
            fi
            ;;
        "closed")
            # Circuit is broken
            if [[ $((now - last_failure)) -gt $reset_timeout ]]; then
                CIRCUIT_BREAKERS[$service]="half-open:0:$now"
                return 0
            else
                return 1
            fi
            ;;
    esac
}

record_circuit_breaker_failure() {
    local service=$1
    local failure_threshold=${2:-5}
    
    local now=$(date +%s)
    local state=$(echo "${CIRCUIT_BREAKERS[$service]}" | cut -d: -f1)
    local failures=$(echo "${CIRCUIT_BREAKERS[$service]}" | cut -d: -f2)
    
    ((failures++))
    
    if [[ $failures -ge $failure_threshold ]]; then
        CIRCUIT_BREAKERS[$service]="closed:$failures:$now"
        log "WARN" "Circuit breaker opened for $service after $failures failures"
    else
        CIRCUIT_BREAKERS[$service]="open:$failures:$now"
    fi
}

record_circuit_breaker_success() {
    local service=$1
    
    CIRCUIT_BREAKERS[$service]="open:0:$(date +%s)"
    log "INFO" "Circuit breaker reset for $service"
}
```

### Graceful Degradation

```bash
#!/bin/bash

# Feature flags for graceful degradation
FEATURE_ADVANCED_LOGGING=${FEATURE_ADVANCED_LOGGING:-true}
FEATURE_NETWORK_CHECKS=${FEATURE_NETWORK_CHECKS:-true}
FEATURE_PERFORMANCE_MONITORING=${FEATURE_PERFORMANCE_MONITORING:-true}

# Degraded logging function
log() {
    local level=$1
    local message=$2
    
    if [[ "$FEATURE_ADVANCED_LOGGING" == true ]]; then
        # Full logging
        local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
        echo "[$timestamp] [$level] $message" >> "$LOG_FILE"
        
        case $level in
            "ERROR") echo -e "${RED}[$level]${NC} $message" >&2 ;;
            "WARN")  echo -e "${YELLOW}[$level]${NC} $message" >&2 ;;
            "INFO")  echo -e "${GREEN}[$level]${NC} $message" >&2 ;;
        esac
    else
        # Basic logging only
        echo "[$level] $message" >&2
    fi
}

# Network check with fallback
check_connectivity() {
    if [[ "$FEATURE_NETWORK_CHECKS" == true ]]; then
        if ping -c 1 -W 2 8.8.8.8 &>/dev/null; then
            return 0
        else
            log "WARN" "Network connectivity check failed, continuing in offline mode"
            FEATURE_NETWORK_CHECKS=false
            return 1
        fi
    fi
    
    # Degraded mode - skip network checks
    return 0
}

# Performance monitoring with fallback
start_performance_monitoring() {
    if [[ "$FEATURE_PERFORMANCE_MONITORING" == true ]]; then
        # Start background monitoring
        monitor_performance &
        MONITOR_PID=$!
        log "INFO" "Performance monitoring started (PID: $MONITOR_PID)"
    else
        log "INFO" "Performance monitoring disabled"
    fi
}

stop_performance_monitoring() {
    if [[ "$FEATURE_PERFORMANCE_MONITORING" == true && -n "${MONITOR_PID:-}" ]]; then
        kill "$MONITOR_PID" 2>/dev/null || true
        log "INFO" "Performance monitoring stopped"
    fi
}

# Auto-detect capabilities and degrade gracefully
detect_capabilities() {
    # Check for optional dependencies
    if ! command -v jq &> /dev/null; then
        log "WARN" "jq not found, disabling advanced JSON processing"
        FEATURE_ADVANCED_LOGGING=false
    fi
    
    if ! command -v curl &> /dev/null; then
        log "WARN" "curl not found, disabling network checks"
        FEATURE_NETWORK_CHECKS=false
    fi
    
    # Check system resources
    local available_memory=$(free -m 2>/dev/null | awk 'NR==2{print $7}' || echo "512")
    if [[ $available_memory -lt 100 ]]; then
        log "WARN" "Low memory detected, disabling performance monitoring"
        FEATURE_PERFORMANCE_MONITORING=false
    fi
}
```

-----

## Summary

Professional shell scripting requires attention to:

✅ **Structure** - Consistent organization and documentation  
✅ **Security** - Input validation, secure practices, privilege management  
✅ **Performance** - Efficient algorithms, caching, memory management  
✅ **Testing** - Unit tests, integration tests, debugging tools  
✅ **Quality** - Linting, formatting, standards compliance  
✅ **Reliability** - Error handling, recovery, graceful degradation  

**Final Project:** Create a production-ready script that incorporates all these best practices. Start with the template provided and build something useful for your workflow.

**Congratulations!** You've completed the comprehensive Shell Scripting Mastery course. You now have the knowledge and skills to write professional, maintainable shell scripts that can automate complex system administration tasks.

**Next Steps:**
- Practice with real-world projects
- Contribute to open-source shell scripts
- Explore advanced topics like process substitution and coprocesses
- Join shell scripting communities for continuous learning
