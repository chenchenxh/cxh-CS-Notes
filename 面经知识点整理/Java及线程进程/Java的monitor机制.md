[Java 中的 Monitor 机制](https://segmentfault.com/a/1190000016417017)

## 基本元素

1. 临界区
2. monitor 对象及锁
3. 条件变量以及定义在 monitor 对象上的 wait，signal 操作。

- 为了做到能够阻塞无法进入临界区的 进程/线程，还需要一个 monitor object 来协助，这个 monitor object 内部会有相应的数据结构，例如列表，来保存被阻塞的线程；同时由于 monitor 机制本质上是基于 mutex 这种基本原语的，所以 monitor object 还必须维护一个基于 mutex 的锁。

- 此外，为了在适当的时候能够阻塞和唤醒 进程/线程，还需要引入一个条件变量，

## Java的支持

Java 语言中的 **java.lang.Object 类，便是满足这个要求的对象**，任何一个 Java 对象都可以作为 monitor 机制的 monitor object。java.lang.Object 类定义了 wait()，notify()，notifyAll() 方法，这些方法的具体实现，依赖于一个叫 **ObjectMonitor 模式**的实现，这是 JVM 内部基于 C++ 实现的一套机制，基本原理如下所示：

![MonitorObject.png](https://segmentfault.com/img/remote/1460000016417020)