# Android 设备有没有唯一 ID？

[问题链接](http://stackoverflow.com/q/2785485/5152089)

Android 设备是否有唯一 id，如果有，通过 Java 来获取它的最简单的方式是什么？

## [Anthony Forloney 的答案](http://stackoverflow.com/a/2785493/5152089)

[`Settings.Secure#ANDROID_ID`](http://developer.android.com/reference/android/provider/Settings.Secure.html#ANDROID_ID) 会以唯一的 64 位十六进制字符串的形式返回 Android ID。

```java
import android.provider.Settings.Secure;

private String android_id = Secure.getString(getContext().getContentResolver(),
                                                        Secure.ANDROID_ID); 
```

**2015 年 6 月 5 日更新**

把这个答案作为可能的解决方法的同时，请将下面的 [Joe 的答案](https://github.com/7bitex/stackoverflow-android-top-qa/blob/master/contents/is-there-a-unique-android-device-id.md#joe-的答案) 作为备选方法。

### 评论

> 这个方法有时候可能会返回 null。文档里写着“重置为出厂设置后可能会改变”。风险自负，而且这个 ID 在已 root 的手机上可以被轻易修改。—— Seva Alekseyev

## [Joe 的答案](http://stackoverflow.com/a/2853253/5152089)

这个问题有很多答案，大多数答案都只能在“某些”时间工作，然而这是行不通的。

基于我在设备上的测试（都是手机，至少有一个是未激活的）：

 1. 所有测试设备调用 `TelephonyManager.getDeviceId()` 都返回了值；
 2. 所有 GSM 设备（测试时都插入了 SIM 卡）调用 `TelephonyManager.getSimSerialNumber()` 都返回了值；
 3. 所有 CDMA 设备调用 `getSimSerialNumber()` 都返回了 null（意料之中）；
 4. 所有添加了 Google 账户的设备对 `ANDROID_ID` 都返回了值；
 5. 所有 CDMA 设备对 `ANDROID_ID` 和 `TelephonyManager.getDeviceId()` 都返回了相同的值（或者是由同一个值派生的）——只要是在设置时添加了 Google 账户；
 6. 我还没有机会测试没插入 SIM 卡的 GSM 设备、没有添加 Google 账户的 GSM 设备，或者任何开启了飞行模式的设备。

所以，如果你想得到某个唯一标识设备的值，`TM.getDeviceId()` *应该*就足够了。很明显一些用户比其他人更偏执，因此对这些标识符做一次或更多次 hash 可能会比较好，这样得到的字符串仍然唯一标识设备，同时也不会暴露出用户的真实设备。例如，用 `String.hashCode()`，结合 UUID：

```java
    final TelephonyManager tm = (TelephonyManager) getBaseContext().getSystemService(Context.TELEPHONY_SERVICE);

    final String tmDevice, tmSerial, androidId;
    tmDevice = "" + tm.getDeviceId();
    tmSerial = "" + tm.getSimSerialNumber();
    androidId = "" + android.provider.Settings.Secure.getString(getContentResolver(), android.provider.Settings.Secure.ANDROID_ID);

    UUID deviceUuid = new UUID(androidId.hashCode(), ((long)tmDevice.hashCode() << 32) | tmSerial.hashCode());
    String deviceId = deviceUuid.toString();
```

可能会得到类似 `00000000-54b3-e7c7-0000-000046bffd97` 的结果。

对我来说这种方法足够好了。

如同 Richard 在下面提到的，不要忘记读 `TelephonyManager` 的属性是需要权限的，所以要在 manifest 里添加：

```xml
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
```
