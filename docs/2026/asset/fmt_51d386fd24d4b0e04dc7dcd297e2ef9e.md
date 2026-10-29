
`Looper`类

```java
public static void loop() {
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;
    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();
    for (;;) {
        Message msg = queue.next(); // might block
        if (msg == null) {
            return;
        }

        Printer logging = me.mLogging;
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }

        msg.target.dispatchMessage(msg);

        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }

        // Make sure that during the course of dispatching the
        // identity of the thread wasn't corrupted.
        final long newIdent = Binder.clearCallingIdentity();
        if (ident != newIdent) {
            Log.wtf(TAG, "Thread identity changed from 0x"
                    + Long.toHexString(ident) + " to 0x"
                    + Long.toHexString(newIdent) + " while dispatching to "
                    + msg.target.getClass().getName() + " "
                    + msg.callback + " what=" + msg.what);
        }

        msg.recycle();
    }
}
```

### 【3】`handleMessage(msg)`进行消息处理

`Handler`类

```java
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}
```

### 【4】通过消息类型分别调用`handleLaunchActivity(r, null)`[Activity流程代码]和`handleBindApplication(data)`[Application流程代码]

`ActivityThread$H`类

```java
public void handleMessage(Message msg) {
    if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));
    switch (msg.what) {
        case LAUNCH_ACTIVITY: {
            Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityStart");
            ActivityClientRecord r = (ActivityClientRecord)msg.obj;

            r.packageInfo = getPackageInfoNoCheck(
                    r.activityInfo.applicationInfo, r.compatInfo);
            handleLaunchActivity(r, null);
            Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        } break;
        case RELAUNCH_ACTIVITY: {
            Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityRestart");
            ActivityClientRecord r = (ActivityClientRecord)msg.obj;
            handleRelaunchActivity(r);
            Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        } break;
        ...
        case BIND_APPLICATION:
            Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "bindApplication");
            AppBindData data = (AppBindData)msg.obj;
            handleBindApplication(data);
            Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
            break;
    }
}
```

### 【5】Activity调用流程

`handleLaunchActivity(r, null)`-->`performLaunchActivity(r, customIntent)`-->`mInstrumentation.callActivityOnCreate(activity, r.state)`-->`activity.performCreate(icicle)`-->`onCreate(icicle)`

注意，此时调用到的`onCreate`方法就是我们平常写Android代码时候的Activity的`onCreate`方法

![图片描述](https://bbs.kanxue.com/upload/attach/202208/941979_B7978ZVTKC78R3A.png)

### 【6】`Application`的调用流程

`handleBindApplication(data)`-->`data.info.makeApplication(data.restrictedBackupMode, null)`-->`mActivityThread.mInstrumentation.newApplication(cl, appClass, appContext)`-->`app.attach(context)`-->`attachBaseContext(context)`

`handleBindApplication(data)`-->`mInstrumentation.callApplicationOnCreate(app)`-->`app.onCreate()`

可以分析到`Application`最终是分别调用了两个方法`attachBaseContext`和`onCreate`，此处的`onCreate`和`Activity`的`onCreate`是两码事

### 【7】小结

综上分析可以得出，在真正调用到`Activity`的`onCreate`方法之前，还会经过两个关键的函数，就是`Application`的`attachBaseContext`和`onCreate`方法

【可以做什么？】

可以在`Application`调用的函数中进行真正的dex代码的加载替换。达到加壳的目的，因为在调用`Activity`之前回调用`Application`的两个函数，所以在此期间将真正程序的代码给替换进来即可。

# 二、ClassLoader的加载流程-找脱壳点

【前言】

了解Android的`ClassLoader`的继承关系

![图片描述](https://bbs.kanxue.com/upload/attach/202208/941979_FG8UQJFGAKXB4GD.png)

## 1.Android4.4 Dalvik模式下加载Dex流程

### 【1】从`DexClassLoader`开始入手分析，因为加载Dex文件一般调用`DexClassLoader`创建对象，发现该类的构造方法调用了父类类构造

【文件目录】libcore

【类名】`DexClassLoader`

【构造方法】`public DexClassLoader(String dexPath, String optimizedDirectory, String libraryPath, ClassLoader parent)`

【父类构造】`super(dexPath, new File(optimizedDirectory), libraryPath, parent);`

继承自父类`BaseDexClassLoader`(`pathClassLoader`也是继承自此父类)

父类构造方法

![图片描述](https://bbs.kanxue.com/upload/attach/202208/941979_PHXTNFDY4CRCGU5.png)

### 【2】`BaseDexClassLoader`是一个关键的类，很多的实现方法都封装于此类，调用构造向父类传递了类加载器的父类

【文件目录】libcore

【类名】`BaseDexClassLoader`

【构造方法】`public BaseDexClassLoader(String dexPath, File optimizedDirectory, String libraryPath, ClassLoader parent)`

【父类构造】`super(parent);`

![图片描述](https://bbs.kanxue.com/upload/attach/202208/941979_9GCEBUYRPCA22TX.png)

### 【3】`ClassLoader`唯一的作用就是指定了父类

【文件目录】libcore

【类名】`ClassLoader`

【构造方法】`protected ClassLoader(ClassLoader parentLoader)`

【初始化】`ClassLoader(ClassLoader parentLoader, boolean nullAllowed){ parent = parentLoader;}`

![图片描述](https://bbs.kanxue.com/upload/attach/202208/941979_46W73MYZ643RMD5.png)

### 【4】`BaseDexClassLoader`创建`DexPathList`对象

【构造方法初始化】`this.pathList = new DexPathList(this, dexPath, libraryPath, optimizedDirectory);`

【类成员】`private final DexPathList pathList;`

![图片描述](https://bbs.kanxue.com/upload/attach/202208/941979_6HBDCY4CQ64F2UF.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202208/941979_9DAN25Z4E97GF7K.png)

### 【5】`DexPathList`构造函数

【文件目录】libcore

【类名】`DexPathList`

【构造方法】`public DexPathList(ClassLoader definingContext, String dexPath, String libraryPath, File optimizedDirectory)`

【成员变量】`private final Element[] dexElements;`

5.1 对类加载器、dex文件路径、文件优化路径等参数基本判断

5.2 调用方法`makeDexElements`对成员变量赋值`dexElements`

![图片描述](https://bbs.kanxue.com/upload/attach/202208/941979_TH5NKRBR8ZBJ86B.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202208/941979_TB3QWWXVUVGVY4H.png)

### 【6】分析`makeDexElements`方法

【方法原型】`private static Element[] makeDexElements(ArrayList<File> files, File optimizedDirectory`

6.1 判断文件名后缀

6.2 `loadDexFile`加载dex后缀的文件

![图片描述](https://bbs.kanxue.com/upload/attach/202208/941979_64J6H55NEAZVZMA.png)

6.3 根据加载进来的dex文件`new Element`对象，并且赋值到成员变量中

6.4 返回局部变量`Element`对象数组

![图片描述](https://bbs.kanxue.com/upload/attach/202208/941979_7GN2TBVNVTKXBX6.png)

### 【7】分析`loadDexFile`方法

【方法原型】`private static DexFile loadDexFile(File file, File optimizedDirectory)`

7.1 如果不存在优化后的dex文件，则直接创建`DexFile`类对象

7.2 如果存在优化后的dex文件，则调用`DexFile`类的`loadDex`方法加载dex文件

![图片描述](https://bbs.kanxue.com/upload/attach/202208/941979_CX5PG2G2ZR5NX35.png)

7.3 这里对于优化后的文件，以及dex如何被优化的，在稍后有专门描述该内容

### 【8】分析`DexFile`类构造方法

【文件目录】libcore

【类名】`DexFile`

【构造方法】`public DexFile(String fileName) throws IOException`

【成员变量】`private int mCookie;`

8.1 调用方法`openDexFile`获取Cookie值

![图片描述](https://bbs.kanxue.com/upload/attach/202208/941979_9D5KV48RZ2MQDY9.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202208/941979_QUW6S7FKWGTEU4Y.png)

### 【9】分析`openDexFile`方法

【参数说明】：参数1：dex路径名；参数2：优化文件路径名

【函数说明】：调用`openDexFileNative`方法进入c++代码，并且返回`cookie`值

![图片描述](https://bbs.kanxue.com/upload/attach/202208/941979_3F3QT2H837ZA463.png)

### 【10】进入native层的`Dalvik_system_DexFile.cpp`文件分析

【文件目录】dalvik

【文件名】`dalvik_system_DexFile`

【方法】`static void Dalvik_dalvik_system_DexFile_openDexFileNative(const u4 *args, JValue* pResult)`

10.1 java层进入native层，获得参数，并且转换为c的类型格式

![图片描述](https://bbs.kanxue.com/upload/attach/202208/941979_Y9PNMUZB95RGHPK.png)

10.2 `dvmClassPathContains`判断dex文件路径是否当前类加载器加载或者存在的

![图片描述](https://bbs.kanxue.com/upload/attach/202208/941979_5KBX6MMPFV8YNEA.png)

10.3 `hasDexExtension`如果以dex结尾返回true

10.4 `dvmRawDexFileOpen`打开没有被优化的dex文件

10.5 `pDexorJar`管理dex文件内部结构
