---
title: 进程控制
date: 2021-06-05
tags: linux
---

# 介绍  

​	

<!-- more -->  

# 文件IO

## open

- 查看man 2 open

  - pathname： 文件名

  - flags 

    - O_RDONLY  只读   

    - O_WDONLY  只写   

    - O_RDWR  读写   

    - 可选项

      ```c
      O_RDWR|O_APPEND
      ```

      - O_APPEND  追加
      - O_CREAT  创建文件
        - O_EXCL与O_CREAT 一起使用，如果文件存在，则报错
        - mode权限位，最终（mode&~umask）
      - O_NONBLOCK  非阻塞

  - 返回值 ： 返回最小的可用文件描述符  失败返回-1，设置errno

```c
int open(const cahr* pathname, int flags);
int open(const cahr* pathname, int flags,mode_t mode);
```



## close

- ```c
  int close()
  ```

- fd open 打开的文件描述符

- 返回值：成功返回0 失败返回-1，设置errno



## read读

- ```c
  ssize_t read(int fd, void* buf,size_t count);
  ```

  - fd 文件描述符‘
  - buf 缓冲区
  - count 缓冲区大小
  - 返回值
    - 失败返回-1，设置errno
    - 成功返回读取到的大小
    - 0代表读到文件末尾
    - 非阻塞的情况下read返回-1，但此时需要判断errno的值



## Write写

- ```c
  ssize_t read(int fd, void* buf,size_t count);
  ```

  - fd 文件描述符‘
  - buf 缓冲区
  - count 缓冲区大小
  - 返回值
    - 失败返回-1，设置errno
    - 成功返回写入的字节数
    - 0，代表未写入

- 实现一个cat功能：读文件，输出到标准输出

  ```c
  #include<stdio.h>
  #include<unistd.h>
  #include<sys/stat.h>
  #include<sys/types.h>
  #include<fcntl.h>
  
  int main(int argc, char *argv[])
  {
      if(argc != 2)
      {
          return -1;
      }
          
      int fd = open(argc[1],O_RDONLY);
      char buf[256];
      while(ret)
      {
          int ret = read(fd, buf ,sizeof(buf));
      
     		write(STDOUT_FILENO, buf , ret);
      }
      
      
      close(fd);
      
      return 0;
  }
  ```

##  lseek - 移动文件读写位置

### 使用

- off_t lseek(int fd, off_t offset,intg whence);

  - fd  文件描述符

  - offset 偏移量

  - whence 

    - SEEK_SET 文件开始位置
    - SEEK_CUR 当前位置
    - SEEK_END 结尾

  - 返回值

    - 成功  返回当前位置到开始的长度
    - 失败  返回-1.设置errno

```c
#include<stdio.h>
#include<unistd.h>
#include<sys/stat.h>
#include<sys/types.h>
#include<fcntl.h>

int main(int argc, char *argv[])
{
	if(argc != 2)
    {
        return -1;
    }
    
	int fd = open(argc[1],O_RDONLY);
    write(fd,"helloworld",11);
    //文件读写位置此时到末尾
    //需要移动读写位置
    lseek(fd, 0 , SEEK_SET);
    char buf[256] = {0};
    
    int ret = read(fd,buf,sizeof(buf));
    
    if(ret){
        write(STDOUT_FILENO,buf,ret);//STDIN_FILENO.STDERR_FILENO
    }
	
	
    return 0;
}
```



### 作用

- 移动文件读写位置

- 计算文件大小

  - ```c
    #include<stdio.h>
    #include<unistd.h>
    #include<sys/stat.h>
    #include<sys/types.h>
    #include<fcntl.h>
    int main(int argc, char *argv[])
    {
        if(argc != 2)
        {
            return -1;
        }
        int fd = open(argc[1],O_RDONLY);
        
        int ret = lseek(fd,0,SEEK_END);
        
        close(fd);
        
        return 0;
    }
    ```



- 拓展文件

  - ```c
    #include<stdio.h>
    #include<unistd.h>
    #include<sys/stat.h>
    #include<sys/types.h>
    #include<fcntl.h>
    int main(int argc, char *argv[])
    {
        if(argc != 2)
        {
            return -1;
        }
        int fd = open(argc[1],O_WDONLY|O_CREAT，0666);
        
        int ret = lseek(fd,1024,SEEK_END);
        //至少写一次才能保存拓展文件
        write(fd,"a",1);
        
        close(fd);
        
        return 0;
    }
    ```

## 阻塞的概念

- read函数在读设备或者读管道，或者网络的时候发生阻塞现象

  - /dev/ttey为输入输出设备对应文件

  - ```c
    #include<stdio.h>
    #include<unistd.h>
    #include<sys/stat.h>
    #include<sys/types.h>
    #include<fcntl.h>
    int main(int argc, char *argv[])
    {
        int fd = open("/dev/ttey",O_RDWR);
        char buf[256];
        int ret = 0;
        while(1)
        {
            ret = read(fd, buf , sizeof(buf));
        }
    	return 0;
    }
    ```

    ​	可以在open的参数中加入可选项O_NONBLOCK ，使得程序在读写时不阻塞。

- fcntl函数

  - 设置非阻塞

  - ```c
    #include<stdio.h>
    #include<unistd.h>
    #include<sys/stat.h>
    #include<sys/types.h>
    #include<fcntl.h>
    int main(int argc, char *argv[])
    {
        int fd = open("/dev/ttey",O_RDWR);
        
        int flags = fcntl(fd,F_GETFL);
        flags |= O_NONBLOCK;
        fcntl(fd,F_GETFL,flags);
        
        char buf[256];
        int ret = 0;
        while(1)
        {
            ret = read(fd, buf , sizeof(buf));
        }
    	return 0;
    }
    ```

    

