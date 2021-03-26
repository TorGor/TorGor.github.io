---
layout: post
title:  Mybatis 缓存，底层实现原理，源码中的设计模式，缓存击穿解决
date:   2020-11-1 00:00:00 +0800
categories: document
tag: mybatis
---

* content
{:toc}

>汇总
1. 一级缓存和二级缓存的使用会解决哪些问题，又会带来哪些问题  
2. 底层是如何实现缓存的  
3. 用到了哪些设计模式  
4. 缓存击穿如何解决  



# 缓存的概念

Mybatis 是现在的主流持久层框架之一，也是支持一级缓存、二级缓存的。  

缓存其实可以把它理解为数据存储在本地内存中，当以后再需要执行 sql 去获取数据的时候，就直接从内存中获取。

## 一级缓存

以 sqlsession 为维度，同一次会话内执行同一个 sqlmapper ，只会在第一次查询数据库，并将查询数据结果集存入 HashMap
中，后面的查询都会在缓存的 map 中拿取，提高了效率，减少了数据库压力。  

注意：
1. Mybatis 默认开启一级缓存
2. sqlsession 关闭，一级缓存就会失效，不同的 sqlsession 缓存互不影响
3. 如果执行了DML (Data Manipulation Language：以 insert，delete，update 指令为主) 操作，就会刷新缓存


## 二级缓存 

以 namespace 为维度，不同的 sqlsession 可以共享缓存。二级缓存默认不开启，需要在配置文件中设置开启。  


### mapper 文件中开启二级缓存：
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.holy.mybatis.dao.Mapper">
    <!--开启本mapper的namespace下的二级缓存-->
    <!--
        eviction:代表的是缓存回收策略，目前MyBatis提供以下策略。
        (1) LRU,最近最少使用的，一处最长时间不用的对象
        (2) FIFO,先进先出，按对象进入缓存的顺序来移除他们
        (3) SOFT,软引用，移除基于垃圾回收器状态和软引用规则的对象
        (4) WEAK,弱引用，更积极的移除基于垃圾收集器状态和弱引用规则的对象。这里采用的是LRU，
                移除最长时间不用的对形象

        flushInterval:刷新间隔时间，单位为毫秒，这里配置的是100秒刷新，如果你不配置它，那么当
        SQL被执行的时候才会去刷新缓存。

        size:引用数目，一个正整数，代表缓存最多可以存储多少个对象，不宜设置过大。设置过大会导致内存溢出。
        这里配置的是1024个对象

        readOnly:只读，意味着缓存数据只能读取而不能修改，这样设置的好处是我们可以快速读取缓存，缺点是我们没有
        办法修改缓存，他的默认值是false，不允许我们修改
    -->
    <cache eviction="LRU" flushInterval="100000" readOnly="true" size="1024"/>
</mapper>

```
### 注解开启缓存 

```
@CacheNamespace(blocking=true)
public interface PersonMapper {
 
  @Select("select id, firstname, lastname from person")
  public List<Person> findAll();
}
```

### 全局开启二级缓存

mybatis-config.xml 配置  ，但是这种全局的我并不建议开启，因为并不是所有的查询都需要缓存。
缓存应该用在经常使用的 sql 上面，这样才能提高效率。如果只是查询一次也使用了缓存，只会造成资源的浪费。  
  
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!--这个配置使全局的映射器(二级缓存)启用或禁用缓存-->
        <setting name="cacheEnabled" value="true" />
        .....
    </settings>
    ....
</configuration>

```

# Mybatis 缓存源码

Mybatis 缓存源码存放在 cache 目录

**CacheBuilder.java**   用来创建缓存。
```
  
  public Cache build() {
    // 这里设置原始缓存为默认缓存
    setDefaultImplementations();
    Cache cache = newBaseCacheInstance(implementation, id);
    setCacheProperties(cache);
    // issue #352, do not apply decorators to custom caches
    if (PerpetualCache.class.equals(cache.getClass())) {
      for (Class<? extends Cache> decorator : decorators) {
        cache = newCacheDecoratorInstance(decorator, cache);
        setCacheProperties(cache);
      }
      cache = setStandardDecorators(cache);
    } else if (!LoggingCache.class.isAssignableFrom(cache.getClass())) {
      cache = new LoggingCache(cache);
    }
    return cache;
  }

  private void setDefaultImplementations() {
    if (implementation == null) {
      implementation = PerpetualCache.class;
      if (decorators.isEmpty()) {
        decorators.add(LruCache.class);
      }
    }
  }
```

![目录](https://torgor.github.io/styles/images/mybatis/cache/cache-source-direct.png)


# Mybatis 设计模式

## 装饰器模式

装饰器模式的说明：使同一个接口为前提，动态地将额外的能力附加到对象上。  
装饰者提供了比继承更有弹性的替代方案。原文是：

> Attach additional responsibilities to an object dynamically keeping the same interface.
> Decorators provide a flexible alternative to subclassing for extending functionality. 

Mybatis 的默认的 cache 实现类是 ：PerpetualCache ，那么它如何实现 cache 额外能力的传递呢？  
在 cache 接口里，我们先把握一根主线，就是我们要获取内存数据，也就是 `Object getObject(Object key);`，
那么在这个接口上面，我们额外添加一个日志功能，包裹在 getObject(key) 方法调用上。

### 额外能力的传递 

在 Mybatis cache 的设计模式中，通过构造函数将其他 cache 的其他装饰器传入当前装饰器中。
所有的装饰器中通过对 getObject(key) 方法包裹，来实现额外的功能。下面就是 loggingCache 的装饰器实现。

``` 
public class LoggingCache implements Cache {  

  private Log log;  
  // 此处为传递的 cache 接口
  private Cache delegate;
  protected int requests = 0;
  protected int hits = 0;

  public LoggingCache(Cache delegate) {
    this.delegate = delegate;
    this.log = LogFactory.getLog(getId());
  }

  @Override
  public Object getObject(Object key) {
    requests++;
    final Object value = delegate.getObject(key);
    if (value != null) {
      hits++;
    }
    if (log.isDebugEnabled()) {
      log.debug("Cache Hit Ratio [" + getId() + "]: " + getHitRatio());
    }
    return value;
  }

}

```

从代码中可以看到，所谓装饰器就是对另一个接口的实现通过包裹模式，来实现功能的继承和传递。这样做的好处就是
相对灵活的功能叠加。举个多功能传递的例子：

``` 
Cache cache = new  PerpetualCache(id);
cache = new LoggingCache(cache);
cache = new LruCache(cache);
cache = new BlockingCache(cache);

```

从这里就可以看到，最终这个 cache 对象拥有了日志能力，LRU 最近最少使用淘汰策略，防止缓存击穿的阻塞能力。  

这里我想重点介绍下这个 LRU 算法实现，Mybatis 的LruCache 的内部代码是使用 LinkedHashMap 来实现的。  
为什么 LinkedHashMap 可以实现？ 这里简单说明下原理，其实很简单，就是我们访问一个 Node 的时候，会进行判断，
如果这个是排序类型的，就是有 orderType 这个参数构造，就在 get（key） 的时候，把最近访问的 node 放到链表的最后。
这样用来，排在最前面的就是最不常使用的 node，最后面的都是最新访问的。  

  








![目录](https://torgor.github.io/styles/images/mybatis/cache/decorator-cache-UML.png)

# 喜欢文章请关注我  
  
> 程序领域  
**点击关注+转发，私信发送【面试】或者【资料】可以收获更多资源**

![公众号](https://torgor.github.io/styles/images/my-public-ma.png)









