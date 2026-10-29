
// `"ro.boottime.goldfish-logcat"`,

// `"ro.hardware.audio.primary"`,

// `"init.svc.ranchu-net"`,

// `"init.svc.ranchu-setup"`,

// `"ro.boottime.ranchu-net"`,

// `"ro.boottime.ranchu-setup"`,

// `"init.svc.droid4x"`,

// `"init.svc.noxd"`,

// `"init.svc.qemud"`,

// `"init.svc.goldfish-setup"`,

// `"init.svc.goldfish-logcat"`,

// `"init.svc.ttVM\_x86-setup"`,

// `"vmos.browser.home"`,

// `"vmos.camera.enable"`,

// `"ro.trd\_yehuo\_searchbox"`,

// `"init.svc.microvirtd"`,

// `"init.svc.vbox86-setup"`,

// `"ro.ndk\_translation.version"`,

// `"redroid.width"`,

// `"redroid.height"`,

// `"redroid.fps"`,

// `"ro.rf.vmname"`

// };

//

// for (`String` `str` : `qemuProps`) {

// `String` `val` = `SystemPropertiesUtils.getProperty`(str, null);

// if (`val` != null) {

// `choose.add`(str);

// }

// }

// } catch (`Throwable` e) {

// `e.printStackTrace`();

// }

//判断是否存在指定硬件

`PackageManager` `pm` = null;

try {

pm = `context.getPackageManager`();

`String[]` `features` = {

//`PackageManager.FEATURE\_RAM\_NORMAL`,//这个存在问题,自己组装的手机可能导致这个痕迹找不到

`PackageManager.FEATURE\_BLUETOOTH`,

`PackageManager.FEATURE\_CAMERA\_FLASH`,

`PackageManager.FEATURE\_TELEPHONY`

};

for (`String` `feature` : `features`) {

if (!pm.hasSystemFeature(feature)) {

`choose.add`(feature);

}

}

} catch (`Throwable` `ignored`) {

}

try {

`String[]` `emuPkgs` = {

`"com.google.android.launcher.layouts.genymotion"`,

`"com.bluestacks"`,

`"com.bignox.app"`

};

for (`String` `pkg` : `emuPkgs`) {

try {

if (pm != null) {

pm.getPackageInfo(pkg, 0);

}

`choose.add`(pkg);

} catch (`Throwable` e) {

//`e.printStackTrace`();

}

}

} catch (`Throwable` `ignored`) {

}

try {

`SensorManager` `sensor` = (`SensorManager`) `context.getSystemService`(Context.SENSOR\_SERVICE);

int `sensorSize` = `sensor.getSensorList`(Sensor.TYPE\_ALL).size();

for (int i = 0; i < `sensorSize`; i++) {

`Sensor` s = `sensor.getDefaultSensor`(i);

if (s != null && s.getName().contains("Goldfish")) {

`choose.add`(s.getName());

}

}

} catch (`Throwable` `ignored`) {

}

try {

if (`checkSelfPermission`(context, "android.permission.READ\_SMS") == 0 ||

`checkSelfPermission`(context, "android.permission.READ\_PHONE\_NUMBERS") == 0 ||

`checkSelfPermission`(context, "android.permission.READ\_PHONE\_STATE") == 0) {

`TelephonyManager` `telephonyManager` = (`TelephonyManager`) `context.getSystemService`(Context.TELEPHONY\_SERVICE);

`String` `phoneNumber` = `telephonyManager.getLine1Number`();

`String[]` `phoneNumbers` = {

"15555215554",

"15555215556",

"15555215558",

"15555215560",

"15555215562",

"15555215564",

"15555215566",

"15555215568",

"15555215570",

"15555215572",

"15555215574",

"15555215576",

"15555215578",

"15555215580",

"15555215582",

"15555215584"

};

if(phoneNumber!=null) {

for (`String` `phone` : `phoneNumbers`) {

if (`phoneNumber.equalsIgnoreCase`(phone)) {

`choose.add`(phone);

break;

}

}

}

}

} catch (`Exception` e) {

`e.printStackTrace`();

}

if (`choose.size`() > 0) {

`ListItemBean` `item` = new `ListItemBean`("检测到APK运行在虚拟机&模拟器中",

`ListItemBean.RiskLeave.Deadly`,

`choose.toString`()

);

for (`String` `str` : `choose`) {

`item.putData`(str);

}

return `item`;

}

return null;

}

## 检测云手机：

这块思路还是很多的，不同的云手机检测的思路也不一样 。大部分云手机做的还是很好的，很多都可以过掉Hunter的检测 。

### 检测电流&电压：

```
private final BroadcastReceiver batteryInfoReceiver = new BroadcastReceiver() {
@Override
public void onReceive(Context context, Intent intent) {
// 电池状态
int plugged = intent.getIntExtra(BatteryManager.EXTRA\_PLUGGED, -1);
// 电压（以毫伏为单位）
int voltage = intent.getIntExtra(BatteryManager.EXTRA\_VOLTAGE, -1);
// 获取电池电流（毫安）
int currentNow = -1;
if (android.os.Build.VERSION.SDK\_INT >= android.os.Build.VERSION\_CODES.LOLLIPOP) {
BatteryManager batteryManager = (BatteryManager) context.getSystemService(Context.BATTERY\_SERVICE);
currentNow = batteryManager.getIntProperty(BatteryManager.BATTERY\_PROPERTY\_CURRENT\_NOW);
}
// 判断是否在充电
if (plugged == BatteryManager.BATTERY\_PLUGGED\_AC || plugged == BatteryManager.BATTERY\_PLUGGED\_USB || plugged == BatteryManager.BATTERY\_PLUGGED\_WIRELESS) {
// 在充电
if (voltage != -1 && currentNow != -1) {
float voltageInVolts = voltage / 1000f; // 将电压转换为伏特
float currentInAmperes = currentNow / 1000000f; // 将电流转换为安培
float chargingPower = voltageInVolts * currentInAmperes; // 计算充电功率（瓦特）
CLog.i(String.format("充电功率: %.2fW", chargingPower));
if (Math.abs(chargingPower) > 300) {
CLog.e("充电功率过高");
handlerItemData(new ListItemBean(
"电池异常:充电功率过高(可能是云手机)",
ListItemBean.RiskLeave.Deadly,
"检测到过大的充电功率 -> " + String.format("%.2fW", Math.abs(chargingPower))
));
}
}
}
}
};
```

### 检测摄像头&传感器相关：

判断摄像头有个数 。

```
try {
CameraManager manager = (CameraManager) context.getSystemService(Context.CAMERA\_SERVICE);
String[] cameraIds = manager.getCameraIdList();
//摄像头个数
CLog.i("cameraIds -> "+ Arrays.toString(cameraIds));
if(cameraIds.length < CAMERA\_MINIMUM\_QUANTITY\_LIMIT){
items.add(
new ListItemBean(
"当前手机可能是模拟器&云手机",
ListItemBean.RiskLeave.Warn,
"camera size -> "+cameraIds.length
));
}
} catch (Throwable ignored) {
}
```

### 检测传感器个数：

这块思路就是直接获取个数，少于10个可以直接认定为黑产 。 我目前没发现那个手机少于10个传感器 ，这块如果可能的话可以尝试调用一下传感器，保证传感器是否可用 ，防止云手机以假乱真 。

```
try {
//3,检测传感器类型,支持的全部类型传感器
SensorManager sm = (SensorManager) context.getSystemService(SENSOR\_SERVICE);
List<Sensor> sensorlist = sm.getSensorList(Sensor.TYPE\_ALL);
ArrayList<Integer> sensorTypeS = new ArrayList<>();
for (Sensor sensor : sensorlist) {
//获取传感器类型
int type = sensor.getType();
if (!sensorTypeS.contains(type)) {
//发现一种类型则添加一种类型
sensorTypeS.add(type);
}
}
//小米k40 51个传感器类型
//普通的pix 27个
//华为荣耀20 18个传感器
CLog.e("sensor types size -> " + sensorlist.size());
//我们认为传感器少于20个则认为是风险设备
if (sensorlist.size() < SENSOR\_MINIMUM\_QUANTITY\_LIMIT) {
items.add(new ListItemBean(
"当前手机可能是模拟器&云手机",
ListItemBean.RiskLeave.Warn,
"sensor size -> ("+ sensorlist.size()+") \n" +
"sensor type size -> ("+sensorTypeS.size()+") \n"
//+ "sensor info -> \n"+ Sensorlist //打印全部传感器信息
));
}
```

### 检测传感器名称：

这块检测思路主要是检测传感器的名称，正常小米之类的手机他是不可能存在叫什么 AOSP的传感器的 。

这种AOSP基本都是自己编译的ROM ，所以这块也可以作为监测点 。可以上报传感器的一些名称信息，也是环境检测一个很重要的抓手 。

一般小白肯定不会说去改传感器名称 。

```
if (Build.VERSION.SDK\_INT >= Build.VERSION\_CODES.N) {
ArrayList<Sensor> aospSensor = new ArrayList<>();
for(Sensor sensor:sensorlist){
if(sensor.getVendor().contains("AOSP")){
aospSensor.add(sensor);
}
}
if (aospSensor.size()>3) {
CLog.e("传感器参数是否异常(生产厂商为AOSP)");
items.add(new ListItemBean(
"当前手机可能是模拟器&云手机",
ListItemBean.RiskLeave.Warn,
aospSensor.size()
+"/"+sensorlist.size()+"传感器参数异常 -> "+ aospSensor
));
}
}
```

### 检测挂载文件：

这块就是去遍历mounts 下面这几个文件，检测里面是否包含docker关键字 ，防止一些云手机搞虚拟化，通过使用docker进行挂载 。

这块也是很好的监测点 。

```
String[] marks = {"docker"};
//检测proc/mounts是否包含docker关键字
String mark = NativeEngine.getZhenxiInfoK("/proc/mounts",marks );
if(mark == null){
mark = NativeEngine.getZhenxiInfoK("/proc/self/mountstats", marks);
if(mark == null){
mark = NativeEngine.getZhenxiInfoK("/proc/self/mountinfo", marks);
}
}
if(mark!=null){
items.add(new ListItemBean(
"当前手机可能是模拟器&云手机",
ListItemBean.RiskLeave.Warn,
"(mounts异常)\n"+mark
));
}
```

### 检测ROM是否Match:

#### 检测环境信息：

这块思路主要好多种 ，主要是为了防止一些自定义ROM ，通过修改机型的方法，绕过自定义ROM检测逃逸 。

可以直接执行getprop 把所有的环境信息都拿到手 ，如果是小米手机，里面环境信息里面，肯定是有MIUI关键字。

比如小米的手机，我会去检测是否包含这几个关键环境信息 。

```
private static final String KEY\_MIUI\_VERSION\_NAME = "ro.miui.ui.version.name";
private static final String KEY\_MIUI\_VERSION\_CODE = "ro.miui.ui.version.code";
private static final String KEY\_MIUI\_INTERNAL\_STORAGE = "ro.miui.internal.storage";
```

这块可以采集以后服务端进行判断 。防止自定义ROM 机型伪造 。

#### 检测服务列表：

这块还是执行 service list ，一般小米手机之类的，都会有小米的系统服务，这种东西很难去伪造，如果他伪造了假的 。你就尝试调用即可 。

这块还是建议上传到服务端，由服务端算法同学去根据相似度算法去推断 ，不要再本地进行判断 ，因为Hunter是非联网Apk，所以只是在客户端打了个样子 。

## 检测当前环境是否被Hook:

这块检测方法千奇百怪，首先最基本maps去检测frida或者根据调用栈检测lsp特征 ，基础的检测方案不说了 。因为我觉得并不是一个很好的方案 。改个名就绕过了 。

比如frida特征三件套 。检测思路主要

```
static const char *FRIDA\_THREAD\_GUM\_JS\_LOOP = "gum-js-loop";
static const char *FRIDA\_THREAD\_GMAIN = "gmain";
static const char *FRIDA\_NAMEDPIPE\_LINJECTOR = "linjector";
```

Hook检测，我们其实只需要检测内存没有被修改即可 。这块需要介绍一下基础原理。和实现的伪代码 。

正常我们知道一个SO加载到内存里，本质上是通过mmap把so分配到内存里面 ，比如A函数的指令是BBB，那么加载到内存里面应该也是BBB 。
