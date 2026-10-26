
# [原创]DMM的DRM解密

大家好, 我又来破解电子书了.

话不多说, 这次搞的是DMM和DLsite的DRM保护. 有认识半仙的观众们可能要说了, 你咋老是盯着电子书电纸书一类的薅, 咋不去搞点硬货? 那没办法, 老了牙口不好只会吃软柿子嘛.

本来上次ida时候打算发一发爆破的优化的, 不过写着写着热度就过去了. 毕竟嘛, 10分钟优化到1秒, 别人看你的目光就是你花了几天搞这个还不如等九分钟这样的.

好了这次DMM的电子书, DRM保护系统使用的是CypherTec的CypherGuard, 包括了授权, 反截图, 文件解密三个模块, 分别是`ctlicis.dll`, `cguard.dll`, `ctcrypt.dll`. 这些dll有Themdia壳, exe无壳.

主程序架构是Qt+cocos2d, 应该是为了多平台支持的缘故. 果然它Mac版, 同样是Qt+cocos2d的.

众所周知, DRM一定存在的弱点就是, 本地在某个阶段存在解密数据. 对于图片构成的电子书讲, 可能是中间数据, 也可能是解码后的RGB像素等.

当然了, 这些做DRM的公司也不是吃干饭的, 一定会想办法比如返回后销毁原数据, 对抗apihook, 使用内嵌memcpy等等, 不过嘛再好的SDK也得写exe的程序员用, 你到了Qt和cocos2d这个大坑里, 哪怕exe程序员小心翼翼地使用抗hook的思维, 他们里面那些默认的对象/结构拷贝一类的还是`memcpy`/`memmove`, 让我们来看看有没有敏感信息.

简单的写了个Logger的dll, 取名`DMMLogger1`, 勾了`memcpy`, `memcpy_s`, 然后发现尺寸较大时候就`hexdump`到日志文件, 果不其然, 里面发现了JPEG的文件头, 一块一块的, 每块4k.

结合静态分析和尝试log一些`ctcrypt_94`的函数, 我发现4634D4这里的

```
    if ( !ctcrypt_94(*(_DWORD *)(this + 4148), workmem, size, this + 52, &procsize, rev1048, isscrap) )
```

这个栈上的`workmem`就是`memcpy`的内容来源, 也是4k一块, 第一块有JFIF文件头, 然后dump下执行`ctcrypt_94`之前的`workmem`, 再在加密的电子书文件`dmmb`里面搜索, 可以发现就是`dmmb`里面的内容.

这时候观众老爷可能就要想了, 啊, 这个破解笔记就到头了, 把dump下来的解密数据组装一下不就jpeg图出来了吗? 比起截图大法已经是原汁原味了.

对于强迫症患者来说, 算法呢, 没有算法, 睡不着觉啊.

刚才在`dmmb`里面搜索解密前数据时候我也没闲着, 用010Editor打开进程内存, 果不其然搜索解密后的JFIF头, 也能找到完整的jpeg解密后的内容, 其后跟随着解码后的RGB像素. 这应该是cocos2d或者是Qt包装后的对象, 这里也是一个入手点, 最早的时候我也是想过hook下`QImage`, 从那里抓; 不过现在已经不重要了.

我找到了更好的软柿子. 那就是Mac版的`DMMbookViewer.app`, 让我们来看看里面有什么敏感信息.

经过一番探索, 在主程序100082150地方发现这么一个函数

```
id __fastcall func_applylicense_by_authType(__int64 a1)

{

    unsigned int authtype; // eax

    id result; // rax

    id mgr; // rax

    id locallic; // rax

    _QWORD *v5; // rax

    id err[3]; // [rsp+8h] [rbp-18h] BYREF

 

    authtype = (unsigned int)objc_msgSend(*(id *)(*(_QWORD *)(a1 + 16) + 8LL), "authenticationType");

    if ( authtype == 2 )

    {

        func_load_service_to_capsulizedfile(a1);

        mgr = objc_msgSend(&OBJC_CLASS___CPSecurityLibraryManager, "sharedManager");

        // contentskey restore from ???

        locallic = objc_msgSend(mgr, "lookupLocalLicenseWithCapsulizedFile:error:", *(_QWORD *)(*(_QWORD *)(a1 + 16) + 8LL), err);

        if ( locallic )

        {

            // char __cdecl -[__CPIR_5V6QgdU9ocuNGcL3 applyLicense:error:](__CPIR_5V6QgdU9ocuNGcL3 *self, SEL a2, id a3, id *a4)

            result = objc_msgSend(*(id *)(*(_QWORD *)(a1 + 16) + 8LL), "applyLicense:error:", locallic, err);

            if ( !(_BYTE)result )

                func_throw_nserror((NSError *)err[0]);

        }

        else

        {

            // lookup server lic?

            result = (id)sub_100082020(a1, a1);

        }

        *(_BYTE *)(a1 + 24) = 1;

    }

    else

    {

        if ( authtype != 6 )

        {

            v5 = __cxa_allocate_exception(8uLL);

            *v5 = &`vtable for'drm::cypherguard::auth_type_not_supported + 2;

            __cxa_throw(v5, (struct type_info *)&`typeinfo for'drm::cypherguard::auth_type_not_supported, (void (__cdecl *)(void *))sub_10001BE70);

        }

        result = objc_msgSend(*(id *)(*(_QWORD *)(a1 + 16) + 8LL), "applyOfflineParameterByNoneWithError:", err);

        if ( !(_BYTE)result )

            func_throw_nserror((NSError *)err[0]);

    }

    return result;

}
```

这里可以看出, `authenticationType`分为2和6两种, case 2时尝试去取缓存到本地的授权信息, 实际测试中`dmmb`文件是走到这里的. `dmmb`文件被他们叫做capsulized file, 直译就是包了胶囊的文件.

刚才说到的windows版有一个`ctcrypt_94`函数会对来自`dmmb`文件中的内容进行解密, 然后我习惯性的hook了OSX下的`CCCryptorUpdate`, 在看到感兴趣的解密后的数据就设法让其崩溃.

我们写好dylib, 使用`DYLD_INSERT_LIBRARIES=./DMMLogger1.dylib ./DMMbookviewer`, 启动后打开一本书, 如愿以偿地崩了.

```
Thread 33 Crashed:

0   libDMMLogger1.dylib             0x000000010609ae3c my_CCCryptorUpdate(_CCCryptor*, void const*, unsigned long, void*, unsigned long, unsigned long*) + 188 (DMMLogger1.mm:140)

1   libcommonCrypto.dylib           0x00007fff83a70c46 CCCrypt + 126

2   jp.co.cyphertec.CypherGuardCore 0x00000001060ad243 0x1060a9000 + 16963

3   libdispatch.dylib               0x00007fff93c5093d _dispatch_call_block_and_release + 12

4   libdispatch.dylib               0x00007fff93c4540b _dispatch_client_callout + 8

5   libdispatch.dylib               0x00007fff93c4a03b _dispatch_queue_drain + 754

6   libdispatch.dylib               0x00007fff93c50707 _dispatch_queue_invoke + 549

7   libdispatch.dylib               0x00007fff93c48d53 _dispatch_root_queue_drain + 538

8   libdispatch.dylib               0x00007fff93c48b00 _dispatch_worker_thread3 + 91

9   libsystem_pthread.dylib         0x00007fff87e064de _pthread_wqthread + 1129

10  libsystem_pthread.dylib         0x00007fff87e04341 start_wqthread + 13
```

这个工作线程崩的不是地方, 因为这是一种Block调用, 是看不到发起方的, 不过崩溃也没白费,下一个线程恰好就是Block的发起方

```
Thread 34:

0   libsystem_kernel.dylib          0x00007fff87348612 __write_nocancel + 10

1   libsystem_c.dylib               0x00007fff8bcf61fa _swrite + 87

2   libsystem_c.dylib               0x00007fff8bcee725 __sflush + 87

3   libsystem_c.dylib               0x00007fff8bcf1039 __sfvwrite + 823

4   libsystem_c.dylib               0x00007fff8bcfb9bd __vfprintf + 17858

5   libsystem_c.dylib               0x00007fff8bd216c1 __v2printf + 669

6   libsystem_c.dylib               0x00007fff8bcf7382 vfprintf_l + 54

7   libDMMLogger1.dylib             0x0000000106099e93 applog + 611 (shelllogger.cpp:37)

8   libDMMLogger1.dylib             0x000000010609adbd my_CCCryptorUpdate(_CCCryptor*, void const*, unsigned long, void*, unsigned long, unsigned long*) + 61 (DMMLogger1.mm:134)

9   jp.co.cyphertec.CypherGuardCore 0x00000001060ad0e0 CypherGuardCore::AES_PCTS_Decrypt_Body(unsigned char vector[16]*, unsigned char vector[16]*, unsigned long, unsigned char vector[16] const*, unsigned long, unsigned char vector[16] const*, unsigned long long, CypherGuardCore::PCTS_IVCache*, unsigned char vector[16]*) + 248

10  Capsule                         0x000000011f13add4 0x11f12f000 + 48596

11  Capsule                         0x000000011f13aef0 0x11f12f000 + 48880

12  jp.co.cyphertec.CypherGuardCore 0x00000001060af5e9 0x1060a9000 + 26089

13  com.dmm.DMMbookviewer           0x0000000105ae1471 0x105a4d000 + 607345

14  com.dmm.DMMbookviewer           0x0000000105b63f1a 0x105a4d000 + 1142554

15  com.dmm.DMMbookviewer           0x0000000105c5b824 0x105a4d000 + 2156580

16  libsystem_pthread.dylib         0x00007fff87e0699d _pthread_body + 131

17  libsystem_pthread.dylib         0x00007fff87e0691a _pthread_start + 168

18  libsystem_pthread.dylib         0x00007fff87e04351 thread_start + 13
```

赶得早不如赶得巧, 忽略后面写日志的调用. 往回看, 有一个函数叫做`CypherGuardCore::AES_PCTS_Decrypt_Body`, 对应的库在`DMMbookviewer.app\Contents\Frameworks\CypherGuardCore.framework`, 没有加壳.

而调用方`Capsule`是加壳了的, 在`DMMbookviewer.app\Contents\PlugIns\Capsule.bundle\Contents\MacOS`中. 看来这个`CypherGuardCore`做加解密操作, 而`Capsule`做文件解析操作.

继续分析主程序可见, 在100214256处, 有如下代码加载`Capsule.bundle`:

```
    if ( !g_capsulebundle_loaded )

    {

        func_init_capsule_key();

        pluginpath = [[NSBundle mainBundle] builtInPlugInsPath];

        bundlepath = [pluginpath stringByAppendingPathComponent:@"Capsule.bundle"];

        newpath = func_decypt_bundle(bundlepath);

        bundle = [[NSBundle alloc] initWithPath:newpath];

        if ( ![bundle loadAndReturnError: &err] )

        {

            [bundle release];

            func_deletefile(newpath);

            [[NSException exceptionWithName:CPGlueException reason:@"Capsule Component did fail to load." userInfo:Nil] raise];

        }

        [bundle release];

        func_deletefile(newpath);

        g_capsulebundle_loaded = 1;

    }
```

`func_decypt_bundle`会在`/Users/Shared/.Cypher License Information`新建包含相对路径的临时文件, 并把原来目录中的capsule解密写入. 然后因为osx允许载入macho后删除, 运行起来后实际是找不到文件的. 它应该是对`NSBundle`加载的hook.

我们可以选择hook `NSBundle`的`loadAndReturnError`方法, 把解密后的文件复制出来, 不过我比较懒, 我hook的是`dlopen`, 然后检查路径是否在此目录下, 并且复制出来.

我们先来看`CypherGuardCore::AES_PCTS_Decrypt_Body`的参数, 好来hook它.
