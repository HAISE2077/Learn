---
title: _C++ 游戏后端
---

# C++ 高性能协程RPC框架设计
来源：https://zhuanlan.zhihu.com/p/460646015

本项目将从零开始搭建出一个基于协程的异步RPC框架。

学习本项目需要有一定的C++ ，网络，RPC知识
项目地址：zavier-wong/acid: A high performance fiber RPC network framework. 高性能协程RPC网络框架 (github.com) https://link.zhihu.com/?target=https%3A//github.com/zavier-wong/acid

通过本项目你将学习到：

* 协程同步原语
* 序列化协议
* 通信协议
* 连接复用
* 服务注册
* 服务发现
* 服务订阅与通知
* 负载均衡
* 健康检查
* 三种异步调用方式

相信大家对RPC的相关概念都已经很熟悉了，这里不做过多介绍，直接进入重点。 本文档将简单介绍框架的设计，在最后的 examples 会给出完整的使用范例。 更多的细节需要仔细阅读源码。

## RPC框架设计
本RPC框架主要有网络模块， 序列化模块，通信协议模块，客户端模块，服务端模块，服务注册中心模块，负载均衡模块

主要有以下三个角色：

### 注册中心 Registry
主要是用来完成服务注册和服务发现的工作。同时需要维护服务下线机制，管理了服务器的存活状态。

### 服务提供方 Service Provider
其需要对外提供服务接口，它需要在应用启动时连接注册中心，将服务名发往注册中心。服务端还需要启动Socket服务监听客户端请求。

### 服务消费方 Service Consumer
客户端需要有从注册中心获取服务的基本能力，它需要在发起调用时， 从注册中心拉取开放服务的服务器地址列表存入本地缓存， 然后选择一种负载均衡策略从本地缓存中筛选出一个目标地址发起调用， 并将这个连接存入连接池等待下一次调用。

### 协程同步原语
我们都知道，一旦协程阻塞后整个协程所在的线程都将阻塞，这也就失去了协程的优势。编写协程程序时难免会对一些数据进行同步，而Linux下常见的同步原语互斥量、条件变量、信号量等基本都会堵塞整个线程，使用原生同步原语协程性能将大幅下降，甚至发生死锁的概率大大增加。

框架实现了一套协程同步原语来解决原生同步原语带来的阻塞问题，在协程同步原语之上实现更高层次同步的抽象——Channel用于协程之间的便捷通信。

具体设计思路见 通用协程同步原语设计 https://zhuanlan.zhihu.com/p/475554418

### 序列化协议
本模块支持了基本类型以及标准库容器的序列化，包括：

* 顺序容器：string, list, vector
* 关联容器：set, multiset, map, multimap
* 无序容器：unordered_set, unordered_multiset, unordered_map, unordered_multimap
* 异构容器：tuple

以及通过以上任意组合嵌套类型的序列化

序列化有以下规则：

1. 默认情况下序列化，8，16位类型以及浮点数不压缩，32，64位有符号/无符号数采用 zigzag 和 varints 编码压缩

2. 针对 std::string 会将长度信息压缩序列化作为元数据，然后将原数据直接写入。char数组会先转换成 std::string 后按此规则序列化

3. 调用 writeFint 将不会压缩数字，调用 writeRowData 不会加入长度信息

对于任意用户自定义类型，只要实现了以下的重载，即可参与传输时的序列化。
```c++
template<typename T>
Serializer &operator >> (Serializer& in, T& i){
   return *this;
}
template<typename T>
Serializer &operator << (Serializer& in, T i){
    return *this;
}
```

rpc调用过程：

* 调用方发起过程调用时，自动将参数打包成tuple，然后序列化传输。
* 被调用方收到调用请求时，先将参数包反序列回tuple，再解包转发给函数。

### 通信协议
>+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
|  BYTE  |        |        |        |        |        |        |        |        |        |        |             ........                                                           |
+--------------------------------------------+--------+--------------------------+--------+-----------------+--------+--------+--------+--------+--------+--------+-----------------+
|  magic | version|  type  |          sequence id              |          content length           |             content byte[]                                                     |
+--------+-----------------------------------------------------------------------------------------------------------------------------+--------------------------------------------+

封装通信协议，使RPC Server和RPC Client 可以基于同一协议通信。

采用私有通信协议，协议如下：
magic: 协议魔法数字
version: 协议版本号，以便对协议进行扩展，使用不同的协议解析器。
type: 消息请求类型
sequence id: 一个32位序列号，用来识别请求顺序。
content length: 消息长度，即后面要接收的内容长度。
content byte: 消息具体内容

目前提供了以下几种请求
```c++
enum class MsgType : uint8_t {
    HEARTBEAT_PACKET,       // 心跳包
    RPC_PROVIDER,           // 向服务中心声明为provider
    RPC_CONSUMER,           // 向服务中心声明为consumer

    RPC_REQUEST,            // 通用请求
    RPC_RESPONSE,           // 通用响应

    RPC_METHOD_REQUEST ,    // 请求方法调用
    RPC_METHOD_RESPONSE,    // 响应方法调用

    RPC_SERVICE_REGISTER,   // 向中心注册服务
    RPC_SERVICE_REGISTER_RESPONSE,

    RPC_SERVICE_DISCOVER,   // 向中心请求服务发现
    RPC_SERVICE_DISCOVER_RESPONSE

    RPC_SUBSCRIBE_REQUEST,  // 订阅
    RPC_SUBSCRIBE_RESPONSE,

    RPC_PUBLISH_REQUEST,    // 发布
    RPC_PUBLISH_RESPONSE
};
```

### 连接复用
对于短连接来说，每次发起rpc调用就创建一条连接，由于没有竞争实现起来比较容易，但开销太大。所以本框架实现了rpc连接复用来支持更高的并发。

连接复用的问题在于，在一条连接上可以有多个并发的调用请求，由于服务器也是并发处理这些请求的，所以导致了服务器返回的响应顺序与请求顺序不一致。

具体的解决方法见 rpc连接复用设计 https://zhuanlan.zhihu.com/p/476741943

### 服务注册
每一个服务提供者对应的机器或者实例在启动运行的时候， 都去向注册中心注册自己提供的服务以及开放的端口。 注册中心维护一个服务名到服务地址的多重映射，一个服务下有多个服务地址， 同时需要维护连接状态，断开连接后移除服务。
```c++
/**
 * 维护服务名和服务地址列表的多重映射
 * serviceName -> serviceAddress1
 *             -> serviceAddress2
 *             ...
 */
std::multimap<std::string, std::string> m_services;
```

### 服务发现
虽然服务调用是服务消费方直接发向服务提供方的，但是分布式的服务，都是集群部署的， 服务的提供者数量也是动态变化的， 所以服务的地址也就无法预先确定。 因此如何发现这些服务就需要一个统一注册中心来承载。

客户端从注册中心获取服务，它需要在发起调用时， 从注册中心拉取开放服务的服务器地址列表存入本地缓存。


### 服务订阅与通知
当一个已有服务提供者节点下线， 或者一个新的服务提供者节点加入时，订阅对应接口的消费者能及时收到注册中心的通知， 并更新本地的服务地址缓存。 这样后续的服务调用就能避免调用已经下线的节点， 并且能调用到新的服务节点。

发布/订阅模式就是解决该问题的重要方法，具体思路见 服务订阅与通知

### 负载均衡
实现通用类型负载均衡路由引擎（工厂）。 通过路由引擎获取指定枚举类型的负载均衡器， 降低了代码耦合，规范了各个负载均衡器的使用，减少出错的可能。

提供了三种路由策略（随机、轮询、哈希）, 由客户端使用，在客户端实现负载均衡

```golang
/**
 * @brief: 路由均衡引擎
 */
template<class T>
class RouteEngine {
public:
    static typename RouteStrategy<T>::ptr queryStrategy(typename RouteStrategy<T>::Strategy routeStrategyEnum) {
        switch (routeStrategyEnum){
            case RouteStrategy<T>::Random:
                return s_randomRouteStrategy;
            case RouteStrategy<T>::Polling:
                return std::make_shared<impl::PollingRouteStrategyImpl<T>>();
            case RouteStrategy<T>::HashIP:
                return s_hashIPRouteStrategy ;
            default:
                return s_randomRouteStrategy ;
        }
    }
private:
    static typename RouteStrategy<T>::ptr s_randomRouteStrategy;
    static typename RouteStrategy<T>::ptr s_hashIPRouteStrategy;
};
```

选择客户端负载均衡策略，根据路由策略选择服务地址。默认随机策略。
```c++
RouteStrategy<int>::ptr strategy =
            RouteEngine<int>::queryStrategy(Strategy::Random);
```

客户端同时会维护RPC连接池，以及服务发现结果缓存，减少频繁建立连接。

通过上述策略尽量消除或减少系统压力及系统中各节点负载不均衡的现象。

### 心跳机制
服务中心必须管理服务器的存活状态，也就是健康检查。 注册服务的这一组机器，当这个服务组的某台机器如果出现宕机或者服务死掉的时候， 就会剔除掉这台机器。这样就实现了自动监控和管理。

项目采用了心跳发送的方式来检查健康状态。

服务器端： 开启一个定时器，定期给注册中心发心跳包，注册中心会回一个心跳包。

注册中心： 开启一个定时器倒计时，每次收到一个消息就更新一次定时器。如果倒计时结束还没有收到任何消息，则判断服务掉线。

### 三种异步调用方式
整个框架最终都要落实在服务消费者。为了方便用户，满足用户的不同需求，项目设计了三种异步调用方式。 三种调用方式的模板参数都是返回值类型，对void类型会默认转换uint8_t 。

#### 1. 以同步的方式异步调用

整个框架本身基于协程池，所以在遇到阻塞时会自动调度实现以同步的方式异步调用
```c++
Result<int> res = con->call<int>("add", 123, 321);
LOG_INFO(g_logger) << res.getVal();
```

#### 2. future 形式的异步调用

调用时会立即返回一个future
```c++
future<Result<int>> res = con->async_call<int>("add", 123, 321);
LOG_INFO(g_logger) << res.get().getVal();
```

#### 3. 异步回调

async_call的第一个参数为函数时，启用回调模式，回调参数必须是返回类型的包装。收到消息时执行回调。
```c++
con->async_call<int>([](Result<int> res){
    LOG_INFO(g_logger) << res.getVal();
}, "add", 123, 321);
```

对调用结果及状态的封装如下
```c++
/**
 * @brief RPC调用状态
 */
enum RpcState{
    RPC_SUCCESS = 0,    // 成功
    RPC_FAIL,           // 失败
    RPC_NO_METHOD,      // 没有找到调用函数
    RPC_CLOSED,         // RPC 连接被关闭
    RPC_TIMEOUT         // RPC 调用超时
};

/**
 * @brief 包装 RPC调用结果
 */
template<typename T = void>
class Result {
    ...
private:
    /// 调用状态
    code_type m_code = 0;
    /// 调用消息
    msg_type m_msg;
    /// 调用结果
    type m_val;
}
```

### 最后
本框架为使用者提供了快速构建分布式服务的能力。使用者可方便地启动注册中心，搭建服务集群，自动服务注册、发现，自动负载均衡。

通过以上介绍，我们粗略地了解了分布式服务的大概流程。但篇幅有限，无法面面俱到， 更多细节就需要去阅读代码来理解了。

这并不是终点，项目只是实现了简单的服务注册、发现。后续将考虑加入注册中心集群，限流，熔断，监控节点等。

## 示例
rpc服务注册中心
```c++
#include "acid/rpc/rpc_service_registry.h"

// 服务注册中心
void Main() {
    acid::Address::ptr address = acid::Address::LookupAny("127.0.0.1:8080");
    acid::rpc::RpcServiceRegistry::ptr server = std::make_shared<acid::rpc::RpcServiceRegistry>();
    // 服务注册中心绑定在8080端口
    while (!server->bind(address)){
        sleep(1);
    }
    server->start();
}

int main() {
    go Main;
}
```

rpc 服务提供者
```c++
#include "acid/rpc/rpc_server.h"

int add(int a,int b){
    return a + b;
}

// 向服务中心注册服务，并处理客户端请求
void Main() {
    int port = 9000;
    acid::Address::ptr local = acid::IPv4Address::Create("127.0.0.1",port);
    acid::Address::ptr registry = acid::Address::LookupAny("127.0.0.1:8080");

    acid::rpc::RpcServer::ptr server = std::make_shared<acid::rpc::RpcServer>();;

    // 注册服务，支持函数指针
    server->registerMethod("add",add);
    // 支持函数对象
    server->registerMethod("echo", [](std::string str){
        return str;
    });
    // 支持标准库容器
    server->registerMethod("revers", [](std::vector<std::string> vec) -> std::vector<std::string>{
        std::reverse(vec.begin(), vec.end());
        return vec;
    });

    // 先绑定本地地址
    while (!server->bind(local)){
        sleep(1);
    }
    // 绑定服务注册中心
    server->bindRegistry(registry);
    // 开始监听并处理服务请求
    server->start();
}

int main() {
    go Main;
}
```

rpc 服务消费者，并不直接用RpcClient，而是采用更高级的封装，RpcConnectionPool。 提供了连接池和服务地址缓存。
```c++
#include "acid/log.h"
#include "acid/rpc/rpc_connection_pool.h"

static acid::Logger::ptr g_logger = ACID_LOG_ROOT();

// 连接服务中心，自动服务发现，执行负载均衡决策，同时会缓存发现的结果
void Main() {
    acid::Address::ptr registry = acid::Address::LookupAny("127.0.0.1:8080");

    // 设置连接池的数量
    acid::rpc::RpcConnectionPool::ptr con = std::make_shared<acid::rpc::RpcConnectionPool>();

    // 连接服务中心
    con->connect(registry);

    // 第一种调用接口，以同步的方式异步调用，原理是阻塞读时会在协程池里调度
    acid::rpc::Result<int> sync_call = con->call<int>("add", 123, 321);
    ACID_LOG_INFO(g_logger) << sync_call.getVal();

    // 第二种调用接口，future 形式的异步调用，调用时会立即返回一个future
    std::future<acid::rpc::Result<int>> async_call_future = con->async_call<int>("add", 123, 321);
    ACID_LOG_INFO(g_logger) << async_call_future.get().getVal();

    // 第三种调用接口，异步回调
    con->async_call<int>([](acid::rpc::Result<int> res){
        ACID_LOG_INFO(g_logger) << res.getVal();
    }, "add", 123, 321);
    
    // 测试并发
    int n=0;
    while(n != 1000) {
        n++;
        con->async_call<int>([](acid::rpc::Result<int> res){
            ACID_LOG_INFO(g_logger) << res.getVal();
        }, "add", 0, n);
    }
}

int main() {
    go Main;
}
```


# select,poll,epoll
来源：https://www.cnblogs.com/losophy/p/9359575.html

先说个故事

鬼子进村

处理大量的连接的读写，select 是够低效的。因为 kernel 每次都要对 select 传入的一组 socket 号做轮询，这叫鬼子进村策略。一遍遍的询问“鬼子进村了吗？”，“鬼子进村了吗？”... 大量的 cpu 时间都耗了进去。（更过分的是在 windows 上，还有个万恶的 64 限制。）使用 kqueue 这些，变成了派一些个人去站岗，鬼子来了就可以拿到通知，效率自然高了许多。

## 缓存I/O
缓存I/O又称为标准I/O，大多数文件系统的默认I/O操作都是缓存I/O。在Linux的缓存I/O机制中，操作系统会将I/O的数据缓存在文件系统的页缓存中，即数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间。

 
## 五种IO模型 
**阻塞式I/O模型**：默认情况下，所有套接字都是阻塞的。recvfrom等待数据准备好，从内核向进程复制数据。

**非阻塞式I/O**： 进程把一个套接字设置成非阻塞是在通知内核，当所请求的I/O操作非得把本进程投入睡眠才能完成时，不要把进程投入睡眠，而是返回一个错误，recvfrom总是立即返回。

**I/O多路复用**：虽然I/O多路复用的函数也是阻塞的，但是其与以上两种还是有不同的，I/O多路复用是阻塞在select，epoll这样的系统调用之上，而没有阻塞在真正的I/O系统调用如recvfrom之上。其本质是通过一种机制（系统内核缓冲I/O数据），让单个进程可以监视多个文件描述符，一旦某个描述符就绪（一般是读就绪或写就绪），能够通知程序进行相应的读写操作。select、poll 和 epoll 都是 Linux API 提供的 IO 复用方式。

**信号驱动式I/O**：用的很少，就不做讲解了。

**异步I/O**：这类函数的工作机制是告知内核启动某个操作，并让内核在整个操作（包括将数据从内核拷贝到用户空间）完成后通知我们。

recvfrom函数(经socket接收数据)。

 

再看POSIX对这两个术语的定义：
* 同步I/O操作：导致请求进程阻塞，直到I/O操作完成；
* 异步I/O操作：不导致请求进程阻塞。


## select运行机制
select()的机制中提供一种`fd_set`的数据结构，实际上是一个long类型的数组，每一个数组元素都能与一打开的文件句柄（不管是Socket句柄,还是其他文件或命名管道或设备句柄）建立联系，建立联系的工作由程序员完成，当调用select()时，由内核根据IO状态修改fd_set的内容，由此来通知执行了select()的进程哪一Socket或文件可读。

从流程上来看，使用select函数进行IO请求和同步阻塞模型没有太大的区别，甚至还多了添加监视socket，以及调用select函数的额外操作，效率更差。但是，使用select以后最大的优势是用户可以在一个线程内同时处理多个socket的IO请求。用户可以注册多个socket，然后不断地调用select读取被激活的socket，即可达到在同一个线程内同时处理多个IO请求的目的。而在同步阻塞模型中，必须通过多线程的方式才能达到这个目的。

## poll运行机制
poll的机制与select类似，与select在本质上没有多大差别，管理多个描述符也是进行轮询，根据描述符的状态进行处理，但是poll没有最大文件描述符数量的限制。
 
## epoll运行机制
epoll在Linux2.6内核正式提出，是基于事件驱动的I/O方式，相对于select来说，epoll没有描述符个数限制，使用一个文件描述符管理多个描述符，将用户关心的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的copy只需一次。
epoll是Linux内核为处理大批量文件描述符而作了改进的poll，是Linux下多路复用IO接口select/poll的增强版本，它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率。原因就是获取事件的时候，它无须遍历整个被侦听的描述符集，只要遍历那些被内核IO事件异步唤醒而加入Ready队列的描述符集合就行了。
epoll除了提供select/poll那种IO事件的水平触发（Level Triggered）外，还提供了边缘触发（Edge Triggered），这就使得用户空间程序有可能缓存IO状态，减少epoll_wait/epoll_pwait的调用，提高应用程序效率。

* **水平触发（LT）**：默认工作模式，即当epoll_wait检测到某描述符事件就绪并通知应用程序时，**应用程序可以不立即处理该事件**；下次调用epoll_wait时，会再次通知此事件
* **边缘触发（ET）**： 当epoll_wait检测到某描述符事件就绪并通知应用程序时，应用程序必须立即处理该事件。如果不处理，下次调用epoll_wait时，不会再次通知此事件。（直到你做了某些操作导致该描述符变成未就绪状态了，也就是说边缘触发只在状态由未就绪变为就绪时只通知一次）。
**LT和ET原本应该是用于脉冲信号的**，可能用它来解释更加形象。Level和Edge指的就是触发点，Level为只要处于水平，那么就一直触发，而Edge则为上升沿和下降沿的时候触发。比如：0->1 就是Edge，1->1 就是Level。

ET模式很大程度上减少了epoll事件的触发次数，因此效率比LT模式下高。

## 总结
一张图总结一下select,poll,epoll的区别：
|            | select                                             | poll                                             | epoll                                                                                             |
| ---------- | -------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------------------------------------------------- |
| 操作方式   | 遍历                                               | 遍历                                             | 回调                                                                                              |
| 底层实现   | 数组                                               | 链表                                             | 哈希表                                                                                            |
| IO效率     | 每次调用都进行线性遍历，时间复杂度为O(n)           | 每次调用都进行线性遍历，时间复杂度为O(n)         | 事件通知方式，每当fd就绪，系统注册的回调函数就会被调用，将就绪fd放到readyList里面，时间复杂度O(1) |
| 最大连接数 | 1024（x86）或2048（x64）                           | 无上限                                           | 无上限                                                                                            |
| fd拷贝     | 每次调用select，都需要把fd集合从用户态拷贝到内核态 | 每次调用poll，都需要把fd集合从用户态拷贝到内核态 | 调用epoll_ctl时拷贝进内核并保存，之后每次epoll_wait不拷贝                                         |

epoll是Linux目前大规模网络并发程序开发的首选模型。在绝大多数情况下性能远超select和poll。目前流行的高性能web服务器Nginx正式依赖于epoll提供的高效网络套接字轮询服务。但是，在并发连接不高的情况下，多线程+阻塞I/O方式可能性能更好。
既然select，poll，epoll都是I/O多路复用的具体的实现，之所以现在同时存在，其实他们也是不同历史时期的产物

* select出现是1984年在BSD里面实现的
* 14年之后也就是1997年才实现了poll，其实拖那么久也不是效率问题， 而是那个时代的硬件实在太弱，一台服务器处理1千多个链接简直就是神一样的存在了，select很长段时间已经满足需求
* 2002, 大神 Davide Libenzi 实现了epoll

 参考：

我的网络开发之旅——socket编程 http://blog.51cto.com/yaocoder/1556742

https://blog.csdn.net/dapengbusi/article/details/50690690

IO多路复用的三种机制Select，Poll，Epoll https://www.jianshu.com/p/397449cadc9a

IOCP , kqueue , epoll ... 有多重要？https://blog.codingnow.com/2006/04/iocp_kqueue_epoll.html


# IO设计模式：Actor、Reactor、Proactor 
先介绍两种高性能服务器模型Reactor、Proactor
Reactor模型： 
1 向事件分发器注册事件回调 
2 事件发生 
4 事件分发器调用之前注册的函数 
4 在回调函数中读取数据，对数据进行后续处理 
Reactor模型实例：libevent，Redis、ACE

Proactor模型： 
1 向事件分发器注册事件回调 
2 事件发生 
3 操作系统读取数据，并放入应用缓冲区，然后通知事件分发器 
4 事件分发器调用之前注册的函数 
5 在回调函数中对数据进行后续处理 
Preactor模型实例：ASIO

## reactor和proactor的主要区别：
### 主动和被动
以主动写为例： 
Reactor将handle放到select()，等待可写就绪，然后调用write()写入数据；写完处理后续逻辑； 
Proactor调用aoi_write后立刻返回，由内核负责写操作，写完后调用相应的回调函数处理后续逻辑；

可以看出，Reactor被动的等待指示事件的到来并做出反应；它有一个等待的过程，做什么都要先放入到监听事件集合中等待handler可用时再进行操作； 
Proactor直接调用异步读写操作，调用完后立刻返回；

### 实现
Reactor实现了一个被动的事件分离和分发模型，服务等待请求事件的到来，再通过不受间断的同步处理事件，从而做出反应；

Proactor实现了一个主动的事件分离和分发模型；这种设计允许多个任务并发的执行，从而提高吞吐量；并可执行耗时长的任务（各个任务间互不影响）

### 优点
Reactor实现相对简单，对于耗时短的处理场景处理高效； 
操作系统可以在多个事件源上等待，并且避免了多线程编程相关的性能开销和编程复杂性； 
事件的串行化对应用是透明的，可以顺序的同步执行而不需要加锁； 
事务分离：将与应用无关的多路分解和分配机制和与应用相关的回调函数分离开来，

Proactor性能更高，能够处理耗时长的并发场景；

### 缺点
Reactor处理耗时长的操作会造成事件分发的阻塞，影响到后续事件的处理；

Proactor实现逻辑复杂；依赖操作系统对异步的支持，目前实现了纯异步操作的操作系统少，实现优秀的如windows IOCP，但由于其windows系统用于服务器的局限性，目前应用范围较小；而Unix/Linux系统对纯异步的支持有限，应用事件驱动的主流还是通过select/epoll来实现；

### 适用场景
Reactor：同时接收多个服务请求，并且依次同步的处理它们的事件驱动程序； 
Proactor：异步接收和同时处理多个服务请求的事件驱动程序；


再说Actor模型： 

Actor模型被称为高并发事务的终极解决方案，
实体之通过消息通讯，各自处理自己的数据，能够实现这并行。 
actor模型实例：skynet，Erlang 

Actor模型是一个概念模型，用于处理并发计算。它定义了一系列系统组件应该如何动作和交互的通用规则，最著名的使用这套规则的编程语言是Erlang。

一个Actor指的是一个最基本的计算单元。它能接收一个消息并且基于其执行计算。

这个理念很像面向对象语言，一个对象接收一条消息（方法调用），然后根据接收的消息做事（调用了哪个方法）。

Actors一大重要特征在于actors之间相互隔离，它们并不互相共享内存。这点区别于上述的对象。也就是说，一个actor能维持一个私有的状态，并且这个状态不可能被另一个actor所改变。


思路方向：

**其实无论是使用数据库锁 还是多线程，这里有一个共同思路，就是将数据喂给线程，就如同计算机是一套加工流水线，数据作为原材料投入这个流水线的开始，流水线出来后就是成品，这套模式的前提是数据是被动的，自身不复杂，没有自身业务逻辑要求。适合大数据处理或互联网网站应用等等。**

**但是如果数据自身要求有严格的一致性，也就是事务机制，数据就不能被动被加工，要让数据自己有行为能力保护实现自己的一致性，就像孩子小的时候可以任由爸妈怎么照顾关心都可以，但是如果孩子长大有自己的思想和要求，他就可能不喜欢被爸妈照顾，他要求自己通过行动实现自己的要求。**

数据也是如此。

只有我们改变思路，让数据自己有行为维护自己的一致性，才能真正安全实现真正的事务。

我们可以看到

Actor模型=数据+行为+消息。

Actor模型内部的状态由自己的行为维护，外部线程不能直接调用对象的行为，必须通过消息才能激发行为，这样就保证Actor内部数据只有被自己修改。

 

参考：

reactor和proactor模式（epoll和iocp）https://blog.csdn.net/zccracker/article/details/38686339

为什么Actor模型是高并发事务的终极解决方案？http://www.jdon.com/45728


# 基于 Asio 的 C++ 网络编程
来源：https://segmentfault.com/a/1190000007225464

## 概述
近期学习 Boost Asio，依葫芦画瓢，写了不少例子，对这个「轻量级」的网络库算是有了一定理解。但是秉着理论与实践结合的态度，决定写一篇教程，把脑子里一知半解的东西，试图说清楚。

Asio，即「异步 IO」（Asynchronous Input/Output），本是一个 独立的 C++ 网络程序库，似乎并不为人所知，后来因为被 Boost 相中，才声名鹊起。

从设计上来看，Asio 相似且重度依赖于 Boost，与 thread、bind、smart pointers 等结合时，体验顺滑。从使用上来看，依然是重组合而轻继承，一贯的 C++ 标准库风格。

什么是「异步 IO」？

简单来说，就是你发起一个 IO 操作，却不用等它结束，你可以继续做其他事情，当它结束时，你会得到通知。

当然这种表述是不精确的，操作系统并没有直接提供这样的机制。以 Unix 为例，有五种 IO 模型可用：

阻塞 I/O
非阻塞 I/O
I/O 多路复用（multiplexing）（select 和 poll）
信号驱动 I/O（SIGIO）
异步 I/O（POSIX aio_ 系列函数）
这五种模型的定义和比较，详见「Unix Network Programming, Volume 1: The Sockets Networking API」一书 6.2 节，或者可参考 这篇笔记。https://segmentfault.com/n/1330000004444307

Asio 封装的正是「I/O 多路复用」。具体一点，epoll 之于 Linux，kqueue 之于 Mac 和 BSD。epoll 和 kqueue 比 select 和 poll 更高效。当然在 Windows 上封装的则是 IOCP（完成端口）。

**Asio 的「I/O 操作」，主要还是指「网络 IO」，比如 socket 读写。由于网络传输的特性，「网络 IO」相对比较费时，设计良好的服务器，不可能同步等待一个 IO 操作的结束，这太浪费 CPU 了。

**对于普通的「文件 IO」，操作系统并没有提供“异步”读写机制，libuv 的做法是用线程模拟异步，为网络和文件提供了一致的接口。Asio 并没有这样做，它专注于网络。提供机制而不是策略，这很符合 C++ 哲学。

下面以示例，由浅到深，由简单到复杂，逐一介绍 Asio 的用法。
简单起见，头文件一律省略。


# 想学下C++ 中的Boost.Asio网络库，但看不懂，能指下需要哪些前置知识吗？
如果以简单使用Asio为目标，我认为需要掌握的前置知识有：
1、了解事件驱动网络编程模型（如果在Linux下就是epoll，Windows下是IOCP，看一个就行）；
2、了解C++ 11标准中的函数对象（std::bind、std::function、lambda），用于Asio中的回调函数；
3、了解C++ 11标准中的智能指针，尤其是std::weak_ptr，避免在回调函数中因为变量生命周期失效导致的Bug；
4、如果需要多线程使用Asio，再了解一下事件循环（Event Loop）的基本原理，合理分配和协调不同的线程。当以上知识点都有个大概的概念后，再上手Asio就会简单很多了。

作者：lbblscy
链接：https://www.zhihu.com/question/625940761/answer/3249190846
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。