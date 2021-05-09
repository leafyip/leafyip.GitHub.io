---
title: redis数据类型
date: 2020-09-05 21:40:34
declare: 1
tags: redis
---
# redis 数据类型

## 1 string
	redis最基本的类型,二进制安全的
## 2 list
	按插入顺序排序的字符串元素的集合.基本上就是链表
## 3 hash
	由field和value都是字符串的键值对组成的集合,类似map
## 4 set
	无序,不重复的字符串集合
<!--more-->
## 5 sortzet
	类似set,但每个字符串元素都会有一个score.里面的元素按照score排序
## 6 geospatial
	地理位置,可计算地址位置信息,两地之间的距离
## 7 bit map 位图
	通过一个bit位来表示某个元素对应的值或状态,其中的key就是元素本身,value对应0或1
	个bit组成一个Byte,所以bitmap本身极大的节省存储空间
## 8 hyperloglog
	用于做基数统计的算法.类似估计一个set中元素数量的概率性数据结构.不存储元素本身