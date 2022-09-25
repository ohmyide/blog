---
title: 'js:变量解析'
date: 2019-02-19 00:17:58
tags: js
---
js在执行前会创建可执行上下文，执行上下文中中包含：
- 变量对象
- 作用域链
- this

执行上下文中的变量我们称为：活动对象(activation object, AO)。

**js对执行上下文中的代码有两部处理：1.进入执行上下文，进行各自的声明；2.执行，对声明赋值**
进入阶段，其中变量对象的状态为：
- arguments即形参，被赋值的形参为key:value，未赋值的为key:undefined
- 函数声明，注意：函数的声明一定提前于变量
- 变量声明，只有声明变量占位，并没有值

demo:

```
function test(item1,item2){
    console.log(item1,item2);  // 1,undefined
    console.log(a); // undefined
    console.log(c); // c(){}
    var a = 3;
    function c(){}
}
test(1);
```

进入阶段的AO是：
```
AO = {
    arguments: {
        0: 1,
        1: undefined,
        length: 2
    },
    a: undefined,
    c: reference to function c(){}
}

```
之所以能正常打印a，是因为执行打印时，函数已经提前【扫描】了内部变量，arguments作为实参，有实参则有值，无实参则undefined，函数提前，变量声明提前，只是没赋值，此时为undefined，执行到后续的的复制命令时a才会为3.

改一下，把内部的var a = 3;去除var，变为a = 3 时：

```
function test(item1,item2){
    console.log(item1,item2);  // 1,undefined
    console.log(a); // 此处会报错：ReferenceError: a is not defined
    console.log(c); 
    a = 3;
    function c(){}
}
test(1);
```
因为去除var后，没有变量声明，在进入时，上下文中根本不会声明变量a，在执行阶段发现本作用域中没有a，则往上找，依然没有，则报错。
在进入阶段的AO为：

```
AO = {
    arguments: {
        0: 1,
        1: undefined,
        length: 2
    },
    // 根本没有a，也不存在undefined,

    c: reference to function c(){}
}
```

注意：
- 函数优于变量提前声明
- 变量与形参重名时，变量声明不影响形参，如下demo：

```
function test(item1,item2){
    console.log(item1,item2); // 1, undefined
    console.log(c); // c(){}  而不是undefined，因为函数提前
    var c = 3;
    function c(){}
    
    var item1;
    console.log(item1); // 依然是1，没赋值，不会影响形参item1
}
test(1);
```

### 与静态作用域的关系
上文中说过js的作用域是静态的，函数无论在哪里执行，其内部变量的作用域始终为创建函数是固定的，通过观察AO发现果然如此，在函数执行前去，创建作用域栈，解析当前作用域下的变量，产生了固定的AO。
