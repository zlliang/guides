# Bash 脚本编写

关于 Bash（Bourne Again Shell）脚本编写的综合指南——类 Unix 系统上使用最广泛的 shell 脚本语言。本指南涵盖从基本语法到高级技术的所有内容，用于编写健壮、可维护的 shell 脚本。

## 快速参考

### 基本语法

```bash
#!/bin/bash

# 变量
name="value"
readonly CONST="constant"

# 数组
arr=(one two three)
echo "${arr[0]}"
echo "${arr[@]}"

# 条件语句
if [[ condition ]]; then
    # code
elif [[ other ]]; then
    # code
else
    # code
fi

# 循环
for item in "${arr[@]}"; do
    echo "$item"
done

while [[ condition ]]; do
    # code
done

# 函数
function_name() {
    local var="$1"
    echo "$var"
}

# 特殊变量
$0      # 脚本名称
$1-$9   # 参数
$@      # 所有参数（数组）
$#      # 参数数量
$?      # 退出状态
$$      # 进程 ID
$!      # 最后一个后台进程 PID
```

### 常见模式

```bash
# 遇到错误时退出
set -e
set -o errexit

# 遇到未定义变量时退出
set -u
set -o nounset

# 管道失败时退出
set -o pipefail

# 调试模式
set -x
set -o xtrace

# 严格模式（推荐）
set -euo pipefail

# 检查命令是否存在
command -v git >/dev/null 2>&1 || { echo "git required"; exit 1; }

# 检查文件是否存在
[[ -f file.txt ]] && echo "exists"

# 默认值
: "${VAR:=default}"
```

## 简介

### 什么是 Bash？

Bash（Bourne Again Shell）是一个 Unix shell 和命令语言，由 Brian Fox 为 GNU 项目编写，作为 Bourne shell (sh) 的免费替代品。它于 1989 年发布，已成为大多数 Linux 发行版和 macOS（直到 Catalina）的默认 shell。

### 为什么使用 Bash 脚本？

**优势**：
- **通用**：几乎在所有类 Unix 系统上都可用
- **可移植**：脚本可在不同平台上运行
- **强大**：完整的编程语言 + shell 命令
- **集成**：直接访问系统命令和实用程序
- **自动化**：非常适合系统管理任务

**使用场景**：
- 系统管理和自动化
- DevOps 和 CI/CD 管道
- 构建脚本和部署
- 数据处理和文本操作
- 程序之间的胶水代码

### 何时使用 Bash

**适合**：
- 快速自动化脚本
- 系统管理任务
- 文本处理和文件操作
- 包装其他程序
- CI/CD 管道

**不理想**：
- 复杂应用程序（使用 Python、Go 等）
- 大量计算
- 跨平台 GUI 应用程序
- 需要强类型时

## 入门

### Shebang

脚本始终以 shebang 开头：

```bash
#!/bin/bash
```

**更好**（在 PATH 中查找 bash）：
```bash
#!/usr/bin/env bash
```

**带选项**：
```bash
#!/bin/bash -e          # 遇到错误退出
#!/bin/bash -x          # 调试模式
#!/bin/bash -eu         # 遇到错误和未定义变量退出
```

### 运行脚本

```bash
# 使其可执行
chmod +x script.sh

# 运行
./script.sh

# 用 bash 显式运行
bash script.sh

# Source（在当前 shell 中运行）
source script.sh
. script.sh
```

### Hello World

```bash
#!/bin/bash

echo "Hello, World!"
```

## 变量

### 声明和赋值

```bash
# 简单赋值（无空格！）
name="John"
count=42

# 只读变量
readonly PI=3.14159

# 命令替换
current_dir=$(pwd)
today=$(date +%Y-%m-%d)

# 算术
count=$((count + 1))
result=$((5 * 3))
```

### 变量展开

```bash
name="John"

# 基本
echo "$name"              # John
echo "${name}"            # John（推荐）

# 默认值
echo "${var:-default}"    # 如果未设置则使用默认值
echo "${var:=default}"    # 如果未设置则使用并赋值默认值
echo "${var:+alternate}"  # 如果已设置则使用替代值
echo "${var:?error}"      # 如果未设置则错误

# 字符串操作
echo "${name:0:2}"        # Jo（子串）
echo "${name#pattern}"    # 从开头移除最短匹配
echo "${name##pattern}"   # 从开头移除最长匹配
echo "${name%pattern}"    # 从末尾移除最短匹配
echo "${name%%pattern}"   # 从末尾移除最长匹配
echo "${name/old/new}"    # 替换第一次出现
echo "${name//old/new}"   # 替换所有出现

# 长度
echo "${#name}"           # 4

# 大小写
echo "${name^^}"          # JOHN
echo "${name,,}"          # john
echo "${name^}"           # John（首字母大写）
```

### 环境变量

```bash
# 导出变量（对子进程可用）
export PATH="/usr/local/bin:$PATH"
export EDITOR=vim

# 或一行
export VAR="value"

# 取消设置变量
unset VAR

# 查看所有环境变量
printenv
env
```

### 特殊变量

```bash
$0          # 脚本名称
$1, $2, ... # 位置参数
$@          # 所有参数作为单独的词
$*          # 所有参数作为单个词
$#          # 参数数量
$?          # 最后一个命令的退出状态
$$          # 当前进程 ID
$!          # 最后一个后台进程的 PID
$_          # 前一个命令的最后一个参数
```

## 数组

### 索引数组

```bash
# 声明
arr=(one two three)
arr=([0]=first [1]=second [2]=third)

# 赋值
arr[3]="fourth"

# 访问
echo "${arr[0]}"          # first
echo "${arr[@]}"          # 所有元素
echo "${arr[*]}"          # 所有元素（单个词）

# 长度
echo "${#arr[@]}"         # 元素数量
echo "${#arr[0]}"         # 第一个元素的长度

# 索引
echo "${!arr[@]}"         # 所有索引

# 追加
arr+=("fifth")

# 遍历
for item in "${arr[@]}"; do
    echo "$item"
done

# 带索引
for i in "${!arr[@]}"; do
    echo "$i: ${arr[$i]}"
done

# 切片
echo "${arr[@]:1:2}"      # 元素 1-2
```

### 关联数组（Bash 4+）

```bash
# 声明
declare -A map

# 赋值
map[key]="value"
map[name]="John"
map[age]="30"

# 访问
echo "${map[key]}"

# 所有键
echo "${!map[@]}"

# 所有值
echo "${map[@]}"

# 遍历
for key in "${!map[@]}"; do
    echo "$key: ${map[$key]}"
done

# 检查键是否存在
if [[ -v map[key] ]]; then
    echo "Key exists"
fi
```

## 控制流

### If 语句

```bash
# 基本 if
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

# 单行
[[ condition ]] && echo "true" || echo "false"
```

### 测试条件

**文件测试**：
```bash
[[ -f file ]]       # 文件存在且是普通文件
[[ -d dir ]]        # 目录存在
[[ -e path ]]       # 路径存在（文件或目录）
[[ -r file ]]       # 文件可读
[[ -w file ]]       # 文件可写
[[ -x file ]]       # 文件可执行
[[ -s file ]]       # 文件存在且非空
[[ -L link ]]       # 路径是符号链接
[[ file1 -nt file2 ]] # file1 比 file2 新
[[ file1 -ot file2 ]] # file1 比 file2 旧
```

**字符串测试**：
```bash
[[ -z "$str" ]]     # 字符串为空
[[ -n "$str" ]]     # 字符串非空
[[ "$a" == "$b" ]]  # 字符串相等
[[ "$a" != "$b" ]]  # 字符串不相等
[[ "$a" < "$b" ]]   # 字典序小于
[[ "$a" > "$b" ]]   # 字典序大于
[[ "$str" =~ regex ]] # 字符串匹配正则
```

**数字测试**：
```bash
[[ $a -eq $b ]]     # 相等
[[ $a -ne $b ]]     # 不相等
[[ $a -lt $b ]]     # 小于
[[ $a -le $b ]]     # 小于或等于
[[ $a -gt $b ]]     # 大于
[[ $a -ge $b ]]     # 大于或等于
```

**逻辑运算符**：
```bash
[[ cond1 && cond2 ]] # AND
[[ cond1 || cond2 ]] # OR
[[ ! cond ]]         # NOT

# 组合
if [[ -f file && -r file ]]; then
    echo "File exists and is readable"
fi
```

### Case 语句

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

# 示例
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

### 循环

**For 循环**：
```bash
# 遍历列表
for item in one two three; do
    echo "$item"
done

# 遍历数组
for item in "${arr[@]}"; do
    echo "$item"
done

# C 风格 for 循环
for ((i=0; i<10; i++)); do
    echo "$i"
done

# 遍历文件
for file in *.txt; do
    echo "$file"
done

# 遍历命令输出
for line in $(cat file.txt); do
    echo "$line"
done
```

**While 循环**：
```bash
i=0
while [[ $i -lt 10 ]]; do
    echo "$i"
    ((i++))
done

# 逐行读取文件
while IFS= read -r line; do
    echo "$line"
done < file.txt
```

**Until 循环**：
```bash
i=0
until [[ $i -ge 10 ]]; do
    echo "$i"
    ((i++))
done
```

**循环控制**：
```bash
# Break（退出循环）
for i in {1..10}; do
    [[ $i -eq 5 ]] && break
    echo "$i"
done

# Continue（跳过迭代）
for i in {1..10}; do
    [[ $i -eq 5 ]] && continue
    echo "$i"
done
```

## 函数

### 基本函数

```bash
# 声明
function_name() {
    # code
}

# 或使用 'function' 关键字
function function_name {
    # code
}

# 带局部变量
greet() {
    local name="$1"
    echo "Hello, $name!"
}

# 调用函数
greet "John"
```

### 函数参数

```bash
process_file() {
    local file="$1"
    local option="$2"
    
    # 检查参数
    if [[ $# -lt 1 ]]; then
        echo "Usage: process_file <file> [option]" >&2
        return 1
    fi
    
    # 使用参数
    echo "Processing $file with option: ${option:-default}"
}

# $1, $2, ... : 位置参数
# $@          : 所有参数（数组）
# $*          : 所有参数（字符串）
# $#          : 参数数量
```

### 返回值

```bash
# 返回状态（0-255）
check_file() {
    [[ -f "$1" ]] && return 0 || return 1
}

if check_file "test.txt"; then
    echo "File exists"
fi

# 通过 stdout 返回数据
get_username() {
    echo "$USER"
}

username=$(get_username)

# 通过变量返回数据（bash 4.3+ 使用 nameref）
get_data() {
    local -n result=$1
    result="some data"
}

get_data myvar
echo "$myvar"
```

### 局部 vs 全局变量

```bash
global_var="global"

my_function() {
    local local_var="local"
    global_var="modified"    # 修改全局变量
    
    echo "$local_var"        # 可访问
}

my_function
echo "$global_var"           # "modified"
echo "$local_var"            # 空（不可访问）
```

## 输入和输出

### 读取输入

```bash
# 读取单个变量
read -p "Enter name: " name

# 读取多个变量
read -p "Enter first and last name: " first last

# 带超时读取
read -t 5 -p "Enter name (5s timeout): " name

# 读取密码（无回显）
read -s -p "Enter password: " password
echo  # 密码后换行

# 逐行读取
while IFS= read -r line; do
    echo "$line"
done < file.txt

# 读入数组
mapfile -t lines < file.txt
# 或
readarray -t lines < file.txt
```

### 输出

```bash
# 打印到 stdout
echo "message"
printf "formatted %s\n" "message"

# 打印到 stderr
echo "error" >&2
printf "error: %s\n" "message" >&2

# 重定向 stdout
command > file.txt

# 重定向 stderr
command 2> error.log

# 重定向两者
command &> output.log
command > output.log 2>&1

# 追加
command >> file.txt
command 2>> error.log

# 丢弃输出
command > /dev/null 2>&1

# Here document
cat << EOF
Line 1
Line 2
EOF

# Here string
grep "pattern" <<< "string to search"
```

## 字符串操作

### 字符串操作

```bash
str="Hello, World!"

# 长度
${#str}                   # 13

# 子串
${str:7}                  # World!
${str:7:5}                # World

# 移除前缀
${str#Hello, }            # World!（最短匹配）
${str##*/}                # 移除到最后一个 /

# 移除后缀
${str%!}                  # Hello, World（最短匹配）
${str%%,*}                # Hello（最长匹配）

# 替换
${str/World/Universe}     # 替换第一个
${str//o/0}              # 替换所有

# 大小写转换
${str^^}                  # HELLO, WORLD!
${str,,}                  # hello, world!
${str^}                   # Hello, world!（首字符）
```

### 字符串比较

```bash
str1="hello"
str2="world"

# 相等
[[ "$str1" == "$str2" ]]
[[ "$str1" = "$str2" ]]   # 与 == 相同

# 不相等
[[ "$str1" != "$str2" ]]

# 模式匹配
[[ "$str1" == h* ]]       # 以 h 开头

# 正则匹配
[[ "$str1" =~ ^[a-z]+$ ]]

# 字典序比较
[[ "$str1" < "$str2" ]]
[[ "$str1" > "$str2" ]]
```

### 字符串分割

```bash
# 使用 IFS
IFS=',' read -ra fields <<< "a,b,c"
for field in "${fields[@]}"; do
    echo "$field"
done

# 分割到数组
str="one:two:three"
IFS=':' read -ra parts <<< "$str"
```

### 字符串连接

```bash
# 用分隔符连接数组
arr=(one two three)
IFS=,
joined="${arr[*]}"        # one,two,three
unset IFS
```

## 文件操作

### 文件测试

```bash
# 存在性
[[ -e file ]]       # 存在
[[ -f file ]]       # 普通文件
[[ -d dir ]]        # 目录
[[ -L link ]]       # 符号链接
[[ -s file ]]       # 非空

# 权限
[[ -r file ]]       # 可读
[[ -w file ]]       # 可写
[[ -x file ]]       # 可执行

# 比较
[[ file1 -nt file2 ]] # 比 file2 新
[[ file1 -ot file2 ]] # 比 file2 旧
[[ file1 -ef file2 ]] # 同一文件
```

### 读取文件

```bash
# 读取整个文件
content=$(<file.txt)

# 逐行读取（保留空格）
while IFS= read -r line; do
    echo "$line"
done < file.txt

# 读入数组
mapfile -t lines < file.txt

# 使用自定义分隔符读取
while IFS=: read -r user pass uid gid info home shell; do
    echo "User: $user, UID: $uid"
done < /etc/passwd
```

### 写入文件

```bash
# 写入
echo "text" > file.txt

# 追加
echo "text" >> file.txt

# 写入多行
cat > file.txt << 'EOF'
Line 1
Line 2
EOF

# 用 printf 写入
printf "%s\n" "line1" "line2" > file.txt
```

### 文件操作

```bash
# 复制
cp source dest
cp -r dir1 dir2           # 递归

# 移动/重命名
mv old new

# 删除
rm file
rm -rf dir                # 递归，强制

# 创建目录
mkdir dir
mkdir -p path/to/dir      # 创建父目录

# 更改权限
chmod +x script.sh
chmod 755 file
chmod u+x file

# 更改所有者
chown user:group file
```

## 进程管理

### 运行命令

```bash
# 在前台运行
command args

# 在后台运行
command args &

# 获取后台进程的 PID
command &
pid=$!

# 等待后台进程
wait $pid

# 在后台运行多个并等待
command1 &
command2 &
wait
```

### 退出状态

```bash
# 检查退出状态
command
if [[ $? -eq 0 ]]; then
    echo "Success"
fi

# 或直接
if command; then
    echo "Success"
fi

# 设置退出状态
exit 0    # 成功
exit 1    # 错误

# 从函数返回
return 0
return 1
```

### 管道

```bash
# 简单管道
cat file.txt | grep "pattern" | sort

# 带错误处理的管道（set -o pipefail）
set -o pipefail
cat file.txt | grep "pattern" | sort
status=$?

# 进程替换
diff <(sort file1.txt) <(sort file2.txt)

# 重定向到多个命令（tee）
command | tee output.txt | grep "pattern"
```

### 作业控制

```bash
# 列出作业
jobs

# 调到前台
fg %1

# 发送到后台
bg %1

# 杀死作业
kill %1

# Disown 作业（shell 退出后继续运行）
disown %1
```

## 高级特性

### 命令替换

```bash
# 现代语法
result=$(command)
files=$(ls *.txt)

# 旧语法（避免）
result=`command`

# 嵌套
outer=$(echo "inner: $(date)")

# 去除尾随换行符
result=$(command | tr -d '\n')
```

### 算术

```bash
# 算术展开
result=$((5 + 3))
result=$((x * y))
result=$((++i))
result=$((i++))

# 算术求值
((i++))
((count += 5))

# 算术上下文中的比较
if ((count > 10)); then
    echo "Count is greater than 10"
fi

# 运算符
$((a + b))          # 加法
$((a - b))          # 减法
$((a * b))          # 乘法
$((a / b))          # 除法
$((a % b))          # 取模
$((a ** b))         # 幂运算
```

### 花括号展开

```bash
# 范围
echo {1..10}              # 1 2 3 4 5 6 7 8 9 10
echo {a..z}               # a b c d ... z
echo {01..10}             # 01 02 03 ... 10

# 列表
echo {cat,dog,bird}       # cat dog bird

# 组合
echo file{1,2,3}.txt      # file1.txt file2.txt file3.txt

# 嵌套
echo {a,b}{1,2}           # a1 a2 b1 b2

# 创建多个文件
touch file{1..100}.txt
```

### 参数展开

```bash
# 默认值
${var:-default}           # 如果未设置/为空则使用默认值
${var:=default}           # 如果未设置/为空则赋值默认值
${var:+alternate}         # 如果已设置则使用替代值
${var:?error}             # 如果未设置则退出并显示错误

# 长度
${#var}

# 子串
${var:offset}
${var:offset:length}

# 移除模式（从开头）
${var#pattern}            # 最短匹配
${var##pattern}           # 最长匹配

# 移除模式（从末尾）
${var%pattern}            # 最短匹配
${var%%pattern}           # 最长匹配

# 替换
${var/pattern/replacement}   # 第一次出现
${var//pattern/replacement}  # 所有出现

# 大小写修改
${var^^}                  # 大写
${var,,}                  # 小写
${var^}                   # 首字母大写
${var~}                   # 切换首字符大小写
```

### Here Documents

```bash
# 基本 here document
cat << EOF
Line 1
Variable: $var
Line 3
EOF

# 引号（无变量展开）
cat << 'EOF'
Literal $var
EOF

# 缩进（去除前导制表符）
cat <<- EOF
	Indented
	Text
EOF

# 赋值给变量
read -r -d '' var << EOF
Multiple
Lines
EOF
```

## 错误处理

### Set 选项

```bash
# 遇到错误退出
set -e
set -o errexit

# 遇到未定义变量退出
set -u
set -o nounset

# 管道失败时退出
set -o pipefail

# 调试模式（打印命令）
set -x
set -o xtrace

# 严格模式（推荐）
set -euo pipefail

# 禁用选项
set +e
```

### Trap

```bash
# 退出时清理
cleanup() {
    echo "Cleaning up..."
    rm -f temp_file
}
trap cleanup EXIT

# 处理信号
trap 'echo "Interrupted"; exit 1' INT TERM

# 处理错误
trap 'echo "Error at line $LINENO"' ERR

# 带清理的示例
temp_file=$(mktemp)
trap "rm -f $temp_file" EXIT

echo "data" > "$temp_file"
# 即使脚本提前退出，文件也会被清理
```

### 错误消息

```bash
# 打印到 stderr
error() {
    echo "ERROR: $*" >&2
}

# 打印并退出
die() {
    echo "FATAL: $*" >&2
    exit 1
}

# 使用
[[ -f file.txt ]] || die "file.txt not found"

# 带行号
error_at() {
    echo "ERROR at line ${BASH_LINENO[0]}: $*" >&2
}
```

### 防御性编程

```bash
#!/bin/bash
set -euo pipefail
IFS=$'\n\t'

# 检查依赖
command -v jq >/dev/null || { echo "jq required"; exit 1; }

# 验证参数
if [[ $# -lt 1 ]]; then
    echo "Usage: $0 <file>" >&2
    exit 1
fi

# 检查文件存在
[[ -f "$1" ]] || { echo "File not found: $1" >&2; exit 1; }

# 带错误处理的主逻辑
main() {
    local file="$1"
    
    # 尝试操作
    if ! process_file "$file"; then
        echo "Failed to process $file" >&2
        return 1
    fi
    
    echo "Success"
}

main "$@"
```

## 参数解析

### 手动解析

```bash
# 简单参数解析
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
            # 位置参数
            files+=("$1")
            shift
            ;;
    esac
done
```

### 使用 getopts

```bash
# 仅短选项
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

# 移过已处理的选项
shift $((OPTIND - 1))

# $@ 中的剩余参数
```

### 长选项（getopt）

```bash
# 使用 getopt（GNU）
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

## 最佳实践

### 脚本模板

```bash
#!/bin/bash
#
# Script description
#
# Usage: script.sh [options] <args>

set -euo pipefail
IFS=$'\n\t'

# 常量
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "${BASH_SOURCE[0]}")"

# 默认值
verbose=0
output=""

# 函数
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

# 解析参数
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

# 验证
[[ -n "${file:-}" ]] || die "File argument required"
[[ -f "$file" ]] || die "File not found: $file"

# 主逻辑
main() {
    [[ $verbose -eq 1 ]] && echo "Processing $file..."
    
    # 你的代码在这里
    
    echo "Done"
}

main
```

### 引号

```bash
# 始终引用变量
file="my file.txt"
cat "$file"               # 正确
cat $file                 # 错误（在空格处分割）

# 引用数组
files=("file 1.txt" "file 2.txt")
for file in "${files[@]}"; do
    echo "$file"
done

# 不引用以进行词分割（罕见）
options="-l -a -h"
ls $options              # 有意的词分割
```

### 错误处理

```bash
# 检查命令成功
if ! command; then
    echo "Command failed" >&2
    exit 1
fi

# 或
command || { echo "Failed" >&2; exit 1; }

# 带错误消息
grep pattern file.txt || {
    echo "Pattern not found" >&2
    exit 1
}

# Try-catch 风格
if result=$(command 2>&1); then
    echo "Success: $result"
else
    echo "Failed: $result" >&2
    exit 1
fi
```

### 可移植脚本

```bash
# 尽可能使用 POSIX 特性
# 使用 [[ ]] 而非 [ ]（bash 特定但更安全）
# 如果目标是 /bin/sh，避免 bashism

# 检查 bash
[[ -n "${BASH_VERSION:-}" ]] || {
    echo "Requires bash" >&2
    exit 1
}

# 需要最低 bash 版本
if [[ "${BASH_VERSINFO[0]}" -lt 4 ]]; then
    echo "Requires bash 4+" >&2
    exit 1
fi
```

## 常见模式

### 检查依赖

```bash
# 检查命令是否存在
check_command() {
    command -v "$1" >/dev/null 2>&1
}

if ! check_command git; then
    echo "git is required" >&2
    exit 1
fi

# 检查多个
for cmd in git curl jq; do
    command -v "$cmd" >/dev/null || {
        echo "$cmd is required" >&2
        exit 1
    }
done
```

### 查找脚本目录

```bash
# 查找脚本目录的可靠方法
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# 或使用 readlink
SCRIPT_DIR="$(dirname "$(readlink -f "${BASH_SOURCE[0]}")")"

# 加载相对于脚本的文件
source "$SCRIPT_DIR/lib/utils.sh"
```

### 临时文件

```bash
# 创建临时文件
temp_file=$(mktemp)
trap "rm -f $temp_file" EXIT

# 创建临时目录
temp_dir=$(mktemp -d)
trap "rm -rf $temp_dir" EXIT

# 使用临时文件
echo "data" > "$temp_file"
process "$temp_file"
```

### 并行执行

```bash
# 并行运行命令
command1 &
command2 &
command3 &
wait

# 带错误处理的并行
pids=()
for item in "${items[@]}"; do
    process "$item" &
    pids+=($!)
done

# 等待并检查状态
status=0
for pid in "${pids[@]}"; do
    if ! wait "$pid"; then
        status=1
    fi
done

exit $status
```

### 配置文件

```bash
# 加载配置文件
config_file="${HOME}/.myapp/config"
if [[ -f "$config_file" ]]; then
    source "$config_file"
fi

# 或带验证
load_config() {
    local config="$1"
    [[ -f "$config" ]] || return 1
    
    # 仅在安全时 source
    if [[ -r "$config" ]]; then
        source "$config"
    fi
}
```

### 日志

```bash
# 简单日志
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*"
}

log "Script started"

# 日志级别
LOG_LEVEL=${LOG_LEVEL:-INFO}

log_debug() { [[ "$LOG_LEVEL" == "DEBUG" ]] && echo "[DEBUG] $*" >&2; }
log_info() { echo "[INFO] $*"; }
log_warn() { echo "[WARN] $*" >&2; }
log_error() { echo "[ERROR] $*" >&2; }

# 使用
log_info "Processing file"
log_error "File not found"
```

### 锁文件

```bash
# 创建锁文件
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
# 你的代码在这里
```

## 安全性

### 输入验证

```bash
# 验证输入
validate_email() {
    local email="$1"
    [[ "$email" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]
}

# 清理文件名
sanitize() {
    echo "$1" | tr -cd '[:alnum:]._-'
}

# 验证数字
is_number() {
    [[ "$1" =~ ^[0-9]+$ ]]
}
```

### 避免注入

```bash
# 不要：eval 用户输入
user_input="$1"
eval "$user_input"        # 危险！

# 不要：命令中的未引用变量
rm -rf $dir               # 危险！（如果 dir="/ "）

# 要：引用变量
rm -rf "$dir"

# 不要：将用户输入传递给 shell
sh -c "$user_input"       # 危险！

# 要：对命令参数使用数组
args=("$@")
command "${args[@]}"
```

### 安全权限

```bash
# 使用特定权限创建文件
(umask 077 && touch sensitive.txt)

# 设置限制性权限
chmod 600 private.key
chmod 700 script.sh

# 使用前检查权限
if [[ ! -r "$file" ]]; then
    echo "Cannot read $file" >&2
    exit 1
fi
```

## 调试

### 调试模式

```bash
# 启用调试输出
bash -x script.sh

# 或在脚本中
set -x

# 调试特定部分
set -x
# 要调试的代码
set +x

# 详细模式
set -v
```

### 调试工具

```bash
# 打印变量
echo "DEBUG: var=$var" >&2

# 带行号打印
echo "DEBUG: ${BASH_SOURCE[0]}:${LINENO}: $var" >&2

# 跟踪函数
trace() {
    echo "TRACE: $*" >&2
    "$@"
}

trace command arg1 arg2
```

### ShellCheck

使用 ShellCheck 查找错误：

```bash
# 安装
brew install shellcheck  # macOS
apt install shellcheck   # Ubuntu

# 检查脚本
shellcheck script.sh

# 禁用特定警告
# shellcheck disable=SC2034
unused_var="value"

# 在脚本头中
# shellcheck disable=SC1090,SC2148
```

## 高级主题

### 子 Shell

```bash
# 在子 shell 中运行（不影响父 shell）
(
    cd /tmp
    echo "In subshell: $(pwd)"
)
echo "In parent: $(pwd)"

# 分组命令
{
    command1
    command2
} > output.txt

# 对比子 shell
(
    command1
    command2
) > output.txt
```

### 进程替换

```bash
# 比较输出
diff <(sort file1.txt) <(sort file2.txt)

# 从多个源读取
while read -r line; do
    echo "$line"
done < <(cat file1.txt file2.txt)

# 向期望文件的命令提供输入
command <(echo "data")
```

### 协进程

```bash
# 创建协进程
coproc bc

# 发送到协进程
echo "2 + 2" >&"${COPROC[1]}"

# 从协进程读取
read -r result <&"${COPROC[0]}"
echo "Result: $result"

# 关闭协进程
kill "$COPROC_PID"
```

### 正则匹配

```bash
# 匹配模式
if [[ "$email" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
    echo "Valid email"
fi

# 捕获组
if [[ "$str" =~ ([0-9]+)-([0-9]+) ]]; then
    first="${BASH_REMATCH[1]}"
    second="${BASH_REMATCH[2]}"
    echo "First: $first, Second: $second"
fi
```

### 选项的关联数组

```bash
declare -A options=(
    [verbose]=0
    [output]=""
    [debug]=0
)

# 解析
while [[ $# -gt 0 ]]; do
    case $1 in
        -v) options[verbose]=1; shift ;;
        -o) options[output]="$2"; shift 2 ;;
        -d) options[debug]=1; shift ;;
    esac
done

# 使用
[[ ${options[verbose]} -eq 1 ]] && echo "Verbose mode"
```

## 性能

### 优化技巧

**避免子 Shell**：
```bash
# 慢
result=$(cat file.txt)

# 快
result=$(<file.txt)
```

**使用内置命令**：
```bash
# 慢
result=$(expr $a + $b)

# 快
result=$((a + b))
```

**最小化外部命令**：
```bash
# 慢
for file in $(ls *.txt); do
    # ...
done

# 快
for file in *.txt; do
    # ...
done
```

**高效读取文件**：
```bash
# 慢（生成 cat）
while read line; do
    # ...
done < <(cat file.txt)

# 快
while read line; do
    # ...
done < file.txt
```

## 测试

### 测试框架（bats）

**安装 bats**：
```bash
brew install bats-core
```

**test.bats**：
```bash
#!/usr/bin/env bats

setup() {
    # 每个测试前运行
    temp_dir=$(mktemp -d)
}

teardown() {
    # 每个测试后运行
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

**运行测试**：
```bash
bats test.bats
```

### 手动测试

```bash
# 测试函数
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

# 使用
result=$(my_function "input")
test_function "expected output" "$result"
```

## 参考资源

### 官方文档

- [GNU Bash 手册](https://www.gnu.org/software/bash/manual/)
- [Bash 参考手册（PDF）](https://www.gnu.org/software/bash/manual/bash.pdf)
- [POSIX Shell 规范](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html)

### 学习资源

- [Bash Guide](https://mywiki.wooledge.org/BashGuide) - 综合 wiki
- [Advanced Bash-Scripting Guide](https://tldp.org/LDP/abs/html/) - 经典资源
- [Bash Hackers Wiki](https://wiki.bash-hackers.org/) - 高级技术
- [ShellCheck Wiki](https://www.shellcheck.net/wiki/) - 常见错误

### 工具

- [ShellCheck](https://www.shellcheck.net/) - Shell 脚本 linter
- [explainshell.com](https://explainshell.com/) - 解析和解释命令
- [bashdb](http://bashdb.sourceforge.net/) - Bash 调试器
- [bats](https://github.com/bats-core/bats-core) - 测试框架

### 书籍

- *Learning the bash Shell* by Cameron Newham (O'Reilly)
- *Bash Cookbook* by Carl Albing and JP Vossen (O'Reilly)
- *Classic Shell Scripting* by Arnold Robbins and Nelson Beebe (O'Reilly)

### 风格指南

- [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html)
- [Chromium Shell Style Guide](https://chromium.googlesource.com/chromium/src/+/master/styleguide/shell.md)

### 社区

- [Stack Overflow - bash 标签](https://stackoverflow.com/questions/tagged/bash)
- [r/bash](https://www.reddit.com/r/bash/)
- [Unix & Linux Stack Exchange](https://unix.stackexchange.com/)
