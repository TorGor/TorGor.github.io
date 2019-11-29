---
layout: post
title:  DevOps(Developmen & Operations)
date:   2019-11-29 00:00:00 +0800
categories: document
tag: DevOps
---

* content
{:toc}


>DevOps(Developmen & Operations) 思考
1.	devops 是什么？
2.	Devops 能够给我们解决哪些问题？
3.	Devops 需要哪些条件？
4.	团队应该怎么做？

# devops 是什么？
我个人理解，devops 就是以提高效率宗旨，利用各种管理方法和技术来辅助，实现项目周期管理：产品需求管理、会议记录、快速开发、加速迭代、运维方便、反馈及时提、反馈快速处理、代码自动测试、自动部署、自动更新。这是一个完整的闭环生态，可以极大的提高团队的效率，听着好像很抽象，这里有一张图可以具象来看。

![一般的 devops 概念](https://torgor.github.io/styles/images/devops/devops-sum.png)

> 引用：
DevOps（Development和Operations的组合词）是一组过程、方法与系统的统称，用于促进开发（应用程序/软件工程）、技术运营和质量保障（QA）部门之间的沟通、协作与整合。
它是一种重视“软件开发人员（Dev）”和“IT运维技术人员（Ops）”之间沟通合作的文化、运动或惯例。透过自动化“软件交付”和“架构变更”的流程，来使得构建、测试、发布软件能够更加地快捷、频繁和可靠。



# Devops 能够给我们解决哪些问题？

 
![devops在团队中的体现](https://torgor.github.io/styles/images/devops/devops-cercle.png) 
 
##  加强各个部门人员协调
Scrum 团队的标志性的特征，从项目开始到线上运行再到运维反馈，都是一个完整的闭环，在这个闭环内，
团队利用各种工具来实现交流和快速迭代，每个人都能够及时收到消息并对消息有快速理解。当然工具很多，
我下面会具体列出一些给大家参考，大家可以根据团队或者公司实际情况选择。最典型的莫过于 Jenkins 、gitlab 的生态。

##   代码检查
代码质量是确保程序稳定的要素，通常在项目中每个人都会负责相应的模块开发，并且需要协调提交合并，在那么多开发者协调操作时，
我们应该如何确保代码质量？
目前我们的方案
1. 采用模块化开发、微服务架构。每个人只负责各自的模块代码，不互相影响。
2. 采用 gitlab 作为代码托管工具，每次git 提交都有相应的代码单元测试，并使用代码覆盖率来作为判断标准，
一般代码覆盖率不低于90%。
3. 采用分支管理，develop分支作为开发分支，每个开发者有对应分支develop-XX，互相不会影响彼此的开发。

##   测试自动化
为每个接口添加单元测试。首先本地运行测试自检，成功后再进行提交测试，通过才会执行下一步pipeline操作，比如构建，
不成功自动停止 pipeline，并发送邮件给管理员和开发者，然后根据错误原因查看是哪些测试不通过。

##  服务容器化
采用docker 对服务进行容器化管理，至于容器化的好处不用多说，环境隔离、部署方便、自动化集成方便…… docker 容器的介绍这里就不具体介绍了，后面我在写一个专栏来介绍docker、docker-compose、k8s 等。
部署自动化
这个前提我觉得必须要以容器化为基础，现在的云端部署基本都是依赖k8s 作为平台来管理容器化的实例部署，google 云平台甚至可以做到灰度发布。这个效率可就和传统模式区分开了，不再有繁琐的环境配置，环境直接命令输入容器，平滑启动部署。镜像从网络或者私有的镜像库获取。K8s 自动根据命令来部署容器运行。

##  Issue git 自动关联检查
遇到现在问题需要及时反馈，创建issue并附上运行时参数，提交到开发者手中修改，开发者根据收到的issue 描述修改，fix 提交代码后 commit记录中带上 issue id ，自动将代码和issue 关联起来。然后继续迭代。这样就可以查看这次的issue 到底修改了哪些代码。


# Devops 需要哪些条件？
## 硬性要求：
代码管理（SCM）：GitHub、GitLab、SubVersion
构建工具：Ant、Gradle、maven
自动部署：K8s、Capistrano、CodeDeploy
持续集成（CI）： Jenkins、gitlab-ci、Bamboo、Hudson
配置管理：Ansible、Chef、Puppet、SaltStack、ScriptRock GuardRail
容器：Docker、LXC、第三方厂商如AWS
编排：Kubernetes、Docker-compose、Core、Apache Mesos、DC/OS
服务注册与发现：Zookeeper、Consul、Eureka
脚本语言：python、ruby、shell
日志管理：ELK、Logentries
系统监控：Prometheus 、Zipkin
性能监控：AppDynamics、New Relic、Splunk
压力测试：JMeter、Blaze Meter、loader.io
消息总线：RabbitMq、ActiveMQ
应用服务器：Tomcat、JBoss
Web服务器：Apache、Nginx、IIS
数据库：MySQL、Oracle、PostgreSQL等关系型数据库；
 mongoDB、redis等NoSQL数据库
项目管理（PM）：Jira、Redmine、Asana、Taiga、Trello、Basecamp、Pivotal Tracker

## 团队要求：
1. 眼界需要开阔，能够学习适应新的技术应用的挑战；
2. 能够承担多个角色带来的压力，随着devops 在项目中的应用，每个人的角色会发生变化，不再是单纯的一个角色，管理者的角色可能会从项目经理干到开发，再干到运维，苦逼如我。
3. 团队人员必须紧密配合，还是需要保持沟通和良好的关系。


![devops 项目架构图](https://torgor.github.io/styles/images/devops/devops-JG.png) 
