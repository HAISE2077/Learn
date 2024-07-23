---
title: C++语法
---

# 1、全局变量
## 1.1 什么是全局变量
C++ 中全局变量一般指定义在函数体外的变量。

全局变量按可访问性可分为**外部变量**和**内部变量**。

## 1.2 内部变量和外部变量的定义
**内部变量**：使用了`static`关键字修饰的全局变量。它的可访问范围（作用域）被限定在本源文件所在的链接文件模块中，不能被其他文件模块引用。

**外部变量**：没有被`static`修饰的全局变量。其他文件模块可以通过`extern`关键字引用该全局变量并访问。

## 1.3 全局变量的声明和定义
全局变量在整个源文件的作用域都是有效的，只需要在一个源文件中定义全局变量，在其他不包含全局变量定义的源文件中用`extern`关键字再次声明这个全局变量即可。

也可以在一个源文件中定义这个全局变量，在头文件中用`extern`关键字再次声明这个全局变量，如果其它源文件要用到这个全局变量，只需要包含这个头文件就可以直接使用了。
```cpp
/*global.h如下：*/

#ifndef GLOBAL_H
#define GLOBAL_H

extern int g_i; //用extern关键字，显示表明只声明不定义;全局变量最好用g开头

#endif
```

```cpp
/*global.cpp如下*/

include "global.h"

int g_i=0; //定义变量
```

## 1.4 全局变量放在头文件会出现哪些问题？
问题一：对内部变量来讲，每个include该头文件的**文件模块**中都会单独为这个内部变量分配静态内存空间，这个空间是相对独立的，是一种空间的浪费，同时还失去全局变量访问一致性的问题，没有什么意义。如果这个头文件只被一个模块使用，对于这个文件模块来说应该没啥问题。

问题二：**对外部变量来讲，这个头文件被多个文件模块include的情况下，链接过程会报错（即使有`#ifndef`和`#pragma once`也一样），因为所有include和这个头文件的模块都会有这个全局变量**。在这个头文件仅仅只被一个模块include的时候可以正常使用。


# 2、static 静态变量
来源：https://blog.csdn.net/lgfun/article/details/105810039
## 2.1 静态变量的声明和定义
c++ 静态变量（const整数类型除外）一定要在类外进行定义和初始化（类内的只是声明，非初始化，不会分配内存）

## 2.2 C++ 中static对象的初始化
### 2.2.1 non-local static对象（函数外）
C++ 规定，non-local static 对象的初始化发生在main函数执行之前，也即main函数之前的单线程启动阶段，所以不存在线程安全问题。但C++ 没有规定多个non-local static 对象的初始化顺序，尤其是来自多个编译单元的non-local static对象，他们的初始化顺序是随机的。

### 2.2.2 local static对象（函数内）
C++ 11规定了local static在多线程条件下的初始化行为，要求编译器保证了内部静态变量的线程安全性。

对于local static 对象，其初始化发生在控制流第一次执行到该对象的初始化语句时。多个线程的控制流可能同时到达其初始化语句。

在C++ 11之前，在多线程环境下local static对象的初始化并不是线程安全的。具体表现就是：如果一个线程正在执行local static对象的初始化语句但还没有完成初始化，此时若其它线程也执行到该语句，那么这个线程会认为自己是第一次执行该语句并进入该local static对象的构造函数中。这会造成这个local static对象的重复构造，进而产生内存泄露问题。所以，local static对象在多线程环境下的重复构造问题是需要解决的。

而C++ 11则在语言规范中解决了这个问题。C++ 11规定，在一个线程开始local static 对象的初始化后到完成初始化前，其他线程执行到这个local static对象的初始化语句就会等待，直到该local static 对象初始化完成。


# 3、C++ 中STL常用容器的优点和缺点
来源：https://www.cnblogs.com/steamedbun/p/9376403.html

## 3.1 vector
vector类似于C语言中的数组，它维护一段连续的内存空间，具有固定的起始地址，因而能非常方便地进行随机存取，即 `[]` 操作符，但因为它的内存区域是连续的，所以在它中间插入或删除某个元素，需要复制并移动现有的元素。此外，当被插入的内存空间不够时，需要重新申请一块足够大的内存并进行内存拷贝。值得注意的是，vector每次扩容为原来的两倍，对小对象来说执行效率高，但如果遇到大对象，执行效率就低了。

## 3.2 list
list类似于C语言中的双向链表，它通过指针来进行数据的访问，因此维护的内存空间可以不连续，这也非常有利于数据的随机存取，因而它没有提供 `[]` 操作符重载。

## 3.3 deque
deque类似于C语言中的双向队列，即两端都可以插入或者删除的队列。queue支持 `[]` 操作符，也就是支持随机存取，而且跟vector的效率相差无几。它支持两端的操作：push_back,push_front,pop_back,pop_front等，并且在两端操作上与list的效率也差不多。或者我们可以这么认为，deque是vector跟list的折中。

## 3.4 map
map类似于数据库中的１:１关系，它是一种关联容器，提供一对一(C++ primer中文版中将第一个译为键，每个键只能在map中出现一次，第二个被译为该键对应的值)的数据处理能力，这种特性了使得map类似于数据结构里的红黑二叉树。

## 3.5 multimap
multimap类似于数据库中的１:Ｎ关系，它是一种关联容器,提供一对多的数据处理能力。

## 3.6 set
set类似于数学里面的集合，不过set的集合中不包含重复的元素，这是和vector的第一个区别，第二个区别是set内部用平衡二叉树实现，便于元素查找，而vector是使用连续内存存储，便于随机存取。

## 3.7 multiset
multiset类似于数学里面的集合，集合中可以包含重复的元素。

## 3.8 总结
1、如果需要高效的随机存取，不在乎插入和删除的效率，使用vector；
2、如果需要大量的插入和删除元素，不关心随机存取的效率，使用list；
3、如果需要随机存取，并且关心两端数据的插入和删除效率，使用deque；
4、如果打算存储数据字典，并且要求方便地根据key找到value，一对一的情况使用map，一对多的情况使用multimap；
5、如果打算查找一个元素是否存在于某集合中，唯一存在的情况使用set，不唯一存在的情况使用multiset。


# 4、C++ STL 各容器简单介绍
来源：https://zhuanlan.zhihu.com/p/130905242?utm_source=qq

## 4.1 什么是STL?
STL（Standard Template Library），即标准模板库，是一个高效的C++ 程序库，包含了诸多常用的基本数据结构和基本算法。为广大C++ 程序员们提供了一个可扩展的应用框架，高度体现了软件的可复用性。

从逻辑层次来看，在STL中体现了泛型化程序设计的思想（generic programming）。在这种思想里，大部分基本算法被抽象，被泛化，独立于与之对应的数据结构，用于以相同或相近的方式处理各种不同情形。

从实现层次看，整个STL是以一种类型参数化（type parameterized）的方式实现的，基于模板（template）。

STL有六大组件，但主要包含容器、迭代器和算法三个部分。
* 容器（Containers）：用来管理某类对象的集合。每一种容器都有其优点和缺点，所以，为了应付程序中的不同需求，STL 准备了七种基本容器类型。
* 迭代器（Iterators）：用来在一个对象集合的元素上进行遍历动作。这个对象集合或许是个容器，或许是容器的一部分。每一种容器都提供了自己的迭代器，而这些迭代器了解该种容器的内部结构。
* 算法（Algorithms）：用来处理对象集合中的元素，比如 Sort，Search，Copy，Erase 那些元素。通过迭代器的协助，我们只需撰写一次算法，就可以将它应用于任意容器之上，这是因为所有容器的迭代器都提供一致的接口。

## 4.2 容器（Containers）
容器用来管理某类对象。为了应付程序中的不同需求，STL 准备了两类共七种基本容器类型：

* 序列式容器（Sequence containers），此为可序群集，其中每个元素均有固定位置—取决于插入时机和地点，和元素值无关。如果你以追加方式对一个群集插入六个元素，它们的排列次序将和插入次序一致。STL提供了三个序列式容器：向量（vector）、双端队列（deque）、列表（list），此外你也可以把 string 和 array 当做一种序列式容器。
* 关联式容器（Associative containers），此为已序群集，元素位置取决于特定的排序准则以及元素值，和插入次序无关。如果你将六个元素置入这样的群集中，它们的位置取决于元素值，和插入次序无关。STL提供了四个关联式容器：集合（set）、多重集合（multiset）、映射（map）和多重映射（multimap）。
 
![enter description here](./images/1679733329142.png)
  
### 4.2.1 vector
vector（向量）: 是一种序列式容器，事实上和数组差不多，但它比数组更优越。一般来说数组不能动态拓展，因此在程序运行的时候不是浪费内存，就是造成越界。而 vector 正好弥补了这个缺陷，它的特征是相当于可拓展的数组（动态数组），它的随机访问快，在中间插入和删除慢，但在末端插入和删除快。

特点：
* 拥有一段连续的内存空间，并且起始地址不变，因此它能非常好的支持随机存取，即 `[]` 操作符，但由于它的内存空间是连续的，所以在中间进行插入和删除会造成内存块的拷贝，另外，当该数组后的内存空间不够时，需要重新申请一块足够大的内存并进行内存的拷贝。这些都大大影响了 vector 的效率。
* 对头部和中间进行插入删除元素操作需要移动内存，如果你的元素是结构或类，那么移动的同时还会进行构造和析构操作，所以性能不高。
* 对最后元素操作最快（在后面插入删除元素最快），此时一般不需要移动内存，只有保留内存不够时才需要。

优缺点和使用场景：
* 优点：支持随机访问，即 `[]` 操作和 `.at()`，所以查询效率高。
* 缺点：当向其头部或中部插入或删除元素时，为了保持原本的相对次序，插入或删除点之后的所有元素都必须移动，所以插入的效率比较低。
* 适用场景：适用于对象简单，变化较小，并且频繁随机访问的场景。

### 4.2.2 deque
deque（double-ended queue）是由一段一段的定量连续空间构成。一旦要在 deque 的前端和尾端增加新空间，便配置一段定量连续空间，串在整个 deque 的头端或尾端。因此不论在尾部或头部安插元素都十分迅速。 在中间部分安插元素则比较费时，因为必须移动其它元素。deque 的最大任务就是在这些分段的连续空间上，维护其整体连续的假象，并提供随机存取的接口。

特点：
* 按页或块来分配存储器的，每页包含固定数目的元素。
* deque 是 list 和 vector 的折中方案。兼有 list 的优点，也有vector 随机线性访问效率高的优点。

优缺点和适用场景：
* 优点：支持随机访问，即 `[]` 操作和 `.at()`，所以查询效率高；可在双端进行 `pop`，`push`。
* 缺点：不适合中间插入删除操作；占用内存多。
* 适用场景：适用于既要频繁随机存取，又要关心两端数据的插入与删除的场景。

### 4.2.3 list
List 由双向链表（doubly linked list）实现而成，元素也存放在堆中，每个元素都是放在一块内存中，他的内存空间可以是不连续的，通过指针来进行数据的访问，这个特点使得它的随机存取变得非常没有效率，因此它没有提供 `[]` 操作符的重载。但是由于链表的特点，它可以很有效率的支持任意地方的插入和删除操作。

特点：
* 没有空间预留习惯，所以每分配一个元素都会从内存中分配，每删除一个元素都会释放它占用的内存。
* 在哪里添加删除元素性能都很高，不需要移动内存，当然也不需要对每个元素都进行构造与析构了，所以常用来做随机插入和删除操作容器。
* 访问开始和最后两个元素最快，其他元素的访问时间一样。

优缺点和适用场景：
* 优点：内存不连续，动态操作，可在任意位置插入或删除且效率高。
* 缺点：不支持随机访问。
* 适用场景：适用于经常进行插入和删除操作并且不经常随机访问的场景。

### 4.2.4 set
set（集合）由红黑树实现，其内部元素依据其值自动排序，每个元素值只能出现一次，不允许重复。

特点：
* set 中的元素都是排好序的，集合中没有重复的元素;
* map 和 set 的插入删除效率比用其他序列容器高，因为对于关联容器来说，不需要做内存拷贝和内存移动。


优缺点和适用场景：
* 优点：使用平衡二叉树实现，便于元素查找，且保持了元素的唯一性，以及能自动排序。
* 缺点：每次插入值的时候，都需要调整红黑树，效率有一定影响。
* 适用场景：适用于经常查找一个元素是否在某群集中且需要排序的场景。

另外 Multiset 和 set 相同，只不过它允许重复元素，也就是说 multiset 可包括多个数值相同的元素。这里不再做过多介绍。

### 4.2.5 map
map 由红黑树实现，其元素都是 “键值/实值” 所形成的一个对组（key/value pairs)。每个元素有一个键，是排序准则的基础。每一个键只能出现一次，不允许重复。

map 主要用于资料一对一映射的情况，map 内部自建一颗红黑树，这颗树具有对数据自动排序的功能，所以在 map 内部所有的数据都是有序的。比如一个班级中，每个学生的学号跟他的姓名就存在着一对一映射的关系。

特点：
* 自动建立 Key - value 的对应。key 和 value 可以是任意你需要的类型。
* 根据 key 值快速查找记录，查找的复杂度基本是 O(logN)，如果有 1000 个记录，二分查找最多查找 10次(1024)。
* 增加和删除节点对迭代器的影响很小，除了那个操作节点，对其他的节点都没有什么影响。
* 对于迭代器来说，可以修改实值，而不能修改 key。

优缺点和适用场景：
* 优点：使用平衡二叉树实现，便于元素查找，且能把一个值映射成另一个值，可以创建字典。
* 缺点：每次插入值的时候，都需要调整红黑树，效率有一定影响。
* 适用场景：适用于需要存储一个数据字典，并要求方便地根据key找value的场景。

multimap 和 map 相同，但允许重复元素，也就是说 multimap 可包含多个键值（key）相同的元素。这里不再做过多介绍。

### 4.2.6 容器配接器
除了以上七个基本容器类别，为满足特殊需求，STL还提供了一些特别的（并且预先定义好的）容器配接器，根据基本容器类别实现而成。包括：

1、stack
名字说明了一切，stack 容器对元素采取 LIFO（后进先出）的管理策略。

2、queue
queue 容器对元素采取 FIFO（先进先出）的管理策略。也就是说，它是个普通的缓冲区（buffer）。

3、priority_queue
priority_queue 容器中的元素可以拥有不同的优先权。所谓优先权，乃是基于程序员提供的排序准则（缺省使用 operators）而定义。Priority queue 的效果相当于这样一个 buffer：“下一元素永远是queue中优先级最高的元素”。如果同时有多个元素具备最髙优先权，则其次序无明确定义。

## 4.3 总结

各容器的特点总结：
* vector 头部与中间插入和删除效率较低，在尾部插入和删除效率高，支持随机访问。
* deque 是在头部和尾部插入和删除效率较高，支持随机访问，但效率没有 vector 高。
* list 在任意位置的插入和删除效率都较高，但不支持随机访问。
* set 由红黑树实现，其内部元素依据其值自动排序，每个元素值只能出现一次，不允许重复，且插入和删除效率比用其他序列容器高。
* map 可以自动建立 Key - value 的对应，key 和 value 可以是任意你需要的类型，根据 key 快速查找记录。

在实际使用过程中，到底选择这几种容器中的哪一个，应该根据遵循以下原则：
1. 如果需要高效的随机存取，不在乎插入和删除的效率，使用 vector。
2. 如果需要大量的插入和删除元素，不关心随机存取的效率，使用 list。
3. 如果需要随机存取，并且关心两端数据的插入和删除效率，使用 deque。
4. 如果打算存储数据字典，并且要求方便地根据 key 找到 value，一对一的情况使用 map，一对多的情况使用 multimap。
5. 如果打算查找一个元素是否存在于某集合中，唯一存在的情况使用 set，不唯一存在的情况使用 multiset。

各容器的时间复杂度分析：
* vector 在头部和中间位置插入和删除的时间复杂度为 O(N)，在尾部插入和删除的时间复杂度为 O(1)，查找的时间复杂度为 O(1)；
* deque 在中间位置插入和删除的时间复杂度为 O(N)，在头部和尾部插入和删除的时间复杂度为 O(1)，查找的时间复杂度为 O(1)；
* list 在任意位置插入和删除的时间复杂度都为 O(1)，查找的时间复杂度为 O(N)；
* set 和 map 都是通过红黑树实现，因此插入、删除和查找操作的时间复杂度都是 O(log N)。

各容器的共性：
各容器一般来说都有下列函数：默认构造函数、复制构造函数、析构函数、empty()、max_size()、size()、operator=、operator<、operator<=、operator>、operator>=、operator== 、operator!=、swap()。

顺序容器和关联容器都共有下列函数：
`begin()` ：返回容器第一个元素的迭代器指针；
`end()`：返回容器最后一个元素后面一位的迭代器指针；
`rbegin()`：返回一个逆向迭代器指针，指向容器最后一个元素；
`rend()`：返回一个逆向迭代器指针，指向容器首个元素前面一位；
`clear()`：删除容器中的所有的元素；
`erase(it)`：删除迭代器指针it处元素。


参考：
《C++ 标准库 - 侯捷》中的 5.2 节-容器

# 5、C/C++ 内存泄漏-原因、避免以及定位
来源：https://blog.csdn.net/weixin_60596960/article/details/124747629

作为C/C++ 开发人员，内存泄漏是最容易遇到的问题之一，这是由C/C++ 语言的特性引起的。C/C++ 语言与其他语言不同，需要开发者去申请和释放内存，即需要开发者去管理内存，如果内存使用不当，就容易造成==段错误(segment fault)==或者==内存泄漏(memory leak)==。

今天，借助此文，分析下项目中经常遇到的导致内存泄漏的原因，以及如何避免和定位内存泄漏。
主要内容如下：

![enter description here](./images/1679975127156.png)

## 5.1 背景
C/C++ 语言中，内存的分配与回收都是由开发人员在编写代码时主动完成的，好处是内存管理的开销较小，程序拥有更高的执行效率；弊端是依赖于开发者的水平，随着代码规模的扩大，极容易遗漏释放内存的步骤，或者一些不规范的编程可能会使程序具有安全隐患。如果对内存管理不当，可能导致程序中存在内存缺陷，甚至会在运行时产生内存故障错误。

内存泄漏是各类缺陷中十分棘手的一种，对系统的稳定运行威胁较大。当动态分配的内存在程序结束之前没有被回收时，则发生了内存泄漏。由于系统软件，如操作系统、编译器、开发环境等都是由C/C++ 语言实现的，不可避免地存在内存泄漏缺陷，特别是一些在服务器上长期运行的软件，若存在内存泄漏则会造成严重后果，例如性能下降、程序终止、系统崩溃、无法提供服务等。

所以，本文从原因、避免以及定位几个方面去深入讲解，希望能给大家带来帮助。


## 5.2 概念
内存泄漏（Memory Leak）是指程序中己动态分配的堆内存由于某种原因程序未释放或无法释放，造成系统内存的浪费，导致程序运行速度减慢甚至系统崩溃等严重后果。

当我们在程序中对原始指针(raw pointer)使用`new`操作符或者`free`函数的时候，实际上是在堆上为其分配内存，这个内存指的是RAM，而不是硬盘等永久存储。持续申请而不释放(或者少量释放)内存的应用程序，最终因内存耗尽导致==OOM(out of memory)==。

![enter description here](./images/1679975736775.png)

方便大家理解内存泄漏的危害，举个简单的例子。有一个宾馆，共有100间房间，顾客每次都是在前台进行登记，然后拿到房间钥匙。如果有些顾客不需要该房间了，既不去前台处登记退房，也不归还钥匙，久而久之，前台处可用房间越来越少，收入也越来越少，濒临倒闭。当程序申请了内存，而不进行归还，久而久之，可用内存越来越少，OS就会进行自我保护，杀掉该进程，这就是我们常说的==OOM(out of memory)==。


## 5.3 内存泄漏的分类
内存泄漏分为以下两类：
* 堆内存泄漏：我们经常说的内存泄漏就是堆内存泄漏，在堆上申请了资源，在结束使用的时候，没有释放归还给OS，从而导致该块内存永远不会被再次使用
* 资源泄漏：通常指的是系统资源，比如socket，文件描述符等，因为这些在系统中都是有限制的，如果创建了而不归还，久而久之，就会耗尽资源，导致其他程序不可用

本文主要分析堆内存泄漏，所以后面的内存泄漏均指的是堆内存泄漏。


## 5.4 根源
内存泄漏，主要指的是在堆(heap)上申请的动态内存泄漏，或者说是指针指向的内存块忘了被释放，导致该块内存不能再被申请重新使用。

之前在知乎上看了一句话，指针是C的精髓，也是初学者的一个坎。换句话说，内存管理是C的精髓，C/C++ 可以直接跟OS打交道，从性能角度出发，开发者可以根据自己的实际使用场景灵活进行内存分配和释放。虽然在C++ 中自C++ 11引入了smart pointer，虽然很大程度上能够避免使用裸指针，但仍然不能完全避免，最重要的一个原因是你不能保证组内其他人不适用指针，更不能保证合作部门不使用指针。

那么为什么C/C++ 中会存在指针呢？

这就得从进程的内存布局说起。

### 5.4.1 进程内存布局
![enter description here](./images/1679976238464.png)
上图为32位进程的内存布局，从上图中主要包含以下几个块：

* 内核空间：供内核使用，存放的是内核代码和数据
* stack：这就是我们经常所说的栈，用来存储自动变量(automatic variable)
* mmap:也成为内存映射，用来在进程虚拟内存地址空间中分配地址空间，创建和物理内存的映射关系
* heap:就是我们常说的堆，动态内存的分配都是在堆上
* bss:包含所有未初始化的全局和静态变量，此段中的所有变量都由0或者空指针初始化，程序加载器在加载程序时为BSS段分配内存
* ds:初始化的数据块
	* 包含显式初始化的全局变量和静态变量
	* 此段的大小由程序源代码中值的大小决定，在运行时不会更改
	* 它具有读写权限，因此可以在运行时更改此段的变量值
	* 该段可进一步分为初始化只读区和初始化读写区
* text：也称为文本段
	* 该段包含已编译程序的二进制文件。
	* 该段是一个只读段，用于防止程序被意外修改
	* 该段是可共享的，因此对于文本编辑器等频繁执行的程序，内存中只需要一个副本

由于本文主要讲内存分配相关，所以下面的内容仅涉及到栈(stack)和堆(heap)。

![enter description here](./images/1679976438948.png)

#### 栈
==栈是一块连续的内存块==，栈上的内存分配就是在这一块连续内存块上进行操作的。编译器在编译的时候，就已经知道要分配的内存大小，当调用函数时候，其内部的遍历都会在栈上分配内存；当结束函数调用时候，内部变量就会被释放，进而将内存归还给栈。
```cpp
class Object {
  public:
    Object() = default;
    // ....
};
 
void fun() {
  Object obj;
  
  // do sth
}
```
在上述代码中，obj就是在栈上进行分配，==当出了fun作用域的时候，会自动调用Object的析构函数对其进行释放==。

前面有提到，局部变量会在作用域（如函数作用域、块作用域等）结束后析构、释放内存。因为分配和释放的次序是刚好完全相反的，所以可用到==堆栈先进后出（first-in-last-out, FILO）的特性==，而 C++ 语言的实现一般也会使用到调用堆栈（call stack）来分配局部变量（但非标准的要求）。

因为栈上内存分配和释放，是一个进栈和出栈的过程(对于编译器只是一个指令)，所以相比于堆上的内存分配，栈要快的多。

虽然栈的访问速度要快于堆，每个线程都有一个自己的栈，栈上的对象是不能跨线程访问的，这就决定了栈空间大小是有限制的，如果栈空间过大，那么在大型程序中几十乃至上百个线程，光栈空间就消耗了RAM，这就导致heap的可用空间变小，影响程序正常运行。

![enter description here](./images/1679976791057.png)


**栈的分配方式**
* 静态分配：
静态分配由编译器完成，假如局部变量以及函数参数等，都在编译期就分配好了。
```cpp
void fun() {
  int a[10];
}
```
上述代码中，a占`10 * sizeof(int)`个字节，在编译的时候直接计算好了，运行的时候，直接进栈出栈。

* 动态分配：
可能很多人认为只有堆上才会存在动态分配，在栈上只可能是静态分配。其实，这个观点是错的，栈上也支持动态分配，该动态分配由`alloca()`函数进行分配。栈的动态分配和堆是不同的，通过`alloca()`函数分配的内存由编译器进行释放，无需手动操作。

**栈的特点**
* 分配速度快：分配大小由编译器在编译期完成
* 不会产生内存碎片：栈内存分配是连续的，以FILO的方式进栈和出栈
* 大小受限：栈的大小依赖于操作系统
* 访问受限：只能在当前函数或者作用域内进行访问

#### 堆
堆（heap）是一种内存管理方式。内存管理对操作系统来说是一件非常复杂的事情，因为首先内存容量很大，其次就是内存需求在时间和大小块上没有规律（操作系统上运行着几十甚至几百个进程，这些进程可能随时都会申请或者是释放内存，并且申请和释放的内存块大小是随意的）。

堆这种内存管理方式的特点就是自由（随时申请、随时释放、大小块随意）。堆内存是操作系统划归给堆管理器（操作系统中的一段代码，属于操作系统的内存管理单元）来管理的，堆管理器提供了对应的接口`_sbrk`、`_mmap`等，只是该接口往往由运行时库(Linux为glibc)进行调用，即也可以说由运行时库进行堆内存管理，运行时库提供了`malloc/free`函数由开发人员调用，进而使用堆内存。

**堆的分配方式**
正如我们所理解的那样，由于是在运行期进行内存分配，分配的大小也在运行期才会知道，所以堆只支持动态分配，内存申请和释放的行为由开发者自行操作，这就很容易造成我们说的内存泄漏。

**堆的特点**
* 变量可以在进程范围内访问，即进程内的所有线程都可以访问该变量
* 没有内存大小限制，这个其实是相对的，只是相对于栈大小来说没有限制，其实最终还是受限于RAM
* 相对栈来说访问比较慢
* 内存碎片
* 由开发者管理内存，即内存的申请和释放都由开发人员来操作

#### 栈和堆的区别
理解堆和栈的区别，对我们开发过程中会非常有用，结合上面的内容，总结下二者的区别。

对于栈来讲，是由编译器自动管理，无需我们手工控制；
对于堆来说，释放工作由程序员控制，容易产生内存泄漏；

* 空间大小不同
	* 一般来讲在 32 位系统下，堆内存可以达到3G的空间，从这个角度来看堆内存几乎是没有什么限制的。
	* 对于栈来讲，一般都是有一定的空间大小的，一般依赖于操作系统(也可以人工设置)

* 能否产生碎片不同
	* 对于堆来讲，频繁的内存分配和释放势必会造成内存空间的不连续，从而造成大量的碎片，使程序效率降低。
	* 对于栈来讲，内存都是连续的，申请和释放都是指令移动，类似于数据结构中的进栈和出栈

* 增长方向不同
	* 对于堆来讲，生长方向是向上的，也就是向着内存地址增加的方向
	* 对于栈来讲，它的生长方向是向下的，是向着内存地址减小的方向增长

* 分配方式不同
	* 堆都是动态分配的，比如我们常见的`malloc`/`new`；而栈则有静态分配和动态分配两种。
	* 静态分配是编译器完成的，比如局部变量的分配，而栈的动态分配则通过`alloca()`函数完成
	* 二者动态分配是不同的，栈的动态分配的内存由编译器进行释放，而堆上的动态分配的内存则必须由开发人自行释放

* 分配效率不同
	* 栈有操作系统分配专门的寄存器存放栈的地址，压栈出栈都有专门的指令执行，这就决定了栈的效率比较高
	* 堆内存的申请和释放专门有运行时库提供的函数，里面涉及复杂的逻辑，申请和释放效率低于栈

截止到这里，栈和堆的基本特性以及各自的优缺点、使用场景已经分析完成，在这里给开发者一个建议，能使用栈的时候，就尽量使用栈，一方面是因为效率高于堆，另一方面内存的申请和释放由编译器完成，这样就避免了很多问题。

#### 扩展
在前面的内容中，我们对比了栈和堆，虽然栈效率比较高，且不存在内存泄漏、内存碎片等，但是由于其本身的局限性(不能多线程、大小受限)，所以在很多时候，还是需要在堆上进行内存。  

我们先看一段代码：
```cpp
#include <stdio.h>
#include <stdlib.h>
 
int main() {
  int a;
  int *p;
  p = (int *)malloc(sizeof(int));
  free(p);
 
  return 0;
}
```
上述代码很简单，有两个变量a和p，类型分别为`int`和`int *`，其中，a和p存储在栈上，p的值为在堆上的某块地址(在上述代码中，p的值为0x1c66010)，上述代码布局如下图所示：

![enter description here](./images/1679989149657.png)


## 5.5 内存泄漏的产生方式
以产生的方式来分类，内存泄漏可以分为四类:
* 常发性内存泄漏
* 偶发性内存泄漏
* 一次性内存泄漏
* 隐式内存泄漏

### 5.5.1 常发性内存泄漏
产生内存泄漏的代码或者函数会被多次执行到，在每次执行的时候，都会产生内存泄漏。

### 5.5.2 偶发性内存泄漏
与常发性内存泄漏不同的是，偶发性内存泄漏函数只在特定的场景下才会被执行。

笔者在19年的时候，曾经遇到一个这种内存泄漏。有一个函数专门进行价格加密，每次泄漏3个字节，且只有在竞价成功的时候，才会调用此函数进行价格加密，因此泄漏的非常不明显。当时发现这个问题，是上线后的第二天，帮忙排查线上问题，发现内存较上线前上涨了点(大概几百兆的样子)，了解glibc内存分配原理的都清楚，调用delete后，内存不一定会归还给OS，但是本着宁可信其有，不可信其无的心态，决定来分析是否真的存在内存泄漏。

当时用了个比较傻瓜式的方法，通过top命令，将该进程所占的内存输出到本地文件，大概几个小时后，将这些数据导入Excel中，内存占用基本呈一条斜线，所以基本能够确定代码存在内存泄漏，所以就对新上线的这部分代码进行重新review，定位到泄漏点，然后修复，重新上线。

### 5.5.3 一次性内存泄漏
这种内存泄漏在程序的生命周期内只会泄漏一次，或者说造成泄漏的代码只会被执行一次。          

有的时候，这种可能不算内存泄漏，或者说设计如此。就以笔者现在线上的服务来说，类似于如下这种：
```cpp
int main() {
  auto *service = new Service;
  // do sth
  service->Run();// 服务启动
  service->Loop(); // 可以理解为一个sleep，目的是使得程序不退出
  return 0;
}
```
这种严格意义上，并不算内存泄漏，因为程序是这么设计的，即使程序异常退出，那么整个服务进程也就退出了，当然，在Loop()后面加个delete更好。

### 5.5.4 隐式内存泄漏
程序在运行过程中不停的分配内存，但是直到结束的时候才释放内存。严格的说这里并没有发生内存泄漏，因为最终程序释放了所有申请的内存。但是对于一个服务器程序，需要运行几天，几周甚至几个月，不及时释放内存也可能导致最终耗尽系统的所有内存。所以，我们称这类内存泄漏为隐式内存泄漏。

比较常见的隐式内存泄漏有以下三种：
* 内存碎片：还记得我们之前的那篇文章**深入理解glibc内存管理精髓**（`https://mp.weixin.qq.com/s?__biz=Mzk0MzI4OTI1Ng==&mid=2247485953&idx=1&sn=f8cd484607ab07f15247ecde773d2e1c&scene=21#wechat_redirect` ），程序跑了几天之后，进程就因为OOM导致了退出，就是因为内存碎片导致剩下的内存不能被重新分配导致
* 即使我们调用了free/delete，运行时库不一定会将内存归还OS，具体看**深入理解glibc内存管理精髓**
* 用过STL的知道，STL内部有一个自己的allocator，我们可以当做一个memory poll，当调用vector.clear()时候，内存并不会归还OS，而是放回allocator，其内部根据一定的策略，在特定的时候将内存归还OS，是不是跟glibc原理很像

## 5.6 分类
### 5.6.1 未释放
这种是很常见的，比如下面的代码：
```cpp
int fun() {
    char * pBuffer = malloc(sizeof(char));
    
    /* Do some work */
    return 0;
}
```
上面代码是非常常见的内存泄漏场景(也可以使用new来进行分配)，我们申请了一块内存，但是在fun函数结束时候没有调用free函数进行内存释放。

在C++ 开发中，还有一种内存泄漏，如下：
```cpp
class Obj {
 public:
   Obj(int size) {
     buffer_ = new char;
   }
   ~Obj(){}
  private:
   char *buffer_;
};
 
int fun() {
  Object obj;
  // do sth
  return 0;
}
```
上面这段代码中，析构函数没有释放成员变量buffer_指向的内存，所以在编写析构函数的时候，一定要仔细分析成员变量有没有申请动态内存，如果有，则需要手动释放，我们重新编写了析构函数，如下：
```cpp
~Object() {
  delete buffer_;
}
```
在C/C++ 中，对于普通函数，如果申请了堆资源，请跟进代码的具体场景调用`free`/`delete`进行资源释放；对于class，如果申请了堆资源，则需要在对应的析构函数中调用`free`/`delete`进行资源释放。

### 5.6.2 未匹配
在C++ 中，我们经常使用new操作符来进行内存分配，其内部主要做了两件事：
1. 通过operator new从堆上申请内存(glibc下，operator new底层调用的是malloc)
2. 调用构造函数(如果操作对象是一个class的话)

对应的，使用delete操作符来释放内存，其顺序正好与new相反：
1. 调用对象的析构函数(如果操作对象是一个class的话)
2. 通过operator delete释放内存
```cpp
void* operator new(std::size_t size) {
    void* p = malloc(size);
    if (p == nullptr) {
        throw("new failed to allocate %zu bytes", size);
    }
    return p;
}
void* operator new[](std::size_t size) {
    void* p = malloc(size);
    if (p == nullptr) {
        throw("new[] failed to allocate %zu bytes", size);
    }
    return p;
}
 
void  operator delete(void* ptr) throw() {
    free(ptr);
}
void  operator delete[](void* ptr) throw() {
    free(ptr);
}
```

为了加深多这块的理解，我们举个例子：
```cpp
class Test {
 public:
   Test() {
     std::cout << "in Test" << std::endl;
   }
   // other
   ~Test() {
     std::cout << "in ~Test" << std::endl;
   }
};
 
int main() {
  Test *t = new Test;
  // do sth
  delete t;
  return 0;
}
```
在上述main函数中，我们使用new 操作符创建一个Test类指针
1.通过operator new申请内存(底层malloc实现)
2.通过placement new在上述申请的内存块上调用构造函数
3.调用ptr->~Test()释放Test对象的成员变量
4.调用operator delete释放内存

上述过程，可以理解为如下：
```cpp
// new
void *ptr = malloc(sizeof(Test));
t = new(ptr)Test
  
// delete
ptr->~Test();
free(ptr);
```
好了，上述内容，我们简单的讲解了C++ 中new和delete操作符的基本实现以及逻辑，那么，我们就简单总结下下产生内存泄漏的几种类型。

#### new 和 free
仍然以上面的Test对象为例，代码如下：
```cpp
Test *t = new Test;
free(t);
```
此处会产生内存泄漏，在上面，我们已经分析过，new操作符会先通过operator new分配一块内存，然后在该块内存上调用placement new即调用Test的构造函数。而在上述代码中，只是通过free函数释放了内存，但是没有调用Test的析构函数以释放Test的成员变量，从而引起内存泄漏。

#### new[] 和 delete
```cpp
int main() {
  Test *t = new Test [10];
  // do sth
  delete t;
  return 0;
}
```
在上述代码中，我们通过new创建了一个Test类型的数组，然后通delete操作符删除该数组，编译并执行，输出如下：
```cpp
in Test
in Test
in Test
in Test
in Test
in Test
in Test
in Test
in Test
in Test
in ~Test
```
从上面输出结果可以看出，调用了10次构造函数，但是只调用了一次析构函数，所以引起了内存泄漏。这是因为调用`delete t`释放了通过`operator new[]`申请的内存，即malloc申请的内存块，且只调用了`t[0]`对象的析构函数，`t[1..9]`对象的析构函数并没有被调用。

#### 虚析构
有一道题，面试官问，std::string能否被继承，为什么？
当时没回答上来，后来过了没多久，进行面试复盘的时候，偶然看到继承需要父类析构函数为virtual，才恍然大悟，原来考察点在这块。

下面我们看下std::string的析构函数定义：
```cpp
~basic_string() { 
  _M_rep()->_M_dispose(this->get_allocator()); 
}
```
这块需要特别说明下，std::basic_string是一个模板，而std::string是该模板的一个特化，即std::basic_string。
`typedef std::basic_string<char> string;`
现在我们可以给出这个问题的答案：**std::string不能被继承，因为std::string的析构函数不为virtual，这样会引起内存泄漏**。
仍然以一个例子来进行证明。
```cpp
class Base {
 public:
  Base(){
    buffer_ = new char[10];
  }
  ~Base() {
    std::cout << "in Base::~Base" << std::endl;
    delete []buffer_;
  }
private:
  char *buffer_;
 
};
 
class Derived : public Base {
 public:
  Derived(){}
  ~Derived() {
    std::cout << "int Derived::~Derived" << std::endl;
  }
};
 
int main() {
  Base *base = new Derived;
  delete base;
  return 0;
}
```
上面代码输出如下：
```
in Base::~Base

```

可见，上述代码并没有调用派生类Derived的析构函数，如果派生类中在堆上申请了资源，那么就会产生内存泄漏。

为了避免因为继承导致的内存泄漏，我们需要将父类的析构函数声明为virtual，代码如下(只列了部分修改代码，其他不变):
```cpp
virtual ~Base() {
    std::cout << "in Base::~Base" << std::endl;
    delete []buffer_;
  }
```

然后重新执行代码，输出结果如下：
```
int Derived::~Derived
in Base::~Base
```

借助此文，我们再次总结下存在继承情况下，构造函数和析构函数的调用顺序。

派生类对象在创建时构造函数调用顺序：
1. 调用父类的构造函数
2. 调用父类成员变量的构造函数
3. 调用派生类本身的构造函数

派生类对象在析构时的析构函数调用顺序：
1.  执行派生类自身的析构函数
2. 执行派生类成员变量的析构函数
3. 执行父类的析构函数

为了避免存在继承关系时候的内存泄漏，请遵守一条规则：无论派生类有没有申请堆上的资源，请将父类的析构函数声明为virtual。

### 5.6.3 循环引用
在C++ 开发中，为了尽可能的避免内存泄漏，自C++ 11起引入了smart pointer，常见的有shared_ptr、weak_ptr以及unique_ptr等(auto_ptr已经被废弃)，其中weak_ptr是为了解决循环引用而存在，其往往与shared_ptr结合使用。

下面，我们看一段代码：
```cpp
class Controller {
 public:
  Controller() = default;
 
  ~Controller() {
    std::cout << "in ~Controller" << std::endl;
  }
 
  class SubController {
   public:
    SubController() = default;
 
    ~SubController() {
      std::cout << "in ~SubController" << std::endl;
    }
 
    std::shared_ptr<Controller> controller_;
  };
 
  std::shared_ptr<SubController> sub_controller_;
};
 
int main() {
  auto controller = std::make_shared<Controller>();
  auto sub_controller = std::make_shared<Controller::SubController>();
 
  controller->sub_controller_ = sub_controller;
  sub_controller->controller_ = controller;
  return 0;
}
```
编译并执行上述代码，发现并没有调用Controller和SubController的析构函数，我们尝试着打印下引用计数，代码如下：
```cpp
int main() {
  auto controller = std::make_shared<Controller>();
  auto sub_controller = std::make_shared<Controller::SubController>();
 
  controller->sub_controller_ = sub_controller;
  sub_controller->controller_ = controller;
 
  std::cout << "controller use_count: " << controller.use_count() << std::endl;
  std::cout << "sub_controller use_count: " << sub_controller.use_count() << std::endl;
  return 0;
}
```
编译并执行之后，输出如下：
```
controller use_count: 2
sub_controller use_count: 2
```

通过上面输出可以发现，因为引用计数都是2，所以在main函数结束的时候，不会调用controller和sub_controller的析构函数，所以就出现了内存泄漏。

上面产生内存泄漏的原因，就是我们常说的循环引用。

![enter description here](./images/1679994399366.png)

为了解决std::shared_ptr循环引用导致的内存泄漏，我们可以使用std::weak_ptr来单面去除上图中的循环。
```cpp
class Controller {
 public:
  Controller() = default;
 
  ~Controller() {
    std::cout << "in ~Controller" << std::endl;
  }
 
  class SubController {
   public:
    SubController() = default;
 
    ~SubController() {
      std::cout << "in ~SubController" << std::endl;
    }
 
    std::weak_ptr<Controller> controller_;
  };
 
  std::shared_ptr<SubController> sub_controller_;
};
```
在上述代码中，我们将SubController类中controller_的类型从std::shared_ptr变成std::weak_ptr，重新编译执行，结果如下：
```
controller use_count: 1
sub_controller use_count: 2
in ~Controller
in ~SubController
```
从上面结果可以看出，controller和sub_controller均以释放，所以循环引用引起的内存泄漏问题，也得以解决。

![enter description here](./images/1679994523309.png)


 可能有人会问，使用std::shared_ptr可以直接访问对应的成员函数，如果是std::weak_ptr的话，怎么访问呢？我们可以使用下面的方式：
```cpp
std::shared_ptr controller = controller_.lock();
```
即在子类SubController中，如果要使用controller调用其对应的函数，就可以使用上面的方式。

## 5.7 避免
### 避免在堆上分配
众所周知，大部分的内存泄漏都是因为在堆上分配引起的，如果我们不在堆上进行分配，就不会存在内存泄漏了(这不废话嘛)，我们可以根据具体的使用场景，如果对象可以在栈上进行分配，就在栈上进行分配，一方面栈的效率远高于堆，另一方面，还能避免内存泄漏，我们何乐而不为呢。

### 手动释放
对于`malloc`函数分配的内存，在结束使用的时候，使用`free`函数进行释放；
对于`new`操作符创建的对象，切记使用`delete`来进行释放；
对于`new []`创建的对象，使用`delete[]`来进行释放(使用`free`或者`delete`均会造成内存泄漏)；

### 避免使用裸指针
```cpp
nt fun(int *ptr) {// fun 是一个接口或lib函数
  // do sth
  return 0;
}
 
int main() {}
  int a = 1000;
  int *ptr = &a;
  // ...
  fun(ptr);
  
  return 0;
}
```
在上面的fun函数中，有一个参数ptr，为`int *`，我们需要根据上下文来分析这个指针是否需要释放，这是一种很不好的设计

### 使用STL中或者自己实现对象
在C++ 中，提供了相对完善且可靠的STL供我们使用，所以能用STL的尽可能的避免使用C中的编程方式，比如：
* 使用std::string 替代char *, string类自己会进行内存管理，而且优化的相当不错
* 使用std::vector或者std::array来替代传统的数组
* 其它适合使用场景的对象

### 智能指针

### RAII
 RAII是Resource Acquisition is Initialization(资源获取即初始化)的缩写，是C++ 语言的一种管理资源，避免泄漏的用法。

利用的就是C++ 构造的对象最终会被销毁的原则。利用C++ 对象生命周期的概念来控制程序的资源,比如内存,文件句柄,网络连接等。

RAII的做法是使用一个对象，在其构造时获取对应的资源，在对象生命周期内控制对资源的访问，使之始终保持有效，最后在对象析构的时候，释放构造时获取的资源。

简单地说，就是把资源的使用限制在对象的生命周期之中，自动释放。

举个简单的例子，通常在多线程编程的时候，都会用到std::mutex，如果一不小心忘记释放，那么就会造成故障，为了解决这个问题，我们使用RAII技术，如：`std::lock_guard`


## 5.8 定位
在发现程序存在内存泄漏后，往往需要定位泄漏点，而定位这一步往往是最困难的，所以经常为了定位泄漏点，采取各种各样的方案，甭管方案优雅与否，毕竟管他黑猫白猫，抓住老鼠才是好猫，所以在本节，简单说下笔者这么多年定位泄漏点的方案，有些比较邪门歪道，您就随便看看就行。

### 日志
这种方案的核心思想，就是在每次分配内存的时候，打印指针地址，在释放内存的时候，打印内存地址，这样在程序结束的时候，通过分配和释放的差，如果分配的条数大于释放的条数，那么基本就能确定程序存在内存泄漏，然后根据日志进行详细分析和定位。

### 统计
统计方案可以理解为日志方案的一种特殊实现，其主要原理是在分配的时候，统计分配次数，在释放的时候，则是统计释放的次数，这样在程序结束前判断这俩值是否一致，就能判断出是否存在内存泄漏。

此方法可帮助跟踪已分配内存的状态。为了实现这个方案，需要创建三个自定义函数，一个用于内存分配，第二个用于内存释放，最后一个用于检查内存泄漏。代码如下：
```cpp
static unsigned int allocated  = 0;
static unsigned int deallocated  = 0;
void *Memory_Allocate (size_t size)
{
    void *ptr = NULL;
    ptr = malloc(size);
    if (NULL != ptr) {
        ++allocated;
    } else {
        //Log error
    }
    return ptr;
}
void Memory_Deallocate (void *ptr) {
    if(pvHandle != NULL) {
        free(ptr);
        ++deallocated;
    }
}
int Check_Memory_Leak(void) {
    int ret = 0;
    if (allocated != deallocated) {
        //Log error
        ret = MEMORY_LEAK;
    } else {
        ret = OK;
    }
    return ret;
}
```

### 工具
在Linux上比较常用的内存泄漏检测工具是==valgrind==，所以咱们就以valgrind为工具，进行检测。
我们首先看一段代码：
```cpp
#include <stdlib.h>
 
void func (void){
    char *buff = (char*)malloc(10);
}
 
int main (void){
    func(); // 产生内存泄漏
    return 0;
}
```
* 通过gcc -g leak.c -o leak命令进行编译

* 执行valgrind --leak-check=full ./leak

在上述的命令执行后，会输出如下：
```
==9652== Memcheck, a memory error detector
==9652== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==9652== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==9652== Command: ./leak
==9652==
==9652==
==9652== HEAP SUMMARY:
==9652==     in use at exit: 10 bytes in 1 blocks
==9652==   total heap usage: 1 allocs, 0 frees, 10 bytes allocated
==9652==
==9652== 10 bytes in 1 blocks are definitely lost in loss record 1 of 1
==9652==    at 0x4C29F73: malloc (vg_replace_malloc.c:309)
==9652==    by 0x40052E: func (leak.c:4)
==9652==    by 0x40053D: main (leak.c:8)
==9652==
==9652== LEAK SUMMARY:
==9652==    definitely lost: 10 bytes in 1 blocks
==9652==    indirectly lost: 0 bytes in 0 blocks
==9652==      possibly lost: 0 bytes in 0 blocks
==9652==    still reachable: 0 bytes in 0 blocks
==9652==         suppressed: 0 bytes in 0 blocks
==9652==
==9652== For lists of detected and suppressed errors, rerun with: -s
==9652== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
```

valgrind的检测信息将内存泄漏分为如下几类：
* definitely lost：确定产生内存泄漏
* indirectly lost：间接产生内存泄漏
* possibly lost：可能存在内存泄漏
* still reachable：即使在程序结束时候，仍然有指针在指向该块内存，常见于全局变量

主要上面输出的下面几句：
```
==9652==    by 0x40052E: func (leak.c:4)
==9652==    by 0x40053D: main (leak.c:8)
```

提示在main函数(leak.c的第8行)fun函数(leak.c的第四行)产生了内存泄漏，通过分析代码，原因定位，问题解决。

valgrind不仅可以检测内存泄漏，还有其他很强大的功能，由于本文以内存泄漏为主，所以其他的功能就不在此赘述了，有兴趣的可以通过`valgrind --help`来进行查看

对于Windows下的内存泄漏检测工具，笔者推荐一款轻量级功能却非常强大的工具==UMDH==，笔者尝试了各种工具(免费的和收费的)，最终发现了==UMDH==，如果你在Windows上进行开发，强烈推荐。

参考：
https://developers.redhat.com/blog/2021/05/05/memory-error-checking-in-c-and-c-comparing-sanitizers-and-valgrind

https://aticleworld.com/what-is-memory-leak-in-c-c-how-can-we-avoid/

https://iq.opengenus.org/memory-leak-in-cpp-and-how-to-avoid-it/

https://blog.nelhage.com/post/three-kinds-of-leaks/#type-1-unreachable-allocations

https://owasp.org/www-community/vulnerabilities/Memory_leak

https://www.usna.edu/Users/cs/roche/courses/s19ic221/lab05.html

https://stackoverflow.com/questions/6261201/how-to-find-memory-leak-in-a-c-code-project


# 6、为什么现代C++ 不建议使用裸指针
1. 裸指针无法说明指向的是单个对象还是一个数组
2. 裸指针无法说明使用完指针是否需要析构，即从声明中看不出来指针是否拥有所指向的对象
3. 即使知道需要析构，也不知道应该用 delete 还是调用某个类似 deinit(p) 的函数
4. 即使知道用 `delete`，也不知道用 `delete` 还是 `delete[]`（见理由 1）
5. 即使知道如何析构，还要保证在整个路径上，刚好只调用一次析构：少调用导致资源泄露，调用多次将产生未定义行为（如同一指针 `delete` 两次可能导致程序崩溃）
6. 空悬指针（dangling pointer）：对象已析构，但仍有指针指向它


# 7、裸指针，弃用auto_ptr原因
来源：https://blog.csdn.net/weixin_52958292/article/details/127599579

## 7.1 裸指针带来的问题
1. 难以区分指向的是单个对象还是一组对象。
2. 使用完指针之后，无法判断是否应该销毁指针，因此也就无法判断指针是否“拥有”指向的对象。
3. 在已经确定需要销毁指针的情况下，也无法确定是用delete关键字删除，还是其他特殊的销毁机制，例如通过将指针传入某个特定的销毁函数来销毁指针。
4. 即便已经确定了销毁的方法，由于1的原因，无法确认用delete还是delete[]。
5. 假设上述问题都解决了，也很难保证在代码的所有路径中（分支结构，异常导致的跳转），有且仅有一次销毁指针的操作，任何一条路径一楼都会导致内存泄漏。
6. 理论上没有办法来分辨一个指针是否处于悬挂状态。

## 7.2 为什么要弃用auto_ptr



# C++ 类的成员函数的连续调用与返回值问题
来源：https://www.cnblogs.com/FengZeng666/p/9324833.html

## 成员函数返回值是引用
```c++
#include <iostream>

using namespace std;

class X
{
public:
    X display()
    {
        cout << a;
        return *this;
    }
    X& set1(int b)
    {
        a = b;
        return *this;
    }

    X& set2(int b)
    {
        a = b;
        return *this;
    }

private:
    int a = 3;
};

int main()
{
    X x;
    x.set1(6).set2(7).display();
    cout << “ ”;
    x.display();


    return 0;

}
```

如果set1与set2的返回值是引用，那么

x.set1(6).set2(7).display();     //等价于x.set1(6);  x.set2(7);  x.display(); 

所以此处输出为：7   7

## 成员函数返回值是引用
```c++
#include <iostream>
using namespace std;

class X
{
public:
    X display()
    {
        cout << a;
        return *this;
    }
    X set1(int b)
    {
        a = b;
        return *this;
    }

    X set2(int b)
    {
        a = b;
        return *this;
    }

private:
    int a = 3;
};

int main()
{
    X x;
    x.set1(6).set2(7).display();
    cout << “ ”;
    x.display();


    return 0;

}
```
如果set1与set2的返回值是引用，那么

x.set1(6).set2(7).display();     //不等价于x.set1(6);  x.set2(7);  x.display(); 此时后面的.set2(7).display()是对x.set1(6)返回的副本进行操作,所以此处只有x.set1(6)对原对象的值进行了操作

故此处输出为：7   6


# 深入浅出c++ 协程
来源：https://www.cnblogs.com/ishen/p/14617708.html

## 什么是协程
**协程是一种函数对象，可以设置锚点做暂停，然后再该锚点恢复继续运行**，我觉得这是最合适的定义，用户态线程，轻量级线程，可中断恢复的函数，这些都不够精确，先来认识一个boost 1.75的一个例子

```c++
#include <iostream>
#include <boost/coroutine2/all.hpp>

void coroutine_function(boost::coroutines2::coroutine<void>::pull_type & coro_back)
{
    std::cout << "a ";
    coro_back(); // 锚点，返回
    std::cout << "b ";
    coro_back(); //锚点 返回
    std::cout << "c ";
}

int main()
{
    boost::coroutines2::coroutine<void>::push_type coroutine_object(coroutine_function); 	// 创建协程
    std::cout << "1 ";
    coroutine_object(); // 运行协程
    std::cout << "2 ";
    coroutine_object(); // 返回锚点，继续运行协程
    std::cout << "3 ";
    coroutine_object(); // 返回锚点，继续运行协程
    return 0;
}

```

```
g++ test.cpp -lboost_coroutine -lboost_context -o test
./pull
--------------输出分割线-------------
1 a 2 b 3 c 
```

在main( )中创建了一个协程 `coroutine_object`，然后调用 `coroutine_object()`去运行，实际上运行的coroutine_function( )函数，而且每次运行到`coro_back();`就中断当前的执行返回，下次调用coroutine_object()就从这个断点继续运行，这就是协程的全部了

为什么会有协程是轻量级线程的说法呢？因为协程具有中断可恢复的特性，那么只需要在开一个全局的数组存储所有的协程，在协程中断时，不断轮转调用下一个协程继续运行即可； 这看起来似乎和线程无异，但其实有巨大的区别，**因为协程本质是函数，调用协程后原来的地方就会被阻塞，协程处理完了才返回结果，这是天然同步的**，而多线程无法做到这点，因为多线程的调度受内核控制，触发点来自于硬件时钟中断不可预见，同时又运行在多核心下，调用后运行次序是不确定的，想实现同步调用就必须通过std::promise/future 去辅佐，但为了性能往往见到的是异步+回调的方式进行多线程的交互，异步回调代码的可读性是很差的而且还需要考虑一大堆并发上锁的情况，**协程因其函数本质，是天然同步的，而在遇到阻塞条件时候，把cpu让给别的协程，等条件满足了再通过中断可恢复的特性再继续运行，就实现了并发，同步+并发就是协程强大的地方，其使用范式和轮转+同步非阻塞很像**

接下来会介绍一些目前的实现的协程，有非官方的: boost.coroutine2的协程，使用起来方便，让我们可以直观了解协程；微信的libco， 源码很好阅读，资料多，可以进一步学习到协程是如何实现运行的；而官方本身的c++ 20协程，还不成熟，使用起来比较复杂，官方的东西还是需要提前了解；

## 一些实现的c++ 协程

### boost中的协程
#### push_type和pull_type
boost自己早就实现了一套协程，先后推出了两个版本boost coroutine和boost coroutine2，现在第一个版本boost coroutine已经弃用， 直接看看coroutin2的简单例子
```c++
#include <iostream>
#include <boost/coroutine2/all.hpp>

void foo(boost::coroutines2::coroutine<int>::push_type & sink)
{
    std::cout<<"start coroutine\n";
    sink(1);
    std::cout<<"finish coroutine\n";
}


int main()
{
    boost::coroutines2::coroutine<int>::pull_type source(foo);
    std::cout<<source.get()<<std::endl;
    std::cout<<source()<<std::endl;
    std::cout<<"finish\n";
    return 0;
}

```

编译链接运行后：
```
g++ pull.cpp -lboost_coroutine -lboost_context -o pull
./pull
--------------输出分割线-------------
start coroutine
1
finish coroutine
finish
```

boost.corountine2中的协程增加了`push_type`和`pull_type`用于提供协程数据的流转，约束了数据的从`push_type`流入，从`pull_type`流出, 上面的demo定义协程对象source的时候使用了`pull_type`，所以协程函数参数类型是`push_type`。

![enter description here](./images/1695734265672.png)
当协程对象被创建之后就直接运行，直到sink(1)的时候暂停返回到main中，main中使用source.get()获取数据，继续使用source()调用协程对象，协程从sink(1)之后继续运行执行完毕，返回main，main也执行完毕。

上面是一个pull的例子，接下来是一个push的例子
```c++
#include <iostream>
#include <boost/coroutine2/all.hpp>

void foo(boost::coroutines2::coroutine<int>::pull_type& sink)
{
    std::cout<<"start coroutine\n";
    //sink();
    int a = sink().get();
    std::cout<<a<<std::endl;
    std::cout<<"finish coroutine\n";
}


int main()
{
    boost::coroutines2::coroutine<int>::push_type source(foo);
    
    std::cout<<"finish\n";
    source(0);
    source(5);
    return 0;
}
```

编译输出：
```
g++ push.cpp -lboost_coroutine -lboost_context
./push 
--------------输出分割线-------------
finish
start coroutine
5
finish coroutine
```

也可以看到一个细节, 当source为`pull_type`的时候，协程是马上运行的，因为此时不用传递数据进行，而`push_type`的时候，需要source()才会运行，第一次需要放一个没用的数据

为了使用方便，boost::coroutine2实现了协程迭代器，如下
```c++
template< typename T >
class push_routine{
    ....
    

    push_coroutine< T > &
    push_coroutine< T >::operator()( T const& t) { //() 切换协程
        cb_->resume( t);
        return * this;
    }
    
    
    class iterator{	// 实现迭代器
        ....
        
        iterator & operator++() noexcept {
            return *this;
        }
       
    }
       
}

```

所以其支持如下用法，直接在range-for切换元素的时候就能恢复运行
```c++
	boost::coroutines2::coroutine<void>::push_type source(foo);
	for(auto& s : source){
		std::cout<<"run"
	}
```

#### fiber
因为push_type和pull_type这样的简洁组合已经可以解决基本问题---同步调用的中断恢复，但是只有多协程并发才能发挥其真正威力，为此需要同步和调度，boost搞了个fiber(纤程，这才是轻量级线程)出来，是在coroutine2的基础上添加了协程调度器以及barrier mutex channel promise future condition_variable, sleep yield 等协程同步工具，这些和线程同步工具很像，因为在多协程场景下，它两模型和解决的问题都是一样的，都是通过调度多实体实现并发，但是协程有很多好处，开销很小，而且调度是运行的协程自己控制让出cpu给下一个要运行的线程，是可预见的，同时调用上是同步的，保证了顺序性就可以避免锁，

下面是boost的fiber的一个例子
```c++

#include <boost/fiber/all.hpp>
#include <iostream>

using namespace std;
using namespace boost;
 
void callMe(fibers::buffered_channel<string>& pipe) {
    pipe.push("hello world");
}
 
 
int main() {
    fibers::buffered_channel<string> pipe(2);
    fibers::fiber f([&]() {callMe(pipe); });
    f.detach();
    string str;
    std::cout<<"start pop"<<std::endl;
    pipe.pop(str); //切换协程运行
    std::cout<<"get str:"<<str<<std::endl;
    return 0;
}
```

编译运行：
```
g++ channel.cpp -o channel -lboost_fiber -lboost_context
./channel 
-------------------输出分割线-------------------
start pop
get str:hello world
```

这是一个最简单的例子，并没有去体现使用一个loop去做调度协程，调度还是由一些函数手动触发的

注意pull_type和push_type的操作已经没有了，那协程是如何切换的呢？ 切换发生在pipe.pop( )中， fibers::buffered_channel是一个缓存队列，用来传输数据，pop的底层检测到没有数据，会就开始让出cpu，底层的协程调度器就开始调度别的协程进行运行，没有看过源码不知道执行到pipe.push的时候是否有没有发生调度，也许有也许没有，但都不太重要，因为这就和线程是一样的；

由于fiber中有调度器的存在，当前协程主动让出cpu，调度器让别的协程运行，比如上面的pipe.pop()，相当执行了一个协程的co_yield()操作让出cpu；所以，某个协程中如果有阻塞操作，将导致整个线程都处于阻塞，所有协程都被阻塞, 此文提出两种解决方法

* 同步改成非阻塞，一旦发现未达到条件直接yield()让出cpu，再后面轮转调度还能回到该店
```c++
int read_chunk( NonblockingAPI & api, std::string & data, std::size_t desired) {
    int error;
    while ( EWOULDBLOCK == ( error = api.read( data, desired) ) ) {
        boost::this_fiber::yield();
    }
    return error;
} 
```

* 同步操作改成异步操作，使用协程级的future和promise进行等待转让cpu给别的协程
```c++
std::pair< AsyncAPI::errorcode, std::string > read_ec( AsyncAPI & api) {
    typedef std::pair< AsyncAPI::errorcode, std::string > result_pair;
    boost::fibers::promise< result_pair > promise;
    boost::fibers::future< result_pair > future( promise.get_future() );
    // We promise that both 'promise' and 'future' will survive until our lambda has been called.
    // Need C++14
    api.init_read([promise=std::move( promise)]( AsyncAPI::errorcode ec, std::string const& data) mutable {
                            promise.set_value( result_pair( ec, data) );
                  });
    return future.get();
}
```


### asio中的协程
asio的协程总感觉有两个版本，一个是C++ 20之前就有的版本，还有一个是在C++ 20的提供的协程的基础上封装的版本；asio的协程是无栈协程(后文会介绍)，无栈协程除了运行高效，节省内存之外，还能通过gdb查看到调用堆栈，有栈协程的堆栈因为被汇编切换走了是没法看到的，asio基于其io_context(详见asio的异步与线程模型解析 https://www.cnblogs.com/ishen/p/14593598.html )实现了多协程调度，所以要使用它的协程就就需要用到它的io_context（在此可以理解成一个跑着loop的协程调度器），该例子取自`asio/src/examples/cpp17/coroutines_ts/echo_server.cpp`

```c++
#include <asio/co_spawn.hpp>
#include <asio/detached.hpp>
#include <asio/io_context.hpp>
#include <asio/ip/tcp.hpp>
#include <asio/signal_set.hpp>
#include <asio/write.hpp>
#include <cstdio>
#include <iostream>

using asio::ip::tcp;
using asio::awaitable;
using asio::co_spawn;
using asio::detached;
using asio::use_awaitable;
namespace this_coro = asio::this_coro;

#if defined(ASIO_ENABLE_HANDLER_TRACKING)
# define use_awaitable \
  asio::use_awaitable_t(__FILE__, __LINE__, __PRETTY_FUNCTION__)
#endif

awaitable<void> echo(tcp::socket socket)
{
  try
  {
    char data[1024];
    for (;;)
    {
      std::size_t n = co_await socket.async_read_some(asio::buffer(data), use_awaitable);
      co_await async_write(socket, asio::buffer(data, n), use_awaitable);
    }
  }
  catch (std::exception& e)
  {
    std::printf("echo Exception: %s\n", e.what());
  }
}
void fn2(){
    std::cout<<"hhh\n";
}

void fn(){
    fn2();
}

awaitable<void> listener()
{
  auto executor = co_await this_coro::executor;
  fn(); 
  tcp::acceptor acceptor(executor, {tcp::v4(), 8988});
  for (;;)
  {
    tcp::socket socket = co_await acceptor.async_accept(use_awaitable); //调用协程，体现同步性
    co_spawn(executor, echo(std::move(socket)), detached);// 创建连接处理线程
  }
}

int main()
{
  try
  {
    asio::io_context io_context(1);

    asio::signal_set signals(io_context, SIGINT, SIGTERM);
    signals.async_wait([&](auto, auto){ io_context.stop(); });

    co_spawn(io_context, listener(), detached); // 创建纤程，体现并发性

    io_context.run();							// 开始调度
  }
  catch (std::exception& e)
  {
    std::printf("Exception: %s\n", e.what());
  }
}

```

代码很长，但只需要看main( )就可以了，co_spawn( )创建了一个协程，然后使用io_context.run( )，对基于该io_context创建的协程进行调度, 上面实现的协程函数listener( )中，使用`co_await acceptor.async_accept(use_awaitable)`做一个协程的阻塞同步调用，async_accept( )中发现没有新的连接就让出cpu给当前io_context下别的协程继续运行，当时间片又切回到该协程时，发现有新的链接时候，往io_context中创建一个新的协程去处理该连接，这里就能很好的体现了协程的同步和并发的应用场景，调度过程；

asio的协程是基于C++ 20实现的，简单的介绍因为asio库很通用，还没有精力继续研究，但可以先来看看C++ 20的协程给的基础设施。

### C++ 20的协程
C++ 20的协程（https://en.cppreference.com/w/cpp/language/coroutines ）目前只是一套框架基础，远未成熟，最好的文档参考还是cppreference，同时的这里两篇很好的文章进行了介绍文章1（ https://blog.csdn.net/guxch/article/details/111315994 ）和文章2（ https://blog.csdn.net/guxch/article/details/111590000#comments_15148577 ）

先看一个非常简化的例子看整体
```c++
#include <iostream>
#include <thread>
#include <coroutine>
#include <future>
#include <chrono>
#include <functional>

struct Result{
  struct promise_type {
    Result get_return_object() { return {}; }
    std::suspend_never initial_suspend() { return {}; }
    std::suspend_never final_suspend() noexcept { return {}; }
    void return_void() {}
    void unhandled_exception() {}
  };
};

std::coroutine_handle<> coroutine_handle;

struct AWaitableObject
{
	AWaitableObject() {}
    
	bool await_ready() const {return false;}

	int await_resume() { return 0; }

	void await_suspend(std::coroutine_handle<> handle){
        coroutine_handle = handle;
    }

};

Result CoroutineFunction()
{
    std::cout<<"start coroutine\n";
	int ret = co_await AWaitableObject(); 
    std::cout<<"finish coroutine\n";
}

int main()
{
    std::cout<<"start \n"; 
    auto coro = CoroutineFunction();
    std::cout<<"coroutine co_await\n"; 
    coroutine_handle.resume();

    return 0;
}
```

对该程序使用如下方式进行编译运行（需g++ 10.2.0及以上）
```
g++ test4.cpp -O0 -g -o test4 -fcoroutines -std=c++20
start 
start coroutine
coroutine co_await
finish coroutine
```

我们可以看到它的运行正如一般协程一样, 在使用了关键字`co_await`后会返回到caller, 在main中使用resume()后，回到co_await的赋值等式中运行

![enter description here](./images/1695735140528.png)

接下来，介绍目前C++ 协程的设计思想和细节
```c++
Results CoroutineFunction(){
	
	co_await AwaitatbleObject();
	
	co_return {};
}

```

一个协程函数形式如上，当函数体内出现了`co_await`, `co_yield`，`co_return`这三个关键字之后，就会被当成一个协程函数；此时，编译器要求返回值类型是否包含一个promise_type的结构以及需要实现必要的函数，以上一个例子中的Result类型为例:

```c++
struct Result{
  struct promise_type {
    Result get_return_object() { return {}; }
    std::suspend_never initial_suspend() { return {}; }
    std::suspend_never final_suspend() noexcept { return {}; }
    void unhandled_exception() {}
    
    suspend_aways yield_value(){} // 对应co_yield
    void return_void() {}	//对应co_return
    Result return_value(const Result& res){ return res;}
  };
};
```

c++ 20的编译器对于协程的运行有一套流程，我们可以通过提供**promise_type**去控制这个流程，同时对于协程的caller而言，协程运行后的只能获得返回值，所以希望通过它与协程进行后续交互的主要对象, 获取返回值， 处理异常等功能，所以这个很重要的控制器`struct promise_type`就放在了返回值类型Result中；

下面介绍promise_type的接口在协程运行如何交互，从头到尾，主要分成下面三个阶段

开头初始化准备:

* 协程函数运行后，首先生成一个promise_type对象
* 调用`get_return_object()`函数创建返回值对象，这个对象会在协程第一次返回时就会把这个对象返回给caller;
* 调用`initial_suspend()`函数，这个返回值有两个选择suspend_never/suspend_always，never表示继续运行，always表示协程挂起，同时把返回值对象返回，所以这个接口的语义是，协程创建后是否马上运行
运行:
* 开始运行协程函数，如果出现异常会调用`unhandled_exception()`去处理
* 如果遇到`co_yield var`这样的表达式，表示想要挂起当前协程，返回一个值给caller店, 编译器调用`yield_value(var)`方法，我们可以此时将值设置到Result的相关变量中，编译器会继续根据函数的返回值判断是否为**suspend_always**判断要返回到caller点
* 如果`co_return`这样的表达式，想要结束协程返回一个对象，则会调用`return_value()`这个函数，设置好要返回的相关值； 如果整个协程都没有出现**co_return**，则会调用`return_void()`

结束善后：
* 最后调用`final_suspend()` 判断协程已处理完毕释放前是否要挂起

其中有一个重要的关键字--**co_await**， 这是一个一元操作符，操作的对象为awaitable类型，就是实现**await_ready()**, **await_resume()**, **await_suspend( ) **的类型，如例子所示的**AWaitableObject**

```c++
struct AWaitableObject
{
	AWaitableObject() {}
    
	bool await_ready() const {return false;}

	int await_resume() { return 0; }

	void await_suspend(std::coroutine_handle<> handle){
        coroutine_handle = handle;
    }
};
```

当使用co_await awaitable_object时:
* 首先运行`await_ready( )`函数，判断是否要挂起当前线程: 如果是**false**，则不挂起； 如果是**true**，则表示要挂起，然后会调用`await_suspend()`，用于提供挂起前的处理，然后协程就被挂在这个点
* 一旦协程被恢复运行时，继续调用`await_resume()`在返回一个值到协程挂起点，如例子所示

![enter description here](./images/1695735796840.png)

co_await除了显示使用之外，promise_type的接口中凡是返回了suspend_never/suspend_always的地方，编译器都是通过co_await的方式调用这些函数的，suspend_never/suspend_always是awaitable类型

```c++
struct suspend_always
  {
    bool await_ready() { return false; }

    void await_suspend(coroutine_handle<>) {}

    void await_resume() {}
  };

  struct suspend_never
  {
    bool await_ready() { return true; }

    void await_suspend(coroutine_handle<>) {}

    void await_resume() {}
  };
```

每个协程都对应一个handle，用来管理协程的挂起和恢复，比如说`handle.resume()`就是用来恢复协程的运行的

协程handle的获取有两种方式：
* 第一种是通过co_await的await_suspend( )方法，该方法被调用时就能拿到协程的handle，但是这个方法肯定是不太好；
* 另一种方法是可以从promise_type对象中拿到，需要使用这个方法`coroutine_handle<promise_type>::from_promise(promise_type obj)`基于此，我们可以对返回值做如下改造

```c++
struct Result{
  //add
  Result(promise_type* obj):promise_type_ptr(obj){}
  //add
  void resume(){
  	promise_type_ptr->resume();
  }

  struct promise_type {
    // mod
    Result get_return_object() { 
    	return Reuslt(this);
   	}
   	
   	// add
   	void resume(){
   		coroutine_handle<promise_type>::from_promise(*this).resume();
   	}
   	
    std::suspend_never initial_suspend() { return {}; }
    std::suspend_never final_suspend() noexcept { return {}; }
    void unhandled_exception() {}
    
    suspend_aways yield_value(){}
    void return_void() {}	
    Result return_value(const Result& res){ return res;}
  };
  
  // add
  promise_type *promise_type_ptr;
};

```

则可以通过如下方式使用
```c++
auto result = CoroutineFunction();
result.resume();
```

从promise_type到awaitable object，C++ 20的协程目前提供的更多的是一个灵活的基础框架，离使用上还有一段距离

除此之外还有大量的优秀的协程库，比如基于C++ 20的libcopp, cppcoro，以及不依赖微信自己实现的libco(由于篇幅原因，libco介绍与实现分析不放在当前文章)

## 协程的一些应用场景
```c++
awaitable<void> listener()
{
  auto executor = co_await this_coro::executor;
  fn(); 
  tcp::acceptor acceptor(executor, {tcp::v4(), 8988});
  for (;;)
  {
    tcp::socket socket = co_await acceptor.async_accept(use_awaitable); //调用协程，体现同步性
    co_spawn(executor, echo(std::move(socket)), detached);// 创建连接处理线程
  }
}

int main()
{
  try
  {
    asio::io_context io_context(1);

    asio::signal_set signals(io_context, SIGINT, SIGTERM);
    signals.async_wait([&](auto, auto){ io_context.stop(); });

    co_spawn(io_context, listener(), detached); // 创建协程，体现并发性

    io_context.run();							// 开始调度
  }
  catch (std::exception& e)
  {
    std::printf("Exception: %s\n", e.what());
  }
}
```

在asio的例子中很好的介绍了协程的使用方式了，主要是不断的创建协程，让调度器调度运行，在协程运行过程对于一些会阻塞的条件，做一个非阻塞的检测中,发现条件不满足就让出cpu，这就是常见轮转+非阻塞同步。

## 协程的分类
### 有栈协程和无栈协程

协程可以分成有栈stackful和无栈stackless两种，比如，libco就是有栈协程, 每个协程创建的时候都会获得一块128k的堆内存，协程运行的时候就是使用这块堆内存当作运行栈使用，切换时候保存/恢复运行栈和相应寄存器，而无栈协程不需要这些，因为无栈协程的实现原理并不是通过切换时保存/恢复运行栈和寄存器实现的，它的实现见下，由于协程的每个中断点都是确定，那其实只需要将函数的代码再进行细分，保存好局部变量，做好调用过程的状态变化, 下面就将一个协程函数fn进行切分后变成一个Struct，这样的实现相对于有栈协程而言使用的内存更少，因为有栈协程的运行栈由堆获得，必须要保证运行栈充足，然而很多时候用不到这么多的内存，会造成内存浪费；

```c++
void fn(){
	int a, b, c;
	a = b + c;
	yield();
	b = c + a;
	yield();
	c = a + b;
}

----------------------------分割线---------------------------------
Struct fn{
	int a, b, c;
	int __state = 0;
	
	void resume(){
		switch(__state) {
        case 0:
             return fn1();
        case 1:
             return fn2();
        case 2:
        	 return fn3();
        }
	}
	
	void fn1(){
		a = b + c;
	}
	
	void fn2(){
		b = c + a;
	}
	
	void fn3(){
		c = a + b;
	}
};
```

## 对称和非对称
boost.coroutine2和libco这类属于非对称协程，这类协程的特点是存在调用链，有调用和返回的关系，比如说coroutine2中进行source()的时候去调用协程了，协程执行到阻塞点sink()返回，而不是让出cpu，随便执行别的协程；


# asio的异步与线程模型解析
来源：https://www.cnblogs.com/ishen/p/14593598.html

本文使用的asio1.76.0从github（ https://github.com/boostorg/asio ）获取，老版本中有一个很重要的结构叫做io_service，新版本中改成了io_context,下面主要通过c++ 14下的一个异步的例子分析boost.asio的线程模型，其代码结构比较复杂，时间有限不能分析的很详细，只是做大体结构分析

## 正文
asio的线程模型和异步的调用如下图

![enter description here](./images/1695736187840.png)
程序以一个io_context为核心，其下有一个scheduler对象(调度器)，scheduler下面放着一个(op_queue_)任务队列，一个epoll_fd，执行io_context.run()的时候，开始一个循环去竞争消费任务队列；任务队列有两种任务，一种是epoll任务，就是去做一次epoll_wait，还有一种任务是执行回调用函数；当使用一个异步调用的时候，本质就是往reactor中添加一个fd的event，将其回调保存进去 event._data中，当执行epoll_wait()监听到该event事件后，取出该fd对应的callback放入到任务队列(op_queue)，由不同的线程去竞争消费；

拿官方的例子简单分析这个流程，该例子来源于`asio/src/examples/cpp14/echo/async_tcp_echo_server.cpp`，

```c++
#include <cstdlib>
#include <iostream>
#include <memory>
#include <utility>
#include <asio/ts/buffer.hpp>
#include <asio/ts/internet.hpp>

using asio::ip::tcp;

class session
  : public std::enable_shared_from_this<session>
{
public:
  session(tcp::socket socket)
    : socket_(std::move(socket))
  {
  }

  void start()
  {
    do_read();
  }

private:
  void do_read()
  {
    auto self(shared_from_this());
    socket_.async_read_some(asio::buffer(data_, max_length),
        [this, self](std::error_code ec, std::size_t length)
        {
          if (!ec)
          {
            do_write(length);
          }
        });
  }

  void do_write(std::size_t length)
  {
    auto self(shared_from_this());
    asio::async_write(socket_, asio::buffer(data_, length),
        [this, self](std::error_code ec, std::size_t /*length*/)
        {
          if (!ec)
          {
            do_read();
          }
        });
  }

  tcp::socket socket_;
  enum { max_length = 1024 };
  char data_[max_length];
};

class server
{
public:
  server(asio::io_context& io_context, short port)
    : acceptor_(io_context, tcp::endpoint(tcp::v4(), port)),
      socket_(io_context)
  {
    do_accept();
  }

private:
  void do_accept()
  {
    acceptor_.async_accept(socket_,
        [this](std::error_code ec)
        {
          if (!ec)
          {
            std::make_shared<session>(std::move(socket_))->start();
          }

          do_accept();
        });
  }

  tcp::acceptor acceptor_;
  tcp::socket socket_;
};

int main(int argc, char* argv[])
{
  try
  {
    if (argc != 2)
    {
      std::cerr << "Usage: async_tcp_echo_server <port>\n";
      return 1;
    }

    asio::io_context io_context;

    server s(io_context, std::atoi(argv[1]));

    io_context.run();
  }
  catch (std::exception& e)
  {
    std::cerr << "Exception: " << e.what() << "\n";
  }

  return 0;
}

```

代码很长，这里重点分析io_context.run();和acceptor_.async_accept以分析其线程模型和异步，main()中创建了io_context 和 server, server中创建了acceptor和socket，然后执行一个异步async_accept，将一个lambda函数注册进去，accept事件发生了，进行accept创建客户端的socket后，调用该lambda回调去处理新accept的socket; 然后执行io_content.run(),做一个循环监听

程序的调用图如下所示(原图 http://naotu.baidu.com/file/a0499ba782512b23907c2fd57aa8bb39?token=701f778af7f82040 )

![enter description here](./images/1695736280955.png)
如图上标黄，主要分成3个比较重要的部分:

第一部分是acceptor的创建，放入epoll进行监听, 在server的构造函数中开始构造acceptor

```c++
class server
{
public:
  server(asio::io_context& io_context, short port)
    : acceptor_(io_context, tcp::endpoint(tcp::v4(), port)), //构造acceptor
      socket_(io_context)
  {
    do_accept();
  }

private:
	......

  tcp::acceptor acceptor_;
  tcp::socket socket_;
};
```

但其实是用了一个typedef，构造的是`class basic_socket_acceptor<tcp>`，在这里，除了看到了熟悉的bind和listen，重要的是open

```c++
  // /usr/local/include/asio/basic_socket_acceptor.hpp
  
  template <typename ExecutionContext>
  basic_socket_acceptor(ExecutionContext& context,
      const endpoint_type& endpoint, bool reuse_addr = true,
      typename constraint<
        is_convertible<ExecutionContext&, execution_context&>::value
      >::type = 0)
    : impl_(0, 0, context)
  {
    asio::error_code ec;
    const protocol_type protocol = endpoint.protocol();
    impl_.get_service().open(impl_.get_implementation(), protocol, ec); //open
    asio::detail::throw_error(ec, "open");
    if (reuse_addr)
    {
      impl_.get_service().set_option(impl_.get_implementation(),
          socket_base::reuse_address(true), ec);
      asio::detail::throw_error(ec, "set_option");
    }
    impl_.get_service().bind(impl_.get_implementation(), endpoint, ec);// bind
    asio::detail::throw_error(ec, "bind");
    impl_.get_service().listen(impl_.get_implementation(),			   // listen
        socket_base::max_listen_connections, ec);
    asio::detail::throw_error(ec, "listen");
  }
```

`open()`是`do_open()`的封装，为了实现跨平台, 作用是创建一个socket，并为其添加相应epoll event

```c++
  // Open a new socket implementation.
  asio::error_code open(implementation_type& impl,
      const protocol_type& protocol, asio::error_code& ec)
  {
    if (!do_open(impl, protocol.family(),
          protocol.type(), protocol.protocol(), ec))
      impl.protocol_ = protocol;
    return ec;
  }
```

以下是do_open( )，socket_holder创建了server端的socket, register_descriptor( )将该socket注册到了该io_context上的epoll fd上

```c++
asio::error_code reactive_socket_service_base::do_open(
    reactive_socket_service_base::base_implementation_type& impl,
    int af, int type, int protocol, asio::error_code& ec)
{
  if (is_open(impl))
  {
    ec = asio::error::already_open;
    return ec;
  }

  socket_holder sock(socket_ops::socket(af, type, protocol, ec)); //创建了socket
  if (sock.get() == invalid_socket)
    return ec;

  // 注册到epoll上
  if (int err = reactor_.register_descriptor(sock.get(), impl.reactor_data_))
  {
    ec = asio::error_code(err,
        asio::error::get_system_category());
    return ec;
  }

  impl.socket_ = sock.release();
  switch (type)
  {
  case SOCK_STREAM: impl.state_ = socket_ops::stream_oriented; break;
  case SOCK_DGRAM: impl.state_ = socket_ops::datagram_oriented; break;
  default: impl.state_ = 0; break;
  }
  ec = asio::error_code();
  return ec;
}
```

到此，一个完整的server的socket就创建完毕，并且进行bind和listen，并放入到epoll中

第二个重要的部分是`async_accept()`这个异步的运行

```c++
class server
{
	......
private:
  void do_accept()
  {
    acceptor_.async_accept(socket_,  //执行异步调用
        [this](std::error_code ec)
        {
          if (!ec)
          {
            std::make_shared<session>(std::move(socket_))->start();
          }

          do_accept();
        });
  }

  tcp::acceptor acceptor_;
  tcp::socket socket_;
};
```

使用第一部分创建好的acceptor进行async_accept()后，来到一个看起来很复杂的模板函数

```c++
  template <typename Protocol1, typename Executor1,
      ASIO_COMPLETION_TOKEN_FOR(void (asio::error_code))
        AcceptHandler ASIO_DEFAULT_COMPLETION_TOKEN_TYPE(executor_type)>
  ASIO_INITFN_AUTO_RESULT_TYPE(AcceptHandler,
      void (asio::error_code))
  async_accept(basic_socket<Protocol1, Executor1>& peer,
      ASIO_MOVE_ARG(AcceptHandler) handler
        ASIO_DEFAULT_COMPLETION_TOKEN(executor_type),
      typename constraint<
        is_convertible<Protocol, Protocol1>::value
      >::type = 0)
  {
    return async_initiate<AcceptHandler, void (asio::error_code)>(
        initiate_async_accept(this), handler,
        &peer, static_cast<endpoint_type*>(0));
  }
```

不用太在意头，看看`async_initiate()`做了啥

```c++
template <typename CompletionToken,
    ASIO_COMPLETION_SIGNATURE Signature,
    typename Initiation, typename... Args>
inline typename constraint<
    !detail::async_result_has_initiate_memfn<CompletionToken, Signature>::value,
    ASIO_INITFN_RESULT_TYPE(CompletionToken, Signature)>::type
async_initiate(ASIO_MOVE_ARG(Initiation) initiation,
    ASIO_NONDEDUCED_MOVE_ARG(CompletionToken) token,
    ASIO_MOVE_ARG(Args)... args)
{
  async_completion<CompletionToken, Signature> completion(token);

  ASIO_MOVE_CAST(Initiation)(initiation)(
      ASIO_MOVE_CAST(ASIO_HANDLER_TYPE(CompletionToken,
        Signature))(completion.completion_handler),
      ASIO_MOVE_CAST(Args)(args)...);

  return completion.result.get();
}
```

来到了一个更加复杂的模板函数，但是函数体只有两三句话，发现其调用的时候模板类Initiation的operator( )，这个模板类是`class initiate_async_accept`, 其对应的operator( )如下

```c++
    template <typename AcceptHandler, typename Protocol1, typename Executor1>
    void operator()(ASIO_MOVE_ARG(AcceptHandler) handler,
        basic_socket<Protocol1, Executor1>* peer,
        endpoint_type* peer_endpoint) const
    {
      // If you get an error on the following line it means that your handler
      // does not meet the documented type requirements for a AcceptHandler.
      ASIO_ACCEPT_HANDLER_CHECK(AcceptHandler, handler) type_check;

      detail::non_const_lvalue<AcceptHandler> handler2(handler);
      self_->impl_.get_service().async_accept(
          self_->impl_.get_implementation(), *peer, peer_endpoint,
          handler2.value, self_->impl_.get_executor());
    }
```

核心就一句`self_->impl_.get_service().async_accept(...)`, 这个impl_.get_service()获取到的是`class reactive_socket_service`，在basic_socket_acceptor中可以看到

```c++
class basic_socket_acceptor
  : public socket_base
{
...
detail::io_object_impl<detail::reactive_socket_service<Protocol>, Executor> impl_;
}
```

为此，找到了async_accept(..)的实现如下，看起来很长，但重点在最后的`start_accept_op()`
```c++
  // Start an asynchronous accept. The peer and peer_endpoint objects must be
  // valid until the accept's handler is invoked.
  template <typename Socket, typename Handler, typename IoExecutor>
  void async_accept(implementation_type& impl, Socket& peer,
      endpoint_type* peer_endpoint, Handler& handler, const IoExecutor& io_ex)
  {
    bool is_continuation =
      asio_handler_cont_helpers::is_continuation(handler);

    // Allocate and construct an operation to wrap the handler.
    typedef reactive_socket_accept_op<Socket, Protocol, Handler, IoExecutor> op;
    typename op::ptr p = { asio::detail::addressof(handler),
      op::ptr::allocate(handler), 0 };
    p.p = new (p.v) op(success_ec_, impl.socket_, impl.state_,
        peer, impl.protocol_, peer_endpoint, handler, io_ex);

    ASIO_HANDLER_CREATION((reactor_.context(), *p.p, "socket",
          &impl, impl.socket_, "async_accept"));

    start_accept_op(impl, p.p, is_continuation, peer.is_open()); // 开始accept 的操作
    p.v = p.p = 0;
  }
```

`start_accept_op()`调用了`start_op()`
```c++
void reactive_socket_service_base::start_accept_op(
    reactive_socket_service_base::base_implementation_type& impl,
    reactor_op* op, bool is_continuation, bool peer_is_open)
{
  if (!peer_is_open)
    start_op(impl, reactor::read_op, op, is_continuation, true, false);
  else
  {
    op->ec_ = asio::error::already_open;
    reactor_.post_immediate_completion(op, is_continuation);
  }
}
```

而`start_op()`中调用的是reactor.start_op()

```c++
void reactive_socket_service_base::start_op(
    reactive_socket_service_base::base_implementation_type& impl,
    int op_type, reactor_op* op, bool is_continuation,
    bool is_non_blocking, bool noop)
{
  if (!noop)
  {
    if ((impl.state_ & socket_ops::non_blocking)
        || socket_ops::set_internal_non_blocking(
          impl.socket_, impl.state_, true, op->ec_))
    {
      reactor_.start_op(op_type, impl.socket_,
          impl.reactor_data_, op, is_continuation, is_non_blocking); //调用reactor的start_op()
      return;
    }
  }

  reactor_.post_immediate_completion(op, is_continuation);
}
```

reacttor.start_op()很长，截取重点如下,其会根据传入的op_type（read_op = 0, write_op = 1,connect_op = 1, except_op = 2） 做一些event的监听事件的epoll_mod，然后把operation（就是传入的回调）放入当前的描述符的op_type下标对应的数组中,用于触发的时候去运行

```c++
void epoll_reactor::start_op(int op_type, socket_type descriptor,
    epoll_reactor::per_descriptor_data& descriptor_data, reactor_op* op,
    bool is_continuation, bool allow_speculative)
{
 
	...
  if (op_type == write_op)
      {
        if ((descriptor_data->registered_events_ & EPOLLOUT) == 0)
        {
          epoll_event ev = { 0, { 0 } };
          ev.events = descriptor_data->registered_events_ | EPOLLOUT;
          ev.data.ptr = descriptor_data;
          if (epoll_ctl(epoll_fd_, EPOLL_CTL_MOD, descriptor, &ev) == 0)
          {
            descriptor_data->registered_events_ |= ev.events;
          }
          else
          {
            op->ec_ = asio::error_code(errno,
                asio::error::get_system_category());
            scheduler_.post_immediate_completion(op, is_continuation);
            return;
          }
        }
      }
  ...

  descriptor_data->op_queue_[op_type].push(op);
  scheduler_.work_started();
}
```

这一小节整体看下来，差不多可以得到这样的关系 acceptor->reacotr_socket_service->reactor

第三部分是io_context.run()的时候如何进行epoll( )并处理事件的

从main开始进行io_context.run( )

```c++
int main(int argc, char* argv[])
{
  try
  {
    if (argc != 2)
    {
      std::cerr << "Usage: async_tcp_echo_server <port>\n";
      return 1;
    }

    asio::io_context io_context;

    server s(io_context, std::atoi(argv[1]));

    io_context.run(); //开始循环处理
  }
  catch (std::exception& e)
  {
    std::cerr << "Exception: " << e.what() << "\n";
  }

  return 0;
}

```

实指执行了impl_.run()

```c++
io_context::count_type io_context::run()
{
  asio::error_code ec;
  count_type s = impl_.run(ec);
  asio::detail::throw_error(ec);
  return s;
}	
```

其实指运行的是scheduler::run( )，看到其中有一个for循环运行do_run_one( )处理

```c++
std::size_t scheduler::run(asio::error_code& ec)
{
  ec = asio::error_code();
  if (outstanding_work_ == 0)
  {
    stop();
    return 0;
  }

  thread_info this_thread;
  this_thread.private_outstanding_work = 0;
  thread_call_stack::context ctx(this, this_thread);

  mutex::scoped_lock lock(mutex_);

  std::size_t n = 0;
  for (; do_run_one(lock, this_thread, ec); lock.lock()) //for循环处理
    if (n != (std::numeric_limits<std::size_t>::max)())
      ++n;
  return n;
}
```

do_run_one( )比较长，但是很好理解，其从任务队列获取任务，然后判断是否为task，如果是则说明需要做一次epoll对去接收fd的事件回来进行处理，如果不是，则是回调任务，把任务对应的回调函数运行即可；

其次，可以看到当判断任务有多个的时候会执行`wakeup_event_.unlock_and_signal_one(lock)`;去唤醒别的线程也来消费该任务队列，锁的管理上，在上面的for循环上，一直都加有锁，而下面真正获取了任务去执行的时候就会把锁给放掉，同时用task_cleanup这个RAII的类，在处理完任务之后把锁给加回来；

```c++
std::size_t scheduler::do_run_one(mutex::scoped_lock& lock,
    scheduler::thread_info& this_thread,
    const asio::error_code& ec)
{
  while (!stopped_)
  {
    if (!op_queue_.empty())
    {
      // Prepare to execute first handler from queue.
      operation* o = op_queue_.front(); //从任务队列中获取任务
      op_queue_.pop();
      bool more_handlers = (!op_queue_.empty());

      if (o == &task_operation_) // epoll任务的话, 进行一次epoll,
      {
        task_interrupted_ = more_handlers;

        if (more_handlers && !one_thread_)
          wakeup_event_.unlock_and_signal_one(lock);//唤醒其它线程
        else
          lock.unlock();

        task_cleanup on_exit = { this, &lock, &this_thread };
        (void)on_exit;

        // Run the task. May throw an exception. Only block if the operation
        // queue is empty and we're not polling, otherwise we want to return
        // as soon as possible.
        task_->run(more_handlers ? 0 : -1, this_thread.private_op_queue);
      }
      else //回调的话，进行回调
      {
        std::size_t task_result = o->task_result_;

        if (more_handlers && !one_thread_)
          wake_one_thread_and_unlock(lock);
        else
          lock.unlock();

        // Ensure the count of outstanding work is decremented on block exit.
        work_cleanup on_exit = { this, &lock, &this_thread };
        (void)on_exit;

        // Complete the operation. May throw an exception. Deletes the object.
        o->complete(this, ec, task_result); //调用回调函数
        this_thread.rethrow_pending_exception();

        return 1;
      }
    }
    else
    {
      wakeup_event_.clear(lock);
      wakeup_event_.wait(lock);
    }
  }

  return 0;
}

```

这里细看一下task->run( )进行epoll时是怎么收任务的, 将epoll返回的event中的事件保存下来以传给回调函数处理事件(上面调用回调函数时候传递的task_result)，把这个描述符数据(包含了回调和结果)放入任务队列；这里有一个疑惑，epoll_reactor::run（）这个函数是没有做加锁操作的，可epoll fd是共享的，存在多个线程同时epoll( )的问题需要做同步，不知道其怎么实现的

```c++
void epoll_reactor::run(long usec, op_queue<operation>& ops)
{

  // Block on the epoll descriptor.
  epoll_event events[128];
  int num_events = epoll_wait(epoll_fd_, events, 128, timeout);


  // Dispatch the waiting events.
  for (int i = 0; i < num_events; ++i)
  {
		...
		void* ptr = events[i].data.ptr;
		descriptor_state* descriptor_data = static_cast<descriptor_state*>(ptr);
        descriptor_data->set_ready_events(events[i].events); //保存事件
        ops.push(descriptor_data);	// 放入任务队列
        ...
  }
}
```



# Asio与Boost.Asio
来源：https://www.cnblogs.com/kolane/p/12071089.html

译自 http://think-async.com/Asio/AsioAndBoostAsio

Asio有两种变体：(非Boost)Asio和Boost.Asio。本文概要描述二者的不同。

## 1. 源代码的差别

* Asio位于名字空间asio::中，而Boost.Asio则位于boost::asio::中。
* Asio的主要头文件是asio.hpp，而Boost.Asio的则是boost/asio.hpp，所有其他头文件作了类似的改动。
* Asio使用或者定义的宏有前缀ASIO_，而Boost.Asio中宏的前缀则是BOOST_ASIO_。
* Asio含有启动线程的类asio::thread，Boost.Asio没有这个类，以免与Boost.Thread库功能重叠。
* Boost.Asio使用Boost.System库提供错误码支持（boost::system::error_code和boost::system::system_error），Asio则将其包含在自己的名字空间中（asio::error_code和asio::system_error）。Boost.System版本的这些类当前能够更好地支持用户定义的错误码。
* Asio只有头文件，多数情况下不需要链接任何Boost库，而Boost.Asio总是要求链接Boost.System库，如果要使用boost::thread启动线程，则还要链接Boost.Thread库。

## 2. 从哪里获取发布包？
Asio可以从SourceForge（ https://sourceforge.net/projects/asio/ ）下载，包名是asio-X.Y.Z.tar.gz（或者.tar.bz2，或者.zip）。

Boost.Asio包含在Boost 1.35发布版中。也可以从SourceForge下载名字为boost_asio_X_Y_Z.tar.gz的单独包。应该把下载的包复制到已有的Boost源代码发布中。

## 3. 源代码库在哪里？
Asio使用sourcforge中的CVS仓库 （ https://sourceforge.net/p/asio/cvs/ ）。关于如何访问CVS仓库的细节请看这里，仓库也可以通过Web浏览。

Boost.Asio的源代码在Boost的SVN代码仓库中。

## 4. 两个版本是如何维护的？
所有的开发都在Asio的CVS仓库中进行。源代码被定期地通过boostify.pl脚本转换成Boost格式，然后将改动合并到Boost的SVN仓库中。

## 5. 现在Boost已经包含Boost.Asio，Asio会不再更新吗？
不会，使用Asio的项目会被持续支持。

## 6. 应该使用Asio还是Boost.Asio？
这取决于各方面的考虑：

如果你选择只有头文件的便利性，则建议使用Asio，而不是Boost.Asio。

如果必须使用1.35版本之前的不包含Boost.Asio的Boost，可以将Boost.Asio复制到Boost发布版本中，但有些人可能不习惯这样做。如果是这样，建议使用Asio，而不是Boost.Asio。

Asio和Boost.Asio的新版本发布周期比Boost短。如果想使用最新的特征，只要将Boost.Asio复制到Boost发布版本中就可以了。如果不想这么做，使用Asio就是了。

## 7. Asio和Boost.Asio可以共存于一个程序中吗？
可以。虽然类型本身显然是不可互换的，但是二者使用不同的名字空间，应该不会有冲突。（如果想知道为什么需要这样做，考虑下程序使用第三方库，而第三方库在内部使用Asio的情况）