
# 破解录音能手

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-313.htm`](https://www.pediy.com/kssd/pediy07/pediy7-313.htm)

【破解作者】 `yijun`
【作者邮箱】 `yijun8354@sina.com`
【使用工具】 `OD`, `PEID`
【破解平台】 `Win9x/NT/2000/XP`
【软件名称】 录音能手
【下载地址】 天空
【软件简介】 通过这款软件，您可以将您的声音录下来送给自己的朋友以表祝福和心意，或者将自己的歌声和将自己用乐器弹奏的乐曲录制下来，自己可以留下来欣赏或者送给亲人或者朋友。
【软件大小】 `419K`
【加壳方式】 `UPX 0.89.6 - 1.02 / 1.05 - 1.24` -> `Markus & Laszlo`
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

--------------------------------------------------------------------------------

## 破解内容

`PEID` 查壳知道该软件为 `UPX 0.89.6 - 1.02 / 1.05 - 1.24` -> `Markus & Laszlo` 壳，没什么难度轻松搞定！`OD` 载入通过字符串查找很容易来到以下关键处：

```
004B9CD0    55             push ebp     //在此下断点
004B9CD1    68 A79E4B00    push lyns\_.004B9EA7
004B9CD6    64:FF30        push dword ptr fs:[eax]
004B9CD9    64:8920        mov dword ptr fs:[eax],esp
004B9CDC    8D55 F8        lea edx,dword ptr ss:[ebp-8]
004B9CDF    8B83 08030000  mov eax,dword ptr ds:[ebx+308]
004B9CE5    E8 7E6AFAFF    call lyns\_.00460768          ; 计算注册码长度送 EAX
004B9CEA    8B45 F8        mov eax,dword ptr ss:[ebp-8]        ; 取注册码送 EAX
004B9CED    8D55 FC        lea edx,dword ptr ss:[ebp-4]        ; [ebp-4]地址送 EDX
004B9CF0    E8 DFEAF4FF    call lyns\_.004087D4
004B9CF5    837D FC 00     cmp dword ptr ss:[ebp-4],0          ; 注册码是否为空
004B9CF9    75 34          jnz short lyns\_.004B9D2F           ; 不为空就跳到 4B9D2F
004B9CFB    B8 209F4B00    mov eax,lyns\_.004B9F20
004B9D00    E8 0B95F7FF    call lyns\_.00433210
004B9D05    FF05 28074C00  inc dword ptr ds:[4C0728]
004B9D0B    833D 28074C00 0>cmp dword ptr ds:[4C0728],4
004B9D12    7C 0E          jl short lyns\_.004B9D22
004B9D14    33C0           xor eax,eax
004B9D16    A3 28074C00    mov dword ptr ds:[4C0728],eax
004B9D1B    8BC3           mov eax,ebx
004B9D1D    E8 7233FCFF    call lyns\_.0047D094
004B9D22    33C0           xor eax,eax
004B9D24    5A             pop edx
004B9D25    59             pop ecx
004B9D26    59             pop ecx
004B9D27    64:8910        mov dword ptr fs:[eax],edx
004B9D2A    E9 82010000    jmp lyns\_.004B9EB1
004B9D2F    A1 04094C00    mov eax,dword ptr ds:[4C0904]
004B9D34    8338 01        cmp dword ptr ds:[eax],1
004B9D37    0F85 28010000  jnz lyns\_.004B9E65
004B9D3D    8D55 F0        lea edx,dword ptr ss:[ebp-10]
004B9D40    8B83 08030000  mov eax,dword ptr ds:[ebx+308]
004B9D46    E8 1D6AFAFF    call lyns\_.00460768          ; 计算注册码长度送 EAX
004B9D4B    8B45 F0        mov eax,dword ptr ss:[ebp-10]       ; 注册码送 EAX
004B9D4E    8D55 F4        lea edx,dword ptr ss:[ebp-C]        ; [ebp-C]送 EDX
004B9D51    E8 7EEAF4FF    call lyns\_.004087D4
004B9D56    8B45 F4        mov eax,dword ptr ss:[ebp-C]        ; 注册码送 EAX
004B9D59    50             push eax                       ; EAX 入栈
004B9D5A    8D55 E4        lea edx,dword ptr ss:[ebp-1C]       ; [ebp-1C]送 EDX
004B9D5D    8B83 00030000  mov eax,dword ptr ds:[ebx+300]
004B9D63    E8 006AFAFF    call lyns\_.00460768          ; 计算机器码长度送 EAX
004B9D68    8B45 E4        mov eax,dword ptr ss:[ebp-1C]       ; 机器码送 EAX
004B9D6B    8D55 E8        lea edx,dword ptr ss:[ebp-18]       ; [ebp-18]地址送 EDX
004B9D6E    E8 61EAF4FF    call lyns\_.004087D4          ;
004B9D73    8B45 E8        mov eax,dword ptr ss:[ebp-18]       ; 机器码送 EAX
004B9D76    E8 8DACF4FF    call lyns\_.00404A08          ; 测试机器码是否为空
004B9D7B    50             push eax                       ; 机器码入栈
004B9D7C    A1 5C084C00    mov eax,dword ptr ds:[4C085C]
004B9D81    8B00           mov eax,dword ptr ds:[eax]
004B9D83    50             push eax
004B9D84    E8 DB140000    call lyns\_.004BB264          ; 跟进
004B9D89    8BD0           mov edx,eax
004B9D8B    8D45 EC        lea eax,dword ptr ss:[ebp-14]
004B9D8E    E8 ADA9F4FF    call lyns\_.00404740
004B9D93    8B55 EC        mov edx,dword ptr ss:[ebp-14]      ; EDX 为真码
004B9D96    58             pop eax                      ; 弹出假码
004B9D97    E8 B8ABF4FF    call lyns\_.00404954          ; 比较 CALL，EDX 中的为真码~~~
004B9D9C    74 49          je short lyns\_.004B9DE7            ; 关键跳
004B9D9E    B8 4C9F4B00    mov eax,lyns\_.004B9F4C
004B9DA3    E8 6894F7FF    call lyns\_.00433210
004B9DA8    8B83 08030000  mov eax,dword ptr ds:[ebx+308]
004B9DAE    33D2           xor edx,edx
004B9DB0    E8 E369FAFF    call lyns\_.00460798
004B9DB5    FF05 28074C00  inc dword ptr ds:[4C0728]
004B9DBB    833D 28074C00 0>cmp dword ptr ds:[4C0728],4
004B9DC2    7C 16          jl short lyns\_.004B9DDA
004B9DC4    B8 749F4B00    mov eax,lyns\_.004B9F74
004B9DC9    E8 4294F7FF    call lyns\_.00433210
004B9DCE    A1 640A4C00    mov eax,dword ptr ds:[4C0A64]
004B9DD3    8B00           mov eax,dword ptr ds:[eax]
004B9DD5    E8 BE69FCFF    call lyns\_.00480798
```

======================================

跟进 `004B9D84` 处 `CALL` 来到：

```
004BB264    55             push ebp
004BB265    8BEC           mov ebp,esp
004BB267    83C4 F8        add esp,-8
004BB26A    53             push ebx
004BB26B    56             push esi
004BB26C    57             push edi
004BB26D    33C0           xor eax,eax
004BB26F    8945 F8        mov dword ptr ss:[ebp-8],eax
004BB272    33C0           xor eax,eax
004BB274    55             push ebp
004BB275    68 F3B24B00    push lyns\_.004BB2F3
004BB27A    64:FF30        push dword ptr fs:[eax]
004BB27D    64:8920        mov dword ptr fs:[eax],esp
004BB280    33C0           xor eax,eax                 ; EAX 清 0
004BB282    55             push ebp
004BB283    68 D3B24B00    push lyns\_.004BB2D3
004BB288    64:FF30        push dword ptr fs:[eax]
004BB28B    64:8920        mov dword ptr fs:[eax],esp
004BB28E    68 08B34B00    push lyns\_.004BB308        ; ASCII "ILOVEYOU"
004BB293    8B45 0C        mov eax,dword ptr ss:[ebp+C]        ; 机器码送 EAX
004BB296    50             push eax                       ; 机器码入栈
004BB297    E8 F0D1FFFF    call lyns\_.004B848C
004BB29C    8BD0           mov edx,eax
004BB29E    8D45 F8        lea eax,dword ptr ss:[ebp-8]
004BB2A1    E8 9A94F4FF    call lyns\_.00404740
004BB2A6    8B45 F8        mov eax,dword ptr ss:[ebp-8]
004BB2A9    E8 5A97F4FF    call lyns\_.00404A08
004BB2AE    50             push eax
004BB2AF    E8 8CE5FFFF    call lyns\_.004B9840        ; 跟进
004BB2B4    8BD0           mov edx,eax
004BB2B6    8D45 F8        lea eax,dword ptr ss:[ebp-8]
004BB2B9    E8 8294F4FF    call lyns\_.00404740
004BB2BE    8B45 F8        mov eax,dword ptr ss:[ebp-8]
004BB2C1    E8 4297F4FF    call lyns\_.00404A08
004BB2C6    8945 FC        mov dword ptr ss:[ebp-4],eax
004BB2C9    33C0           xor eax,eax
004BB2CB    5A             pop edx
004BB2CC    59             pop ecx
004BB2CD    59             pop ecx
004BB2CE    64:8910        mov dword ptr fs:[eax],edx
004BB2D1    EB 0A          jmp short lyns\_.004BB2DD
004BB2D3   ^ E9 A089F4FF  jmp lyns\_.00403C78
004BB2D8    E8 038DF4FF    call lyns\_.00403FE0
004BB2DD    33C0           xor eax,eax
004BB2DF    5A             pop edx
004BB2E0    59             pop ecx
004BB2E1    59             pop ecx
004BB2E2    64:8910        mov dword ptr fs:[eax],edx
004BB2E5    68 F2B24B00    push lyns\_.004BB2FA
004BB2EA    8D45 F8        lea eax,dword ptr ss:[ebp-8]
004BB2ED    E8 5692F4FF    call lyns\_.00404548
```



## `004BB2F2` `retn` 指令分析

`======================================================================================================`

跟进 `004BB2AF` 处 `CALL` 来到：

```
004B9840    55              push ebp  
004B9841    8BEC            mov ebp,esp  
004B9843    83C4 E8         add esp,-18  
004B9846    53              push ebx  
004B9847    56              push esi  
004B9848    57              push edi  
004B9849    33C0            xor eax,eax  
004B984B    8945 F8         mov dword ptr ss:[ebp-8],eax  
004B984E    33C0            xor eax,eax  
004B9850    55              push ebp  
004B9851    68 B5984B00     push lyns_.004B98B5  
004B9856    64:FF30         push dword ptr fs:[eax]  
004B9859    64:8920         mov dword ptr fs:[eax],esp  
004B985C    33C0            xor eax,eax  
004B985E    55              push ebp  
004B985F    68 95984B00     push lyns_.004B9895  
004B9864    64:FF30         push dword ptr fs:[eax]  
004B9867    64:8920         mov dword ptr fs:[eax],esp  
004B986A    8D55 E8         lea edx,dword ptr ss:[ebp-18]  
004B986D    8B45 08         mov eax,dword ptr ss:[ebp+8]  
004B9870    E8 67FEFFFF     call lyns_.004B96DC  
004B9875    8D45 E8         lea eax,dword ptr ss:[ebp-18]  
004B9878    8D55 F8         lea edx,dword ptr ss:[ebp-8]  
004B987B    E8 F4FEFFFF     call lyns_.004B9774                ; 跟进  
004B9880    8B45 F8         mov eax,dword ptr ss:[ebp-8]  
004B9883    E8 80B1F4FF     call lyns_.00404A08                ; 此处的 EAX 中保存的就是真码^-^  
004B9888    8945 FC         mov dword ptr ss:[ebp-4],eax  
004B988B    33C0            xor eax,eax  
004B988D    5A              pop edx  
004B988E    59              pop ecx  
004B988F    59              pop ecx  
004B9890    64:8910         mov dword ptr fs:[eax],edx  
004B9893    EB 0A          jmp short lyns_.004B989F  
004B9895  ^ E9 DEA3F4FF    jmp lyns_.00403C78  
004B989A    E8 41A7F4FF     call lyns_.00403FE0  
004B989F    33C0            xor eax,eax  
004B98A1    5A              pop edx  
004B98A2    59              pop ecx  
004B98A3    59              pop ecx  
004B98A4    64:8910         mov dword ptr fs:[eax],edx  
004B98A7    68 BC984B00     push lyns_.004B98BC  
004B98AC    8D45 F8         lea eax,dword ptr ss:[ebp-8]  
004B98AF    E8 94ACF4FF     call lyns_.00404548  
004B98B4    C3              retn  
```

`======================================================================================================`

跟进 `004B987B` 处 `CALL` 来到：

```
004B9774    55              push ebp  
004B9775    8BEC            mov ebp,esp  
004B9777    83C4 E8         add esp,-18  
004B977A    53              push ebx  
004B977B    56              push esi  
004B977C    57              push edi  
004B977D    33C9            xor ecx,ecx  
004B977F    894D EC         mov dword ptr ss:[ebp-14],ecx  
004B9782    894D E8         mov dword ptr ss:[ebp-18],ecx  
004B9785    8BF0            mov esi,eax  
004B9787    8D7D F0         lea edi,dword ptr ss:[ebp-10]  
004B978A    A5              movs dword ptr es:[edi],dword ptr d>  
004B978B    A5              movs dword ptr es:[edi],dword ptr d>  
004B978C    A5              movs dword ptr es:[edi],dword ptr d>  
004B978D    A5              movs dword ptr es:[edi],dword ptr d>  
004B978E    8BFA            mov edi,edx  
004B9790    33C0            xor eax,eax  
004B9792    55              push ebp  
004B9793    68 31984B00     push lyns_.004B9831  
004B9798    64:FF30         push dword ptr fs:[eax]  
004B979B    64:8920         mov dword ptr fs:[eax],esp  
004B979E    33C0            xor eax,eax  
004B97A0    55              push ebp  
004B97A1    68 0C984B00     push lyns_.004B980C  
004B97A6    64:FF30         push dword ptr fs:[eax]  
004B97A9    64:8920         mov dword ptr fs:[eax],esp  
004B97AC    8BC7            mov eax,edi  
004B97AE    E8 95ADF4FF     call lyns_.00404548  
004B97B3    B3 10           mov bl,10                     ; BL=10  
004B97B5    8D75 F0         lea esi,dword ptr ss:[ebp-10]  
004B97B8    FF37            push dword ptr ds:[edi]       ; [edi]入栈，以下循环出真码^-^  
004B97BA    8D45 EC         lea eax,dword ptr ss:[ebp-14]   ; [ebp-14]送 EAX  
004B97BD    33D2            xor edx,edx                   ; EDX 清 0  
004B97BF    8A16            mov dl,byte ptr ds:[esi]       ; [esi]送 DL  
004B97C1    C1EA 04         shr edx,4                     ; EDX 右移动 4 位  
004B97C4    83E2 0F         and edx,0F                    ; EDX 和 0F 与  
004B97C7    8A92 18074C00   mov dl,byte ptr ds:[edx+4C0718] ; [edx+4C0718]送 DL  
004B97CD    E8 5EAFF4FF     call lyns_.00404730  
004B97D2    FF75 EC         push dword ptr ss:[ebp-14]  
004B97D5    8D45 E8         lea eax,dword ptr ss:[ebp-18]  
004B97D8    8A16            mov dl,byte ptr ds:[esi]       ; [esi]送 DL  
004B97DA    80E2 0F         and dl,0F                     ; DL 和 0F 与  
004B97DD    81E2 FF000000   and edx,0FF                   ; EDX 和 0FF 与  
004B97E3    8A92 18074C00   mov dl,byte ptr ds:[edx+4C0718] ; [edx+4C0718]送 DL  
004B97E9    E8 42AFF4FF     call lyns_.00404730  
004B97EE    FF75 E8         push dword ptr ss:[ebp-18]  
004B97F1    8BC7            mov eax,edi  
004B97F3    BA 03000000     mov edx,3                     ; EDX=3  
004B97F8    E8 CBB0F4FF     call lyns_.004048C8           ; 跟进  
004B97FD    46              inc esi                       ; ESI 加一  
004B97FE    FECB            dec bl                        ; BL 减一  
004B9800  ^ 75 B6         jnz short lyns_.004B97B8       ; BL 不为 0 就继续循环  
004B9802    33C0            xor eax,eax  
004B9804    5A              pop edx  
004B9805    59              pop ecx  
004B9806    59              pop ecx  
004B9807    64:8910         mov dword ptr fs:[eax],edx  
004B980A    EB 0A          jmp short lyns_.004B9816  
004B980C  ^ E9 67A4F4FF    jmp lyns_.00403C78  
004B9811    E8 CAA7F4FF     call lyns_.00403FE0  
004B9816    33C0            xor eax,eax  
004B9818    5A              pop edx  
004B9819    59              pop ecx  
004B981A    59              pop ecx  
004B981B    64:8910         mov dword ptr fs:[eax],edx  
004B981E    68 38984B00     push lyns_.004B9838  
004B9823    8D45 E8         lea eax,dword ptr ss:[ebp-18]  
004B9826    BA 02000000     mov edx,2  
004B982B    E8 3CADF4FF     call lyns_.0040456C  
004B9830    C3              retn  
```

`======================================================================================================`

跟进 `004B97F8` 处 `CALL` 来到：

```
004048C8    53              push ebx  
004048C9    56              push esi  
004048CA    57              push edi  
004048CB    52              push edx  
004048CC    50              push eax  
004048CD    89D3            mov ebx,edx  
004048CF    31FF            xor edi,edi                 ; EDI 清 0  
004048D1    8B4C94 14       mov ecx,dword ptr ss:[esp+edx*4+14] ; 已经循环出的注册码送 ECX  
004048D5    85C9            test ecx,ecx  
004048D7    74 0C           je short lyns_.004048E5     ; 为 0 就跳  
004048D9    3908            cmp dword ptr ds:[eax],ecx   ; [eax]和 ECX 比较  
004048DB    75 08           jnz short lyns_.004048E5    ; 不等就跳  
004048DD    89CF            mov edi,ecx                 ; ECX 送 EDI  
004048DF    8B41 FC         mov eax,dword ptr ds:[ecx-4] ; [ecx-4]送 EAX  
004048E2    4A              dec edx                     ; EDX 减一  
004048E3    EB 02          jmp short lyns_.004048E7  
004048E5    31C0            xor eax,eax  
004048E7    8B4C94 14       mov ecx,dword ptr ss:[esp+edx*4+14] ; [esp+edx*4+14]送 ECX（逐位把真码送到 ECX）  
004048EB    85C9            test ecx,ecx  
```



```
004048ED    74 09            je short lyns\_\_.004048F8  

004048EF    0341 FC          add eax,dword ptr ds:\[ecx\-4\]        ; EAX 加\[ecx-4\]=1 送 EAX（已经计算出的真码长度）  

004048F2    39CF            cmp edi,ecx                             ; ECX 和 EDI 比较  

004048F4    75 02           jnz short lyns\_\_.004048F8               ; 不等就跳  

004048F6    31FF            xor edi,edi  

004048F8    4A              dec edx  

004048F9  ^ 75 EC           jnz short lyns\_\_.004048E7               ; EDX 不为 0 就继续  

004048FB    85FF            test edi,edi  

004048FD    74 17           je short lyns\_\_.00404916  

004048FF    89C2            mov edx,eax  

00404901    8B0424          mov eax,dword ptr ss:\[esp\]  

00404904    8B77 FC         mov esi,dword ptr ds:\[edi\-4\]  

00404907    E8 88020000     call lyns\_\_.00404B94  

0040490C    8B3C24          mov edi,dword ptr ss:\[esp\]  

0040490F    FF37            push dword ptr ds:\[edi\]  

00404911    0337            add esi,dword ptr ds:\[edi\]              ; \[edi\]加 ESI  

00404913    4B              dec ebx                                   ; EBX 减一  

00404914    EB 08           jmp short lyns\_\_.0040491E  

00404916    E8 F1FCFFFF     call lyns\_\_.0040460C  

0040491B    50              push eax  

0040491C    89C6            mov esi,eax  

0040491E    8B449C 18       mov eax,dword ptr ss:\[esp+ebx\*4+18\] ; 计算出的真码保存在 EAX  

00404922    89F2            mov edx,esi  

00404924    85C0            test eax,eax  

00404926    74 0A           je short lyns\_\_.00404932  

00404928    8B48 FC         mov ecx,dword ptr ds:\[eax\-4\]          ; ECX=1  

0040492B    01CE            add esi,ecx                               ; ESI 加 ECX  

0040492D    E8 32E0FFFF     call lyns\_\_.00402964                     ;  

00404932    4B              dec ebx  

00404933  ^ 75 E9           jnz short lyns\_\_.0040491E               ; EBX 不为 0 继续  

00404935    5A              pop edx  

00404936    58              pop eax  

00404937    85FF            test edi,edi  

00404939    75 0C           jnz short lyns\_\_.00404947  

0040493B    85D2            test edx,edx  

0040493D    74 03           je short lyns\_\_.00404942  

0040493F    FF4A F8         dec dword ptr ds:\[edx\-8\]  

00404942    E8 55FCFFFF     call lyns\_\_.0040459C  

00404947    5A              pop edx  

00404948    5F              pop edi  

00404949    5E              pop esi  

0040494A    5B              pop ebx  

0040494B    58              pop eax  

0040494C    8D2494          lea esp,dword ptr ss:\[esp+edx\*4\]  

0040494F    FFE0            jmp eax  

00404951    C3              retn  

 \--------------------------------------------------------------------------------
```

## 破解总结

注册机：

* 中断地址：`4B9D97`
* 中断次数：`1`
* 第一字节：`E8`
* 指令长度：`5`
* 内存方式：寄存器 `EDX`

--------------------------------------------------------------------------------

## 版权声明

本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
