---
title: 守护进程
date: 2021-06-12
tags: linux
---

# 介绍  

​	

<!-- more -->  

# 守护进程

## 进场组

- 代表一个或多个进程的基合。

## 会话--  多个进程组对应一个会话

- 调用进程不能是进程组组长，该进程变成新会话首进程

- 该进程成为一个新进程组的组长进程

- 新会话丢弃原有的控制终端，该会话没有控制终端

- 该调用进程是组长进程，则出错返回

- 建立新会话时，先调用fork，父进程终止，子进程调用setsid

## setsid函数

- 获取进程所属的会话ID

- ```c
  pid_t getsid(pid_t pid); 
  ```
  
  - 成功：返回调用进程的会话ID，失败：-1，设置errno
  
  - pid为0表示察看当前基层session ID

## 守护进程

- 调用setsid函数创建一个新的Session，并成为Session Leader

- 创建守护进程模型
  
  - 创建子进程，父进程退出
    
    - 所有工作在子进程中进行形式上脱离了控制终端
  
  - 在子进程中创建新会话
    
    - setsid()函数
    
    - 使子进程完全独立出来，脱离控制
  
  - 改变当前目录为根目录  -- 非必须
    
    - chdir()函数
    
    - 防止占用可卸载的文件系统
    
    - 也可以换成其他路径
  
  - 重设文件权限掩码
    
    - umask()函数
    
    - 防止继承的文件创建屏蔽子拒绝某些权限
    
    - 增加守护进程灵活性
  
  - 关闭文件描述符
    
    - 继承的打开文件不会用到，浪费系统资源，无法卸载
  
  - 开始执行守护进程核心工作
  
  - 守护进程退出处理程序模型

- 创建守护进程
  
  - ```c
    #include<stdio.h>
    #include<unistd.h>
    #include<sys/types.h>
    #include<sys/stat.h>
    #include<fcntl.h>
    #include<string>
    #include<stdlib.h>
    #include<signal.h>
    #include<sys/time.h>
    #include<time.h>
    #define _FILE_NAME_FORMAT_  "%s/log/mydaemon.%ld"  //定义文件格式化
    
    void touchfile(int num)
    
        char *HomeDir = getenv("HOME");
        char strFilename[256] = {0};
        sprintf(strFilename,_FILE_NAME_FORMAT_ ,HomeDir,time(NULL));
        int fd = open(strFilename,O_RDWR|O_CREAT , 0666);
        if(fd < 0)
        {
            perror("open error");
            exit(1);    
        }
        close(fd;)
    }
    
    
    int main()
    {
        //创建子进程，父进程退出
        pid_t pid = fork();
        if(pid > 0)
        {
            exit(1);
        }
        //当会长
        setsid();
        //设置掩码
        umask(0);
        //切换目录
        chdir(getenv("HOME"));
        //关闭文件描述符
        //close(1),close(2),close(3)
        //执行核心逻辑
        struct itimerval myit = {{60,0},{1,0}};
        setitmer(ITIMER_REAL,&myit,NULL);
        struct sigaction act;
        act.sa_flags = 0;
        sigemtyset(&act.sa_mask);
        act.sa_handler = touchfile;
        sigaction(SIGALRM,&act,NULL);
        while(1)
        {
    
        }
        //退出
        return 0;
    
    }
    
    ```

- 扩展
  
  - 通过nohup指令也可以达到守护进程创建的效果
  
  - nohup cmd [> 1.log]&
    
    - nohup 指令会让cmd收不到SIGHUP信号
    
    - & 代表后台运行
