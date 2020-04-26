https://www.jianshu.com/p/cbcadec171c0

从上面的Activity启动的原理图可以看到大概的流程是:

**应用将要启动的Activity告诉AMS->AMS检查Activity是否注册->AMS让ActivityThread去创建Activity．**

