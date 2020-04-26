## 基本使用： 

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

## 源码解析

https://www.jianshu.com/p/37e26f4ea57b

[从OKHttp框架看代码设计](https://juejin.im/post/581311cabf22ec0068826aff)

[OkHttp设计模式剖析（一）建造者模式](https://www.jianshu.com/p/8d69fd920166)