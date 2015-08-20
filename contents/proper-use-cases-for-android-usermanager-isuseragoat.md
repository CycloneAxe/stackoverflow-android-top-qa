# UserManager.isUserAGoat() 的使用案例

[问题链接](http://stackoverflow.com/questions/13375357/proper-use-cases-for-android-usermanager-isuseragoat)

我正在看 [Android 4.2](http://en.wikipedia.org/wiki/Android_version_history#Android_4.1.2F4.2_Jelly_Bean) 引入的新 API。当看到 [`UserManager`](http://developer.android.com/reference/android/os/UserManager.html) 类的时候遇到了这样一个方法：

> public boolean isUserAGoat ()
> 
> 用于判断调用这个方法的用户是否可以心灵传输。
> 
> 返回调用者是否是一只山羊。

什么时候会用到这个方法，又该怎么使用？

## [meh 的答案](http://stackoverflow.com/questions/13375357/proper-use-cases-for-android-usermanager-isuseragoat/13375461#13375461)

从[源代码](https://android.googlesource.com/platform/frameworks/base/+/android-5.0.0_r6/core/java/android/os/UserManager.java#433)来看，直到在 API 21 中被修改之前，这个方法始终返回 `false`。

```java
/**
 * Used to determine whether the user making this call is subject to
 * teleportations.
 * @return whether the user making this call is a goat 
 */
public boolean isUserAGoat() {
    return false;
}
```

看起来这个方法对开发者来说没有什么作用。之前有人认为这可能是个[彩蛋](http://en.wikipedia.org/wiki/Easter_egg_(media))。

**编辑：**

在 API 21 中，这个方法改为检查是否安装了一个包名为 `com.coffeestainstudios.goatsimulator` 的应用。

```java
/**
 * Used to determine whether the user making this call is subject to
 * teleportations.
 *
 * <p>As of {@link android.os.Build.VERSION_CODES#LOLLIPOP}, this method can
 * now automatically identify goats using advanced goat recognition technology.</p>
 *
 * @return Returns true if the user making this call is a goat.
 */
public boolean isUserAGoat() {
    return mContext.getPackageManager()
            .isPackageAvailable("com.coffeestainstudios.goatsimulator");
}
```

这里是更新后的[源代码链接](https://android.googlesource.com/platform/frameworks/base/+/android-5.0.0_r6/core/java/android/os/UserManager.java)。

### 评论

> 还有猴子：[http://developer.android.com/reference/android/app/ActivityManager.html#isUserAMonkey()](http://developer.android.com/reference/android/app/ActivityManager.html#isUserAMonkey())。——auselen
