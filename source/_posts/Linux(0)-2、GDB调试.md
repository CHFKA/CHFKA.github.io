---
title: GDB调试
date: 2021-06-06 12:02:58
tags: linux
---

# 介绍  

​	这是在linux环境下使用GDB调试使用。

<!-- more -->  

# GDB调试

## 使用

### 命令

- 编译时后缀-g
- 启动 ： gdb app（可执行程序名）
  - r(un) : 执行一遍程序
    - run  (参数1)...
  - start: 执行到主程序前
  - n(ext): 往下执行一步
  - s(tep): 往下执行一步,可以执行到函数内部，库函数不能进入
  - q(uit): 退出
  - set : 设置参数  
    - 可以在调式过程中对程序参数设值，如循环变量
  - list 显示10代码  回车后继续显示10行
    - l 文件名:行号
  - b(reak)  (行数):  设置断点
    - b  (函数名)：对单词所在行数设置断点
    - b *.c:(行数)：可以指定文件 
    - info b 断点信息
    - d (断点序号) ：删除断点
    - 条件断点  b （行数） if（条件）
  - c(ontinue)：继续执行
  - p(rint) : 打印变量信息
    - ptype 查看变量类型
  - display  (参数) ：追踪，变量变化时显示
    - info display  显示display设置信息
    - undisplay：删除显示变量

### 跟踪core

- 设置生成core：ulimit	-c	(大小设置)
- 使用：gdp （可执行程序名） core   
  - where  显示错误行数
- 设置core文件在/proc/sys/kernel/core_pattern
  - %e ----可执行程序名字
  - %t  ----添加core文件生成时的unix时间
- echo ”core-%e-%t“>/proc/sys/kernel/core_pattern  
  - 需要权限  先获取权限  sudo  （管理员名）
  - gdb （可执行程序名） core-(可执行程序名)-(unix时间)
