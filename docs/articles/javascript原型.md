---
title: javascript原型
date: 2016-08-14 22:17:27
tags: javascript
---

js原型是这门语言的最为独到之处，理解了原型，也就理解了js的各种继承原理。
<!-- more -->
# 先说说js中的this
js中的this有多种情况，分别代指：节点元素，window，实例对象，object。

## 代表节点元素的情况：
``` javascript
var box = document.getElementById("box");
box.onclick = function(){
    console.log(this.innerHTML); // box
}
```
## 代表window的情况：
在全局下直接输出this就是window：
``` javascript
console.log(this);  // window
```
普通调用函数（全局函数）中的this也会指向window：
``` javascript
var foo = function(){
    this.name = "tom";
    this.age = 20;
}
foo();
window.name   // tom
```
但有种情况，当也属于普通调用函数（函数中的函数）：
``` javascript 
function foo(){
    function bar(){
        console.log("this:",this);
    }
    bar();
}
foo(); // window
```
上述代码中，虽然bar在foo内部，但调用时this依然指向window而不是foo。

## 代表实例对象的情况：
``` javascript
var foo = function(){
    this.name = "tom";
    this.age = 20;
}
foo.prototype.sayAge = function(){
    console.log(this.age); // prototype中的this也指向实例
}
var a = new foo();
console.log(a.name); // this指向实例a
a.sayAge(); prototype中的this也指向实例a
```
## 代表object的情况：
``` javascript
var obj = {
    text: "hello",
    foo: function(){
        console.log(this.text); // this代表obj
    }
}
obj.foo();  //hello
```
## 特殊情况：被apply，call篡改的this
``` javascript
window.name = "tom"
function foo(){
    console.log(this.name);
}

var obj = {
    name: "jack",
    fn: foo,
    bar: function(){
        console.log(this.name);
    }
}

foo(); // tom  (this指向window)
foo.apply(obj); // jack  (this被改成指向obj)
```

## 总结：正常情况下this都指向被调用者，但要注意普通函数正常调用依然指向window。除此之外，apply和call可串改this的指向。


