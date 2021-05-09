---
title: mysql日志类型
date: 2020-09-22 20:15:08
tags:  mysql
---


## 1 事务日志 包括重做日志redo log 和回滚undo log。
redo log : 保证事务的持久性.防止发生故障时，内存中尚有数据未写入磁盘。在mysql重新启动时，会根据redo log还原数据

undo log 保存事务开启时的数据状态，用于事务回滚
## 2 错误日志（error log）
记录启动，运行，停止的错误日志
## 3 通用查询日志（general log）
<!--more-->
记录客户端发送的请求
## 4 二进制日志(bin log)
记录成功修改数据的日志，如insert，update，delete，grant change（更新授权信息）等
## 5 慢查询日志（slow query log）
记录符合慢查询条件的日志
## 6 中继日志（relay log）
记录从主节点拉取的二进制日志，用于主从复制
