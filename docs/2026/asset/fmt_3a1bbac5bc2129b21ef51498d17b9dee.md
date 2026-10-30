
```
00411BE9  |.  8B7D D8       mov edi,[local.10]  ;输出

00411BEC  |.  33C0          xor eax,eax

00411BEE  |.  40            inc eax

00411BEF  |.  897D EC       mov [local.5],edi

00411BF2  |.  2BC7          sub eax,edi

00411BF4  |.  C745 E4 05000>mov [local.7],0x5

00411BFB  |.  8945 CC       mov [local.13],eax

00411BFE  |>  FF75 E4       /push [local.7]

00411C01  |.  83FB 10       |cmp ebx,0x10

00411C04  |.  8D45 08       |lea eax,[arg.1]

00411C07  |.  0F4345 08     |cmovnb eax,[arg.1]  ;异或后的输入

00411C0B  |.  03C1          |add eax,ecx        

00411C0D  |.  50            |push eax

00411C0E  |.  FF76 04       |push dword ptr ds:[esi+0x4]  ;表尾

00411C11  |.  FF36          |push dword ptr ds:[esi]      ;表头

00411C13  |.  E8 34F3FFFF   |call 7-不问少.00410F4C       ;addr xor_input[i] in table

00411C18  |.  8BC8          |mov ecx,eax

00411C1A  |.  83C4 10       |add esp,0x10

00411C1D  |.  3B4E 04       |cmp ecx,dword ptr ds:[esi+0x4] ;如果没找到，NEXT

00411C20  |.  74 60         |je short 7-不问少.00411C82

00411C22  |.  8AC1          |mov al,cl

00411C24  |.  2A06          |sub al,byte ptr ds:[esi]   ;index

00411C26  |.  8807          |mov byte ptr ds:[edi],al

00411C28  |.  75 05         |jnz short 7-不问少.00411C2F

00411C2A  |.  33DB          |xor ebx,ebx

00411C2C  |.  43            |inc ebx

00411C2D  |.  EB 03         |jmp short 7-不问少.00411C32

00411C2F  |>  0FBED8        |movsx ebx,al

00411C32  |>  837D E8 00    |cmp [local.6],0x0

00411C36  |.  881F          |mov byte ptr ds:[edi],bl

00411C38  |.  7C 3E         |jl short 7-不问少.00411C78

00411C3A  |.  8B45 CC       |mov eax,[local.13]

00411C3D  |.  03C7          |add eax,edi

00411C3F  |.  8945 D4       |mov [local.11],eax

00411C42  |>  33FF          |/xor edi,edi

00411C44  |>  41            ||/inc ecx

00411C45  |.  3B4E 04       |||cmp ecx,dword ptr ds:[esi+0x4]

00411C48  |.  75 02         |||jnz short 7-不问少.00411C4C   ;检查是否到表尾

00411C4A  |.  8B0E          |||mov ecx,dword ptr ds:[esi]

00411C4C  |>  0FBEC3        |||movsx eax,bl

00411C4F  |.  47            |||inc edi

00411C50  |.  99            |||cdq

00411C51  |.  F77D E4       |||idiv [local.7]

00411C54  |.  6A 05         |||push 0x5

00411C56  |.  5A            |||pop edx

00411C57  |.  03C2          |||add eax,edx        

00411C59  |.  3BF8          |||cmp edi,eax         ;index=index+index/5+5

00411C5B  |.^ 75 E7         ||\jnz short 7-不问少.00411C44  

00411C5D  |.  8B7D EC       ||mov edi,[local.5]

00411C60  |.  8AC1          ||mov al,cl

00411C62  |.  2A06          ||sub al,byte ptr ds:[esi]

00411C64  |.  8807          ||mov byte ptr ds:[edi],al

00411C66  |.  75 05         ||jnz short 7-不问少.00411C6D

00411C68  |.  33DB          ||xor ebx,ebx

00411C6A  |.  43            ||inc ebx

00411C6B  |.  EB 03         ||jmp short 7-不问少.00411C70

00411C6D  |>  0FBED8        ||movsx ebx,al

00411C70  |>  836D D4 01    ||sub [local.11],0x1

00411C74  |.  881F          ||mov byte ptr ds:[edi],bl

00411C76  |.^ 75 CA         |\jnz short 7-不问少.00411C42  ;循环输入字节的序号次

00411C78  |>  0FBE0F        |movsx ecx,byte ptr ds:[edi]

00411C7B  |.  8B06          |mov eax,dword ptr ds:[esi]

00411C7D  |.  8A0401        |mov al,byte ptr ds:[ecx+eax]

00411C80  |.  8807          |mov byte ptr ds:[edi],al     ;置换出的字节

00411C82  |>  8B4D E8       |mov ecx,[local.6]

00411C85  |.  41            |inc ecx

00411C86  |.  47            |inc edi

00411C87  |.  894D E8       |mov [local.6],ecx

00411C8A  |.  897D EC       |mov [local.5],edi

00411C8D  |.  3B4D D0       |cmp ecx,[local.12]

00411C90  |.  7D 08         |jge short 7-不问少.00411C9A

00411C92  |.  8B5D 1C       |mov ebx,[arg.6]

00411C95  |.^ E9 64FFFFFF   \jmp 7-不问少.00411BFE   下一个输入字节
```

转换表为：

```
0x89,0xBC,0x95,0xFC,0xFB,0xBA,0xED,0x9A,0xBB,0xAE,0xFE,0x99,0xA2,0x98,0xB9,0xF9

0x9F,0x84,0x9C,0xFD,0x83,0xAD,0xB6,0xA9,0xA5,0xF5,0x8C,0xA7,0x9E,0x96,0x8A,0xF4

0x85,0xBE,0xA8,0x8F,0x86,0xAF,0x88,0x9D,0x87,0xBF,0xFF,0xA1,0x8B,0x81,0xA0,0xAB

0x8E,0xBD,0xB5,0xAA,0x82,0x94,0xA4,0x8D,0xA3,0xF8,0xB4,0xFA,0x9B,0xA6,0xB8,0x80
```

我顺便看了下置换表对应的输入字符，字符范围为数字+大小字母+！+@，共64个。

`sub_411B30`中还调用了`sub_410F75`，对转换后的结果再进行变换，变换公式为`(a^0xcc<<3)|(a^0xcc>>5)`

```
00410F90  |> /8A0A          /mov cl,byte ptr ds:[edx]

00410F92  |. |80F1 CC       |xor cl,0xCC

00410F95  |. |8AC1          |mov al,cl

00410F97  |. |C0E1 03       |shl cl,0x3

00410F9A  |. |C0F8 05       |sar al,0x5

00410F9D  |. |0AC1          |or al,cl

00410F9F  |. |8802          |mov byte ptr ds:[edx],al

00410FA1  |. |42            |inc edx

00410FA2  |. |47            |inc edi

00410FA3  |. |3BFE          |cmp edi,esi

00410FA5  |.^\75 E9         \jnz short 7-不问少.00410F90
```

跟到这已经烦躁不行，有很多的堆函数什么的。所以就想直接看看整体过程。

然后直接进了`sub_411825`，静态的时候已经发现此处有`VirtualAlloc`，又翻看了下，没有明显弹窗的地方，此函数结束后就直接进入事件流程了。所以推测，弹窗在`VirtualAlloc`的空间的代码中。只跟了下流程，大概是：如果`sub_411975`函数返回真，则进行`VirtualAlloc`，拷贝`411A9C`处的0x94个字节数据，并循环与原始输入异或，最后执行代码。 只里有个细节，异或完成后，在复制过来的数据偏移0x61和0x8C处，分别加上了两个地址之间的dword偏移量，然后在数据后又加上了最后一次参加异或的输入后一位，后面就直接执行复制过来的数据了。据此猜测这两处是两个call，改的是操作数，也就是偏移，程序代码如下：

```
.text:004118A9 loc_4118A9:                             ; CODE XREF: sub_411825+5B

.text:004118A9                 mov     eax, [ecx+34h]

.text:004118AC                 mov     [ebp-1Ch], eax

.text:004118AF                 add     ecx, 24h

.text:004118B2                 cmp     dword ptr [ecx+14h], 10h

.text:004118B6                 jb      short loc_4118BA

.text:004118B8                 mov     ecx, [ecx]

.text:004118BA

.text:004118BA loc_4118BA:                             ; CODE XREF: sub_411825+91

.text:004118BA                 mov     eax, esi

.text:004118BC                 xor     edx, edx

.text:004118BE                 div     dword ptr [ebp-1Ch]

.text:004118C1                 mov     al, [edx+ecx]

.text:004118C4                 mov     ecx, [ebp-14h]

.text:004118C7                 xor     [ecx], al

.text:004118C9                 lea     ecx, [edi+60h]     ;virtual_addr+0x60

.text:004118CC                 mov     edx, offset sub_411A9C

.text:004118D1                 mov     eax, edx

.text:004118D3                 sub     eax, ecx

.text:004118D5                 add     eax, 60h

.text:004118D8                 add     [ecx+1], eax   ;eax=411A9C-virtual_addr

.text:004118DB                 lea     ecx, [edi+8Bh] ;virtual_addr+0x8b

.text:004118E1                 mov     eax, [ecx+1]

.text:004118E4                 sub     eax, ecx

.text:004118E6                 add     edx, 8Bh

.text:004118EC                 add     eax, edx

.text:004118EE                 mov     [ecx+1], eax

.text:004118F1                 call    dword ptr [ebp-18h]
```

看到了这，我想偷个懒，尝试能不能直接通过代码的特点和规律，直接还原代码。先从入口，返回和那两个偏移入手。 原始的代码数据如下：

```
0x00  17 FC 82 19 BE 1C B8 60 21 69 1D 93 30 31 37 4B

0x10  08 BA A3 4F E3 F3 6A 33 41 47 95 EC 21 99 29 BF

0x20  75 C5 53 E8 58 39 4F 6B C1 12 B6 73 CC 31 88 35

0x30  E9 24 A5 F5 75 ED 75 1C 16 6A 1E E6 07 9A A9 36

0x40  A1 61 62 47 5B 39 F4 77 D0 F6 72 AF 3A 0A 6E 56

0x50  12 FA 2B A3 86 31 B8 42 12 1D 03 62 F6 74 DB 09

0x60  B0 2B B9 94 BD F4 AA 67 CC 31 B0 0F 24 01 64 0F

0x70  70 31 67 21 58 C6 5A 9B 7F 32 6E F8 0C 80 34 EC

0x80  69 69 79 32 30 68 BC 06 A8 0A 82 83 6D 53 6E 73

0x90  CA 91 0D A6 00 00 00 00 00 00 00 00 00 00 00 00
```

首先，入口通常为`push ebp mov ebp,esp`，对应机器码为`55 8B EC`，得到`Bwn` 再看返回，数据最后一位添加的不可能是`C3`，不在可输入范围，所以`A6`异或后就为`C3`或`C2`，所以此处的异或字符为`e`或`d` 再看两个call，`B0 2B B9 94 BD`和`6D 53 6E 73`，`call`的对应机器码是`E8`，所以`B0`对应的异或字符为`X`，`6D`对应`k`，通常偏移是`xx xx 00 00`或`xx xx FF FF`，所以`94 BD`对应`Kb`，`6E 73`对应`ns`。 似乎没法进行下去了。尝试多次也不见效。继续跟踪。这次跟踪重点是进`VirtualAlloc`的条件，即`sub_411975`函数。

反复跟踪，终于明白，转换后的输入先转成16进制，再将此转换成4进制，最后与一串常量比较，校验的函数为`sub_41612A`，主要校验是经过两轮，每轮比较0x20个四进制数。 常量串为：

```
 02 02 00 03 00 00 02 03 00 01 02 03 02 00 00 02

 02 00 02 02 02 03 02 03 01 00 02 02 02 01 02 03

 02 02 02 01 02 03 02 03 02 01 00 03 02 02 02 02

 02 02 00 03 01 02 02 03 02 00 00 02 02 02 02 02

 00 00 00 02 01 00 02 02 02 03 00 02 01 00 00 03
```

那反算回来不就是key了。动手操作，发现我想到了，最后结果多处有多值。反变换脚本如下：
