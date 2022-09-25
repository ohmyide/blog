---
title: 'text-align:justify两端对齐有个坑'
date: 2016-01-18 22:06:00
tags: css
---
事情是这样的：在切图中遇到这么一个常见文本效果，其中昵称、性别、年龄、头像、需要中间撑开：
![需求](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160122-9.png)
我立马想起``text-align``有个相对来说并不常用的``justify``属性。
<!-- more -->
该属性可以实现两端对齐的效果，遂用之，然后顺利掉坑，文本效果不能两端对齐，始终居左：
![错误样式](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160122-11.png)

于是我新建了个测试demo查找原因，我们的代码是这样的：
``` html
.box{
  width: 400px;
  border:1px solid #999;
}

<p class="box">
这是一个测试this is a test demo用来测试text-align: justify;两端对齐效果
这是一个测试this is a test demo用来测试justify两端对齐效果
这是一个测试this is a test demo用来测试text-align: justify;两端对齐效果
这是一个测试this is a test demo用来测试justify两端对齐效果
</p>
```
为了更完善的测试，文字用的是中英混合，样式不加任何修饰期效果是这样的：（注意箭头处空白）
![未对齐效果](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160122-1.png)
然后我们加上``text-align:justify`` 效果立竿见影：
![最后一行没对齐](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160122-0.png)

但有个很明显的问题：除最后一行外，其余所有都已两端对齐，该问题只针对最后一行文本，在我所要切的图中：昵称、性别、年龄、这些文本都是只有一行，
一行也就意味着只有最后一行，所以**text-align:justify对最后一行文本无效**才是问题所在。
号，问题已经找到，接下来就是解决问题的两个方法：

> - **寻找针对最后一行的有关属性加以控制**
> - **想办法把最后一行变为非最后一行**

## 方法一：单独控制最后一行文本
针对最后一行文本，果不其然，我找到了`text-align-last`这个很少出现的属性，专门用来控制最后一行文本的位置样式，虽然该属性目前浏览器兼容不够完善，虽然有些浏览器下行之有效，但这里并不推荐使用该方法。这里有个小插曲：在我找的所有中文资料中都声明了chrome、safari、Opera等众多浏览器红色警告不支持该属性，可我在[Can I Use](http://caniuse.com/#search=text-align-last) 中发现并不是这样：
![Can I Use](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160122-7.png)
而我就是在chrome做的测试，chrome是支持`text-align-last`的，这只并不想抨击这些中文资料的“错误”只能说有些资料更新并不及时，所以这里推荐大家使用[Can I Use](http://caniuse.com)这个网站。
``` css
.box{
  width: 400px;
  border:1px solid #999;
  text-align: justify;
  text-align-last:justify;
}
```
## 方法二：把最后一行变为非最后一行 （这才是重点）
自然想到了伪元素，并且是伪元素:after，用其放置只在文本末端充当最后一个元素，这样最后一段文本自然就能两端对齐。所以针对最后一行文本不能两端对齐问题的最好解决方案就是这样的：
``` css
.box{
  width: 400px;
  border:1px solid #999;
  text-align: justify;
}
.box:after{
  content: "";
  display:inline-block;
  width: 100%;
}
```
需要注意的是伪元素设置`width: 100%;`目的是让伪元素足够宽以至于被挤到文本最后一行。这样我们在审查元素中发现伪元素处于最后一行底部：
![伪元素位置](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160122-4.png)


如果设为`width: 1%;`宽度不足，那么最后一行依然不能两端对齐，在审查元素中会发现该伪元素处于最后一行文字的末端（红点处），并没又重启一行，所以无效。
![伪元素位置](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160122-3.png)



