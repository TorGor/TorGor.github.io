---
layout: post
title:  分布式系统不可缺少的 MQ （消息机制）
date:   2020-4-3 00:00:00 +0800
categories: document
tag: distribute
---

>开篇思考
1. MQ 为什么用在系统中使用？一定要在分布式系统中使用吗？
2. MQ 有哪些中间件？他们有哪些特点？
3. MQ 给系统带来好处的同时有没有带来什么问题？如何解决？

在阿里的面试中，面试官问到关于 MQ 的几个问题，跟大家分享下：
* 你的项目中 MQ 的作用？
* 为什么选择这款 MQ 作为消息中间件？
* 重复消费怎么办？
* 如何确保消息被消费？
* 有遇到其他问题吗？
那么接下来带着问题先思考下，有好的想法可以在评论区留言，大家一起分享。

# 消息中间件在系统中的使用
我之前写过一篇关于 rocketMQ 实现分布式锁的文章，主要介绍如何使用 RocketMQ 实现分布式锁，[《Springcloud + RocketMQ 解决分布式事务》](https://mp.weixin.qq.com/s/QSxVscvMVuJpt6yXrrkXbg)
但是这个功能并不是 MQ 基本功能，也不是所有 MQ 都有的功能。

MQ 在系统中到底有哪些作用呢？抛开基本的消息发布订阅不说，还有以下几点：
1. 分布式系统解耦
2. 不需要立即返回的业务异步处理
3. 削峰填谷，不直接访问服务，缓解服务压力，增加性能
4. 日志记录

# 分布式系统解耦

![主流 MQ 比较](https://torgor.github.io/styles/images/distribute/MQ-解耦.png)

在分布式系统中，要么是通过 rest 调用，要么是通过 dubbo 等 RPC 调用，但是有些场景需要解耦设计，不能直接调用。  
比如消息驱动的系统中，消息发送者完成本地业务，发送消息，多平台的消息消费者服务需要收到推送的消息，然后继续处理其他业务。

看这两个架构图，第一种 BC 都直接依赖 A 服务，那么如果 A 中的接口修改，BC 都要跟着做修改，耦合度高。  
第二种，通过 MQ 来作为中间件收发消息，BC 只依赖收到的消息而不是具体的接口，这样即使 A 服务修改或者增加其他服务，都只要订阅MQ就行。

# 不要求实时的业务异步处理

用户注册业务流程为例，
1. 用户注册入库
2. 用户验证邮件发送 
3. 用户验证短信发送

原来的系统设计，这样的服务流程会串行处理，即先 1-2-3 ；但是这里可以思考下，如果单个服务单台机器的情况下，注册用户特别多，系统能不能抗住？

这里假设各个阶段的时间 1 = 50ms ， 2 = 50ms ， 3 = 50ms，那么一个请求下来就是 all = 150ms；  
这里再假设，这个服务器 CPU = 1 ， 且只能处理单线程，那么以这种单台服务器单线程的 QPS 来算；QPS = 1000/150 ≈ 7  
现在我要让这个 QPS * 3 提升三倍，这个时候引入 MQ 服务作为中间件

![主流 MQ 比较](https://torgor.github.io/styles/images/distribute/MQ-sync.png)


# 主流 MQ 的特点 
![主流 MQ 比较](https://torgor.github.io/styles/images/distribute/distribute-mq-compare.jpg)

# 喜欢文章请关注我
> 程序领域

![公众号](https://torgor.github.io/styles/images/my-public-ma.png)
![赞赏码](https://torgor.github.io/styles/images/my-zanshang-ma.png)








