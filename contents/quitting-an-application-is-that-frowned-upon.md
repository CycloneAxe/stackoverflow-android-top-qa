# 退出应用是否不妥？

[问题链接](http://stackoverflow.com/questions/2033914/quitting-an-application-is-that-frowned-upon)

在我学习 Android 的时候，我发现了[这样的问答](http://groups.google.com/group/android-developers/browse_thread/thread/1bf0f7a4a9c62edd/a0a4aedf21ae5f76)：

> 问：除非我们在菜单里放一个退出的选项，用户还有其他方法退出应用吗？如果没有这样的选项，用户该怎么结束一个应用呢？
>
> 答（Romain Guy）：用户不能退出，而是系统自动管理这些事情。这就是 `Activity` 生命周期（尤其是 `onPause`/`onStop`/`onDestroy`）的作用。不管你做的是什么，不要放一个“退出”按钮，这对于 Android 的应用模型是没有作用的，与核心应用程序的工作方式也是背道而驰。

呵呵，我在 Android 世界中的每一步探索都会遇到各种类型的问题 =(。

很明显，你在 Android 中无法退出一个应用（但是 Android 系统可以在它喜欢的时候灭掉你的应用）。这是怎么回事？我开始觉得无法写出一个“正常的应用”——用户可以随时退出的应用。这可不是需要依赖系统来做的事情。

我要开发的并不是投放到 Android 市场中的应用。它并不是给大众广泛使用的，而将是在某一个狭窄的商业领域内使用。

我真的非常希望为 Android 平台开发应用，这个平台解决了 Windows Mobile 和 .NET 中存在的许多问题。不过，上一周却让我的观点产生了分歧。我希望我不会抛弃 Android，但现在它看起来没有那么好了 =(。

有没有办法可以**真正地**退出应用呢？

## [CommonsWare 的答案](http://stackoverflow.com/a/2034238/5152089)

我会在之后回答你的问题，但写这篇答案时看到你在各种回答里留下的各种评论，我决定一开始先回答一下你提出的这些问题。我无意改变你的思维——相反，这是给之后读到这些文字的人而写的。

> 关键是我不能允许 Android 来决定我的应用什么时候应该被终止。这应该是由用户决定的。（这是提问者在其他答案中留下的评论，下同。——译者注）

对于系统在程序应当结束的时候关闭这些应用的模型，成千上万的人都为之叫好。这些用户所想的并不是要“终止”应用，更不会想要“终止”一个网页或者“终止”一个温度计。

iPhone 用户也一样，按下 iPhone 的按键并不意味着应用被终止，因为许多 iPhone 应用能够在用户离开的地方继续运行，即使当应用被关掉的时候（iPhone 目前只允许一个第三方应用运行）也一样。

> 就像我上面所说的，我的应用里会发生很多事情（数据被推送到设备，任务列表应该一直在那里工作）。

我不明白“任务列表应该一直在那里工作”是什么意思，但是“数据被推送到设备”是一种令人愉悦的假设，这不应该由 `Activity` 完成，而应该用预订计划（通过 `AlarmManager`）更新数据从而获得最大的可靠性。

> 我们的用户登录进来，总不能每一次有电话打进来，Android 就把应用杀掉，然后用户就需要重新登录。

有许多 iPhone 和 Android 应用都把这一点处理得很好。通常是因为他们使用了登录凭证，而不是每一次都要求用户手动登录。

> 举个例子，我们想在退出应用的时候检查更新。

这放在任何操作系统上都是错误的做法。你要直到，你的应用程序“退出”的原因是系统正在关闭，那么你的更新进程就会中途失败。通常这不是什么好事情。要么在启动时检查更新，要么完全异步地检查（例如通过预订任务），永远不要在退出时检查更新。

> 有一些评论说按下返回键并不会杀掉应用（看我上面问题里的链接）。

按下返回键不会“杀掉应用”，而是结束当前屏幕上的 `Activity`。

> 只有在用户想退出的时候才能退出——其他方式都不可以。如果你写的应用会像 Android 那样运行，那我觉得 Android 不能用来编写真正的应用 =(。

Web 应用也不能。还有 WebOS，如果我没把它的模型理解错的话（我还没有机会玩一玩）。在所有这些系统里，用户都不会“终结”任何东西——他们只是离开。iPhone 有一点不同，它目前一次只允许一个应用运行（有一些特例），所以离开应用意味着结束应用。

> 有办法让我真正地退出应用吗？

就像其他所有人告诉你的，用户（按下返回键）或者通过代码（通过 `finish()`）都可以关掉当前正在运行的 `Activity`。对于质量高的应用，用户通常不需要其他操作，就像在 Web 应用里不需要“退出”选项一样。

- - -

按照定义，没有相同的应用环境。这意味着你可以看到环境发展的趋势，新的环境会崛起，旧的环境会衰落。

例如，有一种要消灭“文件”观念的趋势。大多数 Web 应用都不会要求用户思考关于文件的事情。iPhone 应用通常也不让用户思考文件。Android 应用通常也不让用户思考文件的事情，等等。

相似地，还有一种趋势试图消灭“终结”应用的趋势。大多数 Web 应用都不会要求用户注销账户，而是在一段时间的不活动后隐式地退出用户的账户。对于 Android 也一样，iPhone（WebOS 也有可能）在一定程度上也是这样。

这对于应用的设计、专注于商业目标以及不要坚守之前的应用环境的实现方式有着更高的要求。没有时间和意愿的开发者会因为新的环境打破了他们现有的思维模型而感到受挫，然而这并不是环境的错，就像狂风绕着山峰流动而不是穿过它并不是山的错一样。

例如，有一些开发环境，比如 Hypercard 和 Smalltalk，会把应用程序和开发工具一股脑地都安装进来。这种观念只有在一些应用的语言插件中得以使用（例如 Excel 中的 VBA、AutoCAD 中的 LISP），在其他地方并没有流行起来。如果开发者认为开发工具存在于应用本身，那么他们要么改变自己的思维模型，要么只能为满足他们想法的环境工作。

所以，当你写下：

> 随着我遇到的其他乱七八糟的问题，我觉得我们不会为 Android 开发应用。

这似乎对你来说是目前最好的选择。相似地，我要忠告你不要把你的应用移植到 Web 端，因为你在 Android 中遇到的问题在 Web 应用中同样存在（例如没有“终止”）。或者正相反，有一天如果你真的把应用移植到了 Web 端，你会发现 Web 应用的流程可能会与 Android 更加贴切，那时候你可以再考虑一下移植到 Android。

## [Neil Traft 的答案](http://stackoverflow.com/a/2632649/5152089)

我要为将来的读者做一点纠错。我一直都没有理解这一点点差别，所以我希望将来你们不要犯相同的错误：

**如果你在栈中有不止一个 `Activity`，`System.exit()` 并不会杀死你的应用。**真正发生的事情是**进程被杀死，然后马上重新启动**，栈中的 `Activity` 会减少一个。当你的应用被强制结束的对话框杀死时，或者当你从 DDMS 中杀死进程时，情况都是一样的。据我所知，这件事完全没有写入文档。

简单来说，如果你想退出你的应用，你需要记录下栈中的所有 `Activity` 并且把它们都 `finish()` 掉（并没有遍历 `Activity` 栈的方法，因此你需要亲自完成）。就算这样做，你也无法真正杀掉进程以及你所拥有的那些引用，而只是结束掉了这些 `Activity`。另外，我不确定 `Process.killProcess(Process.myPid())` 是否会好一点，我还没试过。

如果栈里面还留下几个 `Activity` 对你来说没有关系，还有一种超级简单的方法：`Activity.moveTaskToBack(true)` 可以简单地让进程隐藏到后台，显示主屏幕。

详细叙述的话，我会对这种行为背后的哲学做一些解释。这种哲学是建立在一些假设之上的：

 1. 首先，这只对你的应用在前台的情况有效。如果是在后台，那么结束掉进程没什么大问题。但是如果是在前台的话，操作系统假设用户会一直继续他/她正在做的事情（如果你尝试从 DDMS 中结束进程，你需要先点击主页键，然后再杀掉它）；
 2. 每一个 `Activity` 与其他的 `Activity` 都互相独立。这通常是对的，例如如果你的应用启动了浏览器 `Activity`，这是一个完全独立的 `Activity`，而且不是你写出来的。浏览器的 `Activity` 可能会也可能不会建立同样的 `Task`，这要看它的 manifest 属性是什么样的了；
 3. 每一个 `Activity` 都是自主的，可以在片刻间被结束和还原（我非常讨厌这一条假设，因为我的应用有很多 `Activity` 需要依赖大量的缓存数据运行，在 `onSaveInstanceState` 的时候序列化的性能很烂，但是又能怎么样呢）。对于大多数高质量的 Android 应用这一点是成立的，因为你永远不知道在后台的应用会在什么时候被结束掉。
 4. 最后的一个因素并不算是一个假设，而是 OS 的限制：*显式地杀掉一个应用和应用崩溃是相同的，Android 结束应用从而释放内存也是一样*。这给了我们致命一击：既然 Android 都不知道应用是退出了、崩溃了还是在后台被结束了，它假设用户还想回到之前离开的地方继续使用，因此 `ActivityManager` 会重启进程。

这对于平台是合适的举措。首先，当进程在后台被结束掉，然后用户返回到这个进程时就应该是这个效果，因此进程需要被重新启动；此外，应用崩溃并且显示了可怕的强制关闭对话框时，也应当这么做。

例如我想让用户拍一张照片并上传。我从我的 `Activity` 启动了相机的 `Activity`，然后要求它返回一张图片。相机会被推到我当前的 `Task` 的顶端（而不是建立一个它自己的新的 `Task`）（这一点随 `launchMode` 以及系统版本的不同而有所变化，[延伸阅读](http://droidyue.com/blog/2015/08/16/dive-into-android-activity-launchmode/)。——译者注）。如果相机出现了错误崩溃了，难道应该造成整个应用都崩溃吗？从用户的角度来看，只有相机崩溃了，应该可以返回到之前的 `Activity`。因此系统会重启进程，并且保持栈中的 `Activity` 除了相机之外都与之前相同。因为你应当把 `Activity` 设计为可以在瞬间内完成结束和重新载入的操作，这并不是什么大问题。不幸的是，并不是所有应用都有这样的设计，所以不论 Romain Guy 或者其他人是怎么说的,这对我们大多数人来说都仍然是个问题。因此我们需要一些应变方法。

最后，我的建议是：

 - 不要试图结束进程。要么对所有 `Activity` 调用 `finish()`，要么用 `moveTaskToBack(true)`；
 - 如果你的进程崩溃了或者被结束了，或者像我一样，你需要的数据丢失了，你需要返回根 `Activity`。要想这么做，你应该调用 `startActivity()`，并且带上一个包含 `Intent.FLAG_ACTIVITY_CLEAR_TOP` 标记的 `Intent`。
 - 如果你想从 Eclipse DDMS 中结束应用，它最好没有在前台运行，否则它就会重启。你应该先按下主页键，然后再结束进程。
