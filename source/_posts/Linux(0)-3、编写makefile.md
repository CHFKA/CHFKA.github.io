---
title: 编写makefile
date: 2021-6-5 01:13:17
tags: makefile
---

# 介绍  

​	这是在linux环境下编写makefile的要点和自己编写的例子。

<!-- more -->  

# 编写makefile

## 三要素

- 目标
- 依赖
- 规则命令

## 写法

### 第一版

目标：依赖

tab键 规则命令

app(可执行文件) ： *.c

​	gcc -o  app -I include/  *.c

- 缺点：如果更改其中一个文件是，所有的源码都重新编译

### 第二版

- 定义变量---ObjFiles=*.o

- 变量的使用---$(ObjFiles)  $(变量名)

- makefile的隐含规则：默认处理第一个目标

  ```makefile
  ObjFiles=*.o
  app(可执行文件) :$(ObjFiles)
  ​	gcc -o  app -I include/  *.o
  *.o： *.c（代表例子）
  ​	gcc -c  *.c -I include/
  ```

### 第三版

#### 函数

- wildcard 可以进行文件匹配
- patsubst 内容的替换

```makefile
#get all .c files
SrcFiles =$(wildcard *.c)

#all .c files --> .o files
ObjFiles=$(patsubst %.c,%.o,$(SrcFiles))

app(可执行文件) :$(ObjFiles)
​	gcc -o  app -I include/  $(ObjFiles)

*.o： *.c  （代表例子）
​	gcc -c  *.c -I include/
```

#### 变量

- $@  代表目标
- $^    代表全部依赖
- $<    第一个依赖
- $?     第一个变换的依赖
- clean 清理命令
  - @不输出该条规则
  - ’-‘ 报错后继续执行

```makefile
#get all .c files
SrcFiles =$(wildcard *.c)

#all .c files --> .o files
ObjFiles=$(patsubst %.c,%.o,$(SrcFiles))

app(可执行文件) :$(ObjFiles)
​	gcc -o  app -I include/  $(ObjFiles)

#模式匹配规则
%.o： %.c  （代表例子）
​	gcc -c  $<  -I include/
clean:
	@rm -f *.o
	rm -f app
```

### 第四版

- clean存在歧义时  使用.PHONY

```makefile
#get all .c files
SrcFiles =$(wildcard *.c)

#all .c files --> .o files
ObjFiles=$(patsubst %.c,%.o,$(SrcFiles))

all:app

app(可执行文件) :$(ObjFiles)
​	gcc -o app -I include/  $(ObjFiles)

#模式匹配规则 $< 这样的变量只能在规则中
%.o： %.c  （代表例子）
​	gcc -c  $<  -I include/

.PHONY:clean all

clean:
	@rm -f *.o
	rm -f app
```

### 补充

- 使用其他名字的makefile

  - make -f （文件名）

  - 例子

    ![makefile例子](makefile例子.png)

