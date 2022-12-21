

# Makefile和g++学习笔记

## g++部分

  学习C和C++的同学应该都知道，gcc是一款跨平台的C/C++编译器，可以在Linux/Windows平台下使用，具有十分强大的功能，结构也十分灵活，并且可以通过不同的前端模块来支持各种语言，如Java、Fortran、Pascal、Modula-3和Ada的编译。许多有名的工程和库都是使用gcc进行编译的，如nginx,libevent等。今天我们重点介绍gcc组件中可以用来编译C++程序的g++组件的使用。

​    g++可以在命令行使用，也可以通过配置IDE的编译环境来调用系统配置的g++环境，大家可以根据需要自行配置。下面是一些博主本人使用g++编译程序的一些经验和总结。

### 1. g++编译过程

 g++在对源程序执行编译工作时，需要经过以下四个步骤：

1. **预处理**：对源程序中的宏定义、包含头文件等进行处理，生成后缀名为.i的文件（使用）

 使用格式：

```bash
g++ -E hello.cpp -o hello.i
```

  hello.cpp是需要编译的源文件，-o 选项指定输出的文件名。这里使用-E选项编译生成hello.i 文件

2.  **转化为汇编文件**：使用-S 选项，可以将预处理之后的.i文件转换为目标机器的汇编代码.S文件

 使用格式：

```bash
g++ -S hello.i  -o  hello.s
```

 使用该参数可以将.i 文件编译生成.s 文件，输出文件名同样使用-o 选项指定。

3.  **汇编文件->目标文件**，即转换为机器代码：使用-c选项

 使用格式：

```bash
g++ -c hello.s -o hello.o
```

4. **链接**：将上一步产生的目标文件链接为可执行文件，使用-o参数

使用格式：

```bash
g++ hello.o  -o  hello
```

   以上过程是g++工具编译cpp源程序的具体过程，在实际使用时我们可以不用按照流程一步步编译，可以一步到位将源程序编译为可执行文件，只需要使用如下命令：

```bash
g++  hello.cpp -o  hello
```

###  2.g++常用编译选项

   在使用g++工具进行编译时，我们可以附加一些编译选项让编译更加智能，从而方便我们查看编译错误和警告。g++提供了许多有用的编译选项，下面总结一些常用选项：

 -o FILE: 指定输出文件名，在编译为目标代码时，这一选项不是必须的。如果FILE没有指定，缺省文件名是a.out

 -c: 只编译生成目标文件,不链接

 -m486: 针对 486 进行代码优化

 -Wall: 允许发出gcc能提供的所有有用的警告,也可以用-W(warning)来标记指定的警告

 使用格式: 

```bash
1 g++ -Wall hello.cpp -o hello
```

 -Werror: 把所有警告转换为错误,以在警告发生时中止编译过程；

 -v: 显示在编译过程的每一步中用到的命令 ；

-static: 链接静态库，即执行静态链接,g++默认是链接动态库，如果需要链接静态库需要使用本选项进行指定；

-g: 在可执行程序中包含标准调试信息, 使用该选项生成的可执行文件可以用gdb工具进行调试；

-w: 关闭所有警告，建议不要使用该选项

-shared: 生成共享目标文件。通常用在建立共享库时；

-On: 这是一个优化选项，如果在编译时指定该选项，则编译器会根据n的值（n取0到3之间）对代码进行不同程度的优化，其中-O0 表示不优化，n的值越大，优化程度越高；

 -L: 库文件依赖选项，该选项用于指定编译的源程序依赖的库文件路径，库文件可以是静态链接库，也可以是动态链接库，linux系统默认的库路径是/usr/lib，如果需要的库文件不在这个路径下就要用-L指定

```bash
g++  foo.cpp  -L/home/lib  -lfoo  -o   foo
```

 -I: 该选项用于指定编译程序时依赖的头文件路径，linux平台默认头文件路径在/usr/include下，如果不在该目录下，则编译时需要使用该选项指定头文件所在路径

```bash
gcc  foo.cpp  -I/home/include   -o  foo
```

###  3.编译动态库

 g++除了可以编译源程序生成可执行文件，也可以编译动态链接库，方法如下：

 (1) 分步完成

```bash
gcc -fPIC -c func.cpp -o func.o 
gcc -shared -o libfunc.so func.o
```

 (2) 一步完成

```bash
gcc -fPIC -shared -o libfunc.so func.cpp
```

## 

### 静态库的制作

```bash
ar -crv libx.a f1.o f2.o #利用 f1.o 和 f2.o 生产静态库文件 libx.a
```



### 静态库的链接

```bash
gcc hello.c -o hello -static -L . -l x #将hello.c 与 在当前目录下的名为libx.a的静态库文件链接，生产hello可执行文件  编译器会自动补齐 x库文件的前缀和后缀，即 lib 和 .a 与 x 组合成 libx.a 的完整库文件名。
```



### 动态库的制作

```bash
gcc -fPIC -shared -o libxx.so f1.c f2.c #利用 f1.o 和 f2.o 生产动态库文件 libxx.so
```



### 动态库的链接

```bash
gcc hello.c -o hello -L . -l xx #将hello.c 与 在当前目录下的名为libxx.so的静态库文件链接，生产hello可执行文件  编译器会自动补齐 xx库文件的前缀和后缀，即 lib 和 .a 与 xx 组合成 libxx.so 的完整库文件名。
```

但是此时如果直接运行生产的可执行文件还不能正常运行，因为我们的动态库文件没有加入到系统默认的动态库文件夹中，我们需要运行如下命令来查看当前系统默认的动态库文件夹，并将我们的动态库文件加入到该文件夹。

```bash
ldd hello #查看名为hello的可执行文件依赖的动态库（也叫共享库）文件夹所在位置
cp libxx.so /lib/x85_64-linux-gnu/   #将动态链接库复制到上面命令输出的文件夹中。
```

大功告成，程序可以顺利运行了。

### 编译时进行宏定义（一般用于条件编译）

`-D`：我们可以使用-D选项来在编译时进行一个宏定义，假设我们在代码中定义了一个条件编译当定义DEBUG时就可以启用，我们就可以在编译时加上如下代码

```shell
gcc xxxxx -D 
```







## Makefile部分

Makefile的命令结构如下。

```makefile
目标:依赖
	@达成目标所需要运行的命令   #@可以让这条命令不显示命令本身，只输出结果，也可以不加。
```



Makefile可以利用文件的时间来判断文件是否更新。判断依赖和目标文件的最后修改时间，当目标文件比依赖文件旧时，则会重新编译目标文件。

### Makefile的通配符



* `%.o`:表示一个xx.o文件
* `$@`:表示目标文件
* `$<`:表示第一个依赖文件
* `$^`:表示所有依赖



### Makefile的运行

make时后面可以加一个空格，再加上目标的名称，则生成该目标，若没有目标则默认生成第一个目标



### Makefile的假想目标

通常情况下，我们可以用这个命令来清楚之前make的结构，中间文件和可执行文件等。

```makefile
#上面还有其他内容被省略
clean:
	rm *.o test
```

但是当文件夹中有，名为clean的文件时，根据makefile的规则，clean文件已经存在并且它的依赖都比它旧（实际上是没有依赖）所有不会运行下面的代码，也就不会达到我们想要的效果，这种时候我们就需要用到假想目标这种语法了。

```makefile
#上面还有其他内容被省略
clean:
	rm *.o test
.PHONY:clean
```

这样做旧依旧能够运行clean了。



### Makefile的变量

Makefile的变量分为两种

* `:=`:即时变量，该变量的值即刻确定，在**定义时**就被确定了
* `=`:延时变量，该变量的值，在**使用时**才确定
* `?=`:延时变量，如果是**第一次定义**才起效，如果前面被定义过了就忽略这句
* `+=`:附加，它是即时变量还是延时变量取决于 前面的定义

```makefile
A:=$(C)
B=$(C)
C=abc#1

all:
	@echo A = $(A)
	@echo B = $(B)
C=123#2
```

会输出：

```bash
#1
A =
B = abc
#2
A =
B = 123
```



```makefile
A:=$(C)
B=$(C)
C=abc

all:
	@echo A = $(A)
	@echo B = $(B)
C=123#1
C+=123#2
```

会输出：

```bash
#1
A =
B = 123
#2
A =
B = abc 123
```



### Makefile函数

#### 函数遍历

`$(foreach val,list,text)`:对于list（通常用空格隔开）里的每一个变量执行text操作

```makefile
A=a b c
B=$(foreach f,$(A),$(f).o)#用f代表A中的各个变量，执行第三个参数的操作。

all:
	@echo B=$(B)
#输出
B=a.o b.o c.o
```

#### filter函数过滤

`$(filter pattern...,text)`:在text里面取出符合pattern格式的值

`$(filter-out pattern...,text)`:在text里面取出不符合pattern格式的值

```makefile
C= a b c d/
D=$(filter %/,$(C))
E=$(filter-out %/,$(C))

all:
	@echo D=$(D)
	@echo E=$(E)
#输出
D=d/
E=a b c
```

#### wildcard函数查找

`$(wildcard pattern)`:pattern定义了文件名的格式，wildcard取出其中存在的文件

```makefile
files = $(wildcard *.c)
files2=a.c b.c c.c d.c e.c
file3 = $(wildcard $(files2))
all:
	@echo files=$(files)
	@echo files3=$(files3)
#输出
files=a.c b.c c.c
```

也可以用这个函数确定哪些文件真实存在。

```makefile
files2=a.c b.c c.c d.c e.c#其中 d.c e.c 并不存在于文件中
file3 = $(wildcard $(files2))
all:
	@echo files3=$(files3)
#输出
files3=a.c b.c c.c
```

#### patsubst函数替换

`$(patsubst pattern,replacement,$(var))`:从var中将符合patern格式的内容，替换为replacement。

```makefile
files2=a.c b.c c.c d.c e.c abc
dep_files=$(patsubst %.c,%.d,$(file2))
all:
	@echo dep_files=$(dep_files)
#输出
files3=a.d b.d c.d d.d e.d abc
```

### 头文件依赖

在下述情况中，当我们修改了c.h中的内容，不加入新的一行，则c.h中的更改不会被编译，因为make认为对于c.o(%.o)这个目标，它的依赖c.c(%.c)，并没有更改，所以我们就需要手动加入新的一行来解决这个问题，当要make时，我们运行到新加入的一行，发现c.h发生了修改，我们要生成新的目标c.o，如何生成呢，在下面的`%.o:%.c`下面告诉了我们如何生成。

```makefile
test:a.o b.o c.o
	gcc -o test $^
c.o:c.c c.h#加入后可以解决

%.o:%.c
	gcc -c -o $@ $<
clean:
	rm *.o test
.PHONY:clean
```

但我们不能对每一个文件都这样操作，这样使用通配符将失去意义，我们的工作量也会回到最初的起点。这个时候，我们就可以用gcc的一个工具，它可以查看依赖，并生成文件。

```makefile
gcc -M c.c #打印出c.c的依赖

gcc -M -MF c.d c.c  #把依赖文件写入文件c.d

gcc -c -o c.o c.c -MD -MF c.d #编译c.o，把依赖写入文件c.d
```

需要注意的是生成的.d文件是隐藏文件。也就是说它的前面还有一个点。`.xx.o.d`

具体做法如下

```makefile
objs=a.o b.o c.o
dep_files := $(patsubst %,.%.d,$(objs)) #将objs中的所有文件替换成.%.d的形式 使用:= 是因为下一步中如果不这样会出现递归调用
dep_files := $(wildcard $(dep_files)) #将dep_files中确实存在的文件取出，赋给自己，这里用:= 即时变量是因为在这个语句中用到了它本身，但是如果使用=延时变量的话，会在用到dep_files时才生成dep_files，但是我用到的时候还没有，所有是错误的。

test: $(objs)
	gcc -o test $^
ifneq ($(dep_files),) #如果这个变量不等于空
include $(dep_files)
endif

%.o :%.c
	gcc -c -o $@ $< -MD -MF .$@.d

clean:
	rm *.o test
distclean:
	rm $(dep_files)

.PHONY:clean
```

### CFLAGS

`CFLAGS=-Werror` :会把所有警告当成错误

`CFLAGS=-Iinclude`:将include目录添加到GCC搜索头文件默认的路径，







研究了一天了，本来想把 链接库文件也加进来，但是失败了。目前算是够用了，先这样吧。

```bash
.
├── bin
├── include
│   └── sum.h
├── lib
├── Makefile
└── src
    ├── sum.cpp
    └── main.cpp
```

以上是目录结构

```makefile
VPATH = src:include:lib    #包含需要用到的文件所在的目录，不然makefile找不到
cc=g++
prom=sum
src=$(shell find ./ -name "*.cpp")	
include= $(shell find ./ -name "*.hpp") #使用shell命令，输出头文件相对当前目录的文件名，也就是待位置的文件名
obj= main.o sum.o					  #obj文件的宏定义。

CFLAGS = -g -Wall -I include            #-I include 是针对g++的，是头文件的位置，不加这个的话g++会报错，找不到头文件


$(prom):$(obj)	
	$(cc) -o $@  $^ 				# $@ 是所有的目标文件，写在这里的目的也就是让它输出为 $(prom) 即可执行文件的名字
    							   # $^ 是所有的依赖文件，也就是$(obj)

%.o:%.cpp							# %.o 就是要用到的 某.o 文件，后面就是 某.cpp 文件，注意 再一个目标中 % 所指代的
     $(cc) -c $^ $(CFLAGS)             #容是一样的，比如这个就分两次 代表了 main 和 sum 这个语句是会运行多次的。
	
.PHONY : clean	                      # .PHONY 即伪目标。
clean:
	rm -rf $(prom) $(obj)
```

