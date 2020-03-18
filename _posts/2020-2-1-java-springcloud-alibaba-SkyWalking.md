---
layout: post
title:  微服务链路追踪：springcloud + SkyWalking
date:   2020-2-2 00:00:00 +0800
categories: document
tag: lock
---

>开篇思考
1.	为什么需要服务追踪？
2.	分布式事务几种实现方式？
3.	哪种追踪方式性能更优？

![skywalking-控制台](https://torgor.github.io/styles/images/skywalking/skywalking-dashboard.png) 

## 为什么需要服务追踪
在微服务架构下，由于进行了服务拆分，一次请求往往需要涉及多个服务，
每个服务可能是由不同的团队开发，使用了不同的编程语言，还有可能部署在不同的机器上，分布在不同的数据中心。
服务跟踪系统可以跟踪记录一次用户请求都发起了哪些调用，经过哪些服务处理，并且记录每一次调用所涉及的服务的详细信息，
通过查看完整的调用链路，形成拓补图可以更加直观的了解业务，也可以针对当前的系统进行分析，是否需要扩容、优化接口、失败缓解，
还有通过日志快速定位是调用失败的环节。

## 服务追踪的实际应用

### 捋清业务
我们都知道，在一般场景下，我们很难直观的了解系统的运行、业务的流程，因为传统的都是文字需求说明和枯燥的代码。通过链路追踪，可以
根据调用链路来捋清楚服务间的调用关系，如果 API 设计符合规范，甚至可以直观的了解调用的服务作用。这对于刚刚接触系统的开发人员
十分重要。
![skywalking-请求追踪图](https://torgor.github.io/styles/images/skywalking/skywalking-trace.png) 


### 分析耗时
链路的基本功能，服务间的调用耗时记录，如果服务耗时过长，会影响整体的用户体验，甚至会抛出超时异常等，这样的情况在微服务架构中也是
时有发生。
![skywalking-慢追踪](https://torgor.github.io/styles/images/skywalking/dashboard-endpoint.png) 


### 可视化错误
微服务调用链路发生错误，可以直观的显示查看，定位到被调用服务的接口，及时排查微服务中错误原因。

### 优化链路:
显示完整的调用链路，根据业务分析合理性、可读性、健壮性，是否重复调用某一个服务，是否链路过长，有没有可以优化的，链路是否清晰。
有些场景比较复杂，比如数据中心比较分散，服务分布在不同的数据中心，但是服务中心之间因为地域原因，距离远，延迟高，这可能不符合设计
要求，因此就要根据链路来找到最近的数据中心，然后配置调用最近的数据中心的服务。
![skywalking-拓补图](https://torgor.github.io/styles/images/skywalking/skywalking-trace-detail.png) 


### 生成网络拓扑:
通过服务追踪系统中记录的链路信息，可以生成一张系统的网络调用拓扑图，它可以反映系统都依赖了哪些服务，
以及服务之间的调用关系是什么样的，可以一目了然。除此之外，在网络拓扑图上还可以把服务调用的详细信息也标出来，
也能起到服务监控的作用。
![skywalking-慢追踪](https://torgor.github.io/styles/images/skywalking/skywalking-tuobu-detail.png) 




