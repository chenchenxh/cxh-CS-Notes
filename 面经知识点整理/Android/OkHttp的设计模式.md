[OkHttp设计模式剖析（一）建造者模式](https://www.jianshu.com/p/8d69fd920166)

## 建造者模式

**OkHttpClient类、Request类的新建过程都是建造者模式**

```java
OkHttpClient client = new OkHttpClient.Builder()
                                .addInterceptor(new HttpLoggingInterceptor())  //增加拦截器
                                .cache(new Cache(cacheDir, cacheSize))  //设置用于读取和写入缓存响应的响应缓存。
                                .build();
```

## 责任链模式

**拦截器**的方式就是责任链模式，跟View的分发一样是责任链模式

```java
Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors()); // 应用拦截器
    interceptors.add(retryAndFollowUpInterceptor); // 请求重试
    interceptors.add(new BridgeInterceptor(client.cookieJar())); // 桥接
    interceptors.add(new CacheInterceptor(client.internalCache())); // 缓存
    interceptors.add(new ConnectInterceptor(client)); // 连接
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors()); // 网络拦截器
    }
    interceptors.add(new CallServerInterceptor(forWebSocket)); // 网络调用

    Interceptor.Chain chain = new RealInterceptorChain(
        interceptors, null, null, null, 0, originalRequest);
    return chain.proceed(originalRequest);
  }

//....
  public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec,
      Connection connection) throws IOException {

    //调用责任链中下一个拦截器
    RealInterceptorChain next = new RealInterceptorChain(
        interceptors, streamAllocation, httpCodec, connection, index + 1, request);
    Interceptor interceptor = interceptors.get(index);
    Response response = interceptor.intercept(next);

    return response;
  }
```

## 策略模式
**CacheStrategy类中的策略模式**

事实上，CacheStrategy类中的策略模式并不是典型的策略模式（没用定义不同的策略类，依旧使用ifelse做选择），但是其选择合适策略的思想是一致的。

## 享元模式

享元模式（Flyweight）是对象池的一种实现，可以节约内存，适合可能存在大量重复对象的场景。享元对象中的部分状态可以共享，可以共享的状态为内部状态，不可以共享的状态为外部状态。
 许多某某池都使用了享元模式，比如线程池，连接池，数据池，缓存池，消息池等。享元模式会在池中先找，若没有可以复用的对象，才新建一个。

在OkHttp里面就有**连接池、线程池**

## 观察者模式

OkHttp中**WebSocket监听器**中的观察者模式

其他观察者模式：

1、Android的BroadcastRecevier广播机制
2、ListView中的notifyDataSetChanged()方法

## 外观模式

比如实现一个**OkHttp客户端**就是外观模式，在**add拦截器**等，也都是外观模式，直接new一个就可以了

其他外观模式：

1、Android的Context类
2、EventBus的EventBus类
3、GSON的GSON类

## 迭代器模式

迭代器模式（Iterator Pattern）又称作游标（Cursor）模式，用于遍历一个容器对象，将遍历逻辑封装。

**Cache 类中的迭代器模式**

不仅是OkHttp中的缓存（Cache），基本所有的缓存类都会像一个小数据库一样支持“增删改查”。既然有“查”的功能，那迭代器必不可少。

## 工厂模式

**Call.Factory接口的实现**