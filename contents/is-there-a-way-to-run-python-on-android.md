# 有办法在 Android 上面运行 Python 吗？

[问题链接](http://stackoverflow.com/questions/101754/is-there-a-way-to-run-python-on-android)

我们在开发一个 [S60](http://en.wikipedia.org/wiki/S60_%28software_platform%29) 版本，这个平台有一套质量很高的 Python API。

然而，关于 Android 平台的 Python 并没有什么官方的产物。但是既然 [Jython](http://en.wikipedia.org/wiki/Jython) 存在，有没有什么办法能让蛇和机器人一同工作呢？

## [JohnMudd 的答案](http://stackoverflow.com/a/8189603/5152089)

一种方法是使用 [Kivy](http://kivy.org/)：

> 为快速开发带有创新性用户界面（例如多点触控）的应用程序提供的开源 Python 库。
> 
> Kivy 可以在 Linux、Windows、OS X、Android 和 iOS 平台运行。你可以在所有支持的平台上运行同一份 Python 代码。

[Kivy 展示应用](https://play.google.com/store/apps/details?id=org.kivy.showcase)

### 评论

> 如果你使用 Kivy，这里有一个工具可以帮助你把项目打包成 APK：[github.com/kivy/python-for-android](https://github.com/kivy/python-for-android)。——gdw2

## [Heat Miser 的答案](http://stackoverflow.com/a/973786/5152089)

还有一个新的 [Android Scripting Environment](http://www.talkandroid.com/1225-android-scripting-environment/)（ASE）项目，看起来很不错，与原生 Android 组件也有一些集成。

### 评论

> 没错，不过用户必须安装 ASE，因此并不能让你在不安装任何东西的前提下用 Python 编写 Android 应用（一般用户都会说“这个 ASE 是什么鬼？”）。——Stuart Axon

> 此外，ASE 是一个受限制的环境，就算预先安装 ASE，你也无法写出全功能的 Android 应用。参考 [stackoverflow.com/questions/2076381](http://stackoverflow.com/questions/2076381)。——Sridhar Ratnakumar
