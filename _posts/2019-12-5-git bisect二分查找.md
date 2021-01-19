---
layout: post
title:  "git bisect二分查找"
categories: bisect git
---
##  使用  git bisect  二分查找定位bug

## 原理

![gitBisect.png](https://upload-images.jianshu.io/upload_images/18406403-f58d98183c26c91c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**原理： 将代码提交的历史，按照两分法不断缩小定位。
所谓"两分法"就是将代码历史一分为二，确定问题出在前半部分，还是后半部分，不断执行这个过程，直到范围缩小到某一次代码提交。**

## 使用

### 第一步：

使用命令：**git bisect  start  [终点提交hash值]   [起点提交hash值]**

* 栗子：

```java
$  git bisect start 5f1694b2b54e2f805636d0c5e304e0c611fe8d6c  a4bfb09830be3b924f3c34332a9145b25c843734
```

* 结果如下：

![git-bisect-使用描述.png](https://upload-images.jianshu.io/upload_images/18406403-8187ff9da2ad03f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 图中的标注释意：

 * 标注1：可以看到大概还需要几步、还有几个版本要测试
 
 * 标注2： 当前使用二分查找的 起点 和 终点 提交hash值记录的 正中间提交的hash值，及提交的描述文案

 * 标注3： 查看当前git指针所在的位置

> **注意:  使用`git bisect  start  [终点提交hash值]   [起点提交hash值]` 后会直接切换到  起点和终点提交的 正中间分支**

### 第二步：

先运行`第一步`二分查找后所在的 正中间分支，确认要定位的`bug`是否存在

* 无问题

输入 命令，标记当前分支正确

```java
$ git bisect good
```

结果如下：

![git_bisect_good.png](https://upload-images.jianshu.io/upload_images/18406403-7220763356f71986.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 图中的标注释意：

 * 标注1：命令

 * 标注2： 可以看到大概还需要几步、还有几个版本要测试

 * 标注3： 当前使用二分查找的 起点 和 终点 提交hash值记录的 正中间提交的hash值，及提交的描述文案, 当前git指针所在的位置

* 有问题

输入 命令，标记当前分支有问题

```java
$ git bisect bad
```
**其结果 与  问题结果相似**

### 第三步

根据`第二步`每次运行检查的结果 ，循环判断执行 `第二步`的判断，直到出现：`提交hash值   is the first bad commit`

例子：

```java
33417f245510f025da05285d745e8c9205a6d74c is the first bad commit
```
说明 这里就是 出现的问题所在的第一次提交,然后分析定位当次修改的内容是那部分造成的，就可以开始修复错误了。

