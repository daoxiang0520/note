---
title: 机器级编程 II：控制
date: 2026-06-15
tags: [CSAPP, 汇编, 控制流, 条件码, 循环, switch]
---

# Machine-Level Programming II: Control

> CMU 15-213: Introduction to Computer Systems, 6th Lecture, Sep. 17, 2015

---

## 处理器状态

### x86-64 可见状态

- **临时数据**：`%rax`, `%rbx`, `%rcx`, `%rdx`, `%rsi`, `%rdi`, `%r8`-`%r15`
- **运行时栈地址**：`%rsp`
- **当前代码控制点地址**：`%rip`
- **条件码**：`CF`, `ZF`, `SF`, `OF`

```
 Registers      Condition Codes
┌────────┐     ┌──────────────┐
│ %rax   │     │ CF  ZF  SF  OF│
│ %rbx   │     └──────────────┘
│ %rcx   │
│ ...    │     Memory
│ %rsp   │     ┌──────────────┐
│ %rip   │     │ Code & Data  │
└────────┘     └──────────────┘
```

---

## 条件码

### 条件码（Condition Codes）

单个 bit 的寄存器：

| 标志 | 含义 |
|------|------|
| CF | Carry Flag（用于无符号运算） |
| ZF | Zero Flag |
| SF | Sign Flag（用于有符号运算） |
| OF | Overflow Flag（用于有符号运算） |

### 隐式设置

几乎所有的算术运算都会隐式设置条件码（**lea 指令除外**）。

示例：`addq Src, Dest` ⇔ `t = a + b`

- `CF`：最高位产生进位（无符号溢出）
- `ZF`：`t == 0`
- `SF`：`t < 0`（作为有符号数）
- `OF`：补码溢出 `(a>0 && b>0 && t<0) || (a<0 && b<0 && t>=0)`

### 显式设置：compare 指令

`cmpq Src2, Src1` ⇔ 计算 `a - b` 但不设置目标

- `CF`：最高位进位（用于无符号比较）
- `ZF`：`a == b`
- `SF`：`(a - b) < 0`
- `OF`：补码溢出

### 显式设置：test 指令

`testq Src2, Src1` ⇔ 计算 `a & b` 但不修改目标

- `ZF`：`a & b == 0`
- `SF`：`a & b < 0`

常用于检测某个 bit 是否为 0 或 1。

### 条件码的读取：SetX 指令

根据条件码的组合，设置寄存器的低字节为 0 或 1，不修改其余 7 个字节。

| 指令 | 条件 | 描述 |
|------|------|------|
| `sete` | ZF | 相等/零 |
| `setne` | ~ZF | 不相等/非零 |
| `sets` | SF | 负数 |
| `setns` | ~SF | 非负数 |
| `setg` | ~(SF^OF)&~ZF | 大于（有符号） |
| `setge` | ~(SF^OF) | 大于等于（有符号） |
| `setl` | SF^OF | 小于（有符号） |
| `setle` | (SF^OF)\|ZF | 小于等于（有符号） |
| `seta` | ~CF&~ZF | 高于（无符号） |
| `setb` | CF | 低于（无符号） |

示例：

```c
int gt(long x, long y) {
    return x > y;
}
```

```asm
    cmpq    %rsi, %rdi      # Compare x:y
    setg    %al             # Set when >
    movzbl  %al, %eax       # Zero rest of %rax
    ret
```

> 通常使用 `movzbl` 来完成其余高字节的清零工作。

---

## 条件分支

### Jumping 指令

| 指令 | 条件 | 描述 |
|------|------|------|
| `jmp` | 1 | 无条件跳转 |
| `je` | ZF | 相等 |
| `jne` | ~ZF | 不相等 |
| `js` | SF | 负数 |
| `jns` | ~SF | 非负数 |
| `jg` | ~(SF^OF)&~ZF | 大于（有符号） |
| `jge` | ~(SF^OF) | 大于等于（有符号） |
| `jl` | SF^OF | 小于（有符号） |
| `jle` | (SF^OF)\|ZF | 小于等于（有符号） |
| `ja` | ~CF&~ZF | 高于（无符号） |
| `jb` | CF | 低于（无符号） |

### if-else 的汇编实现

C 语言：

```c
long absdiff(long x, long y) {
    long result;
    if (x > y)
        result = x - y;
    else
        result = y - x;
    return result;
}
```

使用 goto 翻译：

```c
long absdiff_j(long x, long y) {
    long result;
    int ntest = x <= y;
    if (ntest) goto Else;
    result = x - y;
    goto Done;
Else:
    result = y - x;
Done:
    return result;
}
```

汇编代码：

```asm
absdiff:
    cmpq    %rsi, %rdi      # x:y
    jle     .L4
    movq    %rdi, %rax
    subq    %rsi, %rax
    ret
.L4:                        # x <= y
    movq    %rsi, %rax
    subq    %rdi, %rax
    ret
```

### 条件表达式的另一种实现：Conditional Move

条件移动指令（CMOV）支持如果条件成立则移动，否则跳过。

```asm
absdiff:
    movq    %rdi, %rax       # x
    subq    %rsi, %rax       # result = x - y
    movq    %rsi, %rdx
    subq    %rdi, %rdx       # eval = y - x
    cmpq    %rsi, %rdi       # x:y
    cmovle  %rdx, %rax       # if <=, result = eval
    ret
```

**优势**：避免分支，防止流水线停顿。
**不适用的场景**：
1. 两个分支计算量很大
2. 分支计算有安全风险（如 `p ? *p : 0`）
3. 分支计算有副作用（如 `x > 0 ? x*=7 : x+=3`）

---

## 循环

### Do-While 循环

```c
// C 代码
long pcount_do(unsigned long x) {
    long result = 0;
    do {
        result += x & 0x1;
        x >>= 1;
    } while (x);
    return result;
}
```

```asm
    movl    $0, %eax        # result = 0
.L2:                        # loop:
    movq    %rdi, %rdx
    andl    $1, %edx        # t = x & 0x1
    addq    %rdx, %rax      # result += t
    shrq    $1, %rdi        # x >>= 1
    jne     .L2             # if (x) goto loop
    ret
```

**通用翻译模式：**

```
do
    Body
while (Test);
```

转换为：

```
loop:
    Body
    if (Test) goto loop;
```

### While 循环（方法一：Jump-to-Middle）

```
    goto test;
loop:
    Body
test:
    if (Test) goto loop;
```

### While 循环（方法二：Do-While 转换）

```
    if (!Test) goto done;
loop:
    Body
    if (Test) goto loop;
done:
```

### For 循环

`for (Init; Test; Update) Body`

先转换为 while 循环：

```
Init;
while (Test) {
    Body
    Update;
}
```

再转换为 do-while 或 jump-to-middle 形式。

---

## Switch 语句

### 跳转表（Jump Table）

Switch 语句的一种优化实现是使用跳转表：

```
jmp *.L4(,%rdi,8)
```

- 从 case 值直接映射到代码块的地址
- 跳转表是只读数据段中的地址数组
- 每个表项是一个 8 字节地址

### 示例

```c
long switch_eg(long x, long y, long z) {
    long w = 1;
    switch (x) {
        case 1:  w = y * z; break;
        case 2:  w = y / z;  /* Fall Through */
        case 3:  w += z; break;
        case 5:
        case 6:  w -= z; break;
        default: w = 2;
    }
    return w;
}
```

汇编中的跳转表：

```asm
.section    .rodata
.align 8
.L4:
    .quad    .L8     # x = 0
    .quad    .L3     # x = 1
    .quad    .L5     # x = 2
    .quad    .L9     # x = 3
    .quad    .L8     # x = 4
    .quad    .L7     # x = 5
    .quad    .L7     # x = 6
```

### 优缺点

| 优点 | 缺点 |
|------|------|
| 运行时间与 case 数目无关（常量级） | 需要 case 值分布有规律 |
| 不影响流水线性能 | 稀疏的 case 值使用 if-else 更合适 |

**何时使用跳转表：**
- case 数目较多（LLVM 中 >= 4）
- case 值的分布比较集中

---

## 总结

| 控制结构 | 汇编实现方式 |
|----------|-------------|
| if-then-else | 条件跳转 / 条件移动 |
| do-while | 先执行 body，再判断跳回 |
| while | Jump-to-middle 或先转 do-while |
| for | 先转 while，再转底层形式 |
| switch | 跳转表（集中）或决策树（稀疏） |

### 关键概念

- **条件码**：Arithmetic、compare、test 指令设置，setX、JX、cmov 指令使用
- **流水线 CPU**：分支指令成本高，条件移动可避免
- **PC 相对寻址**：跳转目标 = PC + 编码偏移

### 条件码的优缺点

**优点（历史价值）：**
- 设计统一、简洁
- 节省编码空间
- 运算 + 判断结合紧密

**缺点（现代问题）：**
- 隐式全局状态（最大问题）
- 限制 OoO 和 ILP
- 难以重命名，产生假依赖
- ISA 无法进化

> RISC-V 架构中没有条件码，使用 `blt x1, x2, label` 形式。
