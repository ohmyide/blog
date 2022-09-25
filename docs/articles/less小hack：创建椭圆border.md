---
title: less hack小技巧
date: 2016-07-11 19:42:26
tags: css
---
## 创建椭圆border
大家都爱用`border-radius`创建圆角,省时省力，但创建椭圆可能用的相对较少：
``` css
border-radius: 100px/50px;
```
这段代码在css下本可以创建一个弧度分别为100、50px的椭圆，但在less下创建的却是一个20px的圆角，原因是less把“/”视为除号运算符，也就是100/50=20。对此的hack技巧为：
<!-- more -->
``` css
border-radius: 100px~"/"10px;
```
用`~"/"`声明`/` 即可。

## 兼容浏览器
有时为了兼容IE浏览器，不得不在css后面加上`\9`,这样在less环境下会报错：
``` css
background-color:blue\9;
```
处理方法：把`/9`定义为一个变量`@hack:~"\9";`，这样less在编译时就能顺利通过：
``` css
background-color:blue@hack;
```
还可以定义为这样：`@hack: ~"/*\**/";`，这样也不会报错，定义后less代码可以写为：
``` css
background-color@{hack}: blue\9; 
```
编译后的css代码为：
``` css
background-color/*\**/: blue\9;
```