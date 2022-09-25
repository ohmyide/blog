---
title: 用python抓取公众号所有文章
date: 2017-03-22 18:14:26
tags: python
desc: python爬虫 抓取公众号文章
---
在我初学IT的时候，『网络爬虫』是个高大上的词汇，感觉『爬虫自动抓取』简直就是神一样的技术，不明觉厉，想到自己这么菜肯定学不会，后来了解`http`，猛然发现所谓网络爬虫本质上就是一个下载器而已。
<!-- more -->
为了消除爬虫的神秘感，重要的事情说三遍：爬虫就是一个下载器，一个下载器，下载器...
## 那什么是下载器？
下载器就是一个能发起http请求的东西，而浏览器可以理解为一个下载器和一个显示器的合体，那么网络爬虫就是一个能发起http请求，并从请求返回内容中筛选信息的程序。所以理解一个爬虫基本要一下几个硬知识：
- http是无状态的，谁都可以请求，且服务器不知道你是谁和上次什么区别（纯条件下，排除cookie、session技术）
- 凡是能发起请求，也就是可以网络编程的语言都可以写爬虫
- 各种语言为爬虫封装了各种请求框架、解析工具，写一个简单的爬虫真的很简单

## 用python写一个爬虫
python是门好语言，语法简洁优雅，还能做数据分析，爬虫技术更是相当成熟，各种包满足你各种要求，之前用python抓取过某网站的图片，感觉跟给力，现在用它做一个实用点的工具——抓取公众号文章。

公众号文章只能在手机上看，且查看历史还要一直往下滚，体验上和操作上都很麻烦，作为优质的公众号，想保存所有历史文章供日后慢慢欣赏在手机上很难实现，而有的PC网站上有但靠复制粘贴保存也费时费力，所以这个爬虫或许能解决这些问题。

首先要找到公众号文章资源，搜狗搜索提供微信公众号搜索，但只能前10篇，虽然后续可以一直抓取最新，但历史文章依然无法获取，还好有个网站，收录了公众号所有文章，就是这个[传送门](http://chuansong.me/)。用到了如下几个包：
``` python
import time
import sys 
import requests ##发起请求
from bs4 import BeautifulSoup ##分析抓取结果
import os
import random
```
## 分析url
找到文章资源后就要分析页面了，我们以[caoz的梦呓](http://chuansong.me/account/caozsay)公众号为例，我们发现其地址信息为`http://chuansong.me/account/`+`公众号id`的形式，所以我们的爬虫要支持输入微信公众号id作为参数进行爬取:
``` python
import sys 
print "正在搜索公众号：", sys.argv[1]
wechat_id = sys.argv[1]
print "抓取地址：", "http://chuansong.me/account/" + wechat_id
```
## 分析页面
分析页面源码发现DOM节点清晰明了，没有后续js动态渲染，简直是爬虫的最爱：
``` html
<a class="question_link" href="/n/1669813351709" target="_blank">
赠人玫瑰，手有余香
</a>
...
<a class="question_link" href="/n/1663085851719" target="_blank">
Google关键词挖掘细分市场实战案例
</a>
...
```
这里说明为什么要查看源码，因为有些网站为了反爬虫，把后台数据放在js里，让前端js进行后续渲染类似前端模板的作用，这样的html源码空空如也，非常不利于抓取，但还好我们有`PhantomJS`相当于一个无界面的浏览器，依然可以搞定，这里似乎可以明白：在互联网中，几乎没什么是抓不到的，差别只在抓取难度上而已，如果难度很大，成本高，则不如自己制造数据，这样抓取就不划算了。

如上源码所示，节点清晰明了，抓取非常容易，轻轻松松获取`href`即可。至于获取方法，`BeautifulSoup`提供了非常方便的方法，如果你用过jquery的话，那更是轻松自如了，这里查看源代码即可，不过多介绍。
``` python
article_list = Soup.find_all('a',class_="question_link")
page_list = Soup.select(".w4_5 > span > a")
```
获得了节点序列，进行遍历，把遍历出的结果写入文件保存即可，无论是抓图还是抓取文字，都是这样。

## 反爬虫
再次说到http，最开始http设计的时候，在当年的互联网环境下就是为了方便简洁，因此http被设计为无状态的，对服务器来说，不知道谁请求的和上次有什么关系，然而现在不能满足需求，为此设计了cookie、session技术来进行身份识别，这些是反爬虫的技术来源之一，而爬虫获取造成的服务器负担，在法律上目前属于灰色地带，如果一个网站每天被爬虫疯狂的访问，服务器看似很繁忙，然而却没有真实用户，也是件悲伤的事。更有甚者，犹豫爬虫的疯狂请求，导致服务器变慢，影响正常使用，这就算是攻击了。

所以发爬虫也很必要，常见策略有：数据用js动态加载，增加抓取难度，限制IP，如果某个IP访问频率很高，可封锁IP，但这要注意标准，以免误伤真实用户，然而封锁IP也不能从跟上解决，因为爬虫可以通过代理不停的更换IP，总之爬虫和反爬虫就是一个博弈，背后比的是成本，时间成本，技术成本，金钱成本。
## 如何避免反爬虫
说白了就是伪装成真实用户，比如时间上不要这么疯狂，间隔个几秒，别被服务器反爬虫盯住，其次就是不停的更换请求头`User-Agent`：
``` python
user_agent_list = [
        "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.1 (KHTML, like Gecko) Chrome/22.0.1207.1 Safari/537.1",
        "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/536.5 (KHTML, like Gecko) Chrome/19.0.1084.9 Safari/536.5",
        "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/535.24 (KHTML, like Gecko) Chrome/19.0.1055.1 Safari/535.24",
        "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/535.24 (KHTML, like Gecko) Chrome/19.0.1055.1 Safari/535.24",
        "Mozilla/5.0 (compatible; MSIE 9.0; Windows Phone OS 7.5; Trident/5.0; IEMobile/9.0; HTC; Titan)",
        "Mozilla/5.0 (BlackBerry; U; BlackBerry 9800; en) AppleWebKit/534.1+ (KHTML, like Gecko) Version/6.0.0.337 Mobile Safari/534.1+",
    ]
```
这样服务器接收到请求信息就来自不同的浏览器，如果对于需要登录的网站，我们还要保存登录信息，维持cookie状态等，这里也有相应的框架和工具。
## 爬虫优化
一个能跑通的程序和可用之间还差很多细节，比如文件夹创建，保存目录，抓取提示，文件排序、代码优化等等，python并不是我的常用语言，整个实现过程也是边试边查文档，最后源代码放在[这里](https://github.com/chayangge/python-crawler)
## 使用方法
下载源码后，安装相应的包，执行脚本+微信id 即可，然后就就可以看到终端显示的一条条住区进度了。
``` shell
python crawler.py caozsay
```
