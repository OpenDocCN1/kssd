
```java
//android 8.0
public BaseDexClassLoader(String dexPath, File optimizedDirectory,
        String librarySearchPath, ClassLoader parent) {
    this(dexPath, optimizedDirectory, librarySearchPath, parent, false);
}

public BaseDexClassLoader(String dexPath, File optimizedDirectory,
        String librarySearchPath, ClassLoader parent, boolean isTrusted) {
    super(parent);
    this.pathList = new DexPathList(this, dexPath, librarySearchPath, null, isTrusted);

    if (reporter != null) {
        reportClassLoaderChain();
    }
}
```

`BaseDexClassLoader`的构造函数并没有把`optimizedDirectory`传递给`DexPathList`。

## 3. 初始化`ClassLoader`

### 3.1. 调用链路

```text
ActivityThread.performLaunchActivity //启动activity
├─ ActivityThread.createBaseContextForActivity //创建ContextImpl，并作为app的Context实现
│    └─ ContextImpl.createActivityContext //从LoadedApk获取ClassLoader，并创建ContextImpl
│        └─ LoadedApk.getClassLoader //判断是否已有ClassLoader，如无则调用createOrUpdateClassLoaderLocked
│            └─ LoadedApk.createOrUpdateClassLoaderLocked
│                └─ ApplicationLoaders.getClassLoader
│                    └─ ClassLoaderFactory.createClassLoader
│                        └─ new PathClassLoader
└─ ContextImpl.getClassLoader //LoadedApk.getClassLoader已经生成了ClassLoader，并存于ContextImpl的mClassLoader中
```

### 3.2. 详细源码

#### 3.2.1. `ActivityThread.performLaunchActivity`

```java
package android.app;

public final class ActivityThread extends ClientTransactionHandler {
    /**
     * Core implementation of activity launch.
     */
    private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
        //...省略，chinamima
        // ==========>>> 创建ContextImpl <<<==========
        ContextImpl appContext = createBaseContextForActivity(r);
        Activity activity = null;
        try {
            //从ContextImpl里获取ClassLoader
            java.lang.ClassLoader cl = appContext.getClassLoader();
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
            //...省略，chinamima
        } catch (Exception e) {
            //...省略，chinamima
        }

        try {
            Application app = r.packageInfo.makeApplication(false, mInstrumentation);
            //...省略，chinamima
            if (activity != null) {
                //...省略，chinamima
                appContext.setOuterContext(activity);
                activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window, r.configCallback);
                //...省略，chinamima
                if (r.isPersistable()) {
                    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
                } else {
                    mInstrumentation.callActivityOnCreate(activity, r.state);
                }
                //...省略，chinamima
                r.activity = activity;
            }
            mActivities.put(r.token, r);
        } catch (SuperNotCalledException e) {
            throw e;
        } catch (Exception e) {
            //...省略，chinamima
        }

        return activity;
    }
}
```

#### 3.2.2. `ActivityThread.createBaseContextForActivity`

```java
package android.app;

public final class ActivityThread extends ClientTransactionHandler {

    private ContextImpl createBaseContextForActivity(ActivityClientRecord r) {
        int displayId = ActivityManager.getService().getActivityDisplayId(r.token);
        //...省略，chinamima
        // ==========>>> 生成ContextImpl实例，里面会生成一个默认ClassLoader成员变量 <<<==========
        ContextImpl appContext = ContextImpl.createActivityContext(
                this, r.packageInfo, r.activityInfo, r.token, displayId, r.overrideConfig);
        //...省略，chinamima
        return appContext;
    }
}
```

#### 3.2.3. `ContextImpl.createActivityContext`

```java
package android.app;

class ContextImpl extends Context {
    static ContextImpl createActivityContext(ActivityThread mainThread,
            LoadedApk packageInfo, ActivityInfo activityInfo, IBinder activityToken,
            int displayId, Configuration overrideConfiguration) {
        //...省略，chinamima
        // ==========>>> 从LoadedApk里获取ClassLoader <<<==========
        ClassLoader classLoader = packageInfo.getClassLoader();
        //...省略，chinamima
        //创建一个ContextImpl实例
        ContextImpl context = new ContextImpl(null, mainThread, packageInfo, activityInfo.splitName,
                activityToken, null, 0, classLoader);
        //...省略，chinamima
        return context;
    }
}
```

#### 3.2.4. `LoadedApk.getClassLoader`

```java
package android.app;

public final class LoadedApk {

    private ClassLoader mClassLoader;

    public ClassLoader getClassLoader() {
        synchronized (this) {
            if (mClassLoader == null) {
                // ==========>>> 关键步奏 <<<==========
                createOrUpdateClassLoaderLocked(null /*addedPaths*/);
            }
            return mClassLoader;
        }
    }
}
```

调用`createOrUpdateClassLoaderLocked`。

#### 3.2.5. `LoadedApk.createOrUpdateClassLoaderLocked`

```java
package android.app;

public final class LoadedApk {

    private void createOrUpdateClassLoaderLocked(List<String> addedPaths) {
        //...省略，chinamima

        // If we're not asked to include code, we construct a classloader that has
        // no code path included. We still need to set up the library search paths
        // and permitted path because NativeActivity relies on it (it attempts to
        // call System.loadLibrary() on a classloader from a LoadedApk with
        // mIncludeCode == false).
        if (!mIncludeCode) {
            if (mClassLoader == null) {
                //...省略，chinamima
                // ==========>>> 关键步奏 <<<==========
                mClassLoader = ApplicationLoaders.getDefault().getClassLoader(
                        "" /* codePath */, mApplicationInfo.targetSdkVersion, isBundledApp,
                        librarySearchPath, libraryPermittedPath, mBaseClassLoader,
                        null /* classLoaderName */);
                //...省略，chinamima
            }

            return;
        }
        //...省略，chinamima
    }
}
```

调用`ApplicationLoaders.getClassLoader`获取`ClassLoader`。

#### 3.2.6. `ApplicationLoaders.getClassLoader`

