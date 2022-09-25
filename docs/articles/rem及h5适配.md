---
title: rem及h5适配
date: 2016-12-12 14:06:05
tags: css
---
## 什么是rem？
rem是css3才有的新度量单位，定义为：font size of the root element。首字母的r是root，这个根代指`<html>`。那em呢？关于rem和em的关系参考[这篇文章](http://www.w3cplus.com/css/rem-vs-em.html)。
<!-- more -->
px是固定尺寸，rem是相对尺寸，rem的值与`<html>`字体px大小成倍数关系，比如：
``` css
html{
	font-size:10px;
}
p{
	font-size:2rem;  // 2*10 = 20px
}
```
那么html默认字体是多少呢？——16px。
``` css
html{
	font-size:100%;
}
p{
	font-size:1rem; // 16px;
}
```
但我们的设计稿的距离都是px，当用rem为单位时总要这么一步计算：设计稿尺寸/16 = 想要的rem。但16这个数不好除，所以经常会看到这样定义html的fontsize：
``` css
html{
	font-size: 62.5%;
}
p{
	font-size: 2rem; // 20px;
}
```
62.5哪里来？答案是10/16 = 0.625。这里相当于把html的font-size定位10px，只要把设计稿的尺寸往左移动一个小数点即可得到rem值，方便了很多。

rem的使用技巧就这些。
## 计算viewport
常见的viewport长这样：
``` html
<meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=0">
```
重点是initial-scale的值，scale由window.devicePixelRatio计算而来：
``` javascript
scale = 1/window.devicePixelRatio
```
### window.devicePixelRatio 是什么？
window.devicePixelRatio是设备像素比，也就是设备的物理像素（屏幕上的点状小颗粒）与设备独立像素（计算机系统屏幕坐标位置点）的比值。其中iphone5、6都为2，plus为3，其他安卓也大不相同。

## h5适配方案
首先根据不同的devicePixelRatio计算出scale，重写进meta标签中。

然后，既然控制根节点html的font-size就能控制整个页面，那么针对不同尺寸的h5浏览器，就可以根据各自尺寸来调节页面了，html的font-size肯定不能写死，需要根据js检测屏幕尺寸后动态计算，这就是适配工具的原理。核心代码：
``` javascript
var dpr = window.devicePixelRatio || 1,
scale = 1 / dpr,
content = 'width=device-width, initial-scale=' + scale + ', minimum-scale=' + scale + ', maximum-scale=' + scale + ', user-scalable=no';

metaEle.setAttribute("content", content); 

var width = document.documentElement.getBoundingClientRect().width || window.innerWidth;

document.documentElement.style.fontSize = width / 16 + "px"; // 默认16，除以10也行，方便计算

```

## 其他细节
除此之外，还要注意一些细节，比如字体，既然css统一用了rem，html根字体是后续js修改的，并不能影响body中的字体，虽然字体默认是继承自父元素，这里的父元素并不是html而是body，所以也要js手动设定body的字体为：devicePixelRatio*12

注意一个bug：当页面背景为整个一张图片时，触发输入键盘后，键盘会从底部弹出，占用屏幕很大一块空间，正常情况下是网页整体也随之上升，但在很多安卓浏览器下并不是这样，文本框上升到可输入位置，但背景并没有被键盘推上去，而是在上面残余的空间内『挤压』的很窄。针对这一现象，在js中计算出屏幕高度后，用次高度写死body的高度即可。





