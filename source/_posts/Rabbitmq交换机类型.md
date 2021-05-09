---
title: Rabbitmq交换机类型
date: 2020-10-10 19:30:30
tags: MQ
---

Rabbitmq 是AMQP的实现,,本文所说的四种交换机实际也是 AMQP 支持的四种交换机.

# 1 直连交换机(Direct Exchange)
队列绑定到某个交换机,同时绑定一个路由键`routing key`,如路由键为`red`
交换机 会把这条消息路由给 同样绑定路由键为`red`的消费者队列

如图
路由键为`red` 的消息只会 路由 到同样 路由键为`red` 的消费者队列里,而不会路由到 `blue` , `green`的队列.

![Alt text](/images/rabbitmq_direct_route.png) 

<!--more-->


# 2 扇形交换机(Fanout Exchange)
扇形交换机会将消息 路由到所有绑定到该交换机的队列,而不理会绑定的路由键.

由于扇形交换机不理会绑定的路由键,所以也没必要给消息绑定路由键.

如图,

![Alt text](/images/rabbitmq_fanout_route.png) 

# 3 主题交换机(Topic Exchange)
主题交换机只会将消息路由给 符合路由表达式的消费者队列, 类似在扇形交换机上根据路由规则过滤,再路由消息给指定的消费者队列.

主题交换机的生产者绑定的路由如果有多个单词需要用`.`分开
消费者绑定的路由必须是符合`*.#.*.... `等的格式
其中
+ `*`表示一个单词(注意是单词,一个单词包含多个字符)
+ `#`表示任意数量(0或者多个)的单词

当一个消费者队列绑定键为`#`时,那么这个队列会接受这个主题交换机的所有消息

假设有一条消息的`route_key` 为`www.baidu.com`,那么带有如下绑定键的消费者队列都会接受到消息
+ `www.*.*`
+ `*.*.com`
+ `www.#`
+ `#.com`

如图 `Queue1` , `Queue2` , `Queue3` 表达式均能匹配`www.baidu.com`,所以能接收消息
`Queue4` 只能匹配`www.baidu` , `Queue5` 只能匹配`baidu.com`,所以都不能接收消息

![Alt text](/images/rabbitmq_topic_route.png) 


# 4 头(首部)交换机(Header Exchange)

头交换机和主题交换机类似,只不过主题交换机是根据路由做过滤.而头交换机是根据消息的多个属性来代替路由键建立过滤规则.
可以类似http的header,但头交换机的头属性值不仅支持字符串,还可以是任意的Object类型.

消费者绑定头交换机的时候必须指定完全匹配,还是任意匹配.
具体的表现为消息携带的hash要添加如下

+ 完全匹配

object.Add("x-match", "all");

+ 任意匹配
object.Add("x-match", "any");

如果为完全匹配,则消费者队列的header必须完全匹配生产者的header.
如果是任意匹配,只要有一个键值对匹配即可

如图 `Queue1` 为任意匹配,只要匹配到用户名为zhangsan,即可接收消息.
		`Queue2`为完全匹配,密码123456不正确,所以无法接收消息
		`Queue3`为完全匹配,用户名,密码都正确,所以也能接收消息.

![Alt text](/images/rabbitmq_header_route.png) 
