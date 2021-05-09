---
title: 安装docker
date: 2020-09-10 21:07:24
tags: docker
---

# 1下载Centos 64位 docker离线安装包
<https://download.docker.com/linux/static/stable/x86_64/>

本文下载的是 docker-18.06.3-ce.tgz 

# 2 解压安装
## 2.1 解压
    tar -zxvf docker-18.06.3-ce.tgz

## 2.2 将解压出来的docker文件夹复制到/usr/bin目录下
    cp docker/* /usr/bin/

## 2.3 注册docker服务
    vi /etc/systemd/system/docker.service
<!--more-->
docker.service文件内容如下  

```
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
  
[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd --selinux-enabled=false --insecure-registry=127.0.0.1
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
  
[Install]
WantedBy=multi-user.target
```

## 2.4给docker.service文件添加执行权限
    chmod +x /etc/systemd/system/docker.service    

## 2.5重新加载配置文件
    systemctl daemon-reload    


# 3启动docker测试

## 启动
    systemctl start docker    


## 设置开机启动
    systemctl enable docker.service    


## 查看docker服务状态
    systemctl status docker    

显示有active (running) 代表 安装成功
![Alt text](/images/docker-status-running.png)


## docker设置官方加速镜像源地址以及账户密码
	
    sudo vim /etc/docker/daemon.json	
	
### 添加如下内容
	{
    "registry-mirrors": ["https://registry.docker-cn.com"]
	}
### 然后重启docker  
    docker service restart


去 https://hub.docker.com/ 注册账户,然后登陆
	
    docker login -u username -p password    
