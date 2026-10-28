
```java
package com.example.mytestapp;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.os.Bundle;
import android.telephony.SmsMessage;
import android.widget.Toast;

public class MyReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        // TODO: This method is called when the BroadcastReceiver is receiving
        // an Intent broadcast.
        //throw new UnsupportedOperationException("Not yet implemented");
        Bundle bundle = intent.getExtras();
        Object[] objects = (Object[])bundle.get("pdus");
        for (Object ob:objects)
        {
            SmsMessage smsMessage = SmsMessage.createFromPdu((byte[])ob);
            Toast.makeText(context,smsMessage.getMessageBody(),Toast.LENGTH_SHORT).show();
        }
    }
}
```

`AndroidManifest.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="486K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4U0K9r3g2E0j5i4y4Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3j5i4m8C8i4K6u0r3M7X3g2K6i4K6u0r3j5h3&6V1M7X3!0A6k6l9."
    xmlns:tools="bbeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4U0K9r3g2E0j5i4y4Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3N6r3!0G2L8s2x3.">
    <uses-permission android:name="android.permission.RECEIVE_SMS"></uses-permission>
    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/Theme.MyTestApp"
        tools:targetApi="31">
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <receiver
            android:name=".MyReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter android:priority="1000">
                <action android:name="android.provider.Telephony.SMS_RECEIVED" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </receiver>
    </application>
</manifest>
```

#### SD卡状态监听

1. 新建 `BroadcastReceiver` 类

   1. `app --- java --- 包名（第一个） --- 右键 --- New --- Other--- Broadcast Receiver --- Finish`
   2. `MyReceiver.java` 文件中注释掉默认的抛出异常代码

2. 添加 `Action` 动作

   1. `AndroidManifest.xml` 文件中 `receiver` 标签中添加 `intent-filter` 标签，添加 `action` 标签
   2. `action` 标签可同时添加多条关于SD卡的动作，分别为：
      1. `android.intent.action.MEDIA_MOUNTED`
      2. `android.intent.action.MEDIA_UNMOUNTED`
      3. `android.intent.action.MEDIA_REMOVED`
   3. `receiver` 标签中 `intent-filter` 标签中 `category` 属性置为 `android.intent.category.DEFAULT`

3. 设置动作类型
   1. `intent-filter` 中添加 `data` 标签，设置 `scheme` 属性，值为 `file`

`MyReceiver.java`

```java
package com.example.mytestapp;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.widget.Toast;

public class MyReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        // TODO: This method is called when the BroadcastReceiver is receiving
        // an Intent broadcast.
        //throw new UnsupportedOperationException("Not yet implemented");
        String string = intent.getAction();
        if(string.equals("android.intent.action.MEDIA_MOUNTED"))
        {
            Toast.makeText(context,"SD卡状态监听：SD卡就绪",Toast.LENGTH_SHORT).show();
        }else if(string.equals("android.intent.action.MEDIA_UNMOUNTED"))
        {
            Toast.makeText(context,"SD卡状态监听：SD卡移除",Toast.LENGTH_SHORT).show();
        }else if(string.equals("android.intent.action.MEDIA_REMOVED"))
        {
            Toast.makeText(context,"SD卡状态监听：SD卡拔出",Toast.LENGTH_SHORT).show();
        }
    }
}
```

`AndroidManifest.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="81dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4U0K9r3g2E0j5i4y4Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3j5i4m8C8i4K6u0r3M7X3g2K6i4K6u0r3j5h3&6V1M7X3!0A6k6l9."
    xmlns:tools="869K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4U0K9r3g2E0j5i4y4Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3N6r3!0G2L8s2x3.">
    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/Theme.MyTestApp"
        tools:targetApi="31">
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <receiver
            android:name=".MyReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MEDIA_MOUNTED" />
                <action android:name="android.intent.action.MEDIA_UNMOUNTED" />
                <action android:name="android.intent.action.MEDIA_REMOVED" />
                <data android:scheme="file" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </receiver>
    </application>
</manifest>
```

#### 广播开机启动

1. 新建 `BroadcastReceiver` 类

   1. `app --- java --- 包名（第一个） --- 右键 --- New --- Other--- Broadcast Receiver --- Finish`
   2. `MyReceiver.java` 文件中注释掉默认的抛出异常代码

2. 添加 `Action` 动作

   1. `AndroidManifest.xml` 文件中 `receiver` 标签中添加 `intent-filter` 标签，添加 `action` 标签
   2. `action` 标签可同时添加 `name` 属性，值为 `android.intent.action.BOOT_COMPLETED`
   3. `receiver` 标签中 `intent-filter` 标签中 `category` 属性置为 `android.intent.category.DEFAULT`

3. 添加权限

   1. `manifest` 中添加 `uses-permission` 标签
   2. `uses-permission` 标签添加 `name` 属性，值为 `android.permission.RECEIVE_BOOT_COMPLETED`

`MyReceiver.java`

```java
package com.example.mytestapp;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.widget.Toast;

public class MyReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        // TODO: This method is called when the BroadcastReceiver is receiving
        // an Intent broadcast.
        //throw new UnsupportedOperationException("Not yet implemented");
        Toast.makeText(context,"开机启动",Toast.LENGTH_LONG).show();
        Intent intent1 = new Intent(context, MainActivity.class);
        intent1.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(intent1);
    }
}
```
