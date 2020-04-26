https://www.infoq.cn/article/3WyReTKqrHIvtw4frmr3

## JVM 运行时内存布局

从概念上大致分为 6 个（逻辑）区域

![ä¸æçæJVMåå­å¸å±åGCåç](https://static001.infoq.cn/resource/image/dd/a9/dd614bf56417939aa0e0694fedf2caa9.png)

一类是每个线程所独享的：PC Register，JVM Stack，Native Method Stack

另一类是所有线程共享的：**Heap**，Method Area，Runtime Constant Pool

## GC 垃圾回收原理

1. 引用计数法
2. 可达性分析

#### 常用的 GC 算法

1. **mark-sweep 标记清除法**
2. **mark-copy 标记复制法**

思路也很简单，将内存对半分，总是保留一块空着（上图中的右侧），将左侧存活的对象（浅灰色区域）复制到右侧，然后左侧全部清空。避免了内存碎片问题，但是内存浪费很严重，相当于只能使用 50% 的内存。

3. **mark-compact 标记 - 整理（也称标记 - 压缩）法**

免了上述两种算法的缺点，将垃圾对象清理掉后，同时将剩下的存活对象进行整理挪动（类似于 windows 的磁盘碎片整理），保证它们占用的空间连续，这样就避免了内存碎片问题，但是整理过程也会降低 GC 的效率

4. **generation-collect 分代收集算法**

将内存分成了三大块：年青代（Young Genaration），老年代（Old Generation）, 永久代（Permanent Generation），其中 Young Genaration 更是又细为分 eden，S0，S1 三个区。

## GC原理

[简述JAVA GC回收机制，深入理解GC原理](https://blog.csdn.net/JavaReact/article/details/83656329)(基本原理)

[关于GC原理和性能调优实践，看这一篇就够了！](https://juejin.im/post/5d8c5a5de51d4578323d51bd)（详细解释）

**！堆内存就是GC管理的主要区域（知识点，重点）**

JVM把堆内存分成三块区域：新生代，老年代和持久代

![img](https://img-blog.csdnimg.cn/20181102153654181.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0phdmFSZWFjdA==,size_16,color_FFFFFF,t_70)

1. 新生代分为三块（Eden，From ，To）

   **刚刚创建的对象会放在Eden区**，Eden区是连续的内存空间，因此在其上分配内存极快。（每次新生代的垃圾回收（又称**Minor GC**，加上标记）采用copy算法）。E**den区满了之后，会清理消亡的对象，然后放入From区。**

   当**Eden区再次满了，就和From一起清理然后放入To区**，并置换To区为From区，From为To区。

   **当To区的填满，将里面的对象的移动到老年代**，如果没有填满，但是里面的对象标记次数大于15次也会移动到老年代。

2. 老年代的垃圾回收（**Major GC**）采用“标记-清理”或“标记-整理”算法。整理包括新生代和老年代的GC称为Full GC

3. 永久代。在JDK8之前的HotSpot实现中，类的元数据如方法数据、方法信息（字节码，栈和变量大小）、运行时常量池等直接被放在永久代。GC在主程序运行期间不会对永久代区域进行清理，就会导致永久代爆满抛出OOM异常。

   JDK8把类的元数据直接保存在本地内存空间（堆外空间），称为元空间。类的元数据放在native momery，字符串池和类的静态变量放在java堆里面。（[元数据](https://www.oracle.com/technetwork/articles/hunter-meta-097643-zhs.html)是指用来描述数据的数据）

- GC策略：Minor GC、Old GC（Major GC）、Full GC、Mixed GC（整理新生代和部分老年代）

- 内存分配策略

  - 存在一个阈值，对象内存大于这个阈值的直接在老年代分配，防止大内存复制。

  - 空间分配担保：首先判断新生代够不够，不够则通过平均担保（大于平均值则Young GC，然后判断够不够，不够则担保失败并Full GC，够的话则担保成功）

    ![img](https://user-gold-cdn.xitu.io/2019/9/26/16d6b1954b158636?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

  - 动态年龄判定。如果 Young GC 之后，新生代存活对象达到相同年龄所有对象大小的总和大于任意  Survivor 空间（S0+S1空间）的一半，此时 S0 或者 S1 区即将容纳不了存活的新生代对象。

年龄大于或等于该年龄的对象就可以直接进入老年代，无须等到 MaxTenuringThreshold 中要求的年龄。

## CMS原理和G1原理（未继续深入了解，看上面的详细解释网站）

- CMS可达性分析

- G1（Garbage-First）是一款面向服务器的垃圾收集器，支持新生代和老年代空间的垃圾收集，主要针对配备多核处理器及大容量内存的机器。

## 可以作为GC Root的对象

重点的几个

1. 由系统ClassLoader加载的对象
2. Thread -- 活着的线程
3. 栈帧中引用的对象
4. native方法的局部变量和全局变量，调用栈指向的对象
5. 不可到达的对象
6. 方法区中静态引用的对象

全面：

```
1.System Class  系统class
----------Class loaded by bootstrap/system class loader. For example, everything from the rt.jar like java.util.* .
2.JNI Local  本地方法局部变量
----------Local variable in native code, such as user defined JNI code or JVM internal code.
3.JNI Global   本地方法全局变量
----------Global variable in native code, such as user defined JNI code or JVM internal code.
4.Thread Block  线程
----------Object referred to from a currently active thread block.
Thread
----------A started, but not stopped, thread.
5.Busy Monitor  管程
----------Everything that has called wait() or notify() or that is synchronized. For example, by calling synchronized(Object) or by entering a synchronized method. Static method means class, non-static method means object.
6.Java Local  本地Java局部变量
----------Local variable. For example, input parameters or locally created objects of methods that are still in the stack of a thread.
7.Native Stack  本地方法调用栈
----------In or out parameters in native code, such as user defined JNI code or JVM internal code. This is often the case as many methods have native parts and the objects handled as method parameters become GC roots. For example, parameters used for file/network I/O methods or reflection.
7.Finalizable  
----------An object which is in a queue awaiting its finalizer to be run.
8.Unfinalized
----------An object which has a finalize method, but has not been finalized and is not yet on the finalizer queue.
9.Unreachable   不可到达的对象
----------An object which is unreachable from any other root, but has been marked as a root by MAT to retain objects which otherwise would not be included in the analysis.
10.Java Stack Frame   栈帧
----------A Java stack frame, holding local variables. Only generated when the dump is parsed with the preference set to treat Java stack frames as objects.
11.Unknown
----------An object of unknown root type. Some dumps, such as IBM Portable Heap Dump files, do not have root information. For these dumps the MAT parser marks objects which are have no inbound references or are unreachable from any other root as roots of this type. This ensures that MAT retains all the objects in the dump.
```



