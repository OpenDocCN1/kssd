
```asm
.text:0803DCF9                 push    ebp

.text:0803DCFA                 sub     esp, 104h           <-----------栈长度0x104

.text:0803DD00                 lea     ebp, [esp-4]

.text:0803DD04                 mov     eax, ___security_cookie    <---------Canary

.text:0803DD09                 xor     eax, ebp

.text:0803DD0B                 mov     [ebp+108h+var_4], eax

.text:0803DD11                 push    4Ch

.text:0803DD13                 mov     eax, offset sub_8184A54

.text:0803DD18                 call    __EH_prolog3_catch

.text:0803DD1D                 mov     eax, [ebp+108h+arg_C]

.text:0803DD23                 mov     edi, [ebp+108h+arg_0]

.text:0803DD29                 mov     ebx, [ebp+108h+arg_4]

.text:0803DD2F                 mov     [ebp+108h+var_130], edi

.text:0803DD32                 mov     [ebp+108h+var_138], eax

.text:0803DD35                 call    sub_804172C

.text:0803DD3A                 xor     esi, esi

.text:0803DD3C                 cmp     dword ptr [edi+8], 3

.text:0803DD40                 mov     [ebp+108h+var_10C], esi

.text:0803DD43                 jz      loc_803DF00

.text:0803DD49                 mov     [ebp+108h+var_124], esi

.text:0803DD4C                 mov     [ebp+108h+var_120], esi

.text:0803DD4F                 cmp     dword ptr [edi+0Ch], 1

.text:0803DD53                 mov     byte ptr [ebp+108h+var_10C], 1

.text:0803DD57                 jnz     loc_803DEA9

.text:0803DD5D                 push    offset aName    ; "name"

.text:0803DD62                 push    edi             ; int

.text:0803DD63                 lea     ecx, [ebp+108h+var_124]

.text:0803DD66                 mov     [ebp+108h+var_119], 0

.text:0803DD6A                 call    sub_80217D7

.text:0803DD6F                 cmp     [ebp+108h+var_124], esi

.text:0803DD72                 jnz     short loc_803DDDD

.text:0803DD74                 push    offset aSing    ; "SING"  <------SING表入栈

.text:0803DD79                 push    edi             ; int

.text:0803DD7A                 lea     ecx, [ebp+108h+var_12C

.text:0803DD7D                 call    sub_8021B06               <------处理sing表

.text:0803DD82                 mov     eax, [ebp+108h+var_12C]

.text:0803DD85                 cmp     eax, esi

.text:0803DD87                 mov     byte ptr [ebp+108h+var_10C], 2

.text:0803DD8B                 jz      short loc_803DDC4

.text:0803DD8D                 mov     ecx, [eax]

.text:0803DD8F                 and     ecx, 0FFFFh

.text:0803DD95                 jz      short loc_803DD9F

.text:0803DD97                 cmp     ecx, 100h

.text:0803DD9D                 jnz     short loc_803DDC0

.text:0803DD9F

.text:0803DD9F loc_803DD9F:                            ; CODE XREF: sub_803DCF9

.text:0803DD9F                 add     eax, 10h                  <-------uniqueName域

.text:0803DDA2                 push    eax             ; char *

.text:0803DDA3                 lea     eax, [ebp+108h+var_108]

.text:0803DDA6                 push    eax             ; char *

.text:0803DDA7                 mov     [ebp+108h+var_108], 0

.text:0803DDAB                 call    strcat                    <-------溢出！
```

先来看看处理 `SING` 表的那一块函数。

先把字符串 `SING` 入栈，后入栈 `edi`，调试可得：

![](https://bbs.kanxue.com/upload/attach/201810/800492_ST24HW6N2RKYMJH.png)

暂且还不知道他的内容是怎么回事。继续看。

后来赋值了 `ecx`。调用完函数之后 `eax` 中函数的返回值就是原 `ecx` 的值，来对比一下前后的值。

### 调用前 `ecx`

```text
0012E4B4  74 F3 A8 04 28 6A 1C 02 00 00 00 00 00 00 00 00  t蟥(j........

0012E4C4  CC B9 E7 00 70 E4 12 00 0C E7 12 00 54 4A 18 08  坦?p?..?.TJ

0012E4D4  01 00 00 00 D8 E4 12 00 B0 E6 12 00 00 00 00 00  ...劁.版.....

0012E4E4  E5 64 20 44 58 E8 12 00 9C AE 23 08 EF 52 08 08  錮 DX?.湲#颮

0012E4F4  50 A6 23 08 BC 2D BB 04 0B 0D 08 08 6C F3 A8 04  P???.l蟥

0012E504  D4 E4 12 00 50 E5 12 00 AF F8 11 01 FF FF FF FF  凿.P?.
```

### 调用后 `eax`

```text
0012E4B4  6C 0B A9 04 DF 1D 00 00 00 00 00 00 00 00 00 00  l??..........

0012E4C4  CC B9 E7 00 70 E4 12 00 0C E7 12 00 54 4A 18 08  坦?p?..?.TJ

0012E4D4  01 00 00 00 D8 E4 12 00 B0 E6 12 00 00 00 00 00  ...劁.版.....

0012E4E4  E5 64 20 44 58 E8 12 00 9C AE 23 08 EF 52 08 08  錮 DX?.湲#颮

0012E4F4  50 A6 23 08 BC 2D BB 04 0B 0D 08 08 6C F3 A8 04  P???.l蟥

0012E504  D4 E4 12 00 50 E5 12 00 AF F8 11 01 FF FF FF FF  凿.P?.
```

前八个字节发生了变化。再看后四字节 `0x1DDF`，这不就是前面所看到的数据长度吗？所以我们可以推测出该函数大致上就是处理 `SING` 表内容的。再看看前四字节，调用完函数之后又重新赋值了一遍 `eax`，刚好是把前四字节赋值给了 `eax`。再看看后面对于 `eax` 的调用，又有一处加上了 `0x10`，所以我们看看其地址处的内容：

```text
d 0x4A90B6C :

04A90B6C  00 00 01 00 01 0E 00 01 00 00 00 00 00 00 00 3A  ...........:

04A90B7C  98 66 51 E6 AB 53 8B E7 14 A7 82 4A 0C 0C 0C 0C  榝Q娅S嬬J....

04A90B8C  16 0A 12 37 7A 8C E7 36 3F DC A9 03 A1 E1 CF CB  .7z岀6?堠♂纤

04A90B9C  7B 78 91 C5 8B 8C F7 5F 3D C8 2F 90 40 D3 35 1E  {x懪媽鱛=?怈?

04A90BAC  24 0B 45 5F 64 6D 61 0A 1D 5B 9E 6C 2E F6 6A EB  $E_dma.[瀕.鰆
```

这不就是前面所看到的 `SING` 表内容吗？后面调用 `eax` 加上 `0x10` 的原因就在于加上 `0x10` 后的地址处内容才是 `uniqueName` 域的内容。

然后看 `edi` 的作用。`edi` 寄存器所指的地址内容设置一下内存访问断点，查看一下被什么调用即可。经多次查看之后就可以发现，`edi` 偏移 `0x30` 处的 `0x4A8F374` 被下面图中代码调用过。

![](https://bbs.kanxue.com/upload/attach/201810/800492_5D2RC8UXQ83E35C.png)

再查看一下该处地址的对应内容：

```text
d 0x4A8F374:

04A8F374  00 01 00 00 00 11 01 00 00 04 00 10 4F 53 2F 32  .......OS/2

04A8F384  B4 5F F4 63 00 00 EB 70 00 00 00 56 50 43 4C 54  確鬰..雙...VPCLT

04A8F394  D1 8A 5E 97 00 00 EB C8 00 00 00 36 63 6D 61 70  褗^?.肴...6cmap

04A8F3A4  A4 C3 E8 A0 00 00 B1 6C 00 00 03 58 63 76 74 20  っ锠..眑..Xcvt

04A8F3B4  FF D3 1D 39 00 00 1E FC 00 00 01 FC 66 70 67 6D  ?9..?.黤pgm

04A8F3C4  E7 B4 F1 C4 00 00 26 60 00 00 00 8B 67 61 73 70  绱衲..&`...媑asp

04A8F3D4  00 07 00 07 00 01 01 48 00 00 00 0C 67 6C 79 66  ...H....glyf

04A8F3E4  0C 74 41 CF 00 00 26 EC 00 00 8A 7E 68 64 6D 78  .tA?.&?.妦hdmx

04A8F3F4  34 F0 21 0E 00 00 EC 00 00 00 15 48 68 65 61 64  4?..?..Hhead
```

这是 PDF 文件中的字体对象。

所以说我们这里已经搞清楚了第一个 `call` 函数的用途：

函数通过传入表目录项的 `TAG` 名称，然后读取表目录项，处理后返回两个元素：第一个元素为表目录的内存映射地址，第二个元素为表目录的长度。

搞清了第一个函数之后，再看看下一个漏洞点的函数：

```asm
.text:0803DD9F                 add     eax, 10h

.text:0803DDA2                 push    eax             ; char *

.text:0803DDA3                 lea     eax, [ebp+108h+var_108]

.text:0803DDA6                 push    eax             ; char *

.text:0803DDA7                 mov     [ebp+108h+var_108], 0

.text:0803DDAB                 call    strcat
```

用 `OllyDbg` 调试看看。

![](https://bbs.kanxue.com/upload/attach/201810/800492_U8VXHEFXZDXAKSP.png)

先是加上了 `0x10`，即前文提到的 `uniqueName` 域，压入栈中，而后将 `ebp` 入栈，再调用 `strcat` 函数，所以说就是将 `uniqueName` 域拼接到 `ebp` 指向的地址。

因为并没有限制拼接的长度，所以就产生了溢出。

### ROP

接下来跟踪查看一下作者所构造的 ROP 链，是如何来利用这个溢出漏洞的。反复跟踪调试之后可以得出，先执行到此处：

```asm
.text:0803DDD2                 pop     ecx

.text:0803DDD3

.text:0803DDD3 loc_803DDD3:                            ; CODE XREF: sub_803DCF9+D1j

.text:0803DDD3                 cmp     [ebp+108h+var_119], 0

.text:0803DDD7                 jnz     loc_803DEA9             <----------跳转实现



---------------------------------------------------------------------------



.text:0803DEA9 loc_803DEA9:                            ; CODE XREF: sub_803DCF9+5Ej

.text:0803DEA9                                         ; sub_803DCF9+DEj ...

.text:0803DEA9                 lea     eax, [ebp+108h+var_124]

.text:0803DEAC                 push    eax

.text:0803DEAD                 push    ebx

.text:0803DEAE                 push    edi

.text:0803DEAF                 call    sub_8016BDE
```

进入 `sub_8016BDE` 函数中：
