---
layout:     post
title:      Centos8 中 mongodb 运维
date:       2020-02-12
categories:	
- Linux运维
tags:
    - 运维
    - Linux
---

## 1.启动和停止
```shell
systemctl start mongod.service
systemctl stop mongod.service
```

<!--more-->

## 2.查看mongodb的状态：
```shell
systemctl status mongod.service
```
## 3.设置开机启动
```shell
systemctl enable mongod.service
```
## 4.进入mongodb客户端
```shell
mongo
```
## 5.默认路径

```txt
1.日志
/var/log/mongodb/mongod.log
2.配置
/etc/mongod.conf
3.dbPath
/var/lib/mongo
4.pid
/var/run/mongodb/mongod.pid
```

注：centos安装 MongoDB [官方文档](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/)。

## 6.mongodb服务关闭，再启动失败
使用命令 systemctl start mongod.service 启动服务失败
查看log

```txt
Failed to unlink socket file /tmp/mongodb-27017.sock errno:1 Operation not permitted
```
解决方法：
- 删除进程文件pid
```shell
rm -rf /var/run/mongodb/mongod.pid
```
- 删除 .sock文件
```shell
rm -rf /tmp/mongodb-27017.sock
```
然后systemctl start mongod.service 就可以了。
注：相关路径可以在配置文件 /etc/mongod.conf 中查看，上面也有列举

## 7.后台 --fork 运维

```shell
mongod --dbpath /data/mongo/db --logpath /data/mongo/log --fork
```
这样在后台起的服务，关闭流程如下：
```shell
ps aux |grep mongod
```
## 8.用户设置
```shell
添加用户
use admin
db.createUser({
	user: "admin",
	pwd: "123456",
	roles: [ 
		{ role: "userAdminAnyDatabase", db: "admin" } 
	]
})

use jwc 
db.createUser({
	user: "jwc",
	pwd: "123456",
	roles: [
		{ role: "readWrite", db: "jwc" } 
	]
})
```
登录认证
```shell
mongo #进入客户端
use jwc #切换到指定数据库
db.auth("jwc","123456") #授权验证，返回 1 表示验证成功，就可以进行相关数据库操作
```
## 9.数据导出
```shell
mongoexport -h localhost:27017 -d news_zzy -c news -o news.json
-h ：数据库地址，MongoDB 服务器所在的 IP 与 端口，如 localhost:27017
-d ：指明使用的数据库实例，如 news_zzy
-c 指明要导出的集合，如 news
-o 指明要导出的文件名，如 news.json
```
## 10.数据导入
```shell
mongoimport -h localhost:27017 -d news_zzy -c news.json
-h ： 数据库地址，MongoDB 服务器所在的 IP 与 端口，如 localhost:27017
-d ：指明使用的库，指明使用的数据库实例，如 news_zzy
-c ：指明要导入的集合，如 news.json，不存在时会直接创建。
```
