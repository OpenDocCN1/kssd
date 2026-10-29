
```c
#include <stdio.h>
#include <stdlib.h>
#include <jni.h>
#include <android/log.h>
#include <cstring>
#define TAG "jni-log"
#define LOGD(...) __android_log_print(ANDROID_LOG_DEBUG,TAG,__VA_ARGS__)
//签名信息
const char *app_sha1="59F8A6B86A367F0586F1A15DDDB63D75263C5D62";
const char hexcode[] = {'0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F'};
char* getSha1(JNIEnv *env, jobject context_object){
//上下文对象
jclass context_class = env->GetObjectClass(context_object);
//反射获取PackageManager
jmethodID methodId = env->GetMethodID(context_class, "getPackageManager", "()Landroid/content/pm/PackageManager;");
jobject package_manager = env->CallObjectMethod(context_object, methodId);
if (package_manager == NULL) {
//LOGD("package_manager is NULL!!!");
return NULL;
}
//反射获取包名
methodId = env->GetMethodID(context_class, "getPackageName", "()Ljava/lang/String;");
jstring package_name = (jstring)env->CallObjectMethod(context_object, methodId);
if (package_name == NULL) {
//LOGD("package_name is NULL!!!");
return NULL;
}
env->DeleteLocalRef(context_class);
//获取PackageInfo对象
jclass pack_manager_class = env->GetObjectClass(package_manager);
methodId = env->GetMethodID(pack_manager_class, "getPackageInfo", "(Ljava/lang/String;I)Landroid/content/pm/PackageInfo;");
env->DeleteLocalRef(pack_manager_class);
jobject package_info = env->CallObjectMethod(package_manager, methodId, package_name, 0x40);
if (package_info == NULL) {
LOGD("getPackageInfo() is NULL!!!");
return NULL;
}
env->DeleteLocalRef(package_manager);
//获取签名信息
jclass package_info_class = env->GetObjectClass(package_info);
jfieldID fieldId = env->GetFieldID(package_info_class, "signatures", "[Landroid/content/pm/Signature;");
env->DeleteLocalRef(package_info_class);
jobjectArray signature_object_array = (jobjectArray)env->GetObjectField(package_info, fieldId);
if (signature_object_array == NULL) {
LOGD("signature is NULL!!!");
return NULL;
}
jobject signature_object = env->GetObjectArrayElement(signature_object_array, 0);
env->DeleteLocalRef(package_info);
//签名信息转换成sha1值
jclass signature_class = env->GetObjectClass(signature_object);
methodId = env->GetMethodID(signature_class, "toByteArray", "()[B");
env->DeleteLocalRef(signature_class);
jbyteArray signature_byte = (jbyteArray) env->CallObjectMethod(signature_object, methodId);
jclass byte_array_input_class=env->FindClass("java/io/ByteArrayInputStream");
methodId=env->GetMethodID(byte_array_input_class,"<init>","([B)V");
jobject byte_array_input=env->NewObject(byte_array_input_class,methodId,signature_byte);
jclass certificate_factory_class=env->FindClass("java/security/cert/CertificateFactory");
methodId=env->GetStaticMethodID(certificate_factory_class,"getInstance","(Ljava/lang/String;)Ljava/security/cert/CertificateFactory;");
jstring x_509_jstring=env->NewStringUTF("X.509");
jobject cert_factory=env->CallStaticObjectMethod(certificate_factory_class,methodId,x_509_jstring);
methodId=env->GetMethodID(certificate_factory_class,"generateCertificate","(Ljava/io/InputStream;)Ljava/security/cert/Certificate;");
jobject x509_cert=env->CallObjectMethod(cert_factory,methodId,byte_array_input);
env->DeleteLocalRef(certificate_factory_class);
jclass x509_cert_class=env->GetObjectClass(x509_cert);
methodId=env->GetMethodID(x509_cert_class,"getEncoded","()[B");
jbyteArray cert_byte=(jbyteArray)env->CallObjectMethod(x509_cert,methodId);
env->DeleteLocalRef(x509_cert_class);
jclass message_digest_class=env->FindClass("java/security/MessageDigest");
methodId=env->GetStaticMethodID(message_digest_class,"getInstance","(Ljava/lang/String;)Ljava/security/MessageDigest;");
jstring sha1_jstring=env->NewStringUTF("SHA1");
jobject sha1_digest=env->CallStaticObjectMethod(message_digest_class,methodId,sha1_jstring);
methodId=env->GetMethodID(message_digest_class,"digest","([B)[B");
jbyteArray sha1_byte=(jbyteArray)env->CallObjectMethod(sha1_digest,methodId,cert_byte);
env->DeleteLocalRef(message_digest_class);
//转换成char
jsize array_size=env->GetArrayLength(sha1_byte);
jbyte* sha1 =env->GetByteArrayElements(sha1_byte,NULL);
char *hex_sha=new char[array_size*2+1];
for (int i = 0; i <array_size ; ++i) {
hex_sha[2*i]=hexcode[((unsigned char)sha1[i])/16];
hex_sha[2*i+1]=hexcode[((unsigned char)sha1[i])%16];
}
hex_sha[array_size*2]='\0';
LOGD("hex_sha %s ",hex_sha);
return hex_sha;
}
jboolean checkValidity(JNIEnv *env,char *sha1){
//比较签名
if (strcmp(sha1,app_sha1)==0)
{
LOGD("验证成功");
return true;
}
LOGD("验证失败");
return false;
}
extern "C"
JNIEXPORT jstring JNICALL
Java_com_example_jnisignatureverify_MainActivity_getSignaturesSha1(JNIEnv *env, jobject thiz,
jobject context) {
// TODO: implement getSignaturesSha1()
return env->NewStringUTF(app_sha1);
}extern "C"
JNIEXPORT jboolean JNICALL
Java_com_example_jnisignatureverify_MainActivity_checkSha1(JNIEnv *env, jobject thiz,
jobject contextObject) {
// TODO: implement checkSha1()
char *sha1 = getSha1(env,contextObject);
jboolean result = checkValidity(env,sha1);
return result;
}extern "C"
JNIEXPORT jstring JNICALL
Java_com_example_jnisignatureverify_MainActivity_getToken(JNIEnv *env, jobject thiz,
jobject contextObject, jstring user_id) {
// TODO: implement getToken()
char *sha1 = getSha1(env,contextObject);
jboolean result = checkValidity(env,sha1);
if(result){
return env->NewStringUTF("获取Token成功");
}else{
return env->NewStringUTF("获取失败，请检查native-lib.cpp文件配置的sha1值");
}
}
```

`activity_main.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:app="http://schemas.android.com/apk/res-auto"
xmlns:tools="http://schemas.android.com/tools"
android:layout_margin="16dp"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:orientation="vertical"
tools:context="com.example.jnisignatureverify.MainActivity">
<LinearLayout
android:layout_width="match_parent"
android:orientation="vertical"
android:layout_height="wrap_content">
<TextView
android:layout_width="wrap_content"
android:layout_height="wrap_content"
android:text="APP签名信息:"/>
<TextView
android:id="@+id/app_signatures_tv"
android:layout_width="wrap_content"
android:layout_height="wrap_content"/>
</LinearLayout>
<LinearLayout
android:layout_marginTop="16dp"
android:orientation="vertical"
android:layout_width="match_parent"
android:layout_height="wrap_content">
<TextView
android:layout_width="wrap_content"
android:layout_height="wrap_content"
android:text="jni配置的签名信息:"/>
<TextView
android:id="@+id/jni_signatures_tv"
android:layout_width="wrap_content"
android:layout_height="wrap_content"/>
</LinearLayout>
<Button
android:id="@+id/check_btn"
android:layout_marginTop="16dp"
android:text="验证签名是否正确"
android:layout_width="wrap_content"
android:layout_height="wrap_content"/>
<Button
android:id="@+id/token_btn"
android:layout_marginTop="16dp"
android:text="获取token"
android:layout_width="wrap_content"
android:layout_height="wrap_content"/>
</LinearLayout>
```

### 4.反调试

#### 4.1 思路一（一个进程最多只能被一个进程`ptrace`）

在处于调试状态时，Linux会向`/proc/pid/status`写入一些进程状态信息，比如`TracerPid`字段会写入调试进程的`pid`，因此我们可以自己`ptrace`自己，然后让`android_server`不能调试

代码如下：

```c
#include<sys/ptrace.h> //这个头文件很重要
void anti_debug01()
{
ptrace(PTRACE_TRACEME,0,0,0);
}
```

```c
jint JNI_Onload(JavaVM* vm,void* reserved)
{
anti_debug01();
}
```

一旦开始调试，就会出现

![在这里插入图片描述](upload/attach/202003/845506_GZH6S5CDTXWZUET.jpg)

反反调试思路：`nop`掉`anti_debug01()`函数调用

#### 4.2检测`Tracepid`的值

根据第一种分析得出`Tracepid`的值只要**不为0** 就能说明进程正在被调试，因此我们只需要检测`Tracepid`的值是不是0，如果不为0，直接退出就行了
