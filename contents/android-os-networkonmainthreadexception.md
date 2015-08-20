# android.os.NetworkOnMainThreadException

[问题链接](http://stackoverflow.com/questions/6343166/android-os-networkonmainthreadexception)

我在运行我的 RssReader 项目时，下面的这段代码出了一个错误。

```java
URL url = new URL(urlToRssFeed);
SAXParserFactory factory = SAXParserFactory.newInstance();
SAXParser parser = factory.newSAXParser();
XMLReader xmlreader = parser.getXMLReader();
RssHandler theRSSHandler = new RssHandler();
xmlreader.setContentHandler(theRSSHandler);
InputSource is = new InputSource(url.openStream());
xmlreader.parse(is);
return theRSSHandler.getFeed();
```

出现了这样的错误：

> android.os.NetworkOnMainThreadException

怎么解决这个问题？

## [spektom 的答案](http://stackoverflow.com/a/6343299/5152089)

如果程序试图在主线程进行网络操作，就会抛出这个异常。把代码放在 [`AsyncTask`](http://developer.android.com/reference/android/os/AsyncTask.html) 中运行：

```java
class RetrieveFeedTask extends AsyncTask<String, Void, RSSFeed> {

    private Exception exception;

    protected RSSFeed doInBackground(String... urls) {
        try {
            URL url= new URL(urls[0]);
            SAXParserFactory factory =SAXParserFactory.newInstance();
            SAXParser parser=factory.newSAXParser();
            XMLReader xmlreader=parser.getXMLReader();
            RssHandler theRSSHandler=new RssHandler();
            xmlreader.setContentHandler(theRSSHandler);
            InputSource is=new InputSource(url.openStream());
            xmlreader.parse(is);
            return theRSSHandler.getFeed();
        } catch (Exception e) {
            this.exception = e;
            return null;
        }
    }

    protected void onPostExecute(RSSFeed feed) {
        // TODO: check this.exception 
        // TODO: do something with the feed
    }
}
```

如何执行这个任务：在 MainActivity.java 的 `onCreate()` 方法中添加这样一行代码：

```java
new RetrieveFeedTask().execute(urlToRssFeed);
```

不要忘了在 AndroidManifest.xml 里写上：

```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

## [user1169115 的答案](http://stackoverflow.com/a/9289190/5152089)

大多数情况下你都应该按照被采纳的那个答案来做，不过如果你了解得很清楚而且必须同步地执行这段代码，你可以像下面这样覆盖掉默认行为。

在你的类中加上：

```java
StrictMode.ThreadPolicy policy = new StrictMode.ThreadPolicy.Builder().permitAll().build();

StrictMode.setThreadPolicy(policy);
```

并且在 AndroidManifest.xml 里加上这个权限：

```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

### 评论

> 这个主意很烂。正确的方法是避免在主线程中进行网络 IO（就像被采纳的答案那样）。—— MByD
>
> 这个答案是正确的，不过仅仅是对那些不笨不傻的程序员而言。如果只是需要一个同步（也就是阻塞程序运行）调用，那就需要这样做。我很高兴能看到 Android 默认会抛出一个异常（依我看来这是很有“帮助”的做法），但我也会说“谢谢，但这就是我想做的”然后覆盖掉它。—— Adam
