---
title: POST提交数据的常见编码方式
date: 2016-12-01 13:01:45
tags: http
---
一个完整的请求包括状态行，请求头，消息主体。而请求要发送的内容包含在消息主体中，本文要说明的是请求内容作为消息体的编码方式。
这里需要声明的是：
在postman中看到的以下四种：form-data，x-www-form-urlencoded，raw，binary并不是这四种方式，真实的提交数据格式分别为：multipart/form-data，application/x-www-form-urlencoded，application/json，text/xml。
<!-- more -->
![注意！并不对应postman中的这四种](http://7xprui.com1.z0.glb.clouddn.com/post8.png)
先说说这四种数据格式：
## application/x-www-form-urlencoded
这种格式最为常见，平时使用的ajax提交就是这种方式。对应postman上的x-www-form-urlencoded选项，请求头中Content-Type为`application/x-www-form-urlencoded;charset=utf-8`，其中请求的数据被编码为`key=value&key=value`的形式（没错，类似浏览器地址栏中的get请求参数拼接方式），其具体值是编码好的：
![x-www-form-urlencoded](http://7xprui.com1.z0.glb.clouddn.com/post2.png)
如上图所示，Form Data中就是要提交的数据样式，可点击旁边的view parsed和view source切换查看。
## multipart/form-data
当表单提交内容中包含文件时（比如图片文件），则需要这种数据格式。通常要设置表单`enctype="multipart/form-data"`：
![multipart/form-data](http://7xprui.com1.z0.glb.clouddn.com/post4.png)
上图中Content-Type为`multipart/form-data; boundary=----WebKitFormBoundary0KNaeL3c0TwjXv6c` 其中以`boundary`作为间隔来分割表单字段。
**注意：** 要在postman中提交到文件的请求，要选择`form-data`，而之所平时提交常规表单也能用`form-data`，是因为`multipart/form-data`即可上传文件也可上传键值对。在postman中选择后，postman自行设定了请求的Content-Type。
以上两种是使用最多的形式，满足了绝大多数的请求数据样式。
## application/json 
application/json 提交的是序列化后的json数据，目前还没使用过，待日后补充，但`application/json; charset=utf-8`作为response响应的头信息经常看到，毕竟返回json型数据的请求最为常见。
## postman中的后两项：raw，binary
通过字面意思可看出raw代指原始数据，binary代指二进制数据。
### raw
在postman中选择raw后出现一个大的输入框，在里面输入什么则提交什么。可以是text,json,xml,html等。
![raw](http://7xprui.com1.z0.glb.clouddn.com/post5.png)
### binary
binary专门用来提交文件的二进制数据，且一次只能提交一个文件。

---
参考链接：
- [http://www.ietf.org/rfc/rfc1867.txt](http://www.ietf.org/rfc/rfc1867.txt)
- [https://imququ.com/post/four-ways-to-post-data-in-http.html#simple_thread](https://imququ.com/post/four-ways-to-post-data-in-http.html#simple_thread)
- [http://blog.csdn.net/wangjun5159/article/details/47781443](http://blog.csdn.net/wangjun5159/article/details/47781443)
