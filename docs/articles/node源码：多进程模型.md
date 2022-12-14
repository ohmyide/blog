---
title: node源码：多进程模型
date: 2020-07-27 18:33:40
tags: node
---
# 多进程模型

每个 worker 进程通过使用 child_process.fork() 函数，基于 IPC（Inter-Process Communication，进程间通信），实现与 master 进程间通信。

字节跳动**面试官问：Node.js 多进程模型，以及多进程监听同一端口的底层原理是如何实现滴？

上下文保存了切换前的：

- 代码和数据
- 通用目的寄存器的内容、程序计数器、环境变量
- 打开的文件描述符的集合

进程有上下文，线程也有上下文

**线程上下文是进程上下文的子集**

## 线程之下，协程（串行执行）

线程和进程是操作系统的支持带来的优化，而协程本质上是一种**应用层面的优化**了。

协程可以理解为特殊的函数，这个函数可以在某个地方挂起，并且可以重新在挂起处外继续运行

## node多进程模型【请求分发策略】

一开始依然是 master 进程监听 80，当收到用户请求之后，**master 并不是直接把这些数据扔给 worker**，而是在 80 端口接收到数据后，生成对应的 socket，再把该 socket 对应的文件描述符通过管道传给 worker，一个 socket 意味着服务端和客户端的一个数据通道，也就意味着 master 把跟客户端的数据通道传给了 worker。

## 为啥这个时候不会**端口冲突**？？

实质上只有master监听了端口；

All [`net.Socket`](https://nodejs.org/dist/latest-v12.x/docs/api/net.html#net_class_net_socket) are set to `SO_REUSEADDR` (see [`socket(7)`](http://man7.org/linux/man-pages/man7/socket.7.html) for details).

第一个是，【原来所有的`net.Socket`都被设置了`SO_REUSEADDR`】Node 对每个端口监听设置了**SO_REUSEADRR**，表示可以允许这个端口被多个进程监听。

第二个点是，用这个的前提是每个监听这个端口的进程，监听的文件描述符要相同。

## SO_REUSEADDR【实现端口复用】

1.允许多个套接字 `bind()/listen() `同一个`TCP/UDP`端口

2.每一个线程拥有自己的服务器套接字

3.在服务器套接字上没有了锁的竞争

4.内核层面实现负载均衡

5.安全层面，监听同一个端口的套接字只能位于同一个用户下面

《 *Unix* 网络编程》卷一*SO_REUSEADDR* 允许同一 *port* 上启动同一服务器的多个实例 *(* 多个进程 *)* 。但*
*每个实例绑定的 *IP* 地址是不能相同的

## 请求分发策略

主进程创建socket，再把socket的文件描述符通过IPC管道传给worker

子进程TCP服务器没有创建底层socket，如何接受请求和发送响应呢？这就要依赖IPC通道了

RoundRobin轮询选择

## PM2多进程负载均衡

`PM2`的原理实现,其实不过是对cluster模式进行了封装，多了很多功能而已

**总结下来，负载均衡大概流程：**

**1.所有请求先同一经过内部TCP服务器，真正监听端口的只有主进程。**

**2.在内部TCP服务器的请求处理逻辑中，有负载均衡地挑选出一个worker进程，将其发送一个newconn内部消息，随消息发送客户端句柄。**

**3.Worker进程接收到此内部消息，根据客户端句柄创建net.Socket实例，执行具体业务逻辑，返回。**

## PM2如何停止不稳定的进程，保证进程稳定存活？

### 采用心跳检测

每隔数秒向子进程发送心跳包，子进程如果不回复，那么调用kill杀死这个进程
然后再重新cluster.fork()一个新的进程

### 子进程主动异常报错

子进程可以监听到错误事件，这时候可以发送消息给主进程，请求杀死自己
并且主进程此时重新调用cluster.fork一个新的子进程



## backlog

backlog是已连接但未进行accept处理的socket队列大小

node默认在socket层设置backlog默认值为511，这是因为nginx和redis默认设置的backlog值也为此



node中并发编程：https://www.cnblogs.com/accordion/p/12533305.html

两次消费请求流，巧妙复制一个流：https://www.cnblogs.com/accordion/p/9504427.html

https://zhuanlan.zhihu.com/p/37921649

- [当我们谈论 cluster 时我们在谈论什么（上）](https://link.zhihu.com/?target=http%3A//taobaofed.org/blog/2015/11/03/nodejs-cluster/)
- [当我们谈论 cluster 时我们在谈论什么（下）](https://link.zhihu.com/?target=http%3A//taobaofed.org/blog/2015/11/10/nodejs-cluster-2/)
- [cluster文档](https://link.zhihu.com/?target=http%3A//nodejs.cn/api/cluster.html)
- [think-cluster](https://link.zhihu.com/?target=https%3A//github.com/thinkjs/think-cluster)



平滑加权算法https://tenfy.cn/2018/11/12/smooth-weighted-round-robin/