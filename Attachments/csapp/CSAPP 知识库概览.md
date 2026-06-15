---
title: CSAPP 知识库概览
date: 2026-06-15
tags:
  - CSAPP
  - 计算机系统
  - CMU-15213
  - 知识库
---

# CSAPP 知识库概览

> 本目录为 CMU 15-213/15-513 (Computer Systems: A Programmer's Perspective) 课程资源，
> 包含课程讲义、课堂活动练习以及工具使用教程。

## 目录结构

| 目录 | 说明 |
|:-----|:-----|
| `originals/` | 原始课件：22 个 PPTX + 5 个 PDF（共 29 个原始文件） |
| `notes/` | 已转换的 Markdown 笔记（共 **29 篇**），可直接在 Obsidian 中阅读 |
| 本文件 | 知识库概览与索引 |

> 原始文件已归档至 `originals/`，笔记均在 `notes/` 中，可直接点击下方链接跳转。

---

## 📖 课程笔记索引

### CH2: 信息的表示与处理

| 笔记 | 原始文件 | 核心内容 |
|:-----|:---------|:---------|
| [[Attachments/csapp/notes/CH2-bit-byte]] | `CH2-bit-byte.pptx` | 位与字节：二进制表示、补码、无符号数、位运算、整数运算 |
| [[Attachments/csapp/notes/CH2-float]] | `CH2-float.pptx` | 浮点数：IEEE 754 标准、单/双精度、舍入、浮点运算 |

### CH3: 机器级程序表示

| 笔记 | 原始文件 | 核心内容 |
|:-----|:---------|:---------|
| [[Attachments/csapp/notes/CH3-machine-basics]] | `CH3-machine-basics.pptx` | x86-64 基础：寄存器、指令格式、MOV/LEA/算术指令、寻址模式 |
| [[Attachments/csapp/notes/CH3-machine-control]] | `CH3-machine-control.pptx` | 控制流：条件码 (ZF/SF/CF/OF)、CMP/TEST、JMP 指令、条件传送、循环、switch |
| [[Attachments/csapp/notes/CH3-machine-data]] | `CH3-machine-data-student.pptx` | 数据布局：数组、struct 对齐与填充、union |
| [[Attachments/csapp/notes/CH3-machine-procedures]] | `CH3-machine-procedures.pptx` | 过程调用：运行时栈、call/ret、寄存器约定、递归 |
| [[Attachments/csapp/notes/CH3-buffer-overflow]] | `CH3-机器级表示的高级主题-缓冲区溢出攻击.pptx` | 缓冲区溢出、注入攻击、canary、ROP |

### CH5: 程序优化

| 笔记 | 原始文件 | 核心内容 |
|:-----|:---------|:---------|
| [[Attachments/csapp/notes/CH5-1-program-optimization]] | `CH5-1-程序优化的思路.pptx` | 优化方法论：消除循环低效、减少过程调用、别名、CSE |
| [[Attachments/csapp/notes/CH5-2-instruction-level-parallelism]] | `CH5-2-指令级并行优化.pptx` | ILP：流水线、延迟与吞吐量、循环展开 |
| [[Attachments/csapp/notes/CH5-3-parallel-optimization]] | `CH5-3-其他并行优化及例子.pptx` | SIMD、分支预测、缓存友好编程 |

### CH6: 存储器层次结构

| 笔记 | 原始文件 | 核心内容 |
|:-----|:---------|:---------|
| [[Attachments/csapp/notes/CH6-memory-hierarchy]] | `CH6-1-存储器层次结构-存储层次与存储技术.pptx` | SRAM/DRAM/磁盘、局部性原理、缓存映射 |

### CH7: 链接

| 笔记 | 原始文件 | 核心内容 |
|:-----|:---------|:---------|
| [[Attachments/csapp/notes/CH7-1-linking-basics]] | `CH7-1-linking原理.pptx` | ELF、符号解析、重定位、可执行文件加载 |
| [[Attachments/csapp/notes/CH7-2-library-linking]] | `CH7-2-库的链接及插桩.pptx` | 动态链接、PIC、GOT/PLT、函数插桩 |

### CH8: 异常控制流

| 笔记 | 原始文件 | 核心内容 |
|:-----|:---------|:---------|
| [[Attachments/csapp/notes/CH8-exceptions-processes]] | `CH8-1-异常控制流：异常与进程.pptx` | 异常分类、中断/陷阱/故障/终止、fork |
| [[Attachments/csapp/notes/CH8-process-management]] | `CH8-2-异常控制流：进程管理-student.pptx` | 进程状态、wait、execve、作业控制 |
| [[Attachments/csapp/notes/CH8-shell-signals]] | `CH8-3-异常控制流：Shell与信号.pptx` | 信号机制、kill/signal、阻塞/等待信号 |
| [[Attachments/csapp/notes/CH8-signal-safety]] | `CH8-4-异常控制流：异步信号安全与非本地跳转-student.pptx` | 可重入函数、sigsetjmp/siglongjmp |

### CH9: 虚拟存储器

| 笔记 | 原始文件 | 核心内容 |
|:-----|:---------|:---------|
| [[Attachments/csapp/notes/CH9-virtual-memory-concepts]] | `CH9-1-虚拟存储器的概念-student.pptx` | 地址空间、VM 作为缓存/内存管理/保护 |
| [[Attachments/csapp/notes/CH9-virtual-memory-details]] | `CH9-2-虚拟存储器详解.pptx` | 页表、TLB、多级页表、地址翻译流程 |
| [[Attachments/csapp/notes/CH9-memory-allocation-basics]] | `CH9-3-分配基础知识.pptx` | malloc/free、隐式/显式空闲链表 |
| [[Attachments/csapp/notes/CH9-memory-allocation-advanced]] | `CH9-4-分配高级知识.pptx` | 分离适配、标记-清除 |
| [[Attachments/csapp/notes/CH9-garbage-collection]] | `CH9-5-自动内存管理.pptx` | GC 类型：引用计数、标记-清除、复制、分代收集 |

---

## ✍️ 课堂活动笔记

CMU 课程配套的 **GDB 动手实验**，基于汇编代码理解底层概念：

| 笔记 | 原始文件 | 核心内容 |
|:-----|:---------|:---------|
| [[Attachments/csapp/notes/activity-machine-control]] | `04-activity fmachine-control.pdf` | 条件码、跳转指令、TEST/CMP、循环翻译、switch 跳转表 |
| [[Attachments/csapp/notes/activity-machine-procedures]] | `05-activity machine-procedures.pdf` | 运行时栈、参数传递、调用约定 |
| [[Attachments/csapp/notes/activity-machine-data]] | `06-activity machine-data.pdf` | 数组步长、struct 对齐、多维数组、大小端 |
| [[Attachments/csapp/notes/activity-machine-advanced]] | `07-activity machine-advanced.pdf` | 缓冲区溢出、gets、canary、ROP |

---

## 🛠️ 工具教程笔记

| 笔记 | 原始文件 | 核心内容 |
|:-----|:---------|:---------|
| [[Attachments/csapp/notes/GDB-tutorial]] | `GDB It's MyGO!!!!!.pptx` | GDB 调试器完整教程 |
| [[Attachments/csapp/notes/GDB-cheatsheet]] | `GDB_Cheat_Sheet (1).pdf` | GDB 命令速查表 |
| [[Attachments/csapp/notes/Linux101]] | `Linux101.pptx` | Linux 基础入门 |

---

## 知识图谱

```
CH2: 信息表示
  ├── 整数编码 ←→ CH3: 机器级算术指令
  └── 浮点编码
  
CH3: 机器级程序            ←→ CH5: 优化（了解汇编才能优化）
  ├── 基础 (指令/寄存器)
  ├── 控制流 ──────────────→ CH8: 异常控制流（信号/进程）
  ├── 数据 (数组/struct)   ─→ CH9: 虚拟内存（地址翻译）
  ├── 过程调用 ────────────→ CH7: 链接（符号解析/重定位）
  └── 缓冲区溢出 ──────────→ 安全编程

CH6: 存储器层次结构
  ├── 缓存原理 ────────────→ CH5: 缓存友好优化
  └── 局部性 ──────────────→ CH9: VM 作为缓存

CH7: 链接
  ├── 静态链接
  └── 动态链接/共享库

CH8: 异常控制流
  ├── 异常/陷阱
  ├── 进程/fork/exec
  └── 信号/安全

CH9: 虚拟内存
  ├── 地址翻译/页表/TLB
  ├── 内存分配 (malloc)
  └── 垃圾回收
```

---

## 提问指南

你可以基于这个知识库向我提问任何 CSAPP 相关内容，例如：

- **概念理解**：解释虚拟地址翻译全过程、什么是 ROP 攻击、canary 如何工作
- **代码分析**：给定一段汇编代码，解释其功能或对应的 C 代码
- **对比辨析**：调用者保存 vs 被调用者保存、静态链接 vs 动态链接
- **实验辅导**：帮助你理解课堂活动里的练习题
- **工具使用**：GDB 调试技巧、Linux 命令

> 只需引用笔记名即可，例如：结合 `CH9-memory-allocation-basics` 解释隐式空闲链表的工作原理

---

*原始课件在 `originals/`，笔记在 `notes/` —— 最后更新: 2026-06-15*
