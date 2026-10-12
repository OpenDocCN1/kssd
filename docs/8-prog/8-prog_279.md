
# HideToolz's Driver 逆向分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-129.htm`](https://www.pediy.com/kssd/pediy09/pediy09-129.htm)
>
> 对这个软件的 `AntiHook` 比较感兴趣，想看看他是怎么实现的。。。于是逆了一下，完成度 90%(那些我不感兴趣的函数就没逆了)
>
> 这个 Driver 没有设备对象，是通过 `ZwTerminateProcess` 来跟 ring3 程序进行通讯的。。。Driver 的其他很多地方实现得比较精妙，值得我们学习
>
> 其中有两个重要的数据结构
>
> 代码:
>
> ```c
> typedef struct _XPROCESS_LIST
> {
>   LIST_ENTRY     Entry;
>   ULONG         ProcessId;
>   ULONG         ParentProcessId;
>   PPEPROCESS    Process;
>   PPEPROCESS    ParentProcess;
>   BOOLEAN       IsProt;                //是否正受保护/隐藏
>   BOOLEAN       unknow4;
>   BOOLEAN       UseOrgKernelCallbackTable;
> } XPROCESS_LIST, *PXPROCESS_LIST;
>
> typedef struct _PARAMS
> {
>   ULONG         SelfSettings;         //by xIkUg/RCT 2007-04-29
>   ULONG         GlobalSettings;
>   ULONG         CallIndex;
>   PVOID         InputBuffer;
>   ULONG         InputBufferLength;
>   PVOID         OutputBuffer;
>   ULONG         OutputBufferLength;
> } PARAMS, *PPARAMS;
> ```

> 源代码就不放出来了。。。
>
> 感兴趣的朋友可以直接看 `idb` 文件。。。各种变量，结构，函数都详细的标出来了
>
> 想要使用其中某些功能就得靠你自己的本事来还原源代码了。。。嘿嘿。。。

`4dd466d0e1e0cc123afbe504da2462ca`
