---
layout: post
title:  Java VisualVM 远程连接
date:   2017-09-21 00:00:00 +0800
categories: document
tag: JVM
---

* content
{:toc}

### Java VisualVM

自Java 6开始，Java程序启动时都会在JVM内部启动一个JMX agent，JMX agent会启动一个MBean server组件，
把MBeans（Java平台标准的MBean + 你自己创建的MBean）注册到它里面，然后暴露给JMX client管理。
简单来说就是每个Java程序都可以通过JMX来被JMX client管理，而且这一切都是自动发生的。
而VisualVm就是一个JMX Client。

![JVVM](https://torgor.github.io/styles/images/jmm/JVVM.png)

### JDK 自带工具，可以直接打开

![JDK-JVVM](https://torgor.github.io/styles/images/jmm/JDK-VisaulVM.png)


### 远程启动 jar

nohup java -Dcom.sun.management.jmxremote.port=4777 
-Dcom.sun.management.jmxremote.ssl=false 
-Dcom.sun.management.jmxremote.authenticate=false 
-Djava.rmi.server.hostname=192.168.31.88 
-jar publishcode-2.0.1.jar &

* -Dcom.sun.management.jmxremote.port                      

   开启远程端口，端口号就是连接端口
   
* -Dcom.sun.management.jmxremote.ssl

  是否启动 ssl 登录，测试环境不需要，建议生产环境使用
  
* -Dcom.sun.management.jmxremote.authenticate

  是否开启远程身份认证
  
* -Djava.rmi.server.hostname

  远程访问主机名，这里一定要填写，否则远程连接异常，显示无法连接
    

### 远程连接 

远程 -》 右键 新建远程 -》 输入远程ip

192.168.31.99:4777


![JDK-JVVM](https://torgor.github.io/styles/images/jmm/JVVM-console.png)


# 求关注
> 程序领域

![公众号](https://torgor.github.io/styles/images/my-public-ma.png)