
# PELock 1.0x -> Bartosz Wojcik 简单脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-640.htm`](https://www.pediy.com/kssd/pediy07/pediy7-640.htm)

PELock 1.0x -> Bartosz Wojcik 简单脱壳

最近碰到 `PELock 1.0x` -> `Bartosz Wojcik` 的壳，以前没见过，索性研究了一下。

准备工作，忽略除内存异常外的其他异常。11 次异常程序运行。

脱这个壳主要是要解决三个问题。

### 1. 输入表

程序的输入表被替换到壳中去了。

```asm
00374661  MOV DWORD PTR DS:[ECX],EBX
00374663  JMP SHORT 00374668
```

`EBX` 中指向壳的地址，`EAX` 为 `api` 的地址。直接改 `EBX` 为 `EAX` 的话后面的 `crc` 校验会出错。所以要想改就要先搞定 `crc`。`jingulong` 大虾给了一段代码，不过本人比较懒，用的是 `fxyang` 的 `script`，速度有点慢，两三分钟才解出整个输入表

```text
=======================================
//获取 iat 表 Script
//by  fxyang  2005.5.20
//由于只想得到 iat 表，所以没有检查表的结束。
var index
#LOG
gpa "LoadLibraryA", "kernel32.dll"
bprm $RESULT,1
mov index,1
eob exp1
run
exp1:
cmp index,0
je exp2
dec index
esto
exp2:
bpmc
rtu
bprm 00374661,1
mov index,1
eob exp3
esto
exp3:
cmp index,0
je exp4
dec index
esto
exp4:
mov [ecx],eax
add eip,2
mov index,1
esto
=========================================
```

### 2. 搞定输入表后，解决壳转移数据的问题

搞定了输入表，接下来就是壳转移数据的问题了。

用 `401000` 内存断点方法 `dump` 下来，修复一下后会出现这个问题

```asm
00402814   $  53            push ebx
00402815   .  85C0         test eax,eax
00402817   .  7E 15        jle short dumped_.0040282E
00402819   .- E9 FFDBE900  jmp 012A041D  //这里出错
0040281E      00          db 00
0040281F   .  8BD8         mov ebx,eax
00402821   .  85DB         test ebx,ebx
```

原来这个壳把程序的一部分参数和函数的调用都弄到了动态申请的地址，`dump` 下来的程序当然没有这个地址了，跟踪一下源程序八次异常后来到处理上面数据的地方

```asm
00378367     3017         xor byte ptr ds:[edi],dl      //特殊处理，edi 放填充地址
00378369     47           inc edi
0037836A     8B16         mov edx,dword ptr ds:[esi]
0037836C     83C6 04      add esi,4
0037836F     C602 E9      mov byte ptr ds:[edx],0E9
00378372     8BC7         mov eax,edi
00378374     2BC2         sub eax,edx
00378376     83E8 05      sub eax,5
00378379     8942 01      mov dword ptr ds:[edx+1],eax
0037837C     8A06         mov al,byte ptr ds:[esi]
0037837E     46           inc esi
0037837F     0FB6C8       movzx ecx,al
00378382     83E0 03      and eax,3
00378385     C1E9 02      shr ecx,2
00378388     F3:A5        rep movs dword ptr es:[edi],dword ptr ds:[esi]
0037838A     8BC8         mov ecx,eax
0037838C     F3:A4        rep movs byte ptr es:[edi],byte ptr ds:[esi]
0037838E     8A06         mov al,byte ptr ds:[esi]
00378390     46           inc esi
00378391     03D0         add edx,eax
00378393     C607 E9      mov byte ptr ds:[edi],0E9
00378396     2BD7         sub edx,edi
00378398     83EA 05      sub edx,5
0037839B     8957 01      mov dword ptr ds:[edi+1],edx
0037839E     83C7 05      add edi,5
003783A1     4B           dec ebx
003783A2  ^ 75 C3        jnz short 00378367         //循环
```

第一次循环的时候在 `xor byte ptr ds:[edi],dl` 中的 `edi` 放的是代码被转移的起始地址，看一下 `edi` 是 `12A000`，原版打算自己添加一个段然后把数据写入这个段里，但是添加失败了，郁闷！不过 `dump` 下来的程序的中有一个起始地址 `5d3000`，大小为 `1c0000` 的段是空的，所以直接把 `edi` 的值改为 `5d3000` 即可。

### 3. 壳转移数据（二）

运行修复了的程序，还是出错

```asm
00480AD4     55           push ebp
00480AD5     68 100C4800  push H_Client.00480C10
00480ADA     64:FF30     push dword ptr fs:[eax]
00480ADD     64:8920     mov dword ptr fs:[eax],esp
00480AE0     A1 E4755C00  mov eax,dword ptr ds:[5C75E4]
00480AE5     8B00        mov eax,dword ptr ds:[eax]
00480AE7     8B10        mov edx,dword ptr ds:[eax]   //指向壳中
00480AE9     FF52 14     call dword ptr ds:[edx+14]
00480AEC     33C0        xor eax,eax
```

跟踪原程序，发现有这么一个很长的表，程序会不断的从这个表中找调用数据

```text
00376C44  0102A50F
00376C48  000000E9
00376C4C  00000000
00376C50  00406E3C  H_Client.00406E3C
00376C54  00406E0C  H_Client.00406E0C
00376C58  00406C30  H_Client.00406C30
00376C5C  00406BDC  H_Client.00406BDC
00376C60  00406F70  H_Client.00406F70
00376C64  00406F40  H_Client.00406F40
00376C68  00408094  H_Client.00408094
00376C6C  00408064  H_Client.00408064
```

这些数据跟壳是在一个段中的，但是单独把这些数据转移会出错，这个段的地址又小于基址，`dump` 下这个段来也不好弄，搞了半天发现这个段的内存是动态申请的，重新来过，下 `he VitrualProtect`

断下后 `ctrl+f9` 返回，`eax` 为 `370000`，看了一下申请的大小要比 `1c0000` 小许多，干脆把 `eax` 改为 `5d3000`，把这个壳整个调到 `5d3000` 段，然后第八个异常的时候再处理一下上面处理的数据

```asm
00378367     3017         xor byte ptr ds:[edi],dl
```

改 `edi` 为 `5e0000`

最后修复下 `stolen code`，纠正 `jmagesize` 后 `dump` 下来，修复输入表，ok

### 总结

一，获取输入表

二，修改 `VirtualProtect` 返回值，修改 `00378367` 处 `edi` 值，使这些数据可以被 `dump` 下来。

三，修复 `stolen code`，`dump` 下来后修复输入表。

by `ak[BCG][DCM][DFCG]`

`4dd466d0e1e0cc123afbe504da2462ca`
