---
title: MongoDB备忘
date: 2018-09-17 14:39:30
tags: MongoDB
---
最近因业务原因，需登录机器手动操作MongoDB，每次操作每次查，现备份一次，方便日后：
<!-- more -->

- 先登录机器
- 链接MongoDB：mongo 10.5.XX.XX:端口
- show dbs
- use 某db
- show tables
- db.sometable.find({'_id':'XX'});
- db.sometable.update({条件},{$set:{key:value}},{multi:true}));

### 模糊查询
```
db.sometable.find({'url':{$regex:/key/i}})
```
### 新建表
```
db.createCollection('表名')
```

### 关闭MongoDB
- 先退出：mongo --port 8415（只用端口号登录）
- use admin
- db.shutdownServer()

### 启动
cd到MongoDB安装文件夹：

```
mongod -f mongod.conf
```

### 复制集
MongoDB数据库有多个节点时，会包含一个Primary节点和多个Secondary节点，数据先写入Primary节点，其他Secondary再从Primary中同步写入的数据，因此当连接数据库后，光标起始位置会显示：
```
某某db:PRIMARY>
```
或：
```
某某db:SECONDARY>
```
当要操作Secondary节点时，由于secondary节点默认不可读，会收到一下错误：
```
"errmsg" : "not master and slaveOk=false",
"code" : 13435,
"codeName" : "NotMasterNoSlaveOk"
```

解决办法，执行：
```
rs.slaveOk();
```
但下次连接是依然报错，若想根治，则：
```
vi ~/.mongorc.js

增加:rs.slaveOk();
```
