# 让 EditText 在 Activity 启动时不自动获得焦点

[问题链接](http://stackoverflow.com/questions/1555109/stop-edittext-from-gaining-focus-at-activity-startup)

我有一个 `Activity`，里面有两个元素：

 1. `EditText`；
 2. `ListView`。

当我的 `Activity` 启动时，`EditText` 会立即得到焦点（光标闪动）。我不希望任何控件在启动时得到焦点。我试过：

```java
EditText.setSelected(false);
```

但是没有用。如何才能让 `EditText` 在 `Activity` 启动时不选中自己呢？

## [Morgan Christiansson 的答案](http://stackoverflow.com/questions/1555109/stop-edittext-from-gaining-focus-at-activity-startup/1662088#1662088)

Luc 和 Mark 的答案很精彩，不过缺少一个好的例子：

```xml
<!-- 用来阻止 AutoCompleteTextView 接受焦点的假元素 -->
<LinearLayout
    android:focusable="true"
    android:focusableInTouchMode="true"
    android:layout_width="0px"
    android:layout_height="0px"/>

<!-- 将这个组件的 :nextFocusUp 和 :nextFocusLeft 属性设置为自身 id，
     防止假元素重新获得焦点 -->
<AutoCompleteTextView
    android:id="@+id/autotext"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:nextFocusUp="@id/autotext"
    android:nextFocusLeft="@id/autotext"/>
```

## [Joe 的答案](http://stackoverflow.com/questions/1555109/stop-edittext-from-gaining-focus-at-activity-startup/2611031#2611031)

你真正想解决的问题真的是不想让它得到焦点吗？还是说不想让虚拟键盘作为 EditText 得到焦点的结果出现？我并不觉得 EditText 在启动时获得焦点有什么问题，但是当用户并没有明显地要求 EditText 得到焦点却弹出软键盘一定是个问题。

如果是虚拟键盘的问题，参考 AndroidManifest.xml 的 [<activity> 元素](http://developer.android.com/intl/zh-CN/guide/topics/manifest/activity-element.html#wsoft)的文档。

 - `android:windowSoftInputMode="stateHidden"`：进入 activity 时永远隐藏键盘；
 - 或者 `android:windowSoftInputMode="stateUnchanged"`：不要做出改变（例如，如果键盘没有显示出来，那就不要显示；如果进入 activity 时键盘已经打开了，那就保持打开）。

## [Silver 的答案](http://stackoverflow.com/questions/1555109/stop-edittext-from-gaining-focus-at-activity-startup/8639921#8639921)

有个更加简单的方法。在父元素中设置以下属性：

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/mainLayout"
    android:descendantFocusability="beforeDescendants"
    android:focusableInTouchMode="true" >
```

这样，activity 启动的时候这个 main layout 就会默认得到焦点。

此外，还可以通过将焦点再次交给 main layout，在运行时去除子 view 的焦点（例如在完成编辑之后），就像这样：

```java
findViewById(R.id.mainLayout).requestFocus();
```

来自 [Guillaume Perror](http://stackoverflow.com/users/598520/guillaume-perrot) 的评论很不错：

> android:descendantFocusability="beforeDescendants" 似乎是默认值（它的整数值是 0）。只要加上 android:focusableInTouchMode="true" 就可以了。

真的，我们可以看到 `beforeDescendants` 是 `ViewGroup.initViewGroup()` 方法的默认值（在 Android 2.2.2 中），然而这个值并不等于 0。`ViewGroup.FOCUS_BEFORE_DESCENDANTS = 0x20000`。

感谢 Guillaume。

## [Luc 的答案](http://stackoverflow.com/questions/1555109/stop-edittext-from-gaining-focus-at-activity-startup/1612017#1612017)

我找到的唯一解决方法是：

 - 建立一个 LinearLayout（我不知道其他类型的 Layout 是否可以）*（所有继承自 `View` 的组件都具有这两个属性——译者注）*；
 - 设置属性 `android:focusable="true"`，`android:focusableInTouchMode="true"`。

这样 EditText 就不会在 activity 启动之后得到焦点了。
