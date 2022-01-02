---
title: 库的制作与使用
date: 2021-06-04 17:46:33
tags: C++
---

# 介绍  

​	这是在linux环境下制作库和使用的过程。

<!-- more -->  

# 库制作与使用

## 静态库---.a后缀名

### 制作步骤

- 编译为.o文件
  - gcc -c *.c -Iinclude/（头文件路径）

- 将.o文件打包
  - ar rcs libCalc.a(库文件名) *.o
  - nm(命令)   查看文件信息
- 将头文件与库一起发布

### 使用

- 编译时需要加静态库名(记得路径)，-I 包含文件
  - gcc main.c -o  app(执行文件名) -I include/(头文件路径)  -L lib/(库文件路径) -lCalc(库文件名省去lib) 

## 动态库---.so后缀名

### 制作步骤

- 编译与位置无关的代码，生成.o，关键参数-fPIC
  - gcc -fPIC -c *.c -I include/
- 将.o文件打包  关键参数-shared
  - gcc -shared -o libCalc.so  *.o
- 将库与头文件一起发布

### 使用

- -L 指定动态库路径 -l指定库名
  - gcc main.c -o newapp(可执行文件名) -I include/(头文件路径)  -L lib/(库文件路径)  -lCalc(库名)
- 环境变量配置--在执行时能找到动态库
  - ldd newapp(可执行文件名)  查看链接
  - 拷贝到系统的库路径下（不推荐）
  - 使用export LD_LIBRARY_PATH=(路径名):$LD_LIBRARY_PATH （不特别推荐使用）
  - 1.使用sudo vim /etc/ld.so.conf  2.在文件中配置共享库路径  3.保存后输入sudo ldconfig