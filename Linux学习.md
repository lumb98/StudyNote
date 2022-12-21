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





## 进程间通信的方式

在Linux操作系统中进程间的通信方式主要有管道、消息队列、共享内存、信号量、信号和Socket这几种。

### 管道

* 管道是一种单向通信方式，要想达到双向通信的目的需要创建两个管道。

平时我们在shell中使用的命令

```shell
ps auxf | grep mysql
```

中的`|`所代表的就算一个管道，它的作用是将前一个命令的输出作为后一个命令的输入，这里面的`|`是一个**匿名管道**，因为它并没有对应的文件，甚至没有名字，它仅存在于内存中，若想创建并命名一个管道，可以使用下面的命令：

```shell
mkfifo myPipe
```

像这这样的**命名管道**也被叫做**FIFO**。

我们可以用以下命令堆管道进行读写：

```shell
echo "hello" > myPipe    #将数据写入管道
#这是命令会在这里卡住，因为我们没有读取整个管道。我们得读了整个管道echo才能正常退出。
cat < myPipe             #读取管道里的数据
hello
```

* 可见管道通信虽然方便但是其效率偏低。

匿名管道的创建可以通过以下命令：

```shell
int pipe(int fd[2])
```

它会返回两个描述符，其中`fd[0]`是读取端的描述符，`fd[1]`是写入端的描述符。Linux操作系统在fork时会保留文件描述符，这样匿名管道就可以达到通信的目的了。



### 消息队列

消息队列可以解决管道效率低的问题。



嵌入式Linux，交叉编译







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



## Linux 文件I/O



## TTY串口

串口的配置主要分为以下几个步骤：

* open
* 设置行规程，比如波特率、数据位、停止位、校验位、RAW模式、一有数据就返回
* read/write

那么如何对这些参数进行设置呢，在Linux中有一个结构体`termios`，可以对这些进行设置。





## socket

socket是UNIX操作系统中的一种重要的通信手段，使用socket的大致流程如下：

1. 利用`int socket (int __domain, int __type, int __protocol)`创建一个套接字的描述符，实际上就是返回的int值，在这里我们要使用一个int类型的变量接收其返回值，以后我们再要使用这个接口就使用这个int变量就可以了，当创建失败时会返回-1。这一步就像是我们创建了一个变量。

   `DOMAIN`则是套接字的类型，比如是ip通信还是CAN通信或者本地通信。

   `TYPE`是通信的类型，一般都使用`SOCK_STREAM`,的流式通信方法，

   `PROTOCOL`这个参数一般会被前两个参数唯一指定，这里我们一般传入0，代表采用默认方式。、

2. 创建完socket后我们要将其与一个自己的地址进行绑定，这一步操作也叫**命名**实际上是给ip地址等命名，这里就需要用到`int bind (int __fd, __CONST_SOCKADDR_ARG __addr, socklen_t __len)`当这个函数返回-1时则表示绑定失败并设置errno，成功时返回0。事实上关于绑定这一步通常只有服务端会做这一步，而客户端通常是匿名访问的，即可忽略该步骤。

   `FD`该参数就是我们第一步调用函数的返回值，这一步操作的目的也是将该值与目标地址进行绑定。

   `ADDR`该参数是目标地址的结构体，不同的通信方式结构体不同。

   `LEN`该参数是ADDR结构体的大小，一般我们直接`sizeof(ADDR)`即可。

3. 绑定完socket，现在服务器还无法接收客户端的请求，需要调用`int listen (int __fd, int __n)`进行监听，当它成功时会返回0，失败则会返回-1并设置errno。

   `FD`socket文件标识符。

   `N`内核监听队列的最大长度。

4. 服务器开始监听客户端的请求了，此时客户端就可以开始发起连接了，调用`int connect (int __fd, __CONST_SOCKADDR_ARG __addr, socklen_t __len)`函数就可以使客户端向服务端发起连接。该函数调用成功时会返回0，失败时会返回-1，一旦连接成功了，输入的标识符就唯一的标识了这个连接，我们就可以通过对这个标识符进行读写来达到发送和接受的目的。

   `FD`socket文件标识符

   `ADDR`连接目标的地址结构体，

   `LEN`连接目标的地址结构体的大小，一般`sizeof(ADDR)`。

5. 现在一个客户端向服务端正在listen()的地址发起了一个连接，那么这个连接的地址会被放入内核的监听队列，此时我们可以调用`int accept (int __fd, __SOCKADDR_ARG __addr,socklen_t *__restrict __addr_len)`,当accept成功时，会返回一个新连接的socket，该socket唯一地标识了被接受的这个连接，服务端可以通过读写该socket来与被接受的客户端进行通信。若失败则会返回-1 并设置errno。

   `FD`之前监听过的socket

   `ADDR`获取被接受客户端的连接地址的结构体指针，这里注意传入的是一个指针因为我们需要在函数内对其进行修改。

   `LEN`这里也是指针，被接受客户端的链接地址的结构体大小的指针。

6. 至此，我们在客户端和服务端都拥有了能够对其读写以达到传输数据目的的socket，接下来我们就可以用读写或者接收发送函数来对这些socket来进行读写了，但是读写函数又很多种，在这一部分就不进行详细的介绍了。

7. 最后通信结束后我们就需要将该连接关闭了，通常我们会调用`int close(int fd)`这个系统调用,但是要注意的是该系统调用实际上不是关闭连接，而是将FD的引用计数减一，当它的引用计数减为0时才会关闭该连接，一次fork系统调用默认将使父进程中打开的socket的引用计数加1，因此我们必须在父进程和子进程中都对该socket执行close调用才能将连接关闭。

   如果无论如何都要关闭连接，可以调用`int shutdown(int sockfd,int howto);`系统调用，该系统调用的第二个参数可以是`SHUT_RD\SHUT_WR\SHUT_RDWR`它们的含有分别是关闭该socket的读、写和读写都关闭。该系统调用在成功时返回0，失败则返回-1并设置errno。

完成上述内容需要包含以下的头文件。

```c
#include <sys/socket.h>
#include <sys/types.h>
int socket (int __domain, int __type, int __protocol);
int bind (int __fd, __CONST_SOCKADDR_ARG __addr, socklen_t __len);
int listen (int __fd, int __n);
int accept (int __fd, __SOCKADDR_ARG __addr,socklen_t *__restrict __addr_len);
int connect (int __fd, __CONST_SOCKADDR_ARG __addr, socklen_t __len);
int shutdown(int sockfd,int howto);

#include <unistd.h>
int close(int fd);
```



### socket CAN

在CAN网络中使用socket与在Internet中使用有一定差别，

#### 首先就是第一步，需要创建套接字。

```cpp
#include <linux/can.h>
int s1 = socket(PF_CAN, SOCK_RAW, CAN_RAW);
int s2 = socket(PF_CAN, SOCK_DGRAM, CAN_BCM);
//上述两个代码都可以。
//CAN_RAW和CAN_BCM,需要用到上面的头文件。
```

#### 第二步，我们需要初始化几个结构体，用来确定我们CAN网络的“地址”——实际上是相对于Linux系统的地址，比如`ifconfig`中的can0和can1。

首先我们需要定义一个 `ifreq`结构体，这个结构体可以用来配置ip地址，激活接口，配置MTU等接口信息。关于这个结构体我们暂时不需要了解太多，我们只需要将我们can设备在`ifconfig`中的名字赋值给`ifreq.name`，然后再通过`ioctl`系统调用，指定设备到套接字，即可。具体操作如下。

```cpp
#include <stdio.h>//strcpy()
#include <string.h>
#include <net/if.h>//ifreq 
#include <sys/ioctl.h>//ioctl()
struct ifreq ifr;
strcpy(ifr.name,"can0");			//将can0 赋值给ifr.name
ioctl(sockte,SIOCGIFINDEX,&ifr);	//指定CAN设备
```

接下来要定义一个结构体`sockaddr_can`用来存放can的相关信息

```cpp
#include <linux/can.h>//sockaddr_can
#include <sys/socket.h>//AF_CAN
struct sockaddr_can can_addr;
can_addr.can_family = AF_CAN;
can_addr.can_ifindex = ifr.ifr_ifindex;
```

#### 第三步将can和socket绑定。

```cpp
bind(socket,(struct sockaddr *)&can_addr, sizeof(can_addr));
```

至此我们的准备工作就完成了，接下来就可以再can总线上进行收发了。

#### 第四步，发送与接收。

首先要定义一个结构体来确定CAN发送帧的信息，其成员变量现在先不定义，后面会对其进行定义。

```cpp
#include <linux/can.h>
struct can_frame duty_frame;
```

我们来看一下这个结构体里面都有什么。

```cpp
struct can_frame {
        canid_t can_id;  /* 32 bit CAN_ID + EFF/RTR/ERR flags */
        union {
                /* CAN frame payload length in byte (0 .. CAN_MAX_DLEN)
                 * was previously named can_dlc so we need to carry that
                 * name for legacy support
                 */
                __u8 len;
                __u8 can_dlc; /* deprecated */
        };
        __u8    __pad;   /* padding */
        __u8    __res0;  /* reserved / padding */
        __u8    len8_dlc; /* optional DLC for 8 byte payload length (9 .. 15) */
        __u8    data[8] __attribute__((aligned(8)));
};
```

常规情况下我们只需要用到 `can_id/can_dlc/data`这三个成员变量。

##### 发送

确定好这三个之后，我们就可以开始发送了。我们可以利用write()函数来进行发送。

```cpp
int nbytes;
nbytes = write(__fd, &duty_frame, sizeof(duty_frame));
if(nbytes != sizeof(duty_frame))
{
    cerr<<"Send Error duty_frame\n!"<<endl;
    return -1;
}
```

其中`__fd`就是socket的名称。

##### 接收

同样的，在我们定义完一个can_frame之后，我们可以通过read来进行帧的读取。

```CPP
can_frame data_frame;
int nbytes;
nbytes = read(__fd, &data_frame, sizeof(data_frame));
if(nbytes!=16)   //if the data is not 16 bytes, that is not we need, 
    return -1;
if(data_frame.can_id!=VESC_reveive_data_DutyCurrentRpm+motor_ID) //判断是不是我们需要的那个一帧。
    return -1;
```

### 浅谈数据拆分和大小端字节序

考虑对0x123456的存储方式

大端模式（网络字节序）

```cpp
"低地址"------>"高地址"
 0x12 | 0x34  | 0x56
```

小端模式(主机字节序)

```cpp
"低地址"------>"高地址"
 0x56 | 0x34  | 0x12
```

从上面的例子很容易看出大端模式和小端模式的差距，在一台计算机中可能这个问题并不会受太大影响，但是一旦数据需要进行传输了，那么问题也就随之而来了，以CAN总线为例，它一次可以传送8个字节，但是如果发送端和接收端不做好统一的规定，那么收到的数据也就不一定了，发送的是`0x123456`收到的可能就是`0x563412`，这将造成很大的问题。

**大小端模式各有优势：**小端模式强制转换类型时不需要调整字节内容，直接截取低字节即可；大端模式由于符号位为第一个字节，很方便判断正负。

做数据拆分时这也是我们需要做的一个事，就是确定数据发送的字节序。

数据的拆分：

```cpp
long long data=0x1234567812345678;    //定义一个8字节的数据
char BED[8],LED[8];//大端和小端数据存放的数组
//大端模式
for (int i = 0; i < sizeof(data); ++i) {
    BED[i] = (data >> 8 * (7-i)) & 0xff; 	//每次右移八个位，那么我们最开始就把最左边的（即高地址）存到了数组的低地址，这就是大端
}
//小端模式
for(int i=0;i<sizeof(data);++i){
    LED[i]=(data>>8*i)&0xff;      			//每次右移八个位，那么我们最开始就把最右边的（即低地址）字节存到了数组的低地址，这就是小端
}

```

数据的合并：

```cpp
long long recv_data;    //收到的8字节的数据
char recv_BED[8],recv_LED[8];//大端和小端数据存放的数组
//大端模式
for (int i = 0; i <sizeof(recv_data); ++i) {
    recv_data |= ((long long)recv_BED[i] << (8*(7-i)));
}
//小端模式
for (int i = 0; i < sizeof(recv_data); ++i) {
     recv_data |= ((long long)recv_LED[i] << (8 * (i)));
}
```

### can总线设置相关命令 netlink interface接口



```shell
    $ ip link set can0 type can help
    Usage: ip link set DEVICE type can
        [ bitrate BITRATE [ sample-point SAMPLE-POINT] ] |
        [ tq TQ prop-seg PROP_SEG phase-seg1 PHASE-SEG1
          phase-seg2 PHASE-SEG2 [ sjw SJW ] ]

        [ loopback { on | off } ]
        [ listen-only { on | off } ]
        [ triple-sampling { on | off } ]

        [ restart-ms TIME-MS ]
        [ restart ]

        Where: BITRATE       := { 1..1000000 }
               SAMPLE-POINT  := { 0.000..0.999 }
               TQ            := { NUMBER }
               PROP-SEG      := { 1..8 }
               PHASE-SEG1    := { 1..8 }
               PHASE-SEG2    := { 1..8 }
               SJW           := { 1..4 }
               RESTART-MS    := { 0 | NUMBER }

  - Display CAN device details and statistics:
```

下面的命令可以查看can当前的状态。

```shell
$ ip -details -statistics link show can0
```

一些常用的设置命令

```shell
6.5.2 Setting the CAN bit - timing

  The CAN bit - timing parameters can always be defined in a hardware
  independent format as proposed in the Bosch CAN 2.0 specification
  specifying the arguments "tq", "prop_seg", "phase_seg1", "phase_seg2"
 and "sjw":

    $ ip link set canX type can tq 125 prop - seg 6 \
                                phase - seg1 7 phase - seg2 2 sjw 1

  If the kernel option CONFIG_CAN_CALC_BITTIMING is enabled, CIA
  recommended CAN bit - timing parameters will be calculated if the bit -
  rate is specified with the argument "bitrate":

    $ ip link set canX type can bitrate 125000

  Note that this works fine for the most common CAN controllers with
  standard bit - rates but may * fail * for exotic bit - rates or CAN system
  clock frequencies.Disabling CONFIG_CAN_CALC_BITTIMING saves some
  space and allows user - space tools to solely determine and set the
  bit - timing parameters.The CAN controller specific bit - timing
  constants can be used for that purpose.They are listed by the
  following command :

    $ ip - details link show can0
    ...
      sja1000 : clock 8000000 tseg1 1..16 tseg2 1..8 sjw 1..4 brp 1..64 brp - inc 1

  6.5.3 Starting and stopping the CAN network device

  A CAN network device is started or stopped as usual with the command
  "ifconfig canX up/down" or "ip link set canX up/down".Be aware that
  you * must * define proper bit - timing parameters for real CAN devices
  before you can start it to avoid error - prone default settings:

    $ ip link set canX up type can bitrate 125000

  A device may enter the "bus-off" state if too much errors occurred on
  the CAN bus.Then no more messages are received or sent.An automatic
  bus - off recovery can be enabled by setting the "restart-ms" to a
  non - zero value, e.g.:

    $ ip link set canX type can restart - ms 100

  Alternatively, the application may realize the "bus-off" condition
  by monitoring CAN error frames and do a restart when appropriate with
  the command :

    $ ip link set canX type can restart

  Note that a restart will also create a CAN error frame(see also
  chapter 3.4).

  6.6 Supported CAN hardware

  Please check the "Kconfig" file in "drivers/net/can" to get an actual
  list of the support CAN hardware.On the Socket CAN project website
  (see chapter 7) there might be further drivers available, also for
  older kernel versions.
```



### IIC 结构体

`struct i2c_msg`这个结构体中有四个主要的成员变量，分别是`addr,flags,len,buf`它们分别是设备地址，读写标志，发送数据的长度，和发送的数据nei'r









## 设置开机启动



## chmod 文件权限

文件权限可以使用`ls -l`命令查看，输入命令后最前面会出现十个字母与`-`在一起从第二位开始分别表示，

1. 文件权限：首位分为“-：文件”，“d：目录”，“l：软链接”，剩下其余9位每三位为一个整体代表所属用户权限、所属组权限、其他用户权限（-：无权限，r：读权限，w：写权限，x：执行权限）。
2. 文件引用次数
3. 文件所属用户
4. 文件所属组
5. 文件大小
6. 文件最后更改日期（若创建后未更改则为创建日期）
7. 文件名

其中字母的含义和其代表的数字分别为

| 字母 | 含义     | 二进制 | 十进制 |
| ---- | -------- | ------ | ------ |
| `r`  | 读取权限 | 100    | 4      |
| `w`  | 写权限   | 010    | 2      |
| `x`  | 执行权限 | 001    | 1      |

