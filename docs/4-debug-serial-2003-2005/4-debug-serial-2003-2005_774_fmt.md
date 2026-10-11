
# PDF2Word 1.6 注册算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-378.htm`](https://www.pediy.com/kssd/pediy07/pediy7-378.htm)

【破文标题】：PDF2Word(pdf to word) 1.6 注册算法分析  
【破文作者】：KuNgBiM\[DFCG\]  
【作者邮箱】：gb\_1227@163.com  
【软件名称】：PDF2Word(pdf to word) 1.6  
【软件大小】：628KB  
【软件语言】：英文  
【软件类别】：国外软件/共享版/编辑软件  
【整理时间】：2005-09-07  
【开 发 商】：[`www.globalpdf.com/pdf2word/`](http://www.globalpdf.com/pdf2word/)  
【下载地址】：[`nj.onlinedown.net/soft/23655.htm`](http://nj.onlinedown.net/soft/23655.htm)  
【软件简介】：PDF2Word(pdf to word)让你可以把文本，图片以及其他内容从 pdf 文档中输出到 word 文档中，所以你能够重新编辑 pdf 内容，PDF2Word 将保存文本，版面和图片到生成的 word 文档中。PDF2Word(pdf to word)是一个独立的软件，所以你不需要安装 Microsoft Word，Adobe Acrobat，甚至 Acrobat Reader 来运行它。  
【保护方式】：注册码 + 启动 NAG + 100 次数使用限制  
【编译语言】：Microsoft Visual C++ 6.0  
【调试环境】：WinXP、PEiD、Ollydbg  
【破解日期】：2005-09-08  
【破解目的】：研究算法分析  
【作者声明】：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

—————————————————————————————————

### 破解过程

**侦测**：用 `PEiD` 查壳，无壳，`Microsoft Visual C++ 6.0` 编译。

**试探**：运行主程序注册，输入试炼码，确认！程序提示："Series number error, please check it and try again."

**下药**：`Ollydbg` 载入主程序，命令下断：`bpx MessageBoxA`，回车，`F9` 运行，输入试炼信息：

\*\*\*\*\*\*\*\*\* 试炼信息 \*\*\*\*\*\*\*\*\*\*

Email：gb\_1227@163.com

Registration Key：9876543210

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```assembly
00429EEB    FF15 60974400   call dword ptr ds:[<&USER32.EndDialog>]         ; USER32.EndDialog
00429EF1    E9 69020000     jmp pdf2rtf.0042A15F
00429EF6    8BB424 E8000000 mov esi,dword ptr ss:[esp+E8]
00429EFD    68 C8000000     push 0C8
00429F02    B9 32000000     mov ecx,32
00429F07    33C0            xor eax,eax
00429F09    BF ECB64600     mov edi,pdf2rtf.0046B6EC                     ; ASCII "9876543210"
00429F0E    68 ECB64600     push pdf2rtf.0046B6EC                       ; ASCII "9876543210"
00429F13    68 FB030000     push 3FB
00429F18    56              push esi
00429F19    F3:AB           rep stos dword ptr es:[edi]
00429F1B    FF15 5C974400   call dword ptr ds:[<&USER32.GetDlgItemTextA>] ; USER32.GetDlgItemTextA
00429F21    68 ECB64600     push pdf2rtf.0046B6EC                       ; ASCII "9876543210"
00429F26    E8 F5F7FFFF     call pdf2rtf.00429720                       ; ★算法 CALL，F7 跟进！★
00429F2B    83C4 04         add esp,4
00429F2E    85C0            test eax,eax
00429F30    74 39           je short pdf2rtf.00429F6B                   ; ★注册验证爆破点★
00429F32    6A 40           push 40
00429F34    68 D0824600     push pdf2rtf.004682D0                       ; ASCII "Thank you registered"
00429F39    68 A4824600     push pdf2rtf.004682A4                       ; ASCII "Thank you registered VeryPDF PDF2Word v1.6."
00429F3E    56              push esi
00429F3F    FF15 64974400   call dword ptr ds:[<&USER32.MessageBoxA>]     ; USER32.MessageBoxA
00429F45    68 ECB64600     push pdf2rtf.0046B6EC                       ; ASCII "9876543210"
00429F4A    56              push esi
00429F4B    E8 40F9FFFF     call pdf2rtf.00429890
00429F50    83C4 08         add esp,8
00429F53    C705 B4B74600 0>mov dword ptr ds:[46B7B4],1
00429F5D    6A 01           push 1
00429F5F    56              push esi
00429F60    FF15 60974400   call dword ptr ds:[<&USER32.EndDialog>]         ; USER32.EndDialog
00429F66    E9 F4010000     jmp pdf2rtf.0042A15F
00429F6B    6A 10           push 10                                       ; 试炼码小于 16 位则直接“挂”！
00429F6D    6A 00           push 0
00429F6F    68 70824600     push pdf2rtf.00468270                       ; ASCII "Series number error, please check it and try again."
00429F74    56              push esi
00429F75    FF15 64974400   call dword ptr ds:[<&USER32.MessageBoxA>]     ; 这里中断，USER32.MessageBoxA
00429F7B    68 FB030000     push 3FB
00429F80    56              push esi
00429F81    FF15 68974400   call dword ptr ds:[<&USER32.GetDlgItem>]      ; USER32.GetDlgItem
00429F87    50              push eax
00429F88    FF15 6C974400   call dword ptr ds:[<&USER32.SetFocus>]        ; USER32.SetFocus
00429F8E    E9 CC010000     jmp pdf2rtf.0042A15F
00429F93    6A 01           push 1

\========================= 跟进 00429F26    E8 F5F7FFFF     call pdf2rtf.00429720 =========
00429720    83EC 30         sub esp,30
00429723    55              push ebp
00429724    56              push esi
00429725    8B7424 3C       mov esi,dword ptr ss:[esp+3C]                 ; 取试炼码(记为：KEY)ASCII "9876543210"
00429729    57              push edi
0042972A    8A06            mov al,byte ptr ds:[esi]                      ; 取 KEY[esi]=KEY[1]
0042972C    8A4E 01         mov cl,byte ptr ds:[esi+1]                    ; 取 KEY[esi+1]=KEY[2]
0042972F    8A56 0E         mov dl,byte ptr ds:[esi+E]                    ; 取 KEY[esi+E]=KEY[15]
00429732    884424 18       mov byte ptr ss:[esp+18],al
00429736    32C0            xor al,al
00429738    884C24 30       mov byte ptr ss:[esp+30],cl
0042973C    8A4E 0F         mov cl,byte ptr ds:[esi+F]                    ; 取 KEY[esi+F]=KEY[16]
0042973F    884424 19       mov byte ptr ss:[esp+19],al                   ; al=0
00429743    884424 31       mov byte ptr ss:[esp+31],al                   ; al=0
00429747    884424 25       mov byte ptr ss:[esp+25],al                   ; al=0
0042974B    884424 0D       mov byte ptr ss:[esp+D],al                    ; al=0
0042974F    8A46 02         mov al,byte ptr ds:[esi+2]                    ; 取 KEY[esi+2]=KEY[3]
00429752    3C 24           cmp al,24                                     ; '$'
00429754    885424 24       mov byte ptr ss:[esp+24],dl                   ; dl=KEY[15]
00429758    884C24 0C       mov byte ptr ss:[esp+C],cl                    ; cl=KEY[16]
0042975C    75 52           jnz short pdf2rtf.004297B0                    ; 要求 KEY[3]='$'，故这里跳则死！
0042975E    8B3D 4C964400   mov edi,dword ptr ds:[<&MSVCRT.atoi>]         ; msvcrt.atoi
00429764    8D5424 0C       lea edx,dword ptr ss:[esp+C]
00429768    52              push edx
00429769    FFD7            call edi
0042976B    8BE8            mov ebp,eax                                    ; eax=atoi(KEY[16])
0042976D    8D4424 1C       lea eax,dword ptr ss:[esp+1C]                 ; KEY[1]
00429771    50              push eax
00429772    FFD7            call edi
00429774    03E8            add ebp,eax                                    ; atoi(KEY[16])+atoi(KEY[1])
00429776    83C4 08         add esp,8
00429779    83FD 0A         cmp ebp,0A                                    ; atoi(KEY[16])+atoi(KEY[1])=0A
0042977C    75 32           jnz short pdf2rtf.004297B0                    ; 这里也不能跳！
0042977E    8D4C24 24       lea ecx,dword ptr ss:[esp+24]                 ; KEY[15]
00429782    51              push ecx
00429783    FFD7            call edi
00429785    8D5424 34       lea edx,dword ptr ss:[esp+34]                 ; KEY[2]
00429789    8BE8            mov ebp,eax
0042978B    52              push edx
0042978C    FFD7            call edi
0042978E    03E8            add ebp,eax
00429790    83C4 08         add esp,8
00429793    83FD 0A         cmp ebp,0A                                    ; atoi(KEY[15])+atoi(KEY[2])=0A
00429796    75 18           jnz short pdf2rtf.004297B0                    ; 这里也不能跳！
00429798    807E 03 24      cmp byte ptr ds:[esi+3],24                    ; 要求 KEY[4]='$'，故这里跳则死！
0042979C    75 12           jnz short pdf2rtf.004297B0
0042979E    8A4E 05         mov cl,byte ptr ds:[esi+5]                    ; KEY[6]
004297A1    33C0            xor eax,eax
```



```
004297A3    80F9 23          cmp cl,23                     ; 要求 KEY[6]='#'

004297A6    5F              pop edi

004297A7    5E              pop esi

004297A8    5D              pop ebp

004297A9    0F94C0          sete al

004297AC    83C4 30         add esp,30

004297AF    C3              retn                          ; 返回程序
```

### 算法总结

注册验证非常简单：

1. 注册码长度要求至少为 `16` 位，并记为 `KEY`
2. 注册码要求 `KEY[3]='$'`（第 3 位必须为“$”）
3. 注册码要求 `KEY[4]='$'`（第 4 位必须为“$”）
4. 注册码要求 `KEY[6]='#'`（第 6 位必须为“#”）
5. 注册码要求 `atoi(KEY[16])+atoi(KEY[1])=0A`（第 16 位与第 1 位的和必须为 10）
6. 注册码要求 `atoi(KEY[15])+atoi(KEY[2])=0A`（第 15 位与第 2 位的和必须为 10）

`Email` 未参加计算，注册码只要符合这个规律即可注册成功。

以规律算出的一组可用注册码：`98$$5#3210123421`

### 注册信息

* `Email`：`gb_1227@163.com` （任意）
* `Registration Key`：`98$$5#3210123421`

版权所有（C）2005 `KuNgBiM[DFCG]` Copyright (C) 2005 `KuNgBiM[DFCG]`

`Cracked By` `KuNgBiM[DFCG]`
2005-09-08
01:20:30 AM

`4dd466d0e1e0cc123afbe504da2462ca`
