---
title: 线程
date: 2021-06-12
tags: linux
---

# 介绍  

​	

<!-- more -->  

# 线程

## 线程相关概念

- 线程man page安装
  
  - ```c
    sudo apt-get installmanpages-posix-dev
    ```

- 概念：
  
  - 轻量级的进程，一个进程内部可以有多个线程，默认情况下一个进程只有一个线程
  
  - 线程时最小的执行单位，进程时最小的系统资源分配单位
  
  - 内核实现都是通过clone函数实现的
  
  - 线程也有自己的PCB
  
  - 使用ps -Lf pid  查看指定线程的lwp号
  
  - malloc和mmap申请的内存可以被其他线程释放
  
  - 可以创建(cpu核数*2 + 2)个线程

## 线程共享资源

- 文件描述符表

- 每种信号的处理方式

- 当前工作目录

- 用户ID和组

- 内存地址空间(.text/.data/.bss/heap/共享库)

## 线程非共享资源

- 线程id

- 处理器线场和栈指针(内核栈)

- 独立的栈空间(用户空间栈)

- errno变量

- 信号屏蔽字

- 调度优先级

## 线程优缺点--优点突出

- 优点：
  
  - 提高程序并发性
  
  - 开销小
  
  - 数据通信、共享数据方便(包括全局变量)

- 缺点
  
  - 库函数，不稳定
  
  - 调试、编写困难
  
  - 对信号支持不好

## 创建一个线程

- 创建函数
  
  ```c
  int pthread_create(pthread_t *thread,const pthread_attr_t *attr,void *(*start_routine)(void *),void *arg);
  ```
  
  - thread 线程id，传出参数
  
  - attr代表线程的属性
  
  - 函数指针
  
  - arg 线程执行函数的参数
  
  - 需要链接-pthread库

- 例子
  
  ```c
  #include<stdio.h>
  #include<uinstd.h>
  #include<pthread.h>
  
  void* thr(void *arg)
  {
      int num = (int)arg;
      printf("num  %d",num);
      printf("I am a thread %d\n",pthread_self());
      return NULL;
  }
  
  int main()
  {
      pthread_t tid;
      int i = 1;
      pthread_creat(&tid,NULL,thr,(void *)i);
      printf("I am a main thread %d\n",pthread_self());
      sleep(1);
      return 0;
  
  }
  ```

## 线程退出

- ```c
  pthread_exit(NULL);
  ```

- 注意：
  
  - 在线程可以使用pthread_exit(NULL);
  
  - 在线程中使用return
  
  - exit代表退出整个进程

## 线程回收--阻塞等待回收

- 函数
  
  ```c
  int pthread_join(pthread_t thread,void **retval);
  ```
  
  - thread 创建的时候传出的第一个参数
  
  - retval  线程的退出信息

## 杀死线程

- 函数
  
  ```c
  int pthread_cancel(pthread_t thread);
  ```
  
  - 被该函数杀死后线程退出信息为-1
  
  - 强行设置取消点
  
  ```c
  int pthread_testcancel();
  ```

## 线程分离

- 线程若有该机制，将不会产生僵尸进程

- 线程分离状态：指定该状态，线程主动与主控线程断开关系。线程结束后，其退出状态不由其他线程获取，而直接自己自动释放,不需要用pthread_join函数

- 函数
  
  ```c
  int pthread_detach(pthread_t thread);
  ```

## 线程属性设置分离

- 初始化线程属性
  
  ```c
  int pthread_attr_init(pthread_attr_t *attr);
  ```

- 销毁线程属性
  
  ```c
  int pthread_attr_destroy(pthread_attr_t *attr);
  ```

- 设置属性分离
  
  ```c
  int pthread_attr_setdetachstate(pthread_attr_t *attr,int detachstate);;
  ```
  
  - attr init 初始化的属性
  
  - detachstate
    
    - PTHREAD_CREATE_DETACHED  线程分离
    
    - PTHREAD_CREATE_JOINABLE  允许回收

## 线程同步--需要加锁

- 协调步骤，顺序执行

- 数据混乱原因
  
  - 资源共享
  
  - 调度随机
  
  - 线程间缺少必要的同步机制

## 互斥量 -- mutex、 建议锁

- 初始化
  
  ```c
  int pthread_mutex_init(pthread_mutex_t *restrict mutex,
      const pthread_mutexattr_t *restrict attr);
  ```
  
  - restrict 约束该块内存区域对应的数据，只能通过后面的变量进行访问和修改
  
  - mutex 互斥量  -- 锁
  
  - attr  互斥量的属性，可以不考虑，传NULL
  
  - 常量初始化，此时可以使用init
    
    ```c
    pthread_mutex_t mutex = PTHREAD_MUTEX_INITALIZER;
    ```

- 销毁
  
  ```c
  int pthread_mutex_destroy(pthread_mutex *mutex)
  ```
  
  - mutex init初始化的锁

- 加锁
  
  ```c
  int pthread_mutex_lock(pthread_mutex *mutex);
  ```
  
  - mutex init初始化的锁
  
  - 如果当前未锁，成功，加锁
  
  - 如果已经加锁，阻塞等待

- 解锁
  
  ```c
  int pthread_mutex_unlock(pthread_mutex_t *mutex);
  ```

- 使用
  
  - 初始化
  
  - 加锁
  
  - 执行逻辑-- 操作共享数据
  
  - 解锁
  
  - 注意：加锁需要最小粒度，不要一直占用临界区

- 尝试加锁
  
  - ```c
    int pthread_mutex_trylock(pthread_mutex *mutex);
    ```

## 死锁

- 原因
  
  - 加锁2次
  
  - 交叉锁

- 解决办法
  
  - 每个线程申请锁的顺序要固定
  
  - 如果申请到一把锁，申请另一把的时候，申请失败，应该释放已经掌握的

## 读写锁

- 特点：读共享，写独占，写优先级高

- 状态
  
  - 未加锁
  
  - 读锁
  
  - 写锁

- 使用场景：适合读的线程多

- 初始化
  
  ```c
  int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock,
          const pthread_rwlockattr_t *restrict attr);
  //或
  pthread_rwlock_t rwlock = PTHREAD_RWLOCK_INITALIZER;
  ```

- 销毁
  
  ```c
  int pthread_rwlock_destroy(pthread_rwlock_t * rwlock);
  ```

- 加读锁
  
  ```c
  int pthread_rwlock_rdlock(pthread_rwlock_t * rwlock);
  ```

- 加写锁
  
  ```c
  int pthread_rwlock_wrlock(pthread_rwlock_t * rwlock);
  ```

- 释放锁
  
  ```c
  int pthread_rwlock_unlock(pthread_rwlock_t * rwlock);
  ```

## 条件变量---pthread_cond

    不是锁，可以引起阻塞，要和互斥量组合使用

- 超时等待
  
  - ty_sec  绝对时间，填写时time(NULL) + 600 ===> 设置超时600s

- 条件变量阻塞等待
  
  - 先释放锁 mutex
  
  - 阻塞在cond条件变量上

- 例子
  
  ```c
  #include<stdio.h>
  #include<unistd.h>
  #include<pthread.h>
  #include<stdlib.h>
  
  pthread_mutex_t mutex = PTHREAD_MUTEX_INITALIZER;
  pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
  
  int beginnum = 1000;
  
  typedef struct _ProdInfo{
      int num;
      struct _ProdInfo *next;
  }
  ProdInfo *Head = NULL;
  
  void *thr_producter(void *arg)
  {
      //负责在链表添加数据
      while(1)
      {
          ProdInfo *prod = malloc(sizeof(ProdInfo));
          prod->num = beginnum++;
          printf("----%s------self=%lu-----%d\n",_FUNCTION_,pthread_self(),prod->num);
          pthread_mutex_lock(&mutex);
          prod->next = Head;
          Head = prod;
          pthread_mutex_unlock(&mutex);
          //发起通知
          pthread_cond_signal(&cond);
          sleep(rand()%4);
      }
      return NULL;
  }
  
  
  void *thr_customer(void *arg)
  {
      ProdInfo *prod = NULL;
      while(1){
          //取链表的数据
          pthread_mutex_lock(&mutex);
          while(Head == NULL)
          {
              pthread_cond_wait(&cond,&mutex);//在此之前必须先加锁
          }
          prod = Head;
          Head = Head->next;
          printf("----%s------self=%lu-----%d\n",_FUNCTION_,pthread_self(),prod->num);
          pthread_mutex_unlock(&mutex);
          sleep(round%4);
          free(prod);
      }
  }
  
  int main()
  {
      pthread_t tid[2];
      pthread_create(&tid[0],NULL,thr_producter,NULL);
      pthread_create(&tid[1],NULL,thr_customer,NULL);
  
      pthread_join(tid[0],NULL);
      pthread_join(tid[1],NULL);
      pthread_mutex_destroy(&mutex);
      pthread_cond_destroy(&cond);
      return 0;
  }
  
  
  ```

## 信号量--sem  加强版互斥量

        信号的量的初值，决定了占用信号量的线程的个数.

- 允许多个线程访问共享资源

- 初始化
  
  ```c
  int sem_init(sem_t *sem,int pshared,unsigned int value);
  ```
  
  - sem 定义的信号量，传出
  
  - pshared
    
    - 0 代表线程信号量
    
    - 非0代表进程信号量
  
  - value 定义信号量的个数

- 销毁

- 申请信号量 ，申请成功value-- 
  
  ```c
  int sem_wait(sem_t *sem);
  ```
  
  - 当信号量为0时，阻塞

- 释放信号量 value++
  
  ```c
  int sem_post(sem_t *sem);
  ```

## 文件锁

```c
struct flock lk;
lk.l_type = F_WRLCK;
lk._whence = SEEK_SET;
lk.start = 0;
lk.l_len = 0;
if(fcntl(fd,F_SETLK,&lk) < 0)
{
    perror("get lock err");
    exit(-1);
}
```
