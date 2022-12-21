# FreeRTOS学习

韦东山老师的讲义。

> http://rtos.100ask.net/freeRTOS%E6%95%99%E7%A8%8B/docs/chap6_semaphore/section1.html#id10



## 堆和栈

堆(heap)：就是一块空闲的内存，我们可以给他分配内存和删除内存。

栈(stack)：

SP寄存器就是栈。

函数开头，

1. 划分栈，让SP寄存器指向一段空闲的内存



## 保存现场

当发生函数调用、中断或者任务切换时，需要将当前执行的任务的CPU寄存器内容保存到栈中，在ARM-coretex-M3中我们需要将R0-R15这16个寄存器的值进行保存，中断的保存现场和任务切换的保存现场略有不同，因为任务切换时，并不知道用了哪些寄存器。

任务切换时的保存现场，需要保存所有的寄存器，因为我们不知道之前的任务使用了哪些寄存器，

函数调用时的保存现场，因为在C语言中R0、R1、R2被约定为给**子函数**传参使用的寄存器，所以不需要保存这三个寄存器。因为这三个寄存器本来就是为将要调用的函数准备的，并没有对需要被保存现场有用的数据。

硬件中断中的保存现场，（这里针对cortex-M3\M4）由硬件保存一部分如R0\R1\R2，由软件来保存一些用到的寄存器

## 前后台系统

没有操作系统的单片机中后台运行一个大的死循环`while(1){}`，这个死循环就是后台系统，前台系统就是中断。

FreeRTOS使用可剥夺型内核。

中断执行完成后，会先判断一下是否有更好优先级的任务就绪，如果有则运行该任务，如果没有则运行被中断的任务。

## FreeRTOSConfig.h文件

使用“INCLUDE_"开头的宏用来表示是能或除能FreeRTOS中相应的API函数，作用就是用来配置FreeRTOS中可选的API函数。

此外还有”config“开始的宏，这些宏可以做一些系统的基本设置，比如系统CPU的频率，是否使用协程，抢占式内核，低功耗模式，可使用的最大优先级，任务名字的字符串商都，等。

## 任务相关

### 任务状态

该系统有四种任务状态，分别是 **运行态**、**就绪态**、**阻塞态**和**挂起态**

### 任务优先级

该系统的任务优先级数字越大优先级越高，数字越低优先级越低，与UCOS相反。

### 任务的两种创建方式

任务创建有动态和静态两种方式

动态的任务创建过程内存由系统自动申请，静态的则需要自己手动申请。

### 任务堆栈

用来保存任务现场，创建任务时需要指定任务堆栈。



任务退出时记得删除任务

 ## 任务的创建和删除（动态方法）

### tskTaskControlBlock（TCB结构体）

```c
typedef struct tskTaskControlBlock
{
	volatile StackType_t	*pxTopOfStack;	/*< Points to the location of the last item placed on the tasks stack.  THIS MUST BE THE FIRST MEMBER OF THE TCB STRUCT. */
	ListItem_t			xStateListItem;	/*< The list that the state list item of a task is reference from denotes the state of that task (Ready, Blocked, Suspended ). */
	ListItem_t			xEventListItem;		/*< Used to reference a task from an event list. */
	UBaseType_t			uxPriority;			/*< The priority of the task.  0 is the lowest priority. */
	StackType_t			*pxStack;			/*< Points to the start of the stack. */
	char				pcTaskName[ configMAX_TASK_NAME_LEN ];/*< Descriptive name given to the task when created.  Facilitates debugging only. */ /*lint !e971 Unqualified char types are allowed for strings and single characters only. */
} tskTCB;
```



### 创建任务的两个核心：栈、任务结构体

其中栈的大小取决于**局部变量的大小**和**函数调用的深度**

栈是从一块空闲的内存分配出来的，是定义的一个全局数组`ucHeap[configTOTAL_HEAP_SIZE]`这里就被作为各个任务的栈空间



**任务的切换**实际上就是将当前任务的**地址赋给PC寄存器**。



### 需要用到的一些函数

| 函数                  | 描述                     |
| --------------------- | ------------------------ |
| `xTaskCreate()`       | 使用动态方法创建一个任务 |
| `xTaskCreateStatic()` | 使用静态方法创建一个任务 |
| `vTaskDelete()`       | 删除任务                 |



#### `xTaskCreate()`函数

```cpp
BaseType_t xTaskCreate(	TaskFunction_t pxTaskCode,
							const char * const pcName,
							const uint16_t usStackDepth,
							void * const pvParameters,
							UBaseType_t uxPriority,
							TaskHandle_t * const pxCreatedTask )
```

`pxTaskCode`			任务函数的函数名（地址）。

`pcName`					任务的名字

`usStackDepth`		任务堆栈大小，实际大小是 `usStackDepth`的四倍（字节）

`pvParameters`		传递给任务函数的参数。

`uxPriority`			任务的优先级，范围从0~`configMAX_PRIORITIES-1`，0是空闲任务优先级。

`pxCreatedTask`		任务句柄，任务创建成功后会返回此任务的任务句柄，这个句柄实际上就是任务的任务堆栈，此参数用来保存这个任务句柄，其他API函数可能会使用到这个句柄。

**返回值**

`pdPASS`任务创建成功

`errCOULD_NOT_ALLOCATE_REQUIRED_MEMORY`任务创建失败，因为堆内存不足。

#### `xTaskCreateStatic()`函数

```cpp
TaskHandle_t xTaskCreateStatic(	TaskFunction_t pxTaskCode,
									const char * const pcName,
									const uint32_t ulStackDepth,
									void * const pvParameters,
									UBaseType_t uxPriority,
									StackType_t * const puxStackBuffer,
									StaticTask_t * const pxTaskBuffer ) PRIVILEGED_FUNCTION;
```

`pxTaskCode`			任务函数的函数名。

`pcName`					任务的名字

`ulStackDepth`		任务堆栈大小，实际大小是 `usStackDepth`的四倍（字节）

`pvParameters`		传递给任务函数的参数。

`uxPriority`			任务的优先级，范围从0~`configMAX_PRIORITIES-1`

在静态的任务创建中需要我们

`puxStackBuffer`	任务堆栈，一般是数组，数组类型位`StackType_t`。

`pxTaskBuffer`		任务控制块。

**返回值**

`NULL`任务创建失败

`other`任务创建成功返回任务的任务句柄。



#### `vTaskDelete()`函数

```cpp
void vTaskDelete( TaskHandle_t xTaskToDelete );
```

`xTaskToDelete`	要删除的任务的任务句柄。



## 调度机制

### 优先级与状态

优先级不同

1. 高优先级的任务，优先执行，可以抢占低优先级的任务
2. 高优先级的任务不停止，低优先级的任务就永远无法执行
3. 同等优先级的任务，轮流执行：时间片轮转

状态

1. 运行态 runing
2. 就绪态 ready
3. 阻塞 blocked，等待某件事（时间、事件）
4. 暂停 suspend 挂起

怎样管理

怎么取出要运行的任务？

1. 找到最高优先级的运行态，就绪态任务，运行它

2. 如果大家平级，轮流执行：排队，链表前面的先运行，运行1个tick后乖乖地区链表尾部排队



三类链表

就绪链表`pxReadyTasksLists[] []`中的是优先级，每个优先级有一个链表。每次同优先级运行完成后会被排到该优先级的最后面

阻塞链表`pxDelayedTaskList`

挂起链表`xPendingReadyList`

### 调度方法

由谁进行调度

 * TICK中断：一般是每一毫秒一次



通过链表深入理解调度机制

* 可抢占：高优先级的任务先运行
* 时间片轮转：同优先级任务轮流执行
* 空闲任务礼让：如果由同时优先级为0的其他就绪任务，空闲任务主动放弃一次运行机会。

空闲任务礼让：空闲任务判断同优先级的`pxReadyTasksLists[tskIDLE_PRIORITY]`中是否有其他任务，如果有则会调用`taskYIELD();`让其他任务优先运行。



### 任务中途放弃几种情况

分为主动放弃和被动放弃

* 主动放弃：读取vTaskDelay，读取队列，但是没有信息
* 被动放弃：被抢占。

如果把系统设置为不抢占的模式，那么除非主动放弃否则会一直运行一个任务。





## 消息队列和互斥量

消息队列的核心是：**关中断**、**环形缓冲区**和**链表**

### 队列如何实现互斥访问

队列通过在对被保护的变量进行操作前，**关中断**，操作结束后再**开中断**的方式实现对队列内容的互斥访问。



任务读队列时，如果发现没有自己需要的数据，就会进入休眠状态，等待唤醒，当有消息进入队列时会被唤醒，

### 环形缓冲区

环形缓冲区用来写数据

```cpp
#define BUFSIZE 4
int buf[BUFSIZE];
int i=0;
while(1){
    buf[i]=1;
    i=(i+1)%BUFSIZE;//取余，保证每次超过4时会回到0，这样就实现了环形的操作，一直从0到4循环
}
```



### 链表

队列有两个链表，分别是**发送队列链表**，和**接收队列链表**（`List_t xTasksWaitingToSend`和`List_t xTasksWaitingToReceive`）。

### 消息队列发送和接收流程

首先说消息队列的读取，当一个任务要读取消息队列时，会判断消息队列中是否又消息，如果消息队列中没有消息那么它就会根据设置的参数选择下一步操作，一般情况下会进入到阻塞状态，等待一定的时间，如果在等待的时间到达之前消息到了就立即开始继续执行任务，当然前提是当前的就绪列表中没有比该任务优先级更高的任务，整个FreeRTOS都遵循这个规则，但要记住中断的最低优先级要比最高优先级的任务的优先级还要高。也就是说任何中断都会阻止任务的运行。

那么在读取消息失败后任务是如何使自己进入阻塞状态的，当消息队列中进入消息时，又是如何将任务唤醒的呢？这就要用到我们上一节中说到的俩个链表了，大致流程如下。

* 首先关中断
* 判断队列中是否有数据
  * 有数据：将数据拷贝
  * 无数据：
    * 返回错误
    * 进入休眠状态
      * 将自己添加到该队列的一个等待链表`xTasksWaitingToReceive`中
      * 然后把自己从内核中的`ReadyList[]`移动到`DelayList`中，实现休眠。
* 终于有了数据
  * 将数据进行copy
  * 唤醒因为队列已满而处于阻塞状态等待发送数据的任务，这些任务在队列的`xTasksWaitingToSend`链表中
    * 把`xTasksWaitingToSend`中的第一个任务移除
    * 把它从`DelayList`移动到`ReadyList[]`
* 超时唤醒
  * 判断是否超时
    * 超时了把超时的任务从`DelayList`移动到`ReadyList[]`。
  * 没超时什么都不做





而写（发送）数据的流程基本与此相似，只不过它判断的是队列是否是满的。



### 消息队列的创建

`xQueueCreate( uxQueueLength, uxItemSize )`该函数实际上是一个宏定义，最后被调用的函数是`QueueHandle_t xQueueGenericCreate( const UBaseType_t uxQueueLength, const UBaseType_t uxItemSize, const uint8_t ucQueueType )`

最常用的是那个较短的宏定义，我们再这里只说这个

```cpp
#define xQueueCreate( uxQueueLength, uxItemSize ) xQueueGenericCreate( ( uxQueueLength ), ( uxItemSize ), ( queueQUEUE_TYPE_BASE ) )
QueueHandle_t xQueueCreate( UBaseType_t uxQueueLength,
                            UBaseType_t uxItemSize );
```

`uxQueueLength `	：要创建的队列长度，队列的项目数

`uxItemSize`		：队列中每个项目（消息）的长度，单位是字节

返回值

其他值：	队列创建成功后返回的队列句柄

NULL：  	队列创建失败。

### 消息队列的读写

#### 消息队列写API

向消息队列发送消息的函数

```cpp
BaseType_t xQueueSend(
							  QueueHandle_t xQueue,
							  const void * pvItemToQueue,
							  TickType_t xTicksToWait
						 );

BaseType_t xQueueSendToBack(
								   QueueHandle_t	xQueue,
								   const void		*pvItemToQueue,
								   TickType_t		xTicksToWait
							   );
BaseType_t xQueueSendToToFront(
								   QueueHandle_t	xQueue,
								   const void		*pvItemToQueue,
								   TickType_t		xTicksToWait
							   );
BaseType_t xQueueGenericSend( QueueHandle_t xQueue,
                              const void * const pvItemToQueue,
                              TickType_t xTicksToWait,
                              const BaseType_t xCopyPosition )
```

前三个函数实际上最终调用的函数都是一样的，都是最后一个`xQueueGenericSend()`函数。

`xQueue`				：队列句柄，指明要向那个队列发送数据，创建队列成功以后会返回此队列的队列句柄。

`pvItemToQueue`	：指向要发送的消息，发送时会将这个消息拷贝到队列中。

`xTicksToWait`		：阻塞时间，分为三种情况：

1. 该参数为0时，当队列满的时候就会立即返回
2. 该参数为portMAX_DELAY时，会一直等待到队列有空闲的队列项。
3. 该参数为0~portMAX_DELAY之间时，会等待这么长时间后继续执行，如果再次期间队列有空闲了就直接发送。

返回值

`pdPASS`					向队列发送消息成功

`errQUEUE_FULL`		队列已满，发送消息失败

#### 队列消息的读API

```cpp
#define xQueueReceive( xQueue, pvBuffer, xTicksToWait ) xQueueGenericReceive( ( xQueue ), ( pvBuffer ), ( xTicksToWait ), pdFALSE )

 BaseType_t xQueueReceive(
								 QueueHandle_t xQueue,
								 void *pvBuffer,
								 TickType_t xTicksToWait
							);</pre>
```

`xQueue`				：队列句柄，指明要读取那个队列的数据，创建队列成功以后会返回此队列的队列句柄。

`pvBuffer`			：保存数据的缓存区，读取队列的过程中会将读取到的数据拷贝到这个缓冲区中。

`xTicksToWait`		：阻塞时间，分为三种情况：

1. 该参数为0时，当队列空的时候就会立即返回
2. 该参数为portMAX_DELAY时，会一直等待到队列有数据。
3. 该参数为0~portMAX_DELAY之间时，会等待这么长时间后继续执行，如果再次期间队列有数据了就直接存入缓冲区。

返回值

`pdTRUE`				从队列中读取数据成功

`pdFALSE`			  从队列中读取数据失败	

## 信号量

信号量实际上就是特殊的队列，其实现方式几乎和队列一致，它内部实现使用的API和逻辑也几乎一样，只有一些细节上有较小的差别，

信号量可以分为二值信号量和计数型信号量，二值信号量实际上是普通信号量的变体，它们本质上并没有区别。

二值信号量通常用来做同步，计数型信号量则通常用来计数。

## 互斥量

但是需要注意信号量和互斥锁的区别，相对于二值信号量，互斥锁拥有优先级继承机制，可以避免优先级反转的情况，

### 优先级反转

什么是优先级反转呢，假设有三个任务，分别是高中低优先级，首先低优先级先运行，因为任务需要，它获得了一个锁，但是它获得锁后被中优先级的任务抢占了，此时又有一个高优先级抢占了中优先级的任务，但是这个高优先级任务需要用到低优先级所持有的锁，因为没有锁高优先级的任务就进入了阻塞状态，而中优先级的任务却可以正常运行，现在就发生了优先级反转，中优先级的任务运行完毕，轮到低优先级的任务放开锁，高优先级的任务才能运行，可见实际上中优先级的任务比高优先级的任务更优先运行了，这就称为**优先级反转**。

### 互斥锁的优先级继承机制

当发生优先级反转时，如果使用二值信号量就只能眼睁睁看着优先级反转的情况发生了，但是倘若我们使用互斥锁，那么问题就会好解决很多了，FreeRTOS提供了一种优先级继承机制，用以改善这个问题，当出现优先级反转的时候，需要用到锁的高优先级任务会查看当前持有锁的低优先级任务，并将该低优先级任务的优先级提升到与自己一致，使其优先运行，好让它运行完后放开锁，自己就可以运行了，当然一旦它放开锁就会将其放回到它的初始优先级。



## 事件组

事件组与其他的任务间通信方式不同的是，其他的任务间通信方式在进行关键操作前都会关中断，而事件组只会关任务调度器，核心原因是事件组不支持在中断中进行操作。

事件组拥有自己的特色，它是按位来进行操作的，并且当我们等待事件组的事件时，可以选择同时等待几个位，这也是它的一个优势。

事件组设置位的任务是不会被阻塞的，因为它不像队列一样如果队列满了就不能进行写入了，它可以随时阻塞，但读取的一端就不是了，它需要阻塞到事件组满足它的要求为止，或者阻塞到设定的最大时间。

## 任务通知

相对于其他的任务间通信，任务通知是不需要创建的，直接可以使用，所有它就有更省内存和更高效的特点。



## 软件定时器

对于软件定时器FreeRTOS相对于其他实时系统不同的地方是，它在硬件定时器中写队列，让一个定时器任务来完成定时器的内容，而不是直接在软件定时器中读写，FreeRTOS的作者认为，我们不知道用户会在定时器中作什么，如果运行的函数会占用太多事件，这会破坏系统的实时性，所以我们干脆让用户的软件定时器运行在任务中，硬件定时器只设置队列。



## 延时阻塞函数

常用的延时阻塞函数有两个

* `vTaskDelay(20)`		：每到这个函数就会阻塞20ms；
* `vTaskDelayUnitil()`：需要输入一个`TickType_t `类型的变量的地址作为第一个参数，一般这个变量会从`xTaskGetTickCount()`函数获取，用以获取当前时间，再输入第二个参数，是一个整形变量，从第一个参数时刻开始，延时第二个参数的时间。

```cpp
TickType_t tStart = xTaskGetTickCount();
{
    //do some thing;
}
vTaskDelayUnitil(&tStart,20);//从开头到这里一共是20ms
```

