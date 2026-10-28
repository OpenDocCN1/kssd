
##### startService 示例2

实现一个电话监听器，其重点在于以下几点：

- 在 `MyService` 中定义一个内部类，用于继承并实现 `PhoneStateListener` 接口
- 添加读取电话状态的权限

  `AndroidManifest.xml` 中添加 `uses-permission` 标签，值为 `android.permission.READ_PHONE_STATE`
- 停止服务时将监听状态置为 `NONE`

**MyService.java**

```
package com.example.mytestapp;
import android.app.Service;
import android.content.Intent;
import android.os.Build;
import android.os.IBinder;
import android.telephony.PhoneStateListener;
import android.telephony.TelephonyManager;
import android.util.Log;
import androidx.annotation.RequiresApi;

public class MyService extends Service {
private TelephonyManager telephonyManager;
private MyPhoneStateListener mListener;

public MyService() {
}
@Override
public IBinder onBind(Intent intent) {
// TODO: Return the communication channel to the service.
throw new UnsupportedOperationException("Not yet implemented");
}
@RequiresApi(api = Build.VERSION_CODES.M)
@Override
public void onCreate() {
Log.d("lxz","service is onCreate");
super.onCreate();
telephonyManager = (TelephonyManager)getSystemService(TELEPHONY_SERVICE);
mListener = new MyPhoneStateListener();
telephonyManager.listen(mListener, PhoneStateListener.LISTEN_CALL_STATE);
}
@Override
public int onStartCommand(Intent intent,int flags,int startId) {
Log.d("lxz","service is onStartCommand");
return super.onStartCommand(intent, flags, startId);
}
@Override
public void onDestroy() {
Log.d("lxz","service is onDestroy");
super.onDestroy();
telephonyManager.listen(mListener, PhoneStateListener.LISTEN_NONE);
}
private class MyPhoneStateListener extends PhoneStateListener{
@Override
public void onCallStateChanged(int state, String phoneNumber) {
super.onCallStateChanged(state, phoneNumber);
switch (state)
{
case TelephonyManager.CALL_STATE_IDLE:
Log.d("lxz","电话闲置");
break;
case TelephonyManager.CALL_STATE_RINGING:
Log.d("lxz","电话响铃");
break;
case TelephonyManager.CALL_STATE_OFFHOOK:
Log.d("lxz","电话接听");
break;
}
}
}
}
```

**AndroidManifest.xml**

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="..." xmlns:tools="...">
<uses-permission android:name="android.permission.READ_PHONE_STATE"></uses-permission>
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
<service
android:name=".MyService"
android:enabled="true"
android:exported="true">
</service>
</application>
</manifest>
```

**MainActivity.java**

```
package com.example.mytestapp;

import androidx.appcompat.app.AppCompatActivity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;

public class MainActivity extends AppCompatActivity{
@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);
}
public void click1(View view) {
Intent intent = new Intent();
intent.setClass(this, MyService.class);
startService(intent);
}
public void click2(View view) {
Intent intent = new Intent();
intent.setClass(this, MyService.class);
stopService(intent);
}
}
```

**activity_main.xml**
