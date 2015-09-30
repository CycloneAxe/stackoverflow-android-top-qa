# 下载一个文件并在 ProgressDialog 中显示进度

[问题链接](http://stackoverflow.com/q/3028306)

## [Cristian 的答案](http://stackoverflow.com/a/3028660)

有很多下载文件的方法。接下来我会介绍几种最常用的方法，由你来决定哪一种更适用于你的应用。

### 1. 使用 `AsyncTask` 并在对话框中显示下载进度

这种方法可以让你运行后台进程并同时更新 UI（这个例子中我们将更新一个进度条。）

下面是示例代码：

```java
// 把对话框定义为 Activity 的域
ProgressDialog mProgressDialog;

// 在 onCreate 方法中将其实例化
mProgressDialog = new ProgressDialog(YourActivity.this);
mProgressDialog.setMessage("A message");
mProgressDialog.setIndeterminate(true);
mProgressDialog.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
mProgressDialog.setCancelable(true);

// 当需要开始下载时执行下面的代码
final DownloadTask downloadTask = new DownloadTask(YourActivity.this);
downloadTask.execute("the url to the file you want to download");

mProgressDialog.setOnCancelListener(new DialogInterface.OnCancelListener() {
    @Override
    public void onCancel(DialogInterface dialog) {
        downloadTask.cancel(true);
    }
});
```

`AsyncTask` 看起来像是这样：

```java
// 通常 AsyncTask 的子类都会在 Activity 类内部定义，
// 这样的话，就可以从这里访问到 UI 线程了
private class DownloadTask extends AsyncTask<String, Integer, String> {

    private Context context;
    private PowerManager.WakeLock mWakeLock;

    public DownloadTask(Context context) {
        this.context = context;
    }

    @Override
    protected String doInBackground(String... sUrl) {
        InputStream input = null;
        OutputStream output = null;
        HttpURLConnection connection = null;
        try {
            URL url = new URL(sUrl[0]);
            connection = (HttpURLConnection) url.openConnection();
            connection.connect();

            // 期望得到 HTTP 200 OK，因此我们不会错误地保存下出错记录
            if (connection.getResponseCode() != HttpURLConnection.HTTP_OK) {
                return "Server returned HTTP " + connection.getResponseCode()
                        + " " + connection.getResponseMessage();
            }

            // 这对于显示下载百分比很有用处
            // 如果服务器没有报告长度，可能是 -1
            int fileLength = connection.getContentLength();

            // 下载文件
            input = connection.getInputStream();
            output = new FileOutputStream("/sdcard/file_name.extension");

            byte data[] = new byte[4096];
            long total = 0;
            int count;
            while ((count = input.read(data)) != -1) {
                // 允许按下返回键取消下载
                if (isCancelled()) {
                    input.close();
                    return null;
                }
                total += count;
                // 发布进度……
                if (fileLength > 0) // only if total length is known
                    publishProgress((int) (total * 100 / fileLength));
                output.write(data, 0, count);
            }
        } catch (Exception e) {
            return e.toString();
        } finally {
            try {
                if (output != null)
                    output.close();
                if (input != null)
                    input.close();
            } catch (IOException ignored) {
            }

            if (connection != null)
                connection.disconnect();
        }
        return null;
    }
```

上面的方法（`doInBackground`）永远都会在后台线程运行，你不应该在里面执行任何 UI 操作。另一方面，`onProgressUpdate` 和 `onPreExecute` 在 UI 线程上运行，因此你可以在这里修改进度条：

```java
    @Override
    protected void onPreExecute() {
        super.onPreExecute();
        // 拿到 CPU 锁从而防止用户在下载过程中按下电源键时 CPU 停止运行
        PowerManager pm = (PowerManager) context.getSystemService(Context.POWER_SERVICE);
        mWakeLock = pm.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK,
             getClass().getName());
        mWakeLock.acquire();
        mProgressDialog.show();
    }

    @Override
    protected void onProgressUpdate(Integer... progress) {
        super.onProgressUpdate(progress);
        // 到了这里长度就确定了，把 indeterminate 设置为 false
        mProgressDialog.setIndeterminate(false);
        mProgressDialog.setMax(100);
        mProgressDialog.setProgress(progress[0]);
    }

    @Override
    protected void onPostExecute(String result) {
        mWakeLock.release();
        mProgressDialog.dismiss();
        if (result != null)
            Toast.makeText(context,"Download error: "+result, Toast.LENGTH_LONG).show();
        else
            Toast.makeText(context,"File downloaded", Toast.LENGTH_SHORT).show();
    }
```

要让以上的代码可以运行，你需要 `WAKE_LOCK` 权限：

```xml
<uses-permission android:name="android.permission.WAKE_LOCK" />
```

### 2. 使用 `Service` 下载

这里有个大问题：*我如何从 `Service` 更新 `Activity`？*在下面的例子中我们会使用两个你可能不知道的类：`ResultReceiver` 和 `IntentService`。`ResultReceiver` 可以让我们从 `Service` 更新一个线程，而 `IntentService` 是 `Service` 的一个子类，可以建立一个新线程来执行后台工作（你应当知道 `Service` 其实是运行在应用的同一个线程中的，当你派生 `Service` 类的时候，你必须手动建立新线程来运行 CPU 阻塞的操作）。

下载服务看起来像是这样：

```java
public class DownloadService extends IntentService {
    public static final int UPDATE_PROGRESS = 8344;
    public DownloadService() {
        super("DownloadService");
    }
    @Override
    protected void onHandleIntent(Intent intent) {
        String urlToDownload = intent.getStringExtra("url");
        ResultReceiver receiver = (ResultReceiver) intent.getParcelableExtra("receiver");
        try {
            URL url = new URL(urlToDownload);
            URLConnection connection = url.openConnection();
            connection.connect();
            // 这个很有用，你可以显示一个典型的 0 到 100% 的进度条
            int fileLength = connection.getContentLength();

            // 下载文件
            InputStream input = new BufferedInputStream(connection.getInputStream());
            OutputStream output = new FileOutputStream("/sdcard/BarcodeScanner-debug.apk");

            byte data[] = new byte[1024];
            long total = 0;
            int count;
            while ((count = input.read(data)) != -1) {
                total += count;
                // publishing the progress....
                Bundle resultData = new Bundle();
                resultData.putInt("progress" ,(int) (total * 100 / fileLength));
                receiver.send(UPDATE_PROGRESS, resultData);
                output.write(data, 0, count);
            }

            output.flush();
            output.close();
            input.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

        Bundle resultData = new Bundle();
        resultData.putInt("progress" ,100);
        receiver.send(UPDATE_PROGRESS, resultData);
    }
}
```

把这个服务添加到 manifest 中：

```xml
<service android:name=".DownloadService"/>
```

`Activity` 会是这样：

```java
// 像第一个例子中那样初始化进度条对话框

// 你要像这样开始下载
mProgressDialog.show();
Intent intent = new Intent(this, DownloadService.class);
intent.putExtra("url", "url of the file to download");
intent.putExtra("receiver", new DownloadReceiver(new Handler()));
startService(intent);
```

这里就要让 `ResultReceiver` 派上用场了：

```java
private class DownloadReceiver extends ResultReceiver{
    public DownloadReceiver(Handler handler) {
        super(handler);
    }

    @Override
    protected void onReceiveResult(int resultCode, Bundle resultData) {
        super.onReceiveResult(resultCode, resultData);
        if (resultCode == DownloadService.UPDATE_PROGRESS) {
            int progress = resultData.getInt("progress");
            mProgressDialog.setProgress(progress);
            if (progress == 100) {
                mProgressDialog.dismiss();
            }
        }
    }
}
```

#### 2.1 使用 Groundy 库

[Groundy](http://casidiablo.github.com/groundy) 是一个帮助你在后台服务中执行代码的库，并且也是基于上面的 `ResultReceiver`。现在这个库已经**废弃**了。以下是**完整**的代码：

要显示对话框的 `Activity`……

```java
public class MainActivity extends Activity {

    private ProgressDialog mProgressDialog;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);

        findViewById(R.id.btn_download).setOnClickListener(new View.OnClickListener() {
            public void onClick(View view) {
                String url = ((EditText) findViewById(R.id.edit_url)).getText().toString().trim();
                Bundle extras = new Bundler().add(DownloadTask.PARAM_URL, url).build();
                Groundy.create(DownloadExample.this, DownloadTask.class)
                        .receiver(mReceiver)
                        .params(extras)
                        .queue();

                mProgressDialog = new ProgressDialog(MainActivity.this);
                mProgressDialog.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
                mProgressDialog.setCancelable(false);
                mProgressDialog.show();
            }
        });
    }

    private ResultReceiver mReceiver = new ResultReceiver(new Handler()) {
        @Override
        protected void onReceiveResult(int resultCode, Bundle resultData) {
            super.onReceiveResult(resultCode, resultData);
            switch (resultCode) {
                case Groundy.STATUS_PROGRESS:
                    mProgressDialog.setProgress(resultData.getInt(Groundy.KEY_PROGRESS));
                    break;
                case Groundy.STATUS_FINISHED:
                    Toast.makeText(DownloadExample.this, R.string.file_downloaded, Toast.LENGTH_LONG);
                    mProgressDialog.dismiss();
                    break;
                case Groundy.STATUS_ERROR:
                    Toast.makeText(DownloadExample.this, resultData.getString(Groundy.KEY_ERROR), Toast.LENGTH_LONG).show();
                    mProgressDialog.dismiss();
                    break;
            }
        }
    };
}
```

实现一个 **Groundy** 的 `GroundyTask` 来下载文件并显示进度：

```java
public class DownloadTask extends GroundyTask {    
    public static final String PARAM_URL = "com.groundy.sample.param.url";

    @Override
    protected boolean doInBackground() {
        try {
            String url = getParameters().getString(PARAM_URL);
            File dest = new File(getContext().getFilesDir(), new File(url).getName());
            DownloadUtils.downloadFile(getContext(), url, dest, DownloadUtils.getDownloadListenerForTask(this));
            return true;
        } catch (Exception pokemon) {
            return false;
        }
    }
}
```

并且在 manifest 里添加这个：

```xml
<service android:name="com.codeslap.groundy.GroundyService"/>
```

我觉得这是最简单的方法了。[从 GitHub](https://github.com/casidiablo/groundy/downloads) 下载最新版本的 jar 文件，就可以用了。记住，**Groundy** 的主要目的是在后台服务中调用外部 REST API 并将结果返回到 UI。如果你要在应用中实现这样的功能，将会是非常有用的。

#### 2.2 使用 [https://github.com/koush/ion](https://github.com/koush/ion)

### 3. 使用 `DownloadManager` 类（只在 Gingerbread 以及更新版本上可用）

这个方法很棒，你不需要操心如何手动下载文件、管理线程和流等等。Gingerbread 带来了一个全新的功能：`DownloadManager`，可以让你轻松地下载文件，把困难的工作交给系统来处理。

首先，来看一下这个工具方法：

```java
/**
 * @param context used to check the device version and DownloadManager information
 * @return true if the download manager is available
 */
public static boolean isDownloadManagerAvailable(Context context) {

    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.GINGERBREAD) {
        return true;
    }
    return false;
}
```

方法的名字就解释了它的作用。只要你确定 `DownloadManager` 可用，你就可以像这样：

```java
String url = "url you want to download";
DownloadManager.Request request = new DownloadManager.Request(Uri.parse(url));
request.setDescription("Some descrition");
request.setTitle("Some title");
// 为了能够成功运行，你必须使用 Android 3.2 以上版本来编译
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
    request.allowScanningByMediaScanner();
    request.setNotificationVisibility(DownloadManager.Request.VISIBILITY_VISIBLE_NOTIFY_COMPLETED);
}
request.setDestinationInExternalPublicDir(Environment.DIRECTORY_DOWNLOADS, "name-of-the-file.ext");

// 获得下载服务，并把文件添加到队列
DownloadManager manager = (DownloadManager) getSystemService(Context.DOWNLOAD_SERVICE);
manager.enqueue(request);
```

下载进度就会显示在通知栏里。

### 最后的一些想法

第一和第二种方法都只是冰山一角，如果你想写出鲁棒的应用，你还需要牢记更多东西。以下是一个简单的列举：

 - 你必须检查用户是否有可用的网络连接；
 - 确保你添加了正确的权限（`INTERNET` 和 `WRITE_EXTERNAL_STORAGE`）。如果要检查网络可用性，还需要 `ACCESS_NETWORK_STATE`；
 - 确保下载文件的目标目录存在并且具有写入权限；
 - 如果下载文件太大，你也许要实现如果上一次下载失败则继续下载的功能；
 - 如果你允许用户中断下载，用户会很感激的。

除非你想全程掌控下载过程，我建议你使用 `DownloadManager`，这东西已经把上面大多数的要求都处理好了。
