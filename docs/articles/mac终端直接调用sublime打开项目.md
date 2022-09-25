---
title: mac终端直接调用sublime打开项目
date: 2016-09-12 16:28:33
tags: shell
---
如果你用mac且喜欢sublime，在开工时往往要在终端cd ,cd 再cd到工程目录，然后再从层层文件夹中找到工程目录，再用sublime打开，费时费力，现在稍微配置一下，就可以在终端切换目录后直接在sublime中打开了：
**前提：本人使用的sublime3，shell为oh-my-zsh**
<!-- more -->
### 第一步：
``` shell
sudo ln -s "/Applications/Sublime\ Text.app/Contents/SharedSupport/bin/subl" /usr/local/bin/subl
```
### 第二步
``` shell
vi ~/.zshrc  或   vi ~/.bashrc
```
### 第三步：进入编辑状态，编辑：
如果使用的是sublime2，则输入：
``` shell
alias subl=\''/Applications/Sublime Text 2.app/Contents/SharedSupport/bin/subl'\'
```
如果使用的是sublime3，则输入：
``` shell
alias subl=\''/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl'\'
```
### 第四步：更新配置
``` shell
source ~/.zshrc
```
最后，在终端cd到工程目录下后，直接：`subl /`即可，sublime直接打开工程项目，爽。当然，如果觉得subl命令不够酷，你也可以自定义。