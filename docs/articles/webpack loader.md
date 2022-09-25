---
title: webpack loader
date: 2018-12-24 01:36:34
tags:
---

### 先熟悉loader
webpack只能处理js，对于非js的文件需要对应的loader转换成webpack能处理的模块，在进行打包，如JSX、TS、less等。

loader本质上是一个function，且按照node普通模块的方式编写，既可以是本地的一个文件，也可以打成一个npm包（项目中都是npm）。function用参数传入代码，然后按转换代码，最后输出代码。

### loader基本用法
官方是使用方法这里过多介绍，只说明一下自己开发的本地loader使用方式，下面自己实现处理less的两个常用loader，分别为'style-loader', 'less-loader'，后者翻译less，前者把翻译后的css插入前端代码中。

在webpack.config.js中：
```
const path = require('path');
module.exports = {
    entry: './src/index.js', // 入口文件
    output: { // webpack打包后出口文件
        filename: 'build.js', // 打包后js文件名称
        path: path.resolve(__dirname, 'dist') // 打包后自动输出目录
    },
    resolveLoader: {
        // 定义自己的loader位置
        modules: [path.resolve(__dirname, 'loaders')]
    },
    devtool: 'source-map',
    module: {
        // loader 默认从右向左、从下往上执行，可用enforce：‘pre’ normal,inlineLoader,post设定最先执行
        rules: [
            {
                test: /\.less$/,
                use: ['style-loader', 'less-loader']
            }
        ]
    }
};

```
如上所示，用resolveLoader定义了loader的读取位置为本地的loaders文件夹，下面在该文件夹中定义这两个laoder的js文件，先以less-loader为例：
```
function loader (source) {
    // 这里处理source
    console.log('我是自定义loader');
    return source;
}
module.exports = loader;

```
此时运行webpack，会执行此loader并打印:'我是自定义loader。

loader作为一个流水线上的某个处理环节，传入了内容（String或Buffer），处理内容，最后输出内容，且输出作为下一个loader的输入，下面加入处理程序：
```
let less = require('less');
function loader (source) {
    console.log('run loader...222', source);
    console.log('this:', this);
    let css;
    less.render(source, function (error, output) {
        css = output.css;
    });
    console.log('我是翻译后的css:', css);
    return css;
}

module.exports = loader;

```
至此，一个最基本的less文件loader就已经实现。

下面是style-loader，其作用是把css代码按最基础的方式插入页面中，所谓最基础的方式就是把css代码拼到style标签中，单后插入head标签里：
```
function loader (source) {
    let str = `
        let style = document.createElement('style');
        style.innerHTML = ${JSON.stringify(source)}
        document.head.appendChild(style)
    `;
    console.log('我是拼接好的插入代码：', str);
    return str;
}

module.exports = loader;

```

运行webpack --mode development，会看到输出结果，并在dist目录中的未压缩build.js中看到具体的css代码：

```
/***/ './src/index.js':
    /*! **********************!*\
  !*** ./src/index.js ***!
  \**********************/
    /*! no static exports found */
    /***/ function (module, exports, __webpack_require__) {
        __webpack_require__(/*! ./index.less */ './src/index.less');
        document.querySelector('#root').innerHTML = '<h1>Hello World!</h1>';
        /***/ },

    /***/ './src/index.less':
    /*! ************************!*\
  !*** ./src/index.less ***!
  \************************/
    /*! no static exports found */
    /***/ function (module, exports) {
        let style = document.createElement('style');
        style.innerHTML = 'h1 {\n  color: red;\n}\n';
        document.head.appendChild(style);
        /***/ }

/******/ });
```
在浏览器打开，将看到红色的hello word，说明两个loader已经正常运行。

### pitch

每个loader的function都有pitch属性方法，虽然loader的执行方式是从右向左，总下往上，但每个loader的pich是按常规顺序执行的，可以在某个laoder中打印：

```
function loader (source) {
    let str = `
        let style = document.createElement('style');
        style.innerHTML = ${JSON.stringify(source)}
        document.head.appendChild(style)
    `;
    console.log('我是拼接好的插入代码：', str);
    return str;
}
loader.pitch = function () {
    console.log('我是less-loader的pitch，我先执行');
    //return 'source' 一旦return，后续的loader将不执行
};

module.exports = loader;

```
当pitch中有return时，不在执行后面的loader，pitch在内部是以递归方式执行，当递归到底时，再执行loader，类似于koa中间件的洋葱模型，或理解为先入后出的栈模型：
```
|- style-loader `pitch`
  |- less-loader `pitch`
    // 递归到底
  |- less-loader normal execution
|- style-loader normal execution

```

上面所写的两个loader过于简单，未涉及loader中的this，在每一个loader的上下文中都能通过this获取到loader的相关API，应对更多的功能场景，关于更多this上的API见[文档](https://webpack.js.org/api/loaders/)