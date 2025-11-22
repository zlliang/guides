# C 语言编程指南

这是一份关于 C 语言的综合指南，涵盖了从基本语法到高级内存管理和系统级编程的所有内容。C 语言是一种功能强大、高效的语言，是现代计算的基石。

## 快速参考 (Quick Reference)

| 概念 | 语法 / 命令 | 示例 |
| :--- | :--- | :--- |
| **打印** | `printf(format, ...)` | `printf("Hello, %s\n", "World");` |
| **输入** | `scanf(format, ...)` | `scanf("%d", &x);` |
| **注释** | `//` 或 `/* ... */` | `// 单行注释` |
| **编译** | `gcc file.c -o output` | `gcc main.c -o main` |
| **数据类型** | `int`, `char`, `float`, `double`, `void` | `int age = 25;` |
| **指针** | `*` (取值), `&` (取地址) | `int *p = &x;` |
| **分配内存** | `malloc(size)` | `int *arr = malloc(5 * sizeof(int));` |
| **释放内存** | `free(ptr)` | `free(arr);` |

---

## 1. 简介 (Introduction)

C 是一种通用的过程式计算机编程语言，支持结构化编程、词法变量作用域和递归，并具有静态类型系统。C 语言的设计目的是提供能高效映射到典型机器指令的构造。

### 主要特点
*   **高效性 (Efficiency):** 生成快速、紧凑的代码。
*   **可移植性 (Portability):** 源代码可以在许多不同的平台上编译。
*   **底层访问 (Low-level Access):** 通过指针直接操作内存。
*   **简洁性 (Simplicity):** 核心语言小巧，但拥有丰富的库。

## 2. 基本结构 (Basic Structure)

标准的 C 程序由函数和变量组成。程序的入口点始终是 `main` 函数。

```c
#include <stdio.h> // 预处理指令

// Main 函数 - 入口点
int main() {
    // 打印到标准输出
    printf("Hello, World!\n");
    
    // 返回 0 表示执行成功
    return 0;
}
```

## 3. 变量与数据类型 (Variables & Data Types)

C 语言要求在使用变量之前必须先声明。

### 基本类型 (Primitive Types)

| 类型 | 描述 | 格式说明符 |
| :--- | :--- | :--- |
| `int` | 整数 (通常 4 字节) | `%d` |
| `char` | 字符 (1 字节) | `%c` |
| `float` | 单精度浮点数 (4 字节) | `%f` |
| `double` | 双精度浮点数 (8 字节) | `%lf` |
| `void` | 空 / 无类型 | N/A |

```c
int count = 10;
float price = 19.99;
char grade = 'A';
double precise_val = 3.14159265359;
```

## 4. 控制流 (Control Flow)

### 条件语句 (Conditionals)

```c
if (age >= 18) {
    printf("成年人\n");
} else if (age >= 13) {
    printf("青少年\n");
} else {
    printf("儿童\n");
}
```

### 开关语句 (Switch)

```c
switch (grade) {
    case 'A': printf("优秀\n"); break;
    case 'B': printf("良好\n"); break;
    default:  printf("其他\n");
}
```

### 循环 (Loops)

```c
// For 循环
for (int i = 0; i < 5; i++) {
    printf("%d ", i);
}

// While 循环
while (condition) {
    // 代码块
}

// Do-while 循环 (至少执行一次)
do {
    // 代码块
} while (condition);
```

## 5. 函数 (Functions)

函数将大型任务分解为更小、更易于管理的单元。

```c
// 函数声明 (原型)
int add(int a, int b);

int main() {
    int result = add(5, 3);
    return 0;
}

// 函数定义
int add(int a, int b) {
    return a + b;
}
```

## 6. 指针 (Pointers)

指针是存储内存地址的变量。它们是 C 语言最强大（但也最危险）的特性之一。

### 基础
*   `&` (取地址运算符): 返回变量的内存地址。
*   `*` (解引用运算符): 访问指针所存储地址处的值。

```c
int var = 20;
int *ptr = &var; // ptr 保存 var 的地址

printf("地址: %p\n", ptr);
printf("值: %d\n", *ptr); // 解引用

*ptr = 30; // 修改 var 的值
```

### 指针与数组
在 C 语言中，数组和指针关系密切。数组名充当指向其第一个元素的指针。

```c
int arr[3] = {10, 20, 30};
int *p = arr;

printf("%d", *p);     // 10
printf("%d", *(p+1)); // 20 (指针运算)
```

## 7. 数组与字符串 (Arrays & Strings)

### 数组
相同类型元素的固定大小集合。

```c
int numbers[5] = {1, 2, 3, 4, 5};
numbers[2] = 10;
```

### 字符串
在 C 语言中，字符串是以空字符 `\0` 结尾的字符数组。

```c
char name[] = "Alice"; // 自动在末尾添加 '\0'
// name 实际上是 ['A', 'l', 'i', 'c', 'e', '\0']

// 使用字符串库
#include <string.h>
strlen(name);   // 长度 (不包括 \0)
strcpy(dest, src); // 复制字符串
strcmp(s1, s2);    // 比较字符串
```

## 8. 内存管理 (Memory Management)

C 允许使用 `<stdlib.h>` 在堆 (heap) 上进行动态内存分配。

### 关键函数
*   `malloc(size)`: 分配 `size` 字节。返回 `void*`。
*   `calloc(n, size)`: 分配 `n` 个元素的空间，并初始化为零。
*   `realloc(ptr, size)`: 调整先前分配的内存大小。
*   `free(ptr)`: 释放分配的内存。

### 示例

```c
int *arr;
int n = 5;

// 为 5 个整数分配内存
arr = (int*) malloc(n * sizeof(int));

if (arr == NULL) {
    printf("内存分配失败\n");
    return 1;
}

// 使用内存...
for(int i=0; i<n; i++) arr[i] = i;

// 释放内存
free(arr);
arr = NULL; // 良好习惯，避免悬空指针
```

> **警告:** 始终记得 `free` 动态分配的内存，以防止内存泄漏。

## 9. 结构体 (Structures)

结构体将不同类型的相关变量组合在一起。

```c
struct Person {
    char name[50];
    int age;
    float salary;
};

// 用法
struct Person p1;
strcpy(p1.name, "John");
p1.age = 30;

// 初始化语法
struct Person p2 = {"Jane", 28, 55000.0};
```

### Typedef
用于为类型创建别名，通常与 struct 一起使用。

```c
typedef struct {
    int x;
    int y;
} Point;

Point p1 = {10, 20}; // 不需要写 'struct'
```

## 10. 文件 I/O (File I/O)

使用 `<stdio.h>` 处理文件。

```c
FILE *fp;

// 写入文件
fp = fopen("data.txt", "w");
if (fp != NULL) {
    fprintf(fp, "Testing...\n");
    fclose(fp);
}

// 读取文件
char buffer[100];
fp = fopen("data.txt", "r");
if (fp != NULL) {
    while (fgets(buffer, 100, fp)) {
        printf("%s", buffer);
    }
    fclose(fp);
}
```

## 11. 预处理指令 (Preprocessor Directives)

在编译之前处理。以 `#` 开头。

*   `#include <file>`: 插入头文件的内容。
*   `#define NAME value`: 定义宏。
*   `#ifdef`, `#ifndef`, `#endif`: 条件编译。

```c
#define PI 3.14159
#define MAX(a,b) ((a) > (b) ? (a) : (b))

double area = PI * r * r;
```

## 参考资料 (References & Resources)

*   **标准:** [ISO/IEC 9899 (C Standard)](https://www.iso.org/standard/74528.html)
*   **参考:** [cppreference.com - C](https://zh.cppreference.com/w/c)
*   **书籍:** 《C程序设计语言》(The C Programming Language) - Brian Kernighan 和 Dennis Ritchie (K&R)
*   **工具:** GCC, Clang, GDB, Valgrind
