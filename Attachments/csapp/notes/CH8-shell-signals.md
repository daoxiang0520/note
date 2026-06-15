---
title: 异常控制流：Shell 与信号
date: 2026-06-15
tags: [CSAPP, 异常控制流, Shell, 信号, 进程管理]
---

# 异常控制流：Shell 与信号

## Linux 进程层次结构

- 0 号进程：idle 进程
- 1 号进程：init（由 idle 创建），所有用户级进程的祖先进程
- 2 号进程：kthreadd（由 idle 创建），所有内核级进程的祖先进程

> 从 2010 年代中期开始，systemd 逐渐替换 init 成为大多数主流 Linux 发行版的默认初始化系统。

## Shell 程序

Shell 是代表用户运行程序的应用程序。常见 Shell：

| Shell | 描述 |
|-------|------|
| sh | Original Unix shell (Stephen Bourne, AT&T Bell Labs, 1977) |
| csh/tcsh | BSD Unix C shell |
| bash | "Bourne-Again" Shell (default Linux shell) |

### 前台进程与后台进程

| 特性 | 前台进程 | 后台进程 |
|------|---------|---------|
| Shell 阻塞 | 阻塞 Shell | 非阻塞 Shell |
| 信号接收 | 接收 Ctrl-C, Ctrl-Z | 忽略 Ctrl-C, Ctrl-Z |
| 标准输入 | 当前键盘 | 无法从键盘读取 |
| 启动方式 | 直接执行 | 命令末尾加 `&` |

### 前后台切换

| 命令 | 作用 |
|------|------|
| `Ctrl-Z` | 暂停前台进程 |
| `fg` | 将进程转前台 |
| `bg` | 将进程转后台 |
| `jobs` | 显示进程队列 |

### 简单的 Shell 实现

基本循环：Read-Eval-Print Loop (REPL)

```c
int main() {
    char cmdline[MAXLINE];
    while (1) {
        printf("> ");
        Fgets(cmdline, MAXLINE, stdin);
        if (feof(stdin)) exit(0);
        eval(cmdline);
    }
}
```

### Shell 的 eval 函数

```c
void eval(char *cmdline) {
    char *argv[MAXARGS];
    char buf[MAXLINE];
    int bg;
    pid_t pid;
    strcpy(buf, cmdline);
    bg = parseline(buf, argv);
    if (argv[0] == NULL) return;
    if (!builtin_command(argv)) {
        if ((pid = Fork()) == 0) {  /* Child runs user job */
            if (execve(argv[0], argv, environ) < 0) {
                printf("%s: Command not found.\n", argv[0]);
                exit(0);
            }
        }
        /* Parent waits for foreground job */
        if (!bg) {
            int status;
            if (waitpid(pid, &status, 0) < 0)
                unix_error("waitfg: waitpid error");
        } else
            printf("%d %s", pid, cmdline);
    }
    return;
}
```

**缺陷**：Shell 使用 wait 回收前台子进程，但后台子进程终止后会变成僵尸进程，因为 Shell 通常不会终止。

## 信号

**信号**是一条小消息，通知进程系统中发生了一个某种类型的事件。从内核发送（有时由其他进程请求）到目标进程。

| ID | 名称 | 默认行为 | 相应事件 |
|----|------|---------|---------|
| 2 | SIGINT | 终止 | 用户键入 Ctrl-C |
| 9 | SIGKILL | 终止 | 杀死程序（不能被覆盖或忽略） |
| 11 | SIGSEGV | 终止 | 段故障 |
| 14 | SIGALRM | 终止 | 定时器信号 |
| 17 | SIGCHLD | 忽略 | 子进程停止或终止 |

### 发送信号的 3 种方式

1. 内核检测到事件，直接发送（除零错误、子进程终止等）
2. 用户进程请求内核发送
   - 使用 `kill` 函数
   - 使用 `/bin/kill` 工具
3. 使用键盘 Ctrl-C（SIGINT）、Ctrl-Z（SIGTSTP）

### 进程组

每个进程都属于一个进程组。

```c
getpgrp()  // 返回当前进程的进程组 ID
setpgid()  // 改变一个进程的进程组
```

从键盘发送信号会发送到前台进程组中的每个作业。

### 接收信号

当目的进程被内核强迫以某种方式对信号的发送做出反应时，它就**接收**了信号。

可能的反应方式：
- **忽略**信号
- **终止**进程（可能会 core dump）
- **捕获**信号，通过执行信号处理程序

### 待处理和阻塞信号

- **待处理 (pending)**：发出而没有接收的信号。每种类型至多只有一个待处理信号（信号不会排队）
- **阻塞 (blocked)**：进程可以阻塞接收某种信号，用 `sigprocmask` 函数设置

内核维护每个进程上下文中的 `pending` 和 `blocked` 位向量。

#### 接收信号时机

内核从异常处理程序返回，并准备将控制权交给进程 p 的用户代码时：
1. 计算 `pnb = pending & ~blocked`
2. 如果 `pnb == 0`，正常返回
3. 否则选择最小的非零位 k，强制进程接收信号 k

### 设置信号处理程序

```c
handler_t *signal(int signum, handler_t *handler);
```

- `SIG_IGN`：忽略信号
- `SIG_DFL`：恢复默认行为
- 用户定义的处理程序地址

```c
void sigint_handler(int sig) {
    printf("So you think you can stop the bomb with ctrl-c, do you?\n");
    sleep(2);
    printf("Well...");
    fflush(stdout);
    sleep(1);
    printf("OK. :-)\n");
    exit(0);
}

int main() {
    if (signal(SIGINT, sigint_handler) == SIG_ERR)
        unix_error("signal error");
    pause();
    return 0;
}
```

### 信号的屏蔽

```c
sigset_t mask, prev_mask;
Sigemptyset(&mask);
Sigaddset(&mask, SIGINT);

// 阻塞 SIGINT 信号
Sigprocmask(SIG_BLOCK, &mask, &prev_mask);

/* Code region that will not be interrupted by SIGINT */

// 恢复之前的阻塞集
Sigprocmask(SIG_SETMASK, &prev_mask, NULL);
```
