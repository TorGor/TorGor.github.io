---
layout: post
title:  聊聊你知道的锁
date:   2020-1-29 00:00:00 +0800
categories: document
tag: lock
---

* content
{:toc}

>开篇思考
1.	你知道哪些锁？
2.	锁解决了哪些应用场景的问题？
3.	锁的底层实现？
4.  java 中的并发包了解吗？
5.  CAS 会有哪些问题？如何解决？
6.  AQS 是并发包的基础，实现原理是什么？
7.  synchronize 是可重入锁吗？

如果上面的思考题都能直接准确回答，直接去面试吧。

# 锁
## 1. 悲观锁
并不是某一个锁，是一个锁类型，无论是否并发竞争资源，都会锁住资源，并等待资源释放下一个线程才能获取到锁。
这明显很悲观，所以就叫悲观锁。这明显可以归纳为一种策略，只要符合这种策略的锁的具体实现，都是悲观锁的范畴。

## 2. 乐观锁
与悲观锁相对的，也是一个锁类型。当线程开始竞争资源时，不是立马给资源上锁，而是进行一些前后值比对，以此来操作资源。

## 3. 无锁
显然，就是不给资源上锁，线程可以直接获取资源。

## 4. 自旋锁
自旋通俗的讲就是轮询，for(;;) 很好理解，就是一直循环等待资源释放后获取锁。

## 5. 偏向锁
初次执行到synchronized代码块的时候，锁对象变成偏向锁（通过CAS修改对象头里的锁标志位），字面意思是
“偏向于第一个获得它的线程”的锁。执行完同步代码块后，线程并不会主动释放偏向锁。当第二次到达同步代码块时，
线程会判断此时持有锁的线程是否就是自己（持有锁的线程ID也在对象头里），如果是则正常往下执行。由于之前没有释放锁，
这里也就不需要重新加锁。如果自始至终使用锁的线程只有一个，很明显偏向锁几乎没有额外开销，性能极高。

## 6. 轻量级锁
一旦有第二个线程加入锁竞争，偏向锁就升级为轻量级锁（自旋锁）。这里要明确一下什么是锁竞争：
如果多个线程轮流获取一个锁，但是每次获取锁的时候都很顺利，没有发生阻塞，那么就不存在锁竞争。只有当某线程尝试获取锁的时候，
发现该锁已经被占用，只能等待其释放，这才发生了锁竞争。在轻量级锁状态下继续锁竞争，没有抢到锁的线程将自旋，
即不停地循环判断锁是否能够被成功获取。获取锁的操作，其实就是通过CAS修改对象头里的锁标志位。
先比较当前锁标志位是否为“释放”，如果是则将其设置为“锁定”，比较并设置是原子性发生的。这就算抢到锁了，
然后线程将当前锁的持有者信息修改为自己。长时间的自旋操作是非常消耗资源的，一个线程持有锁，其他线程就只能在原地空耗CPU，
执行不了任何有效的任务，这种现象叫做忙等（busy-waiting）。如果多个线程用一个锁，但是没有发生锁竞争，或者发生了很轻微的锁竞争，
那么synchronized就用轻量级锁，允许短时间的忙等现象。这是一种折衷的想法，短时间的忙等，换取线程在用户态和内核态之间切换的开销。

## 7. 重量级锁
如果锁竞争情况严重，某个达到最大自旋次数的线程（一般10次），会将轻量级锁升级为重量级锁
（依然是CAS修改锁标志位，但不修改持有锁的线程ID）。
当后续线程尝试获取锁时，发现被占用的锁是重量级锁，则直接将自己挂起（而不是忙等），等待将来被唤醒。在JDK1.6之前，
synchronized直接加重量级锁，很明显现在得到了很好的优化。

## 8. 可重入锁  
可重入锁的字面意思是“可以重新进入的锁”，即允许同一个线程多次获取同一把锁。比如一个递归函数里有加锁操作，
递归过程中这个锁会阻塞自己吗？如果不会，那么这个锁就是可重入锁（因为这个原因可重入锁也叫做递归锁）。

## 9. 公平锁  
多个线程竞争同一把锁，如果依照先来先得的原则，那么就是一把公平锁。

## 10. 非公平锁  
多个线程竞争锁资源，如果是抢占式的，谁都可以先上，那么显然不公平，我先来的凭什么要插队，无耻。

## 11. 可中断锁  
字面意思是“可以响应中断的锁”。这里的关键是理解什么是中断。Java并没有提供任何直接中断某线程的方法，
只提供了中断机制。何谓“中断机制”？线程A向线程B发出“请你停止运行”的请求（线程B也可以自己给自己发送此请求），
但线程B并不会立刻停止运行，而是自行选择合适的时机以自己的方式响应中断，也可以直接忽略此中断。也就是说，
Java的中断不能直接终止线程，而是需要被中断的线程自己决定怎么处理。如果线程A持有锁，线程B等待获取该锁。由于线程A持有锁的时间过长，
线程B不想继续等待了，我们可以让线程B中断自己或者在别的线程里中断它，这种就是可中断锁。在Java中，synchronized就是不可中断锁，
而Lock的实现类都是可中断锁，可以简单看下Lock接口。

```java
/* Lock接口 */
public interface Lock {

    void lock(); // 拿不到锁就一直等，拿到马上返回。

    void lockInterruptibly() throws InterruptedException; // 拿不到锁就一直等，如果等待时收到中断请求，则需要处理InterruptedException。

    boolean tryLock(); // 无论拿不拿得到锁，都马上返回。拿到返回true，拿不到返回false。

    boolean tryLock(long time, TimeUnit unit) throws InterruptedException; // 同上，可以自定义等待的时间。

    void unlock();

    Condition newCondition();
}
```
## 12. 读写锁，互斥锁，共享锁
读写锁其实是一对锁，一个读锁（共享锁）和一个写锁（互斥锁、排他锁）
```java
 /** @see ReentrantReadWriteLock
 * @see Lock
 * @see ReentrantLock
 *
 * @since 1.5
 * @author Doug Lea
 */
public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     *
     * @return the lock used for reading
     */
    Lock readLock();

    /**
     * Returns the lock used for writing.
     *
     * @return the lock used for writing
     */
    Lock writeLock();
}
```
记得之前的乐观锁策略吗？所有线程随时都可以读，仅在写之前判断值有没有被更改。


读写锁其实做的事情是一样的，但是策略稍有不同。很多情况下，线程知道自己读取数据后，是否是为了更新它。
那么何不在加锁的时候直接明确这一点呢？如果我读取值是为了更新它（SQL的for update就是这个意思），
那么加锁的时候就直接加写锁，我持有写锁的时候别的线程无论读还是写都需要等待；如果我读取数据仅为了前端展示，
那么加锁时就明确地加一个读锁，其他线程如果也要加读锁，不需要等待，可以直接获取（读锁计数器+1）。


虽然读写锁感觉与乐观锁有点像，但是读写锁是悲观锁策略。因为读写锁并没有在更新前判断值有没有被修改过，
而是在加锁前决定应该用读锁还是写锁。乐观锁特指无锁编程。
JDK提供的唯一一个ReadWriteLock接口实现类是ReentrantReadWriteLock。看名字就知道，它不仅提供了读写锁，
而是都是可重入锁。 除了两个接口方法以外，ReentrantReadWriteLock还提供了一些便于外界监控其内部工作状态的方法，
这里就不一一展开。

## java 中的悲观锁、乐观锁

我们在Java里使用的各种锁，几乎全都是悲观锁。synchronized从偏向锁、轻量级锁到重量级锁，全是悲观锁。
JDK提供的Lock实现类全是悲观锁。其实只要有“锁对象”出现，那么就一定是悲观锁。因为乐观锁不是锁，
而是一个在循环里尝试CAS的算法。
那JDK并发包里到底有没有乐观锁呢？很多。
java.util.concurrent.atomic包里面的原子类都是利用乐观锁实现的。
有。java.util.concurrent.atomic包里面的原子类都是利用乐观锁实现的。

```java
/**
     * Atomically sets the value to the given updated value
     * if the current value {@code ==} the expected value.
     *
     * @param expect the expected value
     * @param update the new value
     * @return {@code true} if successful. False return indicates that
     * the actual value was not equal to the expected value.
     */
    public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }

```

## ABA问题及解决方案
上面讲到 CAS 是通过比较替换的方式来做实际的更新。
这里存在一个问题，就是一个值从A变为B，又从B变回了A。这种情况下，CAS可能会认为值没有发生过变化，但实际上是有变化的。
对此，并发包下有AtomicStampedReference提供根据版本号判断的实现。
基本思路就是通过版本号来控制，这个也是乐观锁的常用解决方案。
数据库同样可以通过这种版本号的控制方式来实现乐观锁。

## AbstractQueuedSynchronizer （AQS ）抽象的队列式同步器，并发包的基础
简单解释一下J.U.C，是JDK中提供的并发工具包,java.util.concurrent。
里面提供了很多并发编程中很常用的实用工具类，比如atomic原子操作、比如lock同步锁、fork/join等。

AQS 的核心功能，就是用来将当前工作的线程设置为占有资源状态，并将资源状态设置为锁定，如果其他线程继续访问共享资源，
则需要使用队列将其他线程进行管理，这个队列并不是实例化的某种队列，只是一个 Node 节点的双向关联。


下面只列出一些 AQS 核心的东西，后面有机会详细解读 AQS 
1. FIFO 先入先出队列
2. 状态控制（volatile修饰共享变量state）
3. lock获取锁的过程：本质上是通过CAS来获取状态值修改，如果当场没获取到，会将该线程放在线程等待链表中
4. lock释放锁的过程：修改状态值，调整等待链表。

下面这段代码就是 AQS 静态内部类，Node：
```java
    static final class Node {
        /** Marker to indicate a node is waiting in shared mode */
        static final Node SHARED = new Node();
        /** Marker to indicate a node is waiting in exclusive mode */
        static final Node EXCLUSIVE = null;

        /** waitStatus value to indicate thread has cancelled */
        static final int CANCELLED =  1;
        /** waitStatus value to indicate successor's thread needs unparking */
        static final int SIGNAL    = -1;
        /** waitStatus value to indicate thread is waiting on condition */
        static final int CONDITION = -2;
        /**
         * waitStatus value to indicate the next acquireShared should
         * unconditionally propagate
         */
        static final int PROPAGATE = -3;

        /**
         * Status field, taking on only the values:
         *   SIGNAL:     The successor of this node is (or will soon be)
         *               blocked (via park), so the current node must
         *               unpark its successor when it releases or
         *               cancels. To avoid races, acquire methods must
         *               first indicate they need a signal,
         *               then retry the atomic acquire, and then,
         *               on failure, block.
         *   CANCELLED:  This node is cancelled due to timeout or interrupt.
         *               Nodes never leave this state. In particular,
         *               a thread with cancelled node never again blocks.
         *   CONDITION:  This node is currently on a condition queue.
         *               It will not be used as a sync queue node
         *               until transferred, at which time the status
         *               will be set to 0. (Use of this value here has
         *               nothing to do with the other uses of the
         *               field, but simplifies mechanics.)
         *   PROPAGATE:  A releaseShared should be propagated to other
         *               nodes. This is set (for head node only) in
         *               doReleaseShared to ensure propagation
         *               continues, even if other operations have
         *               since intervened.
         *   0:          None of the above
         *
         * The values are arranged numerically to simplify use.
         * Non-negative values mean that a node doesn't need to
         * signal. So, most code doesn't need to check for particular
         * values, just for sign.
         *
         * The field is initialized to 0 for normal sync nodes, and
         * CONDITION for condition nodes.  It is modified using CAS
         * (or when possible, unconditional volatile writes).
         */
        volatile int waitStatus;

        
        volatile Node prev;

        
        volatile Node next;

        /**
         * The thread that enqueued this node.  Initialized on
         * construction and nulled out after use.
         */
        volatile Thread thread;

        
        Node nextWaiter;

        /**
         * Returns true if node is waiting in shared mode.
         */
        final boolean isShared() {
            return nextWaiter == SHARED;
        }

        /**
         * Returns previous node, or throws NullPointerException if null.
         * Use when predecessor cannot be null.  The null check could
         * be elided, but is present to help the VM.
         *
         * @return the predecessor of this node
         */
        final Node predecessor() throws NullPointerException {
            Node p = prev;
            if (p == null)
                throw new NullPointerException();
            else
                return p;
        }

        Node() {    // Used to establish initial head or SHARED marker
        }

        Node(Thread thread, Node mode) {     // Used by addWaiter
            this.nextWaiter = mode;
            this.thread = thread;
        }

        Node(Thread thread, int waitStatus) { // Used by Condition
            this.waitStatus = waitStatus;
            this.thread = thread;
        }
    }
```
重点看下这个 ``volatile int waitStatus;``，这是一个volatile 修饰的int 状态类型，volatile 就是确保该变量是对其他线程可见的，
 是java 内存模型中的重要概念，不理解的可以去看下 JMM 。
 SIGNAL(-1)：表示后继结点在等待当前结点唤醒。后继结点入队时，会将前继结点的状态更新为SIGNAL。
 CANCELLED(1):表示当前结点已取消调度。当timeout或被中断（响应中断的情况下），会触发变更为此状态，进入该状态后的结点将不会再变化。
 CONDITION(-2):表示结点等待在Condition上，当其他线程调用了Condition的signal()方法后，CONDITION状态的结点将从等待队列转移到同步队列中，等待获取同步锁。
 PROPAGATE(-3):共享模式下，前继结点不仅会唤醒其后继结点，同时也可能会唤醒后继的后继结点。
 0:初始状态
 
 

## synchronize 是不是可重入锁
这个需要先理解可重入锁是怎么设计的。
重入锁实现可重入性原理或机制是：每一个锁关联一个线程持有者和计数器，当计数器为 0 时表示该锁没有被任何线程持有，
那么任何线程都可能获得该锁而调用相应的方法；当某一线程请求成功后，JVM会记下锁的持有线程，并且将计数器置为 1；
此时其它线程请求该锁，则必须等待；而该持有锁的线程如果再次请求这个锁，就可以再次拿到这个锁，同时计数器会递增；
当线程退出同步代码块时，计数器会递减，如果计数器为 0，则释放该锁。


# 喜欢文章请关注我    
> 程序领域  
**点击关注+转发，私信发送【面试】或者【资料】可以收获更多资源**

![公众号](https://torgor.github.io/styles/images/my-public-ma.png)