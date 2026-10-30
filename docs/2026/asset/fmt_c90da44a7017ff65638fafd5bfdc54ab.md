
# [原创]CVE-2021-1732提权漏洞学习笔记

## 一.前言

### 1.漏洞描述

`win32kfull!xxxCreateWindowEx`函数创建窗口的过程中，当创建的窗口对象存在扩展内存的时候，会通过函数`KeUserModeCallback`返回用户层，申请需要的内存。返回到内核继续执行的时候，会将用户层函数中指定的地址保存到窗口对象偏移`0x128`的`pExtraBytes`成员中。当用户层对窗口调用`SetWindowLongPtr`函数的时候，函数会将`pExtraBytes`用于指定要写入的目标地址。通过劫持用户层函数的执行，可以让`SetWindowLongPtr`函数对不合法地址进行写入会产生BSOD，也可以通过计算来扩大其他窗口的`cbwndExtra`，从而实现任意地址读写，最终实现提权。

### 2.实验环境

- **操作系统：**Win10 x64 1909 专业版
- **编译器：**Visual Studio 2017
- **调试器：**IDA Pro, WinDbg

## 二.漏洞分析

### 1.关键结构体成员

新的Win10版本修改了比较多的`win32k*`中的结构体，并且没有导出。所以以下部分成员只能是通过推测得出，首先是保存线程信息的`tagTHREADINFO`结构体，其偏移`0x1C0`保存的是`tagDESKTOP`结构体：

```
1: kd> dt tagTHREADINFO

   +0x1C0 rpdesk           : Ptr64 tagDESKTOP
```

`tagDESKTOP`偏移`0x80`处保存的是`pheapDesktop`，该成员保存的是桌面堆的基址：

```
0: kd> dt tagDESKTOP

   +0x080 pheapDesktop     : Ptr64 tagWIN32HEAP
```

`tagWND`有了比较大的变化，窗口的扩展内存不在直接跟在`tagWND`之后，当偏移`0xE8`的`Flags`不包含`0x800`标记的时候，扩展内存的地址直接保存在`0x128`的`pExtraBytes`中，当`Flags`包含`0x800`标记的时候，扩展内存存在于桌面堆中，与桌面堆基址的偏移保存在了`0x128`的`pExtraBytes`中。偏移`0x28`指向了`tagWDNK`结构体，偏移`0x8`和`0x30`处保存了`0x28`所指向的地址于桌面堆地址的偏移：

```
2: kd> dt tagWND

   +0x000 h               : Ptr64 Void

   +0x008 DesktopOffset   : Uint8B

   +0x010 pti             : Ptr64 tagTHREADINFO

   +0x018 rpdesk          : Ptr64 tagDESKTOP

   +0x020 pSelf           : Ptr64 tagWND

   +0x028 ptagWNDK        : Ptr64 tagWNDK

   +0x030 DesktopOffset   : Uint8B

   +0x058 Left            : Uint4B

   +0x05C Right           : Uint4B       

   +0x098 spMenu          : Ptr64 tagMENU

   +0x0C8 cbwndExtra      : UInt4B

   +0x0E8 Flags	          : UInt4B

   +0x128 pExtraBytes     : Uint8B
```

偏移`0xA8`指向的`tagMENU`，这里只需要知道`tagMENU`结构体偏移`0x98`的`pSelf`指向的是`tagMENU`本身，而`0x28`的`tagWDNK`结构体如下：

```
struct tagWNDK

{

    ULONG64 hWnd;               //+0x00

    ULONG64 OffsetToDesktopHeap;//+0x08 tagWNDK相对桌面堆基址偏移

    ULONG64 state;              //+0x10

    DWORD dwExStyle;            //+0x18

    DWORD dwStyle;              //+0x1C

    BYTE gap[0x38];

    DWORD rectBar_Left;         //0x58

    DWORD rectBar_Top;          //0x5C

    BYTE gap1[0x68];

    ULONG64 cbWndExtra;         //+0xC8 窗口扩展内存的大小

    BYTE gap2[0x18];

    DWORD dwExtraFlag;          //+0xE8  决定SetWindowLong寻址模式

    BYTE gap3[0x10];            //+0xEC

    DWORD cbWndServerExtra;     //+0xFC

    BYTE gap5[0x28];

    ULONG64 pExtraBytes;    //+0x128 模式1：内核偏移量 模式2：用户态指针

};
```

### 2.HMAllocObject函数分析

该函数定义如下，当第三个参数`bType`指定为`TYPE_WINDOW(0x1)`的时候，就会用于创建窗口对象：

```
PVOID HMAllocObject(PTHREADINFO ptiOwner,

                    PDESKTOP pdeskSrc,

                    BYTE bType,

                    DWORD size);
```

函数的主要代码如下，

```
unsigned __int64 __fastcall HMAllocObject(__int64 ptiOwner, __int64 pdeskSrc, unsigned __int8 bType, unsigned int size)

{

  v9 = *((_WORD *)&gahti + 0xC * Type + 6);



  if ( (v9 & 0x10) != 0 && pdeskSrc )

  {

    if ( (int)IsDesktopAllocSupported() < 0 )

      goto LABEL_67;

    tagWnd = (unsigned __int64)HMAllocateUserOrIsolatedType(v5, v9, Type);	// 创建tagWND对象

    if ( !tagWnd )

      goto LABEL_67;

    ptagWNDk = DesktopAlloc(pdeskSrc,

                            *(unsigned int *)((char *)&gahti + v38 + 0x10),

                            ((unsigned __int8)Type << 16) | 5u);                // 创建tagWNDK对象

    *(_QWORD *)(tagWnd + 0x28) = ptagWNDk;					// tagWND->ptagWNDK赋值为创建的tagWNDK对象

    if ( !ptagWNDk )

    {

      HMFreeUserOrIsolatedType(v9, Type, (void *)tagWnd);

      goto LABEL_67;

    }

    LockObjectAssignment((void **)(tagWnd + 0x18), (void *)pdeskSrc);

    ptagWNDk = *(_QWORD *)(tagWnd + 0x28);

    *(_QWORD *)(tagWnd + 0x20) = tagWnd;					// 为tagWND->pSelf赋值

    *(_QWORD *)(tagWnd + 0x30) = ptagWNDk - *(_QWORD *)(pdeskSrc + 0x80);       // 将ptagWNDK与pheapDesktop的偏移赋值给tagWND偏移0x30处

  }

   

  hwnd = (int)v15 | (unsigned __int64)(*(unsigned __int16 *)((char *)qword_1C0215758

                                                           + v15 * (unsigned int)dword_1C0215760

                                                           + 0x1A) << 16);	// 获取窗口句柄

  *(_QWORD *)tagWnd = hwnd;							// 为tagWND->hWnd赋值

  if ( *(_DWORD *)((char *)&gahti + v38 + 0x10) )

  {

    ptagWNDk = *(_QWORD *)(tagWnd + 0x28);

    *(_QWORD *)ptagWNDk = hwnd;				       // 为ptagWNDK->hWnd赋值

    *(_QWORD *)(ptagWNDk + 8) = *(_QWORD *)(tagWnd + 0x30);    // 将ptagWNDK与pheapDesktop的偏移赋值给ptagWNDK偏移0x8处

  }

  

  return tagWND;

}
```

### 3.xxxCreateWindowEx函数分析

从`gptiCurrent`中获取`rpdesk`成员，之后调用`HMAllocateObject`函数来申请`tagWND`对象：

```
.text:00000001C003BCCB                 mov     rax, cs:__imp_gptiCurrent

.text:00000001C003BCD2                 mov     r14, [rax]

.text:00000001C003BCD5                 mov     [rsp+4E8h+var_488], r14

.text:00000001C003BCDA                 mov     [rsp+4E8h+var_370], r14

.text:00000001C003C1E2                 mov     r15, [rsp+4E8h+var_370]

.text:00000001C003BDDC                 mov     rax, [r14+1C0h]                  ; rax = tagTHREADINFO->rpdesk

.text:00000001C003BDE3                 mov     [rsp+4E8h+rpdesk], rax

		// 省略部分代码

.text:00000001C003C198                 xor     ebx, ebx

.text:00000001C003C1AC                 lea     esi, [rbx+1]	

.text:00000001C003C347                 mov     r9d, 150h     		; r9d = 0x150

.text:00000001C003C34D                 mov     r8b, sil      		; r8d = 1	

.text:00000001C003C350                 mov     rdx, [rsp+4E8h+pdesk]    ; rdx = rpdesk

.text:00000001C003C358                 mov     rcx, r15        		; rcx = ptiCurrent

.text:00000001C003C35B                 call    cs:__imp_HMAllocObject

.text:00000001C003C362                 nop     dword ptr [rax+rax+00h]

.text:00000001C003C367                 mov     r15, rax                 ; 将tagWND赋给r15

.text:00000001C003C36A                 mov     [rsp+4E8h+var_tagWND], rax

.text:00000001C003C372                 test    rax, rax

.text:00000001C003C375                 jnz     short loc_1C003C3BF
```

调用`tagWND::RedirectedFieldcbwndExtra<int>::operator!=`判断`cbWndExtra`来判断是否存在扩展内存，存在的话就会调用`xxxClientAlloWindowClassExtraBytes`来创建扩展内存：

```
.text:00000001C003CDDB                 mov     dword ptr [rsp+4E8h+var_3F8], edi ; edi=0

.text:00000001C003CDE2                 lea     rcx, [r15+0B1h]

.text:00000001C003CDE9                 lea     rdx, [rsp+4E8h+var_3F8]

.text:00000001C003CDF1                 call    ??9?$RedirectedFieldcbwndExtra@H@tagWND@@QEBAEAEBH@Z ; tagWND::RedirectedFieldcbwndExtra<int>::operator!=(int const &)

.text:00000001C003CDF6                 test    al, al

.text:00000001C003CDF8                 jz      short loc_1C003CE44

.text:00000001C003CDFA                 mov     rax, [r15+28h]  ; rax = tagWND->ptagWNDK

.text:00000001C003CDFE                 mov     ecx, [rax+0C8h] ; rcx = tagWNDK->cbwndExtra

.text:00000001C003CE04                 call    xxxClientAllocWindowClassExtraBytes

.text:00000001C003CE09                 mov     rcx, rax        ; 申请内存地址赋给ecx

.text:00000001C003CE0C                 mov     rax, [r15+28h]  ; rax = tagWND->ptagWNDK

.text:00000001C003CE10                 mov     [rax+128h], rcx ; 申请内存地址赋给tagWNDK->pExtraBytes
```

### 4.xxxClientAllocWindowClassExtraBytes函数分析

`xxxClientAllocWindowClassExtraBytes`函数的主要代码如下，函数会调用`KeUserModeCallback`来发起用户层的回调来申请内存。从用户层返回之后，函数会对输出长度及输出地址进行判断，通过判断后，就会将申请的内存地址返回：

```
const void *__fastcall xxxClientAllocWindowClassExtraBytes(SIZE_T Length)

{

  LODWORD(pInputBuffer) = Length;

  ret = KeUserModeCallback(0x7Bi64, &pInputBuffer, 4i64, &pOutputBuffer, &nOutLen);

  if ( ret < 0 || (_DWORD)nOutLen != 0x18 )     // 输出长度等于0x18

    return 0i64;

  v3 = pOutputBuffer;

  if ( pOutputBuffer + 1 < pOutputBuffer || (unsigned __int64)(pOutputBuffer + 1) > *(_QWORD *)MmUserProbeAddress )

    v3 = *(__int64 **)MmUserProbeAddress;

  pAllocBuffer = (const void *)*v3;

  ProbeForRead(pAllocBuffer, size, v5 != 0 ? 1 : 4);

  return pAllocBuffer;

}
```

用户层函数的实现则下所示，函数通过`RtlAllocateHeap`来申请需要大小的内存，之后将其放入`Result[0]`中，之后会通过`NtCallbackReturn`函数将申请的内存通过`Result`数组来申请的内存返回到内核层，第二个参数用来指定返回的数据的长度：
