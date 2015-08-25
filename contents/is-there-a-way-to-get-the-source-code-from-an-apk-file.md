# 有没有办法从 APK 文件获取源代码？

[问题链接](http://stackoverflow.com/questions/3593420/is-there-a-way-to-get-the-source-code-from-an-apk-file)

笔记本的硬盘坏了，我之前两个月写的源代码都丢失了。现在我只能找到之前用 email 发送给朋友的 APK 文件。

有没有办法从 APK 文件里提取出我的源代码呢？

## [Prankul Garg 的答案](http://stackoverflow.com/a/6081365/5262638)

破解 `.apk` 文件的详细步骤：

#### 第一步：

 1. 建立一个新文件夹，把想要破解的 `.apk` 文件复制过去；
 
 2. 把 `.apk` 文件的后缀名改成 `.zip`（例如把 `filename.apk` 改成 `filename.zip`），然后保存。现在你可以获取到 `classes.dex` 等文件。这时候你能够看到图片资源，但是看不到 XML 和 Java 文件，因此继续。

#### 第二步：

 1. 把 `.zip` 解压到当前目录（或者新的文件夹中）；
 
 2. [下载 dex2jar](https://github.com/pxb1988/dex2jar)，解压到同一文件夹中；
 
 3. 把 `classes.dex` 文件移动到 `dex2jar` 目录中；
 
 4. 打开命令行，把目录切换到刚才的目录。然后输入 `d2j-dex2jar classes.dex`（OS X 或者 Ubuntu 终端则输入 `./d2j-dex2jar.sh classes.dex`），然后按下回车。现在你就会在同一目录下得到 `classes.dex.dex2jar` 文件；
 
 5. [下载 Java 反编译工具](http://jd.benow.ca/)，双击 `jd-gui`，点击打开文件，从刚才的目录中选择 `classes.dex.dex2jar`。现在你得到 `.class` 文件了；
 
 6. 保存这些 `.class` 文件（在 `jd-gui` 中，点击 File -> Save All Sources），名称设为 `src`。这时候你就得到了 Java 源文件，但是 `.xml` 文件还是不可读，所以继续。

#### 第三步：

重新打开一个新文件夹。

 1. 把想要破解的 `.apk` 文件放进来；
 
 2. 下载最新版本的 [Apktool **和** Apktool Install Window](http://ibotpeaches.github.io/Apktool/install/)（都可以在同一个链接中下载），放到同一个文件夹中；
 
 3. 下载 [`framework-res.apk`](https://www.androidfilehost.com/?fid=23212708291677144)，放到同一个文件夹中（并不是所有的 `.apk` 文件都需要这个文件，不过没有影响）；
 
 4. 打开命令行窗口；
 
 5. 切换到 Apktool 的目录，输入以下命令：`apktool if framework-res.apk`；
 
 6. `apktool d myApp.apk`（`myApp.apk` 改成你想要破解的文件名）；

现在你就得到了一个新目录，可以轻松获取 APK 的 XML 文件。

#### 第四步：

只要把上面两步得到的目录合并到一个目录里，就得到源代码了。
