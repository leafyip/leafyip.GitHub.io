---
title: mysql事务特性(ACID)
date: 2020-09-21 20:43:52
tags:  mysql
---


## 1原子性（atomicticy） 
事务内的所有操作是原子性的，要么都执行，要么都不执行
## 2一致性（consistency） 
事务执行完毕后，数据库从一个一致性状态到另一个一致性状态
## 3隔离性（isolation） 
事务之间是互相隔离的
## 4持久性 （durability）
事务一旦提交后，对数据库的改变是永久的

除非显式开启事务，否则mysql事务是自动提交（auto-commit）的。

<!--more-->

# mysql 事务隔离级别

## 1 读未提交（read-uncommited）
存在读脏数据，不可重复读，幻读的问题
## 2 已提交读（read-commited） 
解决读脏数据，但仍会出现不可重复度，幻读的问题
## 3 可重复读 （repeatable-read）
解决读脏数据，不可重复读问题，但仍会出现幻读。
## 4 串行化 （serializable）
所有事务串行化执行，解决读脏数据，不可重复读，幻读问题，但性能低。

mysql默认隔离级别是可重复读