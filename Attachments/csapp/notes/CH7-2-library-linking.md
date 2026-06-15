---
title: CH7-2 库的链接及插桩
date: 2026-06-15
tags:
  - CSAPP
  - 链接
  - 静态库
  - 动态库
  - 插桩
---

# CH7-2 库的链接及插桩

> CMU 15-213: Introduction to Computer Systems

## 概述

本章涵盖静态库的创建与链接、共享库（动态库）的实现原理，以及库插桩技术。

## 静态库

静态库（`.a` 文件）将多个相关的可重定位目标文件串接成一个带索引的单一文件（archive 包）。

### 创建静态库

```bash
ar rs libc.a atoi.o printf.o ... random.o
```

**常用静态库**：
- `libc.a`（C 标准库）：4.6 MB，1496 个目标文件（I/O、内存分配、信号处理、字符串处理等）
- `libm.a`（C 数学库）：2 MB，444 个目标文件（sin, cos, tan, log, exp, sqrt 等）

### 链接静态库

```c
// main2.c
#include <stdio.h>
#include "vector.h"
int x[2] = {1, 2};
int y[2] = {3, 4};
int z[2];
int main() {
    addvec(x, y, z, 2);
    printf("z = [%d %d]\n", z[0], z[1]);
    return 0;
}
```

```bash
gcc -L. main2.o -lvector.a
```

### 链接器解析外部符号的算法

按照**从左至右**的顺序、按需单次扫描处理命令行参数中的文件。在扫描过程中维护 3 个列表：

1. 已定义符号表
2. 未定义符号表（待解析的符号）
3. 未定的弱符号表（如果没找到强符号，作为备用定义）

遇到 `.a` 文件时，遍历其中的每个 `.o` 文件：
- 检查该文件能否补全未定义符号表
- 如果可以，则提取并链接该 `.o` 文件，并同步更新 3 个列表
- 如果不行，则丢弃该 `.o` 文件

### 静态库的问题

**问题 1：无用的库函数冗余**
某个被提取的 .o 文件中，可能包含 90% 未被引用的函数定义，但也会被链接到可执行文件。可通过 LTO（Linking-Time Optimization）识别无用函数并删除。

**问题 2：解析结果受参数顺序影响**
最佳实践：将 `.a` 文件放在 `.o` 文件的后面。

```bash
# 正确
gcc -L. main2.o -lvector.a
# 错误：会导致 undefined reference
gcc -L. -lvector.a main2.o
```

**问题 3：循环依赖**
如果 libx.a 依赖 liby.a 且 liby.a 依赖 libx.a，需要在命令行中重复列出：

```bash
gcc -static -o myfunc func.o libx.a liby.a libx.a
```

### 静态库的缺点

1. 二进制文件中存在大量重复的库函数（每个程序都要使用 libc）
2. 运行时的二进制镜像中存在大量重复的库函数
3. 为修复系统库中一个小 bug，需要每个应用程序都重新链接

## 共享库（动态库）

共享库（`.so` / `.dll` / `.dylib`）中的代码和数据可以在加载时或者运行时动态地链接到应用程序。

### 共享库的优点

- 可以被多个进程共享
- 更新库文件无需重新链接应用程序
- 减小可执行文件体积

### 加载时链接

在可执行文件第一次被加载时完成动态链接，由动态链接器（`ld-linux.so`）自动完成。

```bash
# 创建共享库
gcc -shared -o libvector.so addvec.c multvec.c

# 编译时指定共享库
gcc -o prog2l main2.c libvector.so
```

### 运行时链接

在程序已经开始运行之后完成动态链接，通过 `dlopen()` 接口完成。

```c
#include <stdio.h>
#include <stdlib.h>
#include <dlfcn.h>

int main() {
    void *handle;
    void (*addvec)(int*, int*, int*, int);
    
    // 动态加载共享库
    handle = dlopen("./libvector.so", RTLD_LAZY);
    
    // 获取函数地址
    addvec = dlsym(handle, "addvec");
    
    // 调用函数
    addvec(x, y, z, 2);
    
    // 卸载共享库
    dlclose(handle);
    return 0;
}
```

编译命令：

```bash
gcc -o prog2r dll.c -ldl
```

### Java 中的类似机制

```bash
# 编译时指定库路径（不链接）
javac -cp .:lib/some-library.jar MyClass.java

# 运行时指定库路径（运行时链接/加载）
java -cp .:lib/some-library.jar MyClass
```

### 位置无关代码（PIC）

共享库代码必须是位置无关的（Position-Independent Code），由 GCC 选项 `-fPIC` 生成。

#### 4 种引用情况

| 引用类型 | 解决方法 |
|----------|----------|
| 模块内的代码引用 | PC 相对偏移寻址 |
| 模块内的数据引用 | 运行时 PC + 偏移 |
| 模块间的数据引用 | GOT（全局偏移表） |
| 模块间的代码引用 | GOT + PLT（过程链接表） |

#### GOT（Global Offset Table）

在 `.data` 节中设置一个指针数组，指向全局变量。GOT 与引用数据的指令之间相对距离固定。

> "Any problem in computer science can be solved with another level of indirection." -- David Wheeler

#### PLT（Procedure Linkage Table）

用于**延迟绑定**：不在加载时重定位，而是延迟到第一次函数调用时。

- GOT[0]：`.dynamic` 节首址
- GOT[1]：动态链接器标识信息
- GOT[2]：动态链接器延迟绑定代码的入口地址
- GOT[3+]：各共享库函数的地址

#### so 文件的加载过程

**启动时**：级联搜索和加载该 so 依赖的所有 so 文件

1. 扫描 `.dynamic` 段，识别依赖的 so 文件路径
2. 搜索文件路径，加入 `ld.so.cache`
3. 加载 so 文件到内存
4. 绑定所有库变量
5. 级联搜索、加载、绑定

**运行时**：延迟绑定和访问库函数

## 库插桩（Library Interpositioning）

库插桩是一种强大的链接技术，使得程序员可以截获对任意函数的调用。

### 插桩的应用场景

- **安全**：限制（沙盒）、后台加密
- **调试**：截获函数调用，跟踪问题
- **监控与侧写**：函数调用计数、内存泄漏跟踪、访存地址跟踪

### 三种插桩方式

#### 1. 编译时插桩

```c
// mymalloc.c
#ifdef COMPILETIME
#include <stdio.h>
#include <malloc.h>

void *mymalloc(size_t size) {
    void *ptr = malloc(size);
    printf("malloc(%d)=%p\n", (int)size, ptr);
    return ptr;
}

void myfree(void *ptr) {
    free(ptr);
    printf("free(%p)\n", ptr);
}
#endif
```

```c
// malloc.h
#define malloc(size) mymalloc(size)
#define free(ptr) myfree(ptr)
```

```bash
gcc -DCOMPILETIME -c mymalloc.c
gcc -I. -o intc int.c mymalloc.o
```

#### 2. 链接时插桩

使用 `--wrap` 标志指示链接器进行特殊符号解析：

```bash
gcc -DLINKTIME -c mymalloc.c
gcc -c int.c
gcc -Wl,--wrap,malloc -Wl,--wrap,free -o intl int.o mymalloc.o
```

- 对 `malloc` 的引用解析成 `__wrap_malloc`
- 对 `__real_malloc` 的引用解析成 `malloc`

#### 3. 加载/运行时插桩

使用 `LD_PRELOAD` 环境变量和 `dlsym(RTLD_NEXT, ...)` 实现：

```c
#ifdef RUNTIME
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <dlfcn.h>

void *malloc(size_t size) {
    void *(*mallocp)(size_t size);
    mallocp = dlsym(RTLD_NEXT, "malloc");
    char *ptr = mallocp(size);
    printf("malloc(%d) = %p\n", (int)size, ptr);
    return ptr;
}
#endif
```

```bash
gcc -DRUNTIME -shared -fpic -o mymalloc.so mymalloc.c -ldl
gcc -o intr int.c
LD_PRELOAD="./mymalloc.so" ./intr
```

### 插桩方式对比

| 方式 | 时机 | 原理 |
|------|------|------|
| 编译时 | 编译 | 宏替换，将对 malloc/free 的调用替换为自定义函数 |
| 链接时 | 静态链接 | 使用链接器做特殊名字解析（wrap/unwrap） |
| 加载/运行时 | 动态链接 | 利用 LD_PRELOAD 覆盖库函数 |
