## ThreadLocal

！！**ThreadLocal新的理解：ThreadLocal通过操作Thread中的一个map，用来在Thread类中添加一个逻辑上的成员变量。**

先说总结的结论：

- **每个线程t都有对应的t.threadLocals**（这是一个map，内部是和hashmap类似用Entry数组维护）
- **这个map的key值是ThreadLocal<?>类，将这个类和？对应起来。**而？可以是任何类，因此ThreadLocal可以用来做线程安全，每个线程的各自的ThreadLocal的map都不一样，里面的东西自然不一样。

如下图：

![1](D:\typora\pic\1.png)

所以Looper的机制就很显而易见了，通过线程的threadLocals来存放Looper，myLooper就是从threadLocals中取出looper。

## Thread和Looper！（ThreadLocal）

Looper的prepare实际上是初始化（**关于ThreadLocal的原理可以看Android的ThreadLocal文档**）

```
//Looper类中↓，把
sThreadLocal.set(new Looper(quitAllowed));
```

```
//ThreadLocal类中↓
// 设置当前线程中ThreadLocal变量的值
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);  //获取当前线程的map
    if (map != null)
        map.set(this, value);
    else
        // ThreadLocalMap懒初始化，直到设置值的时候才创建
        createMap(t, value);
}
```

- prepare（插入）：这个语句就通过ThreadLocal这个类，获取当前线程对应的map，然后把这个Looper放入这个map中（注意，插入map的key是“this”也就是ThreadLocal<Looper>）。

- 接着Looper.mylooper的时候，就去这个ThreadLocal里面找到这个map，在通过这个map来获取当前线程的Looper



## ThreadLocalMap

ThreadLocalMap的问题可以见郭婶公众号的这篇文章[想让不同线程间的变量互不影响，你会怎么做](https://mp.weixin.qq.com/s/yxp3K2YiShxXN4zyVeB9jA)

讲得是ThreadLocal的源码。

### 内存问题

在ThreadLocalMap的实现中，**ThreadLocalMap是存储着ThreadLocal的弱引用（key值）**，所以ThreadLocalMap不会影响ThreadLocal的回收

### 实现

ThreadLocalMap使用了**斐波拉契哈希**；存储方式是数组，超出装载因子数组翻倍；解决冲突的方式是开放地址法；冲突探测的方式是**线性探测**；麻烦的地方在于垃圾清理。get方法在线性探测的过程中会做一点儿清理工作（释放**Stale Slot，Full Slot**左移），清理后内存得到释放，提高未来hash命中率；**set方法主要思想是用新的Full Entry替换一段Run中的Stale Entry，并清理Run及Run后面的一段儿（对数扫描log2(n)，平衡垃圾回收和清理时间）。**

ThreadLocald的内存模型

![img](https://mmbiz.qpic.cn/mmbiz_png/v1LbPPWiaSt69WEDCGHjqywGRS5CH0ADA1pgpVIDhDyicWX24BGibaiaBFic23gIKUdjIhUQ2hW6BVnCTzTNa1XaFSQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

