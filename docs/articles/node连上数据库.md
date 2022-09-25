---
title: node与mysql
date: 2017-03-16 11:26:52
tags: node
desc: node使用mysql数据库
---
一直以来都在混迹于『前端』，虽然使用node服务器，但也是用node请求java核心接口，node后台接口只做一些边边角角的逻辑，如上传，图片识别等第三放SDK，所以node对数据库的直接操作经验为零，连sql语句都忘得差不多了，这次就做了个小demo，破除一下对未知的神秘感，[源码在这](https://github.com/chayangge/node-express-mysql)。
<!-- more -->
## 安装mysql
两种安装方式，常规安装包下一步和brew安装，我自豪的采用brew
``` shell
brew install mysql
```
这里有坑：安装完成后终端运行启动mysql命令无效，不给错误样式了，网上找了些处理方法，运行了几个命令就搞定了，具体哪个发挥了直接效果也说不好了，不太好复现问题，解决方法是（设置 MySQL 用户以及数据存放地址）：
``` shell
brew link --overwrite mysql
unset TMPDIR
mysql_install_db --verbose --user=root
```
启动mysql：
``` sql
mysql.server start  // 其他命令： {start|stop|restart|reload|force-reload|status}
```
用户名密码登录mysql：
``` sql
mysql -u root -p 
```
你也可以更改新密码：
``` sql
mysqladmin -u root password
```
## 熟悉的sql命令
登录成功后，进入mysql命令行状态，就可以操作数据库了：
查看数据库：
``` sql
show databases;
```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+

新建数据库：
``` sql
create database myDB;
```
myDB数据库中新建数据表user：
``` sql
use myDB;
create table user (ID int AUTO_INCREMENT PRIMARY KEY,NAME varchar(20) not null,AGE int not null);
```
查看数据表：
``` sql
show tables;
```
插入数据：
``` sql
insert into user values(null,"jim",33);
```
查询：
``` sql
select * from user;
```
+----+------+------+
| id | name | age  |
+----+------+------+
|  1 | jimm |   33 |
+----+------+------+
好，至此数据库这边已经完成，下一步开始node。
## 假设你已安装node和express
还是用[Express应用生成器](http://expressjs.com/en/starter/generator.html)来创建一个完整的供成目录。

和其他平台语言一样，在工程中创建mysql通用配置文件conf.js：
``` javascript
module.exports = {
    mysql: {
        host: '127.0.0.1',
        user: 'root',
        password: '你的密码',
        database: 'myDB',
        port: 3306
    }
};
```
其次为了方便还应该创建一个增删改查的sql语言对象：
``` javascript
module.exports = {
    insert: 'INSERT INTO user(id, name, age) VALUES(0,?,?)',
    update: 'update user set name=?, age=? where id=?',
    delete: 'delete from user where id=?',
    queryById: 'select * from user where id=?',
    queryAll: 'select * from user'
};
```
以后需要连接数据库只需require这两个文件即可，下面开始创建数据库连接，使用的npm包是mysql：
``` shell
npm install mysql
```
安装完毕可以建立数据库连接池了：
``` javascript
require("mysql");
require("conf.js");
require("sqlMap.js");
// ...
var pool = mysql.createPool(conf.mysql);
pool.getConnection(function(err, connection) {
    connection.query("select * from user", function(error, result) {
        res.send(result);
    });
});
```
这样即可获取到数据库中的数据。
