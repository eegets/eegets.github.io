---
layout: post
title:  "Handler与Activity,Fragment生命周期解绑，解决延迟执行崩溃问题"
categories: handler 生命周期绑定
---



## 存在的问题

类似这样的代码

```java
delayRun(Runnable {
		smartLog {
            "HandlerGuardTestFragment 没有HandlerGuard(Activity)"
        }
}, 3000)
activity?.finish()
```

会发生的场景

  * activity 销毁

  * Runanble的内容依然会在执行

  * 如果这其中有在Activity销毁进行置为null的代码，就会产生NPE处理。
  
  
## 解决的思路

  * 在Activity(Fragment)销毁前，清除正在消息队列中的任务

## 实现的方法

```java
/**
 * 处理Handler与生命周期载体的解绑关系
 */
object HandlerGuard {

    /**
     * 监控并处理handler的runnable
     * 当activity销毁的时候，从handler中移除runnable待执行任务
     */
    fun watch(activity: Activity?, runnable: Runnable?, handler: Handler? = mainThreadHandler) {
        smartLogD {
            "watch activity=$activity;handler=$handler;runnable=$runnable"
        }

        activity ?: return
        runnable ?: return
        val finalHandler = handler ?: mainThreadHandler

        activity.doOnDestroy {
            finalHandler.removeCallbacks(runnable)
            smartLogD {
                "watch activity($activity).onDestroy remove $runnable from ${finalHandler.toDescription()}"
            }
        }

    }

    fun watch(fragment: Fragment?, runnable: Runnable?, handler: Handler? = mainThreadHandler) {
        smartLogD {
            "watch fragment=$fragment;runnable=$runnable;handler=$handler"
        }

        fragment ?: return
        runnable ?: return
        val finalHandler = handler ?: mainThreadHandler

        fragment.doOnDestroy {
            finalHandler.removeCallbacks(runnable)
            smartLogD {
                "watch fragment($fragment).onDestroy remove $finalHandler from ${finalHandler.toDescription()}"
            }
        }
    }
}

```

## 调用与验证示例

```java
@TestName("HandlerGuard相关")
class HandlerGuardTestFragment : TestableFragment() {
    override fun addTestItems() {

        addTestItem("没有HandlerGuard(Activity)") {
            delayRun(Runnable {
                smartLog {
                    "HandlerGuardTestFragment 没有HandlerGuard(Activity)"
                }
            }, 3000)
            activity?.finish()
        }


        addTestItem("有HandlerGuard(Activity)") {
            val runable = Runnable {
                smartLog {
                    "HandlerGuardTestFragment 有HandlerGuard(Activity)"
                }
            }
            HandlerGuard.watch(activity, runable, mainThreadHandler)
            delayRun(runable, 3000)
            activity?.finish()
        }


        addTestItem("无HandlerGuard(Fragment)") {
            delayRun(Runnable {
                smartLog {
                    "HandlerGuardTestFragment 没有HandlerGuard(Fragment)"
                }
            }, 3000)
            this.remove()
        }


        addTestItem("有HandlerGuard(Fragment)") {
            val runnable = Runnable {
                smartLog {
                    "HandlerGuardTestFragment 有HandlerGuard(Fragment)"
                }
            }
            HandlerGuard.watch(this, runnable, null)
            delayRun(runnable, 3000)
            this.remove()
        }
    }

    override fun runNonUITests() {
    }
}
```
