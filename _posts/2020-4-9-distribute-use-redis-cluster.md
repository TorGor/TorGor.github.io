---
layout: post
title:  redis 集群搭建方式：docker-compose + 传统模式(未完待续)
date:   2020-4-6 00:00:00 +0800
categories: document
tag: distribute
---

要偷懒用 docker，因此极力推荐使用 docker-compose 一键式命令启动。

# docker-compose 启动 redis sentinel 高可用

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

![keys 命令](https://torgor.github.io/styles/images/redis/redis-string-command.png)

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








