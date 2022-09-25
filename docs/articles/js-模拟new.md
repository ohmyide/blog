---
title: 'js:模拟new'
date: 2019-02-28 13:17:11
tags: js
---

作为关键字new，也是可以模拟的，模拟之前，先看new的基本用法和作用：

```
function bar(name,age) {
    this.name = name;
    this.age = age;
    this.habit = 'game';
}
bar.prototype.strength = 60;
bar.prototype.sayName = function(){
    console.log(this.name);
}
let a = new bar('tom',10);
console.log(a);
console.log(a.name);
console.log(a.age);
console.log(a.strength);
a.sayName();
```
实例a可以访问构造函数中的变量，也可以访问其原型prototype上的变量和方法，下面是实现，命名为objectFactory：

```
objectFactory(fn,a){
    var obj = new Object();
    var Constructor = [].shift.call(arguments);
    obj.__proto__ = Constructor.prototype;
    Constructor.apply(obj, arguments);

    return obj;
}
```
上面这段代码，信息密度大，及其精妙。

补充用构造函数new对象的特例：构造函数返回是一个对象时直接返回对象，则返回对象(如果返回常规类型，则忽略return)：

```
function test(a){
    this.name = '111';
    return {
        sayName:() =>{
            console.log(this.name);
        }
    };
}

let a = new test();
console.log(a);
a.sayName() // 111
```

所以最终实现：

```
function objectFactory(){
    var obj = new Object(),
    // 截取fn
    fn = [].shift.call(arguments);
    obj.__proto__ = fn.prototype;
    // 截取后续参数
    var ret = fn.apply(obj, arguments);

    return typeof ret === 'object' ? ret : obj;
}
```
