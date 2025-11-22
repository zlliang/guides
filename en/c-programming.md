# C Programming Language Guide

The C programming language is one of the most influential and widely used programming languages in the history of computing. Created in the early 1970s by Dennis Ritchie at Bell Labs, it was originally designed for system programming and implementing the UNIX operating system. Despite its age, C remains the backbone of modern infrastructure, powering operating systems, embedded systems, high-performance servers, and language runtimes.

This guide serves as a comprehensive reference and tutorial, covering the language from its fundamental syntax to its most advanced system-level capabilities. It highlights modern standards (C99, C11, C23) and emphasizes safe, efficient coding practices.

## Quick Reference

| Category | Key Syntax / Command | Example |
| :--- | :--- | :--- |
| **Output** | `printf(fmt, ...)` | `printf("Value: %d\n", x);` |
| **Input** | `scanf(fmt, ...)` | `scanf("%d", &x);` |
| **Allocation** | `malloc(size)`, `free(ptr)` | `int *p = malloc(sizeof(int) * 10);` |
| **Types** | `int`, `char`, `float`, `void *` | `uint64_t id = 0xFF;` |
| **Modifiers** | `const`, `static`, `volatile` | `static const int MAX = 100;` |
| **Pointers** | `*` (value), `&` (address) | `int *ptr = &val;` |
| **Structures** | `struct Name { ... };` | `struct Point { int x, y; };` |
| **Preprocessor** | `#include`, `#define`, `#ifdef` | `#define MIN(a,b) ((a)<(b)?(a):(b))` |

---

## History and Standards

Understanding the version of C you are using is critical, as features and safety guarantees have evolved significantly.

### K&R C (Pre-standard)
The original version described in the first edition of "The C Programming Language" (1978). It is now obsolete.
*   *Legacy Feature:* Functions often lacked return types (defaulted to `int`) and parameter type lists in declarations.

### C89 / C90 (ANSI C)
The first standardized version (ANSI X3.159-1989 / ISO/IEC 9899:1990).
*   **Key Features:** Function prototypes, `void` pointers, `enum`, `const` and `volatile`.
*   *Status:* Still widely supported in legacy embedded systems, but generally considered outdated for modern development.

### C99 (ISO/IEC 9899:1999)
A major update that introduced many features used today.
*   **Key Features:** `//` comments, variable declarations anywhere (not just top of scope), `long long`, `<stdint.h>`, variable-length arrays (VLAs), `restrict` qualifier, compound literals, designated initializers, flexible array members.

### C11 (ISO/IEC 9899:2011)
Focused on thread safety and alignment.
*   **Key Features:** Multi-threading support (`<threads.h>`), atomics (`<stdatomic.h>`), `_Generic` (type-generic macros), `_Static_assert`, `_Alignas`/`_Alignof`, `gets()` removed.

### C17 / C18
A bug-fix release for C11 with no new language features.

### C23 (Expected)
The next major revision.
*   **Key Features:** `typeof`, `constexpr`, `nullptr`, `true`/`false` keywords (making `<stdbool.h>` unnecessary), standard attributes (`[[nodiscard]]`, `[[deprecated]]`), binary literals (`0b1010`), `#embed`.

---

## The Compilation Model

C is a compiled language. The transformation from source code to executable involves distinct stages.

### 1. Preprocessing
The preprocessor (`cpp`) handles directives starting with `#`.
*   **Text Substitution:** Macros (`#define`) are expanded.
*   **File Inclusion:** `#include` inserts file contents.
*   **Conditional Compilation:** `#if`, `#ifdef` select code blocks.
*   **Output:** A "translation unit" (pure C code with no directives).

### 2. Compilation
The compiler (`cc1` in GCC) translates C code into assembly language (`.s`).
*   **Syntax Checking:** Validates grammar.
*   **Static Analysis:** Checks types, scope rules, and emits warnings.
*   **Optimization:** Reorders instructions, inlines functions, removes dead code.

### 3. Assembly
The assembler (`as`) translates assembly code into machine code, resulting in an object file (`.o` or `.obj`).
*   Object files contain binary code but symbolic references to external functions (like `printf`) remain unresolved.

### 4. Linking
The linker (`ld`) combines object files and libraries.
*   **Resolution:** Matches function calls to their definitions.
*   **Relocation:** Assigns final memory addresses.
*   **Output:** The executable binary.

---

## Basic Syntax and Structure

### The Entry Point
Every C program starts execution at `main`.

```c
#include <stdio.h>

// Standard signature: returns int, takes arguments
int main(int argc, char *argv[]) {
    printf("Program Name: %s\n", argv[0]);
    return 0; // 0 = Success, Non-zero = Failure
}
```

*   `argc`: Argument Count.
*   `argv`: Argument Vector (array of strings).

### Statements and Blocks
*   **Statements** end with a semicolon `;`.
*   **Blocks** are enclosed in braces `{ ... }` and define a scope.

```c
{
    int x = 5; // x exists only inside this block
}
// x is not accessible here
```

### Comments
*   `// Single line comment` (C99+)
*   `/* Multi-line comment */` (C89+)

---

## Variables and Data Types

C maps types closely to hardware words. Sizes are implementation-defined but strictly ordered (`short` ≤ `int` ≤ `long`).

### Integer Types

| Type | Typical Size (32/64-bit) | Specifier (printf) | Range (Unsigned) |
| :--- | :--- | :--- | :--- |
| `char` | 1 byte | `%c`, `%hhd` | 0 to 255 |
| `short` | 2 bytes | `%hd` | 0 to 65,535 |
| `int` | 4 bytes | `%d` | 0 to 4,294,967,295 |
| `long` | 4 or 8 bytes | `%ld` | Depends on OS (LP64 vs LLP64) |
| `long long` | 8 bytes | `%lld` | 0 to ~1.8 × 10^19 |

**Best Practice:** Use `<stdint.h>` for precise width requirements.
*   `int32_t`, `uint64_t`, `int8_t`, `uint16_t`.
*   `intptr_t`: Integer large enough to hold a pointer.
*   `size_t`: Unsigned integer for object sizes (result of `sizeof`).

### Floating Point Types

| Type | Description | Typical Size | Specifier |
| :--- | :--- | :--- | :--- |
| `float` | Single Precision | 4 bytes | `%f` |
| `double` | Double Precision | 8 bytes | `%lf` |
| `long double` | Extended Precision | 10/12/16 bytes | `%Lf` |

### The Void Type
`void` represents the absence of a value.
*   **Function return:** `void func()` (returns nothing).
*   **Function arguments:** `int func(void)` (takes nothing).
*   **Pointers:** `void *ptr` (generic pointer, must be cast before dereference).

### Constants and Literals
*   **Decimal:** `123`
*   **Hex:** `0x7B`
*   **Octal:** `0173` (Prefix with 0 - **Careful!** `010` is 8, not 10)
*   **Binary:** `0b1111011` (C23)
*   **Suffixes:** `10u` (unsigned), `10L` (long), `10.0f` (float).

---

## Operators and Expressions

### Arithmetic
`+`, `-`, `*`, `/`, `%` (modulo).
*   Integer division truncates (`5 / 2` is `2`).
*   Modulo `%` works only on integers.

### Bitwise Operators
Crucial for systems programming.
*   `&` (AND): Clears bits.
*   `|` (OR): Sets bits.
*   `^` (XOR): Toggles bits.
*   `~` (NOT): Inverts bits (One's complement).
*   `<<` (Left Shift): Multiplies by 2^n.
*   `>>` (Right Shift): Divides by 2^n.
    *   **Note:** Right shift on signed negative numbers is implementation-defined (usually arithmetic shift, preserving sign).

### Logical Operators
*   `&&` (AND), `||` (OR), `!` (NOT).
*   **Short-circuiting:** In `A && B`, if `A` is false, `B` is not evaluated. This is a guaranteed behavior.

### Increment/Decrement
*   `i++` (Post-increment): Returns value, then increments.
*   `++i` (Pre-increment): Increments, then returns value.
*   **Undefined Behavior:** `i = i++` or `func(i++, i++)`. Do not modify a variable twice in one sequence point.

### The Ternary Operator
`condition ? value_if_true : value_if_false`

```c
int max = (a > b) ? a : b;
```

---

## Control Flow

### If-Else

```c
if (temperature > 30) {
    printf("Hot\n");
} else if (temperature < 10) {
    printf("Cold\n");
} else {
    printf("Moderate\n");
}
```

### Switch-Case
Efficient for checking a variable against integer constants.

```c
switch (status) {
    case 200:
        printf("OK\n");
        break; // Don't forget break!
    case 404:
        printf("Not Found\n");
        break;
    default:
        printf("Unknown\n");
}
```
**Fallthrough:** If `break` is omitted, execution continues to the next case.

### Loops

**For Loop:**
```c
for (int i = 0; i < 10; i++) {
    if (i == 5) continue; // Skip this iteration
    if (i == 8) break;    // Exit loop
    printf("%d ", i);
}
```

**While Loop:**
```c
while (ptr != NULL) {
    ptr = ptr->next;
}
```

**Do-While Loop:**
Guarantees execution at least once.
```c
do {
    input = read_sensor();
} while (input < THRESHOLD);
```

### Goto
Transfers control to a labeled statement.
*   **Usage:** Primarily for error cleanup in complex functions.

```c
    void *p = malloc(100);
    if (!p) goto error;
    
    if (do_work() != 0) goto cleanup;

    free(p);
    return 0;

cleanup:
    // Rollback work
error:
    free(p);
    return -1;
```

---

## Pointers and Memory

Pointers are variables that store memory addresses. They provide power and danger in equal measure.

### Declaration and Dereferencing

```c
int val = 42;
int *p = &val;  // & = Address-of operator
int x = *p;     // * = Dereference operator (get value at address)
```

### Pointer Arithmetic
Pointers know the size of the type they point to.
*   If `p` is `int *` (4 bytes), `p + 1` adds 4 to the address.
*   If `p` is `char *` (1 byte), `p + 1` adds 1 to the address.

### Void Pointers
`void *` is a generic pointer. It holds an address but has no type information.
*   Cannot be dereferenced.
*   Cannot perform arithmetic (without casting).
*   Used in functions like `malloc` and `memcpy`.

### Null Pointers
A pointer with value `0` or `NULL` ((void*)0).
*   Dereferencing it causes a segmentation fault (crash).
*   Always check pointers: `if (ptr) { ... }`.

### Function Pointers
Pointers can point to executable code.

```c
// Declaration: A pointer 'fn' to a function taking (int, int) returning int
int (*fn)(int, int);

int add(int a, int b) { return a + b; }

fn = add;
int result = fn(5, 10); // Calls add(5, 10)
```

Use cases: Callbacks, interrupt handlers, sorting (qsort), dynamic dispatch.

---

## Arrays and Strings

### Arrays
Contiguous blocks of memory.

```c
int arr[5] = {1, 2, 3, 4, 5};
// arr decays to &arr[0] in most contexts
```

**Multidimensional Arrays:**
`int grid[3][3]` is an array of 3 arrays of 3 integers.
Memory layout is **row-major** (linear).

### Strings
C strings are character arrays terminated by a **Null Character** (`\0`).

```c
char str[] = "Hello"; // Size is 6 ('H','e','l','l','o','\0')
char *ptr = "World";  // String literal (Read-Only memory!)
```

**Important String Functions (`<string.h>`):**
*   `strlen(s)`: Length excluding `\0`.
*   `strcpy(dst, src)`: Copy string. **Unsafe** (buffer overflow risk).
*   `strncpy(dst, src, n)`: Safer, but tricky (may not null-terminate).
*   `strcat(dst, src)`: Concatenate.
*   `strcmp(s1, s2)`: Compare (returns 0 if equal).
*   `strdup(s)`: Duplicate string (allocates memory, user must free).

**Safety Tip:** Prefer functions that take length arguments (like `snprintf`) over unbounded ones (like `sprintf` or `strcpy`).

---

## Memory Management

C programs use three types of memory:

### 1. Static / Global
*   **Where:** Data segment or BSS.
*   **Lifetime:** Entire program execution.
*   **Scope:** Global variables, `static` local variables.

### 2. Stack (Automatic)
*   **Where:** Stack frames.
*   **Lifetime:** Function scope. destroyed when function returns.
*   **Characteristics:** Fast allocation, limited size (stack overflow risk).

### 3. Heap (Dynamic)
*   **Where:** Managed heap.
*   **Lifetime:** Manual (until freed).
*   **Usage:** Large objects, objects living beyond function scope.

### Dynamic Allocation Functions (`<stdlib.h>`)

*   `malloc(size_t size)`: Allocates `size` bytes. Content is garbage.
*   `calloc(size_t n, size_t size)`: Allocates `n * size` bytes. **Zero-initializes**.
*   `realloc(void *ptr, size_t new_size)`: Resizes a block. Can move data to new location.
*   `free(void *ptr)`: Releases memory.

**Example:**

```c
struct Data *items = malloc(count * sizeof(struct Data));
if (!items) {
    perror("Malloc failed");
    exit(1);
}

// ... use items ...

free(items);
items = NULL; // Prevent dangling pointer usage
```

### Common Memory Errors
1.  **Leak:** Failing to `free`.
2.  **Double Free:** Calling `free` twice on the same pointer.
3.  **Use After Free:** Accessing `ptr` after `free(ptr)`.
4.  **Buffer Overflow:** Writing past the allocated size.

Tools like **Valgrind** or **AddressSanitizer** (`-fsanitize=address`) are essential for detecting these.

---

## Structures and Unions

### Structures (`struct`)
Groups related data types.

```c
struct Pixel {
    uint8_t r, g, b;
};

// Typedef for cleaner usage
typedef struct {
    float x, y;
} Point;

Point p = {1.0f, 2.0f}; // Designated init (C99): {.y=2.0f, .x=1.0f}
```

**Padding and Alignment:**
The compiler inserts "padding" bytes to align members to architecture-friendly addresses.
```c
struct Bad {
    char c;      // 1 byte
    // 3 bytes padding implied here
    int i;       // 4 bytes
}; // sizeof = 8
```
To minimize size, order members from largest alignment to smallest.

### Unions
Allows storing different data types in the same memory location.

```c
union Variant {
    int i;
    float f;
    char c;
};
```
Only one member is valid at a time. Size is `max(sizeof(members))`.

### Bit-fields
Pack variables into specific numbers of bits.

```c
struct Flags {
    unsigned int visible : 1;
    unsigned int editable : 1;
    unsigned int layer : 6; // 0-63
};
```
**Warning:** Implementation-dependent ordering (LSB vs MSB first).

---

## The Preprocessor Deep Dive

The preprocessor runs before compilation.

### Macros
Macros perform text substitution.

```c
#define MAX_SIZE 1024
#define SQUARE(x) ((x) * (x))
```
**Pitfall:** `SQUARE(a++)` expands to `((a++) * (a++))`, incrementing `a` twice (Undefined Behavior).

### Header Guards
Prevents a header file from being included multiple times.

```c
#ifndef MY_HEADER_H
#define MY_HEADER_H

// Declarations...

#endif
```
Modern compilers support `#pragma once` for the same purpose.

### Conditional Compilation
Useful for cross-platform code.

```c
#ifdef _WIN32
    #include <windows.h>
#else
    #include <unistd.h>
#endif
```

### X-Macros
A pattern to generate code for lists.

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

---

## The Standard Library

C comes with a small but essential standard library.

### Input / Output (`stdio.h`)
*   **File handles:** `FILE *` (opaque structure).
*   **Standard streams:** `stdin`, `stdout`, `stderr`.
*   **Operations:** `fopen`, `fclose`, `fread`, `fwrite`, `fprintf`, `fscanf`.

**Buffering:**
*   `_IOFBF` (Full buffering): Writes saved until buffer full (Files).
*   `_IOLBF` (Line buffering): Flush on newline (stdout).
*   `_IONBF` (No buffering): Flush immediately (stderr).

### String Handling (`string.h`)
Manipulation of C strings and raw memory (`memcpy`, `memset`, `memmove`).
*   **Note:** `memcpy` assumes non-overlapping regions. Use `memmove` if regions overlap.

### Character Handling (`ctype.h`)
*   `isalnum`, `isalpha`, `isdigit`, `isspace`, `toupper`, `tolower`.

### General Utilities (`stdlib.h`)
*   **Conversion:** `atoi`, `strtol`, `strtod`.
*   **Random:** `rand`, `srand` (Use modern alternatives if available).
*   **Process:** `exit`, `abort`, `system`.
*   **Sorting/Search:** `qsort`, `bsearch`.

### Assertions (`assert.h`)
Runtime checks.
```c
assert(ptr != NULL);
```
Disable with `#define NDEBUG` before including `<assert.h>`.

---

## Advanced Topics

### Variadic Functions (`stdarg.h`)
Functions that accept a variable number of arguments (like `printf`).

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

### Type Qualifiers

*   **`const`**: Read-only.
    *   `const int *p`: Data is const.
    *   `int * const p`: Pointer is const.
*   **`volatile`**: Prevents compiler optimization. Used for memory-mapped hardware I/O or flags modified by interrupt handlers.
*   **`restrict`** (C99): Promise that pointers don't alias (overlap). Enables vectorization optimizations.

### Storage Class Specifiers

*   **`auto`**: Default for stack variables.
*   **`register`**: Hint to store in CPU register (mostly ignored by modern compilers).
*   **`static`**:
    *   Global scope: File-private visibility.
    *   Local scope: Persists value across function calls.
*   **`extern`**: Declares an object defined elsewhere.

### Setjmp / Longjmp (`setjmp.h`)
Non-local jumps (goto across functions). Saves stack context.
Used for exception-style error handling in C, but tricky (doesn't clean up stack variables automatically).

---

## Modern C Features (C99 / C11 / C23)

### C99
*   **Compound Literals:** Pass structs without variables.
    ```c
    draw_point((struct Point){.x=10, .y=20});
    ```
*   **Variable Length Arrays (VLA):**
    ```c
    void func(int n) { int arr[n]; ... }
    ```
    *Warning:* Can cause stack overflow. Often discouraged in secure code.
*   **Designated Initializers:**
    ```c
    int arr[10] = {[0]=5, [4]=10};
    ```

### C11
*   **_Generic:** Compile-time polymorphism.
    ```c
    #define print(x) _Generic((x), int: print_int, float: print_float)(x)
    ```
*   **Threads (`threads.h`):** Portable threading (thrd_create, mtx_lock).
*   **Atomics (`stdatomic.h`):** `_Atomic int counter;` for lock-free sync.

### C23 (Upcoming)
*   **Type Inference:** `auto x = 10;` (like C++).
*   **Typeof:** `typeof(x) y = x;`
*   **Attributes:** `[[nodiscard]] int important_func();`
*   **Bit-precise integers:** `_BitInt(N)`.

---

## Undefined Behavior (UB)

One of C's most distinctive aspects is "Undefined Behavior". The standard allows the compiler to assume certain errors *never happen*. If they do, the program can do anything (crash, work silently, corrupt data).

**Common UB Sources:**
1.  **Signed Integer Overflow:** `INT_MAX + 1` is UB. (Unsigned overflow is defined as modulo wrapping).
2.  **Dereferencing Null/Uninitialized Pointers.**
3.  **Division by Zero.**
4.  **Shift by Negative Count.**
5.  **Modifying const data** (via casting).
6.  **Violating Strict Aliasing:** Accessing a float as an int pointer.

**Why UB exists:** It allows aggressive optimization. The compiler can assume loop counters don't wrap, pointers point to valid memory, etc.

---

## Best Practices

### 1. Safety First
*   Initialize variables: `int x = 0;`.
*   Check pointer validity before use.
*   Use safe string functions (`snprintf` instead of `sprintf`).
*   Validate all external inputs.

### 2. Code Organization
*   **Header Files (.h):** Declarations, macros, structs.
*   **Source Files (.c):** Implementation.
*   **Static Scope:** Mark helper functions as `static` to avoid polluting the global namespace.

### 3. Const Correctness
Use `const` aggressively. It documents intent and allows optimization.
```c
// Bad
void print_data(char *str);
// Good
void print_data(const char *str);
```

### 4. Error Handling
C lacks exceptions. Common patterns:
*   **Return Codes:** 0 for success, negative for error.
*   **Errno:** Global error variable (check `errno` in `<errno.h>`).
*   **Output Parameters:** Return status, pass result via pointer.

### 5. Opaque Pointers
Hide implementation details (encapsulation).
*   **Header:** `typedef struct Handle_T *Handle;`
*   **Source:** `struct Handle_T { int internal_data; };`
The user only sees the pointer `Handle`, preserving ABI stability.

---

## Tooling and Debugging

### Compilation Flags
Always compile with warnings enabled.
*   **GCC/Clang:** `-Wall -Wextra -pedantic -Wconversion -Wshadow`
*   **Debug symbols:** `-g` (required for GDB).
*   **Optimization:** `-O0` (debug), `-O2` (release), `-O3` (aggressive).

### Debuggers (GDB / LLDB)
*   `break main`: Set breakpoint.
*   `run`: Start program.
*   `next` / `step`: Step over / into.
*   `print x`: Inspect variable.
*   `bt`: Backtrace (call stack).

### Sanitizers
Modern compilers include runtime checkers.
*   `-fsanitize=address` (ASan): Detects buffer overflows, use-after-free.
*   `-fsanitize=undefined` (UBSan): Detects UB like signed overflow.
*   `-fsanitize=leak` (LSan): Detects memory leaks.

### Valgrind
A powerful instrumentation framework.
```bash
valgrind --leak-check=full ./my_program
```
It simulates the CPU to track every bit of memory, finding leaks and invalid accesses that sanitizers might miss (though it is slower).

## References & Resources

*   **Standards:**
    *   [ISO/IEC 9899:2011 (C11 draft)](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf)
    *   [ISO/IEC 9899:2018 (C17 draft)](https://web.archive.org/web/20181230041359/http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2176.pdf)
*   **References:**
    *   [cppreference.com (C)](https://en.cppreference.com/w/c) - Excellent online documentation.
    *   [SEI CERT C Coding Standard](https://wiki.sei.cmu.edu/confluence/display/c) - Security guidelines.
*   **Books:**
    *   *The C Programming Language (2nd Ed)* - Kernighan & Ritchie (The classic).
    *   *Modern C* - Jens Gustedt (Up-to-date look at C11/C17).
    *   *C Interfaces and Implementations* - David R. Hanson (Design patterns).
    *   *Expert C Programming: Deep C Secrets* - Peter van der Linden.
