---
title: shell
date: 2016-10-30 15:15:51
tags: shell
---
## 常用shell收集
### 查看磁盘信息(du,df)
查看文件大小并排序  -h: --human-readable。--max-depth=1 限定目录
``` shell
du --max-depth=1 -h   
```
<!-- more -->
df查看特殊分区加-a
``` shell
df -h
df -ah 
```

清空文件夹内容，但不删除文件夹，文件夹里执行：
``` shell
rm -rf  *
```
### 磁盘格式化（mkfs：make file system）
linux文件系统格式常用的有：ext2、ext3、vfat。若用ext3格式化/dev/xvdc1分区，执行：
``` shell
mkfs.ext3/dev/xvdc1
```
### 磁盘检测(fsck)
强制(-f)检测/dev/xvdc1分区文件系统是否有错：(-t指定分五的文件类型，-C显示进度条)
``` shell 
fsck -C -f -t ext3
```
检测磁盘坏道(-sv表示在屏幕上显示进度条，但此命令在mac上无效)：
``` shell
badblocks -sv
```

### 简单的文件管理
cd不带参数等同于cd ~  回到主目录
cd .. 上级
cd ../.. 上上级目录
#### 创建目录
mkdir data
mkdir data/app 当前目录data下创建app
mkdir /home/bill/data/app  按绝对路径创建
mkdir -p data/app 加-p表示如果路径不存在data则连父文件夹data一并创建
#### 复制文件 （cp）
cp a.text b.text 当前目录下复制a并命名为b.text
cp a.text ../ 复制a到上层目录
cp a.text ../data/  把当前目录下的a.text复制到上一级目录data目录下
cp a.text ../data/b.text 把当前目录下的a.text复制到上一级目录data目录下,并命名为b
cd -r /data  加-r表示复制data目录及目录下所有文件
#### 移动文件（mv 包含重命名）
重命名： mv a.text b.text
mv 操作同cp类似
#### 删除文件 （rm）
rm a.text 删除当前目录下a.text （mac下无提示）
rm -f a.text 强制删除，无提示
rm -r test 删除test文件夹及内容，不带-r 提示test为目录
rm -rf test 强制删除test文件夹与内容
#### 读文件 （cat more less head tail）
cat a.text b.text 屏幕打印显示a，b（连续打印，会填满屏幕，不人性化）
more 会一屛一屏的输出，用空格键翻页
less 和more类似，但less支持命令模式，输入b上滚一屏，d下滚
head 查看文件头
head -5 a.text 查看a的前五行
tail -5 a.text 查看后五行

### 打包和解压 （tar）
#### 打包 -cf
tar -cf test.zip * 把当期那文件夹下所有文件压缩成test.zip内
tar -cf test.rar a.text b.text 把a，b两个文件打包成test.rar
#### 解压 -xf
tar -xf test.rar 提取文件到当前目录

## 后补充 2017-02-06
### wget
``` shell
wget url/img  //下载网页或图片
wget --mirror chayangge.com // 下载整站的资源
wget -t 5 url  // 中断后重新尝试5次
```
### curl
``` shell
curl chayangge.com //下载输出到终端显示
curl chayangge.com -o text.txt --progress //下载后写入index.txt文件，并显示进度 （--progress）
```
### sed
功能强大的sed结合正则，简直无敌:
把test.txt中的所有aa字符替换（s）为bb
``` shell
sed "s/aa/bb/g" test.txt
```
### 查看、杀死进程
``` shell
ps -ef | grep node  // 找出node进程，kill掉PID
```
mac下：终端运行命令 top 可查看进程，记住前面的PID，杀死进程：
``` shell
kill 进程ID
```
### 查看文件个数
``` shell
ls -l |grep "^-"|wc -l
```
或
``` shell
ls -l |wc -l
```




