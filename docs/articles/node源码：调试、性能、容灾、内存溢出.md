---
title: node源码：调试、性能、容灾、内存溢出
date: 2020-07-27 18:33:40
tags: node
---

## 内存泄露leak原因

1. ### 1.闭包引用导致的泄漏


这段代码已经在很多讲解内存泄漏的地方引用了，非常经典，所以拿出来作为第一个例子，以下是泄漏代码：

1. ```js
   const express = require('express');
   const app = express();
   
   //以下是产生泄漏的代码
   let theThing = null;
   let replaceThing = function () {
       let leak = theThing;
       let unused = function () {
           if (leak)
               console.log("hi")
       };
       
       // 不断修改theThing的引用
       theThing = {
           longStr: new Array(1000000),
           someMethod: function () {
               console.log('a');
           }
       };
   };
   
   app.get('/leak', function closureLeak(req, res, next) {
       replaceThing();
       res.send('Hello Node');
   });
   
   app.listen(8082);
   ```


### 2.原生Socket重连策略不恰当导致的泄漏

这种类型的泄漏本质上node中的events模块里的侦听器泄漏，因为比较隐蔽，所以放在第二个例子，以下是泄漏代码：

```js
const net = require('net');
let client = new net.Socket();

function connect() {
    client.connect(26665, '127.0.0.1', function callbackListener() {
    console.log('connected!');
});
}

//第一次连接
connect();

client.on('error', function (error) {
    // console.error(error.message);
});

client.on('close', function () {
    //console.error('closed!');
    //泄漏代码
    client.destroy();
    setTimeout(connect, 1);
});
```

例子里面的泄漏属于非常隐蔽的一种：`net` 模块的重连每一次都会给 `client` 增加一个 `connect事件` 的侦听器，如果一直重连不上，侦听器会无限增加，从而导致泄漏。

### 3.不恰当的全局缓存导致的泄漏

如果我们在项目中不恰当的使用了全局缓存：主要是指只有增加缓存的操作而没有清除的操作，那么就会引起泄漏。

```js
const easyMonitor = require('easy-monitor');
const express = require('express');
const app = express();

const _cached = [];

app.get('/arr', function arrayLeak(req, res, next) {
	//泄漏代码
    _cached.push(new Array(1000000));
    res.send('Hello World');
});

app.listen(8082);
```

### 4.全局变量：把变量放在了global上

```javascript
a = 10;
//未声明对象。

global.b = 11;
//全局变量引用
```

### 5.事件监听

Node.js 的事件监听也可能出现的内存泄漏。例如对同一个事件重复监听，忘记移除（removeListener），将造成内存泄漏。这种情况很容易在复用对象上添加事件时出现，所以事件重复监听可能收到如下警告：

> (node:2752) Warning: Possible EventEmitter memory leak detected。11 haha listeners added。Use emitter。setMaxListeners() to increase limit

例如，Node.js 中 Agent 的 keepAlive 为 true 时，可能造成的内存泄漏。当 Agent keepAlive 为 true 的时候，将会复用之前使用过的 socket，如果在 socket 上添加事件监听，忘记清除的话，因为 socket 的复用，将导致**事件重复监听从而产生内存泄漏**。

原理上与前一个添加事件监听的时候忘了清除是一样的。在使用 Node.js 的 http 模块时，不通过 keepAlive 复用是没有问题的，复用了以后就会可能产生内存泄漏。所以，你需要了解添加事件监听的对象的生命周期，并注意自行移除。

关于这个问题的实例，可以看 Github 上的 issues（[node Agent keepAlive 内存泄漏](http://link.zhihu.com/?target=https%3A//github.com/nodejs/node/issues/9268)）

## 内存泄露排查工具

- heapdump
- v8-profiler

这两个工具的原理都是一致的：调用v8引擎暴露的接口,然后将获取的c++对象数据转换为js对象。

## Easy-Monitor工具自动定位疑似泄漏点



##如何避免内存泄漏

文中的例子基本都可以很清楚的看出内存泄漏，但是在工作中，代码混合上业务以后就不一定能很清楚的看出内存泄漏了，还是得依靠工具来定位内存泄漏。另外下面是一些避免内存泄漏的方法。

1. ESLint 检测代码检查非期望的全局变量。
2. 使用闭包的时候，得知道闭包了什么对象，还有引用闭包的对象何时清除闭包。最好可以避免写出复杂的闭包，因为复杂的闭包引起的内存泄漏，如果没有打印内存快照的话，是很难看出来的。
3. 绑定事件的时候，一定得在恰当的时候清除事件。在编写一个类的时候，推荐使用 init 函数对类的事件监听进行绑定和资源申请，然后 destroy 函数对事件和占用资源进行释放。
4. 不把变量放在global上