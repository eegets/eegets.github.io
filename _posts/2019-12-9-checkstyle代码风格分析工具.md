

## 全局执行checkstyle
```java
./gradlew checkstyle
```

## 分模块执行checkstyle
```java
./gradlew CommonSDK:checkstyle
```

## 查找结果html报告（比较适合人为阅读）
```java
find . -name checkstyle.html
./module-tuning/build/reports/checkstyle/checkstyle.html
./module-lint/build/reports/checkstyle/checkstyle.html
./app/build/reports/checkstyle/checkstyle.html
./module-search/build/reports/checkstyle/checkstyle.html
./module-home/build/reports/checkstyle/checkstyle.html
./common-motion/build/reports/checkstyle/checkstyle.html
./module-figuredpop/build/reports/checkstyle/checkstyle.html
./CommonSDK/build/reports/checkstyle/checkstyle.html
./module-user/build/reports/checkstyle/checkstyle.html
./module-share/build/reports/checkstyle/checkstyle.html
./module-category/build/reports/checkstyle/checkstyle.html
./module-cart/build/reports/checkstyle/checkstyle.html
./module-gooddetails/build/reports/checkstyle/checkstyle.html
./module-order/build/reports/checkstyle/checkstyle.html
./module-mine/build/reports/checkstyle/checkstyle.html
./module-payments/build/reports/checkstyle/checkstyle.html
./module-settlement/build/reports/checkstyle/checkstyle.html
./module-liveplay/build/reports/checkstyle/checkstyle.html
./module-brand/build/reports/checkstyle/checkstyle.html
./module-message/build/reports/checkstyle/checkstyle.html
./CommonRes/build/reports/checkstyle/checkstyle.html
./module-webview/build/reports/checkstyle/checkstyle.html
./module-goodslist/build/reports/checkstyle/checkstyle.html
```

## 查找结果xml报告（一般用作解析使用）
```java
find . -name checkstyle.xml
./module-tuning/build/reports/checkstyle/checkstyle.xml
./module-lint/build/reports/checkstyle/checkstyle.xml
./app/build/reports/checkstyle/checkstyle.xml
./module-search/build/reports/checkstyle/checkstyle.xml
./module-home/build/reports/checkstyle/checkstyle.xml
./common-motion/build/reports/checkstyle/checkstyle.xml
./module-figuredpop/build/reports/checkstyle/checkstyle.xml
./CommonSDK/build/reports/checkstyle/checkstyle.xml
./module-user/build/reports/checkstyle/checkstyle.xml
./module-share/build/reports/checkstyle/checkstyle.xml
./module-category/build/reports/checkstyle/checkstyle.xml
./module-cart/build/reports/checkstyle/checkstyle.xml
./module-gooddetails/build/reports/checkstyle/checkstyle.xml
./module-order/build/reports/checkstyle/checkstyle.xml
./module-mine/build/reports/checkstyle/checkstyle.xml
./module-payments/build/reports/checkstyle/checkstyle.xml
./checkstyle.xml
./module-settlement/build/reports/checkstyle/checkstyle.xml
./module-liveplay/build/reports/checkstyle/checkstyle.xml
./module-brand/build/reports/checkstyle/checkstyle.xml
./module-message/build/reports/checkstyle/checkstyle.xml
./CommonRes/build/reports/checkstyle/checkstyle.xml
./module-webview/build/reports/checkstyle/checkstyle.xml
./module-goodslist/build/reports/checkstyle/checkstyle.xml
```

## 如果出现问题怎么办
  * 按照提示的信息修改
  * 然后跑一下上面的checkstyle任务看结果，验证是否修改完成。



## 附加阅读文章
  * [Android代码规范利器： Checkstyle](https://droidyue.com/blog/2016/05/22/use-checkstyle-for-better-code-style/)