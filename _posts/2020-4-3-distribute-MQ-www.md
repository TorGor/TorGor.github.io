---
layout: post
title:  分布式系统不可缺少的 MQ （消息机制）
date:   2020-4-3 00:00:00 +0800
categories: document
tag: lock
---

>开篇思考
1. MQ 为什么用在系统中使用？一定要在分布式系统中使用吗？
2. MQ 有哪些中间件？他们有哪些特点？
3. MQ 给系统带来好处的同时有没有带来什么问题？如何解决？

# 消息中间件在系统中的使用
我之前写过一篇关于 rocketMQ 实现分布式锁的文章，主要介绍如何使用 RocketMQ 实现分布式锁，[《Springcloud + RocketMQ 解决分布式事务》](https://mp.weixin.qq.com/s/QSxVscvMVuJpt6yXrrkXbg)
但是这个功能并不是 MQ 基本功能，也不是所有 MQ 都有的功能。

MQ 在系统中到底有哪些作用呢？抛开基本的消息发布订阅不说，还有以下几点：
1. 分布式系统解耦
2. 不需要立即返回的业务异步处理
3. 削峰填谷



# 主流 MQ 的特点 
![主流 MQ 比较](https://torgor.github.io/styles/images/distribute/distribute-mq-compare.jpg)

# 喜欢文章请关注我
> 程序领域

![公众号](https://torgor.github.io/styles/images/my-public-ma.png)
![赞赏码](https://torgor.github.io/styles/images/my-zanshang-ma.png)








