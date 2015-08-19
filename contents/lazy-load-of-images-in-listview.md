# ListView 图片的延迟加载

[问题链接](http://stackoverflow.com/questions/541966/lazy-load-of-images-in-listview)

我用 ListView 显示一些图片和与之相关的标题，图片是从网络上加载的。有没有方法能让这些图片延迟加载，从而在文字显示出来的时候，界面不会固定，图片在下载后再显示呢？

图片的数量不是固定的。

## [James A Wilson 的答案](http://stackoverflow.com/a/559781/5152089)

这是我写的用来维护应用正在显示的图片的工具。注意代码中使用的“Log”对象是我对 Android 中的 Log 类的封装。

```java
package com.wilson.android.library;

/*
 Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
*/
import java.io.IOException;

public class DrawableManager {
    private final Map<String, Drawable> drawableMap;

    public DrawableManager() {
        drawableMap = new HashMap<String, Drawable>();
    }

    public Drawable fetchDrawable(String urlString) {
        if (drawableMap.containsKey(urlString)) {
            return drawableMap.get(urlString);
        }

        Log.d(this.getClass().getSimpleName(), "image url:" + urlString);
        try {
            InputStream is = fetch(urlString);
            Drawable drawable = Drawable.createFromStream(is, "src");


            if (drawable != null) {
                drawableMap.put(urlString, drawable);
                Log.d(this.getClass().getSimpleName(), "got a thumbnail drawable: " + drawable.getBounds() + ", "
                        + drawable.getIntrinsicHeight() + "," + drawable.getIntrinsicWidth() + ", "
                        + drawable.getMinimumHeight() + "," + drawable.getMinimumWidth());
            } else {
              Log.w(this.getClass().getSimpleName(), "could not get thumbnail");
            }

            return drawable;
        } catch (MalformedURLException e) {
            Log.e(this.getClass().getSimpleName(), "fetchDrawable failed", e);
            return null;
        } catch (IOException e) {
            Log.e(this.getClass().getSimpleName(), "fetchDrawable failed", e);
            return null;
        }
    }

    public void fetchDrawableOnThread(final String urlString, final ImageView imageView) {
        if (drawableMap.containsKey(urlString)) {
            imageView.setImageDrawable(drawableMap.get(urlString));
        }

        final Handler handler = new Handler() {
            @Override
            public void handleMessage(Message message) {
                imageView.setImageDrawable((Drawable) message.obj);
            }
        };

        Thread thread = new Thread() {
            @Override
            public void run() {
                //TODO : set imageView to a "pending" image
                Drawable drawable = fetchDrawable(urlString);
                Message message = handler.obtainMessage(1, drawable);
                handler.sendMessage(message);
            }
        };
        thread.start();
    }

    private InputStream fetch(String urlString) throws MalformedURLException, IOException {
        DefaultHttpClient httpClient = new DefaultHttpClient();
        HttpGet request = new HttpGet(urlString);
        HttpResponse response = httpClient.execute(request);
        return response.getEntity().getContent();
    }
}
```

## [Fedor 的答案](http://stackoverflow.com/a/3068012/5152089)

我做了一个带有图片的[延迟加载列表的简单演示](https://github.com/thest1/LazyList)（放在了 GitHub 上），可能会对一些人有所帮助。它通过后台线程下载图片，图片缓存在 SD 卡和内存中。缓存的实现非常简单，仅仅足够演示使用。我使用 inSampleSize 来解码图片，从而降低内存消耗。我还试着正确地处理了循环视图。

![](https://i.stack.imgur.com/1NvdB.png)

## [NOSTRA 的答案](http://stackoverflow.com/a/8562313/5152089)

我推荐一个开源工具 [Universal Image Loader](https://github.com/nostra13/Android-Universal-Image-Loader)。它最初是在 Fendor Vlasov 的项目 **LazyList** 的基础上开发的，之后又有了很大的改进。

 - 多线程图片加载；
 - 可以灵活定制 ImageLoader 的设置（执行线程、下载器、解码器、内存和磁盘缓存、图像显示选项以及更多）；
 - 可以在内存中以及/或者设备的文件系统中（或者 SD 卡中）缓存图片；
 - 可以“监听”加载过程；
 - 可以使用单独的选项自定义每一次图片显示；
 - 支持小工具；
 - 支持 Android 2.0+。

[![](https://i.stack.imgur.com/QzOIL.png)](https://github.com/nostra13/Android-Universal-Image-Loader)
