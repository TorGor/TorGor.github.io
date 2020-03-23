---
layout: post
title:  Mycat 实践和思考
date:   2019-12-29 00:00:00 +0800
categories: document
tag: mycat
---

* content
{:toc}


>mycat 使用前思考
1.	mycat 是什么？
2.	mycat 能够给我们解决哪些问题？
3.	mycat 使用中会遇到哪些问题？

# mycat 简介
简单的来说，是一个数据库分库分片工具，程序只需要连接 mycat 的服务和端口，就可以像访问物理数据库一样进行操作。
## 关键特性
支持SQL92标准

遵守Mysql原生协议，跨语言，跨平台，跨数据库的通用中间件代理。

基于心跳的自动故障切换，支持读写分离，支持MySQL主从，以及galera cluster集群。

支持Galera for MySQL集群，Percona Cluster或者MariaDB cluster

基于Nio实现，有效管理线程，高并发问题。

支持数据的多片自动路由与聚合，支持sum,count,max等常用的聚合函数。

支持单库内部任意join，支持跨库2表join，甚至基于caltlet的多表join。

支持通过全局表，ER关系的分片策略，实现了高效的多表join查询。

支持多租户方案。

支持分布式事务（弱xa）。

支持全局序列号，解决分布式下的主键生成问题。

分片规则丰富，插件化开发，易于扩展。

强大的web，命令行监控。

支持前端作为mysql通用代理，后端JDBC方式支持Oracle、DB2、SQL Server 、 mongodb 、巨杉。

支持密码加密

支持服务降级

支持IP白名单

支持SQL黑名单、sql注入攻击拦截

支持分表（1.6）

集群基于ZooKeeper管理，在线升级，扩容，智能优化，大数据处理（2.0开发版）。





