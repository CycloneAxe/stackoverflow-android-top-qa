# 关闭 / 隐藏 Android 软键盘

[问题链接](http://stackoverflow.com/questions/1109022/close-hide-the-android-soft-keyboard)

我在布局上放了一个 `EditText` 和一个 `Button`。在文本框里输入内容并且点击 `Button` 之后，我想把虚拟键盘隐藏起来。我觉得会有一些简单的，一行或者两行的方法来实现这个效果。我该在哪找到这样的例子呢？

## [Reto Meier 的答案](http://stackoverflow.com/questions/1109022/close-hide-the-android-soft-keyboard/1109108#1109108)

你可以使用 [InputMethodManager](http://developer.android.com/reference/android/view/inputmethod/InputMethodManager.html) 强制让 Android 隐藏虚拟键盘。调用 [`hideSoftInputFromWindow`](http://developer.android.com/reference/android/view/inputmethod/InputMethodManager.html#hideSoftInputFromWindow%28android.os.IBinder,%20int%29)，把包含文本框的窗口标记（token）传递进去。

```java
EditText myEditText = (EditText) findViewById(R.id.myEditText);
// Check if no view has focus:
View view = this.getCurrentFocus();
if (view != null) {  
    InputMethodManager imm = (InputMethodManager)getSystemService(Context.INPUT_METHOD_SERVICE);
    imm.hideSoftInputFromWindow(view.getWindowToken(), 0);
}
```

这会在任何情况下都强制隐藏键盘。一些时候你可能会想在第二个参数使用 `InputMethodManager.HIDE_IMPLICIT_ONLY` 来确保仅仅在用户没有显式调用键盘（按下菜单）时隐藏。

## [Garnet Ulrich 的答案](http://stackoverflow.com/questions/1109022/close-hide-the-android-soft-keyboard/2059394#2059394)

另一种隐藏软键盘的方法是：

```java
getWindow().setSoftInputMode(
    WindowManager.LayoutParams.SOFT_INPUT_STATE_ALWAYS_HIDDEN
);
```

这个方法可以在用户触摸 `EditText` 之前让软键盘始终保持隐藏。

## [rmirabelle 的答案](http://stackoverflow.com/questions/1109022/close-hide-the-android-soft-keyboard/17789187#17789187)

为了弄清楚这种疯狂的行为，作为开篇，我要代表所有 Android 用户为 Google 对软键盘完全荒唐的对待方式而道歉。同一个简单的问题有如此多种各不相同的答案，其原因和 Android 的其他许多方面一样，是糟糕的 API 设计。对此我想不出任何礼貌的方式来陈述。

我想把键盘隐藏掉，而我希望通过这样的语句来实现：`Keyboard.hide()`。就这一句，谢谢。但是 Android 有问题，你必须用 `InputMethodManager` 来隐藏键盘。好的，可以，这就是 Android 操作键盘的 API。但是！它要求你必须拥有一个 `Context` 才能访问到 IMM，现在问题来了，我可能想在一个静态或者工具类里隐藏键盘，但是没有办法也没有需要访问任何的 `Context`。或者（也更糟），IMM 需要你指定从哪一个 `View`（或者更糟，从哪一个 `Window`）隐藏键盘。

这就是让隐藏键盘变得如此困难的原因。亲爱的 Google，当我查阅蛋糕的做法时，根本不会有一个让我除非先回答谁会吃蛋糕、在什么地方吃蛋糕，否则就不告诉我蛋糕做法的 `RecipeProvider`！

这个悲伤的故事将会以一个丑陋的事实结束：要隐藏键盘，你需要提供两种鉴定方式——一个是 `Context`，另一个要么是 `View`，要么是 `Window`。

我写了一个静态工具方法，从而能可靠地完成这个操作，只需要提供一个 Activity：

```java
public static void hide_keyboard(Activity activity) {
    InputMethodManager inputMethodManager = (InputMethodManager) activity.getSystemService(Activity.INPUT_METHOD_SERVICE);
    // 找到当前获得焦点的 view，从而可以获得正确的窗口 token
    View view = activity.getCurrentFocus();
    // 如果没有获得焦点的 view，创建一个新的，从而得到一个窗口的 token
    if(view == null) {
        view = new View(activity);
    }
    inputMethodManager.hideSoftInputFromWindow(view.getWindowToken(), 0);
}
```

要注意，这个方法只有从一个 Activity 调用时才有效！这个方法调用了目标 Activity 的 getCurrentFocus 从而获取适当的窗口 token。

但如果你想从一个 DialogFragment 中的 EditText 来隐藏键盘呢？你不能像这样调用上面的方法：

```java
hide_keyboard(get_activity()); // 没用
```

这样不会达到效果，因为你传递的是 Fragment 的宿主 Activity 的引用，而在显示 Fragment 的时候，这个 Activity 是没有获得焦点的控件的！因此，要从 Fragment 隐藏键盘，需要用到更加低级的、更加普遍也更丑陋的方法：

```java
public static void hide_keyboard_from(Context context, View view) {
    InputMethodManager inputMethodManager = (InputMethodManager) context.getSystemService(Activity.INPUT_METHOD_SERVICE);
    inputMethodManager.hideSoftInputFromWindow(view.getWindowToken(), 0);
}
```

以下是为了解决这个问题而四处搜集到的一些额外信息：

**关于 windowSoftInputMode**

还有另一种需要注意的观点。默认情况下，Android 会自动为 Activity 的第一个 EditText 或者其他可获取焦点的控件赋予初始焦点，输入法（通常也就是软键盘）也自然会响应这个事件，自动出现。`AndroidManifest.xml` 中的 `windowSoftInputMode` 属性如果设置为 `stateAlwaysHidden`，键盘就会忽略这个自动分配初始焦点的事件。

```xml
<activity
    android:name=".MyActivity"
    android:windowSoftInputMode="stateAlwaysHidden"/>
```

几乎无法相信的是，上面的代码对于触摸控件居然毫无作用，键盘仍然会打开（除非为控件设置 `focusable="false"` 并且/或者 `focusableInTouchMode="false"`）。很明显，windowSoftInputMode 只对自动获得焦点事件有作用，而对触摸触发的焦点事件没有影响。

因此，`stateAlwaysHidden` 这个名字取得太烂了。或许叫做 `ignoreInitialFocus` 还不错。

希望能有所帮助。
