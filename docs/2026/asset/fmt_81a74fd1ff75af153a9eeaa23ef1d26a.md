
当输入值不为'XUe'时打印出来的结果为:

```
2022-01-14 14:48:41,303   DEBUG        androidemu.internal.modules | Loading module 'example_binaries/32/libc.so'.
2022-01-14 14:48:41,311   DEBUG        androidemu.internal.modules | => Base address: 0xa0000000
2022-01-14 14:48:41,330   ERROR        androidemu.internal.modules | => Undefined external symbol: android_get_application_target_sdk_version
2022-01-14 14:48:41,330   ERROR        androidemu.internal.modules | => Undefined external symbol: dl_unwind_find_exidx
2022-01-14 14:48:42,125   ERROR        androidemu.internal.modules | => Undefined external symbol: android_get_application_target_sdk_version
2022-01-14 14:48:42,125   ERROR        androidemu.internal.modules | => Undefined external symbol: dl_unwind_find_exidx
2022-01-14 14:48:42,529   DEBUG        androidemu.internal.modules | Loading module 'my_binaries/libnative-lib.so'.
2022-01-14 14:48:42,532   DEBUG        androidemu.internal.modules | => Base address: 0xa009b000
2022-01-14 14:48:42,566    INFO                           __main__ | init_array -->0xa00cd1e1
2022-01-14 14:48:42,568   DEBUG            androidemu.java.jni_env | JNIEnv->GetStringUtfChars(1, 0) was called
2022-01-14 14:48:42,568   DEBUG            androidemu.java.jni_env | => XUe
str_utf8:, format_utf8:%s666, args_utf8:XUe
2022-01-14 14:48:42,572   DEBUG            androidemu.java.jni_env | JNIEnv->NewStringUtf(XUe666) was called
2022-01-14 14:48:42,573   DEBUG            androidemu.java.jni_env | JNIEnv->GetStringUtfChars(2, 0) was called
2022-01-14 14:48:42,573   DEBUG            androidemu.java.jni_env | => XUe666
2022-01-14 14:48:42,575   DEBUG            androidemu.java.jni_env | JNIEnv->FindClass(com/kanxue/crackme/MainActivity) was called
2022-01-14 14:48:42,575   DEBUG            androidemu.java.jni_env | JNIEnv->GetStaticMethodId(3, crypt2, (Ljava/lang/String;)Z) was called
2022-01-14 14:48:42,576   DEBUG            androidemu.java.jni_env | JNIEnv->CallStaticBooleanMethodV(com/kanxue/crackme/MainActivity, crypt2 <(Ljava/lang/String;)Z>, 0x100ffea4) was called
crypt2 args : XUe666
2022-01-14 14:48:42,856   DEBUG        androidemu.internal.modules | Loading module 'example_binaries/32/libc.so'.
2022-01-14 14:48:42,864   DEBUG        androidemu.internal.modules | => Base address: 0xa0000000
2022-01-14 14:48:42,883   ERROR        androidemu.internal.modules | => Undefined external symbol: android_get_application_target_sdk_version
2022-01-14 14:48:42,883   ERROR        androidemu.internal.modules | => Undefined external symbol: dl_unwind_find_exidx
2022-01-14 14:48:43,678   ERROR        androidemu.internal.modules | => Undefined external symbol: android_get_application_target_sdk_version
2022-01-14 14:48:43,679   ERROR        androidemu.internal.modules | => Undefined external symbol: dl_unwind_find_exidx
2022-01-14 14:48:44,093   DEBUG        androidemu.internal.modules | Loading module 'my_binaries/libnative-lib.so'.
2022-01-14 14:48:44,096   DEBUG        androidemu.internal.modules | => Base address: 0xa009b000
2022-01-14 14:48:44,131    INFO                           __main__ | init_array -->0xa00cd1e1
2022-01-14 14:48:44,133   DEBUG            androidemu.java.jni_env | JNIEnv->GetStringUtfChars(1, 0) was called
2022-01-14 14:48:44,133   DEBUG            androidemu.java.jni_env | => XUe666
2022-01-14 14:48:44,135    INFO                           __main__ | str_utf8:, format_utf8:%s, args_utf8:XUe666
2022-01-14 14:48:44,143    INFO                           __main__ | s1:WFVlNjY2,s2:WFVlNjY2,ret:0
crypt2 result : 1
jnicheck result : 1
Process finished with exit code 0
```
