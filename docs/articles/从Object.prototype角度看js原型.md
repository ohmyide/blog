---
title: 从Object.prototype角度看js原型
date: 2017-03-08 16:16:46
tags: javascript
desc: javascript原型 原型链
---
js原型是该语言的核心本质，可以说，没理解js原型就没学会js，js中的各个对象通过原型链的层层引用实现了各种方法的继承。其中关系错综复杂，我们先从这些原型链的『根』开始：
<!-- more -->
## Object.prototype
为什么说`Object.prototype`是原型链的根？我们先把它打印出来开始：
![Object.prototype](http://7xprui.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-03-08%20%E4%B8%8B%E5%8D%887.41.38.png)
可见，其原型上有几个熟悉的方法：
- hasOwnProperty
- isPrototypeOf
- toLocaleString
- toString
- valueOf

这里方法的功能不是本文重点，这里不做介绍。只要记着Object的原型上有这几个方法就好,且**留意hasOwnProperty函数，具体原因后续说明**此时我们回忆下js的引用类型有以下几种：

- Object
- Function
- Number
- RegExp
- Date
- Boolean

我们知道这些引用型的实例就是我们常用的对象，函数，数值，正则，日期和布尔型数据，也就是说：
``` javascript
var arr = [];
// 相当于
var arr = new Array();

var o = {};
// 相当于
var o = new Object();

function fn(){};
// 相当于
var fn = new Function();

var d = 2;
// 相当于
var d = new Number(2);
```
首先，我们应该都知道new出的实例会继承父对象中prototype中的方法，比如我们打印Array.prototype：
![Array.prototype](http://7xprui.com1.z0.glb.clouddn.com/QQ20170308-194513.png)
我们看到，数组常用的方法都能在里面找到，所以new出来的数组实例arr自然拥有上述方法。但有个**奇怪的函数hasOwnProperty**：
``` javascript
console.dir(arr.hasOwnProperty)
```
![arr.hasOwnProperty](http://7xprui.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-03-08%20%E4%B8%8B%E5%8D%887.41.59.png)
Array的原型中，也就是Array.prototype中并没有hasOwnProperty。
## arr.hasOwnProperty从哪里来？
答案就是从js的『根』中来，也就是来自Object.prototype。
``` javascript
console.log(arr.hasOwnProperty === Object.prototype.hasOwnProperty) //true
```
下面这句特别重要：
实例arr的属性方法来自实例化它的Array的原型（Array.prototype）,那Array.prototype中的方法又来自哪里？答案是：一部分是js根据Array类型特意内置的，比如说sort方法，其余类型就没有，还有一部分就是来自根：`Object.prototype`,通过神秘的`__proto__`来实现引用：
``` javascript
console.log(arr.hasOwnProperty === Object.prototype.hasOwnProperty) //true
console.log(Array.prototype.hasOwnProperty === Object.prototype.hasOwnProperty) //true
```
而`__proto__`指向实例化它的对象的原型，也就是：实例化它的对象的prototype：
``` javascript
console.log(arr.__proto__ === Array.prototype) //true
```
那Array.prototype对象（原型也是对象）是谁实例化的呢？上面已经揭晓，既然Array.prototype中引用了Object.prototype中的方法，自然：
console.log(Array.prototype.__proto__ === Object.prototype) //true
到这里我们几乎可以这么理解：arr的根是Array.prototype，而Array.prototype的根，也就是js的根：`Object.prototype`,`Object.prototype`上的方法自然能被层层下属使用，hasOwnProperty就是最好的说明。
除Array外，其余也是如此：
``` javascript 
console.log(Function.prototype.__proto__ === Object.prototype) //true
console.log(Boolean.prototype.__proto__ === Object.prototype) //true
console.log(String.prototype.__proto__ === Object.prototype) //true
// .......
```
> 为什么Object.prototype中有这么多方法，本文只拿hasOwnProperty来做说明，答案是因为：其余方法如toString，valueOf会根据数据类型情况而被重写，比如Date型和Boolean型的toString和valueOf就是被重写改造后的，并没用使用根中自带的，这里就不举例了。

## 验证一下
既然Object.prototype是根，且里面的方法可以层层被继承使用，那么这里就验证一下：
我们在Object.prototype上新增一个方法test：
``` javascript
Object.prototype.test = function(){
    console.log("这是来自js根中的方法！")
}

//创建不同数据类型的实例:
var arr = [];
var b = true;
var d = Date();
// ...

arr.test(); //这是来自js根的测试方法！
b.test(); //这是来自js根的测试方法！
d.test(); //这是来自js根的测试方法！

// ...毫无疑问所有的实例都拥有了test方法
```
上述大范围从根上扩充只是测试，一般情况下我们针对某一类型进行扩充足矣，比如js的trim方法只去两头空格，不包括中间空格，那么我们只需针对String类型在其原型上创建去全部空格方法allTrim即可：
``` javascript 
String.prototype.allTrim = function(){
    return this.replace(/\s+/g,"");
}
"   aa  bb  cc   ".allTrim(); //aabbcc
```
这样所有的字符串实例都有了allTrim方法，理解了js的原型，则理解了js的继承机制。
## 一切皆空
既然Array、Function、Number的原型对象都通过`__proto__`引用自根`Object.prototype`，那么`Object.prototype`也是一个原型对象，它的根又来自哪里呢？答案是`null`。
``` javascript
console.log(Object.prototype.__proto__) // null
```
null 才是js原型链的终结。