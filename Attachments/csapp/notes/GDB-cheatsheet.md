---
title: GDB 调试命令速查表
date: 2024-09
tags: #CSAPP #GDB #调试 #工具
---

# GDB 调试命令速查表

## 启动与运行

| 命令 | 说明 |
|------|------|
| `gdb <program>` | 启动 GDB 并加载程序 |
| `gdb <program> <core>` | 启动 GDB 并加载 core dump |
| `gdb --args <program> <args...>` | 启动 GDB 并传递参数 |
| `gdb --pid <pid>` | 附加到正在运行的进程 |
| `set args <args...>` | 设置程序参数（启动后） |
| `run` | 运行程序 |
| `kill` | 终止当前运行的程序 |

---

## 断点 (Breakpoints)

| 命令 | 说明 |
|------|------|
| `break <where>` | 设置断点 |
| `delete <bp#>` | 删除指定断点 |
| `clear` | 删除所有断点 |
| `enable <bp#>` | 启用断点 |
| `disable <bp#>` | 禁用断点 |

### where 可以是

| 格式 | 示例 | 说明 |
|------|------|------|
| 函数名 | `break main` | 在函数入口处断点 |
| 行号 | `break 42` | 在当前文件指定行断点 |
| 文件:行号 | `break file.c:42` | 在指定文件的指定行断点 |
| `if` 条件 | `break main if x==5` | 条件断点 |

---

## 监视点 (Watchpoints)

| 命令 | 说明 |
|------|------|
| `watch <where>` | 设置监视点（变量值变化时停止） |
| `delete <wp#>` | 删除监视点 |
| `enable <wp#>` / `disable <wp#>` | 启用/禁用监视点 |
| `condition <wp#> <condition>` | 设置/更改监视点条件 |

---

## 单步执行 (Stepping)

| 命令 | 说明 |
|------|------|
| `step` | 执行到下一条源码行，**进入**函数内部 |
| `next` | 执行到下一条源码行，**不进入**函数内部 |
| `finish` | 执行到当前函数返回 |
| `continue` | 继续执行直到下一个断点 |

---

## 调用栈 (Call Stack)

| 命令 | 说明 |
|------|------|
| `backtrace` / `where` | 显示调用栈 |
| `backtrace full` / `where full` | 显示调用栈及各帧的局部变量 |
| `frame <frame#>` | 切换到指定的栈帧 |

---

## 变量与内存 (Variables and Memory)

### print 命令

```
print/format <what>
```

| 格式字符 | 含义 |
|---------|------|
| `a` | 指针 |
| `c` | 字符 |
| `d` | 有符号十进制整数 |
| `f` | 浮点数 |
| `o` | 八进制 |
| `s` | C 字符串（以 `\0` 终止） |
| `t` | 二进制（t = "two"） |
| `u` | 无符号十进制整数 |
| `x` | 十六进制 |

### display 命令

| 命令 | 说明 |
|------|------|
| `display/format <what>` | 每次单步后自动打印 |
| `undisplay <display#>` | 移除自动打印 |
| `enable display <display#>` | 启用自动打印 |
| `disable display <display#>` | 禁用自动打印 |

### x 命令 (examine memory)

```
x/nfu <address>
```

参数说明：

| 参数 | 含义 |
|------|------|
| **n** | 要打印的单元数量（默认 1） |
| **f** | 格式字符（同 print） |
| **u** | 单元大小 |

**单元大小 (Unit)**：

| 单元 | 含义 |
|------|------|
| `b` | 字节 (Byte, 1 字节) |
| `h` | 半字 (Half-word, 2 字节) |
| `w` | 字 (Word, 4 字节) |
| `g` | 巨字 (Giant word, 8 字节) |

**常用组合示例**：
```
x/4wx &array       # 以 4 字节为单元显示 4 个十六进制值
x/16bx &array      # 以 1 字节为单元显示 16 个十六进制值
x/12bx $rdi        # 从 $rdi 指向的地址开始显示 12 字节
x/s $rdi           # 显示字符串
x/2xb $rsp         # 从栈顶显示 2 字节
```

### what 可以是

| 格式 | 示例 | 说明 |
|------|------|------|
| 表达式 | `print x + 3` | 几乎任何 C 表达式 |
| 文件::变量 | `print file.c::counter` | 文件中定义的静态变量 |
| 函数::变量 | `print func::local` | 函数中的局部变量 |
| {类型}地址 | `print {int}0x7fff` | 将地址解释为指定类型 |
| `$寄存器` | `print $rdi` | 寄存器内容 |

---

## 程序操纵 (Manipulating the Program)

| 命令 | 说明 |
|------|------|
| `set var <var>=<value>` | 修改变量的值 |
| `return <expr>` | 强制当前函数立即返回指定值 |

---

## 源码浏览 (Sources)

| 命令 | 说明 |
|------|------|
| `directory <dir>` | 添加源码搜索目录 |
| `list` | 显示当前行附近的源码 |
| `list <file>:<func>` | 显示指定函数的源码 |
| `list <file>:<line>` | 显示指定行附近的源码 |
| `list <first>,<last>` | 显示指定范围的源码 |
| `set listsize <count>` | 设置 list 每次显示的行数 |

---

## 信息查询 (Info)

| 命令 | 说明 |
|------|------|
| `disassemble` | 反汇编当前函数 |
| `disassemble <where>` | 反汇编指定函数/地址 |
| `info args` | 打印当前函数的参数 |
| `info breakpoints` | 打印所有断点和监视点 |
| `info display` | 打印所有 display |
| `info locals` | 打印当前栈帧的局部变量 |
| `info sharedlibrary` | 列出已加载的共享库 |
| `info signals` | 列出所有信号及处理方式 |
| `info threads` | 列出所有线程 |
| `show directories` | 显示源码搜索目录 |
| `show listsize` | 显示 list 的行数 |
| `whatis <variable>` | 显示变量的类型 |

---

## 线程 (Threads)

| 命令 | 说明 |
|------|------|
| `thread <thread#>` | 切换到指定线程 |

---

## 信号 (Signals)

| 命令 | 说明 |
|------|------|
| `handle <signal> <options>` | 设置信号处理方式 |

**Options**:
| 选项 | 说明 |
|------|------|
| `print` / `noprint` | 信号发生时（不）打印消息 |
| `stop` / `nostop` | 信号发生时（不）停止程序 |
| `pass` / `nopass` | 信号（不）传递给程序 |

---

## 实用调试流程

### 1. 定位崩溃
```gdb
gdb ./program core
bt                     # 查看崩溃时的调用栈
frame 3                # 切换到问题帧
info locals            # 查看局部变量
```

### 2. 分析数组/结构体
```gdb
x/4wx &array           # 查看数组原始内存
x/16bx &struct_val     # 查看结构体每个字节
print *(struct pair*)ptr  # 按结构体类型打印
```

### 3. 追踪函数调用
```gdb
break main
run
step                   # 单步进入函数
backtrace              # 查看调用链
```

### 4. 条件调试
```gdb
break func if err != 0
watch var if var > 100
```

---

> **参考**：GDB 官方文档 https://sourceware.org/gdb/current/onlinedocs/gdb/
