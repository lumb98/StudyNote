# STL 笔记
在C++primer中看到的 STL 内容，看到一点记录一点。’
## initializer_list
用于表示某种特定类型的值的数组。

initializer_list提供的操作
```c++
initializer_list<T> lst; //默认初始化:T类型元素的空列表
initializer_list<T> lst{a,b,c,d...};//lst的元素数量和初始值一样多；lst的元素是对应初始值的副本；列表中的元素是const
lst2(lst) ; lst2=lst;//拷贝或者赋值一个initializer_list对象不会拷贝列表中的元素；拷贝后原始列表和副本共享元素
lst.size(); //列表中元素的数量。
lst.begin(); //列表首元素指针。
lst.end();//列表尾元素的下一个地址的指针
```
`initializer_list`实际上和`vector`非常类似，他俩之间的区别是前者不能对里面的值进行修改，二后者则比较灵活，这两个容器都适合在输入的元素数量不确定的时候使用。

