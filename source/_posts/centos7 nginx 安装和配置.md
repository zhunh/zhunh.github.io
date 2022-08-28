---
layout:     post
date:       2020-02-12
categories:	
- Linux运维
tags:
    - Centos
    - nginx
---

## 1.安装
```shell
yum install nginx
```
安装目录：`/usr/sbin`
配置目录：`/etc/nginx`

<!--more-->

## 2.配置

```shell
vi /etc/nginx/conf.d/my.conf
```



## 3.阿里云

端口无法访问，去阿里云修改实例的安全组，添加相应端口。然后主机开放相应端口。