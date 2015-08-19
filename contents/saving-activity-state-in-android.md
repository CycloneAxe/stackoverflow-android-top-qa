# 保存 Activity 状态

我最近在玩 Android SDK，在保存程序的状态上有一点困惑。以这个“Hello, Android”的修改版为例：

```java
package com.android.hello;

import android.app.Activity;
import android.os.Bundle;
import android.widget.TextView;

public class HelloAndroid extends Activity {
    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mTextView = new TextView(this);

        if (savedInstanceState == null) {
            mTextView.setText("Welcome to HelloAndroid!");
        } else {
            mTextView.setText("Welcome back.");
        }

        setContentView(mTextView);
    }

    private TextView mTextView = null;
}
```

我以为像上面这样做就完成了，但是不论我怎么离开 app，它总是只显示第一条消息。我确定需要重载 onPause 或者其他的什么方法，但是我翻了 30 分钟文档，也没有找到什么显而易见的解决方法。请大家帮帮我。

## [Reto Meier 的答案](http://stackoverflow.com/questions/151777/saving-activity-state-in-android/151940#151940)

你需要重载 `onSaveInstanceState(Bundle savedInstanceState)` 并且把希望变更的程序状态写入到 `Bundle` 参数里，就像这样：

```java
@Override
public void onSaveInstanceState(Bundle savedInstanceState) {
  super.onSaveInstanceState(savedInstanceState);
  // 把 UI 状态变化保存到 savedInstanceState
  // 当程序被杀死并重新运行时，bundle 会传递到 onCreate 方法
  savedInstanceState.putBoolean("MyBoolean", true);
  savedInstanceState.putDouble("myDouble", 1.9);
  savedInstanceState.putInt("MyInt", 1);
  savedInstanceState.putString("MyString", "Welcome back to Android");
  // etc.
}
```

这个 Bundle 本质上是一种保存 NVP（"Name-Value Pair"，键值对）映射的方法，它会被传递到 `onCreate()` 和 `onRestoreInstanceState()`，你需要从中再次取出值：

```java
@Override
public void onRestoreInstanceState(Bundle savedInstanceState) {
  super.onRestoreInstanceState(savedInstanceState);
  // 从 savedInstanceState 中恢复 UI 状态
  // 这个 bundle 也会被传递到 onCreate
  boolean myBoolean = savedInstanceState.getBoolean("MyBoolean");
  double myDouble = savedInstanceState.getDouble("myDouble");
  int myInt = savedInstanceState.getInt("MyInt");
  String myString = savedInstanceState.getString("MyString");
}
```

通常就是用这种方法为你的应用程序保存实例的值（选项、未保存的文本，等等）。

## [Dave L. 的答案](http://stackoverflow.com/questions/151777/saving-activity-state-in-android/151822#151822)

`savedInstanceState` 只是用于保存与当前 Activity 实例有关的状态，例如当前的导航或者选择，当 Android 销毁并重建这个 Activity 时，可以恢复到之前的样子。参考 [`onCreate`](http://developer.android.com/reference/android/app/Activity.html#onCreate%28android.os.Bundle%29) 和 [`onSavedInstanceState`](http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState%28android.os.Bundle%29) 的文档。

对于长期存在的状态，考虑使用 SQLite 数据库、文件或者选项。参考 [Saving Persistent State](http://developer.android.com/reference/android/app/Activity.html#SavingPersistentState)。

## [Steve Moseley 的答案](http://stackoverflow.com/questions/151777/saving-activity-state-in-android/2909211#2909211)

注意，根据 [Activity 状态的文档](http://developer.android.com/reference/android/app/Activity.html)，使用 `onSavedInstanceState` 和 `onRestoreInstanceState` 来存储**持久化数据**是***不***安全的。

文档中声明（在“Activity Lifecycler”一节）：

> 注意，持久化数据要在 `onPause()` 中保存，而不能在 `onSaveInstanceState(Bundle)` 中保存，因为后者并不是生命周期的回调方法，所以并不会在文档中描述的每一个情形下都被调用。

换句话说，对持久化数据的保存/还原操作，要放在 `onPause()` 和 `onResume()` 中！

**编辑：**为了进一步说明，附上 `onSavedInstanceState()` 的文档：

> 这个方法会在 activity 将要被杀死之前调用，从而在将来回到 activity 的时候恢复之前的状态。例如，如果 activity B 在 activity A 之上启动，某一时刻 activity A 被杀死从而释放资源，那么 activity A 会有机会通过这个方法保存当前的用户界面状态从而在用户返回 activity A 的时候，用户界面的状态可以通过 `onCreate(Bundle)` 或者 `onRestoreInstanceState(Bundle)` 来恢复。

## [Martin Belcher - Eigo 的答案](http://stackoverflow.com/questions/151777/saving-activity-state-in-android/3584836#3584836)

我的同事写了一篇解释 Android 设备应用程序状态的文章，里面包括对 Activity 的生命周期和状态信息、如何存储状态信息以及保存到状态 `Bundle` 和 `SharePreferences` 的解释。可以[到这里看一看](http://www.eigo.co.uk/Managing-State-in-an-Android-Activity.aspx)。

文章里讲了三种方法：

#### 使用实例状态 Bundle 在应用程序运行期间（临时地）存储本地变量、UI 控件数据

```java
[示例代码：在状态 Bundle 中存储状态]
@Override
public void onSaveInstanceState(Bundle savedInstanceState) 
{
  // 将 UI 状态保存至 savedInstanceState
  // 这个 Bundle 会在下一次调用 onCreate 时传递过去。
  EditText txtName = (EditText)findViewById(R.id.txtName);
  String strName = txtName.getText().toString();

  EditText txtEmail = (EditText)findViewById(R.id.txtEmail);
  String strEmail = txtEmail.getText().toString();

  CheckBox chkTandC = (CheckBox)findViewById(R.id.chkTandC);
  boolean blnTandC = chkTandC.isChecked();

  savedInstanceState.putString(“Name”, strName);
  savedInstanceState.putString(“Email”, strEmail);
  savedInstanceState.putBoolean(“TandC”, blnTandC);

  super.onSaveInstanceState(savedInstanceState);
}
```

#### 使用Shared Preferences在应用程序实例之间存储本地变量、UI 控件数据

```java
[示例代码：在 SharedPreferences 中存储状态]
@Override
protected void onPause() 
{
  super.onPause();

  // 将值保存至实例之间
  SharedPreferences preferences = getPreferences(MODE_PRIVATE);
  SharedPreferences.Editor editor = preferences.edit();  // 从 UI 得到值
  EditText txtName = (EditText)findViewById(R.id.txtName);
  String strName = txtName.getText().toString();

  EditText txtEmail = (EditText)findViewById(R.id.txtEmail);
  String strEmail = txtEmail.getText().toString();

  CheckBox chkTandC = (CheckBox)findViewById(R.id.chkTandC);
  boolean blnTandC = chkTandC.isChecked();

  editor.putString(“Name”, strName); // 要存储的值
  editor.putString(“Email”, strEmail); // 要存储的值
  editor.putBoolean(“TandC”, blnTandC); // 要存储的值
  // 保存修改
  editor.commit();
}
```

#### 使用保留的非设置实例在应用程序运行期间将对象实例保留在内存中

```java
[示例代码：存储对象实例]
private cMyClassType moInstanceOfAClass;// 保存对象的实例
@Override
public Object onRetainNonConfigurationInstance() 
{
  if (moInstanceOfAClass != null) // 判断对象是否存在
      return(moInstanceOfAClass);
  return super.onRetainNonConfigurationInstance();
}
```
