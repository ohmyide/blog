---
title: 去除li列表边界多余margin值的方法总结
date: 2016-01-15 20:06:08
tags: css
---
这个问题每个切图匠都会碰到：一个固定宽度的box，内部用li浮动布局，每个li用间距隔开，那么最后一个li的间距怎么破？效果如下图所示：
![效果](http://7xprui.com1.z0.glb.clouddn.com/blog%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-01-11%20%E4%B8%8B%E5%8D%889.09.13.png)
<!-- more -->
我们的html是这样的：
``` html
<div class="box">
  <ul>
    <li>宽100高60</li>
    <li>宽100高60</li>
    <li>宽100高60</li>
    <li>宽100高60</li>
    <li>宽100高60</li>
  </ul>
</div>
```
首先灰色box宽度肯定是有明确要求的，如图所示灰色box宽度为580，li的宽度为100*5+四个20的间距 = 580。每个li分别设置了margin-right:20,那这样的话内容总体宽度为600，已经超出父元素box，于是最后一个li只好被挤到了底下，效果如下：
![问题效果](http://7xprui.com1.z0.glb.clouddn.com/blogddl.png)
下面就是这种现象的集中解决方法，适合各种环境：

## 方法一：用css选择器选中最后一个li，单独去除margin-right
css 代码为：
``` css
li:last-child{
  margin-right: 0px;
}
```
如果你是那种少数喜欢设置margin-left来设置间距的话，用css的选择器还可以选择第一个子元素li:first-child来清除起一个子元素的左margin。这种方法是最基础
的，且仅限于单列的li布局。如果是多列布局的话最后一个子元素是整排的最后一个，这样就选择不了了，情况如下图所示：
![效果](http://7xprui.com1.z0.glb.clouddn.com/blog%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-01-11%20%E4%B8%8B%E5%8D%889.13.17.png)
### 方法二：为最后一个添加类名，单独去除margin-right
这种方法是最小白的了，但的确很实用：
``` html
<ul>
    <li>宽100高60</li>
    <li>宽100高60</li>
    <li>宽100高60</li>
    <li>宽100高60</li>
    <li class="last-li">宽100高60</li>
  </ul>
```
css 代码为：
``` css
li.last-li{
  margin-right: 0px;
}
```
这种方法实用的原因是可以解决多列的问题，只不过是实现的方法比较low而已。因为代码是这样写死的.

html代码要这样：
``` html
<ul>
    <li>宽100高60</li>
    <li>宽100高60</li>
    <li>宽100高60</li>
    <li>宽100高60</li>
    <li class="last-li">宽100高60</li>
    <li>宽100高60</li>
    <li>宽100高60</li>
    <li>宽100高60</li>
    <li>宽100高60</li>
    <li class="last-li">宽100高60</li>
    <li>宽100高60</li>
    <li>宽100高60</li>
    <li>宽100高60</li>
    <li>宽100高60</li>
    <li class="last-li">宽100高60</li>
    <li>宽100高60</li>
    <li>宽100高60</li>
    <li>宽100高60</li>
    <li>宽100高60</li>
    <li class="last-li">宽100高60</li>
  </ul>
```

首先这种写死的方式是要受到严重鄙视的，其次，在开发中都是用模板来循环渲染数据的，所以模板中只定义一个li的html代码，比如我们前端用到的渲染模板[Handlebars](http://handlebarsjs.com/)：
``` html
{{#each RewardList }}
<li>
    <span class="list-content">{{userName}}</span>
    <span class="list-grade">{{goldQuantity}}</span>
</li>
{{/each}}
```
上面的模板代码根本无法为边界li指定class，所以这中方法只适合那些页面写死的静态页，程序员是有追求的，总要寻求最优的解决方法，这种方案肯定不行。

## 方法三：用css选择器
如果不想写死html，那么可以考虑写死css，用:nth-child(n)选择器也可实现效果：
``` css
li:nth-child(5){
  margin-right: 0px;
}
li:nth-child(10){
  margin-right: 0px;
}
li:nth-child(15){
  margin-right: 0px;
}
li:nth-child(20){
  margin-right: 0px;
}
```
总之还是写死的，下面介绍不写死的方法。

## 方法四：加宽ul，用外层父box的overflow:hidden来切割
这种方法"几乎"可以完美解决问题，这样可以不单独针对边界li清除margin，而是增加ul的宽度，是指能容纳所有li及其margin-right为止，例子中的5个li加5个右margin的宽度为600，超出了box的580宽度，这时候针对box设置overflow:hidden，把ul多出的20像素隐藏掉。
代码如下图所示：
``` css
.box{
  width: 580px;
  margin: 0 auto;
  background-color: #ccc;
  overflow: hidden;
}

ul{
  width: 600px;
  list-style: none;
  background: red;
  overflow: hidden;
}
```
几乎完美的原因是因为这种方法必须要手动设置ul的宽度，直到压在底部的最后li归队为止。

## 方法五：设置ul左margin为负值

这种方法我个人比较喜欢，只要设置ul的负margin值为最后li多出的margin值就行。
``` css
ul{
  list-style: none;
  background: red;
  overflow: hidden;
  margin-right: -20px;
}
```
就这样，整个世界都安静了。

---
-------------------补充分割线------------
## 终极方法：功能强大的:nth-child
先看[:nth-child文档](https://developer.mozilla.org/en-US/docs/Web/CSS/:nth-child),其强大功能兼职超乎想象。
针对上述问题，直接`li:nth-child(5n)`全部搞定。除此之外还能自动匹配奇数，偶数，偏差，间隔。
- 1n+0，或n，匹配每一个子元素。
- 2n+0，或2n，匹配位置为2、4、6、8…的子元素，该表达式与关键字even等价。
- 2n+1匹配位置为1、3、5、7…的子元素、该表达式与关键字odd等价。
- 3n+4匹配位置为4、7、10、13…的子元素。
- -n+3匹配前三个子元素中的span元素。

在此推荐一篇神文：[埃拉托斯特尼筛与CSS选择器](http://yuanchuan.name/2013/12/05/%E5%9F%83%E6%8B%89%E6%89%98%E6%96%AF%E7%89%B9%E5%B0%BC%E7%AD%9B%E4%B8%8ECSS%E9%80%89%E6%8B%A9%E5%99%A8.html)，堪称精妙！
