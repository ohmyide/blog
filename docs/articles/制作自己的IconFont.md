---
title: 制作自己的IconFont
date: 2016-12-16 19:02:08
tags: css
---
IconFont已经广为熟知了，与图片相比其优势自然不言而喻，下面就是简单介绍一下其制作过程：
## 所用工具fontLab
![fontLab](http://7xprui.com1.z0.glb.clouddn.com/icon5.png)
<!-- more -->
fontLab可以用来修改或创建图标样式。
## 制作过程
用fontLab打开一个字体文件或创建一个，至于修改过程类似ps，如果ps比较熟的话，一般这类软件应该很容易操作。
![打开后的字体文件](http://7xprui.com1.z0.glb.clouddn.com/icon6.png)
双击打开一个字符，即可在原有图形的基础上修改，也可从新创建，一下就是演示用的一个图标（不太规范）：
![编辑样式](http://7xprui.com1.z0.glb.clouddn.com/icon1.png)
编辑好后，保存，file -> generate font 生成字体文件(.ttf)即可。这里一定要记住该字符的unicode，写在css里，对浏览器来说，unicode就是字符，两者是一对一关系的。比如："A"的unicode是"\u0041",所以有：
``` javascript
"\u0041" === "A"  // true
```
## css部分
我们常用的字体都是系统默认的，比如：
``` css
.test{
 font-family: '微软雅黑';
 font-size: 16px;
}
```
所以，怎样才能让自己定义的字体被css使用呢？——用@font-face：
``` css
@font-face {
  font-family: 'iconfont';
  src: url('./fonts/test.ttf') format('truetype'); // 还有.woff .eot .svg格式
  font-weight: normal;
  font-style: normal;
}
```
自己制作的字体定义为iconfont后，就可以使用了，和系统默认的使用方法一样：
``` css
.test{
 font-family: 'iconfont';
 font-size: 16px;
}
```
这时候就可以使用字体图标了：
``` html
<div id="test">
    <p>测试自己制作的Icon Font：</p>
    <i class="test-icon-font"></i> 
</div>
```
新建个空标签i，把字体作用在其伪元素上：
``` css
.test-icon-font{
  display: inline-block;
  font-family: 'iconfont';
  text-rendering: auto;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.test-icon-font:before{
  content: '\f00c';
  font-size: 80px;
  color: red;
}
```
好了，刷新见效（丑了点哈）：
![自制的图标](http://7xprui.com1.z0.glb.clouddn.com/icon4.png)

