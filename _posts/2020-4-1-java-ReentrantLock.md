---
layout: post
title:  并发编程: AbstractQueuedSynchronizer（AQS）使用，基于可重入锁 ReentrantLock 源码分析
date:   2020-1-29 00:00:00 +0800
categories: document
tag: lock
---
# AQS

AQS ：AbstractQueuedSynchronizer的简称。
AQS提供了一种手动实现锁功能，使用状态管理、FIFO（先入先出队列）等待队列等实现一个同步器功能。
如果 Synchronized 关键字比喻成是一个自动挡汽车，提供自动加锁一系列平顺感，
那么 AbstractQueuedSynchronizer 就是一个手动挡的变速箱，提供一些手动加锁的灵活性和可控性。
乐趣包含：独占、共享、可重入、可自旋、可中断。

![先入先出队列](https://torgor.github.io/styles/images/lock/CLH-FIFO.png) 

# ReentrantLock 可重入锁的实现来分析 AQS

一个对象可以重复递归的获取本对象的锁，而不会出现死锁，这样的锁称为可重入锁。
ReentrantLock 就是这样的一个实现，他通过 AQS 实现状态管理和队列控制。
具体实现的方式就是通过状态位标识，当本线程占用时 state = 1，重入后 state + 1，每以此重入都会执行 +1 操作。
当释放锁的时候，每执行 release 释放操作，则 state -1 。直到 state = 0 结束，代表锁已经完全释放。
volatile 保证这个属性的可见性，可以被其他线程读取修改。

```
    /**
     *  AQS The synchronization state.
     */
    private volatile int state;
```

先来看看该类下的概况：

![可重入锁类结构](https://torgor.github.io/styles/images/lock/ReentrantLock-structure.png) 



## 构造函数

```
    /**
     * 默认的构造函数，采用非公平锁，因为非公平锁效率更高
     * This is equivalent to using {@code ReentrantLock(false)}.
     */
    public ReentrantLock() {
        sync = new NonfairSync();
    }

    /**
     * 构造函数根据参数选择是公平锁还是非公平锁
     * @param fair {@code true} if this lock should use a fair ordering policy
     */
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
    
```

从代码分析可以看到，ReentrantLock 的构造函数有两种：
* 第一种默认的，使用静态内部类属性 `sync = new NonfairSync();`，  默认使用非公平锁方式。
* 也可以通过 boolean 类型参数来指定，fair = true ， sync = 公平锁 .fair = false, sync = 非公平锁。

## 类关系:ReentrantLock 三个内部类 与 AQS
* Sync : 继承自 AbstractQueuedSynchronizer，大名鼎鼎 AQS
* NonfairSync : 非公平同步器实现，继承 Sync
* FairSync ： 公平同步器实现，继承 Sync

### NonfairSync 非公平方式获取锁
```
    /**
     * Sync object for non-fair locks
     */
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * 如果当前 state = 0 ，表示没有线程占有，直接获取锁
         * 如果CAS 失败，则表示抢占失败，需要入对列进行等待，也就是公平锁方式
         */
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        } 
    }
```
### NonfairSync.lock()
NonfairSync.lock() 方式获取，以非公平的方式抢占。当调用 lock 方法时，会进行判断，当前是否有线程占用情况，
利用 unsafe 类的 CAS 比较交换 ，如果当前值是 0 ，符合预期的 expect 参数 0 ，那么将 0 替换为 1 。
如果设置值成功，说明当期没有线程占用，当前线程可以直接获取锁，并设置当前线程为排他锁模式 `setExclusiveOwnerThread`，
这个过程就是抢占，如果CAS操作失败，调用 `acquire（1）`，表示没有抢占成功，会插入 queue 等待，以公平锁方式加入FIFO。

下面看下 AQS 的 acquire 具体实现方式：

```
    /**
     * AQS 中的 acquire ,可以看到有如下判断
     * ① 子类中的 tryAcquire 是否成功：true 
     * 
     */
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```

在 `acquire（1）`函数中会有两个判断条件来决定是否调用 `selfInterrupt()` 来阻塞当前线程。
1. `tryAcquire（1）` 这个方法由子类实现，也就是 NonfairSync 里面的实现。
2. `acquireQueued(addWaiter(Node.EXCLUSIVE), arg)` 这一行代码博大精深，还是待会结合源码具体分析。
也就是说，如果竞争获取 state 失败，并且已经将本线程封装为 Node 加入 FIFO 队列，那么就说明在等待其他线程先执行，需要挂起本线程等待。


### NonfairSync.tryAcquire()

tryAcquire() 方式尝试获取，拿到或者拿不到锁都会直接返回结果。
这个方法在 lock() 里面也会调用，就是在抢占锁失败后走的流程。

从代码中可以看出，`tryAcquire()` 调用的是父类  Sync  `nonfairTryAcquire` 方法,而 sync 实现的是 AQS 中的 tryAcquire()模板方法.

```
        /**
         * Performs non-fair tryLock.  tryAcquire is implemented in
         * subclasses, but both need nonfair try for trylock method.
         */
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();                 // 当前状态
            if (c == 0) {                       // 如果是 0 表示没有线程占用
                if (compareAndSetState(0, acquires)) { // CAS 替换值是否成功
                    setExclusiveOwnerThread(current);  // 将当前线程设置为排他锁模式
                    return true;                // 返回结果 true
                }
            }
            else if (current == getExclusiveOwnerThread()) {   // 当前线程就是正在执行的排他锁模式线程
                int nextc = c + acquires;                      // 通过 state + 1 来进行计数，表示重入层数，没重入一层 +1 
                if (nextc < 0) // overflow                     
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);                               // 设置值
                return true;                                   // 返回结构 true 重入成功
            }
            return false;
        }

```

### AQS addWaiter

当tryAcquire方法获取锁资源（state）失败以后，则会先调用addWaiter将当前线程封装成Node，然后添加到 FIFO 队列

```
    private Node addWaiter(Node mode) {     //mode=Node.EXCLUSIVE
        //将当前线程封装成Node，并且mode为独占锁
        Node node = new Node(Thread.currentThread(), mode); 
        
        // tail是AQS的中表示同步队列队尾的属性，刚开始为null，所以进行enq(node)方法
        Node pred = tail;
        if (pred != null) {                 //tail不为空的情况，说明队列中存在节点数据
            node.prev = pred;               //将当前线程的 Node 的 prev 前置节点指向 tail 尾部节点
            if (compareAndSetTail(pred, node)) {    //通过cas将 node 添加到 FIFO 队列
                pred.next = node;           //cas成功，把旧的tail的next指针指向新的tail
                return node;
            }
        }
        enq(node);                          //tail=null，将node添加到同步队列中
        return node;
    }
```

* 第一步：将当前线程封装成 Node，并且为独占模式
* 判断当前链表中的 tail 尾部节点是否为空，如果不为空，则通过 CAS 操作把当前线程的 Node 添加到 FIFO 队列尾部
* 如果为空 或者 CAS 失败，调用 `enq (node)` 方法自旋来继续加入队列

### AQS.enq()

```
    /**
     * Inserts node into queue, initializing if necessary. See picture above.
     * @param node the node to insert
     * @return node's predecessor
     */
    private Node enq(final Node node) {
        for (;;) {                              // 自旋
            Node t = tail;                      // 尾部节点
            if (t == null) {                    // Must initialize 尾部是否为空
                if (compareAndSetHead(new Node()))  // 初始化设置头节点
                    tail = head;                // 初始化尾部节点
            } else {                            // 尾部节点不为空，队列存在数据
                node.prev = t;                  
                if (compareAndSetTail(t, node)) { // 将当前节点放到尾部节点，并返回当前尾部节点
                    t.next = node;
                    return t;
                }
            }
        }
    }

```
### AQS.acquireQueued()

使线程在等待队列中获取资源，一直获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false

```
    /**
     * Acquires in exclusive uninterruptible mode for thread already in
     * queue. Used by condition wait methods as well as acquire.
     */
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;                // 是否中断：初始false
            for (;;) {                                  // 自旋
                final Node p = node.predecessor();      // 当前线程的前置节点
                if (p == head && tryAcquire(arg)) {     // 如果当前节点是队列的第二个，可以理解 head 为队列第一个，表示可以争抢锁
                    setHead(node);                      // 设置头节点为当前节点
                    p.next = null; // help GC           
                    failed = false;                     
                    return interrupted;                 // 不需要中断
                }
                if (shouldParkAfterFailedAcquire(p, node) && // 如果获得锁失败，则根据waitStatus决定是否需要挂起线程
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);                    // 通过cancelAcquire取消获得锁的操作
        }
    }
```

* 自旋判断当前节点是不是 FIFO 队列中的第二个节点，这里默认 head 是 FIFO 的第一个节点
* 如果是，也就是 head 节点指向 node.predecessor() ,表示可以争取锁
* 获取锁成功，那么设置头节点为当前队列中的第二个节点
* 如果获取锁失败，则根据 waitStatus 决定是否需要挂起线程
* 最后通过 cancelAcquire 取消获得锁的操作


### AQS.shouldParkAfterFailedAcquire
从上面的分析可以看出，如果不是队列的第二个节点，也就是 FIFO 其他的节点，
会进行 shouldParkAfterFailedAcquire(p, node) 来进行判断，当前争取锁的线程是否需要在获取失败后阻塞。
只有当 waitStatus == Node.SIGNAL 是才返回 true

``` 
        /**
         * Checks and updates status for a node that failed to acquire.
         * Returns true if thread should block. This is the main signal
         * control in all acquire loops.  Requires that pred == node.prev.
         *
         * @param pred node's predecessor holding status
         * @param node the node
         * @return {@code true} if thread should block
         */
        private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
            int ws = pred.waitStatus;
            if (ws == Node.SIGNAL)                  // 如果前置节点的状态为 -1 ，表示可以前节点准备取出，后续节点 unparking
                /*
                 * This node has already set status asking a release
                 * to signal it, so it can safely park.
                 */
                return true;
            if (ws > 0) {                           // 如果状态 = 1 说明节点已经释放
                /*
                 * Predecessor was cancelled. Skip over predecessors and
                 * indicate retry.
                 */
                do {
                    node.prev = pred = pred.prev;  // 前置节点是取消状态，将前置节点的前节点，重新赋值到当期节点的前置节点上
                } while (pred.waitStatus > 0);     // 重复次操作
                pred.next = node;                  // 
            } else {
                /*
                 * waitStatus must be 0 or PROPAGATE.  Indicate that we
                 * need a signal, but don't park yet.  Caller will need to
                 * retry to make sure it cannot acquire before parking.
                 */
                compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
            }
            return false;
        }
```

* 它首先判断一个节点的前置节点的状态是否为 Node.SIGNAL = -1 ，是，说明此节点的后续节点可以不用 park 了
* 前置节点是取消状态，将前置节点从链表上移除，使用前置节点的前一个节点赋值到当前节点的 prev 上面
* 重复第二步判断，直到链表中的前一个节点不为 1 ，并设置 prev.next ，链表的下一个节点为当前节点
* CAS 比较替换前置节点为 Node.SIGNAL = -1 

``` 
        /** 
        * indicate thread has cancelled
        * 该线程节点已释放（超时、中断），已取消的节点不会再阻塞
        */
        static final int CANCELLED =  1;
        /** 
        * indicate successor's thread needs unparking
        * 该线程的后续线程需要阻塞，即只要前置节点释放锁，就会通知标识为 SIGNAL 状态的后续节点的线程
        */
        static final int SIGNAL    = -1;
        /** 
        * indicate thread is waiting on condition
        * 该线程在condition队列中阻塞（Condition有使用）
        */
        static final int CONDITION = -2;
        /** 
        * indicate the next acquireShared should unconditionally propagate
        * 表示下一个获取的共享对象的等待状态值应该无条件传播 （CountDownLatch中有使用）
        * 该线程以及后续线程进行无条件传播的共享模式下， PROPAGATE 状态的线程处于可运行状态 
        */
        static final int PROPAGATE = -3;
```

### AQS.parkAndCheckInterrupt
如果 shouldParkAfterFailedAcquire() 返回了true，则会执行：parkAndCheckInterrupt()方法，
它是通过LockSupport.park(this)将当前线程挂起到 WAITING 状态，它需要等待一个中断、unpark方法来唤醒它，
通过这样一种 FIFO 的机制的等待，来实现了 Lock 的操作。
``` 
private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted();
}
```

LockSupport
LockSupport类是Java6引入的一个类，提供了基本的线程同步原语。LockSupport实际上是调用了Unsafe类里的函数，归结到Unsafe里，只有两个函数：

``` 
    public native void unpark(Thread jthread);  
    public native void park(boolean isAbsolute, long time);  

```

LockSupport是用来创建锁和其他同步类的基本线程阻塞原语。LockSupport 提供park()和unpark()方法实现阻塞线程和解除线程阻塞，
LockSupport和每个使用它的线程都与一个许可(permit)关联。permit相当于1，0的开关，默认是0，调用一次unpark就加1变成1，
调用一次park会消费permit, 也就是将1变成0，同时park立即返回。再次调用park会变成block（因为permit为0了，会阻塞在这里，
直到permit变为1）, 这时调用unpark会把permit置为1。每个线程都有一个相关的permit, permit最多只有一个，重复调用unpark也不会积累

### ReentrantLock.tryRelease 释放锁

``` 
        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
```

上面看到release方法首先会调用ReentrantLock的内部类Sync的tryRelease方法。而通过下面代码的分析，大概知道tryRelease做了这些事情。
1. 获取当前AQS的state，并减去1；
2. 判断当前线程是否等于 AQS 的exclusiveOwnerThread，如果不是，就抛IllegalMonitorStateException异常，这就保证了加锁和释放锁必须是同一个线程；
3. 如果(state-1)的结果不为0，说明锁被重入了，需要多次unlock，这也是lock和unlock成对的原因；
4. 如果(state-1)等于0，我们就将AQS的ExclusiveOwnerThread设置为null；
5. 如果上述操作成功了，也就是tryRelase方法返回了true；返回false表示需要多次unlock。

## 模板方法模式应用

``` 
    /**
    * 这个方法是定义一个模板方法供子类实现，既然是模板方法，为什么没有声明 abstract ？
    */
    protected boolean tryAcquire(int arg) {
        throw new UnsupportedOperationException();
    }
```

仔细看这个方法，虽然使用了模板方法模式，却没有使用 abstract 。这点确实有意思，可以先停下思考一下。

先来简单介绍下模板方法模式：
规定好一些公用方法的主流程、算法而不具体实现，只是对流程进行控制，子类对具体的流程进行定义，这样保证其子类实现也按照流程和算法架构进行实现。

举个例子：抽象汽车公用方法 run() 包含以下功能
* start() --- 启动汽车 
* init()  --- 初始化功能 : 可以自定义，打开空调，打开收音机，播放音乐，打开天窗...
* go()    --- 前进：可以自定义，加速前进，百公里 0.1 s
* end()   --- 熄火

BMW888 车子是抽象汽车的具体实现，但是 run() 方法一样要遵守主流程。

再来回顾上面的问题，为啥 AQS 里 `tryAcquire` 是模板方法却不定义成 `abstract`？
因为独占模式下只用实现tryAcquire/tryRelease，而共享模式下只用实现tryAcquireShared/tryReleaseShared。
如果都定义成abstract，那么每个模式也要去实现另一模式下的接口。这样设计不符合设计规范，
违反了接口隔离原则（Interface Segregpation Principle），仔细想想也是不合理的。

接口隔离原则：每个接口中不存在子类用不到却必须实现的方法，如果不然，就要将接口拆分

程序领域（id：think-holy）
作者：holy 程序汪一只


# 喜欢文章请关注我
> 程序领域

![公众号](https://torgor.github.io/styles/images/my-public-ma.png)
![赞赏码](https://torgor.github.io/styles/images/my-zanshang-ma.png)















