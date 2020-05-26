Android的事件分发1在 **书籍笔记/Android开发艺术探索** 的三四章节中。

在理解了事件分发的基本流程之后，事件分发的知识点还很多，这里略做总结（本文最好结合源码阅读，因为源码较长，本文都只是节选）

## cancel事件

### 1. cancel的产生

我们知道事件分发在ViewGroup之后会不断传递给子ViewGroup或者子View，主要的方法dispatchTouchEvent。

而cancel事件的分发主要是在其中的 **dispatchTransformedTouchEvent()方法中，**

先看这个方法的部分代码：

```java
//传入cancel标志，因为一开始全都不是CANCEL，所以oldAction肯定不是CANCEL。所以关键就在cancel标志上
private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
                                              View child, int desiredPointerIdBits) {
    ...
    if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
        event.setAction(MotionEvent.ACTION_CANCEL);
        if (child == null) {
            handled = super.dispatchTouchEvent(event);
        } else {
            handled = child.dispatchTouchEvent(event);
        }
        event.setAction(oldAction);
        return handled;
    }
    ...
}
```

而在dispatch方法中，调用到这个函数的有这几个地方

1. ViewGroup2698行：

   ```java
   if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) //直接传入false
   ```

2. 2739行：

   ```java
   if (mFirstTouchTarget == null) {
       // No touch targets so treat this as an ordinary view.
       handled = dispatchTransformedTouchEvent(ev, canceled, null,
                                               TouchTarget.ALL_POINTER_IDS);
   }
   ```

   传入的是canceled，这个标志的计算如下，在初始化情况下也不是

   ```java
   final boolean canceled = resetCancelNextUpFlag(this)
       || actionMasked == MotionEvent.ACTION_CANCEL;
   ```

3. 2753行：

   ```java
   // 通过cancelChild标志判断，这里可以看到cancelChild是通过intercepted标志判断的
   final boolean cancelChild = resetCancelNextUpFlag(target.child)
       || intercepted;
   if (dispatchTransformedTouchEvent(ev, cancelChild,
                                     target.child, target.pointerIdBits)) {
       handled = true;
   }
   ```

   而intercepted标志位我们就比较熟悉了，就是判断是否拦截这个action（如果自己拦截或者子View拦截就为true）

## 教程1知识点

[中文版](https://juejin.im/entry/57748b5d6be3ff006ae66e3c)

[英文版](https://balpha.de/2013/07/android-development-what-i-wish-i-had-known-earlier/)

- dispatch通过intercept标志来执行逻辑，可以查看源码，逻辑如下表

  | Action类型 | 子容器是否拦截（target==null？） | intercept结果                 |
  | ---------- | -------------------------------- | ----------------------------- |
  | down       | 都行                             | 通过onInterceptTouchEvent判断 |
  | 其他       | 是                               | 通过onInterceptTouchEvent判断 |
  | 其他       | 否                               | true                          |

  所以**对于move和up和cancel，如果子容器说不需要这个event那么就由父容器拦截了；如果子容器要，那么也要父容器再判断一下（如果判断要拦截，那么就会产生一个cancel给子容器）**



## 教程2知识点

[探究Android事件分发](https://cloud.tencent.com/developer/article/1460225)

1. mFirstTarget问题

   在dispatch的过程中，有一个函数dispatchTransformedTouchEvent，就是那个会产生cancel的函数，这个函数会判断子容器是否拦截，如果拦截，那么把当前的元素加入target链表。

   ```java
   newTouchTarget = addTouchTarget(child, idBitsToAssign);
   ```

   最终会形成一个事件消费链表

2. target和拦截的问题

   **形成的target链表是有作用的**，在之后的判断中，不需要再遍历子元素并判断了，只需要直接传给对应的下级元素。 **但是需要主要的是，还是得通过target链表一个个判断，只是省去了遍历子元素而已**