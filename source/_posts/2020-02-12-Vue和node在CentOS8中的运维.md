---
layout:     post
title:      Vue和node在CentOS8中的运维
subtitle:   笔记
date:       2020-02-12
categories:	
- Linux运维
tags:
    - 运维
    - Linux
---

## 1.Nodejs后台
使用pm2管理node进程

<!--more-->

```shell
启动进程/应用 pm2 start bin/www 或 pm2 start app.js
结束进程/应用 pm2 stop www
```
参考：https://www.jianshu.com/p/e15fd72727fe

## 2.Vue前端

```shell
nohup npm run serve &
关闭  ps -ef | grep serve
kill 进程id
```

## 3.nginx
安装
```shell
https://sourceforge.net/projects/pcre/files/pcre/8.43/pcre-8.43.tar.gz/
wget http://nginx.org/download/nginx-1.15.4.tar.gz
​```shell
版本：
./nginx -v
查看 nginx 进程
ps -ef | grep nginx
关闭：
./nginx -s stop
启动：
./nginx
重新加载配置文件（不需要重启服务器）
./nginx -s reload
检测配置文件是否有语法错误，然后退出
nginx -t 
```
配置文件
```txt
server{
	listen 80;
	server_name 172.26.214.151;
	location ~ /edu/ {
		proxy_pass http://127.0.0.1:8080;
	}
	location ~ /hag/ {
		proxy_pass http://127.0.0.1:3000;
	}
}
```

## 4.使用配置nginx代理前端项目
每次更新代码之后执行命令`npm run build`，将前端项目打包到 dist 文件夹，将该文件夹的路径配置到 nginx 代理。

