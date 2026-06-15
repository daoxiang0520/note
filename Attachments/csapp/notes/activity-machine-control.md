---
title: CSAPP 活动笔记 - 机器级控制流
date: 2024-09
tags: #CSAPP #汇编 #控制流 #条件码
---

# 机器级控制流 (Machine-Level Control Flow)

## 条件码 (Condition Codes)

四个单比特寄存器，由大多数算术指令隐式设置（**MOV** 和 **LEA** 不设置条件码）：

| 标志位 | 含义 |
|--------|------|
| **ZF** | 运算结果为零 (Zero Flag) |
| **SF** | 运算结果为负 (Sign Flag) |
| **CF** | 无符号溢出，最高位产生进位 (Carry Flag) |
| **OF** | 有符号溢出，两个输入符号位相同但输出符号位不同 (Overflow Flag) |

> **CMP** 和 **TEST** 指令只设置条件码，不修改目标寄存器。

---

## 跳转指令 (Jump Instructions)

跳转指令改变程序计数器 `%rip`。以下是 15 条基本跳转指令：

| 名称 | 别名 | 跳转条件 | CMP 后含义 |
|------|------|----------|-----------|
| `JMP` | — | 无条件跳转 | — |
| `JS` | — | SF = 1（负数） | — |
| `JNS` | — | SF = 0（非负） | — |
| `JO` | — | OF = 1（有符号溢出） | — |
| `JNO` | — | OF = 0（无有符号溢出） | — |
| `JE` | `JZ` | ZF = 1 | 相等 |
| `JNE` | `JNZ` | ZF = 0 | 不相等 |
| `JB` | `JC, JNAE` | CF = 1 | 无符号低于 |
| `JAE` | `JNC, JNB` | CF = 0 | 无符号高于或等于 |
| `JA` | `JNBE` | CF = 0 且 ZF = 0 | 无符号高于 |
| `JBE` | `JNA` | CF = 1 或 ZF = 1 | 无符号低于或等于 |
| `JL` | `JNGE` | SF != OF | 有符号小于 |
| `JGE` | `JNL` | SF = OF | 有符号大于或等于 |
| `JG` | `JNLE` | ZF = 0 且 SF = OF | 有符号大于 |
| `JLE` | `JNG` | ZF = 1 或 SF != OF | 有符号小于或等于 |

> 注意：`objdump` 和 `gdb` 反汇编时，别名指令会统一显示为"名称"列的形式。

### 跳转指令编码

- 每条跳转指令的第二字节是**偏移量**（相对于下一条指令地址）
- 目标地址 = 当前指令地址 + 指令长度 + 偏移量

---

## CMP 和 TEST 指令

### CMP 指令
- 格式：`CMP src2, src1` → 计算 `src1 - src2`
- 只设置条件码，不写回结果
- 用于条件跳转前的比较

### TEST 指令
- 格式：`TEST src2, src1` → 计算 `src1 & src2`（按位与）
- 只设置条件码（ZF、SF），不写回结果
- 常用于检查寄存器是否为 0 或符号位

### SET 指令（条件设置）

根据条件码将目标寄存器设为 0 或 1：

| 指令 | 条件 |
|------|------|
| `sete` | ZF = 1 |
| `seta` | CF = 0 且 ZF = 0 |
| `setg` | ZF = 0 且 SF = OF |

---

## 条件传送指令 (Conditional Move)

- `cmove`：相等时传送（ZF = 1）
- `cmovs`：负值时传送（SF = 1）
- `cmovc`：进位时传送（CF = 1）

> 条件传送在**流水线友好**方面优于条件分支，因为避免了分支预测错误惩罚。

---

## 循环 (Loops)

GCC 将三种循环结构编译为带**向后跳转**的汇编代码：

### for 循环
```c
int forLoop(int* x, int len) {
    int ret = 0;
    for (int i = 0; i < len; i++) {
        ret += x[i];
    }
    return ret;
}
```

### while 循环
```c
int whileLoop(int* x, int len) {
    int ret = 0;
    int i = 0;
    while (i < len) {
        ret += x[i];
        i++;
    }
    return ret;
}
```

### do-while 循环
```c
int doWhileLoop(int* x, int len) {
    int ret = 0;
    int i = 0;
    do {
        ret += x[i];
        i++;
    } while (i < len);
    return ret;
}
```

### 寄存器的使用
- 计数器 `i` 通常存放在某个寄存器中
- `len` 参数通常通过 `%rsi` 传递（第二个参数）
- `x` 数组指针通常通过 `%rdi` 传递（第一个参数）

### 三种循环的区别

| 循环类型 | 汇编特征 |
|----------|---------|
| **for** | 初始化 → 条件判断 → 循环体 → 更新 → 跳回判断 |
| **while** | 同 for，条件判断在开头 |
| **do-while** | 循环体至少执行一次，条件判断在末尾（无条件执行一次） |

> 实际上，`for` 和 `while` 的汇编代码通常难以区分，因为 GCC 对它们的优化策略基本相同。`do-while` 的区别在于条件判断的位置。

---

## Switch 语句与跳转表 (Jump Tables)

Switch 语句通常通过**跳转表**实现计算跳转：

```asm
jmp *.L4(,%rdi ,8)
```

这表示：取 `.L4` 数组中第 `%rdi` 个条目（每个条目 8 字节），跳转到该地址。

### 示例

```asm
switcher:
    cmpq $7, %rdi
    ja    .L2               # 超出范围 -> default
    jmp   *.L4(, %rdi, 8)   # 跳转表查找

.L7:
    xorq $15, %rsi
    movq %rsi, %rdx

.L3:
    leaq  112(%rdx), %rdi
    jmp   .L6

.L5:
    leaq (%rdx, %rsi), %rdi
    salq  $2, %rdi
    jmp   .L6

.L2:
    movq  %rsi, %rdi

.L6:
    movq  %rdi, (%rcx)
    ret

.section .rodata
.L4:
    .quad   .L3      # case 0
    .quad   .L2      # case 1 -> default
    .quad   .L5      # case 2
    .quad   .L2      # case 3 -> default
    .quad   .L6      # case 4
    .quad   .L7      # case 5
    .quad   .L2      # case 6 -> default
    .quad   .L5      # case 7
```

对应的 C 代码为：

```c
void switcher(long a, long b, long c, long *dest) {
    long val;
    switch (a) {
    case 5:          // .L7
        c = b ^ 15;
        /* fall through */
    case 0:          // .L3
        val = c + 112;
        break;
    case 2:          // .L5
    case 7:          // .L5
        val = (c + b) << 2;
        break;
    case 4:          // .L6
        val = a;
        break;
    default:         // .L2
        val = b;
    }
    *dest = val;
}
```

### 跳转表关键点

- 跳转表存放在只读数据段 `.section .rodata`
- 使用 `ja` 检查索引是否超出范围（无符号比较）
- 跳转表占用的空间是 `O(n)`，但执行时间是 `O(1)`
- 适合**密集**的 case 值（范围小但 case 多）
- 对于**稀疏** case，编译器可能优化为**条件分支树**

---

## 总结

| 主题 | 关键点 |
|------|--------|
| 条件码 | ZF/SF/CF/OF，由算术指令隐式设置 |
| 跳转指令 | 15 条，分无符号和有符号比较两类 |
| CMP/TEST | 只设置条件码，不写结果 |
| 条件传送 | 避免分支预测失败，流水线友好 |
| 循环 | do-while 最基础，for/while 编译结果相似 |
| Switch | 跳转表实现 O(1) 分支，密集 case 更有效 |
