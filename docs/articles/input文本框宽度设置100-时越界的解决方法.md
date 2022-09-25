---
title: 'input文本框宽度设置100%时越界的解决方法'
date: 2016-05-13 18:08:27
tags: css 
---
在开发H5页面时碰到这么一个坑，input文本框的宽度设置为100%时，其实际宽度居然会超出父元素，然而在PC上一切正常。碰到这个坑也是机缘巧合，之前开发H5都直接引用通用样式库文件，由于这次只是单页开发，加上是微信开发需要多一步微信跳转，为加快页面加载速度就没引入这些通用文件，结果问题闪现了：
<!-- more -->
![超出问题](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160513-0.png)
css代码是这样的，“看起来”并没有问题：
``` css
input{
    width: 100%;
    height: 3.5rem;
    border: 1px solid #e6e6e6;
    border-radius: 3px;
    outline: none;
    font-size: 1.4rem;
    padding-left: .5rem;
    background: #f2f2f2;
}
```
## 先说解决办法再分析原因
我google了一下此问题，得到准确的解决方法是在css样式里设置一个css3属性`border-sizing`为`border-box;`，由于是css3属性，安全起见还是需要兼容各个浏览器，具体代码如下：
``` css
input{
    -webkit-box-sizing: border-box;
    -moz-box-sizing: border-box;
    box-sizing: border-box;
}
```
刷新后效果立竿见影：
![解决问题](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160513-1.png)
整个世界都安静了，到此问题已经解决，可作为一名程序员一定要追其究竟，否则下次依然会碰到类似的问题。
## 问题的根本原因： 
上述问题很明显是文本框的实际宽度超出了，那么所谓实际宽度有哪些组成？为什么会超出呢？我们都知道盒模型的占用空间包括：`content+padding+border+margin`然而差异就在content上，这也是问题的根源，因为计算盒模型content的尺寸有两个模式：
> * W3C标准定义：content宽度就是实际定义的宽度，width你定义多款conten就是多宽
> *传统的IE下定义：content包括：width、padding、border，也就是content=width+padding+border

由于计算宽度采用了正统的W3C标准，计算出的实际宽度为：`100%`的元素宽度+`0.5rem`的左padding+左右共计`2px`的border。所以超出的部分是`0.5rem`的左padding和左右共计`2px`的border。

因此我们用到了`box-sizing`
其属性值为：
``` css
box-sizing: content-box|border-box|inherit;
```
> * content-box只把宽高设为content，在宽度和高度之外绘制元素的内边距和边框。
> * border-box把元素的padding和border都在已设定的宽度和高度内进行绘制。

所以我么要把input的box-sizing设为border-box，此元素的内边距和边框不再会增加它的宽度。到此问题才算解决，下面可以把这段代码写到reset.css文件里了，因为我在amazeui.css中也发现了这样的代码：
``` css
/*! Amaze UI v2.6.2 | by Amaze UI Team | (c) 2016 AllMobilize, Inc. | Licensed under MIT | 2016-04-22T15:38:46+0800 */
/* ==========================================================================
   Component: Base
 ============================================================================ */
/**
 * Fix the flawed CSS box model - Yes, IE6's box model is better
 * Browser support: IE8+
 * via: http://paulirish.com/2012/box-sizing-border-box-ftw/
 */
*,
*:before,
*:after {
  -webkit-box-sizing: border-box;
          box-sizing: border-box;
}
```
这个问题估计并不常见，因为在实际开发中，往往直接引用了专门的样式库，这些样式库里自然都由经久沙场的老手写进去了，还好因为这次没用才发现这个问题。题外话：`box-sizing`虽然是css3的内容，但IE8居然也都支持，赞！


-----------------小小分割线-----------
## 去除苹果浏览器默认样式

ios设备上的input，button都有默认样式的，input文本框有内阴影，按钮有圆角，且颜色很怪，用下面代码清除之：
``` css
input[type="submit"],
input[type="text"],
input[type="button"],
input[type="number"],
select,
button {
  -webkit-appearance: none;
}
```
