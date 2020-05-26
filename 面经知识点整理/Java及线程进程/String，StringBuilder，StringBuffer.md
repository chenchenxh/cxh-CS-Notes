## String, StringBuffer,StringBuilder的区别

String不可变，StringBuffer线程安全（使用synchronize），StringBuilder线程不安全

**String为什么不可变**

虽然String、StringBuffer和StringBuilder都是final类，它们生成的对象都是不可变的，而且它们内部也都是靠char数组实现的，但是不同之处在于，String类中定义的char数组是final的，而StringBuffer和StringBuilder都是继承自AbstractStringBuilder类，它们的内部实现都是靠这个父类完成的，而这个父类中定义的char数组只是一个普通是私有变量，可以用append追加。因为AbstractStringBuilder实现了Appendable接口。

**不可变的优势**

- string被频繁用于哈希码，不用重复计算哈希码，性能好
- 安全性
- String常量池的特点

https://juejin.im/post/5a5d5c66f265da3e261bf46c



## String的编码模式

[Java中String类型与默认字符编码](https://blog.csdn.net/Sugar_Rainbow/article/details/76945323)

- **本地JVM的编码方式是和本机OS默认的字符编码方式相关的，但是JVM的编码方式可以被修改**

- Java程序的默认字符集是Unicode，在程序中声明的String类型的编码方式是和JVM编码方式相关的

- **String.getBytes()方法默认的编码方式是JVM编码方式**；同时还可以接收一个字符集名称当作参数，优先使用参数的字符集

- 因为Java代码使用的Unicode字符集，允许各编码方式之间转换，但**不保证bit损失**，所以String类型可以得到不同编码方式的byte数组，只要按照编码解码的方式获取字符串类型显示即可

  