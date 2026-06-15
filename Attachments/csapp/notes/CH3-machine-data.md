---
title: 机器级编程 IV：数据
date: 2026-06-15
tags: [CSAPP, 汇编, 数组, 结构体, 对齐, 浮点]
---

# Machine-Level Programming IV: Data

> CMU 15-213: Introduction to Computer Systems, 8th Lecture, Sep. 24, 2015

---

## 数组

### 一维数组

**原理：** `T A[L]` 在内存中分配连续的、长度为 `L * sizeof(T)` 字节的内存块。

```c
char string[12];    // 12 bytes
int val[5];         // 20 bytes (4*5)
double a[3];        // 24 bytes (8*3)
char *p[3];         // 24 bytes (8*3)
```

**数组的访问：** A 可以视为指向数组第一个元素的常量指针 `T*`。

```c
int val[5] = {1, 5, 2, 1, 3};
// val[4]    → int,  值为 3
// val       → int*, 值为 x
// val+1     → int*, 值为 x+4
// &val[2]   → int*, 值为 x+8
// *(val+1)  → int,  值为 5
```

**汇编访问（元素索引）：** `%rdi` 为基地址，`%rsi` 为索引，`z[digit]` 的地址为 `%rdi + 4*%rsi`。

```asm
movl (%rdi, %rsi, 4), %eax    # z[digit]
```

### 多维数组（嵌套数组）

**原理：** `T A[R][C]` — R 行 C 列，行优先（Row-Major）布局。

- 内存大小：`R * C * K` 字节（K = sizeof(T)）
- 元素 `A[i][j]` 地址：`A + (i * C + j) * K`

```
A[0][0] A[0][C-1] A[1][0] A[1][C-1] ... A[R-1][0] A[R-1][C-1]
<─── 第0行 ───><─── 第1行 ───>     ... <─── 第R-1行 ───>
```

**行向量访问：** `A[i]` 的起始地址 = `A + i * (C * K)`

```c
int *get_pgh_zip(int index) {
    return pgh[index];
}
```

```asm
leaq (%rdi, %rdi, 4), %rax     # 5 * index
leaq pgh(, %rax, 4), %rax      # pgh + (20 * index)
```

**元素访问：** `A[i][j]` 地址 = `A + 4*(C*i + j)`

```c
int get_pgh_digit(int index, int dig) {
    return pgh[index][dig];
}
```

```asm
leaq  (%rdi, %rdi, 4), %rax    # 5*index
addl  %rax, %rsi               # 5*index + dig
movl  pgh(, %rsi, 4), %eax     # M[pgh + 4*(5*index + dig)]
```

### 多级数组

```c
int *univ[UCOUNT] = {mit, cmu, ucb};
```

- univ 是一个包含 3 个元素的数组，每个元素是指针（8 字节）
- 每个指针指向一个 int 数组

**访问元素：** 需要访问 2 次内存

```asm
salq    $2, %rsi                # 4*digit
addq    univ(,%rdi,8), %rsi     # p = univ[index] + 4*digit
movl    (%rsi), %eax            # return *p
```

### 多维数组 vs 多级数组

| 属性 | 多维数组 `int pgh[4][5]` | 多级数组 `int *univ[3]` |
|------|:---:|:---:|
| 访问方式 | `Mem[pgh+20*i+4*j]` | `Mem[Mem[univ+8*i]+4*j]` |
| 内存访问次数 | 1 次 | 2 次 |
| 存储布局 | 连续 | 分开 |

### 定长 vs 变长数组

**定长数组（编译时已知维度）：**

```c
#define N 16
typedef int fix_matrix[N][N];

int fix_ele(fix_matrix a, size_t i, size_t j) {
    return a[i][j];
}
```

```asm
salq $6, %rsi            # 64*i
addq %rsi, %rdi          # a + 64*i
movl (%rdi, %rdx, 4), %eax  # M[a + 64*i + 4*j]
```

**变长数组（运行时维度）：**

```c
int var_ele(size_t n, int a[n][n], size_t i, size_t j) {
    return a[i][j];
}
```

```asm
imulq %rdx, %rdi         # n*i
leaq  (%rsi, %rdi, 4), %rax  # a + 4*n*i
movl  (%rax, %rcx, 4), %eax  # a + 4*n*i + 4*j
```

必须使用整数乘法指令。

---

## 结构体

### 结构分配

结构分配一块连续内存，大小足够装下所有成员变量，按声明顺序分配。

```c
struct rec {
    int a[4];       // offset: 0, size: 16
    size_t i;       // offset: 16, size: 8
    struct rec *next; // offset: 24, size: 8
};
// Total: 32 bytes
```

```
a[0..3]    i      next
│ 16B  │  8B  │  8B  │
0      16     24     32
```

### 结构成员访问

编译器确定结构中每个成员的偏移，机器代码通过内存地址访问。

```c
int *get_ap(struct rec *r, size_t idx) {
    return &(r->a[idx]);  // r + 4*idx
}
```

### 结构对齐

**对齐规则：**
1. 成员变量的类型占用 K 字节，则该成员变量的偏移必须是 K 的倍数
2. 结构体的对齐需求为 K_max（各成员对齐需求最大值）
3. 结构体的起始地址和总大小必须是 K_max 的倍数

**示例：**

```c
struct S1 {
    char c;       // offset: 0, size: 1
    // 3 bytes padding
    int i[2];     // offset: 4, size: 8
    double v;     // offset: 16, size: 8
};
// Total: 24 bytes, alignment: 8
```

```
c │ padding │  i[0..1]  │     v     │
1B    3B       8B          8B
0    4        8          16        24
```

**节省空间的技巧：大的成员变量优先分配**

```c
// 不好的顺序：24 bytes
struct S4 {
    char c;       // +1
    // +3 padding
    int i;        // +4
    char d;       // +1
    // +3 padding
};

// 好的顺序：12 bytes
struct S5 {
    int i;        // +4
    char c;       // +1
    char d;       // +1
    // +2 padding
};
```

### 结构体数组

每个元素都需要满足对齐需求：

```c
struct S2 {
    double v;     // 8 bytes
    int i[2];     // 8 bytes
    char c;       // 1 byte
    // 7 bytes padding
};
struct S2 a[10];  // 每个元素 24 字节
```

### x86-64 基本类型对齐需求

| 类型 | 对齐要求 |
|------|---------|
| `char` | 1 字节（无限制） |
| `short` | 2 字节（地址最低 1 位为 0） |
| `int`, `float` | 4 字节（地址最低 2 位为 00） |
| `double`, `long`, `char*` | 8 字节（地址最低 3 位为 000） |
| `long double` (Linux GCC) | 16 字节（地址最低 4 位为 0000） |

### 为什么需要对齐

- 内存访问以字长为基本单位（32 或 64 位）
- 不对齐的变量需要多次内存访问
- 编译器在内存块中插入填充字节来确保对齐

### 联合类型（Union）

由最大的成员变量决定联合体的大小，一次只能使用一个成员变量。

```c
union U1 {
    char c;
    int i[2];
    double v;
};
// Total: 8 bytes
```

```c
typedef union {
    float f;
    unsigned u;
} bit_float_t;

float bit2float(unsigned u) {
    bit_float_t arg;
    arg.u = u;
    return arg.f;  // 保留位串的值不变，重新解释
}
```

---

## 浮点运算

### 背景

| 指令集 | 说明 |
|--------|------|
| x87 | 过时的浮点指令集 |
| SSE (SSE2/SSE3/SSE4) | 流式 SIMD 扩展，向量指令 |
| AVX | 高级向量扩展，最新 x86-64 扩展 |

### XMM 寄存器（SSE3）

16 个 128 位寄存器，可以看作：

- 16 个单字节整数
- 8 个 16 位整数
- 4 个 32 位整数
- 4 个单精度浮点数
- 2 个双精度浮点数
- 1 个单精度或双精度浮点数

### 标量浮点运算

```c
float fadd(float x, float y) {
    return x + y;
}

double dadd(double x, double y) {
    return x + y;
}
```

```asm
# fadd: x in %xmm0, y in %xmm1
addss   %xmm1, %xmm0      # 单精度加
ret

# dadd: x in %xmm0, y in %xmm1
addsd   %xmm1, %xmm0      # 双精度加
ret
```

### 参数传递约定

- 整数、指针类型参数：通用寄存器（rdi, rsi, ...）
- 浮点类型参数：XMM 寄存器（%xmm0, %xmm1, ...）
- 返回值：%xmm0
- 所有 XMM 寄存器都是 caller 保护

### 浮点内存访问

```asm
# double dincr(double *p, double v)
# p in %rdi, v in %xmm0
movapd  %xmm0, %xmm1      # Copy v
movsd   (%rdi), %xmm0     # x = *p
addsd   %xmm0, %xmm1      # t = x + v
movsd   %xmm1, (%rdi)     # *p = t
ret
```

### 浮点比较

- `ucomiss` 和 `ucomisd` 设置条件码 `CF`、`ZF` 和 `PF`
- 浮点比较在遇到 `NaN` 时会表现奇怪

### 浮点常量

- 常量 0：`xorpd %xmm0, %xmm0`（将 XMM0 寄存器设置为 0）
- 其余常量：从内存读取

---

## 总结

| 数据类型 | 关键点 |
|---------|--------|
| 数组 | 连续内存，基于索引计算地址 |
| 结构体 | 连续内存，基于成员偏移访问，需要对齐 |
| 联合体 | 由最大成员决定大小，共享内存区域 |
| 浮点 | XMM 寄存器进行运算，有独立调用约定 |

### 理解指针和数组（重要辨析）

| 声明 | `A` 类型 | `*A` 类型 | `sizeof(A)` |
|------|----------|----------|:-----------:|
| `int A1[3]` | `int[3]` (数组) | `int` | 12 |
| `int *A2` | `int*` (指针) | `int` | 8 |
| `int A1[3][5]` | `int[3][5]` | `int[5]` | 60 |
| `int *A2[3]` | `int*[3]` | `int*` | 24 |
| `int (*A3)[3]` | `int(*)[3]` | `int[3]` | 8 |
