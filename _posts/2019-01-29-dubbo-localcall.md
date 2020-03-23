---
layout: post
title:  dubbo 绕过注册中心本地调用、测试环境调用
date:   2019-1-29 00:00:00 +0800
categories: document
tag: mycat
---

* content
{:toc}
<!-- TOC -->
[toc]

在开发及测试环境下，经常需要绕过注册中心，只测试指定服务提供者，这时候可能需要点对点直连，点对点直连方式，
将以服务接口为单位，忽略注册中心的提供者列表，A 接口配置点对点，不影响 B 接口从注册中心获取列表。

![dubbo本地调用链](https://torgor.github.io/styles/images/dubbo/dubbo-directly.jpg) 

# 通过 XML 配置

如果是线上需求需要点对点，可在 <dubbo:reference> 中配置 url 指向提供者，将绕过注册中心，多个地址用分号隔开，配置如下 ：
register="false"：禁用注册配置
```
<dubbo:reference id="xxxService" interface="com.alibaba.xxx.XxxService" url="dubbo://localhost:20890" />
```

## 通过 -D 参数指定

通过 -D 参数指定
在 JVM 启动参数中加入-D参数映射服务地址，如：
```
java -Dcom.alibaba.xxx.XxxService=dubbo://localhost:20890
```

## 通过文件映射
如果服务比较多，也可以用文件映射，用 -Ddubbo.resolve.file 指定映射文件路径，
此配置优先级高于 <dubbo:reference> 中的配置 ，如：``java -Ddubbo.resolve.file=xxx.properties``

然后在映射文件 xxx.properties 中加入配置，其中 key 为服务名，value 为服务提供者 URL：

```
com.alibaba.xxx.XxxService=dubbo://localhost:20890
```
**注意 为了避免复杂化线上环境，不要在线上使用这个功能，只应在测试阶段使用。**

> 1.0.6 及以上版本支持
  
>  key 为服务名，value 为服务提供者 url，此配置优先级最高，1.0.15 及以上版本支持 
  
>  1.0.15 及以上版本支持，2.0 以上版本自动加载 ${user.home}/dubbo-resolve.properties文件，不需要配置 

## 测试生成环境中使用 version 或者 group 
为了避免一些麻烦，也可以继续使用zookeeper注册中心，但是需要通过 version 或者 group 来控制，同样切记不要提交测试环境

version：
```
provider:
<dubbo:service id="xxxService" interface="com.alibaba.xxx.XxxService" version="v1" />
consumer:
<dubbo:reference id="xxxService" interface="com.alibaba.xxx.XxxService" version="v1" />
```
group
```
provider:
<dubbo:service id="xxxService" interface="com.alibaba.xxx.XxxService" group="v1" />
consumer:
<dubbo:reference id="xxxService" interface="com.alibaba.xxx.XxxService" group="v1" />
```


# 求关注
> 程序领域

![公众号](https://torgor.github.io/styles/images/my-public-ma.png)
![赞赏码](https://torgor.github.io/styles/images/my-zanshang-ma.jpg)