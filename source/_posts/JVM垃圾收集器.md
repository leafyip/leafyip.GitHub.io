---
title: JVM垃圾收集器
date: 2020-10-15 16:34:10
tags: jvm
---

# 1 Serial/Serial Old收集器

Serial是一个单线程的收集器,作用于新生代,使用`复制`算法.

Serial Old 是Serial收集器的老年代版本,使用`标记-整理`算法

Serial/Serial Old只能使用单核单线程来进行垃圾回收,且收集的时候必须暂停其他所有工作线程(Stop The World),这意味着整个应用将无法对外服务.

![Alt text](/images/jvm_serial.png)

安全点(safe point) : 程序执行时并非在所有地方都能停顿下来进行GC,能停下来进行GC的点叫左安全点.

优点 
简单高效,是client模式下默认的垃圾收集器

缺点
垃圾回收速度慢,频繁地stop the world 导致用户体验差

<!--more-->

# 2 ParNew收集器
ParNew 收集器是Serial收集器的多线程版本,只能作用于新生代,同样需要stop the world.

ParNew 收集器在但CPU环境绝对不会比Serial效率更高,甚至可能由于线程交互,该收集器在通过超线程技术实现的两个CPU环境都不能100%保证性能超越Serial.

![Alt text](/images/jvm_parnew.png)

# 3 Parallel Scavenge 收集器
Parallel Scavenge 收集器作用于新生代,使用的是`复制`算法,也是并行的多线程收集器.
Parallel 收集器关注与整个系统的吞吐量,吞吐量 = 运行用户代码时间 / (运行用户代码时间 + 垃圾收集的时间) . 如虚拟机运行了100分钟,垃圾收集用了1分钟,那吞吐量就是99%.

Parallel 提供两个参数用于精确控制吞吐量 ,
最大垃圾收集停顿时间 (-XX: MaxGCPauseMills)
设置吞吐量大小(-XX: GCTimeRTatio)


# 4 Parallel Old 收集器
Parallel Old 是 Parallel Scavenge 收集器的老年代版本,使用多线程的`标记-整理`
在注重系统吞吐量的即CPU资源敏感的场合,可以优先考虑 Parallel Scavenge 加 Parallel Old 的组合

![Alt text](/images/jvm_parallel_scanvege.png)

# 5 CMS(Concurrent Mark Sweep)收集器
CMS 收集器是一种以获取最短回收停顿时间为目标的收集器.针对老年代,基于`标记-清除`算法.
CMS 符合注重服务响应速度,希望系统停顿时间最短,以带给用户良好的体验的应用.

CMS 主要包含以下7个步骤
(1) 初始标记(init mark) : 需要stop the world,但由于只是标记以下GC Root 能直接关联的对象,速度很快.
(2) 并发标记(concurrent mark) : 从GC root寻找对象的引用链.与用户线程一起运行
(3) 并发标记预清理(concurrent preclean): 并发执行,会查找第二阶段中从新生代分配或者晋升的对象,并发标记预清理阶段可以减少 `重新标记` stop the world 的工作量和时间
(4) 并发可中止的预清理()concurrent abortable preclean) 和第三阶段做的东西一样,目的是减少 `重新标记`的工作量.并发可中止的预清理 阶段可以控制这一阶段的结束时机,例如扫描的时间(默认5秒),或者Eden内存使用率达到期望比例(默认50%)
(5) 重新标记(mark) : 需要stop the world .重新标记阶段是为了修正`并发标记`期间因用户程序继续运行而导致 对象标记 发生变动的标记记录.这个阶段的停顿时间比初始标记稍长一些,但远比并发标记的时间短.
(6) 并发清除 (concurrent sweep) : 并发清除垃圾对象
(7) 重置

由于整个过程耗时最长的`并发标记`和`并发清除`都可以与用户线程一起运行,所以,总体来说,CMS收集器的内存回收过程是与用户线程一起并发执行的.

+ CMS 无法处理浮动垃圾,浮动垃圾是指在并发清理过程中程序产生的新垃圾.
+ 由于存在浮动垃圾,所以CMS不能等到老年代被塞满了才开始垃圾回收.CMS可以设置内存使用率的启动阈值,如设置(-XX:CMSInitiatingOccupancyFraction=90),代表当老年代内存使用率达到90%以后,触发垃圾回收.
+ 如果浮动垃圾大于 CMS预留的内存,那么将产生 "Concurrent Mode Failure",CMS会使用后备方案,临时启用Serial Old,停顿时间将大幅延长.
+ CMS使用`标记-清楚`算法,会导致内存碎片.可以通过设置参数CMSFullGCsBeforeCompaction 参数来设定 执行了多少次不压缩的full gc 后, 会执行一次压缩的full gc

![Alt text](/images/jvm_cms.png)

# 6 G1 (garbage first)收集器

## 6.1 G1 内存结构划分
G1 收集器仍然保留了分代收集的概念, 但它将JAVA堆划分成为多个大小相同的独立区域(region).新生代和老年代不再是物理隔离,它们都是一部分(内存不需要连续)region,每个region的大小一般为1~32MB
G1 包括以下几个类型的region
+ Unused region : 可用region
+ Eden region : 相当于年轻代的eden区
+ Survivor region : 相当于年轻代的survivor区
+ Old region : 相当于老年代
+ humongous region  : 大对象region,老年代region的一部分,某个对象比普通region 大50%,那么这个对象会放进 humongous region中.

PS : 每个region的用途都不是固定的,如一次GC后,eden region的对象被回收,变成 Unused region , 随后可能被分配用作 humongous region

RememberedSet : 每一个region都维护了一个RememberedSet(RSet),即 其他region对象 引用本region对象的记录 . 在回收某个region的时候,因为有RetS存在,可以避免全堆扫描. 一般来说RSet的大小占整个JAVA堆大小的的1%~20%

collection Set (CSet) : 收集集合,保存GC待回收的region集合

Card : region并不是最小的单元,每个region会被进一步划分为若干个大小为512个字节的堆内存块(card)

## 6.2 G1 使用的垃圾回收算法
整理来看,G1 使用`标记-整理`法. 从局部看,region的角度,是基于`复制`算法

## 6.3 G1的GC类型
1. Young Gc : 对年轻代的region进行回收
2. MixedGc : 对所有年轻代以及部分老年代region进行回收
3. FullGc : 类似CMS.退化成 Serial单线程,全堆扫描

G1在无法分配对象或者大对象无法获得内存连续的region,会优先尝试扩展堆的大小来获得更多的空间.
G1可以计算每个region的可回收空间大小,以及回收所需要的时间,每次根据允许的时间,优先回收价值最大的region

## 6.4 并行标记
当老年代占用整个堆内存的45%,可通过-XX:InitatingHeapOccupanyPercent设置,G1会启动并行标记.
不考虑维护 RSet的操作,过程如下
(1) 初始标记 : STW,借用Young GC的暂停阶段来 扫描GC root
(2) 根区间扫描 : 在年轻代回收的初始标记阶段 拷贝到 幸存者 区间 的对象需要被扫描,并被当做标记根元素.
(3) 并发标记 : 从GC root 寻找引用链
(4) 重新标记 : STW , 标记在第二阶段发生变化的对象
(5) 清理阶段 : 如果region全部都是垃圾对象,那么会回收这个region.否则,会等待mixedGc在回收.

## 6.5 MixedGc
通过并行标记阶段,可以统计到老年代的垃圾占比.
如果垃圾占比达到堆大小的5%,可通过XX:G1HeapWastePercent,会触发 MixedGc.

## 6.6 G1使用场景
+ 内存6GB以上
+ 可预测停顿时间小于0.5S