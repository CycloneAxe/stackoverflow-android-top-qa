# Android 中 px、dp、dip 和 sp 的区别？

[问题链接](http://stackoverflow.com/questions/2025282/difference-between-px-dp-dip-and-sp-in-android)

Android 中 `px`、`dip`、`dp` 和 `sp` 四种单位有什么区别？

- - -

[答案链接](http://stackoverflow.com/questions/2025282/difference-between-px-dp-dip-and-sp-in-android/2025541#2025541)

 - **px** 是一个像素；
 - **sp** 是尺度独立（scale-independent）像素；
 - **dip** 是密度独立（density-independent）像素。

| 密度    | 屏幕显示 | 物理尺寸     | 像素尺寸                      |
|:------- |:-------- |:------------ |:----------------------------- |
| ldpi    | 120 dpi  | 0.5 x 0.5 in | 0.5 in * 120 dpi = 60x60 px   | 
| mdpi    | 160 dpi  | 0.5 x 0.5 in | 0.5 in * 160 dpi = 80x80 px   | 
| hdpi    | 240 dpi  | 0.5 x 0.5 in | 0.5 in * 240 dpi = 120x120 px | 
| xhdpi   | 320 dpi  | 0.5 x 0.5 in | 0.5 in * 320 dpi = 160x160 px |
| xxhdpi  | 480 dpi  | 0.5 x 0.5 in | 0.5 in * 480 dpi = 240x240 px |
| xxxhdpi | 640 dpi  | 0.5 x 0.5 in | 0.5 in * 640 dpi = 320x320 px |

你应该在：

 - 文字尺寸上使用 `sp`；
 - 其他地方使用 `dip`。`dip` 和 `dp` 相同。

摘自 [Android Developers](http://developer.android.com/guide/topics/resources/more-resources.html#Dimension)：

> | 单位 | 描述         | 每物理英寸的单位数 | 密度独立 | 在所有屏幕上物理尺寸一致 |
> |:---- |:------------ |:------------------ |:-------- |:------------------------ |
> | px   | 像素         | 可变               | 否       | 否                       |
> | in   | 英寸         | 1                  | 是       | 是                       |
> | mm   | 毫米         | 24.5               | 是       | 是                       |
> | pt   | 点           | 72                 | 是       | 是                       |
> | dp   | 密度独立像素 | ~ 160              | 是       | 否                       |
> | sp   | 尺度独立像素 | ~ 160              | 是       | 否                       |
> 
> **px**
> 
> 像素：与屏幕实际像素一致。
> 
> **in**
>
> 英寸：与屏幕物理尺寸相关。
>
> 1 英寸 = 2.54 厘米。
>
> **mm**
>
> 毫米：与屏幕物理尺寸相关。
>
> **pt**
>
> 点：与屏幕物理尺寸相关，一英寸的 1/72。
>
> **dp**
>
> 密度独立像素：与屏幕物理密度相关的抽象单位。这些单位都是相对于 160 dpi 的屏幕确定，因此 1 dp 相当于 160 dpi 屏幕上的一个像素。dp 与像素的比例随着屏幕密度而改变，但不一定成正比例。注意：尽管 dp 和 sp 看起来更一致，dip 和 dp 编译器都可接受。
>
> **sp**
>
> 尺度独立像素：与 dp 相似，但可以根据用户的字号设置进行缩放。推荐你在文字尺寸上使用这个单位，从而让文字可以同时根据屏幕尺寸和用户的选项来进行调整。
