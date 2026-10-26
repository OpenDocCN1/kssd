
# 谈EBOOK文章类软件的脱壳

## 一、简介

【郑重声明】：本文只为技术而破解，纯属技术交流, 请勿用于非法用途。

【软件名称】：国产的东东，就不说了

【下载页面】：`311K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6K6q4Q4x3X3g2K6K9%4W2U0L8W2)9J5k6h3y4G2L8g2)9J5c8Y4y4G2k6Y4c8Q4x3V1j5J5x3o6l9#2z5g2)9J5k6h3S2@1L8h3H3`.

【软件简介】：EBOOK小说系列

【调试环境】：Win2000 SP4、`ODbyDYK`、`HIEW7.10`、`PEID0.93`

【软件限制】：共享软件，10次使用限制，无功能限制

## 1. 脱壳

用`PEID0.93`查看主程序，壳为`UPX`壳，`UPX 0.89.6 - 1.02 / 1.05 - 1.24 (Delphi) stub -> Markus & Laszlo [Overlay]`

`Ollydbg`载入主程序：

```
004C9A40 古> $  60             pushad; 载入程序后停在这里，F8一次
004C9A41    .  BE 00504800    mov esi,古龙.00485000 ; 到这里，这时查看寄存器窗口
004C9A46    .  8DBE 00C0F7FF  lea edi,dword ptr ds:[esi+F>
004C9A4C    .  C787 9CC00900 >mov dword ptr ds:[edi+9C09C>
004C9A56    .  57             push edi
004C9A57    .  83CD FF        or ebp,FFFFFFFF
004C9A5A    .  EB 0E          jmp short 古龙.004C9A6A
\\\\\\\\\\\\\寄存器\\\\\\\\\\\\\\\\
EAX 00000000
ECX 01010101
EDX FFFFFFFF
EBX 7FFDF000
ESP 0012FFA4
EBP 0012FFF0
ESI 00000000
EDI 00000000
EIP 004C9A41 古龙.004C9A41
\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\
```

根据`ESP`定律规则，现在在命令栏中下`hr 0012ffa4`命令，回车，`F9`运行：

```
004C9BAB     ^\E9 AC30FDFF    jmp 古龙.0049CC5C; 这里断下这里0049CC5C所指的就是OEP
004C9BB0       C89B4C00       dd 古龙.004C9BC8
004C9BB4       D89B4C00       dd 古龙.004C9BD8
0049CC5C   /> /55             push ebp;在这儿用LordPE完全DUMP这个进程
0049CC5D   |. |8BEC           mov ebp,esp
0049CC5F   |. |83C4 EC        add esp,-14
0049CC62   |. |33C0           xor eax,eax
0049CC64   |. |8945 EC        mov dword ptr ss:[ebp-14],e>
0049CC67   |. |B8 BCC84900    mov eax,古龙.0049C8BC
```

### 脱壳修复

运行`ImportREC 1.6`，选择这个进程，把`OEP`改为`0009CC5C`，点`IT AutoSearch`，指针全部有效。`FixDump`！

发现程序挂接的数据段已经不见了，23M大小的程序变成了800多K，运行脱壳后的文件会有提示："This eBook has a wrong signature. The application will be terminated.Signature:0 "

```
0049A967   |.  FF12           call dword ptr ds:[edx]; 建一TFileStream，返回文件大小?
0049A969   |.  8BD0           mov edx,eax
0049A96B   |.  83EA 10        sub edx,10
0049A96E   |.  33C9           xor ecx,ecx
0049A970   |.  8BC3           mov eax,ebx
0049A972   |.  8B38           mov edi,dword ptr ds:[eax]
0049A974   |.  FF57 14        call dword ptr ds:[edi+14]; 文件指针 指向文件大小16字节
0049A977   |.  8D55 F0        lea edx,dword ptr ss:[ebp-10]
0049A97A   |.  B9 10000000    mov ecx,10
0049A97F   |.  8BC3           mov eax,ebx
0049A981   |.  8B38           mov edi,dword ptr ds:[eax]
0049A983   |.  FF57 0C        call dword ptr ds:[edi+C]
0049A986   |.  8B7D FC        mov edi,dword ptr ss:[ebp-4]
0049A989   |.  81FF 97130000  cmp edi,1397 ; 文件最后一个DWORD应该为0x1397
0049A98F   |.  74 40          je short 古龙.0049A9D1; 必须跳哦
0049A991   |.  6A 00          push 0
0049A993   |.  8D4D E8        lea ecx,dword ptr ss:[ebp-18]
0049A996   |.  33D2           xor edx,edx
0049A998   |.  8BC7           mov eax,edi
0049A99A   |.  E8 91E0F6FF    call 古龙.00408A30
0049A99F   |.  8B4D E8        mov ecx,dword ptr ss:[ebp-18]
0049A9A2   |.  8D45 EC        lea eax,dword ptr ss:[ebp-14]; 显示错误信息
0049A9A5   |.  BA 90AA4900    mov edx,古龙.0049AA90             ;  ASCII "This eBook has a wrong
signature. The application will be terminated.
Signature: "
0049A9AA   |.  E8 D59AF6FF    call 古龙.00404484
0049A9D1   |> \8BC3           mov eax,ebx
0049A9D3   |.  8B10           mov edx,dword ptr ds:[eax]
0049A9D5   |.  FF12           call dword ptr ds:[edx]         ;  古龙.0041C39C
0049A9D7   |.  8BF8           mov edi,eax   ; 返回文件大小
0049A9D9   |.  2B7D F0        sub edi,dword ptr ss:[ebp-10]; 文件尾16字节内容：016D0577
即为附加文件大小
0049A9DC   |.  89BE 44030000  mov dword ptr ds:[esi+344],edi
0049A9E2   |.  33C9           xor ecx,ecx
0049A9E4   |.  8BD7           mov edx,edi
0049A9E6   |.  8BC3           mov eax,ebx
0049A9E8   |.  8B38           mov edi,dword ptr ds:[eax]
0049A9EA   |.  FF57 14        call dword ptr ds:[edi+14]; SetFilePointer文件指针 指向取附加文件尾
向上016D0577处
0049A9ED   |.  8BD3           mov edx,ebx
0049A9EF   |.  8BC6           mov eax,esi
0049A9F1   |.  E8 6E030000    call 古龙.0049AD64; 读取并处理电子书内容
```

是不是一目了然啊。

文件尾最后一个`DWORD`是校验信息，倒数第四个`DWORD`是附加的文件大小。

于是用`WinHex`打开原程序，`Alt+G`来到文件尾向上偏移`016D0577`的地方。从这里复制到文件尾再`insert`到脱壳后的文件尾，保存，运行一下，正常了。

下面就是你的工作了

以上参考了《图章制作系统 V3.63》脱壳去校验解除自杀代码[分析篇]，`UPX`壳的`PATCH`------谈EBOOK文章类软件的破解，看雪论坛精华6中电子书程序的中华奇书三篇文章，说难听一点是抄袭，让大家见笑了。

[\[招生\]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
