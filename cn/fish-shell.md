# Fish Shell

关于 Fish（友好的交互式 Shell）的综合指南——一个适用于 Linux、macOS 和其他类 Unix 系统的智能且用户友好的命令行 shell。本指南主要关注交互式使用，并介绍 shell 脚本编写。

## 快速参考

### 基本命令

```fish
# 历史记录搜索
fish_config                 # 打开基于 Web 的配置
history search <term>       # 搜索命令历史
history merge              # 合并所有会话的历史

# 补全
complete -c <cmd>          # 显示命令的补全
funced <function>          # 在编辑器中编辑函数
funcsave <function>        # 永久保存函数

# 变量
set VAR value             # 设置变量
set -e VAR                # 删除变量
set -x VAR value          # 导出变量（环境变量）
set -U VAR value          # 通用变量（跨会话）

# 函数
function name
    # body
end

# 缩写
abbr --add gco 'git checkout'
abbr --erase gco
```

### 键绑定

```
Ctrl+R      # 搜索历史（反向）
Alt+↑/↓     # 上一个/下一个参数
Alt+E       # 在 $EDITOR 中编辑命令
Alt+L       # 列出目录
Tab         # 补全或显示补全
→           # 接受自动建议
Ctrl+→      # 接受自动建议的一个词
```

### 与 Bash 的区别

```fish
# Bash                    # Fish
export VAR=value          set -x VAR value
VAR=value                 set VAR value
if [ "$x" = "y" ]        if test "$x" = "y"
source script.sh          source script.fish
$?                        $status
$@                        $argv
```

## 简介

### 什么是 Fish？

Fish（友好的交互式 Shell）是一个现代命令行 shell，旨在用户友好、功能丰富且使用愉快。与 Bash 或 Zsh 等传统 shell 不同，Fish 将交互式使用作为首要目标。

**核心理念**：
- 开箱即用，默认设置合理
- 交互功能应该是可发现的
- 配置应该简单
- Shell 应该有帮助，而不是神秘

### 为什么使用 Fish？

**优势**：
- **语法高亮**：运行命令前看到错误
- **自动建议**：基于历史和补全
- **Tab 补全**：默认适用于常见命令
- **合理的脚本语法**：没有像 `${var%%suffix}` 这样的神秘语法
- **基于 Web 的配置**：`fish_config` 打开 GUI
- **无需配置**：开箱即用

**何时使用 Fish**：
- 作为日常终端工作的交互式 shell
- 当你想要现代、愉快的终端体验
- 学习 shell 使用时（比 bash 语法更清晰）

**何时使用 Bash**：
- 编写可移植脚本（Fish 语法不同）
- 当你需要 POSIX 兼容性时
- 在 Fish 不可用的环境中

### 安装

**macOS**：
```bash
brew install fish
```

**Ubuntu/Debian**：
```bash
sudo apt install fish
```

**Fedora**：
```bash
sudo dnf install fish
```

**Arch Linux**：
```bash
sudo pacman -S fish
```

**从源码**：
```bash
git clone https://github.com/fish-shell/fish-shell.git
cd fish-shell
cmake . && make && sudo make install
```

**设为默认 Shell**：
```bash
# 如果不存在，将 Fish 添加到 /etc/shells
echo /usr/local/bin/fish | sudo tee -a /etc/shells

# 设为默认 shell
chsh -s /usr/local/bin/fish
```

## 交互功能

### 自动建议

Fish 根据以下内容在你输入时建议命令：
- 命令历史
- 可用补全
- 当前目录内容

**使用自动建议**：
```
$ git che█                    # 你输入 "git che"
$ git checkout█               # Fish 建议 "checkout"（灰色文本）
$ git checkout█               # 按 → 接受
$ git checkout main█          # 继续输入或用 Ctrl+→ 接受下一个词
```

**配置**：
```fish
# 禁用自动建议
set -U fish_autosuggestion_enabled 0

# 更改颜色（默认是灰色）
set -U fish_color_autosuggestion brblack
```

### 语法高亮

Fish 在你输入时高亮命令：
- **有效命令**：默认颜色（通常是白色/亮色）
- **无效命令**：红色
- **字符串**：绿色或黄色
- **选项/标志**：青色
- **注释**：灰色

**示例**：
```fish
$ vim file.txt              # "vim" 被高亮（有效命令）
$ vimm file.txt             # "vimm" 是红色（无效命令）
$ echo "hello"              # "hello" 被高亮为字符串
```

**自定义颜色**：
```fish
# 查看当前颜色
set | grep fish_color

# 更改命令颜色
set -U fish_color_command blue

# 更改错误颜色
set -U fish_color_error red --bold
```

### Tab 补全

Fish 为以下内容提供智能补全：
- 命令及其选项
- 文件路径
- Git 分支、远程、标签
- 包管理器包
- SSH 主机名
- 环境变量

**示例**：
```fish
# 命令补全
$ git che<Tab>              # 列出: checkout, cherry, cherry-pick

# 选项补全
$ ls --<Tab>                # 列出所有 --long-options

# Git 分支补全
$ git checkout <Tab>        # 列出所有分支

# 带描述的文件补全
$ vim <Tab>                 # 显示带预览的文件
```

**自定义补全**：
```fish
# 创建补全文件: ~/.config/fish/completions/mycommand.fish
complete -c mycommand -s h -l help -d 'Show help'
complete -c mycommand -s v -l verbose -d 'Verbose output'
complete -c mycommand -a '(__fish_complete_directories)' -d 'Directory'
```

### 历史

**搜索历史**：
```fish
# 交互式历史搜索（Ctrl+R）
$ <Ctrl+R>
bck-i-search: git_

# 用命令搜索历史
$ history search git
$ history search --contains checkout

# 显示最近历史
$ history | head -20
```

**历史导航**：
- `↑` / `↓`：浏览历史
- `Alt+↑` / `Alt+↓`：浏览历史搜索结果
- `Alt+.`：插入前一个命令的最后一个参数

**管理历史**：
```fish
# 清除历史
$ history clear

# 删除特定项
$ history delete --prefix "rm -rf"

# 合并所有会话的历史
$ history merge

# 保存历史
$ history save
```

### 命令替换

Fish 使用 `()` 而不是反引号或 `$()`：

```fish
# 命令替换
$ echo (date)
$ set current_dir (pwd)

# 对比 bash
$ echo $(date)
$ current_dir=$(pwd)
```

### 通配符和 Glob

**基本 Glob**：
```fish
$ ls *.txt                  # 所有 .txt 文件
$ ls **/*.fish              # 递归所有 .fish 文件
$ ls *.{txt,md}             # 所有 .txt 和 .md 文件
```

**高级模式**：
```fish
# 按类型匹配
$ ls (find . -type f)       # 所有文件
$ ls **.txt                 # 递归 .txt 文件
```

## 配置

### 配置文件

**主配置**：`~/.config/fish/config.fish`
- 每次 shell 启动时运行
- 设置环境变量
- 定义函数和缩写

**函数文件**：`~/.config/fish/functions/`
- 每个函数一个文件
- 需要时自动加载
- 使用 `funcsave` 保存到这里

**补全**：`~/.config/fish/completions/`
- 自定义补全定义
- 每个命令一个文件

### 基本配置

**~/.config/fish/config.fish**：
```fish
# 环境变量
set -x EDITOR vim
set -x VISUAL code
set -x PAGER less

# 添加到 PATH
fish_add_path /usr/local/bin
fish_add_path $HOME/.local/bin
fish_add_path $HOME/go/bin

# 别名（使用缩写代替）
abbr --add ll 'ls -la'
abbr --add g 'git'
abbr --add dc 'docker-compose'

# 问候语
set -U fish_greeting ""  # 禁用问候语

# 颜色
set -U fish_color_command blue
set -U fish_color_error red
set -U fish_color_param cyan
```

### Web 配置

```fish
$ fish_config
```

打开带以下内容的浏览器：
- **Colors**：更改语法高亮颜色
- **Prompt**：从内置提示主题中选择
- **Functions**：查看和编辑函数
- **Variables**：查看和编辑变量
- **History**：浏览命令历史
- **Bindings**：查看和修改键绑定
- **Abbreviations**：管理缩写

## 变量

### 设置变量

```fish
# 局部变量（当前作用域）
set name value

# 通用变量（所有会话，持久）
set -U name value

# 导出变量（环境）
set -x name value

# 多个值（列表）
set colors red blue green

# 追加
set -a PATH /new/path
```

### 变量作用域

**局部**（默认）：
```fish
function test
    set local_var "only in function"
end
```

**全局** (`-g`)：
```fish
set -g global_var "available everywhere in session"
```

**通用** (`-U`)：
```fish
set -U universal_var "persists across sessions"
```

### 特殊变量

```fish
$status                     # 退出状态（类似 bash 中的 $?）
$pipestatus                 # 管道命令的退出状态
$argv                       # 函数/脚本参数
$fish_pid                   # 当前 fish 进程的 PID
$HOME                       # 主目录
$PATH                       # 可执行文件搜索路径
$PWD                        # 当前目录
$SHLVL                      # Shell 嵌套级别
```

### 列表（数组）

```fish
# 创建列表
set colors red blue green

# 访问元素（从 1 开始索引！）
echo $colors[1]             # red
echo $colors[2..3]          # blue green
echo $colors[-1]            # green（最后一个）

# 遍历
for color in $colors
    echo $color
end

# 长度
count $colors               # 3

# 追加
set -a colors yellow

# 检查是否包含
contains blue $colors       # 如果为真返回 0
```

### 环境变量

```fish
# 导出变量
set -x EDITOR vim

# 或使用 -gx 全局导出
set -gx JAVA_HOME /usr/lib/jvm/java-11

# 查看所有导出的变量
set -x

# 取消导出（但保留变量）
set -u VARNAME
```

## 函数

### 定义函数

**内联**：
```fish
$ function hello
    echo "Hello, $argv!"
end
```

**在文件中**（`~/.config/fish/functions/hello.fish`）：
```fish
function hello
    echo "Hello, $argv!"
end
```

**带描述**：
```fish
function hello -d "Greet the user"
    echo "Hello, $argv!"
end
```

### 函数参数

```fish
function greet
    # $argv[1] 是第一个参数
    # $argv 是所有参数
    # count $argv 是参数数量
    
    if test (count $argv) -eq 0
        echo "Usage: greet <name>"
        return 1
    end
    
    echo "Hello, $argv[1]!"
end
```

**参数解析**：
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

### 编辑和保存函数

```fish
# 在 $EDITOR 中编辑函数
$ funced myfunction

# 保存函数到 ~/.config/fish/functions/
$ funcsave myfunction

# 显示函数定义
$ functions myfunction

# 列出所有函数
$ functions
```

### 有用的函数示例

**CD 和 LS**：
```fish
function cd
    builtin cd $argv
    and ls
end
```

**创建目录并进入**：
```fish
function mkcd
    mkdir -p $argv
    and cd $argv
end
```

**解压归档**：
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

## 缩写

缩写在你按空格或回车时展开。与别名不同，你在运行之前可以看到完整的命令。

### 创建缩写

```fish
# 添加缩写
$ abbr --add gco 'git checkout'
$ abbr --add gst 'git status'
$ abbr --add dc 'docker-compose'

# 通用缩写（持久）
$ abbr --add --global gp 'git push'
```

### 使用缩写

```fish
$ gco <Space>               # 展开为: git checkout 
$ gst <Enter>               # 展开并运行: git status
```

### 管理缩写

```fish
# 列出缩写
$ abbr --show

# 删除缩写
$ abbr --erase gco

# 重命名缩写
$ abbr --rename gco gc
```

### 常用缩写

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

# 系统
abbr --add ll 'ls -la'
abbr --add la 'ls -A'
abbr --add l 'ls -CF'
```

## 提示符自定义

### 内置提示符

```fish
# 查看可用提示符
$ fish_config prompt

# 或从命令行选择
$ fish_config prompt choose arrow
$ fish_config prompt choose informative
$ fish_config prompt choose robbyrussell
```

### 自定义提示符

创建 `~/.config/fish/functions/fish_prompt.fish`：

**简单提示符**：
```fish
function fish_prompt
    echo (set_color cyan)(prompt_pwd) (set_color normal)'❯ '
end
```

**Git 感知提示符**：
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

### 右侧提示符

创建 `~/.config/fish/functions/fish_right_prompt.fish`：

```fish
function fish_right_prompt
    # 如果 > 5 秒则显示命令持续时间
    set -l duration $CMD_DURATION
    if test $duration -gt 5000
        set duration (math $duration / 1000)
        echo (set_color yellow)$duration's'(set_color normal)
    end
end
```

### 提示符工具

```fish
# 显示当前目录（缩短）
prompt_pwd

# 显示主机名
prompt_hostname

# 获取 git 分支
fish_git_prompt

# 显示命令状态
fish_status_prompt
```

## 键绑定

### 查看键绑定

```fish
# 列出所有绑定
$ bind

# 搜索绑定
$ bind | grep history

# 显示特定键的绑定
$ bind \cr                  # 显示 Ctrl+R 绑定
```

### 创建键绑定

```fish
# 将键绑定到命令
$ bind \cg 'git status'

# 用函数绑定
function my_function
    echo "Custom function"
    commandline -f repaint
end
bind \cx my_function
```

### 默认键绑定

**导航**：
```
Ctrl+A          # 移到行首
Ctrl+E          # 移到行尾
Alt+←/→         # 按词移动
Ctrl+K          # 删除到行尾
Ctrl+U          # 删除到行首
```

**历史**：
```
↑/↓             # 上一个/下一个命令
Ctrl+R          # 历史搜索
Alt+↑/↓         # 上一个/下一个标记
Alt+.           # 插入最后一个参数
```

**编辑**：
```
Alt+E           # 在 $EDITOR 中编辑
Alt+V           # 在 $VISUAL 中编辑
Ctrl+C          # 取消命令
Ctrl+D          # 删除字符或退出
```

**补全**：
```
Tab             # 补全或显示补全
Shift+Tab       # 上一个补全
Ctrl+→          # 接受自动建议的一个词
→               # 接受完整的自动建议
```

## 脚本编写基础

### Fish 脚本

**创建脚本**（`script.fish`）：
```fish
#!/usr/bin/env fish

# 注释以 # 开头

echo "Hello from Fish!"

# 变量
set name "World"
echo "Hello, $name!"

# 命令替换
set current_time (date)
echo "Current time: $current_time"
```

**使其可执行**：
```bash
chmod +x script.fish
./script.fish
```

### 控制结构

**If 语句**：
```fish
if test $status -eq 0
    echo "Success"
else if test $status -eq 1
    echo "Failed"
else
    echo "Unknown status"
end

# 简短形式
test -f file.txt; and echo "File exists"
test -d /tmp; or echo "Not a directory"
```

**Switch 语句**：
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

**循环**：
```fish
# For 循环
for file in *.txt
    echo "Processing $file"
end

# While 循环
set i 0
while test $i -lt 10
    echo $i
    set i (math $i + 1)
end

# 遍历命令输出
for line in (cat file.txt)
    echo "Line: $line"
end
```

### 测试条件

```fish
test -f file              # 文件存在
test -d dir               # 目录存在
test -x file              # 文件可执行
test -z "$var"            # 字符串为空
test -n "$var"            # 字符串非空
test "$a" = "$b"          # 字符串相等
test "$a" != "$b"         # 字符串不相等
test $a -eq $b            # 数字相等
test $a -ne $b            # 数字不相等
test $a -gt $b            # 大于
test $a -lt $b            # 小于
```

### 输入/输出

**读取输入**：
```fish
read -P "Enter name: " name
echo "Hello, $name!"

# 逐行读取
while read line
    echo "Line: $line"
end < file.txt
```

**重定向输出**：
```fish
echo "text" > file.txt          # 覆盖
echo "text" >> file.txt         # 追加
command 2> error.log            # 重定向 stderr
command > output.txt 2>&1       # 重定向两者
command &> all.log              # 重定向两者（简写）
```

## 与其他 Shell 的比较

### Fish vs Bash

| 功能 | Fish | Bash |
|------|------|------|
| **语法高亮** | 内置 | 需要插件 |
| **自动建议** | 内置 | 需要插件 |
| **Tab 补全** | 默认工作 | 需要配置 |
| **配置** | Web UI + 配置文件 | 仅配置文件 |
| **脚本语法** | 现代、干净 | POSIX、神秘 |
| **POSIX 兼容** | 否 | 是 |
| **可移植性** | 较少可移植 | 通用 |
| **变量语法** | `set VAR value` | `VAR=value` |
| **导出** | `set -x` | `export` |
| **数组** | 原生列表 | 需要特殊语法 |

**示例比较**：

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

| 功能 | Fish | Zsh |
|------|------|-----|
| **开箱体验** | 优秀 | 需要配置 |
| **语法高亮** | 内置 | 插件 (zsh-syntax-highlighting) |
| **自动建议** | 内置 | 插件 (zsh-autosuggestions) |
| **POSIX 兼容** | 否 | 是 |
| **插件生态** | 较小 | 巨大 (Oh My Zsh) |
| **学习曲线** | 平缓 | 较陡 |
| **脚本** | 非 POSIX | POSIX + 扩展 |

**何时选择 Fish**：
- 你想要无需配置的优秀默认设置
- 交互式使用是你的首要任务
- 你可以接受非 POSIX 脚本语法

**何时选择 Zsh**：
- 你需要 POSIX 兼容性
- 你想要最大化定制（Oh My Zsh）
- 你编写可移植脚本

### Bash 脚本兼容性

Fish **不**兼容 bash。常见问题：

**变量赋值**：
```fish
# Bash
VAR=value

# Fish
set VAR value
```

**导出**：
```fish
# Bash
export PATH=$PATH:/new/path

# Fish
set -x PATH $PATH /new/path
# 或
fish_add_path /new/path
```

**条件语句**：
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

**命令替换**：
```fish
# Bash
result=$(command)
result=`command`

# Fish
set result (command)
```

**在 Fish 中运行 Bash 脚本**：
```fish
# 用 bash 执行
bash script.sh

# Source bash 脚本（使用变通方法）
bass source script.sh  # 需要 bass 插件
```

## 插件和工具

### Fisher（插件管理器）

**安装 Fisher**：
```fish
curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher
```

**使用 Fisher**：
```fish
# 安装插件
fisher install jorgebucaran/nvm.fish

# 更新所有插件
fisher update

# 移除插件
fisher remove jorgebucaran/nvm.fish

# 列出插件
fisher list
```

### 有用的插件

**nvm.fish**（Node 版本管理器）：
```fish
fisher install jorgebucaran/nvm.fish
nvm install 18
nvm use 18
```

**z**（目录跳转器）：
```fish
fisher install jethrokuan/z
# 使用: z frequently_used_dir
```

**fzf.fish**（模糊查找器）：
```fish
fisher install PatrickF1/fzf.fish
# Ctrl+Alt+F: 查找文件
# Ctrl+Alt+L: 搜索历史
# Ctrl+Alt+V: 搜索变量
```

**bass**（运行 Bash 脚本）：
```fish
fisher install edc/bass
bass source ~/.bashrc
```

**tide**（提示符）：
```fish
fisher install IlanCosman/tide
tide configure
```

### Starship 提示符

**安装 Starship**：
```bash
curl -sS https://starship.rs/install.sh | sh
```

**配置 Fish 使用 Starship**（`~/.config/fish/config.fish`）：
```fish
starship init fish | source
```

**配置 Starship**（`~/.config/starship.toml`）：
```toml
[character]
success_symbol = "[➜](bold green)"
error_symbol = "[✗](bold red)"

[directory]
truncation_length = 3
truncate_to_repo = true
```

## 最佳实践

### 交互式使用

1. **使用缩写而非别名**：
   - 你可以看到运行的内容
   - 更容易记住
   - 执行前可编辑

2. **利用自动建议**：
   - 按 `→` 接受
   - 按 `Ctrl+→` 接受一个词
   - 训练你的肌肉记忆

3. **自定义你的提示符**：
   - 显示 git 状态
   - 显示退出状态
   - 保持清晰和可读

4. **使用通用变量持久化**：
   ```fish
   set -U fish_greeting ""
   set -U EDITOR vim
   ```

5. **组织函数**：
   - `~/.config/fish/functions/` 中每个函数一个文件
   - 使用 `funced` 和 `funcsave`

### 脚本编写

1. **使用 Bash 编写可移植脚本**：
   - Fish 脚本不会在其他系统上运行
   - 使用 `#!/usr/bin/env bash` 保证可移植性

2. **使用 Fish 编写个人脚本**：
   - 比 bash 语法更清晰
   - 更好的错误处理
   - 现代语言特性

3. **偏好函数而非脚本**：
   - 函数自动加载
   - 用 `funced` 更容易编辑
   - 可以轻松分享

4. **对复杂参数使用 `argparse`**：
   ```fish
   function mycommand
       argparse 'h/help' 'v/verbose' -- $argv
       or return
       
       # 处理标志
   end
   ```

### 配置

1. **保持 config.fish 最小化**：
   - 只放必需的东西
   - 复杂逻辑使用函数
   - 记录你的选择

2. **明智使用通用变量**：
   - 适合：颜色、编辑器、路径
   - 不适合：特定于机器的设置

3. **版本控制你的配置**：
   ```bash
   cd ~/.config/fish
   git init
   git add config.fish functions/
   git commit -m "Initial fish config"
   ```

## 故障排除

### 常见问题

**启动缓慢**：
```fish
# 分析启动时间
fish --profile prompt.prof -c exit
cat prompt.prof

# 常见原因:
# - $PATH 中项目太多
# - config.fish 中的慢函数
# - 提示符中的 Git 操作
```

**PATH 问题**：
```fish
# 查看 PATH
echo $PATH

# 重置 PATH
set -e PATH
set -U fish_user_paths /usr/local/bin /usr/bin /bin

# 正确添加路径
fish_add_path /new/path
```

**补全不工作**：
```fish
# 重建补全
fish_update_completions

# 清除补全缓存
rm ~/.local/share/fish/generated_completions/*
```

**颜色问题**：
```fish
# 检查终端支持 256 色
echo $TERM

# 重置颜色
set -e fish_color_{command,param,error}
```

### 获取帮助

```fish
# Fish 文档
man fish

# 命令帮助
man command_name

# Fish 教程
fish_tutorial

# 在线文档
open https://fishshell.com/docs/current/

# 社区
# - GitHub: https://github.com/fish-shell/fish-shell
# - Gitter: https://gitter.im/fish-shell/fish-shell
```

## 从 Bash/Zsh 迁移

### 转换 Bash 脚本

**变量**：
```fish
# Bash: VAR=value
set VAR value

# Bash: export VAR=value
set -x VAR value
```

**条件语句**：
```fish
# Bash: if [ -f file ]; then ... fi
if test -f file
    # ...
end
```

**循环**：
```fish
# Bash: for i in $(seq 1 10); do ... done
for i in (seq 1 10)
    # ...
end
```

**函数**：
```fish
# Bash: function name() { ... }
function name
    # ...
end
```

### 别名转缩写

```fish
# 转换 bash 别名
# alias ll='ls -la'
abbr --add ll 'ls -la'

# 批量转换脚本
for line in (cat ~/.bash_aliases | grep alias)
    # 解析并创建缩写
end
```

### 环境设置

```fish
# 移植 .bashrc/.zshrc
# 1. 转换 exports
# 2. 转换别名为缩写
# 3. 用 Fish 语法重写函数
# 4. 测试每一部分

# 示例:
# bash: export EDITOR=vim
set -Ux EDITOR vim

# bash: alias g=git
abbr --add g git
```

## 参考资源

### 官方文档

- [Fish Shell 网站](https://fishshell.com/)
- [官方文档](https://fishshell.com/docs/current/)
- [教程](https://fishshell.com/docs/current/tutorial.html)
- [给 bash 用户的 Fish](https://fishshell.com/docs/current/fish_for_bash_users.html)
- [GitHub 仓库](https://github.com/fish-shell/fish-shell)

### 社区资源

**插件仓库**：
- [Fisher 插件](https://github.com/jorgebucaran/fisher)
- [Awesome Fish](https://github.com/jorgebucaran/awsm.fish)
- [Fish 插件](https://github.com/topics/fish-plugin)

**教程和指南**：
- [Fish Shell Cookbook](https://github.com/jorgebucaran/cookbook.fish)
- [Learn Fish in Y Minutes](https://learnxinyminutes.com/docs/fish/)

**主题和提示符**：
- [Oh My Fish Themes](https://github.com/oh-my-fish/oh-my-fish/blob/master/docs/Themes.md)
- [Tide Prompt](https://github.com/IlanCosman/tide)
- [Starship Prompt](https://starship.rs/)

### 工具和集成

**编辑器**：
- [fish-shell/fish-shell VSCode](https://marketplace.visualstudio.com/items?itemName=bmalehorn.vscode-fish)
- [vim-fish](https://github.com/dag/vim-fish)
- [emacs-fish](https://github.com/wwwjfy/emacs-fish)

**有用的工具**：
- [fzf](https://github.com/junegunn/fzf) - 模糊查找器
- [exa](https://github.com/ogham/exa) - 现代 ls 替代品
- [bat](https://github.com/sharkdp/bat) - 更好的 cat
- [fd](https://github.com/sharkdp/fd) - 更好的 find
- [ripgrep](https://github.com/BurntSushi/ripgrep) - 更好的 grep

### 书籍和文章

- Fish Shell 官方教程
- "Modern Shell Environments" 博客文章
- Stack Overflow [fish 标签](https://stackoverflow.com/questions/tagged/fish)

## 结论

### 为什么 Fish 很棒

- **零配置**：开箱即用
- **用户友好**：为人类设计，不仅仅是脚本
- **现代**：根据当今需求构建
- **可发现**：功能易于查找和使用
- **有趣**：让终端工作愉快

### 何时使用什么

**使用 Fish**：
- 日常交互式终端工作
- 个人脚本和自动化
- 学习 shell 使用
- 愉快的终端体验

**使用 Bash**：
- 可移植脚本
- CI/CD 管道
- 系统脚本
- POSIX 要求

**两者都用**：
- Fish 作为交互式 shell
- Bash 用于脚本编写
- 两全其美

### 入门

```fish
# 安装 Fish
brew install fish  # 或你的包管理器

# 试试看
fish

# 喜欢吗？设为默认
chsh -s (which fish)

# 配置
fish_config

# 安装插件管理器
curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher

# 享受！
```

欢迎来到 Fish！🐟
