---
title: node源码：请求压缩
date: 2020-07-27 18:33:40
tags: node
---
## zlib

声明zlib压缩的请求收到的是一个压缩包，通过Response Headers里的 **content-encoding: gzip**。浏览器收到后需要解压。

压缩比率：不同的文件要搜比率不同，文本类80k能压缩到20k （能缩小四分之三）



原理：把相同的字符，用更短的字节码替换，所以js，css能够压缩比很高



## brotli （BR）仅HTTPS

请求头支持的压缩算法：Accept-Encoding: gzip, deflate, sdch, br



Tips:brotli 压缩只能在 https 中生效，因为 在 http 请求中 request header 里的 Accept-Encoding: gzip, deflate 是没有 br 的。



