---
title: 【翻译】jquery是有害的
date: 2016-01-05 14:43:50
tags: 翻译
---
**原文：http://lea.verou.me/2015/04/jquery-considered-harmful/**
（第一次翻译，望大家多批评指正）

   嗨，我总想写一个“X”是有害的帖子。

   在开始写前，我想说jquery 很大程度上促进了前端的发展。它让开发人员实现了以前不敢想象的事情，并促使浏览器开发商做他们本来就该做的事。（没有jquery我们估计现在还不可能用到`document.querySelectorAll`）对于那些不支持现有新技术的IE8及其以下浏览器jQuery依然很有必要。
<!-- more -->

   但无论怎么说那些低端浏览器毕竟是少数，很多开发人员不需要支持只占很小份额的老版本浏览器。别忘了还有那些非专业开发人员：学生和研究人员，他们不仅不需要支持低版本浏览器，而且只需一个浏览器支持就够了。如您所愿，在学术界人人都津津乐道于使用网络开放平台的新技术，对吧？然而我却从未见过jQuery在业界这么突出。为什么？因为众所周知，他们没有时间或兴趣追随网络开放平台的最新动态。他们不知道他们需要什么样的jquery，导致他们只是单纯的使用jquery。然而这并不是我抛弃jquery的唯一理由。
### 是的，你可能真的不需要它
   我当然不是第一个指出jQuery的依赖程度影响你原生js的能力，因此我不想浪费时间重复别人之前所写，你只需访问以下链接：

[你可能不需要jquery](http://youmightnotneedjquery.com/)

[你不需要jquery！](http://blog.garstasio.com/you-dont-need-jquery/)

[你真的需要jquery吗？](http://www.sitepoint.com/do-you-really-need-jquery/)

[脱离jquery编写javascript的10个技巧](http://tutorialzine.com/2014/06/10-tips-for-writing-javascript-without-jquery/)

...还有很多，你只需谷歌一下“you don’t need jQuery”你将发现更多。
我也不再花时间赘述jQuery文件大小以及原生js方法有多高效，这些我之前都讲过。今天，我想说一个不常被提及的要点。

### 但那并不是放弃它的最大原因
   为了避免扩展本地元素的原型，jquery使用它自己的包装对象，扩展本地对象在过去是一个庞大的 不，不。不仅因为潜在冲突，还有低版本IE浏览器内存泄漏。因此当你运行`$(“div”)`时返回的并不是一个元素引用或一个节点集合而是一个jQuery对象。这意味着对于，jquery对象的实现方式完全不同于一个DOM元素的引用、一个数组或其他类型的节点列表。然而，对于这些本地对象，就像jquery试图提取出他们一样，你总要不得不处理他们，哪怕他们包装在$()中。例如：当回掉函数通过jQuery的bind()方法调用时上下文就是一个对HTML元素的引用而不是jquery的一组对象。更别提你的代码还是多源的，有些想当然是jQuery代码，有些则不是，最终代码总会混淆着jQuery对象、本地对象、节点列表，而这正是地狱的开始。

   如果开发人员遵循一个命名规则：用变量包裹jQuery对象（我认为在变量名头部添加一个`$`是常见的一种）和本地元素，这将不再是个问题（但人们总是记不住规则，这里就先假定一个理想世界）然而，现实中并没有那么多规则被遵守，结果就是对于不熟悉代码的人来说代码变得极其难懂。现在每一次编写代码都需要很多尝试和错误（“哦！这不是一个jQuery对象，我要用`$()`来包裹它”或者“哦！这不是一个元素，我要用[0]来获取其中的元素”） 为了避免混淆，开发人员编码时常防御性的用`$()`包裹所有东西，因此总览代码，相同的变量经过`$()`的多重包裹，同样的原因，这会变得很难重构其他jQuery代码，你完全被困住了。

   即使遵循了命名规则，也不能只用在jQuery对象上，你经常需要用到本地DOM方法或调用不属于jQuery而来自其于他脚本中的函数。很快，多次折腾jQuery对象弄得到处都是，把你的代码搞的很乱。

   除此之外，当你往代码库中添加代码的时候，你往往会用$()来包裹每个元素或节点列表。因为你不知道你得到了什么样的输入。所以被困住的不仅仅是你自己，你以后为同一个代码库所写的代码也被困住了。

   获得任何带有jQuery依赖性的随机脚本，你没有自己写并试图重构它，这样它就不需要jQuery。我敢说，你会发现你的主要问题将不会是如何转换功能使用本地APIs, 而是理解这到底是怎么一回事。

### 一个通往原生JS的可靠途径
   当然，现在许多函数库需要jQuery，就像最近我在推特上所说的那样，如果你回避jQuery那么感觉你像是个数码素食者。当然，这并不意味着你必须要使用它。当好的非jQuery代替品可用的时候，函数库也将会被取代。

   同样的，大多数函数库的写法不需要用`$`作为jQuery的别名。用[jQuery.noConflict()](https://api.jquery.com/jquery.noconflict/)方法可更改默认的$并且你也可改成其他你看着顺眼的符号，例如，受命令行API的启发，我经常定义这些帮助函数：
```javascript
//返回匹配到expr的第一个元素
//查询范围限制在container的后代中
function $(expr, container) {
    return typeof expr === "string"? (container || document).querySelector(expr) : expr || null;
}
//以数组的形式返回所有匹配到的expr
//查询范围限制在container的后代中
function $$(expr, container) {
    return [].slice.call((container || document).querySelectorAll(expr));
}
```
   此外，我认为在你每次敲出jQuery来代替$时你会考虑如果真的不需要，是否还要这么过度的使用它，或许我猜错了 。

   同时，如果你喜欢jquery API 但又不喜欢他的臃肿，那么你可以考虑使用[Zepto](http://zeptojs.com/)。

>*很明显，我们的标题显而易见带有开玩笑的意味，但是，这是互联网，没有什么是显而易见的。所以在这里我很清楚[Eric的经典文章](http://meyerweb.com/eric/comment/chech.html)会很反对这种标题。
