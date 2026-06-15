---
title: GDB 调试教程
date: 2026-06-15
tags: [CSAPP, GDB, 调试, 工具]
---

# GDB 调试教程

## 为什么使用 GDB

- 查看程序的运行状态
- 跟踪程序如何运行
- 查看内部数据状态

## 如何启动 GDB

```bash
# 直接运行 gdb，file 加载某一程序
gdb ./program

# 启动后打好断点和数据监测，run 运行程序
```

### 被调试文件类型

- 含有调试符号（通过 `gcc -g` 生成）
- 没有调试符号（gcc 生成时不带 `-g`）
- 运行中进程
- 内核转储文件

## 基础命令

### 运行控制

| 命令 | 简写 | 作用 |
|------|------|------|
| run | r | 运行程序（可加参数） |
| kill | | 终止当前运行的程序 |
| continue | c | 继续执行到下一个断点 |
| step | s | 单步进入（进入函数内部） |
| next | n | 单步跳过（不进入函数） |
| finish | | 执行完当前函数并返回 |
| until | | 跳出循环 |

### 断点

| 命令 | 示例 | 说明 |
|------|------|------|
| break func | `b func` | 在函数入口处打断点 |
| break 10 | `b 10` | 在当前文件第 10 行打断点 |
| break file.c:15 | `b file.c:15` | 在指定文件的第 15 行打断点 |
| info breakpoints | `i b` | 查看所有断点 |
| delete 2 | `d 2` | 删除编号为 2 的断点 |
| disable 3 | `dis 3` | 禁用编号 3 的断点 |
| enable 3 | `en 3` | 启用编号 3 的断点 |

### 程序状态

| 命令 | 作用 |
|------|------|
| `print p` / `p` | 打印变量或表达式的值 |
| `print /x` / `p/x` | 以十六进制打印 |
| `print *array@len` | 打印数组的前 len 个元素 |
| `backtrace` / `bt` | 查看函数调用堆栈 |
| `frame` / `f` | 切换当前栈帧 |
| `info locals` / `i locals` | 查看当前函数局部变量 |
| `info args` / `i args` | 查看当前函数参数 |
| `list` / `l` | 显示源代码 |

### 查看数据：x 命令

`x /<重复次数><格式><大小> <地址>`

**格式**：

| 格式 | 含义 |
|------|------|
| x | 十六进制 |
| d | 有符号十进制 |
| u | 无符号十进制 |
| o | 八进制 |
| t | 二进制 |
| c | 字符 |
| s | 字符串（直到 \0） |
| f | 浮点数 |
| a | 地址（带符号信息） |
| i | 机器指令（反汇编） |

**大小**：

| 大小 | 字节数 | 用途 |
|------|--------|------|
| b | 1 | byte |
| h | 2 | halfword |
| w | 4 | word |
| g | 8 | giant word |

**示例**：

```gdb
x /10xb 0x7fffffffde00    # 显示 10 个字节（十六进制）
x /4wd $rsp               # 显示 4 个有符号十进制字
x /s 0x7fffffffde00       # 将地址视为 C 字符串
x /2gf &my_double          # 显示 2 个巨字的浮点数
x /i $rip                 # 显示当前指令指针处的反汇编
x /4xg $rsp               # 显示栈顶开始的 4 个巨字
```

## 高级技巧

| 技巧 | 命令 |
|------|------|
| 条件断点 | `break <location> if <condition>` |
| 监视 | `watch` / `rwatch` / `awatch` |
| 显示变量 | `display x` / `display /x y` / `info display` / `undisplay 1` |
| 修改变量 | `set var x = 0` |
| 调用函数 | `call func(param)` |
| 跳转 | `jump` |
| 查看栈 | `backtrace` |

## 汇编信息

```gdb
disassemble / disass func    # 反汇编某一函数
disass /r _start             # 反汇编完整文件
layout asm / split / regs    # 布局视图
```

## GDB 初始化

创建一个 `gdb_init.txt`，每次使用 `gdb -x gdb_init.txt`：

```gdb
layout asm
layout reg
```

- `Ctrl+L`：刷新
- `focus asm / cmd`：切换焦点

## GDB 调试多进程

```bash
gdb ./fork
(gdb) set follow-fork-mode child    # 调试子进程
(gdb) b 28                          # 在 if(pid == 0) 处设断点
(gdb) run
```

> GDB 默认调试父进程，可设置 `set follow-fork-mode child` 来调试子进程。
