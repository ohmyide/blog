---
title: 'position: sticky;'
date: 2016-06-15 16:45:29
tags: css
---
前端css又出新属性了，`position: sticky;`，目前还属于W3C的一个草案，未正式纳入标准，所以现在浏览器对齐支持程度很差，截至目前（2016.06）为止，该属性在个浏览器支持程度如下：
![兼容性](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160615-1.png)
兼容性一篇红啊。
<!-- more -->
先看其基本用法：
``` css
.test{
  position: -webkit-sticky;
  position: sticky;
  top: 50px;
}
```
其定位效果很有意思，偏移量参考层级最近的可滚动的祖元素。当滚动距离在目标区内时，其定位效果和relative一样，紧跟着父元素滚动，相对位置不变。一但滚动超出目标区，则以fiex定位方式固定，无论父元素怎么滚动，其为止也不再变化，做了个简单的gif图，展示效果：
![sticky效果]()
``` html
<body>
  <div class="box">
    <div class="test t1"></div>
    <p>我是参考文字</p>
  </div>
</body>
```
css:
``` css
.box{
  width: 300px;
  height: 800px;
  background: #ccc;
  margin-top: 100px;
}
.test{
  width: 100px;
  height: 100px;
  background: orange;
  position: -webkit-sticky;
  position: sticky;
  top: 20px;
  left: 100px;
  margin-bottom: 10px;
}
```
此demo是在火狐浏览器下运行，chrome目前还不支持该属性，纳入标准估计还要等好久，提前感受一下，挺有意思。