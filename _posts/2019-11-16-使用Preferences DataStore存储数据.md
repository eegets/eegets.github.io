---
layout: post
title:  "使用Preferences DataStore存储数据"
categories: dataStore
---


Preferences DataStore 主要应用在 MVVM 当中的 Repository 层，在项目中使用 Preferences DataStore 非常简单，只需要 4 步

### 1. 需要添加 Preferences DataStore 依赖
```
implementation "androidx.datastore:datastore-preferences:1.0.0-alpha01"
```

### 2.创建 DataStore

```
  private val PREFERENCE_NAME = "settings"
val dataStore: DataStore<Preferences> = context.createDataStore(
  name = PREFERENCE_NAME
)

```
### 3. 从 Preferences DataStore 中读取数据

Preferences DataStore 以键值对的形式存储在本地，所以首先我们应该定义一个 Key,与SharedPreferences 的有点不一样，在 Preferences DataStore 中 Key 是一个 Preferences.Key<T> 类型，只支持 `Int` , `Long` , `Boolean` , `Float `, `String`

示例如下

```
val EXAMPLE_COUNTER = preferencesKey<Int>("example_counter")
val exampleCounterFlow: Flow<Int> = dataStore.data
  .map { preferences ->
    // No type safety.
    preferences[EXAMPLE_COUNTER] ?: 0
}

```
### 4.向 Preferences DataStore 中写入数据

 Preferences DataStore 中是通过 DataStore.edit() 写入数据的，DataStore.edit() 是一个` suspend `函数，所以只能在协程体内使用，每当遇到` suspend `函数以挂起的方式运行，并不会阻塞主线程。

以挂起的方式运行，不会阻塞主线程 ：也就是协程作用域被挂起, 当前线程中协程作用域之外的代码不会阻塞。

首先我们需要创建一个 suspend 函数，然后调用 DataStore.edit() 写入数据即可。

```kotlin
suspend fun incrementCounter() {
  dataStore.edit { settings ->
    val currentCounterValue = settings[EXAMPLE_COUNTER] ?: 0
    settings[EXAMPLE_COUNTER] = currentCounterValue + 1
  }
}
```
### Preferences DataStore总结

Preferences DataStore使用起来和SharedPreferences一样简单,但目前Google只是发布了alpha,建议大家等稳定版本出了之后再使用.
