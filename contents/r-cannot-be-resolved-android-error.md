# “R cannot be resolved”错误

[问题链接](http://stackoverflow.com/questions/885009/r-cannot-be-resolved-android-error)

我刚刚下载和安装了 Android SDK，我想创建一个简单的程序来测试一下。

向导生成了这样的代码：

```java
package eu.mauriziopz.gps;

import android.app.Activity;
import android.os.Bundle;

public class ggps extends Activity {
    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
    }
}
```

但是 Eclipse 报了一个错误：

> R cannot be resolved

在

```java
setContentView(R.layout.main);
```

这一行上。为什么？

PS：我真的已经写了 XML 文件，名字叫 `main.xml`，放在了 `res/layout/` 目录里。

## [Michael Levy 的答案](http://stackoverflow.com/a/3259974/5152089)

通过追踪这个问题，我在 Android 文档里发现了这个：

http://source.android.com/source/using-eclipse.html

> 注意：Eclipse 有时候会在文件开头处添加一条 `import android.R` 语句从而使用资源，尤其是在你要求 Eclipse 去排序或者管理 import 的时候。这会让你的生成出错。留意这些错误的 `import` 语句，并且把它们删掉。

在学习 Android 的教程时，我经常会用 <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>O</kbd> 使用“整理 Import”来生成缺少的 `import` 语句。有的时候这样做会生成出错误的 `import` 语句，这样会将构建时自动生成的 `R.java` 类隐藏起来。

## [Luc 的答案](http://stackoverflow.com/questions/885009/r-cannot-be-resolved-android-error)

每次我遇到 `R` 没有生成甚至消失了的问题，都是因为 XML 布局文件里有一些问题，导致应用程序无法构建。

## [Kailash 的答案](http://stackoverflow.com/a/2779119/5152089)

每当你遇到

> R cannot be resolved

时，检查 `/res` 目录，里面一定会有一些文件有错误，导致程序无法构建。例如，在 XML 文件里使用了不存在的布局或者资源文件。

如果在类似 `res/drawables-mdpi` 目录里你有任何多余的，甚至没用过（！）或者没引用过（！）的图像，命名没有按照文件命名的规定（只包含 `[a-z0-9_.]`），`R.java` 可能就不会生成，从而导致其他答案中提到的一系列问题。希望有所帮助！
