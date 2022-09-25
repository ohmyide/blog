---
title: Element.classList
date: 2016-06-17 10:18:26
tags: javascript
---
`classList`是H5的一个新API，然而这个“新”到目前为止也已经好几年了。
## 目前类名的操作方式
我们知道,原生js访问元素类名用的是`ele.className`，其返回一个包含元素所有类名的字符串：
``` html
<div id="box" class="aa bb cc"></div>
var box = document.getElementById("box");
box.className // aa bb cc
typeof box.className //string
```
<!-- more -->
而我们用到各种js库都封装了`addClass`，`removeClass`，`hasClass`等类名操作方法，其封装原理也是在`className`的基础上，对字符获得的类名字符进行各种处理。所以在`zepto.js`的源码上，我们可以看到以看到以上类名操作方法的实现过程：
``` javascript
hasClass: function(name){
  if (!name) return false
  return emptyArray.some.call(this, function(el){
    return this.test(className(el))
  }, classRE(name))
},
addClass: function(name){
  if (!name) return this
  return this.each(function(idx){
    if (!('className' in this)) return
    classList = []
    var cls = className(this), newName = funcArg(this, name, idx, cls)
    newName.split(/\s+/g).forEach(function(klass){
      if (!$(this).hasClass(klass)) classList.push(klass)
    }, this)
    classList.length && className(this, cls + (cls ? " " : "") + classList.join(" "))
  })
},
removeClass: function(name){
  return this.each(function(idx){
    if (!('className' in this)) return
    if (name === undefined) return className(this, '')
    classList = className(this)
    funcArg(this, name, idx, classList).split(/\s+/g).forEach(function(klass){
      classList = classList.replace(classRE(klass), " ")
    })
    className(this, classList.trim())
  })
},
toggleClass: function(name, when){
  if (!name) return this
  return this.each(function(idx){
    var $this = $(this), names = funcArg(this, name, idx, className(this))
    names.split(/\s+/g).forEach(function(klass){
      (when === undefined ? !$this.hasClass(klass) : when) ?
        $this.addClass(klass) : $this.removeClass(klass)
    })
  })
},
```
## classList操作类名
然而有了`classList`再也不用这么麻烦了，其返回一个元素所有class的`DOMTokenList`，但不是数组，通常称为类数组：
``` html
<div id="box" class="aa bb cc"></div>
var box = document.getElementById("box");
console.log(box.classList); // ["bb", "cc", "ee", value: "bb cc ee"]
```
![返回结果](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160617-1.png)
其__prop__指向`DOMTokenList`。
## classList包含以下方法：
- length 返回类数组长度，即类名个数
- item 按下标读取某个类名 
- add 添加一个类名到类名列表中，添加多个：add("xx","xxx");
- remove 删除类名,删除多个：remove("xx","xxx");
- toggle 切换一个类名，该方法第二个参数为布尔型，决定是否添加或删除类名
- contains 检测试都含有每个类名，类似hasClass，返回布尔型

针对以上方法的具体demo：
``` javascript
var box = document.getElementById("box");
box.classList.add("dd","pp"); //aa bb cc dd pp
box.classList.remove("dd","pp"); //aa bb cc 
box.classList.toggle("ee",true); //aa bb cc ee
box.classList.toggle("ee",false);//aa bb cc
box.classList.length // 3
box.classList.item(2)// cc (下标从0开始)
box.classList.contains("cc") // true
```
## 各大浏览器支持情况
已经出道好几年的“新属性”，个浏览器支持情况都还不错，除了IE浏览器需要11才开始支持，移动端支持良好。我搜了一下目前各大js库还没开始应用，估计还要等几年吧，其实针对移动端的各大js库可以考虑使用了，毕竟`classList`能超级节省类名操作相关的逻辑代码。
![兼容性良好](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160617-0.png)