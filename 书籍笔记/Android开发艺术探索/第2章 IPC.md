## IPC场景

需要使用到IPC的场景：

1. 同个应用的业务模式需要多进程，或者需要更多的空间。
2. 两个应用在获取对方数据的情况，比如ContentProvider

## Android的多进程模式

### android启用多进程方法的方法

1. 四大组件中可以指定android:process
2. 通过native方法fork一个进程



### Android：process的不同模式

1. 使用`:name`。单独的进程，属于当前应用的**私有进程**，其他应用组件不可以跟它跑在同一个进程中
2. 使用`.name`。**全局进程**，其他应用可以通过ShareUID和它跑在同个进程



> 我们知道Android系统会为每个应用分配一个唯一的UID，**具有相同的UID的应用才能共享数据**。
>
> 两个应用通过ShareUID跑在同个进程是有要求的，需要这两个应用有相同的ShareUID并且签名相同才可以



### 多进程带来的问题

1. 静态成员和单例模式失效
2. 线程同步机制完全失效
3. SharedPreferences的可靠性下降
4. Application会多次创建

> 我们可以这么理解同一个应用间的多进程：它相当于两个不同的应用间采用了SharedUID的模式。



## IPC基础概念

### 序列化

1. Serializable

   类实现Serialiable接口并声明一个serialVersionUID即可（不声明UID也是可以的，但是反序列化过程会有影响）

   > serialVersionUID的工作机制：在序列化的时候会被加入到序列化文件中，反序列化的时候会检查这个ID是否一致，如果一致那么就说明版本相同，就可以成功反序列，否则则失败crash。
   >
   > 如果我们指定了UID，即使版本不同也不会造成crash，系统会尽最大可能还原数据（但是如果类名或者修改了成员变量的类型，也会crash）

   **transient和静态变量都不会参加序列化**

2. Parcelable

   也是一个接口，序列化过程需要 **当前线程的上下文加载器**

优缺点：

> Serializable是Java中的序列化接口，**使用起来很方便但是开销很大**，I/O操作比较多
>
> Parcelable的Android中的序列化方法，在Android中推荐使用，缺点是**使用麻烦，但是效率更高**
>
> Parcelable主要用在内存序列化上，要在**网络上使用和序列化到存储设备上**会比较麻烦。这两种情况推荐使用Serializable



### Binder

> Binder是Android的一个类，实现了IBinder接口
>
> Binder还可以理解为一种虚拟的物理设备，设别驱动是/dev/binder

Binder的两个主要作用：

1. 在**Android Framework层**是ServiceManager连接各种Manager（ActivityManager、WindowManager等等）和想要的ManagerService的桥梁
2. 在**Android应用层**是连接客户端和服务端的媒介



> Android开发中，Binder主要用在Service中，包括AIDL和Messager，其中普通Service中的Binder不涉及进程间通信，所以比较简单。Messager的底层其实是AIDL。

![13](D:\typora\pic\13.png)

### AIDL（android接口定义语言）

简单的例子：需要三个类Book.java、Book.aidl、IBookManager.aidl

1. Book.java就是一个普通的类，但是需要实现Parcelable接口

2. Book.aidl就简单在AIDL中声明Book类

   ```
   parcelable Book;
   ```

3. IBookManager.aidl则是定义接口：

   ```java
   import com.test.Book;  //需要注意Book虽然在同个包，但是需要声明
   interface IBookManager{
       List<Book> getBookList();
       void addBook(Book book);
   }
   ```

根据上面的代码系统会自动为IBookManager.aidl生成一个Binder类java文件**（ 核心实现是Stub和Proxy类）**，也就是说 **我们可以直接写上面的这个Binder类java文件，而不需要aidl。因为这个文件的模式是一样的，所以使用aidl文件来简化工作**



> Binder的两个重要方法：linkToDeath和unlinkToDeath
>
> 如果Binder异常终止，那么我们可以通过设置代理并通过这两个方法重启连接。
>
> 1. DeathRecipient对象。这是一个接口，实现里面的binderDied即可。
> 2. isBinderAlive也可以判断Binder是否死亡



## Android 的IPC方式

### 1. Bundle

### 2. 文件共享

### 3. Messager

信使，轻量级的IPC方案，底层也是AIDL

使用方法：

> 服务端：
>
> 在服务端创建Service，并创建一个Handler，用这个Handler来创建一个Messager对象，并在bind的时候返回这个Messager对应的IBinder对象即可

> 客户端：
>
> bindService的时候，绑定成功之后用服务端的IBinder来新建一个Messager，然后通过这个Messager来发送Message。如果需要这个服务端返回，那么初始化Message的时候加入replyTo对象即可。

缺点：Messager是以串行方式处理客户端的消息

### 4. 使用AIDL

上面说到Messager的缺点，使用AIDL就可以并发处理

> 服务端：
>
> 创建AIDL文件，在其中声明可以给客户端调用的接口，然后实现这个AIDL即可

> 客户端:
>
> bindService成功之后，将Binder转成AIDL接口所需的类型，接着调用对应的方法。
>
> ```java
> //比如binderService之后得到的IBinder service
> IBookManager bookManager = IBookManager.Stub.asInterface(service);
> ```



#### 注意点

1. AIDL文件支持的数据类型：
   - 基本类型
   - String和CharSequence
   - List（虽然是List，但是可以通过向上转型，所以**只要是子类即可**，Map同理）
   - Map
   - Parceable
   - **AIDL**
2. 客户端和服务端的AIDL相关的类和aidl文件都必须一致（为了反序列化成功）
3. AIDL中不能使用普通接口，只能是AIDL接口（还不懂什么意思）
4. 因为序列化之后是不同的对象，所以**无法使用序列化Listener的方式来解除注册**
   - RomoteCallbakcList是系统专门提供的用于删除跨进程Listener的接口。它是一个Map，key是IBinder类型，通过IBinder来识别不同的Listener。**同时它还有一个很有用的功能：在客户端的进程终止时会自动解除注册**
5. **进行远程方法调用（RPC）的一方不能把任务放在UI线程，不然可能会引起ANR。而持有Binder一方不用担心，因为任务是运行在Binder线程池里面的**



#### Binder以为死亡后的解决

1. 之前说的DeathRecipient监听
2. onServiceDisconnected

#### AIDL进行权限验证

1. 在onbind中验证

   在permission中声明权限。失败会返回null的IBinder

2. 通过服务端的onTransact

   也可以在permission中声明权限，如果验证失败返回false

### 5. 使用ContentProvider

底层也是Binder

### 6. 使用socket

套接字通信



## Binder连接池

当服务增多的时候，我们不能增加Service的数量来解决问题。所以需要使用Binder连接池。

> Binder连接池的作用就是将各个业务模块的Binder请求统一转发到远程Service去执行，从而避免重复创建Service的过程。

通过一个Service中实现接口`queryBinder(int binderCode)`来完成对不同Binder的获取

**如果新添加AIDL，那么只需要修改queryBinder方法，返回正确的Binder对象即可**



## 各种IPC的优缺点

![14](D:\typora\pic\14.png)