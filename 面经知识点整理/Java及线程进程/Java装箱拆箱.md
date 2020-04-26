https://www.cnblogs.com/dolphin0520/p/3780005.html

## 原理

在前面的文章中提到，Java为每种基本数据类型都提供了对应的包装器类型，至于为什么会为每种基本数据类型提供包装器类型在此不进行阐述，有兴趣的朋友可以查阅相关资料。在Java SE5之前，如果要生成一个数值为10的Integer对象，必须这样进行：

```
Integer i = ``new` `Integer(``10``);
```

　　而在从Java SE5开始就提供了自动装箱的特性，如果要生成一个数值为10的Integer对象，只需要这样就可以了：

```
Integer i = ``10``;
```

　　这个过程中会自动根据数值创建对应的 Integer对象，这就是装箱。

　　那什么是拆箱呢？顾名思义，跟装箱对应，就是自动将包装器类型转换为基本数据类型：

```
Integer i = ``10``; ``//装箱``int` `n = i;  ``//拆箱
```

　　简单一点说，装箱就是 自动将基本数据类型转换为包装器类型；拆箱就是 自动将包装器类型转换为基本数据类型。

　下表是基本数据类型对应的包装器类型：

| int（4字节）    | Integer   |
| --------------- | --------- |
| byte（1字节）   | Byte      |
| short（2字节）  | Short     |
| long（8字节）   | Long      |
| float（4字节）  | Float     |
| double（8字节） | Double    |
| char（2字节）   | Character |
| boolean（未定） | Boolean   |

## 实现

装箱过程是通过调用包装器的valueOf方法实现的，而拆箱过程是通过调用包装器的 xxxValue方法实现的。（xxx代表对应的基本数据类型）

## 习题

下面程序的输出结果是什么？

```java
public class Main {
    public static void main(String[] args) {
         
        Integer a = 1;
        Integer b = 2;
        Integer c = 3;
        Integer d = 3;
        Integer e = 321;
        Integer f = 321;
        Long g = 3L;
        Long h = 2L;
         
        System.out.println(c==d);
        System.out.println(e==f);
        System.out.println(c==(a+b));
        System.out.println(c.equals(a+b));
        System.out.println(g==(a+b));
        System.out.println(g.equals(a+b));
        System.out.println(g.equals(a+h));
    }
}
```

　　先别看输出结果，读者自己想一下这段代码的输出结果是什么。这里面需要注意的是：当 "=="运算符的两个操作数都是 包装器类型的引用，则是比较指向的是否是同一个对象，而如果其中有一个操作数是表达式（即包含算术运算）则比较的是数值（即会触发自动拆箱的过程）。另外，对于包装器类型，equals方法并不会进行类型转换。明白了这2点之后，上面的输出结果便一目了然：

```
true
false
true
true
true
false
true
```

　　第一个和第二个输出结果没有什么疑问。第三句由于 a+b包含了算术运算，因此会触发自动拆箱过程（会调用intValue方法），因此它们比较的是数值是否相等。而对于c.equals(a+b)会先触发自动拆箱过程，再触发自动装箱过程，也就是说a+b，会先各自调用intValue方法，得到了加法运算后的数值之后，便调用Integer.valueOf方法，再进行equals比较。同理对于后面的也是这样，不过要注意倒数第二个和最后一个输出的结果（如果数值是int类型的，装箱过程调用的是Integer.valueOf；如果是long类型的，装箱调用的Long.valueOf方法）。

如果对上面的具体执行过程有疑问，可以尝试获取反编译的字节码内容进行查看。