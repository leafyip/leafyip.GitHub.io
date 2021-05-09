---
title: AMQP协议
date: 2020-10-08 17:29:53
tags: MQ
---
# 1 AMQP

AMQP(Advanced Message Queuing Protocol) 一个提供统一消息服务的应用层标准高级消息队列协议,是应用层协议的一个开放标准,为面向消息的中间件设计.
其中,Rabbitmq是基于AMQP实现的

## 1.1 AMQP中的一些概念
+ Server ：接收客户端的连接,实现AMQP实体服务.

+ Connection ：连接,应用程序与Server的网络连接,TCP连接.

+ Channel ：信道,消息读写等操作在信道中进行.客户端可以建立多个信道,每个信道代表一个会话任务.

+ Message ：消息,应用程序和服务器之间传送的数据,消息可以非常简单,也可以很复杂.有Properties和Body组成.Properties为外包装,可以对消息进行修饰,比如消息的优先级、延迟等高级特性；Body就是消息体内容.

+ Virtual Host：虚拟主机,用于逻辑隔离.一个虚拟主机里面可以有若干个Exchange和Queue,同一个虚拟主机里面不能有相同名称的Exchange或Queue.

+ Exchange ：交换器,接收消息,按照路由规则将消息路由到一个或者多个队列.如果路由不到,或者返回给生产者,或者直接丢弃.RabbitMQ常用的交换器常用类型有direct、topic、fanout、headers四种,后面详细介绍.

+ Binding ：绑定,交换器和消息队列之间的虚拟连接,绑定中可以包含一个或者多个RoutingKey.

<!--more-->


+ RoutingKey：路由键,生产者将消息发送给交换器的时候,会发送一个RoutingKey,用来指定路由规则,这样交换器就知道把消息发送到哪个队列.路由键通常为一个“.”分割的字符串,例如“com.rabbitmq”.

+ Queue：消息队列,用来保存消息,供消费者消费.

+ Publisher : 发送消息的程序.

+ Consumer : 接收消息的程序.

## 1.2模型简介

发布者(publisher) 发送 消息(message) 给 交换机(exchange),交换机可比喻成邮箱.交换机按照路由规则将消息分发到绑定的队列,
AMQP代理(消息中间件)会将消息投递给订阅此队列的消费者(consumer).

![Alt text](/images/amqp_model.png) 


 