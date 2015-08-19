# 为什么 Android 模拟器这么慢？如何为它加速？

[问题链接](http://stackoverflow.com/questions/1554099/why-is-the-android-emulator-so-slow-how-can-we-speed-up-the-android-emulator)

我用的是一块 2.67 GHz 的赛扬处理器、1.21 GB 内存和 32 位 Windows XP Professional 系统。我觉得 Android 模拟器在这样一台机器上面应该启动得很快，然而事实并不是这样。我按照所有步骤完成了 IDE、SDK、JDK 等等的配置，有的时候也可以很快地启动模拟器，但是这种情况非常少见。如果可能的话，我该如何解决这个问题？

即使模拟器启动并且加载了主屏幕，也是非常卡顿的。我已经尝试了用 3.5（Galileo）和 3.4（Ganymede）版本的 Eclipse IDE。

## [blaffie 的答案](http://stackoverflow.com/questions/1554099/why-is-the-android-emulator-so-slow-how-can-we-speed-up-the-android-emulator/17433073#17433073)

试试 [Genymotion](http://www.genymotion.com/)。你可以在注册后下载对应 Windows / Mac OS X / Linux 的版本，也可以找到一个用于 Eclipse 的插件：

> 要安装插件，启动 Eclipse 并点击“帮助 - 安装新软件（Help / Install New Software）”菜单，然后添加一个新的更新地址，URL 填写：http://plugins.genymotion.com/eclipse。按照 Eclipse 给出的步骤安装。

这个模拟器运行很快，反应也很迅速。

Genymotion 可以让你控制设备上的各种传感器，包括电池水平、信号强度和 GPS。

**更新**：最新的版本还包含了摄像头工具。

## [Vikas Patidar 的答案](http://stackoverflow.com/questions/1554099/why-is-the-android-emulator-so-slow-how-can-we-speed-up-the-android-emulator/5154636#5154636)

[Android Development Tools (ADT) 9.0.0](http://developer.android.com/sdk/eclipse-adt.html)（或者更新的版本）有一个功能可以让你保存 AVD（模拟器）的状态，从而可以很快地启动模拟器。你需要在创建新的 AVD 或者编辑已有的 AVD 时启用这个功能。

除此之外，我还把`设备内存`加大到了 `1024`，模拟器就运行得很流畅了。

更多的信息参考下面的截图：

> 建立一个带有保存快照功能的 AVD。

![](http://i.stack.imgur.com/ITRGe.png)

> 从快照启动模拟器。

![](http://i.stack.imgur.com/Ml9Cd.png)

为模拟器加速可以参考[Speed up your Android Emulator!](http://jolicode.com/blog/speed-up-your-android-emulator)。

## [Prashanth Sams 的答案](http://stackoverflow.com/questions/1554099/why-is-the-android-emulator-so-slow-how-can-we-speed-up-the-android-emulator/5154636#5154636)

**务必注意**：请**首先**查阅 [Intel 的虚拟化列表](http://ark.intel.com/Products/VirtualizationTechnology)来确保你的 CPU 支持 Intel VT。

#### 用 HAXM 为卡顿的 Android 模拟器加速

HAXM 是 **Intel Hardware Accelerated Execution Manager**（Intel 硬件加速执行管理器）的缩写。

目前它只支持 Intel® VT（Intel 虚拟化技术）。

Android 模拟器是基于 [QEMU](http://en.wikipedia.org/wiki/QEMU) 构建的。QEMU 和宿主系统 HAXM 驱动之间的接口被设计成与厂商无关。

![](http://i.stack.imgur.com/FtslF.jpg)

#### 为你的 Android 开发环境配置 HAXM 的步骤

 1. 更新 Eclipse：确保你的 Eclipse 和 ADT 插件都是最新版本。
 
 2. 更新 Android Tools：在所有的 Eclipse 插件都更新之后，还需要更新 Android SDK Tools。启动 Android SDK Manager，更新所有的 SDK 组件。要使用 HAXML，至少需要更新到版本 17。
    
    ![](http://i.stack.imgur.com/Tmobo.png)
 
    - 下载 **x86 Atom System Iamges** 和 **Intel Hardware Accelerated Execution Manager Driver**，参考下图：
      
      ![](http://i.stack.imgur.com/DufnA.png)
    
    - 运行 IntelHaxm.exe 安装 HAXM 驱动。这个文件会在以下的某个路径里：
      
      - `C:\Program Files\Android\android-sdk\extras\intel\Hardware_Accelerated_Execution_Manager`
      
      - `C:\Users\<user>\adt-bundle-windows-x86_64\sdk\extras\intel\Hardware_Accelerated_Execution_Manager`
      
      如果安装失败，提示必须启用 Intel VT，你需要在 [BIOS](http://en.wikipedia.org/wiki/BIOS) 里开启这个功能。参照[Enabling Intel VT (Virtualization Technology)](http://forums.lenovo.com/t5/T400-T500-and-newer-T-series/Enabling-Intel-VT-Virtualization-Technology/ta-p/388479)。
      
      ![](http://i.stack.imgur.com/Trlj7.png)
    
    - 建立一个*新的* x86 AVD，参考下图：
      
      ![](http://i.stack.imgur.com/V7sf1.png)
    
    - 或者对于新的 SDK：
      
      ![](http://i.stack.imgur.com/2EhQV.jpg)

## [Pehat 的答案](http://stackoverflow.com/questions/1554099/why-is-the-android-emulator-so-slow-how-can-we-speed-up-the-android-emulator/6058689#6058689)

试试 [Android x86](http://www.android-x86.org/)，运行起来比 Google 的 Android 模拟器快得多。按照以下步骤：

 1. 安装 [VirtualBox](http://www.virtualbox.org/)；
 2. 下载需要的 [ISO 文件](http://www.android-x86.org/download)；
 3. 建立一个 Linux 2.6 / Other Linux 的虚拟机，512 MB 内存，2 GB 硬盘。网络方面：PCnet-Fast III，附加到 [NAT](http://en.wikipedia.org/wiki/Network_address_translation)。你也可以用桥接转换器，但是需要在环境里配置一个 [DHCP](http://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol)；
 4. 在虚拟机里安装 Android x86 并运行；
 5. 按下 <kbd>Alt</kbd> + <kbd>F1</kbd>，输入 `netcfg`，记下 IP 地址，按下 <kbd>Alt</kbd> + <kbd>F7</kbd>；
 6. 在你的 Windows XP 系统里运行 cmd，跳转到 Android Tools 的目录，输入 `adb connect <虚拟机 IP>`；
 7. 运行 Eclipse，打开 ADT 插件，找到设备，搞定！
