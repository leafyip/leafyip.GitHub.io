---
title: mysql支持的索引类型
date: 2020-09-28 20:46:26
tags: mysql
---
# 1 B+tree
B+TREE的演化.
推荐一个在线数据结构模拟的网站 
https://www.cs.usfca.edu/~galles/visualization/Algorithms.html

(1)二叉查找树(binarysearchtree)
特征
1 左边的节点小于根节点
2 右边的节点大于根节点
3 左，右子树也是二叉查找树
4 没有健值相等的节点

<!--more-->
存在的问题:极端情况下性能会退化成链表
![Alt text](/images/binary_search_tree.png)  


(2)自平衡二叉查找树(AVL tree)
1 本身是二叉查找树
2 平衡因子的绝对值最多为1（平衡因子=右子树的高度-左子树的高度）

![Alt text](/images/avl_tree.png) 

(3)多路平衡二叉查找树(B-Tree)
B-Tree 是为磁盘等外部存储设计的一种平衡树.
页是计算机管理存储的逻辑块,硬件和操作系统会将主存以及磁盘分割成连续的,大小相等的块,每个存储块称之为页.

在linux操作系统中,每个存储块(页)最少是4K.

InnoDB存储引擎磁盘管理最少单位也是页,可通过参数innodb_page_size来设置,mysql 8.0 innodb_page_size为16K.


B-Tree数据结构可以让系统快速定位到磁盘块.为描述B-Tree，首先定义一条记录为一个二元组[key, data] ，key为记录的键值，对应表中的主键值，data为一行记录中除主键外的数据。对于不同的记录，key值互不相同
B-Tree中的每个节点根据实际情况可以包含大量的关键字信息和分支，如下图所示为一个3阶的B-Tree：
![Alt text](/images/b_tree.png) 

模拟查找关键字29的过程：

根据根节点找到页1,读入内存.[磁盘I/O操作第1次]
比较关键字29在区间(17,35)找到页1的指针P2.
根据P2指针找到页3，读入内存.[磁盘I/O操作第2次]
比较关键字29在区间(26,30),找到页3的指针P2。
根据P2指针找到页8,读入内存.[磁盘I/O操作第3次]
在页8中的关键字列表中找到关键字29.

B-Tree 的一个节点(16K) 等于4个磁盘块.也就是说读取一个节点需要一次磁盘IO,这次磁盘IO需要读取4个磁盘块.

从计算机的局部性原理
1 当一个数据被用到时，其附近的数据也通常会马上被使用
2 程序运行期间所需要的数据通常比较集中,由于磁盘顺序读取的效率很高（不需要寻道时间，只需很少的旋转时间)

可得,磁盘会有预读操作,即磁盘读取完所需数据后,会按顺序读取多一部分数据到内存中.

磁盘的顺序读取比随机读取要快很多,因为不需要寻道时间,只需要很少的旋转时间.


B-Tree存在的问题
每个节点的空间都是有限的,存储了data后,存储key的数量就会变少.所以当数据库存储的记录过多时,会导致树的深度变高,增加磁盘的IO.

(4)B+Tree
B+Tree 对BTree的优化主要在以下.
1 子节点只存储key,不存储data,这样每个节点都可以存储更多的key,减少树的高度,减少磁盘IO.
2 所有叶子节点之间都有指针,提高区间访问性能(即范围查询),和避免文件排序(file sort)
3 数据记录都放在叶子节点上

如图
![Alt text](/images/b+tree.png) 

一个高度为3的的B+Tree ,每行的数据大小在1K左右,那么可以算出最大存储记录

innodb_page_size 为 16K , 那么一个叶子节点能存储的数量为 16 K /1K =16
每行记录的主键为 bigint 占用8个字节,innodb 指针的大小是6个字节
那每一个非叶子节点存储的数量 为16 *1024  / (8 + 6 ) = 1170
树的高度为3,那么最大的存储数量约等于 1170 * 1170 *16 = 2190 2400  

PS 
1 页的16K 不完全用来存放数据,还会存放
2 B+tree的根节点常驻内存,可以减少一次IO.

B+Tree 支持的查询类型

1全值匹配
如 column_a = 'zhangsan'

2匹配最左前缀
比如多列索引 (column_b,column_c)
select * from t_table where column_b = 'xxx'
只查询 column_b 也会使用索引

3匹配列前缀
如 column_a like 'zhang%'

注 : 如果%在列前面,是无法使用索引.
以下语句会全表扫描
select * from t_user where column_a like '%san'

4匹配范围值
如 age > 18 and age <20

# 2 hash(哈希)
mysql目前只有Memory引擎显示支持hash索引。
InnoDb不显示支持hash索引，但如果某些索引值被使用得非常频繁，InnoDb会在内存中基于B-tree上建立hash索引。

hash索引基于哈希表实现。存储引擎对数据列的值进行hash，得到hash码，同时将该数据行的指针保存到哈希表中
如图 2323，2458，7437，8784 即为hash码
![Alt text](/images/slot--hash.png)  

当发生hash碰撞时，哈希表的value会转换成链表形式

使用hash一些注意事项
a. hash索引只保存数据的hash值，而不保存数据，所以无法用于排序

b. hash索引只能精确匹配，如 = ，!= ，in 。 不支持范围查询，如 age > 18

c. 多列hash索引不支持部分列查询。

如存在多列索引 (A,B) ， 只查询A 列是无法使用hash索引的

# 3 fulltext(全文索引)

mysql 5.6版本之前，只有Mysiam支持全文索引.
5.6以后,InnoDb和Mysiam都支持.
全文索引只支持数据类型为char,varchar,text系列



# 4 R-Tree 空间索引
Myisam表支持空间索引，可用于存储地理位置数据。

