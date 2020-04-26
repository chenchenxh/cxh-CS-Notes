## ANR一般有以下四种类型

**KeyDispatchTimeout**
1：KeyDispatchTimeout(5 seconds) –主要类型

按键或触摸事件在特定时间内无响应

**BroadcastTimeout**
2：BroadcastTimeout(10 seconds)

BroadcastReceiver在特定时间内无法处理完成，此处指的是前台广播，后台广播为60秒

**ServiceTimeout**
3：ServiceTimeout(20 seconds) –小概率类型

Service在特定的时间内无法处理完成，此处通常指前台Service，后台Service为200秒

**ContentProviderTimeout**
4:contentprovider的publish在10秒内没进行完，这个比较陌生和少见



## 定位和解决

- 复现
- 日志
- leakCanary性能监控工具



## 一种ANR的实现

```java
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        try {
            Log.d("ANR","开始sleep");
            Thread.sleep(60*1000);
            Log.d("ANR","sleep完成");
            
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```

上述sleep60秒，但是sleep是不会ANR的，但是如果你在sleep的过程中，点击返回或者部分其他操作，那么就会ANR