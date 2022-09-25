---
title: package.json散落知识点
date: 2019-01-02 11:07:16
tags:
---
本文收集散落的package.json相关知识，日后会不断丰富和补充：
<!-- more -->
### 自定义命令和内置命令
我们都知道可以再script中定义命令：

```
"scripts": {
    "start": "gulp",
    "mock-start": "XXX",
    "lc-mock-start": "XXX",
    "test": "npm run lint"
  },
```
但这些命令一些可以
```
npm start
```
有些则需要加run才行：
```
npm run mock-start
```
不需要加run的就是npm内置命令，出start之外还有：
- test
- stop
- restart

### 语义化版本号
约定一个包的版本号包含3个数字：主版本号.小版本号.修订版本号。
- MAJOR 对应大的版本号迭代，做了不兼容旧版的修改时要更新 MAJOR 版本号
- MINOR 对应小版本迭代，发生兼容旧版API的修改或功能更新时，更新MINOR版本号
- PATCH 对应修订版本号，一般针对修复 BUG 的版本号
具体见：[https://semver.org/lang/zh-CN/](https://semver.org/lang/zh-CN/)

### 版本匹配
经常看到入校版本信息：
```
 "dependencies": {
    "copy-to-clipboard": "^3.0.8",
    "immutable": "^3.8.2",
    "react": "~16.3.2",
    "react-dom": "*16.3.2",
  }
```
版本前有^和~两个符号，代表的意思分别为：
- ^ 大版本匹配：如^3.0.8会匹配大版本3下的所有小版本（从3.0.0到3.9.9）
- ~ 小版本匹配：如~3.0.8会匹配小版本3.0.X下的所有小版本(从3.0.0到3.0.9)
- *代表安装最新版本的包

所以不推荐^和*