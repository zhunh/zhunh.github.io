---
layout:     post
title:      Centos 配置 python2.7
subtitle:   笔记
date:       2020-02-12
categories:	
- Linux运维
tags:
    - 运维
    - Linux
---

## 1.下载指定版本包

```shell
wget http://python.org/ftp/python/2.7.14/Python-2.7.14.tar.xz
```
<!--more-->

## 2.解压

```shell
tar xf Python-2.7.14.tar.xz
```
## 3.编译

```shell
cd Python-2.7.14
./configure --prefix=/usr/local/puthon2  --enable-shared --optimazation
make && make altinstall
```
## 4.建立软连接

```shell
ln -sf /usr/local/python2/bin/python2.7 /usr/bin/python
```
PS：删除软连接
```shell
rm /usr/bin/python
```
## 5.运行

```shell
python
```
出错:
error while loading shared libraries: libpython2.7.so.1.0。。。
解决，编辑 /etc/ld.so.conf 文件，将安装路径的lib目录加入：

```shell
vim /etc/ld.so.conf 
```
这里应加入 /usr/local/lib
再次运行

```shell
python
```
成功

题外，npm install 构建项目的时候，提示 python2.7 命令不可用，原因是前面软连接为python命令，此时删除原来软连接，建立python2.7 软连接命令
```shell
ln -sf /usr/local/python2/bin/python2.7 /usr/bin/python2.7
```

