---
layout:     post
date:       2020-02-12
categories:	
- 架构
tags:
    - docker
---

> 关于docker使用，每个image可以去docker官网查看使用说明，包括环境变量等配置。
>例如https://hub.docker.com/_/zookeeper/。 
> docker-compose安装https://docs.docker.com/compose/install/

## 1.查看网络
```shell
docker network ls
docker network inspect [网络id]
```