## Java 线程池

### 几个核心参数的作用

- **corePoolSize：** 线程池核心线程数最大值
- **maximumPoolSize：** 线程池最大线程数大小
- **keepAliveTime：** 线程池中非核心线程空闲的存活时间大小
- **unit：** 线程空闲存活时间单位
- **workQueue：** 存放任务的阻塞队列
- **threadFactory：** 用于设置创建线程的工厂，可以给创建的线程设置有意义的名字，可方便排查问题。
- **handler：**  线城池的饱和策略事件，主要有四种类型。

## 线程池类型

https://juejin.im/post/5d1882b1f265da1ba84aa676

Java通过Executors提供四种线程池，分别为：

**newCachedThreadPool**创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。（重点在灵活回收，如果一个线程执行完60s没有新的任务则会被回收）
**newFixedThreadPool** 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
**newScheduledThreadPool** 创建一个定长线程池，支持定时及周期性任务执行。
**newSingleThreadExecutor** 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

## **为什么要用线程池:**

1.减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务。

2.可以根据系统的承受能力，调整线程池中工作线线程的数目，防止因为消耗过多的内存，而把服务器累趴下(每个线程需要大约1MB内存，线程开的越多，消耗的内存也就越大，最后死机)。

## 异常处理

![img](https://user-gold-cdn.xitu.io/2019/7/14/16bec33ca5559c93?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## **线程池都有哪几种工作队列？**

- ArrayBlockingQueue（定长）
- LinkedBlockingQueue（可定长可无限长）
- DelayQueue（延时队列，用来Schedule）
- PriorityBlockingQueue（优先级队列）
- SynchronousQueue（同步，单线程）

## 线程池状态

线程池有这几个状态：RUNNING,SHUTDOWN,STOP,TIDYING,TERMINATED。

![img](https://user-gold-cdn.xitu.io/2019/7/15/16bf3b10e39a52d0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**RUNNING**

- 该状态的线程池会接收新任务，并处理阻塞队列中的任务;
- 调用线程池的shutdown()方法，可以切换到SHUTDOWN状态;
- 调用线程池的shutdownNow()方法，可以切换到STOP状态;

**SHUTDOWN**

- 该状态的线程池不会接收新任务，但会处理阻塞队列中的任务；
- 队列为空，并且线程池中执行的任务也为空,进入TIDYING状态;

**STOP**

- 该状态的线程不会接收新任务，也不会处理阻塞队列中的任务，而且会中断正在运行的任务；
- 线程池中执行的任务为空,进入TIDYING状态;

**TIDYING**

- 该状态表明所有的任务已经运行终止，记录的任务数量为0。
- terminated()执行完毕，进入TERMINATED状态

**TERMINATED**

- 该状态表示线程池彻底终止

## 两种提交任务的方法

`ExecutorService` 提供了两种提交任务的方法：

1. `execute()`：提交不需要返回值的任务
2. `submit()`：提交需要返回值的任务