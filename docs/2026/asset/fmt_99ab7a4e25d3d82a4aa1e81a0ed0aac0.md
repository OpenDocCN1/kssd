
# 移动安全学习笔记（三）

## Hook

### Xposed 插件开发

#### 环境配置

环境配置较为繁琐，分为以下步骤

- 复制 `XposedBridgeApi-82.jar` 到工程中供使用

切换至 `Project` 模式，在 `app` 目录下新建文件夹 `lib`，将 `XposedBridgeApi-82.jar` 复制到 `app/lib` 文件夹下

- 配置依赖

- 右键工程 — `Open Module Setting` — `Dependencies` — `app` — `Declared Dependencies` — 点击加号 — `JAR/ARR Dependencies`

- Step 1： `lib/XposedBridgeApi-82.jar`

- Step 2： `compileOnly` — `OK`

- 新建 `Empty Activity` 并在 `AndroidManifest.xml` 中添加代码

```xml
<meta-data android:name = "xposedmodule" android:value="true"/>
<meta-data android:name = "xposeddescription" android:value="Xposed模块示例"/>
<meta-data android:name = "xposedminversion" android:value="54"/>
```

- 新建入口类 `Main.java` 并实现 `IXposedHookLoadPackage` 接口

```java
package com.example.xposeddemo;
import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.callbacks.XC_LoadPackage;
public class Main implements IXposedHookLoadPackage {
@Override
public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {
}
}
```

- 复制入口类名

右键入口类 `Main` — `Copy Path/Reference` — `Copy Reference`

- 配置入口类名文件

`app/src/main` 文件夹下新建文件夹 `assets`，`app/src/main/assets` 文件夹下新建文件 `xposed_init`，将复制的入口类名粘贴在文件中即可

#### Hook函数

想要 `Hook` 某一个函数则需要得到该函数的三点关键信息

- 包名： `com.example.a1`
- 类名： `android.telephony.TelephonyManager`
- 方法原型： `public String getDeviceId()`

##### 示例一

使用 `replaceHookedMethod` 方法 `Hook` `TelephonyManager.getDeviceId()` 函数

`Main.java`

```java
package com.example.xposeddemo;
import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XC_MethodReplacement;
import de.robv.android.xposed.XposedHelpers;
import de.robv.android.xposed.callbacks.XC_LoadPackage;
import android.telephony.TelephonyManager;
public class Main implements IXposedHookLoadPackage {
@Override
public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {
// 包名： com.example.a1
// 类名： android.telephony.TelephonyManager
// 方法原型： public String getDeviceId()
String packageName = loadPackageParam.packageName;
if(!packageName.equals("com.example.a1"))
return;
XposedHelpers.findAndHookMethod(
TelephonyManager.class,
"getDeviceId",
new XC_MethodReplacement() {
@Override
protected Object replaceHookedMethod(MethodHookParam methodHookParam) throws Throwable {
return "123456789";
}
});
}
}
```

`findAndHookMethod` 函数还有另一种重载方式

`Main.java`

```java
package com.example.xposeddemo;
import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XC_MethodReplacement;
import de.robv.android.xposed.XposedHelpers;
import de.robv.android.xposed.callbacks.XC_LoadPackage;
import android.telephony.TelephonyManager;
public class Main implements IXposedHookLoadPackage {
@Override
public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {
// 包名： com.example.a1
// 类名： android.telephony.TelephonyManager
// 方法原型： public String getDeviceId()
String packageName = loadPackageParam.packageName;
if(!packageName.equals("com.example.a1"))
return;
XposedHelpers.findAndHookMethod(
"android.telephony.TelephonyManager",
loadPackageParam.classLoader,
"getDeviceId",
new XC_MethodReplacement() {
@Override
protected Object replaceHookedMethod(MethodHookParam methodHookParam) throws Throwable {
return "123456789";
}
});
}
}
```

##### 示例二

`Hook` 程序自身实现的函数

`Main.java`

```java
package com.example.xposeddemo;
import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XC_MethodReplacement;
import de.robv.android.xposed.XposedHelpers;
import de.robv.android.xposed.callbacks.XC_LoadPackage;
public class Main implements IXposedHookLoadPackage {
@Override
public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {
// 包名： com.example.a2
// 类名： com.example.a2.MainActivity
// 方法原型： private boolean check(String str1, String str2)
String packageName = loadPackageParam.packageName;
if(!packageName.equals("com.example.a2"))
return;
XposedHelpers.findAndHookMethod(
"com.example.a2.MainActivity",
loadPackageParam.classLoader,
"check",String.class,String.class,
new XC_MethodReplacement() {
@Override
protected Object replaceHookedMethod(MethodHookParam methodHookParam) throws Throwable {
return true;
}
});
}
}
```

`Hook` 程序自身实现的函数还可以使用更加强大的 `XC_MethodHook` 方法，它内部需要实现两个方法，可以做到修改参数和修改返回值

`Main.java`

```java
package com.example.xposeddemo;
import android.util.Log;
import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XC_MethodHook;
import de.robv.android.xposed.XposedBridge;
import de.robv.android.xposed.XposedHelpers;
import de.robv.android.xposed.callbacks.XC_LoadPackage;
public class Main implements IXposedHookLoadPackage {
@Override
public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {
// 包名： com.example.a2
// 类名： com.example.a2.MainActivity
// 方法原型： private boolean check(String str1, String str2)
String packageName = loadPackageParam.packageName;
if(!packageName.equals("com.example.a2"))
return;
XposedHelpers.findAndHookMethod(
"com.example.a2.MainActivity",
loadPackageParam.classLoader,
"check", String.class, String.class,
new XC_MethodHook() {
@Override
protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
super.beforeHookedMethod(param);
// 打印参数
Log.d("lxz","arg1:" + param.args[0]);
Log.d("lxz","arg2:" + param.args[1]);
// xposed输出日志
XposedBridge.log("arg1:" + param.args[0]);
XposedBridge.log("arg2:" + param.args[1]);
// 修改参数
param.args[0] = "lxz";
param.args[1] = "lxz";
}
@Override
protected void afterHookedMethod(MethodHookParam param) throws Throwable {
super.afterHookedMethod(param);
// xposed输出日志
XposedBridge.log("arg1:" + param.args[0]);
XposedBridge.log("arg2:" + param.args[1]);
// 修改返回值
param.setResult(true);
}
});
}
}
```

##### 示例三

实现 `Hook` `Person` 的构造方法

`Main.java`

```java
package com.example.xposeddemo;
import android.util.Log;
import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XC_MethodHook;
import de.robv.android.xposed.XposedBridge;
import de.robv.android.xposed.XposedHelpers;
import de.robv.android.xposed.callbacks.XC_LoadPackage;
public class Main implements IXposedHookLoadPackage {
@Override
public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {
// 包名： com.example.a3
// 类名： com.example.a3.Person
// 方法原型： public Person(String name, int age)
String packageName = loadPackageParam.packageName;
if(!packageName.equals("com.example.a3"))
return;
Class hookClass = XposedHelpers.findClass(
"com.example.a3.Person",loadPackageParam.classLoader);
XposedHelpers.findAndHookConstructor(
hookClass,
String.class, int.class,
new XC_MethodHook() {
@Override
protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
super.beforeHookedMethod(param);
// 打印参数
Log.d("lxz","arg1:" + param.args[0]);
Log.d("lxz","arg2:" + param.args[1]);
// xposed输出日志
XposedBridge.log("arg1:" + param.args[0]);
XposedBridge.log("arg2:" + param.args[1]);
// 修改参数
param.args[0] = "lxz";
param.args[1] = 29;
}
@Override
protected void afterHookedMethod(MethodHookParam param) throws Throwable {
super.afterHookedMethod(param);
}
}
);
}
}
```

##### 示例四

实现对匿名内部类的 `Hook`
