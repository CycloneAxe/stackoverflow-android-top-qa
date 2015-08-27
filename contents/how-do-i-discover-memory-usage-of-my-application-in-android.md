# 如何获取应用的内存使用情况？

[问题链接](http://stackoverflow.com/questions/2298208/how-do-i-discover-memory-usage-of-my-application-in-android)

如何通过代码获取 Android 应用的内存使用情况？

希望可以有办法实现。另外，怎么获取到手机的空闲内存？

## [hackbod 的答案](http://stackoverflow.com/a/2299813/5262638)

注意，现代操作系统（例如 Linux）的内存使用是非常复杂的，非常难易理解。事实上，你能正确理解获取到的那些数据的可能性微乎其微。（有很多次我与其他的工程师们研究内存使用的数据，都会对它们的意义进行长时间的讨论，然而也只能得到一个含糊不清的结论。）

**注意：现在我们可以在 [Managing Your App's Memory](http://developer.android.com/training/articles/memory.html) 找到更多的文档，包括这里的大多数内容，也与最新版本的 Android 更加符合。**

首先大概需要读一下这篇文章的最后一部分，讨论了 Android 是如何管理内存的：

http://android-developers.blogspot.com/2010/02/service-api-changes-starting-with.html

现在我们有了 `ActivityManager.getMemoryInfo()` 这样的高级 API 来查看总体的内存使用情况，主要用来让应用估计一下还有多久系统会没有足够的内存来运行后台进程，从而需要结束必要的进程（比如服务）。对于纯 Java 程序，这可能没什么用，因为 Java 堆限制大小就是用来防止一个程序占用过多资源的。

而更底层的话，你可以使用 `Debug` API 来获取到关于内存使用的原始内核级信息：[http://developer.android.com/intl/de/reference/android/os/Debug.html#getMemoryInfo(android.os.Debug.MemoryInfo)](http://developer.android.com/intl/de/reference/android/os/Debug.html#getMemoryInfo(android.os.Debug.MemoryInfo))。

注意，从 2.0 版本开始有另一个 API，`ActivityManager.getProcessMemoryInfo`，来获取有关另一个进程的信息：[http://developer.android.com/intl/de/reference/android/app/ActivityManager.html#getProcessMemoryInfo(int[])](http://developer.android.com/intl/de/reference/android/app/ActivityManager.html#getProcessMemoryInfo(int[]))。

这个 API 会返回一个底层的 `MemoryInfo` 结构，其中包含这些数据：

```java
/** Dalvik 的比例集大小 */
public int dalvikPss;
/** Dalvik 使用的私有脏页 */
public int dalvikPrivateDirty;
/** Dalvik 使用的共享脏页 */
public int dalvikSharedDirty;

/** 原生堆的比例集大小 */
public int nativePss;
/** 原生堆使用的私有脏页 */
public int nativePrivateDirty;
/** 原生堆使用的共享脏页 */
public int nativeSharedDirty;

/** 其余部分的比例集大小 */
public int otherPss;
/** 其余部分所使用的私有脏页 */
public int otherPrivateDirty;
/** 其余部分所使用的共享脏页 */
public int otherSharedDirty;
```

但要是说到“PSS”、“private dirty”和“shared dirty”的区别的话……好戏开始了。

Android （以及大体上所有 Linux 系统）的大部分内存实际上都是在多个进程之间共享的，因此一个进程使用了多少内存，真的很难弄清楚。再加上换出到磁盘上的页面（因为 Android 没有交换空间，所以不作考虑），就更难弄明白了。

所以，如果你把映射到每一个进程的物理内存加起来，可能会得到一个比真实的内存使用总量大得多的数字。

PSS（比例集大小）是内核在共享内存的时候采用的一个度量，基本上进程中 RAM 每一页的规模都是按照其他使用同一页的进程数量的比例来确定的。（理论上）你把所有进程的 PSS 加起来就能得到他们一共使用的内存，比较不同进程的 PSS 也可以对他们的分量有一个粗略的认识。

另一个有趣的度量是 private dirty（私有脏页），大体上是进程中不能被换出到磁盘（不能由磁盘中相同的数据作为支撑）、不与其他进程共享的内存数量；而从另一个角度来看，这也是当进程消失（并可能迅速变成缓存或者其他形式）的时候返还给系统的内存。

SDK API 能做的就这么多。然而作为一个开发者，你可以对设备做更多的事情。

使用 adb 可以得到关于一个正在运行的系统的内存使用情况的大量信息。一个常用的命令是 `adb shell dumpsys meminfo`，这条命令会吐出一大坨关于每一个 Java 进程内存使用情况的信息，包括上面提到的内容，也包括其他各种各样的东西。你还可以附上一个进程的名字或者 PID，比如 `adb shell dumpsys meminfo system` 就会给出系统进程的信息：

```
** MEMINFO in pid 890 [system] **
                    native   dalvik    other    total
            size:    10940     7047      N/A    17987
       allocated:     8943     5516      N/A    14459
            free:      336     1531      N/A     1867
           (Pss):     4585     9282    11916    25783
  (shared dirty):     2184     3596      916     6696
    (priv dirty):     4504     5956     7456    17916

 Objects
           Views:      149        ViewRoots:        4
     AppContexts:       13       Activities:        0
          Assets:        4    AssetManagers:        4
   Local Binders:      141    Proxy Binders:      158
Death Recipients:       49
 OpenSSL Sockets:        0

 SQL
            heap:      205          dbFiles:        0
       numPagers:        0   inactivePageKB:        0
    activePageKB:        0
```

最上面的部分是最主要的，其中“size”是一个堆的地址空间的总大小，“allocated”是这个堆认为自己实际分配的空间大小（单位 kb），“free”是堆中空余的、可以进行分配的剩余空间（kb），“Pss”和“priv dirty”与上面讨论的相同，针对与每个堆相关的页。

如果你只是想查看所有进程的内存使用量，可以用 `adb shell procrank` 这条命令，在同一个系统上会得到类似这样的输出：

```
  PID      Vss      Rss      Pss      Uss  cmdline
  890   84456K   48668K   25850K   21284K  system_server
 1231   50748K   39088K   17587K   13792K  com.android.launcher2
  947   34488K   28528K   10834K    9308K  com.android.wallpaper
  987   26964K   26956K    8751K    7308K  com.google.process.gapps
  954   24300K   24296K    6249K    4824K  com.android.phone
  948   23020K   23016K    5864K    4748K  com.android.inputmethod.latin
  888   25728K   25724K    5774K    3668K  zygote
  977   24100K   24096K    5667K    4340K  android.process.acore
...
   59     336K     332K      99K      92K  /system/bin/installd
   60     396K     392K      93K      84K  /system/bin/keystore
   51     280K     276K      74K      68K  /system/bin/servicemanager
   54     256K     252K      69K      64K  /system/bin/debuggerd
```

这里的 Vss 和 Rss 两列都没有用（这些是直接得到的进程地址空间和内存使用，如果你把所有进程的内存使用量加起来的话会得到一个大得离谱的数字）。

Pss 和之前讨论的一样，Uss 就是 private dirty。

有一点有趣的地方值得注意：这里的 Pss 和 Uss 与我们之前在 `meminfo` 里看到的不一样。为什么呢？`procrank` 使用了一个与 `meminfo` 不同的内存机制来收集数据，得到的结果也有所不同。这又是为什么呢？坦白讲，我不知道。我觉得 `procrank` 可能更准确一些……但是说真的，这提醒我们：“对你得到的内存信息保持怀疑，而且通常很可疑”。

最后是这个命令：`adb shell cat /proc/meminfo`，它会给出系统内存使用情况的全面总结。会有很多数据，然而只有一开始的几个数字值得讨论（剩下的很少有人理解，我向这些人询问也经常得到互相矛盾的解释）：

```
MemTotal:         395144 kB
MemFree:          184936 kB
Buffers:             880 kB
Cached:            84104 kB
SwapCached:            0 kB
```

MemTotal 是内核与用户可用的内存总量（通常少于设备的物理内存，因为有一部分要用于收音机、DMA 缓冲等等）。

MemFree 是没有使用的内存量。你可以看到这个数字非常大，而因为我们会使用可用内存来保持进程的运行，通常在 Android 系统里这个数字可能只有几十 MB。

Cached 是用于文件系统缓存以及其他东西的内存数量。通常系统会为缓存开辟 20MB 左右的空间来防止坏页的发生，而 Android 的内存不足“杀手”会为特定的系统进行调节，来确保不会因为后台进程消耗过多缓存导致坏页。
