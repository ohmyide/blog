---
title: vue父子组件通信
date: 2016-11-30 17:07:23
tags: vue
---
## 父子通信
父组件需要把数据传给子组件的时候，通过props，常见的场景：父组件产生的提示信息，需要子组件拿去显示：
<!-- more -->
``` html
<!-- 子组件代码 -->
Vue.component('error-msg', {
    props: ['error'],
    template: __inline("./error_msg.vue")
})

<div class="form-error">
    <p class="weui_toast_content">{{error}}</p>
</div>

```
这样父组件更新error值后，就能传递给子组件，子组件用来显示。
但props默认是单向的，虽然在老版本vue中可以用.sync进项双向通信，但这并不推荐，在2.0中已经废弃，因为所有子组件都是通过props读取父组件的，如果采用了双向通信，一个子组件改变了父组件的值，那么父组件将影响所有使用此数值的子组件，所以子父通信有专门的通信方式。

## 子组件传值给父组件
vue老版本中用`$dispatch`，子组件传递把需要向父组件传递的值通过`$dispatch`自定义一个事件，并传值，然而在新版中`$dispatch`也要废弃了。vue2.0中可以使用`$emit`来向父组件派发事件，父组件中用`$on`来监听自定义的事件，监听后父组件执行具体的方法，且通过事件传递给父组件的数据，以该方法的参数形式传入，父组件在所要执行的函数中得以使用。
``` javascript
<!-- 子组件test -->
var data = {
	a:11,
	b:22
}
this.$emit("senddata", data);
```
父组件：
``` html
<test v-on:senddata="getData"></test>

methods: {
        getData:function(data){
            console.log("子组件传递的数据：",data);
        }
    }
```
这样既可在vue中实现基本的父子组件的通信。

## 子组件相互通信
通信过程：子->父->子，子组件先通信给父组件，父组件再通过`$broadcast`派发给子组件。
## Vuex统一管理
