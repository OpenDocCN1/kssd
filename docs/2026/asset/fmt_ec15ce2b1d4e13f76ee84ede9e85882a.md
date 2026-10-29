
# AndServer服务类

```java
package com.yf.douyintool;
import android.content.Context;
import android.util.Log;
import com.yanzhenjie.andserver.AndServer;
import com.yanzhenjie.andserver.Server;
import java.net.InetAddress;
import java.net.UnknownHostException;
import java.util.concurrent.TimeUnit;
/**
* Created by Zhenjie Yan on 2018/6/9.
*/
public class ServerManager {
private static final String TAG = "ServerManager";
private Server mServer;
/**
* Create server.
*/
public ServerManager(Context context) {
InetAddress inetAddress = null;
try {
inetAddress = InetAddress.getByName("0.0.0.0");
} catch (UnknownHostException e) {
e.printStackTrace();
}
mServer = AndServer.serverBuilder(context)
.inetAddress(inetAddress)
.port(8080)
.timeout(10, TimeUnit.SECONDS)
.listener(new Server.ServerListener() {
@Override
public void onStarted() {
// TODO The server started successfully.
Log.d(TAG, "onStarted: ");
}
@Override
public void onStopped() {
// TODO The server has stopped.
Log.d(TAG, "onStarted: ");
}
@Override
public void onException(Exception e) {
Log.e(TAG, "onException: ",e );
// TODO An exception occurred while the server was starting.
}
})
.build();
}
/**
* Start server.
*/
public void startServer() {
if (mServer.isRunning()) {
// TODO The server is already up.
} else {
mServer.startup();
}
}
/**
* Stop server.
*/
public void stopServer() {
if (mServer.isRunning()) {
mServer.shutdown();
} else {
Log.w("AndServer", "The server has not started yet.");
}
}
}
```

## 在 `MainActivity.onCreate` 方法中加入

```java
serverManager = new ServerManager(this);
serverManager.startServer();
Log.i("address", NetUtils.getLocalIPAddress()+":8080");
application = this;
Thread.setDefaultUncaughtExceptionHandler(handler);
```

## 崩溃重启

```java
private Thread.UncaughtExceptionHandler handler = (t, e) -> {
restartApp(); //发生崩溃异常时,重启应用
};
```

# 测试接口

## getDeviceData

![](upload/attach/202004/877918_FXUVRNJ57XCAY4R.png)

## getQuery

![](upload/attach/202004/877918_J2U63ZJUX6CG79B.png)

## request

![](upload/attach/202004/877918_UGZST773ZXUCTR4.png)

# Cookie获取方式

有爬取登录接口实现的，但其实有一种简单的方式就是二维码登录，<a href="elink@26aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3I4G2N6h3W2K6L8r3W2$3K9g2)9J5k6h3y4G2L8g2)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3x3e0b7&6">链接地址</a>

# 沟通讨论

有问题可以<a href="elink@767K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3I4G2N6h3W2K6L8r3W2$3K9g2)9J5k6h3y4G2L8g2)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3x3e0b7%4">原文链接</a>共同讨论。

# 结合AndServer实现接口开发

## Gradle引入依赖

```
implementation 'com.yanzhenjie.andserver:api:2.0.5'

annotationProcessor 'com.yanzhenjie.andserver:processor:2.0.5'

implementation 'com.alibaba:fastjson:1.1.71.android'
```

## 接口类编写
