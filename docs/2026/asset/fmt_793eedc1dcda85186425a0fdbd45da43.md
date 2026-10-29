
那么现在就比较清楚了，`GetEnv()`方法一共三个参数，第一个是`vm`，第二个是`env`，第三个是`JNIVERSION*`，第三个参数不必在意。

```c
JNIEnv* env = NULL;

jint result = -1;

if ((*vm)->GetEnv(vm, (void**) &env, JNI_VERSION_1_4) != JNI_OK) {
    return -1;
}
```

## IDA中反编译出来的代码

```c
if ((*vm)->GetEnv(vm, (void **)&v6, 65542))
{
  v2 = "debug";
  v3 = "Failed to get the environment";

LABEL_5:

  _android_log_print(6, v2, v3);

  return -1;
}
```

知道了第二个参数是`env`，重命名一下，顺便修改类型为`JNIEnv*`。

这一下看起来好看多了，该有的函数都已经识别出来。

```c
signed int __fastcall JNI_OnLoad(JavaVM *vm, int a2)
{
  const char *v2; // r1@2
  const char *v3; // r2@2
  signed int result; // r0@5
  bool v5; // zf@6
  JNIEnv *env; // [sp+4h] [bp-Ch]@1

  env = (JNIEnv *)a2;

  if ((*vm)->GetEnv(vm, (void **)&env, 65542))
  {
    v2 = "debug";
    v3 = "Failed to get the environment";

LABEL_5:

    _android_log_print(6, v2, v3);

    return -1;
  }

  if (!((int (__cdecl *)(JNIEnv *))(*env)->FindClass)(env))
  {
    v2 = "debug";
    v3 = "failed to get class reference";

    goto LABEL_5;
  }

  v5 = ((int (__cdecl *)(JNIEnv *))(*env)->RegisterNatives)(env) == 0;
  result = 65542;

  if (!v5)
    result = -1;

  return result;
}
```

下面两个`(*env)->`的函数也需要`Force call type`，就是下面这个样子。首先执行`FindClass()`方法，然后执行`RegisterNatives()`动态注册`native`方法，可以看到最后那个参数是`2`，代表什么还记得吗？它表示要注册的`native`方法数量。

```c
signed int __fastcall JNI_OnLoad(JavaVM *vm, int a2)
{
  const char *v2; // r1@2
  const char *v3; // r2@2
  jclass v4; // r1@3
  signed int result; // r0@5
  bool v6; // zf@6
  JNIEnv *env; // [sp+4h] [bp-Ch]@1

  env = (JNIEnv *)a2;

  if ((*vm)->GetEnv(vm, (void **)&env, 65542))
  {
    v2 = "debug";
    v3 = "Failed to get the environment";

LABEL_5:

    _android_log_print(6, v2, v3);

    return -1;
  }

  v4 = (*env)->FindClass(env, "com/ali/mobisecenhance/StubApplication");

  if (!v4)
  {
    v2 = "debug";
    v3 = "failed to get class reference";

    goto LABEL_5;
  }

  v6 = (*env)->RegisterNatives(
           env,
           v4,
           (const JNINativeMethod *)off_54010,
           2) == 0;

  result = 65542;

  if (!v6)
    result = -1;

  return result;
}
```

第三个参数是`gMethods`，我们修改一下变量名。

来看它的结构体定义：

```c
typedef struct {
    const char* name;
    const char* signature;
    void* fnPtr;
} JNINativeMethod;
```

然后跳过去看看数据。从结构体定义可以看出来，它是三个数据为一组。

第一个是Java层的方法名，第二个是签名，第三个就是一个指针了，也就是`native`方法的地址。

```asm
.data:00054010 gMethods        DCD aAttachbasecont     ; DATA XREF: JNI_OnLoad+44
.data:00054010                                         ; .text:off_24784
.data:00054010                                         ; "attachBaseContext"
.data:00054014                 DCD aLandroidCont_1     ; "(Landroid/content/Context;)V"
.data:00054018                 DCD sub_24D3C+1
.data:0005401C                 DCD aOncreate           ; "onCreate"
.data:00054020                 DCD aV                  ; "()V"
.data:00054024                 DCD sub_24498+1
```

那么可以看出来，一共注册了两个方法，一个是`attachBaseContext()`方法，另一个是`onCreate()`方法。这两个方法刚好就对应我们在JEB里看到的那两个`native`方法。对着Java层的名字修改一下方法名。

然后跟入`attachBaseContext()`方法来分析一下代码。
