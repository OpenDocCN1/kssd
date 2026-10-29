
## attachBaseContext() 方法

```java
protected void attachBaseContext(Context base) {
    super.attachBaseContext(base);
    try {
        // 创建两个文件夹payload_odex，payload_lib 私有的，可写的文件目录
        String dexPath = Environment.getExternalStorageDirectory() + File.separator + "classes.dex";
        File odex = this.getDir("payload_odex", MODE_PRIVATE);
        File libs = this.getDir("payload_lib", MODE_PRIVATE);
        String odexPath = odex.getAbsolutePath();
        libPath = libs.getAbsolutePath();
        String apkFileName = odex.getAbsolutePath() + "/payload.apk";
        apkFileName = dexPath;
        File dexFile = new File(apkFileName);
        Log.i("demo", "apk size:" + dexFile.length());
        // 配置动态加载环境
        Object currentActivityThread = RefInvoke.invokeStaticMethod(
                "android.app.ActivityThread", "currentActivityThread",
                new Class[] {}, new Object[] {});// 获取主线程对象
        String packageName = this.getPackageName();// 当前apk的包名
        // 下面两句不是太理解
        ArrayMap mPackages = (ArrayMap) RefInvoke.getFieldOjbect(
                "android.app.ActivityThread", currentActivityThread,
                "mPackages");
        WeakReference wr = (WeakReference) mPackages.get(packageName);
        // 创建被加壳apk的DexClassLoader对象  加载apk内的类和本地代码（c/c++代码）
        PathClassLoader dLoader = new PathClassLoader(odexPath, getClassLoader());
        DexClassLoader dLoader = new DexClassLoader(apkFileName, odexPath,
                libPath, (ClassLoader) RefInvoke.getFieldOjbect(
                "android.app.LoadedApk", wr.get(), "mClassLoader"));
        // base.getClassLoader(); 是不是就等同于 (ClassLoader) RefInvoke.getFieldOjbect()? 有空验证下//?
        // 把当前进程的DexClassLoader 设置成了被加壳apk的DexClassLoader ----有点c++中进程环境的意思~~
        RefInvoke.setFieldOjbect("android.app.LoadedApk", "mClassLoader",
                wr.get(), dLoader);

        Log.i("demo","classloader:"+dLoader);

        try{
            ApplicationInfo ai = this.getPackageManager()
                    .getApplicationInfo(this.getPackageName(),
                    PackageManager.GET_META_DATA);
            Bundle bundle = ai.metaData;
            String frastAct = null;
            if (bundle != null && bundle.containsKey("ACTIVITY_MAIN_CLASS_NAME")) {
                frastAct = bundle.getString("ACTIVITY_MAIN_CLASS_NAME");// className 是配置在xml文件中的。
            } else {
                Log.e("demo", "have no application class name");
            }
            Object actObj = dLoader.loadClass(frastAct);
            Log.e("demo", actObj.toString());

        } catch(Exception e) {
            Log.e("demo", "activity:"+Log.getStackTraceString(e));
        }

    } catch (Exception e) {
        Log.i("demo", "error:"+Log.getStackTraceString(e));
        e.printStackTrace();
    }
}
```

## OnCreate() 方法

```java
public void onCreate() {
    Log.i("demo", "onCreate");
    // 如果源应用配置有Appliction对象，则替换为源应用Applicaiton，以便不影响源程序逻辑。
    String appClassName = null;
    try {
        ApplicationInfo ai = this.getPackageManager()
                .getApplicationInfo(this.getPackageName(),
                PackageManager.GET_META_DATA);
        Bundle bundle = ai.metaData;
        if (bundle != null && bundle.containsKey("APPLICATION_CLASS_NAME")) {
            appClassName = bundle.getString("APPLICATION_CLASS_NAME");// className 是配置在xml文件中的。
        } else {
            Log.i("demo", "have no application class name");
            return;
        }
    } catch (NameNotFoundException e) {
        Log.i("demo", "error:"+Log.getStackTraceString(e));
        e.printStackTrace();
    }
    // 有值的话调用该Applicaiton
    Object currentActivityThread = RefInvoke.invokeStaticMethod(
            "android.app.ActivityThread", "currentActivityThread",
            new Class[] {}, new Object[] {});
    Object mBoundApplication = RefInvoke.getFieldOjbect(
            "android.app.ActivityThread", currentActivityThread,
            "mBoundApplication");
    Object loadedApkInfo = RefInvoke.getFieldOjbect(
            "android.app.ActivityThread$AppBindData",
            mBoundApplication, "info");
    // 把当前进程的mApplication 设置成了null
    RefInvoke.setFieldOjbect("android.app.LoadedApk", "mApplication",
            loadedApkInfo, null);
    Object oldApplication = RefInvoke.getFieldOjbect(
            "android.app.ActivityThread", currentActivityThread,
            "mInitialApplication");
    ArrayList<Application> mAllApplications = (ArrayList<Application>) RefInvoke
            .getFieldOjbect("android.app.ActivityThread",
            currentActivityThread, "mAllApplications");
    mAllApplications.remove(oldApplication);// 删除oldApplication

    ApplicationInfo appinfo_In_LoadedApk = (ApplicationInfo) RefInvoke
            .getFieldOjbect("android.app.LoadedApk", loadedApkInfo,
            "mApplicationInfo");
    ApplicationInfo appinfo_In_AppBindData = (ApplicationInfo) RefInvoke
            .getFieldOjbect("android.app.ActivityThread$AppBindData",
            mBoundApplication, "appInfo");
    appinfo_In_LoadedApk.className = appClassName;
    appinfo_In_AppBindData.className = appClassName;
    Application app = (Application) RefInvoke.invokeMethod(
            "android.app.LoadedApk", "makeApplication", loadedApkInfo,
            new Class[] { boolean.class, Instrumentation.class },
            new Object[] { false, null });// 执行 makeApplication（false,null）
    RefInvoke.setFieldOjbect("android.app.ActivityThread",
            "mInitialApplication", currentActivityThread, app);

    ArrayMap mProviderMap = (ArrayMap) RefInvoke.getFieldOjbect(
            "android.app.ActivityThread", currentActivityThread,
            "mProviderMap");
    Iterator it = mProviderMap.values().iterator();
    while (it.hasNext()) {
        Object providerClientRecord = it.next();
        Object localProvider = RefInvoke.getFieldOjbect(
                "android.app.ActivityThread$ProviderClientRecord",
                providerClientRecord, "mLocalProvider");
        RefInvoke.setFieldOjbect("android.content.ContentProvider",
                "mContext", localProvider, app);
    }

    Log.e("demo", "app:"+app);
    app.onCreate();
    super.onCreate();
}
```

以上为最原始代码（各网站均有讲解），这不是关键（但很核心），要把这些放到 C++ 中执行才安全。解密 dex 后，替换 APK 壳的 dex 过程，可以通过 C++ 调用 Java 来实现，如：`env->FindClass("dalvik/system/PathClassLoader");`

如果说用 C++ 调用 Java 实现比较难，那对于想要逆向的人来说会更难。

## 重点中的重点

先了解逆向过程：

`apk->dex->smali`：可修改后打包，实现破解。

### 加强1：签名验证

由于 APK 发布时打包有签名文件，加壳时可把 sha1 和 sha256 的值保存，在要释放解密后的 dex 前，先做签名验证，不通过不对 dex 进行解密。

### 加强2：伪加密

APK 本身可以认为是一个 ZIP 包，打包后可做伪加密（自行百度）。

### 加强3：root 权限判断

在对 dex 解密前可做 root 权限判断，不符合不解密 dex。

### 加强4：代理判断

有的不破解 APK，直接用抓包方式获取访问数据，很多 app 用的是明文，导致请求数据和返回数据直接暴露，可在解密前做代理判断，不符合要求不解密 dex，或直接关闭网络访问代理（方式百度）。

### 加强5：HOOK 检查

同上，也可在解密前先检查是否有 Xposed，不符合不解密。

### 加强6：SO 层反调试

so 作为解密过程，也是可以被单独拿来调试的，可以在 `JNI_OnLoad` 加载时创建一个线程，用于以上几步的操作验证，以及反调试验证，用一个 `while(1)` 一直做检查（中间间隔几秒）。不用担心耗资源，因很多 app 开发者使用大量无用代码相比要少的多，当程序被销毁时，循环也会退出。

**以上操作可实现：root 手机不能运行，装有 Xposed 不能运行，有网络代理的不能运行，加壳后的 APK 被重新打包后也不能运行（签名发生变化）**

在 C++ 解密的过程里可放烟雾弹，方法名相似，代码相似，但没用到，可以让逆向者认为找到了突破口（我用的是 Java 调用 C++，C++ 再调用 Java，Java 再调用 C++）。

DEX 加载器不仅可以加载单独的 dex 文件，也可以加载 jar 或 ZIP 文件，ZIP 文件中可放置多个 dex，大家可自由发挥，目前好像不能放一个文件目录，官方提示可指定为目录而非一个文件。

## 再放经验

安卓目录中如果一个文件或文件夹的名字前是以 `.` 开头的，这个文件或目录视为隐藏文件，大家可自由发挥，不影响加载 DEX 文件。

APK 伪加密，有些破解工具无法正常打开。

最后外加一个拼接 521 字节（自行百度）和资源混淆。

以上所有操作流程可用写个加密工具进行测试。

## 授人以鱼，不如授人以渔

有什么问题和建议可发在留言区，以上操作均已在加壳工具中实现，可在开头找到下载地址。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2022-9-30 13:39 被富到流油^-^编辑，原因：补充

- [#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)
- [#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)
- [#混淆加固](https://bbs.kanxue.com/forum-161-1-121.htm)
- [#脱壳反混淆](https://bbs.kanxue.com/forum-161-1-122.htm)
