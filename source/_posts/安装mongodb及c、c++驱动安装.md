---
title: 安装mongodb及c、c++驱动安装
date: 2021-07-04 18:27:05
tags: mongodb
---

# 介绍  

​	这是在linux环境下安装mongodb及其c/c++驱动的步骤。

<!-- more -->  

# 安装mongodb

​		使用是ubuntu 20.4版本，更新源后可以直接sudo apt-get 安装

```
sudo apt-get update
sudo apt-get install mongodb
```



# 安装c/c++驱动

​		安装驱动比较麻烦，要依赖的库比较多

## cmake 安装

```
sudo apt-get install cmake
```

## 依赖库安装及驱动安装

参照https://blog.51cto.com/juestnow/1950039

# 2021年7月10日更新--官方教程

推荐从下面链接下载

https://github.com/mongodb/mongo-cxx-driver/releases

安装教程

http://mongocxx.org/mongocxx-v3/installation/linux/

按照教程里的步骤进行先安装c驱动  再安装c++驱动
