## SurfaceView

https://www.jianshu.com/p/b037249e6d31

https://juejin.im/entry/58b4ccd944d904006a1ce446

### 和View的对比

- View适用于主动更新，SurfaceView适用于被动更新
- View使用主线程进行更新，SurfaceView开启子线程进行刷新
- SurfaceView在底层机制实现了双缓冲机制



- 与普通View不同的是，它有自己的Surface。有自己的Surface，在WMS中有对应的WindowState，在SurfaceFlinger中有Layer。
- 虽然在App端它仍在View hierachy中，但在Server端（WMS和SF）中，它与宿主窗口是分离的。这样的好处是对这个Surface的渲染可以放到单独线程去做，渲染时可以有自己的GL context。

![img](https://user-gold-cdn.xitu.io/2017/2/28/89f5ef4c561467bdd35ceffdb02df114?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 双缓冲机制

> 双缓冲技术是游戏开发中的一个重要的技术。当一个动画争先显示时，程序又在改变它，前面还没有显示完，程序又请求重新绘制，这样屏幕就会不停地闪烁。而双缓冲技术是把要处理的图片在内存中处理好之后，再将其显示在屏幕上。双缓冲主要是为了解决 反复局部刷屏带来的闪烁。把要画的东西先画到一个内存区域里，然后整体的一次性画出来。



### 为什么使用SurfaceView

优点

- 一是，如果屏幕刷新频繁，onDraw方法会被频繁的调用，onDraw方法执行的时间过长，会导致掉帧，出现页面卡顿。而SurfaceView采用了**双缓冲技术**，提高了绘制的速度，可以缓解这一现象。
- 二是，view的onDraw方法是运行在主线程中的，会轻微阻塞主线程，对于需要频繁刷新页面的场景，而且onDraw方法中执行的操作比较耗时，会导致主线程阻塞，用户事件的响应受到影响，也就是响应速度下降，影响了用户的体验。而**SurfaceView可以在子线程中更新UI，不会阻塞主线程，提高了响应速度。**

缺点

- 因为这个Surface不在View hierachy中，它的显示也不受View的属性控制，所以不能进行平移，缩放等变换，也不能放在其它ViewGroup中，一些View中的特性也无法使用。
- 还是上一条，因此SurfaceView不能嵌套使用



## TextureView

- 继承自View，可以将内容流直接投影到View上。
- 不会再WMS中单独创建窗口，而是作为一个普通的View来操作
- 必须在有硬件加速的窗口中使用，占用内存比SurfaceView高



## 对比

![img](https://user-gold-cdn.xitu.io/2017/2/28/2b674ce3e59588fabffb47ed2ebbbf58?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

从性能和安全性角度，啥用播放器优先选择SurfaceView



## SurfaceView是为什么可以直接子线程绘制呢?

https://juejin.im/post/5da14e8ae51d45782b0c1c20