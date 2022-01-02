---
title: 进程控制
date: 2021-06-07 22:41:33
tags: linux
---

# 介绍  

​	

<!-- more -->  

# 进程

## 进程控制

### fork函数-创建分支进程

- 函数名

  ```c
  pid_t fork(void);
  ```

  - 返回值
    - 失败 -1
    - 成功返回两次
      - 父进程返回子进程的id
      - 子进程返回0

### getpid函数-获得当前进程的id

- 函数名

  ```c
  pid_t getpid(void);
  ```

### getppid函数-获得当前父进程的id

- 函数名

  ```c
  pid_t getppid(void);
  ```

### 进程命令

- ps
  - ps  aux
  - ps  ajx -- 可以追溯进程之间的血缘关系
- kill
  - 给进程发送一个信号
  - SIGKILL ９号信号
  - kill -SIGKILL pid --杀死进程

### 例子

```c
#include<unistd.h>
#include<stdlib.h>
int main()
{
    int n = 5;
    int i = 0;
    pid_t pid = 0;
    for(i = 0 ; i < 5; i++ )
    {
        pid = fork();
        if(pid == 0)
        {
            printf("I am child pid = %d ，ppid = %d\n",getpid(),getppid());
            break;
        }
    }
    return 0;
}
```

## 进程共享

- 读时共享，写时复制.

- 全局变量不共享

  

