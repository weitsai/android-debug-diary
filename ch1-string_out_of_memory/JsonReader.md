# 解決方案(二) - JsonReader
還記得 [前面提到使用 largeHeap 的解法嗎？](http://www.codedata.com.tw/mobile/android-mine-1-string-out-of-memory/) 這篇文章嗎？　雖然有解決 String Out Of Memory 的問題, 但是其實這樣的解決方法並不好, 因為我們是更改系統本身對每個 App 的限制, 我們這次就用比較正規的解法吧！！


## 前提摘要
使用的是下面這樣的方式將 `Asset` 中的 `dict.json` 資料讀出, 在 `new String` 發生 String：

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

## dict.json 架構
`dict.json` 格式大約如下, 檔案大小 41M.

```json
{
  "八":{
    "ㄅㄚ":[
      "介於七與九之間的自然數。如：「六、七、八、九……」。大寫作「捌」，阿拉伯數字作「8」。",
      "姓。如漢代有西域人八滑。",
      "二一四部首之一。","表示數量是八的。如：「八字」、「八方」。",
      "形容多數或多方面。如：「四通八達」。（「八」字口語連用在去聲字前讀成陽平，如：「八號」、「八拜」。）"
    ]
  },
  "八伯":{
    "ㄅㄚ　ㄅㄛˊ":[
      "古代京畿外八> 州的最高長官，分別掌管四方諸侯，相傳堯､舜時皆有設置。堯時八伯有驩兜、共工、放齊、鯀，其餘則不可考。舜時八伯相傳伯夷為陽伯，羲仲、羲叔之後為二羲伯，棄為夏伯，咎繇為秋伯>    ，和仲、和叔之後為和伯，垂為冬伯，一人不詳。見宋˙王應麟˙小學紺珠˙卷五˙名臣類上。",
      "兗州八伯的簡稱。見「兗州八伯」條。"
    ]
  }
}
```

## 讓我們來不用透過設定 **largeheap** 就解決 OOM 吧！

1.環境介紹
  - 環境：Android
  - 版本：4.3
  - 檔案格式：json
  - 測試機器：genymotion (Nexus s)
  - 測試檔案：[https://github.com/weitsai/hahadict/raw/master/dict.json.bz2](https://github.com/weitsai/hahadict/raw/master/dict.json.bz2)

2.我們先建立 `Dict.java`, 用來儲存每個單字的注音和解釋.
  ```java
  public class Dict {
      private String phonetic;
      private String meaning;

      public Dict() {
      }

      public Dict(String phonetic, String meaning) {
          this.phonetic = phonetic;
          this.meaning = meaning;
      }

      public void setPhonetic(String phonetic) {
          this.phonetic = phonetic;
      }

      public void setMeaning(String meaning) {
          this.meaning = meaning;
      }

      public String getPhonetic() {
          return phonetic;
      }

      public String getMeaning() {
          return meaning;
      }
  }
  ```

3.使用原生 [JsonReader](http://developer.android.com/reference/android/util/JsonReader.html) 的解法

  3-1.方法介紹
  - [hashNext()](http://developer.android.com/reference/android/util/JsonReader.html#hasNext(%29) - 用來判斷是不是還有下一筆資料
  - [beginArray()](http://developer.android.com/reference/android/util/JsonReader.html#beginArray(%29) - 往內一層 Array
  - [endArray()](http://developer.android.com/reference/android/util/JsonReader.html#endArray(%29) -  回到外面一層 Array
  - [beginObject()](http://developer.android.com/reference/android/util/JsonReader.html#beginObject(%29) - 往內一層 Object
  - [endObject()](http://developer.android.com/reference/android/util/JsonReader.html#endObject(%29) - 回到外面一層 Object
  - [nextName()](http://developer.android.com/reference/android/util/JsonReader.html#nextName(%29) - 取得 Object Key
  - [nextString()](http://developer.android.com/reference/android/util/JsonReader.html#nextString(%29) - 取得 String


  > 注意事項：
  > 1.如果下一層是 Array 結果使用 beginObject, 會產生下列訊息：
  > `java.lang.IllegalStateException: Expected BEGIN_OBJECT but was BEGIN_ARRAY`
  > 2.想要 beginObject 之前, 一定要先把前面的 kny 讀出來, 否則也會發生錯誤！！
  > `java.lang.IllegalStateException: Expected BEGIN_OBJECT but was NAME`

  3-2.現在讓我們從內往外讀出來吧
  * 建立 `readStringsArray(JsonReader reader)` 方法, 來得到單字解釋的 Array

  ```java
  private List readStringsArray(JsonReader reader) throws IOException {
      List strings = new ArrayList();
      reader.beginArray();
      // 取得解釋
      while (reader.hasNext()) {
          strings.add(reader.nextString());
      }
      reader.endArray();
      return strings;
  }
  ```
  * 建立 `getDict(JsonReader reader)` 方法, 得到 Dict Object, 裡面存放單字的解釋及注音

  取得注音並透過 `readStringsArray()` 來取得解釋,

  ```java
  private Dict getDict(JsonReader reader) throws IOException {
      HashMap<String,String> map = new HashMap<String, String>();
      // 解開第二層 object
      reader.beginObject();

      Dict mDict = null;
      while (reader.hasNext()) {
      // 取得注音
      String phonetic = reader.nextName();
      String meaning = TextUtils.join(",", readStringsArray(reader).toArray()).replace(",", "\n");
          mDict = new Dict(phonetic, meaning);
      }
      reader.endObject();

      return mDict;
  }

  ```
  * 建立 `readDictJsonStream(InputStream in)`, 取得所有單字的資料

  ```java
  public HashMap<String, Dict> readDictJsonStream(InputStream in) throws IOException {
      JsonReader reader = new JsonReader(new InputStreamReader(in, "UTF-8"));
        // 解開第一層 Object
      reader.beginObject();

      HashMap<String, Dict> dictMap = new HashMap<String, Dict>();

      while (reader.hasNext()) {
          // 取得標題
          String name = reader.nextName();
          Dict mDict = getDict(reader);
          if (mDict == null) {
              continue;
          }
          dictMap.put(name, mDict);
      }

      reader.endObject();
      reader.close();
      return dictMap;
  }
```
  * 接著我們就可以使用剛剛建立的 `readDictJsonStream(InputStream in)`, 使用方式如下：

  ```java
  AssetManager assetManager = getAssets();
  InputStream ims = assetManager.open("dict.json");
  HashMap<String, String> mDictsMap = readDictJsonStream(ims);
  // ㄅㄚ
  Log.d("Dict", mDictsMap.get("八").phonetic());
  ```

4.接著我們用 [JsonReader](http://developer.android.com/reference/android/util/JsonReader.html) + [Gson](https://code.google.com/p/google-gson/) 來試試看會不會更簡單呢？

```java
public HashMap<String, Dict> readDictJsonStream(InputStream in) throws IOException {

  HashMap<String, String> dictMap = new HashMap<String, String>();
  JsonReader reader = new JsonReader(new InputStreamReader(in, "UTF-8"));

  // 解開第一層 Object
  reader.beginObject();

  // 輪詢
  while (reader.hashNext()) {
    // 取得標題
    String name = reader.nextName();
    // 解開第二層 object
    reader.beginObject();
    // 取得注音
    String phonetic = reader.nextNmae();
    // 取得解釋
    String[] meanings = gson.fromJson(reader, String[].class);

    Dict mDict = new Dict(phonetic, TextUtil.join(",", meanings).replace(",", "\n");
    dictMap.put(name, mDict);

  }
}
```

