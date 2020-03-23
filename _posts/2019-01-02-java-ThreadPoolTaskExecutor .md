---
layout: post
title:  Springboot 使用 ThreadPoolTaskExecutor 线程池
date:   2019-01-02 00:00:00 +0800
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


策略
===
1. AbortPolicy （终止执行）：默认策略， Executor会抛出一个RejectedExecutionException 
运行异常到调用者线程来完成终止。

2. CallerRunsPolicy（调用者线程来运行任务）：这种策略会由调用execute方法的线程自身来执行任务，
它提供了一个简单的反馈机制并能降低新任务的提交频率。

3. DiscardPolicy （丢弃策略）：不处理，直接丢弃提交的任务。

4. DiscardOldestPolicy （丢弃队列里最近的一个任务）：如果 Executor 还未shutdown 的话，
则丢弃工作队列的最近的一个任务，然后执行当前任务。


配置线程数依据
===

* CPU密集型任务，那么线程池的线程个数应该尽量少一些，一般为CPU的个数+1条线程。
* IO密集型任务，那么线程池的线程可以放的很大，如2*CPU的个数。
* 混合型任务，如果可以拆分的话，通过拆分成 CPU密集型 和 IO密集型 两种来提高执行效率；
如果不能拆分的的话就可以根据实际情况来调整线程池中线程的个数。

Springboot 配置
===

```java
@Configuration
@ComponentScan("org.jeecg.modules.publishcode.async")
@EnableAsync
public class ThreadingConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        //配置核心线程数
        executor.setCorePoolSize(5);
        //配置最大线程数
        executor.setMaxPoolSize(5);
        //配置队列大小
        executor.setQueueCapacity(8888);
        //当容器停止后允许等待时间
        taskExecutor.setAwaitTerminationSeconds(10);
        //当容器停止是否等待线程结束
        taskExecutor.setWaitForTasksToCompleteOnShutdown(true);
        //策略
        taskExecutor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        taskExecutor.initialize();
        return taskExecutor;
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return AsyncConfigurer.super.getAsyncUncaughtExceptionHandler();
    }

}

```

# 求关注
> 程序领域

![公众号](https://torgor.github.io/styles/images/my-public-ma.png)
![赞赏码](https://torgor.github.io/styles/images/my-zanshang-ma.jpg)

