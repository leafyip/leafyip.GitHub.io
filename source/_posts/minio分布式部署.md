---
title: minio分布式部署
date: 2020-12-03 21:14:06
tags: minio
---


## 1.1 安装前的准备工作
每个节点都要执行以下的所有操作

安装ntpdate,保持时间一致
```
yum install -y ntpdate
ntpdate 0.asia.pool.ntp.org
```


安装结构如下
一共4个节点,每个节点4个硬盘,磁盘目录如下

IP | 磁盘1 | 磁盘2 | 磁盘3 | 磁盘4 
:-: | :-: | :-: | :-: | :-: 
192.168.26.131 | /home/minio/export1 | /home/minio/export2 | /home/minio/export3 | /home/minio/export4 
192.168.26.132 | /home/minio/export1 | /home/minio/export2 | /home/minio/export3 | /home/minio/export4 
192.168.26.133 | /home/minio/export1 | /home/minio/export2 | /home/minio/export3 | /home/minio/export4 
192.168.26.134 | /home/minio/export1 | /home/minio/export2 | /home/minio/export3 | /home/minio/export4 

<!--more-->

创建目录
```
mkdir -p /home/minio/
mkdir -p /home/minio/export1
mkdir -p /home/minio/export2
mkdir -p /home/minio/export3
mkdir -p /home/minio/export4
```

防火墙开放9000端口
```
firewall-cmd --zone=public --add-port=9000/tcp --permanent
```

重启防火墙
```
firewall-cmd --reload
```

查看端口是否成功开放
```
firewall-cmd --list-ports
```

如图则代表成功开放端口
![Alt text](/images/minio_firewall.png) 

## 1.2 下载minio
去官网选择所需的版本下载
```
https://docs.min.io/cn/minio-quickstart-guide.html
```
本文下载的是linux版本
```
https://dl.min.io/server/minio/release/darwin-amd64/minio
```

下载好,将minio复制到/home/minio目录下
此时目录结构如下

![Alt text](/images/minio_1.png) 

修改权限
```
chmod +x /home/minio/minio
```


## 1.3 运行

vi /home/minio/start.sh
设置自启动脚本
```
export MINIO_ACCESS_KEY=admin
export MINIO_SECRET_KEY=AAbb1234
nohup /home/minio/minio server http://192.168.26.13{1...4}/export{1...4} > /home/minio/nohup.out 2>&1 &
```
注意 是三个省略号 ...
假如有n 台服务,m个目录,
那语法糖为
minio server `http://host{1...n}/export{1...m}`

本文的n 和 m 恰好为4而已

`/home/minio/minio http://192.168.26.13{1...4}/export{1...4}`

命令等价于
```
/home/minio/minio server 	   http://192.168.26.131/export1 	http://192.168.26.131/export2 \
								http://192.168.26.131/export3 	http://192.168.26.131/export4 \
								
								http://192.168.26.132/export1 	http://192.168.26.132/export2 \
								http://192.168.26.132/export3 	http://192.168.26.132/export4 \
								
								http://192.168.26.133/export1 	http://192.168.26.133/export2 \
								http://192.168.26.133/export3 	http://192.168.26.133/export4 \
								
								http://192.168.26.131/export4 	http://192.168.26.134/export2 \
								http://192.168.26.131/export4	 http://192.168.26.134/export4 
```

赋予脚本权限
```
chmod +x /home/minio/start.sh
```

启动
```
sh /home/minio/start.sh
```

启动日志如下
![Alt text](/images/minio_start.png) 


随意选择一个ip访问minio的控制台地址
`http://192.168.26.131:9000/minio/login`

![Alt text](/images/minio_login.png) 

如图则代表部署成功


