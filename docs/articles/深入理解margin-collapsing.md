---
title: 深入理解margin collapsing
date: 2016-03-01 16:31:47
tags: css
---
你以为margin折叠只发生在相邻元素之间？别漏掉了父子元素之间也会产生margin折叠。本篇对其产生条件及消除方法进行总结，希望对你有所帮助，欢迎指正错误和不足。
产生margin折叠的两种情况：
- 相邻上下位置关系元素
- 嵌套的父子或祖先元素（本文重点）
<!-- more -->

## 第一种情况：相邻上下位置关系元素
首先了解一下上元素的`margin-bottom`与下面元素的`margin-top`数值的折叠规则：
> 规则1. 正数、正数取最大（最常见）50px,20px --> 50px
> 规则2. 一正、一负取其和（不常见）50px,-20px --> 30px
> 规则3. 负数、负数 绝对值取最大（很少见） -50px,-20px --> -50px

看代码：
``` html
<div class="wrap">
    <div class="pre">pre</div>
    <div class="next">next</div>
</div>
```
css:
``` css
.wrap{
    width: 300px;
    height: 200px;  /*防止父元素塌陷设定了高度*/
    background: #ccc;
}
.pre{
    width: 100%;
    height: 30px;
    background: #8FB2F2;
    margin-bottom: 50px;
}
.next{
    width: 100%;
    height: 40px;
    background: #FE9EA4;
    margin-top: 20px;
}
```
这是最常见的折叠情况，上元素的`margin-bottom`与下面元素的`margin-top`发生了规则1的折叠，取其最大值，所以总体上下间距为`50px`,如下图所示：
![50px](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160301-1.png)
规则2的情况各为50px,-20px时，间距为30xp，效果如下：
![30px](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160301-3.png)
规则3的情况各为-50px,-20px时，间距为-50xp，效果如下：
![-50px](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160301-4.png)
### 消除折叠
在[规范里](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing)有这么一句话：
> **Margins of floating and absolutely positioned elements never collapse.**

也就是说float元素和absolute定位的元素（都脱离了文档流，且形成了BFC）不会产生margin折叠。
于是我测试了下以下下代码：
``` css
.pre{
	float:left;
    width: 100%;
    height: 30px;
    background: #8FB2F2;
    margin-bottom: 50px;
}
.next{
	float:left;
    width: 100%;
    height: 40px;
    background: #FE9EA4;
    margin-top: 20px;
}
```
此时间距为总和70px，没有发生折叠：
![70px](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160301-2.png)
同样，添加以下代码也是如此（对.pre .next两元素设定display: inline-block;）
``` css
.pre,.next{
    display: inline-block;
}
```
为什么也没发生折叠？因为float后的元素效果和`display: inline-block;`效果一样，形成了新的BFC，所以也不会折叠。

## 第二种情况：父子元素之间（本文重点）
先看规范中的详细解释：
> If there is no border, padding, inline content, or clearance to separate the margin-top of a block from the margin-top of its first child block, or no border, padding, inline content, height, min-height, or max-height to separate the margin-bottom of a block from the margin-bottom of its last child, then those margins collapse. The collapsed margin ends up outside the parent.

看代码：
``` html
<div class="wrap">
    <div class="fa">
        <div class="son"></div>
    </div>

</div>
```
css:
``` css
.wrap{
    display: inline-block;
    width: 300px;
    height: 200px;  
    background: #ccc;
}
.fa{
    width: 100%;
    height: 200px;
    background: #8FB2F2;
}

.son{
    width: 90%;
    height: 150px;
    margin: 0 auto;
    margin-top: 50px;
    background: #FE9EA4;
}
```
你本想的效果是这样的：
![想要](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160301-6.png)
可实际折叠后的效果是这样的：
![实际](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160301-5.png)
那么问题来了，怎样禁止子父元素间的margin折叠？
### 清除父子嵌套中产生的折叠的方法
禁止产生折叠就应先知道产生折叠的原因：
> 1.父子元素紧密相邻无其他元素 no border,no padding
> 2.同处一个BFC

所以破坏折叠条件就自然能阻止margin折叠了
针对条件1可采用，增加border、padding、“文字”、伪元素 都可解决:
> 1. 父元素加border：`border-top:.1px solid transparent;` 透明且很窄
> 2. 添加padding：`padding-top: .1px;`
> 3. 添加加伪元素： `.fa:before{content: "";display:inline-block;}`
> 4. 父元素标签中加点“文字”,不让其中间为空，如下图：
![添加文字](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160301-8.png)


针对条件2产生新的BFC即可，方法有设置 display,float,absolute,overflow。具体为：

> 5. 设置float属性不为none
> 6. 设置position为absolute或fixed
> 7. 设置display为inline-block,flex, inline-flex
> 8. 设置overflow不为visible


### 子父关系中最后一个子元素下边距同样也会与父元素折叠

原因和解决方法同上所述

### 子父元素之间的折叠会扩展到祖元素，直到body
删除祖元素.wrap中的`display: inline-block;`使其不产生新的BFC，此时整个页面都会的父子元素的折叠会扩展到body上，其效果是整个body因最里面的子元素的`margin-top`而下沉：
``` css
.wrap{
	/* 删除inlin-block */
    width: 300px;
    height: 200px;  
    background: #ccc;
}
.fa{
    width: 100%;
    background: #8FB2F2;
}

.son{
    width: 90%;
    height: 90%;
    margin: 0 auto;
    margin-top: 50px;
    background: #FE9EA4;
    margin-bottom: 50px;
}
```
效果：
![效果](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160301-9.png)

## 空元素也会产生margin折叠
一个无高度、最小高度，无border,无padding，无内容的元素也会发生margin折叠：
``` css
.wrap{
    width: 300px;
    height: 200px;  
    background: #ccc;
}
.fa{
    width: 100%;
    background: #8FB2F2;
}
.son{
	/* 上述中的三无青年 */
    width: 90%;
    background: #FE9EA4;
    margin-top: 50px;
}
```
效果：
![空元素](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160301-11.png)

关于margin折叠的问题，BFC的概念很重要，理解了它很多问题迎刃而解，多查资料，多用google，多查标准，希望对你有所帮助。

参考资料：
[https://www.w3.org/TR/CSS21/box.html#collapsing-margins](https://www.w3.org/TR/CSS21/box.html#collapsing-margins)
[https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing)