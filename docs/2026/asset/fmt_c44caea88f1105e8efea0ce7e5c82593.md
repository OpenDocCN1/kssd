
`mPackages`是`ActivityThread`的一个非静态成员变量，在函数中的定义如下

```
ArrayMap<String,WeakReference<LoadedApk>>
```

我们通过获取该成员变量就能获取`LoadedApk`对象，进而就能获取到`LoadedApk`的非静态成员变量`mApplication`，最后将其替换成源dex文件的`Application`实例对象

```
/**
*(1)获取壳apk的ActivityThread实例对象的的非静态成员变量mPackages
*mPackages: ArrayMap<String,WeakReference<LoadedApk>> mPackages
*再修改对应的LoadedApk实例对象的非静态成员变量mApplication为被源dex文件的类Application
*java代码： android.util.ArrayMap arrayMap=activityThread.mPackages;
* LoadedApk objLoadedApk=arrayMap.get("thePackagename");
*
*/
//获取android.util.ArrayMap的成员变量mPackages的对象
jobject arraymap_mPackages =(*env)->GetObjectField(env, theCurrentActivityThread, mPackages);
//获取壳apk包名字符串
jstring thePackagename=(*env)->NewStringUTF(env,mPackageName);
LOGI("mPackageName %s",mPackageName);
// 调用arrayMap_get函数 获取WeakReference<LoadedApk>类
jobject v9 = (*env)->CallObjectMethod(env, arraymap_mPackages, ArrayMap_get, thePackagename);
//调用WeakReference_get获取LoadedApk类
jobject v15 = (*env)->CallObjectMethod(env, v9, WeakReference_get);
//修改LoadedApk中的非静态成员变量mApplication为源dex的Application
(*env)->SetObjectField(env, v15, LoadedApk_mApplication, onCreateObj);
```

#### (2) `mInitialApplicaiton`(`Application`类型)

```
/**
*(2)修改壳apk应用的ActivityThread实例对象的成员变量mInitialApplication
*为源dex文件的Application对象实例
*
*
*/
(*env)->SetObjectField(env, theCurrentActivityThread, mInitialApplication,onCreateObj);
```

#### (3) `info.mApplication`

修改壳apk应用的`ActivityThread`实例对象的成员变量`mBoundApplication`中的`LoadApk`非静态成员变量`info`实例对象的实例成员变量`mApplication`为源dex的`Application`对象

```
/**
*(3)修改壳apk应用的ActivityThread实例对象的成员变量mBoundApplication中的LoadApk非静态成员变量info实例对象的
*实例成员变量mApplication为源dex的Application对象
*AppBindData mBoundApplication;
*AppBindData{LoadedApk info;}
*LoadedApk{Application mApplication}
*/
// AppBindData mBoundApplication;
jobject v16 = (*env)->GetObjectField(env, theCurrentActivityThread, mBoundApplication);
//AppBindData_info=(*env)->GetFieldID(env, AppBindData, "info", "Landroid/app/LoadedApk;");
jobject v17 =(*env)->GetObjectField(env, v16, AppBindData_info);
(*env)->SetObjectField(env, v17, LoadedApk_mApplication, onCreateObj);
```

#### (4) `mAllApplication`中的所有`Application`对象

```
/**
*(4)遍历壳apk应用的ActivityThread实例对象的非静态成员变量mAllApplications中的类Application实例对象
*将壳apk的StubApplication替换为源dexapplication实例对象
*StubApplication-->Application
*ArrayList<Application> mAllApplications
*/
//获取allApplications
jobject v11 = (*env)->GetObjectField(env, theCurrentActivityThread, mAllApplications);
//获取ArrayList的大小
int count = (*env)->CallIntMethod(env, v11, arraylist_size);
LOGI("array_size:%d",count);
int index = 0;
while ( index < count )
{
//调用mAllApplication(ArrayList)的get方法获取每一个元素ArrayList.get(index)
jobject v12 = (*env)->CallObjectMethod(env, v11, arraylist_get, index);
LOGI("compare: i=%d item=%p", index, v12);
//在类对象mApplication实例中查找壳apk应用的StubApplication类对象
if ( ((*env)->IsSameObject)(env, obj, v12) == 1 )//是否等于StubApplication类的对象
{
LOGI("replace: find same replace");
//调用ArrayList的set方法替换源Application的实例对象
(*env)->CallObjectMethod(env, v11, arraylist_set, index,onCreateObj);
}
// _JNIEnv::DeleteLocalRef(env, v12);
++index;
}
//IDA反编译这里出错，多了一个参数
(*env)->CallVoidMethod(env, onCreateObj, Application_onCreate);
(*env)->DeleteGlobalRef(env, onCreateObj);
}
```

**本文主要对早期的阿里加固方式进行了一些梳理**

# 加固原理

## 前言

最近一直在分析安卓加固的方案，对很多知识都不太懂，一个偶然的机会在论坛上看到**寒号鸟二代**师傅的一篇仿ali加固一个早期的加固方案（https://bbs.pediy.com/thread-215078.htm），于是花了一段时间来分析这个项目实现流程，遂写下这篇文章，希望对学习jni编程和安卓dex文件运行流程的师傅们有帮助，同时也欢迎各位师傅们提出建议。

完成分析之后，我使用AS3.5的环境下编写了一个加固项目，项目地址如下：8e2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9P5X3R3I4z5o6V1%4x3U0j5I4y4e0l9#2x3g2)9J5c8W2y4Z5k6h3I4D9b7$3!0V1k6e0q4Q4c8f1k6Q4b7V1y4Q4z5p5y4d9k6h3q4V1L8h3g2Q4x3X3g2E0k6q4!0q4z5g2)9^5y4#2)9^5b7#2!0q4z5g2)9&6c8q4!0m8x3W2!0q4y4g2)9&6x3q4!0m8b7W2!0q4y4W2)9&6b7#2)9^5z5g2!0q4y4q4!0n7c8q4!0n7c8W2!0q4y4#2)9&6y4q4!0m8z5q4!0q4y4W2)9&6y4W2!0n7z5g2!0q4y4W2!0n7x3#2)9&6y4g2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4W2!0m8b7#2!0m8x3W2!0q4z5q4!0n7c8W2)9^5c8g2!0q4y4g2)9&6x3q4)9^5y4q4!0q4y4q4!0n7c8q4)9^5c8q4y4@1j5i4u0Q4c8e0g2Q4z5e0u0Q4z5p5y4r3L8%4u0C8

## 一、java层

java层仅有有一个`StubApplication.java`类，该类加载的`onCreate`方法和`attachBaseContext`方法均放在了native层

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202007/845506_85KW2AAYC5UWEYE.jpg)

## 二、native层

### 2.动态注册

首先自然是对`attachBaseContext`和`onCreate`进行了动态注册

```
// 生成jni函数注册的jni函数table

static JNINativeMethod method_table[] = {

    // 被注册的函数的名称（java函数名称）、被注册函数的签名（javah获取）、

    // 被注册函数native实现的函数指针

    { "attachBaseContext", "(Landroid/content/Context;)V", (void*)native_attachContextBaseContext},

    { "onCreate","()V",(void*)native_onCreate},

};





static int registerNativeMethods(JNIEnv* env, const char* className,JNINativeMethod* gMethods, int numMethods)

{

    jclass clazz;

    clazz = (*env)->FindClass(env, className);

    if (clazz == 0) {

        return JNI_FALSE;

    }



    LOGI("gMethods  %s,%s,%p\n ",gMethods[0].name,gMethods[0].signature,gMethods[0].fnPtr);



    if ((*env)->RegisterNatives(env, clazz, gMethods, numMethods) < 0) {

        return JNI_FALSE;

    }

    return JNI_TRUE;

}



int register_ndk_load(JNIEnv *env)

{

    // 对类"com/example/unpack/StubApplication"的函数attachBaseContext和onCreate进行注册

    return registerNativeMethods(env, JNIREG_CLASS,

            method_table, NELEM(method_table));

}



JNIEXPORT jint JNI_OnLoad(JavaVM* vm, void* reserved)

{



    JNIEnv* env = 0;

    jint result = -1;

    // LOGI("JNI_OnLoad is called");

    if ((*vm)->GetEnv(vm, (void**) &env, JNI_VERSION_1_4) != JNI_OK) {

        return result;

    }



    int status=register_ndk_load(env);

    if(!status)

    {

        LOGI("register call failed");

    }



    return JNI_VERSION_1_4;

}

```

### 2.attachBaseContext

阅读安卓相关源码可以知道，`Application`的`attachBaseContext`方法是早于`onCreate`方法执行的：

#### (1). 使用`init_class`和`myFindClass`函数对可能用到的函数和成员变量进行初始化，以及对安卓虚拟机进行判断

##### (a) `myFindClass`封装好了获取指定类的全局引用：

```
// 获取指定名称类的全局引用

jclass myFindClass(JNIEnv *env,jclass* ptr,char* name)

{

     jobject g_cls_string;

     jclass clazz = (*env)->FindClass(env,name);

     if(clazz)

     {

         //// 获取类的全局引用(class)

           g_cls_string = (*env)->NewGlobalRef(env,clazz);

        //将全局引用(class)存到ptr中

           *ptr=g_cls_string;myFindClass

          return g_cls_string;

     }

     else

     {

           return 0;

     }



}
```

`init_class`与`myFindClass`结合初始化所有可能用到的变量，并对安卓虚拟机运行模式进行判断:

##### (b)首先获取当前的SDK版本判断是否能够进行相关的初始化操作

```
// 获取类android.os.Build.VERSION的全局引用(class)

  if( !myFindClass(env,&Build_version,"android/os/Build$VERSION"))

  {

    LOGI("ERROR:Build$VERSION");

    return;

  }

  //通过Build_VERSION获取当前SDK版本的ID，并通过ID调用获取当前的SDK版本

  jfieldID fieldID= ((*env)->GetStaticFieldID)(env, Build_version, "SDK_INT", "I");

  sdk_int=(*env)->GetStaticIntField(env,Build_version,fieldID);

  LOGI("sdk_int %d\n",sdk_int);  

  if(sdk_int>18){



  }
```

##### (c)对可能用到的成员变量和方法进行初始化，同时考虑到不同的sdk版本android源码存在的细微的差异，在初始化时，会对版本进行判断
