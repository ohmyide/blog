---
title: sublime自定义快捷键
date: 2017-01-05 10:39:42
tags: sublime
---
一直用的sublime3，但其默认没有删除整行快捷键，平时一直用，虽然麻烦，但也习惯了，最近搞新东西，来回注释了很多代码，这才想起：该配置一个删除整行的快捷键了。
<!-- more -->
![步骤](http://7xprui.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-01-05%20%E4%B8%8A%E5%8D%8811.02.40.png)
菜单：Preferences -> Key Blinding，进入后在右侧User自定义面板，会看到一个空数组，自己设定的快捷键和对应的命令以key：value对象的形式保存为数组元素：
``` javascript
[
	{ "keys": ["command+d"], "command": "run_macro_file", "args": {"file": "res://Packages/Default/Delete Line.sublime-macro"} }
]

```
这里把删除整行命令定义为`command+d`，如果还要自定义其他命令，继续往数组里添加命令对象即可，更多命令可参考[官方文档](http://docs.sublimetext.info/en/latest/reference/commands.html)。
保存，关闭后，回到代码编辑页，`command+d`立马生效，早该这么干，一次配置终生有效，省时省力。