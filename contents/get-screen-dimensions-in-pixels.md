# 获取屏幕维度的像素数

[问题链接](http://stackoverflow.com/questions/1016896/get-screen-dimensions-in-pixels)

我建立了一些自定义元素，想通过编程的方式把他们放在屏幕右上角（距离顶端 `n` 个像素、距离右侧 `m` 个像素）。因此我需要获得屏幕的宽度和高度，然后设置位置：

```java
int px = screenWidth - m;
int py = screenHeight - n;
```

如何在主 Activity 里得到屏幕的宽度和高度？

## [Josef Pfleger 的答案](http://stackoverflow.com/a/1016941/5152089)

如果你想得到屏幕维度的像素数，可以使用 [`getSize`](http://developer.android.com/reference/android/view/Display.html#getSize%28android.graphics.Point%29)：

```java
Display display = getWindowManager().getDefaultDisplay();
Point size = new Point();
display.getSize(size);
int width = size.x;
int height = size.y;
```

如果你不在 `Activity` 之中，可以用 `WINDOW_SERVICE` 获得默认的 `Display`：

```java
WindowManager wm = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);
Display display = wm.getDefaultDisplay();
```

在引入 `getSize` （API 等级 13）之前，你可以使用现在已经废弃的 `getWidth` 和 `getHeight` 方法：

```java
Display display = getWindowManager().getDefaultDisplay(); 
int width = display.getWidth();  // 已废弃
int height = display.getHeight();  // 已废弃
```

然而根据你的情况，在布局中使用 margin/padding 或许更加合适。

### 评论

> ```java
> try {
>     display.getSize(size);
>     width = size.x;
>     height = size.y;
> } catch (NoSuchMethodError e) {
>     width = display.getWidth();
>     height = display.getHeight();
> }
> ```
> —— Arnaud

> 我不知道为什么你要在这种判断上面用 `try/catch`，为什么不用 `if (android.os.Build.VERSION.SDK_INT >= 13)`，这样根本不会抛出任何异常。我觉得在正常的程序流程中使用 `try/catch` 是一种坏习惯。—— CipherCom
