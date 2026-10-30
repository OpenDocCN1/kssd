
# [原创]CVE-2016-0165提权漏洞学习笔记

# 一.前言

## 1.漏洞描述

该漏洞是一个整数溢出漏洞，存在于`win32k`中的`RGNMEMOBJ::vCreate`函数中。该函数在分配内存的时候，没有对申请的内存大小是否会发生整型溢出进行校验，这将导致分配的内存大小将远小于所期望的大小。但是，在后续的操作中，函数依然以所期望的大小来对内存进行操作，就会对申请的内存块相邻的内存块进行写入操作，最终在释放申请的内存块的时候会因为破坏了相邻内存块的`POOL_HEADER`产生`BSOD`。而通过适当的内存布局，在申请的内存块后跟着的是`BitMap`对象，越界写入操作修改`BitMap`中的关键成员可以实现任意地址读写最终可以实现提权。

## 2.实验环境

- **操作系统：** `Win7 x86 sp1` 专业版
- **编译器：** `Visual Studio 2017`
- **调试器:** `IDA Pro`，`WinDbg`

# 二.漏洞分析

## 1.漏洞原理

该漏洞的产生和`vCreate`函数的第二个参数有关，该参数是一个`EPATHOBJ`结构指针，该结构体的定义如下：

```
typedef struct _EPATHOBJ

{

  PATHOBJ  po;

  PPATH   pPath;

  CLIPOBJ   *pco;

} EPATHOBJ, *PEPATHOBJ;
```

第一个成员`po`是一个`PATHOBJ`结构体，该结构体的定义如下，其中第二个成员`cCurves`代表的是当前`EPATHOBJ`对象的曲线数目。

```
typedef struct _PATHOBJ {

  FLONG  fl;

  ULONG  cCurves;

} PATHOBJ;
```

以下是`vCreate`函数中与本次漏洞有关的关键代码。首先函数会调用`ExAllocatePoolWithTag`申请内存，而申请的内存的大小只验证了是否大于0，却没有验证是否发生了整型溢出，这就将导致当大小发生整型溢出的时候，`ExAllocatePoolWithTag`申请的内存大小会远小于期望的大小。

接着，函数将申请到的内存地址作为第三个参数传递给`vContructGET`函数，在这个函数中将会对申请到的内存进行读写操作，可是整型溢出，导致申请到的实际内存非常小，而`vContructGET`函数却以期望申请到的大小来读写内存，导致将会越界写入其他内存空间而产生了错误。

最后，如果内存申请成功，`vCreate`函数会调用`ExFreePoolWithTag`函数来释放内存。

```
void __thiscall RGNMEMOBJ::vCreate(RGNMEMOBJ *this, struct EPATHOBJ *ePathObj, char a3, struct _RECTL *a4)

{

  unsigned int uiCurves;

  struct EDGE *pEdge1;

  int bFree;

  PVOID pEdge; 



  uiCurves = *((_DWORD *)ePathObj + 1);

  bFree = uiCurves;

  

  if ( uiCurves >= 0x14 )

  {

    if ( 0x28 * (uiCurves + 1) )        // 此处并没有验证(uiCurves + 1) * 0x28是否会发生整型溢出

    {

      v12 = ExAllocatePoolWithTag(PagedPoolSession, 0x28 * (uiCurves + 1), 'ngrG');    // 申请内存

      v7 = a4;

      pEdge = v12;

    }

  }

  

  if ( v37 >= v14 )

  {

    if ( v37 < 0 || //这部分计算后文会讲 )

    {

      pEdge1 = (struct EDGE *)pEdge;

      vConstructGET(ePathObj, (struct EDGE *)&v30, pEdge1, a4);    // 触发漏洞的地方

   }

   

  if ( bFree )            // 申请成功了就会释放申请的内存

    ExFreePoolWithTag(pEdge, 0);

}
```

函数`vConstructGET`通过结构体`EDGE`根据路径建立全局边表，结构体`EDGE`的定义如下。该结构共有`0x28`个字节，由此可以得知上面申请内存的时候是要申请可以容纳`uiCurves+1`个`EDGE`结构体的内存。

```
typedef struct _EDGE {

    PVOID pNext;            //<[00,04]

    INT iScansLeft;         //<[04,04]

    INT X;                  //<[08,04]

    INT Y;                  //<[0C,04]

    INT iErrorTerm;         //<[10,04]

    INT iErrorAdjustUp;     //<[14,04]

    INT iErrorAdjustDown;   //<[18,04]

    INT iXWhole;            //<[1C,04]

    INT iXDirection;        //<[20,04]

    INT iWindingDirection;  //<[24,04]

} EDGE, *PEDGE;
```

函数`vConstructGET`通过`AddEdgeToGET`来实现边表的建立，这里要注意的是第三个参数`pEdge`将作为第二个参数传入`AddEdgeToGET`函数，也就是将申请的内存作为第二个参数来调用`AddEdgeToGET`。

```
void __stdcall vConstructGET(struct EPATHOBJ *a1, struct EDGE *pEdgeHead, struct EDGE *pEdge, struct _RECTL *a4)

{

  struct EDGE *pEdgeHead1; // ebx

  int *v5; // edi

  struct _POINTFIX *v6; // ecx

  struct EDGE *pEdge1; // eax

  struct _POINTFIX *v8; // esi

  struct EPATHOBJ *i; // [esp+10h] [ebp+8h]

  struct EDGE *pEdgeHeada; // [esp+14h] [ebp+Ch]



  pEdgeHead1 = pEdgeHead;

  *(_DWORD *)pEdgeHead1 = pEdgeHead1;

  

  if ( v5 )

  {

    pEdge1 = pEdge;

    do

    {

      for (i = (struct EPATHOBJ *)&v5[2 * v5[3] + 4]; v8 < i; v8 = (struct _POINTFIX *)((char *)v8 + 8))

      {

        pEdge1 = AddEdgeToGET(pEdgeHead1, pEdge1, v6, v8, a4);

        v6 = v8;

      }

      if ( v5[2] & 2 )

      {

        pEdge1 = AddEdgeToGET(pEdgeHead1, pEdge1, v6, pEdgeHeada, a4);

        v6 = 0;

      }

      v5 = (int *)*v5;

    }

    while ( v5 );

  }

}
```

函数中`AddEdgeToGET`实现添加的代码如下，代码有两个内容，首先会判断要加入的边中的结束点的坐标的Y值和起始坐标的Y值是否相同，如果相同则返回。如果不相同，就会通过`EDGE`结构体中的`pNext`实现添加，最终会返回`pEdge`中的下一个`EDGE`结构体，然后会在函数`vConstructGET`中继续添加。

```
struct EDGE *__stdcall AddEdgeToGET(struct EDGE *pEdgeHead, struct EDGE *pEdge, struct _POINTFIX *a3, struct _POINTFIX *a4, struct _RECTL *pRectl)

{

  LONG yEnd; // edi

  struct EDGE *pEdge1; // ecx

  int Edge_Y; // esi

  int EdgeY; // ebx

  struct EDGE *pEdgeHead1; // edx

  int pEdgeNext; // eax

  int HeadY; // esi

  struct EDGE *pEdgea; // [esp+20h] [ebp+Ch]

  struct _POINTFIX *yStart; // [esp+28h] [ebp+14h]

  

    pEdgea = 0;

  if ( pRectl )

  {

    if ( yEnd < pRectl->top || (signed int)yStart > pRectl->bottom )

      return pEdge1;

    if ( (signed int)yStart < pRectl->top )

    {

      pEdgea = (struct EDGE *)1;

      v25 = yStart;

      yStart = (struct _POINTFIX *)pRectl->top;

    }

    if ( yEnd > pRectl->bottom )

      yEnd = pRectl->bottom;

  }

  Edge_Y = ((signed int)yStart + 0xF) >> 4;

  *((_DWORD *)pEdge1 + 3) = Edge_Y;

  *((_DWORD *)pEdge1 + 1) = ((yEnd + 0xF) >> 4) - Edge_Y;

  if ( ((yEnd + 0xF) >> 4) - Edge_Y <= 0 )        // 点的y坐标是否相同，如果相同则跳过添加

    return pEdge1;

  

  // 通过pNext将EDGE加入到

  while ( 1 )

  {

    pEdgeNext = *(_DWORD *)pEdgeHead1;

    Y = *(_DWORD *)(*(_DWORD *)pEdgeHead1 + 0xC);

    if ( v21 <= Y && (v21 != Y || v20 <= *(_DWORD *)(pEdgeNext + 8)) )

      break;

    pEdgeHead1 = *(struct EDGE **)pEdgeHead1;

  }

  *(_DWORD *)pEdge1 = pEdgeNext;

  *(_DWORD *)pEdgeHead1 = pEdge1;

  return (struct EDGE *)((char *)pEdge1 + 0x28);

}
```

其中的几个需要使用的结构体定义如下：

```
typedef struct tagRECT {

	LONG left;

	LONG top;

	LONG right;

	LONG bottom;

} RECT,*PRECT,*NPRECT,*LPRECT;



typedef LONG FIX;



typedef struct _POINTFIX {

  FIX  x;

  FIX  y;

} POINTFIX, *PPOINTFIX;
```

## 2.漏洞验证

由上面分析可以知道，这个漏洞产生的原因是`vCreate`函数没有对`uiCurves+1`和`0x28`相乘的值是否发生整型溢出导致的。所以，要验证该漏洞就需要解决两个问题，首先是要调用`vCreate`函数，其次是要在调用这个函数的时候第二个参数中保存的`uiCurves`可以让后面的计算产生整型溢出。

可以通过函数`PathToRegion`来调用`vCreate`函数，该函数定义如下。

```
WINGDIAPI HRGN WINAPI PathToRegion(__in HDC hdc);
```

该函数只有设备句柄一个参数，该设备句柄对应的对象中的`uiCurves`将会被用于申请内存。在用户层，可以通过`GetDC`，`BeginPath`，`EndPath`这三个参数来实现获取设备句柄，将设备与当前上下文绑定或解绑，函数定义如下：

```
WINUSERAPI

HDC

WINAPI

GetDC(__in_opt HWND hWnd);

    

WINGDIAPI BOOL WINAPI BeginPath(__in HDC hdc);



WINGDIAPI BOOL WINAPI EndPath(__in HDC hdc);
```

对于`GetDC`，可以通过传入`NULL`来获取桌面窗口的设备句柄，通过`BeginPath`将设备与当前上下文绑定就可以对桌面窗口的设备对象进行操作。接下来就可以通过`PolylineTo`函数来增加获取的桌面窗口设备句柄的`uiCurvers`，该函数定义如下：

```
WINGDIAPI BOOL  WINAPI PolylineTo(__in HDC hdc,

                   __in_ecount(cpt) CONST POINT * apt, 

                   __in DWORD cpt);
```

函数第一个参数即使要操作的设备句柄，第二个参数是`POINT`指针，指向要增加的曲线的坐标，第三个参数为坐标个数。其中，`POINT`结构体定义如下：

```
typedef struct tagPOINT

{

    LONG  x;

    LONG  y;

} POINT, *PPOINT, NEAR *NPPOINT, FAR *LPPOINT;
```

该函数通过`POINT`数组中的值来添加边，如果现在输入的点为`(0,0)`, `(1, 1)`, `(2, 2)`这三个点，`PolylineTo`函数就会将`(0, 0)`与`(1, 1)`, `(1, 1)`与`(2, 2)`相连，这样就增加了两条边。而在`vCreate`函数中，又会将曲线闭合，也就是会将 `(2, 2)`与`(0, 0)`连起来，因此`uiCurvers`就会增加3，所以`POINT`数组输入几个点就会增加多少线条。根据上面申请内存时候通过`(uiCurvers + 1) * 0x28`可以算出，只要`uiCurvers`的值为`0x6666665`就会发生整型溢出(`0xFFFFFFFF / 0x28 - 1`)。

但是，在要触发漏洞还需要绕过三处限制，第一处限制是在通过`PolylineTo`来增加设备对象的`uiCurvers`的时候，需要注意在`PolylineTo`中有对`cpt`参数，也就是要添加的边的个数进行判断，判断代码如下。

```
cpt = 0;

for ( i = 0; ; ++i )

{

    v13 = cpt;

    if ( i >= ccpt )

    break;

    cpt += *(Dst + i);

}

if ( cpt > 0x4E2000 )    // 一次增加的线条不能超过0x4E2000，否则会增加失败

    goto LABEL_56;
```

第二处限制是调用`PathToRegion`后，会调用`win32k`中的`NtGdiPathToRegion`函数，在该函数中，要注意和不同`y`值的点不能过大，否则会导致内存资源不足，代码如下：

```
  if ( v8 )

  {

    // 省略成功调用的代码

  }

  else

  {

    // 内存资源不足，GetLastError将会得到8

    EngSetLastError(8);

    v3 = dcObj;

    *(_DWORD *)(v3 + 0x70) &= 0xFFFFFFFE;

    *(_DWORD *)(v3 + 0x6C) = 0;

  }
```

第三处限制是在`vCreate`函数中，在该函数中会将不同`y`值大小的点与`0x20`相乘并加上`0x1F8`，最终的值与`0x7FFFFFFF`进行判断，如果大于该值则会跳转，而不调用`vConstructGET`函数。这部分就是在上述对`vCreate`函数分析的时候，调用`vConstructGET`函数前的`if`判断语句中省略的部分。
