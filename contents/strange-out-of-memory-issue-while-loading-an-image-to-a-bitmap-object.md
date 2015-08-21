# 向 Bitmap 对象加载图片时出现奇怪的内存不足错误

[问题链接](http://stackoverflow.com/questions/477572/strange-out-of-memory-issue-while-loading-an-image-to-a-bitmap-object)

我有一个 `ListView`，每一行都放了一个 `ImageButton`，点击一行的时候就会启动一个新的 `Activity`。因为相机的布局出现了问题，我必须自己实现 tab 布局。启动的 `Activity` 是一个地图。如果我点击按钮打开图片预览（从 SD 卡中读取一张图片），应用程序会返回到包含 `ListView` 的 `Activity`，并且重新启动一个新 `Activity`，里面只有一个图片控件。

`ListView` 里的图片预览是通过光标和 `ListAdapter` 实现的，非常简单，但我不知道如何显示一张调整尺寸后的图片（例如在运行时略微缩小一点），因此我就把手机相机拍摄下来的图片调整了尺寸。

问题是，当我返回并重新运行第二个 `Activity` 时，遇到了一个内存不足的错误。

有没有什么办法可以轻松地一行行地搭建列表的 adapter，同时我还可以动态地更改图片尺寸？因为焦点的问题我没办法用触屏选中一行（我可以用轨迹球），所以我还需要对每一行控件/元素的属性做一些更改，我觉得这样会更好一些。

我知道我可以做一些附加的调整大小的操作并且把图片保存起来，但是这并不是我想实现的方式。不过有这方面的示例代码的话也不错。

只要我把 `ListView` 里的图片禁用掉，就没有问题了。

这是我的代码，仅供参考：

```java
String[] from = new String[] { DBHelper.KEY_BUSINESSNAME,
                               DBHelper.KEY_ADDRESS,
                               DBHelper.KEY_CITY,
                               DBHelper.KEY_GPSLONG,
                               DBHelper.KEY_GPSLAT,
                               DBHelper.KEY_IMAGEFILENAME  + ""};
int[] to = new int[] { R.id.businessname,
                       R.id.address,
                       R.id.city,
                       R.id.gpslong,
                       R.id.gpslat,
                       R.id.imagefilename };
notes = new SimpleCursorAdapter(this, R.layout.notes_row, c, from, to);
setListAdapter(notes);
```

`R.id.imagefilename` 是一个 `ImageButton`。

以及我的 LogCat：

```
01-25 05:05:49.877: ERROR/dalvikvm-heap(3896): 6291456-byte external allocation too large for this process.
01-25 05:05:49.877: ERROR/(3896): VM wont let us allocate 6291456 bytes
01-25 05:05:49.877: ERROR/AndroidRuntime(3896): Uncaught handler: thread main exiting due to uncaught exception
01-25 05:05:49.917: ERROR/AndroidRuntime(3896): java.lang.OutOfMemoryError: bitmap size exceeds VM budget
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.graphics.BitmapFactory.nativeDecodeStream(Native Method)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.graphics.BitmapFactory.decodeStream(BitmapFactory.java:304)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.graphics.BitmapFactory.decodeFile(BitmapFactory.java:149)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.graphics.BitmapFactory.decodeFile(BitmapFactory.java:174)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.graphics.drawable.Drawable.createFromPath(Drawable.java:729)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.widget.ImageView.resolveUri(ImageView.java:484)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.widget.ImageView.setImageURI(ImageView.java:281)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.widget.SimpleCursorAdapter.setViewImage(SimpleCursorAdapter.java:183)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.widget.SimpleCursorAdapter.bindView(SimpleCursorAdapter.java:129)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.widget.CursorAdapter.getView(CursorAdapter.java:150)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.widget.AbsListView.obtainView(AbsListView.java:1057)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.widget.ListView.makeAndAddView(ListView.java:1616)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.widget.ListView.fillSpecific(ListView.java:1177)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.widget.ListView.layoutChildren(ListView.java:1454)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.widget.AbsListView.onLayout(AbsListView.java:937)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.view.View.layout(View.java:5611)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.widget.LinearLayout.setChildFrame(LinearLayout.java:1119)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.widget.LinearLayout.layoutHorizontal(LinearLayout.java:1108)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.widget.LinearLayout.onLayout(LinearLayout.java:922)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.view.View.layout(View.java:5611)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.widget.FrameLayout.onLayout(FrameLayout.java:294)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.view.View.layout(View.java:5611)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.widget.LinearLayout.setChildFrame(LinearLayout.java:1119)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.widget.LinearLayout.layoutVertical(LinearLayout.java:999)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.widget.LinearLayout.onLayout(LinearLayout.java:920)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.view.View.layout(View.java:5611)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.widget.FrameLayout.onLayout(FrameLayout.java:294)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.view.View.layout(View.java:5611)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.view.ViewRoot.performTraversals(ViewRoot.java:771)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.view.ViewRoot.handleMessage(ViewRoot.java:1103)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.os.Handler.dispatchMessage(Handler.java:88)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.os.Looper.loop(Looper.java:123)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at android.app.ActivityThread.main(ActivityThread.java:3742)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at java.lang.reflect.Method.invokeNative(Native Method)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at java.lang.reflect.Method.invoke(Method.java:515)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:739)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:497)
01-25 05:05:49.917: ERROR/AndroidRuntime(3896):     at dalvik.system.NativeStart.main(Native Method)
01-25 05:10:01.127: ERROR/AndroidRuntime(3943): ERROR: thread attach failed
```

在显示图像的时候还出了一个新的错误：

```
01-25 22:13:18.594: DEBUG/skia(4204): xxxxxxxxxxx jpeg error 20 Improper call to JPEG library in state %d
01-25 22:13:18.604: INFO/System.out(4204): resolveUri failed on bad bitmap uri: 
01-25 22:13:18.694: ERROR/dalvikvm-heap(4204): 6291456-byte external allocation too large for this process.
01-25 22:13:18.694: ERROR/(4204): VM won't let us allocate 6291456 bytes
01-25 22:13:18.694: DEBUG/skia(4204): xxxxxxxxxxxxxxxxxxxx allocPixelRef failed
```

## [Fedor 的答案](http://stackoverflow.com/a/823966/5152089)

要解决 `OutOfMemory` 错误，你应该这样做：

```java
BitmapFactory.Options options = new BitmapFactory.Options();
options.inSampleSize = 8;
Bitmap preview_bitmap = BitmapFactory.decodeStream(is, null, options);
```

`inSampleSize` 这个选项可以减小内存消耗。

下面是完整的方法。首先它在不解码内容的情况下读出图片的尺寸，然后找到最佳的 `inSampleSize` 值，应该是 2 的幂。最后图片才会被加载。

```java
// 解码图片并调整尺寸从而减小内存消耗
private Bitmap decodeFile(File f) {
    try {
        // 解析图片尺寸
        BitmapFactory.Options o = new BitmapFactory.Options();
        o.inJustDecodeBounds = true;
        BitmapFactory.decodeStream(new FileInputStream(f), null, o);

        // 想要缩放的新尺寸
        final int REQUIRED_SIZE=70;

        // 找到正确的缩放值，应该是 2 的幂
        int scale = 1;
        while(o.outWidth / scale / 2 >= REQUIRED_SIZE && 
              o.outHeight / scale / 2 >= REQUIRED_SIZE) {
            scale *= 2;
        }

        // 使用 inSampleSize 来解码
        BitmapFactory.Options o2 = new BitmapFactory.Options();
        o2.inSampleSize = scale;
        return BitmapFactory.decodeStream(new FileInputStream(f), null, o2);
    } catch (FileNotFoundException e) {}
    return null;
}
```

# [Thomas Vervest 的答案](http://stackoverflow.com/a/3549021/5152089)

我对 Fedor 的代码做了一点改进。基本做法是一样的，但是去掉了（我认为）很丑陋的、结果是 2 的幂的 `while` 循环。感谢 Fedor 提出了最初的解决方法，知道发现他的答案之前我一直伸手困扰，而自此之后我也得以写出这个版本:)。

```java
private Bitmap decodeFile(File f){
    Bitmap b = null;

    // 解析图片尺寸
    BitmapFactory.Options o = new BitmapFactory.Options();
    o.inJustDecodeBounds = true;

    FileInputStream fis = new FileInputStream(f);
    BitmapFactory.decodeStream(fis, null, o);
    fis.close();

    int scale = 1;
    if (o.outHeight > IMAGE_MAX_SIZE || o.outWidth > IMAGE_MAX_SIZE) {
        scale = (int)Math.pow(2, (int) Math.ceil(Math.log(IMAGE_MAX_SIZE / 
           (double) Math.max(o.outHeight, o.outWidth)) / Math.log(0.5)));
    }

    // 使用 inSampleSize 来解码
    BitmapFactory.Options o2 = new BitmapFactory.Options();
    o2.inSampleSize = scale;
    fis = new FileInputStream(f);
    b = BitmapFactory.decodeStream(fis, null, o2);
    fis.close();

    return b;
}
```

## [AdamK 的答案](http://stackoverflow.com/a/10127787/5152089)

[Android Training](http://developer.android.com/training/index.html) 的 [Displaying Bitmaps Efficiently](http://developer.android.com/training/displaying-bitmaps/index.html) 提供了一些很不错的资料，有助于理解和解决加载图片时出现的 `java.lang.OutOfMemoryError: bitmap size exceeds VM budget when loading Bitmaps` 这一错误。
