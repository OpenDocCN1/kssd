
##### startService 示例3

实现一个定时器，其重点在于以下几点：

- `Service` 与 `Acivity` 以广播方式进行的交互
- 在`Activity`中以内部类方式创建广播接收者，重写`onReceive`方法，用于接收广播
- 在`Activity` 的 `onCreat`方法中动态中注册广播接收者

**MyService.java**

```
package com.example.mytestapp;
import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import android.util.Log;
import java.util.Timer;
import java.util.TimerTask;

public class MyService extends Service {
private Timer timer;
private int time = 0;

public MyService() {
}
@Override
public IBinder onBind(Intent intent) {
// TODO: Return the communication channel to the service.
throw new UnsupportedOperationException("Not yet implemented");
}
@Override
public void onCreate() {
Log.d("lxz","service is onCreate");
super.onCreate();
timer = new Timer();
TimerTask timerTask = new TimerTask() {
@Override
public void run() {
Log.d("lxz","time is run");
Intent intent = new Intent();
intent.setAction("showTime.lxz");
intent.putExtra("time","时间：" + time);
sendBroadcast(intent);
time++;
}
};
timer.schedule(timerTask,0,1000);
}
@Override
public int onStartCommand(Intent intent, int flags, int startId) {
Log.d("lxz","service is onStartCommand");
time = 0;
return super.onStartCommand(intent, flags, startId);
}
@Override
public void onDestroy() {
Log.d("lxz","service is onDestroy");
super.onDestroy();
if(timer != null)
{
timer.cancel();
timer = null;
}
}
}
```

**MainActivity.java**

```
package com.example.mytestapp;

import androidx.appcompat.app.AppCompatActivity;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.os.Bundle;
import android.view.View;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity{
private TimerReceiver timerReceiver;

@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);

timerReceiver = new TimerReceiver();
IntentFilter intentFilter = new IntentFilter();
intentFilter.addAction("showTime.lxz");
registerReceiver(timerReceiver, intentFilter);
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

public class TimerReceiver extends BroadcastReceiver{
@Override
public void onReceive(Context context, Intent intent) {
String action = intent.getAction();
if(action.equals("showTime.lxz"))
{
String time = intent.getStringExtra("time");
Toast.makeText(MainActivity.this,"time：" + time, Toast.LENGTH_SHORT).show();
}
}
}
}
```

**AndroidManifest.xml**

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="cc3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4U0K9r3g2E0j5i4y4Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3j5i4m8C8i4K6u0r3M7X3g2K6i4K6u0r3j5h3&6V1M7X3!0A6k6l9.." xmlns:tools="b3dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4U0K9r3g2E0j5i4y4Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3N6r3!0G2L8s2x3.">
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

**activity_main.xml**
```
