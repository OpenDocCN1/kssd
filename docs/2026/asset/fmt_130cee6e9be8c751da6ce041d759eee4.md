
# [原创]ART/Dalvik环境下Xposed实现异同分析

> 写了一部分，先占个坑，后续补完

## 1. ART/Dalvik环境异同

## 2. Xposed底层实现逻辑变更

了解到`ART/Dalvik`的异同之后，我们再来看看我们的主题，当`Android虚拟机`从`Dalvik`到`ART`，`Xposed`针对这种变化对其底层实现逻辑做了哪些变更呢？

首先从源头入手，`Xposed`框架在编译`app_main`文件时做了系统适配处理，针对不同版本的系统使用不同的`app_main`文件

```makefile
# Android.mk
# PLATFORM_SDK_VERSION字段
# 参考xref: /build/core/version_defaults.mk
# PLATFORM_VERSION := 7.1.2
# PLATFORM_SDK_VERSION := 25
# 根据系统版本不同，7以上的版本使用app_main2.cpp来编译
ifeq (1,$(strip $(shell expr $(PLATFORM_SDK_VERSION) \>= 21)))
LOCAL_SRC_FILES := app_main2.cpp
LOCAL_MULTILIB := both
LOCAL_MODULE_STEM_32 := app_process32_xposed
LOCAL_MODULE_STEM_64 := app_process64_xposed
else
LOCAL_SRC_FILES := app_main.cpp
LOCAL_MODULE_STEM := app_process_xposed
endif
##########################################################
# Library for Dalvik-/ART-specific functions
##########################################################
# 针对libxposed_xxx.cpp文件也是同样的道理
ifeq (1,$(strip $(shell expr $(PLATFORM_SDK_VERSION) \>= 21)))
include frameworks/base/cmds/xposed/ART.mk
else
include frameworks/base/cmds/xposed/Dalvik.mk
endif
```

那么下面我们分析下这两份文件的实现

### 2.1 app_main文件的实现变更

![](upload/attach/202104/878835_HMMXBCQRYQ26FM5.png)

可以看到，在两份文件中针对`AppRuntime`类的修改大致是相同的，同样都是调用了`xposed::onVmCreated`

```c
// xposed.cpp
/** Load the libxposed_*.so library for the currently active runtime. */
// 同样都是调用了xposed.cpp的onVmCreated函数
void onVmCreated(JNIEnv* env) {
// Determine the currently active runtime
// 初始化xposed lib路径
const char* xposedLibPath = NULL;
// 轮询/proc/self/maps结果，查看系统到底是art还是dalvik再决定赋值
// xposed.h
// #define XPOSED_LIB_DALVIK XPOSED_LIB_DIR "libxposed_dalvik.so"
// #define XPOSED_LIB_ART XPOSED_LIB_DIR "libxposed_art.so"
if (!determineRuntime(&xposedLibPath)) {
ALOGE("Could not determine runtime, not loading Xposed");
return;
}
// 通过dlopen so文件，so文件由之前的determineRuntime函数配置
// Load the suitable libxposed_*.so for it
void* xposedLibHandle = dlopen(xposedLibPath, RTLD_NOW);
if (!xposedLibHandle) {
ALOGE("Could not load libxposed: %s", dlerror());
return;
}
// Clear previous errors
dlerror();
// Initialize the library
// dlsym 执行so文件中xposedInitLib函数，so文件就是上面被赋值的文件
bool (*xposedInitLib)(XposedShared* shared) = NULL;
*(void **) (&xposedInitLib) = dlsym(xposedLibHandle, "xposedInitLib");
if (!xposedInitLib) {
ALOGE("Could not find function xposedInitLib");
return;
}
// xposedInitLib函数如下，赋予xposed的onVmCreated为onVmCreatedCommon
// 也就是libxposed_common.cpp的onVmCreatedCommon方法
// bool xposedInitLib(XposedShared* shared) {
// xposed = shared;
// xposed->onVmCreated = &onVmCreatedCommon;
// return true;
// }
#if XPOSED_WITH_SELINUX
xposed->zygoteservice_accessFile = &service::membased::accessFile;
xposed->zygoteservice_statFile = &service::membased::statFile;
xposed->zygoteservice_readFile = &service::membased::readFile;
#endif // XPOSED_WITH_SELINUX
// 方法返回true，执行onVmCreatedCommon方法，到目前为止，流程都是相同的
if (xposedInitLib(xposed)) {
xposed->onVmCreated(env);
}
}
```

```c
// libxposed_common.cpp
// 两份文件公有部分
void onVmCreatedCommon(JNIEnv* env) {
// 判断xposed_bridge.jar是否正常载入以及zygote服务启动是否正常
if (!initXposedBridge(env) || !initZygoteService(env)) {
return;
}
// 判断虚拟机是否正常启动，这里调用onVmCreated就是libxposed_xx.so文件中自己的onVmCreated函数了，这里应该就是第一个变化点
if (!onVmCreated(env)) {
return;
}
// 多重判断之后，返回xposed已正确安装标识
xposedLoadedSuccessfully = true;
return;
}
bool initXposedBridge(JNIEnv* env) {
// #define CLASS_XPOSED_BRIDGE "de/robv/android/xposed/XposedBridge"
// 通过FindClass获取XposedBridge类，判断xposed_bridge类是否正常加载
classXposedBridge = env->FindClass(CLASS_XPOSED_BRIDGE);
if (classXposedBridge == NULL) {
ALOGE("Error while loading Xposed class '%s':", CLASS_XPOSED_BRIDGE);
logExceptionStackTrace();
env->ExceptionClear();
return false;
}
// PS：这里没有理解做了什么
classXposedBridge = reinterpret_cast<jclass>(env->NewGlobalRef(classXposedBridge));
ALOGI("Found Xposed class '%s', now initializing", CLASS_XPOSED_BRIDGE);
// xposed_bridge的函数是否通过RegisterNatives动态注册成功
if (register_natives_XposedBridge(env, classXposedBridge) != JNI_OK) {
ALOGE("Could not register natives for '%s'", CLASS_XPOSED_BRIDGE);
logExceptionStackTrace();
env->ExceptionClear();
return false;
}
// 判断通过jni获取handleHookedMethod函数是否成功
methodXposedBridgeHandleHookedMethod = env->GetStaticMethodID(classXposedBridge, "handleHookedMethod",
"(Ljava/lang/reflect/Member;ILjava/lang/Object;Ljava/lang/Object;[Ljava/lang/Object;)Ljava/lang/Object;");
if (methodXposedBridgeHandleHookedMethod == NULL) {
ALOGE("ERROR: could not find method %s.handleHookedMethod(Member, int, Object, Object, Object[])", CLASS_XPOSED_BRIDGE);
logExceptionStackTrace();
env->ExceptionClear();
return false;
}
return true;
}
int register_natives_XposedBridge(JNIEnv* env, jclass clazz) {
// 动态注册XposedBridge相关的函数，形式如XposedBridge_hookMethodNative
const JNINativeMethod methods[] = {
NATIVE_METHOD(XposedBridge, hadInitErrors, "()Z"),
NATIVE_METHOD(XposedBridge, getStartClassName, "()Ljava/lang/String;"),
NATIVE_METHOD(XposedBridge, getRuntime, "()I"),
NATIVE_METHOD(XposedBridge, startsSystemServer, "()Z"),
NATIVE_METHOD(XposedBridge, getXposedVersion, "()I"),
NATIVE_METHOD(XposedBridge, initXResourcesNative, "()Z"),
// 关键函数，之后在findandhook中会涉及
NATIVE_METHOD(XposedBridge, hookMethodNative, "(Ljava/lang/reflect/Member;Ljava/lang/Class;ILjava/lang/Object;)V"),
NATIVE_METHOD(XposedBridge, setObjectClassNative, "(Ljava/lang/Object;Ljava/lang/Class;)V"),
NATIVE_METHOD(XposedBridge, dumpObjectNative, "(Ljava/lang/Object;)V"),
NATIVE_METHOD(XposedBridge, cloneToSubclassNative, "(Ljava/lang/Object;Ljava/lang/Class;)Ljava/lang/Object;"),
NATIVE_METHOD(XposedBridge, removeFinalFlagNative, "(Ljava/lang/Class;)V"),
#if PLATFORM_SDK_VERSION >= 21
NATIVE_METHOD(XposedBridge, invokeOriginalMethodNative,
"!(Ljava/lang/reflect/Member;I[Ljava/lang/Class;Ljava/lang/Class;Ljava/lang/Object;[Ljava/lang/Object;)Ljava/lang/Object;"),
NATIVE_METHOD(XposedBridge, closeFilesBeforeForkNative, "()V"),
NATIVE_METHOD(XposedBridge, reopenFilesAfterForkNative, "()V"),
#endif
#if PLATFORM_SDK_VERSION >= 24
NATIVE_METHOD(XposedBridge, invalidateCallersNative, "([Ljava/lang/reflect/Member;)V"),
#endif
};
return env->RegisterNatives(clazz, methods, NELEM(methods));
}
```

#### 2.1.1 onVmCreated函数变更

到目前为止可以看到关于`xposed::onVmCreated`的调用流程最终走到了两份文件自己的`onVmCreated`方法，下面来看看这两个方法的差异

##### 2.1.1.1 Dalvik时期的onVmCreated
