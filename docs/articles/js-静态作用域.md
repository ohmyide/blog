---
title: 'js:静态作用域'
date: 2019-02-18 13:03:42
tags: js
---

作用域指代码中定义变量的区域，js是静态作用于，核心点在于：无论function在哪里执行，function中使用的变量，始终为定义function时的变量。
<!-- more -->
demo：
```
var value = 1;

function foo() {
    console.log(value);
}

function bar() {
    var value = 2;
    foo();
}

bar(); // 1
```

无论foo在哪里执行，其寻找变量的始终为创建时全局的value，而非bar中定义的value。如果js是动态作用域那打印的肯定是2了。

换个复杂点的：

```
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope();
```

以及：

```
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}
checkscope()();
```
毫无疑问：都是local scope，无论f在哪执行，哪怕被return出去，其中变量作用域始终为创建时的“local scope”。

### 执行上下文栈

上述demo中两个checkscope的实现方式虽然结果相同，但执行上下文栈是不同的：
当js执行一个函数的时候，会创建一个执行上下文压栈，当有n个执行任务的时候，通过执行上下文栈来管理执行顺序，然后挨个执行出栈，最后的栈低永远是全局上下文globalContext。
demo：
```
function fun3() {
    console.log('fun3')
}

function fun2() {
    fun3();
}

function fun1() {
    fun2();
}

fun1();
```

所以执行上下文栈为：
```
//先执行fn1
ECStack.push(fn1)

// 发现fn1里调用了fn2，再次创建一个执行上下文入栈
ECStack.push(fn2)

// 发现fn2里调用了fn3，再次创建一个执行上下文入栈
ECStack.push(fn3)

// 终于fn3得到执行，pop出栈
ECStack.pop();
// fun2执行完毕
ECStack.pop();

// fun1执行完毕
ECStack.pop();

// 最后ECStack中剩下globalContext
```

那上述checkscope的demo中两个虽结果一样，但执行上线文栈为:

```
ECStack.push(checkscope)

//发现里面调用了f()
ECStack.push(f)

// 终于f得到执行，pop出栈
ECStack.pop();
// checkscope执行完毕
ECStack.pop();

```

第二个checkscope()();的例子为:

```
ECStack.push(checkscope)

//checkscope()运行结束了，pop
ECStack.pop();

// 发现checkscope()后面又有个()，也就是还有执行任务f，那再push
ECStack.push(f)
// f执行完毕
ECStack.pop();
```
完。