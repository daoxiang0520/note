---
title: CH5-3 其他并行优化及例子
date: 2026-06-15
tags:
  - CSAPP
  - SIMD
  - 线程级并行
  - 分支预测
  - 性能优化
---

# CH5-3 其他并行优化及例子

> CMU 15-213: Introduction to Computer Systems, 10th Lecture, Oct. 1, 2015

## 概述

本章涵盖指令内并行（SIMD）、线程级并行、分支预测优化以及性能瓶颈的识别方法。

## 指令内并行优化（SIMD）

### AVX2 向量指令

YMM 寄存器：16 个寄存器，每个 32 字节

- 32 个单字节整数
- 16 个 16 位整数
- 8 个 32 位整数
- 8 个单精度浮点数
- 4 个双精度浮点数

### SIMD 操作示例

```c
void combine_avx2(vec_ptr v, double *dest) {
    long i;
    long size = v->len;
    double *data = v->data;
    __m256d acc = _mm256_set1_pd(1.0);
    
    for (i = 0; i + 3 < size; i += 4) {
        __m256d vals = _mm256_loadu_pd(data + i);
        acc = _mm256_mul_pd(acc, vals);
    }
    
    double tmp = 1.0;
    double res[4];
    _mm256_storeu_pd(res, acc);
    for (int k = 0; k < 4; k++) tmp *= res[k];
    
    for (; i < size; i++) tmp *= data[i];
    *dest = tmp;
}
```

### 向量化性能

| Method | Int Add | Int Mult | Double Add | Double Mult |
|--------|---------|----------|------------|-------------|
| Scalar Best | 0.54 | 1.01 | 1.01 | 0.52 |
| Vector Best | 0.06 | 0.24 | 0.25 | 0.16 |
| Vec Throughput Bound | 0.06 | 0.12 | 0.25 | 0.12 |

### 指令内并行的原理

**定制指令**：将高频的指令序列融合成单一的硬件指令。

- 融合乘加指令（FMA）：`a*b+c` 在一条指令中完成
- 向量指令：同时对多组数据执行相同操作

## 线程级并行优化

### 原理

- **超线程（SMT）**：让一个物理核心中的不同执行单元，去并行处理来自两个或多个不同线程的指令
- **多核并行**：硬件上提供多个 CPU 核心，每个核心可以运行不同的线程
- **GPU 并行**：基于 CUDA、OpenCL 的多线程编程

单线程内部指令之间的依赖比较密集，限制了调度机会。不同线程（甚至进程）的指令之间高度独立，大幅增加了调度机会。

### 三种并行能力的结合

指令级并行、指令内并行、多线程并行可以结合优化。

## 分支预测优化

### 挑战

指令控制单元必须领先于执行单元，但当遇到条件分支时，无法确定接下来需要取出哪条指令，导致执行单元没有足够的指令来并行执行。

### 分支预测

猜测分支的走向，开始执行预测位置的指令，但不修改寄存器或内存数据。

### 分支预测错误

在现代处理器上，分支预测错误需要多个时钟周期来恢复，可能成为主要的性能限制因素。

### 优化分支的代码

1. **书写适合分支预测的代码**：对于可预测数据，CPE 约为 2.5-3.5；对于随机数据，CPE 约为 13.5
2. **书写适合条件传送的代码**：CPE 约为 4.0，无论数据是随机的还是可预测的
3. **利用查找表消除分支**
4. **利用算术表达式消除分支**

## 性能瓶颈的识别

### 引述

> "We should forget about small efficiencies, say about 97% of the time: premature optimization is the root of all evil. Yet we should not pass up our opportunities in that critical 3%." -- Donald Knuth / Tony Hoare

### Perf 工具分析流程

**第 1 步：宏观计数，快速定位问题类型**

```bash
perf stat -e cycles,instructions,cache-misses,branch-misses ./program
```

例如，发现 cache-misses 率高达 15% -> 疑似内存访问问题。

**第 2 步：采样分析，定位具体热点**

```bash
perf record -e cache-misses -c 1000 -g ./program
perf report --sort comm,symbol
```

**第 3 步：代码级优化定位**

```bash
perf annotate -s hot_function
```

**第 4 步：基于插桩的进一步分析**

手动插桩或自动插桩（如 strace）。

### 缓存局部性优化

除了计算优化，还有内存系统优化：

- 层次的内存结构
- 时间局部性、空间局部性
- 性能依赖于访问模式（步长）

```c
// 按列访问（步长 N）：81.8ms
void copyji(int src[2048][2048], int dst[2048][2048]) {
    int i, j;
    for (j = 0; j < 2048; j++)
        for (i = 0; i < 2048; i++)
            dst[i][j] = src[i][j];
}

// 按行访问（步长 1）：4.3ms
void copyij(int src[2048][2048], int dst[2048][2048]) {
    int i, j;
    for (i = 0; i < 2048; i++)
        for (j = 0; j < 2048; j++)
            dst[i][j] = src[i][j];
}
```

## 实现高性能的要点

1. 选择合适的编译器及编译选项
2. 不要做愚蠢的事情，小心算法中的性能缺陷
3. 利用编译器优化能力
4. 小心优化的拦路虎：函数调用 & 内存引用（需要人工编程优化）
5. 关注内层循环（承担大部分计算任务）
6. 针对机器调优：利用指令级并行、向量指令、多线程能力
7. 避免不可预测的分支
8. 编写缓存友好的代码

## 结语：软件优化的扩展

**CSAPP-**: 软硬件协同工作原理（C 代码与汇编映射、数据表示、代码实现对 CPU 性能的影响）

**CSAPP**: 更全面的计算机系统核心概念（存储层次与局部性、动态内存、多线程并发编程）

**CSAPP+**: 全面的程序优化原理（CPU、内存特性）

**CSAPP++**: 自动化的程序分析与代码优化技术（编译原理、软件分析）

除了性能，软件质量还包括：软件工程、软件包体积优化、功耗优化、安全增强等。
