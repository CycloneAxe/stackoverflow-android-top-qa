# 如何让 TextView 的文字水平、垂直居中？

[问题链接](http://stackoverflow.com/questions/432037/how-do-i-center-text-horizontally-and-vertically-in-a-textview-in-android)

我如何把 Android 中的 `TextView` 的文字既水平又垂直居中，从而显示在 `TextView` 的正中间？

## [Bill 的答案](http://stackoverflow.com/a/432155/5152089)

如果你用的是 XML 布局的话：

```xml
<TextView  
    android:layout_width="fill_parent" 
    android:layout_height="fill_parent" 
    android:gravity="center"
    android:text="@string/**yourtextstring**"
/>
```

在 Java 中就像 @stealthcopter 的评论所说的那样：`.setGravity(Gravity.CENTER);`。
