---
title: 虚拟存储器详解
date: 2026-06-15
tags: [CSAPP, 虚拟存储器, Core i7, 内存映射, 写时复制]
---

# 虚拟存储器详解

## 复习：虚拟寻址

每个进程有其自己的虚拟地址空间，页表把虚拟地址映射到物理地址，物理内存可以在进程间共享。

## 缺页故障

### 缺页故障发生了什么

- **什么是缺页故障**：访问的虚存字不在物理内存中
- **为什么会发生**：MMU 查询页表发现有效位为 0
- **OS 怎么知道的**：MMU 触发缺页异常

### 故障类型

| 故障类型 | 描述 |
|---------|------|
| 硬/主要故障 | "正常"缺页故障，地址合法 |
| 保护异常 | 地址合法但权限不够 |
| 分段故障/总线错误 | 地址不合法 |

## Intel Core i7 存储器系统

### Core i7 缓存层次结构

| 缓存 | 大小 | 路数 |
|------|------|------|
| L1 d-cache | 32 KB | 8-way |
| L1 i-cache | 32 KB | 8-way |
| L2 unified cache | 256 KB | 8-way |
| L3 unified cache | 8 MB | 16-way (所有核心共享) |
| L1 d-TLB | 64 项 | 4-way |
| L1 i-TLB | 128 项 | 4-way |
| L2 unified TLB | 512 项 | 4-way |

### Core i7 四级页表

地址转换使用 4 级页表：

```
VA: [VPN1(9)] [VPN2(9)] [VPN3(9)] [VPN4(9)] [VPO(12)]
PA: [PPN(40)] [PPO(12)]
```

- L4 PT (页表)：每项引用 512 GB 区域
- L3 PT (页中层目录)：每项引用 1 GB 区域
- L2 PT (页上层目录)：每项引用 2 MB 区域
- L1 PT (页全局目录)：每项引用 4 KB 区域

### Core i7 页表项重要字段

- **P**：子页表在（1）或不在（0）物理内存中
- **R/W**：只读或读写访问权限
- **U/S**：用户或超级用户模式访问权限
- **WT**：通写或回写缓存策略
- **A**：引用位
- **D**：修改位（由 MMU 在写时设置）
- **G**：全局页（不被驱逐出 TLB）
- **XD**：禁止/允许取指

### 加快 L1 访问的技巧

利用"虚拟 index，物理 tag"设计，在 MMU 地址转换的同时查找 cache 索引。

## 内存映射

在初始化 VM 区域的过程中，将 VM 区域与磁盘上的对象关联起来的过程称为**内存映射**。

关联的磁盘对象可以是：
1. **磁盘上的普通文件**：初始化页面时从对应文件中获取数据
2. **匿名文件**：第一次 fault 时分配全为 0 值的物理页 (demand-zero page)

### 共享对象

多个进程可以映射同一个共享物理对象，虚拟地址可以不同。

### 私有写时复制 (Copy-on-Write, COW)

两个进程映射一个 private copy-on-write 对象时：
1. 私有区域的 PTEs 标记为 read-only
2. 写入私有页的指令触发保护 fault
3. fault 处理程序创建新的 R/W 页
4. 将拷贝操作尽可能往后推迟

### 交换空间

物理内存上每一页都应在磁盘上有对应，以便支持驱逐行为：
- **文件页**：对应磁盘上原有文件
- **匿名页**：对应交换空间 (swap space)

### 重新理解 fork

fork 通过写时复制优化：
1. 只复制 `mm_struct`, `vm_area_struct` 和页表
2. 把所有内容标记为只读
3. 仅在写入时才复制 (Copy On Write, CoW)

### 重新理解 execv

1. 删除旧的用户虚拟地址数据 (`vm_area_struct` 和页表)
2. 建立新的私有 `vm_area_struct` 和页表
   - 文件页：`.data`, `.text`
   - 匿名页：`.bss`, stack, heap
3. 建立共享 vma（若需要链接共享库）
4. 修改 PC 寄存器指向 code 段的入口点

## User-Level Memory Mapping: mmap

```c
void *mmap(void *start, int len, int prot, int flags, int fd, int offset);
```

- `start`：首选地址，可为 0（由系统选择）
- `prot`：PROT_READ, PROT_WRITE, ...
- `flags`：MAP_ANON, MAP_PRIVATE, MAP_SHARED, ...

### 使用 mmap 复制文件

```c
void mmapcopy(int fd, int size) {
    char *bufp;
    bufp = Mmap(NULL, size, PROT_READ, MAP_PRIVATE, fd, 0);
    Write(1, bufp, size);
    return;
}
```

无需将数据复制到用户空间即可完成文件到标准输出的复制。

## 概念小测验总结

1. **单级页表为什么不实用**：48 位地址空间、4KB 页、8 字节 PTE，单级页表占 512 GB
2. **多级页表为什么更慢**：需要 k 次内存加载才能确定物理地址，且没有空间局部性
3. **TLB 是什么**：小型缓存，存储 VPN->PPN 映射，避免多级页表开销
4. **虚拟内存如何与缓存交互**：MMU 在 CPU 和 cache 之间，将 VA 映射成 PA，cache 只使用物理地址
