---
title: jQuery整体架构
date: 2016-08-31 12:29:04
tags: jQuery
---
老文重发。
现在的前端，明显不再是jQuery的巅峰期了，之前一直在用，但还未仔细研究过其源码，现在回头看看jQuery的内部构造。
<!-- more -->
首先看看jQuery的整体组成部分：
``` javascript
;(function(global, factory) {
    factory(global);
}(typeof window !== "undefined" ? window : this, function(window, noGlobal) {
    var jQuery = function( selector, context ) {
        return new jQuery.fn.init( selector, context );
    };
    jQuery.fn = jQuery.prototype = {};
    // 核心方法
    // 回调系统
    // 异步队列
    // 数据缓存
    // 队列操作
    // 选择器引
    // 属性操作
    // 节点遍历
    // 文档处理
    // 样式操作
    // 属性操作
    // 事件体系
    // AJAX交互
    // 动画引擎
    return jQuery;
}));

```
jQuery的实现原理,demo可缩略为：
``` javascript
var $ = jQuery = function(){
    return new jQuery.prototype.init();
}
jQuery.fn = jQuery.prototype = {
    init: function(){
       // console.log(this);
        //this.name = "new name"
        return this;
    },
    name: "my name is jquery",
    sayName: function(){
        console.log(this);
        console.log(this.name);
    },
    sayAge:function(){
        this.age = 300;
        console.log(this.age);
        console.log(this);
    },
    sayNum:function(num){
        this.num = num;
        console.log(this.num);
    }
}
jQuery.fn.init.prototype = jQuery.prototype;
jQuery().sayNum(999);
jQuery().sayNum(9999);
```
## jQuery、jQuery.prototype、jQuery.fn.init.prototype
首先声明jQuery.prototype就是jQuery.fn，别名而已。

每一次调用jQuery都相当于创建了一个jQuery实例，背后的原理是:
``` javascript
return new jQuery.prototype.init();
```
为什么要用new？而不是直接返回一个实例，实例中顺便挂载prorotype上的方法？
为了实例间相互独立。

不用new会把整体prototype对象返回出去，这样每个实例都共同指向该对象，相互干扰，绝不可取。
使用new新建的实例相互独立，无论链式调用的再长也不会干扰其他实例。

但问题是用new之后this将不再指向prototype对象，里面的方法对new后的实例来说全部不能使用！
解决方法就是这句：
``` javascript
jQuery.fn.init.prototype = jQuery.prototype;
```
手动把prototype赋给init的prototype，这样，init作为构造器new出的每个实例都能挂载jQuery.prototype上的方法。

有空仿照jQuery结构造个小js库，针对移动端高级浏览器，实现各种常见方法，搭配Vue，应该比zepto还简洁。

