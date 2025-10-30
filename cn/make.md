# Make 与 Makefile

一份全面的指南，帮助你理解和掌握 Make——构建自动化工具——以及驱动它的配置文件 Makefile。本指南涵盖了 `make` 命令和 Makefile 语法，适用于在类 Unix 系统中管理项目编译、测试、部署等任务。

## 快速参考

### Make 命令

```bash
make                    # 构建默认目标
make target             # 构建特定目标
make target1 target2    # 构建多个目标
make -n                 # 试运行（显示命令但不执行）
make -j4                # 运行 4 个并行任务
make -B                 # 强制重新构建所有目标
make -C dir             # 在指定目录中运行 make
make VAR=value          # 覆盖变量
make -f custom.mk       # 使用自定义 makefile
make --version          # 显示版本
```

### Makefile 语法

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

Make 是一个构建自动化工具，它能自动判断大型程序中哪些部分需要重新编译，并发出命令来重新编译它们。Make 由 Stuart Feldman 于 1976 年在贝尔实验室创建,至今仍然是使用最广泛的构建工具之一，特别是在 C/C++ 项目和系统管理中。

Make 的核心概念是**增量构建**：它比较源文件及其输出的时间戳，以确定需要重新构建什么，通过只重新编译已更改的内容来节省时间。

### 什么是 Makefile？

Makefile 是一个特殊文件，包含一组指令（规则），Make 使用这些指令来构建和管理项目。它定义了：

- **目标（Target）**：要构建什么（可执行文件、目标文件或操作）
- **依赖项（Dependencies）**：需要哪些文件或目标
- **配方（Recipe）**：执行哪些命令来构建目标

默认情况下，Make 会在当前目录中查找名为 `Makefile`、`makefile` 或 `GNUmakefile` 的文件。

### 为什么使用 Make？

- **增量构建**：只重新编译已更改的文件
- **依赖管理**：自动跟踪文件关系
- **自动化**：减少重复的手动命令
- **可移植性**：在类 Unix 系统中通用
- **简洁性**：用声明式语法处理复杂工作流
- **通用性**：受大多数开发工具和 IDE 支持

## Make 命令

### 基本用法

最简单的 Make 调用：

```bash
make
```

这会读取当前目录中的 Makefile 并构建第一个目标（默认目标）。

### 构建特定目标

```bash
make clean          # 构建 'clean' 目标
make test           # 构建 'test' 目标
make install        # 构建 'install' 目标
```

### 构建多个目标

```bash
make clean all      # 清理，然后构建 all
make test install   # 运行测试，然后安装
```

### 常用命令行选项

#### 试运行

预览 Make 将执行的操作而不实际执行：

```bash
make -n
make --dry-run
make --just-print
```

#### 并行执行

并行运行多个任务以加快构建速度：

```bash
make -j4            # 最多运行 4 个并行任务
make -j             # 运行尽可能多的任务
make -j$(nproc)     # 使用 CPU 核心数
```

#### 强制重新构建

无论时间戳如何，重新构建所有目标：

```bash
make -B
make --always-make
```

#### 更改目录

在不同目录中运行 Make：

```bash
make -C src/        # 等同于: cd src && make
make -C ../project
```

#### 指定 Makefile

使用自定义名称的 Makefile：

```bash
make -f build.mk
make --file=custom.makefile
```

#### 覆盖变量

从命令行设置或覆盖 Makefile 变量：

```bash
make CC=clang
make CFLAGS="-Wall -O3"
make DEBUG=1 test
```

#### 静默模式

抑制命令回显：

```bash
make -s
make --silent
```

#### 忽略错误

即使命令失败也继续：

```bash
make -i
make --ignore-errors
```

#### 打印变量

通过打印 Make 的内部数据库进行调试：

```bash
make -p              # 打印所有规则和变量
make --print-data-base
```

#### 继续执行

即使一个目标失败也继续构建其他目标：

```bash
make -k
make --keep-going
```

### 环境变量

Make 遵循多个环境变量：

- `MAKE`：Make 可执行文件的名称（用于递归 Make）
- `MAKEFLAGS`：Make 的默认标志
- `MAKEFILE_LIST`：已解析的 Makefile 列表
- `MAKELEVEL`：递归深度（顶层 Make 为 0）
- `CC`、`CXX`、`CFLAGS` 等：编译器和标志的默认值

### 退出状态

Make 返回：
- `0`：成功（所有目标都是最新的或已成功构建）
- `1`：构建期间发生错误
- `2`：命令行使用错误

## Makefile 语法

### 基本规则

每个 Makefile 由具有以下结构的规则组成：

```makefile
target: dependencies
	recipe
```

- **目标（Target）**：要创建的文件或操作的名称
- **依赖项（Dependencies）**：构建目标之前必须存在的文件
- **配方（Recipe）**：创建目标的 shell 命令（必须用制表符 TAB 缩进）

**重要**：配方必须用 TAB 字符缩进，而不是空格。这是一个常见的错误来源。

### 简单示例

```makefile
hello: hello.c
	gcc -o hello hello.c
```

这条规则表示："要构建 `hello`，我们需要 `hello.c`，通过运行 `gcc -o hello hello.c` 来构建。"

Make 将：
1. 检查 `hello` 是否存在
2. 检查 `hello.c` 是否比 `hello` 更新
3. 如果 `hello` 不存在或 `hello.c` 更新，则运行配方

### 多个规则

```makefile
program: main.o utils.o
	gcc -o program main.o utils.o

main.o: main.c
	gcc -c main.c

utils.o: utils.c
	gcc -c utils.c
```

Make 构建依赖树并按正确顺序执行规则。

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

`.PHONY` 声明可防止与名为 `clean`、`all` 或 `install` 的文件冲突，并确保 Make 始终运行这些目标，无论文件时间戳如何。

## 变量

### 定义变量

```makefile
CC = gcc
CFLAGS = -Wall -O2
OBJECTS = main.o utils.o

program: $(OBJECTS)
	$(CC) $(CFLAGS) -o program $(OBJECTS)
```

变量使用 `$(VAR)` 或 `${VAR}` 引用。

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
- `LDLIBS`：要链接的库
- `AR`：存档命令（默认：`ar`）
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
NPROC = $(shell nproc)
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

# 添加前缀/后缀
$(addprefix prefix_,list)
$(addsuffix _suffix,list)
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

ifneq ($(OS),Windows)
    RM = rm -f
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

### 配方前缀

```makefile
target:
	@echo "静默 - 不回显"        # @ 抑制回显
	-rm nonexistent.txt         # - 忽略错误
	+echo "始终执行"            # + 始终执行（即使使用 -n）
```

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

### 11. 明智地使用并行构建

通过正确指定所有依赖项来确保 Makefile 是并行安全的。

### 12. 尽可能避免递归 Make

考虑使用非递归 Make 模式以获得更好的依赖跟踪和并行构建。请参阅参考资料中的"Recursive Make Considered Harmful"。

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

## 调试 Make 和 Makefile

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
make --debug=b  # 基本调试
make --debug=v  # 详细调试
make --debug=a  # 所有调试信息
```

### 跟踪执行

```bash
make -p         # 打印数据库（所有规则和变量）
make --trace    # 在执行时打印配方行
```

### 常见错误

#### TAB 与空格错误

```
Makefile:3: *** missing separator.  Stop.
```

**解决方案**：确保配方使用 TAB 字符，而不是空格。

#### 循环依赖

```
Makefile:5: Circular main.o <- main.o dependency dropped.
```

**解决方案**：检查依赖链中的循环。

#### 没有制作目标的规则

```
make: *** No rule to make target 'file.c', needed by 'file.o'.  Stop.
```

**解决方案**：确保依赖文件存在或模式规则正确。

## 参考资源

### 官方文档

- [GNU Make 手册](https://www.gnu.org/software/make/manual/) - 完整参考
- [POSIX Make 规范](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/make.html) - 标准规范
- [GNU Make GitHub 仓库](https://github.com/mirror/make) - 源代码

### 教程和指南

- [Make Tutorial by Chase Lambert](https://makefiletutorial.com/) - 交互式教程
- [Recursive Make Considered Harmful](https://aegis.sourceforge.net/auug97.pdf) - 关于 Make 反模式的经典论文
- [Managing Projects with GNU Make (在线)](https://www.oreilly.com/openbook/make3/book/) - 免费在线版本

### 书籍

- *Managing Projects with GNU Make* by Robert Mecklenburg (O'Reilly)
- *The GNU Make Book* by John Graham-Cumming

### 工具

- [remake](http://bashdb.sourceforge.net/remake/) - 带调试支持的增强版 Make
- [CMake](https://cmake.org/) - 现代构建系统生成器
- [Ninja](https://ninja-build.org/) - 快速构建系统（Make 替代品）
- [bear](https://github.com/rizsotto/Bear) - 从 Make 生成编译数据库

### 相关技术

- [Autotools](https://www.gnu.org/software/automake/) - GNU 构建系统
- [Meson](https://mesonbuild.com/) - 现代构建系统
- [Bazel](https://bazel.build/) - Google 的构建系统
- [SCons](https://scons.org/) - 基于 Python 的构建工具
