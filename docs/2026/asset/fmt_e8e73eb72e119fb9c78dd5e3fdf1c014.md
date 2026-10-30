
# [原创]CVE-2015-0057提权漏洞学习笔记

## 一.前言

### 1.漏洞描述

该漏洞是一个UAF漏洞，存在于`win32k!xxxEnableWndSBArrows`函数中。`xxxEnableWndSBArrows`函数会对`tagWND`对象的`pSBInfo`指针所指向的`tagSBINFO`结构体中的成员进行计算，但是在计算之前，`xxxEnableWndSBArrows`函数并没有验证`tagSBINFO`对象是否存在。如果`tagSBINFO`对象不存在，此时就会产生写入错误。通过内存布局，可以在释放的`tagSBINFO`对象中使用`tagPROPLIST`对象进行填充，那么`xxxEnableWndSBArrows`函数就会修改`tagPROPLIST`对象中的成员造成写溢出，利用这个写溢出可以实现任意地址写的操作，最终实现提权。然而，这个实现真的太难了，目前写不出exp利用它，希望以后可以。

### 2.实验环境

- **操作系统：**Win7 x86 sp1 专业版
- **编译器：**Visual Studio 2017
- **调试器：**IDA Pro，OllyDbg，WinDbg

## 二.漏洞分析

### 1.漏洞成因

`xxxEnableWndSBArrows`函数用于开启或关闭一个或两个(水平或垂直)滚动条控件的箭头，窗口对象`tagWND`中保存了指向保存滚动条对象的结构体指针。`tagWND`对象的结构体定义如下：

``` 
2: kd> dt -v win32k!tagWND

struct tagWND, 170 elements, 0xb0 bytes

   +0x000 head             : struct _THRDESKHEAD, 5 elements, 0x14 bytes

   +0x014 state            : Uint4B

   +0x018 state2           : Uint4B

   +0x01c ExStyle          : Uint4B

   +0x020 style            : Uint4B

   +0x024 hModule          : Ptr32 to Void

   +0x028 hMod16           : Uint2B

   +0x02a fnid             : Uint2B

   +0x02c spwndNext        : Ptr32 to struct tagWND, 170 elements, 0xb0 bytes

   +0x030 spwndPrev        : Ptr32 to struct tagWND, 170 elements, 0xb0 bytes

   +0x034 spwndParent      : Ptr32 to struct tagWND, 170 elements, 0xb0 bytes

   +0x038 spwndChild       : Ptr32 to struct tagWND, 170 elements, 0xb0 bytes

   +0x03c spwndOwner       : Ptr32 to struct tagWND, 170 elements, 0xb0 bytes

   +0x040 rcWindow         : struct tagRECT, 4 elements, 0x10 bytes

   +0x050 rcClient         : struct tagRECT, 4 elements, 0x10 bytes

   +0x060 lpfnWndProc      : Ptr32 to     long 

   +0x064 pcls             : Ptr32 to struct tagCLS, 25 elements, 0x5c bytes

   +0x068 hrgnUpdate       : Ptr32 to struct HRGN__, 1 elements, 0x4 bytes

   +0x06c ppropList        : Ptr32 to struct tagPROPLIST, 3 elements, 0x10 bytes

   +0x070 pSBInfo          : Ptr32 to struct tagSBINFO, 3 elements, 0x24 bytes

   +0x074 spmenuSys        : Ptr32 to struct tagMENU, 19 elements, 0x6c bytes

   +0x078 spmenu           : Ptr32 to struct tagMENU, 19 elements, 0x6c bytes

   +0x07c hrgnClip         : Ptr32 to struct HRGN__, 1 elements, 0x4 bytes

   +0x080 hrgnNewFrame     : Ptr32 to struct HRGN__, 1 elements, 0x4 bytes

   +0x084 strName          : struct _LARGE_UNICODE_STRING, 4 elements, 0xc bytes

   +0x090 cbwndExtra       : Int4B

   +0x094 spwndLastActive  : Ptr32 to struct tagWND, 170 elements, 0xb0 bytes

   +0x098 hImc             : Ptr32 to struct HIMC__, 1 elements, 0x4 bytes

   +0x09c dwUserData       : Uint4B

   +0x0a0 pActCtx          : Ptr32 to struct _ACTIVATION_CONTEXT, 0 elements, 0x0 bytes

   +0x0a4 pTransform       : Ptr32 to struct _D3DMATRIX, 16 elements, 0x40 bytes

   +0x0a8 spwndClipboardListenerNext : Ptr32 to struct tagWND, 170 elements, 0xb0 bytes

   +0x0ac ExStyle2         : Uint4B
```

其中偏移0x70保存的是`tagSBINFO`结构体指针，该结构体保存了滚动条对象的信息，结构体定义如下：

``` 
2: kd> dt -v win32k!tagSBINFO  -r

struct tagSBINFO, 3 elements, 0x24 bytes

   +0x000 WSBflags         : Int4B

   +0x004 Horz             : struct tagSBDATA, 4 elements, 0x10 bytes    // 保存水平滚动条的相关信息

      +0x000 posMin           : Int4B

      +0x004 posMax           : Int4B

      +0x008 page             : Int4B

      +0x00c pos              : Int4B

   +0x014 Vert             : struct tagSBDATA, 4 elements, 0x10 bytes    // 保存垂直滚动条的相关信息

      +0x000 posMin           : Int4B

      +0x004 posMax           : Int4B

      +0x008 page             : Int4B

      +0x00c pos              : Int4B
```

`xxxEnableWndSBArrows`函数的反汇编代码如下，第二个参数和第三个参数要设定为特定的值来达到漏洞点，暂时先忽略这两个参数。真正产生漏洞的是第一个参数，该参数就是要设置滚动条的窗口对象`tagWND`。在`xxxEnableWndSBArrows`函数开始会将`tagWND`对象的`tagSBINFO`对象地址取出，之后会调用`xxxDrawScrollBar`，该函数将会返回用户层，执行用户层的`ClientLoadLibrary`函数，之后，`xxxEnableWndSBArrows`函数会对`tagSBINFO`的第一个成员，即`WSBflags`成员进行计算。在对`WSBflags`成员进行计算的时候，`xxxEnableWndSBArrows`函数并没有验证`tagSBINFO`对象是否存在，而在调用`xxxDrawScrollBar`函数的时候，用户可以HOOK `ClientLoadLibrary`函数，在用户定义的函数中可以释放`tagWND`对象，导致`tagSBINFO`对象被释放，从而导致`xxxEnableWndSBArrows`函数对一个被释放的`tagSBINFO`对象的`WSBflags`成员进行计算，造成了错误。

``` 
signed int __stdcall xxxEnableWndSBArrows(int pTagWnd, int wSBflags, int wArrow)

{

  int var_pTagWnd; 

  int *SBInfo; 



  var_pTagWnd = pTagWnd;

  SBInfo = *(int **)(pTagWnd + 0x70);	// tagWND偏移0x70保存的是pSBInfo，对其进行解引用得到结构体tagSBINFO的地址

  

  if ( SBInfo )

  {

    // 取出tagSBINFO的第一个成员，即WSBflags

    WSBflags = *SBInfo;

  }

  

  if ( !wSBflags || wSBflags == 3 )	// 判断wSBflags是否不为0或等于3

  {

    if ( wArrow )		   	// 判断wArrow是否为0

      *SBInfo |= wArrow;		// 不为0，将tagSBINFO的第一个成员，即WSBflags的值与wArrow进行或运算

    else

      *SBInfo &= 0xFFFFFFFC;            // 为0则低两位清0

    if ( *SBInfo != WSBflags )          // 判断WSBflags是否发生更改

    {

      if ( *(_BYTE *)(var_pTagWnd + 0x14) & 4 )

      {

        // IsVisible将会对tagWND是否可见进行判断，可见则返回TRUE

        if ( !(*(_BYTE *)(var_pTagWnd + 0x23) & 0x20) && IsVisible(var_pTagWnd) )

          // 该函数的调用会返回到用户层执行代码,此时用户可以通过HOOK来释放tagWND从而导致对应的tagSBINFO对象也被释放	

          xxxDrawScrollBar((_DWORD *)var_pTagWnd, WinDC, 0);	

      }

    }

  }

  if ( wSBflags == 1 || wSBflags == 3)	// 判断wSBflags是否等于1或者等于3

  {

    // 对WSBflags进行计算，这里没有验证tagSBINFO是否存在，导致漏洞的产生

    *SBInfo = wArrow ? 4 * wArrow | *SBInfo : *SBInfo & 0xFFFFFFF3;	

  }

}
```

### 2.执行用户层函数

`xxxEnableWndSBArrows`函数通过`xxxDrawScrollBar`函数返回到用户层，`xxxDrawScrollBar`函数通过层层调用返回到用户层的。一步步跟进的话，会发现返回到用户层的调用流程是：`xxxDrawScrollBar`->`xxxDrawSB2`->`xxxGetColorObjects`->`xxxDefWindowProc`->`xxxLoadUserApiHook`->`xxLoadHmodIndex`->`ClientLoadLibrary`。在`ClientLoadLibrary`函数中，会调用`KeUserModeCallback`：

``` 
.text:BF895BD9                 lea     eax, [ebp+var_228]

.text:BF895BDF                 push    eax

.text:BF895BE0                 lea     eax, [ebp+var_22C]

.text:BF895BE6                 push    eax

.text:BF895BE7                 push    dword ptr [esi]

.text:BF895BE9                 push    esi

.text:BF895BEA                 push    41h

.text:BF895BEC                 call    ds:__imp__KeUserModeCallback@20 ; KeUserModeCallback(x,x,x,x,x)

.text:BF895BF2                 mov     edi, eax
```

`KeUserModeCallback`函数实现了win32k的回调机制，调用该函数可以返回到用户层执行用户层的函数，这些用户层的函数被预先设定在了回调函数表中，`KeUserModeCallback`函数的定义如下：

``` 
NTSTATUS

NTAPI

KeUserModeCallback(IN ULONG RoutineIndex,

                   IN PVOID Argument,

                   IN ULONG ArgumentLength,

                   OUT PVOID *Result,

                   OUT PULONG ResultLength)
```

回调函数表可以认为是一个数组，里面的每一个元素都保存了不同的用户层函数的函数地址。`KeUserModeCallback`函数的第一个参数就是该回调函数表的索引，通过这个索引就可以从回调函数表中找到要执行的用户层的函数，这里这个索引是0x41。

回调函数表的首地址，保存在了`PEB`结构体偏移0x2C的`KernelCallbackTable`成员中

``` 
kd> dt _PEB

ntdll!_PEB

   +0x02c KernelCallbackTable : Ptr32 Void
```

想要查看函数地址表中的内容，只需要随便将一个文件放入到OllyDbg中，在数据窗口首先通过`fs:[0x30]`获取`PEB`地址：

![](https://bbs.kanxue.com/upload/attach/202205/835440_69JBVNUHZ5KBBWQ.jpg)

在`PEB`偏移0x2C的地址就可以获取函数地址表的地址：

![](https://bbs.kanxue.com/upload/attach/202205/835440_HQANPHD6X7KPQ7D.jpg)

找到函数地址表的地址，就可以根据索引找到相应的函数，由于函数地址表所指向的是一个4字节的数组，索引值乘以4就得到相应的保存了函数地址的地址：

![](https://bbs.kanxue.com/upload/attach/202205/835440_72KFGMG7ZKE6ZAP.jpg)

该函数是`user32.dll`中的函数，根据这里的地址和`user32.dll`的加载地址就可以算出偏移，最后在IDA中可以看到这个函数是`ClientLoadLibrary`，只有一个参数且调用声明是`__stdcall`。

![](https://bbs.kanxue.com/upload/attach/202205/835440_AZE9EBXXMJ973US.jpg)

根据上面的过程，可以通过获取`PEB`中的函数地址表中相应的函数地址来实现HOOK操作，相应代码如下：
