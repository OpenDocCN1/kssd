
  1004E623   .  8B55 DC       mov     edx, dword ptr [ebp-24] 还是0x015F1218，那个IAT相关的指针

  1004E626   .  8B49 02       mov     ecx, dword ptr [ecx+2]

  1004E629   .  3B8C02 AA0400>cmp     ecx, dword ptr [edx+eax+4AA] // 判断1

  1004E630   .  75 73         jnz     short 1004E6A5

  1004E632   .  8B45 FC       mov     eax, dword ptr [ebp-4]

  1004E635   .  0FB600        movzx   eax, byte ptr [eax]

  1004E638   .  3D FF000000   cmp     eax, 0FF                     // 判断2

  1004E63D   .  75 66         jnz     short 1004E6A5

  1004E63F   .  8B45 FC       mov     eax, dword ptr [ebp-4]

  1004E642   .  0FB640 01     movzx   eax, byte ptr [eax+1]

  1004E646   .  83F8 15       cmp     eax, 15                      // 判断3，这3处就是判断是不是call [00528XXX]的形式

  1004E649   .  75 5A         jnz     short 1004E6A5               // 不是就跳走，使用默认函数编号，否则就重新计算编号

  1004E64B   .  8B45 E4       mov     eax, dword ptr [ebp-1C]

看过这段代码之后，大家有在想什么呢？

我想到了几点，说出来，大家看看有没有落下什么。

<1>`call 1004ED1E` 一个校验函数，很重要，但是不难，简单模拟下，嵌入汇编或者`LoadLibrary`都可以。

<2>调用的时候要判断是不是`call [00528XXX]`的形式，这一点用处很多。

例如：

```
mov edi, dword ptr [00528010]
call edi
```

我们不用劳神于类似这样的变形，对于那些经SD处理之后生成的代码也不担心，它们肯定都是调用默认编号指定的函数。

<3>注意判断1处，`[IAT相关的指针 + 默认函数编号 \* 0x4C3 + 0x4AA]`就是默认编号对应的IAT表项地址`00528090`，换句话说就是我们可以推算出所有编号与IAT表项地址的对应关系，再确定编号与函数地址的关系就可以得到完美的IAT表了。

从`$1004E64B~$1004E6A5`之间就是重新计算编号的代码，只是简单的计算，这里就略过了，下面着重分析用模块编号和函数编号计算目标函数地址的代码。

在计算函数名和函数名长度时，SD使用内部函数编号，用目标函数编号求内部函数编号参考后面的公式。

```
1004E1D5  |.  8B45 08       mov     eax, dword ptr [ebp+8] // 取出模块编号，暂时用□标记

1004E1D8  |.  69C0 8D000000 imul    eax, eax, 8D

1004E1DE  |.  8B0D B8CD0610 mov     ecx, dword ptr [1006CDB8] // 这是一个固定指针，暂时用△标记

1004E1E4  |.  8B55 0C       mov     edx, dword ptr [ebp+C] // 取出内部函数编号

1004E1E7  |.  3B5401 58     cmp     edx, dword ptr [ecx+eax+58] // [△+□\*0x8D+0x58]=模块内函数个数，判断是否超出函数总数

1004E1EB  |.  0F83 E1000000 jnb     1004E2D2

1004E1F1  |.  8B45 08       mov     eax, dword ptr [ebp+8]

1004E1F4  |.  69C0 8D000000 imul    eax, eax, 8D

1004E1FA  |.  8B0D B8CD0610 mov     ecx, dword ptr [1006CDB8]

1004E200  |.  8B8401 B50000>mov     eax, dword ptr [ecx+eax+B5] // [△+□\*0x8D+0xB5]=函数名字符串（加密的）地址列表首地址

1004E207  |.  8B4D 0C       mov     ecx, dword ptr [ebp+C]

1004E20A  |.  8B0488        mov     eax, dword ptr [eax+ecx\*4] // 取出函数名字符串地址

1004E20D  |.  8945 F4       mov     dword ptr [ebp-C], eax

。。。。。。

1004E244  |> \8B45 08       mov     eax, dword ptr [ebp+8]

1004E247  |.  69C0 8D000000 imul    eax, eax, 8D

1004E24D  |.  8B0D B8CD0610 mov     ecx, dword ptr [1006CDB8]

1004E253  |.  8B8401 BF0000>mov     eax, dword ptr [ecx+eax+BF] // [△+□\*0x8D+0xBF]=函数名长度地址列表首地址

1004E25A  |.  8B4D 0C       mov     ecx, dword ptr [ebp+C]

1004E25D  |.  8B0488        mov     eax, dword ptr [eax+ecx\*4]

1004E260  |.  8B00          mov     eax, dword ptr [eax] // 取出函数名长度

1004E262  |.  8945 E8       mov     dword ptr [ebp-18], eax

1004E265  |.  A1 B8CD0610   mov     eax, dword ptr [1006CDB8]

1004E26A  |.  83C0 26       add     eax, 26 // △+0x26指向解密用的数据

1004E26D  |.  50            push    eax // 解密数据指针 unsigned char \* arg1

1004E26E  |.  FF75 E8       push    dword ptr [ebp-18] // 函数名长度 int func\_name\_length

1004E271  |.  FF75 F4       push    dword ptr [ebp-C] // 函数名（加密的）指针 char \*func\_name

1004E274  |.  E8 437CFDFF   call    10025EBC // 解密用的函数，有点难，我翻译不出C代码，直接LoadLibrary调用

1004E279  |.  83C4 0C       add     esp, 0C
```

至此我们已经看到了函数调用的曙光，`$1004E274 call 10025EBC`返回之后，函数名就摆在我们面前了。

这里总结一下函数调用中常用的几个地址。

```
设 △ = [1006CDB8];

     □ = 模块编号;

      N = 目标函数编号;（不一定是默认函数编号！）

     ○ = 内部函数编号;

  那么

     [△ + 0xF] = 模块总个数

     [△ + 0x8D \* □ + 0x58] = □模块内函数的总个数

     [△ + 0x8D \* □ + 0xB5] = □模块中函数名字符串指针列表首地址

     [△ + 0x8D \* □ + 0xBF] = □模块中函数名长度指针列表首地址

     [△ + 0x8D \* □ + 0x52] = □模块的句柄，就是GetProcAddress的第一个参数

     [△ + 0x8D \* □ + 0xC3] = □模块中那个与IAT相关的地址，暂用☆标记

     [☆ + 0x4C3 \* N + 0x4AA] = □模块中N号函数对应的IAT表项地址，N是未经过变换的。

     [△ + 0x8D \* □ + 0x4C] = □模块中用于把目标编号转换成内部编号的表的首地址，暂用★标记

     ○ = [★ + 4 \* N] // 查表求得内部标号，计算函数名地址和函数名长度时使用内部编号○
```

最后编写2个程序，一个获得IAT表，一个穷举代码段中的`call [00528XXX]`并输出正确的地址。

注：《Unpack SafeDisc v2.90.40 (更新代码修复)》一文中通过修改代码强制写入原始IAT的方法与我写程序得到的IAT相同，那个方法不用自己写程序，推荐熟悉一下。

还有一部分函数调用伪装成了 `jmp XXXXXXXX` 的形式，这里提供一个快速修复的方法。先来看代码：

```
地址     大小     属主  区段

006E9000 00003000 AsfPc stxt774

006EC000 00004000 AsfPc stxt371

004012C9   . /74 25         je      short 004012F0

004012CB   . |6A 64         push    64

004012CD   .-|E9 FA862E00   jmp     006E99CC      // 跳到壳区段中去了，伪装的函数调用，跟进去

004012D2   . |8647 83       xchg    byte ptr [edi-7D], al

004012D5   . |FF08          dec     dword ptr [eax]

004012D7   .^|72 AE         jb      short 00401287

006E99CC    53              push    ebx

006E99CD    EB 06           jmp     short 006E99D5

006E99D5    E8 CCFEFFFF     call    006E98A6

006E99DA    B2 43           mov     dl, 43 // 记下这个地址1

006E98A6    870424          xchg    dword ptr [esp], eax

006E98A9    9C              pushfd

006E98AA    05 320F0000     add     eax, 0F32 // eax=地址1+0x0F32

006E98AF    8B18            mov     ebx, dword ptr [eax]

006E98B1    6BDB 13         imul    ebx, ebx, 13

006E98B4  ^ EB C7           jmp     short 006E987D

  看下数据

006EA90C|C3 04 00 00|BF 04 62 01|.....

006E987D    0358 04         add     ebx, dword ptr [eax+4] // ebx=0x4C3 \* 0x13 + 0x016204BF=0x01625F38

006E9880    9D              popfd

006E9881    58              pop     eax

006E9882    871C24          xchg    dword ptr [esp], ebx // 上面那个地址主要是用来计算调用的地址

006E9885    C3              retn

01625F38    68 D3124000     push    4012D3

01625F3D    68 3F12EABF     push    BFEA123F

01625F42    9C              pushfd

01625F43    60              pushad

01625F44    54              push    esp

01625F45    68 785F6201     push    1625F78 // 注意这个参数，修复关键全在这里

01625F4A    E8 4684A20E     call    ~df394b.1004E395 // 原来还是这个函数

01625F4F    83C4 08         add     esp, 8

01625F52    6A 00           push    0

01625F54    58              pop     eax

01625F55    61              popad

01625F56    9D              popfd

01625F57    C3              retn

  再看下数据

01625F78|FF FF FF FF|FF FF FF FF|58 20 58 00|...

  回忆一下正常调用时的参数是什么样的

01AA5083|07 00 00 00|0C 00 00 00|00 00 00 00|00 00 00 00
```

看出区别了吗？对了，`$00582058`就是正确的函数调用。

实际上进入函数`1004E395`后，程序会根据模块数、模块内函数数量以及`[☆ + 0x4C3 \* N + 0x4AA]`这三个数据查询`00582058`这个地址，找到后再把模块编号、默认函数编号写入`01625F78`处，最后按正常调用运算。

再根据`jmp xxxxxxxx`不是`call [xxxxxxxx]`的形式，可推出`00582058`就是正确的函数调用。

好了，SD的混乱统治终结了，一个新的时代到来了。

3、伪装成CALL指令的跳转

Dump数据，修复了IAT，所有的函数调用，和大部分错误代码，应该可以运行了吧，F9不久，程序又停了下来，内存访问异常。

跟踪下代码，确定错误的范围，如下：

```
00402777   .  52            push    edx

00402778   .  51            push    ecx

00402779   .  FF15 D4005000 call    dword ptr [5000D4]

0040277F   .  5F            pop     edi

00402780   >  8B8C24 080400>mov     ecx, dword ptr [esp+408]

00402787   .  338C24 0C0400>xor     ecx, dword ptr [esp+40C]

0040278E   .  5E            pop     esi

0040278F   .  81C4 08040000 add     esp, 408

00402795   .  E8 FF010000   call    00402999 // 出错的函数

0040279A      90            nop

0040279B      90            nop

0040279C      90            nop

00402999  /$  51            push    ecx

0040299A  |.  50            push    eax

0040299B  |.  E8 D3F1FFFF   call    00401B73

004029A0  |$  8B4424 0C     mov     eax, dword ptr [esp+C]

00401B73   $  B8 7B310000   mov     eax, 317B

00401B78   .  59            pop     ecx

00401B79   .  8D0408        lea     eax, dword ptr [eax+ecx] // 还是计算一个关键地址0x405B1B

00401B7C   .  8B00          mov     eax, dword ptr [eax] // 取出0x10011387

00401B7E   .  FFE0          jmp     eax // jump过去

10011387   .  58            pop     eax

10011388   .  59            pop     ecx

10011389   .  68 00004000   push    400000 // 算是一个小特征吧

1001138E   .  9C            pushfd

1001138F   .  60            pushad
```
