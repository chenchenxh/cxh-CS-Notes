## String, StringBuffer,StringBuilder的区别

String不可变，StringBuffer线程安全（使用synchronize），StringBuilder线程不安全

**String为什么不可变**

虽然String、StringBuffer和StringBuilder都是final类，它们生成的对象都是不可变的，而且它们内部也都是靠char数组实现的，但是不同之处在于，String类中定义的char数组是final的，而StringBuffer和StringBuilder都是继承自AbstractStringBuilder类，它们的内部实现都是靠这个父类完成的，而这个父类中定义的char数组只是一个普通是私有变量，可以用append追加。因为AbstractStringBuilder实现了Appendable接口。

**不可变的优势**

- string被频繁用于哈希码，不用重复计算哈希码，性能好
- 安全性
- String常量池的特点

https://juejin.im/post/5a5d5c66f265da3e261bf46c