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

## unordered_set

若想使用该容器，必须要`#include <unordered_set> /n using namespace std;`可见该容器也是C++标准库的一部分。

该容器的底层是由哈希表实现的，`unordered`就是无需的意思，实际上它和`set`容器的区别只是该容器不会对存进去的内容进行排序，而`set`容器则会对存入其中的内容进行排序。

unordered_set 类模板成员方法

| 成员方法           | 功能                                                         |
| ------------------ | ------------------------------------------------------------ |
| **begin()**        | 返回指向容器中第一个元素的正向迭代器。                       |
| **end();**         | 返回指向容器中最后一个元素之后位置的正向迭代器。             |
| cbegin()           | 和 begin() 功能相同，只不过其返回的是 const 类型的正向迭代器。 |
| cend()             | 和 end() 功能相同，只不过其返回的是 const 类型的正向迭代器。 |
| **empty()**        | 若容器为空，则返回 true；否则 false。                        |
| **size()**         | 返回当前容器中存有元素的个数。                               |
| max_size()         | 返回容器所能容纳元素的最大个数，不同的操作系统，其返回值亦不相同。 |
| **find(key)**      | 查找以值为 key 的元素，如果找到，则返回一个指向该元素的正向迭代器；反之，则返回一个指向容器中最后一个元素之后位置的迭代器（如果 end() 方法返回的迭代器）。 |
| **count(key)**     | 在容器中查找值为 key 的元素的个数。                          |
| equal_range(key)   | 返回一个 pair 对象，其包含 2 个迭代器，用于表明当前容器中值为 key 的元素所在的范围。 |
| **emplace()**      | 向容器中添加新元素，效率比 insert() 方法高。                 |
| emplace_hint()     | 向容器中添加新元素，效率比 insert() 方法高。                 |
| insert()           | 向容器中添加新元素。                                         |
| erase()            | 删除指定元素。                                               |
| clear()            | 清空容器，即删除容器中存储的所有元素。                       |
| **swap()**         | 交换 2 个 unordered_map 容器存储的元素，前提是必须保证这 2 个容器的类型完全相等。 |
| bucket_count()     | 返回当前容器底层存储元素时，使用桶（一个线性链表代表一个桶）的数量。 |
| max_bucket_count() | 返回当前系统中，unordered_map 容器底层最多可以使用多少桶。   |
| bucket_size(n)     | 返回第 n 个桶中存储元素的数量。                              |
| bucket(key)        | 返回值为 key 的元素所在桶的编号。                            |
| load_factor()      | 返回 unordered_map 容器中当前的负载因子。负载因子，指的是的当前容器中存储元素的数量（size()）和使用桶数（bucket_count()）的比值，即 load_factor() = size() / bucket_count()。 |
| max_load_factor()  | 返回或者设置当前 unordered_map 容器的负载因子。              |
| rehash(n)          | 将当前容器底层使用桶的数量设置为 n。                         |
| reserve()          | 将存储桶的数量（也就是 bucket_count() 方法的返回值）设置为至少容纳count个元（不超过最大负载因子）所需的数量，并重新整理容器。 |
| hash_function()    | 返回当前容器使用的哈希函数对象。                             |







```cpp
unordered_set<int> uset{10,2,3,4,5,6,8,7};//值得注意的是当我们用下面的循环（范围for循环）打印时是与当前顺序相反的。
for(int print:uset){
  cout<<print<<endl;
}
//输出为7、8、6、5、4、3、2、10
```







