
# [原创]线程初始化过程 PK 加载DLL过程(详细)

线程初始化过程 PK 加载DLL过程 ：

## 线程初始化过程

```asm
7C92E450 ; __stdcall KiUserApcDispatcher(x, x, x, x, x)
```

![](https://bbs.kanxue.com/upload/attach/201403/545453_qv4s94mljq9zw2p.jpg)

```asm
7C92E455 FF D0 call eax //这里eax是LdrInitializeThunk函数地址
```

![](https://bbs.kanxue.com/upload/attach/201403/545453_wbkckksw16shb53.jpg)

![](https://bbs.kanxue.com/upload/attach/201403/545453_jyy65u1k2bci4ti.jpg)

```asm
7C939962 E9 07 FF FF FF jmp __LdrpInitialize@12 ; _LdrpInitialize(x,x,x) //具体的实现代码在里面
```

**__LdrpInitialize@12的大体流程：**

```
__LdrpInitialize@12
{
    ...
    RtlEnterCriticalSection(&LdrpLoaderLock);
    if (Ldr == NULL)
    {
        call _LdrpInitializeProcess@20
    }
    else
    {
        call _LdrpInitializeThread@4 //↓↓↓解释
    }
    RtlLeaveCriticalSection(&LdrpLoaderLock);
    ...
}
```

**_LdrpInitializeThread@4的大体流程：**

```
_LdrpInitializeThread@4
{
    ...
    for (遍历Ldr链表)//以InMemoryOrderModuleList顺序
    {
        if ( !(Ldr->Flags & LDRP_DONT_CALL_FOR_THREADS) )//LDRP_DONT_CALL_FOR_THREADS被置位，就不会调用dll的oep和tls
        {
            if (Dll 有 tls)
            {
                LdrpCallTlsInitializers(LdrEntry->DllBase, DLL_THREAD_ATTACH);
            }
            LdrpCallInitRoutine(Ldr->EntryPoint, Ldr>DllBase, DLL_THREAD_ATTACH, NULL); //call dll的oep
        }
    }
    if (exe 有 tls)
    {
        LdrpCallTlsInitializers(Peb->ImageBaseAddress, DLL_THREAD_ATTACH);
    }
    ...
}
```

### __LdrpInitialize@12 的分析

⒈`RtlEnterCriticalSection(&LdrpLoaderLock);` //`LdrpLoaderLock`是一个全局锁，此锁还在`LdrpLoadDll`函数里面使用，这会造成一些有意思的情况↓

如果想在`DllMain`里面`CreateThread`创个线程，并且不让`DllMain`返回（白利用的节奏，怎么白利用自己玩吧，说出来被开...），还是会出现被`LdrpLoaderLock`全局锁卡主的情况。不想被卡主可以自己主动释放掉这把锁，锁的位置在`PEB+0x0A0`偏移处（`+0x0a0 LoaderLock : Ptr32 Void`），是在进程初始化的时候由全局变量`LdrpLoaderLock`赋给`PEB+0x0A0`的！！！

⒉在`__LdrpInitialize@12` 里面既有`LdrpInitializeProcess`，也有`LdrpInitializeThread`，系统是根据`PEB+0x0C`（`+0x00c Ldr : Ptr32 _PEB_LDR_DATA`）是否有指针，来决定调用`LdrpInitializeProcess`还是`LdrpInitializeThread`，进程初始化（第一个线程）时`Ldr`为`NULL`，此时会调用`LdrpInitializeProcess`；之后再创建线程时`Ldr`就不为`NULL`了，会调用`LdrpInitializeThread`。

⒊`LdrpInitializeThread`分析

①在`LdrpInitializeThread`里面，会遍历`Ldr`链表（以`InMemoryOrderModuleList`顺序）call每个`Dll`的`tls`回调（如果有）和`EntryPoint`，`EntryPoint`最终会call `DllMain`的`DLL_THREAD_ATTACH`，这里发现了一个有意思的`Flags`，`LDRP_DONT_CALL_FOR_THREADS`（`0x0040000`），在`_LDR_DATA_TABLE_ENTRY.flags & LDRP_DONT_CALL_FOR_THREADS`就不会调用指定`Dll`的`DllMain`（这是`DisableThreadLibraryCalls` 函数的基本原理）。

②call 完每个`Dll`的`tls`回调（如果有）和`EntryPoint`之后，会判断`exe`是否有`tls`，有的话会根据数据目录表的`tls`找到`callback`数组，按顺序call每个`callback`。

ps：①有`tls`的`exe`，每创一个新线程都会调二次`tls`的回调（`DLL_THREAD_ATTACH`一次，`DLL_THREAD_DETACH`一次；新线程永不结束例外...）；

②并且经测试，最初的进程初始化时，如果`tls`的第一次`DLL_PROCESS_ATTACH`失败（失败原因一般是进程还没初始化完成，某些`dll`的`IAT`未完成，call未完成的`IAT`会失败），会在`DLL_PROCESS_DETACH`的时候补一次`tls`的回调执行（进程初始化的`tls`只执行一次，`DLL_PROCESS_ATTACH`或`DLL_PROCESS_DETACH`；之后的新线程初始化时的`tls`会执行两次！！！）

③`LdrpCallTlsInitializers`内部call `tls callback`数组；`LdrpCallInitRoutine` 这个... （都是`LdrpInitializeThread`内部调的）

## Dll加载过程

**_LdrLoadDll@16的大体流程：**

```
_LdrLoadDll@16
{
    ...
    call _LdrLockLoaderLock@12 //内部调用RtlEnterCriticalSection(&LdrpLoaderLock);
    call _LdrpLoadDll@24 //↓↓↓解释
    call _LdrUnlockLoaderLock@8//内部调用RtlLeaveCriticalSection(&LdrpLoaderLock);
    ...
}
```

**_LdrpLoadDll@24的大体流程：**

```
_LdrpLoadDll@24
{
    ...
    call _LdrpMapDll@24//内部调NtCreateSection、NtMapViewOfSection；插入Ldr
    call _LdrpRunInitializeRoutines@4//初始化；理论上只会调用新load的dll的tls(如果有)和oep，但是代码太风骚了，可以做手脚↓↓↓解释
    ...
}
```

**_LdrpRunInitializeRoutines@4的大体流程：**

```
_LdrpRunInitializeRoutines@4
{
    ...
    for(遍历Ldr链表) //InInitializationOrderModuleList顺序
    {
        if ( !(Ldr->Flags & LDRP_ENTRY_PROCESSED) )//没有LDRP_ENTRY_PROCESSED这个标志位的就放入数组中
        {
            存到一个数组中ArrayBug; //理论上如果我们自己没改过Ldr->Flags，那么这个遍历最终只会有一个Ldr放入数组中，那就是最新Load的
            //那个dll的Ldr；但是我们可以把自己的dll(防别人注入的dll)去掉这个标志位！！！
        }
        Ldr->Flags |= LDRP_ENTRY_PROCESSED;
    }
    for(循环遍历ArrayBug数组)
    {
        LdrpCallInitRoutine(EntryPoint, LdrEntry->DllBase, DLL_PROCESS_ATTACH, Context);
    }
    ...
}
```

**LoadLibraryExW --->>> LdrLoadDll**

### ⒈ _LdrLoadDll@16 的分析

①`LdrLockLoaderLock(1, 0, &v11);` 内部调用了`RtlEnterCriticalSection(&LdrpLoaderLock);` 跟`__LdrpInitialize@12`里面共用一把锁

②`_LdrLoadDll@16 -> call _LdrpLoadDll@24 ; LdrpLoadDll(x,x,x,x,x,x)` //主要功能在`_LdrpLoadDll@24`内部实现

### ⒉ _LdrpLoadDll@24 的分析

①`call _LdrpMapDll@24 ; LdrpMapDll(x,x,x,x,x,x)` //总体的作用就是把`dll`映射进进程的地址空间
　　//内部的`LdrpCreateDllSection`是创建`NtCreateSection`创建区块对象；之后映射`NtMapViewOfSection`；
　　//再然后是调用`_LdrpAllocateDataTableEntry@4`、`_LdrpInsertMemoryTableEntry@4`等操作把这个`dll`的`_LDR_DATA_TABLE_ENTRY`结构插入到`Ldr`链表里

②在映射完之后，就是对`dll`进行初始化，call `_LdrpRunInitializeRoutines@4 ; LdrpRunInitializeRoutines(x)`
　　内部会调用`LdrpCallTlsInitializers`（调用`dll`的`tls`回调，如果有的话）、`LdrpCallInitRoutine`（调用`dll`的`EntryPoint`）
　　代码很风骚，具体自己结合`od`、`ida`、`reactos`看吧，风骚大了就出问题了，有一个有意思的`flags`，`LDRP_ENTRY_PROCESSED`（`0x0004000`），如果把某个`dll`这个标志位去掉，那么当有`dll`加载时，被去掉`LDRP_ENTRY_PROCESSED`标志位的`dll`的`oep`就会先于要加载的`dll`的`oep`执行！！！
　　这完全可以风骚的做到用自己的`dll`拦截其他人加载`dll`（`XP`下通过；`Win7`下如果只这样做，退出程序时会崩，发现是在`ExitProcess`前有 没有释放的内存，因为我们的`dll`等于是多次进入`DLL_PROCESS_ATTACH`初始化，办法我已经想到了，并且测试通过了，自己想办法吧，就是不让他申请那没用的内存就可以）

ps：`LdrpRunInitializeRoutines`函数的参数系统默认会传`NULL`，此时加载`dll`过程就不会调用`dll`的`tls`也不会调用`exe`的`tls`；
如果传非`NULL`，代码貌似会执行`dll`的`tls`（如果有）和`exe`的`tls`（如果有）。

![](https://bbs.kanxue.com/upload/attach/201403/545453_t0erk90yajpl5h6.jpg)

![](https://bbs.kanxue.com/upload/attach/201403/545453_jw3n5f58mbhj10g.jpg)

![](https://bbs.kanxue.com/upload/attach/201403/545453_bd9stelvmej6s8w.jpg)

[Release.zip](https://bbs.kanxue.com/attach-download-87936.htm)

↑↑↑先于要加载的dll弹框

xp,win7都通过，当然win7下还要改一下　　

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)

上传的附件：

- [1.jpg](javascript:void(0);)

  （35.88kb，30次下载）

- [2.jpg](javascript:void(0);)

  （26.74kb，19次下载）

- [3.jpg](javascript:void(0);)

  （63.13kb，11次下载）

- [first.dll测试代码.jpg](javascript:void(0);)

  （15.13kb，9次下载）

- [second.dll测试代码.jpg](javascript:void(0);)

  （10.38kb，8次下载）

- [exe测试代码.jpg](javascript:void(0);)

  （42.96kb，39次下载）

- [Release.zip](javascript:void(0);)

  （43.33kb，235次下载）
