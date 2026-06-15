---
title: 机器级编程 III：过程调用
date: 2026-06-15
tags: [CSAPP, 汇编, 过程调用, 栈, 递归, 调用约定]
---

# Machine-Level Programming III: Procedures

> CMU 15-213: Introduction to Computer Systems

---

## 过程调用的机理

过程调用涉及三个核心机制：

1. **传递控制**：跳转到被调用函数，返回到调用点的下一条语句
2. **传递数据**：参数传递和返回值传递
3. **内存管理**：进入函数时分配，退出函数时释放

```
P(...) {
    ...
    y = Q(x);
    print(y);
    ...
}

int Q(int i) {
    int t = 3*i;
    int v[10];
    ...
    return v[t];
}
```

---

## 栈结构

### x86-64 栈

- 栈地址**往低地址增长**
- `%rsp` 寄存器中存放当前栈的最低地址（栈顶元素的地址）
- 支持 `pushq`（入栈）和 `popq`（出栈）操作

```
高地址            ┌──────────────┐ ← 栈底
                  │              │
                  │     ...      │
                  │              │
                  ├──────────────┤
                  │  栈帧数据    │
                  ├──────────────┤
低地址            │              │ ← %rsp (栈顶)
                  └──────────────┘
```

### pushq 入栈

```asm
pushq Rsrc
# 等价于：
subq $8, %rsp
movq Rsrc, (%rsp)
```

### popq 出栈

```asm
popq Rdest
# 等价于：
movq (%rsp), Rdest
addq $8, %rsp
```

---

## 调用约定

### 过程控制流

```asm
call Label    # 将返回地址入栈，然后跳转到 Label
              # 等价于：push %rip; jmp Label

ret           # 从栈中弹出返回地址，跳转到该地址
              # 等价于：pop %rip
```

### 控制流示例

```
地址     指令
400540: push %rbx
400541: mov %rdx, %rbx
400544: callq 400550 <mult2>   # 将 0x400549 入栈
400549: mov %rax, (%rbx)       # ← 返回地址
...

400550: mov %rdi, %rax
400553: imul %rsi, %rax
400557: retq                   # 弹出 0x400549，跳回
```

### 数据流：参数传递

x86-64 Linux 调用约定：

| 参数位置 | 寄存器 |
|:--------:|:------:|
| 1st | `%rdi` |
| 2nd | `%rsi` |
| 3rd | `%rdx` |
| 4th | `%rcx` |
| 5th | `%r8` |
| 6th | `%r9` |
| 7th+ | 栈上 |
| 返回值 | `%rax` |

> 优先使用寄存器传递参数，仅在需要时才使用栈空间传递参数。

---

## 栈帧管理

### 基于栈的编程语言

支持递归的语言（C/C++、Pascal、Java）必须是"可重入"（Reentrant）的：
- 同一个代码块可以并发或同时运行多个实例
- 需要为每个实例保存状态信息（参数、局部变量、返回指针）

### 栈帧结构

被调用函数（callee）一定在调用函数（caller）之前返回（LIFO）。

```
Caller Frame        ┌──────────────┐
                    │  Arguments 7+ │ (如果有)
                    ├──────────────┤
                    │  返回地址    │ ← push by call
Callee Frame        ├──────────────┤
                    │ Saved %rbp   │ (可选)
                    ├──────────────┤
                    │ 局部变量     │
                    ├──────────────┤
                    │ Saved Regs   │
                    └──────────────┘ ← %rsp
```

### 示例：incr 调用

```c
long incr(long *p, long val) {
    long x = *p;
    long y = x + val;
    *p = y;
    return x;
}

long call_incr() {
    long v1 = 15213;
    long v2 = incr(&v1, 3000);
    return v1 + v2;
}
```

```asm
call_incr:
    subq    $16, %rsp           # 分配栈空间
    movq    $15213, 8(%rsp)     # v1 = 15213
    movl    $3000, %esi         # 2nd arg = 3000
    leaq    8(%rsp), %rdi       # 1st arg = &v1
    call    incr
    addq    8(%rsp), %rax       # v1 + v2
    addq    $16, %rsp           # 释放栈空间
    ret
```

---

## 寄存器保护约定

### 问题

当函数 `yoo` 调用 `who` 时，`who` 可能覆盖 `yoo` 正在使用的寄存器。

### 策略

| 类型 | 寄存器 | 责任方 |
|:----:|:------:|:------:|
| Caller Saved | `%rax`, `%rdi`-`%r9`, `%r10`, `%r11` | 调用者在调用前后进行保存和恢复 |
| Callee Saved | `%rbx`, `%r12`-`%r15`, `%rbp`, `%rsp` | 被调用者在入口保存、返回前恢复 |

### Caller 保护示例

```c
long call_incr2(long x) {
    long v1 = 15213;
    long v2 = incr(&v1, 3000);
    return x + v2;
}
```

```asm
call_incr2:
    pushq   %rbx                # 保存 callee-saved 寄存器
    subq    $16, %rsp
    movq    %rdi, %rbx          # 保存 x 到 %rbx（callee-saved）
    movq    $15213, 8(%rsp)
    movl    $3000, %esi
    leaq    8(%rsp), %rdi
    call    incr
    addq    %rbx, %rax          # x + v2
    addq    $16, %rsp
    popq    %rbx                # 恢复 callee-saved 寄存器
    ret
```

### 何时使用栈帧

- 寄存器不够时
- 参数数目大于 6 个
- 局部变量过多
- 需要取地址（`&x`）的局部变量
- 数组或结构类型变量

---

## 递归

### 递归函数示例

```c
long pcount_r(unsigned long x) {
    if (x == 0)
        return 0;
    else
        return (x & 1) + pcount_r(x >> 1);
}
```

```asm
pcount_r:
    movl    $0, %eax            # 写返回值
    testq   %rdi, %rdi
    je      .L6
    pushq   %rbx                # 保护寄存器
    movq    %rdi, %rbx          # %rbx = x (callee-saved)
    andl    $1, %ebx            # %ebx = x & 1
    shrq    %rdi                # x >>= 1
    call    pcount_r            # 递归调用
    addq    %rbx, %rax          # (x&1) + 返回值
    popq    %rbx                # 恢复寄存器
.L6:
    ret
```

### 递归支持的关键

- 每个函数调用实例都有一个私有的栈帧
- 栈帧保存寄存器、局部变量、返回地址
- 寄存器保护约定避免数据覆盖
- 栈的 LIFO 特性完美匹配函数调用的嵌套模式

---

## 总结

### 三要素

1. **传递控制**：`call` + `ret`
2. **传递数据**：寄存器参数（前 6 个）+ 栈参数（7+）+ `%rax` 返回值
3. **管理内存**：栈帧的入栈和出栈

### 寄存器分工

| 寄存器 | 用途 | 保护责任 |
|--------|------|:--------:|
| `%rax` | 返回值 | Caller |
| `%rdi`, `%rsi`, `%rdx`, `%rcx`, `%r8`, `%r9` | 参数 | Caller |
| `%r10`, `%r11` | 临时数据 | Caller |
| `%rbx`, `%r12`-`%r15` | 临时/局部数据 | Callee |
| `%rbp` | 帧指针（可选） | Callee |
| `%rsp` | 栈指针 | 特殊 |

### 关键特点

- 不需要为递归做特殊处理
- 支持互递归：P 调用 Q，Q 调用 P
- 指针是内存地址，包括局部变量的地址（栈内存段）和全局变量的地址（全局内存段）
