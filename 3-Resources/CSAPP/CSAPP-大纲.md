---
title: CSAPP 大纲
date: 2026-06-16
tags: [导航, CSAPP, MOC]
---

# CSAPP 大纲导航

> CMU 15-213 深入理解计算机系统 · 合并版笔记 `[[3-Resources/CSAPP/CSAPP]]`
> 在右侧 **大纲侧栏** 中可直接看到全部标题并跳转。

---

## CH2 — 信息的表示与处理

- [[3-Resources/CSAPP/CSAPP#CH2: 位与字节 — 整数编码与位运算|CH2: 整数编码与位运算]]
  - 无符号/补码编码、类型转换、位运算、移位
- [[3-Resources/CSAPP/CSAPP#CH2: 浮点数 — IEEE 754 标准|CH2: IEEE 754 浮点数]]
  - 浮点表示、规格化/非规格化、舍入、浮点运算

---

## CH3 — 程序的机器级表示

- [[3-Resources/CSAPP/CSAPP#CH3: 机器级基础 — 指令与寄存器|CH3: 指令与寄存器]]
  - x86-64 寄存器、MOV/LEA、算术指令
- [[3-Resources/CSAPP/CSAPP#CH3: 机器级控制流 — 条件码与跳转|CH3: 条件码与跳转]]
  - CF/ZF/SF/OF、CMP/TEST、JMP、条件分支、循环、Switch
- [[3-Resources/CSAPP/CSAPP#CH3: 机器级数据 — 数组与结构体对齐|CH3: 数组与结构体]]
  - 数组分配/访问、结构体布局、对齐填充、联合体、字节序
- [[3-Resources/CSAPP/CSAPP#CH3: 机器级过程 — 运行时栈与调用|CH3: 运行时栈与调用]]
  - 栈帧、call/ret、参数传递、寄存器保存、递归、尾调用优化
- [[3-Resources/CSAPP/CSAPP#CH3: 缓冲区溢出 — 攻击与防御|CH3: 缓冲区溢出]]
  - 栈溢出原理、代码注入、ROP、金丝雀、ASLR、NX

---

## CH5 — 程序性能优化

- [[3-Resources/CSAPP/CSAPP#CH5: 程序优化思路 — 编译器优化|CH5: 编译器优化]]
  - 五大优化思路、编译器优化技术、优化拦路虎
- [[3-Resources/CSAPP/CSAPP#CH5: 指令级并行 — 流水线与循环展开|CH5: 指令级并行]]
  - CPE 度量、CPU 微架构、循环展开、寄存器溢出
- [[3-Resources/CSAPP/CSAPP#CH5: 并行优化 — SIMD 与缓存|CH5: SIMD 与缓存]]
  - SIMD 向量化、线程并行、分支预测、缓存友好编程

---

## CH6 — 存储器层次结构

- [[3-Resources/CSAPP/CSAPP#CH6: 存储器层次结构 — 缓存与局部性|CH6: 缓存与局部性]]
  - SRAM/DRAM、局部性原理、缓存组织、Cache Miss 分类

---

## CH7 — 链接

- [[3-Resources/CSAPP/CSAPP#CH7: 链接原理 — ELF 与符号解析|CH7: 链接原理]]
  - ELF 格式、符号解析、重定位
- [[3-Resources/CSAPP/CSAPP#CH7: 库链接 — 动态库与 GOT/PLT|CH7: 库链接]]
  - 静态/动态库、GOT、PLT、位置无关代码 (PIC)

---

## CH8 — 异常控制流 (ECF)

- [[3-Resources/CSAPP/CSAPP#CH8: 异常与进程 — 异常分类与 fork|CH8: 异常与进程]]
  - 异常分类、进程模型、fork、进程图
- [[3-Resources/CSAPP/CSAPP#CH8: 进程管理 — wait/execve/作业|CH8: 进程管理]]
  - wait、execve、前台/后台作业
- [[3-Resources/CSAPP/CSAPP#CH8: Shell 与信号 — 信号机制|CH8: Shell 与信号]]
  - 信号发送/接收/阻塞、信号处理程序
- [[3-Resources/CSAPP/CSAPP#CH8: 信号安全 — 可重入与非本地跳转|CH8: 信号安全]]
  - 可重入函数、异步信号安全、sigsetjmp/siglongjmp

---

## CH9 — 虚拟内存

- [[3-Resources/CSAPP/CSAPP#CH9: 虚拟内存概念 — 地址空间与页表|CH9: 虚拟内存概念]]
  - 地址空间、分页、页表结构
- [[3-Resources/CSAPP/CSAPP#CH9: 虚拟内存详 — 多级页表与 TLB|CH9: 多级页表与 TLB]]
  - 多级页表、TLB 加速、地址翻译流程
- [[3-Resources/CSAPP/CSAPP#CH9: 内存分配基础 — malloc 与空闲链表|CH9: malloc 与空闲链表]]
  - 隐式/显式空闲链表、边界标记
- [[3-Resources/CSAPP/CSAPP#CH9: 内存分配进阶 — 分离适配|CH9: 内存分配进阶]]
  - 分离适配、slab 分配器
- [[3-Resources/CSAPP/CSAPP#CH9: 垃圾回收 — GC 算法|CH9: 垃圾回收]]
  - Mark & Sweep、引用计数

---

## 🧪 课堂活动

- [[3-Resources/CSAPP/CSAPP#活动: 机器级控制流 — 条件码实践|活动: 控制流实践]]
- [[3-Resources/CSAPP/CSAPP#活动: 机器级过程 — 调用约定实践|活动: 调用约定实践]]
- [[3-Resources/CSAPP/CSAPP#活动: 内存数据布局 — 对齐与大小端|活动: 内存数据布局]]
- [[3-Resources/CSAPP/CSAPP#活动: 缓冲区溢出 — 安全实践|活动: 缓冲区溢出实践]]

---

## 🛠️ 工具与参考

- [[3-Resources/CSAPP/CSAPP#GDB 命令速查表|GDB 命令速查表]]
- [[3-Resources/CSAPP/CSAPP#GDB 调试器教程|GDB 调试器教程]]
- [[3-Resources/CSAPP/CSAPP#Linux 基础入门|Linux 基础入门]]
