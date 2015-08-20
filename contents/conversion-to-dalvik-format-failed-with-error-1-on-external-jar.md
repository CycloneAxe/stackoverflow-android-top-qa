# 外部 JAR 提示“Conversion to Dalvik format failed with error 1”

[问题链接](http://stackoverflow.com/questions/2680827/conversion-to-dalvik-format-failed-with-error-1-on-external-jar)

用 Eclipse 开发 Android 应用的时候遇到了这样的错误：

> UNEXPECTED TOP-LEVEL EXCEPTION:
>
> java.lang.IllegalArgumentException: already added: Lorg/xmlpull/v1/XmlPullParser;
>
> ....
>
> Conversion to Dalvik format failed with error 1

当我在添加一个特定的外部 JAR 文件时出现了这个错误。我花了很长时间搜索各种可能的解决方法，但是没有一个起作用。我还试过从 Android 1.5（目前我正在使用的版本）切换到 Android 1.6。

## [user408841 的答案](http://stackoverflow.com/a/3389640/5152089)

打开 Project » Properties » Java Build Path » Libraries，将除了“Android X.Y”（例如在我这边是 Android 1.5）之外的全部删除，点击确定。点击 Project » Clean » Clean projects selected below » 选择你的项目然后点击确定。这样应该就可以了。

也可能是你把 JAR 文件放在了项目目录的某个地方（我把 Admob 的 JAR 文件放在了我的 src 目录里），又把它添加为 Java Path Library。它不会在 Package Explorer 中显示，所以你并不会察觉，但是这个文件会被计数两次，从而造成可怕的 Dalvik error 1。

还有一种可能是包名冲突。假如你有一个包 `com.abc.xyz`，包中有一个名叫 `A.java` 的类。另一个库项目（添加到了这个项目的依赖项中）包含同样的 `com.abc.xyz.A.java`，那么你就会遇到同样的错误。这意味着多次引用了同一个文件 `A.java` ，无法正确地构建项目。

如果你有意或者无意地修改或者添加了 classpath 文件，也可能会导致这个问题的出现。一些情况下，我们为了生成 javadoc 可能会手动向 classpath 中添加 `android.jar`，生成了 javadoc 之后把它删除掉就可以恢复正常。如果错误仍然发生，请试试这个方法。

## [michel 的答案](http://stackoverflow.com/a/2681165/5152089)

我解决了这个问题。

似乎是因为我的 buildpath 里有两个 JAR 文件包含了相同的包和类。

`smack.jar` 和 `android_maps_lib-1.0.2`。

从其中一个 JAR 文件里删除掉这个包就可以解决问题。
