
# 英语学习软件 2006 build 1201 真人发音版 算法分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-392.htm`](https://www.pediy.com/kssd/pediy09/pediy09-392.htm)

> 【文章标题】: 英语学习软件 2006 build 1201 真人发音版 算法分析
> 【文章作者】: `anchovy`
> 【作者邮箱】: `canyun3160@tom.com`
> 【作者 QQ 号】: `276055658`
> 【软件名称】: 英语学习软件
> 【下载地址】: `http://www.yyxxi.com/download.html`
> 【保护方式】: 注册码
> 【编写语言】: `Borland Delphi 6.0 - 7.0`
> 【使用工具】: `OllyICE`
> 【操作平台】: `WinXP`
> 【软件介绍】: 助您学英语、单词、口语、音标和语法的极品英语学习
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

### 【详细过程】

助您学英语、单词、口语、音标和语法的极品英语学习软件。

提供了幼儿，小学，中学，高中，大学，研究生，托福等各级别的 230 多个常用词库。

提供了情景对话，常用口语等 10000 多条。

软件用两个可爱的卡通人物对话的形式学习，十分有趣，让你在轻松的环境中学习，成倍提高学习效果。

软件用 10 多个动画文件帮助你全面学会弄懂音标，用 100 多个动画文件帮助你全面学会弄懂语法，每个动画里面还配有小测试哦。

学习内容可以转成 `mp3`，可以拷到 `mp3` 随身播放器中播放。提供英语跟读功能，实时检查你的语音和标准语音的差别。

`PEID` 检查无壳，乌龟不顶壳子，好危险啦！！！

用 `DeDe` 反编译，找到按钮事件，了解大概的流程，用 `OllyICE` 加载动态分析！

```assembly
0049A848   /.  55             push    ebp
0049A849   |.  8BEC           mov     ebp, esp
0049A84B   |.  B9 08000000    mov     ecx, 8
0049A850   |>  6A 00          /push    0
0049A852   |.  6A 00          |push    0
0049A854   |.  49             |dec     ecx
0049A855   |.^ 75 F9          \jnz     short 0049A850
0049A857   |.  8945 FC        mov     dword ptr [ebp-4], eax
0049A85A   |.  33C0           xor     eax, eax
0049A85C   |.  55            push    ebp
0049A85D   |.  68 37AA4900   push    <->System.@HandleFinally;>
0049A862   |.  64:FF30       push    dword ptr fs:[eax]
0049A865   |.  64:8920       mov     dword ptr fs:[eax], esp
0049A868   |.  8D55 F0       lea     edx, dword ptr [ebp-10]
0049A86B   |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0049A86E   >|.  8B80 08030000 mov     eax, dword ptr [eax+308]
0049A874   >|.  E8 1390FBFF  call    0045388C                   ; 得到用户名
0049A879   |.  8B45 F0       mov     eax, dword ptr [ebp-10]
0049A87C   |.  8D55 F4       lea     edx, dword ptr [ebp-C]
0049A87F   >|.  E8 20E1F6FF  call    004089A4                   ; 用户名不能为空！
0049A884   |.  837D F4 00    cmp     dword ptr [ebp-C], 0
0049A888   |.  0F84 4C010000 je      0049A9DA
0049A88E   |.  8D55 E8       lea     edx, dword ptr [ebp-18]
0049A891   |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0049A894   >|.  8B80 10030000 mov     eax, dword ptr [eax+310]
0049A89A   >|.  E8 ED8FFBFF  call    0045388C                   ; 得到输入的注册码
0049A89F   |.  8B45 E8       mov     eax, dword ptr [ebp-18]
0049A8A2   |.  8D55 EC       lea     edx, dword ptr [ebp-14]
0049A8A5   >|.  E8 FAE0F6FF  call    004089A4
0049A8AA   |.  837D EC 00    cmp     dword ptr [ebp-14], 0        ; 注册码不能为空！
0049A8AE   |.  0F84 26010000 je      0049A9DA
0049A8B4   |.  8D55 E0       lea     edx, dword ptr [ebp-20]
0049A8B7   |.  A1 E0774B00   mov     eax, dword ptr [4B77E0]
0049A8BC   |.  8B00          mov     eax, dword ptr [eax]
0049A8BE   >|.  E8 5D93FDFF  call    00473C20                   ; GetExeName(TDdeMgr)
0049A8C3   |.  8B45 E0       mov     eax, dword ptr [ebp-20]
0049A8C6   |.  8D55 E4       lea     edx, dword ptr [ebp-1C]
0049A8C9   >|.  E8 AAF6FFFF  call    00499F78                   ; ExtractFilePath(AnsiString):AnsiString
0049A8CE   |.  8B55 E4       mov     edx, dword ptr [ebp-1C]
0049A8D1   |.  8D45 F8       lea     eax, dword ptr [ebp-8]
0049A8D4   |.  B9 4CAA4900   mov     ecx, 0049AA4C
0049A8D9   >|.  E8 969FF6FF  call    00404874                   ; LStrCat3
0049A8DE   |.  8D55 D4       lea     edx, dword ptr [ebp-2C]
0049A8E1   |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0049A8E4   >|.  8B80 10030000 mov     eax, dword ptr [eax+310]
0049A8EA   >|.  E8 9D8FFBFF  call    0045388C
0049A8EF   |.  8B45 D4       mov     eax, dword ptr [ebp-2C]
0049A8F2   |.  8D55 D8       lea     edx, dword ptr [ebp-28]
0049A8F5   >|.  E8 AAE0F6FF  call    004089A4
0049A8FA   |.  8B45 D8       mov     eax, dword ptr [ebp-28]
0049A8FD   |.  50            push    eax
0049A8FE   |.  8D55 CC       lea     edx, dword ptr [ebp-34]
0049A901   |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0049A904   >|.  8B80 0C030000 mov     eax, dword ptr [eax+30C]
0049A90A   >|.  E8 7D8FFBFF  call    0045388C                   ; 得到机器码
0049A90F   |.  8B45 CC       mov     eax, dword ptr [ebp-34]
0049A912   |.  8D55 D0       lea     edx, dword ptr [ebp-30]
0049A915   >|.  E8 8AE0F6FF  call    004089A4
0049A91A   |.  8B45 D0       mov     eax, dword ptr [ebp-30]
0049A91D   |.  8D4D DC       lea     ecx, dword ptr [ebp-24]
0049A920   |.  5A            pop     edx
0049A921   >|.  E8 EEF1FFFF  call    00499B14
0049A926   |.  8B55 DC       mov     edx, dword ptr [ebp-24]
0049A929   |.  A1 8C744B00   mov     eax, dword ptr [4B748C]
0049A92E   >|.  E8 899CF6FF  call    004045BC
0049A933   |.  6A 00         push    0
0049A935   |.  8B15 8C744B00 mov     edx, dword ptr [4B748C]      ; English.004B8E3C
0049A93B   |.  8B12          mov     edx, dword ptr [edx]
0049A93D   |.  8B4D F8       mov     ecx, dword ptr [ebp-8]
0049A940   |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0049A943   >|.  E8 E0F6FFFF  call    0049A028                   ; ->关键 call
0049A948   |.  84C0          test    al, al
0049A94A   |.  74 6A         je      short 0049A9B6               ; ->关键跳转
0049A94C   |.  A1 00764B00   mov     eax, dword ptr [4B7600]
0049A951   |.  C600 01       mov     byte ptr [eax], 1
0049A954   |.  8D55 C4       lea     edx, dword ptr [ebp-3C]
0049A957   |.  A1 B0754B00   mov     eax, dword ptr [4B75B0]
0049A95C   |.  8B00          mov     eax, dword ptr [eax]
0049A95E   >|.  E8 298FFBFF  call    0045388C
0049A963   |.  FF75 C4       push    dword ptr [ebp-3C]
0049A966   |.  68 68AA4900   push    0049AA68                   ; ASCII " - \["
0049A96B   |.  8D55 C0       lea     edx, dword ptr [ebp-40]
0049A96E   |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0049A971   >|.  8B80 08030000 mov     eax, dword ptr [eax+308]
0049A977   >|.  E8 108FFBFF  call    0045388C
0049A97C   |.  FF75 C0       push    dword ptr [ebp-40]
0049A97F   |.  68 78AA4900   push    0049AA78
0049A984   |.  8D45 C8       lea     eax, dword ptr [ebp-38]
0049A987   |.  BA 04000000   mov     edx, 4
0049A98C   >|.  E8 579FF6FF  call    004048E8
0049A991   |.  8B55 C8       mov     edx, dword ptr [ebp-38]
0049A994   |.  A1 B0754B00   mov     eax, dword ptr [4B75B0]
0049A999   |.  8B00          mov     eax, dword ptr [eax]
0049A99B   >|.  E8 1C8FFBFF  call    004538BC
0049A9A0   |.  55            push    ebp
0049A9A1   |.  E8 BAFDFFFF  call    0049A760
0049A9A6   |.  59            pop     ecx
0049A9A7   |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0049A9AA   |.  C780 4C020000 mov     dword ptr [eax+24C], 1
0049A9B4   |.  EB 24         jmp     short 0049A9DA
0049A9B6   |>  6A 10         push    10
0049A9B8   |.  68 7CAA4900   push    0049AA7C
0049A9BD   |.  68 88AA4900   push    0049AA88                   ; 注册码输入错误
0049A9C2   |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0049A9C5   >|.  E8 E2F6FBFF  call    0045A0AC
0049A9CA   |.  50            push    eax                        ; |hOwner
```



```asm
  0049A9CB >|.  E8 70C9F6FF   call    <jmp.&user32.MessageBoxA>        ; \MessageBoxA
  0049A9D0  |.  A1 8C744B00   mov     eax, dword ptr [4B748C]
  0049A9D5 >|.  E8 8E9BF6FF   call    00404568
  0049A9DA  |>  33C0          xor     eax, eax
  0049A9DC  |.  5A            pop     edx
  0049A9DD  |.  59            pop     ecx
  0049A9DE  |.  59            pop     ecx
  0049A9DF  |.  64:8910       mov     dword ptr fs:[eax], edx
  0049A9E2  |.  68 3EAA4900   push    0049AA3E
  0049A9E7  |>  8D45 C0       lea     eax, dword ptr [ebp-40]
  0049A9EA  |.  BA 04000000   mov     edx, 4
  0049A9EF >|.  E8 989BF6FF   call    0040458C
  0049A9F4  |.  8D45 D0       lea     eax, dword ptr [ebp-30]
  0049A9F7 >|.  E8 6C9BF6FF   call    00404568
  0049A9FC  |.  8D45 D4       lea     eax, dword ptr [ebp-2C]
  0049A9FF >|.  E8 649BF6FF   call    00404568
  0049AA04  |.  8D45 D8       lea     eax, dword ptr [ebp-28]
  0049AA07  |.  BA 04000000   mov     edx, 4
  0049AA0C >|.  E8 7B9BF6FF   call    0040458C
  0049AA11  |.  8D45 E8       lea     eax, dword ptr [ebp-18]
  0049AA14 >|.  E8 4F9BF6FF   call    00404568
  0049AA19  |.  8D45 EC       lea     eax, dword ptr [ebp-14]
  0049AA1C >|.  E8 479BF6FF   call    00404568
  0049AA21  |.  8D45 F0       lea     eax, dword ptr [ebp-10]
  0049AA24 >|.  E8 3F9BF6FF   call    00404568
  0049AA29  |.  8D45 F4       lea     eax, dword ptr [ebp-C]
  0049AA2C  |.  BA 02000000   mov     edx, 2
  0049AA31 >|.  E8 569BF6FF   call    0040458C
  0049AA36  \.  C3            retn
  0049AA37  > .^ E9 1095F6FF  jmp     00403F4C
  0049AA3C   .^  EB A9        jmp     short 0049A9E7
  0049AA3E   .   8BE5         mov     esp, ebp
  0049AA40   .   5D           pop     ebp
  0049AA41   .   C3           retn
```

关键`call`的代码如下：

```asm
  0049A028  /$  55            push    ebp
  0049A029  |.  8BEC          mov     ebp, esp
  0049A02B  |.  81C4 70FAFFFF add     esp, -590
  0049A031  |.  53            push    ebx
  0049A032  |.  56            push    esi
  0049A033  |.  57            push    edi
  0049A034  |.  33DB          xor     ebx, ebx
  0049A036  |.  895D E8       mov     dword ptr [ebp-18], ebx
  0049A039  |.  895D D0       mov     dword ptr [ebp-30], ebx
  0049A03C  |.  894D F8       mov     dword ptr [ebp-8], ecx
  0049A03F  |.  8955 FC       mov     dword ptr [ebp-4], edx
  0049A042  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
  0049A045  |.  E8 CEA9F6FF   call    00404A18
  0049A04A  |.  8B45 F8       mov     eax, dword ptr [ebp-8]
  0049A04D  |.  E8 C6A9F6FF   call    00404A18
  0049A052  |.  33C0          xor     eax, eax
  0049A054  |.  55            push    ebp
  0049A055  |.  68 10A54900   push    0049A510
  0049A05A  |.  64:FF30       push    dword ptr fs:[eax]
  0049A05D  |.  64:8920       mov     dword ptr fs:[eax], esp
  0049A060  |.  8D45 E8       lea     eax, dword ptr [ebp-18]
  0049A063  |.  8B55 FC       mov     edx, dword ptr [ebp-4]
  0049A066  |.  E8 95A5F6FF   call    00404600
  0049A06B  |.  33DB          xor     ebx, ebx
  0049A06D  |.  8B45 F8       mov     eax, dword ptr [ebp-8]
  0049A070  |.  E8 23FEFFFF   call    00499E98                   ;  判断"scene\\英语 900 句.xml"文件是否存在
  0049A075  |.  84C0          test    al, al
  0049A077  |.  0F84 68040000 je      0049A4E5
  0049A07D  |.  BA 40000000   mov     edx, 40
  0049A082  |.  8B45 F8       mov     eax, dword ptr [ebp-8]
  0049A085  |.  E8 5EFDFFFF   call    00499DE8
  0049A08A  |.  8BF0          mov     esi, eax
  0049A08C  |.  85F6          test    esi, esi
  0049A08E  |.  0F8C 51040000 jl      0049A4E5
  0049A094  |.  8BC6          mov     eax, esi
  0049A096  |.  E8 19FEFFFF   call    00499EB4                   ;  返回文件长度
  0049A09B  |.  8BF8          mov     edi, eax
  0049A09D  |.  83FF 08       cmp     edi, 8                       ;  长度小于 8 则 Game Over
  0049A0A0  |.  0F8C 3F040000 jl      0049A4E5
  0049A0A6  |.  33C9          xor     ecx, ecx
  0049A0A8  |.  33D2          xor     edx, edx
  0049A0AA  |.  8BC6          mov     eax, esi
  0049A0AC  |.  E8 F7FDFFFF   call    00499EA8
  0049A0B1  |.  8D55 F4       lea     edx, dword ptr [ebp-C]
  0049A0B4  |.  B9 04000000   mov     ecx, 4
  0049A0B9  |.  8BC6          mov     eax, esi
  0049A0BB  |.  E8 2CFEFFFF   call    00499EEC                   ;  文件首读取了 4 字节,比较是否是 09750409
  0049A0C0  |.  817D F4 09047 cmp     dword ptr [ebp-C], 19750409
  0049A0C7  |.  0F94C3        sete    bl                         ;  (initial cpu selection)
  0049A0CA  |.  80F3 01       xor     bl, 1
  0049A0CD  |.  84DB          test    bl, bl
  0049A0CF  |.  75 0D         jnz     short 0049A0DE
  0049A0D1  |.  33C9          xor     ecx, ecx
  0049A0D3  |.  33D2          xor     edx, edx
  0049A0D5  |.  8BC6          mov     eax, esi
  0049A0D7  |.  E8 CCFDFFFF   call    00499EA8
  0049A0DC  |.  EB 65         jmp     short 0049A143
  0049A0DE  |>  B9 02000000   mov     ecx, 2
  0049A0E3  |.  BA F8FFFFFF   mov     edx, -8
  0049A0E8  |.  8BC6          mov     eax, esi
  0049A0EA  |.  E8 B9FDFFFF   call    00499EA8
  0049A0EF  |.  8D55 F0       lea     edx, dword ptr [ebp-10]
  0049A0F2  |.  B9 04000000   mov     ecx, 4
  0049A0F7  |.  8BC6          mov     eax, esi
  0049A0F9  |.  E8 EEFDFFFF   call    00499EEC
  0049A0FE  |.  8D55 F4       lea     edx, dword ptr [ebp-C]
  0049A101  |.  B9 04000000   mov     ecx, 4
  0049A106  |.  8BC6          mov     eax, esi
  0049A108  |.  E8 DFFDFFFF   call    00499EEC
  0049A10D  |.  817D F4 09047 cmp     dword ptr [ebp-C], 19750409
  0049A114  |.  75 2D         jnz     short 0049A143
  0049A116  |.  8BC7          mov     eax, edi
  0049A118  |.  83E8 0C       sub     eax, 0C
  0049A11B  |.  3B45 F0       cmp     eax, dword ptr [ebp-10]
  0049A11E  |.  7E 23         jle     short 0049A143
  0049A120  |.  8B55 F0       mov     edx, dword ptr [ebp-10]
  0049A123  |.  F7DA          neg     edx
  0049A125  |.  83EA 0C       sub     edx, 0C
  0049A128  |.  B9 02000000   mov     ecx, 2
  0049A12D  |.  8BC6          mov     eax, esi
  0049A12F  |.  E8 74FDFFFF   call    00499EA8
  0049A134  |.  8D55 EC       lea     edx, dword ptr [ebp-14]
  0049A137  |.  B9 04000000   mov     ecx, 4
  0049A13C  |.  8BC6          mov     eax, esi
  0049A13E  |.  E8 A9FDFFFF   call    00499EEC
  0049A143  |>  8D55 AD       lea     edx, dword ptr [ebp-53]
  0049A146  |.  B9 23000000   mov     ecx, 23
  0049A14B  |.  8BC6          mov     eax, esi
  0049A14D  |.  E8 9AFDFFFF   call    00499EEC                   ;  文件是否小于 23H
  0049A152  |.  83F8 23       cmp     eax, 23
  0049A155  |.  0F9CC3        setl    bl
  0049A158  |.  84DB          test    bl, bl
  0049A15A  |.  75 0A         jnz     short 0049A166
  0049A15C  |.  817D AD 09047 cmp     dword ptr [ebp-53], 19750409
  0049A163  |.  0F95C3        setne   bl
  0049A166  |>  8B45 B3       mov     eax, dword ptr [ebp-4D]      ;  文件偏移 7h 处
  0049A169  |.  8BD0          mov     edx, eax
  0049A16B  |.  83E2 04       and     edx, 4
  0049A16E  |.  83FA 04       cmp     edx, 4
  0049A171  |.  0F9445 E4     sete    byte ptr [ebp-1C]            ;  若相等则置此内存块为 1，后面会用到此内存块
  0049A175  |.  8BD0          mov     edx, eax
  0049A177  |.  83E2 20       and     edx, 20
```



```
0049A17A  |.  83FA 20        cmp     edx, 20  
0049A17D  |.  0F9445 E3     sete    byte ptr [ebp-1D]            ;  若相等则置此内存块为 1，后面会用到此内存块  
0049A181  |.  83E0 01       and     eax, 1                       ;  eax 与 4h 相与等于 4h 并且 与 20h 相与要等于 20h  
0049A184  |.  48           dec     eax  
0049A185  |.  75 22        jnz     short 0049A1A9  
0049A187  |.  8D85 73FAFFFF lea     eax, dword ptr [ebp-58D]  
0049A18D  |.  8945 DC      mov     dword ptr [ebp-24], eax  
0049A190  |.  84DB         test    bl, bl  
0049A192  |.  75 15        jnz     short 0049A1A9  
0049A194  |.  8B55 DC      mov     edx, dword ptr [ebp-24]  
0049A197  |.  B9 10000000  mov     ecx, 10  
0049A19C  |.  8BC6         mov     eax, esi  
0049A19E  |.  E8 49FDFFFF  call    00499EEC                       ;  从文件 23h 处读 10h 字节  
0049A1A3  |.  83F8 10      cmp     eax, 10  
0049A1A6  |.  0F9CC3       setl    bl  
0049A1A9  |>  84DB         test    bl, bl  
0049A1AB  |.  75 06        jnz     short 0049A1B3  
0049A1AD  |.  8A5D E3      mov     bl, byte ptr [ebp-1D]  
0049A1B0  |.  80F3 01      xor     bl, 1  
0049A1B3  |>  84DB         test    bl, bl  
0049A1B5  |.  75 13        jnz     short 0049A1CA  
0049A1B7  |.  8D55 E6      lea     edx, dword ptr [ebp-1A]  
0049A1BA  |.  B9 01000000  mov     ecx, 1  
0049A1BF  |.  8BC6         mov     eax, esi  
0049A1C1  |.  E8 26FDFFFF  call    00499EEC                       ;  文件偏移 33h 处读 1 字节  
0049A1C6  |.  48           dec     eax  
0049A1C7  |.  0F9CC3       setl    bl  
0049A1CA  |>  84DB         test    bl, bl  
0049A1CC  |.  75 1C        jnz     short 0049A1EA  
0049A1CE  |.  33C9         xor     ecx, ecx  
0049A1D0  |.  8A4D E6      mov     cl, byte ptr [ebp-1A]           ;  要读取的字节数  
0049A1D3  |.  8D95 93FCFFFF lea     edx, dword ptr [ebp-36D]  
0049A1D9  |.  8BC6         mov     eax, esi  
0049A1DB  |.  E8 0CFDFFFF  call    00499EEC  
0049A1E0  |.  33D2         xor     edx, edx  
0049A1E2  |.  8A55 E6      mov     dl, byte ptr [ebp-1A]  
0049A1E5  |.  3BC2         cmp     eax, edx  
0049A1E7  |.  0F9CC3       setl    bl  
0049A1EA  |>  84DB         test    bl, bl  
0049A1EC  |.  0F85 64020000 jnz     0049A456  
0049A1F2  |.  8D55 93      lea     edx, dword ptr [ebp-6D]  
0049A1F5  |.  B9 1A000000  mov     ecx, 1A  
0049A1FA  |.  8BC6         mov     eax, esi  
0049A1FC  |.  E8 EBFCFFFF  call    00499EEC  
0049A201  |.  83F8 1A      cmp     eax, 1A  
0049A204  |.  0F9CC3       setl    bl  
0049A207  |.  84DB         test    bl, bl  
0049A209  |.  0F85 2D020000 jnz     0049A43C  
0049A20F  |.  8B45 97      mov     eax, dword ptr [ebp-69]  
0049A212  |.  8945 F0      mov     dword ptr [ebp-10], eax  
0049A215  |.  807D E3 00   cmp     byte ptr [ebp-1D], 0  
0049A219  |.  74 1D        je      short 0049A238  
0049A21B  |.  EB 03        jmp     short 0049A220  
0049A21D  |>  FF45 F0      /inc    dword ptr [ebp-10]  
0049A220  |>  8B45 F0      mov     eax, dword ptr [ebp-10]  
0049A223  |.  25 0F000080  |and    eax, 8000000F  
0049A228  |.  79 05        |jns    short 0049A22F  
0049A22A  |.  48           |dec    eax  
0049A22B  |.  83C8 F0      |or     eax, FFFFFFF0  
0049A22E  |.  40           |inc    eax  
0049A22F  |>  85C0         |test   eax, eax  
0049A231  |.^ 75 EA        \jnz    short 0049A21D                   ;  小型循环,修改 ebp-10 内存处的值,后面要用到。  
0049A233  |.  EB 16        jmp     short 0049A24B  
0049A235  |>  FF45 F0      /inc    dword ptr [ebp-10]  
0049A238  |>  8B45 F0      mov     eax, dword ptr [ebp-10]  
0049A23B  |.  25 07000080  |and    eax, 80000007  
0049A240  |.  79 05        |jns    short 0049A247  
0049A242  |.  48           |dec    eax  
0049A243  |.  83C8 F8      |or     eax, FFFFFFF8  
0049A246  |.  40           |inc    eax  
0049A247  |>  85C0         |test   eax, eax  
0049A249  |.^ 75 EA        \jnz    short 0049A235  
0049A24B  |>  84DB         test    bl, bl  
0049A24D  |.  75 13        jnz     short 0049A262  
0049A24F  |.  8D55 E6      lea     edx, dword ptr [ebp-1A]  
0049A252  |.  B9 01000000  mov     ecx, 1  
0049A257  |.  8BC6         mov     eax, esi  
0049A259  |.  E8 8EFCFFFF  call    00499EEC                       ;  又读 1 字节  
0049A25E  |.  48           dec     eax  
0049A25F  |.  0F9CC3       setl    bl  
0049A262  |>  84DB         test    bl, bl  
0049A264  |.  0F85 82000000 jnz     0049A2EC  
0049A26A  |.  807D E4 00   cmp     byte ptr [ebp-1C], 0  
0049A26E  |.  75 20        jnz     short 0049A290  
0049A270  |.  33C9         xor     ecx, ecx  
0049A272  |.  8A4D E6      mov     cl, byte ptr [ebp-1A]  
0049A275  |.  41           inc     ecx  
0049A276  |.  8D95 93FDFFFF lea     edx, dword ptr [ebp-26D]  
0049A27C  |.  8BC6         mov     eax, esi  
0049A27E  |.  E8 69FCFFFF  call    00499EEC  
0049A283  |.  33D2         xor     edx, edx  
0049A285  |.  8A55 E6      mov     dl, byte ptr [ebp-1A]  
0049A288  |.  42           inc     edx  
0049A289  |.  3BC2         cmp     eax, edx  
0049A28B  |.  0F9CC3       setl    bl  
0049A28E  |.  EB 5C        jmp     short 0049A2EC  
0049A290  |>  33C0         xor     eax, eax  
0049A292  |.  8A45 E6      mov     al, byte ptr [ebp-1A]           ;  将上次读取的 1 字节，经过一系列运算后，作为下次将读取的字节数  
0049A295  |.  83C0 0F      add     eax, 0F  
0049A298  |.  C1E8 04      shr     eax, 4  
0049A29B  |.  8BC8         mov     ecx, eax  
0049A29D  |.  C1E1 04      shl     ecx, 4  
0049A2A0  |.  41           inc     ecx  
0049A2A1  |.  8D95 93FEFFFF lea     edx, dword ptr [ebp-16D]  
0049A2A7  |.  8BC6         mov     eax, esi  
0049A2A9  |.  E8 3EFCFFFF  call    00499EEC  
0049A2AE  |.  33D2         xor     edx, edx  
0049A2B0  |.  8A55 E6      mov     dl, byte ptr [ebp-1A]  
0049A2B3  |.  83C2 0F      add     edx, 0F  
0049A2B6  |.  C1EA 04      shr     edx, 4  
0049A2B9  |.  C1E2 04      shl     edx, 4  
0049A2BC  |.  42           inc     edx  
0049A2BD  |.  3BC2         cmp     eax, edx  
0049A2BF  |.  0F9CC3       setl    bl  
0049A2C2  |.  8D85 94FDFFFF lea     eax, dword ptr [ebp-26C]  
0049A2C8  |.  50           push    eax  
0049A2C9  |.  33C9         xor     ecx, ecx  
0049A2CB  |.  8A4D E6      mov     cl, byte ptr [ebp-1A]  
0049A2CE  |.  8D95 94FEFFFF lea     edx, dword ptr [ebp-16C]  
0049A2D4  |.  8B45 E8      mov     eax, dword ptr [ebp-18]  
0049A2D7  |.  E8 CCEAFFFF  call    00498DA8  
0049A2DC  |.  33D2         xor     edx, edx  
0049A2DE  |.  8A55 E6      mov     dl, byte ptr [ebp-1A]  
0049A2E1  |.  8D85 93FDFFFF lea     eax, dword ptr [ebp-26D]  
0049A2E7  |.  E8 5488F6FF  call    00402B40  
0049A2EC  |>  84DB         test    bl, bl  
0049A2EE  |.  75 13        jnz     short 0049A303  
0049A2F0  |.  8D55 E5      lea     edx, dword ptr [ebp-1B]        ;  J  
0049A2F3  |.  B9 01000000  mov     ecx, 1  
0049A2F8  |.  8BC6         mov     eax, esi  
0049A2FA  |.  E8 EDBFFFF  call    00499EEC                       ;  再读 1 字节  
0049A2FF  |.  48           dec     eax  
0049A300  |.  0F9CC3       setl    bl  
0049A303  |>  807D E3 00   cmp     byte ptr [ebp-1D], 0  
0049A307  |.  74 13        je      short 0049A31C  
0049A309  |.  33C0         xor     eax, eax  
```



```text
0049A30B  |.  8A45 E5       mov     al, byte ptr [ebp-1B]  
0049A30E  |.  83C0 0F       add     eax, 0F  
0049A311  |.  C1E8 04       shr     eax, 4  
0049A314  |.  C1E0 04       shl     eax, 4  
0049A317  |.  8845 E7       mov     byte ptr [ebp-19], al  
0049A31A  |.  EB 11         jmp     short 0049A32D  
0049A31C  |>  33C0          xor     eax, eax  
0049A31E  |.  8A45 E5       mov     al, byte ptr [ebp-1B]  
0049A321  |.  83C0 07       add     eax, 7  
0049A324  |.  C1E8 03       shr     eax, 3  
0049A327  |.  C1E0 03       shl     eax, 3  
0049A32A  |.  8845 E7       mov     byte ptr [ebp-19], al  
0049A32D  |>  84DB          test    bl, bl  
0049A32F  |.  75 1C         jnz     short 0049A34D  
0049A331  |.  33DB          xor     ebx, ebx  
0049A333  |.  8A5D E7       mov     bl, byte ptr [ebp-19]            ;  对这次读取的字节做一系运算  
0049A336  |.  43            inc     ebx  
0049A337  |.  8BCB          mov     ecx, ebx  
0049A339  |.  8D95 93FAFFFF lea     edx, dword ptr [ebp-56D]  
0049A33F  |.  8BC6          mov     eax, esi  
0049A341  |.  E8 A6FBFFFF   call    00499EEC  
0049A346  |.  3BC3          cmp     eax, ebx  
0049A348  |.  0F9CC0        setl    al  
0049A34B  |.  8BD8          mov     ebx, eax  
0049A34D  |>  84DB          test    bl, bl  
0049A34F  |.  0F85 91000000 jnz     0049A3E6  
0049A355  |.  EB 79         jmp     short 0049A3D0  
0049A357  |>  8D55 E6       /lea    edx, dword ptr [ebp-1A]  
0049A35A  |.  B9 01000000   |mov    ecx, 1  
0049A35F  |.  8BC6          |mov    eax, esi  
0049A361  |.  E8 86FBFFFF   |call   00499EEC  
0049A366  |.  48            |dec    eax  
0049A367  |.  0F9CC3        |setl   bl  
0049A36A  |.  84DB          |test   bl, bl  
0049A36C  |.  75 78         |jnz    short 0049A3E6  
0049A36E  |.  807D E6 00    |cmp    byte ptr [ebp-1A], 0  
0049A372  |.  74 72         |je     short 0049A3E6  
0049A374  |.  807D E4 00    |cmp    byte ptr [ebp-1C], 0  
0049A378  |.  75 20         |jnz    short 0049A39A  
0049A37A  |.  33C9          |xor    ecx, ecx  
0049A37C  |.  8A4D E6       |mov    cl, byte ptr [ebp-1A]  
0049A37F  |.  41            |inc    ecx  
0049A380  |.  8D95 93FBFFFF |lea    edx, dword ptr [ebp-46D]  
0049A386  |.  8BC6          |mov    eax, esi  
0049A388  |.  E8 5FFBFFFF   |call   00499EEC  
0049A38D  |.  33D2          |xor    edx, edx  
0049A38F  |.  8A55 E6       |mov    dl, byte ptr [ebp-1A]  
0049A392  |.  42            |inc    edx  
0049A393  |.  3BC2          |cmp    eax, edx  
0049A395  |.  0F9CC3        |setl   bl  
0049A398  |.  EB 32         |jmp    short 0049A3CC  
0049A39A  |>  33C0          |xor    eax, eax  
0049A39C  |.  8A45 E6       |mov    al, byte ptr [ebp-1A]  
0049A39F  |.  83C0 0F       |add    eax, 0F  
0049A3A2  |.  C1E8 04       |shr    eax, 4  
0049A3A5  |.  8BC8          |mov    ecx, eax  
0049A3A7  |.  C1E1 04       |shl    ecx, 4  
0049A3AA  |.  41            |inc    ecx  
0049A3AB  |.  8D95 93FEFFFF |lea    edx, dword ptr [ebp-16D]  
0049A3B1  |.  8BC6          |mov    eax, esi  
0049A3B3  |.  E8 34FBFFFF   |call   00499EEC  
0049A3B8  |.  33D2          |xor    edx, edx  
0049A3BA  |.  8A55 E6       |mov    dl, byte ptr [ebp-1A]  
0049A3BD  |.  83C2 0F       |add    edx, 0F  
0049A3C0  |.  C1EA 04       |shr    edx, 4  
0049A3C3  |.  C1E2 04       |shl    edx, 4  
0049A3C6  |.  42            |inc    edx  
0049A3C7  |.  3BC2          |cmp    eax, edx  
0049A3C9  |.  0F9CC3        |setl   bl  
0049A3CC  |>  84DB          |test   bl, bl  
0049A3CE  |.  75 16         |jnz    short 0049A3E6  
0049A3D0  |>  B9 01000000    mov    ecx, 1  
0049A3D5  |.  33D2          |xor    edx, edx  
0049A3D7  |.  8BC6          |mov    eax, esi  
0049A3D9  |.  E8 CAF9FFFF   |call   00499EA8                             ;  文件总长度  
0049A3DE  |.  3BF8          |cmp    edi, eax  
0049A3E0  |.^ 0F8F 71FFFFFF \jg     0049A357  
0049A3E6  |>  84DB          test    bl, bl  
0049A3E8  |.  75 18         jnz     short 0049A402  
0049A3EA  |.  B9 01000000   mov     ecx, 1  
0049A3EF  |.  33D2          xor     edx, edx  
0049A3F1  |.  8BC6          mov     eax, esi  
0049A3F3  |.  E8 B0FAFFFF   call    00499EA8  
0049A3F8  |.  8BD7          mov     edx, edi  
0049A3FA  |.  2BD0          sub     edx, eax  
0049A3FC  |.  3B55 F0       cmp     edx, dword ptr [ebp-10]  
0049A3FF  |.  0F9CC3        setl    bl  
0049A402  |>  84DB          test    bl, bl  
0049A404  |.  75 36         jnz     short 0049A43C  
0049A406  |.  8B45 F0       mov     eax, dword ptr [ebp-10]  
0049A409  |.  05 FF1F0000   add     eax, 1FFF  
0049A40E  |.  25 00E0FFFF   and     eax, FFFFE000  
0049A413  |.  8945 D8       mov     dword ptr [ebp-28], eax  
0049A416  |.  A1 98784B00   mov     eax, dword ptr [4B7898]  
0049A41B  |.  0FB700        movzx   eax, word ptr [eax]  
0049A41E  |.  8B55 D8       mov     edx, dword ptr [ebp-28]  
0049A421  |.  E8 DED0F6FF   call    00407504  
0049A426  |.  8945 D4       mov     dword ptr [ebp-2C], eax  
0049A429  |.  8B55 D4       mov     edx, dword ptr [ebp-2C]  
0049A42C  |.  8B4D F0       mov     ecx, dword ptr [ebp-10]  
0049A42F  |.  8BC6          mov     eax, esi  
0049A431  |.  E8 B6FAFFFF   call    00499EEC  
0049A436  |.  3B45 F0       cmp     eax, dword ptr [ebp-10]  
0049A439  |.  0F9CC3        setl    bl  
0049A43C  |>  84DB          test    bl, bl  
0049A43E  |.  75 16         jnz     short 0049A456  
0049A440  |.  B9 01000000   mov     ecx, 1  
0049A445  |.  33D2          xor     edx, edx  
0049A447  |.  8BC6          mov     eax, esi  
0049A449  |.  E8 5AFAFFFF   call    00499EA8  
0049A44E  |.  83EF 08       sub     edi, 8  
0049A451  |.  3BC7          cmp     eax, edi  
0049A453  |.  0F9CC3        setl    bl  
0049A456  |>  8BC6          mov     eax, esi  
0049A458  |.  E8 CBF9FFFF   call    00499E28  
0049A45D  |.  84DB          test    bl, bl  
0049A45F  |.  75 73         jnz     short 0049A4D4  
0049A461  |.  8B45 D4       mov     eax, dword ptr [ebp-2C]  
0049A464  |.  50            push    eax  
0049A465  |.  8B4D 97       mov     ecx, dword ptr [ebp-69]  
0049A468  |.  8B55 D4       mov     edx, dword ptr [ebp-2C]  
0049A46B  |.  8B45 E8       mov     eax, dword ptr [ebp-18]  
0049A46E  |.  E8 25EAFFFF   call    00498E98  
0049A473  |.  8D8D 83FAFFFF lea     ecx, dword ptr [ebp-57D]  
0049A479  |.  8B55 97       mov     edx, dword ptr [ebp-69]  
0049A47C  |.  8B45 D4       mov     eax, dword ptr [ebp-2C]  
0049A47F  |.  E8 28FBFFFF   call    00499FAC                             ;  生成一字串数据 D4 1D 8C D9 8F 00 B2 04 E9 80 09 98 EC F8 42 7E  
0049A484  |.  BE 10000000   mov     esi, 10  
0049A489  |.  8D85 83FAFFFF lea     eax, dword ptr [ebp-57D]  
0049A48F  |.  8D95 73FAFFFF lea     edx, dword ptr [ebp-58D]  
0049A495  |>  8A08          /mov    cl, byte ptr [eax]  
0049A497  |.  3A0A          |cmp    cl, byte ptr [edx]                   ;  与一串字符进行比较  
0049A499  |.  74 02         |je     short 0049A49D  
0049A49B  |.  B3 01         |mov    bl, 1  
0049A49D  |>  42            |inc    edx  
0049A49E  |.  40            |inc    eax  
0049A49F  |.  4E            |dec    esi  
0049A4A0  |.^ 75 F3         \jnz    short 0049A495  
0049A4A2  |.  84DB          test    bl, bl
```



```
0049A4A4  |.  75 2E          jnz     short 0049A4D4
0049A4A6  |.  8D45 D0       lea     eax, dword ptr [ebp-30]
0049A4A9  |.  8B55 D4       mov     edx, dword ptr [ebp-2C]
0049A4AC  |.  E8 AFA2F6FF   call    00404760
0049A4B1  |.  8D45 D0       lea     eax, dword ptr [ebp-30]
0049A4B4  |.  8B55 97       mov     edx, dword ptr [ebp-69]
0049A4B7  |.  E8 F8A6F6FF   call    00404BB4
0049A4BC  |.  837D 08 00    cmp     dword ptr [ebp+8], 0
0049A4C0  |.  74 12         je      short 0049A4D4
0049A4C2  |.  8B45 D0       mov     eax, dword ptr [ebp-30]
0049A4C5  |.  E8 5EA5F6FF   call    00404A28
0049A4CA  |.  8BD0          mov     edx, eax
0049A4CC  |.  8B45 08       mov     eax, dword ptr [ebp+8]
0049A4CF  |.  8B08          mov     ecx, dword ptr [eax]
0049A4D1  |.  FF51 7C       call    dword ptr [ecx+7C]
0049A4D4  |>  837D D8 00    cmp     dword ptr [ebp-28], 0
0049A4D8  |.  74 08         je      short 0049A4E2
0049A4DA  |.  8B45 D4       mov     eax, dword ptr [ebp-2C]
0049A4DD  |.  E8 4ED0F6FF   call    00407530
0049A4E2  |>  80F3 01       xor     bl, 1
0049A4E5  |>  33C0          xor     eax, eax
0049A4E7  |.  5A            pop     edx
0049A4E8  |.  59            pop     ecx
0049A4E9  |.  59            pop     ecx
0049A4EA  |.  64:8910       mov     dword ptr fs:[eax], edx
0049A4ED  |.  68 17A54900   push    0049A517
0049A4F2  |>  8D45 D0       lea     eax, dword ptr [ebp-30]
0049A4F5  |.  E8 6EA0F6FF   call    00404568
0049A4FA  |.  8D45 E8       lea     eax, dword ptr [ebp-18]
0049A4FD  |.  E8 66A0F6FF   call    00404568
0049A502  |.  8D45 F8       lea     eax, dword ptr [ebp-8]
0049A505  |.  BA 02000000   mov     edx, 2
0049A50A  |.  E8 7DA0F6FF   call    0040458C
0049A50F  \.  C3            retn
0049A510    .^ E9 379AF6FF  jmp     00403F4C
0049A515    .^ EB DB        jmp     short 0049A4F2
0049A517    .  8BC3         mov     eax, ebx
0049A519    .  5F          pop     edi
0049A51A    .  5E          pop     esi
0049A51B    .  5B          pop     ebx
0049A51C    .  8BE5        mov     esp, ebp
0049A51E    .  5D          pop     ebp
0049A51F    .  C2 0400     retn    4
0049A522      8BC0         mov     eax, eax
0049A524  >  .  E8 375AFDFF  call    0046FF60                     ;  ->Forms.TCustomForm.Close(TCustomForm);
0049A529    .  C3          retn
```

---

### 【经验总结】

*   用户名，注册码均不能为空
*   跟踪程序时并未发现机器码参与运算
*   首先判断安装目录下是否存在`"scene\\英语 900 句.xml"`
*   从文件`0h`偏移处读取`4`字节与`19750409`相比较，这个可能是软件作者的生日，呵！！
*   第`5`个和第`6`个字节任意
*   从`6h`处读`4`个字节，与`4h`相与要等于`4h`并且与`20h`相与要等于`20h`
*   从`0ah`处放任意`25`个字节
*   `23h`处放`D4 1D 8C D9 8F 00 B2 04 E9 80 09 98 EC F8 42 7E`
*   `33h`处的`1`个字节表示长度，后面紧跟此长度的任意数据
*   `39h`处的`4`个字节任意
*   `3dh`处的`4`个字节循环(自加`1`，然后与`8000000F`相与)循环结束时，这`4`字节的值要等于`0`
*   `41h`处`12h`个任意数据
*   `53h`处`1`个字节任意，
*   `54h`处把`53h`偏移的`1`个字节加`0Fh`然后右移`4`位，再左移动`4`位再加`1`得出一个数字，`54h`处就存放此长度的数据
*   紧接着存放任意`1`字节，设为`A`
*   对`A`加上`7`，然后右移`3`位，再左移`3`位，再自加`1`，得出一个值设为`B`
*   接着后面存放`B`长度的字节,内容任意

我伪造的`"英语 900 句.xml"`数据如下`16`进制格式:

```
0904751906072500000009090909090909090909090909090909090909090909090909D41D8CD98F00B204E9800998ECF8427E050C0C
0C0C0C07070707000000000707070707070707070707070707070707070D0E0E0E0E0E0E0E0E0E0E0E0E0E0E0E0E0E0F010101010101
0101010101010101010101
```

---

### 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 04 月 05 日 18:28:00![](img/4ec4a010bb6e6d79731981cd94fbb640.jpg "尴尬")
4dd466d0e1e0cc123afbe504da2462ca
