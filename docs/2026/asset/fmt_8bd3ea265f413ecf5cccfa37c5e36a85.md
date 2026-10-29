
#### （2）读取可能用到的文件路径（so、assets 等）

通过 Native 层的反射，生成相关的文件夹，存放源 APK 所含有的所有文件，同时获取必要的 APK 包名等信息，并构建临时内存空间的字符串。

```c
/**
 * 通过 init_class 生成的 ContextWrapper 和获取的 attachBaseContext
 * 调用父类的方法 android.content.ContextWrapper.attachBaseContext(ctx)
 */

(*env)->CallNonvirtualVoidMethod(env, application_obj, myContextWrapper, ContextWrapper_attachBaseContext, ctx);

/**
 * 首先通过 obj 和 env 获取 StubApplication 类，再通过其获取 getFilesDir() 方法。
 * 然后调用 getFileDir() 方法获取能得到文件路径的 File 实例对象。
 * 再调用 File 对象获取 File 类的 getAbsolutePath() 方法，获取到文件路径的对象，
 * 再将其转化成 C 语言字符串。
 * StubApplication-->getFilesDir()-->File(obj)-->File(class)-->getAbsolutePath()-->AbsolutePath(obj)-->AbsolutePath(jstring)
 */

jclass v12 = (*env)->GetObjectClass(env, application_obj); // 获取 StubApplication 类

jmethodID v13 = (*env)->GetMethodID(env, v12, "getFilesDir", "()Ljava/io/File;"); // 获取 getFilesDir() 方法

// 调用 File.getFilesDir() 方法，该方法返回 /data/data/<packagename>/files 的 File 实例对象
jobject file_obj = (*env)->CallObjectMethod(env, obj, v13);

// file_obj(obj)-->file_classz(class)，通过 File 对象获取 File 类
jclass file_classz = (*env)->GetObjectClass(env, file_obj);

// 获取 File 类的 getAbsolutePath 方法 ID
jmethodID v18 = (*env)->GetMethodID(env, file_classz, "getAbsolutePath", "()Ljava/lang/String;");

// 调用 File.getAbsolutePath() 获取文件路径 /data/data/<packagename>/files
jobject mAbsolutePath = (*env)->CallObjectMethod(env, file_obj, v18);

// 6.0 下为 /data/user/0/packagename/files/ 目录

// 将文件路径对象转化成字符串
mAbsolutePath_str = (*env)->GetStringUTFChars(env, mAbsolutePath, 0);

LOGI("global files path is %s", mAbsolutePath_str);

// return ApplicationInfo

/**
 * 调用一些初始化的方法获取外壳 APK 的 so 文件路径。
 * 首先根据当前 application_obj 调用 Context 的 getApplicationInfo 方法获取 ApplicationInfo 实例对象，
 * 然后通过 Application 对象的 nativeLibraryDir 方法获取 so 文件路径的对象，
 * 最后将 so 文件路径对象转化成 C 语言字符串，存放到 mNativeLibraryDir 对象中。
 * context.getApplicationInfo()-->ApplicationInfo(obj)-->ApplicationInfo.nativeLibraryDir()-->v24(obj)-->mNativeLibraryDir(const char)
 */

// 调用 getApplicationInfo 方法获取当前 ApplicationInfo
jobject ApplicationInfo = (*env)->CallObjectMethod(env, application_obj, context_getApplicationInfo);

// 获取 so 文件路径对象
jobject v24 = (*env)->GetObjectField(env, ApplicationInfo, ApplicationInfo_nativeLibraryDir);

// 获取外壳 APK 的 so 文件路径
const char* mNativeLibraryDir = (*env)->GetStringUTFChars(env, v24, 0);

// LOGI("mNativeLibraryDir is %s", mNativeLibraryDir);

/**
 * 调用一些初始化的方法 ID 获取 APK 的资源文件路径，然后指定外壳 APK 的进程 APKPATH 作为资源存放路径。
 * 调用 StubApplication 的 getPackageResourcePath() 获取资源文件路径对象，然后将其转化成 char，
 * 再把它设置为 APKPATH 的值，设置为 APK 的资源文件存放路径。
 * StubApplication.getPackageResourcePath()-->v32(obj)-->mPackageResourePath(char*)->setenv("APKPATH", char*, 1)
 */

// 调用 StubApplication 的 getPackageResourcePath() 获取资源文件路径对象
jobject v32 = (*env)->CallObjectMethod(env, application_obj, context_getPackageResourePath);

// 将资源文件路径对象转化成 char*
const char* mPackageResourePath = (*env)->GetStringUTFChars(env, v32, 0);

// 将环境变量 "APKPATH" 的值设置为 APK 的资源文件存放路径
setenv("APKPATH", mPackageResourePath, 1);

// LOGI("APK Path is %s", mPackageResourePath);

/**
 * 调用 Context 的 getPackageName 方法获取 APK 包名。
 * mPackageName 接受包名字符串。
 */

// 调用 Context 的 getPackageName() 方法，获取 APK 包名对象
jobject v36 = (*env)->CallObjectMethod(env, ctx, context_getPackageName);

// 获取 APK 包名字符串（jstring）
mPackageName = (*env)->GetStringUTFChars(env, v36, 0);

LOGI("mPackageName:%s", mPackageName);

/**
 * 调用 Context 的 getClassLoader() 方法获取 APK 当前加载的 ClassLoader，
 * 以便替换 mCookie。
 */

// public ClassLoader getClassLoader()

// 调用 Context 的 getClassLoader() 方法，获取当前 APK 加载使用的 ClassLoader 实例
jobject classLoader = (*env)->CallObjectMethod(env, ctx, context_getClassLoader);

LOGI("classLoader:%p", classLoader);

/**
 * 调用获取到的 mPackageName，从 /data/data/<packagename>/files 目录下释放出 dump.dex。
 */

char szPath[260] = {0};

// sprintf(szPath, "%s/dump.dex", mAbsolutePath_str);

// 使用获取 /data/data/<packagename>/files/dump.dex 的完整路径
sprintf(szPath, "/data/data/%s/files/dump.dex", mPackageName);

LOGI("szPath:%s", szPath);

/**
 * 将 /data/data/<packagename>/assets/dump.dex 文件释放到内存中，再写入到
 * /data/data/<packagename>/files/dump.dex 文件中。
 */
```

#### （3）为 DEX 文件分配临时的内存空间

该函数调用资源管理器对象，打开 APK 资源文件目录下的源 DEX 文件，将其进行解密操作，并把解密后的 DEX 文件存放到临时目录中。

```c
myExtractFile(env, application_obj, szPath);
```

```c
void myExtractFile(JNIEnv* env, jobject application_obj, const char* szPath)
{
    // szPath=/data/data/%s/files/dump.dex
    AAssetManager* mgr; // 声明资源管理器对象 mgr

    // 判断 szPath 路径下是否已经存在 dump.dex 文件
    if (access(szPath, R_OK))
    {
        // 通过 application 对象获取 application 类
        jclass v19 = (*env)->GetObjectClass(env, application_obj);

        // 通过 application 类获取 getAssets() 方法的 ID
        jmethodID v20 = (*env)->GetMethodID(
            env,
            v19,
            "getAssets",
            "()Landroid/content/res/AssetManager;"
        );

        // 调用 getAssets() 方法返回值给 v22(obj)
        jobject v22 = (*env)->CallObjectMethod(env, application_obj, v20);

        // 给定一个 Dalvik AssetManager 对象，获取相应的本地 AAssetManager 对象
        mgr = AAssetManager_fromJava(env, v22);

        if (mgr == NULL)
        {
            // 获取失败
            LOGI("%s", "AAssetManager==NULL");
            return;
        }

        // 打开 APK 资源文件夹下的 dump.dex
        // 即 data/data/<packagename>/assets/dump.dex
        AAsset* asset = AAssetManager_open(
            mgr,
            "dump.dex",
            AASSET_MODE_STREAMING
        );

        FILE* file;
        void* buffer;
        int numBytesRead;

        if (asset)
        {
            // 以 szPath 的字符串路径创建新的 file(dump.dex)，位于 /data/data/<packagename>/files/ 目录下
            file = fopen(szPath, "w");

            // int bufferSize = AAsset_getLength(asset);
            // LOGI("buffersize is %d", bufferSize);

            // 申请 1024 字节临时空间
            buffer = malloc(1024);

            while (1)
            {
                // 将资源文件目录下的 dump.dex 文件读取出来，放到申请的临时空间 buffer 中
                numBytesRead = AAsset_read(asset, buffer, 1024);

                // 读取完成，结束 while 循环
                if (numBytesRead <= 0)
                {
                    break;
                }

                // 将内容写入 /data/data/<packagename>/files/file 文件中
                fwrite(buffer, numBytesRead, 1, file);
            }

            free(buffer);
            fclose(file);
            AAsset_close(asset);
        }
        else
        {
            LOGI("%s", "Error AAssetManager_open");
            return;
        }
    }
    else
    {
        LOGI("%s", "dump.dex existed");
    }
}
```

#### （4）加载源 DEX 文件

加载模式一共有两种：ART 模式和 DVM 模式，详细原理将在下面说明。

```c
if (isDalvik)
{
    myLoadDex_dvm(env, szPath);
}
else
{
    myLoadDex_art(env, szPath);
}
```
