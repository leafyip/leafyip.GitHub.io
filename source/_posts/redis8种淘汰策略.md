---
title: redis8种淘汰策略
date: 2020-09-03 20:34:17
tags: redis
---
  

# redis键的淘汰策略

## 1 noevication 
不删除，内存过大直接报错
## 2 allkeys-random
针对所有的key随机删除
## 3 volatile-random
针对过期的key随机删除
<!--more-->
## 4 allkeys-lru
针对所有的key 实行LRU（最近最少未使用）
## 5 volatile-lru
针对过期的key实行LRU
## 6 volatile-ttl
删除过期剩余时间最短的key（随机挑选几个设置了ttl的key，删除最快过期的）
## 7 allkeys-lfu
针对所有的key执行LFU (4.0以上版本可用)
## 8 volatile-lfu
针对设置了过期时间的key执行LFU (4.0版本以上可用)