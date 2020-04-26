https://www.jianshu.com/p/c5d200dde486



在onCreate方法执行以后调用了setContentView等调用链最后调用了requestLayout和invalidate，但是并没有触发真正的绘制操作。绘制操作是在onResume方法执行的时候。



> 我们知道，在Activity启动的过程中，在ActivityThread中有几个关键的方法:**handleLaunchActivity，performLaunchActivity，handleResumeActivity**，这三个主要的方法基本完成了Activity的创建到启动工作。



1. 先是handleLaunchActivtiy。调用performLaunchActivity，在调用handleResumeActivity

   ```java
   private void handleLaunchActivity(ActivityClientRecord r, Intent customIntent, String reason) { 
       ...
        Activity a = performLaunchActivity(r, customIntent);
        if(a != null) {
            ...
             handleResumeActivity(r.token, false, r.isForward,
                       !r.activity.mFinished && !r.startsNotResumed, r.lastProcessedSeq, reason);
        }
   }
   ```

2. performLaunchActivity做了三件事情

   - 调用了Instrumentation#newActivity创建了Activity
   - 调用了attach方法，新建了一个PhoneWindow对象，并把Activity设置为这个PhoneWindow的Callback回调（此时Activity与Window有了联系，但window中还没有View）
   - 执行了Activity的onCreate\onStart回调

   ```java
    private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
          ...
          //new出了一个Activity
          activity = mInstrumentation.newActivity(
                       cl, component.getClassName(), r.intent);
           ...
           //创建Application对象，是个单例，没有则创建，有了则返回，一个进程一个Application
           Application app = r.packageInfo.makeApplication(false, mInstrumentation);
           ...
           //attch方法内部创建了PhoneWindow对象，并把Activity设置为它的回调
           activity.attach(appContext, this, getInstrumentation(), r.token,
                   r.ident, app, r.intent, r.activityInfo, title, r.parent,
                   r.embeddedID, r.lastNonConfigurationInstances, config,
                   r.referrer, r.voiceInteractor, window, r.configCallback);
           ...
           //执行Activity的onCreate回调
           if (r.isPersistable()) {
               mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
           } else {
               mInstrumentation.callActivityOnCreate(activity, r.state);
           }
           ...
           //执行Activity的onStart回调
           if (!r.activity.mFinished) {
               activity.performStart();
               r.stopped = false;
           }
           ...
    }
   ```

3. handleResumeActivity主要执行两件事

   - 执行了Activity的onResume回调：
   - 把DecorView添加到Window中；

   ```java
   final void handleResumeActivity(IBinder token,
       boolean clearHide, boolean isForward, boolean reallyResume, int seq, String reason) {
           //执行Activity的onResume回调
           r = performResumeActivity(token, clearHide, reason);
           ...
           //将DecorView添加到Window中，然后将这个Window添加到WindowManagerService中
           //然后开始可以执行整个View树的测量布局绘制工作
           if (!a.mWindowAdded) {
               a.mWindowAdded = true;
                   wm.addView(decor, l);
           }
       }
   ```

综上讨论，是在handleResumeActivity的时候才加入view。