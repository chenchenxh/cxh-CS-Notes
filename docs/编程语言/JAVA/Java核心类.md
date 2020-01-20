# JAVA核心类

## StringBuilder

对String类的优化。原本将String相加会不断创建新的String对象，然后抛弃旧的对象。但是这种现象很**浪费内存，而且影响效率**。所以对于变化较多的String可以创建为StringBuilder类。

原来还有一个同步版本的StringBuffer类（不常用），而且StringBuilder一般在JAVA编译器编译的时候就会对连续+的String进行优化为StringBuilder类。

## StringJoiner

用来快速处理String数组的拼合问题，如String[] names = {"Bob", "Alice", "Grace"};

可以用StringJoiner来将string数组拼合

```java
//如下，用，隔开，左右各是{}
var sj = new StringJoiner(", ", "{", "}");
```

## JavaBean

其实就是Set和Get方法这些对属性的操作，有些IDE可以直接对属性进行Get和Set的Generate，方便编程



