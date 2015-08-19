# Eclipse Android 插件提示“Debug certificate expired”错误

[问题链接](http://stackoverflow.com/questions/2194808/debug-certificate-expired-error-in-eclipse-android-plugins)

我用 Eclipse Android 插件构建一个项目，但是在终端窗口里看到了这样的错误：

> [2010-02-03 10:31:14 - androidVNC]Error generating final archive:
> 
> Debug certificate expired on 1/30/10 2:35 PM!

如何解决？

## [Christopher Orr 的答案](http://stackoverflow.com/questions/2194808/debug-certificate-expired-error-in-eclipse-android-plugins/2196397#2196397)

在 **Linux 和 Mac OS X** 中，删除 `~/.android/debug.keystore` 目录下的 debug 证书；在 **Windows** 中，这个目录大概是在 `%USERPROFILE%/.android`。

在下一次构建 debug 包时，Eclipse 插件会生成一个新的证书。你可能需要清理然后重新构建从而生成证书。

## [Jeff Gilfelt 的答案](http://stackoverflow.com/questions/2194808/debug-certificate-expired-error-in-eclipse-android-plugins/2197293#2197293)

在安装之后，Android SDK 会在 debug.keystore 中生成一个“debug”签名证书，Eclipse 插件用这个证书来为每一个生成的程序签名。

不幸的是，debug 证书的有效期只有 365 天。要生成一个新的证书，你必须删除现有的 debug.keystore 文件。它的路径与平台相关，你可以从 **Preferences - Android - Build - Default debug store** 找到它。

## [Dave MacLean 的答案](http://stackoverflow.com/questions/2194808/debug-certificate-expired-error-in-eclipse-android-plugins/4055893#4055893)

要删除所有的 .apk 文件是很痛苦的一件事，因为新的证书不匹配，你无法在 AVD 里升级这些应用。你还需要生成另外一个 MAP-API 开发密钥。以下是另一种解决方法。

你可以在 `debug.keystore` 里创建你自己的 debug 证书，并且任意指定到期时间。在 `HOME` 目录的 `.android` 文件夹内执行以下命令：

```
keytool -genkey -v -keystore debug.keystore -alias androiddebugkey -storepass android -keypass android -keyalg RSA -validity 14000
```

`keytool.exe` 可以在 JDK 的 bin 目录里找到（在 Windows 上例如 `C:\Program Files\Java\jdk1.6.0_31\bin\`）。

ADT 将证书中的名字设置为“Android Debug”，组织是“Android”，两位的国家代码是“US”。你可以将组织、城市和州的值设为“Unknown”。例子中的有效时间是 14000 天，你可以用任何你想用的值。
