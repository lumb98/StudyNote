# C++ 并发编程

 C++并发编程是在C++11标准开始支持的，想要使用多线程，需要包含头文件 `#include <thread>`。并且在编译时需要链接多线程的库，`-lthread`。

## 前言

`thread`是C++标准库提供的类。



## 线程的创建

线程的创建实际上就是要先写一个函数，然后把这个函数作为入口参数传给`thread`的构造函数。

```cpp
#include <thread>
int dosomething(){
   
    return 0;
}

int main(){
    thread FirstThread(dosometing);//这里就已经完成了对新线程的创建了，
    
    return 0;
}
explicit
```



## join和detach

`join()`就是汇入，而`detach()`则是分离。

### join 汇入

当对一个对象调用`join()`方法时，主进程就会陷入阻塞，等待该对象，等该对象运行完成才会继续运行，并且将该对象线程的资源进行回收，但是这里会出现一个问题，就是调用`join()`方法的时机，如果在调用`join()`方法前，该线程就已经抛出错误，那么主线程就无法回收该线程的资源了，这应该很容易理解，因为我们调用join方法的目的，是为了等待线程的运行完毕并且汇入，但是当调用这个线程的时候该线程已经结束了，那么我们也就无法再回收它的资源了。

### joinable

joinable()方法可以用来检测当前的线程是否可以，进行 join() 和 detach() 操作，它会返回一个bool值，如果返回值为 true 则可以进行上述两个操作，如果返回值为 false 则不能进行这两个操作。



### detach 分离

使用`detach`可以让线程与当前线程分离，从而达到在后台运行的效果，但这也意味着它与主线程不能再通过`thread`进行直接的交互了，也就没法再次汇入。但C++运行库可以保证在线程退出时，相关资源能够正确回收。

分离线程通常称为_守护线程_(daemon threads)，在UNIX中守护线程是指没有任何显示接口并在后台运行的线程。




```cpp
#include <mutex>

mutex my_mutex,my_mutex2;
lock_guard<mutex> lockg;
void hello(){
    for(int i=0;i<1000;++i){
        // my_mutex.lock();
        lock_guard<mutex> lockg(my_mutex);
        cout<<"I am second thread."<<i<<endl;
        cout<<"I am second thread."<<i<<endl;
        // my_mutex.unlock();
        usleep(1000);
    }
    lock(my_mutex,my_mutex2);//可以同时锁定多个锁，如果一个没锁住，就会解锁其他，过一段时间再尝试
    //调用adopt_lock后 只有在析构时会调用unlock()，而构造时不会调用lock()，就可以使其自动unloc(),

    lock_guard<mutex> lockg2(my_mutex2,adopt_lock);
    
    unique_lock<mutex> ulock2(my_mutex,adopt_lock);
    
    unique_lock<mutex> ulock2(my_mutex,try_to_lock);
    
    unique_lock<mutex> ulock2(my_mutex,defer_lock);
    
    /*  第二参数
        adopt_lock,前提，已经加锁了，用这个参数后会自动解锁，当然要在一个代码块内。
        try_to_lock,前提，没有加锁，用这个参数可以尝试加锁，如果加锁失败也不会反复尝试，可以通过调用owns_lock( )方法，根据其返回值确定是否得到锁，如果得到返回true。
        defer_lock ，前提，没有加锁，用这个参数也不会加锁，目前理解的这样操作的原因是方便使用下面的那些成员函数，自己确定加锁的位置，
    */
    
    /*  unique_lock的成员函数
        lock();             将该unique_lock加锁。
        unlock();           将该unique_lock解锁
        try_lock();         尝试加锁，如果加锁成功返回true，如果失败返回false。
        release();          解绑当前mutex锁，并把该锁指针返回，解绑后，该mutex锁不会自动解锁，需要用新的指针调用unlock()手动解锁。
    */
    mutex mutex1;
    unique_lock<mutex> ulock11(mutex1);
    unique_lock<mutex> ulock22(move(ulock11));
    
    /*  unique_lock的所有权
        所有权的转移
        上面的代码就可以将mutex1的所有权从ulock11转移给ulock22。除此之外也可以用返回值的方式将它的所有权转移
    */
        unique_lock<mutex> rtn_unique_lock(){
            unique_lock<mutex> tmpguard(mutex1);
            return tmpguard;
        }
    /*
        unique_lock<mutex> ulock33 = rtn_unique_lock();
        此时mutex1的所有权就转移到了ulock33了；  
    */
}
```





### 单例设计模式

```cpp
class MyCAS{
private:
    MyCAS(){}       //构造函数放在私有空间，在外部无法实例化该类。只能通过调用 GetInstanc()来实例化。
  	static MyCAS *m_instance=NULL;
public:
    static MyCAS *GetInstance(){   //检测是否实例化过没实例化就是空的，然后返回类的指针。不管调用多少次，都只有一个该类。
        if(m_instance==NULL){
            m_instance =new MyCAS();
            static CGarRecycle cl;
        }
        return m_instance;
    }
    class CGarRecycle{//当程序结束时 静态变量cl的声明周期结束了，就会调用该类的析构函数，而该类的析构函数会检查instance是否为空，如果不为空就会释放内存。
        ~CGarRecycle(){
            if(MyCAS::m_instance){
                delete MyCAS::m_instance;
                MyCAS::m_instance=NULL;
            }
        }
    }
}
MyCAS *MyCAS::m_instance = NULL;


MyCAS *p_a = MyCAS::GetInstance();
```

`std::this_thread::get_id()`可以获取当前线程的id。

call_once()



condition_variable模板类，可以用该类生产一个条件对象。

```cpp
std::condition_cariable<std::mutex> my_cond;
```



**wait()**方法，用来等一个东西

如果第二个参数lambda表达式返回值是true，那么wait()直接返回；

xxxxxxxxxx5 1void func(void *copy_p)2{3    // 看上去怪怪的，因为常规用法是copy_p-> = 。。。4    *copy_p = new SomeType();     5}c

如果**wait()**没有第二个参数，：`my_cond.wait(sbguard1)`；那么wait( )将解锁互斥量，并堵塞到本行，一直堵塞到其他某个线程调用notify_one()为止。

当其他线程调用notify_one()，将本wait**(原本是睡着/堵塞)**的状态唤醒后，wait就开始恢复干活，分为两种情况

1. wait()不断尝试重新获取互斥量锁，如果获取不到，那么流程就会卡在wait这里等待获取，如果获得了锁，就会继续向下执行。
2. 如果wait有第二个参数，就会判断lambda表达式，如果lambda表达式为false，那么wait又对互斥量解锁，然和在这里休眠等待下一次被notify唤醒。		
3. 如果lambda表达式为true，则wait返回，流程继续往下运行。此时互斥锁处于被锁状态

**notify_one与notify_all**，二者的区别是，notify_one只能唤醒一个线程， 而notify_all可以同时唤醒多个被该锁wait的线程。



#### 虚假唤醒

在wait()中设置第二个参数，一般是一个lambda表达式，并且这个表达式要正确判断要处理的公共数据是否存在。

```cpp
my_cond.wait(sbguard1,[this]{
    if()
        return true;
    else
        return false;
})
```





## async和future创建后台任务，并返回值

### async

是一个函数模板，可以用来启动一个异步任务，启动起来一个异步任务后，他会返回一个future对象，future也是一个类模板。

启动一个**异步任务**，会根据系统当前情况选择一种启动方式，并开始执行对应的线程入口函数，它会返回一个future对象。

​	这个future对象里面就含有线程入口函数所返回的结果(也就是线程返回的结果)，我们可以通过调用future对象的成员函数，get()来获取结果。

​	**future** ：将来，也被称为一种访问异步操作结果的机制，也就是说这个结果不会马上拿到，但哪那个线程执行完毕的时候，就可以拿到结果了。

​	**get()**方法如果不拿到返回值，会一直被阻塞，直到拿到目标线程的返回值为止。并且该方法只能调用**一次**。

async会根据系统情况自动选择是否创建新线程，来运行该任务，它有一个参数，默认值为`std::launch::async |std::launch::deferred`。

async与detach的一个区别是，**async之后如果主线程执行完毕**，会在主线程的return前等待async线程的返回之后才退出，而detach则会引发异常。

async与thread的区别是，当需要启动线程但是当前系统资源不允许时，使用thread启动会导致整个系统崩溃，async则不会，因为系统资源紧张时，async会选择deferred作为启动方式，等到下次调用get时在当前线程运行，当然这种情况async的启动方式必须为**默认**或`std::launch::async |std::launch::deferred`

```cpp
#include <future>
int mythread(){
    cout<<"mythread()start"<<"threadid = "<<std::this_thread::get_id()<<endl;//打印新线程的ID
    std::chrono::millisseconds dura(5000);//定义5秒的时间。
    std::this_thread::sleep_for(dura);//休息dura毫秒。
    cout<<"mythread()end"<<"threadid = "<<std::this_thread::get_id()<<endl;//打印新线程的ID
    return 5;
}
int main(){
    cout<<"main"<<"threadid = "<<std::this_thread::get_id()<<endl;//打印线程的ID
    std::future<int> result = std::async(mythread);            //启动一个异步任务。
    cout<<"continue......!"<<endl;
    int def=0;
    cout<<result.get()<<endl;//必须要等到 mythread 线程执行完，程序才能继续进行。如果拿不到就会阻塞在这里。
    cout<<"I love China"<<endl;
}
```

#### async的 launch 启动方式

我们可以通过额外向async()传递第一个参数，该参数的类型是std::launch(枚举类型),来达到一些特殊的目的。

1. std::launch::deferred 表示线程入口函数调用被延迟到，std::future的wait() 或者 get() 函数调用时才执行，并且不会创建新线程，之后再当前线程像调用函数一样调用线程。如果不调用，甚至不会创建线程。

   ```cpp
   class A{
   public:
       int func(int a){
           return a;
       }
   };
   
   A a;//实例化类
   int temp;
   future<int> result = async(std::launch::deferred,&A::func,&a,temp);
   ```

   

2. std::launch::async，指定async的的启动方式为新建一个线程。

我们可以用 `future_status::deferred/future_status::ready/future_status::timeout`来对它的运行状态进行判断。他们分别是

1. future_status::deferred 线程处于deferred状态，如果想要使用其返回值，或者令其运行，就需要get()或者wait()，但此时会在当前线程运行。
2. future_status::ready/future_status::timeout 分别是目标线程已经运行完毕可以直接get()值了和目标任务还没有运行完毕，需要等待。

## packaged_task 打包任务

std::package_task 是个类模板，它的模板参数是各种可调用对象，通过std::package_task 来吧各种可调用对象包装起来，方便将来作为线程入口函数来调用。利用 package 打包后的函数会自动加入future，我们就可以直接调用`get_futuren()`方法，来获取线程的返回值了。

```cpp
int mythread(int a){
    cout<<"mythread()start"<<"threadid = "<<std::this_thread::get_id()<<endl;//打印新线程的ID
    std::chrono::millisseconds dura(5000);//定义5秒的时间。
    std::this_thread::sleep_for(dura);//休息dura毫秒。
    cout<<"mythread()end"<<"threadid = "<<std::this_thread::get_id()<<endl;//打印新线程的ID
    return 5;
}
int main(){
    cout<<"main"<<"threadid = "<<std::this_thread::get_id()<<endl;//打印线程的ID
    std::packaged_task<int(int)> mypt(mythread);//我们把mythread通过packaged_task,给包装起来。
    std::thread t1(std::ref(mypt),1);
    t1.join();
    std::future<int> result =mypt.get_future();//调用get_future()方法，注意该方法是packaged_task对象的。
    cout<<rusult.get()<<endl;
    //还可以通过下面的方法直接调用。
    mypt(4);			//需要注意的是这样的调用方式，与  async 函数的 std::launch::deferred 类似，会使函数再当前线程中执行，不会新建线程
    
}
```



### 将打包的任务放进容器中，和取出

```cpp
vector <std::packaged_task<int(int)>> mytasks;
mytasks.push_back(std::move(mypt));
std::packaged_task<int(int)> mypt2;
auto iter =mytasks.begin();
mypt2=std::move(*iter);     //移动语义
mytasks.erase(iter);		//删除第一个元素，迭代器已经失效了，所以后续代码不能再用iter了；
mypt2(1231);				//再次提醒，这样使用packaged_task 不会新建线程。
std::future<int> result =mypt2.get_future();
cout<<result.get()<<endl;
```



## promise,类模板



std::promise 我们能够在某个线程中给它复制，然后我们可以在其他线程中把这个值取出。

```cpp
void mythread(std::promise<int> &tmpp, int calc){
    calc++;
    calc *=10;
    std::chrono::millisseconds dura(5000);//定义5秒的时间。
    std::this_thread::sleep_for(dura);//休息dura毫秒。
    //终于计算出结果
    int result =calc;
    tmpp.set_value(result);
    return;
}
int main(){
    std::promise<int> myprom;
    std::thread t1(mythread,std::ref(myprom),180);
    t1.join();
    //获取结果值
    std::future<int> fu1=myprom.get_future();
    auto result =fu1.get();
    cout<<"result = "<<result <<endl;
   	cout<<"I love china = "<<endl;
}
```



## 递归mutex --- std::recursive_mutex

常规的mutex只能单次上锁，而 recurisive_mutex 则可以在一个线程中多次上锁，多次解锁。





## 带超时的互斥量timed_mutex 和 recursive_timed_mutex

`try_lock_for()`:等待一段时间，如果拿到了锁，或者等待一段时间拿到了，就会返回true，若两次都没拿到就返回false。

`tre_lock_until()`:它的参数是未来的一个时间点，它会一直尝试获取锁直到那个时间，在那个时间内拿到就返回true，没拿到就返回false。



```cpp
timed_mutex my_mutex;
std::chrono::millisseconds dura(100);
my_mutex.try_lock_for(dura);  //如果获取锁失败，则等阻塞dura的时间，再次尝试。 拿到锁返回真，没拿到阻塞。

```





























## 类中的方法所谓线程的一般调用手法

```cpp
class A{
public:
    int func(int a){
        return a;
    }
};
A a;//实例化类
int temp;
future<int> result = async(&A::func,&a,temp);//temp为func的实参。用引用输入，不如会创建新的对象。
```



## chrono时间获取

```cpp
chrono::milliseconds timeout(100);//timeout=100毫秒。
chrono::steade_clock::now();//当前时间点。
chrono::steade_clock::now()+time;//100毫秒后
```







## lambda表达式随记

[this]表示调用当前函数。

lambda表达式，实际上就是一个匿名的函数，它的形式为

```cpp
[](int a,int b){
    return a+b;
}
```

它的优势是它可以写在几乎任何地方，甚至可以写在函数的入口参数上，只作为一个值来使用，当然那个表达式的函数需要又返回值。
