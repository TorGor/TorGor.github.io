---
layout: post
title:  Springboot 使用 ThreadPoolTaskExecutor 线程池
date:   2019-01-22 00:00:00 +0800
categories: document
tag: springboot
---

* content
{:toc}



ThreadPoolTaskExecutor	
===
Springframework 提供的线程池处理类，可以帮助我们利用线程池管理线程。

参数：
* corePoolSize：线程池维护线程的最少数量
  
* keepAliveSeconds：允许的空闲时间
  
* maxPoolSize：线程池维护线程的最大数量
  
* queueCapacity：缓存队列
  
* rejectedExecutionHandler：对拒绝task的处理策略，发生在队列已满，且超出了最大线程数后

execute(Runable)方法执行过程
===
* 线程池中的数量小于corePoolSize，即使线程池中的线程都处于空闲状态，也要创建新的线程来处理被添加的任务。

* 线程池中的数量等于 corePoolSize，但是缓冲队列 workQueue未满，那么任务被放入缓冲队列。

* 线程池中的数量大于corePoolSize，缓冲队列workQueue满，并且线程池中的数量小于maxPoolSize，建新的线程来处理被添加的任务。

* 线程池中的数量大于corePoolSize，缓冲队列workQueue满，并且线程池中的数量等于maxPoolSize，
 那么通过handler所指定的策略来处理此任务。也就是：处理任务的优先级为：核心线程corePoolSize、任务队列workQueue、
 最大线程 maximumPoolSize，如果三者都满了，使用handler处理被拒绝的任务。

* 线程池中的数量大于corePoolSize时，如果某线程空闲时间超过keepAliveTime，线程将被终止。这样，线程池可以动态的调整池中的线程数。


```flow
st=>start: 开始
opCore=>operation: 核心线程中获取
opQueue=>operation: 放入阻塞缓冲队列
opTread=>operation: 新建非核心线程
opHandler=>operation: 执行 handler 策略
condCore=>condition: activeSize < corePoolSize
condQueue=>condition: Yes or No?
condTread=>condition: Yes or No?
condHandler=>condition: Yes or No?
e=>end
st->op->cond
condCore(yes)->e
condCore(no)->op
&```


![/JMM-Thread-JVM.png]({{ '/styles/images/jmm/JMM-Thread-JVM.png' | prepend: site.baseurl  }})


```flow
st=>start: 开始
op=>operation: My Operation
cond=>condition: Yes or No?
e=>end
st->op->cond
cond(yes)->e
cond(no)->op
&```