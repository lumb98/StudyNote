# STL 笔记


## 一些技巧

可以用vector的swap进行空间的收缩。方法如下 ：

```cpp
vector<int> (v).swap(v);
```

这个操作实际上是使用了一个匿名对象的方法，假设之前的v占用空间为10000，但我们实际只用了5000，就可以用这个操作来进行空间收缩，因为匿名对象的内存在调用后会被回收，我们用v来初始化这个匿名对象，初始化后占用空间为5000，然后再调用swap方法来进行交换，10000的空间会再调用后呗删除，而v现在的空间刚好只有5000了。

## deque和vector的区别

deque容器相对于vector，支持双端的操作，可以分别调用`deque.push_back(elem);`、`deque.push_front(elem);`、`deque.pop_back(elem);`和`deque.pop_front(elem);`来进行头部和尾部的添加和删除操作。

它的`insert()`方法相对于vector来说多了一个可以插入区间的方法，`deque.inster(deque.begin(),d2.begin(),d2.end())`,这行代码的医生就是在deque的头部插入d2的全部数据。

`erase()`方法也具有区间删除的能力。删除后会返回下一个数据的位置。



## stack

构造函数：

* `stack<T> stk;`                                 //stack采用模板类实现， stack对象的默认构造形式
* `stack(const stack &stk);`            //拷贝构造函数

赋值操作：

* `stack& operator=(const stack &stk);`           //重载等号操作符

数据存取：

* `push(elem);`      //向栈顶添加元素
* `pop();`                //从栈顶移除第一个元素
* `top(); `                //返回栈顶元素

大小操作：

* `empty();`            //判断堆栈是否为空
* `size(); `              //返回栈的大小



## queue

构造函数：

- `queue<T> que;`                                 //queue采用模板类实现，queue对象的默认构造形式
- `queue(const queue &que);`            //拷贝构造函数

赋值操作：

- `queue& operator=(const queue &que);`           //重载等号操作符

数据存取：

- `push(elem);`                             //往队尾添加元素
- `pop();`                                      //从队头移除第一个元素
- `back();`                                    //返回最后一个元素
- `front(); `                                  //返回第一个元素

大小操作：

- `empty();`            //判断堆栈是否为空
- `size(); `              //返回栈的大小



## pair 对组

对组不需要包含头文件

pair容器的创建

* `pair<string,int>p("tom",20);`
* `pair<string,int>p2=make_pair("jerry",30);`

访问pair容器的值

访问第一个(键key)`pair.first`

访问第二个(值val)`pair.second`

## set 容器

构造：

* `set<T> st;`                        //默认构造函数：
* `set(const set &st);`       //拷贝构造函数

赋值：

* `set& operator=(const set &st);`    //重载等号操作符 

set 容器的赋值操作是将另一个set复制给当前set。



set容器默认从小到大排序，如果想要更改排序规则，可以在创建容器时加入一个仿函数类。同时我们也可以用这种方式排序自定义的数据类型

```cpp
class MyCompare 
{
public:
	bool operator()(int v1, int v2) {
		return v1 > v2;
	}
};
int main(){
    set<int,MyCompare> s2;//这样做之后插入的数据就会按照仿函数的内容进行排序。
}
```

自定义数据类型排序

```cpp
class Person
{
public:
	Person(string name, int age)
	{
		this->m_Name = name;
		this->m_Age = age;
	}

	string m_Name;
	int m_Age;

};
class comparePerson
{
public:
	bool operator()(const Person& p1, const Person &p2)
	{
		//按照年龄进行排序  降序
		return p1.m_Age > p2.m_Age;
	}
};
```



## map

map的使用与set的使用非常类似，区别在于map中的元素都是pair类型，是key和value一一对应的格式，

* 它的排序是按照键值进行排序的

**构造：**

* `map<T1, T2> mp;`                     //map默认构造函数: 
* `map(const map &mp);`             //拷贝构造函数



**赋值：**

* `map& operator=(const map &mp);`    //重载等号操作符

**大小**

- `size();`          //返回容器中元素的数目
- `empty();`        //判断容器是否为空
- `swap(st);`      //交换两个集合容器

**插入和删除：**

- `insert(elem);`           //在容器中插入元素。
- `clear();`                    //清除所有元素
- `erase(pos);`              //删除pos迭代器所指的元素，返回下一个元素的迭代器。
- `erase(beg, end);`    //删除区间[beg,end)的所有元素 ，返回下一个元素的迭代器。
- `erase(key);`            //删除容器中值为key的元素。

**查找和统计：**

- `find(key);`                  //查找key是否存在,若存在，返回该键的元素的迭代器；若不存在，返回set.end();
- `count(key);`                //统计key的元素个数



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



## C++ unordered_map容器的成员方法

unordered_map 既可以看做是关联式容器，更属于自成一脉的无序容器。因此在该容器模板类中，既包含一些在学习关联式容器时常见的成员方法，还有一些属于无序容器特有的成员方法。

表 2 列出了 unordered_map 类模板提供的所有常用的成员方法以及各自的功能。



| 成员方法           | 功能                                                         |
| ------------------ | ------------------------------------------------------------ |
| begin()            | 返回指向容器中第一个键值对的正向迭代器。                     |
| end()              | 返回指向容器中最后一个键值对之后位置的正向迭代器。           |
| cbegin()           | 和 begin() 功能相同，只不过在其基础上增加了 const 属性，即该方法返回的迭代器不能用于修改容器内存储的键值对。 |
| cend()             | 和 end() 功能相同，只不过在其基础上，增加了 const 属性，即该方法返回的迭代器不能用于修改容器内存储的键值对。 |
| empty()            | 若容器为空，则返回 true；否则 false。                        |
| size()             | 返回当前容器中存有键值对的个数。                             |
| max_size()         | 返回容器所能容纳键值对的最大个数，不同的操作系统，其返回值亦不相同。 |
| operator[key]      | 该模板类中重载了 [] 运算符，其功能是可以向访问数组中元素那样，只要给定某个键值对的键 key，就可以获取该键对应的值。注意，如果当前容器中没有以 key 为键的键值对，则其会使用该键向当前容器中插入一个新键值对。 |
| at(key)            | 返回容器中存储的键 key 对应的值，如果 key 不存在，则会抛出 out_of_range 异常。 |
| find(key)          | 查找以 key 为键的键值对，如果找到，则返回一个指向该键值对的正向迭代器；反之，则返回一个指向容器中最后一个键值对之后位置的迭代器（如果 end() 方法返回的迭代器）。 |
| count(key)         | 在容器中查找以 key 键的键值对的个数。                        |
| equal_range(key)   | 返回一个 pair 对象，其包含 2 个迭代器，用于表明当前容器中键为 key 的键值对所在的范围。 |
| emplace()          | 向容器中添加新键值对，效率比 insert() 方法高。               |
| emplace_hint()     | 向容器中添加新键值对，效率比 insert() 方法高。               |
| insert()           | 向容器中添加新键值对。                                       |
| erase()            | 删除指定键值对。                                             |
| clear()            | 清空容器，即删除容器中存储的所有键值对。                     |
| swap()             | 交换 2 个 unordered_map 容器存储的键值对，前提是必须保证这 2 个容器的类型完全相等。 |
| bucket_count()     | 返回当前容器底层存储键值对时，使用桶（一个线性链表代表一个桶）的数量。 |
| max_bucket_count() | 返回当前系统中，unordered_map 容器底层最多可以使用多少桶。   |
| bucket_size(n)     | 返回第 n 个桶中存储键值对的数量。                            |
| bucket(key)        | 返回以 key 为键的键值对所在桶的编号。                        |
| load_factor()      | 返回 unordered_map 容器中当前的负载因子。负载因子，指的是的当前容器中存储键值对的数量（size()）和使用桶数（bucket_count()）的比值，即 load_factor() = size() / bucket_count()。 |
| max_load_factor()  | 返回或者设置当前 unordered_map 容器的负载因子。              |
| rehash(n)          | 将当前容器底层使用桶的数量设置为 n。                         |
| reserve()          | 将存储桶的数量（也就是 bucket_count() 方法的返回值）设置为至少容纳count个元（不超过最大负载因子）所需的数量，并重新整理容器。 |
| hash_function()    | 返回当前容器使用的哈希函数对象。                             |



