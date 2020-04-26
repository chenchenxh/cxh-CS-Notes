## 能不能new Activity

[android中能不能new Activity()对象引发的思考](https://blog.csdn.net/smileiam/article/details/88686637)

**可以new 一个Activity**

### Android系统的Activity怎么创建的？

> Android系统为了便于管理Activity，回调其生命周期，把Activity实例化相关操作都封装好了，我们只要轻松调用startActivity操作，就能实现Activity中各种生命周期的回调了。

Android系统通过类加载器的newInstance方法创建（也就是**反射**的方式，为的是解耦）

### 自己创建的活动有没有生命周期

但是自己创建的活动是不能直接调用Activity的生命周期方法的，**因为自己创建的活动是没有窗口的**，调用super.onCreate()方法是会报错的。如果使用不跟View相关的方法就能正常运行，跟一个普通类一样。

### 启动一个活动为什么也要跨进程通信

首先应该知道，Android为了统一管理活动和生命周期，所有启动的活动都要在AMS中注册。而这个自己创建的活动也一样，但是因为这个活动和其他活动是不同的活动栈中，在不同的进程中。

### 启动一个新的活动要出发几次跨进程通信

1. ActivityManagerProxy.startActivity()->ActivityManagerService.startActivity() 应用进程告诉AMS我要启动新的Acitvity组件；
2. ApplicationThreadProxy.schedulePauseActivity()->ApplicationThread.schedulePauseActivity() AMS通知应用进程将前一个Activity组件pause掉；
3.  ActivityManagerProxy.activityPaused() -->ActivityManagerService.activityPaused() 应用进程通知AMS上一个Activity已经paused掉了
4. ApplicationThreadProxy.scheduleLaunchActivity()-->ApplicationThread.scheduleLaunchActivity() AMS通知应用进程通信启动新的Activity组件

> 这里要注意如果启动的其他进程的组件，还要多一次跨进程通信的，AMS通知Zgote进程fork出一个子进程出来。