---
title: webpack入门教程
date: 2016-09-20 10:46:39
tags: webpack
---
之前项目一直用百度的fis，功能非常齐全，对于最近比较流行的`webpack`还比较陌生，体验一下前段工程化的不同之处，目前貌似使用`Vue`框架的都在使用`webpack`，很多项目基本都是这么模式：`Vue`+`webpack`+`ES6`。
<!-- more -->
## 全局安装webpack
``` shell
npm install webpack -g
```
安装后可用`webpack -h`命令，执行查看webpack提供的所有命令
``` shell
webpack -h
```
## 新建项目
新建一个项目文件夹testwebpack，终端cd到该目录执行初始化项目命令：
``` shell
npm init -y
```
带上`-y`表示yes，省去了一步步的询问步骤，直接初始化，这时空的项目文件夹下会有个package.json文件。
## 在当前项目下安装webpack
执行：
``` shell
npm install webpack --save-dev
```
安装完毕后，此时项目文件夹下多出了`node_modules`文件夹。
## 新建测试用例
1.项目根目录下新建index.html
页面中先引入bundle.js文件，该文件是页面打包后的js
``` html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>webpack入门教程</title>
      <script src="./bundle.js"></script>  
  </head>
  <body>
  test webpack！
  </body>
  </html>
```
2.新建main.js作为入口js
``` javascript
console.log("测试webpack！");
```
## 配置webpack
webpack需要配置文件，配置文件标明了所需js，css等文件的打包和输出，在根目录下新建`webpack.config.js`，内容：
``` javascript
module.exports = {
  entry: './main.js',  // 所需js
  output: {
      path: __dirname, // 输出文件路径
      filename: 'bundle.js' // 输出文件名称  已经在index.html中引用
  }
}
```
## 初次打包
此时执行命令：
``` shell
webpack
```
执行后，项目中自动生成打包后的`bundle.js`文件。如果顺利，项目文件夹下的文件依次为：
.
├── bundle.js
├── index.html
├── main.js
├── node_modules
├── package.json
└── webpack.config.js

如果出什么差错，需要重装node_modules的话，推荐一道一步到位的命令：
``` shell 
rm -rf node_modules && npm cache clean && npm install
```

## 预览：安装webpack-dev-server
执行命令：
``` shell
npm install webpack-dev-server -g
```
安装完server服务后执行：
``` shell
webpack-dev-server
```
此时打开浏览器，输入地址 [http://localhost:8080/webpack-dev-server/](http://localhost:8080/webpack-dev-server/) 即可看到index页面，查看源码即可看到压缩后的js。
