---
title: RequireJS入门
date: 2016-02-28 17:58:53
tags: RequireJS
---
javascript语言本身没有“文件包”的概念，不像其他“高大上”的语言那样需要哪个模块就导入相关“包”。最开始js只是写一个个简单的function，后来function多的时候，开始用µ一个大的自执行函数对这些function进行简易封装，比如jQuery就是一个大的function库，最常见的封装模式就是return出一个对象，对象中包含一个个function，并把这些功能模块相关代码单独抽出来成为一个个js文件，通过`<script>`标签摆放好引入到网页中，但js文件又逐渐多了起来，怎么破？那就开始用js“加载器”来管理这里js模块文件了，这里简单介绍一下require.js。
<!-- more -->
先看一张常见的图，就知道js模块化的必要了：
![没有使用RequireJS](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160301-10.png)

## 引用require.js  

首先，在页面head中引入require.js:
``` html
<head>
    <title>RequireJS</title>
    <script src="js/lib/require.js" data-main="js/main.js"></script>
</head>
```
其中`data-main="js/main.js"`定义了该页面js的入口为js文件夹下的main.js,可简写为`data-main="js/main"`。
## 入口main.js
下面是入口main.js:
``` javascript
require(['a', 'b'], function(a, b){
     console.log('main.js执行');
     a.sayA();
     b.sayB();
})
```
假设该页面需要a,b两个模块，那么就如上所示使用require()函数来实现加载执行，require有两个参数，第一个参数为数组，数组内包含了需索模块的集合，第二个参数是回调函数，该回调函数的参数就是前面需要加载的依赖模块，通过参数传入后在回电函数内使用，这里需要注意：
- 数组中的各个模块根据AMD规范是异步加载的
- 依赖模块全部加载后，回调函数才会执行，不会产生第一张图中js文件上下顺序放置错误的问题
上述代码是在“理想”条件下，这里理想的假设：a.js和b.js和main.js都在同一目录下。
如果不在同意目录下呢？使用require.config()方法：
``` javascript
require.config({
　　　　baseUrl: "js/lib",   //定义根部路
　　　　paths: {
　　　　　　"a": "a.js",
　　　　　　"b": "b.js"
　　　　}
　　});
```
如果有外部资源CDN文件，或分散在各处的js文件，那就只能逐个定义路径：
``` javascript
require.config({
　　　　paths: {
　　　　　　"a": "./js/lib/a.js",
　　　　　　"b": "http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"
　　　　}
　　});
```
## define各个模块
这里该介绍每个模块的书写方式了，AMD规范要求每个模块都要单独抽成一个独立的js文件，且AMD规范要求每个模块包含在define()当中，所以单独的模块a应该这样：
``` javascript
define(function(){
	var a = function(){
		console.log("hello a!");
	}
	return {
		sayA:a
	};
});
```
因此，现在很多插件的开头都做了AMD写法，经常看到如下代码：
``` javascript
function( root, factory ) {
	if( typeof define === 'function' && define.amd ) {
		// AMD. Register as an anonymous module.
		define( function() {
			root.Reveal = factory();
			return root.Reveal;
		} );
	} else if( typeof exports === 'object' ) {
		// Node. Does not work with strict CommonJS.
		module.exports = factory();
	} else {
		// Browser globals.
		root.Reveal = factory();
	}
}
```
那对于没有按照AMD规范书写的插件呢？当然可以加载，单用要shim配置一下：
``` javascript
require.config({
　　　　shim: {
　　　　　　'a':{
　　　　　　　　exports: 'a'
　　　　　　},
　　　　　　'b': {
　　　　　　　　deps: ['c'],
　　　　　　　　exports: 'b'
　　　　　　}
　　　　}
　　});
```
可看到shim对象里有两个属性，exports和deps：
- exports暴漏模块接口，相当于定义了该模块的名字，外部环境要使用该模块就调用exports导出的模块名,比如`jQuery`导出为`$`
- deps也是个数组，定义了改模块的依赖模块（模块c），更多的话`deps: ['c','d','e']`
require.js基本用法就这些，查看更多当然去[官网](http://requirejs.org/)了，如需要还有[插件](https://github.com/requirejs/requirejs/wiki/Plugins)。
## 别被RequireJS，CommonJS，AMD，CMD搞混了
RequireJS只是实现AMD规范的一个js文件"加载器",类似的加载器有很多，github上很多大牛都写过。
[AMD](https://github.com/amdjs/amdjs-api/wiki/AMD)作为一种规范，在[CommonJS](https://webpack.github.io/docs/commonjs.html)规范上做了改良，但业界也有争议和不足，比如：*所有加载的模块不管用没用到，都已经预先执行*。所以又产生了[CMD](https://github.com/cmdjs/specification/blob/master/draft/module.md)集各规范之所长的[SeaJS](http://seajs.org/docs/)。这些规范从CommonJS，到AMD，再到CMD，在社区内讨论争议的各个派系很多，里面很有故事，程序员之间也有种“文人相轻”的感觉。
