---
layout: post
title:  重构-代码中的坏味道如何去除
date:   2020-3-11 00:00:00 +0800
categories: document
tag: distribute
---

>开篇思考：哪些坏味道
1. 大量的 if else 
2. 对象关系不够清晰、细致
3. 逻辑结构层层嵌套，难阅读

程序代码应该是一首诗，是一件艺术。任何影响艺术的美都应该被消除。程序代码会经历不断的打磨推敲，来达到艺术品的精美程度。  

最近我接到一个需求，内容是修改之前的账单发送功能，本来以为会是既简单又愉快的工作，但当我第一次看到遗留的代码，
我开始怀疑我的想法，最终不得不只在原来代码的基础修改，只为了不影响线上的正常功能。

既然不能全部重构，又要让我写的代码看上去舒服，我采用了《重构》中的多种方式来优化，
包括提炼函数，对象思维函数迁移，对象思维变量迁移，简化表达式，将值对象改为引用对象，工厂模式，策略模式。

《重构》这本书初读的时候还不是完全理解，现在随着编程经验的增加，重读这本书发现有些思想确实是精妙，是我以前完全忽略的点。
下面我们就通过代码来说说如何来去除代码中的坏味道。

# 谈恋爱策略
现在假设，一个渣男张三，需要轰轰烈烈的谈一场恋爱，同时约三个女孩，那么我们该怎么谈？
这个恐怕没有统一的方程式，不同的女孩喜好不一样，那么男孩张三需要挑选的方式也不一样。
以女孩的三个喜好的策略来：
* Tom : 一个喜欢手机的女孩
* Wish: 一个喜欢房子的女孩
* Yohn：一个喜欢鲜花的女孩

# 原始模式
if(Tom) send phone  
if(Wish) send house  
if(Yohn) send flower  

分析一下，这样的设计有一些问题：
* 不符合“开闭原则”，如果我要增加一个策略需要修改原来的代码
* 策略越多代码越长，不利于阅读，虽然更容易理解代码的意思，但是代码也会变成裹脚布，又臭又长
* 维护成本随时间增加而增加，随着时间推移，越来越多的人维护过代码，那么很难想想最后这个代码会变得多畸形

# 策略模式
![策略模式 UML ](https://torgor.github.io/styles/images/designpattern/strategy/Strategy-uml.png)

在 UML 图中，我们看到所有的策略都实现了抽象接口，根据依赖倒置原则，我们可以实现更多策略，而我们要做的只是通过
上层接口直接调用即可。
[策略模式源码仓库](https://github.com/TorGor/abitstep/tree/master/src/main/java/org/holy/leetcode/designpattern/strategy) 



先实现接口抽象,把所有追女孩的方法都抽象为 getGirl，但是不具体实现

```java
public interface Strategy {
    /**
     * 追女孩
     */
    void getGirl();

}
```
   
接下来实现抽象接口，分为三个策略来实现，分别是追 Tom 的策略，追 Wish 的策略，追 Yohn 的策略  
```java
public class TomStrategy implements Strategy {
    /**
     * 追女孩
     */
    @Override
    public void getGirl() {
        System.out.println("Tom: Send Huawei Phone !");
    }
}

public class WishStrategy implements Strategy {
    /**
     * 追女孩
     */
    @Override
    public void getGirl() {
        System.out.println("Wish: Send House !");
    }
}

public class YohnStrategy implements Strategy {
    /**
     * 追女孩
     */
    @Override
    public void getGirl() {
        System.out.println("Yohn： Send flowers !");
    }
}

```    

有了这三个策略，我们可以去根据环境来使用了，具体使用情况根据上下文，也因此，我们需要在上下文中注入策略
```java
public class ApplicationContext {

    private Strategy strategy;

    /**
     *
     * @param strategy
     */
    public void setStrategy(Strategy strategy){
        this.strategy = strategy;
    }

    /**
     * 调用策略
     */
    public void invokeStrategy(){
        strategy.getGirl();
    }

}
```

# 策略模式的优缺点
策略模式的优点  
* 易扩展，当我们需要扩展的时候，只要实现接口即可，不破坏原有的代码逻辑，符可开闭原则
* 简化代码，再没有裹脚布一样的代码，方便阅读维护
* 可以用来装B，凸显自己的设计能力

策略模式的缺点 
* 上下文必须清楚的知道需要使用哪种策略
* 会有一定的复杂性，不能陷入为了设计而设计


# 喜欢文章请关注我  
  
> 程序领域  
**点击关注+转发，私信发送【面试】或者【资料】可以收获更多资源**

![公众号](https://torgor.github.io/styles/images/my-public-ma.png)

![赞赏码](https://torgor.github.io/styles/images/my-zanshang-ma.png)








