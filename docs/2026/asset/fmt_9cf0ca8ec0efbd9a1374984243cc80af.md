
```cpp
//#include <stdio.h>

//#include <unistd.h>

#define LOG_TAG "FileHelper"

#include <string.h>
#include <android/log.h>
#include "jni.h"

//#include "JNIHelp.h"

#include <sys/errno.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <fcntl.h>
#include <signal.h>

//struct passwd *getpwuid(uid_t uid);

//getpwuid函数是通过用户的uid查找用户的passwd数据。如果出错时，它们都返回一个空指针并设置errno的值，用户可以根据perror函数查看出错的信息。

//strcut group * getgrgid(gid_t gid);

//getgrgid（）用来依参数gid指定的组识别码逐一搜索组文件，找到时便将该组的数据以group结构返回。

//group结构请参考getgrent（）。返回group结构数据，如果返回NULL则表示已无数据，或有错误发生。

#include <sys/types.h>
#include <pwd.h>
#include <grp.h>

//int chmod( const char *filename, int pmode );

//改变文件的读写许可设置，如果改变成功返回0，否则返回-1

#include <C:\cygwin\usr\include\io.h>

//int chown(const char * path, uid_t owner, gid_t group);

//改变文件所有者 types.h,unistd.h 成功则返回0, 失败返回-1, 错误原因存于errno.

//　chown()会将参数path 指定文件的所有者变更为参数owner 代表的用户，而将该文件的组变更为参数group 组。

//如果参数owner 或group 为-1，对应的所有者或组不会有所改变。

//root 与文件所有者皆可改变文件组，但所有者必须是参数group 组的成员。

//当root 用chown()改变文件所有者或组时，该文件若具有S_ISUID或S_ISGID 权限，则会清除此权限位，

//此外如果具有S_ISGID 权限但不具S_IXGRP 位，则该文件会被强制锁定,文件模式会保留。

#include <unistd.h>
#include <assert.h>

#if HAVE_ANDROID_OS

#include <sys/ioctl.h>
#include <linux/msdos_fs.h>

#endif

#define li(x...) __android_log_print(ANDROID_LOG_INFO, LOG_TAG, x)

#ifdef __cplusplus

extern "C" {

#endif

namespace android {

/*
static jclass gFileStatusClass;
static jfieldID gFileStatusDevFieldID;
static jfieldID gFileStatusInoFieldID;
static jfieldID gFileStatusModeFieldID;
static jfieldID gFileStatusNlinkFieldID;
static jfieldID gFileStatusUidFieldID;
static jfieldID gFileStatusGidFieldID;
static jfieldID gFileStatusSizeFieldID;
static jfieldID gFileStatusBlksizeFieldID;
static jfieldID gFileStatusBlocksFieldID;
static jfieldID gFileStatusAtimeFieldID;
static jfieldID gFileStatusMtimeFieldID;
static jfieldID gFileStatusCtimeFieldID;

static jclass gPassWordClass;
static jfieldID gPassWordNameFieldID;
static jfieldID gPassWordPasswdFieldID;
static jfieldID gPassWordUidFieldID;
static jfieldID gPassWordGidFieldID;
static jfieldID gPassWordGecosFieldID;
static jfieldID gPassWordDirFieldID;
static jfieldID gPassWordShellFieldID;

static jclass gGroupClass;
static jfieldID gGroupNameFieldID;
static jfieldID gGroupPasswdFieldID;
static jfieldID gGroupGidFieldID;
static jfieldID gGroupMemFieldID;
*/

jint Java_android_os_FileHelper_setPermissions(
        JNIEnv* env,
        jobject clazz,
        jstring file,
        jint mode,
        jint uid,
        jint gid) {
#if HAVE_ANDROID_OS
    const jchar* str = env->GetStringCritical(file, 0);
    String8 file8;

    if (str) {
        file8 = String8(str, env->GetStringLength(file));
        env->ReleaseStringCritical(file, str);
    }

    if (file8.size() <= 0) {
        return ENOENT;
    }

    if (uid >= 0 || gid >= 0) {
        int res = chown(file8.string(), uid, gid);

        if (res != 0) {
            return errno;
        }
    }

    return chmod(file8.string(), mode) == 0 ? 0 : errno;
#else
    return ENOSYS;
#endif
}

jint Java_android_os_FileHelper_getPermissions(
        JNIEnv* env,
        jobject clazz,
        jstring file,
        jintArray outArray) {
#if HAVE_ANDROID_OS
    const jchar* str = env->GetStringCritical(file, 0);
    String8 file8;

    if (str) {
        file8 = String8(str, env->GetStringLength(file));
        env->ReleaseStringCritical(file, str);
    }

    if (file8.size() <= 0) {
        return ENOENT;
    }

    struct stat st;

    if (stat(file8.string(), &st) != 0) {
        return errno;
    }

    jint* array = (jint*)env->GetPrimitiveArrayCritical(outArray, 0);

    if (array) {
        int len = env->GetArrayLength(outArray);

        if (len >= 1) {
            array[0] = st.st_mode;
        }

        if (len >= 2) {
            array[1] = st.st_uid;
        }

        if (len >= 3) {
            array[2] = st.st_gid;
        }
    }

    env->ReleasePrimitiveArrayCritical(outArray, array, 0);

    return 0;
#else
    return ENOSYS;
#endif
}

//Landroid/os/FileHelper;.stat (Ljava/lang/String;Landroid/os/FileHelper$FileStatus;)Z

jboolean Java_android_os_FileHelper_stat(
        JNIEnv* env,
        jobject clazz,
        jstring path,
        jobject fileStatus) {
    li("-------stat");

    static jclass gFileStatusClass =
            env->FindClass("android/os/FileHelper$FileStatus");

    if (gFileStatusClass == NULL) {
        li("Unable to find class android.os.FileHelper$FileStatus");
        return false;
    }

    jfieldID gFileStatusDevFieldID =
            env->GetFieldID(gFileStatusClass, "dev", "I");

    jfieldID gFileStatusInoFieldID =
            env->GetFieldID(gFileStatusClass, "ino", "I");

    jfieldID gFileStatusModeFieldID =
            env->GetFieldID(gFileStatusClass, "mode", "I");

    jfieldID gFileStatusNlinkFieldID =
            env->GetFieldID(gFileStatusClass, "nlink", "I");

    jfieldID gFileStatusUidFieldID =
            env->GetFieldID(gFileStatusClass, "uid", "I");

    jfieldID gFileStatusGidFieldID =
            env->GetFieldID(gFileStatusClass, "gid", "I");

    jfieldID gFileStatusSizeFieldID =
            env->GetFieldID(gFileStatusClass, "size", "J");

    jfieldID gFileStatusBlksizeFieldID =
            env->GetFieldID(gFileStatusClass, "blksize", "I");

    jfieldID gFileStatusBlocksFieldID =
            env->GetFieldID(gFileStatusClass, "blocks", "J");

    jfieldID gFileStatusAtimeFieldID =
            env->GetFieldID(gFileStatusClass, "atime", "J");

    jfieldID gFileStatusMtimeFieldID =
            env->GetFieldID(gFileStatusClass, "mtime", "J");

    jfieldID gFileStatusCtimeFieldID =
            env->GetFieldID(gFileStatusClass, "ctime", "J");

    const char* pathStr = env->GetStringUTFChars(path, NULL);
    jboolean ret = false;

    struct stat s;
    int res = stat(pathStr, &s);

    if (res == 0) {
        ret = true;

        if (fileStatus != NULL) {
            env->SetIntField(fileStatus, gFileStatusDevFieldID, s.st_dev);
            env->SetIntField(fileStatus, gFileStatusInoFieldID, s.st_ino);
            env->SetIntField(fileStatus, gFileStatusModeFieldID, s.st_mode);
            env->SetIntField(fileStatus, gFileStatusNlinkFieldID, s.st_nlink);
            env->SetIntField(fileStatus, gFileStatusUidFieldID, s.st_uid);
            env->SetIntField(fileStatus, gFileStatusGidFieldID, s.st_gid);
            env->SetLongField(fileStatus, gFileStatusSizeFieldID, s.st_size);
            env->SetIntField(fileStatus, gFileStatusBlksizeFieldID, s.st_blksize);
            env->SetLongField(fileStatus, gFileStatusBlocksFieldID, s.st_blocks);
            env->SetLongField(fileStatus, gFileStatusAtimeFieldID, s.st_atime);
            env->SetLongField(fileStatus, gFileStatusMtimeFieldID, s.st_mtime);
            env->SetLongField(fileStatus, gFileStatusCtimeFieldID, s.st_ctime);
        }
    }

    env->ReleaseStringUTFChars(path, pathStr);

    return ret;
}

jobject Java_android_os_FileHelper_getpwuid(
        JNIEnv* env,
        jobject clazz,
        jint uid) {
    li("---------getpwuid");

    jclass cPassWord =
            env->FindClass("android/os/FileHelper$PassWord");

    if (cPassWord == NULL) {
        li("Unable to find class android.os.FileHelper$PassWord");
        return 0;
    }

    jfieldID gPassWordNameFieldID =
            env->GetFieldID(cPassWord, "name", "Ljava/lang/String;");

    jfieldID gPassWordPasswdFieldID =
            env->GetFieldID(cPassWord, "passwd", "Ljava/lang/String;");

    jfieldID gPassWordUidFieldID =
            env->GetFieldID(cPassWord, "uid", "I");

    jfieldID gPassWordGidFieldID =
            env->GetFieldID(cPassWord, "gid", "I");

    //jfieldID gPassWordGecosFieldID = env->GetFieldID(cPassWord, "gecos", "Ljava/lang/String;");

    jfieldID gPassWordDirFieldID =
            env->GetFieldID(cPassWord, "dir", "Ljava/lang/String;");

    jfieldID gPassWordShellFieldID =
            env->GetFieldID(cPassWord, "shell", "Ljava/lang/String;");

    jobject passwd = (env)->AllocObject(cPassWord);
    struct passwd *pw;

    pw = getpwuid(uid);

    if (pw == 0) {
        return 0;
    }

    li(pw->pw_name);

    env->SetObjectField(
            passwd,
            gPassWordNameFieldID,
            env->NewStringUTF(pw->pw_name));

    env->SetObjectField(
            passwd,
            gPassWordPasswdFieldID,
            env->NewStringUTF(pw->pw_passwd));

    env->SetIntField(passwd, gPassWordUidFieldID, pw->pw_uid);
    env->SetIntField(passwd, gPassWordGidFieldID, pw->pw_gid);

    //env->SetObjectField(passwd, gPassWordGecosFieldID, env->NewStringUTF(pw->pw_gecos));

    env->SetObjectField(
            passwd,
            gPassWordDirFieldID,
            env->NewStringUTF(pw->pw_dir));

    env->SetObjectField(
            passwd,
            gPassWordShellFieldID,
            env->NewStringUTF(pw->pw_shell));

    return passwd;
}

//#define SetObjectField(pEnv,pObj, pFieldID, pVal) pEnv->SetObjectField(pObj, pFieldID, pVal);DeleteLocalRef(pEnv, pVal)

jobject Java_android_os_FileHelper_getgrgid(
        JNIEnv* env,
        jobject clazz,
        jint gid) {
    //获取Java中的实例类
    li("----------getgrgid");

    jclass cGroup =
            env->FindClass("android/os/FileHelper$Group");

    if (cGroup == NULL) {
        li("Unable to find class android.os.FileHelper$Group");
        return 0;
    }

    //获取类中每一个变量的定义
    jfieldID gGroupNameFieldID =
            env->GetFieldID(cGroup, "name", "Ljava/lang/String;");

    jfieldID gGroupPasswdFieldID =
            env->GetFieldID(cGroup, "passwd", "Ljava/lang/String;");

    jfieldID gGroupGidFieldID =
            env->GetFieldID(cGroup, "gid", "I");

    jfieldID gGroupMemFieldID =
            env->GetFieldID(cGroup, "mem", "[Ljava/lang/String;");

    //分配新 Java 对象而不调用该对象的任何构造函数。返回该对象的引用。 clazz 参数务必不要引用数组类。
    jobject group = (env)->AllocObject(cGroup);
    struct group *gr;

    gr = getgrgid(gid);

    if (gr == 0) {
        return 0;
    }

    jclass jcstring =
            env->FindClass("java/lang/String"); //定义数组中元素类型

    jstring strTemp;

    //给每一个实例的变量赋值
    strTemp = env->NewStringUTF(gr->gr_name);
    env->SetObjectField(group, gGroupNameFieldID, strTemp);
    env->SetObjectField(
            group,
            gGroupPasswdFieldID,
            env->NewStringUTF(gr->gr_passwd));

    env->SetIntField(group, gGroupGidFieldID, gr->gr_gid);

    int i = 0;

    while (gr->gr_mem[i++])
        ;

    jobjectArray mems;

    //创建一个数组类型为String类

    //
    //jarray NewObjectArray(JNIEnv *env, jsize length, jclass elementClass, jobject initialElement);

    //构造新的数组，它将保存类 elementClass 中的对象。所有元素初始值均设为 initialElement。

    //参数：

    //env：JNI 接口指针。

    //length：数组大小。

    //elementClass：数组元素类。

    //initialElement：初始值。

    //返回值：

    //Java 数组对象。如果无法构造数组，则为 NULL。

    //抛出：

    //OutOfMemoryError：如果系统内存不足。

    mems = (env)->NewObjectArray(i, jcstring, 0);

    //env->SetIntField(group, gGroupMemFieldID, gr->mem);

    for (--i; i >= 0; i--) {
        strTemp = env->NewStringUTF(gr->gr_mem[i]);

        //添加到objcet数组中
        (env)->SetObjectArrayElement(mems, i, strTemp);
        (env)->DeleteLocalRef(strTemp);
    }

    env->SetObjectField(group, gGroupMemFieldID, mems);
    (env)->DeleteLocalRef(mems);

    return group;
}

/*
jstring
Java_android_os_FileHelper_stringFromJNI(JNIEnv* env, jobject thiz)
{
    return (env)->NewStringUTF("Hello from JNI !");
}
*/

jint Java_android_os_FileHelper_chown(
        JNIEnv* env,
        jobject clazz,
        jstring file,
        jint uid,
        jint gid) {
#if HAVE_ANDROID_OS
    const jchar* str = env->GetStringCritical(file, 0);
    String8 file8;

    if (str) {
        file8 = String8(str, env->GetStringLength(file));
        env->ReleaseStringCritical(file, str);
    }

    if (file8.size() <= 0) {
        return ENOENT;
    }

    if (uid >= 0 || gid >= 0) {
        int res = chown(file8.string(), uid, gid);

        if (res != 0) {
            return errno;
        }
    }

    return res;
#else
    return ENOSYS;
#endif
}

jint Java_android_os_FileHelper_chmod(
        JNIEnv* env,
        jobject clazz,
        jstring file,
        jint mode) {
#if HAVE_ANDROID_OS
    const jchar* str = env->GetStringCritical(file, 0);
    String8 file8;

    if (str) {
        file8 = String8(str, env->GetStringLength(file));
        env->ReleaseStringCritical(file, str);
    }

    if (file8.size() <= 0) {
        return ENOENT;
    }

    return chmod(file8.string(), mode) == 0 ? 0 : errno;
#else
    return ENOSYS;
#endif
}

jint Java_android_os_FileHelper_getFatVolumeId(
        JNIEnv* env,
        jobject clazz,
        jstring path) {
#if HAVE_ANDROID_OS
    if (path == NULL) {
        jniThrowException(
                env,
                "java/lang/IllegalArgumentException",
                NULL);

        return -1;
    }

    const char *pathStr = env->GetStringUTFChars(path, NULL);
    int result = -1;

    // only if our system supports this ioctl
#ifdef VFAT_IOCTL_GET_VOLUME_ID
    int fd = open(pathStr, O_RDONLY);

    if (fd >= 0) {
        result = ioctl(fd, VFAT_IOCTL_GET_VOLUME_ID);
        close(fd);
    }
#endif

    env->ReleaseStringUTFChars(path, pathStr);

    return result;
#else
    return -1;
#endif
}

/**
 * Table of methods associated with a single class.
 */

/*
 //Landroid/os/FileHelper;.stringFromJNI ()Ljava/lang/String;

 static JNINativeMethod gMethods[] = {
     {
         "setPermissions",
         "(Ljava/lang/String;III)I",
         (void*) Java_android_os_FileHelper_setPermissions
     },
     {
         "getPermissions",
         "(Ljava/lang/String;[I)I",
         (void*) Java_android_os_FileHelper_getPermissions
     },
     {
         "getFatVolumeId",
         "(Ljava/lang/String;)I",
         (void*) Java_android_os_FileHelper_getFatVolumeId
     },
     {
         "stat",
         "(Ljava/lang/String;Landroid/os/FileHelper$FileStatus;)Z",
         (void*) Java_android_os_FileHelper_stat
     },
     {
         "getpwuid",
         "(I)Landroid/os/FileHelper$PassWord;",
         (void*) Java_android_os_FileHelper_getpwuid
     },
     {
         "getgrgid",
         "(II)Landroid/os/FileHelper$Group;",
         (void*) Java_android_os_FileHelper_getgrgid
     },
     {
         "chown",
         "(Ljava/lang/String;II)I",
         (void*) Java_android_os_FileHelper_chown
     },
     {
         "chmod",
         "(Ljava/lang/String;I)I",
         (void*) Java_android_os_FileHelper_chmod
     },
     {
         "stringFromJNI",
         "()Ljava/lang/String;",
         (void*) Java_android_os_FileHelper_stringFromJNI
     }
 };

 static const char* const kFileHelperPathName =
         "android/os/FileUtils";

 #define JNIREG_CLASS "android/os/FileUtils" //指定要注册的类

 // Register several native methods for one class.

 static int registerNativeMethods(
         JNIEnv* env,
         const char* className,
         JNINativeMethod* gMethods,
         int numMethods) {
     jclass clazz;

     clazz = (env)->FindClass(className);

     if (clazz == NULL) {
         return JNI_FALSE;
     }

     if ((env)->RegisterNatives(clazz, gMethods, numMethods) < 0) {
         return JNI_FALSE;
     }

     return JNI_TRUE;
 }

 // Register native methods for all classes we know about.

 static int registerNatives(JNIEnv* env) {
     if (!registerNativeMethods(
             env,
             JNIREG_CLASS,
             gMethods,
             sizeof(gMethods) / sizeof(gMethods[0]))) {
         return JNI_FALSE;
     }

     return JNI_TRUE;
 }

 // Set some test stuff up.
 //
 //Returns the JNI version on success, -1 on failure.

 JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void* reserved) {
     JNIEnv* env = NULL;
     jint result = -1;

     if (vm->GetEnv(
             (void**) &env,
             JNI_VERSION_1_4) != JNI_OK) {
         return -1;
     }

     assert(env != NULL);

     if (!registerNatives(env)) { //注册
         return -1;
     }

     // success -- return valid version number
     result = JNI_VERSION_1_4;

     return result;
 }
*/

/*
int register_Java_android_os_FileHelper(JNIEnv* env) {
    jclass clazz;

    clazz = env->FindClass(kFileHelperPathName);

    LOG_FATAL_IF(
            clazz == NULL,
            "Unable to find class android.os.FileHelper");

    gFileStatusClass =
            env->FindClass("android/os/FileHelper$FileStatus");

    LOG_FATAL_IF(
            gFileStatusClass == NULL,
            "Unable to find class android.os.FileHelper$FileStatus");

    gFileStatusDevFieldID =
            env->GetFieldID(gFileStatusClass, "dev", "I");

    gFileStatusInoFieldID =
            env->GetFieldID(gFileStatusClass, "ino", "I");

    gFileStatusModeFieldID =
            env->GetFieldID(gFileStatusClass, "mode", "I");

    gFileStatusNlinkFieldID =
            env->GetFieldID(gFileStatusClass, "nlink", "I");

    gFileStatusUidFieldID =
            env->GetFieldID(gFileStatusClass, "uid", "I");

    gFileStatusGidFieldID =
            env->GetFieldID(gFileStatusClass, "gid", "I");

    gFileStatusSizeFieldID =
            env->GetFieldID(gFileStatusClass, "size", "J");

    gFileStatusBlksizeFieldID =
            env->GetFieldID(gFileStatusClass, "blksize", "I");

    gFileStatusBlocksFieldID =
            env->GetFieldID(gFileStatusClass, "blocks", "J");

    gFileStatusAtimeFieldID =
            env->GetFieldID(gFileStatusClass, "atime", "J");

    gFileStatusMtimeFieldID =
            env->GetFieldID(gFileStatusClass, "mtime", "J");

    gFileStatusCtimeFieldID =
            env->GetFieldID(gFileStatusClass, "ctime", "J");

    gPassWordClass =
            env->FindClass("android/os/FileHelper$PassWord");

    LOG_FATAL_IF(
            gFileStatusClass == NULL,
            "Unable to find class android.os.FileHelper$PassWord");

    gPassWordNameFieldID =
            env->GetFieldID(
                    gPassWordClass,
                    "name",
                    "Ljava/lang/String;");

    gPassWordPasswdFieldID =
            env->GetFieldID(
                    gPassWordClass,
                    "passwd",
                    "Ljava/lang/String;");

    gPassWordUidFieldID =
            env->GetFieldID(gPassWordClass, "uid", "I");

    gPassWordGidFieldID =
            env->GetFieldID(gPassWordClass, "gid", "I");

    gPassWordGecosFieldID =
            env->GetFieldID(
                    gPassWordClass,
                    "gecos",
                    "Ljava/lang/String;");

    gPassWordDirFieldID =
            env->GetFieldID(
                    gPassWordClass,
                    "dir",
                    "Ljava/lang/String;");

    gPassWordShellFieldID =
            env->GetFieldID(
                    gPassWordClass,
                    "shell",
                    "Ljava/lang/String;");

    gGroupClass =
            env->FindClass("android/os/FileHelper$Group");

    LOG_FATAL_IF(
            gFileStatusClass == NULL,
            "Unable to find class android.os.FileHelper$Group");

    gGroupNameFieldID =
            env->GetFieldID(
                    gGroupClass,
                    "name",
                    "Ljava/lang/String;");

    gGroupPasswdFieldID =
            env->GetFieldID(
                    gGroupClass,
                    "passwd",
                    "Ljava/lang/String;");

    gGroupGidFieldID =
            env->GetFieldID(gGroupClass, "gid", "I");

    gGroupMemFieldID =
            env->GetFieldID(
                    gGroupClass,
                    "mem",
                    "[Ljava/lang/String;");

    return AndroidRuntime::registerNativeMethods(
            env,
            kFileHelperPathName,
            methods,
            NELEM(methods));
}
*/

}

#ifdef __cplusplus
}

#endif
```
