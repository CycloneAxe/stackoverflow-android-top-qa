# Eclipse 导入项目之后出现“必须重写父类方法”错误

[问题链接](http://stackoverflow.com/questions/1678122/must-override-a-superclass-method-errors-after-importing-a-project-into-eclips)

每次我重新把项目导入到 Eclipse 之后（比如重新安装了 Eclipse，或者改变了项目的位置），**几乎所有**重写方法的格式都会不正确，导致出现 `The method must override a superclass method`（这个方法必须重写一个父类的方法）这样的错误。

可能值得注意的是，这个问题只存在于 Android 项目，不知道是什么原因。方法的参数有时候不会被填充，我必须手动填上这些参数，比如：

```java
list.setOnCreateContextMenuListener(new OnCreateContextMenuListener() {

    // 参数的名称是正确的
    public void onCreateContextMenu(ContextMenu menu, View v, 
                                    ContextMenuInfo menuInfo) {                 
    }

});
```

会被填充成这样：

```java
list.setOnCreateContextMenuListener(new OnCreateContextMenuListener() {

    // 参数的名称是自动生成的
    public void onCreateContextMenu(ContextMenu arg1, View arg2,
                                    ContextMenuInfo arg3) {
    }

});
```

奇怪的是，如果我删除了代码，让 Eclipse 自动建立方法，生成的参数名会和我之前写的一样，所以我不知道问题出在哪。

重新手动创建所有的重写方法太痛苦了，如果有人可以解释这个问题出现的原因，或者该如何解决，我会很开心的。

也许是因为我写的方法不对，一个方法不该出现在另一个方法的参数里？

## [alphazero 的答案](http://stackoverflow.com/a/1678170/5152089)

Eclipse 默认用的是 Java 1.5，而你的类实现了接口的方法（在 Java 1.6 中可以用 `@Override` 来标记，而在 Java 1.5 中这个标记只能用于重写父类的方法）。

打开项目或者 IDE 的选项，把 Java 编译器版本改成 1.6。同时确保 Eclipse 运行程序的时候使用的也是 JRE 1.6。

Go to your project/ide preferences and set the java compiler level to 1.6 and also make sure you select JRE 1.6 to execute your program from eclipse.
