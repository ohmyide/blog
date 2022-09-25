---
title: 公钥免密码ssh登录阿里云服务器
date: 2016-12-23 12:33:04
tags: shell
---
从终端ssh登录阿里云服务器每次都要输入服务器ip后再输密码，很麻烦：
``` shell
ssh root@101.201.XX.XX
root@101.201.XX.XX's password: // 再输入密码
```
登陆一次还有时间限制，过会没动静就掉线了，一天来回要登录好几次。
<!-- more -->
## 解决方法：上传公钥
把自己的公钥上传到服务器上即可，如果还没有公钥，请百度、谷歌查阅生成，这里不做介绍。
有了公钥后即可使用一下命令上传至服务器：
``` shell
ssh-copy-id root@101.201.XX.XX
```
## 背后原理
上传成功后，下次登录时，服务器会先发出一段随机字符码，我方收到后用自己的私钥加密，注意是私钥加密，然后把加密后的码发给服务器，服务器用我方之前上传的公钥进行解密，解密成功则说明身份无误，这样就省去了输入密码这一步，设计的很机智啊。
## 成功提示
上传成功后，输入以下密码，会看到以下提示：
``` html
Number of key(s) added:        1

Now try logging into the machine, with:   "ssh 'root@101.201.XX.XX'"
and check to make sure that only the key(s) you wanted were added.
```
## 意外报错与解决方法
在我的mac上运行`ssh-copy-id`上传公钥命令后报错：
```
zsh: command not found: ssh-copy-id
```
原因是mac上没有安装`ssh-copy-id`，执行以下命令安装：
Homebrew安装：
``` shell
brew install ssh-copy-id
```
什么？你还没有安装Homebrew？用：
``` shell
curl -L https://raw.githubusercontent.com/beautifulcode/ssh-copy-id-for-OSX/master/install.sh | sh
```
安装完成后运行上传公钥命令即可，此时再ssh登录就不用输入密码了。
## ssh登录依然麻烦？还有招

每次登录输入`ssh root@101.201.XX.XX`还是不够简洁，那就用alias自定义一个命令吧，比如定义为：`goaliyun`，具体步骤在这一篇博客：[用alias自定义shell命令](http://chayangge.com/2016/07/18/mac%E7%BB%88%E7%AB%AF%E7%94%A8alias%E8%87%AA%E5%AE%9A%E4%B9%89shell%E5%91%BD%E4%BB%A4/)

---

## 补充：coding.net上免用户名和密码更新代码
运行：
``` shell
git config remote.origin.url https://username:password@git.coding.net/XX/XX.git
```