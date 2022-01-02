---
title:  学习windows（一）    

date: 2020-11-23    

tags: windons 
---

​         这篇是记录了自己学习windows程序设计所碰到问题记录的Blog

<!-- more -->  

## 句柄（handle）是什么  

​        通常需要用某种方式能让你访问一个资源，而这个资源通常存放在一个表结构中，但操作系统不会给你对应项的地址让你去访问其中一项，而是给你这个项在表中的下标，这样你只要知道这个下标就能指明要访问的项了，这个表（其实就是个数组）就像整齐排列的一些抽屉，你要用抽屉的时候必须得通过抽屉的把手来打开它，这个表呢就像这些抽屉一样，而这个句柄就被比喻为抽屉的把手（柄）

##   undefined reference to`__imp_GetStockObject'的处理  

​       编程环境是windows+vs code。

​		在tasks.json中的args下加入“-lwinmm”.同理当继续碰到undefined reference to这类问题可以在tasks.json文件中链接相对应库

