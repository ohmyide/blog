---
title: 'js:按值传递'
date: 2019-02-16 15:29:10
tags: js
---
js是按值传递的，这个值对原始数据类型来说是value，对引用型数据来说是引用（即地址）；
<!-- more -->
demo1：
```
function f(a,b,c) {
   a = 3;
   b.push("foo");
   c.first = false;
}

var x = 4;
var y = ["eeny", "miny", "mo"];
var z = {first: true};
f(x,y,z); // x:3 y:["eeny", "miny", "mo", 'foo']  z: {first: false}
```
x 是原始数据，y、z是引用数据，传入函数内的是x的值3，y、z的引用地址，所以内部通过地址引用操作内存【堆】中的引用型数据。

### 共享传递 ？
demo2：
```
var obj = {
    value: 1
};
function foo(o) {
    o = 2;
    console.log(o); //2
}
foo(obj);
console.log(obj.value) // 1
```
上述例子中obj传入函数中的是引用地址的拷贝，被命令为o，当这个地址被重新复制后，切断了原有向堆中的指向，被新赋值为原始数据2，而堆中的数据不变，且依然被obj变量指向着。 

这也被称为共享传递，但因为共享传递中传递的是地址，地址被一个变量代表着，所以在《js高程》中称js为值传递。

demo2：

```
let a = {a:1};
a.x = a = {a:2};
console.log(a); // {a:2}
console.log(a.x); // undefined  ??
```
首先复制运算从右向左，且.符号优先级最大，a为{a:1}的引用地址（是个变量值），真正的引用数据有两个分别是：{a:1}和{a:2}，那么赋值运算可拆解为：

a.x = a = {a:2}; （新内存：{a:2}）赋值给 ==> (引用地址a) 赋值给==> （原内存a：{a:1}）.x

所以原内位置的数据被修改为：{a:1,x:{a:2}}
变量a指向的是新创建的内存：{a:2}
所以：a是{a:2} a.x是undefined
而老内存{a:1,x:{a:2}} 因为已经没有被任何变量引用，而被销毁。

为了验证老内存，防止被销毁，新建一个变量b来指向它：
```
let a = {a:1};
let b = a;
a.x = a = {a:2};
console.log(a); // {a:2}
console.log(a.x); // undefined 
console.log(b); // {a:1,x:{a:2}}
```
变量b始终指向老内存{a:1}，且老内存被追加属性x后，不会被销毁。

