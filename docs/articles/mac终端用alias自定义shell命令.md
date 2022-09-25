---
title: mac终端用alias自定义shell命令
date: 2016-07-18 15:29:36
tags: shell
---
项目尾期，频繁登录测试环境发布测试版本，每次登录`ssh root@123.XX.XX.XX`异常繁杂，稍有不慎记错ip地址，造成不必要的麻烦，于是用alias自定义一个登录命令，一步到位（我用的shell是Oh My ZSH）：
<!-- more -->
第一步：
``` shell
vi ~/.zshrc  或   vi ~/.bashrc
```

第二步：

使用编辑（e），输入自定义的命令：
``` shell
alias gotest='ssh root@123.XX.XX.XX'
```
输入完后，ESC结束编辑，:wq保存退出

第三步：
``` shell
source ~/.zshrc
```
更新修改后的文件配置。

最后享受一下输入gotest的一步到位。