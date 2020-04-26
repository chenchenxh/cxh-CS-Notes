## 一个RESTful的HTTP网络请求框架




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

