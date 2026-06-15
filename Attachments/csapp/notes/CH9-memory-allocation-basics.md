---
title: 动态内存分配：基础知识
date: 2026-06-15
tags: [CSAPP, 动态内存分配, malloc, free, 隐式空闲列表]
---

# 动态内存分配：基础知识

## 动态内存分配

程序员使用**动态内存分配器**（如 `malloc`）获取运行时虚拟内存，对仅在运行时才知道其大小的数据结构特别有用。

动态内存分配器管理被称为**堆**的进程 VM 区域。

## 堆可视化

堆位于进程地址空间中的用户栈和共享库之间，使用 `sbrk` 系统调用扩展或收缩。

```
共享库的内存映射区域
运行时堆（由 malloc 创建）
读/写段（.data, .bss）
只读段（.init, .text, .rodata）
```

## malloc 程序包

```c
#include <stdlib.h>

void *malloc(size_t size);
```

- 成功：返回指向内存块的指针，至少 size 字节，x86-64 上 16 字节对齐
- 失败：返回 NULL 并设置 errno

```c
void free(void *p);
```

- 把 p 指向的块返回到可用内存池
- p 必须来自之前调用的 malloc, calloc 或 realloc

其他函数：
- `Calloc`：初始化为零的 malloc
- `Realloc`：修改之前已分配块的大小
- `Sbrk`：扩展或收缩堆

## 约束条件

- 应用程序可以按任意顺序发出 `malloc` 和 `free` 请求
- `free` 请求必须对应 `malloc` 分配的内存块
- 必须立即响应 `malloc` 请求（不能重排序或缓存）
- 必须从空闲内存中分配
- 必须对齐块以满足所有对齐要求（64 位系统上 16 字节对齐）
- 只能操作和修改空闲内存
- **不能移动已被分配的块**（不允许压缩）

## 性能目标

### 吞吐量

每单位时间完成的请求数量。

### 碎片

碎片导致堆利用率很低：

- **内部碎片**：有效负载小于内存块大小（维护开销、对齐填充、策略决定）
- **外部碎片**：总堆内存足够，但没有单个空闲块足够大

## 实现问题

1. 仅根据一个指针，如何知道要释放多少内存？
2. 如何跟踪空闲块？
3. 如何分配比空闲块更小的结构？
4. 如何选择合适的内存块？
5. 如何重复使用已释放的内存块？

### 知道要释放多少空间

标准方法：把保存内存块长度的字放在块前面（头部字段）。

```
[头部: 内存块大小 | 有效负载 | 填充]
```

## 隐式空闲列表

### 内存块格式

利用对齐特性（低位始终为 0），将低位用作已分配/空闲标志：

```
[内存块大小 | a(=1:已分配, =0:空闲)] [有效负载] [填充]
```

### 数据结构

```c
typedef uint64_t word_t;
typedef struct block {
    word_t header;
    unsigned char payload[0];  // Zero length array (GCC扩展)
} block_t;
```

### 表头访问

```c
int get_alloc(word_t header) { return header & 0x1; }
size_t get_size(word_t header) { return header & ~0xfL; }
```

### 遍历列表

```c
static block_t *find_next(block_t *block) {
    return (block_t *)((unsigned char *)block + get_size(block));
}
```

### 查找空闲块

| 策略 | 描述 | 特点 |
|------|------|------|
| **首次适配** | 从头搜索，选第一个合适的 | 线性时间，开头易产生碎片 |
| **下一次适配** | 从上一次位置开始搜索 | 比首次适配快，碎片更多 |
| **最佳适配** | 选剩余字节最少的 | 碎片最少，通常更慢 |

### 分配空闲块：分割

```c
static void split_block(block_t *block, size_t asize) {
    size_t block_size = get_size(block);
    if ((block_size - asize) >= min_block_size) {
        write_header(block, asize, true);
        block_t *block_next = find_next(block);
        write_header(block_next, block_size - asize, false);
    }
}
```

### 释放内存块：合并

最简单的实现只需清除"已分配"标志，但可能导致"虚假碎片"。

**边界标记 (footer)**[Knuth73]：在空闲块底部复制内存块大小/已分配字，允许反向遍历。

```
[头部: size|a] [有效负载和填充] [尾部: size|a]
```

### 常数时间合并

利用边界标记，释放时可以检查前后块的空闲状态，进行四种场景的合并：

1. 前后都已分配：不合并
2. 前已分配、后空闲：与后合并
3. 前空闲、后已分配：与前合并
4. 前后都空闲：三者合并

### 堆结构

- 第一个表头前设置虚拟表尾（已分配），防止合并第一个块
- 最后一个表尾后设置虚拟表头（已分配），防止合并最后一个块

### 顶层 malloc 代码

```c
void *mm_malloc(size_t size) {
    size_t asize = round_up(size + dsize, dsize);
    block_t *block = find_fit(asize);
    if (block == NULL) return NULL;
    size_t block_size = get_size(block);
    write_header(block, block_size, true);
    write_footer(block, block_size, true);
    split_block(block, asize);
    return header_to_payload(block);
}
```

### 顶层 free 代码

```c
void mm_free(void *bp) {
    block_t *block = payload_to_header(bp);
    size_t size = get_size(block);
    write_header(block, size, false);
    write_footer(block, size, false);
    coalesce_block(block);
}
```

## 隐式列表总结

- **实现**：非常简单
- **分配成本**：最坏情况线性时间
- **释放成本**：最坏情况常数时间
- 分割和边界标记合并的概念通用于所有分配器
