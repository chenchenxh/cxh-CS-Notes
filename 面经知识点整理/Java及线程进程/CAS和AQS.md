[TOC]

## 参考资料

[CAS和AQS的初步理解](https://blog.csdn.net/u010862794/article/details/72892300)

讲java多线程最好的还是这个教程，很清晰很详细：[Java多线程](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/Multithread)

**两者也都是java并发编程的一部分**

## CAS

### CAS（Compare And Swap）

比较交换。是为了解决多线程并行下使用锁的性能损耗较大的一种机制。**CAS包含三个操作数--内存位置（V）、预期原值（A）和新值（B）**。如果内存位置的值和预期原值相匹配，那么赋予新值，如果不同则取出当前的值。

- 使用：java的atomic类，比如AtomicInteger和AtomicBoolean等。
- 属于**乐观锁**的一种。比悲观锁的性能损耗低（如果资源竞争非常激烈的话，其实应该考虑降低资源竞争）
- Java中，**sum.misc.Unsafe**这个类提供了硬件级别的原子操作来实现CAS，所以可以直接使用Unsafe这个方法。

### Atomic使用

利用CAS+volatile+native的原子方法完成功能。

 ```
  public final int get() //获取当前的值
  public final int getAndSet(int newValue)//获取当前的值，并设置新的值
  public final int getAndIncrement()//获取当前的值，并自增
  public final int getAndDecrement() //获取当前的值，并自减
  public final int getAndAdd(int delta) //获取当前的值，并加上预期的值
  boolean compareAndSet(int expect, int update) //如果输入的数值等于预期值，则以原子方式将该值设置为输入值（update）
  public final void lazySet(int newValue)//最终设置为newValue,使用 lazySet 设置之后可能导致其他线程在之后的一小段时间内还是可以读到旧的值。
 ```

### 缺点

- 如果资源竞争大，循环时间长
- 只能保证一个共享变量的原子操作
- ABA问题：从A改成B再改成A会导致判断错误。解决方法有使用**带标记的原子类**“AtomicStampedReference”。

## AQS

Abstract Queued Synchronizer

基于FIFO队列的阻塞锁和同步器的框架

[AQS详解](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/Multithread/AQS.md)

**AQS核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是用 CLH 队列锁实现的，即将暂时获取不到锁的线程加入到队列中。**

- 使用：CountDownLatch和Semaphore都有继承自这个抽象类的成员。ReetrantLock和FutureTask也是（FutureTask现版本已经修改了，不使用AQS了）

> CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS 是将每条请求共享资源的线程封装成一个 CLH 锁队列的一个结点（Node）来实现锁的分配。

### AQS的状态

AQS的state状态就是使用的上面的CAS来实现的。

### AQS对资源的两种共享方式

- Exclusive（独占）

  如ReentrantLock，又可以分为**公平锁和非公平锁**（公平锁和非公平锁的区别见下一章）

- Share（共享）

  如Semaphore、countDownLatch、CyclicBarrier、ReadWriteLock。允许多个线程访问同个资源

### AQS底层的模板方法模式

> 模板方法模式是基于“继承”的，跟接口不同。

AQS同步器的设计是基于模板方法模式的。即给了一个**AbstractQueuedSynchronizer模板**，然后重写这个模板的方法来实现独立的功能。

需要重写的几个方法如下，没有重写的话默认throw UnsupportedOperationException，而且**除了这几个方法，其他方法都是final不可重写的**

```
isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
```

### CountDownLatch、Semaphore、CyclicBarrier原理

根据上面的分析，这些工具的原理：

- CountDownLatch：N个子线程执行，那么state初始化为N，然后每个线程执行countdown，那么state就CAS减一（**不断循环CAS，直到减一成功**），等到state减到0，那么await取消阻塞（其实是**每次countdown都会去检查state是否为0，为0则调用doReleaseShared()，去唤醒await的线程**）
- Semaphore同理，只是实现的模板是可以加可以减（信号量的获取和释放）
- CyclicBarrier同理，但是比起CountDownLatch，这个自动reset，即每次state=0则reset。

## 公平锁和非公平锁

公平锁：第一次申请判断资源是否占用，如果占用则排队

非公平锁：**第一次申请就直接CAS抢锁**，如果成功则成功占用，否则进入tryAcquire方法，但是**tryAcquire的时候也会再次CAS抢锁一次**。再次失败就排队。（两次抢锁）