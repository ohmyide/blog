---
title: node源码：FS文件系统
date: 2020-07-27 18:33:40
tags: node
---
# FS文件系统

文件系统：是对磁盘设备的抽象，屏蔽了具体的存储类型，比如磁盘，内存。

## 文件系统有好多，如ext2、ext3 、ext4 ，那么操作系统是如何兼容这些不同的文件系统？

统一抽象组件，叫做虚拟文件系统(Virtual File System ,VFS),VFS使用得用户可以直接调用write、read 这样的文件系统调用，而不用考虑底层是什么文件系统。

### 文件主要操作

- open
- read
- write
- close

## 文件IO：异步非阻塞IO

本质解决：慢速的 IO 设备和高速的 CPU；

## require() 是同步加载文件的

CommonJS 社区对此就有很多争议，导致了坚持异步的 AMD 从 CommonJS 中分裂出来。

- CommonJS 模块是同步加载和同步执行
- AMD 模块是异步加载和异步执行
- CMD（Sea.js）模块是异步加载和同步执行。【要扫描代码提取依赖】
- ES6 的模块体系最后选择的是异步加载和同步执行。也就是 Sea.js 的行为是最接近 ES6 模块的。

Node.js 的模块是来自于本地文件系统，最后 Node.js 选择了看上去更简单的 CommonJS 模块规范，直到今天。