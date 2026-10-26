
# [原创]: 4U WMA MP3 Converter V5.9.2 汉化版算法分析

【文章标题】: 4U WMA MP3 Converter V5.9.2 汉化版算法分析
【文章作者】: foresee
【作者邮箱】: vangjian@hotmail.com
【作者主页】: \*\*\*\*\*\*
【作者QQ号】: \*\*\*\*\*\*
【软件名称】: 4U WMA MP3 Converter V5.9.2 汉化版
【软件大小】: 6580 KB
【下载地址】: 汉化版5.9.2：`6a3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4C8P5h3y4F1i4K6u0W2j5$3!0E0i4K6u0r3M7$3!0X3N6q4)9J5c8U0t1K6x3K6j5K6i4K6u0W2K9s2c8E0L8q4)9J5x3$3c8G2N6$3&6D9L8$3q4V1` 英文版5.9.3：`4efK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6E0j5g2)9J5k6r3#2H3x3#2)9J5k6r3y4G2L8Y4k6W2M7Y4c8W2M7W2)9J5k6h3&6W2N6q4)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0W2K9s2c8E0`
【加壳方式】: 无
【保护方式】: 注册号
【编写语言】: Borland Delphi 6.0 - 7.0
【使用工具】: OD
【操作平台】: Win2000
【软件介绍】: 4U WMA MP3 Converter V5.9.2 汉化版
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 【详细过程】

OD载入，查找->所有参考文本字串，找到文本字串ASCII "Invalid Registration Code! ",CR,LF,"Please enter an available Registration Code."

双击来到

```
0048DB27  |.  68 A0DB4800   push    0048DBA0                         ; |Text = "Invalid Registration Code! ",CR,LF,"Please enter an available Registration Code."
0048DB2C  |.  A1 ECEF4B00   mov     eax, dword ptr [4BEFEC]          ; |
0048DB31  |.  8B00          mov     eax, dword ptr [eax]             ; |
0048DB33  |.  8B40 30       mov     eax, dword ptr [eax+30]          ; |
0048DB36  |.  50            push    eax                              ; |hOwner
0048DB37  |.  E8 1C9BF7FF   call    <jmp.&user32.MessageBoxA>        ; \MessageBoxA
0048DB3C  |>  33C0          xor     eax, eax
0048DB3E  |.  5A            pop     edx
0048DB3F  |.  59            pop     ecx
0048DB40  |.  59            pop     ecx
0048DB41  |.  64:8910       mov     dword ptr fs:[eax], edx
0048DB44  |.  68 5EDB4800   push    0048DB5E
0048DB49  |>  8D45 EC       lea     eax, dword ptr [ebp-14]
0048DB4C  |.  BA 05000000   mov     edx, 5
0048DB51  |.  E8 AE6BF7FF   call    00404704
0048DB56  \.  C3            retn
```

往上翻翻看看，发现此段即是关键，在`0048DA60`处下断，全段代码如下

```
0048DA60  /$  55            push    ebp
0048DA61  |.  8BEC          mov     ebp, esp
0048DA63  |.  6A 00         push    0
0048DA65  |.  6A 00         push    0
0048DA67  |.  6A 00         push    0
0048DA69  |.  6A 00         push    0
0048DA6B  |.  6A 00         push    0
0048DA6D  |.  53            push    ebx
0048DA6E  |.  56            push    esi
0048DA6F  |.  894D F8       mov     dword ptr [ebp-8], ecx           ;  ecx=00DA2AB4, (ASCII "123456789")
0048DA72  |.  8955 FC       mov     dword ptr [ebp-4], edx           ;  edx=00DA2DF4, (ASCII "foresee")
0048DA75  |.  8BF0          mov     esi, eax
0048DA77  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0048DA7A  |.  E8 0171F7FF   call    00404B80
0048DA7F  |.  8B45 F8       mov     eax, dword ptr [ebp-8]
0048DA82  |.  E8 F970F7FF   call    00404B80
0048DA87  |.  33C0          xor     eax, eax
0048DA89  |.  55            push    ebp
0048DA8A  |.  68 57DB4800   push    0048DB57
0048DA8F  |.  64:FF30       push    dword ptr fs:[eax]
0048DA92  |.  64:8920       mov     dword ptr fs:[eax], esp
0048DA95  |.  33DB          xor     ebx, ebx
0048DA97  |.  33D2          xor     edx, edx
0048DA99  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0048DA9C  |.  E8 3372F7FF   call    00404CD4
0048DAA1  |.  85C0          test    eax, eax
0048DAA3  |.  7E 0B         jle     short 0048DAB0
0048DAA5  |.  8D45 F8       lea     eax, dword ptr [ebp-8]
0048DAA8  |.  8B55 FC       mov     edx, dword ptr [ebp-4]
0048DAAB  |.  E8 C86CF7FF   call    00404778
0048DAB0  |>  8D4D F4       lea     ecx, dword ptr [ebp-C]           ;  (ASCII "1234567890")
0048DAB3  |.  8B55 FC       mov     edx, dword ptr [ebp-4]           ;   (ASCII "foresee")
0048DAB6  |.  8BC6          mov     eax, esi
0048DAB8  |.  E8 2F010000   call    0048DBEC                         ;  关键call，注册号计算
0048DABD  |.  8B55 F4       mov     edx, dword ptr [ebp-C]           ;  此处存放着注册号
0048DAC0  |.  8B45 F8       mov     eax, dword ptr [ebp-8]
0048DAC3  |.  E8 DCAFF7FF   call    00408AA4                         ;
0048DAC8  |.  85C0          test    eax, eax
0048DACA  |.  75 41         jnz     short 0048DB0D                   ;  注册不正确，则跳转到注册失败窗口
0048DACC  |.  8B55 FC       mov     edx, dword ptr [ebp-4]
0048DACF  |.  8BC6          mov     eax, esi
0048DAD1  |.  E8 DAF3FFFF   call    0048CEB0
0048DAD6  |.  84C0          test    al, al
0048DAD8  |.  74 62         je      short 0048DB3C
0048DADA  |.  B3 01         mov     bl, 1
0048DADC  |.  6A 40         push    40
0048DADE  |.  8D55 F0       lea     edx, dword ptr [ebp-10]
0048DAE1  |.  A1 ECEF4B00   mov     eax, dword ptr [4BEFEC]
0048DAE6  |.  8B00          mov     eax, dword ptr [eax]
0048DAE8  |.  E8 0B97FDFF   call    004671F8
0048DAED  |.  8B45 F0       mov     eax, dword ptr [ebp-10]
0048DAF0  |.  E8 9B70F7FF   call    00404B90
0048DAF5  |.  50            push    eax                              ; |Title
0048DAF6  |.  68 68DB4800   push    0048DB68                         ; |Text = "Registered successfully, Thanks for your registration."
0048DAFB  |.  A1 ECEF4B00   mov     eax, dword ptr [4BEFEC]          ; |
0048DB00  |.  8B00          mov     eax, dword ptr [eax]             ; |
0048DB02  |.  8B40 30       mov     eax, dword ptr [eax+30]          ; |
0048DB05  |.  50            push    eax                              ; |hOwner
0048DB06  |.  E8 4D9BF7FF   call    <jmp.&user32.MessageBoxA>        ; \MessageBoxA
0048DB0B  |.  EB 2F         jmp     short 0048DB3C
0048DB0D  |>  6A 10         push    10
0048DB0F  |.  8D55 EC       lea     edx, dword ptr [ebp-14]
0048DB12  |.  A1 ECEF4B00   mov     eax, dword ptr [4BEFEC]
0048DB17  |.  8B00          mov     eax, dword ptr [eax]
0048DB19  |.  E8 DA96FDFF   call    004671F8
0048DB1E  |.  8B45 EC       mov     eax, dword ptr [ebp-14]
0048DB21  |.  E8 6A70F7FF   call    00404B90
0048DB26  |.  50            push    eax                              ; |Title
0048DB27  |.  68 A0DB4800   push    0048DBA0                         ; |Text = "Invalid Registration Code! ",CR,LF,"Please enter an available Registration Code."
0048DB2C  |.  A1 ECEF4B00   mov     eax, dword ptr [4BEFEC]          ; |
0048DB31  |.  8B00          mov     eax, dword ptr [eax]             ; |
0048DB33  |.  8B40 30       mov     eax, dword ptr [eax+30]          ; |
0048DB36  |.  50            push    eax                              ; |hOwner
0048DB37  |.  E8 1C9BF7FF   call    <jmp.&user32.MessageBoxA>        ; \MessageBoxA
0048DB3C  |>  33C0          xor     eax, eax
0048DB3E  |.  5A            pop     edx
0048DB3F  |.  59            pop     ecx
0048DB40  |.  59            pop     ecx
0048DB41  |.  64:8910       mov     dword ptr fs:[eax], edx
0048DB44  |.  68 5EDB4800   push    0048DB5E
0048DB49  |>  8D45 EC       lea     eax, dword ptr [ebp-14]
0048DB4C  |.  BA 05000000   mov     edx, 5
0048DB51  |.  E8 AE6BF7FF   call    00404704
0048DB56  \.  C3            retn
0048DB57   .^ E9 0865F7FF   jmp     00404064
0048DB5C   .^ EB EB         jmp     short 0048DB49
0048DB5E   .  8BC3          mov     eax, ebx
0048DB60   .  5E            pop     esi
0048DB61   .  5B            pop     ebx
0048DB62   .  8BE5          mov     esp, ebp
0048DB64   .  5D            pop     ebp
0048DB65   .  C3            retn
```

`0048DBEC`处是关键`call`，跟进

此处待补充

--------------------------------------------------------------------------------

## 【算法总结】

将用户输入的用户名和固定字符串`"Jt^S0Mvx5C1"`连接，之后生成15位注册码，每五位用`-`号连接

一组可用的注册码

用户名：`foresee`
注册码：`10027-4F3CB-849AE`

注册表变化，未注册前

```
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\WMAConvert Options]

"ConnectionOption"="NHMZCN"
"ConnectionDate"=hex:43,bb,58,95,ff,26,e3,40
"LastDate"=hex:43,bb,58,95,ff,26,e3,40
```

注册以后

```
Windows Registry Editor Version 5.00
```
