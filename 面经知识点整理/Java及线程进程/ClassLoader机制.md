## 类加载过程

https://blog.csdn.net/javazejian/article/details/73413292

![img](https://img-blog.csdn.net/20170430160610299?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF2YXplamlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 加载：类加载过程的一个阶段：通过一个类的完全限定查找此类字节码文件，并利用字节码文件创建一个Class对象
- 验证：目的在于确保Class文件的字节流中包含信息符合当前虚拟机要求，不会危害虚拟机自身安全。主要包括四种验证，文件格式验证，元数据验证，字节码验证，符号引用验证。
- 准备：为类变量(即static修饰的字段变量)分配内存并且设置该类变量的初始值即0(如static int i=5;这里只将i初始化为0，至于5的值将在初始化时赋值)，这里不包含用final修饰的static，因为final在编译的时候就会分配了，注意这里不会为实例变量分配初始化，类变量会分配在方法区中，而实例变量是会随着对象一起分配到Java堆中。
- 解析：主要将常量池中的符号引用替换为直接引用的过程。符号引用就是一组符号来描述目标，可以是任何字面量，而直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。有类或接口的解析，字段解析，类方法解析，接口方法解析(这里涉及到字节码变量的引用，如需更详细了解，可参考《深入Java虚拟机》)。
- 初始化：类加载最后阶段，若该类具有超类，则对其进行初始化，执行静态初始化器和静态初始化成员变量(如前面只初始化了默认值的static变量将会在这个阶段赋值，成员变量也将被初始化)。

## classLoader

[深入分析Java ClassLoader原理](https://blog.csdn.net/xyang81/article/details/7292380)

### 默认的三个classloader

1. **BootStrap ClassLoader**：称为启动类加载器，是Java类加载层次中最顶层的类加载器，**负责加载JDK中的核心类库，如：rt.jar、resources.jar、charsets.jar等**

2. **Extension ClassLoader**：称为扩展类加载器，负责加载Java的扩展类库，默认加载JAVA_HOME/jre/lib/ext/目下的所有jar。
3. **App ClassLoader**：称为系统类加载器，负责加载应用程序classpath目录下的所有jar和class文件。

可以自己定义ClassLoader，但是必须继承自ClassLoader或者Extension ClassLoader或者App ClassLoader，但不可以是BootStrap ClassLoader，因为后者是用C++编写的



### 双亲委托

每个ClassLoader都有一个父类加载器的引用。要加载一个类之前就会先把这个类委托给他的父类加载器：从BootStrap--Extension--App--委托的发起者。

- 避免重复加载
- 安全问题。防止委托者恶意修改核心api的类

所以用户自定义的ClassLoader永远无法加载自己写的一个String类，除非修改JDK



特点：即使是同个文件，编译后的字节码相同，但是如果**是不同的ClassLoader加载的，那么也会被判断不是同一个**

### 定义自己的加载器

1. 继承自ClassLoader

2. 重写findClass方法

   还有一个loadClass方法，里面默认了寻找Class的方法（从Boot开始往下），不建议修改。



## Android的类加载器

### 1. 过程

[Android的类加载器](http://gityuan.com/2017/03/19/android-classloader/)

> Android从5.0开始就采用art虚拟机, 该虚拟机有些类似Java虚拟机, 程序运行过程也需要通过ClassLoader 将目标类加载到内存.
>
> 传统Jvm主要是通过读取class字节码来加载, 而art则是从dex字节码来读取. 这是一种更为优化的方案, 可以将多个.class文件合并成一个classes.dex文件. 下面直接来看看ClassLoader的关系

![img](http://gityuan.com/images/classloader/classloader.jpg)

**总结**

几种类加载器：

- PathClassLoader: 主要用于系统和app的类加载器,**其中optimizedDirectory默认为null**, 采用默认目录/data/dalvik-cache/，**不能加载未安装的apk。**
- DexClassLoader: 可以从包含classes.dex的jar或者apk中，加载类的类加载器, 可用于执行动态加载,但必须是app私有可写目录来缓存odex文件. **能够加载系统没有安装的apk或者jar文件**， 因此很多插件化方案都是采用DexClassLoader;
- BaseDexClassLoader: 比较基础的类加载器, PathClassLoader和DexClassLoader都只是在构造函数上对其简单封装而已.
- BootClassLoader: 作为父类的类构造器。

热修复核心逻辑：在DexPathList.findClass()过程，一个Classloader可以包含多个dex文件，每个dex文件被封装到一个Element对象，这些Element对象排列成有序的数组dexElements。当查找某个类时，会遍历所有的dex文件，如果找到则直接返回，不再继续遍历dexElements。也就是说当两个类不同的dex中出现，**会优先处理排在前面的dex文件**，这便是热修复的核心精髓，将需要修复的类所打包的dex文件插入到dexElements前面。

### 2. dex和odex的区别

> 其实一个APK是一个程序压缩包，里面有个执行程序包含dex文件，**ODEX优化就是把包里面的执行程序提取出来，就变成ODEX文件。**因为你提取出来了，系统第一次启动的时候就不用去解压程序压缩包，少了一个解压的过程。这样的话系统启动就加快了。为什么说是第一次呢？是因为DEX版本的也只有第一次会解压执行程序到 /data/dalvik-cache（针对PathClassLoader）或者optimizedDirectory(针对DexClassLoader）目录，之后也是直接读取目录下的的dex文件，所以第二次启动就和正常的差不多了。当然这只是简单的理解，实际生成的ODEX还有一定的优化作用。ClassLoader只能加载内部存储路径中的dex文件，所以这个路径必须为内部路径。

### 3. 注意的问题

以为上面的热修复的问题，所以在动态加载的时候，如果要记载新版本的dex，必须保证旧类还没有被加载，否则就会一直加载旧类。解决办法：

- 直接编程一个新类，那么两者都会被加载
- 或者把新类插入在前面