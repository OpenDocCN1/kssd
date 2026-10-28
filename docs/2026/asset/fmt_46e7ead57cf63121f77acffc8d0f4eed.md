
图形化工具下载地址：<a href="elink@210K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6n7L8r3q4U0K9@1W2z5g2o6y4Q4x3V1k6a6M7r3g2F1b7i4u0C8"><mark class="encrypted">3b1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6n7L8r3q4U0K9@1W2z5g2o6y4Q4x3V1k6a6M7r3g2F1b7i4u0C8</mark></a>，点击内核--进入内核模式--查看系统热键：

![](upload/attach/202007/857678_S9U87PM27UDAMGZ.png)

Thanks for reading...

本文未考虑XP，感兴趣的可以参看其它同学的文章。

* [《Windows热键注册》](https://bbs.pediy.com/thread-135455.htm)
* [《在内核里查看所有的快捷键》](https://bbs.pediy.com/thread-111585.htm)

# 一种枚举系统热键的思路及代码实现(Win7&Win10)

## 前言

现在使用全局快捷键的软件越来越多，经常遇到快捷键被占用的情况，想找出被谁占用了，网上找了一些工具，不过都不支持新版本的Win10。于是自己写了一个（支持Win7到最新的Win10 2004 32/64位，XP就不考虑了），本文顺带记录一下分析思路。

## 思路分析

```
说明：Win7/8和Win10的热键数据结构基本一致，本文以Win10 1903 x64作为分析对象，重点是在Hash表的搜索方法上。
```

众所周知，注册热键需要调用[`RegisterHotKey`](<https://bbs.kanxue.com/elink@8caK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8Y4N6A6L8Y4g2K6k6i4u0Q4x3V1k6F1k6W2)9J5k6s2N6A6L8Y4g2K6k6i4u0Q4x3X3c8J5k6h3N6A6M7%4c8W2M7X3S2G2N6r3E0W2P5b7`.`.)>)，首先看一下函数原型：

```
BOOL RegisterHotKey(
  HWND hWnd,
  int  id,
  UINT fsModifiers,
  UINT vk
);
```

- `hWnd` 窗口句柄
- `id` 热键ID
- `fsModifiers` 控制位，如Ctrl/Alt/Shift...
- `vk` 虚拟键码 [Virtual Key Codes](https://bbs.kanxue.com/elink@8acK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6V1k6i4y4C8N6r3!0H3i4K6u0r3K9h3&6H3N6i4c8V1k6i4k6Q4x3V1k6$3K9i4u0@1N6h3q4D9i4K6u0V1K9$3g2&6i4K6u0V1j5$3!0V1k6i4x3`.)

简单跟一下`RegisterHotKey`函数，到`ntdll!NtUserRegisterHotKey`，看一下`NtUserRegisterHotKey`函数原型：

```
BOOL APIENTRY NtUserRegisterHotKey(HWND hWnd,
  int     id,
  UINT     fsModifiers,
  UINT     vk 
);
```

可以看到Native的参数和上层API一致，往下则是Shadow SSDT，进入`win32k`，Win7/Win8/8.1还是`win32k.sys`，Win10已拆分成`win32k/win32kfull/win32kbase`三个模块，其中`NtUserRegisterHotKey`是`win32kfull`的导出函数。

打开IDA，定位到`NtUserRegisterHotKey`函数，其调用`RegisterHotKey`如下：

```
mov     r9d, edi
mov     dword ptr [rsp+48h+BugCheckParameter2], ebp ; BugCheckParameter2
mov     r8d, r14d
xor     edx, edx
mov     rcx, rax        ; struct tagWND *
call    _RegisterHotKey
```

IDA推导的参数显示有问题，跟一下参数来源，不难知道`RegisterHotKey`参数1是`hWnd`，参数2是`NULL`，参数3是`id`，参数4是`fsModifiers`，参数5是`vk`。

继续跟进`RegisterHotKey`，直接F5，有两段代码需要注意：

第一段很意图很明显，通过`FindHotKey`查找热键是否已经注册。

![](https://bbs.kanxue.com/upload/attach/202007/857678_42S9WJGHF47VQRQ.png)

第二段则是若`HotKey`未找到，则从`Win32kPool`中分配`HotKey`数据结构，并填充相应的字段，最后加入Hash表`gphkHashTable`中。接下来看看结构的填充方式以及Hash表的Index如何计算的。

![](https://bbs.kanxue.com/upload/attach/202007/857678_UTYTVY7ZNXHGMK4.png)

`v19`则是`HotKey`结构体，对照`RegisterHotKey`参数，很容易分析出下面的偏移代表的字段：

```
v27 = *(struct tagTHREADINFO **)gptiCurrent;

...

*(_QWORD *)v19 = v27;               // wndinfo

*(_DWORD *)(v19 + 32) = v30;        // id

*(_WORD *)(v19 + 26) = v12 | v22; // modifiers2

*(_WORD *)(v19 + 24) = v11;       // modifiers1

*(_DWORD *)(v19 + 28) = BugCheckParameter2; // 这里是vk

*(_QWORD *)(v19 + 8) = v29;   // callback

v24 = *(_BYTE *)(v19 + 28) & 0x7F;    // Hash表，0x80个Buckets，Index是vk取模7f

*(_QWORD *)(v19 + 40) = gphkHashTable[v24]; // 将之前的节点插入单链表

gphkHashTable[v24] = (struct tagHOTKEY * near *)v19; // 将节点插入Hash表
```

最终可得到结构如下：

```
typedef struct _THREADINFO {
    PETHREAD thread;
  //..省略其它字段..
} *PTHREADINFO;

typedef struct _WNDINFO {
    HWND wnd;
  //..省略其它字段..
} *PWNDINFO;

typedef struct _HOT_KEY {
    PTHREADINFO thdinfo;
    PVOID callback;
    PWNDINFO wndinfo;
    UINT16 modifiers1;        //eg:MOD_CONTROL(0x0002)
    UINT16 modifiers2;        //eg:MOD_NOREPEAT(0x4000)
    UINT32 vk;
    UINT32 id;
#ifdef _AMD64_
    PADDING32 pad;
#endif
    struct _HOT_KEY *slist;
  //..省略其它字段..
} HOT_KEY, * PHOT_KEY;
```

至此，热键数据结构已经分析清楚，本文目的是枚举热键，因此关键问题是如何定位`gphkHashTable`，通常能想到两种方式：

- 通过解析PDB符号定位，然而Windows符号服务器，它配拥有mirror吗。
- 通过代码特征码搜索，想到要兼容Win10各种版本头大。

思考片刻，想了一种搜索思路，既然`gphkHashTable`是全局Hash表，位于`DATA`段，0x80个Bucket，里面全是`HotKey`结构，那么可以校验`HotKey`的`vk%0x7F`得到Index来检查合法性。其次`win32k`的`DATA`段大小也比较合理，因此搜索范围也不大。如果能将表填充完，过滤出内核地址，再配合校验Hash表的`HotKey`的合法性，应该就能搜索到。

`RegHotkey`的代码都是一些内存结构运算，依赖少，因此可以注册0x80个`vk`从1到0x80的`vk`值来填满Hash表，如果注册成功就记录，枚举完后再取消注册，做清理工作。

如果找到Hash表，删除热键就很简单了，找到对应的`HotKey`，常规摘单链节点的操作即可。

## 代码及实现

由于代码全在内核层实现（WDK7601），并且调用者不是GUI线程、Win7注册热键的函数未导出，因此调用`RegHotKey`会繁琐点，整体流程如下：

- 找到当前session的`csrss`，插内核APC，实现切换到GUI线程。
- 遍历找`win32k`模块基址（Win7 `win32k.sys`，Win10 `win32kfull.sys`）
- 解析得到`.data`段地址区域
- 获取`NtUser\*Hotkey`函数地址，Win7从Shadow SSDT表中查，Win10从`win32kfull.sys`导出表中获取。
- 注册热键，1~0x80的`vk`值，记录注册成功的值。
- 搜索`data`段，首先过滤内核地址，搜索满足条件的0x80个区域，再校验是否满足`HotKey` Hash表条件。
- 取消注册成功的热键。
- 递归枚举热键列表，解析对应结构:`hWnd`、`hk`、`id`、`fsModifiers`。

下面分段解析，由于热键操作必须访问`win32k` session空间，而且`ThreadInfo`还必须存在，因此光`Attach`到GUI进程是不行的，最简单的方式就插APC到GUI线程。

```
NTSTATUS DriverEntry(PDRIVER_OBJECT drvobj, PUNICODE_STRING registry)
{
    NTSTATUS status;

    UNREFERENCED_PARAMETER(registry);

    KdPrint(("OpsHotkey Running..."));

    DoEnumHotkeys();

    return STATUS_SUCCESS;
}

NTSTATUS DoEnumHotkeys()
{
    NTSTATUS Status;
    PETHREAD Thread;
    PKAPC Apc = NULL;
    BOOLEAN Inserted;

  // 获取csrss进程ID
    ULONG csrss_pid = GetSessionProcessId();

  // 得到进程第一个线程
    Status = GetProcessFirstThread((ULONG)csrss_pid, &Thread);
    if (!NT_SUCCESS(Status)) {
        return Status;
    }

  // 分配内存
    Apc = (PKAPC)ExAllocatePoolWithTag(NonPagedPool, sizeof(KAPC), 'cpak');
    if (Apc == NULL) {
        ObDereferenceObject(Thread);
        return STATUS_UNSUCCESSFUL;
    }

  // 初始化APC对象
    KeInitializeApc(Apc,
        (PKTHREAD)Thread,
        OriginalApcEnvironment,
        &KernelApcRoutine,
        NULL,
        DoEnumHotkeysApc,
        KernelMode,
        NULL);

  // 插APC
    Inserted = KeInsertQueueApc(Apc, NULL, NULL, IO_NO_INCREMENT);
    if (Inserted) {
        KdPrint(("[%s] KeInsertQueueApc ok.", __FUNCTION__));
    } else {
        KdPrint(("[%s] KeInsertQueueApc failed.", __FUNCTION__));
        ExFreePool(Apc);
    }
    ObDereferenceObject(Thread);

    return Status;
}
```

接着获取`NtUser\*Hotkey`相关函数，注册热键填充Hash表后，后面就可以开始搜索搜索Hash表了，`HotKey`是由`win32k`从`NonPagedPool`中分配，因此用`MmIsAddressValid`检查无须担心换页问题，具体代码如下：
