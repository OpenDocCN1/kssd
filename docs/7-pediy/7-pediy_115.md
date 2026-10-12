
# MoleBox 中的 SMC 分析

> 原文：[`www.pediy.com/kssd/pediy11/105626.html`](https://www.pediy.com/kssd/pediy11/105626.html)

**【文章标题】**: `MoleBox` 中的 `SMC` 分析  
**【文章作者】**: 一朵雪花  
**【软件名称】**: `MoleBox2.x`  
**【下载地址】**: 自己搜索下载  
**【加壳方式】**: `Molebox2.x`  
**【软件介绍】**: `OllyDBG`  
**【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

## 【详细过程】

在解包 `MoleBox` 文件的时候，发现 `MoleBox` 中有用到 `SMC` 的技术，顺便分析一下。

因为只是分析 `SMC` 部分，所以就略过跟踪过程，直接分析他的机制。

下面是 `SMC` 函数的汇编代码

```
00560A4D    55              push    ebp  
00560A4E    8BEC            mov     ebp, esp  
00560A50    53              push    ebx  
00560A51    56              push    esi  
00560A52    57              push    edi  
00560A53    60              pushad  
00560A54    33D2            xor     edx, edx                          ;edx 清 0  
00560A56    8D4D 08         lea     ecx, dword ptr [ebp+8]            ;参数一的地址，后面通过该地址加 4 找到要修改的代码地址  
00560A59    E8 11FFFFFF     call    0056096F  
00560A5E    61              popad  
00560A5F    5F              pop     edi  
00560A60    5E              pop     esi  
00560A61    5B              pop     ebx  
00560A62    5D              pop     ebp  
00560A63    C2 0400         retn    4  
```

下面我们进入 `0056096F` 处看看

```
0056096F    55              push    ebp  
00560970    8BEC            mov     ebp, esp  
00560972    6A FF           push    -1  
00560974    68 58A45600     push    0056A458  
00560979    68 4C855500     push    0055854C  
0056097E    64:A1 00000000  mov     eax, dword ptr fs:[0]  
00560984    50              push    eax  
00560985    64:8925 0000000>mov    dword ptr fs:[0], esp  
0056098C    51              push    ecx  
0056098D    51              push    ecx  
0056098E    83EC 1C         sub     esp, 1C  
00560991    53              push    ebx  
00560992    56              push    esi  
00560993    57              push    edi  
```

上面是设置 SEH 栈

```
00560994    8955 CC         mov     dword ptr [ebp-34], edx  
00560997    894D D0         mov     dword ptr [ebp-30], ecx        ; 参数地址  
```

```
temp_34 = edx;  //前面知道 edx=0  
temp_30 = ecx;  //ecx 为参数的地址  
```

```
0056099A    B8 C0975500     mov     eax, 005597C0  
0056099F    FF70 04         push    dword ptr [eax+4]  
005609A2    FF15 58D75600   call    dword ptr [56D758]           ; ntdll.RtlEnterCriticalSection  
005609A8    8365 FC 00      and     dword ptr [ebp-4], 0  
```

//进入 `CriticalSection`

```
005609AC    8B45 D0         mov     eax, dword ptr [ebp-30]  
005609AF    8B00            mov     eax, dword ptr [eax]  
005609B1    8945 DC         mov     dword ptr [ebp-24], eax  
```

```
temp_24 = 参数 1 的值  
```

```
005609B4    8B45 D0         mov     eax, dword ptr [ebp-30]  
005609B7    8B40 FC         mov     eax, dword ptr [eax-4]  
005609BA    8945 D4         mov     dword ptr [ebp-2C], eax  
```

```
temp_2c = 函数的返回地址的值  
```

```
005609BD    8B45 DC         mov     eax, dword ptr [ebp-24]  
005609C0    25 FFFF0000     and     eax, 0FFFF  
005609C5    8B4D D4         mov     ecx, dword ptr [ebp-2C]  
005609C8    2BC8            sub     ecx, eax  
005609CA    894D E4         mov     dword ptr [ebp-1C], ecx  
```

```
temp_1c = temp_2c + LOWWORD(temp_24) = 函数的返回地址－LOWWORD(arg1);  
```

```
005609CD    8B45 DC         mov     eax, dword ptr [ebp-24]  
005609D0    C1E8 10         shr     eax, 10  
005609D3    8B4D D4         mov     ecx, dword ptr [ebp-2C]  
005609D6    03C8            add     ecx, eax  
005609D8    894D D8         mov     dword ptr [ebp-28], ecx  
```

```
temp_28 = 函数的返回地址－HIGHWORD(arg1);  
```

```
005609DB    8B45 E4         mov     eax, dword ptr [ebp-1C]  
005609DE    8945 E0         mov     dword ptr [ebp-20], eax  
005609E1    EB 07           jmp     short 005609EA  
005609E3    8B45 E4         mov     eax, dword ptr [ebp-1C]  
005609E6    40              inc     eax  
005609E7    8945 E4         mov     dword ptr [ebp-1C], eax  
005609EA    8B45 E4         mov     eax, dword ptr [ebp-1C]  
005609ED    3B45 D8         cmp     eax, dword ptr [ebp-28]  
005609F0    74 1C           je      short 00560A0E  
005609F2    8B45 E4         mov     eax, dword ptr [ebp-1C]  
005609F5    69C0 0D661900   imul    eax, eax, 19660D  
005609FB    05 5FF36E3C     add     eax, 3C6EF35F  
00560A00    8B4D E4         mov     ecx, dword ptr [ebp-1C]  
00560A03    8A09            mov     cl, byte ptr [ecx]  
00560A05    32C8            xor     cl, al  
00560A07    8B45 E4         mov     eax, dword ptr [ebp-1C]  
00560A0A    8808            mov     byte ptr [eax], cl  
00560A0C   ^ EB D5         jmp     short 005609E3  
```

```
for(int i=temp_1c; i<temp_28; i++){  
    *((char*)i) ^= (char)(i * 0x19660D + 0x3C6EF35F);  
}  
```

```
00560A0E    8B45 D8         mov     eax, dword ptr [ebp-28]  
00560A11    2B45 E0         sub     eax, dword ptr [ebp-20]  
00560A14    50              push    eax  
00560A15    FF75 E0         push    dword ptr [ebp-20]  
00560A18    FF35 20D75600   push    dword ptr [56D720]  
00560A1E    FF15 04D75600   call    dword ptr [56D704]           ; kernel32.FlushInstructionCache  
```

//把指令写入缓存

```
00560A24    834D FC FF      or      dword ptr [ebp-4], FFFFFFFF  
00560A28    E8 02000000     call    00560A2F                    ;退出 CriticalSection  
00560A2D    EB 0F           jmp     short 00560A3E  
00560A2F    B8 C0975500     mov     eax, 005597C0  
00560A34    FF70 04         push    dword ptr [eax+4]  
00560A37    FF15 E8D75600   call    dword ptr [56D7E8]           ; ntdll.RtlLeaveCriticalSection  
00560A3D    C3              retn  
00560A3E    8B4D F0         mov     ecx, dword ptr [ebp-10]  
00560A41    64:890D 0000000>mov    dword ptr fs:[0], ecx  
00560A48    5F              pop     edi  
00560A49    5E              pop     esi  
00560A4A    5B              pop     ebx  
00560A4B    C9              leave  
00560A4C    C3              retn  
```

上面就是 `SMC` 的原理：函数有一个整型参数,参数的低 16 位保存这要往前修改的指令字节码数，高 16 位保存向后修改的指令字节码数。

下面我们看一个该函数的完整过程。

```
005600AC   > \68 00001200   push    120000  
005600B1   .   E8 689BFFFF   call    <fn_smc>                     ;  要改变 005600B6 开始的 0x12 个字节  
005600B6   .   0D 3A840D54   or      eax, 540D843A  
005600BB   .   17            pop     ss  
005600BC   .   9E            sahf  
005600BD   .   16            push    ss  
005600BE   .   6D            ins     dword ptr es:[edi], dx  
005600BF   .   001F          add     byte ptr [edi], bl  
005600C1   .   05 39AE05FB   add     eax, FB05AE39  
005600C6   .   92            xchg    eax, edx  
005600C7   .   8517          test    dword ptr [edi], edx  
005600C9   .   04 92         add     al, 92  
```

函数执行后指令变为

```
005600B6   .   90            nop  
005600B7   ?   90            nop  
005600B8   ?   33C9          xor     ecx, ecx  
005600BA   ?   85C9          test    ecx, ecx  
005600BC   .^  75 EE         jnz     short 005600AC  
005600BE   .   68 12002900   push    290012                        ;  UNICODE "yy-MM-dd"  
005600C3   ?   E8 569BFFFF   call    <fn_smc>                      
005600C8   ?   17            pop     ss  
005600C9   .   04 92         add     al, 92  
005600CB   .   7C 3E         jl      short 0056010B  
005600CD   .   1AA0 0C4EE4DE sbb     ah, byte ptr [eax+DEE44E0C]  
005600D3   .   40            inc     eax  
005600D4   .   23BB 7566D4A5 and     edi, dword ptr [ebx+A5D46675]  
005600DA   .   51            push    ecx  
005600DB   .   2F            das  
005600DC   .   06            push    es                            ;  保存和密钥相关的东西  
005600DD   .   15 754F4033   adc     eax, 33404F75  
005600E2   .   319411 FFF272>xor     dword ptr [ecx+edx+E72F2FF], edx  
```



```
  005600E9   .  34 69         xor     al, 69  
  005600EB   .  4E            dec     esi  
  005600EC   .  B3 45         mov     bl, 45  
  005600EE   .  EE            out     dx, al  
  005600EF   .  7D 70         jge     short 00560161  
  005600F1   .  0C 39         or      al, 39  
  005600F3   .  8511          test    dword ptr [ecx], edx  
```

可见这里要修改 `005600C8` 前 `0x12` 个字节开始修改(`0x12` + `0x29`)个字节,首先恢复（因为实际是指令的改变是异或运算）上次调用修改的 `0x12` 个字节。

执行后指令变为

```
  005600C8   ?  90            nop  
  005600C9   ?  90            nop  
  005600CA   ?  33D2          xor     edx, edx  
  005600CC   ?  85D2          test    edx, edx  
  005600CE   ?^ 75 EE         jnz     short 005600BE  
  005600D0   .  A1 18D75600   mov     eax, dword ptr [56D718]  
  005600D5   .  8B48 2C       mov     ecx, dword ptr [eax+2C]  
  005600D8   .  83C1 20       add     ecx, 20  
  005600DB   .  51            push    ecx  
  005600DC   .  8D8D D0FDFFFF lea     ecx, dword ptr [ebp-230]        ;   
  005600E2   .  E8 72E2FFFF   call    0055E359                       ;这里调用了一个函数  
  005600E7   .  68 29002800   push    280029                         ;这里恢复前面修改的 0x29 个字节，修改下面的 0x28 个字节  
  005600EC   .  E8 2D9BFFFF   call    <fn_smc>                       ;执行 SMC 函数  
  005600F1   .  0C 39         or      al, 39  
  005600F3   .  8511          test    dword ptr [ecx], edx  
  005600F5   .  55            push    ebp  
  005600F6   .  0F9F19        setg    byte ptr [ecx]  
  005600F9     8F            db      8F  
  005600FA   .  54            push    esp  
  005600FB   .  BE EAD04602  mov     esi, 246D0EA  
  00560100   .  D4 21         aam     21  
  00560102   .  A5            movs    dword ptr es:[edi], dword ptr [e>  
  00560103   .  D7            xlat    byte ptr [ebx+al]  
  00560104   .  1E            push    ds  
  00560105   .  2D 7D47382B  sub     eax, 2B38477D  
  0056010A   .  096A 16       or      dword ptr [edx+16], ebp  
  0056010D   .  F7EA          imul    edx  
  0056010F   >  4A            dec     edx  
  00560110   .  07            pop     es  
  00560111   .  3C 49         cmp     al, 49  
  00560113   .  56            push    esi  
  00560114   .  8B75 E6       mov     esi, dword ptr [ebp-1A]  
  00560117   .  75 68         jnz     short 00560181  
  00560119   .  90            nop  
```

执行后指令变为

```
  005600F1   .  90            nop  
  005600F2   ?  90            nop  
  005600F3   .  33D2          xor     edx, edx  
  005600F5   .  85D2          test    edx, edx  
  005600F7   ?^ 75 EE         jnz     short 005600E7  
  005600F9     8B            db      8B  
  005600FA   .  45            inc     ebp  
  005600FB   .  A0 C1E80350  mov     al, byte ptr [5003E8C1]  
  00560100   .  8B4D DC       mov     ecx, dword ptr [ebp-24]  
  00560103   .  51            push    ecx  
  00560104   .  8D8D D0FDFFFF lea     ecx, dword ptr [ebp-230]  
  0056010A   .  E8 84EDFFFF   call    0055EE93                       ；这里执行了一个函数  
  0056010F   >  68 28000000   push    28  
  00560114   .  E8 059BFFFF   call    <fn_smc>                       ；向前恢复 0x28 个字节指令。  
  00560119   .  90            nop  
```

至此指令数据完全恢复原状。

SMC 函数的参数依次为 `0x120000`, `0x290012`, `0x280029`, `0x28`, 恰好是一个闭合环。

下面使我对 Molebox 如何实现这一点做的一些猜测：

1. 首先是利用下面的字节序列做特征串。

```
  005600F1   .  90            nop  
  005600F2   ?  90            nop  
  005600F3   .  33D2          xor     edx, edx  
  005600F5   .  85D2          test    edx, edx  
  005600F7   ?^ 75 EE         jnz     short 005600E7  
```

2. 函数修改时按修改后的代码进行编写。

3. 在生成的目标文件中搜索特征串，并计算该处指令映射到内存中后的地址，然后对地址做加密操作。

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2010 年 01 月 21 日 15:21:14
