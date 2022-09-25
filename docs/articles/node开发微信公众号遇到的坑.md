---
title: node开发微信公众号遇到的坑
date: 2016-05-13 18:18:25
tags: node
---

## 配置微信
严格遵循官方文档，配置微信信息。这里有个坑：*微信测试号和正式号是两个公众号，要分别配置。
<!-- more -->
``` javascript
var testConfig = {
    "url": "http://xxx.com",
    "weixin": {   
        //测试号
        appid: "wx0fe305afb42xxxxx", //测试号的配置，和正式号不同
        secret: "d4624c36b6795d1d99dcf0xxxxxxxxxx", 
        token: "chayangged1d99dcf0547af5443d"
    },
    redisServer: '182.92.111.111',
    redisPass: "redis-dev-1106",
    port: 8890,
    redisPort: 6379,
}

var onlineConfig = {
    "url": "http://xxx.com",
    "weixin": {
        //正式号
        appid: 'wxfceadcf30d8xxxxx',
        secret: "10653cb914c98973096b65xxxxxxxxxx",
        token: '8SwLJnusbnqjpyvbdi',
        encodingAESKey: 'ajQVhriCeo25Md3Kn0NtTOPxR3omy8QqslTr7Y47F85'

    },
    redisServer: '182.92.111.111',
    redisPass: "redis-dev-1106",
    port: 9094,
    redisPort: 6379,
}
```
## 最常见的错误：redirect_uri 参数错误
![错误提示](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160613-2.png)
遇到该问题首先要找配置信息，且公众号和测试号需要单独配置，各配各的环境。
![解决方法](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160613-0.png)
注意，此处不需要加http。
![解决方法](http://7xprui.com1.z0.glb.clouddn.com/blogQQ20160613-1.png)

## 兼容微信和H5
先判断运行环境是微信还是H5手机浏览器，然后决定传参微信id或token，用以下代码做判断：
``` javascript
var ua = req.headers['user-agent'];
    if (ua && !/MicroMessenger/gi.test(ua)) { //ua存在，而且不是从微信访问的话，直接next
        next();
        return;
    }
```
## 微信分享
在测试帐号上分享出去的样式，标题，图片信息只有关注测试号的账户才可以完美展现，在测试帐号上分享出去的样式，标题，图片信息只有关注测试号的账户才可以完美展现，在测试帐号上分享出去的样式，标题，图片信息只有关注测试号的账户才可以完美展现。
重要的事说三遍。整个办公室参加测试的同学有的关注了测试号，有的没关注，导致分享出去的样式差异，私活找不到原因，最后打开debug才发现需要关注才行，正式帐号上分享没有该问题。
