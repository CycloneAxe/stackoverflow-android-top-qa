# match_parent 和 fill_parent 的区别是什么？

[问题链接](http://stackoverflow.com/questions/5761960/what-is-the-difference-between-match-parent-and-fill-parent)

我弄不太清楚这两个 XML 属性：`match_parent` 和 `fill_parent`，看起来好像是一样的。它们之间有什么区别吗？

## [Matt Ball 的答案](http://stackoverflow.com/a/5761970/5152089)

它们是一样的（API 等级 8 以上）。用 `match_parent` 吧。

> `FILL_PARENT`（在 API 等级 8 及以上改名为 `MATCH_PARENT`），意味着一个 `View` 期望和父元素一样大（除去内边距）。
>
> ……
> 
> `fill_parent`：`View` 应该和父元素一样大（除去内边距）。这个常量从 API 等级 8 开始被废弃了，改为 `match_parent`。

http://developer.android.com/reference/android/view/ViewGroup.LayoutParams.html

## [Tapirboy 的答案](http://stackoverflow.com/a/13821387/5152089)

为了避免误解，Google 改掉了它的名字。

以前的名字 `fill_parent` 隐含了一种影响父元素尺寸的意思，而 `match_parent` 表达得更准确——与父元素的尺寸相符。

这两个常量最终的值都是 `-1`，所以应用会得到相同的行为。讽刺的是，这次为了澄清事情的改名并没能消除误会，反而造成了更多的误会。
