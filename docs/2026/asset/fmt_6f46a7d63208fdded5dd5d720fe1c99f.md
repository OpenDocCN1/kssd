
```java
public static int checkPackage(Context context) {

    int i = 0;

    PackageManager packageManager = context.getPackageManager();

    try {

        packageManager.getInstallerPackageName("de.robv.android.xposed.installer");

        i = 1;

    } catch (Exception e) {

    }

    try {

        packageManager.getInstallerPackageName("com.saurik.substrate");

        return i | 2;

    } catch (Exception e2) {

        return i;

    }

}
```

## 检测/proc/mypid/maps

这里我对Linux的proc文件系统没有研究过的童鞋非常简单的搜一下盲.

首先proc文件系统的设计目的之一就是允许更方便的对进程信息进行访问.

每当一个进程创建的时候,/proc目录下就会有和该进程id对应的目录产生.

目录名称就是进程id.里面记录该进程的各种信息.其中maps记录了进程的

内存信息,更具体的说内存分段信息.具体什么含义我就不说了,反正我们可以知道它可以记录加载了那些模块就OK了.如下图

![](https://bbs.kanxue.com/upload/attach/201904/793792_4EHU6BBHH5MGUQD.png)

对Xposed有研究过的人应该都知道,xposed会对进程注入这些模块:

```
XposedBridge.jar

libxposed_art.so

app_process32_xposed
```

所以我们检测这些模块是否存在也是可以检测出xposed和substrate的.

```java
public static int checkMap() throws Throwable {

    UnsupportedEncodingException unsupportedEncodingException;

    BufferedReader bufferedReader;

    Throwable th;

    int i = 0;

    BufferedReader bufferedReader2;

    int i2;

    int result = 0;

    try {

        HashSet hashSet = new HashSet();

        bufferedReader2 = new BufferedReader(new InputStreamReader(new FileInputStream("/proc/" + android.os.Process.myPid() + "/maps"), "utf-8"));

        while (true) {

            try {

                String readLine = bufferedReader2.readLine();

                if (readLine == null) {

                    break;

                } else if (readLine.endsWith(".so") || readLine.endsWith(".jar")) {

                    hashSet.add(readLine.substring(readLine.lastIndexOf(" ") + 1));

                }

            } catch (UnsupportedEncodingException e) {

                unsupportedEncodingException = e;

                i2 = 0;

                bufferedReader = bufferedReader2;

                try {

                    unsupportedEncodingException.printStackTrace();

                    if (bufferedReader != null) {

                    }

                } catch (Throwable th2) {

                    th = th2;

                    bufferedReader2 = bufferedReader;

                    if (bufferedReader2 != null) {

                    }

                    throw th;

                }

            }

        }

        Iterator it = hashSet.iterator();

        while (it.hasNext()) {

            int i3;

            Object next = it.next();

            if (((String) next).toLowerCase().contains("xposed")) {

                result = result | 64;

            }

            if (((String) next).toLowerCase().contains("com.saurik.substrate")) {

                result = result | 128;

            }
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
    return result;
}
```

代码逻辑就是一行一行的读`/proc/mypid/mpas`.看是否包含xposed或com.saurik.substrate.

## 检测堆栈信息

先看图.

![](https://bbs.kanxue.com/upload/attach/201904/793792_NFBR7DTBUJ2RS96.png)

啊啊,我们只要检测堆栈是否包含红色圈圈de.robv.android.xposed.XposedBridge就可以了.

看代码

```java
public static int checkStackTraceElement() {
    int i = 0;
    try {
        throw new Exception("detect hook");
    } catch (Exception e) {
        int i2 = 0;
        for (StackTraceElement stackTraceElement : e.getStackTrace()) {
            if (stackTraceElement.getClassName().equals("de.robv.android.xposed.XposedBridge") && stackTraceElement.getMethodName().equals("main")) {
                i2 |= 4;
            }
            if (stackTraceElement.getClassName().equals("de.robv.android.xposed.XposedBridge") && stackTraceElement.getMethodName().equals("handleHookedMethod")) {
                i2 |= 8;
            }
            if (stackTraceElement.getClassName().equals("com.saurik.substrate.MS$2") && stackTraceElement.getMethodName().equals("invoked")) {
                i2 |= 16;
            }
            if (stackTraceElement.getClassName().equals("com.android.internal.os.ZygoteInit")) {
                i++;
                if (i == 2) {
                    i2 |= 32;
                }
            }
        }
        return i2;
    }
}
```

其实我们可以依据该方法检测自己的方法是否被Hook了.道理是一样的.就是检测堆栈是否包含有afterHookedMethod,beforeHookedMethod.

## 检测ActivityManagerNative

话说这个我不清楚啥原理.有懂得留言告诉我一下呗.

```java
public static int checkActivityManagerNative() {
    try {
        Method method = Class.forName("android.app.ActivityManagerNative").getMethod("getDefault", new Class[0]);
        method.setAccessible(true);
        if (method.invoke(null, new Object[0]).getClass().getName().startsWith("$Proxy")) {
            return 256;
        }
        return 0;
    } catch (Exception e) {
        return 256;
    }
}
```

## 模拟器检测

```java
public static String getVMDesc()
{
    StringBuilder stringBuilder = new StringBuilder();
    String VM = AdbShell.getprop("ro.genymotion.version");//判断genymotion模拟器
    if (VM != null) {
        stringBuilder.append("ro.genymotion.version");
        stringBuilder.append("|");
        stringBuilder.append(VM);
        stringBuilder.append("\n");
    }
    VM = AdbShell.getprop("androVM.vbox_dpi");//判断使用了vbox的模拟器,目前很多市面上的安卓模拟器都是基于vbox的
    if (VM != null) {
        stringBuilder.append("androVM.vbox_dpi");
        stringBuilder.append("|");
        stringBuilder.append(VM);
        stringBuilder.append("\n");
    }
    VM = AdbShell.getprop("qemu.sf.fake_camera");//检测安卓自身的模拟器
    if (VM != null) {
        stringBuilder.append("qemu.sf.fake_camera");
        stringBuilder.append("|");
        stringBuilder.append(VM);
    }
    return stringBuilder.toString();
}
```

模拟器都具有一些特殊的属性.查找这个特殊的属性就可以判断是否是模拟器.

其中`AdbShell.getprop` 等效于 `Systemproperties.get(name)`.

## 检测Debug

一个是检测debuggable标志.一个是检测TracerPid.

```java
//检测是否拥有调试属性
public static String HaveDebugProp()
{
    //ro.debuggable表示调试权限,默认为0,1表示可以调试
    StringBuilder builder = new StringBuilder();
    builder.append("ro.debuggable");
    builder.append(AdbShell.getprop("ro.debuggable"));
    return builder.toString();
}

//如果进程被调试TracerPid不为0
public static String getTracerPid()
{
    BufferedReader bufferedReader;
    String readLine = "";
    try {
        bufferedReader = new BufferedReader(new FileReader("/proc/self/status"));
        do{
            readLine = bufferedReader.readLine();
            if (readLine == null) {
                break;
            }
        }while (!readLine.startsWith("TracerPid:"));
        readLine = readLine.substring(10).trim();
    } catch (Exception e) {
        e.printStackTrace();
    }
    return readLine;
}
```

## Root检测

一个是检测是否有su文件.一个是检测属性.
