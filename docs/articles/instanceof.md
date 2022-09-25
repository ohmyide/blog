---
title: instanceof
date: 2019-04-28 16:42:47
tags: js
---

instanceof用来判断对象是否属于某种类型，比如：判断一个示例是否是某个对象的实现，也就是改示例是否属于该对象类型。

其原理呢，用另一个定义即可说明：instanceof运算符用于测试构造函数的prototype属性是否出现在对象的原型链中的任何位置。

由此可得：一个对象instanceof于各个层级的__proto__

```
function Person(){
    this.method1 = function(){}
}
Person.prototype.method2 = function(){}
 
function Man(){
    this.name = 'tom';
}
Man.prototype = new Person();
 
Man.prototype.m1 = function(){}
Man.prototype.m2 = function(){}
 
var m = new Man();


console.log('instace',m instanceof Man); // true
console.log('instace',m instanceof Person); // true
console.log('instace',m instanceof Object); // true

// 还有以下都为true
console.log('Person instanceof Function:',Person instanceof Function);
console.log('Person instanceof Object:',Person instanceof Object);
console.log('Function instanceof Object:',Function instanceof Object);
```

下面是递归实现：

```
function _instanceof(left,right){
    if(left === null){
        return false;
    }
    let leftProto = left.__proto__;
    let rightProto = right.prototype;

    if(leftProto === rightProto){
        return true;
    }
    console.log('222');
    return _instanceof(leftProto,right);
}

console.log(_instanceof(m,Man));
console.log(_instanceof(m,Person));
console.log(_instanceof(m,Object));
console.log(_instanceof(Person,Function));
console.log(_instanceof(Function,Object));
```

实现2：

```
function _instanceof(leftVaule, rightVaule) { 
    let rightProto = rightVaule.prototype; // 取右表达式的 prototype 值
    leftVaule = leftVaule.__proto__; // 取左表达式的__proto__值
    while (true) {
    	if (leftVaule === null) {
            return false;	
        }
        if (leftVaule === rightProto) {
            return true;	
        } 
        leftVaule = leftVaule.__proto__ 
    }
}
```

题外话：有些属性在对象上，有些属性在对象的原型链上，有些属性在原型链的原型链上，有且仅有hasOwnProperty能够判断是否直接在对象属性上，因此for in遍历对象时要按需选择是否要判断hasOwnProperty。

当对象的一个属性是在其原型链上时，hasOwnProperty肯定为false，但取其原型链再hasOwnProperty则为true，以此类推后续原型链。

此外：
```
m.__proto__ === Man.prototype === Object.getPrototypeOf(m)
```