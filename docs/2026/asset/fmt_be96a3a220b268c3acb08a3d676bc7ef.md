
| 1  2 | `art::DexFileLoader::open`(hook后：`0x6D39C`/`0x6D3E8`)  `art::OatFileManager::OpenDexFilesFromOat`(hook后：`0x6A2C0`/`0x6AF14`/`0x6B9B0`/`0x6C188`/`0x6CB5C`) |

### 0x6D4A0

patch掉`art::Runtime::IsVerificationEnabled`。

### 0x6DAD8

hook `art::DexFileVerifier::Verify`(hook后：`0x6D38C`/`0x6D394`，直接返回`1`)。

### 0x6E40C

hook `art::DexFileLoader::open`(hook后：`0x6D39C`/`0x6D3E8`)。

### 0x70410

hook下列函数：

|  |  |

| --- | --- |

| 1  2  3 | `art::DexFileVerifier::Verify`(hook后：`0x6EB04`/`0x6EB0C`/`0x6EB14`，直接返回`1`)  `art::DexFile::OpenMemory`(hook后：`0x74EE8`/`0x74E90`/`0x74F38`)  `Art::DexFile`(hook后：`0x74E30`/`0x74F88`) |

### 0x75054

hook `libdvm.so`中的函数，具体没仔细看，`0x6EB1C`/`0x74DEC`/`0x6FFBC`都是被hook后的实现。

### 0x75AA8

读`java.lang.DexCache.dexfile`（这个dexfile就是解压apk之后根目录的那个`classes.dex`）。

### 0x767F8

参数是so文件路径，打开该so文件。

### 0x76C8C

参数是`libart.so`中的一个函数，返回该函数地址。

### 0x76CCC

第一个参数是so中的函数名，第二个参数是so的相对路径，返回该函数在so中的地址。

### 0x76D90

参数是`libdexfile.so`中的一个函数，返回该函数地址。

### 0x76DD4

参数是`libjdwp.so`中的一个函数，返回该函数地址。

### 0x76E18

md5。

### 0x7783C

字符串解密函数。

### 0x79270

计算传入字符串的hash（不完全是md5）。

### 0x7A240

热补丁检测，特征字符串：

|  |  |

| --- | --- |

| 1  2  3  4  5 | `nuwa`  `andfix`  `hotfix`  `.RiskStu`  `tinker` |

### 0x80458

调用`0x79270`。

### 0x804A8

hook libc中的下列函数：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12 | `msync`(hook后：`0x78470`)  `close`(hook后：`0x7AF50`)  `munmap`(hook后：`0x7A568`)  `openat64`(hook后：`0x7DC48`)  `__open_2`(hook后：`0x7DC80`)  `_open64`(hook后：`0x7DCB8`)  `_openat_2`(hook后：`0x7DCF0`)  `ftruncate64`(hook后：`0x7DD30`)  `mmap64`(hook后：`0x7EF60`)  `pread64`(hook后：`0x7F5D0`)  `read`(hook后：`0x7F7DC`)  `write`(hook后：`0x8022C`) |

### 0x87F98

hook `libdvm.so`中的函数（类似于`0x44708`），具体没仔细看，`0x856C0`/`0x87F00`/`0x87F4C`都是被hook后的实现。

### 0x889B0

patch掉`art::Runtime::UseJitCompilation`。

### 0x8A794/0x8B71C/0x8B890

hook函数实现。

### 0x917E8

读`/proc/sys/fs/inotify/max_queued_watches`。

### 0x91848

读`/proc/sys/fs/inotify/max_user_instances`。

### 0x918A8

读`/proc/sys/fs/inotify/max_user_watches`。

### 0x95778

看起来好像是通过判断时间实现的反调试。

### 0x95A28

字符串查找函数。

### 0x95B9C

字符串解密函数。

### 0x95D60

socket连接。

### 0x96398

frida检测，读`/proc/self/task`，特征字符串：`gum-js-loop`；读`/proc/self/fd`，特征字符串`linjector`。

### 0x995D0

xposed检测，特征字符串：

|  |  |

| --- | --- |

| 1  2  3 | `.xposed.`  `xposedbridge`  `xposed_art` |

### 0x99D28

hook框架检测，特征字符串：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22 | `frida`  `ddi_hook`  `dexposed`  `substrate`  `adbi_hook`  `MSFindSymbol`  `hook_precall`  `hook_postcall`  `MSHookFunction`  `DexposedBridge`  `MSCloseFunction`  `dexstuff_loaddex`  `dexposedIsHooked`  `ALLINONEs_arthook`  `dexstuff_resolv_dvm`  `dexposedCallHandler`  `art_java_method_hook`  `artQuickToDispatcher`  `dexstuff_defineclass`  `dalvik_java_method_hook`  `art_quick_call_entrypoint`  `frida_agent_main` |

### 0x9C0BC

调用`0x96398`检测frida，`system_property_get` `ro.product.model`，调用`0x9FD88`检测xposed和自动脱壳机，hook `dlopen`(hook后：`0x9B89C`)和`ptrace`(hook后：`0x95BF8`)。

### 0x9CFCC

通过读取`/proc/%d/status`判断TracerPid等实现反调试。

### 0x9D878

通过读取`/proc/%d/wchan`判断是不是`ptrace_stop`实现反调试。

### 0x9DCF4

通过读取`/proc/%ld/task/%ld/status`判断TracerPid等实现反调试。

### 0x9ED44

通过`java.lang.StackTraceElement.getClassName`打印函数调用栈进行xposed检测。

### 0x9F770

通过`java.lang.ClassLoader.getSystemClassLoader.loadClass`打印类加载器进行xposed检测。

### 0x9FD88

调用`0x9ED44`和`0x9F770`，通过判断ServiceManager里是否有`user.xposed.system`进行xposed检测，然后检测自动脱壳机：

fart(2ddK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5j5h3&6T1K9h3&6Y4L8r3g2F1k6%4W2#2k6g2)9J5c8V1k6m8f1W2c8Q4x3U0V1`.FUPK3(4bbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3z5p5I4q4c8W2c8Q4x3V1k6r3g2g2m8w2x3#2)9J5z5b7`.
Youpk(fd8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6k6L8%4g2D9L8%4u0Q4x3V1k6k6L8%4g2H3K9#2)9J5z5b7`.
检测方法是判断下列类或者方法是否存在：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8 | `dumpMethodCode`  `fartthread`  `fart`  `android/app/fupk3/Fupk`  `android/app/fupk3/Global`  `android/app/fupk3/UpkConfig`  `android/app/fupk3/FRefInvoke`  `cn/youlor/Unpacker` |

### 0xA18D4

`getInstalledApplications`获取系统中安装的APP信息。

### 0xA7D3C

解密出字符串`Java`和`JNI_OnLoad`，hook了几个函数，被hook的原地址未知，新地址：`0xA43A0`/`0xA485C`/`0xA48F4`/`0xA54B0`；hook `dlsym`(hook后：`0xA4554`)和`dlopen`(hook后：`0xA4D30`)。

### 0xB4B94

hook libc中的下列函数：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10 | `write`(hook后：`0xAA2CC`)  `pwrite64`(hook后：`0xAA51C`)  `close`(hook后：`0xAA774`)  `read64`(hook后：`0xAAA9C`)  `openat64`(hook后：`0xAACB8`)  `__openat_2`(hook后：`0xAB6D4`)  `__open_2`(hook后：`0xAC0F4`)  `open64`(hook后：`0xACB10`)  `read`(hook后：`0xAFE18`)  `mmap64`(hook后：`0xB1C54`) |

`system_property_get` `debug.atrace.tags.enableflags`，hook `bionic_trace_begin`和`bionic_trace_end`(hook后：`0xA8EF4`和`0xA8EF8`，直接返回)，没有找到则hook `g_trace_marker_fd`(hook后：`0xA8EFC`，返回`-1`)。

这些hook是为了透明加密，具体没仔细看，之前论坛也有人分析过，估计应该没有太大的变化：[梆梆加固之透明加密分析](https://bbs.pediy.com/thread-226358.htm)。

### 0xB9BEC

sha1。

### 0xBAE64

md5init。

### 0xC3378

base64。

### 0xC5DDC

base64。

### 0xC7B18

APK签名相关。

### 0xD024C

sha1。

### 0xD1C04

sha1init。

### 0xD2E98

md5。

### 0xD6484

调用`0xD75A0`。

### 0xD5CDC

读`/proc/self/cmdline`。

### 0xD6578

hook `libdvm.so`中的函数（hook后：`0xD6988`）。

### 0xD68A8

根据`off_12EF10`处的值判断调用`0xD6484`还是`0xD6578`。

### 0xD75A0

hook `libaoc.so`中的函数（hook后：`0xD69BC`）。

### 0x3EA68

`JNI_OnLoad`。分析环境pixel4 android10，动态分析过程中一些没有被调用的函数不再分析。

1.初始化cpuabi字符串(`arm64`)于`0x12E7C8`

2.初始化so名字符串(`libDexHelper`)于`0x12EC38`

3.初始化字符串`com/secneo/apkwrapper/H`于`0x137B10`

4.调用`0x1E520`

5.

|  |  |

| --- | --- |
| 1  2  3  4 | `JNIEnv->FindClass(com/secneo/apkwrapper/H)`  `JNIEnv->GetStaticFieldID(com/secneo/apkwrapper/H.PKGNAMELjava/lang/String;)`  `JNIEnv->GetStaticObjectField(class com/secneo/apkwrapper/H, PKGNAME Ljava/lang/String; => "cn.missfresh.application")`  `JNIEnv->GetStringUtfChars("cn.missfresh.application")` |

6.将包名存于`0x138040`

7.

|  |  |

| --- | --- |
