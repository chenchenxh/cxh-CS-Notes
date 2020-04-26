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



