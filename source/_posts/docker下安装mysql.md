---
title: docker下安装mysql
date: 2020-09-16 20:49:19
tags: docker
---

# 此篇博客仅供学习使用,实际生产环境中 mysql,redis等数据库不建议安装在容器中.
- 详细可查看另一篇博客 {% post_link docker不适合安装数据库的原因 %}


## 1搜索MySQL镜像
    docker search mysql

我在搜索过程中报了如下错误
     Error response from daemon: Get https://index.docker.io/v1/search?q=mysql&n=25: x509: certificate has expired or is not yet valid

<!--more-->

经排查,原因可能有二种  

(1)系统时间不对,执行如下命令更新时间
	
    ntpdate cn.pool.ntp.org
	

(2)证书问题,添加国内加速镜像地址
	
    sudo vim /etc/docker/daemon.json	
	
### 添加如下内容
	{
    "registry-mirrors": ["https://registry.docker-cn.com"]
	}
然后重启docker  

    docker service restart

重启后再次搜索
![Alt text](/images/docker_search_mysql.png)

OFFICIAL 列的值为OK代表此镜像是官方镜像

拉取最新的官方镜像
    docker pull mysql
	

## 2 在宿主机创建持久化文件夹
说明(本次安装的mysql版本为5.7,所以此处的文件夹名称为mysql5.7)
/usr/local/mysqlData/mysql5.7/conf  配置文件夹

/usr/local/mysqlData/mysql5.7/data  数据持久化数据文件夹

/usr/local/mysqlData/mysql5.7/logs  日志文件夹

    
	sudo mkdir -p /usr/local/mysqlData/mysql5.7/conf
	sudo mkdir -p /usr/local/mysqlData/mysql5.7/data
	sudo mkdir -p /usr/local/mysqlData/mysql5.7/logs

### 2.1创建外部挂载配置文件
    
	sudo vi /usr/local/mysqlData/mysql5.7/conf/mysql.cnf
    
### 2.2文件内容如下(此处是mysql5.7的配置文件,如果是其他版本,请自行搜索对应的文件)
    
	# Copyright (c) 2014, 2016, Oracle and/or its affiliates. All rights reserved.
	#
	# This program is free software; you can redistribute it and/or modify
	# it under the terms of the GNU General Public License, version 2.0,
	# as published by the Free Software Foundation.
	#
	# This program is also distributed with certain software (including
	# but not limited to OpenSSL) that is licensed under separate terms,
	# as designated in a particular file or component or in included license
	# documentation.  The authors of MySQL hereby grant you an additional
	# permission to link the program and your derivative works with the
	# separately licensed software that they have included with MySQL.
	#
	# This program is distributed in the hope that it will be useful,
	# but WITHOUT ANY WARRANTY; without even the implied warranty of
	# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
	# GNU General Public License, version 2.0, for more details.
	#
	# You should have received a copy of the GNU General Public License
	# along with this program; if not, write to the Free Software
	# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA
	
	#
	# The MySQL  Server configuration file.
	#
	# For explanations see
	# http://dev.mysql.com/doc/mysql/en/server-system-variables.html
	
	[mysqld]
	pid-file	= /var/run/mysqld/mysqld.pid
	socket		= /var/run/mysqld/mysqld.sock
	datadir		= /var/lib/mysql
	sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
	#log-error	= /var/log/mysql/error.log
	# By default we only accept connections from localhost
	#bind-address	= 127.0.0.1
	# Disabling symbolic-links is recommended to prevent assorted security risks
	symbolic-links=0
	
	
## 3 docker运行mysql镜像(请自行修改对应步骤2的持久化文件夹)
    sudo docker run -itd -p 3306:3306 --name mysql5.7 --privileged=true  -v /usr/local/mysqlData/mysql5.7/conf:/etc/mysql/conf.d -v /usr/local/mysqlData/mysql5.7/data:/var/lib/mysql -v /usr/local/mysqlData/mysql5.7/logs:/logs -e MYSQL_ROOT_PASSWORD=root mysql:5.7

### 3.1 docker 查看运行中的容器
    docker ps     
如图mysql容器已经在运行中

![Alt text](/images/docker_ps.png)



### 3.2 进入mysql 容器中

    docker exec -it 173813af43c4 /bin/sh    

在容器内进入mysql 
	
	mysql -u root -p     

输入密码后即可进入到mysql中
退出mysql exit;

### 3.3 退出容器
如果要正常退出且不关闭容器,请按Ctrl+P+Q进行退出容器

### 3.4 设置mysql开机启动
    docker update mysql5.7 --restart=always    
mysql5.7 为步骤3 --name的容器名