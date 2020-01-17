# Java反射

反射就是Reflection，Java的反射是指程序在运行期可以拿到一个对象的所有信息。

这种一般用在运行时完全不知道某个类的内部情况，甚至不知道这个类的名称（也就是没有import），通过反射直接调用其方法。

## Class类

除了int等基本类型，java的其他类型都是class。例如每次JVM加载String的时候，都是读取String.class文件到内存，然后创建一个String的Class示例，而且这个构造方法是privete，也就是只能JVM来构造Class类。

```java
Class cls = new Class(String);
```

因此，回到反射上。有了这个Class类，就可以通过创建的示例-->该示例的class-->class的方法，接口等。这样就能完成反射。方法如下：

```java
//方法一
Class cls = String.class
//方法二
String s = "Hello";
Class cls = s.getClass();
//方法三
Class cls = Class.forName("java.lang.String");
```

*动态加载，JVM在执行java程序的时候，不是一次性把所有的class加载的，而是每次使用的时候才加载。*

## 访问方法

```java
Field getField(name)	//根据字段名获取某个public的field（包括父类）
Field getDeclaredField(name)	//根据字段名获取当前类的某个field（不包括父类）
Field[] getFields()	//获取所有public的field（包括父类）
Field[] getDeclaredFields()	//获取当前类的所有field（不包括父类）
```

**其实在访问到field之后，可以通过field,get(object)的方式来拿到privete的字段的值（需要加上f.setAccessible(true);），但是这样就破坏了封装。当然可以设置SecurityManager规则，就可以阻止f.setAccessible(true);**

## 动态代理

因为接口无法实例化，但是java标准库提供一种动态代理（Dynamic Proxy）的机制：可以在运行期动态创建某个`interface`的实例。

例如，我们仍然先定义了接口`Hello`，但是我们并不去编写实现类，而是直接通过JDK提供的一个`Proxy.newProxyInstance()`创建了一个`Hello`接口对象。这种没有实现类但是在运行期动态创建了一个接口对象的方式，我们称为动态代码。JDK提供的动态创建接口对象的方式，就叫动态代理。