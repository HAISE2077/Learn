---
title: C++设计模式
---

# 1、C++ 单例模式总结与剖析
来源：https://www.cnblogs.com/sunchaothu/p/10389842.html


## 1.1 什么是单例模式
特点是只提供唯一一个类的实例，具有全局变量的特点，在任何位置都可以通过接口获取到那个唯一实例。

具体运用场景如：
设备管理器，系统中可能有多个设备，但是只有一个设备管理器，用于管理设备驱动;
数据池，用来缓存数据的数据结构，需要在一处写，多处读取或者多处写，多处读取;

## 1.2 实现要点
* 全局只有一个实例：static 特性，同时禁止用户自己声明并定义实例（把构造函数设为 private）；
* 线程安全；
* 禁止赋值和拷贝；
* 用户通过接口获取实例：使用 static 类成员函数。

## 1.3 单例实现
### 1.3.1 有缺陷的懒汉式单例
懒汉式(Lazy-Initialization)的方法是直到使用时才实例化对象，也就说直到调用get_instance() 方法的时候才 new 一个单例的对象， 如果不被调用就不会占用内存。
```cpp
#include <iostream>
// version1:
// with problems below:
// 1. thread is not safe
// 2. memory leak

class Singleton{
private:
    Singleton(){
        std::cout<<"constructor called!"<<std::endl;
    }
    Singleton(Singleton&)=delete;
    Singleton& operator=(const Singleton&)=delete;
    static Singleton* m_instance_ptr;
public:
    ~Singleton(){
        std::cout<<"destructor called!"<<std::endl;
    }
    static Singleton* get_instance(){
        if(m_instance_ptr==nullptr){
              m_instance_ptr = new Singleton;
        }
        return m_instance_ptr;
    }
    void use() const { std::cout << "in use" << std::endl; }
};

Singleton* Singleton::m_instance_ptr = nullptr;

int main(){
    Singleton* instance = Singleton::get_instance();
    Singleton* instance_2 = Singleton::get_instance();
    return 0;
}

```
运行结果：
```txt
constructor called!

```
可以看到，获取了两次类的实例，却只有一次类的构造函数被调用，表明只生成了唯一实例，这是个最基础版本的单例实现，他有哪些问题呢？

1. **线程安全的问题**,当多线程获取单例时有可能引发竞态条件：第一个线程在if中判断 `m_instance_ptr`是空的，于是开始实例化单例；同时第2个线程也尝试获取单例，这个时候判断`m_instance_ptr`还是空的，于是也开始实例化单例;这样就会实例化出两个对象,这就是线程安全问题的由来; 
**解决办法**:加锁。
2. **内存泄漏**：注意到类中只负责new出对象，却没有负责delete对象，因此只有构造函数被调用，析构函数却没有被调用;因此会导致内存泄漏。
**解决办法**： 使用共享指针。

### 1.3.2 线程安全、内存安全的懒汉式单例（智能指针，锁）
```cpp
#include <iostream>
#include <memory> // shared_ptr
#include <mutex>  // mutex

// version 2:
// with problems below fixed:
// 1. thread is safe now
// 2. memory doesn't leak

class Singleton{
public:
    typedef std::shared_ptr<Singleton> Ptr;
    ~Singleton(){
        std::cout<<"destructor called!"<<std::endl;
    }
    Singleton(Singleton&)=delete;
    Singleton& operator=(const Singleton&)=delete;
    static Ptr get_instance(){

        // "double checked lock"
        if(m_instance_ptr==nullptr){
            std::lock_guard<std::mutex> lk(m_mutex);
            if(m_instance_ptr == nullptr){
              m_instance_ptr = std::shared_ptr<Singleton>(new Singleton);
            }
        }
        return m_instance_ptr;
    }


private:
    Singleton(){
        std::cout<<"constructor called!"<<std::endl;
    }
    static Ptr m_instance_ptr;
    static std::mutex m_mutex;
};

// initialization static variables out of class
Singleton::Ptr Singleton::m_instance_ptr = nullptr;
std::mutex Singleton::m_mutex;

int main(){
    Singleton::Ptr instance = Singleton::get_instance();
    Singleton::Ptr instance2 = Singleton::get_instance();
    return 0;
}


```
运行结果：
```txt
constructor called!
destructor called!
```
发现确实只构造了一次实例，并且发生了析构。

shared_ptr和mutex都是C++ 11的标准，以上这种方法的优点是：
* 基于 shared_ptr, 用了C++ 比较倡导的 RAII思想，用对象管理资源,当 shared_ptr 析构的时候，new 出来的对象也会被 delete掉。以此避免内存泄漏。
* 加了锁，使用互斥量来达到线程安全。这里使用了两个 if判断语句的技术称为**双检锁**；好处是，只有判断指针为空的时候才加锁，避免每次调用 get_instance的方法都加锁，锁的开销毕竟还是有点大的。

不足之处在于： 使用智能指针会要求用户也得使用智能指针，非必要不应该提出这种约束; 使用锁也有开销; 同时代码量也增多了，实现上我们希望越简单越好。

还有更加严重的问题，在某些平台（与编译器和指令集架构有关），==双检锁会失效！ #E91E63== 具体可以看这篇文章（http://www.drdobbs.com/cpp/c-and-the-perils-of-double-checked-locki/184405726 ），解释了为什么会发生这样的事情。

因此这里还有第三种的基于 Magic Staic的方法达到线程安全

### 1.3.3 最推荐的懒汉式单例（magic static) ——局部静态变量
```cpp
#include <iostream>

class Singleton
{
public:
    ~Singleton(){
        std::cout<<"destructor called!"<<std::endl;
    }
    Singleton(const Singleton&)=delete;
    Singleton& operator=(const Singleton&)=delete;
    static Singleton& get_instance(){
        static Singleton instance;
        return instance;

    }
private:
    Singleton(){
        std::cout<<"constructor called!"<<std::endl;
    }
};

int main(int argc, char *argv[])
{
    Singleton& instance_1 = Singleton::get_instance();
    Singleton& instance_2 = Singleton::get_instance();
    return 0;
}


```
运行结果：
```txt
constructor called!
destructor called!
```
这种方法又叫做 Meyers' SingletonMeyer's的单例， 是著名的写出《Effective C++ 》系列书籍的作者 Meyers 提出的。所用到的特性是在C++ 11标准中的Magic Static特性：
>If control enters the declaration concurrently while the variable is being initialized, the concurrent execution shall wait for completion of the initialization.
如果当变量在初始化的时候，并发同时进入声明语句，并发线程将会阻塞等待初始化结束。

这样保证了并发线程在获取静态局部变量的时候一定是初始化过的，所以具有线程安全性。

C++ 静态变量的生存期 是从声明到程序结束，这也是一种懒汉式。这样保证了并发线程在获取静态局部变量的时候一定是初始化过的，所以具有线程安全性。

**这是最推荐的一种单例实现方式：**
1. 通过局部静态变量的特性保证了线程安全 (C++ 11, GCC > 4.3, VS2015支持该特性);
2. 不需要使用共享指针，代码简洁；
3. 注意在使用的时候需要声明单例的引用 `Single&` 才能获取对象。

另外网上有人的实现返回指针而不是返回引用：
```cpp
static Singleton* get_instance(){
    static Singleton instance;
    return &instance;
}

```
这样做并不好，理由主要是无法避免用户使用`delete instance`导致对象被提前销毁。还是建议大家使用返回引用的方式。

### 1.3.4 函数返回引用
```cpp
#include <iostream>

class A
{
public:
	A& ret_singleton(){
    static A instance;
    return instance;
	}
};
```
严格来说，这不属于单例了，因为类A只是个寻常的类，可以被定义出多个实例，但是亮点在于提供了`ret_singleton`的方法，可以返回一个全局（静态）变量，起到类似单例的效果，这要求用户必须保证想要获取 全局变量A ，只通过`ret_singleton()`的方法。

## 1.4 单例模板
### 1.4.1 CRTP 奇异递归模板模式实现
```cpp
// brief: a singleton base class offering an easy way to create singleton
#include <iostream>

template<typename T>
class Singleton{
public:
    static T& get_instance(){
        static T instance;
        return instance;
    }
    virtual ~Singleton(){
        std::cout<<"destructor called!"<<std::endl;
    }
    Singleton(const Singleton&)=delete;
    Singleton& operator =(const Singleton&)=delete;
protected:
    Singleton(){
        std::cout<<"constructor called!"<<std::endl;
    }

};
/********************************************/
// Example:
// 1.friend class declaration is requiered!
// 2.constructor should be private


class DerivedSingle:public Singleton<DerivedSingle>{
   // !!!! attention!!!
   // needs to be friend in order to
   // access the private constructor/destructor
   friend class Singleton<DerivedSingle>;
public:
   DerivedSingle(const DerivedSingle&)=delete;
   DerivedSingle& operator =(const DerivedSingle&)= delete;
private:
   DerivedSingle()=default;
};

int main(int argc, char* argv[]){
    DerivedSingle& instance1 = DerivedSingle::get_instance();
    DerivedSingle& instance2 = DerivedSingle::get_instance();
    return 0;
}

```
以上实现一个单例的模板基类，使用方法如例子所示意，子类需要将自己作为模板参数T 传递给 `Singleton<T>` 模板; 同时需要将基类声明为友元，这样才能调用子类的私有构造函数。

基类模板的实现要点是：
1. 构造函数需要是 **protected**，这样子类才能继承；
2. 使用了奇异递归模板模式CRTP(Curiously recurring template pattern)；
3. `get_instance` 方法和 1.3.3 的static local方法一个原理；
4. 在这里基类的析构函数可以不需要 virtual ，因为子类在应用中只会用 Derived 类型，保证了析构时和构造时的类型一致。

### 1.4.2 不需要在子类声明友元的实现方法
在 stackoverflow上， 有大神给出了不需要在子类中声明友元的方法，在这里一并放出；精髓在于使用一个代理类 token，子类构造函数需要传递token类才能构造，但是把 token保护其起来， 然后子类的构造函数就可以是公有的了，这个子类只有 `Derived(token)`的这样的构造函数，这样用户就无法自己定义一个类的实例了，起到控制其唯一性的作用。代码如下。
```cpp
// brief: a singleton base class offering an easy way to create singleton
#include <iostream>

template<typename T>
class Singleton{
public:
    static T& get_instance() noexcept(std::is_nothrow_constructible<T>::value){
        static T instance{token()};
        return instance;
    }
    virtual ~Singleton() =default;
    Singleton(const Singleton&)=delete;
    Singleton& operator =(const Singleton&)=delete;
protected:
    struct token{}; // helper class
    Singleton() noexcept=default;
};


/********************************************/
// Example:
// constructor should be public because protected `token` control the access
class DerivedSingle:public Singleton<DerivedSingle>{
public:
   DerivedSingle(token){
       std::cout<<"destructor called!"<<std::endl;
   }

   ~DerivedSingle(){
       std::cout<<"constructor called!"<<std::endl;
   }
   DerivedSingle(const DerivedSingle&)=delete;
   DerivedSingle& operator =(const DerivedSingle&)= delete;
};

int main(int argc, char* argv[]){
    DerivedSingle& instance1 = DerivedSingle::get_instance();
    DerivedSingle& instance2 = DerivedSingle::get_instance();
    return 0;
}


```

### 1.4.3 函数模板返回引用
在 1.3.4 中提供了一种类型的全局变量的方法，可以把一个一般的类，通过这种方式提供一个类似单例的
全局性效果（但是不能阻止用户自己声明定义这样的类的对象）；在这里我们把这个方法变成一个 template 模板函数，然后就可以得到任何一个类的全局变量。
```cpp
#include <iostream>

class A
{
public:
    A() {
        std::cout<<"constructor" <<std::endl;
    }
    ~A(){
        std::cout<<"destructor"<<std::endl;
    }
};

template<typename T>
T& get_global(){
    static T instance;
    return instance;
}

int main(int argc, char *argv[])
{
    A& instance_1 = get_global<A>();
    A& instance_2 = get_global<A>();
    return 0;
}


```
可以看到这种方式确实非常简洁，同时类仍然具有一般类的特点而不受限制，当然也因此失去了单例那么强的约束（禁止赋值、构造和拷贝构造）。
这里把函数命名为 `get_global()` 是为了强调，这里可以通过这种方式获取得到单例最重要的全局变量特性；但是并不是单例的模式。

## 1.5 何时应该使用或者不使用单例
根据stackoverflow上的一个高票答案 singleton-how-should-it-be-used：
https://stackoverflow.com/questions/86582/singleton-how-should-it-be-used
> You need to have one and only one object of a type in system
你需要系统中只有唯一一个实例存在的类的全局变量的时候才使用单例。

> 如果使用单例，应该用什么样子的：
>* How to create the best singleton:
>* The smaller, the better. I am a minimalist
>* Make sure it is thread safe
>* Make sure it is never null
>* Make sure it is created only once
>* Lazy or system initialization? Up to your requirements
>* Sometimes the OS or the JVM creates singletons for you (e.g. in Java every class definition is a singleton)
>* Provide a destructor or somehow figure out how to dispose resources
>* Use little memory
越小越好，越简单越好，线程安全，内存不泄露

## 1.6 参考文章
在本文写作的过程中参考了一些博客和stackoverflow 的回答，以超链接的方式体现在文中。另外还有一些我觉得非常精彩的回答，放在下面供读者拓展阅读

推荐阅读：
高票回答中提供了一系列有益的链接(https://stackoverflow.com/questions/1008019/c-singleton-design-pattern/1008289#1008289)
面试中的单例(http://www.cnblogs.com/loveis715/archive/2012/07/18/2598409.html)
一些观点(https://segmentfault.com/q/1010000000593968)


# 2、C++ 单例模式
来源：https://zhuanlan.zhihu.com/p/37469260
https://blog.csdn.net/song240948380/article/details/123074885


# 3、单例模式-析构函数的深入理解
来源：https://blog.csdn.net/littesss/article/details/78413149


# 4、面试中的Singleton
来源：https://www.cnblogs.com/loveis715/archive/2012/07/18/2598409.html

## 4.1 引子
　　“请写一个Singleton。”面试官微笑着和我说。
　　“这可真简单。”我心里想着，并在白板上写下了下面的Singleton实现
```cpp
class Singleton
{
public:
    static Singleton& Instance()
    {
        static Singleton singleton;
        return singleton;
    }

private:
    Singleton() { };
};
```
　　那请你讲解一下该实现的各组成。”面试官的脸上仍然带着微笑。

　　“首先要说的就是Singleton的构造函数。由于Singleton限制其类型实例有且只能有一个，因此我们应通过将构造函数设置为非公有来保证其不会被用户代码随意创建。而在类型实例访问函数中，我们通过局部静态变量达到实例仅有一个的要求。另外，通过该静态变量，我们可以将该实例的创建延迟到实例访问函数被调用时才执行，以提高程序的启动速度。”

## 4.2 保护
　　说得不错，而且更可贵的是你能注意到对构造函数进行保护。毕竟中间件代码需要非常严谨才能防止用户代码的误用。那么，除了构造函数以外，我们还需要对哪些组成进行保护？”

　　“还需要保护的有拷贝构造函数，析构函数以及赋值运算符。或许，我们还需要考虑取址运算符。这是因为编译器会在需要的时候为这些成员创建一个默认的实现。”

　　“那你能详细说一下编译器会在什么情况下创建默认实现，以及创建这些默认实现的原因吗？”面试官继续问道。

　　“在这些成员没有被声明的情况下，编译器将使用一系列默认行为：对实例的构造就是分配一部分内存，而不对该部分内存做任何事情；对实例的拷贝也仅仅是将原实例中的内存按位拷贝到新实例中；而赋值运算符也是对类型实例所拥有的各信息进行拷贝。而在某些情况下，这些默认行为不再满足条件，那么编译器将尝试根据已有信息创建这些成员的默认实现。这些影响因素可以分为几种：类型所提供的相应成员，类型中的虚函数以及类型的虚基类。”

　　“就以构造函数为例，如果当前类型的成员或基类提供了由用户定义的构造函数，那么仅进行内存拷贝可能已经不是正确的行为。这是因为该成员的构造函数可能包含了成员初始化，成员函数调用等众多执行逻辑。此时编译器就需要为这个类型生成一个默认构造函数，以执行对成员或基类构造函数的调用。另外，如果一个类型声明了一个虚函数，那么编译器仍需要生成一个构造函数，以初始化指向该虚函数表的指针。如果一个类型的各个派生类中拥有一个虚基类，那么编译器同样需要生成构造函数，以初始化该虚基类的位置。这些情况同样需要在拷贝构造函数中考虑：如果一个类型的成员变量拥有一个拷贝构造函数，或者其基类拥有一个拷贝构造函数，位拷贝就不再满足要求了，因为拷贝构造函数内可能执行了某些并不是位拷贝的逻辑。同时如果一个类型声明了虚函数，拷贝构造函数需要根据目标类型初始化虚函数表指针。如基类实例经过拷贝后，其虚函数表指针不应指向派生类的虚函数表。同理，如果一个类型的各个派生类中拥有一个虚派生，那么编译器也应为其生成拷贝构造函数，以正确设置各个虚基类的偏移。”

　　“当然，析构函数的情况则略为简单一些：只需要调用其成员的析构函数以及基类的析构函数即可，而不需要再考虑对虚基类偏移的设置及虚函数表指针的设置。”

　　“在这些默认实现中，类型实例的各个原生类型成员并没有得到初始化的机会。但是这一般被认为是软件开发人员的责任，而不是编译器的责任。”说完这些，我长出一口气，心里也暗自庆幸曾经研究过该部分内容。

　　“你刚才提到需要考虑保护取址运算符，是吗？我想知道。”

　　“好的。首先要声明的是，几乎所有的人都会认为对取址运算符的重载是邪恶的。甚至说，boost为了防止该行为所产生的错误更是提供了addressof()函数。而另一方面，我们需要讨论用户为什么要用取址运算符。Singleton所返回的常常是一个引用，对引用进行取址将得到相应类型的指针。而从语法上来说，引用和指针的最大区别在于是否可以被delete关键字删除以及是否可以为NULL。但是Singleton返回一个引用也就表示其生存期由非用户代码所管理。因此使用取址运算符获得指针后又用delete关键字删除Singleton所返回的实例明显是一个用户错误。综上所述，通过将取址运算符设置为私有没有多少意义。”


## 4.3 重用
　　“好的，现在我们换个话题。如果我现在有几个类型都需要实现为Singleton，那我应怎样使用你所编写的这段代码呢？”

　　刚刚还在洋洋自得的我恍然大悟：这个Singleton实现是无法重用的。没办法，只好一边想一边说：“一般来说，较为流行的重用方法一共有三种：组合、派生以及模板。首先可以想到的是，对Singleton的重用仅仅是对Instance()函数的重用，因此通过从Singleton派生以继承该函数的实现是一个很好的选择。而Instance()函数如果能根据实际类型更改返回类型则更好了。因此奇异递归模板（CRTP，The Curiously Recurring Template Pattern）模式则是一个非常好的选择。”于是我在白板上飞快地写下了下面的代码：
```cpp
template <typename T>
class Singleton
{
public:
    static T& Instance()
    {
        static T s_Instance;
        return s_Instance;
    }

protected:
    Singleton(void) {}
    ~Singleton(void) {}

private:
    Singleton(const Singleton& rhs) {}
    Singleton& operator = (const Singleton& rhs) {}
};
```
　　同时我也在白板上写下了对该Singleton实现进行重用的方法：
```cpp
class SingletonInstance : public Singleton<SingletonInstance>…
```
　　“在需要重用该Singleton实现时，我们仅仅需要从Singleton派生并将Singleton的泛型参数设置为该类型即可。”

## 4.4 生存期管理
　　“我看你在实现中使用了静态变量，那你是否能介绍一下上面Singleton实现中有关生存期的一些特征吗？毕竟生存期管理也是编程中的一个重要话题。”面试官提出了下一个问题。

　　“嗯，让我想一想。我认为对Singleton的生存期特性的讨论需要分为两个方面：Singleton内使用的静态变量的生存期以及Singleton外在用户代码中所表现的生存期。Singleton内使用的静态变量是一个局部静态变量，因此只有在Singleton的Instance()函数被调用时其才会被创建，从而拥有了延迟初始化（Lazy）的效果，提高了程序的启动性能。同时该实例将生存至程序执行完毕。而就Singleton的用户代码而言，其生存期贯穿于整个程序生命周期，从程序启动开始直到程序执行完毕。当然，Singleton在生存期上的一个缺陷就是创建和析构时的不确定性。由于Singleton实例会在Instance()函数被访问时被创建，因此在某处新添加的一处对Singleton的访问将可能导致Singleton的生存期发生变化。如果其依赖于其它组成，如另一个Singleton，那么对它们的生存期进行管理将成为一个灾难。甚至可以说，还不如不用Singleton，而使用明确的实例生存期管理。”

　　“很好，你能提到程序初始化及关闭时单件的构造及析构顺序的不确定可能导致致命的错误这一情况。可以说，这是通过局部静态变量实现Singleton的一个重要缺点。而对于你所提到的多个Singleton之间相互关联所导致的生存期管理问题，你是否有解决该问题的方法呢？”

　　我突然间意识到自己给自己出了一个难题：“有，我们可以将Singleton的实现更改为使用全局静态变量，并将这些全局静态变量在文件中按照特定顺序排序即可。”

　　“但是这样的话，静态变量将使用eager initialization的方式完成初始化，可能会对性能影响较大。其实，我想听你说的是，对于具有关联的两个Singleton，对它们进行使用的代码常常局限在同一区域内。该问题的一个解决方法常常是将对它们进行使用的管理逻辑实现为Singleton，而在内部逻辑中对它们进行明确的生存期管理。但不用担心，因为这个答案也过于经验之谈。那么下一个问题，你既然提到了全局静态变量能解决这个问题，那是否可以讲解一下全局静态变量的生命周期是怎样的呢？”

　　“编译器会在程序的main()函数执行之前插入一段代码，用来初始化全局变量。当然，静态变量也包含在内。该过程被称为静态初始化。”

　　“嗯，很好。使用全局静态变量实现Singleton的确会对性能造成一定影响。但是你是否注意到它也有一定的优点呢？”

　　见我许久没有回答，面试官主动帮我解了围：“是线程安全性。由于在静态初始化时用户代码还没有来得及执行，因此其常常处于单线程环境下，从而保证了Singleton真的只有一个实例。当然，这并不是一个好的解决方法。所以，我们来谈谈Singleton的多线程实现吧。”

## 4.5 多线程
　　“首先请你写一个线程安全的Singleton实现。”

　　我拿起笔，在白板上写下早已烂熟于心的多线程安全实现：
```cpp
template <typename T>
class Singleton
{
public:
    static T& Instance()
    {
        if (m_pInstance == NULL)
        {
            Lock lock;
            if (m_pInstance == NULL)
            {
                m_pInstance = new T();
                atexit(Destroy);
            }
            return *m_pInstance;
        }
        return *m_pInstance;
    }

protected:
    Singleton(void) {}
    ~Singleton(void) {}

private:
    Singleton(const Singleton& rhs) {}
    Singleton& operator = (const Singleton& rhs) {}

    void Destroy()
    {
        if (m_pInstance != NULL)
            delete m_pInstance;
        m_pInstance = NULL;
    }

    static T* volatile m_pInstance;
};

template <typename T>
T* Singleton<T>::m_pInstance = NULL;
```
　　“写得很精彩。那你是否能逐行讲解一下你写的这个Singleton实现呢？”

　　“好的。首先，我使用了一个指针记录创建的Singleton实例，而不再是局部静态变量。这是因为局部静态变量可能在多线程环境下出现问题。”

　　“我想插一句话，为什么局部静态变量会在多线程环境下出现问题？”

　　“这是由局部静态变量的实际实现所决定的。为了能满足局部静态变量只被初始化一次的需求，很多编译器会通过一个全局的标志位记录该静态变量是否已经被初始化的信息。那么，对静态变量进行初始化的伪码就变成下面这个样子：”。
```cpp
bool flag = false;
if (!flag)
{
    flag = true;
    staticVar = initStatic();
}
```
　　“那么在第一个线程执行完对flag的检查并进入if分支后，第二个线程将可能被启动，从而也进入if分支。这样，两个线程都将执行对静态变量的初始化。因此在这里，我使用了指针，并在对指针进行赋值之前使用锁保证在同一时间内只能有一个线程对指针进行初始化。同时基于性能的考虑，我们需要在每次访问实例之前检查指针是否已经经过初始化，以避免每次对Singleton的访问都需要请求对锁的控制权。”

　　“同时，”我咽了口口水继续说，“因为new运算符的调用分为分配内存、调用构造函数以及为指针赋值三步，就像下面的构造函数调用：”
```cpp
SingletonInstance pInstance = new SingletonInstance();
```
　　“这行代码会转化为以下形式：”
```cpp
SingletonInstance pHeap = __new(sizeof(SingletonInstance));
pHeap->SingletonInstance::SingletonInstance();
SingletonInstance pInstance = pHeap;
```
　　“这样转换是因为在C++ 标准中规定，如果内存分配失败，或者构造函数没有成功执行， new运算符所返回的将是空。一般情况下，编译器不会轻易调整这三步的执行顺序，但是在满足特定条件时，如构造函数不会抛出异常等，编译器可能出于优化的目的将第一步和第三步合并为同一步：”
```cpp
SingletonInstance pInstance = __new(sizeof(SingletonInstance));
pInstance->SingletonInstance::SingletonInstance();
```
　　“这样就可能导致其中一个线程在完成了内存分配后就被切换到另一线程，而另一线程对Singleton的再次访问将由于pInstance已经赋值而越过if分支，从而返回一个不完整的对象。因此，我在这个实现中为静态成员指针添加了volatile关键字。该关键字的实际意义是由其修饰的变量可能会被意想不到地改变，因此每次对其所修饰的变量进行操作都需要从内存中取得它的实际值。它可以用来阻止编译器对指令顺序的调整。只是由于该关键字所提供的禁止重排代码是假定在单线程环境下的，因此并不能禁止多线程环境下的指令重排。”

　　“最后来说说我对atexit()关键字的使用。在通过new关键字创建类型实例的时候，我们同时通过atexit()函数注册了释放该实例的函数，从而保证了这些实例能够在程序退出前正确地析构。该函数的特性也能保证后被创建的实例首先被析构。其实，对静态类型实例进行析构的过程与前面所提到的在main()函数执行之前插入静态初始化逻辑相对应。

## 4.6 引用还是指针
　　“既然你在实现中使用了指针，为什么仍然在Instance()函数中返回引用呢？”面试官又抛出了新的问题。

　　“这是因为Singleton返回的实例的生存期是由Singleton本身所决定的，而不是用户代码。我们知道，指针和引用在语法上的最大区别就是指针可以为NULL，并可以通过delete运算符删除指针所指的实例，而引用则不可以。由该语法区别引申出的语义区别之一就是这些实例的生存期意义：通过引用所返回的实例，生存期由非用户代码管理，而通过指针返回的实例，其可能在某个时间点没有被创建，或是可以被删除的。但是这两条Singleton都不满足，因此在这里，我使用指针，而不是引用。”

　　“指针和引用除了你提到的这些之外，还有其它的区别吗？”

　　“有的。指针和引用的区别主要存在于几个方面。从低层次向高层次上来说，分为编译器实现上的，语法上的以及语义上的区别。就编译器的实现来说，声明一个引用并没有为引用分配内存，而仅仅是为该变量赋予了一个别名。而声明一个指针则分配了内存。这种实现上的差异就导致了语法上的众多区别：对引用进行更改将导致其原本指向的实例被赋值，而对指针进行更改将导致其指向另一个实例；引用将永远指向一个类型实例，从而导致其不能为NULL，并由于该限制而导致了众多语法上的区别，如dynamic_cast对引用和指针在无法成功进行转化时的行为不一致。而就语义而言，前面所提到的生存期语义是一个区别，同时一个返回引用的函数常常保证其返回结果有效。一般来说，语义区别的根源常常是语法上的区别，因此上面的语义区别仅仅是列举了一些例子，而真正语义上的差别常常需要考虑它们的语境。”

　　“你在前面说到了你的多线程内部实现使用了指针，而返回类型是引用。在编写过程中，你是否考虑了实例构造不成功的情况，如new运算符运行失败？”

　　“是的。在和其它人进行讨论的过程中，大家对于这种问题有各自的理解。首先，对一个实例的构造将可能在两处抛出异常：new运算符的执行以及构造函数抛出的异常。对于new运算符，我想说的是几点。对于某些操作系统，例如Windows，其常常使用虚拟地址，因此其运行常常不受物理内存实际大小的限制。而对于构造函数中抛出的异常，我们有两种策略可以选择：在构造函数内对异常进行处理，以及在构造函数之外对异常进行处理。在构造函数内对异常进行处理可以保证类型实例处于一个有效的状态，但一般不是我们想要的实例状态。这样一个实例会导致后面对它的使用更为繁琐，例如需要更多的处理逻辑或再次导致程序执行异常。反过来，在构造函数之外对异常进行处理常常是更好的选择，因为软件开发人员可以根据产生异常时所构造的实例的状态将一定范围内的各个变量更改为合法的状态。举例来说，我们在一个函数中尝试创建一对相互关联的类型实例，那么在一个实例的构造函数抛出了异常时，我们不应该在构造函数里对该实例的状态进行维护，因为前一个实例的构造是按照后一个实例会正常创建来进行的。相对来说，放弃后一个实例，并将前一个实例删除是一个比较好的选择。”

　　我在白板上比划了一下，继续说到：“我们知道，异常有两个非常明显的缺陷：效率，以及对代码的污染。在太小的粒度中使用异常，就会导致异常使用次数的增加，对于效率以及代码的整洁型都是伤害。同样地，对拷贝构造函数等组成常常需要使用类似的原则。”

　　“反过来说，Singleton的使用也可以保持着这种原则。Singleton仅仅是一个包装好的全局实例，对其的创建如果一旦不成功，在较高层次上保持正常状态同样是一个较好的选择。”

## 4.7 Anti-Patten
　　“既然你提到了Singleton仅仅是一个包装好的全局变量，那你能说说它和全局变量的相同与不同么？”

　　“单件可以说是全局变量的替代品。其拥有全局变量的众多特点：全局可见且贯穿应用程序的整个生命周期。除此之外，单件模式还拥有一些全局变量所不具有的性质：同一类型的对象实例只能有一个，同时适当的实现还拥有延迟初始化（Lazy）的功能，可以避免耗时的全局变量初始化所导致的启动速度不佳等问题。要说明的是，Singleton的最主要目的并不是作为一个全局变量使用，而是保证类型实例有且仅有一个。它所具有的全局访问特性仅仅是它的一个副作用。但正是这个副作用使它更类似于包装好的全局变量，从而允许各部分代码对其直接进行操作。软件开发人员需要通过仔细地阅读各部分对其进行操作的代码才能了解其真正的使用方式，而不能通过接口得到组件依赖性等信息。如果Singleton记录了程序的运行状态，那么该状态将是一个全局状态。各个组件对其进行操作的调用时序将变得十分重要，从而使各个组件之间存在着一种隐式的依赖。”

　　“从语法上来讲，首先Singleton模式实际上将类型功能与类型实例个数限制的代码混合在了一起，违反了SRP。其次Singleton模式在Instance()函数中将创建一个确定的类型，从而禁止了通过多态提供另一种实现的可能。”

　　“但是从系统的角度来讲，对Singleton的使用则是无法避免的：假设一个系统拥有成百上千个服务，那么对它们的传递将会成为系统的一个灾难。从微软所提供的众多类库上来看，其常常提供一种方式获得服务的函数，如GetService()等。另外一个可以减轻Singleton模式所带来不良影响的方法则是为Singleton模式提供无状态或状态关联很小的实现。”

　　“也就是说，Singleton本身并不是一个非常差的模式，对其使用的关键在于何时使用它并正确的使用它。”

　　面试官抬起手腕看了看时间：“好了，时间已经到了。你的C++ 功底已经很好了。我相信，我们会在不久的将来成为同事。”

 

笔者注：这本是Writing Patterns Line by Line的一篇文章

# 5、单例实现 来源：weiyingtao
```cpp
//***************************************************************
// create date:  2022/05/06
// author:     weiyingtao
// description:     单例模板
//***************************************************************

#pragma once

template <typename T>
class TSingleton
{
private:
	struct Destroy
	{
		explicit Destroy(T** p)
			: m_p(p)
		{
		}
		~Destroy()
		{
			if (*m_p)
			{
				delete (*m_p);
				*m_p = nullptr;
				m_p = nullptr;
			}
		}
		T** m_p;
	};

public:
	typedef T SingletonType;

public:
	typedef T SingleInstanceType;

	/// <获取单例>
	static SingleInstanceType* Singleton();

	/// <释放单例>
	static void Release();

private:
	/// <单例实体指针>
	static SingleInstanceType* p_single_instance_;

private:
	TSingleton(const TSingleton&);
	const TSingleton& operator=(const TSingleton&);

private:
	/// <此类不可继承，不可实例化>
	TSingleton() {}
	~TSingleton() {}

private:
	static struct StaticInit
	{
		StaticInit() { TSingleton<T>::Singleton(); }
	} s_init_;
};


/// <类ChaSingleInstance的定义>
template <typename T>
typename TSingleton<T>::SingleInstanceType*
TSingleton<T>::p_single_instance_ = nullptr;

template <typename T>
typename TSingleton<T>::SingleInstanceType* TSingleton<T>::Singleton()
{
	if (nullptr == p_single_instance_)
	{
		p_single_instance_ = new SingleInstanceType();
		static Destroy des(&p_single_instance_);
	}
	return p_single_instance_;
}

template <typename T>
void TSingleton<T>::Release()
{
	if (p_single_instance_)
	{
		delete p_single_instance_;
		p_single_instance_ = nullptr;
	}
}

template <typename T>
typename TSingleton<T>::StaticInit TSingleton<T>::s_init_;

```


# C++ 设计模式之观察者模式(observer)(行为型)
来源：https://blog.csdn.net/janeqi1987/article/details/104048035

## 一、定义
**观察者模式**：定义了一种一对多的依赖关系，让多个观察者对象同时监听某一主题对象。这个主题对象在状态发生变化时，会通知所有观察者对象，使他们能够自动更新自己。

观察者模式又叫做发布-订阅(Publish-Subscribe)模式、模型-视图(Model-View)模式、源-监听器(Source-Listener)模式。

Observer模式应该可以说是应用最多、影响最广的模式之一，因为Observer的一个实例Model/View/Control（MVC）结构在系统开发架构设计中有着很重要的地位和意义，MVC实现了业务逻辑和表示层的解耦。

![enter description here](./images/1695868506223.png)
观测模式允许一个对象关注其他对象的状态，并且，观测模式还为被观测者提供了一种观测结构，或者说是一个主体和一个客体。主体，也就是被观测者，可以用来联系所有的观测它的观测者。客体，也就是观测者，用来接受主体状态的改变 观测就是一个可被观测的类（也就是主题）与一个或多个观测它的类（也就是客体）的协作。不论什么时候，当被观测对象的状态变化时，所有注册过的观测者都会得到通知。

观测模式将被观测者（主体）从观测者（客体）种分离出来。这样，每个观测者都可以根据主体的变化分别采取各自的操作。（观测模式和Publish/Subscribe模式一样，也是一种有效描述对象间相互作用的模式。）观测模式灵活而且功能强大。对于被观测者来说，那些查询哪些类需要自己的状态信息和每次使用那些状态信息的额外资源开销已经不存在了。另外，一个观测者可以在任何合适的时候进行注册和取消注册。你也可以定义多个具体的观测类，以便在实际应用中执行不同的操作。

将一个系统分割成一系列相互协作的类有一个常见的副作用：需要维护相关对象间的一致性。我们不希望为了维持一致性而使各类紧密耦合，因为这样降低了它们的可重用性。

观察者模式所做的工作其实就是在解除耦合，让耦合的双方都依赖于抽象，而不是依赖于具体。从而使得各自的变化都不会影响另一边的变化。

## 二、ULM图

![enter description here](./images/1695868636624.png)
### 角色：

* **Subject（目标）**：目标又称为主题，它是指被观察的对象。在目标中定义了一个观察者集合，一个观察目标可以接受任意数量的观察者来观察，它提供一系列方法来增加和删除观察者对象，同时它定义了通知方法notify()。目标类可以是接口，也可以是抽象类或具体类。 

* **ConcreteSubject（具体目标）**：具体目标是目标类的子类，通常它包含有经常发生改变的数据，当它的状态发生改变时，向它的各个观察者发出通知；同时它还实现了在目标类中定义的抽象业务逻辑方法（如果有的话）。如果无须扩展目标类，则具体目标类可以省略。 

* **Observer（观察者）**：观察者将对观察目标的改变做出反应，观察者一般定义为接口，该接口声明了更新数据的方法collect()，因此又称为抽象观察者。 

* **ConcreteObserver（具体观察者）**：在具体观察者中维护一个指向具体目标对象的引用，它存储具体观察者的有关状态，这些状态需要和具体目标的状态保持一致；它实现了在抽象观察者Observer中定义的collect()方法。

一共四个类，两个接口，两个接口实现类，被观察者方法参数引用的是观察者对象。

观察者只定义一个自己的行为。具体观察者重写观察者的行为后还提供了构造方法为客户端调用。具体被观察者，这个类有些复杂，简单的说就是把外界消息通过该类的方法调用 notifyObserver() 方法给提前准备好的Observer (观察者) 接口中的方法去发送。

### 模式优点：

1. 观察者模式可以实现表示层和数据逻辑层的分离,并定义了稳定的消息更新传递机制，抽象了更新接口，使得可以有各种各样不同的表示层作为具体观察者角色。

2. 在观察目标和观察者之间建立一个抽象的耦合 ：一个目标所知道的仅仅是它有一系列观察者 , 每个都符合抽象的Observer类的简单接口。目标不知道任何一个观察者属于哪一个具体的类。这样目标和观察者之间的耦合是抽象的和最小的。因为目标和观察者不是紧密耦合的, 它们可以属于一个系统中的不同抽象层次。一个处于较低层次的目标对象可与一个处于较高层次的观察者通信并通知它 , 这样就保持了系统层次的完整。如果目标和观察者混在一块 , 那么得到的对象要么横贯两个层次 (违反了层次性), 要么必须放在这两层的某一层中(这可能会损害层次抽象)。

3. 支持广播通信 :不像通常的请求, 目标发送的通知不需指定它的接收者。通知被自动广播给所有已向该目标对象登记的有关对象。目标对象并不关心到底有多少对象对自己感兴趣 ;它唯一的责任就是通知它的各观察者。这给了你在任何时刻增加和删除观察者的自由。处理还是忽略一个通知取决于观察者。

4. 观察者模式符合“开闭原则”的要求。

### 模式缺点：

1. 如果一个观察目标对象有很多直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间。

2. 如果在观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行循环调用，可能导致系统崩溃。

3. 观察者模式没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化。

4. 意外的更新 因为一个观察者并不知道其它观察者的存在 , 它可能对改变目标的最终代价一无所知。在目标上一个看似无害的的操作可能会引起一系列对观察者以及依赖于这些观察者的那些对象的更新。此外 , 如果依赖准则的定义或维护不当，常常会引起错误的更新 , 这种错误通常很难捕捉。

### 适用场景：

1. 当一个对象的改变需要同时改变其他对象的时候；

2. 而且不知道具体有多少对象有待改变时，应该考虑使用观察者模式；

3. 当一个抽象模型有两个方面，其中一方面依赖于另一方面，这时用观察者模式可以将这两者封装在独立的对象中使他们各自独立地改变和复用。

## 三、实例
### 1. 博客

当博主发表新文章的时候，即博主状态发生了改变，那些订阅的读者就会收到通知，然后进行相应的动作，比如去看文章，或者收藏起来。博主与读者之间存在种一对多的依赖关系。下面给出相应的UML图设计。 

![enter description here](./images/1695869095245.png)
可以看到博客类中有一个观察者链表（即订阅者），当博客的状态发生变化时，通过Notify成员函数通知所有的观察者，告诉他们博客的状态更新了。而观察者通过Update成员函数获取博客的状态信息。

```c++
#include <iostream>
#include <string>
#include <list>
#include <memory>
 
//observer
class Observer
{
public:
	Observer() = default;
	virtual ~Observer() = default;
	virtual void Update() = 0;
};
 
//subject
class Blog
{
public:
	Blog() = default;
	virtual ~Blog() = default;
	void Attach(std::shared_ptr<Observer> observer)
	{ 
		observers.push_back(observer); 
	}
	void Remove(std::shared_ptr<Observer> observer)
	{ 
		observers.remove(observer); 
	}
	void Notify()
	{
		for (auto iter = observers.begin(); iter != observers.end(); iter++)
		{
			(*iter)->Update();
		}
	}
	virtual void SetStatus(const std::string& status)
	{ 
		this->status = status;
	}
	virtual std::string GetStatus() 
	{ 
		return status; 
	}
private:
	std::list<std::shared_ptr<Observer>> observers; //观察者链表
protected:
	std::string status; //状态
};
 
//concrete subject
class BlogCSDN : public Blog
{
public:
	BlogCSDN(const std::string& bloggerName) : bloggerName{ bloggerName }{}
	~BlogCSDN() = default;
 
	void SetStatus(const std::string& state) 
	{ 
		status = "CSDN通知 : " + bloggerName + state;
	} 
	std::string GetStatus() 
	{
		return status; 
	}
private:
	std::string bloggerName;
};
 
//concreteObserver
class ObserverBlog : public Observer
{
private:
	std::string observerName;
	std::shared_ptr<Blog> blog;   //观察的博客，当然以链表形式更好，就可以观察多个博客
public:
	ObserverBlog(const std::string& observerName, std::shared_ptr<Blog> blog)
		: observerName{ observerName }
		, blog{ blog }
	{}
	~ObserverBlog() = default;
	void Update()
	{
		std::string status = blog->GetStatus();//收到更新通知后,阅读关注博客更新的内容等操作
		std::cout << observerName << status << std::endl;
	}
};
 
int main()
{
	auto blog = std::make_shared<BlogCSDN>("秋云");//创建一个CSDN博客的博主---秋云
	auto observer = std::make_shared<ObserverBlog>("子之", blog);//子之关注了秋云的博客
	blog->Attach(observer);//把子之放到了博主秋云的粉丝集合中
	blog->SetStatus("发表设计模式——观察者模式");//SetStatus是博主更新了内容。
	blog->Notify();
 
	return 0;
}
```

### 2. 猫狗老鼠
猫"喵喵"叫了一声，狗也跟着"汪汪"叫,老鼠听到猫叫声后跑了。

![enter description here](./images/1695869161646.png)
```c++
#include <iostream>
#include <vector>
 
//Observer
class MyObserver 
{
public:
	virtual void response() = 0;
};
 
//Subject
class MySubject 
{
public:
 
	void attach(MyObserver* observer) 
	{
		observers.push_back(observer);
	}
 
	void detach(MyObserver* observer) 
	{
		for (auto it = observers.begin(); it != observers.end(); it++) 
		{
			if (*it == observer) {
				observers.erase(it);
				break;
			}
		}
	}
protected:
	std::vector<MyObserver*> observers;
};
 
//concrete subject
class Cat : public MySubject 
{
public:
	void cry() 
	{
		std::cout << "猫喵喵叫!" << std::endl;
		for (auto obs : observers) 
		{
			obs->response();
		}
	}
};
 
//concrete observer
class Mouse : public MyObserver 
{
public:
	void response() 
	{
		std::cout << "老鼠逃跑!" << std::endl;
	}
};
 
//concrete observer
class Dog : public MyObserver 
{
public:
	void response() 
	{
		std::cout << "狗汪汪叫!" << std::endl;
	}
};
 
 
int main(void) 
{
	Cat cat;
	Mouse m1, m2;
	Dog dog;
 
	cat.attach(&m1);
	cat.attach(&m2);
	cat.attach(&dog);
 
	cat.cry();
 
	return 0;
}
```

运行结果如下：

![enter description here](./images/1695869210295.png)


# C++ 屌屌的观察者模式-同步回调和异步回调
来源：https://www.cnblogs.com/swarmbees/p/11155072.html

## 一、概述
说起观察者模式，也是比较简单的一种模式了，稍微工作有1年经验的同学，写起来都是666...

**想看观察者模式的说明可以直接上 菜鸟教程|观察者模式 （ https://www.runoob.com/design-pattern/observer-pattern.html ）这个地址去看。**

本篇文章其实就是一个简单的观察者模式，只是使用了模板的方式，把我们的回调接口进行了参数化，这样有什么好处呢？

好处当然是大大的有了。 平时我们在不同业务逻辑之间写观察者模式呢，都得写好多个，大家有没有发现，所有的被观察者Subject其实很多操作都是一样的。

本篇我们带来两种观察者模式：同步观察者和异步观察者

### 1、同步观察者
顾名思义，同步观察者其实就是不管是谁，触发了Subject的Update操作，该操作都是同步进行的，他会调用所有的观察者(Observer)的OnUpdate接口，来通知Observer处理改变操作。

如效果展示图中的第一个`单次拉取`页签，当我们点击拉取按钮时，就相当于触发了一次Subject对象的Update操作

### 2、异步观察者
异步观察者模式上和同步观察者基本一样，只是在事件处理上有稍微不同

1. 执行Update操作是由Subject自己去完成的
2. 调用Observer的OnUpdate回调接口时，处于工作线程中
3. Subject所有的请求操作都是在工作现场中进行

如效果图所示，`定时拉取`观察者模式，Subject启动了一个后台线程，3秒钟拉取一次数据，并回调到界面

## 二、效果展示
如下图所示，是一个简单的观察者模式事例。

单次拉取：演示了同步观察者模式

定时拉取：演示了异步观察者模式

动图：https://images.cnblogs.com/cnblogs_com/swarmbees/1454266/o_subject_syn_asyn.gif

工程结构如图所示，这里只把头文件的目录展示出来了。

实现文件的目录和头文件类似，为了截图方便所以做了隐藏操作。

![enter description here](./images/1695915722927.png)
Header Files目录下有2个虚拟文件夹，分别就是对单次拉取和定时拉取功能的实践


## 三、同步观察者
1、首先就是定义一堆接口和回调参数
```c++
struct DataItem
{
	std::string		strID;	
	std::string		strName;		
};

typedef IUpdate1<DataItem>			ISignalObserver;

//单次回调
struct ISignal : public SubjectBase<ISignalObserver>
{
	virtual void RequestData() = 0;
};

```

2、业务观察者
这里我定义了一个SignalResponse业务观察者，也就是我们在开发工程中的实际功能类。
```c++
class SignalResponse : public ISignal
{
public:
	SignalResponse();
	~SignalResponse();

public:
	virtual void RequestData() override;

private:
	
};

```

3、获取观察者指针*
通过一个门面接口获取观察者指针
1. 调用ISignal的Attach接口，就可以把自己添加到观察者列表。
2. 调用ISignal的RequestData接口，就可以拉取数据。
3. 调用ISignal的Detach接口，就可以把自己从观察者列表中移除。

```c++
ISignal * GetSignalCommon();

```

4、UI界面
接下来就是写一个UI界面啦，当我们通过上一步调用拉取数据接口后，我们的UI上相应的OnUpdate接口就会被回调
```c++
class SignalWidget : public QWidget, public ISignalObserver
{
	Q_OBJECT

public:
	SignalWidget(QWidget * parent = 0);
	~SignalWidget();

protected:
	virtual void OnUpdate(const DataItem &) override;

private slots:
	void on_pushButton_clicked();

private:
	Ui::SignalWidget *ui;
};

```

通过以上四步，就可以很方便的实现一个现在业务中的观察者，是不是很简单呢，编写过程中，需要完成这几个地方

1. 需要定义我们回调函数的参数结构
2. 需要实例化一个被观察者接口类
3. 实例化一个业务观察者
4.做一个UI界面，并集成第二步实例化的被观察者的模板参数(接口类)

注意看这里的ISignalObserver，是不是很眼熟，其实他就是我们的模板被观察者SubjectBase的模板参数。

讲到这里，大家是不是都很关心这个模板观察者到底是何方神圣，居然这么叼。那么接下来就是模板SubjectBase出场啦。。。

下面我直接给出代码，学过C++ 的同学阅读起来应该都不难。

```c++
template <typename T>
struct ISubject
{
	virtual void Attach(T * pObserver) = 0;
	virtual void Detach(T * pObserver) = 0;
};

template <typename P>
struct IUpdate1
{
	virtual void OnUpdate(const P& data) = 0;
};

template <typename P1, typename P2>
struct IUpdate2
{
	virtual void OnUpdate2(const P1 & p1, const P2 & p2) = 0;
};

template <typename P>
struct IUpdate1_P
{
	virtual void OnUpdate(const P * data) = 0;
};

template <typename T>
struct SubjectBase
{
public:
	virtual void Attach(T * pObserver)
	{
		std::lock_guard<std::mutex> lg(m_mutex);
#ifdef _DEBUG
		if (m_observers.end() != std::find(m_observers.begin(), m_observers.end(), pObserver))
		{
			assert(false);
		}
#endif // _DEBUG
		m_observers.push_back(pObserver);
	}

	virtual void Detach(T * pObserver)
	{
		std::lock_guard<std::mutex> lg(m_mutex);
		auto it = std::find(m_observers.begin(), m_observers.end(), pObserver);
		if (it != m_observers.end())
		{
			m_observers.erase(it);
		}
		else
		{
			assert(false);
		}
	}

	//protected:
	template <typename P>
	void UpdateImpl(const P & data)
	{
		std::lock_guard<mutex> lg(m_mutex);
		for (T * observer : m_observers)
		{
			observer->OnUpdate(data);
		}
	}

	template <typename P>
	void UpdateImpl(P & data)
	{
		std::lock_guard<std::mutex> lg(m_mutex);
		for (T* observer : m_observers)
		{
			observer->OnUpdate(data);
		}
	}

	template <typename P1, typename P2>
	void UpdateImpl(const P1& p1, const P2& p2)
	{
		std::lock_guard<mutex> lg(m_mutex);
		for (T* observer : m_observers)
		{
			observer->OnUpdate2(p1, p2);
		}
	}

	template <typename P1, typename P2>
	void UpdateImpl(P1& p1, P2& p2)
	{
		std::lock_guard<mutex> lg(m_mutex);
		for (T* observer : m_observers)
		{
			observer->OnUpdate2(p1, p2);
		}
	}

	template <typename P>
	void UpdateImpl(const P * data)
	{
		std::lock_guard<mutex> lg(m_mutex);
		for (T * observer : m_observers)
		{
			observer->OnUpdate(data);
		}
	}

	template <typename P>
	void UpdateImpl(P * data)
	{
		std::lock_guard<mutex> lg(m_mutex);
		for (T* observer : m_observers)
		{
			observer->OnUpdate(data);
		}
	}

protected:
	std::mutex		m_mutex;
	std::list<T *>	m_observers;
};


```

## 四、异步观察者
异步观察者的实现和同步观察者的结构基本一样，都是使用同样的套路，唯一有区别的地方就是，异步观察者所有的逻辑处理操作都是在工作线程中的。

由于ITimerSubject和SubjectBase很多接口都是一样的，因此我这里就只把差异的部分贴出来。

1、线程
ITimerSubject对象在构造时，就启动了一个线程，然后在线程中定时执行TimerNotify函数
```c++
ITimerSubject()
{
	m_thread = std::thread(std::bind(&ITimerSubject::TimerNotify, this));
}

virtual ~ITimerSubject()
{
	m_thread.join();
}

```

再来看下定时处理任务这个函数，这个函数本身是用boost的库实现我的，我改成C++ 11的模式的，新城退出这块有些问题，我没有处理，这个也不是本篇文章的核心要讲解的东西。

>怎么优雅的退出std::thread，这个从网上查下资料吧，我能想到的也就是加一个标识，然后子线程去判断。如果大家有更好的办法的话可以私信我，或者在底部留言。

```c++
void TimerNotify()
{
	for (;;)
	{
		//std::this_thread::interruption_point();

		bool bNotify = false;
		{
			std::lock_guard<std::mutex> lg(m_mutex);
			bNotify = m_sleeping_observers.size() < m_observers.size() ? true : false;
		}

		if (bNotify)
		{
			OnTimerNotify();
		}

		//std::this_thread::interruption_point();

		std::chrono::milliseconds timespan(GetTimerInterval() * 1000); // or whatever
		std::this_thread::sleep_for(timespan);
	}
}

```

2、定义一堆接口和回调参数
```c++
struct TimerDataItem
{
	std::string		strID;
	std::string		strName;
};

typedef IUpdate1<TimerDataItem>		ITimerObserver;

//定时回调
struct ITimer : public ITimerSubject<ITimerObserver, std::string, TimerDataItem>{};

```

3、业务观察者
这里我定义了一个TimerResponse业务观察者，也就是我们在开发工程中的实际功能类。
```c++
class TimerResponse : public ITimer
{
public:
	TimerResponse();
	~TimerResponse();

protected:
	virtual void OnNotify() override;

private:
	
};

```

TimerResponse::OnNotify()这个接口的实现就像这样，这里需要注意的一点是，这个函数的执行位于工作线程中，也就意味着UI界面的回调函数也在工作线程中，操作UI界面时，一定需要抛事件到UI线程中。
```c++
void TimerResponse::OnNotify()
{
	static int id = 0;
	static std::string name = "miki";
	id += 1;
	TimerDataItem item;

	std::stringstream ss;
	ss << "timer" << id;

	item.strID = ss.str();
	item.strName = name;

	UpdateImpl(item);
}

```

OnNotify会定时被调用，然后去更新UI上的内容。

4、获取观察者指针
通过一个门面接口获取观察者指针，调用ITimer的Attach接口把自己添加到观察者列表，然后就可以定时获取到数据，反之也能把自己从观察者列表中移除，并停止接收到数据。
```c++
ITimer * GetTimerCommon();

```

5、UI界面
定时回调功能测试界面

1. on_pushButton_clicked槽函数只是为了把当前线程唤醒，并定时回调
2. OnUpdate属于定时回调接口

```c++
class TimerWidget : public QWidget, public ITimerObserver
{
	Q_OBJECT

public:
	TimerWidget(QWidget *parent = 0);
	~TimerWidget();

protected:
	virtual void OnUpdate(const TimerDataItem &) override;

private slots:
	void on_pushButton_clicked();

signals:
	void RerfushData(TimerDataItem);

private:
	Ui::TimerWidget *ui;
};

```

上边也强调过了，OnUpdate的执行是在工作线程中的，因此实现的时候，如果涉及到访问UI界面，一定要注意切换线程
```c++
void TimerWidget::OnUpdate(const TimerDataItem & item)
{
	//注意这里的定时回调都在工作线程中 需要切换到主线程

	emit RerfushData(item);
}

```


