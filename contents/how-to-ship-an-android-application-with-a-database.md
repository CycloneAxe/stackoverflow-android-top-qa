# 如何将 Android 应用与数据库一同发布？

[问题链接](http://stackoverflow.com/questions/513084/how-to-ship-an-android-application-with-a-database)

如果你的应用需要使用数据库，而且还有需要内置的数据，那么打包发布应用最佳的方式是什么？

 1. 预先生成好 SQLite 数据库文件，包含在 apk 文件里？
 2. 在应用中包含 SQL 命令，从而在第一次使用时创建数据库并插入数据？

我觉得缺点分别是

 1. SQLite 版本不匹配可能会造成问题，而且我目前也不知道应该把数据库文件放在哪、怎么访问；
 2. 在设备上创建和填充数据库可能需要很长时间。

你们的想法是什么？如果有文档的话就更好了。

## [Heikki Toivonen 的答案](http://stackoverflow.com/a/620086/5152089)

我刚刚在 ReignDesign 博客上发现了一篇解决这个问题的文章，标题是[《在 Android 应用中使用自己的 SQLite 数据库》](http://www.reigndesign.com/blog/using-your-own-sqlite-database-in-android-applications/)。总地来说，你要预先生成数据库，放进 apk 文件的 assets 目录中，然后在初次使用的时候复制到 `/data/data/YOUR_PACKAGE/databases/` 目录。
