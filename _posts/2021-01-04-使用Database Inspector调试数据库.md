---
layout: post
title:  "使用Database Inspector调试数据库"
categories: Database 数据库
---

在 Android Studio 4.1 及更高版本中，您可以利用 Database Inspector 在应用运行时检查、查询和修改应用的数据库。这对于数据库调试尤为有用。Database Inspector 可处理普通的 SQLite 数据库以及在 SQLite 的基础上构建的库（例如 [Room](https://developer.android.com/training/data-storage/room?hl=zh-cn)）。

> **注意：** Database Inspector 仅可与 API 级别 26 及更高版本的 Android 操作系统中所包含的 SQLite 库结合使用。它无法处理与您的应用捆绑的其他 SQLite 库。

## 打开 Database Inspector

如需在 Database Inspector 中打开数据库，请执行以下操作：

1.  在模拟器或搭载 API 级别 26 或更高版本的已连接设备上[运行您的应用](https://developer.android.com/studio/run?hl=zh-cn)。

    > **注意**：与 Android 11 模拟器有关的已知问题会导致应用在连接到 DB Inspector 时发生崩溃。如需解决此问题，请[按以下步骤操作](https://developer.android.com/studio/known-issues?hl=zh-cn#ki-android-11-db-inspector)。
2.  从菜单栏中依次选择 **View > Tool Windows > Database Inspector**。

3.  从下拉菜单中选择正在运行的应用进程。

4.  当前正在运行的应用中的数据库显示在 **Databases** 窗格中。展开要检查的数据库的节点。

## 查看和修改数据

Databases 窗格显示应用中的数据库列表以及每个数据库包含的表格。双击表格名称即可在检查器窗口的右侧显示其数据。您可以点击列标题，按该列对检查器窗口中的数据进行排序。
![db-inspector-window.png](https://upload-images.jianshu.io/upload_images/18406403-60653cdfcc0b5c18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

您可以通过以下方式修改表中的数据：双击单元格，输入新值，然后按 Enter 键。如果您的应用使用 Room 并且界面会观察数据库（例如使用 LiveData 或 Flow），那么您对数据所做的任何更改会立即显示在正在运行的应用中。否则，只有当应用下次从数据库中读取修改后的数据时，您才会看到更改。

#### 查看实时数据库更改

如果您希望 Database Inspector 在您与正在运行的应用交互时自动更新它呈现的数据，请勾选检查器窗口顶部的 Live updates 复选框。启用实时更新后，检查器窗口中的表格将变为只读状态，您无法修改其中的值。

或者您也可以通过点击检查工具窗口顶部的 Refresh table 按钮以手动更新数据。

## 查询数据库

Database Inspector 可以在应用运行时对应用的数据库运行查询。Database Inspector 可以在您的应用使用 Room 的情况下使用 DAO 查询，但也支持自定义 SQL 查询。

#### 运行 DAO 查询

如果您的应用使用 Room，那么 Android Studio 会提供边线操作，让您可以快速运行您已在 [DAO 类](https://developer.android.com/training/data-storage/room/accessing-data?hl=zh-cn)中定义的查询方法。如果您的应用正在运行且 Database Inspector 已在 Android Studio 中打开，就可以执行这些操作。您可以在 DAO 中运行任何查询方法，方法是点击 `@Query` 注解旁边的 **Run SQLite statement in Database Inspector** 按钮。
![db-inspector-dao-gutters.png](https://upload-images.jianshu.io/upload_images/18406403-79fb44e2940bcf30.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如果应用包含多个数据库，Android Studio 会提示您从下拉列表中选择要查询的数据库。如果您的查询方法包含已命名的[绑定参数](https://developer.android.com/training/data-storage/room/accessing-data?hl=zh-cn#query-params)，Android Studio 会在运行查询之前请求获取每个参数的值。查询结果会显示在检查器窗口中。

#### 运行自定义 SQL 查询

您也可以在应用运行时使用数据库 Database Inspector 运行自定义 SQL 查询来查询您应用的数据库。如需查询数据库，请按以下步骤操作：

1.  点击 **Databases** 窗格顶部的 **Open New Query 标签页** ，在检查器窗口中打开新标签页。
![db-inspector-new-query.png](https://upload-images.jianshu.io/upload_images/18406403-207f95dbaeed07c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.  如果应用包含多个数据库，请从 New Query 标签页顶部的下拉列表中选择要查询的数据库。

3.  在 New Query 标签页顶部的文本字段中输入您的自定义 SQL 查询，然后点击 Run。

New Query 标签页中显示的查询结果是只读的，无法修改。但是，您可以使用自定义 SQL 查询字段运行 UPDATE、INSERT 或 DELETE 等修饰符语句。如果您的应用使用 Room 并且界面会观察数据库（例如使用 LiveData 或 Flow），那么您对数据所做的任何更改会立即显示在正在运行的应用中。否则，只有当应用下次从数据库中读取修改后的数据时，您才会看到更改。

## 其他资源

如需详细了解 Database Inspector，请参阅下面列出的其他资源：

### 博文

*   [Database Inspector：我们期待已久的实时数据库工具！](https://medium.com/androiddevelopers/database-inspector-9e91aa265316)

### 视频

*   [Database Inspector](https://www.youtube.com/watch?v=UMc7Tu0nKYQ&hl=zh-cn) (11 Weeks of Android)

文章来源

[使用 Database Inspector 调试数据库](https://developer.android.com/studio/inspect/database?hl=zh-cn)

扩展阅读

[我又开发了一个非常好用的开源库，调试Android数据库有救了]([https://guolin.blog.csdn.net/article/details/111120730](https://guolin.blog.csdn.net/article/details/111120730)
)