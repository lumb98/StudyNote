# Linux 学习





## 进程

C程序的启动方式

利用启动历程启动。这个历程放在lib/libc/*.so中。编译器再编译时会将启动历程编译到可执行文件里。

启动历程的作用

启动历程可以将命令行参数传递给main函数中的`argc`和`argv`，并且搜集环境信息构建环境表，并传递给main函数，还有等级进程的终止函数





程序的终止方式

正常终止

1. 通过``return``
2. 通过调用``exit()``                                    //实际上这个函数是标准库函数，它在帮我们调用完终止函数后又去调用了`_exit()`或`_Exit`()的系统调用
3. 调用`_exit()`或`_Exit`()(系统调用)        //利用这种方式终止程序，不会调用终止函数。
4. 最后一个线程从其启动例程返回
5. 最后一个线程调用`pthread_exit`

异常终止

1. 调用`abort`
2. 接受到一个信号并终止（常用）
3. 最后一个线程对取消请求做处理响应

进程返回

1. 通常程序运行成功返回0，否则返回非0.
2. 再shell中可以利用`echo $?`查看进程返回值。

自定义进程的终止函数

```c
#include <stdlib.h>
int atexit(void (*function)(void));//可以等级多个函数，登记时采用栈的方式，先登记后调用，
```



|  进程终止方式的区别  | return | exit() | _exit()\_Exit() |
| :------------------: | :----: | :----: | :-------------: |
| 是否刷新标准I/O缓存  |   是   |   是   |       否        |
| 是否自动调用种植函数 |   是   |   是   |       否        |



### PS指令

可以利用该命令查看进程ID

```bash
ps 					#显示信息
ps -ef                #显示详细信息
ps -ef | more
ps -aux  			#查看占用CPU 和内存  
```

PPID为当前进程的父进程。

<img src="C:\Users\WaitingForWind\OneDrive\学习相关\研究生\学习笔记\image-20220409110454046.png" alt="PS命令输出信息" style="zoom:67%;" />



进程常见状态

<img src="C:\Users\WaitingForWind\OneDrive\学习相关\研究生\学习笔记\image-20220409110612658.png" alt="进程常见状态" style="zoom:67%;" />

STAT列值为S 为可中断的等待，而D则为不可中断等待。

### 进程标识

![image-20220409113210883](C:\Users\WaitingForWind\OneDrive\学习相关\研究生\学习笔记\image-20220409113210883.png)





### 僵尸进程的回收

可以利用`wite()`和`witepid()`两个参数





### exec函数

`exec`函数，如果调用成功，会把当前进程的正文段即代码段，替换成入口参数函数的代码，包括堆栈等，实际上就说让这个进程去执行另外一个程序了。

若执行成功不会返回任何值，因为已经开始执行新的程序了，如果出错会返回-1。

```cpp
#include <unistd.h>
int execl(const char *pathname, const char *arg0, ... /* (char *)0 */ );

int execv(const char *pathname, char *const argv[]); 

int execle(const char *pathname, const char *arg0, .../* (char *)0, char *const envp[] */ );

int execve(const char *pathname, char *const argv[], char*const envp[]);

int execlp(const char *filename, const char *arg0, ... /*(char *)0 */ );

int execvp(const char *filename, char *const argv[]);

int fexecve(int fd, char *const argv[], char *const envp[]);
```

第5位字母含义
l：表示参数传递为逐个列举方式
execl、execle、execlp
v：表示参数传递为构造指针数组方式
execv、execve、execvp
第6位字母含义
e：可传递新进程环境变量
execle、execve
p：表示系统会自动从环境变量“$PATH”所指出的路径中查找可执行程序。
execlp、execvp



`execvp`是系统调用





### system 函数

`system`实际上就是调用`exec`函数，它首先调用fork函数创建一个新的进程，然后让新的进程运行`exec`函数，将`system`的参数传给`exec`。

```cpp
#include <stdlib.h>
int system(const char *command);
```



## 线程









### 线程的创建

```cpp
#include <pthread.h>
int pthread_create(pthread_t *restrict tidp,
                  const pthread_attr_t *restrict attr,
                   void *(*start_rtn)(void*),void *restrict arg);
//成功返回0，否则返回错误编号。
```

`tidp`:线程标识符指针

`attr`:线程属性指针

`start_rtn`:线程运行函数的起始地址

`arg`:传递给线程运行函数的参数的指针。其实这个参数就是前面的那个函数的入口参数的指针，这个参数有时候需要**多个参数输入**，这时候就可以将变量写成一个**结构体**，然后传递指针一起传入。



在多线程编程中尽量使用局部变量，因为线程在运行事，会在栈中申请自己的空间，不会出现线程不安全的情况。



### 线程的终止





#### 主动终止

线程的执行函数中调用return语句

调用 void pheread_exit(void *retval)

```CPP
void pheread_exit(void *retval);
```



#### 被动终止

同一个进程中的其他线程调用 int phtread_cancel(pthread_t tid);

```cpp
int phtread_cancel(pthread_t tid);
```



#### 线程的清理和控制函数

```cpp
#include <pthread.h>
void pthread_cleanup_push(void (*rtn)(void *),void* arg);
/*
* rtn:清理函数指针
* arg：调用清理函数传递的参数
*/
void pthread_cleanup_pop(int execute);
/*
* execute: 值为1时执行线程清理函数，值0时不执行线程清理函数。
*/
```

