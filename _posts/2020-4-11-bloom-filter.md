---
layout: post
title:  Bloom Filter 布隆过滤器
date:   2020-3-11 00:00:00 +0800
categories: document
tag: distribute
---

>开篇思考  
1. 你能想到哪些方式判断一个元素是否存在集合中？
2. 布隆过滤器并不存储数据本身，那么是怎么做到过滤的？
3. 布隆过滤器实现？参数配置？

一般我们用来判断一个元素是否存在，会想到用 List，Map，Set 等，会将元素先保存下来，然后进行筛选。  
但是这样的形式都有一个弊端就是一定要保存数据才行，可是我们仅仅想知道是否存在数据，并不要求获取实际数据，
这时候就会觉得这种方式实在是浪费空间。  

什么情况下我们只需要判断是否存在这个元素呢？  
在系统设计的时候，我们会考虑大量并发的形式，但是很多请求可能是在访问不存在的数据，
那么我们就没有必要继续这个请求，可以在 API 网关层就直接过滤掉。  

# Bloom Filter 布隆过滤器原理

Bloom filter 是由 Howard Bloom 在 1970 年提出的二进制向量数据结构，它具有很好的空间和时间效率，
被用来检测一个元素是不是集合中的一个成员。  

布隆过滤器实现是不保存数据本身，而是通过 K 个 hash 函数来计算在 byte[] 数组中的存放位置，
并把这个位置的值设置为 1， 而这个 K 到底是多少个呢，要根据公式来算出，待会列出。  
除了这个 K 值，我们还要计算 byte[] 数组的长度 m ，下面一并列出计算公式：

![m 值计算](https://torgor.github.io/styles/images/redis/bloom-filter-m-equation.png)  

* fpp : 误判率参数，(must be 0 < fpp < 1)
* n   ：预估的需要过滤的总数量
* ln  ：求对数，不会的把高中老师的名字写下来  

![K 值计算](https://torgor.github.io/styles/images/redis/bloom-filter-k-equation.png)  

* m ：数组长度
* n ：预估的需要过滤的总数量

 下面我们以数字 11 为例来使用，有个网站可以测试布隆过滤器，
 [在线测试布隆](https://www.jasondavies.com/bloomfilter/)
 
 ![11 过滤](https://torgor.github.io/styles/images/redis/bloom-filter-step-11.png)  
 
# 布隆过滤器的优点、缺点
优点：  
* 节省空间，不用保存所有数据，知识通过 hash 值来计算位置，并通过 byte[] 记录下来。
* 速度快，时间复杂度低 O(1);

缺点：  
* 精度低，假设：a 计算的位置 1 ，3 ；b 计算的位置 5，7；c 计算的位置 1，7，那么 c 一定存在吗？
* 不能直接删除，因为想要删除就要把对应的位置置为 0 ，如果这样做，可能会影响其他值的过滤。


 ![11 过滤](https://torgor.github.io/styles/images/redis/bloom-filter-conflict.png)  

# 喜欢文章请关注我  
  
> 程序领域  
**点击关注+转发，私信发送【面试】或者【资料】可以收获更多资源**

![公众号](https://torgor.github.io/styles/images/my-public-ma.png)

![赞赏码](https://torgor.github.io/styles/images/my-zanshang-ma.png)








