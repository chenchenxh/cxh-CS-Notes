## 字节跳动的面试问题：**为什么要有**接口和abstract class的区别？

我给的几个关键点：

- 接口是一种特性，**拥有某种特性**的不一定是**某个东西**
- 抽象类是一个群相似类的抽象，继承自它，那么这个类**就是某个东西**

比如，一个角色是一个类（攻击方式只有拳头），一个角色拥有了一把斧头就多了一种攻击方式：砍。但是这个角色还是“角色”这个类，他只是多了一个特性：砍，这个特性就是一个接口，这个角色就可以去实现一个”砍“这个接口。

总结起来就是：抽象类是一群类的**通性**，而接口是某个类去实现的一个**特性**

由此延伸出一个问题：如何判断一个类是否继承自某个类或者是否实现了某个接口

```java
new Thread(new MyThread());
boolean re1 = Object.class.isAssignableFrom(MyThread.class);  //判断MyThread是否继承自Object
boolean re2 = Runnable.class.isAssignableFrom(MyThread.class);  //判断MyThread是否实现了Runable接口
System.out.println(re1+" "+re2);
```



## 线程的新建方式

1. new Thread
2. 线程池
3. Runable
4. Callable

## Runable和Callable和Thread的区别

Runnable与Callable不同点：

1. Runnable不返回任务执行结果，Callable可返回任务执行结果
2. Callable在任务无法计算结果时抛出异常，而Runnable不能
3. Runnable任务可直接由Thread的start方法或ExecutorService的submit方法去执行



## Runable和Thread新建线程的区别

！可以结合最开头的接口和抽象类看。**自己的总结：Runable是接口，是一种特性，可以让一个类实现这个接口，那么这个类还是原来那个类，但是他具有了Runable这个特性，那么就可以把他当做一个线程来执行**（特殊情况也可以让多个线程来运行它，这时候只需要提供同步机制，就可以实现多线程操作，而继承自Thread就很有限制）

https://blog.csdn.net/ns_code/article/details/17161237

Java中实现多线程有两种方法：继承Thread类、实现Runnable接口，在程序开发中只要是多线程，肯定永远以实现Runnable接口为主，因为实现Runnable接口相比继承Thread类有如下优势：

1、可以避免由于Java的单继承特性而带来的局限；
2、线程代码和任务代码分离，具有好的解耦性。
3、适合多个相同程序代码的线程区处理同一资源的情况。

例如下面这个程序（未同步，可能会出现同步问题）：

```java
//下面代码运行结果可能还是正确的，因为速度够快，但是如果在run里面sleep一段时间，很可能就会出问题
class MyThread implements Runnable{
	private int ticket = 5;
	public void run(){
		for (int i=0;i<10;i++)
		{
			if(ticket > 0){
				System.out.println("ticket = " + ticket--);
			}
		}
	}
}
 
public class RunnableDemo{
	public static void main(String[] args){
		MyThread my = new MyThread();
		new Thread(my).start();
		new Thread(my).start();
		new Thread(my).start();
	}
}
```

一个Runable对象可以让多个线程运行，一方面是代码复用，一方面是多个程序可以处理同个资源。



## 使用Future的好处

https://www.liaoxuefeng.com/wiki/1252599548343744/1306581155184674

Future的get方法很好的替代的了Thread.join或Thread,join(long millis)

一个`Future`接口表示一个未来可能会返回的结果，它定义的方法有：

- `get()`：获取结果（可能会等待）
- `get(long timeout, TimeUnit unit)`：获取结果，但只等待指定的时间；
- `cancel(boolean mayInterruptIfRunning)`：取消当前任务；
- `isDone()`：判断任务是否已完成。

执行下面代码，会发现会**阻塞**。而且submmit还可以是submit(runable, result)，然后这个result最后就会在runable运行结束后放到future里面。

```java
        ExecutorService executorService = Executors.newFixedThreadPool(4);
        Callable<String> task = new Callable<String>() {
            @Override
            public String call() throws Exception {
                Thread.sleep(1000);
                return "string";
            }
        };
        Future<String> future = executorService.submit(task);
        try{
            System.out.println(future.get());
        }catch (Exception e){
            e.printStackTrace();
        }
//会阻塞，等待上面的get方法
        System.out.println(future.isDone());
```

