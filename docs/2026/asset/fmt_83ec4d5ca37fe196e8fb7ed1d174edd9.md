
```
CODE:0049F2EC                add

  esp, 0Ch

CODE:0049F2EF                lea

  eax, [ebp+var\_88]

CODE:0049F2F5                push

  eax

CODE:0049F2F6                push

  esi

CODE:0049F2F7                call

  sub\_49E386    //copy tempBuf到 pDest

CODE:0049F2FC                add

  esp, 8

CODE:0049F2FF

CODE:0049F2FF loc\_49F2FF:

            ; CODE XREF: sub\_49F15D+17Dj

CODE:0049F2FF                shr

  bx, 1        //bx移位

CODE:0049F302                test

  bx, bx

CODE:0049F305                jnz

  short loc\_49F30E

CODE:0049F307                mov

  bx, 8000h

CODE:0049F30B                sub

  edi, 2        //edi指向pBuff的当前字

CODE:0049F30E

CODE:0049F30E loc\_49F30E:

            ; CODE XREF: sub\_49F15D+14Ej

CODE:0049F30E

; sub\_49F15D+157j ...

CODE:0049F30E                mov

  eax, [ebp+var\_4]

CODE:0049F311                add

  [ebp+var\_4], 0FFFFFFFFh    //当前bit位减一

CODE:0049F315                test

  eax, eax            //遍历完pBuff?

CODE:0049F317                jnz

  short loc\_49F2B6        --〉no则跳

CODE:0049F319                push

  0

CODE:0049F31B                lea

  edx, [ebp+var\_88]

CODE:0049F321                push

  edx

CODE:0049F322                call

  sub\_49E3AC

CODE:0049F327                add

  esp, 8

CODE:0049F32A                call

  sub\_49F0DA

CODE:0049F32F                mov

  cx, [ebp+var\_6]

CODE:0049F333                mov

  ds:word\_4DCF68, cx

CODE:0049F33A                xor

  eax, eax

CODE:0049F33C

CODE:0049F33C loc\_49F33C:

            ; CODE XREF: sub\_49F15D+4Ej

CODE:0049F33C

; sub\_49F15D+55j ...

CODE:0049F33C                pop

  edi

CODE:0049F33D                pop

  esi

CODE:0049F33E                pop

  ebx

CODE:0049F33F                mov

  esp, ebp

CODE:0049F341                pop

  ebp

CODE:0049F342                retn

CODE:0049F342 sub\_49F15D      endp
```

这里有两个call，`sub_49E8B3` 和 `sub_49e94f`。

先看`sub_49e8b3`， 首先将`pLongIntZ`的地址存入`4DD76C+0`。`4dd76c+4`开始的空间存放了16个地址，

那些地址指向的空间用来存放`pLongIntZ*2^n` (n=1到16）， 把`pLongIntZ*2^n`（n=0到16）的

最高字节存到`4DD7B0+2*n`, 把`pLongIntZ*2^n`（n=0到16）的次高字节存到`4DD7D2+2*n`, 便于

后面比较大小时使用。

`sub_49e94f`, 是最核心的函数了，好长啊，从`49e94f`到`49f0d9`，好在结构还比较简单，呵呵

来看`sub_49e94f(char \*pTempDest, char \*pLongIntX, char \*pLongIntY)`

```
CODE:0049E94F sub\_49E94F      proc near

      ; CODE XREF: sub\_49F15D+162p

CODE:0049E94F

; sub\_49F15D+18Ap

CODE:0049E94F

CODE:0049E94F var\_6          = word ptr -6

CODE:0049E94F var\_4          = dword ptr -4

CODE:0049E94F arg\_0          = dword ptr  8

CODE:0049E94F arg\_4          = dword ptr  0Ch

CODE:0049E94F arg\_8          = dword ptr  10h

CODE:0049E94F

CODE:0049E94F                push

  ebp

CODE:0049E950                mov

  ebp, esp

CODE:0049E952                add

  esp, 0FFFFFFF8h

CODE:0049E955                push

  ebx

CODE:0049E956                push

  esi

CODE:0049E957                push

  edi

CODE:0049E958                mov

  edi, [ebp+arg\_8]

CODE:0049E95B                mov

  ebx, [ebp+arg\_0]

CODE:0049E95E                push

  [ebp+arg\_4]

CODE:0049E961                push

  offset dword\_4DDF74

CODE:0049E966                call

  sub\_49E872    //该函数首先将pLongIntX的地址存入4ddf74。

          4ddf74+4开始的15个DW空间存放地址，这些地

          址指向的空间的值为 pLongIntX\*2^n,n=1到15

CODE:0049E96B                add

  esp, 8

CODE:0049E96E                movsx

eax, ds:word\_4DCF68

CODE:0049E975                add

  eax, eax

CODE:0049E977                add

  eax, ebx

CODE:0049E979                add

  eax, 0FFFFFFFEh

CODE:0049E97C                mov

  [ebp+var\_4], eax    //ebp+var4保存pTempDest的最高字地址

CODE:0049E97F                mov

  esi, [ebp+var\_4]

CODE:0049E982                sub

  esi, 2        //esi保存pTempDest次高字地址

CODE:0049E985                push

  0

CODE:0049E987                push

  ebx

CODE:0049E988                call

  sub\_49E3AC    //初始化ptempDest指向的空间为0

CODE:0049E98D                add

  esp, 8

CODE:0049E990                push

  edi

CODE:0049E991                call

  sub\_49E3DB    //计算pLongIntY的字长

CODE:0049E996                pop

  ecx

CODE:0049E997                mov

  [ebp+var\_6], ax    //pLongIntY字长存入ebp+bar\_6

CODE:0049E99B                cmp

  [ebp+var\_6], 0

CODE:0049E9A0                jnz

  short loc\_49E9A9    -->nz跳

CODE:0049E9A2                xor

  eax, eax

CODE:0049E9A4                jmp

  loc\_49F0D3

CODE:0049E9A9 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

CODE:0049E9A9

CODE:0049E9A9 loc\_49E9A9:

            ; CODE XREF: sub\_49E94F+51j

CODE:0049E9A9                movsx

edx, [ebp+var\_6]

CODE:0049E9AD                add

  edx, edx

CODE:0049E9AF                add

  edi, edx

CODE:0049E9B1                add

  edi, 0FFFFFFFEh    //edi保存pLongIntY的当前字的地址

CODE:0049E9B4                jmp

  loc\_49F0BF    --〉跳

CODE:0049E9B9 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

CODE:0049E9B9

CODE:0049E9B9 loc\_49E9B9:

            ; CODE XREF: sub\_49E94F+77Cj

CODE:0049E9B9                push

  ebx

CODE:0049E9BA                call

  sub\_49E83C        //将pTempDest移高一个字，也就是

              将pTempDest\*2^16(这步非常重要噢！）

CODE:0049E9BF                pop

  ecx

CODE:0049E9C0                test

  byte ptr [edi+1], 80h    //比较pLongIntY当前字的第15bit,

CODE:0049E9C4                jz

    short loc\_49E9D7

CODE:0049E9C6                push

  0

CODE:0049E9C8                push

  ds:off\_4DDFB0        //ds:off\_4DDFB0存有pLongIntX\*2^15的指针

CODE:0049E9CE                push

  ebx

CODE:0049E9CF                call

  sub\_49E199        //两超长正整数相加，存回pTempDest，

              //字节间的进位处理还晃了本菜一下，呵

CODE:0049E9D4                add

  esp, 0Ch

CODE:0049E9D7

CODE:0049E9D7 loc\_49E9D7:

            ; CODE XREF: sub\_49E94F+75j

......

CODE:0049EB12

CODE:0049EB12 loc\_49EB12:

            ; CODE XREF: sub\_49E94F+1B0j

CODE:0049EB12                test

  byte ptr [edi], 1

CODE:0049EB15                jz

    short loc\_49EB28

CODE:0049EB17                push

  0

CODE:0049EB19                push

  ds:dword\_4DDF74

CODE:0049EB1F                push

  ebx

CODE:0049EB20                call

  sub\_49E199

CODE:0049EB25                add

  esp, 0Ch

CODE:0049EB28

//以上对edi+1，和edi的各个非零比特，将pTempDest加上相应的pLongIntX\*2^n

CODE:0049EB28 loc\_49EB28:

            ; CODE XREF: sub\_49E94F+1C6j

CODE:0049EB28                mov

  ecx, [ebp+var\_4]

CODE:0049EB2B                mov

  ax, [ecx]            //ax=pTempDest的最高字的值

CODE:0049EB2E                sub

  ax, ds:word\_4DD7D0    //4dd7d0保存了 pLongIntZ\*2^16

              的最高字的值

CODE:0049EB35                test

  ax, ax

CODE:0049EB38                jg

    short loc\_49EB6B        //pTempDest大就跳到49eb6b

CODE:0049EB3A                test

  ax, ax

CODE:0049EB3D                jnz

  short loc\_49EB7C

CODE:0049EB3F                mov

  dx, [esi]            //dx=pTempDest的次高字的值

CODE:0049EB42                cmp

  dx, ds:word\_4DD7F2    //4dd7d0保存了 pLongIntZ\*2^16

              的次高字的值

CODE:0049EB49                ja

    short loc\_49EB6B        //pTempDest大就跳到49eb6b

CODE:0049EB4B                mov

  cx, [esi]

CODE:0049EB4E                cmp

  cx, ds:word\_4DD7F2

CODE:0049EB55                jnz

  short loc\_49EB7C        //小于就跳过49eb6b

CODE:0049EB57                push

  ds:off\_4DD7AC

CODE:0049EB5D                push

  ebx

CODE:0049EB5E                call

  sub\_49E2B7        //比较pTempDest与pLongIntZ\*2^16大小

CODE:0049EB63                add

  esp, 8

CODE:0049EB66                test

  ax, ax

CODE:0049EB69                jl

    short loc\_49EB7C        //小于就跳走

CODE:0049EB6B

CODE:0049EB6B loc\_49EB6B:

            ; CODE XREF: sub\_49E94F+1E9j

CODE:0049EB6B

; sub\_49E94F+1FAj

CODE:0049EB6B                push

  0

CODE:0049EB6D                push

  ds:off\_4DD7AC

CODE:0049EB73                push

  ebx

CODE:0049EB74                call

  sub\_49E1E6        //将pTempDest指向的值减去pLongIntZ\*2^16

CODE:0049EB79                add
```
