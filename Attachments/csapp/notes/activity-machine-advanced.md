---
title: CSAPP 活动笔记 - 机器级高级话题：安全性
date: 2024-09
tags: #CSAPP #汇编 #缓冲区溢出 #ROP #安全
---

# 机器级高级话题：安全性 (Machine-Level Security)

## 缓冲区溢出 (Buffer Overflow)

### gets() 的不安全性

```c
char *gets(char *dest) {
    int c = getchar();
    char *p = dest;
    while (c != EOF && c != '\n') {
        *p++ = c;
        c = getchar();
    }
    *p = '\0';
    return dest;
}
```

**核心问题**：
- `gets()` **不知道**目标缓冲区 `dest` 有多大
- 它只有在遇到 `EOF` 或 `\n` 时才停止读取
- 如果输入超过缓冲区大小，就会发生**缓冲区溢出**，覆盖栈上的其他数据

> `gets()` 已被 C11 标准废弃，永远不要在生产代码中使用。

### 栈溢出攻击原理

```asm
echo:
    sub   $0x18, %rsp       # 分配 24 字节栈空间
    mov   %rsp, %rdi        # 缓冲区的起始地址
    call  gets              # 读取输入到栈缓冲区
    mov   %rsp, %rdi
    call  puts              # 打印字符串
    add   $0x18, %rsp
    ret
```

#### 栈布局

```
        高地址
        +------------------+  0x414140 (%rsp 初始值)
+0x28   | 调用者的栈帧      |
+0x20   | 返回地址          |  ← 溢出后可被覆盖
+0x18   |（填充）           |
+0x10   |                  |
+0x08   | 缓冲区 (24 字节)  |
+0x00   +------------------+  0x414140 (%rsp)
        低地址
```

#### 攻击输入示例

```
123456781234567812345678@AA
```

- 前 24 个字符填满缓冲区
- `@`（ASCII 0x40）和 `A`（ASCII 0x41）覆盖返回地址 → 返回地址变为 `0x414140`（指向栈）

> **攻击流程**：输入超过缓冲区长度 → 覆盖返回地址 → 函数返回时跳转到攻击者指定的地址执行恶意代码。

---

## 防御机制 (Defenses)

### 1. 使用安全函数 fgets()

```c
char *fgets(char *s, int size, FILE *stream);
```

- `size` 参数指定最大读取长度
- 即使输入过长也不会溢出缓冲区
- 始终优先于 `gets()` 使用

### 2. 栈随机化 (ASLR)

- 操作系统将栈基址随机化
- 每次运行栈的起始地址不同
- 攻击者难以预测返回地址
- 仅靠随机化不能完全防御（暴力猜测仍可能成功），但大大降低了成功概率

### 3. 栈保护者 / Canary (Stack Canary)

编译器在栈上插入一个"金丝雀"值，用于检测栈是否被破坏：

```asm
echo_with_canary:
    sub   $0x18, %rsp
    mov   %fs:0x28, %rax    # * 从线程本地存储加载 canary
    mov   %rax, 0x8(%rsp)   # * 将 canary 存入栈上
    xor   %eax, %eax        # 清零 %eax
    mov   %rsp, %rdi
    call  gets
    mov   %rsp, %rdi
    call  puts
    mov   0x8(%rsp), %rax   # * 从栈上取出 canary
    xor   %fs:0x28, %rax    # * 与原始 canary 比较
    jz    .Lok              # * 如果相等，跳转到正常返回
    call  __stack_chk_fail  # * 不相等 → 调用栈检查失败处理
.Lok:
    add   $0x18, %rsp
    ret
```

**伪代码逻辑**：
```
canary = *(%fs:0x28)                 # 获取 canary 值
stack[canary_offset] = canary        # 存入栈
gets(buffer)                         # 有风险的调用
if stack[canary_offset] != canary:   # 检查 canary
    __stack_chk_fail()               # 栈被破坏 → 终止程序
```

| Canary 特性 | 说明 |
|-------------|------|
| **存储位置** | 线程本地存储，`%fs:0x28` |
| **检查时机** | 函数返回前 |
| **溢出检测** | 缓冲区溢出会覆盖 canary，导致检查失败 |
| **程序行为** | canary 被破坏 → 程序立即终止 |

**Canary 的成本**：
- 内存：多占用栈上 8 字节
- 时间：每次调用增加 3 条额外指令（加载、存储、检查）
- 局限性：无法防御非连续内存访问（如通过指针间接覆盖返回地址）

---

## 面向返回编程 (ROP, Return-Oriented Programming)

当栈设置了**不可执行位**（NX-bit / W^X）后，攻击者无法直接在栈上执行代码。ROP 应运而生。

### ROP 的核心思想

1. 攻击者不再注入新代码
2. 而是从已有的可执行代码（如 libc）中寻找以 `ret` 结尾的指令片段 —— **gadget**
3. 将多个 gadget 的地址串联在栈上
4. 每次 `ret` 执行一个 gadget，链式完成攻击

### ret 指令的字节值

`ret` 指令的机器码是 **`0xc3`**。

### Gadget 示例

```asm
<setval>:
   4004d0: c7 07 d4 48 89 c7    movl $0xc78948d4, (%rdi)
   4004d6: c3                    ret
```

寻找 `48 89 c7`（即 `mov %rax, %rdi`）这个字节序列：

- 该序列出现在地址 `0x4004d2`（从 `48 89 c7` 开始，跨越指令边界）
- 这个 gadget 的功能是：`mov %rax, %rdi; ret`
- 将栈设为该地址 → 执行 `ret` 时跳转到 gadget

### ROP 攻击流程

```
栈布局（从低到高）:
+------------------------+
| gadget1_addr            |  ← ret 跳转到 gadget1 执行
+------------------------+
| gadget2_addr            |  ← gadget1 的 ret 跳转到 gadget2
+------------------------+
| gadget3_addr            |  ← gadget2 的 ret 跳转到 gadget3
+------------------------+
```

### ROP 的关键要求

- 每个 gadget 必须以 `ret`（`0xc3`）结尾
- 因为 `ret` 指令会弹出下一条地址，形成连续的 gadget 链
- 攻击者需要足够多的 gadget 来完成任意计算（图灵完备）

---

## 一个完整的攻击示例

1. **漏洞**：程序使用 `gets()` 读取用户输入到栈缓冲区
2. **溢出**：输入超过缓冲区大小的内容覆盖返回地址
3. **注入代码**：
   ```asm
   mov  $0xdecafbad, %eax    # 机器码：b8 ad fb ca de
   ret                        # 机器码：c3
   ```
4. **放置**：代码字节放在缓冲区中，返回地址指向该缓冲区
5. **触发**：函数 `ret` 时跳转到攻击代码

```
输入字符串布局:
[填充缓冲区 24 字节] [指向缓冲区的地址] [shellcode]
```

---

## 总结

| 攻击/防御 | 原理 | 应对措施 |
|-----------|------|---------|
| **缓冲区溢出** | gets() 不限制输入长度，覆盖栈上返回地址 | 使用 fgets() |
| **栈上执行代码** | 返回地址指向栈上的注入代码 | NX-bit（栈不可执行） |
| **ASLR** | 栈地址随机化，使攻击者难以猜测 | 暴力枚举仍可能成功 |
| **Stack Canary** | 在返回地址前插入检查值，溢出检测 | __stack_chk_fail 终止程序 |
| **ROP** | 利用现有代码中的 gadget 链绕过 NX-bit | 控制流完整性 (CFI) |

> **安全原则**：永远不要假设用户输入是安全的。使用边界检查的函数，启用编译器和操作系统提供的所有防御机制。
