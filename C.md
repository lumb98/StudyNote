C++ 中的 void **

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