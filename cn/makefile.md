# Makefile

一份全面的指南，帮助你理解和掌握 Makefile——这一在类 Unix 系统中广泛使用的构建自动化工具，用于管理项目编译、测试、部署等任务。

## 快速参考

```makefile
# 基本规则语法
target: dependencies
	command

# 变量
VAR = value
VAR := value        # 简单展开
VAR ?= value        # 条件赋值
VAR += value        # 追加

# 自动变量
$@    # 目标名称
$<    # 第一个依赖项
$^    # 所有依赖项
$?    # 比目标更新的依赖项
$*    # 模式匹配的主干部分

# 模式规则
%.o: %.c
	$(CC) -c $< -o $@

# 函数
$(wildcard *.c)           # 查找文件
$(patsubst %.c,%.o,...)   # 模式替换
$(shell command)          # 执行 shell 命令
$(foreach var,list,text)  # 遍历列表

# 伪目标
.PHONY: clean all test
```

## 简介

### 什么是 Make？

Make 是一个构建自动化工具，它能自动判断大型程序中哪些部分需要重新编译，并发出命令来重新编译它们。Make 由 Stuart Feldman 于 1976 年在贝尔实验室创建，至今仍然是使用最广泛的构建工具之一，特别是在 C/C++ 项目和系统管理中。

### 什么是 Makefile？

Makefile 是一个特殊文件，包含一组指令（规则），Make 使用这些指令来构建和管理项目。它定义了：

- **目标（Target）**：要构建什么（可执行文件、目标文件或操作）
- **依赖项（Dependencies）**：需要哪些文件或目标
- **配方（Recipe）**：执行哪些命令来构建目标

### 为什么使用 Make？

- **增量构建**：只重新编译已更改的文件
- **依赖管理**：自动跟踪文件关系
- **自动化**：减少重复的手动命令
- **可移植性**：在类 Unix 系统中通用
- **简洁性**：用声明式语法处理复杂工作流

## 基本语法

### 基本规则

每个 Makefile 由具有以下结构的规则组成：

```makefile
target: dependencies
	recipe
```

- **目标（Target）**：要创建的文件或操作的名称
- **依赖项（Dependencies）**：构建目标之前必须存在的文件
- **配方（Recipe）**：创建目标的 shell 命令（必须用制表符 TAB 缩进）

### 简单示例

```makefile
hello: hello.c
	gcc -o hello hello.c
```

这条规则表示："要构建 `hello`，我们需要 `hello.c`，通过运行 `gcc -o hello hello.c` 来构建。"

### 多个规则

```makefile
program: main.o utils.o
	gcc -o program main.o utils.o

main.o: main.c
	gcc -c main.c

utils.o: utils.c
	gcc -c utils.c
```

### 默认目标

Make 会构建它遇到的第一个目标。要构建不同的目标，请指定它：`make utils.o`

### 伪目标

不代表文件的目标：

```makefile
.PHONY: clean all install

clean:
	rm -f *.o program

all: program

install: program
	cp program /usr/local/bin/
```

`.PHONY` 声明可防止与名为 `clean`、`all` 或 `install` 的文件冲突。

## 变量

### 定义变量

```makefile
CC = gcc
CFLAGS = -Wall -O2
OBJECTS = main.o utils.o

program: $(OBJECTS)
	$(CC) $(CFLAGS) -o program $(OBJECTS)
```

### 变量赋值类型

```makefile
# 递归（惰性）展开 - 使用时求值
VAR = $(OTHER_VAR)

# 简单展开 - 立即求值
VAR := $(shell date)

# 条件赋值 - 仅在未定义时赋值
VAR ?= default_value

# 追加
CFLAGS += -g
```

### 内置变量

常见的预定义变量：

- `CC`：C 编译器（默认：`cc`）
- `CXX`：C++ 编译器（默认：`g++`）
- `CFLAGS`：C 编译器标志
- `CXXFLAGS`：C++ 编译器标志
- `LDFLAGS`：链接器标志
- `RM`：删除命令（默认：`rm -f`）

### 使用变量

```makefile
CC = gcc
CFLAGS = -Wall -Werror
TARGET = myprogram

$(TARGET): main.o
	$(CC) $(CFLAGS) -o $(TARGET) main.o
```

## 自动变量

自动变量由 Make 为每条规则设置，并提供有关当前上下文的信息。

### 核心自动变量

```makefile
program: main.o utils.o config.o
	@echo "目标: $@"           # program
	@echo "第一个: $<"         # main.o
	@echo "所有依赖: $^"       # main.o utils.o config.o
	@echo "更新的依赖: $?"     # 比目标更新的依赖项
	gcc -o $@ $^
```

### 模式规则中的自动变量

```makefile
%.o: %.c
	@echo "主干: $*"          # 不含扩展名的文件名
	gcc -c $< -o $@
```

### 目录和文件组成部分

```makefile
$(@D)    # 目标的目录部分
$(@F)    # 目标的文件部分
$(<D)    # 第一个依赖项的目录部分
$(<F)    # 第一个依赖项的文件部分
```

## 模式规则

模式规则允许你使用 `%` 作为通配符定义通用的构建指令。

### 基本模式规则

```makefile
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@
```

这条规则表示："任何 `.o` 文件都可以从相应的 `.c` 文件构建。"

### 多个模式规则

```makefile
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

%.o: %.cpp
	$(CXX) $(CXXFLAGS) -c $< -o $@

%: %.o
	$(CC) -o $@ $^
```

### 静态模式规则

将模式规则应用于特定的目标列表：

```makefile
OBJECTS = main.o utils.o config.o

$(OBJECTS): %.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@
```

## 函数

Make 提供了用于文本处理和文件操作的内置函数。

### 通配符函数

```makefile
SOURCES = $(wildcard *.c)
OBJECTS = $(wildcard *.o)
```

### 模式替换

```makefile
SOURCES = main.c utils.c config.c
OBJECTS = $(SOURCES:.c=.o)
# 或
OBJECTS = $(patsubst %.c,%.o,$(SOURCES))
```

### Shell 函数

```makefile
TODAY = $(shell date +%Y-%m-%d)
FILES = $(shell find . -name "*.c")
```

### Foreach 函数

```makefile
DIRS = src test lib
ALL_SOURCES = $(foreach dir,$(DIRS),$(wildcard $(dir)/*.c))
```

### If 函数

```makefile
DEBUG ?= 0
CFLAGS = -Wall $(if $(filter 1,$(DEBUG)),-g -O0,-O2)
```

### 其他有用的函数

```makefile
# 字符串替换
$(subst old,new,text)

# 过滤
$(filter %.c %.h,$(FILES))

# 过滤排除
$(filter-out test%,$(FILES))

# 排序（并去重）
$(sort list)

# 目录名/基本名
$(dir src/main.c)      # src/
$(notdir src/main.c)   # main.c
$(basename main.c)     # main
$(suffix main.c)       # .c
```

## 高级特性

### 条件指令

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

### Include 指令

```makefile
include config.mk
-include optional.mk    # 如果文件不存在不报错
```

### 自动生成依赖项

```makefile
DEPS = $(OBJECTS:.o=.d)

-include $(DEPS)

%.o: %.c
	$(CC) $(CFLAGS) -MMD -c $< -o $@
```

`-MMD` 标志会生成包含依赖信息的 `.d` 文件。

### 多行变量

```makefile
define HELP_TEXT
使用方法:
  make build    - 构建项目
  make clean    - 删除构建产物
  make test     - 运行测试
endef

export HELP_TEXT

help:
	@echo "$$HELP_TEXT"
```

### 目标特定变量

```makefile
program: CFLAGS += -O2
debug: CFLAGS += -g -O0

program: main.o utils.o
	$(CC) $(CFLAGS) -o $@ $^

debug: main.o utils.o
	$(CC) $(CFLAGS) -o program $^
```

### 仅顺序的前置条件

```makefile
BUILD_DIR = build

$(BUILD_DIR)/program: main.o | $(BUILD_DIR)
	$(CC) -o $@ $<

$(BUILD_DIR):
	mkdir -p $@
```

`|` 分隔符表示仅顺序的前置条件——它们必须存在，但它们的时间戳不会触发重新构建。

### 递归

```makefile
SUBDIRS = src test lib

all:
	for dir in $(SUBDIRS); do \
		$(MAKE) -C $$dir; \
	done

# 或更好的方式:
all:
	@for dir in $(SUBDIRS); do \
		$(MAKE) -C $$dir || exit 1; \
	done
```

### 并行执行

```bash
make -j4        # 最多运行 4 个并行作业
make -j         # 运行尽可能多的作业
```

通过正确指定依赖项来构造 Makefile 以支持并行构建。

## 最佳实践

### 1. 对非文件目标使用 `.PHONY`

```makefile
.PHONY: clean all test install
```

### 2. 在顶部定义变量

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -std=c11
LDFLAGS = -lm
TARGET = myprogram
```

### 3. 使用模式规则

避免重复：

```makefile
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@
```

### 4. 利用自动变量

```makefile
program: main.o utils.o
	$(CC) -o $@ $^
```

### 5. 支持自定义

```makefile
CC ?= gcc
PREFIX ?= /usr/local
```

用户可以覆盖：`make CC=clang PREFIX=/opt/local`

### 6. 添加帮助目标

```makefile
.PHONY: help
help:
	@echo "可用的目标:"
	@echo "  make build  - 构建项目"
	@echo "  make clean  - 清理构建产物"
	@echo "  make test   - 运行测试"
```

将其设为默认：

```makefile
.DEFAULT_GOAL := help
```

### 7. 使用 `@` 抑制命令回显

```makefile
clean:
	@rm -f *.o program
	@echo "已清理！"
```

### 8. 结构清晰

```makefile
# 变量
CC = gcc
CFLAGS = -Wall

# 目标文件
OBJECTS = main.o utils.o

# 主目标
all: program

# 构建规则
program: $(OBJECTS)
	$(CC) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

# 工具目标
.PHONY: clean
clean:
	rm -f $(OBJECTS) program
```

### 9. 处理依赖项

使用编译器生成的依赖项：

```makefile
DEPS = $(OBJECTS:.o=.d)
-include $(DEPS)

%.o: %.c
	$(CC) $(CFLAGS) -MMD -MP -c $< -o $@
```

### 10. 创建构建目录

```makefile
BUILD_DIR = build
OBJECTS = $(addprefix $(BUILD_DIR)/,main.o utils.o)

$(BUILD_DIR)/%.o: %.c | $(BUILD_DIR)
	$(CC) $(CFLAGS) -c $< -o $@

$(BUILD_DIR):
	mkdir -p $@
```

## 常见模式

### C/C++ 项目模板

```makefile
# 编译器和标志
CC = gcc
CFLAGS = -Wall -Wextra -std=c11 -O2
LDFLAGS = -lm

# 目录
SRC_DIR = src
BUILD_DIR = build
BIN_DIR = bin

# 文件
SOURCES = $(wildcard $(SRC_DIR)/*.c)
OBJECTS = $(SOURCES:$(SRC_DIR)/%.c=$(BUILD_DIR)/%.o)
DEPS = $(OBJECTS:.o=.d)
TARGET = $(BIN_DIR)/program

# 主目标
all: $(TARGET)

# 链接
$(TARGET): $(OBJECTS) | $(BIN_DIR)
	$(CC) $(OBJECTS) -o $@ $(LDFLAGS)

# 编译
$(BUILD_DIR)/%.o: $(SRC_DIR)/%.c | $(BUILD_DIR)
	$(CC) $(CFLAGS) -MMD -MP -c $< -o $@

# 创建目录
$(BUILD_DIR) $(BIN_DIR):
	mkdir -p $@

# 包含依赖项
-include $(DEPS)

# 清理
.PHONY: clean
clean:
	rm -rf $(BUILD_DIR) $(BIN_DIR)

# 安装
.PHONY: install
install: $(TARGET)
	install -m 755 $(TARGET) /usr/local/bin/
```

### 测试框架

```makefile
TEST_DIR = test
TEST_SOURCES = $(wildcard $(TEST_DIR)/*.c)
TEST_BINS = $(TEST_SOURCES:$(TEST_DIR)/%.c=$(BUILD_DIR)/test_%)

.PHONY: test
test: $(TEST_BINS)
	@for test in $(TEST_BINS); do \
		echo "运行 $$test..."; \
		$$test || exit 1; \
	done
	@echo "所有测试通过！"

$(BUILD_DIR)/test_%: $(TEST_DIR)/%.c $(OBJECTS)
	$(CC) $(CFLAGS) $< $(filter-out $(BUILD_DIR)/main.o,$(OBJECTS)) -o $@ $(LDFLAGS)
```

### 多目录项目

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

## 调试 Makefile

### 打印变量值

```makefile
$(info VAR = $(VAR))
$(warning 这是一个警告)
$(error 这是一个错误)

print-%:
	@echo $* = $($*)
```

用法：`make print-OBJECTS`

### 试运行

```bash
make -n         # 打印命令但不执行
make --dry-run
```

### 调试输出

```bash
make -d         # 打印调试信息
make --debug
```

### 跟踪执行

```bash
make -p         # 打印数据库（所有规则和变量）
```

## 参考资源

### 官方文档

- [GNU Make 手册](https://www.gnu.org/software/make/manual/) - 完整参考
- [POSIX Make 规范](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/make.html) - 标准规范

### 教程和指南

- [Make Tutorial by Chase Lambert](https://makefiletutorial.com/) - 交互式教程
- [Recursive Make Considered Harmful](https://aegis.sourceforge.net/auug97.pdf) - 关于 Make 反模式的经典论文

### 书籍

- *Managing Projects with GNU Make* by Robert Mecklenburg (O'Reilly)
- *The GNU Make Book* by John Graham-Cumming

### 工具

- [remake](http://bashdb.sourceforge.net/remake/) - 带调试支持的增强版 Make
- [CMake](https://cmake.org/) - 现代构建系统生成器
- [Ninja](https://ninja-build.org/) - 快速构建系统（Make 替代品）

### 相关技术

- [Autotools](https://www.gnu.org/software/automake/) - GNU 构建系统
- [Meson](https://mesonbuild.com/) - 现代构建系统
- [Bazel](https://bazel.build/) - Google 的构建系统
