# Zig Programming Language

A comprehensive guide to Zig — a general-purpose programming language and toolchain for maintaining robust, optimal, and reusable software. This guide covers Zig version 0.15.2, including its philosophy, features, syntax, and ecosystem.

## Quick Reference

```zig
// Variables
const x = 5;                    // Immutable
var y: i32 = 10;               // Mutable with type
var z = @as(f64, 3.14);        // Type coercion

// Functions
fn add(a: i32, b: i32) i32 {
    return a + b;
}

// Error handling
fn mayFail() !void {
    try somethingFallible();
    const result = operation() catch 0;
}

// Optionals
const maybe: ?i32 = null;
const value = maybe orelse 0;

// Comptime
fn List(comptime T: type) type {
    return struct {
        items: []T,
    };
}

// Memory allocation
const allocator = std.heap.page_allocator;
const list = try std.ArrayList(i32).initCapacity(allocator, 10);
defer list.deinit();

// C interop
const c = @cImport(@cInclude("stdio.h"));
_ = c.printf("Hello\n");
```

## Introduction

### What is Zig?

Zig is a general-purpose programming language created by **Andrew Kelley** in 2015. It's designed to be a better alternative to C for systems programming, with a focus on:

- **Robustness**: Correct behavior even in edge cases like out of memory
- **Optimal Performance**: Write the best possible code with zero-cost abstractions
- **Reusability**: Same code works across different environments and constraints
- **Maintainability**: Clear intent, low overhead to reading code

Unlike Rust (which emphasizes safety through ownership) or Go (which emphasizes simplicity through a garbage collector), Zig takes a different approach: **manual memory management with maximum clarity and no hidden control flow**.

### Why Use Zig?

**Instead of C**:
- Compile-time code execution instead of macros
- Explicit error handling instead of error codes
- Optional types instead of null pointers
- No undefined behavior in safe builds
- First-class cross-compilation support

**Instead of C++**:
- No hidden control flow
- No operator overloading
- Simpler language with fewer features
- Faster compile times
- No exceptions

**Instead of Rust**:
- Simpler language (no lifetimes, no borrow checker)
- Manual memory management gives you full control
- More transparent about what code does
- Easier to integrate with C

**Unique Strengths**:
- Compile-time execution for metaprogramming
- Drop-in C/C++ compiler with cross-compilation
- No hidden memory allocations
- Build system written in Zig

### Current Status

- **Version**: 0.15.2 (as of October 2025)
- **Development Stage**: Pre-1.0, actively evolving
- **Stability**: Breaking changes expected between versions
- **Production Use**: Suitable for projects that can update with language changes
- **License**: MIT (Expat)

## History and Philosophy

### Origins

Zig was created by **Andrew Kelley** who serves as the project's BDFN (Benevolent Dictator For Now). Development began in August 2015 with these core goals:

1. Provide a better alternative to C with modern features
2. Enable compile-time code execution without a macro system
3. Offer first-class cross-compilation support
4. Eliminate hidden control flow and memory allocations
5. Support manual memory management without a default allocator

The language is supported by the **Zig Software Foundation (ZSF)**, a 501(c)(3) non-profit organization that funds core development through monthly donations.

### Design Principles

**No Hidden Control Flow**:
Every line of Zig code is transparent. No operator overloading, no exceptions, no properties. Function calls look like function calls.

**No Hidden Memory Allocations**:
All memory allocations are explicit. Libraries accept an `Allocator` parameter instead of assuming a global allocator.

**Manual Memory Management**:
Like C, but with better tools. No garbage collector, but also no implicit malloc/free.

**Compile-Time Execution**:
Run Zig code at compile time for metaprogramming without a separate template or macro language.

**First-Class Cross-Compilation**:
Cross-compile to any supported target without separate toolchains. Zig ships with libc headers for multiple platforms.

## Installation and Setup

### Download

Visit [ziglang.org/download](https://ziglang.org/download/) for official releases.

### Package Managers

```bash
# Homebrew (macOS/Linux)
brew install zig

# Python (pip)
pip install ziglang

# Arch Linux
pacman -S zig

# From source
git clone https://github.com/ziglang/zig
cd zig
zig build
```

### Verify Installation

```bash
zig version  # Should show 0.15.2
```

### Hello World

**hello.zig**:
```zig
const std = @import("std");

pub fn main() void {
    std.debug.print("Hello, World!\n", .{});
}
```

**Compile and run**:
```bash
zig build-exe hello.zig
./hello

# Or run directly
zig run hello.zig
```

## Language Basics

### Variables

```zig
// Constants (immutable)
const x = 5;
const y: i32 = 10;  // Explicit type

// Variables (mutable)
var z = 20;
var w: f64 = 3.14;

// Undefined (must be initialized before use)
var a: i32 = undefined;
a = 42;
```

### Types

**Integers**:
```zig
i8, i16, i32, i64, i128    // Signed
u8, u16, u32, u64, u128    // Unsigned
isize, usize               // Pointer-sized
```

**Floats**:
```zig
f16, f32, f64, f80, f128
```

**Other Types**:
```zig
bool                 // true or false
void                 // No value
type                 // Type of types
anyerror            // Any error
comptime_int        // Compile-time integer
comptime_float      // Compile-time float
```

**Type Coercion**:
```zig
const a: i32 = 5;
const b: f64 = @floatFromInt(a);  // Explicit
const c = @as(u8, 255);           // Type assertion
```

### Functions

```zig
// Basic function
fn add(a: i32, b: i32) i32 {
    return a + b;
}

// Void return (no return value)
fn greet() void {
    std.debug.print("Hello!\n", .{});
}

// Public function (exported from module)
pub fn publicFunc() void {
    // ...
}

// Inline function
inline fn fastFunc() void {
    // ...
}

// Export to C
export fn cFunc(x: i32) i32 {
    return x * 2;
}

// External C function
extern "c" fn malloc(size: usize) ?*anyopaque;
```

### Control Flow

**If Statements**:
```zig
if (condition) {
    // ...
} else if (other) {
    // ...
} else {
    // ...
}

// If as expression
const result = if (x > 0) 1 else -1;
```

**While Loops**:
```zig
var i: u32 = 0;
while (i < 10) : (i += 1) {
    std.debug.print("{}\n", .{i});
}

// With continue expression
while (i < 10) {
    i += 1;
}
```

**For Loops**:
```zig
const items = [_]i32{1, 2, 3, 4, 5};

// Iterate over array
for (items) |item| {
    std.debug.print("{}\n", .{item});
}

// With index
for (items, 0..) |item, i| {
    std.debug.print("{}: {}\n", .{i, item});
}

// Iterate over range
for (0..10) |i| {
    std.debug.print("{}\n", .{i});
}
```

**Switch**:
```zig
const x = 2;
const result = switch (x) {
    0 => "zero",
    1, 2 => "one or two",
    3...10 => "three to ten",
    else => "other",
};
```

### Arrays and Slices

**Arrays**:
```zig
// Fixed-size arrays
const arr = [_]i32{1, 2, 3, 4, 5};  // Type inferred
const arr2: [5]i32 = undefined;      // Uninitialized
const arr3 = [_]i32{1} ** 10;        // Repeat: [1,1,1,...]

// Multidimensional
const matrix = [_][3]i32{
    [_]i32{1, 2, 3},
    [_]i32{4, 5, 6},
};
```

**Slices** (view into arrays):
```zig
const arr = [_]i32{1, 2, 3, 4, 5};
const slice: []const i32 = &arr;        // Full slice
const sub_slice = arr[1..4];            // [2, 3, 4]
```

### Structs

```zig
const Point = struct {
    x: f64,
    y: f64,

    // Method
    pub fn distance(self: Point, other: Point) f64 {
        const dx = self.x - other.x;
        const dy = self.y - other.y;
        return @sqrt(dx * dx + dy * dy);
    }

    // Associated function (no self)
    pub fn origin() Point {
        return Point{ .x = 0, .y = 0 };
    }
};

const p1 = Point{ .x = 1.0, .y = 2.0 };
const p2 = Point.origin();
const dist = p1.distance(p2);
```

### Enums

```zig
const Color = enum {
    red,
    green,
    blue,

    pub fn toRGB(self: Color) [3]u8 {
        return switch (self) {
            .red => [_]u8{255, 0, 0},
            .green => [_]u8{0, 255, 0},
            .blue => [_]u8{0, 0, 255},
        };
    }
};

// Tagged union (enum with payload)
const Value = union(enum) {
    int: i32,
    float: f64,
    string: []const u8,
};

const v = Value{ .int = 42 };
```

### Pointers

```zig
// Single-item pointer
var x: i32 = 5;
const ptr: *i32 = &x;
ptr.* = 10;  // Dereference

// Many-item pointer (like C pointer)
const arr = [_]i32{1, 2, 3};
const ptr_many: [*]const i32 = &arr;

// Slice (fat pointer: pointer + length)
const slice: []const i32 = &arr;
```

## Error Handling

### Error Sets

```zig
const FileError = error{
    FileNotFound,
    PermissionDenied,
    OutOfMemory,
};

const AllErrors = error{
    NetworkError,
} || FileError;  // Merge error sets
```

### Error Unions

```zig
fn divide(a: i32, b: i32) !i32 {
    if (b == 0) return error.DivisionByZero;
    return @divTrunc(a, b);
}

// Using error unions
const result = divide(10, 2) catch |err| {
    std.debug.print("Error: {}\n", .{err});
    return;
};
```

### try Keyword

```zig
// Propagate errors up
fn parseAndDouble(str: []const u8) !i32 {
    const num = try std.fmt.parseInt(i32, str, 10);
    return num * 2;
}

// Equivalent to:
fn parseAndDouble(str: []const u8) !i32 {
    const num = std.fmt.parseInt(i32, str, 10) catch |err| return err;
    return num * 2;
}
```

### catch and errdefer

```zig
// Provide default value
const num = parseNumber(str) catch 0;

// Handle specific errors
const num = parseNumber(str) catch |err| switch (err) {
    error.Overflow => return error.TooLarge,
    error.InvalidCharacter => return error.BadInput,
};

// Clean up on error
fn createResource() !*Resource {
    const resource = try allocate();
    errdefer deallocate(resource);  // Only runs if function returns error
    
    try initialize(resource);
    return resource;
}
```

## Optional Types

### Basics

```zig
// Optional integer (can be null)
const maybe: ?i32 = null;
const definitely: ?i32 = 42;

// Unwrap with orelse
const value = maybe orelse 0;

// Check with if
if (maybe) |val| {
    // val is i32
    std.debug.print("Value: {}\n", .{val});
} else {
    std.debug.print("No value\n", .{});
}
```

### Optional Pointers

```zig
// Optional pointers compile to regular pointers
// null is represented as address 0
const ptr: ?*i32 = null;

// This is memory-efficient
const std = @import("std");
std.debug.assert(@sizeOf(?*i32) == @sizeOf(*i32));
```

## Memory Management

### Allocators

All memory allocation in Zig requires an explicit allocator:

```zig
const std = @import("std");

// General-purpose allocator with leak detection
var gpa = std.heap.GeneralPurposeAllocator(.{}){};
defer _ = gpa.deinit();
const allocator = gpa.allocator();

// Allocate memory
const buffer = try allocator.alloc(u8, 1024);
defer allocator.free(buffer);

// Allocate single item
const item = try allocator.create(MyStruct);
defer allocator.destroy(item);
```

### Standard Allocators

```zig
// General purpose (leak detection in debug)
std.heap.GeneralPurposeAllocator(.{})

// C's malloc/free
std.heap.c_allocator

// OS page allocator
std.heap.page_allocator

// Arena (bulk deallocation)
var arena = std.heap.ArenaAllocator.init(std.heap.page_allocator);
defer arena.deinit();  // Frees everything at once
const allocator = arena.allocator();

// Fixed buffer (stack allocation)
var buffer: [1024]u8 = undefined;
var fba = std.heap.FixedBufferAllocator.init(&buffer);
const allocator = fba.allocator();

// Testing allocator (detects leaks)
const allocator = std.testing.allocator;
```

### ArrayList Example

```zig
const std = @import("std");

pub fn main() !void {
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer _ = gpa.deinit();
    const allocator = gpa.allocator();

    var list = std.ArrayList(i32).init(allocator);
    defer list.deinit();

    try list.append(1);
    try list.append(2);
    try list.append(3);

    for (list.items) |item| {
        std.debug.print("{}\n", .{item});
    }
}
```

## Compile-Time Execution (comptime)

### Basic Comptime

```zig
// Computed at compile time
const result = comptime fibonacci(10);

fn fibonacci(n: u32) u32 {
    if (n == 0 or n == 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}
```

### Generic Data Structures

```zig
fn List(comptime T: type) type {
    return struct {
        items: []T,
        allocator: std.mem.Allocator,

        pub fn init(allocator: std.mem.Allocator) List(T) {
            return .{
                .items = &[_]T{},
                .allocator = allocator,
            };
        }

        pub fn append(self: *List(T), item: T) !void {
            // Implementation
        }
    };
}

// Use it
const IntList = List(i32);
const FloatList = List(f64);
```

### Comptime Parameters

```zig
fn max(comptime T: type, a: T, b: T) T {
    return if (a > b) a else b;
}

const result = max(i32, 10, 20);  // T is i32
const result2 = max(f64, 3.14, 2.71);  // T is f64
```

### Compile-Time Reflection

```zig
fn printFieldNames(comptime T: type) void {
    const info = @typeInfo(T);
    inline for (info.@"struct".fields) |field| {
        std.debug.print("{s}\n", .{field.name});
    }
}

const Point = struct {
    x: f64,
    y: f64,
};

comptime {
    printFieldNames(Point);  // Prints "x" and "y"
}
```

### Comptime Blocks

```zig
const primes = comptime blk: {
    var result: [25]i32 = undefined;
    var i: usize = 0;
    var n: i32 = 2;
    while (i < 25) : (n += 1) {
        if (isPrime(n)) {
            result[i] = n;
            i += 1;
        }
    }
    break :blk result;
};
```

## Testing

### Test Blocks

```zig
const std = @import("std");

fn add(a: i32, b: i32) i32 {
    return a + b;
}

test "add function" {
    try std.testing.expectEqual(@as(i32, 5), add(2, 3));
    try std.testing.expectEqual(@as(i32, -1), add(2, -3));
}

test "memory allocation" {
    const allocator = std.testing.allocator;
    
    const list = try std.ArrayList(i32).initCapacity(allocator, 10);
    defer list.deinit();
    
    try std.testing.expect(list.items.len == 0);
}
```

### Running Tests

```bash
zig test myfile.zig
zig test --test-filter "add" myfile.zig  # Run specific tests
```

## C Interoperability

### Importing C Headers

```zig
const c = @cImport({
    @cInclude("stdio.h");
    @cInclude("stdlib.h");
    @cDefine("_GNU_SOURCE", "1");
});

pub fn main() void {
    _ = c.printf("Hello from C\n");
}
```

### Calling C Functions

```zig
extern "c" fn malloc(size: usize) ?*anyopaque;
extern "c" fn free(ptr: *anyopaque) void;

pub fn main() void {
    const ptr = malloc(100).?;
    defer free(ptr);
}
```

### Exporting to C

```zig
export fn add(a: i32, b: i32) i32 {
    return a + b;
}

// From C:
// extern int add(int a, int b);
```

### C Types

```zig
c_char, c_short, c_int, c_long, c_longlong
c_ushort, c_uint, c_ulong, c_ulonglong
c_longdouble
```

### translate-c

```bash
zig translate-c header.h > header.zig
```

## Build System

### build.zig

```zig
const std = @import("std");

pub fn build(b: *std.Build) void {
    const target = b.standardTargetOptions(.{});
    const optimize = b.standardOptimizeOption(.{});

    const exe = b.addExecutable(.{
        .name = "myapp",
        .root_source_file = b.path("src/main.zig"),
        .target = target,
        .optimize = optimize,
    });

    b.installArtifact(exe);

    // Run command
    const run_cmd = b.addRunArtifact(exe);
    run_cmd.step.dependOn(b.getInstallStep());

    const run_step = b.step("run", "Run the app");
    run_step.dependOn(&run_cmd.step);

    // Tests
    const tests = b.addTest(.{
        .root_source_file = b.path("src/main.zig"),
        .target = target,
        .optimize = optimize,
    });

    const test_step = b.step("test", "Run tests");
    test_step.dependOn(&b.addRunArtifact(tests).step);
}
```

### build.zig.zon (Package Manifest)

```zig
.{
    .name = "mypackage",
    .version = "0.1.0",
    .minimum_zig_version = "0.15.2",
    .dependencies = .{
        .some_dep = .{
            .url = "https://example.com/package.tar.gz",
            .hash = "1220abcd...",
        },
    },
    .paths = .{
        "build.zig",
        "build.zig.zon",
        "src",
    },
}
```

### Build Commands

```bash
zig build                  # Build project
zig build run              # Build and run
zig build test             # Run tests
zig build -Doptimize=ReleaseFast  # Optimized build
zig build --help           # Show build options
```

## Cross-Compilation

### Build for Different Targets

```bash
# Linux x86_64
zig build-exe main.zig -target x86_64-linux

# macOS ARM64
zig build-exe main.zig -target aarch64-macos

# Windows
zig build-exe main.zig -target x86_64-windows

# WebAssembly
zig build-exe main.zig -target wasm32-wasi

# Embedded (ARM Cortex-M)
zig build-exe main.zig -target thumb-freestanding -mcpu=cortex_m4
```

### List Available Targets

```bash
zig targets  # Shows all supported targets and CPUs
```

### Using Zig as a C Compiler

```bash
# Drop-in replacement for gcc/clang
zig cc -o program program.c
zig c++ -o program program.cpp

# Cross-compile C code
zig cc -target aarch64-linux -o program program.c
```

## Standard Library Highlights

### Data Structures

```zig
// Dynamic array
std.ArrayList(T)

// Hash maps
std.StringHashMap(V)
std.AutoHashMap(K, V)

// Linked lists
std.SinglyLinkedList(T)
std.DoublyLinkedList(T)

// Priority queue
std.PriorityQueue(T, Context, compareFn)

// Bit set
std.DynamicBitSet
```

### File I/O

```zig
const std = @import("std");

pub fn main() !void {
    // Read file
    const file = try std.fs.cwd().openFile("input.txt", .{});
    defer file.close();

    const content = try file.readToEndAlloc(allocator, 1024 * 1024);
    defer allocator.free(content);

    // Write file
    const out_file = try std.fs.cwd().createFile("output.txt", .{});
    defer out_file.close();

    try out_file.writeAll("Hello, file!\n");
}
```

### JSON

```zig
const std = @import("std");

const Person = struct {
    name: []const u8,
    age: u32,
};

pub fn main() !void {
    const allocator = std.heap.page_allocator;

    // Parse JSON
    const json_str = "{\"name\":\"Alice\",\"age\":30}";
    const parsed = try std.json.parseFromSlice(Person, allocator, json_str, .{});
    defer parsed.deinit();

    // Stringify
    var buffer = std.ArrayList(u8).init(allocator);
    defer buffer.deinit();

    try std.json.stringify(parsed.value, .{}, buffer.writer());
}
```

### HTTP

```zig
const std = @import("std");

pub fn main() !void {
    var client = std.http.Client{ .allocator = allocator };
    defer client.deinit();

    const uri = try std.Uri.parse("https://example.com");
    var request = try client.open(.GET, uri, .{});
    defer request.deinit();

    try request.send();
    try request.wait();

    const body = try request.reader().readAllAlloc(allocator, 8192);
    defer allocator.free(body);

    std.debug.print("{s}\n", .{body});
}
```

## Innovative Features

### 1. Comptime Metaprogramming

Unlike C macros or C++ templates, Zig runs actual Zig code at compile time, enabling type-safe generic programming without a separate language.

### 2. No Hidden Control Flow

Every operation is explicit. No exceptions, no operator overloading, no hidden function calls. What you see is what executes.

### 3. Explicit Allocators

No global allocator. Every allocation is visible and controlled, making it easy to track memory usage and swap allocation strategies.

### 4. Error Unions

Errors are values in the type system. No exceptions, no error codes hidden in return values. Errors must be handled or explicitly propagated.

### 5. Optional Types Without Runtime Cost

Optional pointers compile to regular pointers with null as address 0, combining safety with zero overhead.

### 6. First-Class Cross-Compilation

Cross-compile to any supported target without installing separate toolchains. Zig ships with everything needed.

### 7. Integrated Build System

Build configuration in Zig, not Make/CMake/shell scripts. Type-safe, cross-platform, and powerful.

### 8. Zig as a C/C++ Compiler

Use Zig to compile C/C++ code with better cross-compilation support than traditional toolchains.

## Roadmap and Evolution

### Current State (v0.15.2)

- **Pre-1.0 Status**: Language is stable enough for real projects but breaking changes occur
- **Self-Hosted Compiler**: Now written in Zig (previously bootstrapped from C++)
- **Package Manager**: Basic functionality available through `build.zig.zon`
- **Async/Await**: Temporarily regressed, being redesigned

### Path to 1.0

1. **Language Stabilization**: Finalize core syntax and semantics
2. **Standard Library**: Stabilize commonly-used APIs
3. **Async Redesign**: Implement stackless coroutines as primitives
4. **Incremental Compilation**: Improve build performance
5. **Tooling**: Mature IDE support, debugging

### After 1.0

- Strong backward compatibility guarantees
- Focus on incremental improvements
- Growing ecosystem and library support
- Wider production adoption

### Philosophy

- Simplicity over feature accretion
- Breaking changes acceptable before 1.0
- Community input valued, BDFN governance
- Focus on robust, optimal, reusable code

## Ecosystem and Tooling

### Official Tools

- **Compiler**: `zig build-exe`, `zig build-lib`, `zig build-obj`
- **Build System**: `zig build`
- **Test Runner**: `zig test`
- **Formatter**: `zig fmt`
- **Package Manager**: Built into `zig build`
- **C Translation**: `zig translate-c`

### Language Server

**ZLS (Zig Language Server)**:
- Autocompletion
- Go-to-definition
- Find references
- Hover documentation
- Diagnostics

Install: Download from [zigtools/zls](https://github.com/zigtools/zls)

### Editor Support

- **VSCode**: [vscode-zig](https://github.com/ziglang/vscode-zig)
- **Vim/Neovim**: [zig.vim](https://github.com/ziglang/zig.vim)
- **Emacs**: [zig-mode](https://github.com/ziglang/zig-mode)
- **Sublime Text**: Language support available

### Learning Resources

- **Ziglings**: Interactive exercises to learn Zig
- **Zig Book**: Comprehensive technical introduction
- **Language Reference**: Official documentation
- **Standard Library Docs**: `zig std` command

### Notable Projects

- **Bun**: Fast JavaScript runtime (uses Zig)
- **TigerBeetle**: Distributed financial database (written in Zig)
- **zig-gamedev**: Game development libraries

## Use Cases

### Systems Programming

- Operating systems
- Drivers
- Embedded systems
- Bootloaders

### Application Development

- CLI tools
- Network servers
- Game engines
- WebAssembly modules

### Build Tool

- Compiling C/C++ projects
- Cross-platform builds
- Replacing Make/CMake

### Education

- Teaching systems programming
- Understanding memory management
- Learning metaprogramming

## Community

### Official Resources

- **Website**: [ziglang.org](https://ziglang.org)
- **Documentation**: [ziglang.org/documentation](https://ziglang.org/documentation/master/)
- **GitHub**: [github.com/ziglang/zig](https://github.com/ziglang/zig)

### Communication

- Discord servers
- IRC channels
- Reddit: r/Zig
- Community forums
- Local meetups

### Contributing

1. Use Zig for personal projects
2. Report bugs with real examples
3. Contribute to "contributor friendly" issues
4. Only work on "accepted" proposals

**Note**: Andrew Kelley (BDFN) has final say on language decisions.

### Support

- **Fund Development**: [Monthly donations](https://ziglang.org/zsf/)
- **Report Bugs**: GitHub Issues
- **Get Help**: Community forums and chat

## References & Resources

### Official Documentation

- [Zig Language Reference](https://ziglang.org/documentation/master/) - Complete language specification
- [Standard Library Documentation](https://ziglang.org/documentation/master/std/) - API reference
- [Zig Build System](https://ziglang.org/learn/build-system/) - Build.zig guide

### Learning Materials

- [Ziglings](https://github.com/ratfactor/ziglings) - Learn by fixing tiny programs
- [Zig Book](https://github.com/pedropark99/zig-book) - Technical introduction
- [Zig Cookbook](https://github.com/zigcc/zig-cookbook) - Code examples
- [Awesome Zig](https://github.com/zigcc/awesome-zig) - Curated resources

### Community Projects

- [zig-gamedev](https://github.com/zig-gamedev/zig-gamedev) - Game development
- [http.zig](https://github.com/karlseguin/http.zig) - HTTP library
- [raylib-zig](https://github.com/raylib-zig/raylib-zig) - Raylib bindings

### Books and Articles

- [Zig Language Reference](https://ziglang.org/documentation/master/) - Primary documentation
- Community blogs and tutorials
- Release notes for version updates

### Tools and Infrastructure

- [ZLS](https://github.com/zigtools/zls) - Language server
- [zig-pypi](https://github.com/ziglang/zig-pypi) - Python packaging
- [docker-zig](https://github.com/ziglang/docker-zig) - Docker images
