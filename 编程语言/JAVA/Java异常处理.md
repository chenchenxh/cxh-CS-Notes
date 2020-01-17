# Java异常处理

## java的异常

下图来自菜鸟教程

![image-20200116104325762](https://www.runoob.com/wp-content/uploads/2013/12/12-130Q1234I6223.jpg)

抛出和捕获异常的方法和C++一样，都是try和catch

```java
try {
    String s = processFile(“C:\\test.txt”);
    // ok:
} catch (FileNotFoundException e) {
    // file not found:
} catch (SecurityException e) {
    // no read permission:
} catch (IOException e) {
    // io error:
} catch (Exception e) {
    // other error:
}
```

抛出异常的方法是throw，没什么好说的。

**需要注意的是异常屏蔽**，对于抛出多个异常或者被普通的异常和finally混杂的时候，如下:

```java
//会输出catched和finally，但是会throw finally的异常，一般情况下不使用finally的异常，因为我们并不关心finally的异常情况。
public class Main {
    public static void main(String[] args) {
        try {
            Integer.parseInt("abc");
        } catch (Exception e) {
            System.out.println("catched");
            throw new RuntimeException(e);
        } finally {
            System.out.println("finally");
            throw new IllegalArgumentException();
        }
    }
}
```



## 断言

断言（Assertion）是一种调试程序的方式。在Java中，使用`assert`关键字来实现断言。

```java
//assert就是期望后面的语句的判断结果是true，如果是false那么就会抛出AssertionError异常
public static void main(String[] args) {
    double x = Math.abs(-123.45);
    assert x >= 0;
    System.out.println(x);
}
```



## JDK Logging帮助调试

作用：帮助调试，免去麻烦的println语句调试

示例如下：

```java
//java的logging可以设置level，下面的fine就是属于级别较低的，可以被屏蔽，所以不会输出
import java.util.logging.Level;
import java.util.logging.Logger;
public class Hello {
    public static void main(String[] args) {
        Logger logger = Logger.getGlobal();
        logger.info("start process...");
        logger.warning("memory is running out...");
        logger.fine("ignored.");	//不会输出
        logger.severe("process will be terminated...");
    }
}
```

还有Commons Logging，Apache创建的日志模块，一般自动使用log4j系统（[参考资料](https://www.liaoxuefeng.com/wiki/1252599548343744/1264738932870688)）