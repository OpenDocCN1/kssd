
# 手机号码助手 1.0 算法分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-100.htm`](https://www.pediy.com/kssd/pediy09/pediy09-100.htm)

> 【文章标题】: 手机号码助手 1.0 算法分析  
> 【文章作者】: `KuNgBiM`  
> 【作者邮箱】: `kungbim@163.com`  
> 【作者主页】: `http://www.crkcn.com`  
> 【软件名称】: 手机号码助手 1.0  
> 【软件大小】: `1190KB`  
> 【下载地址】: 自己搜索下载  
> 【加壳方式】: `N/A`  
> 【保护方式】: 序列号+重启验证  
> 【编写语言】: `Borland Delphi 6.0 - 7.0`  
> 【使用工具】: `OD`  
> 【操作平台】: 盗版非标准 `XPsp2`  
> 【软件介绍】: 手机号码助手软件是专为大规模的短信群发所需号码段生成，群发日志生成而设计的号码生成整理与日志工具。通过本软件，您可以在浩如烟海的手机号码中轻易挑选出发送价值高的号码，同时短信群发过程中产生的日志可以轻松生成，应对企业客户。从而节省您的时间和金钱，使您的短信广告收效更明显。  
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 【详细过程】

这个程序比较好玩，网上有 `MemKeygen` 下载，但不爽，老是被我杀软给 `Kill` 了，于是我算法分析一下。

### 本机合法注册信息：

- 机器码：`E19YNZVE`
- 注册码：`6610842--1625409035-4470987`

### 汇编代码

```
005654B8   .  55              push    ebp                          ;  注册部分
005654B9   .  8BEC            mov     ebp, esp
005654BB   .  83C4 D8         add     esp, -28
005654BE   .  53              push    ebx
005654BF   .  33C9            xor     ecx, ecx
005654C1   .  894D E0         mov     dword ptr [ebp-20], ecx
005654C4   .  894D E4         mov     dword ptr [ebp-1C], ecx
005654C7   .  894D E8         mov     dword ptr [ebp-18], ecx
005654CA   .  894D F0         mov     dword ptr [ebp-10], ecx
005654CD   .  894D EC        mov     dword ptr [ebp-14], ecx
005654D0   .  8BD8            mov     ebx, eax
005654D2   .  33C0            xor     eax, eax
005654D4   .  55              push    ebp
005654D5   .  68 F2565600     push    005656F2
005654DA   .  64:FF30        push    dword ptr fs:[eax]
005654DD   .  64:8920        mov     dword ptr fs:[eax], esp
005654E0   .  33C9            xor     ecx, ecx
005654E2   .  B2 01           mov     dl, 1
005654E4   .  A1 20355500     mov     eax, dword ptr [553520]
005654E9   .  E8 7A4DF0FF     call    0046A268
005654EE   .  8945 F8         mov     dword ptr [ebp-8], eax
005654F1   .  33C0            xor     eax, eax
005654F3   .  55              push    ebp
005654F4   .  68 C8565600     push    005656C8
005654F9   .  64:FF30        push    dword ptr fs:[eax]
005654FC   .  64:8920        mov     dword ptr fs:[eax], esp
005654FF   .  8B45 F8         mov     eax, dword ptr [ebp-8]
00565502   .  8B10            mov     edx, dword ptr [eax]
00565504   .  FF92 EC000000   call    dword ptr [edx+EC]
0056550A   .  48              dec     eax
0056550B   .  0F85 A1010000   jnz     005656B2
00565511   .  8D55 F0         lea     edx, dword ptr [ebp-10]
00565514   .  8B45 F8         mov     eax, dword ptr [ebp-8]
00565517   .  8B80 04030000  mov     eax, dword ptr [eax+304]
0056551D   .  E8 46BFEEFF     call    00451468
00565522   .  8B45 F0         mov     eax, dword ptr [ebp-10]
00565525   .  50              push    eax
00565526   .  8D55 EC         lea     edx, dword ptr [ebp-14]
00565529   .  8B45 F8         mov     eax, dword ptr [ebp-8]
0056552C   .  8B80 FC020000  mov     eax, dword ptr [eax+2FC]
00565532   .  E8 31BFEEFF     call    00451468
00565537   .  8B4D EC         mov     ecx, dword ptr [ebp-14]
0056553A   .  8B83 60030000   mov     eax, dword ptr [ebx+360]
00565540   .  BA 83850C00     mov     edx, 0C8583                    ;  关键码，DEC(0xC8583)=820611，作者生日？
00565545   .  E8 124CFAFF     call    0050A15C                       ;  ★注册验证，跟进！★
0056554A   .  84C0            test    al, al
0056554C   .  0F84 4D010000   je      0056569F                       ;  注册码不对，挂！
00565552   .  B8 00020000     mov     eax, 200
00565557   .  E8 7839EAFF     call    00408ED4
0056555C   .  8945 FC         mov     dword ptr [ebp-4], eax
0056555F   .  8B45 FC         mov     eax, dword ptr [ebp-4]
00565562   .  33C9            xor     ecx, ecx
00565564   .  BA 00020000     mov     edx, 200
00565569   .  E8 1ADCE9FF     call    00403188
0056556E   .  33C0            xor     eax, eax
00565570   .  55              push    ebp
00565571   .  68 98565600     push    00565698
00565576   .  64:FF30        push    dword ptr fs:[eax]
00565579   .  64:8920        mov     dword ptr fs:[eax], esp
0056557C   .  8B4D FC         mov     ecx, dword ptr [ebp-4]
0056557F   .  BA 02000000     mov     edx, 2
00565584   .  B8 01000000     mov     eax, 1
00565589   .  E8 CAE3FEFF     call    00553958
0056558E   .  8B45 FC         mov     eax, dword ptr [ebp-4]
00565591   .  C640 78 C7      mov     byte ptr [eax+78], 0C7
00565595   .  8B4D FC         mov     ecx, dword ptr [ebp-4]
00565598   .  BA 02000000     mov     edx, 2
0056559D   .  B8 01000000     mov     eax, 1
005655A2   .  E8 D1E4FEFF     call    00553A78
005655A7   .  B2 01           mov     dl, 1
005655A9   .  A1 34B14300     mov     eax, dword ptr [43B134]
005655AE   .  E8 ED5CEDFF     call    0043B2A0
005655B3   .  8945 F4         mov     dword ptr [ebp-C], eax
005655B6   .  33C0            xor     eax, eax
005655B8   .  55              push    ebp
005655B9   .  68 3C565600     push    0056563C
005655BE   .  64:FF30        push    dword ptr fs:[eax]
005655C1   .  64:8920        mov     dword ptr fs:[eax], esp
005655C4   .  BA 02000080     mov     edx, 80000002
005655C9   .  8B45 F4         mov     eax, dword ptr [ebp-C]
005655CC   .  E8 AB5DEDFF     call    0043B37C
005655D1   .  B1 01           mov     cl, 1
005655D3   .  BA 08575600     mov     edx, 00565708                  ;  \\Software\\splog
005655D8   .  8B45 F4         mov     eax, dword ptr [ebp-C]
005655DB   .  E8 045EEDFF     call    0043B3E4
005655E0   .  84C0            test    al, al
005655E2   .  74 42           je      short 00565626
005655E4   .  8D55 E8         lea     edx, dword ptr [ebp-18]
005655E7   .  8B45 F8         mov     eax, dword ptr [ebp-8]
005655EA   .  8B80 FC020000  mov     eax, dword ptr [eax+2FC]
005655F0   .  E8 73BEEEFF     call    00451468
005655F5   .  8B4D E8         mov     ecx, dword ptr [ebp-18]
005655F8   .  BA 20575600     mov     edx, 00565720                  ;  hsn
005655FD   .  8B45 F4         mov     eax, dword ptr [ebp-C]
00565600   .  E8 3361EDFF     call    0043B738
00565605   .  8D55 E4         lea     edx, dword ptr [ebp-1C]
00565608   .  8B45 F8         mov     eax, dword ptr [ebp-8]
0056560B   .  8B80 04030000  mov     eax, dword ptr [eax+304]
00565611   .  E8 52BEEEFF     call    00451468
00565616   .  8B4D E4         mov     ecx, dword ptr [ebp-1C]
00565619   .  BA 2C575600     mov     edx, 0056572C                  ;  sn
0056561E   .  8B45 F4         mov     eax, dword ptr [ebp-C]
00565621   .  E8 1261EDFF     call    0043B738
00565626   >  33C0            xor     eax, eax
00565628   .  5A              pop     edx
00565629   .  59              pop     ecx
0056562A   .  59              pop     ecx
0056562B   .  64:8910        mov     dword ptr fs:[eax], edx
0056562E   .  68 43565600     push    00565643
00565633   >  8B45 F4         mov     eax, dword ptr [ebp-C]
```



```asm
00565636   .  E8 C5E3E9FF      call    00403A00
0056563B   .  C3               retn
0056563C   .^ E9 53EBE9FF      jmp     00404194
00565641   .^ EB F0            jmp     short 00565633
00565643   .  8D45 E0         lea     eax, dword ptr [ebp-20]
00565646   .  50               push    eax                         ; /Arg1
00565647   .  B8 38575600     mov     eax, 00565738                 ; |手机号码助手 V1.0(注册版)
0056564C   .  8945 D8         mov     dword ptr [ebp-28], eax     ; |
0056564F   .  C645 DC 0B      mov     byte ptr [ebp-24], 0B       ; |
00565653   .  8D55 D8         lea     edx, dword ptr [ebp-28]     ; |
00565656   .  33C9            xor     ecx, ecx                    ; |
00565658   .  B8 5C575600     mov     eax, 0056575C               ; |%s
0056565D   .  E8 A64EEAFF     call    0040A508                    ; \Log.0040A508
00565662   .  8B55 E0         mov     edx, dword ptr [ebp-20]
00565665   .  A1 9CED5600     mov     eax, dword ptr [56ED9C]
0056566A   .  E8 29BEEEFF     call    00451498
0056566F   .  6A 30           push    30                          ; /Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL
00565671   .  68 60575600     push    00565760                    ; |成功
00565676   .  68 68575600     push    00565768                    ; |注册成功
0056567B   .  6A 00           push    0                           ; |hOwner = NULL
0056567D   .  E8 4A23EAFF     call    <jmp.&user32.MessageBoxA>   ; \MessageBoxA
00565682   .  33C0           xor     eax, eax
00565684   .  5A             pop     edx
00565685   .  59             pop     ecx
00565686   .  59             pop     ecx
00565687   .  64:8910        mov     dword ptr fs:[eax], edx
0056568A   .  68 B2565600     push    005656B2
0056568F   >  8B45 FC        mov     eax, dword ptr [ebp-4]
00565692   .  E8 51D1E9FF     call    004027E8
00565697   .  C3             retn
00565698   .^ E9 F7EAE9FF     jmp     00404194
0056569D   .^ EB F0          jmp     short 0056568F
0056569F   >  6A 10          push    10                          ; /Style = MB_OK|MB_ICONHAND|MB_APPLMODAL
005656A1   .  68 74575600     push    00565774                    ; |错误
005656A6   .  68 7C575600     push    0056577C                    ; |注册失败
005656AB   .  6A 00          push    0                           ; |hOwner = NULL
005656AD   .  E8 1A23EAFF     call    <jmp.&user32.MessageBoxA>   ; \MessageBoxA
005656B2   >  33C0          xor     eax, eax
005656B4   .  5A             pop     edx
005656B5   .  59             pop     ecx
005656B6   .  59             pop     ecx
005656B7   .  64:8910        mov     dword ptr fs:[eax], edx
005656BA   .  68 CF565600     push    005656CF
005656BF   >  8B45 F8        mov     eax, dword ptr [ebp-8]
005656C2   .  E8 39E3E9FF     call    00403A00
005656C7   .  C3             retn
005656C8   .^ E9 C7EAE9FF     jmp     00404194
005656CD   .^ EB F0          jmp     short 005656BF
005656CF   .  33C0          xor     eax, eax
005656D1   .  5A             pop     edx
005656D2   .  59             pop     ecx
005656D3   .  59             pop     ecx
005656D4   .  64:8910        mov     dword ptr fs:[eax], edx
005656D7   .  68 F9565600     push    005656F9
005656DC   >  8D45 E0        lea     eax, dword ptr [ebp-20]
005656DF   .  E8 E8F0E9FF     call    004047CC
005656E4   .  8D45 E4        lea     eax, dword ptr [ebp-1C]
005656E7   .  BA 04000000     mov     edx, 4
005656EC   .  E8 FFF0E9FF     call    004047F0
005656F1   .  C3             retn
005656F2   .^ E9 9DEAE9FF     jmp     00404194
005656F7   .^ EB E3          jmp     short 005656DC
005656F9   .  5B             pop     ebx
005656FA   .  8BE5          mov     esp, ebp
005656FC   .  5D             pop     ebp
005656FD   .  C3             retn                          ;  返回程序
```

跟进 `00565545`：

```asm
0050A15C  /$  55              push    ebp                       ;  注册验证开始
0050A15D  |.  8BEC          mov     ebp, esp
0050A15F  |.  83C4 F8       add     esp, -8
0050A162  |.  53            push    ebx
0050A163  |.  56            push    esi
0050A164  |.  33DB          xor     ebx, ebx
0050A166  |.  895D F8       mov     dword ptr [ebp-8], ebx
0050A169  |.  894D FC       mov     dword ptr [ebp-4], ecx
0050A16C  |.  8BF2          mov     esi, edx
0050A16E  |.  8BD8          mov     ebx, eax
0050A170  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0050A173  |.  E8 14ABEFFF   call    00404C8C                    ;  计算机器码
0050A178  |.  8B45 08       mov     eax, dword ptr [ebp+8]
0050A17B  |.  E8 0CABEFFF   call    00404C8C                    ;  计算机器码长度
0050A180  |.  33C0          xor     eax, eax
0050A182  |.  55            push    ebp
0050A183  |.  68 DFA15000   push    0050A1DF
0050A188  |.  64:FF30       push    dword ptr fs:[eax]
0050A18B  |.  64:8920       mov     dword ptr fs:[eax], esp
0050A18E  |.  837D FC 00    cmp     dword ptr [ebp-4], 0
0050A192  |.  74 26         je      short 0050A1BA
0050A194  |.  85F6          test    esi, esi                    ;  关键码是否为 0
0050A196  |.  74 22         je      short 0050A1BA
0050A198  |.  8D45 F8       lea     eax, dword ptr [ebp-8]
0050A19B  |.  50            push    eax
0050A19C  |.  8B4D FC       mov     ecx, dword ptr [ebp-4]
0050A19F  |.  8BD6          mov     edx, esi
0050A1A1  |.  8BC3          mov     eax, ebx
0050A1A3  |.  E8 A4FEFFFF   call    0050A04C                    ;  ★注册算法 CALL，跟进！★
0050A1A8  |.  8B55 F8       mov     edx, dword ptr [ebp-8]      ;  真码出现，ASCII "6610842--1625409035-4470987"
0050A1AB  |.  8B45 08       mov     eax, dword ptr [ebp+8]      ;  假码出现，ASCII "99999999999999"
0050A1AE  |.  E8 35AAEFFF   call    00404BE8                    ;  用经典的比较一下
0050A1B3  |.  0F94C0        sete    al
0050A1B6  |.  8BD8          mov     ebx, eax
0050A1B8  |.  EB 02         jmp     short 0050A1BC
0050A1BA  |>  33DB          xor     ebx, ebx
0050A1BC  |>  33C0          xor     eax, eax
0050A1BE  |.  5A            pop     edx
0050A1BF  |.  59            pop     ecx
0050A1C0  |.  59            pop     ecx
0050A1C1  |.  64:8910       mov     dword ptr fs:[eax], edx
0050A1C4  |.  68 E6A15000   push    0050A1E6
0050A1C9  |>  8D45 F8       lea     eax, dword ptr [ebp-8]
0050A1CC  |.  BA 02000000   mov     edx, 2
0050A1D1  |.  E8 1AA6EFFF   call    004047F0
0050A1D6  |.  8D45 08       lea     eax, dword ptr [ebp+8]
0050A1D9  |.  E8 EEA5EFFF   call    004047CC
0050A1DE  \.  C3            retn
0050A1DF   .^ E9 B09FEFFF   jmp     00404194
0050A1E4   .^ EB E3          jmp     short 0050A1C9
0050A1E6   .  8BC3          mov     eax, ebx
0050A1E8   .  5E            pop     esi
0050A1E9   .  5B            pop     ebx
0050A1EA   .  59            pop     ecx
0050A1EB   .  59            pop     ecx
0050A1EC   .  5D            pop     ebp
0050A1ED   .  C2 0400       retn    4                           ;  返回上一级
```



## 跟进 0050A1A3

```
0050A04C  /$  55                 push    ebp                     ;  注册算法开始
0050A04D  |.  8BEC               mov     ebp, esp
0050A04F  |.  6A 00              push    0
0050A051  |.  6A 00              push    0
0050A053  |.  6A 00              push    0
0050A055  |.  6A 00              push    0
0050A057  |.  53                 push    ebx
0050A058  |.  56                 push    esi
0050A059  |.  57                 push    edi
0050A05A  |.  894D FC            mov     dword ptr [ebp-4], ecx
0050A05D  |.  8BFA               mov     edi, edx
0050A05F  |.  8B5D 08            mov     ebx, dword ptr [ebp+8]
0050A062  |.  8B45 FC            mov     eax, dword ptr [ebp-4]
0050A065  |.  E8 22ACEFFF        call    00404C8C                ;  取机器码
0050A06A  |.  33C0               xor     eax, eax
0050A06C  |.  55                 push    ebp
0050A06D  |.  68 3EA15000        push    0050A13E
0050A072  |.  64:FF30           push    dword ptr fs:[eax]
0050A075  |.  64:8920           mov     dword ptr fs:[eax], esp
0050A078  |.  85FF               test    edi, edi                ;  是否有关键码
0050A07A  |.  0F84 97000000     je      0050A117                ;  跳则挂！
0050A080  |.  837D FC 00        cmp     dword ptr [ebp-4], 0    ;  机器码是否小于等于 0
0050A084  |.  0F84 8D000000     je      0050A117                ;  跳则挂！
0050A08A  |.  8B45 FC            mov     eax, dword ptr [ebp-4]    ;  /*注册码第一段*/
0050A08D  |.  E8 0AAAEFFF        call    00404A9C                ;  计算机器码长度
0050A092  |.  8BF0               mov     esi, eax                ;  传送数据，EAX=0x8,EDI=0xC8583
0050A094  |.  0FAFF7             imul    esi, edi                ;  整数乘法，ESI=ESI*ESI=0x642C18
0050A097  |.  8B45 FC            mov     eax, dword ptr [ebp-4]    ;  取机器码，ASCII "E19YNZVE"
0050A09A  |.  0FB600             movzx   eax, byte ptr [eax]       ;  取机器码第一位的 ASCII 值，EAX=0x45 ('E')
0050A09D  |.  69C0 9A020000      imul    eax, eax, 29A             ;  整数乘法，EAX=EAX*0x29A=0xB382
0050A0A3  |.  03F0               add     esi, eax                ;  ESI=EAX+ESI=0x64DF9A
0050A0A5  |.  8D55 F8            lea     edx, dword ptr [ebp-8]
0050A0A8  |.  8BC6               mov     eax, esi                ;  传送数据，ESI=0x64DF9A
0050A0AA  |.  E8 59F6EFFF        call    00409708                ;  结果转换为十进制数
0050A0AF  |.  8B55 F8            mov     edx, dword ptr [ebp-8]    ;  /*注册码第一段结果*/ ASCII "6610842"
0050A0B2  |.  8BC3               mov     eax, ebx
0050A0B4  |.  B9 58A15000        mov     ecx, 0050A158           ;  取连接符“-”连接
0050A0B9  |.  E8 2AAAEFFF        call    00404AE8                ;  /*注册码第二段*/
0050A0BE  |.  8B45 FC            mov     eax, dword ptr [ebp-4]    ;  取机器码，ASCII "E19YNZVE"
0050A0C1  |.  0FB600             movzx   eax, byte ptr [eax]       ;  取机器码第一位的 ASCII 值，EAX=0x45 ('E')
0050A0C4  |.  F7EF               imul    edi                       ;  整数乘法，EDI=0xC8583，EAX=EAX*EDI=0x35FFC4F
0050A0C6  |.  6BF0 7B            imul    esi, eax, 7B              ;  整数乘法，ESI=EAX*0x7B=0x19F1E39F5
0050A0C9  |.  FF33               push    dword ptr [ebx]           ;  取前面计算好的注册码，ASCII "6610842-"
0050A0CB  |.  8D55 F4            lea     edx, dword ptr [ebp-C]    ;  ESI 取变量范围内的数值“0x0-0xFFFFFFFF”
0050A0CE  |.  8BC6               mov     eax, esi                ;  传送数据，ESI=9F1E39F5
0050A0D0  |.  E8 33F6EFFF        call    00409708                ;  结果转换为十进制数
0050A0D5  |.  FF75 F4            push    dword ptr [ebp-C]        ;  /*注册码第二段结果*/ ASCII "-1625409035"
0050A0D8  |.  68 58A15000        push    0050A158                ;  取连接符“-”连接
0050A0DD  |.  8BC3               mov     eax, ebx
0050A0DF  |.  BA 03000000        mov     edx, 3
0050A0E4  |.  E8 73AAEFFF        call    00404B5C                ;  /*注册码第三段*/
0050A0E9  |.  8B45 FC            mov     eax, dword ptr [ebp-4]    ;  取机器码，ASCII "E19YNZVE"
0050A0EC  |.  E8 ABA9EFFF        call    00404A9C                ;  计算机器码长度
0050A0F1  |.  8B55 FC            mov     edx, dword ptr [ebp-4]
0050A0F4  |.  0FB612             movzx   edx, byte ptr [edx]       ;  取机器码第一位的 ASCII 值，EDX=0x45 ('E')
0050A0F7  |.  F7EA               imul    edx                       ;  整数乘法，EAX=EDX*(机器码长度)=0x228
0050A0F9  |.  69F0 D5190000      imul    esi, eax, 19D5            ;  整数乘法，ESI=EAX*0x19D5=0x37B348
0050A0FF  |.  03F7               add     esi, edi                ;  ESI=ESI+0xC8583=0x4438CB
0050A101  |.  8D55 F0            lea     edx, dword ptr [ebp-10]
0050A104  |.  8BC6               mov     eax, esi
0050A106  |.  E8 FDF5EFFF        call    00409708                ;  结果转换为十进制数
0050A10B  |.  8B55 F0            mov     edx, dword ptr [ebp-10]   ;  /*注册码第三段结果*/ ASCII "4470987"
0050A10E  |.  8BC3               mov     eax, ebx
0050A110  |.  E8 8FA9EFFF        call    00404AA4                ;  注册码一、二、三段全部连接
0050A115  |.  EB 0C              jmp     short 0050A123
0050A117  |>  8BD3               mov     edx, ebx
0050A119  |.  A1 08B95600        mov     eax, dword ptr [56B908]
0050A11E  |.  E8 95C9EFFF        call    00406AB8
0050A123  |>  33C0               xor     eax, eax
0050A125  |.  5A                 pop     edx
0050A126  |.  59                 pop     ecx
0050A127  |.  59                 pop     ecx
0050A128  |.  64:8910           mov     dword ptr fs:[eax], edx
0050A12B  |.  68 45A15000        push    0050A145
0050A130  |>  8D45 F0            lea     eax, dword ptr [ebp-10]
0050A133  |.  BA 04000000        mov     edx, 4
0050A138  |.  E8 B3A6EFFF        call    004047F0                ;  检查注册码一、二、三段是否合法
0050A13D  \\. C3                 retn
0050A13E    .^ E9 51A0EFFF        jmp     00404194
0050A143    .^ EB EB             jmp     short 0050A130
0050A145    .  5F                 pop     edi
0050A146    .  5E                 pop     esi
0050A147    .  5B                 pop     ebx
0050A148    .  8BE5               mov     esp, ebp
0050A14A    .  5D                 pop     ebp
0050A14B    .  C2 0400            retn    4                        ;  返回程序
```

---

### 经验总结

算法不难，也就是机器码头一位与关键码 `0xC8583` 十进制数为 `820611` 作者生日？不停的做整数加减乘除运算而已。

正因为算法如此简单，我修改了它的算法，让它在所有机器上的注册码都为 `0-0-0`，呵呵（修改代码如下）

| 虚拟地址    | 原始字节 | 新的字节 |
|-------------|----------|----------|
| `0050A07A`  | `0F`     | `90`     |
| `0050A07B`  | `84`     | `90`     |
| `0050A07C`  | `97`     | `90`     |
| `0050A07D`  | `00`     | `90`     |
| `0050A07E`  | `00`     | `90`     |
| `0050A07F`  | `00`     | `90`     |
| `0050A0A3`  | `03`     | `21`     |
| `0050A0A4`  | `F0`     | `C6`     |
| `0050A0FF`  | `03`     | `21`     |
| `0050A100`  | `F7`     | `FE`     |
| `0050A196`  | `74`     | `90`     |
| `0050A197`  | `22`     | `90`     |
| `00565541`  | `83`     | `00`     |
| `00565542`  | `85`     | `00`     |
| `00565543`  | `0C`     | `00`     |
| `0056588A`  | `83`     | `00`     |
| `0056588B`  | `85`     | `00`     |
| `0056588C`  | `0C`     | `00`     |

代码:
```



```vb
'/*========================================*/
'/*     手机号码助手 1.0 注册机源码      */
'/*                                     */
'/*    程序编写：KuNgBiM               */
'/*                                     */
'/*  感谢 tracky 百忙之中抽空与我一同测试  */
'/*========================================*/

'固定值(常量)
Const Sign1 As Double = &HC8583
Const Sign2 As Double = &H29A
Const Sign3 As Double = &H7B
Const Sign4 As Double = &H19D5

Private Declare Function ShellExecute Lib "shell32.dll" Alias "ShellExecuteA" _
    (ByVal hwnd As Long, ByVal lpOperation As String, ByVal lpFile As String, _
     ByVal lpParameters As String, ByVal lpDirectory As String, ByVal nShowCmd As Long) As Long
'函数声明，产生网络链接效果'

Private Sub Command1_Click()
    '变量声明
    Dim Mac, Link As Variant
    Dim LenMac, FirstMac As Double
    Dim a, b, c, d As Double
    Dim Key1, Key2, Key3 As Long
    
    '固定值
    Link = "-"   '连接符
    
    '初始计算变量
    Mac = Text1.Text
    LenMac = Len(Mac)   '机器码长度
    FirstMac = Asc(Mac) '机器码首位的 ASCII 值
    
    If LenMac <= 0 Then   '是否有输入机器码
    Else
        Key1 = LenMac * Sign1 + FirstMac * Sign2   '注册码第一段
        Key2 = FirstMac * Sign1 * Sign3   '注册码第二段
        
        '取 imul 之后的低八位
        a = Key2 / &HFFFFFFF    '先取出 b 中的高 16 位
        a = Int(a / &H10)       '完整取出 b 中的高 16 位
        a = a * (2 ^ 32)        '将它还原成 32 位，但是低 16 位全部为零
        Key2 = Key2 - a         '用减法才不会溢出，这样就提取了我们所有的低 16 位
        
        c = c + Key2   '累加
        
        If c >= 4294967296# Then   '如果超出了 16 位，自然又要运算，取它的低 16 位
            b = c / &HFFFFFFF
            b = Int(b / &H10)
            b = b * (2 ^ 32)
            c = c - Key2
        End If
        
        '如果进位有符号产生
        If c + c > 2 ^ 32 Then
            d = 4294967296# - c  '而 4294967296# 就是 &H100000000 从而避开溢出
            Key2 = "-" & CStr(d)
        Else
            Key2 = CStr(c)
        End If
        
        Key3 = FirstMac * LenMac * Sign4 + Sign1   '注册码第三段
    End If
    
    Text2.Text = Key1 & Link & Key2 & Link & Key3   '组合后输出注册码
End Sub

Private Sub Command2_Click()
    NL = Chr(10) + Chr(13)
    MsgBox "KeyGen by KuNgBiM" & NL & NL & "2007-04-05", vbInformation, "KuNgBiM"
End Sub

Private Sub Label3_Click()
    ShellExecute 0, "open", "mailto:kungbim@163.com", vbNullString, vbNullString, 0
End Sub

Private Sub Text1_Click()
    Text1.Text = ""
End Sub
```

--------------------------------------------------------------------------------

**【版权声明】**: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 03 月 30 日 PM 07:12:52

`4dd466d0e1e0cc123afbe504da2462ca`
