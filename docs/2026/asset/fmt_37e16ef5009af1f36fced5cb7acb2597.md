
## 对应的按钮事件

```
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
    public void onClick(View view) {
        Intent intent = new Intent(this, Main2Activity.class);
        startActivity(intent);
    }
}
```

`activity_main2.xml`

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_margin="5dp"
    android:padding="5dp"
>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
    >
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"
            android:text="第二个Activity"
            android:textSize="20dp" />
    </LinearLayout>
</LinearLayout>
```

编译运行， 功能正常

3. 编译程序，得到apk, 使用Android Killer/ apktool反编译得到项目

```
java -jar apktool_latest.jar d app-debug.apk
```

在逆向工程中把 `Main2Activity.smali` 单独提取出来， 删除其他所有不需要的 `smali` 文件

![image-20220418203555018](upload/attach/202208/890847_M7RKQKHJU6JEWS3.jpg)

将仅含`Main2Activity.smali`的项目工程重新编译为dex文件

```
java -jar smali.jar ./smali_classes3
```

将生成的`out.dex`改名为 `Main2Activity.dex`

4. 在`main`文件夹 右键--> new Directory: `assets` （自定义资源目录） --> 把`mytest.dex` 复制过来

![image-20220418204309400](upload/attach/202208/890847_YNGJ8FQXYAN7DKM.jpg)

然后`Main2Activity.java` 就没用了，直接删除即可（注意不能在IDE中删除，这样会把`AndroidManifest`中的`Activity`声明给删除）, 这样就保持了相同包名和项目结构

## 动态加载dex文件

确认总体代码功能

1. 拷贝自定义资源（`assets`）的`dex`到程序目录下

2. 创建一个`DexClassLoader`， 加载`dex`

3. 调用加载`dex`中的`Activity`

`MainActivity.java`

```
public class MainActivity extends AppCompatActivity {
    private static final String TAG = "ayuan-log";
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Log.d(TAG, "onCreate: MainActivity启动");
    }
    public void onClick(View view) {
        loadDex();
    }
    private void loadDex() {
        Log.d(TAG, "loadDex: 开始加载");
        //1. 拷贝自定义资源（assets）的dex到程序目录下
        String dexPath = copyDex("Main2Activity.dex");
        //2. 创建一个加载了该Dex文件的Dex类加载器, 可以反射获取其中的类和函数
        DexClassLoader dexClassLoader = getLoader(dexPath);
        //3. 通过反射， 启动dex中的Activity
        execClassMethod(
                dexClassLoader, // classLoader
                "com.example.myapplication.Main2Activity", //类名
                "onCreate", // 方法名
                this); // 参数: MainActivity.this
    }
    private void execClassMethod(DexClassLoader dexClassLoader, // 类所属的ClassLoader
                                 String className, // 函数所处的类名
                                 String methodName, // 函数名
                                 Activity as ) { // 函数参数, 此处传入了MainActivity.this 用于显示
        try{
            // 类类型
            Class clazz = dexClassLoader.loadClass(className);
            Intent intent = new Intent(MainActivity.this, clazz);
            startActivity(intent);
            // 调用构造函数(无参构造)， 获取实例 getConstructor(参数类型数组)
            // Constructor cons = clazz.getConstructor(new Class[]{});
            // 调用构造方法创建对象， newInstance(参数数组)
            // Object instance = cons.newInstance(new Object[]{});
            // 获取成员方法
            // Method methodTest = clazz.getDeclaredMethod(methodName, new Class[]{Activity.class});
            // 取消java 访问检查
            // methodTest.setAccessible(true);
            // 调用方法 invoke(实例, 方法参数数组)
            // methodTest.invoke(instance, new Object[]{as});
        }catch(Exception e){
            e.printStackTrace();
        }
    }
    // 创建一个加载了该Dex文件的Dex类加载器, 可以反射获取其中的类和函数
    private DexClassLoader getLoader(String dexPath) {
        DexClassLoader dexClassLoader = new DexClassLoader(
                dexPath, // 要加载的Dex文件路径
                getCacheDir().toString(), // 优化之后的文件路径
                null, // native 库路径， 由于没有使用到native，null即可
                getClassLoader() // 父ClassLoader
        );
        Log.d(TAG, "getLoader: "+dexClassLoader);
        return dexClassLoader;
    }
    // 拷贝assets/文件 到 app/files/ 下
    private String copyDex(String dexName) {
        // 获取assets目录管理器
        AssetManager as = getAssets();
        // 目的地址
        String path = getFilesDir() + File.separator + dexName;
        // /data/user/0/com.example.myapplication/files/Main2Activity.dex
        Log.d(TAG, "copyDex: path: " + path);
        // 将文件拷贝到目的地址
        try {
            // 创建文件流
            FileOutputStream out = new FileOutputStream(path);
            // 打开文件 assets/dexName
            InputStream is = as.open(dexName);
            // 循环读取并写入文件
            byte[] buffer = new byte[1024];
            int len =0;
            while((len = is.read(buffer))!= -1){
                out.write(buffer, 0, len);
            }
            // 关闭文件
            out.close();
        }catch(Exception e){
            e.printStackTrace();
            return "";
        }
        return path;
    }
}
```

此时编译运行会报错， 无法实例化`Main2Activity`

```
Unable to instantiate activity ComponentInfo{com.example.myapplication/com.example.myapplication.Main2Activity}:
java.lang.ClassNotFoundException: Didn't find class "com.example.myapplication.Main2Activity" on path: DexPathList[[zip file "/data/app/com.example.myapplication-1/base.apk"]
```

这是因为虽然我们用自己的`DexClassLoader`加载了`Main2Activity`, 但是`startActivity`等系统函数依然无法实例化`Main2Activity`, 系统函数是从程序的`PathClassLoader`中寻找并加载类的

解决方法：

* 直接使用`PathClassLoader`加载我们需要的类

* 使用`ClassLoader`替换掉`PathClassLoader`

分析Android源码， 利用java的反射机制修改保存的`PathClassLoader`

* 把自己的`DexClassLoader` 改为 `PathClassLoader`的父`ClassLoader`

树状结构 `BootClassLoader` <-- `PathClassLoader` <-- 自定义的`DexClassLoader`

## 分析apk启动的过程

### 程序启动过程分析

**参考**

安卓源码分析: 应用程序的启动过程 <mark class="encrypted">aceK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8V1I4#2L8%4y4Z5k6h3&6Y4P5h3q4F1k6#2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4U0j5^5z5e0M7@1z5l9`.`.</mark>

整个应用程序的启动过程要执行很多步骤，但是整体来看，主要分为以下五个阶段：

1. Step1 - Step 11：`Launcher`通过`Binder`进程间通信机制通知`ActivityManagerService`，它要启动一个`Activity`；

   1. `Launcher.startActivitySafely`： 点击应用图标时，`Launcher`就会对应的应用程序启动起来

   2. `Activity.startActivity`： 调用`startActivityForResult`来进一步处理

   3. `Activity.startActivityForResult`：定义成员变量 `Activity.Intrumentation` ： 监控应用程序和系统的交互； `Activity.mMainThread`： `Launcher`应用程序运行的进程； 调用`Instrumentation.execStartActivity` 来进一步处理

   4. `Instrumentation.execStartActivity`：调用`ActivityManagerProxy.startActivity`

   5. `ActivityManagerProxy.startActivity`： 通过`Binder`调用`ActivityManagerService.startActivity`

   6. `ActivityManagerService.startActivity`：调用`mMainStack.startActivityMayWait`

   7. `ActivityStack.startActivityMayWait`: 定义`ActivityInfo aInfo` 局部变量保存`MainActivity`的信息

   8. `ActivityStack.startActivityLocked`: 定义`ActivityRecord r` 创建要启动的`Activity`的信息

   9. `ActivityStack.startActivityUncheckedLocked`: 根据`launchMode`和`Stack` 新建`Task`

   10. `Activity.resumeTopActivityLocked` 根据栈顶`Activity`是否为本`Activity`来启动`Activity`

   11. `ActivityStack.startPausingLocked`: `Launcher`进入`Paused`状态

2. Step 12 - Step 16：`ActivityManagerService`通过`Binder`进程间通信机制通知`Launcher`进入`Paused`状态；

   1. `ApplicationThreadProxy.schedulePauseActivity`: 通过`Binder`进程间通信机制进入到`ApplicationThread.schedulePauseActivity`函数

   2. `ApplicationThread.schedulePauseActivity`: 调用`queueOrSendMessage`

   3. `ActivityThread.queueOrSendMessage`： 整理信息， 发送给`ActivityThread.H.handleMessage`

   4. `ActivityThread.H.handleMessage`:

   5. `ActivityThread.handlePauseActivity`: 做了三个事情：1. 通过调用`performUserLeavingActivity`函数来`Activity`用户要离开它了；2. 调用`performPauseActivity`函数来调用`Activity.onPause`函数；3. 它通知`ActivityManagerService`，这个`Activity`已经进入`Paused`状态了，`ActivityManagerService`现在可以完成未竟的事情，即启动`MainActivity`了

3. Step 17 - Step 24：`Launcher`通过`Binder`进程间通信机制通知`ActivityManagerService`，它已经准备就绪进入`Paused`状态，于是`ActivityManagerService`就创建一个新的进程，用来启动一个`ActivityThread`实例，即将要启动的`Activity`就是在这个`ActivityThread`实例中运行；

   1. `ActivityManagerProxy.activityPaused`:

   2. `ActivityManagerService.activityPaused`

   3. `ActivityStack.activityPaused`

   4. `ActivityStack.completePauseLocked`

   5. `ActivityStack.resumeTopActivityLokced`

   6. `ActivityStack.startSpecificActivityLocked` 检查是否已经有以`process + uid`命名的进程存在

   7. `ActivityManagerService.startProcessLocked`： 调用`Process.start`接口来创建一个新的进程，新的进程会导入`android.app.ActivityThread`类，并且执行它的`main`函数

   8. `ActivityThread.main` 这个函数在进程中创建一个`ActivityThread`实例，然后调用它的`attach`函数，接着就进入消息循环了，直到最后进程退出。`attach`函数调用`ActivityManagerProxy.attachApplication`函数

4. Step 25 - Step 27：`ActivityThread`通过`Binder`进程间通信机制将一个`ApplicationThread`类型的`Binder`对象传递给`ActivityManagerService`，以便以后`ActivityManagerService`能够通过这个`Binder`对象和它进行通信；

   1. `ActivityManagerProxy.attachApplication` 通过`Binder`驱动程序，最后进入`ActivityManagerService.attachApplication`函数中

   2. `ActivityManagerService.attachApplication` 转发给`attachApplicationLocked`

