
# [原创]windows下32位汇编语言学习笔记 第十章 内存管理部分 1

前面5 6 7 8 9章都是介绍windows界面设计有关的内容，这些内容大概看一下就可以，等需要用的时候再回过头来查。一次性记住的可能性不大。这些章节也没有什么难度，自己看看就可以。

我所关心的还是windows系统相关知识，说道界面设置，对RC资源再熟悉做界面还是Delphi,C++builder比较快速。

本章介绍了很多windows下关于内存管理的函数,书上有句话说的很好，不要去深究这些函数分配的内存具体放在内存寻址空间的什么地方,只需要知道什么时候该用什么类型的内存管理函数就可以。

## Global标准内存管理

`GlobalAlloc`函数使用`GMEM_MOVEABLE`参数返回的是个内存句柄，内容是一个地址，这个地址才是可以使用的内存块。比如：

`PVOID lpMem = GlobalAlloc(GHND, 1000);`

现在`lpMem`指向的内容是一个地址，必须通过`GlobalLock(lpMem)`函数返回这个指针才能使用。

实际上`GMEM_MOVEABLE`参数和`GMEM_FIXED`区别就是前者必须要经过`GlobalLock`才能使用，后者则通过返回的指针直接使用。

关于什么时候可移动，我没法测，不知道windows在什么情况下去移动这块内存。

`GlobalAlloc`分配内存是紧凑的，就是说分配的多个内存空间可以放在一个内存页面里，只要能放的下。

## Heap堆内存管理

`Heap`内存管理非配基本上和`Global`的用法相当,一般的用法都是`GetProcessHeap`得到进程默认堆，然后`HeapAlloc`分配。

## 虚拟内存管理

有一点必须知道，`VirtualAlloc`,所分配的空间都是页对齐的，分配1字节的内存空间也会占用4096字节的一个内存页，再次使用`VirtualAlloc`分配内存会从新的一页开始。

每个虚拟内存页面有三种状态：`Free`：自由状态,`Commit`：提交状态,`Reserve`：保留状态。

八种保护属性:

`PAGE_NOACCESS`,`PAGE_READONLY`,`PAGE_READWRITE`,`PAGE_EXECUTE`,`PAGE_EXECUTE_READ`,`PAGE_EXECUTE_READWRITE`,`PAGE_EXECUTE_WRITECOPY`,`PAGE_WRITECOPY`

这八种属性在第一章的笔记里有介绍。

虚拟内存页面还有4种类型

`Free`：空闲，没有提交也没有使用的。`Private`：私有，该区域的虚拟地址和系统页文件关联。`Image`：映像，改区域的虚拟内存地址和exe文件或dll文件关联。

`Mapped`：映射，改区域的内存和其他硬盘上的文件关联。

这些页面的属性，类型，状态，就是80386保护模式下，windows系统管理虚拟内存的机制。

虚拟内存管理函数中提供了几个可以操作其他进程虚拟地址的函数。

`LPVOID VirtualAllocEx(HANDLE hProess,LPVOID lpAddress,SIZE_T dwSize,DWORD flProtect)`

这个函数可以在别的进程空间保留或者提交一块内存。比如DLL的远程注入就是使用这个函数在目标进程开辟一块虚拟内存空间，将要注入的dll名通过`WriteProcessMemory`函数写进目标进程的虚拟内存空间，然后再通过`CreateRemoteThread`函数运行`LoadLibraryA`加载写入的dll文件，`LoadLibraryA`函数位于`kernel32.dll`中，由于每个进程运行都会加载这个dll，所以`LoadLibraryA`函数在不同进程中的地址是一样的，可以直接再目标进程使用。

比如下面的例子:

```
// 向目标进程地址空间写入DLL名称

DWORD dwSize, dwWritten;

dwSize = lstrlenA( lpszDll ) + 1;

LPVOID lpBuf = VirtualAllocEx( hProcess, NULL, dwSize, MEM_COMMIT, PAGE_READWRITE );

if ( NULL == lpBuf )

{

     CloseHandle( hProcess );

    // 失败处理

}

if ( WriteProcessMemory( hProcess, lpBuf, (LPVOID)lpszDll, dwSize, &dwWritten ) )

{

    // 要写入字节数与实际写入字节数不相等，仍属失败

    if ( dwWritten != dwSize )

     {

         VirtualFreeEx( hProcess, lpBuf, dwSize, MEM_DECOMMIT );

         CloseHandle( hProcess );

        // 失败处理

     }

}

else

{

     CloseHandle( hProcess );

    // 失败处理

}

// 使目标进程调用LoadLibrary，加载DLL

DWORD dwID;

LPVOID pFunc = LoadLibraryA;

HANDLE hThread = CreateRemoteThread( hProcess, NULL, 0, (LPTHREAD_START_ROUTINE)pFunc, lpBuf, 0, &dwID );
```

---

`DWORD VirtualQuery(PVOID pvAddress,PMEMORY_BASIC_INFORMATION lpBuffer,SIZE_T dwLength);`

这个函数可以用来查询`pvAddress`指定的内存空间的状态`State`,类型`Type`,和保护属性`Protect`,地址空间的属性保存在`MEMORY_BASIC_INFO`结构中。用这个函数，就可以写一个和Cheat Engine里一样的查看内存`Region`的程序。

下面这个小程序就是使用`VirtualQuery`函数实现查询当前进程中的内存区域状态。实现的功能比Cheat Engine里的还多了可以知道哪里的内存是进程堆，那里映射，映像了什么文件。

部分参考windows核心编程第10章的内容

```
;显示内存信息

.386

.model flat,stdcall

option casemap:none



include windows.inc

include kernel32.inc

includelib kernel32.lib

include macros.asm

include user32.inc

includelib user32.lib

include Psapi.inc

includelib Psapi.lib



IDD_DIALOG	equ 104

IDC_LIST	equ 1001	



.data?

	hInstance	dd ?

	hWinMain	dd ?

	hProcess	dd ?

.const

	msg			db	"%10p %15s %15s %10p %10s",0

	memfree		db	"MEM_FREE   ",0

	memcommit	db	"MEM_COMMIT ",0

	memreserve	db	"MEM_RESERVE",0

	unknow		db	'           ',0

	memimage	db	"MEM_IMAGE  ",0

	memmapped	db	"MEM_MAPPED ",0

	memprivate	db	"MEM_PRIVATE",0

	

.code

;内存状态

_FormatState proc wState

	mov eax,wState

	.if eax == MEM_COMMIT

		lea eax,offset memcommit

	.elseif eax == MEM_FREE

		lea eax,offset memfree

	.elseif eax == MEM_RESERVE

		lea eax,offset memreserve

	.else

		lea eax,offset unknow		

	.endif

	ret

_FormatState endp

;内存类型

_FormatType proc wType

	mov eax,wType

	.if	eax == MEM_IMAGE

		lea eax,offset memimage

	.elseif eax == MEM_MAPPED

		lea eax,offset memmapped

	.elseif eax == MEM_PRIVATE

		lea eax,offset memprivate

	.else

		lea eax,offset unknow				

	.endif

	ret

_FormatType endp

;保护属性

_FormatProtect proc wProtect,szBuffer

	mov eax,wProtect

	.if eax == PAGE_READONLY

		invoke wsprintf,szBuffer,offset CTXT('%s'),CTXT('-R--')

	.elseif eax == PAGE_READWRITE

		invoke wsprintf,szBuffer,offset CTXT('%s'),CTXT("-RW-")

	.elseif eax == PAGE_WRITECOPY

		invoke wsprintf,szBuffer,offset CTXT('%s'),CTXT("-RWC")

	.elseif eax == PAGE_EXECUTE

		invoke wsprintf,szBuffer,offset CTXT('%s'),CTXT("E---")

	.elseif eax == PAGE_EXECUTE_READ

		invoke wsprintf,szBuffer,offset CTXT('%s'),CTXT("ER--")

	.elseif eax == PAGE_EXECUTE_READWRITE

		invoke wsprintf,szBuffer,offset CTXT('%s'),CTXT("ERW-")

	.elseif eax == PAGE_EXECUTE_WRITECOPY

		invoke wsprintf,szBuffer,offset CTXT('%s'),CTXT("ERWC")

	.else

		invoke wsprintf,szBuffer,offset CTXT('%s'),CTXT("----")

	.endif

	ret

_FormatProtect endp



;显示内存信息

_ShowMemoryState proc uses ecx hwndLB

	local	@msg[1024]:byte

	local	@path[MAX_PATH]:byte

	local   @mbi:MEMORY_BASIC_INFORMATION

	local	@Ret:dword

	local	@szState:dword

	local	@szType:dword	

	local	@szProtect[5]:byte

	local	@pHeapAddress:dword

	

	;ebx作为下一个查询的内存地址

	xor ebx,ebx

_loopbegin:

	invoke VirtualQuery,ebx,addr @mbi,sizeof @mbi	

	mov @Ret,eax

	mov eax,@mbi.BaseAddress	

	invoke _FormatState,@mbi.State

	mov @szState,eax

	invoke _FormatType,@mbi.lType

	mov @szType,eax

	invoke _FormatProtect,@mbi.Protect,addr @szProtect

	invoke wsprintf,addr @msg,offset msg,@mbi.BaseAddress,@szState,@szType,@mbi.RegionSize,addr @szProtect

	

	;标志内存映射，映像文件

	.if @mbi.State != MEM_PRIVATE		

		invoke GetCurrentProcess

		mov hProcess,eax

		invoke GetMappedFileName,hProcess,ebx,addr @path,MAX_PATH

		.if eax

			invoke lstrcat,addr @msg,CTXT('          ')

			invoke lstrcat,addr @msg,addr @path

		.endif		

	.endif

	;标志堆地址

	invoke GetProcessHeap

	mov @pHeapAddress,eax

	.if  ebx==eax

		invoke lstrcat,addr @msg,CTXT('          ')

		invoke lstrcat,addr @msg,CTXT('Process Heap Address')

	.endif		

	invoke SendMessage,hwndLB,LB_ADDSTRING,0,addr @msg

	

	add ebx,@mbi.RegionSize

	cmp @Ret,sizeof @mbi

	je _loopbegin 

	ret

_ShowMemoryState endp



_DlgProc	proc hWnd,wMsg,wParam,lParam

	mov eax,wMsg

	.if eax == WM_CLOSE

		invoke EndDialog,hWnd,0

	.elseif	eax == WM_INITDIALOG

		invoke GetDlgItem,hWnd,IDC_LIST	

		invoke _ShowMemoryState,eax

	.else

		mov eax,FALSE

		ret

	.endif

	mov eax,TRUE

		ret

_DlgProc endp



start:

	invoke GetModuleHandle,NULL

	mov hInstance,eax

	invoke DialogBoxParam,hInstance,IDD_DIALOG,NULL,offset _DlgProc,WM_INITDIALOG

	invoke ExitProcess,NULL

end start
```

```
//资源文件 使用ResEdit编辑

// Generated by ResEdit 1.4.4.19

// Copyright (C) 2006-2008

// 810K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0W2M7$3g2V1K9i4c8Q4x3X3g2F1k6i4b7`.



#include "res.h"

#include <windows.h>

#include <commctrl.h>

#include <richedit.h>



//

// Dialog resources

//

IDD_DIALOG DIALOGEX 0, 0, 535, 252

STYLE DS_MODALFRAME | DS_SETFONT | WS_VISIBLE | WS_BORDER | WS_CAPTION | WS_DLGFRAME | WS_POPUP | WS_SYSMENU

EXSTYLE WS_EX_WINDOWEDGE

CAPTION "Dialog"

FONT 10, "Courier New", 400, 0, 0

BEGIN

    LISTBOX         IDC_LIST, 0, 0, 535, 250, WS_TABSTOP | WS_VSCROLL | LBS_NOINTEGRALHEIGHT | LBS_SORT | LBS_NOTIFY

END
```

通过循环调用`VirtualQuery`,通过返回的`mbi`结构获得所有进程地址空间的各种信息，从0地址开始查找，每次查找后把地址加上`mbi`结构的`RegionSize`(区域尺寸)再查找下一个区域的信息。

还可以通过`DWORD GetMappedFileName(HANDLE hProcess,LPVOID,lpv,LPTSTR lpFilename,DWORD nSize)`检查类型为`MEM_IMAGE`，`MEM_MAPPED`的内存区域加载文件的路径。

还可以通过`HANDLE GetProcessHeap(VOID)`获得进程堆地址。

程序里用到了Masm宏`CTEXT()`,用这个宏就不必每次使用个字符串都去`.data`或`.const`里定义，这个宏实际上就是实现再`.data` 或者`.const`定义一个字符串。

程序的运行结果如下：

![](https://bbs.kanxue.com/upload/attach/200905/82380_vuh0l7ttay7xf6b.jpg)

依次显示：`Address`,`State`,`Type`,`RegionSize`,`Protect`,其他描述

## 汇编的串指令
