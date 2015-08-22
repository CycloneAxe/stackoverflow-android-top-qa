# gravity 和 layout_gravity 的区别

[问题链接](http://stackoverflow.com/questions/3482742/gravity-and-layout-gravity-on-android)

我知道 `android:gravity` 和 `android:layout_gravity` 属性可以设置为以下值：

 - `center`；
 - `center_vertical`；
 - `center_horizontal`，等等。

但我对这些东西都很困惑。

`android:gravity` 和 `android:layout_gravity` 在用法上有什么区别？

## [Sephy 的答案](http://stackoverflow.com/a/3482757/5152089)

他们的名字可能会帮助你理解：

 - `android:gravity` 设置 `View` 内容的对齐方式；
 - `android:layout_gravity` 设置 `View` 或者 `Layout` 在父元素中的对齐方式。

[这里](http://thinkandroid.wordpress.com/2010/01/14/how-to-position-views-properly-in-layouts/)有一个例子。
