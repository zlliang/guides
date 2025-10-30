# Make & Makefile

A comprehensive guide to understanding and mastering Make — the build automation tool — and Makefiles, the configuration files that drive it. This guide covers both the `make` command and Makefile syntax used across Unix-like systems for managing project compilation, testing, deployment, and more.

## Quick Reference

### Make Command

```bash
make                    # Build default target
make target             # Build specific target
make target1 target2    # Build multiple targets
make -n                 # Dry run (show commands without executing)
make -j4                # Run 4 parallel jobs
make -B                 # Force rebuild all targets
make -C dir             # Run make in directory
make VAR=value          # Override variable
make -f custom.mk       # Use custom makefile
make --version          # Show version
```

### Makefile Syntax

```makefile
# Basic rule syntax
target: dependencies
	command

# Variables
VAR = value
VAR := value        # Simple expansion
VAR ?= value        # Conditional assignment
VAR += value        # Append

# Automatic variables
$@    # Target name
$<    # First dependency
$^    # All dependencies
$?    # Dependencies newer than target
$*    # Stem of pattern match

# Pattern rules
%.o: %.c
	$(CC) -c $< -o $@

# Functions
$(wildcard *.c)           # Find files
$(patsubst %.c,%.o,...)   # Pattern substitution
$(shell command)          # Execute shell command
$(foreach var,list,text)  # Loop over list

# Phony targets
.PHONY: clean all test
```

## Introduction

### What is Make?

Make is a build automation tool that automatically determines which pieces of a large program need to be recompiled and issues commands to recompile them. Created by Stuart Feldman in 1976 at Bell Labs, Make remains one of the most widely used build tools, particularly in C/C++ projects and system administration.

The core concept behind Make is **incremental building**: it compares timestamps of source files and their outputs to determine what needs to be rebuilt, saving time by only recompiling what has changed.

### What is a Makefile?

A Makefile is a special file containing a set of directives (rules) that Make uses to build and manage projects. It defines:

- **Targets**: What to build (executables, object files, or actions)
- **Dependencies**: What files or targets are required
- **Recipes**: Commands to execute to build the target

By default, Make looks for files named `Makefile`, `makefile`, or `GNUmakefile` in the current directory.

### Why Use Make?

- **Incremental builds**: Only recompiles changed files
- **Dependency management**: Automatically tracks file relationships
- **Automation**: Reduces repetitive manual commands
- **Portability**: Works across Unix-like systems
- **Simplicity**: Declarative syntax for complex workflows
- **Universal**: Supported by most development tools and IDEs

## The Make Command

### Basic Usage

The simplest invocation of Make:

```bash
make
```

This reads the Makefile in the current directory and builds the first target (the default goal).

### Building Specific Targets

```bash
make clean          # Build the 'clean' target
make test           # Build the 'test' target
make install        # Build the 'install' target
```

### Building Multiple Targets

```bash
make clean all      # Clean, then build all
make test install   # Run tests, then install
```

### Common Command-Line Options

#### Dry Run

Preview what Make would do without actually executing:

```bash
make -n
make --dry-run
make --just-print
```

#### Parallel Execution

Run multiple jobs in parallel to speed up builds:

```bash
make -j4            # Run up to 4 jobs in parallel
make -j             # Use as many jobs as possible
make -j$(nproc)     # Use number of CPU cores
```

#### Force Rebuild

Rebuild all targets regardless of timestamps:

```bash
make -B
make --always-make
```

#### Change Directory

Run Make in a different directory:

```bash
make -C src/        # Equivalent to: cd src && make
make -C ../project
```

#### Specify Makefile

Use a custom-named Makefile:

```bash
make -f build.mk
make --file=custom.makefile
```

#### Override Variables

Set or override Makefile variables from the command line:

```bash
make CC=clang
make CFLAGS="-Wall -O3"
make DEBUG=1 test
```

#### Silent Mode

Suppress command echoing:

```bash
make -s
make --silent
```

#### Ignore Errors

Continue even if commands fail:

```bash
make -i
make --ignore-errors
```

#### Print Variables

Debug by printing Make's internal database:

```bash
make -p              # Print all rules and variables
make --print-data-base
```

#### Keep Going

Continue building other targets even if one fails:

```bash
make -k
make --keep-going
```

### Environment Variables

Make respects several environment variables:

- `MAKE`: The name of the Make executable (for recursive Make)
- `MAKEFLAGS`: Default flags for Make
- `MAKEFILE_LIST`: List of parsed Makefiles
- `MAKELEVEL`: Recursion depth (0 for top-level Make)
- `CC`, `CXX`, `CFLAGS`, etc.: Compiler and flag defaults

### Exit Status

Make returns:
- `0`: Success (all targets up to date or successfully built)
- `1`: Error occurred during build
- `2`: Command-line usage error

## Makefile Syntax

### The Basic Rule

Every Makefile consists of rules with this structure:

```makefile
target: dependencies
	recipe
```

- **Target**: The file to be created or the name of an action
- **Dependencies**: Files that must exist before the target can be built
- **Recipe**: Shell commands to create the target (must be indented with a TAB character)

**Important**: Recipes must be indented with TAB characters, not spaces. This is a common source of errors.

### Simple Example

```makefile
hello: hello.c
	gcc -o hello hello.c
```

This rule says: "To build `hello`, we need `hello.c`, and we build it by running `gcc -o hello hello.c`."

Make will:
1. Check if `hello` exists
2. Check if `hello.c` is newer than `hello`
3. If `hello` doesn't exist or `hello.c` is newer, run the recipe

### Multiple Rules

```makefile
program: main.o utils.o
	gcc -o program main.o utils.o

main.o: main.c
	gcc -c main.c

utils.o: utils.c
	gcc -c utils.c
```

Make builds the dependency tree and executes rules in the correct order.

### The Default Target

Make builds the first target it encounters. To build a different target, specify it: `make utils.o`

### Phony Targets

Targets that don't represent files:

```makefile
.PHONY: clean all install

clean:
	rm -f *.o program

all: program

install: program
	cp program /usr/local/bin/
```

The `.PHONY` declaration prevents conflicts with files named `clean`, `all`, or `install`, and ensures Make always runs these targets regardless of file timestamps.

## Variables

### Defining Variables

```makefile
CC = gcc
CFLAGS = -Wall -O2
OBJECTS = main.o utils.o

program: $(OBJECTS)
	$(CC) $(CFLAGS) -o program $(OBJECTS)
```

Variables are referenced with `$(VAR)` or `${VAR}`.

### Variable Assignment Types

```makefile
# Recursive (lazy) expansion - evaluated when used
VAR = $(OTHER_VAR)

# Simple expansion - evaluated immediately
VAR := $(shell date)

# Conditional assignment - only if undefined
VAR ?= default_value

# Append
CFLAGS += -g
```

### Built-in Variables

Common predefined variables:

- `CC`: C compiler (default: `cc`)
- `CXX`: C++ compiler (default: `g++`)
- `CFLAGS`: C compiler flags
- `CXXFLAGS`: C++ compiler flags
- `LDFLAGS`: Linker flags
- `LDLIBS`: Libraries to link
- `AR`: Archive command (default: `ar`)
- `RM`: Remove command (default: `rm -f`)

### Using Variables

```makefile
CC = gcc
CFLAGS = -Wall -Werror
TARGET = myprogram

$(TARGET): main.o
	$(CC) $(CFLAGS) -o $(TARGET) main.o
```

## Automatic Variables

Automatic variables are set by Make for each rule and provide information about the current context.

### Core Automatic Variables

```makefile
program: main.o utils.o config.o
	@echo "Target: $@"           # program
	@echo "First: $<"            # main.o
	@echo "All deps: $^"         # main.o utils.o config.o
	@echo "Newer deps: $?"       # Dependencies newer than target
	gcc -o $@ $^
```

### Pattern Rule Automatic Variables

```makefile
%.o: %.c
	@echo "Stem: $*"             # Filename without extension
	gcc -c $< -o $@
```

### Directory and File Components

```makefile
$(@D)    # Directory part of target
$(@F)    # File part of target
$(<D)    # Directory part of first dependency
$(<F)    # File part of first dependency
```

## Pattern Rules

Pattern rules allow you to define generic build instructions using `%` as a wildcard.

### Basic Pattern Rule

```makefile
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@
```

This rule says: "Any `.o` file can be built from a corresponding `.c` file."

### Multiple Pattern Rules

```makefile
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

%.o: %.cpp
	$(CXX) $(CXXFLAGS) -c $< -o $@

%: %.o
	$(CC) -o $@ $^
```

### Static Pattern Rules

Apply a pattern rule to a specific list of targets:

```makefile
OBJECTS = main.o utils.o config.o

$(OBJECTS): %.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@
```

## Functions

Make provides built-in functions for text manipulation and file operations.

### Wildcard Function

```makefile
SOURCES = $(wildcard *.c)
OBJECTS = $(wildcard *.o)
```

### Pattern Substitution

```makefile
SOURCES = main.c utils.c config.c
OBJECTS = $(SOURCES:.c=.o)
# or
OBJECTS = $(patsubst %.c,%.o,$(SOURCES))
```

### Shell Function

```makefile
TODAY = $(shell date +%Y-%m-%d)
FILES = $(shell find . -name "*.c")
NPROC = $(shell nproc)
```

### Foreach Function

```makefile
DIRS = src test lib
ALL_SOURCES = $(foreach dir,$(DIRS),$(wildcard $(dir)/*.c))
```

### If Function

```makefile
DEBUG ?= 0
CFLAGS = -Wall $(if $(filter 1,$(DEBUG)),-g -O0,-O2)
```

### Additional Useful Functions

```makefile
# String substitution
$(subst old,new,text)

# Filter
$(filter %.c %.h,$(FILES))

# Filter out
$(filter-out test%,$(FILES))

# Sort (and remove duplicates)
$(sort list)

# Directory/basename
$(dir src/main.c)      # src/
$(notdir src/main.c)   # main.c
$(basename main.c)     # main
$(suffix main.c)       # .c

# Add prefix/suffix
$(addprefix prefix_,list)
$(addsuffix _suffix,list)
```

## Advanced Features

### Conditional Directives

```makefile
ifeq ($(CC),gcc)
    CFLAGS += -Wextra
endif

ifdef DEBUG
    CFLAGS += -g -DDEBUG
else
    CFLAGS += -O2
endif

ifndef VERSION
    VERSION = 1.0
endif

ifneq ($(OS),Windows)
    RM = rm -f
endif
```

### Include Directives

```makefile
include config.mk
-include optional.mk    # Don't error if file doesn't exist
```

### Generating Dependencies Automatically

```makefile
DEPS = $(OBJECTS:.o=.d)

-include $(DEPS)

%.o: %.c
	$(CC) $(CFLAGS) -MMD -c $< -o $@
```

The `-MMD` flag generates `.d` files containing dependency information.

### Multi-line Variables

```makefile
define HELP_TEXT
Usage:
  make build    - Build the project
  make clean    - Remove build artifacts
  make test     - Run tests
endef

export HELP_TEXT

help:
	@echo "$$HELP_TEXT"
```

### Target-specific Variables

```makefile
program: CFLAGS += -O2
debug: CFLAGS += -g -O0

program: main.o utils.o
	$(CC) $(CFLAGS) -o $@ $^

debug: main.o utils.o
	$(CC) $(CFLAGS) -o program $^
```

### Order-only Prerequisites

```makefile
BUILD_DIR = build

$(BUILD_DIR)/program: main.o | $(BUILD_DIR)
	$(CC) -o $@ $<

$(BUILD_DIR):
	mkdir -p $@
```

The `|` separator indicates order-only prerequisites — they must exist but their timestamps don't trigger rebuilds.

### Recursion

```makefile
SUBDIRS = src test lib

all:
	for dir in $(SUBDIRS); do \
		$(MAKE) -C $$dir; \
	done

# Or better:
all:
	@for dir in $(SUBDIRS); do \
		$(MAKE) -C $$dir || exit 1; \
	done
```

### Recipe Prefixes

```makefile
target:
	@echo "Silent - not echoed"        # @ suppresses echoing
	-rm nonexistent.txt                # - ignores errors
	+echo "Always executed"            # + always executes (even with -n)
```

## Best Practices

### 1. Use `.PHONY` for Non-file Targets

```makefile
.PHONY: clean all test install
```

### 2. Define Variables at the Top

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -std=c11
LDFLAGS = -lm
TARGET = myprogram
```

### 3. Use Pattern Rules

Avoid repetition:

```makefile
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@
```

### 4. Leverage Automatic Variables

```makefile
program: main.o utils.o
	$(CC) -o $@ $^
```

### 5. Support Customization

```makefile
CC ?= gcc
PREFIX ?= /usr/local
```

Users can override: `make CC=clang PREFIX=/opt/local`

### 6. Add a Help Target

```makefile
.PHONY: help
help:
	@echo "Available targets:"
	@echo "  make build  - Build the project"
	@echo "  make clean  - Clean build artifacts"
	@echo "  make test   - Run tests"
```

Make it default:

```makefile
.DEFAULT_GOAL := help
```

### 7. Use `@` to Suppress Echo

```makefile
clean:
	@rm -f *.o program
	@echo "Cleaned!"
```

### 8. Structure for Clarity

```makefile
# Variables
CC = gcc
CFLAGS = -Wall

# Object files
OBJECTS = main.o utils.o

# Main target
all: program

# Build rules
program: $(OBJECTS)
	$(CC) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

# Utility targets
.PHONY: clean
clean:
	rm -f $(OBJECTS) program
```

### 9. Handle Dependencies

Use compiler-generated dependencies:

```makefile
DEPS = $(OBJECTS:.o=.d)
-include $(DEPS)

%.o: %.c
	$(CC) $(CFLAGS) -MMD -MP -c $< -o $@
```

### 10. Create Build Directories

```makefile
BUILD_DIR = build
OBJECTS = $(addprefix $(BUILD_DIR)/,main.o utils.o)

$(BUILD_DIR)/%.o: %.c | $(BUILD_DIR)
	$(CC) $(CFLAGS) -c $< -o $@

$(BUILD_DIR):
	mkdir -p $@
```

### 11. Use Parallel Builds Wisely

Ensure your Makefile is parallel-safe by correctly specifying all dependencies.

### 12. Avoid Recursive Make When Possible

Consider non-recursive Make patterns for better dependency tracking and parallel builds. See "Recursive Make Considered Harmful" in references.

## Common Patterns

### C/C++ Project Template

```makefile
# Compiler and flags
CC = gcc
CFLAGS = -Wall -Wextra -std=c11 -O2
LDFLAGS = -lm

# Directories
SRC_DIR = src
BUILD_DIR = build
BIN_DIR = bin

# Files
SOURCES = $(wildcard $(SRC_DIR)/*.c)
OBJECTS = $(SOURCES:$(SRC_DIR)/%.c=$(BUILD_DIR)/%.o)
DEPS = $(OBJECTS:.o=.d)
TARGET = $(BIN_DIR)/program

# Main target
all: $(TARGET)

# Link
$(TARGET): $(OBJECTS) | $(BIN_DIR)
	$(CC) $(OBJECTS) -o $@ $(LDFLAGS)

# Compile
$(BUILD_DIR)/%.o: $(SRC_DIR)/%.c | $(BUILD_DIR)
	$(CC) $(CFLAGS) -MMD -MP -c $< -o $@

# Create directories
$(BUILD_DIR) $(BIN_DIR):
	mkdir -p $@

# Include dependencies
-include $(DEPS)

# Clean
.PHONY: clean
clean:
	rm -rf $(BUILD_DIR) $(BIN_DIR)

# Install
.PHONY: install
install: $(TARGET)
	install -m 755 $(TARGET) /usr/local/bin/
```

### Testing Framework

```makefile
TEST_DIR = test
TEST_SOURCES = $(wildcard $(TEST_DIR)/*.c)
TEST_BINS = $(TEST_SOURCES:$(TEST_DIR)/%.c=$(BUILD_DIR)/test_%)

.PHONY: test
test: $(TEST_BINS)
	@for test in $(TEST_BINS); do \
		echo "Running $$test..."; \
		$$test || exit 1; \
	done
	@echo "All tests passed!"

$(BUILD_DIR)/test_%: $(TEST_DIR)/%.c $(OBJECTS)
	$(CC) $(CFLAGS) $< $(filter-out $(BUILD_DIR)/main.o,$(OBJECTS)) -o $@ $(LDFLAGS)
```

### Multi-directory Project

```makefile
SUBDIRS = lib src test

.PHONY: all $(SUBDIRS)

all: $(SUBDIRS)

$(SUBDIRS):
	$(MAKE) -C $@

src: lib
test: lib src

.PHONY: clean
clean:
	for dir in $(SUBDIRS); do $(MAKE) -C $$dir clean; done
```

## Debugging Make and Makefiles

### Print Variable Values

```makefile
$(info VAR = $(VAR))
$(warning This is a warning)
$(error This is an error)

print-%:
	@echo $* = $($*)
```

Usage: `make print-OBJECTS`

### Dry Run

```bash
make -n         # Print commands without executing
make --dry-run
```

### Debug Output

```bash
make -d         # Print debugging information
make --debug=b  # Basic debugging
make --debug=v  # Verbose debugging
make --debug=a  # All debugging info
```

### Trace Execution

```bash
make -p         # Print database (all rules and variables)
make --trace    # Print recipe lines as they execute
```

### Common Errors

#### TAB vs Spaces Error

```
Makefile:3: *** missing separator.  Stop.
```

**Solution**: Ensure recipes use TAB characters, not spaces.

#### Circular Dependency

```
Makefile:5: Circular main.o <- main.o dependency dropped.
```

**Solution**: Review your dependency chain for loops.

#### No Rule to Make Target

```
make: *** No rule to make target 'file.c', needed by 'file.o'.  Stop.
```

**Solution**: Ensure the dependency file exists or the pattern rule is correct.

## References & Resources

### Official Documentation

- [GNU Make Manual](https://www.gnu.org/software/make/manual/) - The complete reference
- [POSIX Make Specification](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/make.html) - Standard specification
- [GNU Make GitHub Repository](https://github.com/mirror/make) - Source code

### Tutorials and Guides

- [Make Tutorial by Chase Lambert](https://makefiletutorial.com/) - Interactive tutorial
- [Recursive Make Considered Harmful](https://aegis.sourceforge.net/auug97.pdf) - Classic paper on Make anti-patterns
- [Managing Projects with GNU Make (Online)](https://www.oreilly.com/openbook/make3/book/) - Free online version

### Books

- *Managing Projects with GNU Make* by Robert Mecklenburg (O'Reilly)
- *The GNU Make Book* by John Graham-Cumming

### Tools

- [remake](http://bashdb.sourceforge.net/remake/) - Enhanced Make with debugging support
- [CMake](https://cmake.org/) - Modern build system generator
- [Ninja](https://ninja-build.org/) - Fast build system (Make alternative)
- [bear](https://github.com/rizsotto/Bear) - Generate compilation database from Make

### Related Technologies

- [Autotools](https://www.gnu.org/software/automake/) - GNU Build System
- [Meson](https://mesonbuild.com/) - Modern build system
- [Bazel](https://bazel.build/) - Google's build system
- [SCons](https://scons.org/) - Python-based build tool
