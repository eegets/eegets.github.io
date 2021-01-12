---
layout: post
title:  "KotlinKnife异常java.lang.IllegalStateException ID not found"
date:   2020-12-15 21:03:36
categories: Exception KotlinKnife
---
我们项目中使用KotlinKnife实例化界面布局时有时候会出现如下错误

```kotlin
Caused by: java.lang.IllegalStateException: View ID 2131297254 for 'playView' not found.
at kotterknife.KotlinKnifeKt.viewNotFound(KotlinKnife.kt:100)
at kotterknife.KotlinKnifeKt.access$viewNotFound(KotlinKnife.kt:1)
at kotterknife.KotlinKnifeKt$required$1.invoke(KotlinKnife.kt:104)
at kotterknife.KotlinKnifeKt$required$1.invoke(Unknown Source:2)
at kotterknife.Lazy.getValue(KotlinKnife.kt:127)
at com.xxx.xxx.live.activity.LivePlayExplainActivity.getPlayView(Unknown Source:7)
at com.xxx.xxx.live.activity.LivePlayExplainActivity.initData(LivePlayExplainActivity.kt:144)
at com.xxx.xxx.arms.base.BaseActivity.onCreate(BaseActivity.java:140)
at com.xxx.xxx.live.activity.LivePlayExplainActivity.onCreate(LivePlayExplainActivity.kt:130)
at android.app.Activity.performCreate(Activity.java:7984)
at android.app.Activity.performCreate(Activity.java:7973)
at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1307)
at com.qiyukf.unicorn.m.a.callActivityOnCreate(QiyuInstrumentation.java:258)
at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:3523)
at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:3705)
at android.app.servertransaction.LaunchActivityItem.execute(LaunchActivityItem.java:83)
at android.app.servertransaction.TransactionExecutor.executeCallbacks(TransactionExecutor.java:140)
at android.app.servertransaction.TransactionExecutor.execute(TransactionExecutor.java:100)
at android.app.ActivityThread$H.handleMessage(ActivityThread.java:2254)
at android.os.Handler.dispatchMessage(Handler.java:107)
at android.os.Looper.loop(Looper.java:238)
at android.app.ActivityThread.main(ActivityThread.java:7937)
at java.lang.reflect.Method.invoke(Native Method)
at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:492)
at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:996)
```
可以看到是因为我们某一个id没找到

```kotlin
Caused by: java.lang.IllegalStateException: View ID 2131297254 for 'playView' not found.
```
究其原因可能是因为id重名了，可以查找一下对应的`2131297254`是否有重名id，如果有的话修改一下id命名，让其每一个都唯一，大概就可以解决该问题
