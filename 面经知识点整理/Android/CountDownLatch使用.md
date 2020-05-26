## Android多线程之 等待多线程运行结束+加载框

https://blog.csdn.net/javaloveiphone/article/details/54729821

使用countDownLatch发射器来计数，等待结果。

1. 定义一个新的Runable抽象类

```java
import java.util.concurrent.CountDownLatch;

public abstract class CountDownLatchRunable implements Runnable{
    CountDownLatch countDownLatch;
    public CountDownLatchRunable(CountDownLatch countDownLatch){
        this.countDownLatch = countDownLatch;
    }
    public void threadCompleted(){
        countDownLatch.countDown(); //计数不断减少，直到0
    }
}
```

2. 在主线程中如下：

```
                //展示加载框
                progressDialog = ProgressDialog.show(AddArticle.this, "", "上传图片中...");
				//定义一定数量，数量看要等待的进程数
				CountDownLatch countDownLatch = new CountDownLatch(2);
                for (final String imagePath : add_photos.getData()) {
                    singleThreadPool.execute(new CountDownLatchRunable(countDownLatch) {
                        @Override
                        public void run() {
                            //上传图片到ftp
                            DoSomething();
                            this.threadCompleted();
                        }
                    });
                }
                try {
                    countDownLatch.await();
                    Log.d(TAG, "all end");
                }catch (Exception e){
                    e.printStackTrace();
                }finally {
                    progressDialog.dismiss(); //关闭加载框
                    singleThreadPool.shutdown(); //关闭线程池
                }
```