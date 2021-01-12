---
layout: post
title:  "Rxjava Observable转LiveData"
categories: rxjava livedata
---
目前我的网络使用的是Retrofit，通常得到的结果是Observable，同时我们采用了Google最新的LiveData, 下面介绍如何把Observable转换成LiveData

## Observable返回值方法

出于简单介绍目的，我们使用一个很简单的返回Observable的方法

```kotlin
private fun getVideoStatusObservable(): Observable<Boolean> {
        return Observable.just(System.currentTimeMillis() % 2L == 0L)
}
```

## 使用 LiveDataObserverAdapter 转换

```kotlin
fun requestVideoStatus(): LiveData<Boolean> {
        return MutableLiveData<Boolean>().apply {
            getVideoStatusObservable().subscribe(LiveDataObserverAdapter<Boolean>(this, false))
        }
}
```

## 调用 requestVideoStatus

```kotlin
private fun testObservableToLiveData() {
        viewModel(VideoStatusViewModel::class.java).requestVideoStatus().observe(this, Observer {
            ToastUtil.show("testObservableToLiveData value=$it")
        })
}
```

## LiveDataObserverAdapter 具体怎么实现的

```java
package com.secoo.commonsdk.wrapper;

import android.arch.lifecycle.MutableLiveData;

public class LiveDataObserverAdapter<T> extends ObserverAdapter<T>{
    private MutableLiveData<T> mLiveData;
    /**
     * 默认值，用来处理Rx异常的时候，设置LiveData的值
     */
    private T mFallbackValue;

    public LiveDataObserverAdapter(MutableLiveData<T> liveData, T fallbackValue) {
        mLiveData = liveData;
        mFallbackValue = fallbackValue;
    }

    @Override
    public void onError(Throwable e) {
        super.onError(e);
        mLiveData.setValue(mFallbackValue);
    }

    @Override
    public void onNext(T t) {
        super.onNext(t);
        mLiveData.setValue(t);
    }
}
```

主要的有几点

  * 持有外部传入的LiveData，当Observable 有返回值时设置`LiveData.setValue`
  
  * 接收外部传入的fallbackValue，当Observable 出现异常时，使用fallbackValue设置LiveData,如 `mLiveData.setValue(mFallbackValue);`



## 具体的示例代码
  * TestListActivity 搜索即可