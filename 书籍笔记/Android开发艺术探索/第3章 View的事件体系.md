# View的基础知识

- getX/getY和getRawX/getRawY

  前者计算的是相对于当前View的x和y，后者计算的是相对于手机屏幕左上角的x和y

- TouchSlop

  系统所能识别出的被认为是滑动的最小距离。设备相关，是一个常量，可以通过config.xml设置



1. VelocityTracker

   可以通过getXVelocity和getYVelocity获得

2. GestureDetector

   判断onTouchEvent(event)可以判断是否被消耗

   再通过onGestureListener，onDoubleTapListener

3. Scroller

   弹性滑动对象，用于实现View的弹性滑动



# View的滑动

三种方式：

1. **View本身提供的scrollTo/scrollBy方法**

   只能改变View的内容位置而不能概念View在布局中的位置

2. **通过动画施加平移效果**

   视图动画或者属性动画。操作影像，如果没有设置fillAfter，那么动画结束会回到原来位置。但是即便设置了，View的图像停留在新位置，但是View本身还是在旧位置。

3. **通过改变View的LayoutParams使得View重新布局实现滑动**

   比如通过设置padding和margin等方法，或者左右放个0dp的View，通过对左右的View填充空白即可。



# 弹性滑动

思路：将一次大的滑动分成若干个小的滑动

1. Scroller

   当我们设置了一个Scroller对象并start之后，内部其实什么都没做。 **而是会等invalidate()方法调用的时候才会移动**。invalidate方法会导致View重绘（View在重绘的draw步骤会调用computeScroll这个空方法，所以我们可以通过 **修改computeScroll方法来实现弹性滑动**。

2. 通过动画

   动画内部会自动实现时间内的每一帧动画的位置，但是移动的只是View的内容

3. 通过延时策略

   使用Handler或者View的postDelayed方法或者线程的Sleep方法，在延时之后去修改View的内容



# View的事件分发机制

## 原理

三个方法：dispatchTouchEvent、onInterceptTouchEvent、onTouchEvent

![11](D:\typora\pic\11.png)

> 当一个View需要处理事件，如果它设置了onTouchEvent，那么onTouch就会被调用......平时我们常用的onClickListener优先级最低，处于事件传递的尾端。

传递的顺序：Activity -> Window -> View

有下面结论：

1. 正常一个事件序列只被一个View处理。当然这个View可以交给其他View处理
2. 一个View一旦拦截，那么那么某个事件序列就会全部由它处理（如果能够传递的话），它的onInterceptTouchEvent也不会继续调用了
3. 某个View开始处理事件，如果它不处理ACTION_DOWN，那么同一事件序列的其他事件将一起交给父元素处理。
4. 如果View消耗了DOWN，但是不消耗其他MotionEvent，那么这些Event将会消失（交给Activity处理）
5. ViewGroup默认不拦截事件
6. View没有onInterceptTouchEvent，一旦有时间传递给他，那么它的onTouchEvent将会调用
7. View默认会消耗事件（返回true），除非是不可点击的。但是enable的属性不影响，也会返回true

## 源码

### 1. Activity开始分发

1. 先从Activity的分发开始。 **给window去分发，如果返回true，那么说明子View有对应的处理，如果返回false，那么就调用Activity自己的onTouchEvent来处理**

   ```java
   public boolean dispatchTouchEvent(MotionEvent ev) {
       if (ev.getAction() == MotionEvent.ACTION_DOWN) {
           onUserInteraction();  //默认空方法
       }
       if (getWindow().superDispatchTouchEvent(ev)) {
           return true;
       }
       return onTouchEvent(ev);  //这个方法其实也很简单，就是判断是否是window关闭的事件，是就finish自己并返回true，否则就是false
   }
   ```

2. 上面的window的superDispatchTouchEvent会继续调用decorView的方法，decorView会调用super的superDispatchTouchEvent。而decorView继承自FrameLayou继承自ViewGroup，所以就调用的ViewGroup的DispatchTouchEvent。

   **顺序：window -> decorView -> FramLayout -> ViewGroup** 

### 2. ViewGroup的分发

顶级View（也就是ViewGroup）对事件的分发

这个分发过程就是我们上面说的拦截和消耗过程。里面有个比较关键的代码。

- **如果被当前ViewGroup处理，那么这个target为null，那么在MOVE和UP方法来的时候，就不会去判断是否拦截了**
- **target是一个链表**

```java
final boolean intercepted;
if (actionMasked == MotionEvent.ACTION_DOWN
    || mFirstTouchTarget != null) {    //如果被子元素消耗，这个target会指向那个子元素
    final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
    if (!disallowIntercept) {  //注意这个标志
        intercepted = onInterceptTouchEvent(ev);   //拦截方法在这里
        ev.setAction(action);
    } else {
        intercepted = false;
    }
} else {
    intercepted = true;
}
```

1. 上面有个**标志disallowIntercept**，这个标志的初始化如下：

   mGroupFlags暂时不管，后面这个标志是子元素可以设置的一个标志位（在后面子元素影响父元素拦截，处理滑动冲突的时候可以用到），**这个标志位在每一次DOWN到来的时候就会重置，所以子元素的设置对DOWN没有作用**

   ```java
   final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
   ```

2. 如果ViewGroup不拦截，那么会逐个判断子元素是否进行拦截。 **判断的依据有两个：是否在播放动画和坐标是否在子元素内**

   这里有个关键点就是mFirstTouchTarget的作用还未理解

### 3. View的分发

View就比较简单了

1. 首先判断**onTouch**是否返回true
2. 如果为否，那么判断**onTouchEvent**
3. 如果是click，会在UP事件发生后触发 **performClick方法**



## View的滑动冲突

常见的滑动冲突场景：

![12](D:\typora\pic\12.png)

滑动冲突的处理规则： **根据左右滑动和上下滑动的不同来选择不同的View拦截，或者根据速度角度等判断**

1. **外部拦截法**

   **点击事件都先经过父容器的拦截处理，如果父容器需要此事件那么就拦截，如果不需要就不拦截**

   注意：

   - DOWN不能拦截，不然父容器就会直接处理后续的MOVE和UP了
   - UP也不能拦截，不然会导致子元素的Click失效，而且父容器本身拦截UP没有太大意义

2. **内部拦截法**

   **父容器不拦截任何时间，所有都交给子元素，如果子元素不消耗，那么交给父容器消耗**

   注意：

   - 这和Android的事件分发机制不一致，需要配合**requestDisallowInterceptTouchEvent方法**
   - 方法是：子元素通过**disallow父容器**来让自己一直获取事件，在MOVE内部通过判断父容器是否需要，**如果需要，那么取消disallow，再调用父容器的dispatch方法**，这样子就可以让父容器去处理这个事件了