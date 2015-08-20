# 旋转屏幕导致 Activity 重新启动

[问题链接](http://stackoverflow.com/questions/456211/activity-restart-on-rotation-android)

在我的 Android 应用里，当我旋转屏幕（滑出侧滑键盘）时我的 `Activity` 会重新启动（调用 `onCreate`）。这大概是应当会发生的行为，但是我在 `onCreate` 中做了大量的初始化工作，所以我需要：

 1. 把所有的初始化工作放进另一个方法里，从而在屏幕旋转的时候不会丢失；或者
 2. 在不重新调用 `onCreate` 的条件下调整布局；或者
 3. 限制应用只使用竖屏模式从而不会再次调用 `onCreate`。

## [Reto Meier 的答案](http://stackoverflow.com/a/456918/5152089)

**使用 `Application` 类**

根据你在初始化中所做的工作，你可能需要建立一个继承自 `Application` 的类，将你的初始化代码放到那个类重写的 `onCreate` 方法里面。

```java
public class MyApplicationClass extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        // TODO 把初始化代码放在这里
    }
}
```

`Application` 类的 `onCreate` 方法只有在整个应用程序被创建的时候才会调用，所以当 `Activity` 在旋转屏幕时重新启动或者键盘可见性发生改变的时候都不会再次调用。

将这个类的实例设为单例模式并通过 getter 和 setter 方法访问应用程序变量是一种好习惯。

*注意：你需要在 manifest 里指定你的 `Application` 类的名字从而将其注册并使用：*

```xml
<application
    android:name="com.you.yourapp.MyApplicationClass"
```

**响应配置变化** （更新：从 API 13 开始以下方法废弃，[请参阅另外一种方法](http://www.androiddesignpatterns.com/2013/04/retaining-objects-across-config-changes.html)）

另一种方案是，你可以让应用监听是由什么引起了重新启动，例如屏幕旋转或者键盘可见性发生改变，并且在 `Activity` 中做出处理。

首先，在你的 `Activity` 的 manifest 节点中加上 `android:configChanges`：

```xml
android:configChanges="keyboardHidden|orientation"
```

或者在 Android 3.2（API 等级 13）以及更新的版本中：

```xml
android:configChanges="keyboardHidden|orientation|screenSize"
```

然后在 `Activity` 里面重写 `onConfigurationChanged` 方法，并调用 `setContentView` 来强制在新的屏幕方向中生成 GUI 布局。

```java
@Override
public void onConfigurationChanged(Configuration newConfig) {
  super.onConfigurationChanged(newConfig);
  setContentView(R.layout.myLayout);
}
```
