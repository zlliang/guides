# Fish Shell

A comprehensive guide to Fish (the Friendly Interactive Shell) — a smart and user-friendly command-line shell for Linux, macOS, and other Unix-like systems. This guide focuses primarily on interactive usage, with an introduction to shell scripting.

## Quick Reference

### Essential Commands

```fish
# History search
fish_config                 # Open web-based configuration
history search <term>       # Search command history
history merge              # Merge history from all sessions

# Completions
complete -c <cmd>          # Show completions for command
funced <function>          # Edit function in editor
funcsave <function>        # Save function permanently

# Variables
set VAR value             # Set variable
set -e VAR                # Erase variable
set -x VAR value          # Export variable (universal)
set -U VAR value          # Universal variable (across sessions)

# Functions
function name
    # body
end

# Abbreviations
abbr --add gco 'git checkout'
abbr --erase gco
```

### Key Bindings

```
Ctrl+R      # Search history (reverse)
Alt+↑/↓     # Previous/next argument
Alt+E       # Edit command in $EDITOR
Alt+L       # List directory
Tab         # Complete or show completions
→           # Accept autosuggestion
Ctrl+→      # Accept one word of autosuggestion
```

### Differences from Bash

```fish
# Bash                    # Fish
export VAR=value          set -x VAR value
VAR=value                 set VAR value
if [ "$x" = "y" ]        if test "$x" = "y"
source script.sh          source script.fish
$?                        $status
$@                        $argv
```

## Introduction

### What is Fish?

Fish (Friendly Interactive Shell) is a modern command-line shell designed to be user-friendly, feature-rich, and fun to use. Unlike traditional shells like Bash or Zsh, Fish is designed with interactive use as the primary goal.

**Key Philosophy**:
- Work out of the box with sensible defaults
- Interactive features should be discoverable
- Configuration should be simple
- The shell should be helpful, not arcane

### Why Use Fish?

**Advantages**:
- **Syntax highlighting**: See errors before you run commands
- **Autosuggestions**: Based on history and completions
- **Tab completions**: Work by default for common commands
- **Sensible scripting**: No cryptic syntax like `${var%%suffix}`
- **Web-based configuration**: `fish_config` opens a GUI
- **No configuration needed**: Works great out of the box

**When to Use Fish**:
- As your interactive shell for daily terminal work
- When you want a modern, pleasant terminal experience
- When learning shell usage (clearer syntax than bash)

**When to Use Bash Instead**:
- Writing portable scripts (Fish syntax is different)
- When you need POSIX compatibility
- In environments where Fish isn't available

### Installation

**macOS**:
```bash
brew install fish
```

**Ubuntu/Debian**:
```bash
sudo apt install fish
```

**Fedora**:
```bash
sudo dnf install fish
```

**Arch Linux**:
```bash
sudo pacman -S fish
```

**From Source**:
```bash
git clone https://github.com/fish-shell/fish-shell.git
cd fish-shell
cmake . && make && sudo make install
```

**Set as Default Shell**:
```bash
# Add Fish to /etc/shells if not present
echo /usr/local/bin/fish | sudo tee -a /etc/shells

# Set as default shell
chsh -s /usr/local/bin/fish
```

## Interactive Features

### Autosuggestions

Fish suggests commands as you type based on:
- Command history
- Available completions
- Current directory contents

**Using Autosuggestions**:
```
$ git che█                    # You type "git che"
$ git checkout█               # Fish suggests "checkout" (gray text)
$ git checkout█               # Press → to accept
$ git checkout main█          # Continue typing or accept next word with Ctrl+→
```

**Configuration**:
```fish
# Disable autosuggestions
set -U fish_autosuggestion_enabled 0

# Change color (default is gray)
set -U fish_color_autosuggestion brblack
```

### Syntax Highlighting

Fish highlights commands as you type:
- **Valid commands**: Default color (usually white/bright)
- **Invalid commands**: Red
- **Strings**: Green or yellow
- **Options/flags**: Cyan
- **Comments**: Gray

**Example**:
```fish
$ vim file.txt              # "vim" is highlighted (valid command)
$ vimm file.txt             # "vimm" is red (invalid command)
$ echo "hello"              # "hello" is highlighted as string
```

**Customize Colors**:
```fish
# View current colors
set | grep fish_color

# Change command color
set -U fish_color_command blue

# Change error color  
set -U fish_color_error red --bold
```

### Tab Completions

Fish provides intelligent completions for:
- Commands and their options
- File paths
- Git branches, remotes, tags
- Package manager packages
- SSH hostnames
- Environment variables

**Examples**:
```fish
# Command completion
$ git che<Tab>              # Lists: checkout, cherry, cherry-pick

# Option completion
$ ls --<Tab>                # Lists all --long-options

# Git branch completion
$ git checkout <Tab>        # Lists all branches

# File completion with description
$ vim <Tab>                 # Shows files with preview
```

**Custom Completions**:
```fish
# Create completion file: ~/.config/fish/completions/mycommand.fish
complete -c mycommand -s h -l help -d 'Show help'
complete -c mycommand -s v -l verbose -d 'Verbose output'
complete -c mycommand -a '(__fish_complete_directories)' -d 'Directory'
```

### History

**Search History**:
```fish
# Interactive history search (Ctrl+R)
$ <Ctrl+R>
bck-i-search: git_

# Search history with command
$ history search git
$ history search --contains checkout

# Show recent history
$ history | head -20
```

**History Navigation**:
- `↑` / `↓`: Navigate through history
- `Alt+↑` / `Alt+↓`: Navigate through history search results
- `Alt+.`: Insert last argument from previous command

**Manage History**:
```fish
# Clear history
$ history clear

# Delete specific item
$ history delete --prefix "rm -rf"

# Merge history from all sessions
$ history merge

# Save history
$ history save
```

### Command Substitution

Fish uses `()` instead of backticks or `$()`:

```fish
# Command substitution
$ echo (date)
$ set current_dir (pwd)

# Versus bash
$ echo $(date)
$ current_dir=$(pwd)
```

### Wildcards and Globbing

**Basic Globbing**:
```fish
$ ls *.txt                  # All .txt files
$ ls **/*.fish              # All .fish files recursively
$ ls *.{txt,md}             # All .txt and .md files
```

**Advanced Patterns**:
```fish
# Match by type
$ ls (find . -type f)       # All files
$ ls **.txt                 # Recursive .txt files

# Exclude patterns
$ ls !(*.txt)               # All except .txt (not in fish, use different approach)
```

## Configuration

### Configuration Files

**Main Config**: `~/.config/fish/config.fish`
- Runs on every shell start
- Set environment variables
- Define functions and abbreviations

**Function Files**: `~/.config/fish/functions/`
- One file per function
- Auto-loaded when needed
- Use `funcsave` to save here

**Completions**: `~/.config/fish/completions/`
- Custom completion definitions
- One file per command

### Basic Configuration

**~/.config/fish/config.fish**:
```fish
# Environment variables
set -x EDITOR vim
set -x VISUAL code
set -x PAGER less

# Add to PATH
fish_add_path /usr/local/bin
fish_add_path $HOME/.local/bin
fish_add_path $HOME/go/bin

# Aliases (use abbreviations instead)
abbr --add ll 'ls -la'
abbr --add g 'git'
abbr --add dc 'docker-compose'

# Greeting
set -U fish_greeting ""  # Disable greeting

# Colors
set -U fish_color_command blue
set -U fish_color_error red
set -U fish_color_param cyan
```

### Web Configuration

```fish
$ fish_config
```

Opens a web browser with:
- **Colors**: Change syntax highlighting colors
- **Prompt**: Choose from built-in prompt themes
- **Functions**: View and edit functions
- **Variables**: View and edit variables
- **History**: Browse command history
- **Bindings**: View and modify key bindings
- **Abbreviations**: Manage abbreviations

## Variables

### Setting Variables

```fish
# Local variable (current scope)
set name value

# Universal variable (all sessions, persists)
set -U name value

# Exported variable (environment)
set -x name value

# Multiple values (list)
set colors red blue green

# Append
set -a PATH /new/path
```

### Variable Scopes

**Local** (default):
```fish
function test
    set local_var "only in function"
end
```

**Global** (`-g`):
```fish
set -g global_var "available everywhere in session"
```

**Universal** (`-U`):
```fish
set -U universal_var "persists across sessions"
```

### Special Variables

```fish
$status                     # Exit status (like $? in bash)
$pipestatus                 # Exit status of pipe commands
$argv                       # Function/script arguments
$fish_pid                   # PID of current fish process
$HOME                       # Home directory
$PATH                       # Executable search path
$PWD                        # Current directory
$SHLVL                      # Shell nesting level
```

### Lists (Arrays)

```fish
# Create list
set colors red blue green

# Access elements (1-indexed!)
echo $colors[1]             # red
echo $colors[2..3]          # blue green
echo $colors[-1]            # green (last)

# Iterate
for color in $colors
    echo $color
end

# Length
count $colors               # 3

# Append
set -a colors yellow

# Check if contains
contains blue $colors       # Returns 0 if true
```

### Environment Variables

```fish
# Export variable
set -x EDITOR vim

# Or use -gx for global export
set -gx JAVA_HOME /usr/lib/jvm/java-11

# View all exported variables
set -x

# Unexport (but keep variable)
set -u VARNAME
```

## Functions

### Defining Functions

**Inline**:
```fish
$ function hello
    echo "Hello, $argv!"
end
```

**In File** (`~/.config/fish/functions/hello.fish`):
```fish
function hello
    echo "Hello, $argv!"
end
```

**With Description**:
```fish
function hello -d "Greet the user"
    echo "Hello, $argv!"
end
```

### Function Arguments

```fish
function greet
    # $argv[1] is first argument
    # $argv is all arguments
    # count $argv is number of arguments
    
    if test (count $argv) -eq 0
        echo "Usage: greet <name>"
        return 1
    end
    
    echo "Hello, $argv[1]!"
end
```

**Argument Parsing**:
```fish
function mycommand
    argparse 'h/help' 'v/verbose' 'o/output=' -- $argv
    or return
    
    if set -q _flag_help
        echo "Usage: mycommand [options]"
        return 0
    end
    
    if set -q _flag_verbose
        echo "Verbose mode enabled"
    end
    
    if set -q _flag_output
        echo "Output: $_flag_output"
    end
end
```

### Editing and Saving Functions

```fish
# Edit function in $EDITOR
$ funced myfunction

# Save function to ~/.config/fish/functions/
$ funcsave myfunction

# Show function definition
$ functions myfunction

# List all functions
$ functions
```

### Useful Function Examples

**CD and LS**:
```fish
function cd
    builtin cd $argv
    and ls
end
```

**Make Directory and Enter**:
```fish
function mkcd
    mkdir -p $argv
    and cd $argv
end
```

**Extract Archives**:
```fish
function extract
    if test -f $argv[1]
        switch $argv[1]
            case '*.tar.bz2'
                tar xjf $argv[1]
            case '*.tar.gz'
                tar xzf $argv[1]
            case '*.bz2'
                bunzip2 $argv[1]
            case '*.gz'
                gunzip $argv[1]
            case '*.tar'
                tar xf $argv[1]
            case '*.zip'
                unzip $argv[1]
            case '*'
                echo "Unknown archive type"
        end
    else
        echo "File not found"
    end
end
```

## Abbreviations

Abbreviations expand when you press Space or Enter. Unlike aliases, you see the full command before running it.

### Creating Abbreviations

```fish
# Add abbreviation
$ abbr --add gco 'git checkout'
$ abbr --add gst 'git status'
$ abbr --add dc 'docker-compose'

# Universal abbreviation (persists)
$ abbr --add --global gp 'git push'
```

### Using Abbreviations

```fish
$ gco <Space>               # Expands to: git checkout 
$ gst <Enter>               # Expands and runs: git status
```

### Managing Abbreviations

```fish
# List abbreviations
$ abbr --show

# Erase abbreviation
$ abbr --erase gco

# Rename abbreviation
$ abbr --rename gco gc
```

### Common Abbreviations

```fish
# Git
abbr --add g 'git'
abbr --add ga 'git add'
abbr --add gc 'git commit'
abbr --add gco 'git checkout'
abbr --add gst 'git status'
abbr --add gp 'git push'
abbr --add gl 'git pull'

# Docker
abbr --add d 'docker'
abbr --add dc 'docker-compose'
abbr --add dps 'docker ps'

# System
abbr --add ll 'ls -la'
abbr --add la 'ls -A'
abbr --add l 'ls -CF'
```

## Prompt Customization

### Built-in Prompts

```fish
# View available prompts
$ fish_config prompt

# Or choose from command line
$ fish_config prompt choose arrow
$ fish_config prompt choose informative
$ fish_config prompt choose robbyrussell
```

### Custom Prompt

Create `~/.config/fish/functions/fish_prompt.fish`:

**Simple Prompt**:
```fish
function fish_prompt
    echo (set_color cyan)(prompt_pwd) (set_color normal)'❯ '
end
```

**Git-Aware Prompt**:
```fish
function fish_prompt
    set -l last_status $status
    
    # User@Host
    echo -n (set_color brblue)(whoami)@(hostname)
    
    # Current directory
    echo -n (set_color cyan)' '(prompt_pwd)
    
    # Git branch
    if git rev-parse --git-dir >/dev/null 2>&1
        echo -n (set_color yellow)' '(git branch 2>/dev/null | sed -n '/\* /s///p')
    end
    
    # Status indicator
    if test $last_status -ne 0
        echo -n (set_color red)' ✘'
    end
    
    echo -n (set_color normal)' ❯ '
end
```

### Right Prompt

Create `~/.config/fish/functions/fish_right_prompt.fish`:

```fish
function fish_right_prompt
    # Show command duration if > 5 seconds
    set -l duration $CMD_DURATION
    if test $duration -gt 5000
        set duration (math $duration / 1000)
        echo (set_color yellow)$duration's'(set_color normal)
    end
end
```

### Prompt Utilities

```fish
# Show current directory (shortened)
prompt_pwd

# Show hostname
prompt_hostname

# Get git branch
fish_git_prompt

# Show command status
fish_status_prompt
```

## Key Bindings

### Viewing Key Bindings

```fish
# List all bindings
$ bind

# Search bindings
$ bind | grep history

# Show binding for specific key
$ bind \cr                  # Shows Ctrl+R binding
```

### Creating Key Bindings

```fish
# Bind key to command
$ bind \cg 'git status'

# Bind with function
function my_function
    echo "Custom function"
    commandline -f repaint
end
bind \cx my_function
```

### Default Key Bindings

**Navigation**:
```
Ctrl+A          # Move to beginning of line
Ctrl+E          # Move to end of line
Alt+←/→         # Move by word
Ctrl+K          # Kill to end of line
Ctrl+U          # Kill to beginning of line
```

**History**:
```
↑/↓             # Previous/next command
Ctrl+R          # History search
Alt+↑/↓         # Previous/next token
Alt+.           # Insert last argument
```

**Editing**:
```
Alt+E           # Edit in $EDITOR
Alt+V           # Edit in $VISUAL
Ctrl+C          # Cancel command
Ctrl+D          # Delete character or exit
```

**Completion**:
```
Tab             # Complete or show completions
Shift+Tab       # Previous completion
Ctrl+→          # Accept one word of autosuggestion
→               # Accept full autosuggestion
```

## Scripting Basics

### Fish Scripts

**Create Script** (`script.fish`):
```fish
#!/usr/bin/env fish

# Comments start with #

echo "Hello from Fish!"

# Variables
set name "World"
echo "Hello, $name!"

# Command substitution
set current_time (date)
echo "Current time: $current_time"
```

**Make Executable**:
```bash
chmod +x script.fish
./script.fish
```

### Control Structures

**If Statements**:
```fish
if test $status -eq 0
    echo "Success"
else if test $status -eq 1
    echo "Failed"
else
    echo "Unknown status"
end

# Short form
test -f file.txt; and echo "File exists"
test -d /tmp; or echo "Not a directory"
```

**Switch Statements**:
```fish
switch $argv[1]
    case start
        echo "Starting..."
    case stop
        echo "Stopping..."
    case restart
        echo "Restarting..."
    case '*'
        echo "Unknown command"
end
```

**Loops**:
```fish
# For loop
for file in *.txt
    echo "Processing $file"
end

# While loop
set i 0
while test $i -lt 10
    echo $i
    set i (math $i + 1)
end

# Loop over command output
for line in (cat file.txt)
    echo "Line: $line"
end
```

### Test Conditions

```fish
test -f file              # File exists
test -d dir               # Directory exists
test -x file              # File is executable
test -z "$var"            # String is empty
test -n "$var"            # String is not empty
test "$a" = "$b"          # Strings equal
test "$a" != "$b"         # Strings not equal
test $a -eq $b            # Numbers equal
test $a -ne $b            # Numbers not equal
test $a -gt $b            # Greater than
test $a -lt $b            # Less than
```

### Input/Output

**Reading Input**:
```fish
read -P "Enter name: " name
echo "Hello, $name!"

# Read line by line
while read line
    echo "Line: $line"
end < file.txt
```

**Redirecting Output**:
```fish
echo "text" > file.txt          # Overwrite
echo "text" >> file.txt         # Append
command 2> error.log            # Redirect stderr
command > output.txt 2>&1       # Redirect both
command &> all.log              # Redirect both (shorthand)
```

## Comparison with Other Shells

### Fish vs Bash

| Feature | Fish | Bash |
|---------|------|------|
| **Syntax highlighting** | Built-in | Requires plugin |
| **Autosuggestions** | Built-in | Requires plugin |
| **Tab completion** | Works by default | Needs configuration |
| **Configuration** | Web UI + config files | Config files only |
| **Scripting syntax** | Modern, clean | POSIX, cryptic |
| **POSIX compatible** | No | Yes |
| **Portability** | Less portable | Universal |
| **Variable syntax** | `set VAR value` | `VAR=value` |
| **Export** | `set -x` | `export` |
| **Arrays** | Native lists | Needs special syntax |

**Example Comparison**:

```fish
# Fish
set name "John"
set -x PATH $PATH /new/path
for file in *.txt
    echo $file
end

# Bash
name="John"
export PATH="$PATH:/new/path"
for file in *.txt; do
    echo "$file"
done
```

### Fish vs Zsh

| Feature | Fish | Zsh |
|---------|------|-----|
| **Out-of-box experience** | Excellent | Needs configuration |
| **Syntax highlighting** | Built-in | Plugin (zsh-syntax-highlighting) |
| **Autosuggestions** | Built-in | Plugin (zsh-autosuggestions) |
| **POSIX compatible** | No | Yes |
| **Plugin ecosystem** | Smaller | Huge (Oh My Zsh) |
| **Learning curve** | Gentle | Steeper |
| **Scripting** | Non-POSIX | POSIX + extensions |

**When to Choose Fish**:
- You want great defaults without configuration
- Interactive use is your priority
- You're okay with non-POSIX scripting syntax

**When to Choose Zsh**:
- You need POSIX compatibility
- You want maximum customization (Oh My Zsh)
- You write portable scripts

### Bash Script Compatibility

Fish is **not** bash-compatible. Common issues:

**Variable Assignment**:
```fish
# Bash
VAR=value

# Fish
set VAR value
```

**Export**:
```fish
# Bash
export PATH=$PATH:/new/path

# Fish
set -x PATH $PATH /new/path
# Or
fish_add_path /new/path
```

**Conditionals**:
```fish
# Bash
if [ "$x" = "y" ]; then
    echo "equal"
fi

# Fish
if test "$x" = "y"
    echo "equal"
end
```

**Command Substitution**:
```fish
# Bash
result=$(command)
result=`command`

# Fish
set result (command)
```

**Running Bash Scripts in Fish**:
```fish
# Execute with bash
bash script.sh

# Source bash script (with workarounds)
bass source script.sh  # Requires bass plugin
```

## Plugins and Tools

### Fisher (Plugin Manager)

**Install Fisher**:
```fish
curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher
```

**Using Fisher**:
```fish
# Install plugin
fisher install jorgebucaran/nvm.fish

# Update all plugins
fisher update

# Remove plugin
fisher remove jorgebucaran/nvm.fish

# List plugins
fisher list
```

### Useful Plugins

**nvm.fish** (Node Version Manager):
```fish
fisher install jorgebucaran/nvm.fish
nvm install 18
nvm use 18
```

**z** (Directory Jumper):
```fish
fisher install jethrokuan/z
# Use: z frequently_used_dir
```

**fzf.fish** (Fuzzy Finder):
```fish
fisher install PatrickF1/fzf.fish
# Ctrl+Alt+F: Find file
# Ctrl+Alt+L: Search history
# Ctrl+Alt+V: Search variables
```

**bass** (Run Bash Scripts):
```fish
fisher install edc/bass
bass source ~/.bashrc
```

**tide** (Prompt):
```fish
fisher install IlanCosman/tide
tide configure
```

### Starship Prompt

**Install Starship**:
```bash
curl -sS https://starship.rs/install.sh | sh
```

**Configure Fish to Use Starship** (`~/.config/fish/config.fish`):
```fish
starship init fish | source
```

**Configure Starship** (`~/.config/starship.toml`):
```toml
[character]
success_symbol = "[➜](bold green)"
error_symbol = "[✗](bold red)"

[directory]
truncation_length = 3
truncate_to_repo = true
```

## Best Practices

### Interactive Usage

1. **Use Abbreviations Over Aliases**:
   - You see what runs
   - Easier to remember
   - Can be edited before execution

2. **Leverage Autosuggestions**:
   - Press `→` to accept
   - Press `Ctrl+→` to accept one word
   - Train your muscle memory

3. **Customize Your Prompt**:
   - Show git status
   - Show exit status
   - Keep it clean and readable

4. **Use Universal Variables for Persistence**:
   ```fish
   set -U fish_greeting ""
   set -U EDITOR vim
   ```

5. **Organize Functions**:
   - One file per function in `~/.config/fish/functions/`
   - Use `funced` and `funcsave`

### Scripting

1. **Use Bash for Portable Scripts**:
   - Fish scripts won't run on other systems
   - Use `#!/usr/bin/env bash` for portability

2. **Use Fish for Personal Scripts**:
   - Clearer syntax than bash
   - Better error handling
   - Modern language features

3. **Prefer Functions Over Scripts**:
   - Functions are auto-loaded
   - Easier to edit with `funced`
   - Can be shared easily

4. **Use `argparse` for Complex Arguments**:
   ```fish
   function mycommand
       argparse 'h/help' 'v/verbose' -- $argv
       or return
       
       # Handle flags
   end
   ```

### Configuration

1. **Keep config.fish Minimal**:
   - Only essentials
   - Use functions for complex logic
   - Document your choices

2. **Use Universal Variables Wisely**:
   - Good for: colors, editor, paths
   - Bad for: machine-specific settings

3. **Version Control Your Config**:
   ```bash
   cd ~/.config/fish
   git init
   git add config.fish functions/
   git commit -m "Initial fish config"
   ```

## Troubleshooting

### Common Issues

**Slow Startup**:
```fish
# Profile startup time
fish --profile prompt.prof -c exit
cat prompt.prof

# Common culprits:
# - Too many items in $PATH
# - Slow functions in config.fish
# - Git operations in prompt
```

**PATH Issues**:
```fish
# View PATH
echo $PATH

# Reset PATH
set -e PATH
set -U fish_user_paths /usr/local/bin /usr/bin /bin

# Add path correctly
fish_add_path /new/path
```

**Completions Not Working**:
```fish
# Rebuild completions
fish_update_completions

# Clear completion cache
rm ~/.local/share/fish/generated_completions/*
```

**Color Issues**:
```fish
# Check terminal supports 256 colors
echo $TERM

# Reset colors
set -e fish_color_{command,param,error}
```

### Getting Help

```fish
# Fish documentation
man fish

# Command help
man command_name

# Fish tutorial
fish_tutorial

# Online docs
open https://fishshell.com/docs/current/

# Community
# - GitHub: https://github.com/fish-shell/fish-shell
# - Gitter: https://gitter.im/fish-shell/fish-shell
```

## Migrating from Bash/Zsh

### Converting Bash Scripts

**Variables**:
```fish
# Bash: VAR=value
set VAR value

# Bash: export VAR=value
set -x VAR value
```

**Conditionals**:
```fish
# Bash: if [ -f file ]; then ... fi
if test -f file
    # ...
end
```

**Loops**:
```fish
# Bash: for i in $(seq 1 10); do ... done
for i in (seq 1 10)
    # ...
end
```

**Functions**:
```fish
# Bash: function name() { ... }
function name
    # ...
end
```

### Aliases to Abbreviations

```fish
# Convert bash aliases
# alias ll='ls -la'
abbr --add ll 'ls -la'

# Bulk conversion script
for line in (cat ~/.bash_aliases | grep alias)
    # Parse and create abbreviations
end
```

### Environment Setup

```fish
# Port .bashrc/.zshrc
# 1. Convert exports
# 2. Convert aliases to abbreviations
# 3. Rewrite functions in Fish syntax
# 4. Test each piece

# Example:
# bash: export EDITOR=vim
set -Ux EDITOR vim

# bash: alias g=git
abbr --add g git
```

## Resources

### Official Documentation

- [Fish Shell Website](https://fishshell.com/)
- [Official Documentation](https://fishshell.com/docs/current/)
- [Tutorial](https://fishshell.com/docs/current/tutorial.html)
- [Fish for bash users](https://fishshell.com/docs/current/fish_for_bash_users.html)
- [GitHub Repository](https://github.com/fish-shell/fish-shell)

### Community Resources

**Plugin Repositories**:
- [Fisher Plugins](https://github.com/jorgebucaran/fisher)
- [Awesome Fish](https://github.com/jorgebucaran/awsm.fish)
- [Fish Plugins](https://github.com/topics/fish-plugin)

**Tutorials and Guides**:
- [Fish Shell Cookbook](https://github.com/jorgebucaran/cookbook.fish)
- [Learn Fish in Y Minutes](https://learnxinyminutes.com/docs/fish/)

**Themes and Prompts**:
- [Oh My Fish Themes](https://github.com/oh-my-fish/oh-my-fish/blob/master/docs/Themes.md)
- [Tide Prompt](https://github.com/IlanCosman/tide)
- [Starship Prompt](https://starship.rs/)

### Tools and Integrations

**Editors**:
- [fish-shell/fish-shell VSCode](https://marketplace.visualstudio.com/items?itemName=bmalehorn.vscode-fish)
- [vim-fish](https://github.com/dag/vim-fish)
- [emacs-fish](https://github.com/wwwjfy/emacs-fish)

**Useful Tools**:
- [fzf](https://github.com/junegunn/fzf) - Fuzzy finder
- [exa](https://github.com/ogham/exa) - Modern ls replacement
- [bat](https://github.com/sharkdp/bat) - Better cat
- [fd](https://github.com/sharkdp/fd) - Better find
- [ripgrep](https://github.com/BurntSushi/ripgrep) - Better grep

### Books and Articles

- Fish Shell Official Tutorial
- "Modern Shell Environments" blog posts
- Stack Overflow [fish tag](https://stackoverflow.com/questions/tagged/fish)

## Conclusion

### Why Fish is Great

- **Zero configuration**: Works beautifully out of the box
- **User-friendly**: Designed for humans, not just scripts
- **Modern**: Built with today's needs in mind
- **Discoverable**: Features are easy to find and use
- **Fun**: Makes terminal work enjoyable

### When to Use What

**Use Fish for**:
- Daily interactive terminal work
- Personal scripts and automation
- Learning shell usage
- Pleasant terminal experience

**Use Bash for**:
- Portable scripts
- CI/CD pipelines
- System scripts
- POSIX requirements

**Use Both**:
- Fish as your interactive shell
- Bash for scripting
- Best of both worlds

### Getting Started

```fish
# Install Fish
brew install fish  # or your package manager

# Try it out
fish

# Like it? Set as default
chsh -s (which fish)

# Configure
fish_config

# Install a plugin manager
curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher

# Enjoy!
```

Welcome to Fish! 🐟
