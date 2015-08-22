# "Activity has leaked window that was originally added"错误

[问题链接](http://stackoverflow.com/questions/2850573/activity-has-leaked-window-that-was-originally-added)

这是什么错误，为什么会发生？

```
05-17 18:24:57.069: ERROR/WindowManager(18850): Activity com.mypkg.myP has leaked window com.android.internal.policy.impl.PhoneWindow$DecorView@44c46ff0 that was originally added here
05-17 18:24:57.069: ERROR/WindowManager(18850): android.view.WindowLeaked: Activity ccom.mypkg.myP has leaked window com.android.internal.policy.impl.PhoneWindow$DecorView@44c46ff0 that was originally added here
05-17 18:24:57.069: ERROR/WindowManager(18850):     at android.view.ViewRoot.<init>(ViewRoot.java:231)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at android.view.WindowManagerImpl.addView(WindowManagerImpl.java:148)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at android.view.WindowManagerImpl.addView(WindowManagerImpl.java:91)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at android.view.Window$LocalWindowManager.addView(Window.java:424)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at android.app.Dialog.show(Dialog.java:239)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at com.mypkg.myP$PreparePairingLinkageData.onPreExecute(viewP.java:183)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at android.os.AsyncTask.execute(AsyncTask.java:391)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at com.mypkg.myP.onCreate(viewP.java:94)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1047)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2544)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2621)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at android.app.ActivityThread.access$2200(ActivityThread.java:126)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1932)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at android.os.Handler.dispatchMessage(Handler.java:99)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at android.os.Looper.loop(Looper.java:123)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at android.app.ActivityThread.main(ActivityThread.java:4595)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at java.lang.reflect.Method.invokeNative(Native Method)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at java.lang.reflect.Method.invoke(Method.java:521)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:860)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:618)
05-17 18:24:57.069: ERROR/WindowManager(18850):     at dalvik.system.NativeStart.main(Native Method)
```

## [Alex Volovoy 的答案](http://stackoverflow.com/a/2850597/5152089)

你在退出了 `Activity` 之后显示了一个 `Dialog`。

### 评论

> 这个错误在一些情况下可能会有点误导（尽管这个答案完全正确）。例如，我在 `AsyncTask` 里遇到了一个未处理的异常，导致 `Activity` 结束，然后一个进度对话框引发了这个问题。因此“真正的”异常应该在记录里稍微靠前一点的地方。—— Bobby

## [Márton Molnár 的答案](http://stackoverflow.com/a/2851833/5152089)

解决方法是让你在 `viewP.java:183` 创建的 `Dialog` 在退出 `Activity` 之前调用 `dismiss()`，例如在 `onPause()` 方法中。所有的 `Window` 和 `Dialog` 都应该在离开 `Activity` 之前关闭。
