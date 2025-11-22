# C Programming Language Guide

A comprehensive guide to the C programming language, covering everything from basic syntax to advanced memory management and system-level programming. C is a powerful, efficient language that forms the foundation of modern computing.

## Quick Reference

| Concept | Syntax / Command | Example |
| :--- | :--- | :--- |
| **Print** | `printf(format, ...)` | `printf("Hello, %s\n", "World");` |
| **Input** | `scanf(format, ...)` | `scanf("%d", &x);` |
| **Comments** | `//` or `/* ... */` | `// Single line` |
| **Compile** | `gcc file.c -o output` | `gcc main.c -o main` |
| **Data Types** | `int`, `char`, `float`, `double`, `void` | `int age = 25;` |
| **Pointer** | `*` (value at), `&` (address of) | `int *p = &x;` |
| **Alloc** | `malloc(size)` | `int *arr = malloc(5 * sizeof(int));` |
| **Free** | `free(ptr)` | `free(arr);` |

---

## 1. Introduction

C is a general-purpose procedural computer programming language supporting structured programming, lexical variable scope, and recursion, with a static type system. By design, C provides constructs that map efficiently to typical machine instructions.

### Key Features
*   **Efficiency:** Produces fast, compact code.
*   **Portability:** Source code can be compiled on many different platforms.
*   **Low-level Access:** Direct manipulation of memory via pointers.
*   **Simplicity:** Small core language with a rich set of libraries.

## 2. Basic Structure

A standard C program consists of functions and variables. The entry point is always the `main` function.

```c
#include <stdio.h> // Preprocessor directive

// Main function - entry point
int main() {
    // Print to standard output
    printf("Hello, World!\n");
    
    // Return 0 indicates successful execution
    return 0;
}
```

## 3. Variables & Data Types

C requires variables to be declared before use.

### Primitive Types

| Type | Description | Format Specifier |
| :--- | :--- | :--- |
| `int` | Integer (usually 4 bytes) | `%d` |
| `char` | Character (1 byte) | `%c` |
| `float` | Floating point (4 bytes) | `%f` |
| `double` | Double precision float (8 bytes) | `%lf` |
| `void` | Empty / No type | N/A |

```c
int count = 10;
float price = 19.99;
char grade = 'A';
double precise_val = 3.14159265359;
```

## 4. Control Flow

### Conditionals

```c
if (age >= 18) {
    printf("Adult\n");
} else if (age >= 13) {
    printf("Teenager\n");
} else {
    printf("Child\n");
}
```

### Switch

```c
switch (grade) {
    case 'A': printf("Excellent\n"); break;
    case 'B': printf("Good\n"); break;
    default:  printf("Other\n");
}
```

### Loops

```c
// For loop
for (int i = 0; i < 5; i++) {
    printf("%d ", i);
}

// While loop
while (condition) {
    // code
}

// Do-while (executes at least once)
do {
    // code
} while (condition);
```

## 5. Functions

Functions break large tasks into smaller, manageable units.

```c
// Function declaration (prototype)
int add(int a, int b);

int main() {
    int result = add(5, 3);
    return 0;
}

// Function definition
int add(int a, int b) {
    return a + b;
}
```

## 6. Pointers

Pointers are variables that store memory addresses. They are one of C's most powerful (and dangerous) features.

### Basics
*   `&` (Address-of operator): Returns the memory address of a variable.
*   `*` (Dereference operator): Accesses the value at the address stored in a pointer.

```c
int var = 20;
int *ptr = &var; // ptr holds the address of var

printf("Address: %p\n", ptr);
printf("Value: %d\n", *ptr); // Dereferencing

*ptr = 30; // Changes value of var
```

### Pointers and Arrays
In C, arrays and pointers are closely related. The name of an array acts as a pointer to its first element.

```c
int arr[3] = {10, 20, 30};
int *p = arr;

printf("%d", *p);     // 10
printf("%d", *(p+1)); // 20 (Pointer arithmetic)
```

## 7. Arrays & Strings

### Arrays
Fixed-size collection of elements of the same type.

```c
int numbers[5] = {1, 2, 3, 4, 5};
numbers[2] = 10;
```

### Strings
In C, strings are arrays of characters terminated by a null character `\0`.

```c
char name[] = "Alice"; // Automatically adds '\0' at the end
// name is ['A', 'l', 'i', 'c', 'e', '\0']

// Using string library
#include <string.h>
strlen(name);   // Length (excluding \0)
strcpy(dest, src); // Copy string
strcmp(s1, s2);    // Compare strings
```

## 8. Memory Management

C allows dynamic memory allocation on the heap using `<stdlib.h>`.

### Key Functions
*   `malloc(size)`: Allocates `size` bytes. Returns `void*`.
*   `calloc(n, size)`: Allocates space for `n` elements, initialized to zero.
*   `realloc(ptr, size)`: Resizes previously allocated memory.
*   `free(ptr)`: Releases allocated memory.

### Example

```c
int *arr;
int n = 5;

// Allocate memory for 5 integers
arr = (int*) malloc(n * sizeof(int));

if (arr == NULL) {
    printf("Memory allocation failed\n");
    return 1;
}

// Use memory...
for(int i=0; i<n; i++) arr[i] = i;

// Free memory
free(arr);
arr = NULL; // Good practice
```

> **Warning:** Always `free` dynamically allocated memory to prevent memory leaks.

## 9. Structures (struct)

Structures group related variables of different types.

```c
struct Person {
    char name[50];
    int age;
    float salary;
};

// Usage
struct Person p1;
strcpy(p1.name, "John");
p1.age = 30;

// Initializer syntax
struct Person p2 = {"Jane", 28, 55000.0};
```

### Typedef
Used to create an alias for a type, often used with structs.

```c
typedef struct {
    int x;
    int y;
} Point;

Point p1 = {10, 20}; // No need to write 'struct'
```

## 10. File I/O

Handling files using `<stdio.h>`.

```c
FILE *fp;

// Writing to a file
fp = fopen("data.txt", "w");
if (fp != NULL) {
    fprintf(fp, "Testing...\n");
    fclose(fp);
}

// Reading from a file
char buffer[100];
fp = fopen("data.txt", "r");
if (fp != NULL) {
    while (fgets(buffer, 100, fp)) {
        printf("%s", buffer);
    }
    fclose(fp);
}
```

## 11. Preprocessor Directives

Processed before compilation. Starts with `#`.

*   `#include <file>`: Inserts content of a header file.
*   `#define NAME value`: Defines a macro.
*   `#ifdef`, `#ifndef`, `#endif`: Conditional compilation.

```c
#define PI 3.14159
#define MAX(a,b) ((a) > (b) ? (a) : (b))

double area = PI * r * r;
```

## References & Resources

*   **Standard:** [ISO/IEC 9899 (C Standard)](https://www.iso.org/standard/74528.html)
*   **Reference:** [cppreference.com - C](https://en.cppreference.com/w/c)
*   **Book:** "The C Programming Language" by Brian Kernighan and Dennis Ritchie (K&R)
*   **Tools:** GCC, Clang, GDB, Valgrind
