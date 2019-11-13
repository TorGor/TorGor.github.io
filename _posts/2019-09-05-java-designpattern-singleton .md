---
layout: post
title:  设计模式-单例
date:   2019-09-05 00:00:00 +0800
categories: document
tag: designpattern
---

* content
{:toc}


> 带着问题思考：
1. 单例模式存在的意义?
2. 单例模式的几种写法?
3. 单例模式在实战中的使用如何？
4. 如何应对面试官？


# 为什么要用单例模式

单例模式要写都很简单，都知道最终就是返回一个单实例的对象.
那么为什么要那么折腾？因为这样可以确保 java 内存中只有一个这个类的实例，减少了内存开支。
可能大家觉得这才多少内存? 试想一下，1000 个对象，在同一个程序里面有大量的线程同时使用且 GC 没有及时回收，这会造成多大的资源占用。

## Java对象的大小

基本数据的类型的大小是固定的，这里就不多说了。对于非基本类型的Java对象，其大小就值得商榷。
在Java中，一个空Object对象的大小是8byte，这个大小只是保存堆中一个没有任何属性的对象的大小。

看下面语句：
Object ob = new Object();
这样在程序中完成了一个Java对象的生命，但是它所占的空间为：4byte+8byte。4byte是Java栈中保存引用的所需要的空间。

而那8byte则是Java堆中对象的信息。因为所有的Java非基本类型的对象都
需要默认继承Object对象，因此不论什么样的Java对象，其大小都必须是大于8byte。

有了Object对象的大小，我们就可以计算其他对象的大小了。
Class NewObject {
int count;
boolean flag;
}
其大小为：空对象大小(8byte)+int大小(4byte)+Boolean大小(1byte)+空Object引用的大小(4byte)=17byte。按照最小8byte的计算=24 byte。
这还只是基本类型的对象，封装类型的最少16byte，也就是基本翻倍。在实际项目中，我们一个对象的大小一般都是超过 1024 byte。

# 单例模式的几种写法
* 饿汉模式
* 懒汉模式
* 枚举模式
* 内部类模式

## 饿汉模式
特点：
1. 私有的构造器
2. 私有静态的对象本身属性，且初始化 new 对象本身
3. 公有的获取对象的方法

* 优点：从它的实现中我们可以看到，这种方式的实现比较简单，在类加载的时候就完成了实例化，避免了线程的同步问题。
* 缺点：由于在类加载的时候就实例化了，所以没有达到Lazy Loading(懒加载)的效果，也就是说可能我没有用到这个实例，但是它
  也会加载，会造成内存的浪费(但是这个浪费可以忽略，所以这种方式也是推荐使用的)。

```java
public class SingletonEHan {

    private SingletonEHan() {}

    /**
     * 单例模式的饿汉式[可用]
     */
    private static SingletonEHan singletonEHan = new SingletonEHan();

    public static SingletonEHan getInstance() {
        return singletonEHan;
    }
}
```

## 懒汉模式（线程不安全模式），不推荐使用

* 缺点：线程不同步，可能会产生多个实例，因此不推荐使用，这里贴上代码主要用来比较写法。

```java
public class SingletonLanHan {
        
    private SingletonLanHan() { }
    /**
     * 单例模式的懒汉式[线程不安全，不可用]
     */
    private static SingletonLanHan singletonLanHan;

    public static SingletonLanHan getInstance() {
        if (singletonLanHan == null) { // 这里线程是不安全的,可能得到两个不同的实例
            singletonLanHan = new SingletonLanHan();
        }
        return singletonLanHan;
    }
}
```

## 懒汉模式（线程安全，但是效率不高）

* 优点：延迟加载，需要使用的时候才实例化，不会浪费性能，但其实浪费也是微乎其微。线程安全，保证只有一个实例。
* 缺点：每次使用 getSingletonLanHanTwo 方法的时候都会加锁。

其实这个方法里面只要求第一次实例化对象的时候加锁，然后返回对象，没有必要在每次请求的时候都加锁。这样我们还可以继续优化代码，做到双重验证+实例化时加锁。
 
```java
public class SingletonLanHan {
        
    private SingletonLanHan() { }


    /**
     * 4. 懒汉式线程安全的:[线程安全，效率低不推荐使用]
     * <p>
     * 缺点：效率太低了，每个线程在想获得类的实例时候，执行getInstance()方法都要进行同步。
     * 而其实这个方法只执行一次实例化代码就够了，
     * 后面的想获得该类实例，直接return就行了。方法进行同步效率太低要改进。
     */
    private static SingletonLanHan singletonLanHanTwo;

    public static synchronized SingletonLanHan getSingletonLanHanTwo() {
        if (singletonLanHanTwo == null) { // 这里线程是不安全的,可能得到两个不同的实例
            singletonLanHanTwo = new SingletonLanHan();
        }
        return singletonLanHanTwo;
    }

}
```

## 懒汉模式（双重验证+锁）
特点：
1. 私有的构造器
2. 私有静态的对象本身属性
3. 公有的获取对象的方法

* 优点：延迟加载，需要使用的时候才实例化，不会浪费性能，但其实浪费也是微乎其微。线程安全，双重验证机制，可以保证线程同步且只会实例一个对象；
  
  ```java
  public class SingletonLanHan {
    
    private SingletonLanHan() {  
    }

    /**
     * 单例模式懒汉式双重校验锁[推荐用]
     * 懒汉式变种,属于懒汉式的最好写法,保证了:延迟加载和线程安全
     */
    private static SingletonLanHan singletonLanHanFour;

    public static SingletonLanHan getSingletonLanHanFour() {
        if (singletonLanHanFour == null) {
            synchronized (SingletonLanHan.class) {
                if (singletonLanHanFour == null) {
                    singletonLanHanFour = new SingletonLanHan();
                }
            }
        }
        return singletonLanHanFour;
    }
  }
  ```
  
 ## 枚举模式
 
 懒汉模式、饿汉模式 真的万无一失吗？不见得。
 
* 私有化构造器并不保险
  >“享有特权的客户端可以借助AccessibleObject.setAccessible方法，通过反射机制调用私有构造器。如果需要低于这种攻击，可以修改构造器，让它在被要求创建第二个实例的时候抛出异常。
* 序列化问题
  > 任何一个readObject方法，不管是显式的还是默认的，它都会返回一个新建的实例，这个新建的实例不同于该类初始化时创建的实例。”当然，这个问题也是可以解决的，想详细了解的同学可以翻看《effective java》第77条：对于实例控制，枚举类型优于readResolve

 枚举模式：
 
 * 优点：避免同步问题；防止反序列化重新创建对象。
  
```java
  /**
   * Created by holy
   * 枚举 [极推荐使用]
   *
   * 这里SingletonEnum.INSTANCE
   * 这里的INSTANCE 即为 SingletonEnum 类型的引用所以得到它就可以调用枚举中的方法了。
   * 借助JDK1.5中添加的枚举来实现单例模式。不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象
   */
  
  public enum SingletonEnum {
  
      INSTANCE;
  
      private SingletonEnum() {
      }
  
      public void whateverMethod() {
  
      }
  
  }

```
  
  上面的代码在编译后，会编译成 
```java
public final class SingletonEnum extends java.lang.Enum<SingletonEnum> {
    
    public static final SingletonEnum instance;

    public static SingletonEnum[] values();


    public static SingletonEnum valueOf(java.lang.String);


    public void whateverMethod();


    static {}
}
```
  > Java规范中规定，每个枚举类型及其定义的枚举变量在JVM中都是唯一的.
  
  因此在枚举类型的序列化和反序列化上，
  Java做了特殊的规定。在序列化的时候Java仅仅是将枚举对象的name属性输到结果中，反序列化的时候则是通过
  java.lang.Enum的valueOf()方法来根据名字查找枚举对象。也就是说，序列化的时候只将属性名称输出，
  反序列化的时候再通过这个名称，查找对应的枚举类型，因此反序列化后的实例也会和之前被序列化的对象实例相同。
  
  
  
  
 ## 内部类模式（推荐）
 
 这种方式跟饿汉式方式采用的机制类似，但又有不同。
 两者都是采用了类装载的机制来保证初始化实例时只有一个线程。
 不同的地方:
 在饿汉式方式是只要Singleton类被装载就会实例化,
 内部类是在需要实例化时，调用getInstance方法，才会装载SingletonHolder类
 
 * 优点：避免了线程不安全，延迟加载，效率高。
 
 ```java

    public class SingletonIn {
    
        private SingletonIn() {
        }
    
        private static class SingletonInHodler {
            private static SingletonIn singletonIn = new SingletonIn();
        }
    
        public static SingletonIn getSingletonIn() {
            return SingletonInHodler.singletonIn;
        }
    }
```

# spring 中获取单例源码，采用的哪种方式？在面试中如何应对？

```java

    @Nullable
    protected Object getSingleton(String beanName, boolean allowEarlyReference) {
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null && this.isSingletonCurrentlyInCreation(beanName)) {
            Map var4 = this.singletonObjects;
            synchronized(this.singletonObjects) {
                singletonObject = this.earlySingletonObjects.get(beanName);
                if (singletonObject == null && allowEarlyReference) {
                    ObjectFactory<?> singletonFactory = (ObjectFactory)this.singletonFactories.get(beanName);
                    if (singletonFactory != null) {
                        singletonObject = singletonFactory.getObject();
                        this.earlySingletonObjects.put(beanName, singletonObject);
                        this.singletonFactories.remove(beanName);
                    }
                }
            }
        }
        return singletonObject;
    }
```
从源码分析来看，采用的就是双重验证+锁机制。现在为止，我们接触了单例的几种写法，也明白了其中的优点和缺点，在spring 中的源码我们也看了，
在面对面试官的提问的时候，就可以尽情聊（chui）天（niu）了。
我也面试过很多人，其实作为一个面试官，在单例这个问题上，只要回答到他们一些优缺点就感觉很不错了，因为说明真正理解了这种模式且对 JVM 
有一定的理解，如果能够详细的做出详细比较并说出自己的理解，就算非常优秀了。

程序领域（id：think-holy）
作者：holy
