---
title: 异常控制流：进程管理
date: 2026-06-15
tags: [CSAPP, 异常控制流, 进程管理, fork, wait]
---

# 异常控制流：进程管理

## 系统调用错误处理

出错时，Linux 系统级函数通常返回 -1，并设置全局变量 `errno` 以指示错误原因。

**硬性规则**：必须检查每个系统级函数的返回状态。

```c
if ((pid = fork()) < 0) {
    fprintf(stderr, "fork error: %s\n", strerror(errno));
    exit(1);
}
```

### 错误报告函数

```c
void unix_error(char *msg) {
    fprintf(stderr, "%s: %s\n", msg, strerror(errno));
    exit(1);
}
```

### 错误处理包装

```c
pid_t Fork(void) {
    pid_t pid;
    if ((pid = fork()) < 0)
        unix_error("Fork error");
    return pid;
}
pid = Fork();  // Only returns if successful
```

## 获取进程 ID

```c
pid_t getpid(void);   // 返回当前进程的 PID
pid_t getppid(void);  // 返回父进程的 PID
```

## 进程的状态

| 状态 | 描述 |
|------|------|
| Running | 进程正在执行指令 |
| Blocked/Sleeping | 进程无法继续，直到外部事件发生 |
| Stopped | 进程被用户操作阻止执行（如 Ctrl-Z） |
| Terminated/Zombie | 进程已完成，但其父进程尚未收到通知 |

## 终止进程

进程因三种原因终止：
1. 从 main 程序返回（自动调用 exit 函数）
2. 显示调用 `exit` 函数
3. 收到一个信号，其默认行为是终止进程

```c
void exit(int status);
```

`exit` 被调用一次，但从不返回。内核执行时：
- 立即终止该进程
- 通知该进程的父进程
- 释放该进程占用的资源

## 创建进程：fork

父进程通过调用 `fork` 函数创建一个新的运行的子进程。

```c
int fork(void);
```

- 返回 0 给子进程，返回子进程的 PID 给父进程
- 子进程**几乎**和父进程一模一样
- fork 函数被调用**一次**，却会返回**两次**

### fork 概念视图

创建执行状态的完整副本，包括用户级虚拟地址空间（独立的副本）和打开文件描述符（副本）。优化：使用**写时复制**避免内存拷贝。

### fork 示例

```c
int main(int argc, char** argv) {
    pid_t pid;
    int x = 1;
    pid = Fork();
    if (pid == 0) {  /* Child */
        printf("child: x=%d\n", ++x);
        return 0;
    }
    /* Parent */
    printf("parent: x=%d\n", --x);
    return 0;
}
```

**关键点**：
- 调用一次，返回两次
- 并发执行，无法预测父子进程的执行顺序
- 两份同样但独立的地址空间
- 共享文件 stdout

### 进程图

进程图是刻画并发程序中语句的偏序关系的工具。每个顶点对应于一条语句的执行，有向边 a->b 表示语句 a 发生在语句 b 前。

## 回收子进程

当进程终止时，它仍然占用系统资源，称为 **zombie**（僵尸进程）。

- 父进程主动回收：使用 `wait` 或 `waitpid` 获取子进程退出状态
- 内核随后删除僵尸子进程
- 如果父进程先于子进程终止，孤儿进程会被 `init` 进程回收（pid == 1）

### wait：与子进程同步

```c
pid_t wait(int *status);
```

挂起当前进程，直到其一个子进程终止。返回子进程的 PID。

```c
pid_t waitpid(pid_t pid, int *status, int options);
```

wait 的更灵活版本，可以等待特定子进程或子进程组。

### 状态代码宏

使用 `sys/wait.h` 中定义的宏来解读状态：

| 宏 | 功能 |
|----|------|
| WIFEXITED | 子进程是否正常退出 |
| WEXITSTATUS | 获取退出状态 |
| WIFSIGNALED | 子进程是否被信号终止 |
| WTERMSIG | 获取终止信号 |
| WIFSTOPPED | 子进程是否暂停 |
| WSTOPSIG | 获取暂停信号 |

### wait 示例

```c
void fork10() {
    pid_t pid[N];
    int i, child_status;
    for (i = 0; i < N; i++)
        if ((pid[i] = fork()) == 0) {
            sleep(3);
            exit(100 + i);  /* Child */
        }
    for (i = 0; i < N; i++) {  /* Parent */
        pid_t wpid = wait(&child_status);
        if (WIFEXITED(child_status))
            printf("Child %d terminated with exit status %d\n",
                   wpid, WEXITSTATUS(child_status));
        else
            printf("Child %d terminate abnormally\n", wpid);
    }
}
```

## 创建不同的进程：execve

```c
int execve(char *filename, char *argv[], char *envp[]);
```

- 在当前进程中加载和运行，覆盖代码段、数据段和栈
- 保留 PID、打开的文件和信号上下文
- 调用一次，永不返回（除非有错误）

### execve 组合 fork

```c
int main(int argc, char** argv) {
    pid_t pid;
    pid = Fork();
    if (pid == 0) {  /* Child */
        char *program = "./hello";
        char *args[] = {NULL};
        int ret = execv(program, args);
    }
    /* Parent */
    printf("parent sleeps for 2s...\n");
    sleep(2);
    printf("parent: x=%d\n", --x);
    return 0;
}
```

### 新程序启动时的栈结构

- `argc` 在 `%rdi`
- `argv` 在 `%rsi`
- `envp` 在 `%rdx`

### execve 和进程内存布局

通过 execve 加载并运行新程序时：
- 释放旧区域的 `vm_area_struct` 和页表
- 为新区域创建 `vm_area_struct` 和页表
- 设置 PC 为 `.text` 段的入口点

## 总结

| 操作 | 函数 | 特点 |
|------|------|------|
| 创建进程 | `fork` | 一次调用，两次返回 |
| 进程完成 | `exit` | 一次调用，不返回 |
| 回收和等待 | `wait` / `waitpid` | 与子进程同步 |
| 加载和运行程序 | `execve` | 一次调用，（通常）不返回 |
