---
layout: post
title:  "使用限定数量的List FixedSizeList"
date:   2019-12-14 21:03:36
categories: fixedSizeList
---

## 是什么

  * 一个限定数量的List实现

  * 支持设置一个最大的List大小

  * 在达到最大容量时，自动实现删除旧的元素，增加新的元素
  
## 实现

```kotlin
/**
 * 一个固定内容大小的容器，设定固定大小的item数量，当超过去除旧的item
 * 新的item位于最开始的位置，旧的位于后面的位置
 * 该容器只支持写入，不支持外部显式删除操作
 */
class FixedSizeList<T>(private val maxItemCount: Int = 5) {
    private val backingCollection = LinkedList<T>()

    fun add(item: T?) {
        try {
            if (backingCollection.size >= maxItemCount) {
                backingCollection.removeLast()
            }
            backingCollection.addFirst(item)
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }

    override fun toString(): String {
        return "SizedList(maxItemCount=$maxItemCount, backingCollection=$backingCollection)"
    }

    fun getItems(): List<T?> = backingCollection.toList()
}
```

## 什么时候使用

  * 当记录数据需要进行限制，比如听云现场数据收集，不能无限增加，我们需要保持最近的某些元素，以确保关键信息保留

  * 其他需要进行大小限制的列表
  

## 使用示例

```kotlin
private fun testSizedList() {
    val sizedList = FixedSizeList<String>(5)
    for (n in 0..10) {
        sizedList.add(n.toString())
    }
    sizedList.add(null)
    ToastUtil.show(sizedList.getItems().joinToString())
}
```