---
title: InnoDB和Myasiam存储引擎的区别
date: 2020-10-03 16:28:05
tags: mysql
---


\ | InnoDB | Mysiam 
:-: | :-: | :-: 
事务和外键 | 支持 | 不支持 
锁的级别 | 行锁,页锁,表锁 | 表锁 
支持索引的类别 | 支持聚簇索引和非聚簇索引 | 仅支持非聚簇索引 
 | 支持Full-Text(全文索引),B-tree,自适应Hash,不支持R-Tree(空间索引) | 支持B-Tree,R-Tree,Full-Text,不支持Hash; 
 | 不支持压缩索引 | 支持压缩索引 
是否维护表记录总数 | 否 | 是 
其他特性 | 插入缓冲,二次写,自适应hash,预读 | - 
