[TOC]

## MeasureSpec

每个View对应的 **测量规格**，是一个int值，每个View有两个MeasureSpec（宽和高）

- 高30位为测量值，低2位为mode
- 低2位：3--MODE_MASK，0-2分别对应三种模式（UNSPECIFIED，EXACTLY，AT_MOST）
- MeasureSpec类中有一个getMode和getSize方法，就是跟MODE_MASK进行与操作

模式：

- UNSPECIFIED

  父容器对View没有限制

- EXACTLY

  父容器已经检测出所需要的精确大小，对应Match_Parent

- AT_MOST

  父容器指定一个可用大小，View不能大于这个值，对应Wrap_Content

MeasureSpec的测量方法。子View的MeasureSpec由父类的spec和padding决定，dimension决定的是此时测量的是宽还是高

```java
//MeasureSpec中的获取方法
public static int getChildMeasureSpec(int spec, int padding, int childDimension){
    ...
}
```

![9](D:\typora\pic\9.png)

## View的工作流程

### Measure

#### 1. View的measure

- SpecSize是View**测量后的大小**，View的最终大小是在Layout阶段决定的，但是几乎所有情况下这两个大小是一样的（作者没提到什么情况是不一样大小的）

- onMeasure中，调用的是getDefaultSize方法来测量：

  ```java
  public static int getDefaultSize(int size, int measureSize){
      //...
      switch(specMode){
          case MeasureSpec.UNSPECIFIED:
              return size;
          case MeasureSpec.AT_MOST:
          case MeasureSpec.EXACTLY:
              return specSize;
      }
  }
  ```

  对于UNSPECIFIED情况，一般是系统内部的测量过程。调用的是getSuggestedMinimum方法，这个方法返回Drawble的原始宽度（前提是这个Drawble有原始宽度，如ShapeDrawble就没有，BitmapDrawble图片就有）

- 因为常用的View的模式都不是UNSPECIFIED，所以我们的VIew的宽高是由**SpecSize决定**。

  直接继承View的自定义控件要重写onMeasure方法并设置wrap_content时的对应大小，否则会被当做使用Match_Parent来匹配大小。可以看上面代码，默认的Size两个模式都是specSize，**wrap_content是AT_MOST，而Match_Parent是EXACTLY，但是两者的specSize数值上都是可以有最大值。所以要实现wrap_content都需要重写，textView等都是重写onMeasure方法**
  
  ？为什么要重写onMeasure呢？因为measure和getDefaultSize都是public的，onMeasure是protected的，可以起到一定的防止调用和保护作用。



#### 2. ViewGroup的measure

ViewGroup中包含了很多个View，那么ViewGroup就要去测量各个View的MeasureSpec：调用MeasureChildren方法去遍历child，然后 **MeasureSpec的结果存在child中**。测量方法如下：

```java
    protected void measureChild(View child, int parentWidthMeasureSpec,
            int parentHeightMeasureSpec) {
        final LayoutParams lp = child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```

两个方法：getChildMeasureSpec和measure方法。

**调用过程：**

1. **ViewGroup的MeasureChildren**
2. **ViewGroup的MeasureChild**
3. **getChildMeasureSpec（其实就是根据parent的Spec和child的layoutParms来计算child的Spec）**
4. **child.measure（这个方法就进入到View的测量中了，就是上一小节的内容）**
   1. **再一通判断之后，调用onMeasure**
   2. **onMeasure调用getDefaultSize（根据mode和计算出size）**

注意ViewGroup是一个抽象类，上述的过程只是对各个child的测量，并没有ViewGroup的测量。所以需要各个VIewGroup的子类（如LinearLayout和RelativeLayout）**各自去实现自己的onMeasure方法**。

#### 3. LinearLayout的测量

LinearLayout重写onMeasure，根据LinearLayout是水平的还是竖直的来 **累加child的measrue**，然后加上自己的padding和margin来获得自己最终的totalLength

需要注意的是，可以通过getMeasuredWidth/Height来获得measure后的长度，但是**这个值不一定是正确的，因为有些View需要多次Measure，更好地方法是在Layout中去获取测量宽/高**

#### 4. 获取View的Measure结果

通过上面的分析我们知道可以通过get来获得View的Measure结果，但是 **View的测量过程和活动的生命周期是不绑定的**，所以没办法确定在哪个阶段去调用才会得到正确的结果。解决方法：

1. View.onWindowFocusChanged

   这个方法表示view已经初始化完毕。需要注意的是活动暂停和继续执行也就是onPause和onResume都会执行这个方法。

2. view.post(Runable)

   这个方法是用来把Runable对象放置到View的HandlerActionQueue中，而这个队列中已经存在着Measure的一系列操作，那么把这个Runable放置在队列的末尾，就可以正确的获取measure

   ```java
   view.post(new Runable(){
       @Override
       public void run(){
           int width = view.getMeasureWidth();
           int height = view.getMeasureHeight();
       }
   });
   ```

3. ViewTreeObserver

   这是View树的观察者，可以通过这个对象的很多方法来获取。比如发现要求measure的View已经添加到Tree里面等。

4. View.measure()

   手动测量，一般不建议。



### Layout

View和ViewGroup的Layout方法的首先会通过 **setFrame方法**求出mLeft、mRight、mTop、mBottom四个值；接着会执行onLayout方法，这个方法跟onMeasure一样，都是子类自己去实现的方法（抽象类Layout）

例如LinearLayout的竖直Layout方法：不断计算子View的位置 **setChildFrame(child, childLeft childTop+getLacationOffset, childWidth, childHeight)方法**，其中childTop会随着child的增加而不断增加

这里回答了 **测量宽/高和最终宽/高的区别**：

- 这个问题也可以是View的getMeasureWidth和getWidth方法有什么区别

- 直接看实现

  ```java
  public final int getWidth() {
      return mRight - mLeft;
  }
  public final int getMeasuredWidth() {
      return mMeasuredWidth & MEASURED_SIZE_MASK;
  }
  ```

- 在默认的View实现中，两者都是相等的。不过两者的生成时机不同，一个在Measure阶段，一个在Layout阶段

- 如果重写Layout方法，那么可能会导致两者的大小不同



### Draw

1. 绘制背景background.draw(canvas)
2. 绘制自己（onDraw）
3. 绘制children（dispatchDraw）
4. 绘制装饰（onDrawScrollBars）





## 自定义View

作者将自定义View分为四类：

1. 继承View重写onDraw方法
2. 继承ViewGroup派生特殊的Layout
3. 继承特定的View（如TextView）
4. 继承特定的ViewGroup（如LinearLayout）

自定义View的注意点：

1. 支持wrap_content

   如果是直接继承View或者ViewGroup的话，重写onMeasure

2. 如果有必要，支持padding和margin

   需要在测量和Layout将padding加入考虑，不然padding就会无效。如果是ViewGroup，需要考虑内部View的margin

3. 尽量不要在View中使用Handler

4. View中如果有线程或者动画，记得及时停止

   可以使用方法onDetachedFromWindow

5. 如果带有滑动嵌套的，需要处理好

