---
layout: post
title:  正经的聊聊架构中的 redis
date:   2020-4-8 00:00:00 +0800
categories: document
tag: distribute
---

>开篇思考
1. Redis 为什么在系统中使用？解决了哪些问题？
2. Redis 如何保证和数据库同步？
3. Redis 缓存操作是在操作数据库前还是操作数据库后？

还记得税务 APP 吗？真是谁用谁知道。周围响起了熟悉的芬芳：“SX系统，SX软件...”，身为程序员的我听到总是百感交集。。。
呦呵，背后可能又是一堆人背锅了。  
我也特别喜欢吐槽，我觉得正确的吐槽姿势有助于系统的良性发展，就像父母的爱强烈扎刺着程序员面临崩溃的心灵，并浇灌给系统茁壮成长。 
  
![公众号](https://torgor.github.io/styles/images/biaoqingbao/xintai-hao.png)

系统稳定，快速，美如画谁都想追求，可是往往美好的东西后面代价也不小。  
追求可靠，我就要集群部署，容错容灾，那么就需要更多的机器设施及其他附属服务。   
追求快速，我们需要解决地域限制，全球部署战斗机，DNS 快速定位访问，软件层面缓存技术。   

那么我们接下来就依照软件层面的缓存技术来接着进入正题，不再扯蛋，容易疼。

# Redis 简介  

1. 内存存储，速度极快
2. key-value 存储结构
3. 支持 string，list，set，zset，hash 类型，其实还有一些不常用的
4. 基于 epoll 多路复用，串行效率高
5. 可以持久化数据，遇到宕机可以快速恢复
6. redis 支持主从模式、哨兵模式
7. 使用场景丰富：热点数据缓存、临时会话存储、消息发布订阅、网页计数

上面的介绍中，我基本扒出了 redis 的主要特点，外衣都给你扒了，这么赤裸的诱惑你们都不要吗？觉得还是不够吸引吗?
那我们就继续来扒拉扒拉。。。

# 内存

Redis 都是通过计算机内存来存取的，这点都知道吧，不用多解释。它为什么快？  
JMM java 中的内存模型大家了解吧，java 中每个线程会有自己的内存，要想达成可见性，需要同步主内存，这一操作听起来
很简单，但其实里面数据被拷贝了多次。这里简单介绍下传统的磁盘到网络的数据拷贝流程：
1. 磁盘到 read buffer， 快
2. read buffer 到 user buffer ，此处很慢，上下文有切换
3. user buffer 到 socket buffer ，快
4. socket buffer 写入到网卡 buffer 发送，快

好家伙，不扒不知道，原来底层数据是这么传输的。Redis 为什么快呢，因为它官方只支持 linux 系统，而 Linux 
本身还支持零拷贝技术，并且这里都是纯内存操作，所有的数据操作都非常快。那么究竟有多快呢，一秒真男人：10 w/s。
当然数据只能是小数据流量的。

# redis 支持的数据类型及应用场景

redis 比 memcached 的优点多，其中有一个就是支持多种数据格式。
下面看下具体的数据格式操作：

* String 
```
#设值
set stra aa

#取值
get stra 
 
#查看所有key
keys * 

```
string 适用场景：  
1. 计数器
2. json 格式对象信息
3. 接口幂等性控制，根据唯一标识
4. 分布式锁 setnx 

![keys 命令](https://torgor.github.io/styles/images/redis/redis-key-command.png)

* List
```
#设值-从左侧开始 push
lpush mylist value [value ...]

#取值-指定长度（会从左边开始取）
lrange mylist 0 1 

#查看 list 长度
llen mylist

```

从上面也看出来了，list 其实是 linked list 结构。这样的结构方便插入和查询。  
list 适用场景：  
1. 评论、商品等列表数据的缓存
2. 当作消息队列存储

![List 命令](https://torgor.github.io/styles/images/redis/redis-list-command.png)

* sets
```
#设值
sadd myset a b c d e f g 

#取值
smembers myset

#合并 set 到新的 set 中，
#集群中会报  don't hash to the same slot ，意思是不在一个hash 槽中
#可以适用 Hash Tag 指定到同一槽中.
#Hash Tag 允许用key的部分字符串来计算hash。 sadd {myset}1 value 
sunionstore myustore myset1 myset2
```
这里因为我本地使用了 redis 集群来测试，遇到了 don't hash to the same slot 异常。  
为什么会有这个异常这里我觉得有必要解释下，因为 redis 的集群是通过 hash 值计算确定存放哪个机器上的。  
redis 集群待会介绍。  

Hash Tag 允许用 key 的部分字符串来计算 hash，那么怎么截取这部分字符串呢，就适用特殊字符 {}。  
当一个key包含 {} 的时候，就不对整个key做hash，而仅对 {} 包括的字符串做hash。  

```
sadd {myset}1 a b c
sadd {myset}2 a b c
```

![set 命令](https://torgor.github.io/styles/images/redis/redis-sets-command.png)

* sort set
```
#设值
zadd {myZset}1 1 a 2 b 3 c 4 d

#范围取值 0-3
zrange {myZset}1 0 3
```
ZADD 参数（options） (>= Redis 3.0.2)  
ZADD 命令在key后面分数/成员（score/member）对前面支持一些参数：  

* XX: 仅仅更新存在的成员，不添加新成员。
* NX: 不更新存在的成员。只添加新成员。
* CH: 修改返回值为发生变化的成员总数，原始是返回新添加成员的总数 (CH 是 changed 的意思)。  
更改的元素是新添加的成员，已经存在的成员更新分数。 所以在命令中指定的成员有相同的分数将不被计算在内。  
注：在通常情况下，ZADD返回值只计算新添加成员的数量。
* INCR: 当ZADD指定这个选项时，成员的操作就等同ZINCRBY命令，对成员的分数进行递增操作。

zset 适用场景：  
1. 排行榜
2. 

![zset 命令](https://torgor.github.io/styles/images/redis/redis-zset-command.png)

* hash
```
#设值
192.168.244.88:7002> hmset myhash user holy name place age 13
-> Redirected to slot [9295] located at 192.168.244.88:7001
OK
192.168.244.88:7001> hget myhash user
"holy"
192.168.244.88:7001> hget myhash name
"place"
192.168.244.88:7001> hget myhash age
"13"
192.168.244.88:7001> 

```
hash 适用场景：  
1. 对象存储，适合经常改变属性的对象

![hash 命令](https://torgor.github.io/styles/images/redis/redis-hash-command.png)


# 喜欢文章请关注我  
  
> 程序领域  
**点击关注+转发，私信发送【面试】或者【资料】可以收获更多资源**

![公众号](https://torgor.github.io/styles/images/my-public-ma.png)

![赞赏码](https://torgor.github.io/styles/images/my-zanshang-ma.png)








