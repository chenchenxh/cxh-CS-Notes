[TOC]

## OkHttp

https://juejin.im/post/5a31db566fb9a045257820b2

> 使用OkHttp进行`Get`请求只需要四步即可完成。

1 . 拿到OkHttpClient对象

```
OkHttpClient client = new OkHttpClient();
```

2 . 构造Request对象

```
Request request = new Request.Builder()
                .get()
                .url("https:www.baidu.com")
                .build();
```

> 这里我们采用建造者模式和链式调用指明是进行Get请求,并传入Get请求的地址

**如果我们需要在get请求时传递参数,我们可以以下面的方式将参数拼接在url之后**

```
https:www.baidu.com?username=admin&password=admin
```

3 . 将Request封装为Call

```
Call call = client.newCall(request);	
```

4 . 根据需要调用同步或者异步请求方法

```
//同步调用,返回Response,会抛出IO异常
Response response = call.execute();

//异步调用,并设置回调函数
call.enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {
        Toast.makeText(OkHttpActivity.this, "get failed", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onResponse(Call call, final Response response) throws IOException {
        final String res = response.body().string();
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                contentTv.setText(res);
            }
        });
    }
});
```

## Retrofit

> 一个RESTful的HTTP网络请求框架




```java
NewsService api = retrofit.create(NewsService .class);
Call<News> call = service.getNews("123456");

call.enqueue(new Callback<News>(){
 @Override
 public void onResponse(Response<News> response) {
 //成功返回数据后在这里处理，使用response.body();获取得到的结果
 News news = response.body();
 }
 @Override
 public voidonFailure(Throwable t) {
 //请求失败在这里处理
 }
 });
```



![ç½ç»è¯·æ±åæ°æ³¨è§£](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS1jNTQ3ZjIzNDRlZWY2MzBiLnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA)

- 注解

方法注解，包含@GET、@POST、@PUT、@DELETE、@PATH、@HEAD、@OPTIONS、@HTTP。
 标记注解，包含@FormUrlEncoded、@Multipart、@Streaming。
 参数注解，包含@Query,@QueryMap、@Body、@Field，@FieldMap、@Part，@PartMap。
 其他注解，@Path、@Header,@Headers、@Url



## RxJava

## 一个在 Java VM 上使用可观测的序列来组成异步的、基于事件的程序的库

- observer和subscriber的区别

第一：onStart()，这是 Subscriber 增加的方法。它会在 subscribe 刚开始，而事件还未发送之前被调用，可以用于做一些准备工作，例如数据的清零或重置。它总是在 subscribe 所发生的线程被调用，而不能指定线程。要在指定的线程来做准备工作，可以使用 doOnSubscribe() 方法。

第二：unsubscribe()，这是 Subscriber 所实现的另一个接口 Subscription 的方法，用于取消订阅。在这个方法被调用后，Subscriber 将不再接收事件。一般在这个方法调用前，可以使用 isUnsubscribed() 先判断一下状态，可以在onPause() onStop() 等方法中调用 unsubscribe() 来解除引用关系，以避免内存泄露的发生。

- 下面就是子线程加载图片，主线程显示的代码。

```java
        final ImageView img = (ImageView) findViewById(R.id.img);
        Observable.create(new Observable.OnSubscribe<Drawable>() {
            @Override
            public void call(Subscriber<? super Drawable> subscriber) {
                Drawable drawable = getResources().getDrawable(R.mipmap.my);
                subscriber.onNext(drawable);
                subscriber.onCompleted();
            }
        }).subscribeOn(Schedulers.io())//上面的call方法发生在io线程
                . observeOn(AndroidSchedulers.mainThread())//下面显示图片发生在主线程
                . subscribe(new Observer<Drawable>() {
                    @Override
                    public void onCompleted() {
                        Log.d(TAG, "onCompleted方法执行了");
                    }
                    @Override
                    public void onError(Throwable e) {
                        Toast.makeText(MainActivity.this, e.toString(), Toast.LENGTH_SHORT).show();
                    }
                    @Override
                    public void onNext(Drawable drawable) {
                        Log.d(TAG, "onNext方法执行了");
                        img.setImageDrawable(drawable);
                        Log.d(TAG, "图片加载完成了！");
                    }
                });
```

- Func1 和 Action1 非常相似，也是 RxJava 的一个接口，用于包装含有一个参数的方法。 Func1 和 Action 的区别在于， Func1 包装的是有返回值的方法。

- **Action其实就是onNext，onError，onCompleted的super，也就是可以用三个Action来代替这三个，但是onNext一定要有。**
- 要实现异步的关键：Scheduler

`Schedulers.io()`: I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 `Scheduler`。行为模式和 `newThread()` 差不多，区别在于 `io()` 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 `io()` 比 `newThread()` 更有效率。不要把计算工作放在 `io()` 中，可以避免创建不必要的线程。

- 变换之map()，flatMap

RxJava 提供了对事件序列进行变换的支持，这是它的核心功能之一，也是大多数人说『RxJava 真是太好用了』的最大原因。**所谓变换，就是将事件序列中的对象或整个序列进行加工处理，转换成不同的事件或事件序列。**

关键点在于**拦截！**，第一个observable的执行序列照常，但是执行标志被第二个observable拦截，然后在通知。可以看下面的示例：

```java
        Subscriber<String> subscriber = new Subscriber<String>()
        {
            @Override
            public void onCompleted()
            {
                Log.d(TAG, "Completed方法执行了");
            }
            @Override
            public void onError(Throwable e)
            {
                Log.d(TAG, "onError方法执行了!");
            }
            @Override
            public void onNext(String s)
            {
                Log.d(TAG, "onNext方法执行了: " + s);
            }
        };
        final String string = "string1";
        final String string2 = "string2";
        Observable<String> observable1 = Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                Log.d(TAG,"obserable1: onnext1");
                subscriber.onNext(string);
                Log.d(TAG,"obserable1: nnext2");
                subscriber.onNext(string2);
                Log.d(TAG,"obserable1: onnext3");
                subscriber.onCompleted();
            }
        });
        Observable observable2 =  observable1.map(new Func1<String, String>() {
            @Override
            public String call(String string) {
                Log.d(TAG,"obserable2: incall");
                return string;
            }
        });
        observable2.subscribe(subscriber);
```

执行结果为

```
obserable1: onnext1
obserable2: incall
onNext方法执行了: string1
obserable1: onnext2
obserable2: incall
onNext方法执行了: string2
obserable1: onnext3
Completed方法执行了
```

- 线程

Schedulers.immediate(): 直接在当前线程运行，相当于不指定线程。这是默认的 Scheduler。

Schedulers.newThread(): 总是启用新线程，并在新线程执行操作。

Schedulers.io(): I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。

Schedulers.computation(): 计算所使用的 Scheduler。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算

Android 有一个专用AndroidSchedulers.mainThread()，它指定的操作将在 Android 主线程运行。



## Retrofit+RxJava

1. 只用Retrofit的话，那么interface是**返回一个Call**，然后用Call去调用

```java
public interface MyService{
    @GET("class")
    Call<RecipeClass> getRecipeCall(
            @Query("appkey") String appkey
    );
}
```

2. 如果是RxJava+Retrofit的话，那么是interface是**返回一个Observable**

```java
public interface MyService{
    @GET("class")
    Observable<RecipeClass> getRecipeObservable(
            @Query("appkey") String appkey
    );
}
```

3. 在这个Observable的基础上，就可以使用**doOnNext,doOnSubscribe,doOnCompleted**来实现被观测者的定义

注意：上面三个doOn，**同一项（例如多个doOnNext内）顺序决定执行顺序，不同项之间按doOnSubscribe--doOnNext--doOnCompleted的顺序**

示例如下：

```java
Retrofit retrofit1 = new Retrofit.Builder()
    .addConverterFactory(GsonConverterFactory.create())
    .addCallAdapterFactory(RxJavaCallAdapterFactory.create())//新的配置
    .baseUrl(baseurl)
    .build();
MyService myService = retrofit1.create(MyService.class);

myService.getRecipeObservable(appkey)
    .doOnSubscribe(new Action0() {
        @Override
        public void call() {
            Log.d(TAG,"do subscribe");
        }
    })
    .doOnNext(new Action1<RecipeClass>() {
        @Override
        public void call(RecipeClass recipeClass) {
            Log.d(TAG,"do  next" + recipeClass.getMsg());
        }
    })
    .doOnCompleted(new Action0() {
        @Override
        public void call() {
            Log.d(TAG,"do  completed");
        }
    })
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Observer<RecipeClass>() {
        @Override
        public void onCompleted() {
            Log.d(TAG,"completed");
        }

        @Override
        public void onError(Throwable e) {
            Log.d(TAG,"error:"+e.getMessage());
        }

        @Override
        public void onNext(RecipeClass recipeClass) {
            Log.d(TAG,"next"+ recipeClass.getMsg());
        }
    });
```