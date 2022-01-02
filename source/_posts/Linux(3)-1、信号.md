---
title: 信号
date: 2021-06-09 23:37:06
tags: linux
---

# 介绍  

​	

<!-- more -->  

# 信号

## 信号的基本概念

- 概念
  - 简单，不能带大量信息，满足特定条件发生
  - 内核产生信号，内核处理
  - 产生
    - 按键
    - 调用函数
    - 定时器
    - 命令 --系统api产生信号  kill函数
    - 硬件异常 
  - 状态
    - 产生
    - 递达 信号到达并且处理完
    - 未决  信号被阻塞
  - 默认处理方式
    - 忽略
    - 执行默认动作
    - 捕获
  - 4要素
    - 编号
    - 事件
    - 名称
    - 默认处理动作
      - 忽略
      - 终止
      - 终止+core
      - 暂停
      - 继续
  - 软件产生的中断，会有延迟
  - 19号信号不能捕捉，暂停，不能阻塞
  - 0号信号有特殊的含义
  
- 阻塞信号集

  将某些信号加入集合，对他们设置屏蔽，当屏蔽x信号后，再收到该信号，该信号的处理将推后(解除屏蔽后)

- 未决信号集

  - 信号产生，未决信号集中描述该信号的位立刻翻转为1， 表信号处于未决状态。当信号被处理对应位翻转回0。这一时刻往往非常短暂。
  - 信号产生后由于某些原因(主要是阻塞)，不能抵达。这类信号的集合称之为未决信号集。在屏蔽解除前，信号一直处于未决状态。
  
- kill(pid,sig) 用于发送信号。

## 信号的产生

- 系统api产生信号

  - kill函数

  ```c
  int kill(pid_t pid , int sig);
  //pid > 0 , 要发送进程ID
  
  // pid = 0，代表当前调用进程组内所有进程
  // pid = -1，代表有权限发送的所有进程
  // pid < 0 ,  代表-pid对应的组内所有进程
  // sig 对应的信号
  ```

  - raise函数---给自己发信号

  - abort函数-- 给自己发送异常信号

  - 时钟信号 -- alarm函数

    - 一次性

    - ```c
      unsigned int alarm(unsigned int seconds);
      ```

    - 定时给自己发送SIGALRM信号.
    - 返回值，上次定时剩余的描述
    - 如果参数为0，取消定时

  - setitimer函数

    - 周期性发送信号

    - ```c
      int setitimer(int which ,const struct itimerval *new_value , struct itimerval *old_value);
      ```
      
      - which
        - ITIMER_REAL 自然定时法  SIGALLRM
        - ITIMER_VIRTUAL 计算进程执行时间  SIGVTALRM
        - ITIMER_PROT 进程执行时间+调度时间  ITIMER_VIRTUAL
      - new_value 要设置的定时时间
      - old_value  原定时时间
      - 返回值  0成功  -1失败

## 信号集的操作

- 操作函数

  ```c
  int sigemptyset(sigset_t *set);//清空信号集
  int sigfillset(sigset_t *set);//填充信号集
  int sigaddset(sigset_t *set,int signum);//从集合中添加某个信号
  int sigdelset(sigset_t *set,int signum);//从集合中删去某个信号
  int sigismember(const sigset_t *set,int signum);//是否为集合里的成员
  ```

- 设置阻塞或解除阻塞信号集

  ```c
  int sigprocmask(int how, const sigset_t *set ,sigset_t *oldset)
  ```

  - how

    - SIG_BLOCK 设置阻塞

    - SIG_UNBLOCK  解除阻塞

    - SIG_SETMASK 设置set为新的阻塞信号集 

  - set 传入的信号集

  - oldset 旧的信号集，传出

- 获取未决信号集

  ```c
  int sigpending(sigset_t *set);
  ```

  - set 传出参数，当前的未决信号集

## 信号捕捉

- 防止进程意外死亡

- 捕捉函数

  ```c
  typedef void (*sighandler_t)(int);
  singhandler_t signal(int signum,sighandler_t handler);
  ```

  - signum 要捕捉的信号
  - handler 要执行的捕捉函数指针，函数应该声明 void func(int);

- 注册捕捉函数

  ```c
  int sigaction(int signum,const struct sigaction *act ,struct sigaction *oldact);
  ```

  - signum 要捕捉的信号
  - act  传入的动作
  - oldact  原动作  恢复现场

- 例子-- sigaction 捕捉 setitimer产生信号

  ```c
  #include<signal.h>
  #include<stdio.h>
  #include<unistd.h>
  #include<sys/time.h>
  void catch_sig(int num)
  {
      printf("catch %d sig\n",num);
  }
  int main()
  {
      srtuct sigaction act;
      act.sa_flags = 0;
      act.sa_handler = catch_sig;
      sigemptyset(&act.sa_mask);
      sigaction(SIGALRM,&act,NULL);
      struct itimerval myit = {{3,0},{5,0}};
      setitimer(ITIMER_REAL,&myit,NULL);
      while(1)
      {
          sleep(1);
      }
      return 0;
  }
  ```
  
- 特性

  - 捕捉什么信号，捕捉期间自动屏蔽该信号
  - 阻塞的常规信号不支持排队，产生多次只记录一次（后32个信号支持排队）
  - 进程正常运行时，默认PCB中有一个信号屏蔽字，它决定了进程自动屏蔽哪些信号。捕捉到了某个信号后,执行期间的要屏蔽的信号不是由信号屏蔽字决定，而是用sa_mask来指定，信号处理函数调用完后再恢复.

## SIGCHLD信号

- 子进程暂停或者退出的时候会发送SIGCHILD信号，我们可以通过捕捉SIGCHILD信号来回收子进程

  ```c
  #include<signal.h>
  #include<stdio.h>
  #include<unistd.h>
  #include<sys/wait.h>
  
  void catch_sig(int num)
  {
      pid_t wpid ;
      while((wpid = waitpid(-1 , NULL , WNOHANG)) > 0)
      {
          printf("wait child %d ok \n",wpid);
      }
  }
  
  int main()
  {
      int i = 0;
      pid_t pid;
      for(i = 0;i < 10;i++)
      {
          pid = fork();
          if(pid == 0)
              break;
      }
      if( i == 10)
      {
          struct sigaction act ;
          act.as_flags = 0;
          sigemptyset(&act.sa_mask);
          act.ac_handler = catch_sig;
          sigaction(SIGCHLD,&act,NULL);
          while(1)
          {
              sleep(1);
          }
      }else if(i < 10)
      {
       	
      }
      return 0;
  }
  ```

- 注意事项--当注册捕捉晚于子进程死亡时

  ```c
  #include<signal.h>
  #include<stdio.h>
  #include<unistd.h>
  #include<sys/wait.h>
  
  void catch_sig(int num)
  {
      pid_t wpid ;
      while((wpid = waitpid(-1 , NULL , WNOHANG)) > 0)
      {
          printf("wait child %d ok \n",wpid);
      }
  }
  
  int main()
  {
      int i = 0;
      pid_t pid;
      
      //在创建子进程前屏蔽SIGCHLD信号
      sigset_t myset,oldset;
      sigemptyset(&myset);
      sigaddset(&myset,SIGCHLD);
      //oldset 保留现场  设置了SIGCHLD到阻塞信号集
      sigprocmask(SIG_BLOCK,&oldset);
      
      for(i = 0;i < 10;i++)
      {
          pid = fork();
          if(pid == 0)
              break;
      }
      if( i == 10)
      {
      	sleep(2);//模拟注册捕捉晚于子进程死亡时
          struct sigaction act ;
          act.as_flags = 0;
          sigemptyset(&act.sa_mask);
          act.ac_handler = catch_sig;
          sigaction(SIGCHLD,&act,NULL);
          //解除屏蔽现场
          sigprocmask(SIG_SETMASK,&oldset ,NULL);
          while(1)
          {
              sleep(1);
          }
      }else if(i < 10)
      {
       	
      }
      return 0;
  }
  ```

  

