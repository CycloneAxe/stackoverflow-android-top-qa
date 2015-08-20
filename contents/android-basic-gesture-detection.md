# 基础的手势识别

[问题链接](http://stackoverflow.com/questions/937313/android-basic-gesture-detection)

今天我想试着在我的 Android 应用中实现“抛掷”的手势识别。我看了这些内容：

 - [Detect Gestures - Tutorial](http://www.anddev.org/gesturedetector_and_gesturedetectorongesturelistener-t3204.html)
 - [SDK 文档](http://developer.android.com/reference/android/view/GestureDetector.SimpleOnGestureListener.html)
 - [计算器的代码](http://www.google.co.in/codesearch/p?hl=en#RzGvLIykRFs/src/com/android/calculator2/PanelSwitcher.java&q=android%20package%3agit://android.git.kernel.org%20Calculator2)

至今也没找到能用的东西，希望有人能给我一些提示。

我的界面上有一个 `GridLayout`，里面是 9 个 `ImageView`。代码在这里：[Romain Guy's Grid Layout](http://code.google.com/p/miffed/source/browse/GUI/src/uk/ac/ic/doc/gea05/miffed/wigets/GridLayout.java)。

这个文件是从 Romain Guy 的[照片流程序](http://code.google.com/p/apps-for-android/)里提取出来的，做了一点修改。

为了实现点击功能，我只需要为每一个添加到主 `Activity` 的 `ImageView` 设置一个实现 `View.OnClickListener` 的 `onClickListener`，然而要实现识别抛掷手势的功能，似乎要复杂得多。我猜测原因可能是手势会跨越多个 `View`？

 - 如果我的 `Activity` 实现了 `OnGestureListener`，我不直到如何将它监听 `GridLayout` 或者 `ImageView` 的手势。
   
   ```java
   public class SelectFilterActivity extends Activity implements
       View.OnClickListener, OnGestureListener { ...
   ```
  
 - 如果我的 `Activity` 实现了 `OnTouchListener`，我就没有 `onFling` 方法可以重写（它的参数中有两个事件让我决定这个抛掷手势是否值得注意）。
   
   ```java
   public class SelectFilterActivity extends Activity implements
       View.OnClickListener, OnTouchListener { ...
   ```
  
 - 如果我建立一个自定义的 `View`，例如 `GestureImageView` 并且继承自 `ImageView`，我不知道如何从这个 `View` 告诉 `Activity` 发生了抛掷手势。不论我怎么尝试触摸屏幕，这些方法都不会被调用。

我需要一个在多个 View 之间识别手势的具体例子。我应该什么时候、用什么方式添加一个什么样的 listener？我同时还需要能够检测点击事件。

```java
// 手势识别
mGestureDetector = new GestureDetector(this, new GestureDetector.SimpleOnGestureListener() {

    public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
        int dx = (int) (e2.getX() - e1.getX());
        // 如果动作很短就忽略
        // 因为可能会和按下按钮有冲突
        if (Math.abs(dx) > MAJOR_MOVE && Math.abs(velocityX) > Math.absvelocityY)) {
            if (velocityX > 0) {
                moveRight();
            } else {
                moveLeft();
            }
            return true;
        } else {
            return false;
        }
    }
});
```

是否能够在屏幕顶层添加一个透明的 `View` 来监听手势？如果我不去填充 XML 中的 `ImageView`，我能把 `GestureDetector` 作为一个构造方法的参数传递给我创建的 `ImageView` 子类吗？

这是我想实现手势识别的一个非常简单的 Activity：[SelectFilterActivity（修改自照片流）](http://code.google.com/p/miffed/source/browse/GUI/src/uk/ac/ic/doc/gea05/miffed/gui/SelectFilterActivity.java?r=210)。

欢迎提出任何建议。如果我的问题看起来很乱，非常抱歉，可以向我询问详细的解释，我也会告诉你我尝试过的细节。

## [gav 的答案](http://stackoverflow.com/a/938657/5152089)

感谢 [Code Shogun](http://www.codeshogun.com/blog/2009/04/16/how-to-implement-swipe-action-in-android/)，提供的代码我修改了一下就可以用了。

让你的 `Activity` 像往常一样实现 `OnClickListener`：

```java
public class SelectFilterActivity extends Activity implements OnClickListener {

    private static final int SWIPE_MIN_DISTANCE = 120;
    private static final int SWIPE_MAX_OFF_PATH = 250;
    private static final int SWIPE_THRESHOLD_VELOCITY = 200;
    private GestureDetector gestureDetector;
    View.OnTouchListener gestureListener;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        /* ... */

        // Gesture detection
        gestureDetector = new GestureDetector(this, new MyGestureDetector());
        gestureListener = new View.OnTouchListener() {
            public boolean onTouch(View v, MotionEvent event) {
                return gestureDetector.onTouchEvent(event);
            }
        };

    }

    class MyGestureDetector extends SimpleOnGestureListener {
        @Override
        public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
            try {
                if (Math.abs(e1.getY() - e2.getY()) > SWIPE_MAX_OFF_PATH)
                    return false;
                // right to left swipe
                if(e1.getX() - e2.getX() > SWIPE_MIN_DISTANCE && Math.abs(velocityX) > SWIPE_THRESHOLD_VELOCITY) {
                    Toast.makeText(SelectFilterActivity.this, "Left Swipe", Toast.LENGTH_SHORT).show();
                }  else if (e2.getX() - e1.getX() > SWIPE_MIN_DISTANCE && Math.abs(velocityX) > SWIPE_THRESHOLD_VELOCITY) {
                    Toast.makeText(SelectFilterActivity.this, "Right Swipe", Toast.LENGTH_SHORT).show();
                }
            } catch (Exception e) {
                // nothing
            }
            return false;
        }

            @Override
        public boolean onDown(MotionEvent e) {
              return true;
        }
    }
}
```

让你的手势 listener 监听所有添加到布局上的 `View`：

```java
// 为每一个添加到 GridLayout 的 ImageView 都这样做
imageView.setOnClickListener(SelectFilterActivity.this); 
imageView.setOnTouchListener(gestureListener);
```

你会惊讶地发现你重写的方法，包括 `Activity` 的 `onClick` 和手势 listener 的 `onFling` 都会被触发。

```java
public void onClick(View v) {
    Filter f = (Filter) v.getTag();
    FilterFullscreenActivity.show(this, input, f);
}
```

之后的“抛掷”动作是可选的，不过推荐看一下。

希望能有所帮助！

Gav
