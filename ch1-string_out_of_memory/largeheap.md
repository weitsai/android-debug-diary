# 解決方案(一) - largeHeap

## 分析現場
現場還原了, 但是還是不知道兇手是怎麼樣殺死我的 App 的阿!!
原來是每個手機中各自的 App 可以使用的記憶體上限不同, 也就是有些手機可能是 96M, 而有一些則是 20M.

我們可以透過下面這段程式來瞭解自己手機每個 App 記憶體使用量上限是多少：
```java
ActivityManager am = (ActivityManager) getSystemService(ACTIVITY_SERVICE);
int mMemoryClass = am.getMemoryClass();
long mLargeMemoryClass = am.getLargeMemoryClass();
```

而 `android:largeHeap="true"` 就是告訴系統這個 App 使用量可能會比較大, 請放寬限制!!

## 什麼 2.3.3 因為 android:largeHeap="true" 而掛了？
雖然現在都是 Android 4.x 居多的時代, 但抱持著研究的精神當然還是得解決 2.3.3 閃退的問題, 現在讓我們來一步一步解決問題吧.

1. 建立 4.0 以上用的 values 資料夾
![建立 4.0 專用的 values 資料夾](http://api.drp.io/files/5481dc386da05.png)
![建立 4.0 專用的 values 資料夾](http://api.drp.io/files/5481dc3867c65.png)
2. 在 `values-v14` 下建立 `bool.xml`
![在 values-v14 下建立 bool.xml](http://api.drp.io/files/5481dc387e2d7.png)
![在 values-v14 下建立 bool.xml](http://api.drp.io/files/5481dc388e49c.png)
3. 在 `bool.xml` 加入 `<bool name="largeheap">true</bool>` </br>
`bool.xml` 檔案內容如下：
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <bool name="largeheap">true</bool>
</resources>
```
4. 在 `AndroidManifst.xml` 加入 `android:largeHeap="@bool/largeheap"`
5. 大功告成啦！！


## 參考文獻

- [http://stackoverflow.com/questions/14564146/out-of-memory-string-android](http://stackoverflow.com/questions/14564146/out-of-memory-string-android)
- [http://stackoverflow.com/questions/14918028/enable-androidlargeheap-in-android-4-and-disable-it-in-android-2-3?answertab=active#tab-top](http://stackoverflow.com/questions/14918028/enable-androidlargeheap-in-android-4-and-disable-it-in-android-2-3?answertab=active#tab-top)
- [http://developer.android.com/guide/topics/manifest/application-element.html](http://developer.android.com/guide/topics/manifest/application-element.html)
- [http://developer.android.com/reference/android/app/ActivityManager.html#getLargeMemoryClass()](http://developer.android.com/reference/android/app/ActivityManager.html#getLargeMemoryClass())
- [http://developer.android.com/reference/android/app/ActivityManager.html#getMemoryClass()](http://developer.android.com/reference/android/app/ActivityManager.html#getMemoryClass())

## 補充

那 largeHeap 的上限到底怎麼設定進去的呢?　那就得看一下 system 的設定檔, 如果你的裝置是 root 就可以使用下面的指令：

```
adb pull /system/build.prop [指定路徑]
```

接著打開 **build.prop** 然後跳到 **85** 行, 就可以看見你這個裝置的 hepap 的相關設定啦！

![build.prop](http://api.drp.io/files/54812a0b33b67.png)

如果有興趣也可以看看 system 在 build 的時候的設定檔唷~
[https://github.com/android/platform_frameworks_base/blob/master/core/jni/AndroidRuntime.cpp#L655](https://github.com/android/platform_frameworks_base/blob/master/core/jni/AndroidRuntime.cpp#L655)


## 有趣的小發現

在研究的過程竟然發現有人做了一個可以編輯 build.prop 的 App, 作者我就沒有玩過了, 請勇者們去嘗試吧！！

[https://play.google.com/store/apps/details?id=org.nathan.jf.build.prop.editor&hl=zh_HK](https://play.google.com/store/apps/details?id=org.nathan.jf.build.prop.editor&hl=zh_HK)

