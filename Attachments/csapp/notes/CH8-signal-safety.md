---
title: 异常控制流：异步信号安全与非本地跳转
date: 2026-06-15
tags: [CSAPP, 异常控制流, 信号安全, 非本地跳转, setjmp, longjmp]
---

# 异常控制流：异步信号安全与非本地跳转

## 信号处理程序作为并发逻辑流

信号处理程序是一个独立的逻辑流（而不是一个进程），它与主程序并发运行。

### 嵌套信号处理程序

信号处理程序可以被其他信号处理程序中断。

## 安全的信号处理

信号处理程序很麻烦，因为 handler 和主函数存在并发，handler 之间存在嵌套并发。如果共享相同的全局数据结构，可能被破坏。

### 安全编程原则

#### G0: 处理程序要尽可能简单

handler 中只是记录一个全局标记，并通过该标记通知主函数。

```c
volatile sig_atomic_t flag = 0;
void handler(int sig) {
    flag = 1;  // 唯一操作，复杂任务交给主函数
}
int main() {
    while (flag) {
        // 在这里安全地响应信号
    }
}
```

#### G1: 只调用异步信号安全的函数

异步信号安全的函数要么是可重入的，要么不能被信号处理程序中断。

- **安全的**：`_exit`, `write`, `wait`, `waitpid`, `sleep`, `kill`
- **不安全的**：`printf`, `sprintf`, `malloc`, `exit`

> `write` 是唯一的异步信号安全的输出函数。

使用 Safe I/O (SIO) 库：

```c
void sigint_handler(int sig) {
    Sio_puts("So you think you can stop the bomb with ctrl-c, do you?\n");
    sleep(2);
    Sio_puts("Well...");
    sleep(1);
    Sio_puts("OK. :-)\n");
    _exit(0);
}
```

#### G2: 保存和恢复 errno

防止其他处理程序覆盖 errno 值。

#### G3: 阻塞所有信号，保护共享全局数据结构

```c
sigset_t mask_all, prev_all;
Sigfillset(&mask_all);
// 在访问共享数据前阻塞信号
Sigprocmask(SIG_BLOCK, &mask_all, &prev_all);
modify_shared_data();
Sigprocmask(SIG_SETMASK, &prev_all, NULL);
```

#### G4: 用 volatile 声明全局变量

防止错误的编译器优化。

#### G5: 用 volatile sig_atomic_t 声明标志

仅被单独读取或单独写入的变量（如 `flag = 1`，不是 `flag++`）。

### 可移植的信号处理

使用基于 `sigaction` 封装的 `Signal()`：

```c
handler_t *Signal(int signum, handler_t *handler) {
    struct sigaction action, old_action;
    action.sa_handler = handler;
    sigemptyset(&action.sa_mask);
    action.sa_flags = SA_RESTART;
    if (sigaction(signum, &action, &old_action) < 0)
        unix_error("Signal error");
    return (old_action.sa_handler);
}
```

## 典型的信号处理例子

### 不正确的 SIGCHLD 处理

信号不会排队，同时发送的多个同类信号只会保留一个。

```c
void child_handler(int sig) {
    int olderrno = errno;
    pid_t pid;
    if ((pid = wait(NULL)) < 0)
        Sio_error("wait error");
    ccount--;
    errno = olderrno;
}
```

问题：N=5 却只回收了 2 个子进程，因为信号不排队。

### 正确的 SIGCHLD 处理

必须等待所有终止的子进程，将 wait 放在循环中：

```c
void child_handler2(int sig) {
    int olderrno = errno;
    pid_t pid;
    while ((pid = wait(NULL)) > 0) {
        ccount--;
        Sio_puts("Handler reaped child ");
        Sio_putl((long)pid);
        Sio_puts("\n");
    }
    if (errno != ECHILD)
        Sio_error("wait error");
    errno = olderrno;
}
```

### 同步控制流避免并发错误

**问题**：在 addjob 前子进程可能已经结束，导致先 deletejob 再 addjob。

**解决方案**：在 fork 前阻塞 SIGCHLD，在 addjob 后才恢复。

```c
int main(int argc, char **argv) {
    sigset_t mask_all, mask_one, prev_one;
    Sigfillset(&mask_all);
    Sigemptyset(&mask_one);
    Sigaddset(&mask_one, SIGCHLD);
    Signal(SIGCHLD, handler);
    initjobs();
    while (1) {
        Sigprocmask(SIG_BLOCK, &mask_one, &prev_one);  // Block SIGCHLD
        if ((pid = Fork()) == 0) {  /* Child */
            Sigprocmask(SIG_SETMASK, &prev_one, NULL);  // Unblock SIGCHLD
            Execve("/bin/date", argv, NULL);
        }
        Sigprocmask(SIG_BLOCK, &mask_all, NULL);
        addjob(pid);  /* Add the child to the job list */
        Sigprocmask(SIG_SETMASK, &prev_one, NULL);
    }
    exit(0);
}
```

### 使用 sigsuspend 等待信号

`sigsuspend` 是一个原子过程：临时解除信号阻塞并挂起进程等待信号，不存在信号丢失的窗口期。

```c
while (!pid)
    Sigsuspend(&prev);  // 原子地同意接收 SIGCHLD 信号以便唤醒
```

## 非本地跳转：setjmp/longjmp

强大（但危险）的用户级机制，用于将控制转移到任意位置，以受控方式打破过程调用/返回的规则。

### setjmp

```c
int setjmp(jmp_buf j);
```

- 必须在调用 longjmp 之前调用
- 设置返回点，记录程序计数器、寄存器上下文和栈指针
- 调用一次，返回一次或多次
- 返回 0

### longjmp

```c
void longjmp(jmp_buf j, int i);
```

- 从设置跳转缓冲区 j 的 setjmp 返回一次
- 这次 setjmp 返回 i 而不是 0
- 调用一次，从不返回

### 示例

```c
jmp_buf buf;
int error1 = 0;
int error2 = 1;

void foo(void) {
    if (error1) longjmp(buf, 1);
    bar();
}
void bar(void) {
    if (error2) longjmp(buf, 2);
}

int main() {
    switch (setjmp(buf)) {
        case 0: foo(); break;
        case 1: printf("Detected an error1 condition in foo\n"); break;
        case 2: printf("Detected an error2 condition in bar\n"); break;
    }
    exit(0);
}
```

### 在 C 中模拟 try-catch

```c
#include <stdio.h>
#include <setjmp.h>

jmp_buf env;

void riskyFunction() {
    printf("Starting risky function...\n");
    if (1) {  // 模拟错误条件
        printf("Error occurred, jumping back...\n");
        longjmp(env, 1);  // 跳转回 setjmp 的位置
    }
}

int main() {
    if (setjmp(env) == 0) {
        riskyFunction();
    } else {
        printf("Caught an error!\n");
    }
    printf("Program continues...\n");
    return 0;
}
```

### 非本地跳转的限制

只能跳转到已被调用但尚未完成的函数的环境，遵循栈的规则。

### 典型应用

- **深层嵌套的错误处理**：longjmp 直接跳回顶层的 setjmp 位置
- **模拟异常处理机制**：在 C 语言中模拟 try-catch
- **信号处理中的特殊跳转**：使用 sigsetjmp/siglongjmp
- **实现轻量级协程**：保存和恢复 CPU 上下文

### 在按下 Ctrl-C 时重启的程序

```c
#include "csapp.h"
sigjmp_buf buf;

void handler(int sig) {
    siglongjmp(buf, 1);
}

int main() {
    if (!sigsetjmp(buf, 1)) {
        Signal(SIGINT, handler);
        Sio_puts("starting\n");
    } else
        Sio_puts("restarting\n");
    while (1) {
        Sleep(1);
        Sio_puts("processing...\n");
    }
    exit(0);
}
```

## 总结

- **信号**提供进程级别的异常处理，需小心编写信号处理程序
- **非局部跳转**提供进程内的异常控制流，遵循栈规则的约束
