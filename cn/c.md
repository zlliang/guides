# C 编程语言

C 语言是计算历史上最具影响力且应用最广泛的编程语言之一。它由 Dennis Ritchie 于 20 世纪 70 年代初在贝尔实验室创建，最初设计用于系统编程和实现 UNIX 操作系统。尽管历史悠久，C 语言仍然是现代基础设施的支柱，支撑着操作系统、嵌入式系统、高性能服务器和语言运行时。

本指南作为一份全面的参考资料和教程，涵盖了从基础语法到最先进的系统级功能的所有内容。它重点介绍了现代标准（C99、C11、C23），并强调了安全、高效的编码实践。

## 快速参考

| 类别 | 关键语法 / 命令 | 示例 |
| :--- | :--- | :--- |
| **输出** | `printf(fmt, ...)` | `printf("Value: %d\n", x);` |
| **输入** | `scanf(fmt, ...)` | `scanf("%d", &x);` |
| **分配** | `malloc(size)`, `free(ptr)` | `int *p = malloc(sizeof(int) * 10);` |
| **类型** | `int`, `char`, `float`, `void *` | `uint64_t id = 0xFF;` |
| **修饰符** | `const`, `static`, `volatile` | `static const int MAX = 100;` |
| **指针** | `*` (取值), `&` (取地址) | `int *ptr = &val;` |
| **结构体** | `struct Name { ... };` | `struct Point { int x, y; };` |
| **预处理** | `#include`, `#define`, `#ifdef` | `#define MIN(a,b) ((a)<(b)?(a):(b))` |

## 历史与标准

了解您正在使用的 C 语言版本至关重要，因为功能和安全保证已经发生了显著变化。

### K&R C (标准前)
《C 程序设计语言》第一版（1978 年）中描述的原始版本。现已过时。
*   *遗留特征：* 函数通常缺少返回类型（默认为 `int`）和声明中的参数类型列表。

### C89 / C90 (ANSI C)
第一个标准化版本 (ANSI X3.159-1989 / ISO/IEC 9899:1990)。
*   **关键特性：** 函数原型、`void` 指针、`enum`、`const` 和 `volatile`。
*   *状态：* 仍在旧的嵌入式系统中广泛支持，但通常被认为已不适合现代开发。

### C99 (ISO/IEC 9899:1999)
一次重大更新，引入了许多今天仍在使用的功能。
*   **关键特性：** `//` 注释、任意位置变量声明（不仅仅是作用域顶部）、`long long`、`<stdint.h>`、变长数组 (VLA)、`restrict` 限定符、复合字面量、指定初始化器、柔性数组成员。

### C11 (ISO/IEC 9899:2011)
专注于线程安全和对齐。
*   **关键特性：** 多线程支持 (`<threads.h>`)、原子操作 (`<stdatomic.h>`)、`_Generic` (泛型宏)、`_Static_assert`、`_Alignas`/`_Alignof`，移除了 `gets()`。

### C17 / C18
C11 的错误修复版本，没有新的语言特性。

### C23 (预期)
下一个主要修订版。
*   **关键特性：** `typeof`、`constexpr`、`nullptr`、`true`/`false` 关键字（使 `<stdbool.h>` 不再必须）、标准属性 (`[[nodiscard]]`, `[[deprecated]]`)、二进制字面量 (`0b1010`)、`#embed`。

## 编译模型

C 是一种编译型语言。从源代码到可执行文件的转换涉及几个不同的阶段。

### 1. 预处理 (Preprocessing)
预处理器 (`cpp`) 处理以 `#` 开头的指令。
*   **文本替换：** 宏 (`#define`) 被展开。
*   **文件包含：** `#include` 插入文件内容。
*   **条件编译：** `#if`、`#ifdef` 选择代码块。
*   **输出：** 一个“翻译单元” (translation unit)，即没有指令的纯 C 代码。

### 2. 编译 (Compilation)
编译器（GCC 中的 `cc1`）将 C 代码翻译成汇编语言 (`.s`)。
*   **语法检查：** 验证语法。
*   **静态分析：** 检查类型、作用域规则并发出警告。
*   **优化：** 重排指令、内联函数、删除死代码。

### 3. 汇编 (Assembly)
汇编器 (`as`) 将汇编代码翻译成机器码，生成目标文件 (`.o` 或 `.obj`)。
*   目标文件包含二进制代码，但对外部函数（如 `printf`）的符号引用仍未解析。

### 4. 链接 (Linking)
链接器 (`ld`) 组合目标文件和库。
*   **解析：** 将函数调用与其定义匹配。
*   **重定位：** 分配最终内存地址。
*   **输出：** 可执行二进制文件。

## 基本语法与结构

### 入口点
每个 C 程序都从 `main` 开始执行。

```c
#include <stdio.h>

// 标准签名：返回 int，接受参数
int main(int argc, char *argv[]) {
    printf("程序名称: %s\n", argv[0]);
    return 0; // 0 = 成功，非零 = 失败
}
```

*   `argc`: 参数计数 (Argument Count)。
*   `argv`: 参数向量 (Argument Vector)（字符串数组）。

### 语句与块
*   **语句** 以分号 `;` 结束。
*   **块** 包围在花括号 `{ ... }` 中并定义一个作用域。

```c
{
    int x = 5; // x 仅在此块内存在
}
// x 在此处不可访问
```

### 注释
*   `// 单行注释` (C99+)
*   `/* 多行注释 */` (C89+)

## 变量与数据类型

C 语言的类型与硬件字密切对应。大小是实现定义的，但严格遵循顺序 (`short` ≤ `int` ≤ `long`)。

### 整数类型

| 类型 | 典型大小 (32/64位) | 说明符 (printf) | 范围 (无符号) |
| :--- | :--- | :--- | :--- |
| `char` | 1 字节 | `%c`, `%hhd` | 0 到 255 |
| `short` | 2 字节 | `%hd` | 0 到 65,535 |
| `int` | 4 字节 | `%d` | 0 到 4,294,967,295 |
| `long` | 4 或 8 字节 | `%ld` | 取决于 OS (LP64 vs LLP64) |
| `long long` | 8 字节 | `%lld` | 0 到 ~1.8 × 10^19 |

**最佳实践：** 对于精确的宽度要求，使用 `<stdint.h>`。
*   `int32_t`, `uint64_t`, `int8_t`, `uint16_t`。
*   `intptr_t`: 足够容纳指针的整数。
*   `size_t`: 用于对象大小的无符号整数（`sizeof` 的结果）。

### 浮点类型

| 类型 | 描述 | 典型大小 | 说明符 |
| :--- | :--- | :--- | :--- |
| `float` | 单精度 | 4 字节 | `%f` |
| `double` | 双精度 | 8 字节 | `%lf` |
| `long double` | 扩展精度 | 10/12/16 字节 | `%Lf` |

### Void 类型
`void` 表示没有值。
*   **函数返回：** `void func()` (不返回任何内容)。
*   **函数参数：** `int func(void)` (不接受任何参数)。
*   **指针：** `void *ptr` (通用指针，解引用前必须转换)。

### 常量与字面量
*   **十进制：** `123`
*   **十六进制：** `0x7B`
*   **八进制：** `0173` (以 0 开头 - **小心！** `010` 是 8，不是 10)
*   **二进制：** `0b1111011` (C23)
*   **后缀：** `10u` (无符号), `10L` (长整型), `10.0f` (浮点型)。

## 运算符与表达式

### 算术运算
`+`, `-`, `*`, `/`, `%` (取模)。
*   整数除法会截断 (`5 / 2` 结果是 `2`)。
*   取模 `%` 仅适用于整数。

### 位运算符
对系统编程至关重要。
*   `&` (与): 清除位。
*   `|` (或): 设置位。
*   `^` (异或): 切换位。
*   `~` (非): 反转位（一的补码）。
*   `<<` (左移): 乘以 2^n。
*   `>>` (右移): 除以 2^n。
    *   **注意：** 对有符号负数进行右移是实现定义的（通常是算术移位，保留符号）。

### 逻辑运算符
*   `&&` (与), `||` (或), `!` (非)。
*   **短路求值：** 在 `A && B` 中，如果 `A` 为假，则不会计算 `B`。这是一种保证行为。

### 自增/自减
*   `i++` (后自增): 返回值，然后增加。
*   `++i` (前自增): 增加，然后返回值。
*   **未定义行为：** `i = i++` 或 `func(i++, i++)`。不要在一个序列点内修改变量两次。

### 三元运算符
`condition ? value_if_true : value_if_false`

```c
int max = (a > b) ? a : b;
```

## 控制流

### If-Else

```c
if (temperature > 30) {
    printf("炎热\n");
} else if (temperature < 10) {
    printf("寒冷\n");
} else {
    printf("适中\n");
}
```

### Switch-Case
用于根据整数常量检查变量，效率很高。

```c
switch (status) {
    case 200:
        printf("OK\n");
        break; // 别忘了 break!
    case 404:
        printf("未找到\n");
        break;
    default:
        printf("未知\n");
}
```
**贯穿 (Fallthrough)：** 如果省略 `break`，执行将继续到下一个 case。

### 循环

**For 循环：**
```c
for (int i = 0; i < 10; i++) {
    if (i == 5) continue; // 跳过本次迭代
    if (i == 8) break;    // 退出循环
    printf("%d ", i);
}
```

**While 循环：**
```c
while (ptr != NULL) {
    ptr = ptr->next;
}
```

**Do-While 循环：**
保证至少执行一次。
```c
do {
    input = read_sensor();
} while (input < THRESHOLD);
```

### Goto
将控制权转移到带标签的语句。
*   **用法：** 主要用于复杂函数中的错误清理。

```c
    void *p = malloc(100);
    if (!p) goto error;
    
    if (do_work() != 0) goto cleanup;

    free(p);
    return 0;

cleanup:
    // 回滚工作
error:
    free(p);
    return -1;
```

## 指针与内存

指针是存储内存地址的变量。它们提供了同等程度的能力和危险。

### 声明与解引用

```c
int val = 42;
int *p = &val;  // & = 取地址运算符
int x = *p;     // * = 解引用运算符（获取地址处的值）
```

### 指针算术
指针知道它们所指向类型的大小。
*   如果 `p` 是 `int *` (4 字节)，`p + 1` 会将地址加 4。
*   如果 `p` 是 `char *` (1 字节)，`p + 1` 会将地址加 1。

### Void 指针
`void *` 是通用指针。它保存地址但没有类型信息。
*   不能解引用。
*   不能进行算术运算（不进行转换的情况下）。
*   用于 `malloc` 和 `memcpy` 等函数。

### 空指针 (Null Pointers)
值为 `0` 或 `NULL` ((void*)0) 的指针。
*   解引用它会导致段错误 (crash)。
*   始终检查指针：`if (ptr) { ... }`。

### 函数指针
指针可以指向可执行代码。

```c
// 声明：一个指针 'fn' 指向接受 (int, int) 并返回 int 的函数
int (*fn)(int, int);

int add(int a, int b) { return a + b; }

fn = add;
int result = fn(5, 10); // 调用 add(5, 10)
```

用例：回调、中断处理程序、排序 (qsort)、动态分派。

## 数组与字符串

### 数组
连续的内存块。

```c
int arr[5] = {1, 2, 3, 4, 5};
// arr 在大多数上下文中退化为 &arr[0]
```

**多维数组：**
`int grid[3][3]` 是一个包含 3 个数组（每个数组包含 3 个整数）的数组。
内存布局是**行主序**（线性的）。

### 字符串
C 字符串是以**空字符** (`\0`) 结尾的字符数组。

```c
char str[] = "Hello"; // 大小为 6 ('H','e','l','l','o','\0')
char *ptr = "World";  // 字符串字面量（只读内存！）
```

**重要的字符串函数 (`<string.h>`):**
*   `strlen(s)`: 长度，不包括 `\0`。
*   `strcpy(dst, src)`: 复制字符串。**不安全**（缓冲区溢出风险）。
*   `strncpy(dst, src, n)`: 更安全，但棘手（可能不以空字符结尾）。
*   `strcat(dst, src)`: 连接。
*   `strcmp(s1, s2)`: 比较（如果相等返回 0）。
*   `strdup(s)`: 复制字符串（分配内存，用户必须释放）。

**安全提示：** 优先使用接受长度参数的函数（如 `snprintf`），而不是无边界函数（如 `sprintf` 或 `strcpy`）。

## 内存管理

C 程序使用三种类型的内存：

### 1. 静态 / 全局 (Static / Global)
*   **位置：** 数据段或 BSS。
*   **生命周期：** 整个程序执行期间。
*   **作用域：** 全局变量，`static` 局部变量。

### 2. 栈 (Stack / Automatic)
*   **位置：** 栈帧。
*   **生命周期：** 函数作用域。函数返回时销毁。
*   **特点：** 分配快，大小有限（栈溢出风险）。

### 3. 堆 (Heap / Dynamic)
*   **位置：** 托管堆。
*   **生命周期：** 手动（直到被释放）。
*   **用法：** 大对象，生命周期超出函数作用域的对象。

### 动态分配函数 (`<stdlib.h>`)

*   `malloc(size_t size)`: 分配 `size` 字节。内容是垃圾值。
*   `calloc(size_t n, size_t size)`: 分配 `n * size` 字节。**初始化为零**。
*   `realloc(void *ptr, size_t new_size)`: 调整块大小。可能会将数据移动到新位置。
*   `free(void *ptr)`: 释放内存。

**示例：**

```c
struct Data *items = malloc(count * sizeof(struct Data));
if (!items) {
    perror("Malloc failed");
    exit(1);
}

// ... 使用 items ...

free(items);
items = NULL; // 防止使用悬空指针
```

### 常见内存错误
1.  **泄漏 (Leak):** 忘记 `free`。
2.  **重复释放 (Double Free):** 对同一个指针调用两次 `free`。
3.  **释放后使用 (Use After Free):** 在 `free(ptr)` 之后访问 `ptr`。
4.  **缓冲区溢出 (Buffer Overflow):** 写入超过分配的大小。

诸如 **Valgrind** 或 **AddressSanitizer** (`-fsanitize=address`) 之类的工具对于检测这些错误至关重要。

## 结构体与联合体

### 结构体 (`struct`)
组合相关的数据类型。

```c
struct Pixel {
    uint8_t r, g, b;
};

// Typedef 使使用更整洁
typedef struct {
    float x, y;
} Point;

Point p = {1.0f, 2.0f}; // 指定初始化 (C99): {.y=2.0f, .x=1.0f}
```

**填充与对齐：**
编译器插入“填充”字节，以将成员对齐到对架构友好的地址。
```c
struct Bad {
    char c;      // 1 字节
    // 此处隐含 3 字节填充
    int i;       // 4 字节
}; // sizeof = 8
```
为了最小化大小，将成员按从大到小的对齐顺序排列。

### 联合体 (Unions)
允许在同一内存位置存储不同的数据类型。

```c
union Variant {
    int i;
    float f;
    char c;
};
```
一次只有一个成员有效。大小是 `max(sizeof(members))`。

### 位域 (Bit-fields)
将变量打包成特定数量的位。

```c
struct Flags {
    unsigned int visible : 1;
    unsigned int editable : 1;
    unsigned int layer : 6; // 0-63
};
```
**警告：** 顺序依赖于实现（先 LSB 还是 MSB）。

## 预处理器深度解析

预处理器在编译之前运行。

### 宏
宏执行文本替换。

```c
#define MAX_SIZE 1024
#define SQUARE(x) ((x) * (x))
```
**陷阱：** `SQUARE(a++)` 展开为 `((a++) * (a++))`，导致 `a` 增加两次（未定义行为）。

### 头文件保护
防止头文件被多次包含。

```c
#ifndef MY_HEADER_H
#define MY_HEADER_H

// 声明...

#endif
```
现代编译器支持 `#pragma once` 达到同样目的。

### 条件编译
对于跨平台代码很有用。

```c
#ifdef _WIN32
    #include <windows.h>
#else
    #include <unistd.h>
#endif
```

### X-Macros
一种为列表生成代码的模式。

```c
#define COLORS \
    X(RED, 0) \
    X(GREEN, 1) \
    X(BLUE, 2)

enum Color {
    #define X(name, val) name = val,
    COLORS
    #undef X
};
```

## 标准库

C 附带了一个小巧但必不可少的标准库。

### 输入 / 输出 (`stdio.h`)
*   **文件句柄：** `FILE *` (不透明结构体)。
*   **标准流：** `stdin`, `stdout`, `stderr`。
*   **操作：** `fopen`, `fclose`, `fread`, `fwrite`, `fprintf`, `fscanf`。

**缓冲：**
*   `_IOFBF` (全缓冲): 写入保存直到缓冲区满（文件）。
*   `_IOLBF` (行缓冲): 换行时刷新 (stdout)。
*   `_IONBF` (无缓冲): 立即刷新 (stderr)。

### 字符串处理 (`string.h`)
操作 C 字符串和原始内存 (`memcpy`, `memset`, `memmove`)。
*   **注意：** `memcpy` 假设区域不重叠。如果区域重叠，请使用 `memmove`。

### 字符处理 (`ctype.h`)
*   `isalnum`, `isalpha`, `isdigit`, `isspace`, `toupper`, `tolower`.

### 通用工具 (`stdlib.h`)
*   **转换：** `atoi`, `strtol`, `strtod`.
*   **随机数：** `rand`, `srand` (如果有，使用现代替代品)。
*   **进程：** `exit`, `abort`, `system`.
*   **排序/搜索：** `qsort`, `bsearch`.

### 断言 (`assert.h`)
运行时检查。
```c
assert(ptr != NULL);
```
在包含 `<assert.h>` 之前使用 `#define NDEBUG` 可禁用断言。

## 高级主题

### 变参函数 (`stdarg.h`)
接受可变数量参数的函数（如 `printf`）。

```c
void log_msg(int count, ...) {
    va_list args;
    va_start(args, count);
    for(int i=0; i<count; i++) {
        int val = va_arg(args, int);
        // ...
    }
    va_end(args);
}
```

### 类型限定符

*   **`const`**: 只读。
    *   `const int *p`: 数据是 const。
    *   `int * const p`: 指针是 const。
*   **`volatile`**: 防止编译器优化。用于内存映射硬件 I/O 或中断处理程序修改的标志。
*   **`restrict`** (C99): 承诺指针不别名（重叠）。启用向量化优化。

### 存储类说明符

*   **`auto`**: 栈变量的默认值。
*   **`register`**: 提示存储在 CPU 寄存器中（现代编译器大多忽略）。
*   **`static`**:
    *   全局作用域：文件私有可见性。
    *   局部作用域：跨函数调用保留值。
*   **`extern`**: 声明在别处定义的对象。

### Setjmp / Longjmp (`setjmp.h`)
非本地跳转（跨函数的 goto）。保存栈上下文。
用于 C 中的异常风格错误处理，但很棘手（不会自动清理栈变量）。

## 现代 C 特性 (C99 / C11 / C23)

### C99
*   **复合字面量：** 传递没有变量的结构体。
    ```c
    draw_point((struct Point){.x=10, .y=20});
    ```
*   **变长数组 (VLA):**
    ```c
    void func(int n) { int arr[n]; ... }
    ```
    *警告：* 可能导致栈溢出。在安全代码中通常不鼓励使用。
*   **指定初始化器：**
    ```c
    int arr[10] = {[0]=5, [4]=10};
    ```

### C11
*   **_Generic:** 编译时多态。
    ```c
    #define print(x) _Generic((x), int: print_int, float: print_float)(x)
    ```
*   **线程 (`threads.h`):** 可移植线程 (thrd_create, mtx_lock)。
*   **原子操作 (`stdatomic.h`):** `_Atomic int counter;` 用于无锁同步。

### C23 (即将到来)
*   **类型推断：** `auto x = 10;` (像 C++)。
*   **Typeof:** `typeof(x) y = x;`
*   **属性：** `[[nodiscard]] int important_func();`
*   **位精确整数：** `_BitInt(N)`.

## 未定义行为 (Undefined Behavior, UB)

C 语言最独特的方面之一是“未定义行为”。标准允许编译器假设某些错误*永远不会发生*。如果发生了，程序可以做任何事情（崩溃、静默工作、损坏数据）。

**常见的 UB 来源：**
1.  **有符号整数溢出：** `INT_MAX + 1` 是 UB。（无符号溢出定义为模运算回绕）。
2.  **解引用空/未初始化指针。**
3.  **除以零。**
4.  **移动负数位。**
5.  **修改 const 数据**（通过转换）。
6.  **违反严格别名 (Strict Aliasing)：** 作为 int 指针访问 float。

**为什么存在 UB：** 它允许激进的优化。编译器可以假设循环计数器不会回绕，指针指向有效内存等。

## 最佳实践

### 1. 安全第一
*   初始化变量：`int x = 0;`。
*   使用前检查指针有效性。
*   使用安全的字符串函数（`snprintf` 而不是 `sprintf`）。
*   验证所有外部输入。

### 2. 代码组织
*   **头文件 (.h):** 声明、宏、结构体。
*   **源文件 (.c):** 实现。
*   **静态作用域：** 将辅助函数标记为 `static` 以避免污染全局命名空间。

### 3. Const 正确性
积极使用 `const`。它记录了意图并允许优化。
```c
// 不好
void print_data(char *str);
// 好
void print_data(const char *str);
```

### 4. 错误处理
C 缺乏异常。常见模式：
*   **返回码：** 0 表示成功，负数表示错误。
*   **Errno:** 全局错误变量（检查 `<errno.h>` 中的 `errno`）。
*   **输出参数：** 返回状态，通过指针传递结果。

### 5. 不透明指针 (Opaque Pointers)
隐藏实现细节（封装）。
*   **头文件：** `typedef struct Handle_T *Handle;`
*   **源文件：** `struct Handle_T { int internal_data; };`
用户只能看到指针 `Handle`，保持 ABI 稳定性。

## 工具与调试

### 编译标志
始终启用警告进行编译。
*   **GCC/Clang:** `-Wall -Wextra -pedantic -Wconversion -Wshadow`
*   **调试符号:** `-g` (GDB 需要)。
*   **优化:** `-O0` (调试), `-O2` (发布), `-O3` (激进)。

### 调试器 (GDB / LLDB)
*   `break main`: 设置断点。
*   `run`: 启动程序。
*   `next` / `step`: 单步跳过 / 进入。
*   `print x`: 检查变量。
*   `bt`: 回溯 (调用栈)。

### Sanitizers
现代编译器包含运行时检查器。
*   `-fsanitize=address` (ASan): 检测缓冲区溢出、释放后使用。
*   `-fsanitize=undefined` (UBSan): 检测 UB，如有符号溢出。
*   `-fsanitize=leak` (LSan): 检测内存泄漏。

### Valgrind
强大的插桩框架。
```bash
valgrind --leak-check=full ./my_program
```
它模拟 CPU 以跟踪每一位内存，发现 Sanitizers 可能错过的泄漏和无效访问（虽然它速度较慢）。

## 参考资料与资源

*   **标准：**
    *   [ISO/IEC 9899:2011 (C11 草案)](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf)
    *   [ISO/IEC 9899:2018 (C17 草案)](https://web.archive.org/web/20181230041359/http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2176.pdf)
*   **参考：**
    *   [cppreference.com (C)](https://zh.cppreference.com/w/c) - 优秀的在线文档。
    *   [SEI CERT C 编码标准](https://wiki.sei.cmu.edu/confluence/display/c) - 安全指南。
*   **书籍：**
    *   《C 程序设计语言（第 2 版）》 - Kernighan & Ritchie (经典)。
    *   《Modern C》 - Jens Gustedt (C11/C17 的最新视角)。
    *   《C 语言接口与实现》 - David R. Hanson (设计模式)。
    *   《C 专家编程》 - Peter van der Linden。
