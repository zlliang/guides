# Makefile

A comprehensive guide to understanding and mastering Makefiles — the build automation tool used across Unix-like systems for managing project compilation, testing, deployment, and more.

## Quick Reference

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

### What is a Makefile?

A Makefile is a special file containing a set of directives (rules) that Make uses to build and manage projects. It defines:

- **Targets**: What to build (executables, object files, or actions)
- **Dependencies**: What files or targets are required
- **Recipes**: Commands to execute to build the target

### Why Use Make?

- **Incremental builds**: Only recompiles changed files
- **Dependency management**: Automatically tracks file relationships
- **Automation**: Reduces repetitive manual commands
- **Portability**: Works across Unix-like systems
- **Simplicity**: Declarative syntax for complex workflows

## Basic Syntax

### The Basic Rule

Every Makefile consists of rules with this structure:

```makefile
target: dependencies
	recipe
```

- **Target**: The file to be created or the name of an action
- **Dependencies**: Files that must exist before the target can be built
- **Recipe**: Shell commands to create the target (must be indented with a TAB character)

### Simple Example

```makefile
hello: hello.c
	gcc -o hello hello.c
```

This rule says: "To build `hello`, we need `hello.c`, and we build it by running `gcc -o hello hello.c`."

### Multiple Rules

```makefile
program: main.o utils.o
	gcc -o program main.o utils.o

main.o: main.c
	gcc -c main.c

utils.o: utils.c
	gcc -c utils.c
```

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

The `.PHONY` declaration prevents conflicts with files named `clean`, `all`, or `install`.

## Variables

### Defining Variables

```makefile
CC = gcc
CFLAGS = -Wall -O2
OBJECTS = main.o utils.o

program: $(OBJECTS)
	$(CC) $(CFLAGS) -o program $(OBJECTS)
```

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

### Parallel Execution

```bash
make -j4        # Run up to 4 jobs in parallel
make -j         # Run as many jobs as possible
```

Structure your Makefile to support parallel builds by correctly specifying dependencies.

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

## Debugging Makefiles

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
make --debug
```

### Trace Execution

```bash
make -p         # Print database (all rules and variables)
```

## References & Resources

### Official Documentation

- [GNU Make Manual](https://www.gnu.org/software/make/manual/) - The complete reference
- [POSIX Make Specification](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/make.html) - Standard specification

### Tutorials and Guides

- [Make Tutorial by Chase Lambert](https://makefiletutorial.com/) - Interactive tutorial
- [Recursive Make Considered Harmful](https://aegis.sourceforge.net/auug97.pdf) - Classic paper on Make anti-patterns

### Books

- *Managing Projects with GNU Make* by Robert Mecklenburg (O'Reilly)
- *The GNU Make Book* by John Graham-Cumming

### Tools

- [remake](http://bashdb.sourceforge.net/remake/) - Enhanced Make with debugging support
- [CMake](https://cmake.org/) - Modern build system generator
- [Ninja](https://ninja-build.org/) - Fast build system (Make alternative)

### Related Technologies

- [Autotools](https://www.gnu.org/software/automake/) - GNU Build System
- [Meson](https://mesonbuild.com/) - Modern build system
- [Bazel](https://bazel.build/) - Google's build system
