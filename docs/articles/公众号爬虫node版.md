---
title: 公众号爬虫node版
date: 2017-03-28 20:53:23
tags: node
desc: node爬虫
---
之前用[python实现](http://chayangge.com/2017/03/22/%E7%94%A8python%E6%8A%93%E5%8F%96%E5%85%AC%E4%BC%97%E5%8F%B7%E6%96%87%E7%AB%A0/)了一盒简单爬虫，这次还用node实现，体会两者的差异。原理都一样，代码不到100行，这里是[源码](https://github.com/chayangge/node-crawler)，数据来源[在这里](http://chuansong.me/)。
<!-- more -->
## 使用方法
clone并安装：
``` javascript
git clone https://github.com/chayangge/node-crawler.git
cd node-crawler
npm install
```
运行：
``` javascript
node app
```
会在项目根目录下创建以微信ID命名的文件夹，里面存放抓取后的文章。