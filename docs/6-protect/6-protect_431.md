
# GalaxyAP 过游戏反调试框架

> 原文：[`www.pediy.com/kssd/pediy12/142443.html`](https://www.pediy.com/kssd/pediy12/142443.html)

这个工程我以前用来在 XP 上跑过 DNF，DN，只为好玩目的，还没写过外挂。

声明：用这套代码干了什么坏事与我无关

`BypassAntiDebug.rar`

## 工程说明

1. `GalaxyAP` 应用程序
2. `HookPortBypass` 重加载内核（包括 `nt` 和 `win32k`）
3. `TEST_KIDISPAT` 自分发异常（朋友帮忙实现了部分，代码主要来源 WRK）

ps: `hookport` 代码抄网上的，来源 `http://www.m5home.com/bbs/forum.php`

## 主要实现

这套 GalaxyAP 工程主要实现了：

1. 重加载内核（借助 `hookport` 思路配合 `peloader`）
2. 自实现异常分发的几个主要函数
3. 自创建 `debugobject`，使用另一个 `object`
4. 也使用了另一个 `fastmute`
5. 自己发现吧。。。

PS: 驱动的加载有时候在游戏前好使有时候在启动游戏后好使

PPS：这货也只能在 XP 上跑了

## 配置框架

实现了 HOOK 框架，只需要按下面填写即可

```c
//////////////////////////////////////////////

//这是为了 HOOK 原来的 ntos，转到新的 os 中
VOID (WINAPI *DUMMYFUCK)(IN  PVOID   Object);
// VOID (WINAPI *PspUserThreadStartup)(IN  PVOID   Object);
// VOID (WINAPI *PspSystemThreadStartup)(IN  PVOID   Object);

HX_DYNC_FUNCTION dync_Old2New[]={
      DECL_DYNCFUN_HOOK_Old2New(PspUserThreadStartup),
        DECL_DYNCFUN_HOOK_Old2New(PspSystemThreadStartup),
        DECL_DYNCFUN_HOOK_Old2New(ObCloseHandle),
        DECL_DYNCFUN_HOOK_Old2New(PspProcessDelete),
        DECL_DYNCFUN_HOOK_Old2New(pIofCallDriver),
        DECL_DYNCFUN_HOOK_Old2New(KiTrap03),
        DECL_DYNCFUN_HOOK_Old2New(ObpCreateHandle),  //为了跳过 ObCheckObjectAccess
/*
        kd> dps nt!pIofCallDriver l8
        8054c400  804eedc8 nt!IopfCallDriver    //就搞这个。归类到 fengyue 驱动去   
        8054c404  804f12c0 nt!IopfCompleteRequest
        8054c408  804f0a00 nt!IopAllocateIrpPrivate
        8054c40c  804ef0e6 nt!IopFreeIrp
        8054c410  00000000
8054c414  00000000
*/
  //  DECL_DYNCFUN_HOOK_Old2New(KeStackAttachProcess),
  //  DECL_DYNCFUN_HOOK_Old2New(KeAttachProcess),
    };

//这个需要另外 HOOK 的函数 ，old os 中的，把这些不经过 hookport 的转到自己的实现中
HX_DYNC_FUNCTION dync_funs_hook[]={
        DECL_DYNCFUN_HOOK(DbgkCreateThread),
        DECL_DYNCFUN_HOOK(DbgkExitThread),
        DECL_DYNCFUN_HOOK(DbgkExitProcess),
        DECL_DYNCFUN_HOOK(DbgkMapViewOfSection),
        DECL_DYNCFUN_HOOK(DbgkpMarkProcessPeb),
  //  DECL_DYNCFUN_HOOK(NtCreateDebugObject),
    DECL_DYNCFUN_HOOK(DbgkForwardException),
//    DECL_DYNCFUN_HOOK(KiDispatchException),     
    //使用强大的特征码直接 patch KiDispatchException 中比较 debugport 部分
    //s -b 804d8000 806ce100 64 A1 24 01 00 00 8B 40  44 39 B8 BC 00 00 00
    };

//这是实现自己分发函数需要用到的
HX_DYNC_FUNCTION dync_funs[]={
    DECL_DYNCFUN(KeUserExceptionDispatcher),
    DECL_DYNCFUN(KeI386XMMIPresent),
    DECL_DYNCFUN(PsImageNotifyEnabled),
    DECL_DYNCFUN(PsNtDllPathName),
    DECL_DYNCFUN(KeFeatureBits),
    DECL_DYNCFUN(PsSystemDllBase),
    DECL_DYNCFUN(PsGetNextProcess),
    DECL_DYNCFUN(PsGetNextProcessThread),
```
