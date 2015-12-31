# 如何从应用中启动 Android 的浏览器打开一个 URL？

[问题链接](http://stackoverflow.com/q/2201917)

如何通过代码在内置的网页浏览器中打开一个 URL，而不是在我程序里打开？

我试过这段代码：

```java
try {
    Intent myIntent = new Intent(Intent.ACTION_VIEW, Uri.parse(download_link));
    startActivity(myIntent);
} catch (ActivityNotFoundException e) {
    Toast.makeText(this, "No application can handle this request."
        + " Please install a webbrowser",  Toast.LENGTH_LONG).show();
    e.printStackTrace();
}
```

但是出现了一个异常：

```
No activity found to handle Intent{action=android.intent.action.VIEW data =www.google.com
```

## [Mark B 的答案](http://stackoverflow.com/a/2201999)

试试这个：

```java
Intent browserIntent = new Intent(Intent.ACTION_VIEW, Uri.parse("http://www.google.com"));
startActivity(browserIntent);
```

在我这边是有效的。

至于缺少“http://”的情况，我做了如下的处理：

```java
if (!url.startsWith("http://") && !url.startsWith("https://"))
    url = "http://" + url;
```

我或许还会在用户要输入 URL 的 `EditText` 里预先放上一个“http://”。

### 评论

 > `if (!url.startsWith("http://") && !url.startsWith("https://"))` 是一个常见的错误，在一些例如 file:// 的 URL 里会出现问题。试着用 `URI` 类来解析 URI，并且判断是否有一个 schema。如果没有的话，再加上“http://”。—— tuxSlayer
