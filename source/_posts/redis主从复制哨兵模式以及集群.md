---
title: redis主从复制哨兵模式以及集群
date: 2020-09-21 20:43:52
tags:  redis
---

# 1.redis主从复制
## 1.2 redis支持主从复制,经配置后,可以将主节点的数据复制到多个从节点.
同步策略分为全量同步和增量同步

### 1.2.1全量同步 

	1 slave初始化的时候,会向master 节点发送PSYNC 命令,第一次连接会触发 全量更新(full resnyc)
	2 主节点收到命令后BGSAVE 后台开启进程来保存镜像(可设置保存镜像到磁盘,或者保存到内存), 并记录RDB生成这段时间内客户端发送的命令
	3 从节点收到主节点的RDB,保存到本地磁盘,清空所有旧数据,并加载主节点的RDB文件
	4 主节点发送快照完毕后,向从节点写入缓存区的命令
	5 从节点接收并执行主节点发送的命令
	
### 1.2.2 增量同步 
master 和slave 内部都会维护一个offset,以此来判断当前的数据集是否相同.
slave 还会维护一个 master replication id (如果master 重启后,master replication id 会变化). 
slave 发送psync 命令 会发送 pysnc master replication id  offset等参数,如果master replication id 或者offset 不存在.那么会进行全量更新

主从节点正常工作后都是增量同步.主节点每执行完一条命令都会发送给从节点执行.

注意
	-如果多个slave同时向master进行全量更新,master只会生成一个RDB.
	-master同时向slave发送RDB文件,有可能导致磁盘IO瓶颈.
	-主从模式下，如果主节点宕机，客户端需要手动将连接地址改成slave的地址。
<!--more-->
# 2 redis哨兵(Sentiel)模式

为了解决主从模式下,主节点宕机,客户端需要修改连接地址的问题,redis引入了哨兵模式.
- 哨兵的主要作用是监控主从节点的运行状态以及进行故障转义.
- 哨兵模式是redis高可用的一种方案,建议至少配置3个哨兵服务.
- 哨兵模式下,客户端连接的地址将指向哨兵系统.


## 2.1监控 
	每10秒定时向主节点发送info命令(获取最新的主从节点列表)
	每1秒向所有节点发送ping心跳包(检测节点是否存活).


## 2.2 通知
	被监控的master 或者 slave出现问题,会发送报警通知

## 2.3故障转移
	如果一个sentinel在给定的时间内检测到master没有响应,该sentinel会将master标记为主观下线.
	随后,所有sentinel都会向master节点发送命令,如果超过一定的数量(quorum)sentinel都认定master下线
	那么master会被标记为客观下线.
	master被标记为下线后,sentiel集群内部会使用Raft协议选举出一个 sentiel leader来执行故障转移操作.(转移完后又变成平等关系)
	该sentiel leader会在slave列表中选取一个节点成为新master.如果原来的旧master恢复了服务,它会成为新master的从节点.
	
	选举参考因素 
	(1)与master断开的时间
	(2)配置文件配置的优先级
	(3) 如果2相同,取 replication offset 大
	(4) 如果2 且 3相同 , 则取runid小
	
	sentiel leader 每执行一次主备切换都会生成configuration epoch 作为version号.这个leader 切换失败后,换另一个哨兵继续切换也会生成新的epoch.
	当主备切换成功后,leader会先在本地更新配置,然后通过发布/订阅 让其他的sentiel根据epoch来更新配置
