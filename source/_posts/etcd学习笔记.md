---
layout:     post
date:       2019-10-17
categories:	
- 后端
tags:
		- 后端
		- 架构
---

## 1.what？
- etcd是CoreOS团队2013年6月发起的开源项目。
- 它是一个强一致性的分布式键值数据库，基于go语言实现。
- 用来存储需要由分布式系统或机器集群访问的数据。
- 一个应用是将数据库连接详细信息或功能标志作为键值对存储在etcd中，并watch这些值，以便值改变时应用程序可以自行重新配置。
<!--more-->
## 2.why

## 3.how

### 3.1 安装
各系统二进制文件下载[地址](https://github.com/etcd-io/etcd/releases)
以macOS为例，下载下来的可执行二进制文件有etcd和etcdctl两个，复制到 /usr/local/bin 目录下即可。

### 3.2 命令行使用
```shell
$ etcdctl put foo 45  #写
OK
$ etcdctl get foo			#读
foo
45
$ etcdctl put foo/ty 45
OK
$ etcdctl del foo/ty 	#删除
1
$ etcdctl put foo 99
OK
$ etcdctl get --prefix foo 
foo
99
$ etcdctl get --prefix --rev=4 foo  #获取最近4个版本的值
foo
45
foo/ty
45
$ etcdctl watch foo 	#监听变化，在另一个终端操作foo，变化会反映到这个监听的终端
```

### 3.3工具库
etcd提供了的go操作库https://github.com/etcd-io/etcd/blob/master/clientv3
demo：

```go
package main

import (
	"context"
	"fmt"
	"go.etcd.io/etcd/clientv3"
	"time"
)

func main()  {
	//1.获取client
	cli, err := clientv3.New(clientv3.Config{
		Endpoints:[]string{"localhost:2379"},
		DialTimeout: 5 * time.Second,
	})

	if err != nil{
		fmt.Println("conn err: ", err)
		return
	}
	defer cli.Close()

	//kv 为client包装过的高级CRUD操作对象
	kv := clientv3.NewKV(cli)

	ctx, cancel := context.WithTimeout(context.Background(), time.Second * 5)

	//2.put
	resp, err := kv.Put(ctx, "clientv3_key", "0507")
	//cancel()

	if err != nil{
		fmt.Println("put err: ", err)
	}
	fmt.Println(resp)

	//3.get
	getRsp, err := kv.Get(ctx, "clientv3_key")
	if err != nil {
		fmt.Println("get err:", err)
		return
	}
	fmt.Println(getRsp)
	//4.watch
	rch := cli.Watch(context.Background(), "foo")
	for wresp := range rch {
		for _, ev := range wresp.Events {
			fmt.Printf("%s %q : %q\n", ev.Type, ev.Kv.Key, ev.Kv.Value)
		}
	}
	cancel()
}
```