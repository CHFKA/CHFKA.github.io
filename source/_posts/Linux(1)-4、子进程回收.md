---
title: 子进程回收
date: 2021-06-08
tags: linux
---

# 介绍  

​	

<!-- more -->  

# 子进程回收

## wait函数

- 回收子进程，知道子进程的死亡原因

  - ```c
    pid_t wait(int *status);
    ```

    - status传出参数
    - 返回值
      - 成功返回终止的子进程ID
      - 失败返回-1

  - 作用：

    - 阻塞等待
    - 回收子进程资源
    - 查看死亡原因

  - 死亡原因

    - 正常死亡WIFEXITED
      - 如果WIFEXITED为真，使用WEXITSTATUS得到退出状态
    - 非正常死亡WIFSIGNALED
      - 如果WIFSIGNALED为真，使用WTERMSIG得到信号


## waitpid函数

- pid_t waitpid(pid_t pid,int *status, int options);

  - pid

    - 小于-1  组id
    - -1 回收任意
    - 0 回收和调用进程组id相同组内的子进程
    - 大于0 回收指定的pid

  - option

    - 0与wait相同，阻塞

    - WNOHANG如果当前没有子进程退出的，会立刻返回

  - 返回值

    - 如果设置了WHOHANG，那么如果没有子进程退出，返回0
      - 如果有子进程退出，返回退出的pid
    - 失败返回-1 （没有子进程）

- 回收多个子进程

  - ```
    while(1)
    {
    	pid_t wpid = waitpid(-1,NULL,WNOHANG);
    	if(wpid == -1)
    		break;
    }
    ```

    
