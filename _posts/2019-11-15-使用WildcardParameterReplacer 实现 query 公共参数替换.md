## 痛点
  * 各种接口随意传递多余的公共参数，比如upk
  * 每次显式传递这种参数使得方法参数变得臃肿
  
## 怎么解决
  * 利用OKHttp的Interceptor 可以实现的更加简洁和高效

## 我需要怎么做

已位置信息为例，比如我们需要增加一个位置信息

### 添加到URL中
```java
/**
     * 搜索页面，猜你喜欢
     *
     * @return
     */
    @Headers({DOMAIN_NAME_HEADER + Constants.LAS_DOMAIN_NAME})
    @GET("/recommend/recommend?from=search&count=40&locationInfo=_jisuda_location_info_")
    Observable<RecommendListModel> queryRecommendGoods();
```

如上增加了`locationInfo=_jisuda_location_info_`

  * `_jisuda_location_info_ `不再需要encode，这种形式较好
  * 注意使用`%xxxx%`会存在一定的问题，不建议使用
  * 如果所增加参数位于第一个位置，前面是`?`，否则前面是`&`这是一个常识
  
### 创建一个WildcardParameterReplacer
```kotlin
object JisudaLocationInfoReplacer : WildcardParameterReplacer() {
    override fun getWildcard(): String {
        return "_jisuda_location_info_"
    }

    override fun getRealValue(): String {
        return LocationRetriever.getJisudaAddressCodeValue()
    }
}
```

我们创建了一个`JisudaLocationInfoReplacer`用来实现替换`_jisuda_location_info_`为真实值

需要做两件事

  1. 提供想要替换的内容字符串`_jisuda_location_info_`   
  2. 提供替换后的值`LocationRetriever.getJisudaAddressCodeValue()`
  
### 注册WildcardParameterReplacer  
```kotlin
package com.secoo.app.network.url.rewrite

import com.secoo.app.network.url.rewrite.replacer.JisudaLocationInfoReplacer
import com.secoo.commonsdk.arms.di.module.GlobalConfigModule
import com.secoo.commonsdk.http.interceptor.ReplaceParameterInterceptor
import com.secoo.commonsdk.http.interceptor.WildcardParameterReplacer

object AppUrlRewriter {

    fun applyReplaceParameterInterceptor(builder: GlobalConfigModule.Builder?) {
        builder?.addInterceptor(getReplaceParameterInterceptor())
    }

    private fun getReplaceParameterInterceptor(): ReplaceParameterInterceptor {
        return ReplaceParameterInterceptor().apply {
            this.addWildcardParameterReplacer(getParameterReplacerList())
        }
    }

    private fun getParameterReplacerList(): List<WildcardParameterReplacer> {
        return listOf<WildcardParameterReplacer>(
                JisudaLocationInfoReplacer
        )
    }
}
```

其中`getParameterReplacerList`用来注册`WildcardParameterReplacer`


### 运行

## 核心文件

* ReplaceParameterInterceptor.kt

```kotlin
package com.secoo.commonsdk.http.interceptor

import com.secoo.commonsdk.ktx.alsoWithLog
import com.secoo.commonsdk.ktx.smartLog
import okhttp3.HttpUrl
import okhttp3.Interceptor
import okhttp3.Request
import okhttp3.Response

class ReplaceParameterInterceptor : Interceptor {
    private val replacerList = mutableListOf<WildcardParameterReplacer>()

    fun addWildcardParameterReplacer(replacers: List<WildcardParameterReplacer?>?) {
        replacers?.filterNotNull()?.let {
            replacerList.addAll(it)
        }
    }

    override fun intercept(chain: Interceptor.Chain): Response {
        val request = replaceUrlParameter(chain.request())
        return chain.proceed(request)
    }

    private fun replaceUrlParameter(originalRequest: Request): Request {
        val newHttpUrl = replaceUrl(originalRequest.url())
        smartLog {
            "replaceUrlParameter after replacement newHttpUrl=$newHttpUrl"
        }
        return originalRequest.newBuilder().url(newHttpUrl).build()
    }

    private fun replaceUrl(httpUrl: HttpUrl): HttpUrl {
        return replacerList.fold(httpUrl) { httpUrl, replacer ->
            replacer.safeReplace(httpUrl).alsoWithLog(this@ReplaceParameterInterceptor) {
                "replaceUrl url=$httpUrl;replacer=$replacer;result=$it"
            }
        }
    }

}

```

* WildcardParameterReplacer.kt

```kotlin
package com.secoo.commonsdk.http.interceptor

import com.secoo.commonsdk.ktx.getValueSafely
import com.secoo.commonsdk.ktx.okhttp.getQueryParamNameByValue
import com.secoo.commonsdk.ktx.smartLog
import okhttp3.HttpUrl

abstract class WildcardParameterReplacer {

    abstract fun getWildcard(): String

    abstract fun getRealValue(): String

    private fun logWildcardParameterReplacement(httpUrl: HttpUrl, name: String, value: String) {
        smartLog {
            "logWildcardParameterReplacement httpUrl=$httpUrl;name=$name;value=$value"
        }
    }

    private fun replaceNames(httpUrl: HttpUrl, names: List<String>): HttpUrl {
        val realValue = getRealValue()
        val httpUrlBuilder = httpUrl.newBuilder()
        return names.fold(httpUrlBuilder) { builder, name ->
            logWildcardParameterReplacement(httpUrl, name, realValue)
            builder.removeAllQueryParameters(name)
            builder.addQueryParameter(name, realValue)
        }.build()
    }

    fun safeReplace(httpUrl: HttpUrl): HttpUrl {
        return getValueSafely {
            replace(httpUrl)
        } ?: httpUrl
    }

    private fun replace(httpUrl: HttpUrl): HttpUrl {
        val targetParameterNames = httpUrl.getQueryParamNameByValue(getWildcard())
        return if (targetParameterNames.isEmpty()) {
            httpUrl
        } else {
            replaceNames(httpUrl, targetParameterNames)
        }
    }
}

```