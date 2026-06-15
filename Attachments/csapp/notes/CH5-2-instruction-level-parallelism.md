---
title: CH5-2 指令级并行优化
date: 2026-06-15
tags:
  - CSAPP
  - 指令级并行
  - 性能优化
  - 循环展开
---

# CH5-2 指令级并行优化

> CMU 15-213: Introduction to Computer Systems, 10th Lecture, Oct. 1, 2015

## 概述

现代处理器具备指令级并行能力，但并行能力受到数据依赖的限制。通过理解超标量和流水线技术，可以显著提升程序性能。

## 程序性能的测量：CPE

**CPE（Cycles Per Element）** 是一种用于表示向量或列表运算性能的指标。

```
T = CPE * n + Overhead
```

- 运算时间（时钟数）是元素数目的线性拟合
- CPE 是斜率

## 案例：向量运算优化

### 初始版本

```c
void combine1(vec_ptr v, data_t *dest) {
    long int i;
    *dest = IDENT;
    for (i = 0; i < vec_length(v); i++) {
        data_t val;
        get_vec_element(v, i, &val);
        *dest = *dest OP val;
    }
}
```

| Method | Int Add | Int Mult | Double Add | Double Mult |
|--------|---------|----------|------------|-------------|
| Combine1 (未优化) | 22.68 | 20.02 | 19.98 | 20.18 |
| Combine1 (-O1) | 10.12 | 10.12 | 10.17 | 11.14 |

### 人工优化 1：消除函数调用和内存别名

```c
void combine4(vec_ptr v, data_t *dest) {
    long i;
    long length = vec_length(v);
    data_t *d = get_vec_start(v);
    data_t t = IDENT;
    for (i = 0; i < length; i++)
        t = t OP d[i];
    *dest = t;
}
```

| Method | Int Add | Int Mult | Double Add | Double Mult |
|--------|---------|----------|------------|-------------|
| Combine4 | 1.27 | 3.01 | 3.01 | 5.01 |

## 现代 CPU 设计：超标量和流水线

### 超标量处理器

能够在单个时钟发射并执行多条指令，从顺序指令流中获取多条指令，并可以动态调度指令的执行顺序。

### 流水线处理器

将计算划分成多个阶段，每个阶段完成子计算后传递给下个阶段，然后立即开始新的子计算。

### Haswell CPU 功能单元

总共 8 个功能单元，可以并行执行多条指令：

- 2 个 load（含地址计算）
- 1 个 store（含地址计算）
- 4 个 integer
- 2 个 FP multiply
- 1 个 FP add
- 1 个 FP divide

| 指令 | 延迟周期 | 周期数/并行发射 |
|------|----------|----------------|
| Load/Store | 4 | 1 |
| Integer Multiply | 3 | 1 |
| Integer/Long Divide | 3-30 | 3-30 |
| Single/Double FP Multiply | 5 | 1 |
| Single/Double FP Add | 3 | 1 |

### 延迟上限 vs 吞吐量上限

| | Int Add | Int Mult | Double Add | Double Mult |
|--|---------|----------|------------|-------------|
| Combine4 CPE | 1.27 | 3.01 | 3.01 | 5.01 |
| 延迟上限 | 1.00 | 3.00 | 3.00 | 5.00 |
| 吞吐量上限 | 0.50 | 1.00 | 1.00 | 0.50 |

## 循环展开优化

### 普通循环展开 (2x1)

```c
void unroll2a_combine(vec_ptr v, data_t *dest) {
    long length = vec_length(v);
    long limit = length-1;
    data_t *d = get_vec_start(v);
    data_t x = IDENT;
    long i;
    for (i = 0; i < limit; i += 2) {
        x = (x OP d[i]) OP d[i+1];
    }
    for (; i < length; i++) {
        x = x OP d[i];
    }
    *dest = x;
}
```

**效果**：提升了整数加法的性能（达到延迟上限），但其他运算没有提升。原因是仍然存在串行依赖。

### 右结合的循环展开 (2x1a)

```c
void unroll2aa_combine(vec_ptr v, data_t *dest) {
    long length = vec_length(v);
    long limit = length-1;
    data_t *d = get_vec_start(v);
    data_t x = IDENT;
    long i;
    for (i = 0; i < limit; i += 2) {
        x = x OP (d[i] OP d[i+1]);
    }
    for (; i < length; i++) {
        x = x OP d[i];
    }
    *dest = x;
}
```

**效果**：几乎 2 倍加速比（Int *, FP +, FP *），原因是打破了串行依赖。

### 独立累计器的循环展开 (2x2)

```c
void unroll2a_combine(vec_ptr v, data_t *dest) {
    long length = vec_length(v);
    long limit = length-1;
    data_t *d = get_vec_start(v);
    data_t x0 = IDENT;
    data_t x1 = IDENT;
    long i;
    for (i = 0; i < limit; i += 2) {
        x0 = x0 OP d[i];
        x1 = x1 OP d[i+1];
    }
    for (; i < length; i++) {
        x0 = x0 OP d[i];
    }
    *dest = x0 OP x1;
}
```

### 综合性能

| Method | Int Add | Int Mult | Double Add | Double Mult |
|--------|---------|----------|------------|-------------|
| Combine1 (未优化) | 22.68 | 20.02 | 19.98 | 20.18 |
| Combine1 (-O1) | 10.12 | 10.12 | 10.17 | 11.14 |
| Best | 0.54 | 1.01 | 1.01 | 0.52 |
| 延迟上限 | 1.00 | 3.00 | 3.00 | 5.00 |
| 吞吐量上限 | 0.50 | 1.00 | 1.00 | 0.50 |

## 指令级并行优化的原理

### 关键概念

- **指令级并行原理**：超标量 CPU（n 发射）每个时钟并行发射 n 条指令；流水线 CPU（k 级流水）每个时钟并行运行 k 条指令
- **数据依赖**：阻塞指令并行，导致循环寄存器的依赖
- **控制依赖**：后续指令需要等待控制指令的结果才能运行

### 指令调度

如果 `inst[i+1]` 对 `inst[i]` 有依赖，则先调度无依赖的 `inst[i+k]` 和 `inst[i]` 并发执行。

约束条件：
- 硬件计算资源总量
- 每条指令在不同流水阶段的计算资源消耗
- 指令间的依赖关系

### 循环展开

通过循环展开，绕过分支控制指令，提供更多可供调度的指令机会。循环是热点代码，是指令调度的重点。

**挑战**：
- 循环体的指令变长（消耗内存资源）
- 需要使用更多的寄存器资源
- 由于依赖关系的约束，调度机会仍然有限

### 限制因素：寄存器溢出

当并行过多时，由于寄存器不够，会使得一些计算必须访问内存，从而使得性能反而下降。

```c
// 10x10 循环展开：使用寄存器
acc0 *= data[i]
vmulsd (%rdx), %xmm0, %xmm0

// 20x20 循环展开：寄存器溢出，需要访存
vmovsd 40(%rsp), %xmm0
vmulsd (%rdx), %xmm0, %xmm0
vmovsd %xmm0, 40(%rsp)
```
