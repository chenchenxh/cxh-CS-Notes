[Android多进程使用及其带来的问题](https://blog.csdn.net/u014088294/article/details/52266601)

## 使用多进程

默认情况下，一个Android程序的所有组件都是在一个进程中执行的，该进程的名字就是程序的包名。同时，Android也允许开发者将程序的一些组件在其他进程中执行。四大组件均支持android:process属性，我们可以设置该属性的名字，将组件运行在指定的进程。参考以下的例子，

        <activity
            android:name=".SecActivity"
            android:label="@string/title_activity_sec"
            android:process=":secActivity" >
        </activity>
如果一个组件指定的进程名是以“：”开头的，就意味着该进程是一个**私有进程**。那么何为私有进程？私有进程是和全局进程对应，如果我们在android:process中填写的是完整的包名，那么该进程就是一个全局进程。全局进程允许两个不同的应用（或组件）运行在同一个进程中（使用ShareUID，且签名完全一致）；私有进程则不可以。


Application也可以设置android:process属性，来修改所有组件运行进程的名称。

## 为什么使用多进程

1. 更大的内存分配

   对运行时内存要求较大的可以使用多进程，如果应用经常OOM的话也可以考虑。

2. 防止进程被杀死

   可以使用双进程，比如播放音乐和UI分为两个进程，UI进程就可以在后台杀死而不会影响音乐。

   也可以让进程互相监督唤醒。

3. 分担进程的压力

   分担主进程的运行压力。

## 多进程的缺点

**因为一个进程意味着一个JVM示例**，所以不同的进程那么就会影响数据的共享。

还可以使用的通信方式：Intent，Handler，AIDL，Binder以及其他常用进程通信方法。

多进程带来的问题：

- 静态变量和单例模式失效（不是同一个内存）
- 线程同步方案失效（类锁和对象锁不是同一个，不能保证同步）
- SharedPreferences 可靠性下降(SharedPreferences不支持多个进程同时写，会有一定的几率丢失数据)
- Application 多次创建(Android为每个进程分配独立的虚拟机，这个过程其实就是启动一个应用，所以Application会被创建多次)，所以我们不能直接将一些数据保存在Application中。