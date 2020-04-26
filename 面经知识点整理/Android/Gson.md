## GSON 的容错

### 1. 注解

#### GSON 提供了注解的方式，来配置最简单的灵活性，这里介绍两个注解 @SerializedName 和 @Expose。

@SerializedName 可以用来配置 JSON 字段的名字，最常见的场景来自不同语言的命名方式不统一，有些使用下划线分割，有些使用驼峰命名法。

还是拿 User 类来举例，Java 对象中定义的用户名称，使用的是userName，而返回的 JSON 数据中，用的是user_name，这样的差异就可以用 @SerializedName 来解决。

#### 再来看看 @Expose，它是用来配置一些例外的字段。

在序列化和反序列化的过程中，总有一些字段是和本地业务相关的，并不需要从 JSON 中序列化出来，也不需要在传递 JSON 数据的时候，将其序列化。

这样的情况用 @Expose 就很好解决。从字面上理解，将这个字段暴露出去，就是参与序列化和反序列化。而一旦使用 @Expose，那么常规的new Gson()的方式已经不适用了，需要GsonBuilder配合.excludeFieldsWithoutExposeAnnotation()方法使用。

@Expose 有两个配置项，分别是serialize和deserialize，他们用于指定序列化和反序列化是否包含此字段，默认值均为 True。

### 2. TypeAdapter

TypeAdapter 是 GSON 2.1 版本开始支持的一个抽象类，用于接管某些类型的序列化和反序列化。TypeAdapter 最重要的两个方法就是 `write()` 和 `read()` ，它们分别接管了序列化和反序列化的具体过程。

**重点在重写Serializer和DeSerializer方法**

比如返回预期是[Object, Object]，但是返回了“”

可以修改如下：

```java
public class DefaultAdapter implements JsonDeserializer<RecipeSearchResult> {
    @Override
    public Object deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context) {
        if (json.isJsonObject()) {
            //如果是array则解析并返回
            Gson gson = new Gson();
            return gson.fromJson(json, typeOfT);
        } else {
            //如果不是array，那么返回默认值
            return new Object();
        }
    }
}
```

**然后重新注册Adapter**

```java
Gson gson = new GsonBuilder()
        .registerTypeAdapter(RecipeSearchResult.class, new DefaultAdapter())
        .create();

private ApiRetrofit(){
    myService = new Retrofit.Builder()
            .addConverterFactory(GsonConverterFactory.create(gson))
            .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
            .baseUrl(MyService.BASE_URL)
            .build()
            .create(MyService.class);
```