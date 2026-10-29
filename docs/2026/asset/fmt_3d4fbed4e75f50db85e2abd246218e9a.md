
vdog的`JNI_OnLoad`里面有6个分发器,不知道是改了`ollvm`还是`inline`进来的。

## 参考资源

* 思路和部分代码源于[llvm-deobfuscator](elink@03aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6d9f1p5W2e0c8f1y4Q4x3V1k6D9L8s2k6E0i4K6u0V1k6r3g2G2j5X3k6#2M7$3y4S2N6r3!0J5i4K6u0W2k6$3W2@1)

* `z3`相关代码源于[f-ing-around-with-binaryninja](elink@2c8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6B7L8%4y4Z5N6$3q4@1M7$3!0F1i4K6u0r3k6W2)9J5k6r3W2F1k6#2)9J5k6r3q4J5L8%4g2F1k6q4)9J5k6s2N6A6N6r3S2Q4x3X3c8T1K9h3&6S2M7Y4W2F1K9h3&6B7j5g2)9J5k6h3N6A6N6l9`.`.)

附件包含所有的代码和样本.

所有代码和样本也可从`github`下载: [ollvm-breaker](elink@c7fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8h3W2E0L8#2)9J5c8X3!0D9L8s2k6E0i4K6u0V1j5Y4u0W2j5h3E0W2M7R3`.`.)

另外, 反混淆脚本是在`binja` `GUI-less`模式下运行, 需要`Binary Ninja`商业版.

最后,放效果.

在`IDA`中打开`cff`或`vdog`后,运行对应的`fix-xxx.py`脚本反混淆.

我修复了`vdog`五个函数,`JNI_OnLoad`,`crazy::GetPackageName`,`prevent_attach_one`,`attach_thread_scn`,`crazy::CheckDex`.在`IDA`中运行修复脚本,重新分析分析程序后,可以查看这些函数的反混淆效果.

先看`cff`的原始代码

```c
unsigned int target_function(unsigned int n)
__attribute__((__annotate__(("fla"))))
__attribute((noinline))
{
unsigned int mod = n % 4;
unsigned int result = 0;
if (mod == 0) result = (n | 0xBAAAD0BF) * (2 ^ n);
else if (mod == 1) result = (n & 0xBAAAD0BF) * (3 + n);
else if (mod == 2) result = (n ^ 0xBAAAD0BF) * (4 | n);
else result = (n + 0xBAAAD0BF) * (5 & n);
return result;
}
```

还原后

```c
unsigned int __fastcall target_function(unsigned int n)
{
unsigned int v2; // [xsp+Ch] [xbp-14h]
v2 = n & 3;
if ( (n & 3) == 0 )
return (n | 0xBAAAD0BF) * (n ^ 2);
if ( v2 == 1 )
return (n & 0xBAAAD0BF) * (n + 3);
if ( v2 == 2 )
return (n ^ 0xBAAAD0BF) * (n | 4);
return (n - 1163210561) * (n & 5);
}
```

`vdog`的`JNI_OnLoad`还原后

```c
jint JNI_OnLoad(JavaVM *vm, void *reserved)
{
int v2; // w9
_JNIEnv *v3; // x1
_BOOL4 v5; // w8
crazy *v6; // x0
int v7; // w8
crazy *v8; // x0
_JNIEnv *v9; // x1
const char *v10; // x0
_JNIEnv *v12; // x1
crazy *v13; // x0
char *v14; // x0
const char *v15; // x0
crazy *v16; // x0
_JNIEnv *v17; // x1
int v18; // w0
crazy *v19; // x8
unsigned int v20; // w0
crazy *v21; // x0
__int64 v23; // [xsp-FD0h] [xbp-1240h]
__int64 v24; // [xsp-FC0h] [xbp-1230h]
__int64 v25; // [xsp-FA0h] [xbp-1210h]
__int64 v26; // [xsp-7D0h] [xbp-A40h]
char *v27; // [xsp+8h] [xbp-268h]
void *v28; // [xsp+10h] [xbp-260h]
int v29; // [xsp+18h] [xbp-258h]
int v30; // [xsp+1Ch] [xbp-254h]
JavaVM *v31; // [xsp+20h] [xbp-250h]
__int64 *v32; // [xsp+28h] [xbp-248h]
__int64 *v33; // [xsp+30h] [xbp-240h]
crazy::String *v34; // [xsp+38h] [xbp-238h]
void (__fastcall **v35)(JavaVM *, void *); // [xsp+40h] [xbp-230h]
const char *v36; // [xsp+48h] [xbp-228h]
int v37; // [xsp+54h] [xbp-21Ch]
int v38; // [xsp+58h] [xbp-218h]
int v39; // [xsp+5Ch] [xbp-214h]
char *v40; // [xsp+60h] [xbp-210h]
__int64 *v41; // [xsp+68h] [xbp-208h]
const char *v42; // [xsp+70h] [xbp-200h]
int v43; // [xsp+7Ch] [xbp-1F4h]
__int64 *v44; // [xsp+80h] [xbp-1F0h]
FILE *v45; // [xsp+88h] [xbp-1E8h]
int v46; // [xsp+94h] [xbp-1DCh]
crazy *v47; // [xsp+98h] [xbp-1D8h]
__int64 v48; // [xsp+A0h] [xbp-1D0h]
int v49; // [xsp+A8h] [xbp-1C8h]
int v50; // [xsp+ACh] [xbp-1C4h]
crazy *v51; // [xsp+B0h] [xbp-1C0h]
char *v52; // [xsp+B8h] [xbp-1B8h]
crazy *v53; // [xsp+C0h] [xbp-1B0h]
crazy *v54; // [xsp+C8h] [xbp-1A8h]
_QWORD v55[3]; // [xsp+D0h] [xbp-1A0h]
_QWORD *v56; // [xsp+E8h] [xbp-188h]
const char *v57; // [xsp+F0h] [xbp-180h]
char *v58; // [xsp+F8h] [xbp-178h]
int v59; // [xsp+104h] [xbp-16Ch]
const char *v60; // [xsp+108h] [xbp-168h]
JavaVM *v61; // [xsp+110h] [xbp-160h]
_QWORD v62[32]; // [xsp+118h] [xbp-158h]
__int64 v63; // [xsp+218h] [xbp-58h]
v28 = reserved;
v31 = vm;
v63 = *(__QWORD *)off_DFF90;
v29 = v2;
v30 = v2;
v32 = &v26;
v33 = &v25;
v34 = (crazy::String *)&v24;
v35 = (void (__fastcall **)(JavaVM *, void *))&v23;
v54 = 0LL;
v61 = vm;
v55[0] = (char *)*vm + 48;
v62[0] = *(__QWORD *)v55[0];
if ( ((unsigned int (__fastcall *)(JavaVM *, crazy **, __int64))v62[0])(vm, &v54, 65540LL) != 0 )
return -1;
v60 = (const char *)v54;
v61 = *(JavaVM **)v54;
v55[0] = v61 + 219;
v62[0] = v61[219];
((void (*)(void))v62[0])();
v16 = v54;
*off_DFFF8 = (__int64)v54;
v18 = crazy::GetApiLevel(v16, v17);
v19 = v54;
*off_DFF18 = v18;
v47 = v19;
v10 = (const char *)crazy::GetPlatformVersion(v19, v9);
if ( strchr(v10, 77) != 0LL )
*off_DFF18 = 23;
v38 = *off_DFF18;
if ( v38 > 23 )
*off_DFEE8 = 1;
v49 = sub_2E738();
if ( v49 == 2 )
{
v53 = v54;
v7 = sub_76F60(v53) & 1 ? -1618151004 : -990688990;
if ( v7 > -990688991 )
return -1;
}
v44 = v32;
memset(v32, 0, 0x7D0u);
v27 = (char *)v32;
v20 = getpid();
sprintf(v27, "/proc/%d/cmdline", v20);
v45 = fopen(v27, "r");
if ( v45 != 0LL )
{
v41 = v33;
memset(v33, 0, 0x7D0u);
v42 = (const char *)v33;
fscanf(v45, "%s", v33);
fclose(v45);
v5 = strchr(v42, 58) == 0LL;
if ( v5 || (v52 = strstr(v42, "sg.bigo.enterprise.live:service"), v52 != 0LL) )
anti_debug_start();
}
v43 = *off_DFF18;
if ( v43 == 15 )
j_aop_init();
anti_section_hook();
v13 = (crazy *)crazy::checkSignature_1(v54, v12);
if ( ((unsigned __int8)v13 & 1) == 0 )
crazy::AbortProcess(v13);
v14 = (char *)sub_2E998();
v40 = v14;
if ( *v14 )
{
crazy::GetPackageName((crazy *)v14);
v62[0] = v34;
v36 = *(const char **)v34;
v6 = (crazy *)strcmp(v36, v40);
v37 = (int)v6;
if ( v37 != 0 )
crazy::AbortProcess(v6);
crazy::String::~String(v34);
}
v46 = sub_2E738();
if ( v46 == 1 )
{
v50 = sub_F688();
if ( v50 == 0 )
return -1;
}
v51 = v54;
v8 = (crazy *)crazy::checkdex_1(v54, v3);
if ( ((unsigned __int8)v8 & 1) == 0 )
crazy::AbortProcess(v8);
v39 = sub_E7EC(*off_DFE98[0], "JNI_OnLoad", v35);
if ( v39 != 0 )
(*v35)(v31, v28);
v15 = (const char *)sub_2E728();
v48 = strlen(v15);
if ( v48 != 0 )
{
v56 = v62;
v21 = (crazy *)memset(v62, 0, sizeof(v62));
crazy::GetPackageName(v21);
v61 = (JavaVM *)v55;
v57 = (const char *)v55[0];
crazy::String::~String((crazy::String *)v55);
v58 = (char *)v62;
v60 = (const char *)sub_2E728();
sprintf(v58, "/data/data/%s/.hide/%s", v57, v60);
v59 = remove(v58);
}
return 65540;
}
```

流程平坦化混淆是`ollvm`里面最难还原,最影响分析效率的一个混淆`pass`,最近花了点时间,终于把它搞掉了.

刚开始是打算使用`IDA`的微码进行反混淆,搜到了`Rolf Rolles`大佬的[HexRaysDeob](https://bbs.kanxue.com/elink@d88K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6d9L8$3I4X3f1X3!0D9L8r3g2K6i4K6u0r3d9r3g2^5f1X3q4&6M7@1c8W2L8$3u0Q4x3X3g2Y4K9i4b7`.)

还有它的`Python`版本[PyHexRaysDeob](https://bbs.kanxue.com/elink@03bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6k6r3q4H3P5i4c8Z5L8$3&6Q4x3V1k6H3P5h3S2W2P5s2u0S2P5i4y4V1k6h3!0T1i4K6u0W2k6$3W2@1).

研究了下发现微码`API`确实是太难用了,而且很多分析好像都得自己做,所以我暂时放弃该方案.

随后转到了`binja`, 发现这篇文章[Dissecting LLVM Obfuscator Part 1](https://bbs.kanxue.com/elink@07dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5M7r3W2K6i4K6u0W2k6h3y4Q4x3V1k6T1L8r3!0Y4i4K6u0r3k6r3W2K6M7$3g2U0N6r3W2G2L8W2)9J5k6r3I4D9N6X3#2Q4x3X3c8G2j5X3k6#2M7$3y4S2N6r3!0J5i4K6u0V1M7o6q4Q4x3V1j5`.`),貌似简单可行,修改了下,用无名侠大佬[基于Unicorn 的ARM64 OLLVM反混淆](https://bbs.pediy.com/thread-252321.htm)帖子的样本跑,失败了.

看来还是得自己干.

下文中反混淆用的样本是无名侠的`libvdog.so(vdog)`和自己编译的另外一个小程序(`cff`).

反混淆主要分两步:分析和修复.

## 分析

这里思路源于`llvm-defobfuscator`,熟悉它的话可以直接跳到后面修复内容.

流程平坦混淆代码结构:

```
...

state_var = n;

while(1) {

    switch (state_var) {

        case x:

            code_region_x

        case y:

            code_region_y

        case z:

            code_region_z

        case ...

    }

}

```

反混淆需要的信息主要有:

- `switch`语句各`case`的入口, 出口基本块.即上面各个`case`对应的代码区域(`code_region_xxx`).

- `case`编号和`case`入口对应关系

- 各`case`出口的跳转目标`case`.对于条件跳转,我们还得知道真假分支和目标`case`的对应关系.

这些信息通过`binja`还是很容易分析出来的.

`vdog`中的代码:

```
  43 @ 0007067c  bool cond:4_1 = x9.w9 != 0x76579ace

  44 @ 00070680  uint64_t x8_1 = zx.q(x9.w9)

  45 @ 00070684  if (cond:4_1) then 10 @ 0x704f0 else 47 @ 0x70688
```
