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



## Makefile部分

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
	$(cc) -o $@  $^ 				# $@ 是所有的目标文件，xie'z这里的目的也就是让它输出为 $(prom) 即可执行文件的名字
    							   # $^ 是所有的依赖文件，也就是$(obj)

%.o:%.cpp							# %.o 就是要用到的 某.o 文件，后面就是 某.cpp 文件，注意 再一个目标中 % 所指代的
     $(cc) -c $^ $(CFLAGS)             #容是一样的，比如这个就分两次 代表了 main 和 sum 这个语句是会运行多次的。
	
.PHONY : clean	                      # .PHONY 即伪目标。
clean:
	rm -rf $(prom) $(obj)
```

