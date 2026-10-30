
进入 `MSCOMCTL!DllGetClassObject` 调试流程。

```
> 275a24c3 0f8533150300    jne     MSCOMCTL!DllGetClassObject+0x3c715 (275d39fc) [br=0]

> 0:000> p

> eax=00000000 ebx=023a57b8 ecx=0012181c edx=00000000 esi=00000000 edi=00008282

> eip=275a24c9 esp=001216b8 ebp=001216c8 iopl=0         nv up ei pl zr na pe nc

> cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246

> MSCOMCTL!DllGetClassObject+0xb1e2:

> 275a24c9 57              push    edi

> 0:000> p

> eax=00000000 ebx=023a57b8 ecx=0012181c edx=00000000 esi=00000000 edi=00008282

> eip=275a24ca esp=001216b4 ebp=001216c8 iopl=0         nv up ei pl zr na pe nc

> cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246

> MSCOMCTL!DllGetClassObject+0xb1e3:

> 275a24ca 56              push    esi

> 0:000> p

> eax=00000000 ebx=023a57b8 ecx=0012181c edx=00000000 esi=00000000 edi=00008282

> eip=275a24cb esp=001216b0 ebp=001216c8 iopl=0         nv up ei pl zr na pe nc

> cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246

> MSCOMCTL!DllGetClassObject+0xb1e4:

> 275a24cb ff3520e06227    push    dword ptr [MSCOMCTL!DllUnregisterServer+0x2dfe8 (2762e020)] ds:0023:2762e020=00140000

> 0:000> p

> eax=00000000 ebx=023a57b8 ecx=0012181c edx=00000000 esi=00000000 edi=00008282

> eip=275a24d1 esp=001216ac ebp=001216c8 iopl=0         nv up ei pl zr na pe nc

> cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246

> MSCOMCTL!DllGetClassObject+0xb1ea:

> 275a24d1 ff1568115827    call    dword ptr [MSCOMCTL+0x1168 (27581168)] ds:0023:27581168={ntdll!RtlAllocateHeap (7c9300c4)}
```

`0:000> dd esp`

```
001216ac  00140000 00000000 00008282 00000000
001216bc  0679cd5c 023a57b8 00008282 001216fc
001216cc  275a273d 001216f4 023a57b8 00008282
001216dc  00000000 0679cd5c 023a57b8 6a626f43
001216ec  00000064 00008282 00121744 275876c7
001216fc  00121724 275e727b 0679cd5c 023a57b8
0012170c  00000000 0679cd38 036520d8 275a688f
0012171c  00000001 00121744 00121744 275e75c2
```

此时调用的是在堆上分配内存的函数 `RtlAllocateHeap`，函数定义：

```
PVOID RtlAllocateHeap( _In_     PVOID  HeapHandle,_In_opt_ ULONG  Flags， _In_     SIZE_T Size);
```

可以看到在堆上分配大小为 `0x8282` 的内存，此处的 `0x8282` 即来自 `edi` 也就是 `vul` 函数的第三个参数。

继续单步：

```
0:000> p

eax=03646008 ebx=023a57b8 ecx=7c9301db edx=00140608 esi=00000000 edi=00008282

eip=275a24d7 esp=001216b8 ebp=001216c8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

MSCOMCTL!DllGetClassObject+0xb1f0:

275a24d7 3bc6            cmp     eax,esi
```

`0:000> r eax`

`eax=03646008`

`0:000> dd eax`

```
03646008  baadf00d baadf00d baadf00d baadf00d
03646018  baadf00d baadf00d baadf00d baadf00d
03646028  baadf00d baadf00d baadf00d baadf00d
03646038  baadf00d baadf00d baadf00d baadf00d
03646048  baadf00d baadf00d baadf00d baadf00d
03646058  baadf00d baadf00d baadf00d baadf00d
03646068  baadf00d baadf00d baadf00d baadf00d
03646078  baadf00d baadf00d baadf00d baadf00d
```

此处 `eax` 就是分配内存的首地址

继续单步走：

```
0:000> p

eax=03646008 ebx=023a57b8 ecx=7c9301db edx=00140608 esi=00000000 edi=00008282

eip=275a24d9 esp=001216b8 ebp=001216c8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

MSCOMCTL!DllGetClassObject+0xb1f2:

275a24d9 89450c          mov     dword ptr [ebp+0Ch],eax ss:0023:001216d4=023a57b8
```

`0:000> p`

```
eax=03646008 ebx=023a57b8 ecx=7c9301db edx=00140608 esi=00000000 edi=00008282

eip=275a24dc esp=001216b8 ebp=001216c8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

MSCOMCTL!DllGetClassObject+0xb1f5:

275a24dc 0f8424150300    je      MSCOMCTL!DllGetClassObject+0x3c71f (275d3a06) [br=0]
```

`0:000> p`

```
eax=03646008 ebx=023a57b8 ecx=7c9301db edx=00140608 esi=00000000 edi=00008282

eip=275a24e2 esp=001216b8 ebp=001216c8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

MSCOMCTL!DllGetClassObject+0xb1fb:

275a24e2 8b0b            mov     ecx,dword ptr [ebx]  ds:0023:023a57b8={ole32!CExposedStream::`vftable' (7699dd78)}
```

`0:000> p`

```
eax=03646008 ebx=023a57b8 ecx=7699dd78 edx=00140608 esi=00000000 edi=00008282

eip=275a24e4 esp=001216b8 ebp=001216c8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

MSCOMCTL!DllGetClassObject+0xb1fd:

275a24e4 56              push    esi
```

`0:000> p`

```
eax=03646008 ebx=023a57b8 ecx=7699dd78 edx=00140608 esi=00000000 edi=00008282

eip=275a24e5 esp=001216b4 ebp=001216c8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

MSCOMCTL!DllGetClassObject+0xb1fe:

275a24e5 57              push    edi
```

`0:000> p`

```
eax=03646008 ebx=023a57b8 ecx=7699dd78 edx=00140608 esi=00000000 edi=00008282

eip=275a24e6 esp=001216b0 ebp=001216c8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

MSCOMCTL!DllGetClassObject+0xb1ff:

275a24e6 50              push    eax
```

`0:000> p`

```
eax=03646008 ebx=023a57b8 ecx=7699dd78 edx=00140608 esi=00000000 edi=00008282

eip=275a24e7 esp=001216ac ebp=001216c8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

MSCOMCTL!DllGetClassObject+0xb200:

275a24e7 53              push    ebx
```

`0:000> p`

```
eax=03646008 ebx=023a57b8 ecx=7699dd78 edx=00140608 esi=00000000 edi=00008282

eip=275a24e8 esp=001216a8 ebp=001216c8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

MSCOMCTL!DllGetClassObject+0xb201:

275a24e8 ff510c          call    dword ptr [ecx+0Ch]  ds:0023:7699dd84=30ed7be6
```

`0:000> ln 0ed7be6`

`0:000> ln 30ed7be6`

`(30ed7b91)   mso!Ordinal1077+0x55   |  (30ed7d09)   mso!Ordinal6286`

`0:000> dd esp`

```
001216a8  023a57b8 03646008 00008282 00000000
001216b8  00000000 0679cd5c 023a57b8 00008282
001216c8  001216fc 275a273d 001216f4 03646008
001216d8  00008282 00000000 0679cd5c 023a57b8
001216e8  6a626f43 00000064 00008282 00121744
001216f8  275876c7 00121724 275e727b 0679cd5c
00121708  023a57b8 00000000 0679cd38 036520d8
00121718  275a688f 00000001 00121744 00121744
```

分析一下这个函数使 `mso` 组件中的，第一个参数是对象地址，第二个参数是刚才在堆上分配的内存首地址，第三个参数分配内存的大小，第四个参数是 `0x0`，可以看到这个函数可能就是往刚才分配的内存上写数据，数据来源是从 `poc.doc` 上读取的数据，注意此时 `edi=0x8282`, `poi(ebp+8)=001216f4` `poi(ebp+c)=03646008`, 此处的 `poi(ebp+c)` 的值是 `0x275a24d9 mov     dword ptr [ebp+0Ch],eax` 传入的，即堆上分配的内存。

单步走一下：

```
0:000> p

eax=00000000 ebx=023a57b8 ecx=0012181c edx=00000000 esi=00000000 edi=00008282

eip=275a24eb esp=001216b8 ebp=001216c8 iopl=0         nv up ei pl zr na pe nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246

MSCOMCTL!DllGetClassObject+0xb204:

275a24eb 8bf0            mov     esi,eax
```

`0:000> dd 03646008`

```
03646008  00000000 00000000 00000000 41414141
03646018  00000000 00000000 00000000 00000000
03646028  00000000 00000000 00000000 00000000
03646038  00000000 00000000 00000000 00000000
```

函数执行成功返回 `0`，堆分配内存中新写入了数据，这些数据来自 `poc.doc` 文件

经过上边分析，我们总结一下，在函数 `vul` 中，`275a24fe rep movs dword ptr es:[edi],dword ptr [esi]`，此处 `ecx` 来自 `->275a24fb shr     ecx,2 ->0x275a24f4  mov     ecx,edi->0x275a24bd mov     edi,dword ptr [ebp+10h]` `->vul` 的第三个参数，说明 `vul` 函数的第三个参数除以四就是 `ecx`

`esi` 来自 `->275a24f1 mov     esi,dword ptr [ebp+0Ch]->275a24d9 mov     dword ptr [ebp+0Ch],eax ->275a24d1 call    dword ptr [MSCOMCTL+0x1168 (27581168)]` 说明 `esi` 来自堆分配的一块大小为 `0x8282` 的内存，内存被函数 `275a24e8 call    dword ptr [ecx+0Ch]` 写入 `poc.doc` 中的数据

`edi` 来自 `275a24f6 mov     edi,dword ptr [ebp+8]` 即函数 `vul` 的第一个参数

上边已经分析完了函数 `vul`，我们继续分析 `vul_parent` 函数，从 `0x275a2738` 往上分析，即内存低址：

```
275a270d e88efdffff      call    MSCOMCTL!DllGetClassObject+0xb1b9 (275a24a0)

275a2712 83c40c          add     esp,0Ch
```
