---
title: 进程间通信
date: 2021-06-09 11:44:02
tags: linux
---

# 介绍  

​	

<!-- more -->  

# 进程间通信

## IPC概念

- interProcess Communication 进程间通信 ，通过内核提供的缓冲区进行数据交换的机制
- 通信方式：
  - pipe 管道
  - fifo  有名管道
  - mmap  文件映射共享IO--速度最快
  - 本地socket  -- 最稳定
  - 信号  携带信息量最小
  - 共享内存
  - 消息队列

## pipe 管道

- 半双工通信---同一个时刻只能一方发送

- 管道函数：

  ```c
  int pipe(int pipefd[2]);
  ```

  - pipefd 读写文件描述符 ， 0 -读 1- 写
  - 返回值 失败返回-1，成功返回 0.

- 例子

  ```c
  #include<stdio.h>
  #include<unistd.h>
  
  int main()
  {
      int fd[2];
      pipe(fd);
      pid_t pid = fork();
      if(pid == 0)
      {
          write(fd[1],"hello" ,5);
      }else if(pid > 0)
      {
          char buf[12] = {0};
          int ret = read(fd[0],buf,sizeof(buf));
          if(ret > 0)
          {
              write(STDOUT_FILENO,buf,ret);
          }
      }
      return 0;
  }
  
  ```

- 父子进程实现ps-grep

  ```c
  #include<stdio.h>
  #include<unistd.h>
  
  int main()
  {
      int fd[2];
      pipe(fd);
      pid_t pid = fork();
      if(pid == 0)
      {
          //关闭读端
          close（fd[0]）;
          //重定向
          dup2(fd[1],STDOUT_FILENO);//标准输出重定向到管道写端
          //excelp
          excelp("ps","ps","aux",NULL);
      }else if(pid > 0)
      {
          //关闭写端
          close(fd[1]);
          //重定向,标准输入重定向到管道读端
          dup2(fd[0],STDIN_FILENO);
          //excelp
          exclp("grep","grep","bash",NULL);
      }
      return 0;
  }
  
  ```

  - 不关闭读端或写端时，会认为还存在数据要发送或写入，造成阻塞。

- 读管道

  - 写端全部关闭 -- read 读到 0，相当于读到文件末尾
  - 写端没有全部关闭
    - 有数据  -- read 读到数据
    - 没有数据 -- read 阻塞 fcntl函数可以更改非阻塞

- 写管道

  - 读端全部关闭  -- 产生一个信号SIGPIPE ，程序异常终止
  - 读端未全部关闭
    - 管道已满  --  write阻塞
    - 管道未满  --  write正常写入

- 计算管道大小 -- 512*8

  - ```c
    long fpathconf(int fd, int name);
    ```

  - ulimit -a

- 管道的优缺点

  - 优点
    - 简单
  - 缺点
    - 只能有血缘关系的进程通信
    - 父子进程单方向通信，如果需要双向通信，需要创建多根管道

## FIFO

​		命名管道，实现无血缘关系进程通信。

- 创建一个管道的伪文件

  - mkfifo myfifo 命令创建

  - ```c
    int mkfifo(const char* pathname,mode_t mode);
    ```

  - 内核会针对fifo文件开辟一个缓冲区从，操作fifo文件，可以操作缓冲区，实现进程间通信--文件读写

- 打开fifo文件的时候吗，read端会阻塞等待write端open，write端同理，也会阻塞等待另一端打开

## mmap-- 共享映射区

- 创建映射区

  ```c
  void *mmap(void *addr,size_t length,int prot ,int flags ,int fd,off_t offset);
  ```

  - addr  -- 传NULL
  - length -- 映射区的长度
  - port  
    - PROT_READ  可读
    - PROT_WRITE 可写
  - flags
    - MAP_SHARED 共享的，对内存的修改会影响源文件
    - MAP_PRIVATE 私有的
  - fd 文件描述符 ，open 打开一个文件
  - offset  偏移量
  - 返回值
    - 成功  返回 可用内存首地址
    - 失败  MAP_FAILED

- 释放映射区

  ```c
  int munmap(void *addr,size_t leength);
  ```

  - addr 传mmap的返回值
  - length mmap创建的长度
  - 返回值
    - 成功返回0
    - 失败返回-1

- 注意事项

  - 不能改变mem变量的地址
  - 文件的大小对映射区操作有影响，尽量避免对mem越界操作
  - offset必须是4k的整数倍
  - 如果文件描述符先关闭，对mmap映射没有影响
  - open的时候可以创建一个文件来创建映射区，但大小不能为0
  - open文件不可以选择只选择O_WRONLY，否则没有权限
  - 当选择MAP_SHARED的时候，open文件选择O_RDONLY,prot不可以选择PROT_READ|PROT_WRITE,因为没有权限对文件进行修改。SHARED的时候要<= open文件的权限

- 实现父子进程间的通信

  ```c
  #include<stdio.h>
  #include<unistd.h>
  #include<sys/types.h>
  #include<sys/stat.h>
  #include<sys/mman.h>
  #include<fcntl.h>
  #include<sys/wait.h>
  int main()
  {
      //创建映射区
      int fd = open("mem.txt",O_RDWR);
      int *mem = mmap(NULL,4,PROT_READ|PROT_WRITE,MAP_SHARED ,fd , 0);
      if(mem == MAP_FAILED)
      {
          return -1;
      }
      
      pid_t pid = fork();
      if(pid == 0)
      {
          *mem = 100 ;
          sleep(3);
      }
      else if(pid > 0)
      {
          sleep(1);
          *mem = 1001 ;
          wait(NULL);
      }
      munmap(mem,4);
      close(fd);
      return 0;
  }
  ```

- 匿名映射

  可以创建一个临时的映射区域,不用创建一个文件。

  - MAP_ANON  MAP_ANONMOUS这两个宏在有些unix系统没有
    - 可以在/dev/zero ，可以随意映射
    - /dev/null ，一般错误信息重定向到这个文件中

  ```c
  #include<stdio.h>
  #include<unistd.h>
  #include<sys/types.h>
  #include<sys/stat.h>
  #include<sys/mman.h>
  #include<fcntl.h>
  #include<sys/wait.h>
  int main()
  {
      //创建映射区
      int fd = open("mem.txt",O_RDWR);//mem.txt 创建的一个用来做映射的文件
      int *mem = mmap(NULL,4,PROT_READ|PROT_WRITE,MAP_SHARED|MAP_ANON ,-1, 0);
      if(mem == MAP_FAILED)
      {
          return -1;
      }
      
      pid_t pid = fork();
      if(pid == 0)
      {
          *mem = 100 ;
          sleep(3);
      }
      else if(pid > 0)
      {
          sleep(1);
          *mem = 1001 ;
          wait(NULL);
      }
      munmap(mem,4);
      close(fd);
      return 0;
  }
  ```

- 无血缘关系进程通信

  - 写

  ```c
  #include<stdio.h>
  #include<unistd.h>
  #include<sys/types.h>
  #include<sys/stat.h>
  #include<sys/mman.h>
  #include<fcntl.h>
  #include<sys/wait.h>
  typedef struct _Student{
  	int sid;
  	char sname[20];
  }Student;
  
  int main(int argc ,char *argc[])
  {
  	if(argc != 2)
  	{
  		return -1;
  	}
  	int fd = open(argv[1],O_RDWR|O_CREAT|O_TRUNC, 0666);
  	int length = sizeof(Student);
  	ftruuncate(fd,length);
  	
  	Student *stu = mmap(NULL , length ,PROT_READ|PROT_WRITE,MAP_SHARED,fd,0);
      if(stu== MAP_FAILED)
      {
          return -1;
      }
      
      int num = 1;
  	while(1)
  	{
  		stu->sid = num ;
          sprintf(stu->sname,"xiaoming-%03d",num++);
          sleep(1);
  	}
  	mummap(stu,length);
      close(fd);
  	
     	return 0;
  }
  ```

  - 读

  ```c
  #include<stdio.h>
  #include<unistd.h>
  #include<sys/types.h>
  #include<sys/stat.h>
  #include<sys/mman.h>
  #include<fcntl.h>
  #include<sys/wait.h>
  typedef struct _Student{
  	int sid;
  	char sname[20];
  }Student;
  int main(int argc ,char *argc[])
  {
      int fd = open(argv[1],O_RDWD);
      int length = sizeof(Student);
      Student *stu = mmap(NULL , length ,PROT_READ|PROT_WRITE,MAP_SHARED,fd,0);
      if(stu== MAP_FAILED)
      {
          return -1;
      }
      while(1)
      {
          printf("sid = %d, sname = %s\n",stu->sid,stu->sname);
          sleep(1);
      }
      munmap(stu,length);
      close(fd);
      return 0;
  }
  ```

  - 如果进程要通信，flags必须设为MAP_SHARED,数据修改不能改变映射区的内容





