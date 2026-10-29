
# [原创]全能HOOK框架 JNI NATIVE JAVA ART DALVIK

OneHook

目前比较流行的几个安卓HOOK方案，都有功能上的欠缺，有的不支持art模式，有的不支持jni层，有的不支持侵入HOOK。

所以OneHook诞生了！

**这是一个同时支持ART和Dalvik两种模式，理论上支持安卓4.0.3以上所有版本，同时支持JAVA和NATIVE层，使用全局注入技术的侵入式HOOK框架。**

本框架不需要额外的安装，可以静态编译到自己的APP中。

首先感谢看雪论坛中很多大牛提供的技术资料，没有你们的分享，就没有目前的这个OneHook的诞生。

**感谢ele7enxxh提供的Android Inline Hook模块**

**项目地址：756K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8r3f1%4k6h3&6^5P5r3S2Q4x3V1k6m8L8X3c8J5L8$3W2V1i4K6u0V1d9h3&6D9K9h3&6W2i4K6u0V1d9r3!0G2K9H3`.`.**

**项目博客: 8cbK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2D9k6e0N6W2L8Y4S2^5K9q4)9J5k6h3y4G2L8g2)9J5c8V1q4F1k6s2u0G2K9h3c8Q4x3X3c8m8M7X3#2Q4x3X3c8u0L8X3I4A6L8X3g2Q4x3X3c8t1L8$3!0C8i4K6u0W2K9s2c8E0L8l9`.`.**

**讨论帖: http://bbs.pediy.com/thread-205741.htm**

**感谢asLody提供的Legend模块**

**项目地址：2baK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7$3I4G2k6s2W2Q4x3V1k6D9k6h3N6W2L8X3b7`.**

**还感谢其他贡献知识的同学们。**

## 功能实现：

### 0x01

首先考虑如何进入系统，如何在合适的位置和时间启动我的模块。之前做WIN32的时候就有人说过，如果在WINDOWS启动前前运行我的代码，那么我无所不能。理论上，这句话是成立的。

所以在安卓系统，我也需要找一个相对足够早的位置来启动我的模块，这个位置我没有选择init进程，而是`libc.so`。

为什么？

因为安卓的底层是LINUX构建的，LINUX的底层就是C语言，而`libc.so`就是C语言的具体实现库，在这个位置上插入代码显然是合适的。而且在`libc.so`中去执行我们的代码，就不需要去考虑如何注入模块，因为安卓的所有进程都会主动加载`libc.so`。这是一个一举两得的方案，首先解决了启动代码，其次解决了注入问题。

原理/过程:

`so`文件的初始化工作 是在`init_array`段中定义的

![](http://bbs.pediy.com/upload/attach/201705/170818_ob1tg856zehwqlu.png)

`.init_array`段的起始位置为:0X000683C0

![](http://bbs.pediy.com/upload/attach/201705/170818_dgfqyi7teq68utq.png)

初始化时 会按照顺序去执行初始化函数

`_ZL14....`

`_ZL30....`

`_ZL29....`

`_ZL31....`

那么如果把`_ZL14__libc_preinitv`的指针重定向到一个物理地址，使之`dlopen`我们自己的`so`，就可以实现我们想要的功能。

![](http://bbs.pediy.com/upload/attach/201705/170818_3f7xpvsx93snl6i.png)

修改`init_array`的段的第一个指针 `_ZL14__libc_preinitv` ---> 0x55D24

![](http://bbs.pediy.com/upload/attach/201705/170818_ka95547x236dzwb.png)

指向位置的机器码

机器码对应的C语言代码

```
int _fun()

{

  dlopen((const char *)"libckis.so", 0);

  return __libc_preinit();

}
```

这样就可以顺利的在每个进程中加载`libckis.so`文件

关于如何修改`libc.so`文件使之加载自定义库文件的方法 可以参考我的另外一篇文章

http://bbs.pediy.com/thread-213043.htm

需要注意的是

1 在不同安卓版本中，`libc.so`是不一样的。

2 自定义的代码位置不一定必须在`.text`段中，在很多情况下`.text`段没有足够的空间让你去插入代码，我的解决方案是把这段代码放在`.rodata`段，如果放在`rodata`段，在运算地址偏移时，需要+0x10000

3 `libc.so`修改后需要妥善的push到手机的`/system/lib`目录下，要注意权限问题，否则手机会死机

### 0x02

现在我们已经进入了安卓的Native层，在这里，我们可以做很多事情，比如HOOK `fopen`来控制底层文件的访问 也可以HOOK `__system_property_get`来修改`ro`属性 等等....

文档可以参考 **360K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8r3f1%4k6h3&6^5P5r3S2Q4x3V1k6m8L8X3c8J5L8$3W2V1i4K6u0V1d9h3&6D9K9h3&6W2i4K6u0V1d9r3!0G2K9H3`.`.**

附一个简单的例子

```
#include <stdio.h>
#include <stdint.h>
#include <dlfcn.h>
#include "inlineHook.h"
int my_init(void) __attribute__((constructor));
typedef int (*t_system_property_get)(const char *name, char *value);
t_system_property_get __system_property_get=NULL;
int (*old__system_property_get)(const char *name, char *value)=NULL;
int new__system_property_get(const char *name, char *value)
{
if(strstr(name,"ro.serialno")) return strlen(strcpy(value,"AABBCC"));
    else return old__system_property_get(name,value);
}
int my_init(void)
{
void* libc=dlopen("/system/lib/libc.so",0);
__system_property_get=(t_system_property_get)dlsym(libc,"__system_property_get");
if(registerInlineHook((uint32_t)__system_property_get,(uint32_t) new__system_property_get,(uint32_t **) &old__system_property_get)!= ELE7EN_OK) printf("error find __system_property_get ");
else if (inlineHook((uint32_t) __system_property_get) != ELE7EN_OK) printf("error hook __system_property_get ");
return 0;
}
```

那么，我们现在已经可以完整的实现Native/JNI层的HOOK了。

在`adb shell`下 输入`getprop ro.serialno` 会得到返回值 AABBCC 如图:

![](http://bbs.pediy.com/upload/attach/201705/170818_p79ts6nldp8qfd2.png)

并且我们已经完成了了进入JAVA层的先决条件。

### 0x03

进入JAVA世界

论坛中如何从native层进入java层有很多种方案吗，在这里 我使用了ADBI的解决方案,在`libc.so`库中的`epoll_wait`函数挂钩，进而跳转到APP进程中去，然后使用`DexClassLoader`加载自己的`dex`文件，达到JAVA HOOK的目的

```
bool IsInToJava=false;
int (*oldepoll_wait)(int epfd, int events, int maxevents, int timeout)=NULL;
int newepoll_wait(int epfd, int events, int maxevents, int timeout)
{
if(!IsInToJava)
{
IsInToJava=true;
invoke_dex_method("/data/local/tmp/xxx.dex",path,"com.xxx.xxx.inject","main",AppName);
return oldepoll_wait(epfd,events,maxevents,timeout);
}
}
void my_init(void)
{
void* libc = dlopen("/system/lib/libc.so", 0);
void *hookepoll_wait = dlsym(libc, "epoll_wait");
if (registerInlineHook((uint32_t)hookepoll_wait, (uint32_t)newepoll_wait, (uint32_t **)&oldepoll_wait) != ELE7EN_OK) printf("error find epoll_wait\n");
else if (inlineHook((uint32_t)hookepoll_wait) != ELE7EN_OK) printf("error hook epoll_wait ");
}
```

在APP进程调用`epoll_wait`函数时 就会加载指定的`dex`文件，并且执行HOOK的初始化方法。

注意`epoll_wait`可能会被多次调用，我们只需要执行一次即可。

### 0x04

JAVA层HOOK

JAVA层的HOOK我使用了Legend的方案，因为已经进入了APP进程内，就和热补丁的流程是一样的。

```
public class inject{
    public static final String TAG = "CKIS_INJECT";    
    public static Context context;    
    public static void main(String packageName)
    {
        Log.e(TAG, "Dex Inject in by: "+packageName);        
        HookManager.getDefault().applyHooks(inject.class);        
        Log.e(TAG, "DEX called success:"+packageName);    
    }
    @Hook("android.telephony.TelephonyManager::getDeviceId")
    public static String TelephonyManager_getDeviceId(TelephonyManager thiz) 
    {
        return "123456789012345";    
    }
}
```

在这里 我们HOOK了`TelephonyManager.getDeviceId`方法，实现了系统的IMEI的修改。

在执行IMEI获取操作时 调用`getDeviceID`会获得123456789012345

![](http://bbs.pediy.com/upload/attach/201705/170818_qsv592gnb7c0rh3.png)

具体HOOK方案 也可以参考我另外一篇基于XPOSED框架的文章

http://bbs.pediy.com/thread-213042.htm

### 0x05

环境部署

本框架需要ROOT权限

1 首先读取`/system/lib/libs.so` 并在本地解析elf文件格式 添加加载自定义库的代码 然后再回写到`/system/lib/libc.so`目录下

2 释放DEXPOSED的运行库 `libdexposed.so` `libdexposed_l.so` `libdexposed_l51.so` 到`\system\lib\`或`\vendor\lib`目录下

3 释放`libckis.so`、`libjava.so`到`/system/lib`目录下

4 释放DEX文件到指定的目录下 本例为 `/data/local/tmp/xxx.dex`

5 重启生效

目前该框架的完成度已经接近90%,支持ART、DALVIK，支持JAVA、JNI/NATIVE

理论上此方案支持安卓4.0.3以上所有版本 但是Legend的HOOK模块支持

- Dalvik & Android 4.2
- Dalvik & Android 4.3
- Art & Android 5.0
- Art & Android 5.0.1
- Art & Android 5.1
- Art & Android 6.0
- Art & Android 6.0.1

所以考虑后期JAVA层可能会更换为可以支持7.0的AndFix

目前已测试的版本有

4.0.3

4.4.2

4.4.4

5.0

5.1

欢迎有兴趣的朋友一起交流

QQ:586593 迟到 by 2017 05 13

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
