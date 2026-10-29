
# [原创]【脱壳一】某壳分析+修复

## 声明

仅限技术讨论，不得用于非法途径，后果自负。

分析中有什么错误欢迎大家指出

## Java层分析

java层简单分析下流程，重点放在so文件分析

加固效果主要把类方法抽取（其实并没有抽取，还是在dex文件中，只是修改了`accessFlags`和`codeoff`）

![](https://bbs.kanxue.com/upload/attach/201805/781572_362WJSJ2D4CMQQA.png)

通过查看`AndroidManifest`文件`application`被修改成壳入口的文件（此壳加固的原apk必须自己声明`Application`文件，否则会加固失败，加固网站也不会提示，只是在加固apk中有文本提示，感觉用户体验不是很好）

![](https://bbs.kanxue.com/upload/attach/201805/781572_6XKJCUCQZ9PF88H.jpg)

这里可以根据壳入口点自己跟着走一遍，我只说下大体流程吧

![](https://bbs.kanxue.com/upload/attach/201805/781572_Q8FTUM95ZM39GBP.jpg)

当执行完`application`就该执行`AndroidManifest`声明的`activity`，执行`activity`先执行里面的方法`static`方法，`dofixc`方法最终会执行到`native`的`soInit`和`dofixs`来修复方法

## Native方法分析

通过java层分析，最终会加载`libzuma.so`

`soinit`方法分析，f5大法好

此壳f5一些函数损坏不是很严重，大部分都保留原样

方法大部分都是修复的，我只大体说下函数作用，其他具体自己看代码

```c
int __fastcall Java_com_ali_mobisecenhance_Fix_soInit(int env, int a2, ali *zumaDataPath, int packageName, int context, int cookies, ali *dexnumbers)

{

  int packageName1; // r6

  JNIEnv *env1; // r7

  int v9; // r0

  int androidDevice; // r5

  int *androidDeviceAddr; // r3

  AndroidDevice **androidDeviceAddr1; // ST0C_4

  AndroidDevice *androidDevice1; // r5

  ali::AppInfo *appinfo; // r6

  int appinfoAddr; // r5

  ZumaInfo *zumaInfo; // r6

  ZumaInfo **zumaInfo1; // r5

  bool v18; // r2

  AndroidDevice **androidDeviceAddr2; // r6

  ZumaInfo **zumaInfo2; // r4

  int i; // r5

  int v22; // r3

  unsigned int *v23; // r1

  int v24; // r3

  int v25; // r3

  ZumaInfo *zumaInfo3; // r0

  int v27; // r0

  int v28; // r2

  int v29; // r2

  ArtMethodInfo *methodInfo1; // r4

  int *v32; // [sp+0h] [bp-100h]

  int v33; // [sp+8h] [bp-F8h]

  int v34; // [sp+Ch] [bp-F4h]

  ali *zumaDataPath1; // [sp+10h] [bp-F0h]

  char v36; // [sp+1Ch] [bp-E4h]

  char v37; // [sp+34h] [bp-CCh]

  char v38; // [sp+4Ch] [bp-B4h]

  char v39; // [sp+64h] [bp-9Ch]



  packageName1 = packageName;

  zumaDataPath1 = zumaDataPath;

  env1 = (JNIEnv *)env;

  v32 = &GLOBAL_OFFSET_TABLE_;

  enter("jni/demolish.cpp", "Java_com_ali_mobisecenhance_Fix_soInit", 584);

  v9 = ali::hookinit();

  if ( v9 )

    v9 = my_abort(

           "jni/demolish.cpp",

           "Java_com_ali_mobisecenhance_Fix_soInit",

           587,

           "hook init error! ",

           &GLOBAL_OFFSET_TABLE_);

  j_clock(v9);                                  // 测试函数执行时间

  ali::jstringToStdString(&v36, env1, packageName1);// [v38+4]=v38地址+16 [v38+5]存入packageName1开始[v38+4]存入packageName1结束

  std::string::string((int)&v38, (int)&v36);    // [v40+5]存入pName开始[v40+4]存入packageName1结束

  androidDevice = operator new(0x48u);

  ali::AndroidDevice::AndroidDevice(androidDevice, env1);

  androidDeviceAddr = (int *)*(v32 - 61);

  *androidDeviceAddr = androidDevice;

  androidDeviceAddr1 = (AndroidDevice **)androidDeviceAddr;

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v38);

  androidDevice1 = *androidDeviceAddr1;

  std::string::string((int)&v38, (int)&v36);

  ali::AndroidDevice::get_art_so_address(androidDevice1, (int)&v38);// 获取art地址和classes.dex地址

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v38);

  std::string::string((int)&v38, (int)&v36);

  appinfo = (ali::AppInfo *)operator new(0x64u);

  ali::AppInfo::AppInfo(appinfo, (_JNIEnv *)env1, (int)&v38);

  appinfoAddr = *(v32 - 62);

  *(_DWORD *)appinfoAddr = appinfo;

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v38);

  *(_DWORD *)(*(_DWORD *)appinfoAddr + 96) = dexnumbers;

  ali::jstringToStdString(&v37, env1, (int)zumaDataPath1);

  std::string::string((int)&v38, (int)&v37);

  zumaInfo = (ZumaInfo *)operator new(0x34u);

  ali::ZumaInfo::ZumaInfo(zumaInfo, (const char **)&v38);// 读取zumadata信息

  zumaInfo1 = (ZumaInfo **)*(v32 - 63);

  *zumaInfo1 = zumaInfo;

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v38);

  androidDeviceAddr2 = androidDeviceAddr1;

  zumaInfo2 = zumaInfo1;

  if ( cookies )                                // cookies

  {

    v34 = ((int (__fastcall *)(JNIEnv *, int, _DWORD))(*env1)->GetIntArrayElements)(env1, cookies, 0);

    v33 = ((int (__fastcall *)(JNIEnv *, int))(*env1)->GetArrayLength)(env1, cookies);

    j_memset(&v39, 0, 128);

    for ( i = 0; i < v33; ++i )

    {

      v18 = v34;

      v22 = *(_DWORD *)(v34 + 4 * i);

      v23 = *(unsigned int **)(v22 + 16);

      if ( !v23 )

      {

        v24 = *(_DWORD *)(v22 + 12);

        if ( !v24 )

          continue;

        v25 = *(_DWORD *)(v24 + 40);

        if ( !v25 )

          continue;

        v23 = *(unsigned int **)(v25 + 4);

        if ( !v23 )

          continue;

      }

      ali::ZumaInfo::set_dex_file_by_address(*zumaInfo2, v23, &v39);

    }

  }

  if ( (unsigned int)((*androidDeviceAddr2)->system_sdk_version - 21) <= 2 )

    ali::findDexByCookies(env1, (_JNIEnv *)((char *)&dword_0 + 1), v18);

  zumaInfo3 = *zumaInfo2;

  if ( (*androidDeviceAddr2)->systemModeBoolean )

  {

    if ( !ali::ZumaInfo::is_all_dex_address_found(zumaInfo3) )// 返回1

    {

      if ( (*androidDeviceAddr2)->system_sdk_version == 19 )

        ali::get_all_dex_header_address_dalvik(dexnumbers, 0, v29);

      else

        ali::get_dex_header_address_art((ali *)env1, 0);

    }

    methodInfo1 = (ArtMethodInfo *)operator new(0x24u);

    ali::ARTMethodInfo::ARTMethodInfo(methodInfo1, (_JNIEnv *)env1);

    *(_DWORD *)*(v32 - 60) = methodInfo1;

    showArtImportantInfo();

  }

  else

  {

    v27 = ali::ZumaInfo::is_all_dex_address_found(zumaInfo3);

    if ( !v27 )

      v27 = ali::get_all_dex_header_address_dalvik(dexnumbers, 0, v28);

  }

  j_clock(v27);

  quit("jni/demolish.cpp", "Java_com_ali_mobisecenhance_Fix_soInit", 677);

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v37);

  return std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v36);

}
```

主要3个方法

1. `ali::AndroidDevice::AndroidDevice(androidDevice, env1);`

主要判断sdk和dalvik还是art

```c
AndroidDevice *__fastcall ali::AndroidDevice::AndroidDevice(int androidDevice, JNIEnv *env)

{

  AndroidDevice *androidDevice1; // r4

  JNIEnv *env1; // r7

  int v4; // r6

  int v5; // r0

  int v6; // r0

  _JNIEnv *v7; // r1

  _BOOL4 v8; // r0



  androidDevice1 = (AndroidDevice *)androidDevice;

  env1 = env;

  v4 = androidDevice + 4;

  *(_DWORD *)(androidDevice + 20) = androidDevice + 4;

  *(_DWORD *)(androidDevice + 24) = androidDevice + 4;

  std::priv::_String_base<char,std::allocator<char>>::_M_allocate_block(androidDevice + 4, 16);

  *(_BYTE *)androidDevice1->field_14 = 0;

  androidDevice1->systemModeAddr = 0;

  androidDevice1->field_30 = 0;

  androidDevice1->field_34 = 0;

  androidDevice1->dexAddr = 0;

  androidDevice1->field_3C = 0;

  androidDevice1->field_40 = 0;

  androidDevice1->env = (int)env1;

  std::string::operator=(v4, &unk_28EB8);

  androidDevice1->systemModeBoolean = 0;

  androidDevice1->system_sdk_version = ali::AndroidDevice::get_system_sdk_version((ali::AndroidDevice *)androidDevice1);// 魅族5.0.1->21

  v5 = ali::AndroidDevice::get_runtime_by_mmap((ali::AndroidDevice *)androidDevice1);// 获取系统模式dalvik返回1 art返回2

  v6 = ali::AndroidDevice::get_system_runtime((ali::AndroidDevice *)androidDevice1, v5);// 如果上边判断不出是dalvik还是art继续判断

  androidDevice1->systemMode = v6;

  androidDevice1->systemModeBoolean = (unsigned int)(v6 - 1) >= 1;// dalvik=false art=true

  if ( j_strstr(androidDevice1->systemModeContent, "libaoc.so") )

    androidDevice1->isYunOs = 1;

  else

    androidDevice1->isYunOs = ali::check_for_yun_os((JNIEnv *)androidDevice1->env, v7);// 有ro.yunos.version的话返回1

  v8 = ali::check_for_yun_os_TV((ali *)androidDevice1->env, v7);

  androidDevice1->isYunOsTv = v8;               // yun_os_tv返回1

  if ( v8 )

    androidDevice1->isYunOs = 1;

  return androidDevice1;

}
```

2. `ali::AppInfo::AppInfo(appinfo, (_JNIEnv *)env1, (int)&v38);`

主要是获取一些文件目录
