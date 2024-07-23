---
title: C++多线程
---


# 1、线程池中的线程数量是依据什么确定的？

在StackOverflow上面发现了一个还不错的回答，意思是：
线程池中的线程数量最直接的限制因素是中央处理器(CPU)的处理器(processors/cores)的数量**N**：如果你的CPU是4-cores的，对于CPU密集型的任务(如视频剪辑等消耗CPU计算资源的任务)来说，那线程池中的线程数量最好也设置为4（或者+1防止其他因素造成的线程阻塞）；对于IO密集型的任务，一般要多于CPU的核数，因为线程间竞争的不是CPU的计算资源而是IO，IO的处理一般较慢，多于cores数的线程将为CPU争取更多的任务，不至在线程处理IO的过程造成CPU空闲导致资源浪费，公式：`最佳线程数 = CPU当前可使用的Cores数 * 当前CPU的利用率 * (1 + CPU等待时间 / CPU处理时间)`（还有回答里面提到的Amdahl准则可以了解一下）



# 2、函数对象传入线程构造函数

## 2.1 传入函数对象临时变量
如果传入的是**函数对象的临时变量**，而不是一个命名的变量，C++ 编译器会将其解析成函数声明，而不是std::thread类型对象的定义。
例如：
`std::thread my_thread(BackgroundTask());`
这里相当于声明了一个名为my_thread的函数，这个函数带有一个参数（函数指针指向没有参数并返回BackgroundTask对象的函数），返回一个std::thread对象的函数，而非启动了一个线程。

## 2.2 如何避免上述情况
1. 使用命名的函数对象变量；
2. 使用多组括号；
`std::thread my_thread((BackgroundTask()));`
3. 使用新统一的初始化语法；
`std::thread my_thread{background_task()};`
4. 使用lambda表达式；


# 3、函数返回后，线程仍然访问局部变量
```cpp
struct func  
{  
    int& i;  
    func(int& i_) : i(i_) {}  
    void operator() ()  
    {  
        for (unsigned j=0 ; j<1000000 ; ++j)  
        {  
            do_something(i); // 1 潜在访问隐患：悬空引用  
        }  
    }  
};  
  
void oops()  
{  
    int some_local_state=0;  
    func my_func(some_local_state);  
    std::thread my_thread(my_func);  
    my_thread.detach(); // 2 不等待线程结束  
} // 3 新线程可能还在运行
```
处理这种情况的常规方法：
使线程函数的功能齐全，将数据复制到线程中，而非复制到共享数据中。如果使用一个可调用的对象作为线程函数，这个对象就会复制到线程中，而后原始对象就会立即销毁。但对于对象中包含的指针和引用还需谨慎。
使用一个能访问局部变量的函数去创建线程是一个糟糕的主意。
**可以通过join()函数来确保线程在函数完成前结束。**


# 4、分离线程（守护线程）
通常称分离线程为守护线程，UNIX中守护线程是指没有任何显式的用户接口，并在后台运行的线程。
这种线程的特点就是长时间运行；线程的生命周期可能会从某一个应用起始到结束，可能会在后台监视文件系统，还有可能对缓存进行清理，亦或对数据结构进行优化。
另一方面，分离线程的另一方面只能确定线程什么时候结束，发后即忘(fire and forget)的任务就使用到线程的这种方式。


# 5、互斥量


# 6、避免死锁

## 6.1 避免嵌套锁
最简单的建议：
一个线程已获得一个锁时，再别去获取第二个。因为每个线程只持有一个锁，锁上就不会产生死锁。即使互斥锁造成死锁的最常见原因，也可能会在其他方面受到死锁的困扰(比如：线程间的互相等待)。 
当你需要获取多个锁，使用一个 std::lock 来做这件事(对获取锁的操作上锁)，避免产生死锁。
## 6.2 避免在持有锁时调用用户提供的代码
次简单的建议：
因为代码是用户提供的，你没有办法确定用户要做什么；用户程序可能做任何事情，包括获取锁。你在持有锁的情况下，调用用户提供的代码；如果用户代码要获取一个锁，就会违反第一个指导意见，并造成 死锁(有时，这是无法避免的)。
## 6.3 使用固定顺序获取锁
当硬性条件要求你获取两个或两个以上的锁，并且不能使用 std::lock 单独操作来获取它们；那么最好在每个线程上，用固定的顺序获取它们(锁)。
因为不同的线程以相反顺序访问列表可能会造成死锁。
这里提供一种避免死锁的方式，定义遍历的顺序，一个线程必须先锁住A才能获取B的锁，在锁住B之后才能获取C的锁。这将消除死锁发生的可能性，不允许反向遍历的列表上。类似的约定常被用来建立其他的数据结构。
## 6.4 使用锁的层次结构

## 6.5 超越锁的延伸扩展


# 7、std::unique_lock第二个参数与成员函数
```cpp
#include<iostream>
#include<thread>
#include<mutex>
std::mutex myMutex1;
void G(int&i) { std::cout << i++ << std::endl; }
//lock_guard可以带第二个参数：
/*.1 std::adopt_lock：
    表示这个互斥量已经被lock()，即不需要在构造函数中lock这个互斥量了。
    前提：必须提前lock,否则会报异常
    之前的lock_guard中也可以用这个参数*/
void test01() {
    int i = 0;          //如果把i写到全局那么就会打印到19，主要是此函数在另一个线程运行的时候被上锁
    static std::mutex my_mutex;
    my_mutex.lock();    //要先lock()后序才能用std::adopt_lock参数
    std::unique_lock<std::mutex>A{ my_mutex,std::adopt_lock };
    for (int n = 0; n < 10; n++)G(i);
    std::chrono::milliseconds dura(100);//1秒=1000毫秒
    std::this_thread::sleep_for(dura);//休息一定的时长
    A.unlock(); //可写可不写，主要是表示可以这样
}
 
/*2.2 std::try_to_lock：
    尝试用mutex的lock()去锁定这个mutex，但如果没有锁定成功，会立即返回，不会阻塞在那里；
    使用try_to_lock的原因是防止其他的线程锁定mutex太长时间，导致本线程一直阻塞在lock这个地方
    前提：不能提前lock();  
    owns_locks()方法判断是否拿到锁，如拿到返回true
    我们考虑使用try_to_lock，线程尝试获取锁，如果没有锁定成功，它不会阻塞在那里，可以去执行其他代码。*/
 
void InMsg()noexcept
{
    auto id = std::this_thread::get_id();
    for (int i = 0; i < 10; i++) {
        std::unique_lock<std::mutex> guard(myMutex1, std::try_to_lock);
        if (guard.owns_lock())  //判断是否拿到锁
        {
            //std::chrono::microseconds time(100);
            //std::this_thread::sleep_for(time);
            std::cout << id << "push\n";
        }
        else
        {
            
            std::cout << id << "wait\n";
        }
    }
}
 
/*2.3 std::defer_lock：
    如果没有第二个参数就对mutex进行加锁，加上defer_lock是始化了一个没有加锁的mutex
    不给它加锁的目的是以后可以调用unique_lock的一些方法
    前提：不能提前lock*/
void test03() {                     //成员函数lock()加锁，unlock()解锁
    std::unique_lock<std::mutex>sbguard1(myMutex1,std::defer_lock);//没有加锁的myMutex1
    sbguard1.lock();     
    for (int i = 0; i < 10; i++)
        std::cout << i << std::endl;
    sbguard1.unlock();//只是示范一下，没必要一定这样写
}
void test03_() {
    std::unique_lock<std::mutex>sbguard1(myMutex1);//去掉那些一样可以，只不过可以更加灵活
    for (int i = 0; i < 10; i++)
        std::cout << i << std::endl;
}
 
//try_lock()成员函数，和std::try_to_lock有点相似
void test04() {
    std::unique_lock<std::mutex>sbguard1(myMutex1,std::defer_lock);
    if (sbguard1.try_lock() == true)//返回true表示拿到锁了          try_lock()要和std::defer_lock配合使用
    {
        //其他代码
        printf("拿到锁\n");
    }
    else
    {
        printf("没拿到锁\n");
        //没拿到锁处理的代码
    }
}
 
//release()成员函数，返回它所管理的mutex对象指针，并释放所有权；也就是说，这个unique_lock和mutex不再有关系
void test05() {
    std::unique_lock<std::mutex>abguard1(myMutex1);
    std::mutex *ptx=abguard1.release();//现在你有责任自己解锁这个myMutex1
    //ptx->lock();
    for (int i = 0; i < 10; i++)
        std::cout << i << std::endl;
    ptx->unlock();
}
 
 
//四：unique_lock所有权的传递mutex，
//std::unique_lock<std::mutex>sbguard1(myMutex1):  所有权概念
//sbguard1拥有myMutex1的所有权
//subguard1可以把自己对mutex（myMutex1)的所有权转移给其他的unique_lock对象；
//所以，unique_lock对象这个mutex的所有权属于 可以转移，但是不能复制
void test06() {
    std::unique_lock<std::mutex>abguard1(myMutex1);
    //std::unique_lock<std::mutex>abguard2(abguard1); //复制所有权，报错
    std::unique_lock<std::mutex>sbguard3(std::move(abguard1));//移动语义，相当于sbguard3和myMutex绑定在一起
                                                              //现在sbguard1指向空,sbguard3指向myMutex1
}
 
std::unique_lock<std::mutex>rtn_unique_lock()
{
    std::unique_lock<std::mutex>tmpguard(myMutex1);
    return tmpguard; //从函数返回一个局部的unique_lock对象是可以的。
                     //返回这种局部对象tmpguard会导致系统生成临时unique_lock对象，并调用unique_lcok的移动构造函数
}
 
void test07() {
    std::unique_lock<std::mutex>sbguard = rtn_unique_lock();//第二种所有权转移的方式
}
 
int main() {
    //std::jthread a(test01), a2(test01);
    
    //std::jthread b(InMsg),b2(InMsg);
    
    //std::jthread c(test03), c2(test03),c3(test03);
 
    //std::jthread d(test04), d2(test04);
 
    //std::jthread e(test05), e2(test05);
 
	return 0;
}
```

## 7.1 std::unique_lock构造函数
* `unique_lock() noexcept;`
  这种默认的构造函数，构造出来的对象不管理任何 mutex 对象。
  
* `explicit unique_lock (mutex_type& m);`
  这种构造函数构造出来的unique_lock 对象接管一个没有lock的mutex对象，并且在构造函数中调用mutex对象的lock函数，失败会阻塞，直到lock成功。由于加了explicit ，这种构造方式只能显式的构造。
  新创建的 unique_lock 对象管理 Mutex 对象 m，并尝试调用 m.lock() 对 Mutex 对象进行上锁，如果此时另外某个 unique_lock 对象已经管理了该 Mutex 对象 m，则当前线程将会被阻塞。
  
* `unique_lock (mutex_type& m, try_to_lock_t tag);`
  这样的构造函数，会在构造函数中调用mutex对象的try_lock函数，锁失败会立刻返回；如果上锁不成功，并不会阻塞当前线程。
  
* `unique_lock (mutex_type& m, defer_lock_t tag) noexcept;`
  这样构造出来的unique_lock 只是单纯的接管mutex对象，不会上锁。
  
*` unique_lock (mutex_type& m, adopt_lock_t tag);`
  这种构造会接管一个已经lock的mutex对象，就是在构造函数中不再调用mutex的lock函数。
  
* `template <class Rep, class Period>
unique_lock (mutex_type& m, const chrono::duration<Rep,Period>& rel_time);`
这种构造函数会调用try_lock_for,在rel_time这个时间段内尝试lock接管的mutex对象，超时会立即返回。

* `template <class Clock, class Duration>
unique_lock (mutex_type& m, const chrono::time_point<Clock,Duration>& abs_time);`
这种构造函数会调用try_lock_until,在abs_time这个时间点之前尝试lock接管的mutex对象，超时立即返回。

* `unique_lock (unique_lock&& x);`
  更换mutex所有权，新创建出来的unique_lock对象会接管参数中对象的mutex对象。

## 7.2 std::unique_lock主要成员函数

由于 std::unique_lock 比 std::lock_guard 操作灵活，因此它提供了更多成员函数。具体分类如下：
1. 上锁/解锁操作：lock，try_lock，try_lock_for，try_lock_until 和 unlock
2. 修改操作：移动赋值(move assignment)，交换(swap)（与另一个 std::unique_lock 对象交换它们所管理的 Mutex 对象的所有权），释放(release)（返回指向它所管理的 Mutex 对象的指针，并释放所有权）
3. 获取属性操作：owns_lock（返回当前 std::unique_lock 对象是否获得了锁）、operator bool()（与 owns_lock 功能相同，返回当前 std::unique_lock 对象是否获得了锁）、mutex（返回当前 std::unique_lock 对象所管理的 Mutex 对象的指针）。

## 7.3 对比std::unique_lock 和 std::lock_guard
二者都是自释放锁；`lock_guard` 在时间和空间上都比`unique_lock`要快；`lock_guard` 功能单一，只能用作自释放锁；u`nique_lock`具备`lock_guard`的所有能力，同时提供更多的能力，比如锁的成员函数都会被封装后导出，同时不会引入double lock和 double unlock；

* 那么为什么有时候需要unlock()？
因为lock()锁住的代码段越少，执行越快，整个程序运行效率越高。
**锁头锁住的代码的多少称为锁的粒度**，粒度一般用粗细来描述。
锁住的代码少，这个粒度叫细，执行效率高。
锁住的代码多，粒度叫粗，执行效率就低。
要学会尽量选择合适粒度的代码进行保护，力度太细，可能漏掉共享数据的保护，粒度太粗，影响效率。
选择合适的粒度，是高级程序员的能力和实力的体现。


# 8、async、packaged_task、promise 区别及使用
来源：https://blog.csdn.net/weiwei9363/article/details/106418146

## 8.1 std::async
当不着急要任务结果的时候，可以使用 `std::async` 启动一个异步任务，`std::async` 返回一个 `std::future` 对象，`std::future` 对象中存放着最终计算的结果。
一切都这么简单，这是获取 `std::future` 最简洁的方式。

当需要最终结果时，调用  `std::future::get() ` 方法即可，该方法会**阻塞线程直到期望值状态就绪为止**。下面的代码是个简单例子：
```cpp
#include <future>
#include <iostream>
int find_the_answer_to_ltuae();
void do_other_stuff();
int main()
{
    std::future<int> the_answer=std::async(find_the_answer_to_ltuae);
    do_other_stuff();
    std::cout<<"The answer is "<<the_answer.get()<<std::endl;
}

```

`std::async` 并不总会开启新的线程来执行任务，你可以指定 `std::launch::async` 来强制开启新线程
```cpp
auto f = std::async(std::launch::async, func);

```

值得注意的是，`std::future` 析构函数会阻塞，直到线程结束。通常我们认为 `std::future::get()` 和 `std::future::wait()` 才会阻塞，析构函数同样也会，这一点需要特别小心。
```cpp
auto sleep = [](int s) { std::this_thread::sleep_for(std::chrono::seconds(s)); };
{
    auto f = std::async( std::launch::async, sleep, 5 ); // 开启一个异步任务，睡眠 5s
    // future 对象析构，等待睡眠结束
}

```

如果 `std::future` 被存放在一个临时对象中，那么`std::async`会立马阻塞，因为临时对象在返回后立马被析构了。例如下面的代码中将会阻塞 10s，但是如果加上 `auto f =` 那么只会阻塞 5s
```cpp
auto sleep = [](int s) { std::this_thread::sleep_for(std::chrono::seconds(s)); };

{
    std::async( std::launch::async, sleep, 5 ); // 临时对象被析构，阻塞 5s
    std::async( std::launch::async, sleep, 5 ); // 临时对象被析构，阻塞 5s
    
    //auto f1 = std::async( std::launch::async, sleep, 5 );
    //auto f2 = std::async( std::launch::async, sleep, 5 );
}

```

## 8.2 std::packaged_task
`std::packaged_task` 本身和线程没啥关系，它只是一个关联了 `std::future` 的仿函数。看下面这个例子：
```cpp
auto task = [](int i) { 
std::this_thread::sleep_for(std::chrono::seconds(5)); return i+100; 
};

std::packaged_task< int(int) > package{ task };
std::future<int> f = package.get_future();
package(1);
std::cout << f.get() << "\n";

```

我们调用 `package(1)` 就开始执行函数，就像执行了 `task(1)` 一样。运行结束后，`f` 已经处在就绪转态，因此 `.get()` 并不会阻塞。

因为 `std::packaged_task` 是个仿函数，我们用它来创建 `std::thread`，并通过 `std::future` 来获取运行的结果，看这个例子：
```cpp
std::packaged_task< int(int) > package{ task };
std::future<int> f = package.get_future();
std::thread t { std::move(package), 5 };

std::cout << f.get() << std::endl; // 阻塞，直到线程 t 结束

t.join();
```

可以看到，`std:packaged_task` 的使用稍微麻烦一些，需要显式的调用或者传递给`std::thread`进行异步调用，但其具有更加灵活的控制调用方式，并且可以选择什么时间开始任务，而 `std::async` 则是一旦调用立马开始执行，并且直接调用 `std::async()`中临时变量析构的导致阻塞的坑，`std::packaged_task` 没有。

需要注意的是，**一定要在调用 f.get() 之前执行 `std::packaged_task `，否则程序会一直阻塞在那：**
```cpp
std::packaged_task<int(int,int)> task(...);
auto f = task.get_future();
std::cout << f.get() << "\n"; // oops!
task(2,3);
```

## 8.3 std::promise
`std::promise` 是一种非常强大的机制。例如，你可以将一个值传递给新进程，而不需要任何额外的同步操作。
```cpp
auto task = [](std::future<int> i) {
    std::cout << i.get() << std::flush; // 阻塞，直到 p.set_value() 被调用
};

std::promise<int> p;
std::thread t{ task, p.get_future() };

std::this_thread::sleep_for(std::chrono::seconds(5));
p.set_value(5);

t.join();
```

## 8.4 自顶向下
介绍完如何使用，现在我们来思考 `std::async`、`std::packaged_task` 和 `std::promise` 之间的关系。总体来说，`std::async` 接口最简单，做的事情最多，抽象程度最高；`std::packaged_task`，抽象程度次之，需要额外的操作但却比较灵活；`std::promise` 功能最为单一，是三者中抽象程度最低的。

我们用 `std::packaged_task` 来实现 `std::async` 的功能：
```cpp
std::future<int> my_async(function<int(int i)> task, int i)
{
    std::packaged_task<int(int)> package{task};	//1
    std::future<int> f = package.get_future();	//2

    std::thread t(std::move(package), i);
    t.detach();	//3
    return f;
}

int main()
{
    auto task = [](int i) { std::this_thread::sleep_for(std::chrono::seconds(5)); return i+100; };

    std::future<int> f = my_async(task, 5);
    std::cout << f.get() << std::endl;
    return 0;
}
```

1 处我们将`std::packaged_task` 用新线程启动，等于 `std::async` 中使用了 `std::launch::async` 参数; 在 2 处通过 `std::packaged_task::get_future` 获得一个 `std::future` 用于返回；在 3 处，无需等待新线程运行结束，因此直接`.detach()` 线程。可以看到，`std::packaged_task` 完全可以实现 `std::async` 的全部功能，并且返回的 `std::future` 并不会在析构的时候阻塞。

接下来我们用 `std::promise` 来实现 `std::packaged_task`。
```cpp
template <typename> class my_task;

template <typename R, typename ...Args>
class my_task<R(Args...)>
{
    std::function<R(Args...)> fn;
    std::promise<R> pr;             // the promise of the result
public:
    template <typename ...Ts>
    explicit my_task(Ts &&... ts) : fn(std::forward<Ts>(ts)...) { }

    template <typename ...Ts>
    void operator()(Ts &&... ts)
    {
        pr.set_value(fn(std::forward<Ts>(ts)...));  // fulfill the promise
    }

    std::future<R> get_future() { return pr.get_future(); }

    // disable copy, default move
};
```

这个简易版的 `my_task` 和 `std::packaged_task` 使用上没啥区别，内部用了 `std::promise` 来获取 `std::future`，在 `operator()` 中通过 `std_value()` 方法来传递数据，让 `std::future` 处于就绪状态。

## 8.5 总结
通过上面的描述，你应该对如何使用，以及它们之间的层级关系有一定了解了。总结一下：

* 用 `std::async` 来做简单的事情，例如异步执行一个任务。但是要注意 `std::future` 析构阻塞的问题。
* `std::packaged_task` 能够很轻松的拿到 `std::future`，选择是否配合 `std::thread` 进行异步处理。同时没有析构阻塞的问题。
* `std::promise` 是三者中最底层的能力，可以用来同步不同线程之间的消息。


# 9、std::async的使用
来源：https://blog.csdn.net/lizhichao410/article/details/123732787

## 9.1 背景
在 C++ 中使用一个可调用对象构造一个 `std::thread` 对象，即可创建一个线程；使用互斥量 `std::mutex` 来确保多个线程对共享数据的读写操作的同步问题；使用 `std::condition_variable` 来解决线程执行顺序的同步问题。

## 9.2 异步操作
①．同步操作：在发出一个方法调用时，在没有得到结果之前该调用就不返回。由调用者主动等待这个调用的结果。

②．异步操作：在发出一个方法调用之后，这个调用就直接返回了，没有返回结果。即当一个异步过程调用发出后，调用者不会立刻得到结果。

## 9.3 std::future
std::future 是一个类模板，用来保存一个异步操作的结果，即这是一个未来值，只能在未来某个时候进行获取。

①．`get()`：等待异步操作执行结束并返回结果，若得不到结果就会一直等待。
②．`wait()`：用于等待异步操作执行结束，但并不返回结果。
③．`wait_for()`：阻塞当前流程，等待异步任务运行一段时间后返回其状态 std::future_status，状态是枚举值：

`deferred`：异步操作还没开始；
`ready`：异步操作已经完成；
`timeout`：异步操作超时。

## 9.4 std::async
`std::async` 是一个函数模板，用来启动一个异步任务。相对于 thread ，`std::future` 是更高级的抽象，异步返回结果保存在 `std::future` 中，使用者可以不必进行线程细节的管理。
`std::async` 有两种启动策略：
①．`std::launch::async`:：函数必须以异步方式运行，即创建新的线程。
②．`std::launch::deferred`：函数只有在 std:async 所返回的期值的 get 或 wait 得到调用时才执行、并且调用方会阻塞至运行结束，否则不执行。
若没有指定策略，则会执行默认策略，将会由操作系统决定是否启动新的线程。

## 9.5 代码示例
```cpp
#include "iostream"
#include "future"
using namespace std;

int getDataDemo()
{
  cout << "数据查询开始" << " threadID:" << this_thread::get_id() << endl;
  this_thread::sleep_for(chrono::seconds(5));//模拟耗时
  return 100;
}
int main()
{
  cout << "执行数据查询" << " threadID:" << this_thread::get_id() << endl;
  future<int> m_future = async(launch::async, getDataDemo);//异步执行
//  future<int> m_future = async(launch::deferred, getDataDemo);//延迟执行
//  future<int> m_future = async(getDataDemo);//默认策略
  future_status m_status;
  do 
  {
    m_status = m_future.wait_for(chrono::seconds(1));
    switch (m_status)
    {
    case std::future_status::ready:
      cout << "数据查询完成" << " threadID:" << this_thread::get_id() << endl;
      break;
    case std::future_status::timeout:
      cout << "数据查询中..." << " threadID:" << this_thread::get_id() << endl;
      break;
    case std::future_status::deferred:
      cout << "数据查询延迟" << " threadID:" << this_thread::get_id() << endl;
		 //wait()等待共享状态就绪。如果共享状态尚未就绪(即提供者尚未设置其值或异常)，则该函数将阻塞调用的线程直到就绪
      m_future.wait();
      break;
    default:
      break;
    }
  }while (m_status != future_status::ready);
  
  int ret = m_future.get();
  cout << "数据查询结果:"<< ret << " threadID:" << this_thread::get_id() << endl;

  system("pause");
    return 0;
}

```
执行结果：
①．异步执行
![异步执行](./images/1678249046240.png)
②．延迟执行
![延迟执行](./images/1678249776268.png)
③．默认策略
![默认策略](./images/1678249799029.png)


# 10、std::lock的使用
锁定给定的可锁定 对象`lock1` 、 `lock2` 、 ... 、 `lockn` ，可以避免死锁。它使用一种避免死锁的算法对多个待加锁对象进行lock操作。当待加锁的对象中有不可获得锁的对象时`std::lock`会阻塞当前线程知道所有对象都可用。

我们看个例子，以下代码运行时有可能出现死锁的情况：
```cpp
std::mutex mt1, mt2;
// thread 1
void fun1()
{
    std::lock_guard<std::mutex> lck1(mt1);
    std::lock_guard<std::mutex> lck2(mt2);
    // do something
}
// thread 2
void fun2()
{
    std::lock_guard<std::mutex> lck2(mt2);  //先lock mt2
    std::lock_guard<std::mutex> lck1(mt1);
    // do something
}
 
int main ()
{
    // 两个线程的互斥量锁定顺序不同，可能造成死锁
    std::thread t1(func1);
    std::thread t2(func2);
 
    t1.join();
    t2.join();
 
    return 0;
}

```

为了避免发生这类死锁，对于任意两个互斥对象，在多个线程中进行加锁时应保证其先后顺序是一致。前面的代码应修改成：
```cpp
std::mutex mt1, mt2;
// thread 1
void fun1()
{
    std::lock_guard<std::mutex> lck1(mt1);
    std::lock_guard<std::mutex> lck2(mt2);
    // do something
}
// thread 2
void fun2()
{
    std::lock_guard<std::mutex> lck1(mt1); // 与线程1一致，先lock mt1,再lock mt2
    std::lock_guard<std::mutex> lck2(mt2);
    // do something
}

```

更好的做法是使用标准库中的`std::lock`函数来对多个Lockable对象加锁。std::lock(或std::try_lock)会使用一种避免死锁的算法对多个待加锁对象进行lock操作，当待加锁的对象中有不可用(无法获得锁)对象时`std::lock`会阻塞当前线程直到所有对象都可用。使用`std::lock`改写前面的代码：
```cpp
std::mutex mt1, mt2;
// thread 1
void fun1()
{
    std::unique_lock<std::mutex> lck1(mt1, std::defer_lock);
    std::unique_lock<std::mutex> lck2(mt2, std::defer_lock);
    std::lock(lck1, lck2);  // lck1和lck2顺序可以任意
    // do something
}
// thread 2
void fun2()
{
    std::unique_lock<std::mutex> lck1(mt1, std::defer_lock);
    std::unique_lock<std::mutex> lck2(mt2, std::defer_lock);
    std::lock(lck2, lck1);  // lck1和lck2顺序可以任意
    // do something
}

```

# 11、std::condition_variable
来源：https://blog.csdn.net/iuices/article/details/123240544?spm=1001.2014.3001.5502
## 11.1 官方定义
在多线程编程中，有一种十分常见的行为：线程同步。线程同步是指线程间需要按照预定的先后次序顺序进行的行为。C++ 11对这种行为也提供了有力的支持，这就是条件变量(`condition_variable`和`condition_variable_any`)。条件变量位于头文件condition_variable下。

`condition_variable`/`condition_variable_any`类是一个synchronization primitive，可用于阻止一个线程或同时阻止多个线程，直到另一个线程修改共享变量（condition），并通知condition_variable，才会继续执行。

当调用它的wait函数时，它使用一个mutex来锁定线程。使得该线程保持阻塞状态，直到被另一个线程调用同一个`condition_variable`对象上的notify函数才被唤醒。**condition_variable类型的对象必须使用`unique_lock<mutex>`等待**，而 `std::condition_variable_any`可以跟任何其他可锁定对象绑定使用, 也可以使用自定义类型。

## 11.2 原理
其实，条件变量跟 c++ 11 没特别大关系，它是操作系统实现的（Linux下使用 `pthread`库中的 `pthread_cond_*()` 函数提供了与条件变量相关的功能）。现在的关键在于理解为啥要有它，而且需注意一点，条件变量自身并不包含条件。因为它通常和 if (或者while) 一起用，所以叫条件变量。

并发有两大需求，一是互斥，二是等待(同步)。互斥是因为线程间存在共享数据，等待则是因为线程间存在依赖。互斥的话，通过互斥锁能搞定，常见的有依赖操作系统的 mutex。条件变量，是为了解决等待需求。考虑实现生产者消费者队列，生产者和消费者各是一个线程。一个明显的依赖是，消费者线程依赖生产者线程 push 元素进队列。
没有条件变量，你会怎么实现消费者呢？让消费者线程一直轮询队列（需要加 mutex)。如果是队列里有值，就去消费；如果为空，要么是继续查，要么sleep一下，让系统过一会再唤醒你，你再次查。可以想到，无论哪种策略，都不通用，要么费cpu，要么线程过分sleep，影响该线程的性能。
有条件变量后，你就可以用事件模式了。上面的消费者线程，发现队列为空，就告诉操作系统，我要wait，一会肯定有其他线程发信号来唤醒我的。这个其他线程，实际上就是生产者线程。生产者线程push队列之后，则调用signal，告诉操作系统，之前有个线程在wait，你现在可以唤醒它了。

上述两种等待方式，前者是轮询(poll)，后者是事件(event)。一般来说，事件方式比较通用，性能不会太差(但存在切换上下文的开销)。轮询方式的性能，就非常依赖并发pattern，也特别消耗cpu。

条件变量要和锁一起使用，锁提供了互斥这一机制，而条件变量在其基础上提供了同步的机制（同步是比互斥更严格的关系，互斥只要求线程间访问某一资源时不存在同时处理或者交替处理的可能，而对线程本身的调度顺序没有限制，也就是说谁先访问都行但你们一个个来，这就是互斥。同步就是在互斥的基础上，虽然线程之间的调度我们没办法控制，但我们可以原子的让某些线程在唤醒时检查某个条件，如果条件不满足就释放锁然后进入阻塞，通过这种方式达到控制不同线程按照某一种你设定的顺序访问资源）。一般条件变量，锁和用户提供的判定条件这三个因素一起组合使用，上文中的某个条件就是指用户提供的判定条件，而线程在检查这个条件，如果不满足就释放锁然后进入阻塞这个过程的原子性由条件变量提供，这也是条件变量的意义。

## 11.3 wait
```cpp
// 当前线程的执行会被阻塞，直到收到 notify 为止。
void wait (unique_lock<mutex>& lck);

// 当前线程仅在pred=false时阻塞；如果pred=true时，不阻塞。
template <class Predicate>
void wait (unique_lock<mutex>& lck, Predicate pred);

```
`wait`会阻塞当前线程直至条件变量被通知，或虚假唤醒发生。

调用`wait`时，该函数会自动调用 `lck.unlock()` 释放锁，使得其他被阻塞在锁竞争上的线程得以继续执行。然后阻塞当前执行线程，另外，一旦当前线程获得通知(notified，通常是另外某个线程调用 notify_* 唤醒了当前线程)，`wait`函数再次调用 `lck.lock()`重新上锁然后wait返回退出，可以理解lck的状态变换和 wait 函数被调用(退出)是同时进行的。

`std::condition_variable`提供了两种 wait() 函数。第二种情况多了条件参数 `Predicate`，只有当 `pred` 条件为false 时调用 wait() 才会**阻塞当前线程且解锁互斥量**，并且在收到其他线程的通知后只有当 `pred` 为 true 时才会被解除阻塞。因此第二种情况类似以下代码：
```cpp
while (!pred()) {
    wait(unique_lock);
}

```

## 11.4 notify
```cpp
// Unblocks当前正在等待此条件的一个线程。
// 如果没有线程在等待，则函数不执行任何操作。
// 如果有多个线程在等待，它不会指定具体哪个线程。
void notify_one() noexcept;

// Unblocks当前等待此条件的所有线程。
// 如果没有线程在等待，则函数不执行任何操作。
void notify_all() noexcept;

```

## 11.5 虚假唤醒
wait类型函数返回时要不是因为被唤醒，要不是因为超时才返回，但是在实际中发现，因此操作系统的原因，wait类型在不满足条件时，它也会返回，这就导致了虚假唤醒。因此，我们一般都是使用带有谓词参数的wait函数，因为这种(xxx, Predicate pred )类型的函数等价于：
```cpp
while (!pred()) {
    wait(unique_lock);
}

```
假设系统不存在虚假唤醒的时，代码只要像下面这样写就可以了：
```cpp
if (不满足条件) {
    //没有虚假唤醒，wait函数可以一直等待，直到被唤醒或者超时，没有问题。
    //但实际中却存在虚假唤醒，导致假设不成立，wait不会持续等待，会跳出if语句，
    //提前执行其他代码，流程异常
    wait();  
}

```

# 12、线程池实现
来源：https://blog.csdn.net/zdarks/article/details/46994607
```cpp

 
#ifndef ILOVERS_THREAD_POOL_H
#define ILOVERS_THREAD_POOL_H
 
#include <iostream>
#include <functional>
#include <thread>
#include <condition_variable>
#include <future>
#include <atomic>
#include <vector>
#include <queue>
 
// 命名空间
namespace ilovers {
    class TaskExecutor;
}
 
class ilovers::TaskExecutor{
    using Task = std::function<void()>;
private:
    // 线程池
    std::vector<std::thread> pool;
    // 任务队列
    std::queue<Task> tasks;
    // 同步
    std::mutex m_task;
    std::condition_variable cv_task;
    // 是否关闭提交
    std::atomic<bool> stop;
    
public:
    // 构造
    TaskExecutor(size_t size = 4): stop {false}{
        size = size < 1 ? 1 : size;
        for(size_t i = 0; i< size; ++i){
            pool.emplace_back(&TaskExecutor::schedual, this);    // push_back(std::thread{...})
        }
    }
    
    // 析构
    ~TaskExecutor(){
        for(std::thread& thread : pool){
            thread.detach();    // 让线程“自生自灭”
            //thread.join();        // 等待任务结束， 前提：线程一定会执行完
        }
    }
    
    // 停止任务提交
    void shutdown(){
        this->stop.store(true);
    }
    
    // 重启任务提交
    void restart(){
        this->stop.store(false);
    }
    
    // 提交一个任务
    template<class F, class... Args>
    auto commit(F&& f, Args&&... args) ->std::future<decltype(f(args...))> {
        if(stop.load()){    // stop == true ??
            throw std::runtime_error("task executor have closed commit.");
        }
        
        using ResType =  decltype(f(args...));    // typename std::result_of<F(Args...)>::type, 函数 f 的返回值类型
        auto task = std::make_shared<std::packaged_task<ResType()>>(
                        std::bind(std::forward<F>(f), std::forward<Args>(args)...)
                );    // wtf !
        {    // 添加任务到队列
            std::lock_guard<std::mutex> lock {m_task};
            tasks.emplace([task](){   // push(Task{...})
                (*task)();
            });
        }
        cv_task.notify_all();    // 唤醒线程执行
        
        std::future<ResType> future = task->get_future();
        return future;
    }
    
private:
    // 获取一个待执行的 task
    Task get_one_task(){
        std::unique_lock<std::mutex> lock {m_task};
        cv_task.wait(lock, [this](){ return !tasks.empty(); });    // wait 直到有 task
        Task task {std::move(tasks.front())};    // 取一个 task
        tasks.pop();
        return task;
    }
    
    // 任务调度
    void schedual(){
        while(true){
            if(Task task = get_one_task()){
                task();    //
            }else{
                // return;    // done
            }
        }
    }
};
 
#endif
 
void f()
{
    std::cout << "hello, f !" << std::endl;
}
 
struct G{
    int operator()(){
        std::cout << "hello, g !" << std::endl;
        return 42;
    }
};
 
 
int main()
try{
    ilovers::TaskExecutor executor {10};
    
    std::future<void> ff = executor.commit(f);
    std::future<int> fg = executor.commit(G{});
    std::future<std::string> fh = executor.commit([]()->std::string { std::cout << "hello, h !" << std::endl; return "hello,fh !";});
    
    executor.shutdown();
    
    ff.get();
    std::cout << fg.get() << " " << fh.get() << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(5));
    executor.restart();    // 重启任务
    executor.commit(f).get();    //
    
    std::cout << "end..." << std::endl;
    return 0;
}catch(std::exception& e){
    std::cout << "some unhappy happened... " << e.what() << std::endl;
}
```

## 12.1 实现原理
接着前面的废话说。“管理一个任务队列，一个线程队列，然后每次取一个任务分配给一个线程去做，循环往复。” 这个思路有神马问题？线程池一般要复用线程，所以如果是取一个 task 分配给某一个 thread，执行完之后再重新分配，在语言层面基本都是不支持的：一般语言的 thread 都是执行一个固定的 task 函数，执行完毕线程也就结束了(至少 c++ 是这样)。so 要如何实现 task 和 thread 的分配呢？

让每一个 thread 都去执行调度函数：循环获取一个 task，然后执行之。

idea 是不是很赞！保证了 thread 函数的唯一性，而且复用线程执行 task 。

* 即使理解了 idea，me 想代码还是需要详细解释一下的。
	* 一个线程 pool，一个任务队列 queue ，应该没有意见；
	* 任务队列是典型的**生产者-消费者模型**，本模型至少需要两个工具：一个 mutex + 一个条件变量，或是一个 mutex + 一个信号量。mutex 实际上就是锁，保证任务的添加和移除(获取)的互斥性，一个条件变量是保证获取 task 的同步性：一个 empty 的队列，线程应该等待(阻塞)；
	* stop 控制任务提交，是受了 Java 的影响，还有实现类不叫 ThreadPool 而是叫 TaskExecutor；
	* `atomic<bool>` 本身是原子类型，从名字上就懂：它们的操作 load()/store() 是原子操作，所以不需要再加 mutex。

## 12.2 C++ 语言细节
即使懂原理也不代表能写出程序，上面用了众多c++ 11的“奇技淫巧”，下面简单描述之。

1. using Task = function<void()> 是类型别名，简化了 typedef 的用法。function<void()> 可以认为是一个函数类型，接受任意原型是 void() 的函数，或是函数对象，或是匿名函数。void() 意思是不带参数，没有返回值。最初的实现版本 Task 类型不是单纯的函数类型，而是一个 class，包含一个 status 字段，表明 Task 的状态：未调度、执行中、执行结束。后来因为简化，故删掉了。
2. pool.emplace_back(&TaskExecutor::schedual, this); 和 pool.push_back(thread{&TaskExecutor::schedual, this}) 功能一样，只不过前者性能会更好；
3. thread{&TaskExecutor::schedual, this} 是构造了一个线程对象，执行函数是成员函数 TaskExecutor::schedual ；
4. 所有对象的初始化方式均采用了 {}，而不再使用之前的 () 方式，因为风格不够一致且容易出错；
5. 匿名函数： `[](int a, int b)->int { return a+b; }` 不多说。`[]` 是捕捉器，&r 是引用域外的变量 r， =r 是拷贝域外的 r 值；
6. delctype(expr) 用来推断 expr 的类型，和 auto 是类似的，相当于类型占位符，占据一个类型的位置；auto f(A a, B b) -> decltype(a+b) 是一种用法，不能写作 decltype(a+b) f(A a, B b)，为啥？！ c++ 就是这么规定的！
7. commit 方法是不是略奇葩！可以带任意多的参数，第一个参数是 f，后面依次是函数 f 的参数！ 可变参数模板是 c++ 11 的一大亮点，够亮！至于为什么是 Arg... 和 arg... ，因为规定就是这么用的！
8. make_shared 用来构造 shared_ptr 智能指针。用法大体是 `shared_ptr<int> p = make_shared<int>(4)` 然后 *p == 4 。智能指针的好处就是， 自动 delete ！
9. bind 函数，接受函数 f 和部分参数，返回currying后的匿名函数，譬如 bind(add, 4) 可以实现类似 add4 的函数！
10. forward() 函数，类似于 move() 函数，后者是将参数右值化，前者是... 肿么说呢？大概意思就是：不改变最初传入的类型的引用类型(左值还是左值，右值还是右值)；
11. packaged_task 就是任务函数的封装类，通过 get_future 获取 future ， 然后通过 future 可以获取函数的返回值(future.get())；packaged_task 本身可以像函数一样调用 () ；
12. queue 是队列类， front() 获取头部元素， pop() 移除头部元素；back() 获取尾部元素，push() 尾部添加元素；
13. lock_guard 是 mutex 的 stack 封装类，构造的时候 lock()，析构的时候 unlock()，是 c++ RAII 的 idea；
14. condition_variable cv; 条件变量， 需要配合 unique_lock 使用；unique_lock 相比 lock_guard 的好处是：可以随时 unlock() 和 lock()。 cv.wait() 之前需要持有 mutex，wait 本身会 unlock() mutex，如果条件满足则会重新持有 mutex。




# C++ 11 并发指南四(< future > 详解一 std::promise 介绍)
来源：https://www.cnblogs.com/haippy/p/3239248.html

前面两讲《C++ 11 并发指南二(std::thread 详解)》，《C++ 11 并发指南三(std::mutex 详解)》分别介绍了 std::thread 和 std::mutex，相信读者对 C++ 11 中的多线程编程有了一个最基本的认识，本文将介绍 C++ 11 标准中 `<future>` 头文件里面的类和相关函数。

`<future>` 头文件中包含了以下几个类和函数：
* Providers 类：std::promise, std::package_task
* Futures 类：std::future, shared_future.
* Providers 函数：std::async()
* 其他类型：std::future_error, std::future_errc, std::future_status, std::launch.

## std::promise 类介绍
promise 对象可以保存某一类型 T 的值，该值可被 future 对象读取（可能在另外一个线程中），因此 promise 也提供了一种线程同步的手段。在 promise 对象构造时可以和一个共享状态（通常是std::future）相关联，并可以在相关联的共享状态(std::future)上保存一个类型为 T 的值。

可以通过 get_future 来获取与该 promise 对象相关联的 future 对象，调用该函数之后，两个对象共享相同的共享状态(shared state)

* promise 对象是异步 Provider，它可以在某一时刻设置共享状态的值。
* future 对象可以异步返回共享状态的值，或者在必要的情况下阻塞调用者并等待共享状态标志变为 ready，然后才能获取共享状态的值。

下面以一个简单的例子来说明上述关系
```c++
#include <iostream>       // std::cout
#include <functional>     // std::ref
#include <thread>         // std::thread
#include <future>         // std::promise, std::future

void print_int(std::future<int>& fut) {
    int x = fut.get(); // 获取共享状态的值.
    std::cout << "value: " << x << '\n'; // 打印 value: 10.
}

int main ()
{
    std::promise<int> prom; // 生成一个 std::promise<int> 对象.
    std::future<int> fut = prom.get_future(); // 和 future 关联.
    std::thread t(print_int, std::ref(fut)); // 将 future 交给另外一个线程t.
    prom.set_value(10); // 设置共享状态的值, 此处和线程t保持同步.
    t.join();
    return 0;
}
```

### std::promise 构造函数
### std::promise::get_future 介绍
### std::promise::set_value 介绍
### std::promise::set_exception 介绍
### std::promise::set_value_at_thread_exit 介绍
### std::promise::swap 介绍



# C++ 11 并发指南四(< future > 详解二 std::packaged_task 介绍)
来源：https://www.cnblogs.com/haippy/p/3279565.html

上一讲《C++ 11 并发指南四(< future > 详解一 std::promise 介绍)》主要介绍了 < future > 头文件中的 std::promise 类，本文主要介绍 std::packaged_task。

std::packaged_task 包装一个可调用的对象，并且允许异步获取该可调用对象产生的结果，从包装可调用对象意义上来讲，std::packaged_task 与 std::function 类似，只不过 std::packaged_task 将其包装的可调用对象的执行结果传递给一个 std::future 对象（该对象通常在另外一个线程中获取 std::packaged_task 任务的执行结果）。

std::packaged_task 对象内部包含了两个最基本元素，一、被包装的任务(stored task)，任务(task)是一个可调用的对象，如函数指针、成员函数指针或者函数对象，二、共享状态(shared state)，用于保存任务的返回值，可以通过 std::future 对象来达到异步访问共享状态的效果。

可以通过 std::packged_task::get_future 来获取与共享状态相关联的 std::future 对象。在调用该函数之后，两个对象共享相同的共享状态，具体解释如下：

* std::packaged_task 对象是异步 Provider，它在某一时刻通过调用被包装的任务来设置共享状态的值。
* std::future 对象是一个异步返回对象，通过它可以获得共享状态的值，当然在必要的时候需要等待共享状态标志变为 ready.

std::packaged_task 的共享状态的生命周期一直持续到最后一个与之相关联的对象被释放或者销毁为止。下面一个小例子大致讲了 std::packaged_task 的用法：
```c++
#include <iostream>     // std::cout
#include <future>       // std::packaged_task, std::future
#include <chrono>       // std::chrono::seconds
#include <thread>       // std::thread, std::this_thread::sleep_for

// count down taking a second for each value:
int countdown (int from, int to) {
    for (int i=from; i!=to; --i) {
        std::cout << i << '\n';
        std::this_thread::sleep_for(std::chrono::seconds(1));
    }
    std::cout << "Finished!\n";
    return from - to;
}

int main ()
{
    std::packaged_task<int(int,int)> task(countdown); // 设置 packaged_task
    std::future<int> ret = task.get_future(); // 获得与 packaged_task 共享状态相关联的 future 对象.

    std::thread th(std::move(task), 10, 0);   //创建一个新线程完成计数任务.

    int value = ret.get();                    // 等待任务完成并获取结果.

    std::cout << "The countdown lasted for " << value << " seconds.\n";

    th.join();
    return 0;
}
```

执行结果为：
```
10
9
8
7
6
5
4
3
2
1
Finished!
The countdown lasted for 10 seconds.
```

## std::packaged_task 构造函数



# C++ 11 并发指南四(< future > 详解三 std::future & std::shared_future)

来源：https://www.cnblogs.com/haippy/p/3280643.html

上一讲《C++ 11 并发指南四(< future > 详解二 std::packaged_task 介绍)》主要介绍了 < future > 头文件中的 std::packaged_task 类，本文主要介绍 std::future，std::shared_future 以及 std::future_error，另外还会介绍 < future > 头文件中的 std::async，std::future_category 函数以及相关枚举类型。

## std::future 介绍
前面已经多次提到过 std::future，那么 std::future 究竟是什么呢？简单地说，std::future 可以用来获取异步任务的结果，因此可以把它当成一种简单的线程间同步的手段。std::future 通常由某个 Provider 创建，你可以把 Provider 想象成一个异步任务的提供者，Provider 在某个线程中设置共享状态的值，与该共享状态相关联的 std::future 对象调用 get（通常在另外一个线程中） 获取该值，如果共享状态的标志不为 ready，则调用 std::future::get 会阻塞当前的调用者，直到 Provider 设置了共享状态的值（此时共享状态的标志变为 ready），std::future::get 返回异步任务的值或异常（如果发生了异常）。

一个有效(valid)的 std::future 对象通常由以下三种 Provider 创建，并和某个共享状态相关联。Provider 可以是函数或者类，其实我们前面都已经提到了，他们分别是：

* std::async 函数，本文后面会介绍 std::async() 函数。
* std::promise::get_future，get_future 为 promise 类的成员函数，详见 C++ 11 并发指南四(< future > 详解一 std::promise 介绍)。
* std::packaged_task::get_future，此时 get_future为 packaged_task 的成员函数，详见C++ 11 并发指南四(< future > 详解二 std::packaged_task 介绍)。

一个 std::future 对象只有在有效(valid)的情况下才有用(useful)，由 std::future 默认构造函数创建的 future 对象不是有效的（除非当前非有效的 future 对象被 move 赋值另一个有效的 future 对象）。

 在一个有效的 future 对象上调用 get 会阻塞当前的调用者，直到 Provider 设置了共享状态的值或异常（此时共享状态的标志变为 ready），std::future::get 将返回异步任务的值或异常（如果发生了异常）。

下面以一个简单的例子说明上面一段文字吧（参考）：
```c++
// future example
#include <iostream>             // std::cout
#include <future>               // std::async, std::future
#include <chrono>               // std::chrono::milliseconds

// a non-optimized way of checking for prime numbers:
bool is_prime(int x)
{
    for (int i = 2; i < x; ++i)
        if (x % i == 0)
            return false;
    return true;
}

int main()
{
    // call function asynchronously:
    std::future < bool > fut = std::async(is_prime, 444444443);

    // do something while waiting for function to set future:
    std::cout << "checking, please wait";
    std::chrono::milliseconds span(100);
    while (fut.wait_for(span) == std::future_status::timeout)
        std::cout << '.';

    bool x = fut.get();         // retrieve return value

    std::cout << "\n444444443 " << (x ? "is" : "is not") << " prime.\n";

    return 0;
}
```