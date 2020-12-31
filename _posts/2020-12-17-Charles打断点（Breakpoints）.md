
# Charles实现打断点操作
> 在开发过程中我们有时候想让服务器返回一些特定的内容，方便我们调试一些特定情况。有两种办法：第一种就是苦口婆心的求接口制造一些特定数据，第二种就是不靠别人，靠Charles就可以实现。

Charles Breakpoints 功能就比较适合做一些临时性的修改。比如编辑request参数、重定向request请求资源、编辑response数据。

### 1\. 启用charles断点功能

禁用状态![image](https://upload-images.jianshu.io/upload_images/18406403-52a98ab6a3749e8f.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

启用状态

![image](https://upload-images.jianshu.io/upload_images/18406403-8462137f9a4e8c8b.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2\. 设置断点

*   选择我们要设置断点的接口，双击勾选Breakpoints![image](https://upload-images.jianshu.io/upload_images/18406403-ef252ea688002c27.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   点击Proxy-Breakpoints
*   ![image](https://upload-images.jianshu.io/upload_images/18406403-48129f378f2d1016.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   设置断点条件
*   ![image](https://upload-images.jianshu.io/upload_images/18406403-021aab2fe8d5af4f.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   应用断点条件
*   ![image](https://upload-images.jianshu.io/upload_images/18406403-f5bc52928ccb1851.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3\. 手机端发起请求，执行抓包，修改Response数据

*   我们可以看到 `Edit Response` 选项，点击可以把抓到的Response替换为我们自己想要的json数据。（如果我们设置断点时也选择了Request，这里就会多一个 `Edit Resquest` 选项，我们可以修改Request数据）![image](https://upload-images.jianshu.io/upload_images/18406403-206da8f6cbc5ba68.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   然后点击 `Execute` 继续执行
*   ![image](https://upload-images.jianshu.io/upload_images/18406403-b49590cd2ce6fe21.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这时我们查看请求结果，response 数据已经替换为我们自己想要的了（我这里是把list替换为空数组了）![image](https://upload-images.jianshu.io/upload_images/18406403-28f19136532d4241.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 注意：

使用 `Breakpoints 功能`将网络请求截获并修改过程中，整个网络请求的计时并不会暂停，所以长时间的暂停可能导致客户端的请求超时。

refrence:

[Charles 断点功能(Breakpoints)](https://juejin.cn/post/6857777989829984264)