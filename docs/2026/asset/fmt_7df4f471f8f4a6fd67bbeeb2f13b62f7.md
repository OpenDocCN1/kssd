
```
JNIEnv *__fastcall ali::start_fix_this_class(JNIEnv *result, unsigned __int8 *a2, _jmethodID *cls, int dexAddr, int a5, int a6, int a7, unsigned int nativeCount, int methodOffset)

{

  int dexAddr1; // r4

  unsigned int i; // r3

  int methodIdIndex; // r7

  unsigned __int16 *dexMethodId; // r3

  unsigned __int16 protoIdIndex; // r6

  char *methodName; // r7

  char *methodSign; // r6

  unsigned int v16; // [sp+14h] [bp-3Ch]

  int string_ids_addr; // [sp+18h] [bp-38h]

  int methodAddr; // [sp+1Ch] [bp-34h]

  int className; // [sp+20h] [bp-30h]

  int type_ids_addr; // [sp+24h] [bp-2Ch]

  int method_ids_addr; // [sp+28h] [bp-28h]

  char *env; // [sp+2Ch] [bp-24h]

  _jmethodID *cls1; // [sp+30h] [bp-20h]

  int proto_ids_addr; // [sp+34h] [bp-1Ch]



  dexAddr1 = dexAddr;

  env = (char *)result;

  string_ids_addr = dexAddr + *(_DWORD *)(dexAddr + 0x3C);

  cls1 = cls;

  type_ids_addr = dexAddr + *(_DWORD *)(dexAddr + 0x44);

  method_ids_addr = dexAddr1 + *(_DWORD *)(dexAddr1 + 0x5C);

  proto_ids_addr = dexAddr + *(_DWORD *)(dexAddr + 0x4C);

  for ( i = 0; ; i = v16 + 1 )

  {

    v16 = i;

    if ( i >= nativeCount )                     // 被native的方法数量

      break;

    methodAddr = 0x14 * ali::readUint32((ali *)(methodOffset + 4 * i), a2);// methodOffset=2e

                                                // 0000A230=0x081c*0x14  1C 08 00 00 1D 08 00 00  B9 D7 9C DA 00 00 00 00

    methodIdIndex = *(_DWORD *)(ali::g_zuma_infos->methodInfo + methodAddr + 4);// 0x4846

    dexMethodId = (unsigned __int16 *)(method_ids_addr + 8 * methodIdIndex);// dex中某个地址

    protoIdIndex = dexMethodId[1];

    className = ali::get_class_name(dexAddr1, string_ids_addr, type_ids_addr, *dexMethodId);// typeIdIndex

    methodName = (char *)ali::get_method_name(dexAddr1, string_ids_addr, method_ids_addr, methodIdIndex);

    methodSign = ali::get_method_sig(dexAddr1, string_ids_addr, type_ids_addr, proto_ids_addr, protoIdIndex);

    if ( ali::g_device_infos->systemModeBoolean )

      ali::fix_method_in_art(

        env,

        cls1,

        (_jmethodID *)className,

        methodName,

        methodSign,

        *(_jmethodID **)(ali::g_zuma_infos->methodInfo + methodAddr + 8),

        (_jmethodID *)dexAddr1);

    else

      ali::fix_method_in_dalvik(

        env,

        cls1,

        (_jmethodID *)className,

        methodName,

        methodSign,

        (_jmethodID *)(dexAddr1 + *(_DWORD *)(ali::g_zuma_infos->methodInfo + methodAddr + 8)),

        (_jmethodID *)dexAddr1);

    j_free(className);

    j_free(methodName);

    result = (JNIEnv *)j_free(methodSign);

  }

  return result;

}
```

```
int __fastcall ali::fix_dalvik_method_struct(ali *this, _JNIEnv *a2, int a3, unsigned int a4)

{

  int v4; // r5

  int v5; // r4

  unsigned __int8 *v6; // r1

  unsigned __int8 *v7; // r1

  int codeOff; // r3

  unsigned __int8 *v9; // r1

  unsigned int v10; // r1

  int result; // r0

  unsigned __int8 *v12; // r1

  unsigned int v13; // r1



  v4 = a3;

  v5 = (int)a2;

  ali::getRegistersSize((ali *)a3, (unsigned __int8 *)a2);

  ali::getOutsSize((ali *)v4, v6);

  codeOff = v4 + 0x10;

  if ( ali::g_device_infos->isYunOs )

  {

    *(_DWORD *)(v5 + 0x18) = codeOff;

    *(_WORD *)(v5 + 0x20) = ali::getRegistersSize((ali *)v4, v7);

    *(_WORD *)(v5 + 0x22) = ali::getOutsSize((ali *)v4, v9);

    result = ali::getAccessFlags((ali *)*(unsigned __int16 *)(v5 + 6), v10);

    *(_WORD *)(v5 + 6) = result;

  }

  else

  {

    *(_DWORD *)(v5 + 0x20) = codeOff;

    *(_WORD *)(v5 + 0xA) = ali::getRegistersSize((ali *)v4, v7);

    *(_WORD *)(v5 + 0xC) = ali::getOutsSize((ali *)v4, v12);

    result = ali::getAccessFlags(*(ali **)(v5 + 4), v13);

    *(_DWORD *)(v5 + 4) = result;

  }

  return result;

}
```

至此壳的流程基本走完了

分析完可知

header

![](https://bbs.kanxue.com/upload/attach/201805/781572_7UEYEB32BE8ETMQ.jpg)

![](https://bbs.kanxue.com/upload/attach/201805/781572_A6TBBYVV2Q92ECS.jpg)

# 修复

因为通过上面分析，我们能找到native方法对应的`codeoff`，所以可以动态写脚本来修复

但是会有一个问题，`codeoff`可能会超出原来大小，这样dex整个偏移都要变，修复太麻烦，不现实

参考文章https://bbs.pediy.com/thread-220717.htm

通过修改baksmali源码来修复`codeoff`

说一下修复思路，源码我就不放出来了

1.通过上面源码分析，还原出`gen_dex_info_list`、`gen_clazz_info_list`、`gen_method_info_list`的代码

2.通过还原出的类，主动去加载所有需要修复的类

3.还原出修复类方法代码`start_fix_this_class`，由此可以得到`codeoff`

4.因为`codeoff`在dex文件中，所以直接在电脑64位系统加载到内存会有问题，我直接把类、方法、签名、`codeoff`保存到文件中

5.`baksmali`中去加载保存的文件信息，存到集合中

6.当`baksmali`反编译时候修复`accessflag`和`codeoff`

我分析的样本是之前的，修复脚本最新免费版一样适用

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2018-5-7 16:26

被Roselia编辑

，原因：

上传的附件：

- [test\_20171127.apk](javascript:void(0);)

  （1.47MB，232次下载）
