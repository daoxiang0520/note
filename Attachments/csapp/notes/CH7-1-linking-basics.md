---
title: CH7-1 Linking 原理
date: 2026-06-15
tags:
  - CSAPP
  - 链接
  - ELF
  - 符号解析
  - 重定位
---

# CH7-1 Linking 原理

> CMU 15-213: Introduction to Computer Systems

## 概述

链接（Linking）是将多个代码和数据片段合并成一个单一文件的过程。该文件可以被复制到内存中并执行。

## 为什么要使用链接器？

### 理由 1: 模块化
- 允许程序被分解成多个更小的模块，解耦合，支持并行开发
- 允许将常用函数组织成库，便于复用

### 理由 2: 效率
- **时间效率**：修改单个源文件只需重新编译该文件，然后重新链接，不必重新编译其他源文件
- **空间效率**：可执行文件和运行的内存镜像中，只需要包含真正使用的那部分函数

## 示例 C 程序

```c
// main.c
int sum(int *a, int n);
int array[2] = {1, 2};
int main() {
    int val = sum(array, 2);
    return val;
}

// sum.c
int sum(int *a, int n) {
    int i, s = 0;
    for (i = 0; i < n; i++) {
        s += a[i];
    }
    return s;
}
```

编译和链接：

```bash
gcc -Og -o prog main.c sum.c
```

## 链接器的两个主要任务

### 任务 1：符号解析

程序会定义和引用符号（特指全局的变量和符号）：

```c
void swap() {...}   /* 定义符号 swap */
swap();             /* 引用符号 swap */
int *xp = &x;       /* 定义符号 xp，引用符号 x */
```

符号的定义由汇编器写入目标文件的**符号表**中。符号表是一个 struct 类型的数组，每个元素包括：符号名、符号的内存占用大小、符号定义所在的位置。

### 任务 2：重定位

1. **合并**：将多个 .o 文件合并到同一个文件
2. **分配地址**：为每个符号分配绝对地址
3. **更新引用**：更新对所有符号的引用，以反映新的绝对地址

## ELF 目标文件格式

ELF（Executable and Linkable Format）是目标文件的标准二进制格式，支持：

- 可重定位的目标文件（.o）
- 可执行的目标文件（a.out）
- 共享的目标文件（.so）
- Core Dumps

### 三种目标文件

| 类型 | 后缀 | 说明 |
|------|------|------|
| 可重定位目标文件 | .o | 可与其他可重定位文件合并成可执行文件 |
| 可执行目标文件 | (无) | 可直接拷贝到内存并执行 |
| 共享目标文件 | .so | 可加载到内存进行动态链接（Windows 中为 DLL） |

### ELF 文件结构

```
ELF header
Program header table (required for executables)
.text section        # 代码
.rodata section      # 只读数据（跳转表、常量字符串、常量浮点数）
.data section        # 初始化的全局或静态变量
.bss section         # 未初始化的全局或静态变量（不占文件大小）
.symtab section      # 符号表
.rel.text section    # .text 节对应的重定位信息
.rel.data section    # .data 节对应的重定位信息
.debug section       # 调试符号信息
Section header table
```

### 链接视图 vs 执行视图

- **链接视图**（可重定位文件）：由节（section）组成，.text, .data, .rodata, .bss 等
- **执行视图**（可执行文件）：由段（segment）组成，描述节如何映射到存储段中

### 可执行目标文件的额外内容

- ELF 头中 `e_entry` 字段给出执行时第一条指令的地址
- 多一个 `.init` 节（初始化代码）
- 少两个 `.rel` 节（无需重定位）
- 多一个**程序头表**（segment header table），描述如何将节映射到存储空间

## 链接器符号的分类

| 符号类型 | 说明 | 示例 |
|----------|------|------|
| **全局符号**（Global） | 当前模块定义，可被其他模块引用 | 非 static 函数和全局变量 |
| **外部符号**（External） | 当前模块引用，在其他模块中定义 | 引用外部函数或变量 |
| **局部符号**（Local） | 当前模块定义，只被当前模块引用 | static 修饰的函数和全局变量 |

> 注意：局部的链接器符号 != 程序的局部变量。局部 non-static 变量存放于寄存器或栈上，局部 static 变量存放于 .bss 或 .data 节。

## 符号解析规则

### 强符号 vs 弱符号

| 类型 | 定义 |
|------|------|
| **强符号** | 函数定义、初始化的全局变量 |
| **弱符号** | 未初始化的全局变量 |

### 三条核心规则

1. **规则 1**：不允许存在多个强符号（否则链接器报错）
2. **规则 2**：如果有 1 个强符号定义和多个弱符号定义，将所有引用解析到强符号定义
3. **规则 3**：如果有多个弱符号，任意选择一个（dangerous!）

### 重复定义举例

**两种类型强定义**：链接报错

```c
// main.c              // p1.c
int x=10;               int x=20;
int p1(void);           int p1() { return x; }
int main() {
    x = p1();
    return x;
}
```

**不同类型的强定义+弱定义**：可能导致难以理解的错误

```c
// main.c (int d=100, int x=200)
// p1.c    (double d; ... d=1.0;)
// 结果：d=0, x=0x3FF00000 (1.0 的 double 表示)
```

### 全局变量的使用建议

- 如果可以，尽量避免使用全局变量
- 如果可以，尽量使用 `static`
- 如果定义了一个全局变量，请初始化
- 如果引用了外部的全局变量，请使用 `extern`
- GCC 选项：`-fno-common`

## 重定位

### 重定位过程

1. **地址分配**：多文件合并，为每个节和符号分配地址
2. **符号重写**：借助重定位表，重写寻址字段

### 重定位类型

- **绝对寻址**（Abs32）：将符号的绝对地址写入指令
- **PC 相对寻址**：`jump-target = %RIP + offset`

### 重定位项格式

```
offset: 在节中的偏移
symbol: 引用的符号
type:   重定位类型（绝对寻址或 PC 相对寻址）
addend: 与下一条指令地址的距离
```

### PC 相对寻址示例

```
callq sum 指令的编码：
待修改位置：main.address (0x4004d0) + offset(0xf) = 0x4004df
当前 PC：refAddr - addend = 0x4004df - (-4) = 0x4004e3
新值：sum.addr - 当前 PC = 0x4004e8 - 0x4004e3 = 0x5
```

## 工具和参考

**常用工具**：
- `objdump`：查看目标文件
- `readelf`：读取 ELF 文件信息
- GNU Binutils：ELF Swiss Army Knife

**课外参考**：《程序员的自我修养：链接、装载与库》
