---
layout:     post
date:       2019-10-17
categories:	
- 机器学习
tags:
    - AI
    - 机器学习
---

## 1.zk客户端操作

### 在zk安装bin目录运行zkCli.sh脚本进入zk客户端
```shell
./bin/zkCli.sh -timeout 5000 -r -server zk1:2181
./bin/zkCli.sh -timeout 5000 -r -server zk2:2181
```
<!--more-->
连接出问题参考http://wenda.chinahadoop.cn/question/2993
1. create命令创建节点，一定要加数据
```sh
#eg：
[zk: localhost:2181(CONNECTED) 5] create /test "hello zookeeper"
Created /test
#在/test节点下增加节点/first
[zk: localhost:2181(CONNECTED) 12] create /test/first "first data"
Created /test/first
#加参数 -e 创建短暂节点，客户端重启后短暂节点消失
[zk: localhost:2181(CONNECTED) 19] create -e /test/second "second data"
Created /test/second
#加参数 -s 创建带序号的节点,序号可以看出节点创建的先后顺序
[zk: localhost:2181(CONNECTED) 1] create -s /test/third "third data"
Created /test/third0000000002
[zk: localhost:2181(CONNECTED) 2] create -s /test/third "third data"
Created /test/third0000000003
[zk: localhost:2181(CONNECTED) 3] 
```
2. ls命令查看节点，若没有子节点，使用get命令查看节点数据
```sh
#eg：
[zk: localhost:2181(CONNECTED) 6] ls /
[test, zookeeper]
[zk: localhost:2181(CONNECTED) 15] ls /test
[first]
[zk: localhost:2181(CONNECTED) 11] get /test
hello zookeeper
#查看test子节点first的数据
[zk: localhost:2181(CONNECTED) 18] get /test/first
first data
```
3. set 命令修改节点的值
```sh
[zk: localhost:2181(CONNECTED) 5] set /test/first "1111"
[zk: localhost:2181(CONNECTED) 6] get /test/first
1111
```
4. watch 监听

## 2.zookeeper集群部署
### 1.docker-compose部署
```yml
version: '3.1'

services:
  zoo1:
    image: zookeeper
    restart: always
    hostname: zoo1
    container_name: zk1
    ports:
      - 2181:2181
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181

  zoo2:
    image: zookeeper
    restart: always
    hostname: zoo2
    container_name: zk2
    ports:
      - 2182:2181
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=0.0.0.0:2888:3888;2181 server.3=zoo3:2888:3888;2181

  zoo3:
    image: zookeeper
    restart: always
    hostname: zoo3
    container_name: zk3
    ports:
      - 2183:2181
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=0.0.0.0:2888:3888;2181
```