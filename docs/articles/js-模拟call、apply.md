---
title: 'js:模拟call、apply'
date: 2019-02-26 23:08:36
tags: js
---
call的用法：借窝下蛋，使用一个指定的 this 值和若干个指定的参数值的前提下调用某个函数或方法。

看个应用场景：

```
var value = 456;
var obj = {
    value: 11,
    test: function(){
        console.log(123);
    }
};

function test(){
    console.log(this.value);
}
test();
test.call(obj);
```

如果不用call，我们的代码不得不这样，把test放到obj里才行：

```
var foo = {
    value: 1,
    // 需要把函数放在obj里才行
    bar: function() {
        console.log(this.value)
    }
};

foo.bar(); // 1
```

demo中通过调用call，让test中this的指向改为obj。

### 模拟call
原生的call方法是挂载在Function.prototype上的，下面就模拟实现一个call2：
模拟原理：
- 先把函数放在obj里，获得了想要的this指向
- 执行obj.函数
- 删除obj.函数，保证obj原状不变

```
Function.prototype.call2 = function(obj){
    obj.fn = this; // 重点：this指向的实例，Function的实例就是函数bar
    obj.fn();
    delete fn;
}

// 测试一下
var foo = {
    value: 1
};

function bar() {
    console.log(this.value);
}
bar.call2(foo); // 1
```

到这原理就讲完了，但call是能带参数的，且还有类似固定参数的apply呢
实现带参call2：

```
Function.prototype.call2 = function(obj,a,b){
    obj.fn = this;
    obj.fn(a,b);
    delete fn;
}

// 测试一下
var foo = {
    value: 1
};

function bar(a,b) {
    console.log(this.value,a,b);
}
bar.call2(foo,1,2); // 1,1,2
```

但现实中call的参数是不固定的，第第一位始终是要嫁接的obj，所以参数只能从arguments数组的第1为开始取：

```
Function.prototype.call2 = function(context) {
    context.fn = this;
    var args = [];
    for(var i = 1, len = arguments.length; i < len; i++) {
        args.push('arguments[' + i + ']');
    }
    eval('context.fn(' + args +')'); // 重点：用eval拼接出参数（a,b,c）的效果，args会自动调用toString方法
    delete context.fn;
}

// 测试一下
var foo = {
    value: 1
};

function bar(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.value);
}

bar.call2(foo, 'kevin', 18); 
// kevin
// 18
// 1
```

有些函数需要返回：比如函数结果是return一个什么东西，比如return 一个obj对象之类:
还有 call可以传null的，传null意味着在全局window上执行：

```
function bar(name, age) {
    return {
        value: this.value,
        name: name,
        age: age
    }
}
bar() // return出一个对象
```

这里也是可以轻松实现的，原理是把fn执行的结果收集起来，再用return扔出去：

```
Function.prototype.call2 = function(context) {
    var context = context || window; // 为null时，context为window
    context.fn = this;
    var args = [];
    for(var i = 1, len = arguments.length; i < len; i++) {
        args.push('arguments[' + i + ']');
    }
    let result = eval('context.fn(' + args +')'); // 重点：用result手机函数运行结果，并return出去
    delete context.fn;
    return result; // return 出去
}

// 测试一下
var foo = {
    value: 1
};

function bar(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.value);
}

bar.call2(foo, 'kevin', 18); 
// kevin
// 18
// 1
```

### apply的模拟实现
apply的唯一区别就是参数固定，所以第二个参数直接一个数组：

```
Function.prototype.apply = function (context, arr) {
    var context = Object(context) || window;
    context.fn = this;

    var result;
    if (!arr) {
        result = context.fn();
    }
    else {
        var args = [];
        for (var i = 0, len = arr.length; i < len; i++) {
            args.push('arr[' + i + ']');
        }
        result = eval('context.fn(' + args + ')')
    }

    delete context.fn
    return result;
}
```


### 模拟bind
先看看bind的特性：
- 依然是借窝下蛋的功能
- 但返回一个新的函数
- 函数可传参，且参数可以再bind的时候产一部分，在执行的时候再传一部分

最简单实现：

```
Function.prototype.bind2 = function(obj){
    var self = this;
    return function () {
        return self.apply(obj);
    }
}

// 测试一下
var foo = {
    value: 1
};

function bar() {
    console.log(this.value);
}
let a = bar.bind2(foo);
console.log('a:',a);
a() // 1

```

实现和原理见：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind