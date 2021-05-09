---
title: docker常用命令
date: 2020-09-13 20:18:46
tags: docker
---
# 1环境信息

## 1.1查看详细信息 
    docker info    

## 1.2查看版本信息
    docker version

# 2镜像相关
## 2.1登录到远程仓库
	docker login -u username -p password

## 2.2 搜索镜像
	docker search mysql
<!--more-->
## 2.3拉取镜像
	docker pull mysql
默认使用最新的版本(即docker pull mysql:latest)

## 2.4列出本地镜像
	docker images

## 2.5 删除本地镜像
	docker rmi mysql:5.7

## 2.6  指定Dockerfile创建镜像
	docker build -f /path/Dockerfile

## 2.7 将指定镜像保存成tar镜像归档文件
	docker save mysql.tar mysql:5.7
将mysql5.7镜像保存成mysql.tar文件

## 2.7 加载tar镜像
	docker load -i mysql.tar


# 3容器相关
## 3.1列出正在运行的容器
    docker ps    
如果要列出所有容器 docker ps -a

## 3.2 docker inspect [imageId/containerId]
列出镜像或者容器的元数据

## 3.3 docker logs containerId
	查看容器的日志

## 3.4 将容器保存成tar镜像归档文件
	docker export -o mysql_test.tar containerId

## 3.5 从镜像归档tar文件创建镜像
	docker import mysql_test.tar mysql:5.7

注意
- docker save(操作对象是镜像) 出来的文件只能用load加载.
- docker save 保存完整记录,load时无法指定标签等元数据信息.

- docker export(操作对象是容器) 出来的文件只能用import加载.
- docker export 导出的容器快照文件,会丢失所有的历史记录以及元数据信息,import时可以指定标签等元数据信息

# 4 容器生命周期管理
## 4.1 运行
	docker run containerId
可选参数
-a stdin: 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；

-d: 后台运行容器，并返回容器ID；

-i: 以交互模式运行容器，通常与 -t 同时使用；

-P: 随机端口映射，容器内部端口随机映射到主机的端口

-p: 指定端口映射，格式为：主机(宿主)端口:容器端口

-t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；

--name="nginx-lb": 为容器指定一个名称；

--dns 8.8.8.8: 指定容器使用的DNS服务器，默认和宿主一致；

--dns-search example.com: 指定容器DNS搜索域名，默认和宿主一致；

-h "mars": 指定容器的hostname；

-e username="ritchie": 设置环境变量；

--env-file=[]: 从指定文件读入环境变量；

--cpuset="0-2" or --cpuset="0,1,2": 绑定容器到指定CPU运行；

-m :设置容器使用内存最大值；

--net="bridge": 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；

--link=[]: 添加链接到另一个容器；

--expose=[]: 开放一个端口或一组端口；

--volume , -v: 绑定一个卷

## 4.2 启动/停止/重启
	docker start/stop/restart containerId

## 4.3 进入到容器中
	docker exec -it containerId /bin/sh

## 4.4 删除容器
	docker rmi containerId

## 4.4 