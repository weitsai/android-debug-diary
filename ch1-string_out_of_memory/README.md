# String_Out_Of_Memory
今天在開發字典查詢 App 的時候發生了檔案讀出來存成 buffer 後, 再轉成 String 的過程中發生 Out Of Memory, 一直在百思不解的情況下上社群問了解法, 得到的結果是在 `AndroidManifast.xml` 的 `<applicaton>` 標籤加入 `android:largeHeap="true"` 屬性解決, 可是有些手機卻不用就能夠正常執行, 到底是發生了什麼原因呢？讓我們來還原現場吧！

## 還原現場步驟：

1. 環境介紹
	- 環境：Android
	- 版本：4.3
	- 檔案格式：json
	- 測試機器：genymotion (Nexus s)
	- 測試檔案：[https://github.com/weitsai/hahadict/raw/master/dict.json.bz2](https://github.com/weitsai/hahadict/raw/master/dict.json.bz2)

2. 把檔案放到 Assess 裡面</br>
![把檔案放到 Assess 裡面](http://api.drp.io/files/548125031a7ed.png)
3. 在 Activity 把這個檔案讀出來並轉成 String
	```java
	AssetManager assetManager = getAssets();
	InputStream ims = assetManager.open("dict.json");
	int size = ims.available();
	byte[] buffer = new byte[size];
	ims.read(buffer);
	ims.close();
	// 41175344
	System.out.println(buffer.length);
	// java.lang.OutOfMemoryError
	String s = new String(buffer);
	```
4. 執行後就會發生 Out Of Memory
	```
	7975 AndroidRuntime E FATAL EXCEPTION: main
	7975 AndroidRuntime E java.lang.OutOfMemoryError
	7975 AndroidRuntime E at java.lang.String.<init>(String.java:255)
	7975 AndroidRuntime E at java.lang.String.<init>(String.java:171)
	7975 AndroidRuntime E at java.lang.String.<init>(String.java:141)
	7975 AndroidRuntime E at org.im.sundictionary.MainActivity.onCreate(MainActivity.java:66)
	7975 AndroidRuntime E at android.app.Activity.performCreate(Activity.java:5133)
	7975 AndroidRuntime E at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1087)
	7975 AndroidRuntime E at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2175)
	7975 AndroidRuntime E at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2261)
	7975 AndroidRuntime E at android.app.ActivityThread.access$600(ActivityThread.java:141)
	7975 AndroidRuntime E at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1256)
	7975 AndroidRuntime E at android.os.Handler.dispatchMessage(Handler.java:99)
	7975 AndroidRuntime E at android.os.Looper.loop(Looper.java:137)
	7975 AndroidRuntime E at android.app.ActivityThread.main(ActivityThread.java:5103)
	7975 AndroidRuntime E at java.lang.reflect.Method.invokeNative(Native Method)
	7975 AndroidRuntime E at java.lang.reflect.Method.invoke(Method.java:525)
	7975 AndroidRuntime E at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:737)
	7975 AndroidRuntime E at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:553)
	7975 AndroidRuntime E at dalvik.system.NativeStart.main(Native Method)

	```
