### synchronized与Lock的区别

我把两者的区别分类到了一个表中，方便大家对比：

| 类别     | synchronized                                                 | Lock                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 存在层次 | Java的关键字，在jvm层面上                                    | 是一个类                                                     |
| 锁的释放 | 1、以获取锁的线程执行完同步代码，释放锁 2、线程执行发生异常，jvm会让线程释放锁 | 在finally中必须释放锁，不然容易造成线程死锁                  |
| 锁的获取 | 假设A线程获得锁，B线程等待。如果A线程阻塞，B线程会一直等待   | 分情况而定，Lock有多个锁获取的方式，具体下面会说道，大致就是可以尝试获得锁，线程可以不用一直等待 |
| 锁状态   | 无法判断                                                     | 可以判断                                                     |
| 锁类型   | 可重入，不可中断，非公平                                     | 可重入 可判断 可公平（两者皆可）                             |
| 性能     | 少量同步                                                     | 大量同步                                                     |

## 锁的条件

- 不可抢占
- 持有等待
- 互斥条件
- 循环等待

## 锁的类型

- 公平锁/非公平锁
- 可重入锁（可重入锁又名递归锁，是指在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁。）
- 独享锁/共享锁
- 互斥锁/读写锁
- 乐观锁/悲观锁（乐观锁StampedLock，读线程非常多而写线程非常少）
- 分段锁
- 偏向锁/轻量级锁/重量级锁
- 自旋锁

## synchronize修改静态和非静态方法的不同

- **Synchronized修饰非静态方法，实际上是对调用该方法的对象加锁，俗称“对象锁”**

  修饰非静态方法，是对该对象加锁。例如Class Test里面synchronize一个方法，那么Test整个类都加锁了。除非new另外一个Test类

- **Synchronized修饰静态方法，实际上是对该类对象加锁，俗称“类锁”。**

这个可以这么理解：**静态方法不属于任何一个对象，所以同时访问同个类的静态方法和非静态方法是不会冲突的**

### 总结：synchronize有三种类型：对象锁，类锁，Object锁，方法锁

| 类型     | Synchronize修饰    |
| -------- | ------------------ |
| 类锁     | static方法；.class |
| 对象锁   | 非static方法；this |
| object锁 | Object             |

其中对象锁和类锁锁的区域完全不同，互相不影响。！对象锁只会锁该对象的同步非静态方法，类锁只会锁该对象的同步静态方法，**也就是说，Synchronize不会影响没有加Synchronize的方法的调用**

自己测试，发现这个Object锁只有在同时synchronized同个Object的时候才会有效果，和上面的对象锁和类锁都不同）

## Syschronized的底层实现原理

[synchronized底层语义原理](https://blog.csdn.net/javazejian/article/details/72828483#synchronized底层语义原理)

（这一篇教程讲得很好）

### Java对象头和Monitor

在JVM中，对象在内存中分为三块区域：对象头、实例数据和对齐填充。

![img](https://img-blog.csdn.net/20170603163237166?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF2YXplamlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 实例变量。存放类的属性数据信息，包括父类的属性信息，如果是数组则包含数组的长度
- 填充数据。直接的填充数据
- 对象头。2个字节（如果是数组则是三个字节），主要结构是由Mark Work和Class Metadata Address。
  - MarkWord：存储对象的hashcode，锁信息和分代年龄和GC标志灯
  - 类元数据：存储指向对象的类元数据，表示这个对象是哪个类

![img](https://img-blog.csdn.net/20170603172215966?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF2YXplamlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

Monitor（管程或监视器锁）

> 每个对象都存在着一个 monitor 与之关联，对象与其 monitor 之间的关系有存在多种实现方式。**当一个 monitor 被某个线程持有后，它便处于锁定状态。**(monitor是由ObjectMonitor实现功能，底层是C++代码)

![img](https://img-blog.csdn.net/20170604114223462?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF2YXplamlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



### Synchronized修饰原理

如果是this这类，那么就按照上述的方式去获取monitor。如果是修饰方法，那么就去**方法常量池的方法表结构中的ACC_SYNCHRONIZED标志判断一个方法是否是同步方法**。如果是，再执行对象锁的获取流程。

> 字节码中可知同步语句块的实现使用的是monitorenter 和 monitorexit 指令，

### JVM对锁的优化

> 锁的状态总共有四种，无锁状态、偏向锁、轻量级锁和重量级锁。

锁是随着竞争不断升级的。先看看偏向锁和轻量级锁的概念：

1. **偏向锁**。Java6加入的新锁。因为在大多数情况下，锁不仅不存在多线程竞争，甚至一直都是由同一个线程多次获得，那么就可以减少统一现场获得锁的代价--偏向锁。如果竞争激烈化，那么就转为轻量级锁。
2. **轻量级锁**。“对于绝大部分锁，在整个同步周期内都不存在竞争”，适应轻微为线程交替执行同步块的场合。如果同一时间内访问，那么就会膨胀为重量级锁。

另外还有一个自旋锁和锁消除的概念。

- **自旋锁**。基于“线程持有锁的时间一般不会很长”，所以让线程每次获得不到锁就挂起的代价过大，就让线程“自旋“--执行一定数量的循环等待锁。如果等待失败，再挂起到用户态。
- **锁消除**。消除锁的JVM的优化，JVM在JIT编译期间，通过上下文的判断，会取出 **不可能**存在资源竞争的锁，通过这种方式消除没有必要的锁。比如在一个线程中使用StringBuffer，可能会被优化为StringBuilder类型。

## 修饰静态方法和非静态方法的monitor

如果synchronized修饰静态方法，那么会持有的monitor是 **Class对象的monitor**

而非静态方法就是持有的monitor就是 **对应的对象的monitor**

