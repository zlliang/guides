# Zig 编程语言

一份关于 Zig 的全面指南——一种通用编程语言和工具链，用于维护健壮、优化和可重用的软件。本指南涵盖 Zig 0.15.2 版本，包括其理念、特性、语法和生态系统。

## 快速参考

```zig
// 变量
const x = 5;                    // 不可变
var y: i32 = 10;               // 可变且带类型
var z = @as(f64, 3.14);        // 类型转换

// 函数
fn add(a: i32, b: i32) i32 {
    return a + b;
}

// 错误处理
fn mayFail() !void {
    try somethingFallible();
    const result = operation() catch 0;
}

// 可选类型
const maybe: ?i32 = null;
const value = maybe orelse 0;

// 编译期计算
fn List(comptime T: type) type {
    return struct {
        items: []T,
    };
}

// 内存分配
const allocator = std.heap.page_allocator;
const list = try std.ArrayList(i32).initCapacity(allocator, 10);
defer list.deinit();

// C 互操作
const c = @cImport(@cInclude("stdio.h"));
_ = c.printf("Hello\n");
```

## 简介

### 什么是 Zig？

Zig 是由 **Andrew Kelley** 于 2015 年创建的通用编程语言。它被设计为系统编程中 C 语言的更好替代品，专注于：

- **健壮性**：即使在内存不足等边缘情况下也能正确运行
- **最优性能**：编写尽可能好的代码，零成本抽象
- **可重用性**：相同的代码可在不同环境和约束下工作
- **可维护性**：意图清晰，代码阅读开销低

与 Rust（通过所有权强调安全性）或 Go（通过垃圾回收器强调简单性）不同，Zig 采用了不同的方法：**手动内存管理，最大化清晰度，无隐藏控制流**。

### 为什么使用 Zig？

**替代 C**：
- 编译期代码执行而非宏
- 显式错误处理而非错误码
- 可选类型而非空指针
- 安全构建中无未定义行为
- 一流的跨平台编译支持

**替代 C++**：
- 无隐藏控制流
- 无运算符重载
- 功能更少、更简单的语言
- 更快的编译时间
- 无异常

**替代 Rust**：
- 更简单的语言（无生命周期，无借用检查器）
- 手动内存管理给你完全的控制权
- 代码行为更透明
- 更容易与 C 集成

**独特优势**：
- 用于元编程的编译期执行
- 可作为支持跨编译的 C/C++ 编译器
- 无隐藏的内存分配
- 用 Zig 编写的构建系统

### 当前状态

- **版本**：0.15.2（截至 2025 年 10 月）
- **开发阶段**：1.0 之前，积极演进
- **稳定性**：版本之间可能有破坏性更改
- **生产使用**：适合能够随语言更新的项目
- **许可证**：MIT (Expat)

## 历史和理念

### 起源

Zig 由 **Andrew Kelley** 创建，他担任项目的 BDFN（暂时的仁慈独裁者）。开发始于 2015 年 8 月，核心目标如下：

1. 提供具有现代特性的 C 的更好替代品
2. 启用无需宏系统的编译期代码执行
3. 提供一流的跨平台编译支持
4. 消除隐藏的控制流和内存分配
5. 支持无默认分配器的手动内存管理

该语言由 **Zig 软件基金会 (ZSF)** 支持，这是一个 501(c)(3) 非营利组织，通过每月捐款资助核心开发。

### 设计原则

**无隐藏控制流**：
每行 Zig 代码都是透明的。无运算符重载，无异常，无属性。函数调用看起来就像函数调用。

**无隐藏内存分配**：
所有内存分配都是显式的。库接受 `Allocator` 参数，而不是假设全局分配器。

**手动内存管理**：
像 C 一样，但有更好的工具。无垃圾回收器，但也无隐式的 malloc/free。

**编译期执行**：
在编译时运行 Zig 代码进行元编程，无需单独的模板或宏语言。

**一流的跨平台编译**：
无需单独的工具链即可跨编译到任何支持的目标。Zig 自带多个平台的 libc 头文件。

## 安装和设置

### 下载

访问 [ziglang.org/download](https://ziglang.org/download/) 获取官方版本。

### 包管理器

```bash
# Homebrew (macOS/Linux)
brew install zig

# Python (pip)
pip install ziglang

# Arch Linux
pacman -S zig

# 从源码
git clone https://github.com/ziglang/zig
cd zig
zig build
```

### 验证安装

```bash
zig version  # 应显示 0.15.2
```

### Hello World

**hello.zig**:
```zig
const std = @import("std");

pub fn main() void {
    std.debug.print("Hello, World!\n", .{});
}
```

**编译和运行**:
```bash
zig build-exe hello.zig
./hello

# 或直接运行
zig run hello.zig
```

## 语言基础

### 变量

```zig
// 常量（不可变）
const x = 5;
const y: i32 = 10;  // 显式类型

// 变量（可变）
var z = 20;
var w: f64 = 3.14;

// 未定义（使用前必须初始化）
var a: i32 = undefined;
a = 42;
```

### 类型

**整数**：
```zig
i8, i16, i32, i64, i128    // 有符号
u8, u16, u32, u64, u128    // 无符号
isize, usize               // 指针大小
```

**浮点数**：
```zig
f16, f32, f64, f80, f128
```

**其他类型**：
```zig
bool                 // true 或 false
void                 // 无值
type                 // 类型的类型
anyerror            // 任何错误
comptime_int        // 编译期整数
comptime_float      // 编译期浮点数
```

**类型转换**：
```zig
const a: i32 = 5;
const b: f64 = @floatFromInt(a);  // 显式
const c = @as(u8, 255);           // 类型断言
```

### 函数

```zig
// 基本函数
fn add(a: i32, b: i32) i32 {
    return a + b;
}

// Void 返回（无返回值）
fn greet() void {
    std.debug.print("Hello!\n", .{});
}

// 公共函数（从模块导出）
pub fn publicFunc() void {
    // ...
}

// 内联函数
inline fn fastFunc() void {
    // ...
}

// 导出到 C
export fn cFunc(x: i32) i32 {
    return x * 2;
}

// 外部 C 函数
extern "c" fn malloc(size: usize) ?*anyopaque;
```

### 控制流

**If 语句**：
```zig
if (condition) {
    // ...
} else if (other) {
    // ...
} else {
    // ...
}

// If 作为表达式
const result = if (x > 0) 1 else -1;
```

**While 循环**：
```zig
var i: u32 = 0;
while (i < 10) : (i += 1) {
    std.debug.print("{}\n", .{i});
}

// 带 continue 表达式
while (i < 10) {
    i += 1;
}
```

**For 循环**：
```zig
const items = [_]i32{1, 2, 3, 4, 5};

// 遍历数组
for (items) |item| {
    std.debug.print("{}\n", .{item});
}

// 带索引
for (items, 0..) |item, i| {
    std.debug.print("{}: {}\n", .{i, item});
}

// 遍历范围
for (0..10) |i| {
    std.debug.print("{}\n", .{i});
}
```

**Switch**：
```zig
const x = 2;
const result = switch (x) {
    0 => "zero",
    1, 2 => "one or two",
    3...10 => "three to ten",
    else => "other",
};
```

### 数组和切片

**数组**：
```zig
// 固定大小数组
const arr = [_]i32{1, 2, 3, 4, 5};  // 类型推断
const arr2: [5]i32 = undefined;      // 未初始化
const arr3 = [_]i32{1} ** 10;        // 重复: [1,1,1,...]

// 多维数组
const matrix = [_][3]i32{
    [_]i32{1, 2, 3},
    [_]i32{4, 5, 6},
};
```

**切片**（数组视图）：
```zig
const arr = [_]i32{1, 2, 3, 4, 5};
const slice: []const i32 = &arr;        // 完整切片
const sub_slice = arr[1..4];            // [2, 3, 4]
```

### 结构体

```zig
const Point = struct {
    x: f64,
    y: f64,

    // 方法
    pub fn distance(self: Point, other: Point) f64 {
        const dx = self.x - other.x;
        const dy = self.y - other.y;
        return @sqrt(dx * dx + dy * dy);
    }

    // 关联函数（无 self）
    pub fn origin() Point {
        return Point{ .x = 0, .y = 0 };
    }
};

const p1 = Point{ .x = 1.0, .y = 2.0 };
const p2 = Point.origin();
const dist = p1.distance(p2);
```

### 枚举

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

// 标记联合（带载荷的枚举）
const Value = union(enum) {
    int: i32,
    float: f64,
    string: []const u8,
};

const v = Value{ .int = 42 };
```

### 指针

```zig
// 单项指针
var x: i32 = 5;
const ptr: *i32 = &x;
ptr.* = 10;  // 解引用

// 多项指针（类似 C 指针）
const arr = [_]i32{1, 2, 3};
const ptr_many: [*]const i32 = &arr;

// 切片（胖指针：指针 + 长度）
const slice: []const i32 = &arr;
```

## 错误处理

### 错误集

```zig
const FileError = error{
    FileNotFound,
    PermissionDenied,
    OutOfMemory,
};

const AllErrors = error{
    NetworkError,
} || FileError;  // 合并错误集
```

### 错误联合

```zig
fn divide(a: i32, b: i32) !i32 {
    if (b == 0) return error.DivisionByZero;
    return @divTrunc(a, b);
}

// 使用错误联合
const result = divide(10, 2) catch |err| {
    std.debug.print("Error: {}\n", .{err});
    return;
};
```

### try 关键字

```zig
// 向上传播错误
fn parseAndDouble(str: []const u8) !i32 {
    const num = try std.fmt.parseInt(i32, str, 10);
    return num * 2;
}

// 等价于：
fn parseAndDouble(str: []const u8) !i32 {
    const num = std.fmt.parseInt(i32, str, 10) catch |err| return err;
    return num * 2;
}
```

### catch 和 errdefer

```zig
// 提供默认值
const num = parseNumber(str) catch 0;

// 处理特定错误
const num = parseNumber(str) catch |err| switch (err) {
    error.Overflow => return error.TooLarge,
    error.InvalidCharacter => return error.BadInput,
};

// 错误时清理
fn createResource() !*Resource {
    const resource = try allocate();
    errdefer deallocate(resource);  // 仅在函数返回错误时运行
    
    try initialize(resource);
    return resource;
}
```

## 可选类型

### 基础

```zig
// 可选整数（可以为 null）
const maybe: ?i32 = null;
const definitely: ?i32 = 42;

// 使用 orelse 解包
const value = maybe orelse 0;

// 使用 if 检查
if (maybe) |val| {
    // val 是 i32
    std.debug.print("Value: {}\n", .{val});
} else {
    std.debug.print("No value\n", .{});
}
```

### 可选指针

```zig
// 可选指针编译为常规指针
// null 表示为地址 0
const ptr: ?*i32 = null;

// 这是内存高效的
const std = @import("std");
std.debug.assert(@sizeOf(?*i32) == @sizeOf(*i32));
```

## 内存管理

### 分配器

Zig 中的所有内存分配都需要显式的分配器：

```zig
const std = @import("std");

// 带泄漏检测的通用分配器
var gpa = std.heap.GeneralPurposeAllocator(.{}){};
defer _ = gpa.deinit();
const allocator = gpa.allocator();

// 分配内存
const buffer = try allocator.alloc(u8, 1024);
defer allocator.free(buffer);

// 分配单个项
const item = try allocator.create(MyStruct);
defer allocator.destroy(item);
```

### 标准分配器

```zig
// 通用（调试模式下检测泄漏）
std.heap.GeneralPurposeAllocator(.{})

// C 的 malloc/free
std.heap.c_allocator

// 操作系统页分配器
std.heap.page_allocator

// Arena（批量释放）
var arena = std.heap.ArenaAllocator.init(std.heap.page_allocator);
defer arena.deinit();  // 一次释放所有
const allocator = arena.allocator();

// 固定缓冲区（栈分配）
var buffer: [1024]u8 = undefined;
var fba = std.heap.FixedBufferAllocator.init(&buffer);
const allocator = fba.allocator();

// 测试分配器（检测泄漏）
const allocator = std.testing.allocator;
```

### ArrayList 示例

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

## 编译期执行 (comptime)

### 基本 Comptime

```zig
// 在编译时计算
const result = comptime fibonacci(10);

fn fibonacci(n: u32) u32 {
    if (n == 0 or n == 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}
```

### 泛型数据结构

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
            // 实现
        }
    };
}

// 使用它
const IntList = List(i32);
const FloatList = List(f64);
```

### Comptime 参数

```zig
fn max(comptime T: type, a: T, b: T) T {
    return if (a > b) a else b;
}

const result = max(i32, 10, 20);  // T 是 i32
const result2 = max(f64, 3.14, 2.71);  // T 是 f64
```

### 编译期反射

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
    printFieldNames(Point);  // 打印 "x" 和 "y"
}
```

### Comptime 块

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

## 测试

### 测试块

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

### 运行测试

```bash
zig test myfile.zig
zig test --test-filter "add" myfile.zig  # 运行特定测试
```

## C 互操作性

### 导入 C 头文件

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

### 调用 C 函数

```zig
extern "c" fn malloc(size: usize) ?*anyopaque;
extern "c" fn free(ptr: *anyopaque) void;

pub fn main() void {
    const ptr = malloc(100).?;
    defer free(ptr);
}
```

### 导出到 C

```zig
export fn add(a: i32, b: i32) i32 {
    return a + b;
}

// 从 C 调用:
// extern int add(int a, int b);
```

### C 类型

```zig
c_char, c_short, c_int, c_long, c_longlong
c_ushort, c_uint, c_ulong, c_ulonglong
c_longdouble
```

### translate-c

```bash
zig translate-c header.h > header.zig
```

## 构建系统

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

    // 运行命令
    const run_cmd = b.addRunArtifact(exe);
    run_cmd.step.dependOn(b.getInstallStep());

    const run_step = b.step("run", "Run the app");
    run_step.dependOn(&run_cmd.step);

    // 测试
    const tests = b.addTest(.{
        .root_source_file = b.path("src/main.zig"),
        .target = target,
        .optimize = optimize,
    });

    const test_step = b.step("test", "Run tests");
    test_step.dependOn(&b.addRunArtifact(tests).step);
}
```

### build.zig.zon（包清单）

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

### 构建命令

```bash
zig build                  # 构建项目
zig build run              # 构建并运行
zig build test             # 运行测试
zig build -Doptimize=ReleaseFast  # 优化构建
zig build --help           # 显示构建选项
```

## 跨平台编译

### 为不同目标构建

```bash
# Linux x86_64
zig build-exe main.zig -target x86_64-linux

# macOS ARM64
zig build-exe main.zig -target aarch64-macos

# Windows
zig build-exe main.zig -target x86_64-windows

# WebAssembly
zig build-exe main.zig -target wasm32-wasi

# 嵌入式（ARM Cortex-M）
zig build-exe main.zig -target thumb-freestanding -mcpu=cortex_m4
```

### 列出可用目标

```bash
zig targets  # 显示所有支持的目标和 CPU
```

### 将 Zig 用作 C 编译器

```bash
# gcc/clang 的替代品
zig cc -o program program.c
zig c++ -o program program.cpp

# 跨编译 C 代码
zig cc -target aarch64-linux -o program program.c
```

## 标准库亮点

### 数据结构

```zig
// 动态数组
std.ArrayList(T)

// 哈希映射
std.StringHashMap(V)
std.AutoHashMap(K, V)

// 链表
std.SinglyLinkedList(T)
std.DoublyLinkedList(T)

// 优先队列
std.PriorityQueue(T, Context, compareFn)

// 位集
std.DynamicBitSet
```

### 文件 I/O

```zig
const std = @import("std");

pub fn main() !void {
    // 读取文件
    const file = try std.fs.cwd().openFile("input.txt", .{});
    defer file.close();

    const content = try file.readToEndAlloc(allocator, 1024 * 1024);
    defer allocator.free(content);

    // 写入文件
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

    // 解析 JSON
    const json_str = "{\"name\":\"Alice\",\"age\":30}";
    const parsed = try std.json.parseFromSlice(Person, allocator, json_str, .{});
    defer parsed.deinit();

    // 序列化
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

## 创新特性

### 1. Comptime 元编程

与 C 宏或 C++ 模板不同，Zig 在编译时运行实际的 Zig 代码，实现类型安全的泛型编程，无需单独的语言。

### 2. 无隐藏控制流

每个操作都是显式的。无异常，无运算符重载，无隐藏的函数调用。你看到的就是执行的。

### 3. 显式分配器

无全局分配器。每次分配都是可见和可控的，便于跟踪内存使用并交换分配策略。

### 4. 错误联合

错误是类型系统中的值。无异常，无隐藏在返回值中的错误码。错误必须处理或显式传播。

### 5. 零运行时成本的可选类型

可选指针编译为常规指针，null 为地址 0，结合了安全性和零开销。

### 6. 一流的跨平台编译

无需安装单独的工具链即可跨编译到任何支持的目标。Zig 自带所需的一切。

### 7. 集成构建系统

用 Zig 而非 Make/CMake/shell 脚本配置构建。类型安全、跨平台且功能强大。

### 8. Zig 作为 C/C++ 编译器

使用 Zig 编译 C/C++ 代码，比传统工具链具有更好的跨编译支持。

## 路线图和演进

### 当前状态 (v0.15.2)

- **1.0 之前状态**：语言对于实际项目足够稳定，但会发生破坏性更改
- **自托管编译器**：现在用 Zig 编写（之前从 C++ 引导）
- **包管理器**：通过 `build.zig.zon` 提供基本功能
- **Async/Await**：暂时退化，正在重新设计

### 通往 1.0 的道路

1. **语言稳定化**：最终确定核心语法和语义
2. **标准库**：稳定常用 API
3. **Async 重新设计**：将无栈协程实现为原语
4. **增量编译**：提高构建性能
5. **工具**：成熟的 IDE 支持、调试

### 1.0 之后

- 强向后兼容性保证
- 专注于增量改进
- 不断增长的生态系统和库支持
- 更广泛的生产采用

### 理念

- 简单性胜过功能堆积
- 1.0 之前可接受破坏性更改
- 重视社区意见，BDFN 治理
- 专注于健壮、优化、可重用的代码

## 生态系统和工具

### 官方工具

- **编译器**：`zig build-exe`、`zig build-lib`、`zig build-obj`
- **构建系统**：`zig build`
- **测试运行器**：`zig test`
- **格式化器**：`zig fmt`
- **包管理器**：内置于 `zig build`
- **C 翻译**：`zig translate-c`

### 语言服务器

**ZLS (Zig 语言服务器)**：
- 自动补全
- 跳转到定义
- 查找引用
- 悬停文档
- 诊断

安装：从 [zigtools/zls](https://github.com/zigtools/zls) 下载

### 编辑器支持

- **VSCode**：[vscode-zig](https://github.com/ziglang/vscode-zig)
- **Vim/Neovim**：[zig.vim](https://github.com/ziglang/zig.vim)
- **Emacs**：[zig-mode](https://github.com/ziglang/zig-mode)
- **Sublime Text**：语言支持可用

### 学习资源

- **Ziglings**：交互式练习学习 Zig
- **Zig Book**：全面的技术介绍
- **语言参考**：官方文档
- **标准库文档**：`zig std` 命令

### 知名项目

- **Bun**：快速 JavaScript 运行时（使用 Zig）
- **TigerBeetle**：分布式金融数据库（用 Zig 编写）
- **zig-gamedev**：游戏开发库

## 使用场景

### 系统编程

- 操作系统
- 驱动程序
- 嵌入式系统
- 引导加载程序

### 应用开发

- CLI 工具
- 网络服务器
- 游戏引擎
- WebAssembly 模块

### 构建工具

- 编译 C/C++ 项目
- 跨平台构建
- 替代 Make/CMake

### 教育

- 教授系统编程
- 理解内存管理
- 学习元编程

## 社区

### 官方资源

- **网站**：[ziglang.org](https://ziglang.org)
- **文档**：[ziglang.org/documentation](https://ziglang.org/documentation/master/)
- **GitHub**：[github.com/ziglang/zig](https://github.com/ziglang/zig)

### 交流

- Discord 服务器
- IRC 频道
- Reddit：r/Zig
- 社区论坛
- 本地聚会

### 贡献

1. 在个人项目中使用 Zig
2. 用真实示例报告错误
3. 为"contributor friendly"问题做贡献
4. 只处理"accepted"提案

**注意**：Andrew Kelley（BDFN）对语言决策有最终决定权。

### 支持

- **资助开发**：[每月捐赠](https://ziglang.org/zsf/)
- **报告错误**：GitHub Issues
- **获取帮助**：社区论坛和聊天

## 参考资源

### 官方文档

- [Zig 语言参考](https://ziglang.org/documentation/master/) - 完整的语言规范
- [标准库文档](https://ziglang.org/documentation/master/std/) - API 参考
- [Zig 构建系统](https://ziglang.org/learn/build-system/) - Build.zig 指南

### 学习材料

- [Ziglings](https://github.com/ratfactor/ziglings) - 通过修复小程序学习
- [Zig Book](https://github.com/pedropark99/zig-book) - 技术介绍
- [Zig Cookbook](https://github.com/zigcc/zig-cookbook) - 代码示例
- [Awesome Zig](https://github.com/zigcc/awesome-zig) - 精选资源

### 社区项目

- [zig-gamedev](https://github.com/zig-gamedev/zig-gamedev) - 游戏开发
- [http.zig](https://github.com/karlseguin/http.zig) - HTTP 库
- [raylib-zig](https://github.com/raylib-zig/raylib-zig) - Raylib 绑定

### 书籍和文章

- [Zig 语言参考](https://ziglang.org/documentation/master/) - 主要文档
- 社区博客和教程
- 版本更新的发行说明

### 工具和基础设施

- [ZLS](https://github.com/zigtools/zls) - 语言服务器
- [zig-pypi](https://github.com/ziglang/zig-pypi) - Python 打包
- [docker-zig](https://github.com/ziglang/docker-zig) - Docker 镜像
