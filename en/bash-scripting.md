# Bash Scripting

A comprehensive guide to Bash (Bourne Again Shell) scripting — the most widely used shell scripting language on Unix-like systems. This guide covers everything from basic syntax to advanced techniques for writing robust, maintainable shell scripts.

## Quick Reference

### Basic Syntax

```bash
#!/bin/bash

# Variables
name="value"
readonly CONST="constant"

# Arrays
arr=(one two three)
echo "${arr[0]}"
echo "${arr[@]}"

# Conditionals
if [[ condition ]]; then
    # code
elif [[ other ]]; then
    # code
else
    # code
fi

# Loops
for item in "${arr[@]}"; do
    echo "$item"
done

while [[ condition ]]; do
    # code
done

# Functions
function_name() {
    local var="$1"
    echo "$var"
}

# Special variables
$0      # Script name
$1-$9   # Arguments
$@      # All arguments (array)
$#      # Number of arguments
$?      # Exit status
$$      # Process ID
$!      # Last background process PID
```

### Common Patterns

```bash
# Exit on error
set -e
set -o errexit

# Exit on undefined variable
set -u
set -o nounset

# Exit on pipe failure
set -o pipefail

# Debug mode
set -x
set -o xtrace

# Strict mode (recommended)
set -euo pipefail

# Check if command exists
command -v git >/dev/null 2>&1 || { echo "git required"; exit 1; }

# Check if file exists
[[ -f file.txt ]] && echo "exists"

# Default value
: "${VAR:=default}"
```

## Introduction

### What is Bash?

Bash (Bourne Again Shell) is a Unix shell and command language written by Brian Fox for the GNU Project as a free replacement for the Bourne shell (sh). Released in 1989, it has become the default shell on most Linux distributions and macOS (until Catalina).

### Why Bash Scripting?

**Advantages**:
- **Universal**: Available on virtually all Unix-like systems
- **Portable**: Scripts run across different platforms
- **Powerful**: Full programming language + shell commands
- **Integrated**: Direct access to system commands and utilities
- **Automation**: Perfect for system administration tasks

**Use Cases**:
- System administration and automation
- DevOps and CI/CD pipelines
- Build scripts and deployment
- Data processing and text manipulation
- Glue code between programs

### When to Use Bash

**Good For**:
- Quick automation scripts
- System administration tasks
- Text processing and file manipulation
- Wrapping other programs
- CI/CD pipelines

**Not Ideal For**:
- Complex applications (use Python, Go, etc.)
- Heavy computation
- Cross-platform GUI applications
- When you need strong typing

## Getting Started

### Shebang

Always start scripts with a shebang:

```bash
#!/bin/bash
```

**Better** (finds bash in PATH):
```bash
#!/usr/bin/env bash
```

**With Options**:
```bash
#!/bin/bash -e          # Exit on error
#!/bin/bash -x          # Debug mode
#!/bin/bash -eu         # Exit on error and undefined variable
```

### Running Scripts

```bash
# Make executable
chmod +x script.sh

# Run
./script.sh

# Run with bash explicitly
bash script.sh

# Source (run in current shell)
source script.sh
. script.sh
```

### Hello World

```bash
#!/bin/bash

echo "Hello, World!"
```

## Variables

### Declaration and Assignment

```bash
# Simple assignment (no spaces!)
name="John"
count=42

# Read-only variable
readonly PI=3.14159

# Command substitution
current_dir=$(pwd)
today=$(date +%Y-%m-%d)

# Arithmetic
count=$((count + 1))
result=$((5 * 3))
```

### Variable Expansion

```bash
name="John"

# Basic
echo "$name"              # John
echo "${name}"            # John (preferred)

# Default values
echo "${var:-default}"    # Use default if unset
echo "${var:=default}"    # Use and assign default if unset
echo "${var:+alternate}"  # Use alternate if set
echo "${var:?error}"      # Error if unset

# String manipulation
echo "${name:0:2}"        # Jo (substring)
echo "${name#pattern}"    # Remove shortest match from start
echo "${name##pattern}"   # Remove longest match from start
echo "${name%pattern}"    # Remove shortest match from end
echo "${name%%pattern}"   # Remove longest match from end
echo "${name/old/new}"    # Replace first occurrence
echo "${name//old/new}"   # Replace all occurrences

# Length
echo "${#name}"           # 4

# Uppercase/lowercase
echo "${name^^}"          # JOHN
echo "${name,,}"          # john
echo "${name^}"           # John (capitalize first)
```

### Environment Variables

```bash
# Export variable (available to child processes)
export PATH="/usr/local/bin:$PATH"
export EDITOR=vim

# Or in one line
export VAR="value"

# Unset variable
unset VAR

# View all environment variables
printenv
env
```

### Special Variables

```bash
$0          # Script name
$1, $2, ... # Positional arguments
$@          # All arguments as separate words
$*          # All arguments as single word
$#          # Number of arguments
$?          # Exit status of last command
$$          # Current process ID
$!          # PID of last background process
$_          # Last argument of previous command
```

## Arrays

### Indexed Arrays

```bash
# Declaration
arr=(one two three)
arr=([0]=first [1]=second [2]=third)

# Assignment
arr[3]="fourth"

# Access
echo "${arr[0]}"          # first
echo "${arr[@]}"          # All elements
echo "${arr[*]}"          # All elements (single word)

# Length
echo "${#arr[@]}"         # Number of elements
echo "${#arr[0]}"         # Length of first element

# Indices
echo "${!arr[@]}"         # All indices

# Append
arr+=("fifth")

# Iterate
for item in "${arr[@]}"; do
    echo "$item"
done

# With index
for i in "${!arr[@]}"; do
    echo "$i: ${arr[$i]}"
done

# Slice
echo "${arr[@]:1:2}"      # Elements 1-2
```

### Associative Arrays (Bash 4+)

```bash
# Declare
declare -A map

# Assignment
map[key]="value"
map[name]="John"
map[age]="30"

# Access
echo "${map[key]}"

# All keys
echo "${!map[@]}"

# All values
echo "${map[@]}"

# Iterate
for key in "${!map[@]}"; do
    echo "$key: ${map[$key]}"
done

# Check if key exists
if [[ -v map[key] ]]; then
    echo "Key exists"
fi
```

## Control Flow

### If Statements

```bash
# Basic if
if [[ condition ]]; then
    # code
fi

# If-else
if [[ condition ]]; then
    # code
else
    # code
fi

# If-elif-else
if [[ condition1 ]]; then
    # code
elif [[ condition2 ]]; then
    # code
else
    # code
fi

# One-line
[[ condition ]] && echo "true" || echo "false"
```

### Test Conditions

**File Tests**:
```bash
[[ -f file ]]       # File exists and is regular file
[[ -d dir ]]        # Directory exists
[[ -e path ]]       # Path exists (file or directory)
[[ -r file ]]       # File is readable
[[ -w file ]]       # File is writable
[[ -x file ]]       # File is executable
[[ -s file ]]       # File exists and is not empty
[[ -L link ]]       # Path is symbolic link
[[ file1 -nt file2 ]] # file1 newer than file2
[[ file1 -ot file2 ]] # file1 older than file2
```

**String Tests**:
```bash
[[ -z "$str" ]]     # String is empty
[[ -n "$str" ]]     # String is not empty
[[ "$a" == "$b" ]]  # Strings are equal
[[ "$a" != "$b" ]]  # Strings are not equal
[[ "$a" < "$b" ]]   # Lexicographic less than
[[ "$a" > "$b" ]]   # Lexicographic greater than
[[ "$str" =~ regex ]] # String matches regex
```

**Numeric Tests**:
```bash
[[ $a -eq $b ]]     # Equal
[[ $a -ne $b ]]     # Not equal
[[ $a -lt $b ]]     # Less than
[[ $a -le $b ]]     # Less than or equal
[[ $a -gt $b ]]     # Greater than
[[ $a -ge $b ]]     # Greater than or equal
```

**Logical Operators**:
```bash
[[ cond1 && cond2 ]] # AND
[[ cond1 || cond2 ]] # OR
[[ ! cond ]]         # NOT

# Combining
if [[ -f file && -r file ]]; then
    echo "File exists and is readable"
fi
```

### Case Statements

```bash
case "$var" in
    pattern1)
        # code
        ;;
    pattern2|pattern3)
        # code
        ;;
    *)
        # default
        ;;
esac

# Example
case "$1" in
    start)
        echo "Starting..."
        ;;
    stop)
        echo "Stopping..."
        ;;
    restart)
        echo "Restarting..."
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac
```

### Loops

**For Loop**:
```bash
# Iterate over list
for item in one two three; do
    echo "$item"
done

# Iterate over array
for item in "${arr[@]}"; do
    echo "$item"
done

# C-style for loop
for ((i=0; i<10; i++)); do
    echo "$i"
done

# Iterate over files
for file in *.txt; do
    echo "$file"
done

# Iterate over command output
for line in $(cat file.txt); do
    echo "$line"
done
```

**While Loop**:
```bash
i=0
while [[ $i -lt 10 ]]; do
    echo "$i"
    ((i++))
done

# Read file line by line
while IFS= read -r line; do
    echo "$line"
done < file.txt
```

**Until Loop**:
```bash
i=0
until [[ $i -ge 10 ]]; do
    echo "$i"
    ((i++))
done
```

**Loop Control**:
```bash
# Break (exit loop)
for i in {1..10}; do
    [[ $i -eq 5 ]] && break
    echo "$i"
done

# Continue (skip iteration)
for i in {1..10}; do
    [[ $i -eq 5 ]] && continue
    echo "$i"
done
```

## Functions

### Basic Functions

```bash
# Declaration
function_name() {
    # code
}

# Or with 'function' keyword
function function_name {
    # code
}

# With local variables
greet() {
    local name="$1"
    echo "Hello, $name!"
}

# Call function
greet "John"
```

### Function Arguments

```bash
process_file() {
    local file="$1"
    local option="$2"
    
    # Check arguments
    if [[ $# -lt 1 ]]; then
        echo "Usage: process_file <file> [option]" >&2
        return 1
    fi
    
    # Use arguments
    echo "Processing $file with option: ${option:-default}"
}

# $1, $2, ... : Positional arguments
# $@          : All arguments (array)
# $*          : All arguments (string)
# $#          : Number of arguments
```

### Return Values

```bash
# Return status (0-255)
check_file() {
    [[ -f "$1" ]] && return 0 || return 1
}

if check_file "test.txt"; then
    echo "File exists"
fi

# Return data via stdout
get_username() {
    echo "$USER"
}

username=$(get_username)

# Return data via variable (use nameref in bash 4.3+)
get_data() {
    local -n result=$1
    result="some data"
}

get_data myvar
echo "$myvar"
```

### Local vs Global Variables

```bash
global_var="global"

my_function() {
    local local_var="local"
    global_var="modified"    # Modifies global
    
    echo "$local_var"        # Accessible
}

my_function
echo "$global_var"           # "modified"
echo "$local_var"            # Empty (not accessible)
```

## Input and Output

### Reading Input

```bash
# Read single variable
read -p "Enter name: " name

# Read multiple variables
read -p "Enter first and last name: " first last

# Read with timeout
read -t 5 -p "Enter name (5s timeout): " name

# Read password (no echo)
read -s -p "Enter password: " password
echo  # Newline after password

# Read line by line
while IFS= read -r line; do
    echo "$line"
done < file.txt

# Read into array
mapfile -t lines < file.txt
# or
readarray -t lines < file.txt
```

### Output

```bash
# Print to stdout
echo "message"
printf "formatted %s\n" "message"

# Print to stderr
echo "error" >&2
printf "error: %s\n" "message" >&2

# Redirect stdout
command > file.txt

# Redirect stderr
command 2> error.log

# Redirect both
command &> output.log
command > output.log 2>&1

# Append
command >> file.txt
command 2>> error.log

# Discard output
command > /dev/null 2>&1

# Here document
cat << EOF
Line 1
Line 2
EOF

# Here string
grep "pattern" <<< "string to search"
```

## String Manipulation

### String Operations

```bash
str="Hello, World!"

# Length
${#str}                   # 13

# Substring
${str:7}                  # World!
${str:7:5}                # World

# Remove prefix
${str#Hello, }            # World! (shortest match)
${str##*/}                # Removes up to last /

# Remove suffix
${str%!}                  # Hello, World (shortest match)
${str%%,*}                # Hello (longest match)

# Replace
${str/World/Universe}     # Replace first
${str//o/0}              # Replace all

# Case conversion
${str^^}                  # HELLO, WORLD!
${str,,}                  # hello, world!
${str^}                   # Hello, world! (first char)
```

### String Comparison

```bash
str1="hello"
str2="world"

# Equality
[[ "$str1" == "$str2" ]]
[[ "$str1" = "$str2" ]]   # Same as ==

# Inequality
[[ "$str1" != "$str2" ]]

# Pattern matching
[[ "$str1" == h* ]]       # Starts with h

# Regex matching
[[ "$str1" =~ ^[a-z]+$ ]]

# Lexicographic comparison
[[ "$str1" < "$str2" ]]
[[ "$str1" > "$str2" ]]
```

### String Splitting

```bash
# Using IFS
IFS=',' read -ra fields <<< "a,b,c"
for field in "${fields[@]}"; do
    echo "$field"
done

# Split into array
str="one:two:three"
IFS=':' read -ra parts <<< "$str"
```

### String Joining

```bash
# Join array with delimiter
arr=(one two three)
IFS=,
joined="${arr[*]}"        # one,two,three
unset IFS
```

## File Operations

### File Tests

```bash
# Existence
[[ -e file ]]       # Exists
[[ -f file ]]       # Regular file
[[ -d dir ]]        # Directory
[[ -L link ]]       # Symbolic link
[[ -s file ]]       # Not empty

# Permissions
[[ -r file ]]       # Readable
[[ -w file ]]       # Writable
[[ -x file ]]       # Executable

# Comparison
[[ file1 -nt file2 ]] # Newer than
[[ file1 -ot file2 ]] # Older than
[[ file1 -ef file2 ]] # Same file
```

### Reading Files

```bash
# Read entire file
content=$(<file.txt)

# Read line by line (preserving whitespace)
while IFS= read -r line; do
    echo "$line"
done < file.txt

# Read into array
mapfile -t lines < file.txt

# Read with custom delimiter
while IFS=: read -r user pass uid gid info home shell; do
    echo "User: $user, UID: $uid"
done < /etc/passwd
```

### Writing Files

```bash
# Write
echo "text" > file.txt

# Append
echo "text" >> file.txt

# Write multiple lines
cat > file.txt << 'EOF'
Line 1
Line 2
EOF

# Write with printf
printf "%s\n" "line1" "line2" > file.txt
```

### File Manipulation

```bash
# Copy
cp source dest
cp -r dir1 dir2           # Recursive

# Move/rename
mv old new

# Delete
rm file
rm -rf dir                # Recursive, force

# Create directory
mkdir dir
mkdir -p path/to/dir      # Create parent dirs

# Change permissions
chmod +x script.sh
chmod 755 file
chmod u+x file

# Change owner
chown user:group file
```

## Process Management

### Running Commands

```bash
# Run in foreground
command args

# Run in background
command args &

# Get PID of background process
command &
pid=$!

# Wait for background process
wait $pid

# Run multiple in background and wait
command1 &
command2 &
wait
```

### Exit Status

```bash
# Check exit status
command
if [[ $? -eq 0 ]]; then
    echo "Success"
fi

# Or directly
if command; then
    echo "Success"
fi

# Set exit status
exit 0    # Success
exit 1    # Error

# Return from function
return 0
return 1
```

### Pipelines

```bash
# Simple pipe
cat file.txt | grep "pattern" | sort

# Pipe with error handling (set -o pipefail)
set -o pipefail
cat file.txt | grep "pattern" | sort
status=$?

# Process substitution
diff <(sort file1.txt) <(sort file2.txt)

# Redirect to multiple commands (tee)
command | tee output.txt | grep "pattern"
```

### Job Control

```bash
# List jobs
jobs

# Bring to foreground
fg %1

# Send to background
bg %1

# Kill job
kill %1

# Disown job (keep running after shell exit)
disown %1
```

## Advanced Features

### Command Substitution

```bash
# Modern syntax
result=$(command)
files=$(ls *.txt)

# Old syntax (avoid)
result=`command`

# Nested
outer=$(echo "inner: $(date)")

# Strip trailing newlines
result=$(command | tr -d '\n')
```

### Arithmetic

```bash
# Arithmetic expansion
result=$((5 + 3))
result=$((x * y))
result=$((++i))
result=$((i++))

# Arithmetic evaluation
((i++))
((count += 5))

# Comparison in arithmetic context
if ((count > 10)); then
    echo "Count is greater than 10"
fi

# Operators
$((a + b))          # Addition
$((a - b))          # Subtraction
$((a * b))          # Multiplication
$((a / b))          # Division
$((a % b))          # Modulus
$((a ** b))         # Exponentiation
```

### Brace Expansion

```bash
# Range
echo {1..10}              # 1 2 3 4 5 6 7 8 9 10
echo {a..z}               # a b c d ... z
echo {01..10}             # 01 02 03 ... 10

# List
echo {cat,dog,bird}       # cat dog bird

# Combinations
echo file{1,2,3}.txt      # file1.txt file2.txt file3.txt

# Nested
echo {a,b}{1,2}           # a1 a2 b1 b2

# Create multiple files
touch file{1..100}.txt
```

### Parameter Expansion

```bash
# Default values
${var:-default}           # Use default if unset/empty
${var:=default}           # Assign default if unset/empty
${var:+alternate}         # Use alternate if set
${var:?error}             # Exit with error if unset

# Length
${#var}

# Substring
${var:offset}
${var:offset:length}

# Remove pattern (from start)
${var#pattern}            # Shortest match
${var##pattern}           # Longest match

# Remove pattern (from end)
${var%pattern}            # Shortest match
${var%%pattern}           # Longest match

# Replace
${var/pattern/replacement}   # First occurrence
${var//pattern/replacement}  # All occurrences

# Case modification
${var^^}                  # Uppercase
${var,,}                  # Lowercase
${var^}                   # Capitalize first
${var~}                   # Toggle case of first
```

### Here Documents

```bash
# Basic here document
cat << EOF
Line 1
Variable: $var
Line 3
EOF

# Quoted (no variable expansion)
cat << 'EOF'
Literal $var
EOF

# Indented (strip leading tabs)
cat <<- EOF
	Indented
	Text
EOF

# Assign to variable
read -r -d '' var << EOF
Multiple
Lines
EOF
```

## Error Handling

### Set Options

```bash
# Exit on error
set -e
set -o errexit

# Exit on undefined variable
set -u
set -o nounset

# Exit on pipe failure
set -o pipefail

# Debug mode (print commands)
set -x
set -o xtrace

# Strict mode (recommended)
set -euo pipefail

# Disable option
set +e
```

### Trap

```bash
# Cleanup on exit
cleanup() {
    echo "Cleaning up..."
    rm -f temp_file
}
trap cleanup EXIT

# Handle signals
trap 'echo "Interrupted"; exit 1' INT TERM

# Handle errors
trap 'echo "Error at line $LINENO"' ERR

# Example with cleanup
temp_file=$(mktemp)
trap "rm -f $temp_file" EXIT

echo "data" > "$temp_file"
# File will be cleaned up even if script exits early
```

### Error Messages

```bash
# Print to stderr
error() {
    echo "ERROR: $*" >&2
}

# Print and exit
die() {
    echo "FATAL: $*" >&2
    exit 1
}

# Usage
[[ -f file.txt ]] || die "file.txt not found"

# With line number
error_at() {
    echo "ERROR at line ${BASH_LINENO[0]}: $*" >&2
}
```

### Defensive Programming

```bash
#!/bin/bash
set -euo pipefail
IFS=$'\n\t'

# Check dependencies
command -v jq >/dev/null || { echo "jq required"; exit 1; }

# Validate arguments
if [[ $# -lt 1 ]]; then
    echo "Usage: $0 <file>" >&2
    exit 1
fi

# Check file exists
[[ -f "$1" ]] || { echo "File not found: $1" >&2; exit 1; }

# Main logic with error handling
main() {
    local file="$1"
    
    # Try operation
    if ! process_file "$file"; then
        echo "Failed to process $file" >&2
        return 1
    fi
    
    echo "Success"
}

main "$@"
```

## Argument Parsing

### Manual Parsing

```bash
# Simple argument parsing
while [[ $# -gt 0 ]]; do
    case $1 in
        -h|--help)
            show_help
            exit 0
            ;;
        -v|--verbose)
            verbose=1
            shift
            ;;
        -o|--output)
            output="$2"
            shift 2
            ;;
        -*)
            echo "Unknown option: $1" >&2
            exit 1
            ;;
        *)
            # Positional argument
            files+=("$1")
            shift
            ;;
    esac
done
```

### Using getopts

```bash
# Short options only
while getopts "hvo:" opt; do
    case $opt in
        h)
            show_help
            exit 0
            ;;
        v)
            verbose=1
            ;;
        o)
            output="$OPTARG"
            ;;
        \?)
            echo "Invalid option: -$OPTARG" >&2
            exit 1
            ;;
        :)
            echo "Option -$OPTARG requires argument" >&2
            exit 1
            ;;
    esac
done

# Shift past processed options
shift $((OPTIND - 1))

# Remaining arguments in $@
```

### Long Options (getopt)

```bash
# Using getopt (GNU)
OPTS=$(getopt -o hvo: --long help,verbose,output: -n "$0" -- "$@")
eval set -- "$OPTS"

while true; do
    case "$1" in
        -h|--help)
            show_help
            exit 0
            ;;
        -v|--verbose)
            verbose=1
            shift
            ;;
        -o|--output)
            output="$2"
            shift 2
            ;;
        --)
            shift
            break
            ;;
        *)
            echo "Internal error!" >&2
            exit 1
            ;;
    esac
done
```

## Best Practices

### Script Template

```bash
#!/bin/bash
#
# Script description
#
# Usage: script.sh [options] <args>

set -euo pipefail
IFS=$'\n\t'

# Constants
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "${BASH_SOURCE[0]}")"

# Default values
verbose=0
output=""

# Functions
show_help() {
    cat << EOF
Usage: $SCRIPT_NAME [options] <file>

Options:
    -h, --help      Show this help
    -v, --verbose   Verbose output
    -o, --output    Output file

Examples:
    $SCRIPT_NAME input.txt
    $SCRIPT_NAME -o result.txt input.txt
EOF
}

error() {
    echo "ERROR: $*" >&2
}

die() {
    error "$*"
    exit 1
}

# Parse arguments
while [[ $# -gt 0 ]]; do
    case $1 in
        -h|--help)
            show_help
            exit 0
            ;;
        -v|--verbose)
            verbose=1
            shift
            ;;
        -o|--output)
            output="$2"
            shift 2
            ;;
        -*)
            die "Unknown option: $1"
            ;;
        *)
            file="$1"
            shift
            ;;
    esac
done

# Validate
[[ -n "${file:-}" ]] || die "File argument required"
[[ -f "$file" ]] || die "File not found: $file"

# Main logic
main() {
    [[ $verbose -eq 1 ]] && echo "Processing $file..."
    
    # Your code here
    
    echo "Done"
}

main
```

### Quoting

```bash
# Always quote variables
file="my file.txt"
cat "$file"               # Correct
cat $file                 # Wrong (splits on space)

# Quote arrays
files=("file 1.txt" "file 2.txt")
for file in "${files[@]}"; do
    echo "$file"
done

# Unquoted for word splitting (rare)
options="-l -a -h"
ls $options              # Intentional word splitting
```

### Error Handling

```bash
# Check command success
if ! command; then
    echo "Command failed" >&2
    exit 1
fi

# Or
command || { echo "Failed" >&2; exit 1; }

# With error message
grep pattern file.txt || {
    echo "Pattern not found" >&2
    exit 1
}

# Try-catch style
if result=$(command 2>&1); then
    echo "Success: $result"
else
    echo "Failed: $result" >&2
    exit 1
fi
```

### Portable Scripts

```bash
# Use POSIX features when possible
# Use [[ ]] instead of [ ] (bash-specific but safer)
# Avoid bashisms if targeting /bin/sh

# Check for bash
[[ -n "${BASH_VERSION:-}" ]] || {
    echo "Requires bash" >&2
    exit 1
}

# Require minimum bash version
if [[ "${BASH_VERSINFO[0]}" -lt 4 ]]; then
    echo "Requires bash 4+" >&2
    exit 1
fi
```

## Common Patterns

### Checking Dependencies

```bash
# Check if command exists
check_command() {
    command -v "$1" >/dev/null 2>&1
}

if ! check_command git; then
    echo "git is required" >&2
    exit 1
fi

# Check multiple
for cmd in git curl jq; do
    command -v "$cmd" >/dev/null || {
        echo "$cmd is required" >&2
        exit 1
    }
done
```

### Finding Script Directory

```bash
# Reliable way to find script directory
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# Or with readlink
SCRIPT_DIR="$(dirname "$(readlink -f "${BASH_SOURCE[0]}")")"

# Load file relative to script
source "$SCRIPT_DIR/lib/utils.sh"
```

### Temporary Files

```bash
# Create temp file
temp_file=$(mktemp)
trap "rm -f $temp_file" EXIT

# Create temp directory
temp_dir=$(mktemp -d)
trap "rm -rf $temp_dir" EXIT

# Use temp file
echo "data" > "$temp_file"
process "$temp_file"
```

### Parallel Execution

```bash
# Run commands in parallel
command1 &
command2 &
command3 &
wait

# Parallel with error handling
pids=()
for item in "${items[@]}"; do
    process "$item" &
    pids+=($!)
done

# Wait and check status
status=0
for pid in "${pids[@]}"; do
    if ! wait "$pid"; then
        status=1
    fi
done

exit $status
```

### Configuration Files

```bash
# Load config file
config_file="${HOME}/.myapp/config"
if [[ -f "$config_file" ]]; then
    source "$config_file"
fi

# Or with validation
load_config() {
    local config="$1"
    [[ -f "$config" ]] || return 1
    
    # Only source if safe
    if [[ -r "$config" ]]; then
        source "$config"
    fi
}
```

### Logging

```bash
# Simple logging
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*"
}

log "Script started"

# Log levels
LOG_LEVEL=${LOG_LEVEL:-INFO}

log_debug() { [[ "$LOG_LEVEL" == "DEBUG" ]] && echo "[DEBUG] $*" >&2; }
log_info() { echo "[INFO] $*"; }
log_warn() { echo "[WARN] $*" >&2; }
log_error() { echo "[ERROR] $*" >&2; }

# Usage
log_info "Processing file"
log_error "File not found"
```

### Lock Files

```bash
# Create lock file
lock_file="/tmp/myapp.lock"

acquire_lock() {
    if [[ -f "$lock_file" ]]; then
        echo "Already running" >&2
        exit 1
    fi
    
    echo $$ > "$lock_file"
    trap "rm -f $lock_file" EXIT
}

acquire_lock
# Your code here
```

## Security

### Input Validation

```bash
# Validate input
validate_email() {
    local email="$1"
    [[ "$email" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]
}

# Sanitize filename
sanitize() {
    echo "$1" | tr -cd '[:alnum:]._-'
}

# Validate number
is_number() {
    [[ "$1" =~ ^[0-9]+$ ]]
}
```

### Avoiding Injection

```bash
# DON'T: eval user input
user_input="$1"
eval "$user_input"        # DANGEROUS!

# DON'T: Unquoted variables in commands
rm -rf $dir               # DANGEROUS! (what if dir="/ ")

# DO: Quote variables
rm -rf "$dir"

# DON'T: Pass user input to shell
sh -c "$user_input"       # DANGEROUS!

# DO: Use array for command arguments
args=("$@")
command "${args[@]}"
```

### Safe Permissions

```bash
# Create file with specific permissions
(umask 077 && touch sensitive.txt)

# Set restrictive permissions
chmod 600 private.key
chmod 700 script.sh

# Check permissions before use
if [[ ! -r "$file" ]]; then
    echo "Cannot read $file" >&2
    exit 1
fi
```

## Debugging

### Debug Mode

```bash
# Enable debug output
bash -x script.sh

# Or in script
set -x

# Debug specific section
set -x
# code to debug
set +x

# Verbose mode
set -v
```

### Debug Utilities

```bash
# Print variable
echo "DEBUG: var=$var" >&2

# Print with line number
echo "DEBUG: ${BASH_SOURCE[0]}:${LINENO}: $var" >&2

# Trace function
trace() {
    echo "TRACE: $*" >&2
    "$@"
}

trace command arg1 arg2
```

### ShellCheck

Use ShellCheck to find bugs:

```bash
# Install
brew install shellcheck  # macOS
apt install shellcheck   # Ubuntu

# Check script
shellcheck script.sh

# Disable specific warnings
# shellcheck disable=SC2034
unused_var="value"

# In script header
# shellcheck disable=SC1090,SC2148
```

## Advanced Topics

### Subshells

```bash
# Run in subshell (doesn't affect parent)
(
    cd /tmp
    echo "In subshell: $(pwd)"
)
echo "In parent: $(pwd)"

# Grouping commands
{
    command1
    command2
} > output.txt

# Versus subshell
(
    command1
    command2
) > output.txt
```

### Process Substitution

```bash
# Compare outputs
diff <(sort file1.txt) <(sort file2.txt)

# Read from multiple sources
while read -r line; do
    echo "$line"
done < <(cat file1.txt file2.txt)

# Provide input to command expecting file
command <(echo "data")
```

### Coprocesses

```bash
# Create coprocess
coproc bc

# Send to coprocess
echo "2 + 2" >&"${COPROC[1]}"

# Read from coprocess
read -r result <&"${COPROC[0]}"
echo "Result: $result"

# Close coprocess
kill "$COPROC_PID"
```

### Regex Matching

```bash
# Match pattern
if [[ "$email" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
    echo "Valid email"
fi

# Capture groups
if [[ "$str" =~ ([0-9]+)-([0-9]+) ]]; then
    first="${BASH_REMATCH[1]}"
    second="${BASH_REMATCH[2]}"
    echo "First: $first, Second: $second"
fi
```

### Associative Arrays for Options

```bash
declare -A options=(
    [verbose]=0
    [output]=""
    [debug]=0
)

# Parse
while [[ $# -gt 0 ]]; do
    case $1 in
        -v) options[verbose]=1; shift ;;
        -o) options[output]="$2"; shift 2 ;;
        -d) options[debug]=1; shift ;;
    esac
done

# Use
[[ ${options[verbose]} -eq 1 ]] && echo "Verbose mode"
```

## Performance

### Optimization Tips

**Avoid Subshells**:
```bash
# Slow
result=$(cat file.txt)

# Fast
result=$(<file.txt)
```

**Use Built-ins**:
```bash
# Slow
result=$(expr $a + $b)

# Fast
result=$((a + b))
```

**Minimize External Commands**:
```bash
# Slow
for file in $(ls *.txt); do
    # ...
done

# Fast
for file in *.txt; do
    # ...
done
```

**Read Files Efficiently**:
```bash
# Slow (spawns cat)
while read line; do
    # ...
done < <(cat file.txt)

# Fast
while read line; do
    # ...
done < file.txt
```

## Testing

### Test Framework (bats)

**Install bats**:
```bash
brew install bats-core
```

**test.bats**:
```bash
#!/usr/bin/env bats

setup() {
    # Run before each test
    temp_dir=$(mktemp -d)
}

teardown() {
    # Run after each test
    rm -rf "$temp_dir"
}

@test "addition works" {
    result=$((2 + 2))
    [[ $result -eq 4 ]]
}

@test "function returns correct value" {
    source script.sh
    result=$(my_function "input")
    [[ "$result" == "expected" ]]
}
```

**Run tests**:
```bash
bats test.bats
```

### Manual Testing

```bash
# Test functions
test_function() {
    local expected="$1"
    local actual="$2"
    
    if [[ "$expected" == "$actual" ]]; then
        echo "✓ Pass"
        return 0
    else
        echo "✗ Fail: expected '$expected', got '$actual'" >&2
        return 1
    fi
}

# Usage
result=$(my_function "input")
test_function "expected output" "$result"
```

## References & Resources

### Official Documentation

- [GNU Bash Manual](https://www.gnu.org/software/bash/manual/)
- [Bash Reference Manual (PDF)](https://www.gnu.org/software/bash/manual/bash.pdf)
- [POSIX Shell Specification](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html)

### Learning Resources

- [Bash Guide](https://mywiki.wooledge.org/BashGuide) - Comprehensive wiki
- [Advanced Bash-Scripting Guide](https://tldp.org/LDP/abs/html/) - Classic resource
- [Bash Hackers Wiki](https://wiki.bash-hackers.org/) - Advanced techniques
- [ShellCheck Wiki](https://www.shellcheck.net/wiki/) - Common mistakes

### Tools

- [ShellCheck](https://www.shellcheck.net/) - Linter for shell scripts
- [explainshell.com](https://explainshell.com/) - Parse and explain commands
- [bashdb](http://bashdb.sourceforge.net/) - Bash debugger
- [bats](https://github.com/bats-core/bats-core) - Testing framework

### Books

- *Learning the bash Shell* by Cameron Newham (O'Reilly)
- *Bash Cookbook* by Carl Albing and JP Vossen (O'Reilly)
- *Classic Shell Scripting* by Arnold Robbins and Nelson Beebe (O'Reilly)

### Style Guides

- [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html)
- [Chromium Shell Style Guide](https://chromium.googlesource.com/chromium/src/+/master/styleguide/shell.md)

### Community

- [Stack Overflow - bash tag](https://stackoverflow.com/questions/tagged/bash)
- [r/bash](https://www.reddit.com/r/bash/)
- [Unix & Linux Stack Exchange](https://unix.stackexchange.com/)
