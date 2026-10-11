
### 0045B1E7 区域：改写 PEB 信息

```
0045B1E7    8D58 20         lea ebx,dword ptr ds:[eax+20]  

0045B1EA    8D48 18         lea ecx,dword ptr ds:[eax+18]  

0045B1ED    8103 C8000000   add dword ptr ds:[ebx],0C8  

0045B1F3    B8 00000000     mov eax,0  

0045B1F8    0101           add dword ptr ds:[ecx],eax  

0045B1FA    33C9           xor ecx,ecx  

0045B1FC    E8 00000000     call 复件_[MS.0045B201
```

改写 `PEB` 中程序在内存中的基址和大小信息，大小`+0c8`，基址`+0`。有意义？

```
0045B1FC    E8 00000000     call 复件_[MS.0045B201  

0045B201    5F             pop edi                                   ; 复件_[MS.0045B201  

0045B202    81C7 C1090000  add edi,9C1  

0045B208    0FB60439       movzx eax,byte ptr ds:[ecx+edi]  

0045B20C    83F0 11        xor eax,11  

0045B20F    880439        mov byte ptr ds:[ecx+edi],al  

0045B212    41             inc ecx  

0045B213    81F9 521D0000  cmp ecx,1D52  

0045B219   ^ 72 ED        jb short 复件_[MS.0045B208  

0045B21B    EB 05         jmp short 复件_[MS.0045B222
```

`0045bbc2-0045d914` 解码，这次是 `XOR 11`。

```
0045C56C    E8 00000000     call [MSLRH].0045C571  

0045C571    810424 CA090000 add dword ptr ss:[esp],9CA  

0045C578    64:FF35 0000000>push dword ptr fs:[0]  

0045C57F    64:8925 0000000>mov dword ptr fs:[0],esp  

0045C586    33DB          xor ebx,ebx  

0045C588    8B1B          mov ebx,dword ptr ds:[ebx]
```

安装完 `SEH` 就立刻产生内存访问错误进入了。

```
0045D8DF    8B4424 0C      mov eax,dword ptr ss:[esp+C]  

0045D8E3    33C9          xor ecx,ecx  

0045D8E5    3348 04       xor ecx,dword ptr ds:[eax+4]  

0045D8E8    3348 08       xor ecx,dword ptr ds:[eax+8]  

0045D8EB    3348 0C       xor ecx,dword ptr ds:[eax+C]  

0045D8EE    3348 10       xor ecx,dword ptr ds:[eax+10]
```

继续检查 `DRX`。

```
0045DA76    884E 0D        mov byte ptr ds:[esi+D],cl
```

这里将检查结果存在 `0046800D` 了。

```
0045E42A    FF56 14        call dword ptr ds:[esi+14]                ; KERNEL32.OutputDebugStringA  

0045EDD4    FF56 18        call dword ptr ds:[esi+18]                ; KERNEL32.GetCommandLineA  

0045EDD7    40             inc eax  

0045EDD8    33C9          xor ecx,ecx  

0045EDDA    41             inc ecx  

0045EDDB    803C01 00      cmp byte ptr ds:[ecx+eax],0  

0045EDDF    74 0C         je short 复件_[MS.0045EDED  

0045EDE1    803C01 22      cmp byte ptr ds:[ecx+eax],22  

0045EDE5   ^ 75 F3        jnz short 复件_[MS.0045EDDA  

0045EDE7    C60401 00      mov byte ptr ds:[ecx+eax],0  

0045EDEB   ^ EB ED        jmp short 复件_[MS.0045EDDA  

0045EDED    6A 00         push 0  

0045EDEF    6A 00         push 0  

0045EDF1    6A 03         push 3  

0045EDF3    6A 00         push 0  

0045EDF5    6A 00         push 0  

0045EDF7    68 00000080   push 80000000  

0045EDFC    50             push eax  

0045EDFD    FF56 1C        call dword ptr ds:[esi+1C]
```

一段计算字符长度未经优化的代码。然后是 `CREATEFILE` 打开自己，这样就 `DUMP` 不了了。修改 `EIP` 直接跳过。

```
0045F7A7    837E 40 00     cmp dword ptr ds:[esi+40],0  

0045F7AB    74 24          je short [MSLRH].0045F7D1  

0045F7AD    FF56 24        call dword ptr ds:[esi+24]  

0045F7B0    50             push eax  

0045F7B1    6A 00          push 0  

0045F7B3    68 00040000    push 400  

0045F7B8    FF56 28        call dword ptr ds:[esi+28]  

0045F7BB    8BDC           mov ebx,esp  

0045F7BD    83EB 04        sub ebx,4  

0045F7C0    6A 00          push 0  

0045F7C2    6A 00          push 0  

0045F7C4    6A 04          push 4  

0045F7C6    53             push ebx  

0045F7C7    6A 07          push 7  

0045F7C9    50             push eax  

0045F7CA    FF56 40        call dword ptr ds:[esi+40]  

0045F7CD    58             pop eax  

0045F7CE    8846 0E        mov byte ptr ds:[esi+E],al
```

比较 `ntdll.` 是 `ZwQueryInformationProcess` 否获取成功，获取成功就用该 `API` 检查是否被 `DEBUG`，将结果写入 `0046800E`。

```
00460178    8CC9           mov cx,cs  

0046017A    32C9          xor cl,cl  

0046017C    83F9 00        cmp ecx,0  

0046017F    0F84 A1130000  je [MSLRH].00461526  

00460185    8B46 38        mov eax,dword ptr ds:[esi+38]  

00460188    8078 01 4C     cmp byte ptr ds:[eax+1],4C  

0046018C    0F85 94130000  jnz [MSLRH].00461526
```

检查系统，前半部分检查是不是 `WIN9X`，如果是则通过。

`SetUnhandledExceptionFilter`

在 `KERNEL32` 中的偏移来确定具体是哪一个系统？我这里是 `2K`，跳走了。

```
00461ED2    58             pop eax                                   ; [MSLRH].00461ED2  

00461ED3    2D E2BD0000    sub eax,0BDE2  

00461ED8    B0 00          mov al,0  

00461EDA    05 00200100    add eax,12000  

00461EDF    8BF0           mov esi,eax  

00461EE1    807E 0C 00     cmp byte ptr ds:[esi+C],0  

00461EE5    74 51          je short [MSLRH].00461F38  

00461F38    807E 0D 00     cmp byte ptr ds:[esi+D],0  

00461F3C   ^ 0F85 AF41FFFF jnz [MSLRH].004560F1  

00461F42    74 04          je short [MSLRH].00461F48  

00461F44    75 02          jnz short [MSLRH].00461F48  

00461F4B    807E 0E 00     cmp byte ptr ds:[esi+E],0  

00461F4F   ^ 0F85 9C41FFFF jnz [MSLRH].004560F1  

00461F55    E8 0A000000    call [MSLRH].00461F64  

00461F6C    807E 0F 00     cmp byte ptr ds:[esi+F],0  

00461F70   ^ 0F85 7B41FFFF jnz [MSLRH].004560F1  

00461F76    50             push eax  

00461F77    E8 02000000    call [MSLRH].00461F7E
```

`0046800F` 不知道是什么的校验值，估计是针对某操作系统的。后面开始自校验了，清除所有断点，还原花指令。

```
00461F92    59             pop ecx  

00461F9D    83E9 05        sub ecx,5  

00461FAA    33DB          xor ebx,ebx  

00461FB6    B8 9CBE0000    mov eax,0BE9C  

00461FC5    8BF9           mov edi,ecx                                 ; [MSLRH].00461F8D  

00461FD1    2BF8           sub edi,eax  

00461FDD    0FB607        movzx eax,byte ptr ds:[edi]  

00461FEA    03D8           add ebx,eax  

00461FF6    47             inc edi                                     ; [MSLRH].004560F1  

00462001    3BF9           cmp edi,ecx                                 ; [MSLRH].00461F8D  

0046200D  ^\72 CE          jb short [MSLRH].00461FDD
```

计算 `004560f1` 到 `00461f8d` 的校验和（从花指令入口处到进入自校验之前的代码）。

```
0046200F    BF 00704400    mov edi,[MSLRH].00447000  

00462014    B9 00BC0000    mov ecx,0BC00  

00462023    0FB607        movzx eax,byte ptr ds:[edi]  

00462030    02DF           add bl,bh  

00462032    32DF           xor bl,bh  

00462034    32C3           xor al,bl  

00462040    8807           mov byte ptr ds:[edi],al  

0046204C    47             inc edi  

00462057    49             dec ecx  

00462062  ^\75 B5          jnz short [MSLRH].00462019
```

对 `00447000` 开始大小为 `BC00` 的区域解码，`KEY` 就是刚才的校验和。

```
00462064    E8 00000000    call [MSLRH].00462069  

00462069    59             pop ecx  

0046206A    2959 16        sub dword ptr ds:[ecx+16],ebx  

0046206D    61             popad  

0046206E    60             pushad  

0046206F    BE 00704400    mov esi,[MSLRH].00447000  

00462074    8DBE 00A0FBFF  lea edi,dword ptr ds:[esi+FFFBA000]  

0046207A    57             push edi  

0046207B    83CD FF        or ebp,FFFFFFFF  

0046207E    68 0FDE9F00     push 9FDE0F  

00462083    C3             retn
```

这里的结果仍然跟校验和关联。

```
0046206A    2959 16        sub dword ptr ds:[ecx+16],ebx
```

之后，最后两条指令便变成：

```
0046207E    68 D0284500    push [MSLRH].004528D0  

00462083    C3             retn
```

跳到下一部分，据前辈的文章说，就是 `UPX` 核心了。

`004528d8-004529a1` 是将 `00447000` 解压到 `00401000`。

`004529e0-00452a1a` 是填写 `IAT`（在 `00450fe7` 获得 `API` 名字，将地址填进 `00415028`，就是说没加密）。

用 `OLLYDUMP` 下来就完成脱壳了。

### 后记

同一个壳的不同版本，由于有了前一个版本的经验，这次跟起来很轻松，显然经验是大多数新人所欠缺的，只能将勤补拙了。另外一个问题就是，当你遇到你所不知道的 `ANTI` 时，你会怎么办？到论坛请教高手？还是尝试自己弄清楚？我只想说，探索精神很重要。有的时候，象这里的例子，其实完全可以自己解决的。此外通过脱这个壳，有的 `API` 的运用，其实可以很巧妙，例如本例子中的 `SetUnhandledExceptionFilter`，无论你怎么 `BP`，都是断不下来的。

**源程序下载**  
`http://emadicius.rvlcnsecurity.com/programas/MSLRHv0.32a.zip`

**已脱壳的程序**  
`http://www.nxer.cn/709394/attachment/1163728509_0.rar`

用 `OD` 加载程序时报错。现在用 `WINDBG` 载入 `OD`，`bp messageboxa`。用 `OD` 载入 `MSLRHv0.32a`。断下。

```
0:000> kb  

ChildEBP RetAddr  Args to Child             
```
