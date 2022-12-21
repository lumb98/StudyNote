# C语言知识点



## C 中的 void **

1. 如何看待C/C++中的指针。

```c
SomeType something;
SomeType *p = &something;
```

上面这个代码中，我是这样看的。p是一个object，它是一个指针object，它的类型（Type）是SomeType*。在64位系统里，p是8字节。

p的值，是另外一个object，即something。something的类型是SomeType。比如SomeType如果是int的话，多数系统里，SomeType是4字节。

2. 函数的parameter传递是copy by value

```c
// this is caller
{
    SomeType something;
    SomeType *p = &something;

    func(p);
}
...
...c
// this is callee
void func(SomeType *copy_p)
{
     // use copy_p
     copy_p->member = some_value;    // for example
}
```

在上面的代码中，在caller里，p是一个SomeType*指针对象，其值是SomeType对象，即something。

在callee的func()里，copy_p是另外一个对象，类型也是SomeType*指针对象，但由于函数传递argument时，是copy by value，所以，在func()里，虽然copy_p是另外一个指针对象，但是，其值（即指针所指）和caller是同一对象，即something。

因此copy_p->member = some_value时，实际操作的是something这个对象（修改其内部成员对象member）。因此当func()返回到caller时，caller的p虽然不受影响（因为是copy），但caller的p的值（即指针所指的对象，即something这个对象），是被修改了。即caller调用了func()后，something这个对象实际发生了修改，其内部的member成员对象，变成了some_value。

所以，上面的func()就被理解为，我copy了一个指针，然后我通过copy_p->去操作这个指针所指的对象。

3. 如何看待指针的指针

那么对于void **p，如何看待？

首先，将第一个*隔离出来，这样p还是一个指针，它的值（即所指）的对象是SomePointerType。用下面的代码会有助于理解。

```c
SomeType something;
SomeType *p_to_something = &something;     // SomePointerType == SomeType*
void **p = &p_to_something;     // 也就是说，void ** == SomeType **
```

上面的代码中，有三个对象:

第一个对象something，其类型（Type）是SomeType。

第二个对象，是p_to_something，其类型是SomeType*，其所指的对象是something。

第三个对象，是p，其类型是SomeType**，其所指的对象是p_to_something

4. 函数调用时，用了void **p，如何理解

还是基于函数的调用是copy by value，然后去理解

```c
// this is caller
{
     SomeType something;
     SomeType *p_to_something = &something;     
     void **p = &p_to_something;

     func(p);
}
...
...
// this is callee
void func(void **copy_p)
{
     SomeType *total_different_something = new SomeType();

     // 因为copy_p所指的对象是指针，替换一个指针很正常，代码也很清晰
     // 因为它等价于 copy_p->对象 = some new value（这个new value应该是个指针）
     *copy_p = total_different_something;        
}
```

这时，你会发现很有趣的现象。当caller调用func()返回后，p这个对象没有任何改变（注意：是p所指的对象改变了），但p_to_something被改变了。这时，p_to_something的值根本就不是some_thing*，*而是一个全新的new SomeType()。

因此，在caller里，当调用完func()后，我们有两个SomeType对象，一个是something，另外一个是被new出来的total different something，它被p_to_something所指的。

当然，如果你用将void func(void **copy_p)改成void func(void *copy_p)，功能上没有任何变化，但是从代码理解上，我们认为void *copy_p指向的不是一个指针对象，而是一个实际对象。

也就是说

```c
void func(void *copy_p)
{
    // 看上去怪怪的，因为常规用法是copy_p-> = 。。。
    *copy_p = new SomeType();     
}
```





## 赋值语句作为判断条件会发生什么

所赋的值会变成判断条件，只有为`0`或`false`时才会被判断为false。



## 指针常量和常量指针

### 指针常量

是说指针是常量

```c
int *const p=&x;//指针p是一个常量，不可修改，但不影响它指向的值。
```

### 常量指针

一个常量的指针，也就是说修饰变量补修饰指针

```c
const int *p=&x;//指针p指向的值不可修改
int const *p=&x;//同上
```

### 两个const

```c
const int * const p=&x;//显然，两种都不能更改，当然我要是直接修改x的值他也可以。
```

### 对const取地址

```c
const int b = 10;
int *pb=&b;//会报错，不能对const类型取地址，因为编译器将 const int 单独视为一种类型，而 指针指向的变量类型是int 不是 const int
const int *pb=&b;//正确，因为指针是指向类型为const int 的，而b变量也是 const int 的。
```



## sizeof 详解

sizeof()是一个操作符，而不是函数，这一点需要注意，它可以用于数据类型和变量名，当对变量名使用时可以不用括号

```c
int a;
int sizea;
sizea=sizeof(int);
printf("sizea=sizeof(int)===%d",sizea);
sizea=sizeof (a);
printf("sizea=sizeof (a)===%d",sizea);
sizea=sizeof a;
printf("sizea=sizeof a===%d",sizea);
```

当 sizeof 对数组进行操作时,计算的是数组整体的大小。

```c
char a[5];
int  b[5];
char c[]={"12345" } ;            
sizeof(a) = 5;
sizeof(b) = 20;
sizeof(c) = 6;    //字符串赋值后面加上’\n‘作为结束符，sizeof()会统计，但strlen(c)=5,不统计'\n'。
```



当操作数是具体的字符串或者数值时，会根据具体的类型进行相应转化。

 ```c
 sizeof(8)  = 4; //自动转化为int类型
 sizeof(8.8) = 8; //自动转化为double类型，注意，不是float类型
 sizeof("ab") = 3  //自动转化为数组类型，
 ​               //长度是4，不是3，因为加上了最后的'\n'符
 ​               //有资料说，会自动转化为指针类型(Linux为4)
 ​               //可能和操作系统与编译器有关系
 ```

当对union，联合体使用时，输出的是联合体中内存最大的那一个。





## 内存对齐

### 结构体的内存对齐

结构体的内存对齐主要有三个要点，**自身对齐**、**指定对齐**和**结构体圆整**。

在对齐时数据会选择`min(自身对齐，指定对齐)`的方式进行对齐，其中指定对齐值可以通过**宏`#pragma pack (N)`指定值**（N的大小必须是2的幂次方）,在C++中可以使用`alignof()`查看指定对齐值的大小。还可以使用`alignas()`的方式指定对齐值，但需要注意的是，使用这种方式如果小于自然对齐的最小单位，那么它将会被忽略。

在每个元素完成对齐后，还会进行结构体的圆整，结构体的圆整就是将结构体的最后几个字节进行对齐，对齐原则是：在结构体数据成员中自身的对齐值最大的那一个与指定对齐值较小的一个；min(4.4)

```c
struct Info2 {
  char a;
  short b;
  char c;
};

std::cout << sizeof(Info2) << std::endl;   // 6     内存结构101110


struct alignas(4) Info2 {
  char a;
  short b;
  char c;
};

std::cout << sizeof(Info2) << std::endl;   // 8     内存结构10001110
std::cout << alignof(Info2) << std::endl;  // 4

struct info {
    char a;
    short b;
    int c;
};
cout<<sizeof(info)<<endl;		//7 内存结构 10111111

#pragma pack(1)
struct info {
    char a;
    short b;
    int c;
};

cout<<sizeof(info)<<endl;		//7 内存结构 1111111

struct info {
    short a;
    int b;
    char c;
};

cout<<sizeof(info)<<endl;		//12 内存结构 110011111000   其中后三位空位就是结构体圆整的结果
```



### 位域结构体

位域操作可以指定结构体中的成员属于一个字节中的哪个位，实际上如果使用位域结构体，结构体的成员名也就不叫成员名了，而叫位域名，格式如下`类型说明符 位域名:位域长度;`，也可以不定义位域名，但是这样做我们就无法使用这个位域，这个位域就只能用于占位和调整位域的结构。

在位域的定义中需要注意以下几点：

1、一个位域必须存储在同一个字节中，不能跨两个字节存储。如果一个字节所剩空间不够存放另一位域时，剩余的空间应该使用空域填充或无名位域填充，声明不使用，然后从下一单元开始存放这个位域。例如：

```cpp
struct 
	{
　　	unsigned a:4
　　	unsigned :0 /*空域,用于填充，声明本字节中剩余位不使用（空穴）*/
　　	unsigned b:4 /*从下一单元开始存放*/
　　	unsigned c:4
	}TowByte;
    
struct{
    char a:4;
    char b:5;	//这样做是不对的，因为第一个位域占用了4个位，这个字节只剩4个位了如果下一个位域想要使用5个位，需要加一个占位域
};
struct{
    char a:4;
    char :4;
    char b:5;	//应该这样做
};

```

2、一个位域的长度不能大于一个int的长度（32bit位）

3、一个位域可以不定义位域名，但此时它只能用来作填充或调整位置。无名位域是不能在程序中使用的。

```c
struct
	{
		unsigned int: 1;		// bit_0 位域名缺省, 无名位域
		unsigned int bit_1 : 1;	// bit_ 位定义域名为 bit_1
		unsigned int bit_2 : 1;			    
		unsigned int bit_3 : 2;				
		unsigned int bit_5 : 1;		
		unsigned int bit_6 : 1; 			
		unsigned int bit_7 : 1;			
	} OneByte;						// 一个字节共8位
```



### \__attribute__((aligned(n)))

` __attribute__((aligned(n)))`：此属性指定了指定类型的变量的最小对齐(以字节为单位)。如果结构中有成员的长度大于n，则按照最大成员的长度来对齐。

注意：对齐属性的有效性会受到链接器(linker)固有限制的限制，即如果你的链接器仅仅支持8字节对齐，即使你指定16字节对齐，那么它也仅仅提供8字节对齐。

` __attribute__((packed))`：此属性取消在编译过程中的优化对齐。

> https://blog.csdn.net/fengbingchun/article/details/81270326



### malloc()如何分配内存

会通过`brk()`和`mmap()`两种方式，二者的主要区别是`brk()`是将堆顶指针向高地址移动，从而获得一块空闲的内存，而`mmap()`则是通过在文件映射区分配一块内存来创建内存。

什么时候用哪一个的主要通过`malloc()`源码中定义的一个阈值来确定：

- 如果用户分配的内存小于 128 KB，则通过 `brk()` 申请内存；
- 如果用户分配的内存大于 128 KB，则通过 `mmap()` 申请内存；

并且`malloc()`分配的内存如果**没有被访问，是不会映射到物理内存中的。**只有在访问已分配的虚拟地址空间的时候，操作系统通过查找页表，发现虚拟内存对应的页没有在物理内存中，就会触发缺页中断，然后操作系统会建立虚拟内存和物理内存之间的映射关系。

并且这两种分配方式在`free()`的时候也会有不同的结果。

- malloc 通过 **`brk()`** 方式申请的内存，`free `释放内存的时候，并不会把内存归还给操作系统，**而是缓存在 malloc 的内存池中，**待下次使用,下次再申请内存时就不用进行**内核态和用户态的切换**，也**不会出现缺页**的情况。；
- malloc 通过 **`mmap()`** 方式申请的内存，`free `释放内存的时候，**会把内存归还给操作系统，内存得到真正的释放**。

#### `malloc(1)`会分配多大的内存？

`malloc()`并不会完全根据我们输入的字节大小来分配内存，它会根据情况，预分配更大的空间作为内存池。之前做了一个实验当我们`malloc(1)`时实际分配的内存有132K。

###  free() 函数只传入一个内存地址，为什么能知道要释放多大的内存？

还记得，我前面提到， malloc 返回给用户态的内存起始地址比进程的堆空间起始地址多了 16 字节吗？

这个多出来的 16 字节就是保存了该内存块的描述信息，比如有该内存块的大小。

这样当执行 free() 函数时，free 会对传入进来的内存地址向左偏移 16 字节，然后从这个 16 字节的分析出当前的内存块的大小，自然就知道要释放多大的内存了。
