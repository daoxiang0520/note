---
title: Linux 101
date: 2026-06-15
tags: [CSAPP, Linux, 工具, 命令行]
---

# Linux 101

## 前置知识

### 推荐阅读

- [提问的智慧](http://www.catb.org/~esr/faqs/smart-questions.html)
- How To Ask Questions The Smart Way
- [Runoob 菜鸟教程](https://www.runoob.com/)

### 安装 Linux

**常见问题**：
- WSL 下载慢：前往 GitHub 下载离线版安装包
- 安装软件慢：配置镜像源

## 文件系统

### 基本概念

- 文件系统在硬盘上承载了你的文件
- Linux 下没有盘符的概念，所有硬盘要么未挂载，要么挂载在根目录及其子目录下
- Linux 不使用 Windows 的 NTFS、exFAT，也不使用 Mac OS 的 APFS

### 常用目录标识

| 符号 | 含义 |
|------|------|
| `~` | 当前用户的家目录（`/home/...`） |
| `.` | 当前目录 |
| `..` | 上级目录 |
| `/` | 路径分隔符（Windows 使用 `\`） |

### 基本命令

```bash
pwd     # 查看当前路径
ls      # 列出目录内容
cd      # 切换目录
mkdir   # 创建目录
cp      # 复制
rm      # 删除
mv      # 移动/重命名
```

### Linux 下的文件

- **换行符**：Linux/macOS 使用 LF (`\n`)，Windows 使用 CRLF (`\r\n`)
- **文本文件**：可能存在不同编码格式（UTF-8, GB2312 等），通常使用 UTF-8
- **二进制文件**：存在各种格式（可执行文件、压缩文件等）
- 使用 `file` 命令判断文件类型

### 常见问题

- **执行当前目录下的程序**：使用 `./program`
- **文件权限问题**：使用 `chmod` 修改权限
- **回收站**：Linux 没有回收站概念，删除需谨慎

## 进入 Linux 的世界

### 要点

- Linux 默认不提供图形界面
- 没有文件资源管理器，使用终端命令与文件系统交互
- 系统自带 `man` 手册
- 命令行程序一般可附加 `-h` / `--help` 获取帮助

### 用户机制

- 文件与目录存在针对不同用户/用户组的可读/可写/可执行三重权限

### Root

- root 为 Linux 下具有最高权限的用户
- 被授权的用户使用 `sudo` 使用 root 权限
- 使用 `su` 命令切换用户

### 重定向 / 管道

```bash
>    # 把输出写进文件（覆盖）
>>   # 把输出写进文件（追加）
<    # 把文件内容作为输入
|    # 管道：把上一个命令的输出作为下一个命令的输入
```

### 终端

Linux 下一般默认使用 bash 处理用户命令。终端实现了管道、重定向、进程管理、变量管理等功能。

- [Bash 快捷键](https://www.runoob.com/w3cnote/bash-shortcut.html)

## 工具链

### 编译器：gcc

将 C 语言源代码编译为可执行文件：预处理 -> 编译 -> 汇编 -> 链接

```bash
gcc -o program source.c    # 编译链接一步完成
gcc -c source.c            # 只编译不链接
gcc -S source.c            # 只生成汇编代码
gcc -E source.c            # 只进行预处理
```

### 构建系统：makefile

本质是定义一系列文件编译的方式以及它们的依赖关系。

### GDB

GDB 是广泛使用的调试器，需要掌握：

```bash
disas                   # 反汇编
break <symbol>          # 设置断点
print / x / display     # 查看寄存器与内存
layout asm / regs       # 布局视图
```

### 编辑器

| 编辑器 | 特点 |
|--------|------|
| nano | 适合初学者 |
| Vim | 较难但有好处，`i` 插入模式，`Esc` 普通模式，`:wq` 保存退出，`:q!` 强制退出 |
| gedit / Emacs | 也可解决问题 |

### Python

现代 Linux 环境都自带 Python。进制转换辅助工具：

```python
hex(255)    # -> '0xff'
int('0xff', 16)  # -> 255
bin(255)    # -> '0b11111111'
```

## 搜索技巧

- **精确匹配**：使用引号 `"keyword"`
- **排除关键词**：`-keyword`
- **限定站点**：`site:example.com`
- **限定格式**：`filetype:pdf`

## 使用 AI

- 理解 AI 什么时候会犯错
- 给 AI 充足的信息
- 要求 AI 给出分析过程，自主交叉验证
- 避免直接使用生成的代码
- 要求 AI 给出建议和教程而非完整解决方案

### 使用 AI Agent

- 使用高质量的 Agent 工具（如 Claude Code, Open Code, Codex 等）
- 给 AI 准确的指示
- 认真阅读学习 AI 解决问题的流程
- 避免直接使用 AI 生成的结果
- 为 Agent 圈定能力边界
- 审查 AI 的操作
