---
title: node源码：Timer
date: 2020-07-27 18:33:40
tags: node
---
# Timer

### 使用场景

定时器主要的使用场景或者说适用场景：

- 定时任务，比如业务中定时检查状态等；
- 超时控制，比如网络超时控制重传。

为啥在 HTTP 模块要用呢？

KeepAlive模式时：连接不能就这么一直保持着，所以一般都会有一个超时时间，超过这个时间客户端还没有发送新的http请求，那么服务器就需要自动断开从而继续为其他客户端提供服务。 Node.js的HTTP 模块对于每一个新的连接创建一个 socket 对象，调用socket.setTimeout设置一个定时器用于超时后自动断开连接。



主要分为 javascript 层面的实现和 `libuv` 层面的实现, 而timer_wrap.cc 作为一个bridge,完成 javascript 和 C++的交互调用。

对象复用

- 相同超时时间的定时器共享一个底层的 C的 timer。