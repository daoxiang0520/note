---
title: 机器级编程 I：入门
date: 2026-06-15
tags: [CSAPP, 汇编, x86-64, 寄存器, 机器码]
---

# Machine-Level Programming I: Basics

> CMU 15-213: Introduction to Computer Systems, 5th Lecture, Sep. 15, 2015

---

## Intel 处理器历史

### Intel x86 进化里程碑

| 名称 | 日期 | 晶体管数 | MHz | 特点 |
|------|:---:|:--------:|:---:|------|
| 8086 | 1978 | 29K | 5-10 | 首个 16-bit 处理器，IBM PC & DOS 基础，1MB 地址空间 |
| 386 | 1985 | 275K | 16-33 | 首个 32-bit 处理器（IA32），支持 Unix |
| Pentium 4E | 2004 | 125M | 2800-3800 | 首个 64-bit x86（x86-64） |
| Core 2 | 2006 | 291M | 1060-3500 | 首个多核处理器 |
| Core i7 | 2008 | 731M | 1700-3900 | 四核 |

### CISC vs RISC

x86 是 **CISC**（复杂指令集计算机）：
- 不同指令有不同的格式和长度
- 理论上难以赶上 RISC 性能，但 Intel 在性能上做到了

### AMD

- 曾跟随 Intel 之后，更便宜更慢
- 开发了 x86-64（64 位扩展）
- 目前旗舰 CPU 性能已不输 Intel

---

## C 语言到机器码的翻译过程

### 编译流程

```
C program (p1.c p2.c)
    ↓ gcc -Og -S
Asm program (p1.s p2.s)
    ↓ gcc/as (汇编器)
Object program (p1.o p2.o)
    ↓ gcc/ld (链接器)
Executable program (p)
```

### 示例：翻译成汇编

```c
long plus(long x, long y);

void sumstore(long x, long y, long *dest) {
    long t = plus(x, y);
    *dest = t;
}
```

生成的 x86-64 汇编：

```asm
sumstore:
    pushq   %rbx
    movq    %rdx, %rbx
    call    plus
    movq    %rax, (%rbx)
    popq    %rbx
    ret
```

### 不同语言层次的数据类型

| C 语言 | 汇编语言 |
|--------|----------|
| 基本类型（char, int, long, float, double） | 1、2、4、8 字节整数 |
| 复合类型（array, struct） | 连续分配的字节（无复合类型概念） |
| 指针 | 地址（无类型指针） |
| 代码 | 字节序列编码的指令流 |

### 指令对应关系

| C 语言 | 汇编语言 |
|--------|----------|
| `+`, `-`, `*`, `/` | `add`, `sub`, `mul`, `div` |
| `&&`, `||`, `!` | `and`, `or`, `not` |
| `x = a` | `mov` |
| if-else, for, while | `je`, `jne`, `jmp` |
| 函数调用 `f()` | `call` |

---

## 汇编入门：寄存器、操作数、mov 指令

### x86-64 寄存器

```
%rax   %rcx   %rdx   %rbx   %rsi   %rdi   %rsp   %rbp
%r8    %r9    %r10   %r11   %r12   %r13   %r14   %r15
```

可以引用低 4 字节（%eax, %ebx 等），以及低 1 和 2 字节。

### 操作数的三种类型

1. **立即数**：`$0x400`, `$-533`（以 `$` 为前缀）
2. **寄存器**：16 个通用寄存器之一，如 `%rax`, `%r13`
3. **内存**：由寄存器指定地址处的连续 8 个字节，如 `(%rax)`

### movq 指令

`movq Source, Dest`

| 源/目的 | Imm | Reg | Mem |
|---------|:---:|:---:|:---:|
| Reg | movq $0x4,%rax | - | movq %rax,(%rdx) |
| Mem | movq $-147,(%rax) | movq (%rax),%rdx | **不允许** |

对应 C 语言：

```asm
movq $0x4, %rax    # temp = 0x4;
movq $-147, (%rax) # *p = -147;
movq %rax, %rdx    # temp2 = temp1;
movq %rax, (%rdx)  # *p = temp;
movq (%rax), %rdx  # temp = *p;
```

### Swap 示例

```c
void swap(long *xp, long *yp) {
    long t0 = *xp;
    long t1 = *yp;
    *xp = t1;
    *yp = t0;
}
```

```asm
swap:
    movq    (%rdi), %rax    # t0 = *xp
    movq    (%rsi), %rdx    # t1 = *yp
    movq    %rdx, (%rdi)    # *xp = t1
    movq    %rax, (%rsi)    # *yp = t0
    ret
```

### 完整 mov 指令变体

| 指令 | 描述 |
|------|------|
| `movb` | 移动 1 字节 |
| `movw` | 移动 2 字节 |
| `movl` | 移动 4 字节（也会将高 4 字节清零） |
| `movq` | 移动 8 字节 |
| `movabsq` | 移动 64 位立即数到寄存器 |
| `movslq` | 将 4 字节（int）符号扩展为 8 字节（long） |
| `cltq` | 将 %eax 符号扩展为 %rax（更紧凑编码） |

### 内存寻址模式

| 形式 | 计算方式 | C 语言对应 |
|------|----------|-----------|
| `(R)` | `Mem[Reg[R]]` | `*p` |
| `D(R)` | `Mem[Reg[R] + D]` | `a[2]` |
| `(Rb, Ri)` | `Mem[Reg[Rb] + Reg[Ri]]` | 指针运算 |
| `D(Rb, Ri)` | `Mem[Reg[Rb] + Reg[Ri] + D]` | 结构体访问 |
| `(Rb, Ri, S)` | `Mem[Reg[Rb] + S*Reg[Ri]]` | `a[i]` |
| `D(Rb, Ri, S)` | `Mem[Reg[Rb] + S*Reg[Ri] + D]` | 结构体数组 |

其中 S 可以是 1、2、4 或 8。

### 寻址计算示例

假设 `%rdx = 0xf000`, `%rcx = 0x0100`

| 表达式 | 计算结果 |
|--------|---------|
| `0x8(%rdx)` | `0xf000 + 0x8 = 0xf008` |
| `(%rdx, %rcx)` | `0xf000 + 0x100 = 0xf100` |
| `(%rdx, %rcx, 4)` | `0xf000 + 4*0x100 = 0xf400` |
| `0x80(,%rdx,2)` | `2*0xf000 + 0x80 = 0x1e080` |

---

## 算术逻辑运算

### 地址计算指令 leaq

`leaq Src, Dst` — 将 Src 表示的地址存入 Dst（不修改 flags）

**用途：**
1. 计算地址本身：`p = &x[i]`
2. 计算算术表达式：形如 `x + k*y`（k = 1, 2, 4, 8）

```c
long m12(long x) {
    return x * 12;
}
```

```asm
leaq    (%rdi, %rdi, 2), %rax   # t = x + x*2
salq    $2, %rax                 # return t << 2
```

> LEA 是性能关键指令，它是一条"免费算术指令"：不改 flags、功能强、执行快、对流水线友好。

### 二元算术运算

| 指令 | 含义 |
|------|------|
| `addq Src, Dest` | `Dest += Src` |
| `subq Src, Dest` | `Dest -= Src` |
| `imulq Src, Dest` | `Dest *= Src` |
| `salq Src, Dest` | `Dest <<= Src`（算术左移） |
| `sarq Src, Dest` | `Dest >>= Src`（算术右移） |
| `shrq Src, Dest` | `Dest >>= Src`（逻辑右移） |
| `xorq Src, Dest` | `Dest ^= Src` |
| `andq Src, Dest` | `Dest &= Src` |
| `orq Src, Dest` | `Dest |= Src` |

### 一元算术运算

| 指令 | 含义 |
|------|------|
| `incq Dest` | `Dest++` |
| `decq Dest` | `Dest--` |
| `negq Dest` | `Dest = -Dest` |
| `notq Dest` | `Dest = ~Dest` |

### 算术运算示例

```c
long arith(long x, long y, long z) {
    long t1 = x + y;
    long t2 = z + t1;
    long t3 = x + 4;
    long t4 = y * 48;
    long t5 = t3 + t4;
    long rval = t2 * t5;
    return rval;
}
```

```asm
arith:
    leaq    (%rdi, %rsi), %rax    # t1 = x + y
    addq    %rdx, %rax            # t2 = z + t1
    leaq    (%rsi, %rsi, 2), %rdx # t4_part = 3*y
    salq    $4, %rdx              # t4 = 48*y
    leaq    4(%rdi, %rdx), %rcx   # t5 = x + 4 + t4
    imulq   %rcx, %rax            # rval = t2 * t5
    ret
```

---

## 反汇编

### 使用 objdump

```bash
objdump -d sum
```

### 使用 gdb

```bash
gdb sum
disassemble sumstore
x/14xb sumstore    # 查看从 sumstore 开始的 14 个字节
```

### 什么是可以反汇编的？

任何可被解释为可执行代码的内容都可以反汇编。反汇编工具检查字节并重建汇编源码。

---

## 总结

- Intel 处理器的进化设计导致了许多特殊之处和遗留问题
- C 语言、汇编语言和机器码之间存在明确的翻译关系
- 汇编程序的状态包括：程序计数器（PC）、寄存器、条件码、内存
- x86-64 mov 指令支持各种数据移动操作
- 编译器组合多条汇编指令来实现 C 语言中的算术运算
