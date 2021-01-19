---
layout: post
title:  "Charles重定向本地json文件调试"
categories: Charles 工具
---
项目中经常遇到需要联调后台接口事宜，由于需求时间点排期，导致接口开发人员不能很早的提供json数据，所以我们要自己改造json数据，实现前期不需要依赖接口开发人员就可以进行数据调试。

# 重定向本地json文件调试

运行App，连接Charles代理抓取网络请求
* ##### 运行APP,Charles抓取网络请求

![这里写图片描述](https://upload-images.jianshu.io/upload_images/18406403-7769e4e3efad77eb?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* ##### 粘贴复制响应的json数据到本地创建的json文件夹

![这里写图片描述](https://upload-images.jianshu.io/upload_images/18406403-f8c07a4a03914a98?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
https://blog.csdn.net/kuangdacaikuang/article/details/79573236
* ##### 选择对应的网络请求,右键选择Map Local

![这里写图片描述](https://upload-images.jianshu.io/upload_images/18406403-5a21fc40e8d1488e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* ##### 点击choose选择本地json文件,ok

如此便重定向到本地json文件,每次网络请求返回的都是本地的json数据
![这里写图片描述](https://upload-images.jianshu.io/upload_images/18406403-68dbe888d9d53b83?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* ##### 再次进入App页面即可看到效果

refrence:

[Charles Map Local重定向本地json文件进行调试(图文教程)](https://blog.csdn.net/kuangdacaikuang/article/details/79573236)
