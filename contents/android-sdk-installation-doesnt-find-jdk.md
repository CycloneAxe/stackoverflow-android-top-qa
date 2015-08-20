# 安装 Android SDK 时找不到 JDK

[问题链接](http://stackoverflow.com/questions/4382178/android-sdk-installation-doesnt-find-jdk)

我正试着在 Windows 7 64 位系统上安装 Android SDK。已经安装了 `jdk-6u23-windows-x64.exe`，但是 Android SDK 安装程序不能继续执行，因为找不到 JDK。

这是不是一个已知的问题？有没有解决方法？

![](http://i.stack.imgur.com/pZjuL.jpg)

## [Jurgen 的答案](http://stackoverflow.com/a/5115968/5152089)

看到这个提示时点击***返回***，然后再点击***下一步***。这次就能找到 JDK 了。

### 评论

> 虽然听起来很蠢，不过确实管用。—— ajlane

## [Kenton Price 的答案](http://stackoverflow.com/a/9818884/5152089)

具体的安装：

 - 系统: Windows 8.1
 - JDK 安装文件：jdk-8u11-windows-x64.exe
 - ADT 安装文件：installer_r23.0.2-windows.exe

安装 64 位 JDK，先试试返回-下一步大法，然后试着根据错误信息所说的设置 `JAVA_HOME`。如果还不起作用，试一下这样做：

按照它的说法，设置 `JAVA_HOME` 环境变量，但是路径里用正斜线代替反斜线。

真的。

在我这，把 `JAVA_HOME` 设置成 `C:\Program Files\Java\jdk1.6.0_31` 会出错，但是改成 `C:/Program Files/Java/jdk1.6.0_31` 就没有问题——我快要疯了。

如果还是不行，在环境变量 `Path` 的开头加上 **`%JAVA_HOME;`**。

修改后的系统环境变量：

 - `JAVA_HOME=C:/Program Files/Java/jdk1.8.0_11`
 - `JRE_HOME=C:/Program Files/Java/jre8`
 - `Path=%JAVA_HOME%;C:...`
