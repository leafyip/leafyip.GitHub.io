---
title: minio分布式扩容
date: 2020-12-05 21:39:49
tags: minio
---

查看第一个集群的启动日志

![Alt text](/images/minio_start.png) 



minio暂不支持动态扩容,所以只能停机扩容.

minio扩容后原来的数据并不会迁移,往扩容后的集群添加新数据,会优先往空余分区存放.

<!--more-->

扩容必须的磁盘数量必须 是原有集群的倍数,以便维护相同的数据冗余(SLA).
如上文部署的磁盘是4 * 4 =16个, 那新添加的集群磁盘数量可以是 16 , 32 等.

minio 有一个 zone  set 和 drives的概念.
看启动日志的第一行 
```
formatting 1st zone, 1 set(s) , 16 drives per set
```
说明minio划分了一个节点,一个纠删码集合,每个纠删码集合里有16块硬盘.
如果客户端上传了一个32MB的文件,minio会选择空闲的分区,然后再在zone内选择set,每个set有多少个drives,就会把文件分成多少分.
32MB的文件会分成16份数据块,每份2MB,然后再加上2MB校验块(默认配置下),16个磁盘都保存4MB. 所以一个1MB的文件,在minio集群中实际会占用2MB空间.

minio会根据传入的参数自动划分 zone,set,drives
zone 相当于分区的概念,比如第一个集群只划分了一个分区.
set 纠删码集合,minio官方规定,每个纠删码包含的磁盘(drives)只能是4-16的倍数,所以磁盘的总数量必须是4-16其中一个数字的倍数
drives 磁盘,minio会将传入的目录当成一个磁盘


原来集群的启动方式命令是
```
export MINIO_ACCESS_KEY=admin
export MINIO_SECRET_KEY=AAbb1234
nohup /home/minio/minio server http://192.168.26.13{1...4}/export{1...4} > /home/minio/nohup.out 2>&1 &
```

假设要在原有集群上新添加一个集群,修改脚本
```
export MINIO_ACCESS_KEY=admin
export MINIO_SECRET_KEY=AAbb1234
nohup /home/minio/minio server http://192.168.26.13{1...4}/export{1...4}  http://192.168.26.13{5...8}/export{1...4} > /home/minio/nohup.out 2>&1 &
```

停止原来的集群,然后所有节点执行以上命令即可