## 自定义View的实现方法

郭霖大神：https://blog.csdn.net/guolin_blog/article/details/17357967

## 自定义圆角的原理

https://juejin.im/post/5b305f73f265da59b653b08d

1. 继承自AppcompatImage
2. 设置Paint的PorterDuff属性
3. 设置paint的属性（圆角等）
4. 重写draw，修改里面canvas的drawPath(path, paint)
5. canvas.restore()

示意代码如下:

```java
@Override
    protected void onDraw(Canvas canvas) {
        // 使用离屏缓存，新建一个srcRectF区域大小的图层
        canvas.saveLayer(srcRectF, null, Canvas.ALL_SAVE_FLAG);
        // ImageView自身的绘制流程，即绘制图片
        super.onDraw(canvas);
        // 给path添加一个圆角矩形或者圆形
        if (isCircle) {
            path.addCircle(width / 2.0f, height / 2.0f, radius, Path.Direction.CCW);
        } else {
            path.addRoundRect(srcRectF, srcRadii, Path.Direction.CCW);
        }
        paint.setAntiAlias(true);
        // 画笔为填充模式
        paint.setStyle(Paint.Style.FILL);
        // 设置混合模式
        paint.setXfermode(xfermode);
        // 绘制path
        canvas.drawPath(path, paint);
        // 清除Xfermode
        paint.setXfermode(null);
        // 恢复画布状态
        canvas.restore();
    }
```

## PorterDuff

里面的各种模式就是用来定义下面四种属性的组合。

```
Sa：全称为Source alpha，表示源图的Alpha通道；
Sc：全称为Source color，表示源图的颜色；
Da：全称为Destination alpha，表示目标图的Alpha通道；
Dc：全称为Destination color，表示目标图的颜色.
```

## 自定义View

[郭神的自定义View博客](https://blog.csdn.net/guolin_blog/article/details/17357967)

自定义VIew分为三类：

1. 直接自己重写
2. 组合控件（比如Title栏）
3. 继承控件（比如继承ImageView或者ListVIew等，重写一些方法即可）