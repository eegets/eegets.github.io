## 什么是DataStore

Jetpack DataStore是一种新型数据存储解决方案

Jetpack DataStore是以 `Kotlin协程` 和`Flow`功能为基础提供了两种方式


* Proto DataStore    存储类的对象,通过 protocol buffers 将对象序列化存储在本地
* Preferences DataStore  以键值对的形式存储在本地和SharedPreferences类似,基于 Flow 实现的，不会阻塞主线程，并且保证类型安全

## 为什么Google要替换SharedPreferences

  
  其实SharedPreferences作为一种轻量级的数据存储方式，使用起来也非常方便,以键值对的形式存储在本地，初始化 SharedPreference 的时候，会将整个文件内容加载内存中，因此会带来以下问题：
  
  * `getXXX()`获取值的形式有可能会导致主线程阻塞
  * SharedPreferences不能保证类型安全, 加载的数据会一直留在内存中，浪费内存
  * `apply()`方法虽然是异步的，可能会发生 ANR
  *  `apply()` 方法无法获取到操作成功或者失败的结果
  
  
  
### (1) 为什么getXXX()方法会导致主线程阻塞

因为getXXX()都是同步的，在主线程调用 get 方法时，同步方法内调用了 wait() 方法，会一直等待 getSharedPreferences() 方法开启的线程读取完数据才能继续往下执行，如果数据量读取的小,并没有什么影响,如果读取的文件较大会导致主线程阻塞


具体大家可以查看haredPreferences源码
```
frameworks/base/core/java/android/app/SharedPreferencesImpl.java
```


### (2) SharedPreferences不能保证类型安全,并且一直会留在内存中

看一个例子
``` 
    val key = "DataStore"
    val sp = getSharedPreferences("文件名", Context.MODE_PRIVATE) 
    sp.edit { putInt(key, 0) } // 使用 Int 类型的数据覆盖相同的 key
    sp.getString(key, ""); // 使用相同的 key 读取 Sting 类型的数据
    
```
使用 Int 类型的数据覆盖掉相同的 key，然后使用相同的 key 读取 Sting 类型的数就会造成`ClassCastException`异常
而通过 getSharedPreferences() 方法加载的数据，最后会将数据存储在静态的成员变量中。

### (3 apply()方法是异步的,可能会发生ANR

异步的提交为什么会发生ANR呢,这里有一个关于这个问题的文章分析

[剖析 SharedPreference apply 引起的 ANR 问题](https://mp.weixin.qq.com/s?__biz=MzI1MzYzMjE0MQ==&mid=2247484387&idx=1&sn=e3c8d6ef52520c51b5e07306d9750e70&scene=21#wechat_redirect)


## DataStore带来了哪些改变呢?

与其说DataStore相对SharedPreference的改变,不如说是Preferences DataStore,因为Preferences DataStore主要是替换SharedPreference的,并且解决了SharedPreference所有问题

* DataStore 是基于 Flow 实现的，所以保证了在主线程的安全性
* 以事务方式处理更新数据，事务有四大特性（原子性、一致性、 隔离性、持久性）
* 没有 apply() 和 commit() 等等数据持久的方法
* 自动完成 SharedPreferences 迁移到 DataStore，保证数据一致性，不会造成数据损坏
* 可以监听到操作成功或者失败结果


## 目前DataStore已经发布alpha01版本

具体可以参看[JetPack DataStore](https://developer.android.com/topic/libraries/architecture/datastore)