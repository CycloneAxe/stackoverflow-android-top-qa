# “Failed to install *.apk on device *: timeout”错误

[问题链接](http://stackoverflow.com/questions/4775603/android-error-failed-to-install-apk-on-device-timeout)

我经常会遇到这个错误，也不知道到底是由什么引起的：当我试着在真机（我用的是三星 Galaxy S）上运行/调试 Android 应用的时候，在终端里看到了这样的错误：

> Failed to install *.apk on device *:
>
> timeout Launch canceled!

终端里显示的就这么多。LogCat 里没有显示任何信息，Eclipse Problems View 也没有显示任何问题。

我试了以下几种方法，但都没有成功：

 1. 清理我的项目（Project > Clean）；
 2. 重启设备、Eclipse、电脑……；
 3. 根据[这个问题](http://stackoverflow.com/questions/4552435/failed-to-install-apk-on-device-emulator-5554-timeout)，将项目移动到一个路径没有空格的地方。

之前我在设备上调试过几次（应用已经在商店上线了），但是这个问题发生得太频繁了，也让人非常沮丧……

任何帮助都可以，谢谢！

## [HpTerm 的答案](http://stackoverflow.com/a/4786299/5152089)

尝试更改 ADB 连接的时限。我记得默认值是 **5000 毫秒**，我改成了 **10000 毫秒**，就没有这个问题了。

如果你用的是 Eclipse，可以到这里修改：

**Window -> Preferences -> Android -> DDMS -> ADB Connection Timeout (ms)**
