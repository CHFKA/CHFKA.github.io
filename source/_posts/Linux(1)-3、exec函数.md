---
title: exec函数族
date: 2021-06-08
tags: linux
---

# 介绍  

​	

<!-- more -->  

# exec函数族

- 将当前进程的.text、.data替换为所要加载的程序的.text、.data，然后让进程从新的.text第一条指令开始执行，但进程ID不变，换核不换壳。

## execl函数

- 执行其他程序

  ```c
  int execl(const char *path,const char *arg,...);
  ```

## excelp函数

- 执行程序的时候，使用PATH环境变量，执行的程序可以不用加路径

  ```c
  int execlp(const char *file,const char *arg,...);
  ```

  - flie 要执行的程序
  - arg参数列表
    - 参数列表最后需要一个NULL作为结尾，作为哨兵。
  - 返回值
    - 只有失败才返回

![exce函数族](exce函数族.png)

