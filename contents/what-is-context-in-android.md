# Context 是什么？

[问题链接](http://stackoverflow.com/questions/3572463/what-is-context-in-android)

在 Android 编程中，`Context` 类到底是什么，是用来做什么的？我在开发者网站上读了有关它的内容，但无法清楚地理解。

## [Sameer Segal 的答案](http://stackoverflow.com/a/3572553/5152089)

简单来说：

正如它名字的含义，它是应用程序/对象运行的上下文。它可以让新创建的对象知道之前发生了什么。通常你会调用它来获取有关程序另一部分（`Activity`、包或者应用程序）的信息。

你可以通过调用 `getApplicationContext()`、`getContext()`、`getBaseContext()` 或者 `this`（在 `Activity` 类中）来获得 `Context`。

`Context` 的典型用例：

 - **创建新对象**：创建 `View`、`Adapter`、listener 等：
   
   ```java
   TextView tv = new TextView(getContext());
   ListAdapter adapter = new SimpleCursorAdapter(getApplicationContext(), ...);
   ```
 
 - **访问标准的常用资源**：例如 `LAYOUT_INFLATER_SERVICE` 的服务、`SharedPreferences`：
   
   ```java
   context.getSystemService(LAYOUT_INFLATER_SERVICE);
   getApplicationContext().getSharedPreferences(*name*, *mode*);
   ```
 
 - **隐式访问组件**：相关的内容提供者、广播、`Intent`：
   
   ```java
   getApplicationContext().getContentResolver().query(uri, ...);
   ```
