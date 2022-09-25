---
title: javascript中replace()详解
date: 2016-01-24 20:48:53
tags: javascript
---

功能强大的replace()是开发中的利器，往往一段复杂的逻辑代码只需一个replace就能搞定。但掌握好它也挺不容易，因为我们不仅要了解它的基本用法、参数，还要掌握正则、算法等技能，综合起来才能让它的功力发挥的淋淋尽致，以下是我在开发中遇到的用法，留作总结。
首先介绍一下replace()的基本用法：
``` javascript
string.replace(regexp, replacement或函数)
```
<!-- more -->
第一个参数可以为一个字符串，也可以十一个强大灵活的正则，正则怎么写这里不做重点介绍。需要注意的是第二个参数，该参数既可以是个字符串也可以是个函数，形形色色的替换规则就是靠第二个参数来实现的，尤其是字符串中夹杂的$,非常有意思：
![$的不同含义（图片来自犀牛书）](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160215-3.png)

当第二个参数为函数时，每个匹配项都调用该函数，该函数返回的字符串作为替换文本。想到了`sort(compare)`? 
**重点是该函数的参数不是固定的且至少有4个！**不同的参数又代表不同含义,下面是四个参数时的情况，匹配[]中的两个汉字：
``` javascript
var str = "这是[第一]用来[第二]测试的[第三]文本！";
var newStr = str.replace(/\[([\u4e00-\u9fa5]{2})\]/g, function(a, b, c, d) {
    console.log(a);  
    console.log(b);
    console.log(c);
    console.log(d); //如果继续打印e,f,g...多出的参数会undefined
});
```
第一个参数a代表匹配项： [第一]
第二个参数b代表匹配项的组匹配：  第一 （注意：只有一个组匹配，它决定了参数的个数）
第三个参数c代表匹配到的内容在字符串中的位置：  2
第四个参数d代表整个字符串：  这是[第一]用来[第二]测试的[第三]文本！

由于每个匹配项都执行该函数，则以上代码运行结果为：
![运行结果](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160215-4.png)

下面是4个以上参数的情况，也就是有多个组匹配的情况,匹配[]中的所有的两个汉字，此时参数为5个：
``` javascript
var str = "这是用来[两对汉字]测试的文本！";
var newStr = str.replace(/\[([\u4e00-\u9fa5]{2})([\u4e00-\u9fa5]{2})\]/g, function(a, b, c, d,e) {
    console.log(a);   // [两对汉字]
    console.log(b);   // 两对
    console.log(c);   // 汉字
    console.log(d);   // 4
    console.log(e);   // 这是用来[两对汉字]测试的文本！
});
```
以上代码运行结果为：
![运行结果](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160215-6.png)
以此类推，函数的倒数第一个参数始终是整个文本，倒数第二个始终表示位置，第一个参数始终表示匹配项，重点是从第二个参数开始，组匹配的个数决定了函数参数的个数。

好了，难点就这些，下面是我用到的小例子，供大家参考，有不足或优化建议还请多多指教：

## 1.加密手机号码
``` javascript
var phoneNum = "18842961561"
phoneNum.replace(/([1-9]{3})\d{4}(\d{4})/,"$1****$2");
// 188****1561
```

## 2. 去空格
去空格是最常见的使用方法了，去空格包括去除所有空格，去除首尾空格，很多js类库都用它实现了trim方法:
### 去除首尾空格
``` javascript
var str = "   abc def jh  "
str.replace(/^\s+|\s+$/g, ""); //去首尾空格。去所有空格：/\s/g
"abc def jh"
```
比如mootools的源码：
![mootools源码](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160215-5.png)

## 3.匹配聊天表情
在网络聊天中经常要插入表情，如： 早点休息[晚安]拜拜，要匹配出晚安二字并替换为晚安的表情，那么具体实现方法如下：
先建立一个对象，保存表情名与对应的表情图片链接地址，然后匹配表情名，如果对象里有该表情名则替换出相应的图片地址：
``` javascript

var json = { // 表情  链接 存储对象
        "抽搐": "./img/expression/01.gif",
        "大哭": "./img/expression/01.gif",
        "点头": "./img/expression/03.gif",
        "调皮": "./img/expression/04.gif",
        "奋斗": "./img/expression/05.gif",
        "害怕": "./img/expression/06.gif",
        "好冷": "./img/expression/07.gif"
    };

var content = "好感动[大哭]太感人了！";
var newContent = content.replace(/\[([\u4e00-\u9fa5]{2})\]/g, function(a, b) {
    console.log(a); // [大哭]
    console.log(b); //大哭
    if(json[b]){ 
      return "<img src='" + json[b] + "' />"   //<img src='./img/expression/01.gif' />
    }
});

```

## 添加千位符
``` javascript
function addComma(num){
    return (num+"").replace(/(\d)(?=(\d\d\d)+(?!\d))/g, "$1,");
}
addComma(10086) // 10,086
addComma(-10086) // -10,086
addComma(1008.6) // 1,008.6
addComma(-100000.90989); // -100,000.90,989
```

就这些，日后再加。

## 添加千位符
``` javascript
function addComma(num){
    return (num+"").replace(/(\d)(?=(\d\d\d)+(?!\d))/g, "$1,"); 
}
addComma(10086) // 10,086
addComma(-10086) // -10,086
addComma(1008.6) // 1,008.6
addComma(-100000.90989); // -100,000.90,989
```
## 裁剪数字
``` javascript
function cutNumber (value,len) {
    var str = value.toString();
    return str.length <= len ? str :str.substr(0,len) + '…';
}
cutNumber(1000000,4);  //1000...
```
## 匹配中文姓名
``` javascript
var reg = /^([\u4e00-\u9fa5]{2,8})$/
if (reg.test("峰谁说的峰谁说的是")) {
    console.log("yes")
}else{
  console.log("no")
}
```


>参考资料：
>《javascript权威指南》
>[《javascript标准参考教程》](http://javascript.ruanyifeng.com/stdlib/regexp.html#toc8)
