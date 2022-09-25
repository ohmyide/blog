---
title: Promise
date: 2018-11-26 01:52:02
tags:
---
越是高级语言，高封装度的框架，使用起来代码越倾向于配置，就像css，几乎就是一门纯配置语言。
Promise一直在用，基本都是在写Promise的配置，现在想想还是研究一下内部。
<!-- more -->
Promise主要解决嵌套异步回调问题，退一步想，只是代码形式上的“重组”，而promise就是重组嵌套代码的盒子，这个盒子规定：把触发逻辑写在resolve里，回调都写在then里，这样的代码形式则变得优雅很多。

回调地狱：
```
fs.readFile('./test.txt','utf8',(err,data)=>{
    fs.readFile(data,'utf8',(err,data)=>{
        fs.readFile(data,'utf8',(err,data)=>{
            console.log(data);
        })
    })
})

```
Promis后：
```
function read(filePath,encoding){
    return new Promise((resolve,reject)=>{
        fs.readFile(filePath,encoding,(err,data)=>{
            if(err) {
            	reject(err)
            }
            resolve()
        })
    })
}

read('./test.txt','utf8')
.then((data)=>{
    return read(data,'utf8')
})
.then((data)=>{
    return read(data,'utf8')
})
.then((data)=>{
    return data.split('').reverse().join('')
})
.then((data)=>{
    console.log(data)
})
.then((data)=>{
    console.log(data)
})...
```
当然，应用广泛的Promise是有规范的，可以根据规范自己实现。规范也是人定的，甚至还可以抛弃规范去写自己心目中的代码。

如上面所说，Promise是代码形式上的“重组”工具，把触发代码和回调代码，以利于阅读和维护的方式重组，其中触发代码为：
```
fs.readFile(filePath,encoding,(err,data)=>{
    if(err) {
    	reject(err)
    }
    resolve()
})
```
回调代码为：
```
.then((data)=>{
    console.log(data)
})
```

这样理解起来，对Promise整体就有了全局的认识，接下来要实现的就是这个工具，这个工具的核心是：
- 1.触发函数在新建Promise时先执行，并根据结果设定好是走resolve还是reject
- 2.then里的回调函数在触发后执行，这就意味着要先注册这些回调（resolve回调和reject的回调）
- 3.Promise要能链式调用，且每个链式产生的是一个全新的Promise（非jQuery中直接return this，而是return new Promise）

###最小Promise
先实现一个最小功能的Promise，仅实现上述1，2功能：
```
function Promisee(fn){
    this.state = 'PENDING';
    this.value = null;

    resolve = (value) => {
        this.state = 'RESOLVED'
        this.value = value;
    };
    reject = (error) => {
        this.state = 'REJECTED'
        this.value = error;
    };
    fn(resolve,reject)
}
Promisee.prototype.then = function(onResole,onReject){
        if(this.state === 'RESOLVED'){
            onResole(this.value)
        }
        else if(this.state === 'REJECTED'){
            onReject(this.value)
        }
    };
```
触发逻辑：
```
let t = new Promisee((resolve,reject) => {
    let a = 1;
    resolve(a);
});

t.then((value)=>{
    console.log('执行成功：',value);
},(value) => {
    console.log('执行失败：',value);
});
```
上述就是最简易的Promise，上来就先执行了resolve，然后执行了then中onResole的回调。

但在实际中Promise的触发逻辑都是异步的，比如发起请求，这里用setTimeout模拟：
```
let t = new Promisee((resolve,reject) => {
    let a = 1;
    setTimeout(() => {
        resolve(a);
    }, 1000);
});
```
这里因为触发函数是异步的，所以then优先于resolve执行，而此时的状态还是最初的PEDDING，为此，Promise要继续丰富这种情况，保证then里的代码优先注册，必须当resolve执行后，才执行then中注册的回调：
```
function Promisee(fn){
    this.state = 'PENDING';
    this.value = null;
    this.onResolvedCallback = [];
    this.onRejectedCallback = [];

    resolve = (value) => {
        this.state = 'RESOLVED'
        this.value = value;
        console.log(this.onResolvedCallback.length);
        for(var i = 0; i < this.onResolvedCallback.length; i++) {
            this.onResolvedCallback[i](value)
        }
    };
    reject = (error) => {
        this.state = 'REJECTED'
        this.value = error;
        for(var i = 0; i < this.onRejectedCallback.length; i++) {
            this.onRejectedCallback[i](error)
        }
    };
    fn(resolve,reject)
}
Promisee.prototype.then = function(onResole,onReject){
        if(this.state === 'RESOLVED'){
            onResole(this.value)
        }
        else if(this.state === 'REJECTED'){
            onReject(this.value)
        }
        else if(this.state === 'PENDING'){
            this.onResolvedCallback.push(onResole);
            this.onRejectedCallback.push(onReject);
        }
    };
```
仔细对比改造前后的代码，会发现有以下核心改动点：
- then里当PENDING状态时分别存到onResolvedCallback和onRejectedCallback数组里，解决了上述中因异步触发函数导致缺失PENDING状态的问题
- resolve或reject执行时，遍历各自的数组栈，把之前then中存进去的回调函数通通在此时执行

这时候的触发函数终于可以写成常见的异步了。

下面只剩下一个问题3：then链式调用，then返回的是一个全新的Promise，需在then的三种情况下都返回一个new Promise，且新promise的输入是上一个promise的输出，也就是上一个promise中then里的回调函数执行结果。