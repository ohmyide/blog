---
title: javascript自定义事件
date: 2016-04-29 18:09:07
tags: javascript
---

js这门语言最重要的主题莫过于事件了，网页用户所有操作的反馈,都是通过事件获得响应，点击，双击，提交，选中等等。浏览器提供了多少事件呢？在控制台运行如下代码，你会很吃惊：
``` javascript
for( i in window){
    if(/^on/ig.test(i)){
        console.log(i);
    }
}
```
<!-- more -->
事件的数量远超你想象：
![事件截图](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160503-0.png)
js提供的事件基本满足了常见需求，对特殊需求我们要用到自定义事件，自定义事件有两种实现方式：依赖DOM元素的自定义事件和js自定义事件。

## 基于DOM的自定义事件
基于DOM的自定义事件，首先用`new Event()`初始化一个事件实例，然后在某个DOM元素上添加改事件的监听函数，最后由`dispatchEvent()`触发：
``` javascript
var elem = document.getElementsByTagName("body")[0];
var event = new Event('test');
elem.addEventListener('test', function (e) { 
    console.log("test事件被触发了！");
}, false);
elem.dispatchEvent(event); //test事件被触发了！
```
## js自定义事件
用纯js实现自定义事件核心有三部分：定义一个事件容器，存储事件；一个触发函数，一个取消事件函数。说白了就是为每一个事件新建一个数组，在数组中依次存入事件的回调函数，触发是找到该事件名，并执行该事件数组中的各个函数。
``` javascript
function customEvents(){
    return {

        callbacks: {}, //空对象用来存储事件队列

        addEvent: function(msg,callback){
            this.callbacks[msg] = this.callbacks[msg] || [];
            this.callbacks[msg].push(callback); // 存入事件
        },
        fireEvent: function(msg){
            this.callbacks[msg] = this.callbacks[msg] || [];
            for(var i = 0,len = this.callbacks[msg].length; i < len; i++){
                this.callbacks[msg][i].apply(this,arguments); // 触发
            };
        },
        removeEvent: function(msg){
            this.callbacks[msg] = null; //删除该事件
        }

    };
}
```
自定义事件机制写好后，就可以用实例来应用一下了：
``` javascript
var a = new customEvents();
function bigbang(){
    console.log("bigbang!!");
}

a.addEvent("fire",bigbang); // 绑定fire事件，执行bigbang
a.fireEvent("fire"); //bigbang！！
a.removeEvent("fire"); //解除该事件绑定
a.fireEvent("fire"); //

```

