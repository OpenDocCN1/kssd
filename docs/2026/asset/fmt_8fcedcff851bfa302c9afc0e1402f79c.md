
# [原创] cve-2012-0158漏洞分析

参考《漏洞战争》分析下，记下自己的心得，主要还是汇编代码比较麻烦。

环境是xpsp3+word2003(11.8324.8324):

打开windbg、word软件，windbg附加word软件进程，word打开`poc.doc`文件，发现windbg已经断下，`eip`指向0x41414141，说明已经发生了溢出，`eip`被控制了。

```
0:009> g

(15c.510): Access violation - code c0000005 (first chance)

First chance exceptions are reported before any exception handling.

This exception may be expected and handled.

eax=00000000 ebx=024957b8 ecx=7c93005d edx=00140608 esi=0022ed4c edi=00000000

eip=41414141 esp=0012170c ebp=00000000 iopl=0         nv up ei pl zr na pe nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00010246

41414141 ??              ???
```

看一下栈回溯:

```
0:000> kvn

 # ChildEBP RetAddr  Args to Child

WARNING: Frame IP not in any known module. Following frames may be wrong.

00 00121708 00000000 00000000 00000000 00000000 0x41414141

0:000> kb

ChildEBP RetAddr  Args to Child

WARNING: Frame IP not in any known module. Following frames may be wrong.

00121708 00000000 00000000 00000000 00000000 0x41414141
```

说明栈回溯已经被破坏掉了，看一下栈中的信息:

```
dps esp-0x50:

0:000> dps esp-0x50

001216bc  0022ed4c

001216c0  024957b8

001216c4  00008282

001216c8  001216fc

001216cc  275a273d MSCOMCTL!DllGetClassObject+0xb456

001216d0  001216f4

001216d4  001c8008

001216d8  00008282

001216dc  00000000

001216e0  0022ed4c

001216e4  024957b8

001216e8  6a626f43

001216ec  00000064

001216f0  00008282

001216f4  00000000

001216f8  00000000

001216fc  00000000

00121700  41414141

00121704  00000000

00121708  00000000

0012170c  00000000

00121710  00000000

00121714  00000000
```

根据相关资料，可以知道这个漏洞就是在组件`mscomctl.ocx`中的，所以`275a273d MSCOMCTL!DllGetClassObject+0xb456`保存的可能是返回地址，查看一下这个地址之前的汇编代码：

```
0:000> ub 275a273d

MSCOMCTL!DllGetClassObject+0xb439:

275a2720 0f8539030300    jne     MSCOMCTL!DllGetClassObject+0x3b778 (275d2a5f)

275a2726 837df408        cmp     dword ptr [ebp-0Ch],8

275a272a 0f822f030300    jb      MSCOMCTL!DllGetClassObject+0x3b778 (275d2a5f)

275a2730 ff75f4          push    dword ptr [ebp-0Ch]

275a2733 8d45f8          lea     eax,[ebp-8]

275a2736 53              push    ebx

275a2737 50              push    eax

275a2738 e863fdffff      call    MSCOMCTL!DllGetClassObject+0xb1b9 (275a24a0)
```

可以看到漏洞很可能出现在0x275a2738地址call的函数里边。

反编译一下函数`MSCOMCTL!DllGetClassObject+0xb1b9`：

```
0:000> uf MSCOMCTL!DllGetClassObject+0xb1b9

MSCOMCTL!DllGetClassObject+0xb1b9:

275a24a0 55              push    ebp

275a24a1 8bec            mov     ebp,esp

275a24a3 51              push    ecx

275a24a4 53              push    ebx

275a24a5 8b5d0c          mov     ebx,dword ptr [ebp+0Ch]

275a24a8 56              push    esi

275a24a9 33f6            xor     esi,esi

275a24ab 8b03            mov     eax,dword ptr [ebx]

275a24ad 57              push    edi

275a24ae 56              push    esi

275a24af 8d4dfc          lea     ecx,[ebp-4]

275a24b2 6a04            push    4

275a24b4 51              push    ecx

275a24b5 53              push    ebx

275a24b6 ff500c          call    dword ptr [eax+0Ch]

275a24b9 3bc6            cmp     eax,esi

275a24bb 7c78            jl      MSCOMCTL!DllGetClassObject+0xb24e (275a2535)

MSCOMCTL!DllGetClassObject+0xb1d6:

275a24bd 8b7d10          mov     edi,dword ptr [ebp+10h]

275a24c0 397dfc          cmp     dword ptr [ebp-4],edi

275a24c3 0f8533150300    jne     MSCOMCTL!DllGetClassObject+0x3c715 (275d39fc)

MSCOMCTL!DllGetClassObject+0xb1e2:

275a24c9 57              push    edi

275a24ca 56              push    esi

275a24cb ff3520e06227    push    dword ptr [MSCOMCTL!DllUnregisterServer+0x2dfe8 (2762e020)]

275a24d1 ff1568115827    call    dword ptr [MSCOMCTL+0x1168 (27581168)]

275a24d7 3bc6            cmp     eax,esi

275a24d9 89450c          mov     dword ptr [ebp+0Ch],eax

275a24dc 0f8424150300    je      MSCOMCTL!DllGetClassObject+0x3c71f (275d3a06)

MSCOMCTL!DllGetClassObject+0xb1fb:

275a24e2 8b0b            mov     ecx,dword ptr [ebx]

275a24e4 56              push    esi

275a24e5 57              push    edi

275a24e6 50              push    eax

275a24e7 53              push    ebx

275a24e8 ff510c          call    dword ptr [ecx+0Ch]

275a24eb 8bf0            mov     esi,eax

275a24ed 85f6            test    esi,esi

275a24ef 7c31            jl      MSCOMCTL!DllGetClassObject+0xb23b (275a2522)

MSCOMCTL!DllGetClassObject+0xb20a:

275a24f1 8b750c          mov     esi,dword ptr [ebp+0Ch]

275a24f4 8bcf            mov     ecx,edi

275a24f6 8b7d08          mov     edi,dword ptr [ebp+8]

275a24f9 8bc1            mov     eax,ecx

275a24fb c1e902          shr     ecx,2

275a24fe f3a5            rep movs dword ptr es:[edi],dword ptr [esi]

275a2500 8bc8            mov     ecx,eax

275a2502 8b4510          mov     eax,dword ptr [ebp+10h]

275a2505 83e103          and     ecx,3

275a2508 6a00            push    0

275a250a 8d5003          lea     edx,[eax+3]

275a250d 83e2fc          and     edx,0FFFFFFFCh

275a2510 2bd0            sub     edx,eax

275a2512 f3a4            rep movs byte ptr es:[edi],byte ptr [esi]

275a2514 8b0b            mov     ecx,dword ptr [ebx]

275a2516 52              push    edx

275a2517 68e03f6327      push    offset MSCOMCTL!DllUnregisterServer+0x33fa8 (27633fe0)

275a251c 53              push    ebx

275a251d ff510c          call    dword ptr [ecx+0Ch]

275a2520 8bf0            mov     esi,eax

MSCOMCTL!DllGetClassObject+0xb23b:

275a2522 ff750c          push    dword ptr [ebp+0Ch]

275a2525 6a00            push    0

275a2527 ff3520e06227    push    dword ptr [MSCOMCTL!DllUnregisterServer+0x2dfe8 (2762e020)]

275a252d ff1574115827    call    dword ptr [MSCOMCTL+0x1174 (27581174)]

275a2533 8bc6            mov     eax,esi

MSCOMCTL!DllGetClassObject+0xb24e:

275a2535 5f              pop     edi

275a2536 5e              pop     esi

275a2537 5b              pop     ebx

275a2538 c9              leave

275a2539 c3              ret

MSCOMCTL!DllGetClassObject+0x3c715:

275d39fc b8ffff0080      mov     eax,8000FFFFh

275d3a01 e92febfcff      jmp     MSCOMCTL!DllGetClassObject+0xb24e (275a2535)

MSCOMCTL!DllGetClassObject+0x3c71f:

275d3a06 b80e000780      mov     eax,8007000Eh

275d3a0b e925ebfcff      jmp     MSCOMCTL!DllGetClassObject+0xb24e (275a2535)
```

我们将该函数`MSCOMCTL!DllGetClassObject+0xb1b9`命名为`vul`。

将调用`vul`的函数命名为`vul_parent`：

查看一下调用函数：

```
0:000> ub 275a273d l20

MSCOMCTL!DllGetClassObject+0xb407:

275a26ee 59              pop     ecx

275a26ef 7c02            jl      MSCOMCTL!DllGetClassObject+0xb40c (275a26f3)

275a26f1 33c0            xor     eax,eax

275a26f3 5f              pop     edi

275a26f4 5e              pop     esi

275a26f5 5b              pop     ebx

275a26f6 c9              leave

275a26f7 c20800          ret     8

275a26fa 55              push    ebp

275a26fb 8bec            mov     ebp,esp

275a26fd 83ec14          sub     esp,14h

275a2700 53              push    ebx

275a2701 8b5d0c          mov     ebx,dword ptr [ebp+0Ch]

275a2704 56              push    esi

275a2705 57              push    edi

275a2706 6a0c            push    0Ch

275a2708 8d45ec          lea     eax,[ebp-14h]

275a270b 53              push    ebx

275a270c 50              push    eax

275a270d e88efdffff      call    MSCOMCTL!DllGetClassObject+0xb1b9 (275a24a0)

275a2712 83c40c          add     esp,0Ch

275a2715 85c0            test    eax,eax

275a2717 7c6c            jl      MSCOMCTL!DllGetClassObject+0xb49e (275a2785)

275a2719 817dec436f626a  cmp     dword ptr [ebp-14h],6A626F43h

275a2720 0f8539030300    jne     MSCOMCTL!DllGetClassObject+0x3b778 (275d2a5f)

275a2726 837df408        cmp     dword ptr [ebp-0Ch],8

275a272a 0f822f030300    jb      MSCOMCTL!DllGetClassObject+0x3b778 (275d2a5f)
```
