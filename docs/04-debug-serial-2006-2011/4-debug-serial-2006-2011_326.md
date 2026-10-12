
# 傲群选股专家 5.5.0 算法分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-48.htm`](https://www.pediy.com/kssd/pediy09/pediy09-48.htm)
>
> 【文章标题】: 傲群选股专家 5.5.0 算法分析
>
> 【文章作者】: `KuNgBiM`
>
> 【作者邮箱】: `kungbim@163.com`
>
> 【作者主页】: `http://www.crkcn.com`
>
> 【详细过程】
>
> 试用注册：
>
> 用户姓名：`KuNgBiM`
>
> 电子邮件：`kungbim@163.com`
>
> 注册密码：`1234567890123456`
>
> 点击“注册”后提示“输入密码不正确！”
>
> 程序无壳，为 `Borland C++ 1999` 所编译，`OD` 直接载入调试。
>
> `OD` 加载插件查找该提示字符串找到：
>
> ```
> 0045A539   |.   BA 34895200          |mov     edx, 00528934              ;  输入密码不正确！
> ```
>
> 我们来到 `00459EEC` 这里下断并进行正式调试：
>
> ```
> 00459EEC   /$  55                   push    ebp                          ;  来到这里下断后 F9 运行！
> 00459EED   |.   8BEC                 mov     ebp, esp
> 00459EEF   |.   81C4 60FFFFFF        add     esp, -0A0
> 00459EF5   |.   B8 C0945200          mov     eax, 005294C0
> 00459EFA   |.   53                   push    ebx
> 00459EFB   |.   56                   push    esi
> 00459EFC   |.   57                   push    edi
> 00459EFD   |.   8D9D 64FFFFFF        lea     ebx, dword ptr [ebp-9C]
> 00459F03   |.   8B75 0C              mov     esi, dword ptr [ebp+C]
> 00459F06   |.   E8 15D00900          call    004F6F20
> 00459F0B   |.   66:C743 10 0800      mov     word ptr [ebx+10], 8       ;  检查用户姓名长度
> 00459F11   |.   33D2                 xor     edx, edx
> 00459F13   |.   8955 FC              mov     dword ptr [ebp-4], edx
> 00459F16   |.   8D55 FC              lea     edx, dword ptr [ebp-4]
> 00459F19   |.   FF43 1C              inc     dword ptr [ebx+1C]
> 00459F1C   |.   8B86 EC020000        mov     eax, dword ptr [esi+2EC]
> 00459F22   |.   E8 BB210B00          call    <jmp.&VCL50.Controls::TControl::>;  命令查找文本框中的文本
> 00459F27   |.   837D FC 00           cmp     dword ptr [ebp-4], 0       ;  ASCII "KuNgBiM"
> 00459F2B   |.   0F94C1               sete    cl
> 00459F2E   |.   83E1 01              and     ecx, 1
> 00459F31   |.   8D45 FC              lea     eax, dword ptr [ebp-4]     ;  得到用户姓名长度
> 00459F34   |.   51                   push    ecx                        ; /Arg1
> 00459F35   |.   BA 02000000          mov     edx, 2                     ; |
> 00459F3A   |.   FF4B 1C              dec     dword ptr [ebx+1C]         ; |
> 00459F3D   |.   E8 7ED20900          call    004F71C0                   ; \\expert.004F71C0
> 00459F42   |.   59                   pop     ecx
> 00459F43   |.   84C9                 test    cl, cl                     ;  比较 CL 中的值是否为 0
> 00459F45   |.   74 53                je      short 00459F9A             ;  不为 0 则跳
> 00459F47   |.   66:C743 10 1400      mov     word ptr [ebx+10], 14
> 00459F4D   |.   BA 77885200          mov     edx, 00528877              ;  用户姓名不能为空！
> 00459F52   |.   8D45 F8              lea     eax, dword ptr [ebp-8]
> 00459F55   |.   E8 C2D00900          call    004F701C
> 00459F5A   |.   FF43 1C              inc     dword ptr [ebx+1C]
> 00459F5D   |.   8D55 F8              lea     edx, dword ptr [ebp-8]
> 00459F60   |.   8B45 08              mov     eax, dword ptr [ebp+8]
> 00459F63   |.   E8 88D20900          call    004F71F0
> 00459F68   |.   8B45 08              mov     eax, dword ptr [ebp+8]
> 00459F6B   |.   BA 02000000          mov     edx, 2
> 00459F70   |.   66:C743 10 2000      mov     word ptr [ebx+10], 20
> 00459F76   |.   50                   push    eax
> 00459F77   |.   8D45 F8              lea     eax, dword ptr [ebp-8]
> 00459F7A   |.   FF4B 1C              dec     dword ptr [ebx+1C]
> 00459F7D   |.   E8 3ED20900          call    004F71C0
> 00459F82   |.   58                   pop     eax
> 00459F83   |.   66:C743 10 1400      mov     word ptr [ebx+10], 14
> 00459F89   |.   FF43 1C              inc     dword ptr [ebx+1C]
> 00459F8C   |.   8B13                 mov     edx, dword ptr [ebx]
> 00459F8E   |.   64:8915 00000000     mov     dword ptr fs:[0], edx
> 00459F95   |.   E9 B6060000          jmp     0045A650
> 00459F9A   |>   66:C743 10 2C00      mov     word ptr [ebx+10], 2C      ;  检查电子邮件长度
> 00459FA0   |.   33C9                 xor     ecx, ecx
> 00459FA2   |.   894D F4              mov     dword ptr [ebp-C], ecx
> 00459FA5   |.   8D55 F4              lea     edx, dword ptr [ebp-C]
> 00459FA8   |.   FF43 1C              inc     dword ptr [ebx+1C]
> 00459FAB   |.   8B86 F0020000        mov     eax, dword ptr [esi+2F0]
> 00459FB1   |.   E8 2C210B00          call    <jmp.&VCL50.Controls::TControl::>;  命令查找文本框中的文本
> 00459FB6   |.   837D F4 00           cmp     dword ptr [ebp-C], 0       ;  ASCII "kungbim@163.com"
> 00459FBA   |.   0F94C1               sete    cl
> 00459FBD   |.   83E1 01              and     ecx, 1
> 00459FC0   |.   8D45 F4              lea     eax, dword ptr [ebp-C]     ;  得到电子邮件长度
> 00459FC3   |.   51                   push    ecx                        ; /Arg1
> 00459FC4   |.   BA 02000000          mov     edx, 2                     ; |
> 00459FC9   |.   FF4B 1C              dec     dword ptr [ebx+1C]         ; |
> 00459FCC   |.   E8 EFD10900          call    004F71C0                   ; \\expert.004F71C0
> 00459FD1   |.   59                   pop     ecx
> 00459FD2   |.   84C9                 test    cl, cl                     ;  比较 CL 中的值是否为 0
> 00459FD4   |.   74 53                je      short 0045A029             ;  不为 0 则跳
> 00459FD6   |.   66:C743 10 3800      mov     word ptr [ebx+10], 38
> 00459FDC   |.   BA 8A885200          mov     edx, 0052888A              ;  电子邮件地址不能为空！
> 00459FE1   |.   8D45 F0              lea     eax, dword ptr [ebp-10]
> 00459FE4   |.   E8 33D00900          call    004F701C
> 00459FE9   |.   FF43 1C              inc     dword ptr [ebx+1C]
> 00459FEC   |.   8D55 F0              lea     edx, dword ptr [ebp-10]
> 00459FEF   |.   8B45 08              mov     eax, dword ptr [ebp+8]
> 00459FF2   |.   E8 F9D10900          call    004F71F0
> 00459FF7   |.   8B45 08              mov     eax, dword ptr [ebp+8]
> 00459FFA   |.   BA 02000000          mov     edx, 2
> 00459FFF   |.   66:C743 10 4400      mov     word ptr [ebx+10], 44
> 0045A005   |.   50                   push    eax
> 0045A006   |.   8D45 F0              lea     eax, dword ptr [ebp-10]
> 0045A009   |.   FF4B 1C              dec     dword ptr [ebx+1C]
> 0045A00C   |.   E8 AFD10900          call    004F71C0
> 0045A011   |.   58                   pop     eax
> 0045A012   |.   66:C743 10 3800      mov     word ptr [ebx+10], 38
> 0045A018   |.   FF43 1C              inc     dword ptr [ebx+1C]
> 0045A01B   |.   8B13                 mov     edx, dword ptr [ebx]
> 0045A01D   |.   64:8915 00000000     mov     dword ptr fs:[0], edx
> 0045A024   |.   E9 27060000          jmp     0045A650
> 0045A029   |>   66:C743 10 5000      mov     word ptr [ebx+10], 50      ;  检查注册密码长度
> 0045A02F   |.   33C9                 xor     ecx, ecx
> 0045A031   |.   894D EC              mov     dword ptr [ebp-14], ecx
> 0045A034   |.   8D55 EC              lea     edx, dword ptr [ebp-14]
> 0045A037   |.   FF43 1C              inc     dword ptr [ebx+1C]
> 0045A03A   |.   8B86 F4020000        mov     eax, dword ptr [esi+2F4]
> 0045A040   |.   E8 9D200B00          call    <jmp.&VCL50.Controls::TControl::>;  命令查找文本框中的文本
> 0045A045   |.   837D EC 00           cmp     dword ptr [ebp-14], 0      ;  ASCII "1234567890123456"
> 0045A049   |.   0F94C1               sete    cl
> 0045A04C   |.   83E1 01              and     ecx, 1
> 0045A04F   |.   8D45 EC              lea     eax, dword ptr [ebp-14]    ;  得到注册密码长度
> 0045A052   |.   51                   push    ecx                        ; /Arg1
> ```



```
0045A053  |.  BA 02000000        mov     edx, 2                              ; |  
0045A058  |.  FF4B 1C            dec     dword ptr [ebx+1C]                   ; |  
0045A05B  |.  E8 60D10900        call    004F71C0                             ; \expert.004F71C0  
0045A060  |.  59                 pop     ecx                                  
0045A061  |.  84C9               test    cl, cl                               ;  比较 CL 中的值是否为 0  
0045A063  |.  74 53              je      short 0045A0B8                       ;  不为 0 则跳  
0045A065  |.  66:C743 10 5C00     mov     word ptr [ebx+10], 5C              
0045A06B  |.  BA A1885200        mov     edx, 005288A1                        ;  注册密码不能为空！  
0045A070  |.  8D45 E8            lea     eax, dword ptr [ebp-18]             
0045A073  |.  E8 A4CF0900        call    004F701C                             
0045A078  |.  FF43 1C            inc     dword ptr [ebx+1C]                   
0045A07B  |.  8D55 E8            lea     edx, dword ptr [ebp-18]             
0045A07E  |.  8B45 08            mov     eax, dword ptr [ebp+8]               
0045A081  |.  E8 6AD10900        call    004F71F0                             
0045A086  |.  8B45 08            mov     eax, dword ptr [ebp+8]               
0045A089  |.  BA 02000000        mov     edx, 2                               
0045A08E  |.  66:C743 10 6800    mov     word ptr [ebx+10], 68                
0045A094  |.  50                 push    eax                                  
0045A095  |.  8D45 E8            lea     eax, dword ptr [ebp-18]             
0045A098  |.  FF4B 1C            dec     dword ptr [ebx+1C]                   
0045A09B  |.  E8 20D10900        call    004F71C0                             
0045A0A0  |.  58                 pop     eax                                  
0045A0A1  |.  66:C743 10 5C00    mov     word ptr [ebx+10], 5C                
0045A0A7  |.  FF43 1C            inc     dword ptr [ebx+1C]                   
0045A0AA  |.  8B13               mov     edx, dword ptr [ebx]                 
0045A0AC  |.  64:8915 00000000   mov     dword ptr fs:[0], edx                
0045A0B3  |.  E9 98050000        jmp     0045A650                             
0045A0B8  |>  66:C743 10 7400    mov     word ptr [ebx+10], 74                ;  检查用户姓名的合法性  
0045A0BE  |.  33C9               xor     ecx, ecx                             
0045A0C0  |.  894D E4            mov     dword ptr [ebp-1C], ecx              
0045A0C3  |.  8D55 E4            lea     edx, dword ptr [ebp-1C]              
0045A0C6  |.  FF43 1C            inc     dword ptr [ebx+1C]                   
0045A0C9  |.  8B86 EC020000      mov     eax, dword ptr [esi+2EC]             
0045A0CF  |.  E8 0E200B00        call    <jmp.&VCL50.Controls::TControl::>;  命令查找文本框中的文本  
0045A0D4  |.  8D45 E4            lea     eax, dword ptr [ebp-1C]              ;  计算用户姓名长度，便于 0045A0F3 处检查  
0045A0D7  |.  33D2               xor     edx, edx                             
0045A0D9  |.  8955 E0            mov     dword ptr [ebp-20], edx              
0045A0DC  |.  8D55 E0            lea     edx, dword ptr [ebp-20]              
0045A0DF  |.  FF43 1C            inc     dword ptr [ebx+1C]                   
0045A0E2  |.  E8 11D40900        call    004F74F8                             
0045A0E7  |.  8D45 E0            lea     eax, dword ptr [ebp-20]              
0045A0EA  |.  50                 push    eax                                  
0045A0EB  |.  BA B4885200        mov     edx, 005288B4                        
0045A0F0  |.  8D45 DC            lea     eax, dword ptr [ebp-24]              
0045A0F3  |.  E8 24CF0900        call    004F701C                             ;  依次检查比较文本框内字符 ASCII 值  
0045A0F8  |.  FF43 1C            inc     dword ptr [ebx+1C]                   ;  反复检查  
0045A0FB  |.  8D55 DC            lea     edx, dword ptr [ebp-24]              
0045A0FE  |.  58                 pop     eax                                  
0045A0FF  |.  E8 C0D20900        call    004F73C4                             
0045A104  |.  50                 push    eax                                  
0045A105  |.  FF4B 1C            dec     dword ptr [ebx+1C]                   
0045A108  |.  8D45 DC            lea     eax, dword ptr [ebp-24]              
0045A10B  |.  BA 02000000        mov     edx, 2                               
0045A110  |.  E8 ABD00900        call    004F71C0                             
0045A115  |.  FF4B 1C            dec     dword ptr [ebx+1C]                   
0045A118  |.  8D45 E0            lea     eax, dword ptr [ebp-20]              
0045A11B  |.  BA 02000000        mov     edx, 2                               
0045A120  |.  E8 9BD00900        call    004F71C0                             
0045A125  |.  FF4B 1C            dec     dword ptr [ebx+1C]                   ;  |  
0045A128  |.  8D45 E4            lea     eax, dword ptr [ebp-1C]              ;  |  
0045A12B  |.  BA 02000000        mov     edx, 2                               ;  |  
0045A130  |.  E8 8BD00900        call    004F71C0                             ;  \expert.004F71C0  
0045A135  |.  59                 pop     ecx                                  
0045A136  |.  85C9               test    ecx, ecx                             ;  比较 ECX 中的值是否为 0  
0045A138  |.  74 53              je      short 0045A18D                       ;  不为 0 则跳  
0045A13A  |.  66:C743 10 8000    mov     word ptr [ebx+10], 80                
0045A140  |.  BA B6885200        mov     edx, 005288B6                        ;  用户姓名中不能存在空格！  
0045A145  |.  8D45 D8            lea     eax, dword ptr [ebp-28]              
0045A148  |.  E8 CFCE0900        call    004F701C                             
0045A14D  |.  FF43 1C            inc     dword ptr [ebx+1C]                   
0045A150  |.  8D55 D8            lea     edx, dword ptr [ebp-28]              
0045A153  |.  8B45 08            mov     eax, dword ptr [ebp+8]               
0045A156  |.  E8 95D00900        call    004F71F0                             
0045A15B  |.  8B45 08            mov     eax, dword ptr [ebp+8]               
0045A15E  |.  BA 02000000        mov     edx, 2                               
0045A163  |.  66:C743 10 8C00    mov     word ptr [ebx+10], 8C                
0045A169  |.  50                 push    eax                                  
0045A16A  |.  8D45 D8            lea     eax, dword ptr [ebp-28]              
0045A16D  |.  FF4B 1C            dec     dword ptr [ebx+1C]                   
0045A170  |.  E8 4BD00900        call    004F71C0                             
0045A175  |.  58                 pop     eax                                  
0045A176  |.  66:C743 10 8000    mov     word ptr [ebx+10], 80                
0045A17C  |.  FF43 1C            inc     dword ptr [ebx+1C]                   
0045A17F  |.  8B13               mov     edx, dword ptr [ebx]                 
0045A181  |.  64:8915 00000000   mov     dword ptr fs:[0], edx                
0045A188  |.  E9 C3040000        jmp     0045A650                             
0045A18D  |>\66:C743 10 9800     mov     word ptr [ebx+10], 98                ;  检查电子邮件的合法性  
0045A193  |.  33C9               xor     ecx, ecx                             
0045A195  |.  894D D4            mov     dword ptr [ebp-2C], ecx              
0045A198  |.  8D55 D4            lea     edx, dword ptr [ebp-2C]              
0045A19B  |.  FF43 1C            inc     dword ptr [ebx+1C]                   
0045A19E  |.  8B86 F0020000      mov     eax, dword ptr [esi+2F0]             
0045A1A4  |.  E8 391F0B00        call    <jmp.&VCL50.Controls::TControl::>;  命令查找文本框中的文本  
0045A1A9  |.  8D45 D4            lea     eax, dword ptr [ebp-2C]              ;  计算电子邮件长度，便于 0045A1C8 处检查  
0045A1AC  |.  33D2               xor     edx, edx                             
0045A1AE  |.  8955 D0            mov     dword ptr [ebp-30], edx              
0045A1B1  |.  8D55 D0            lea     edx, dword ptr [ebp-30]              
0045A1B4  |.  FF43 1C            inc     dword ptr [ebx+1C]                   
0045A1B7  |.  E8 3CD30900        call    004F74F8                             
0045A1BC  |.  8D45 D0            lea     eax, dword ptr [ebp-30]              
0045A1BF  |.  50                 push    eax                                  
0045A1C0  |.  BA CF885200        mov     edx, 005288CF                        
0045A1C5  |.  8D45 CC            lea     eax, dword ptr [ebp-34]              
0045A1C8  |.  E8 4FCE0900        call    004F701C                             ;  依次检查比较文本框内字符 ASCII 值  
0045A1CD  |.  FF43 1C            inc     dword ptr [ebx+1C]                   ;  反复检查  
0045A1D0  |.  8D55 CC            lea     edx, dword ptr [ebp-34]              
0045A1D3  |.  58                 pop     eax                                  
0045A1D4  |.  E8 EBD10900        call    004F73C4                             
0045A1D9  |.  50                 push    eax                                  
0045A1DA  |.  FF4B 1C            dec     dword ptr [ebx+1C]                   
0045A1DD  |.  8D45 CC            lea     eax, dword ptr [ebp-34]              
```



```
>   0045A1E0  |.  BA 02000000        mov     edx, 2  

>   0045A1E5  |.  E8 D6CF0900        call    004F71C0  

>   0045A1EA  |.  FF4B 1C            dec     dword ptr [ebx+1C]  

>   0045A1ED  |.  8D45 D0            lea     eax, dword ptr [ebp-30]  

>   0045A1F0  |.  BA 02000000        mov     edx, 2  

>   0045A1F5  |.  E8 C6CF0900        call    004F71C0  

>   0045A1FA  |.  FF4B 1C            dec     dword ptr [ebx+1C]                 ; |  

>   0045A1FD  |.  8D45 D4            lea     eax, dword ptr [ebp-2C]           ; |  

>   0045A200  |.  BA 02000000        mov     edx, 2                            ; |  

>   0045A205  |.  E8 B6CF0900        call    004F71C0                          ; \expert.004F71C0  

>   0045A20A  |.  59                 pop     ecx  

>   0045A20B  |.  85C9               test    ecx, ecx                          ;  比较 ECX 中的值是否为 0  

>   0045A20D  |.  74 53              je      short 0045A262                   ;  不为 0 则跳  

>   0045A20F  |.  66:C743 10 A400    mov     word ptr [ebx+10], 0A4  

>   0045A215  |.  BA D1885200        mov     edx, 005288D1                    ;  电子邮件地址中不能存在空格！  

>   0045A21A  |.  8D45 C8            lea     eax, dword ptr [ebp-38]  

>   0045A21D  |.  E8 FACD0900        call    004F701C  

>   0045A222  |.  FF43 1C            inc     dword ptr [ebx+1C]  

>   0045A225  |.  8D55 C8            lea     edx, dword ptr [ebp-38]  

>   0045A228  |.  8B45 08            mov     eax, dword ptr [ebp+8]  

>   0045A22B  |.  E8 C0CF0900        call    004F71F0  

>   0045A230  |.  8B45 08            mov     eax, dword ptr [ebp+8]  

>   0045A233  |.  BA 02000000        mov     edx, 2  

>   0045A238  |.  66:C743 10 B000    mov     word ptr [ebx+10], 0B0  

>   0045A23E  |.  50                 push    eax  

>   0045A23F  |.  8D45 C8            lea     eax, dword ptr [ebp-38]  

>   0045A242  |.  FF4B 1C            dec     dword ptr [ebx+1C]  

>   0045A245  |.  E8 76CF0900        call    004F71C0  

>   0045A24A  |.  58                 pop     eax  

>   0045A24B  |.  66:C743 10 A400    mov     word ptr [ebx+10], 0A4  

>   0045A251  |.  FF43 1C            inc     dword ptr [ebx+1C]  

>   0045A254  |.  8B13               mov     edx, dword ptr [ebx]  

>   0045A256  |.  64:8915 00000000   mov     dword ptr fs:[0], edx  

>   0045A25D  |.  E9 EE030000        jmp     0045A650  

>   0045A262  |> \66:C743 10 BC00    mov     word ptr [ebx+10], 0BC          ;  验证电子邮件的合法性  

>   0045A268  |.  33C9               xor     ecx, ecx  

>   0045A26A  |.  894D C4            mov     dword ptr [ebp-3C], ecx  

>   0045A26D  |.  8D55 C4            lea     edx, dword ptr [ebp-3C]  

>   0045A270  |.  FF43 1C            inc     dword ptr [ebx+1C]  

>   0045A273  |.  8B86 F0020000      mov     eax, dword ptr [esi+2F0]  

>   0045A279  |.  E8 641E0B00        call    <jmp.&VCL50.Controls::TControl::>;  命令查找文本框中的文本  

>   0045A27E  |.  8D4D C4            lea     ecx, dword ptr [ebp-3C]  

>   0045A281  |.  51                 push    ecx  

>   0045A282  |.  BA EE885200        mov     edx, 005288EE                    ;  检查是否文本中有固定字符“@”  

>   0045A287  |.  8D45 C0            lea     eax, dword ptr [ebp-40]  

>   0045A28A  |.  E8 8DCD0900        call    004F701C                        ;  依次检查比较文本框内字符 ASCII 值  

>   0045A28F  |.  FF43 1C            inc     dword ptr [ebx+1C]               ;  反复检查  

>   0045A292  |.  8D55 C0            lea     edx, dword ptr [ebp-40]  

>   0045A295  |.  58                 pop     eax  

>   0045A296  |.  E8 29D10900        call    004F73C4  

>   0045A29B  |.  85C0               test    eax, eax  

>   0045A29D  |.  8D45 C0            lea     eax, dword ptr [ebp-40]  

>   0045A2A0  |.  0F94C1             sete    cl  

>   0045A2A3  |.  83E1 01            and     ecx, 1  

>   0045A2A6  |.  BA 02000000        mov     edx, 2  

>   0045A2AB  |.  51                 push    ecx  

>   0045A2AC  |.  FF4B 1C            dec     dword ptr [ebx+1C]  

>   0045A2AF  |.  E8 0CCF0900        call    004F71C0  

>   0045A2B4  |.  FF4B 1C            dec     dword ptr [ebx+1C]               ; |  

>   0045A2B7  |.  8D45 C4            lea     eax, dword ptr [ebp-3C]          ; |  

>   0045A2BA  |.  BA 02000000        mov     edx, 2                           ; |  

>   0045A2BF  |.  E8 FCCE0900        call    004F71C0                          ; \expert.004F71C0  

>   0045A2C4  |.  59                 pop     ecx  

>   0045A2C5  |.  84C9               test    cl, cl                           ;  比较 CL 中的值是否为 0  

>   0045A2C7  |.  74 53              je      short 0045A31C                   ;  不为 0 则跳  

>   0045A2C9  |.  66:C743 10 C800    mov     word ptr [ebx+10], 0C8  

>   0045A2CF  |.  BA F0885200        mov     edx, 005288F0                    ;  电子邮件地址不正确！  

>   0045A2D4  |.  8D45 BC            lea     eax, dword ptr [ebp-44]  

>   0045A2D7  |.  E8 40CD0900        call    004F701C  

>   0045A2DC  |.  FF43 1C            inc     dword ptr [ebx+1C]  

>   0045A2DF  |.  8D55 BC            lea     edx, dword ptr [ebp-44]  

>   0045A2E2  |.  8B45 08            mov     eax, dword ptr [ebp+8]  

>   0045A2E5  |.  E8 06CF0900        call    004F71F0  

>   0045A2EA  |.  8B45 08            mov     eax, dword ptr [ebp+8]  

>   0045A2ED  |.  BA 02000000        mov     edx, 2  

>   0045A2F2  |.  66:C743 10 D400    mov     word ptr [ebx+10], 0D4  

>   0045A2F8  |.  50                 push    eax  

>   0045A2F9  |.  8D45 BC            lea     eax, dword ptr [ebp-44]  

>   0045A2FC  |.  FF4B 1C            dec     dword ptr [ebx+1C]  

>   0045A2FF  |.  E8 BCCE0900        call    004F71C0  

>   0045A304  |.  58                 pop     eax  

>   0045A305  |.  66:C743 10 C800    mov     word ptr [ebx+10], 0C8  

>   0045A30B  |.  FF43 1C            inc     dword ptr [ebx+1C]  

>   0045A30E  |.  8B13               mov     edx, dword ptr [ebx]  

>   0045A310  |.  64:8915 00000000   mov     dword ptr fs:[0], edx  

>   0045A317  |.  E9 34030000        jmp     0045A650  

>   0045A31C  |> \66:C743 10 E000    mov     word ptr [ebx+10], 0E0          ;  检查注册密码的合法性  

>   0045A322  |.  33C9               xor     ecx, ecx  

>   0045A324  |.  894D B8            mov     dword ptr [ebp-48], ecx  

>   0045A327  |.  8D55 B8            lea     edx, dword ptr [ebp-48]  

>   0045A32A  |.  FF43 1C            inc     dword ptr [ebx+1C]  

>   0045A32D  |.  8B86 F4020000      mov     eax, dword ptr [esi+2F4]  

>   0045A333  |.  E8 AA1D0B00        call    <jmp.&VCL50.Controls::TControl::>;  命令查找文本框中的文本  

>   0045A338  |.  8D45 B8            lea     eax, dword ptr [ebp-48]          ;  计算电子邮件长度，便于 0045A357 处检查  

>   0045A33B  |.  33D2               xor     edx, edx  

>   0045A33D  |.  8955 B4            mov     dword ptr [ebp-4C], edx  

>   0045A340  |.  8D55 B4            lea     edx, dword ptr [ebp-4C]  

>   0045A343  |.  FF43 1C            inc     dword ptr [ebx+1C]  

>   0045A346  |.  E8 ADD10900        call    004F74F8  

>   0045A34B  |.  8D45 B4            lea     eax, dword ptr [ebp-4C]  

>   0045A34E  |.  50                 push    eax  

>   0045A34F  |.  BA 05895200        mov     edx, 00528905  

>   0045A354  |.  8D45 B0            lea     eax, dword ptr [ebp-50]  

>   0045A357  |.  E8 C0CC0900        call    004F701C                        ;  依次检查比较文本框内字符 ASCII 值  

>   0045A35C  |.  FF43 1C            inc     dword ptr [ebx+1C]               ;  反复检查  

>   0045A35F  |.  8D55 B0            lea     edx, dword ptr [ebp-50]  

>   0045A362  |.  58                 pop     eax  

>   0045A363  |.  E8 5CD00900        call    004F73C4  

>   0045A368  |.  50                 push    eax  

>   0045A369  |.  FF4B 1C            dec     dword ptr [ebx+1C]  

>   0045A36C  |.  8D45 B0            lea     eax, dword ptr [ebp-50]  

>   0045A36F  |.  BA 02000000        mov     edx, 2  
```



```
  0045A374  |.  E8 47CE0900        call    004F71C0  
  0045A379  |.  FF4B 1C           dec     dword ptr [ebx+1C]  
  0045A37C  |.  8D45 B4          lea     eax, dword ptr [ebp-4C]  
  0045A37F  |.  BA 02000000      mov     edx, 2  
  0045A384  |.  E8 37CE0900      call    004F71C0  
  0045A389  |.  FF4B 1C          dec     dword ptr [ebx+1C]               ; |  
  0045A38C  |.  8D45 B8          lea     eax, dword ptr [ebp-48]         ; |  
  0045A38F  |.  BA 02000000      mov     edx, 2                          ; |  
  0045A394  |.  E8 27CE0900      call    004F71C0                        ; \\expert.004F71C0  
  0045A399  |.  59               pop     ecx  
  0045A39A  |.  85C9             test    ecx, ecx                        ;  比较 ECX 中的值是否为 0  
  0045A39C  |.  74 53            je      short 0045A3F1                  ;  不为 0 则跳  
  0045A39E  |.  66:C743 10 EC00  mov     word ptr [ebx+10], 0EC  
  0045A3A4  |.  BA 07895200      mov     edx, 00528907                   ;  输入密码中不能存在空格！  
  0045A3A9  |.  8D45 AC          lea     eax, dword ptr [ebp-54]  
  0045A3AC  |.  E8 6BCC0900      call    004F701C  
  0045A3B1  |.  FF43 1C          inc     dword ptr [ebx+1C]  
  0045A3B4  |.  8D55 AC          lea     edx, dword ptr [ebp-54]  
  0045A3B7  |.  8B45 08          mov     eax, dword ptr [ebp+8]  
  0045A3BA  |.  E8 31CE0900      call    004F71F0  
  0045A3BF  |.  8B45 08          mov     eax, dword ptr [ebp+8]  
  0045A3C2  |.  BA 02000000      mov     edx, 2  
  0045A3C7  |.  66:C743 10 F800  mov     word ptr [ebx+10], 0F8  
  0045A3CD  |.  50               push    eax  
  0045A3CE  |.  8D45 AC          lea     eax, dword ptr [ebp-54]  
  0045A3D1  |.  FF4B 1C          dec     dword ptr [ebx+1C]  
  0045A3D4  |.  E8 E7CD0900      call    004F71C0  
  0045A3D9  |.  58               pop     eax  
  0045A3DA  |.  66:C743 10 EC00  mov     word ptr [ebx+10], 0EC  
  0045A3E0  |.  FF43 1C          inc     dword ptr [ebx+1C]  
  0045A3E3  |.  8B13             mov     edx, dword ptr [ebx]  
  0045A3E5  |.  64:8915 00000000 mov     dword ptr fs:[0], edx  
  0045A3EC  |.  E9 5F020000      jmp     0045A650  
  0045A3F1  |>  \\66:C743 10 0401  mov     word ptr [ebx+10], 104               ;  验证注册密码的合法性  
  0045A3F7  |.  33C9             xor     ecx, ecx  
  0045A3F9  |.  894D A8          mov     dword ptr [ebp-58], ecx  
  0045A3FC  |.  8D55 A8          lea     edx, dword ptr [ebp-58]  
  0045A3FF  |.  FF43 1C          inc     dword ptr [ebx+1C]  
  0045A402  |.  8B86 F4020000    mov     eax, dword ptr [esi+2F4]  
  0045A408  |.  E8 D51C0B00      call    <jmp.&VCL50.Controls::TControl::>;  命令查找文本框中的文本  
  0045A40D  |.  8D45 A8          lea     eax, dword ptr [ebp-58]         ;  计算得到注册密码长度  
  0045A410  |.  33D2             xor     edx, edx  
  0045A412  |.  8955 A4          mov     dword ptr [ebp-5C], edx  
  0045A415  |.  8D55 A4          lea     edx, dword ptr [ebp-5C]  
  0045A418  |.  FF43 1C          inc     dword ptr [ebx+1C]  
  0045A41B  |.  E8 D8D00900      call    004F74F8  
  0045A420  |.  837D A4 00       cmp     dword ptr [ebp-5C], 0           ;  验证是否存在注册密码  
  0045A424  |.  74 08            je      short 0045A42E                  ;  有，则直接去验证有效性  
  0045A426  |.  8B45 A4          mov     eax, dword ptr [ebp-5C]         ;  取注册密码  
  0045A429  |.  8B48 FC          mov     ecx, dword ptr [eax-4]          ;  得到密码长度  
  0045A42C  |.  EB 02            jmp     short 0045A430  
  0045A42E  |>   33C9            xor     ecx, ecx  
  0045A430  |>   83F9 10        cmp     ecx, 10  
  0045A433  |.  BA 02000000      mov     edx, 2  
  0045A438  |.  0F95C0           setne   al  
  0045A43B  |.  83E0 01          and     eax, 1  
  0045A43E  |.  50               push    eax  
  0045A43F  |.  8D45 A4          lea     eax, dword ptr [ebp-5C]  
  0045A442  |.  FF4B 1C          dec     dword ptr [ebx+1C]  
  0045A445  |.  E8 76CD0900      call    004F71C0                        ;  这里检查注册密码中是否全为阿拉伯数字  
  0045A44A  |.  FF4B 1C          dec     dword ptr [ebx+1C]             ; |  
  0045A44D  |.  8D45 A8          lea     eax, dword ptr [ebp-58]         ; |  
  0045A450  |.  BA 02000000      mov     edx, 2                        ; |  
  0045A455  |.  E8 66CD0900      call    004F71C0                        ; \\expert.004F71C0  
  0045A45A  |.  59               pop     ecx  
  0045A45B  |.  84C9             test    cl, cl                          ;  比较 CL 中的值是否为 0  
  0045A45D  |.  74 53            je      short 0045A4B2                  ;  不为 0 则跳  
  0045A45F  |.  66:C743 10 1001  mov     word ptr [ebx+10], 110  
  0045A465  |.  BA 20895200      mov     edx, 00528920                   ;  输入密码不足 16 位！  
  0045A46A  |.  8D45 A0          lea     eax, dword ptr [ebp-60]  
  0045A46D  |.  E8 AACB0900      call    004F701C  
  0045A472  |.  FF43 1C          inc     dword ptr [ebx+1C]  
  0045A475  |.  8D55 A0          lea     edx, dword ptr [ebp-60]  
  0045A478  |.  8B45 08          mov     eax, dword ptr [ebp+8]  
  0045A47B  |.  E8 70CD0900      call    004F71F0  
  0045A480  |.  8B45 08          mov     eax, dword ptr [ebp+8]  
  0045A483  |.  BA 02000000      mov     edx, 2  
  0045A488  |.  66:C743 10 1C01  mov     word ptr [ebx+10], 11C  
  0045A48E  |.  50               push    eax  
  0045A48F  |.  8D45 A0          lea     eax, dword ptr [ebp-60]  
  0045A492  |.  FF4B 1C          dec     dword ptr [ebx+1C]  
  0045A495  |.  E8 26CD0900      call    004F71C0  
  0045A49A  |.  58               pop     eax  
  0045A49B  |.  66:C743 10 1001  mov     word ptr [ebx+10], 110  
  0045A4A1  |.  FF43 1C          inc     dword ptr [ebx+1C]  
  0045A4A4  |.  8B13             mov     edx, dword ptr [ebx]  
  0045A4A6  |.  64:8915 00000000 mov     dword ptr fs:[0], edx  
  0045A4AD  |.  E9 9E010000      jmp     0045A650  
  0045A4B2  |>  \\66:C743 10 3401  mov     word ptr [ebx+10], 134               ;  最后的检查用户信息  
  0045A4B8  |.  33C9             xor     ecx, ecx  
  0045A4BA  |.  894D 9C          mov     dword ptr [ebp-64], ecx  
  0045A4BD  |.  8D55 9C          lea     edx, dword ptr [ebp-64]  
  0045A4C0  |.  FF43 1C          inc     dword ptr [ebx+1C]  
  0045A4C3  |.  8B86 F4020000    mov     eax, dword ptr [esi+2F4]  
  0045A4C9  |.  E8 141C0B00      call    <jmp.&VCL50.Controls::TControl::>;  命令查找文本框中的文本  
  0045A4CE  |.  8D45 9C          lea     eax, dword ptr [ebp-64]         ;  计算得到注册密码长度  
  0045A4D1  |.  33D2             xor     edx, edx  
  0045A4D3  |.  8955 98          mov     dword ptr [ebp-68], edx  
  0045A4D6  |.  8D55 98          lea     edx, dword ptr [ebp-68]  
  0045A4D9  |.  FF43 1C          inc     dword ptr [ebx+1C]  
  0045A4DC  |.  E8 17D00900      call    004F74F8  
  0045A4E1  |.  837D 98 00       cmp     dword ptr [ebp-68], 0           ;  ASCII "1234567890123456"  
  0045A4E5  |.  74 05            je      short 0045A4EC  
  0045A4E7  |.  8B7D 98          mov     edi, dword ptr [ebp-68]         ;  ASCII "1234567890123456"  
  0045A4EA  |.  EB 05            jmp     short 0045A4F1  
  0045A4EC  |>   BF 33895200      mov     edi, 00528933  
  0045A4F1  |>   FF4B 1C          dec     dword ptr [ebx+1C]  
  0045A4F4  |.  8D45 98          lea     eax, dword ptr [ebp-68]  
  0045A4F7  |.  BA 02000000      mov     edx, 2  
  0045A4FC  |.  E8 BFCC0900      call    004F71C0  
```



```
0045A501  |.  FF4B 1C             dec     dword ptr [ebx+1C]
0045A504  |.  8D45 9C             lea     eax, dword ptr [ebp-64]
0045A507  |.  BA 02000000         mov     edx, 2
0045A50C  |.  E8 AFCC0900         call    004F71C0
0045A511  |.  66:C743 10 2801     mov     word ptr [ebx+10], 128
0045A517  |.  33C9                xor     ecx, ecx
0045A519  |.  898D 60FFFFFF       mov     dword ptr [ebp-A0], ecx
0045A51F  |.  8BC7                mov     eax, edi
0045A521  |.  8BF8                mov     edi, eax
0045A523  |.  EB 68               jmp     short 0045A58D
0045A525  |>  /0FBE17             /movsx   edx, byte ptr [edi]             ;  依次传入注册密码的 ASCII 值
0045A528  |.  |52                 |push    edx                              ; /c
0045A529  |.  |E8 E2230B00        |call    <jmp.&CC3250MT._isxdigit>       ; \_isxdigit
0045A52E  |.  |59                 |pop     ecx
0045A52F  |.  |85C0              |test    eax, eax
0045A531  |.  |75 53             |jnz     short 0045A586
0045A533  |.  |66:C743 10 4C01   |mov     word ptr [ebx+10], 14C
0045A539  |.  |BA 34895200       |mov     edx, 00528934                    ;  输入密码不正确！
0045A53E  |.  |8D45 8C           |lea     eax, dword ptr [ebp-74]
0045A541  |.  |E8 D6CA0900       |call    004F701C
0045A546  |.  |FF43 1C          |inc     dword ptr [ebx+1C]
0045A549  |.  |8D55 8C           |lea     edx, dword ptr [ebp-74]
0045A54C  |.  |8B45 08           |mov     eax, dword ptr [ebp+8]
0045A54F  |.  |E8 9CCC0900       |call    004F71F0
0045A554  |.  |8B45 08           |mov     eax, dword ptr [ebp+8]
0045A557  |.  |BA 02000000       |mov     edx, 2
0045A55C  |.  |66:C743 10 5801   |mov     word ptr [ebx+10], 158
0045A562  |.  |50                 |push    eax
0045A563  |.  |8D45 8C           |lea     eax, dword ptr [ebp-74]
0045A566  |.  |FF4B 1C          |dec     dword ptr [ebx+1C]
0045A569  |.  |E8 52CC0900       |call    004F71C0
0045A56E  |.  |58                 |pop     eax
0045A56F  |.  |66:C743 10 4C01   |mov     word ptr [ebx+10], 14C
0045A575  |.  |FF43 1C          |inc     dword ptr [ebx+1C]
0045A578  |.  |8B13              |mov     edx, dword ptr [ebx]
0045A57A  |.  |64:8915 00000000  |mov     dword ptr fs:[0], edx
0045A581  |.  |E9 CA000000       |jmp     0045A650
0045A586  |>  |FF85 60FFFFFF     |inc     dword ptr [ebp-A0]
0045A58C  |.  |47                 |inc     edi
0045A58D  |>  |66:C743 10 4000    mov     word ptr [ebx+10], 140
0045A593  |.  |33C9              |xor     ecx, ecx
0045A595  |.  |894D 94           |mov     dword ptr [ebp-6C], ecx
0045A598  |.  |8D55 94           |lea     edx, dword ptr [ebp-6C]
0045A59B  |.  |FF43 1C          |inc     dword ptr [ebx+1C]
0045A59E  |.  |8B86 F4020000     |mov     eax, dword ptr [esi+2F4]        ;  注册密码进栈，准备计算验证
0045A5A4  |.  |E8 391B0B00       |call    <jmp.&VCL50.Controls::TControl:>
0045A5A9  |.  |8D45 94           |lea     eax, dword ptr [ebp-6C]        ;  计算注册密码长度
0045A5AC  |.  |33D2              |xor     edx, edx
0045A5AE  |.  |8955 90           |mov     dword ptr [ebp-70], edx
0045A5B1  |.  |8D55 90           |lea     edx, dword ptr [ebp-70]
0045A5B4  |.  |FF43 1C          |inc     dword ptr [ebx+1C]
0045A5B7  |.  |E8 3CCF0900       |call    004F74F8
0045A5BC  |.  |837D 90 00        |cmp     dword ptr [ebp-70], 0
0045A5C0  |.  |74 08             |je      short 0045A5CA
0045A5C2  |.  |8B45 90           |mov     eax, dword ptr [ebp-70]
0045A5C5  |.  |8B48 FC           |mov     ecx, dword ptr [eax-4]
0045A5C8  |.  |EB 02             |jmp     short 0045A5CC
0045A5CA  |>  |33C9              |xor     ecx, ecx
0045A5CC  |>  |3B8D 60FFFFFF     |cmp     ecx, dword ptr [ebp-A0]
0045A5D2  |.  |BA 02000000       |mov     edx, 2
0045A5D7  |.  |0F9FC0            |setg    al
0045A5DA  |.  |83E0 01           |and     eax, 1
0045A5DD  |.  |50                 |push    eax
0045A5DE  |.  |8D45 90           |lea     eax, dword ptr [ebp-70]
0045A5E1  |.  |FF4B 1C          |dec     dword ptr [ebx+1C]
0045A5E4  |.  |E8 D7CB0900       |call    004F71C0
0045A5E9  |.  |FF4B 1C          |dec     dword ptr [ebx+1C]              ; |
0045A5EC  |.  |8D45 94           |lea     eax, dword ptr [ebp-6C]        ; |
0045A5EF  |.  |BA 02000000       |mov     edx, 2                         ; |
0045A5F4  |.  |E8 C7CB0900       |call    004F71C0                       ; \expert.004F71C0
0045A5F9  |.  |59                 |pop     ecx
0045A5FA  |.  |85C9              |test    ecx, ecx
0045A5FC  |.^\\0F85 23FFFFFF     \\jnz     0045A525
0045A602  |.  66:C743 10 6401    mov     word ptr [ebx+10], 164
0045A608  |.  BA 45895200        mov     edx, 00528945
0045A60D  |.  8D45 88            lea     eax, dword ptr [ebp-78]
0045A610  |.  E8 07CA0900        call    004F701C
0045A615  |.  FF43 1C           inc     dword ptr [ebx+1C]
0045A618  |.  8D55 88            lea     edx, dword ptr [ebp-78]
0045A61B  |.  8B45 08            mov     eax, dword ptr [ebp+8]
0045A61E  |.  E8 CDCB0900        call    004F71F0
0045A623  |.  8B45 08            mov     eax, dword ptr [ebp+8]
0045A626  |.  BA 02000000        mov     edx, 2
0045A62B  |.  66:C743 10 7001    mov     word ptr [ebx+10], 170
0045A631  |.  50                  push    eax
0045A632  |.  8D45 88            lea     eax, dword ptr [ebp-78]
0045A635  |.  FF4B 1C            dec     dword ptr [ebx+1C]
0045A638  |.  E8 83CB0900        call    004F71C0
0045A63D  |.  58                  pop     eax
0045A63E  |.  66:C743 10 6401    mov     word ptr [ebx+10], 164
0045A644  |.  FF43 1C            inc     dword ptr [ebx+1C]
0045A647  |.  8B13               mov     edx, dword ptr [ebx]
0045A649  |.  64:8915 00000000   mov     dword ptr fs:[0], edx
0045A650  |>  5F                  pop     edi
0045A651  |.  5E                  pop     esi
0045A652  |.  5B                  pop     ebx
0045A653  |.  8BE5               mov     esp, ebp
0045A655  |.  5D                  pop     ebp
0045A656  \\.  C3                 retn                               ;  返回后继续跟进

返回到：

00458A63  |.  66:C743 10 0800    mov     word ptr [ebx+10], 8        ;  返回到这里
00458A69  |.  66:C743 10 2000    mov     word ptr [ebx+10], 20
00458A6F  |.  83C4 08            add     esp, 8
00458A72  |.  8BD7               mov     edx, edi
00458A74  |.  8D45 EC            lea     eax, dword ptr [ebp-14]
00458A77  |.  E8 A0E50900        call    004F701C
00458A7C  |.  FF43 1C            inc     dword ptr [ebx+1C]
00458A7F  |.  8D55 EC            lea     edx, dword ptr [ebp-14]
00458A82  |.  8D45 FC            lea     eax, dword ptr [ebp-4]
00458A85  |.  E8 1EE80900        call    004F72A8
00458A8A  |.  50                  push    eax                          ; /Arg1
00458A8B  |.  FF4B 1C            dec     dword ptr [ebx+1C]           ; |
00458A8E  |.  8D45 EC            lea     eax, dword ptr [ebp-14]       ; |
00458A91  |.  BA 02000000        mov     edx, 2                       ; |
00458A96  |.  E8 25E70900        call    004F71C0                     ; \expert.004F71C0
00458A9B  |.  59                  pop     ecx
```



```
00458A9C  |.  84C9               test    cl, cl  
00458A9E  |.  74 3D              je      short 00458ADD  
00458AA0  |.  6A 30              push    30  
00458AA2  |.  8D4F 01            lea     ecx, dword ptr [edi+1]  
00458AA5  |.  837D FC 00         cmp     dword ptr [ebp-4], 0  
00458AA9  |.  74 05              je      short 00458AB0  
00458AAB  |.  8B55 FC            mov     edx, dword ptr [ebp-4]  
00458AAE  |.  EB 03              jmp     short 00458AB3  
00458AB0  |>  8D57 06            lea     edx, dword ptr [edi+6]  
00458AB3  |>  A1 80DF5500        mov     eax, dword ptr [<&VCL50.Forms::A>  
00458AB8  |.  8B00               mov     eax, dword ptr [eax]  
00458ABA  |.  E8 09E50900        call    004F6FC8  
00458ABF  |.  FF4B 1C            dec     dword ptr [ebx+1C]  
00458AC2  |.  8D45 FC            lea     eax, dword ptr [ebp-4]  
00458AC5  |.  BA 02000000        mov     edx, 2  
00458ACA  |.  E8 F1E60900        call    004F71C0  
00458ACF  |.  8B0B               mov     ecx, dword ptr [ebx]  
00458AD1  |.  64:890D 00000000   mov     dword ptr fs:[0], ecx  
00458AD8  |.  E9 880C0000        jmp     00459765  
00458ADD  |>  66:C743 10 2C00    mov     word ptr [ebx+10], 2C  
00458AE3  |.  33C0               xor     eax, eax  
00458AE5  |.  8945 E8            mov     dword ptr [ebp-18], eax  
00458AE8  |.  8D55 E8            lea     edx, dword ptr [ebp-18]  
00458AEB  |.  FF43 1C            inc     dword ptr [ebx+1C]  
00458AEE  |.  8B86 F4020000      mov     eax, dword ptr [esi+2F4]  
00458AF4  |.  E8 E9350B00        call    <jmp.&VCL50.Controls::TControl::>  
00458AF9  |.  8D45 E8            lea     eax, dword ptr [ebp-18]  
00458AFC  |.  33D2               xor     edx, edx  
00458AFE  |.  8955 E4            mov     dword ptr [ebp-1C], edx  
00458B01  |.  8D55 E4            lea     edx, dword ptr [ebp-1C]  
00458B04  |.  FF43 1C            inc     dword ptr [ebx+1C]  
00458B07  |.  E8 ECE90900        call    004F74F8  
00458B0C  |.  8D45 E4            lea     eax, dword ptr [ebp-1C]  
00458B0F  |.  33C9               xor     ecx, ecx  
00458B11  |.  894D F8            mov     dword ptr [ebp-8], ecx  
00458B14  |.  8D55 F8            lea     edx, dword ptr [ebp-8]  
00458B17  |.  FF43 1C            inc     dword ptr [ebx+1C]  
00458B1A  |.  E8 51E90900        call    004F7470  
00458B1F  |.  FF4B 1C            dec     dword ptr [ebx+1C]  
00458B22  |.  8D45 E4            lea     eax, dword ptr [ebp-1C]  
00458B25  |.  BA 02000000        mov     edx, 2  
00458B2A  |.  E8 91E60900        call    004F71C0  
00458B2F  |.  FF4B 1C            dec     dword ptr [ebx+1C]  
00458B32  |.  8D45 E8            lea     eax, dword ptr [ebp-18]  
00458B35  |.  BA 02000000        mov     edx, 2  
00458B3A  |.  E8 81E60900        call    004F71C0  
00458B3F  |.  66:C743 10 0800    mov     word ptr [ebx+10], 8  
00458B45  |.  837D F8 00         cmp     dword ptr [ebp-8], 0  
00458B49  |.  74 05              je      short 00458B50  
00458B4B  |.  8B45 F8            mov     eax, dword ptr [ebp-8]  
00458B4E  |.  EB 03              jmp     short 00458B53  
00458B50  |>  8D47 07            lea     eax, dword ptr [edi+7]  
00458B53  |>  50                 push    eax                             ; /Arg1  
00458B54  |.  E8 6BC7FCFF        call    004252C4                         ; \\注册码计算 CALL，跟进  
00458B59  |.  59                 pop     ecx  
00458B5A  |.  83F8 09            cmp     eax, 9                          ;  如果 EAX 为 9，那么继续验证更高的版本  
00458B5D  |.  75 42              jnz     short 00458BA1                  ;  EAX 不为 9 则直接注册失败，必须跳  
                                                            ;  试用版注册识别为 9  
00458B5F  |.  A1 80DF5500        mov     eax, dword ptr [<&VCL50.Forms::A>  
00458B64  |.  6A 30              push    30  
00458B66  |.  8D4F 19            lea     ecx, dword ptr [edi+19]  
00458B69  |.  8D57 08            lea     edx, dword ptr [edi+8]  
00458B6C  |.  8B00               mov     eax, dword ptr [eax]  
00458B6E  |.  E8 55E40900        call    004F6FC8  
00458B73  |.  FF4B 1C            dec     dword ptr [ebx+1C]  
00458B76  |.  8D45 F8            lea     eax, dword ptr [ebp-8]  
00458B79  |.  BA 02000000        mov     edx, 2  
00458B7E  |.  E8 3DE60900        call    004F71C0  
00458B83  |.  FF4B 1C            dec     dword ptr [ebx+1C]  
00458B86  |.  8D45 FC            lea     eax, dword ptr [ebp-4]  
00458B89  |.  BA 02000000        mov     edx, 2  
00458B8E  |.  E8 2DE60900        call    004F71C0  
00458B93  |.  8B0B               mov     ecx, dword ptr [ebx]  
00458B95  |.  64:890D 00000000   mov     dword ptr fs:[0], ecx  
00458B9C  |.  E9 C40B0000        jmp     00459765                         ;  直接返回失败信息  
00458BA1  |>  66:C743 10 3800    mov     word ptr [ebx+10], 38           ;  检查用户版本类型  
00458BA7  |.  8D57 1E            lea     edx, dword ptr [edi+1E]  
00458BAA  |.  8D45 E0            lea     eax, dword ptr [ebp-20]  
00458BAD  |.  E8 6AE40900        call    004F701C  
00458BB2  |.  FF43 1C            inc     dword ptr [ebx+1C]  
00458BB5  |.  8D55 DC            lea     edx, dword ptr [ebp-24]  
00458BB8  |.  8B08               mov     ecx, dword ptr [eax]  
00458BBA  |.  33C0               xor     eax, eax  
00458BBC  |.  51                 push    ecx                             ; /Arg2  
00458BBD  |.  8945 DC            mov     dword ptr [ebp-24], eax         ; |  
00458BC0  |.  52                 push    edx                             ; |Arg1  
00458BC1  |.  FF43 1C            inc     dword ptr [ebx+1C]              ; |  
00458BC4  |.  E8 D3A6FCFF        call    0042329C                         ; \\expert.0042329C  
00458BC9  |.  83C4 08            add     esp, 8  
00458BCC  |.  8D45 DC            lea     eax, dword ptr [ebp-24]  
00458BCF  |.  33D2               xor     edx, edx  
00458BD1  |.  E8 66EB0900        call    004F773C  
00458BD6  |.  50                 push    eax                             ; /Arg3  
00458BD7  |.  6A 21              push    21                              ; |Arg2 = 00000021  
00458BD9  |.  6A 07              push    7                               ; |Arg1 = 00000007  
00458BDB  |.  E8 D4B2FCFF        call    00423EB4                         ; \\expert.00423EB4  
00458BE0  |.  83C4 0C            add     esp, 0C  
00458BE3  |.  8985 ECFEFFFF      mov     dword ptr [ebp-114], eax  
00458BE9  |.  FF4B 1C            dec     dword ptr [ebx+1C]  
00458BEC  |.  8D45 DC            lea     eax, dword ptr [ebp-24]  
00458BEF  |.  BA 02000000        mov     edx, 2  
00458BF4  |.  E8 C7E50900        call    004F71C0  
00458BF9  |.  FF4B 1C            dec     dword ptr [ebx+1C]  
00458BFC  |.  8D45 E0            lea     eax, dword ptr [ebp-20]  
00458BFF  |.  BA 02000000        mov     edx, 2  
00458C04  |.  E8 B7E50900        call    004F71C0  
00458C09  |.  66:C743 10 0800    mov     word ptr [ebx+10], 8  
00458C0F  |.  837D F8 00         cmp     dword ptr [ebp-8], 0  
00458C13  |.  74 05              je      short 00458C1A  
00458C15  |.  8B4D F8            mov     ecx, dword ptr [ebp-8]  
00458C18  |.  EB 03              jmp     short 00458C1D  
00458C1A  |>  8D4F 27            lea     ecx, dword ptr [edi+27]  
00458C1D  |>  51                 push    ecx                             ; /Arg1  
```



```asm
   00458C1E  |.  E8 A1C6FCFF        call    004252C4                   ; \expert.004252C4
   00458C23  |.  59                 pop     ecx
   00458C24  |.  8985 E8FEFFFF      mov     dword ptr [ebp-118], eax
   00458C2A  |.  83BD ECFEFFFF 00   cmp     dword ptr [ebp-114], 0
   00458C31  |.  74 69              je      short 00458C9C
   00458C33  |.  8B85 ECFEFFFF      mov     eax, dword ptr [ebp-114]
   00458C39  |.  3B85 E8FEFFFF      cmp     eax, dword ptr [ebp-118]
   00458C3F  |.  7C 5B              jl      short 00458C9C             ; 这里暂时不能跳
   00458C41  |.  E8 0EB6FCFF        call    00424254
   00458C46  |.  99                 cdq
   00458C47  |.  F7BD ECFEFFFF      idiv    dword ptr [ebp-114]
   00458C4D  |.  85D2               test    edx, edx
   00458C4F  |.  75 4B              jnz     short 00458C9C             ; 这里暂时不能跳
   00458C51  |.  83BD ECFEFFFF 06   cmp     dword ptr [ebp-114], 6
   00458C58  |.  74 42              je      short 00458C9C             ; 这里必须跳了
   00458C5A  |.  A1 80DF5500        mov     eax, dword ptr [<&VCL50.Forms::A>
   00458C5F  |.  6A 40              push    40
   00458C61  |.  8D4F 58            lea     ecx, dword ptr [edi+58]
   00458C64  |.  8D57 28            lea     edx, dword ptr [edi+28]
   00458C67  |.  8B00               mov     eax, dword ptr [eax]
   00458C69  |.  E8 5AE30900        call    004F6FC8
   00458C6E  |.  FF4B 1C            dec     dword ptr [ebx+1C]
   00458C71  |.  8D45 F8            lea     eax, dword ptr [ebp-8]
   00458C74  |.  BA 02000000        mov     edx, 2
   00458C79  |.  E8 42E50900        call    004F71C0
   00458C7E  |.  FF4B 1C            dec     dword ptr [ebx+1C]
   00458C81  |.  8D45 FC            lea     eax, dword ptr [ebp-4]
   00458C84  |.  BA 02000000        mov     edx, 2
   00458C89  |.  E8 32E50900        call    004F71C0
   00458C8E  |.  8B0B               mov     ecx, dword ptr [ebx]
   00458C90  |.  64:890D 00000000   mov     dword ptr fs:[0], ecx
   00458C97  |.  E9 C90A0000        jmp     00459765                   ; 直接返回失败信息（未注册版）
   00458C9C  |>  66:C743 10 4400    mov     word ptr [ebx+10], 44
   00458CA2  |.  33C0               xor     eax, eax
   00458CA4  |.  8945 F4            mov     dword ptr [ebp-C], eax
   00458CA7  |.  FF43 1C            inc     dword ptr [ebx+1C]
   00458CAA  |.  66:C743 10 0800    mov     word ptr [ebx+10], 8
   00458CB0  |.  83BD E8FEFFFF 01   cmp     dword ptr [ebp-118], 1      ; 普及版注册识别为 1
   00458CB7  |.  75 3E              jnz     short 00458CF7
   00458CB9  |.  FF4B 1C            dec     dword ptr [ebx+1C]
   00458CBC  |.  8D45 F4            lea     eax, dword ptr [ebp-C]
   00458CBF  |.  BA 02000000        mov     edx, 2
   00458CC4  |.  E8 F7E40900        call    004F71C0
   00458CC9  |.  FF4B 1C            dec     dword ptr [ebx+1C]
   00458CCC  |.  8D45 F8            lea     eax, dword ptr [ebp-8]
   00458CCF  |.  BA 02000000        mov     edx, 2
   00458CD4  |.  E8 E7E40900        call    004F71C0
   00458CD9  |.  FF4B 1C            dec     dword ptr [ebx+1C]
   00458CDC  |.  8D45 FC            lea     eax, dword ptr [ebp-4]
   00458CDF  |.  BA 02000000        mov     edx, 2
   00458CE4  |.  E8 D7E40900        call    004F71C0
   00458CE9  |.  8B0B               mov     ecx, dword ptr [ebx]
   00458CEB  |.  64:890D 00000000   mov     dword ptr fs:[0], ecx
   00458CF2  |.  E9 6E0A0000        jmp     00459765                   ; 直接返回失败信息
   00458CF7  |>  83BD E8FEFFFF 02   cmp     dword ptr [ebp-118], 2      ; 高级版注册识别为 2
   00458CFE  |.  0F85 F7010000      jnz     00458EFB
   00458D04  |.  66:C743 10 5000    mov     word ptr [ebx+10], 50
   00458D0A  |.  33C0               xor     eax, eax
   00458D0C  |.  8945 C4            mov     dword ptr [ebp-3C], eax
   00458D0F  |.  8D55 C4            lea     edx, dword ptr [ebp-3C]
   00458D12  |.  FF43 1C            inc     dword ptr [ebx+1C]
   00458D15  |.  8B86 F0020000      mov     eax, dword ptr [esi+2F0]
   00458D1B  |.  E8 C2330B00        call    <jmp.&VCL50.Controls::TControl::>
   00458D20  |.  8D45 C4            lea     eax, dword ptr [ebp-3C]
   00458D23  |.  33D2               xor     edx, edx
   00458D25  |.  8955 C0            mov     dword ptr [ebp-40], edx
   00458D28  |.  8D55 C0            lea     edx, dword ptr [ebp-40]
   00458D2B  |.  FF43 1C            inc     dword ptr [ebx+1C]
   00458D2E  |.  E8 C5E70900        call    004F74F8
   00458D33  |.  8D45 C0            lea     eax, dword ptr [ebp-40]
   00458D36  |.  33C9               xor     ecx, ecx
   00458D38  |.  50                 push    eax
   00458D39  |.  894D D8            mov     dword ptr [ebp-28], ecx
   00458D3C  |.  FF43 1C            inc     dword ptr [ebx+1C]
   00458D3F  |.  8D55 D8            lea     edx, dword ptr [ebp-28]
   00458D42  |.  8B86 EC020000      mov     eax, dword ptr [esi+2EC]
   00458D48  |.  E8 95330B00        call    <jmp.&VCL50.Controls::TControl::>
   00458D4D  |.  8D45 D8            lea     eax, dword ptr [ebp-28]
   00458D50  |.  33D2               xor     edx, edx
   00458D52  |.  8955 D4            mov     dword ptr [ebp-2C], edx
   00458D55  |.  8D55 D4            lea     edx, dword ptr [ebp-2C]
   00458D58  |.  FF43 1C            inc     dword ptr [ebx+1C]
   00458D5B  |.  E8 98E70900        call    004F74F8
   00458D60  |.  8D55 D4            lea     edx, dword ptr [ebp-2C]
   00458D63  |.  33C0               xor     eax, eax
   00458D65  |.  8945 D0            mov     dword ptr [ebp-30], eax
   00458D68  |.  8D4D D0            lea     ecx, dword ptr [ebp-30]
   00458D6B  |.  FF43 1C            inc     dword ptr [ebx+1C]
   00458D6E  |.  8D47 5D            lea     eax, dword ptr [edi+5D]
   00458D71  |.  E8 E2EA0900        call    004F7858
   00458D76  |.  8D55 D0            lea     edx, dword ptr [ebp-30]
   00458D79  |.  52                 push    edx
   00458D7A  |.  8D97 90000000      lea     edx, dword ptr [edi+90]
   00458D80  |.  8D45 CC            lea     eax, dword ptr [ebp-34]
   00458D83  |.  E8 94E20900        call    004F701C
   00458D88  |.  FF43 1C            inc     dword ptr [ebx+1C]
   00458D8B  |.  33C0               xor     eax, eax
   00458D8D  |.  8945 C8            mov     dword ptr [ebp-38], eax
   00458D90  |.  8D55 CC            lea     edx, dword ptr [ebp-34]
   00458D93  |.  FF43 1C            inc     dword ptr [ebx+1C]
   00458D96  |.  8D4D C8            lea     ecx, dword ptr [ebp-38]
   00458D99  |.  58                 pop     eax
   00458D9A  |.  E8 65E40900        call    004F7204
   00458D9F  |.  8D45 C8            lea     eax, dword ptr [ebp-38]
   00458DA2  |.  33D2               xor     edx, edx
   00458DA4  |.  8955 BC            mov     dword ptr [ebp-44], edx
   00458DA7  |.  8D4D BC            lea     ecx, dword ptr [ebp-44]
   00458DAA  |.  FF43 1C            inc     dword ptr [ebx+1C]
   00458DAD  |.  5A                 pop     edx
   00458DAE  |.  E8 51E40900        call    004F7204
   00458DB3  |.  8D45 BC            lea     eax, dword ptr [ebp-44]
   00458DB6  |.  50                 push    eax
   00458DB7  |.  8D97 98000000      lea     edx, dword ptr [edi+98]
   00458DBD  |.  8D45 B8            lea     eax, dword ptr [ebp-48]
   00458DC0  |.  E8 57E20900        call    004F701C
   00458DC5  |.  FF43 1C            inc     dword ptr [ebx+1C]
```



```
>   00458DC8  |.  33C0             xor     eax, eax  

>   00458DCA  |.  8945 B4          mov     dword ptr [ebp-4C], eax  

>   00458DCD  |.  8D55 B8          lea     edx, dword ptr [ebp-48]  

>   00458DD0  |.  FF43 1C          inc     dword ptr [ebx+1C]  

>   00458DD3  |.  8D4D B4          lea     ecx, dword ptr [ebp-4C]  

>   00458DD6  |.  58               pop     eax  

>   00458DD7  |.  E8 28E40900      call    004F7204  

>   00458DDC  |.  8D45 B4          lea     eax, dword ptr [ebp-4C]  

>   00458DDF  |.  33D2             xor     edx, edx  

>   00458DE1  |.  8955 B0          mov     dword ptr [ebp-50], edx  

>   00458DE4  |.  8D4D B0          lea     ecx, dword ptr [ebp-50]  

>   00458DE7  |.  FF43 1C          inc     dword ptr [ebx+1C]  

>   00458DEA  |.  8D55 F8          lea     edx, dword ptr [ebp-8]  

>   00458DED  |.  E8 12E40900      call    004F7204  

>   00458DF2  |.  8D45 B0          lea     eax, dword ptr [ebp-50]  

>   00458DF5  |.  50               push    eax  

>   00458DF6  |.  8D97 A2000000    lea     edx, dword ptr [edi+A2]  

>   00458DFC  |.  8D45 AC          lea     eax, dword ptr [ebp-54]  

>   00458DFF  |.  E8 18E20900      call    004F701C  

>   00458E04  |.  FF43 1C          inc     dword ptr [ebx+1C]  

>   00458E07  |.  33C0             xor     eax, eax  

>   00458E09  |.  8945 A8          mov     dword ptr [ebp-58], eax  

>   00458E0C  |.  8D55 AC          lea     edx, dword ptr [ebp-54]  

>   00458E0F  |.  FF43 1C          inc     dword ptr [ebx+1C]  

>   00458E12  |.  8D4D A8          lea     ecx, dword ptr [ebp-58]  

>   00458E15  |.  58               pop     eax  

>   00458E16  |.  E8 E9E30900      call    004F7204  

>   00458E1B  |.  8D55 A8          lea     edx, dword ptr [ebp-58]  

>   00458E1E  |.  8D45 F4          lea     eax, dword ptr [ebp-C]  

>   00458E21  |.  E8 CAE30900      call    004F71F0  

>   00458E26  |.  FF4B 1C          dec     dword ptr [ebx+1C]  

>   00458E29  |.  8D45 A8          lea     eax, dword ptr [ebp-58]  

>   00458E2C  |.  BA 02000000      mov     edx, 2  

>   00458E31  |.  E8 8AE30900      call    004F71C0  

>   00458E36  |.  FF4B 1C          dec     dword ptr [ebx+1C]  

>   00458E39  |.  8D45 AC          lea     eax, dword ptr [ebp-54]  

>   00458E3C  |.  BA 02000000      mov     edx, 2  

>   00458E41  |.  E8 7AE30900      call    004F71C0  

>   00458E46  |.  FF4B 1C          dec     dword ptr [ebx+1C]  

>   00458E49  |.  8D45 B0          lea     eax, dword ptr [ebp-50]  

>   00458E4C  |.  BA 02000000      mov     edx, 2  

>   00458E51  |.  E8 6AE30900      call    004F71C0  

>   00458E56  |.  FF4B 1C          dec     dword ptr [ebx+1C]  

>   00458E59  |.  8D45 B4          lea     eax, dword ptr [ebp-4C]  

>   00458E5C  |.  BA 02000000      mov     edx, 2  

>   00458E61  |.  E8 5AE30900      call    004F71C0  

>   00458E66  |.  FF4B 1C          dec     dword ptr [ebx+1C]  

>   00458E69  |.  8D45 B8          lea     eax, dword ptr [ebp-48]  

>   00458E6C  |.  BA 02000000      mov     edx, 2  

>   00458E71  |.  E8 4AE30900      call    004F71C0  

>   00458E76  |.  FF4B 1C          dec     dword ptr [ebx+1C]  

>   00458E79  |.  8D45 BC          lea     eax, dword ptr [ebp-44]  

>   00458E7C  |.  BA 02000000      mov     edx, 2  

>   00458E81  |.  E8 3AE30900      call    004F71C0  

>   00458E86  |.  FF4B 1C          dec     dword ptr [ebx+1C]  

>   00458E89  |.  8D45 C0          lea     eax, dword ptr [ebp-40]  

>   00458E8C  |.  BA 02000000      mov     edx, 2  

>   00458E91  |.  E8 2AE30900      call    004F71C0  

>   00458E96  |.  FF4B 1C          dec     dword ptr [ebx+1C]  

>   00458E99  |.  8D45 C4          lea     eax, dword ptr [ebp-3C]  

>   00458E9C  |.  BA 02000000      mov     edx, 2  

>   00458EA1  |.  E8 1AE30900      call    004F71C0  

>   00458EA6  |.  FF4B 1C          dec     dword ptr [ebx+1C]  

>   00458EA9  |.  8D45 C8          lea     eax, dword ptr [ebp-38]  

>   00458EAC  |.  BA 02000000      mov     edx, 2  

>   00458EB1  |.  E8 0AE30900      call    004F71C0  

>   00458EB6  |.  FF4B 1C          dec     dword ptr [ebx+1C]  

>   00458EB9  |.  8D45 CC          lea     eax, dword ptr [ebp-34]  

>   00458EBC  |.  BA 02000000      mov     edx, 2  

>   00458EC1  |.  E8 FAE20900      call    004F71C0  

>   00458EC6  |.  FF4B 1C          dec     dword ptr [ebx+1C]  

>   00458EC9  |.  8D45 D0          lea     eax, dword ptr [ebp-30]  

>   00458ECC  |.  BA 02000000      mov     edx, 2  

>   00458ED1  |.  E8 EAE20900      call    004F71C0  

>   00458ED6  |.  FF4B 1C          dec     dword ptr [ebx+1C]  

>   00458ED9  |.  8D45 D4          lea     eax, dword ptr [ebp-2C]  

>   00458EDC  |.  BA 02000000      mov     edx, 2  

>   00458EE1  |.  E8 DAE20900      call    004F71C0  

>   00458EE6  |.  FF4B 1C          dec     dword ptr [ebx+1C]  

>   00458EE9  |.  8D45 D8          lea     eax, dword ptr [ebp-28]  

>   00458EEC  |.  BA 02000000      mov     edx, 2  

>   00458EF1  |.  E8 CAE20900      call    004F71C0  

>   00458EF6  |.  E9 FD040000      jmp     004593F8  

>   00458EFB  |>  83BD E8FEFFFF 03   cmp     dword ptr [ebp-118], 3           ;  超级版注册识别为 3  

>   00458F02  |.  0F85 1B020000     jnz     00459123  

>   00458F08  |.  66:C743 10 5C00   mov     word ptr [ebx+10], 5C  

>   00458F0E  |.  33C9             xor     ecx, ecx  

>   00458F10  |.  894D 90          mov     dword ptr [ebp-70], ecx  

>   00458F13  |.  8D55 90          lea     edx, dword ptr [ebp-70]  

>   00458F16  |.  FF43 1C          inc     dword ptr [ebx+1C]  

>   00458F19  |.  8B86 F0020000     mov     eax, dword ptr [esi+2F0]  

>   00458F1F  |.  E8 BE310B00      call    <jmp.&VCL50.Controls::TControl::>  

>   00458F24  |.  8D45 90          lea     eax, dword ptr [ebp-70]  

>   00458F27  |.  33D2             xor     edx, edx  

>   00458F29  |.  8955 8C          mov     dword ptr [ebp-74], edx  

>   00458F2C  |.  8D55 8C          lea     edx, dword ptr [ebp-74]  

>   00458F2F  |.  FF43 1C          inc     dword ptr [ebx+1C]  

>   00458F32  |.  E8 C1E50900      call    004F74F8  

>   00458F37  |.  8D45 8C          lea     eax, dword ptr [ebp-74]  

>   00458F3A  |.  33C9             xor     ecx, ecx  

>   00458F3C  |.  50               push    eax  

>   00458F3D  |.  894D A4          mov     dword ptr [ebp-5C], ecx  

>   00458F40  |.  FF43 1C          inc     dword ptr [ebx+1C]  

>   00458F43  |.  8D55 A4          lea     edx, dword ptr [ebp-5C]  

>   00458F46  |.  8B86 EC020000     mov     eax, dword ptr [esi+2EC]  

>   00458F4C  |.  E8 91310B00      call    <jmp.&VCL50.Controls::TControl::>  

>   00458F51  |.  8D45 A4          lea     eax, dword ptr [ebp-5C]  

>   00458F54  |.  33D2             xor     edx, edx  

>   00458F56  |.  8955 A0          mov     dword ptr [ebp-60], edx  

>   00458F59  |.  8D55 A0          lea     edx, dword ptr [ebp-60]  

>   00458F5C  |.  FF43 1C          inc     dword ptr [ebx+1C]  

>   00458F5F  |.  E8 94E50900      call    004F74F8  

>   00458F64  |.  8D55 A0          lea     edx, dword ptr [ebp-60]  

>   00458F67  |.  33C0             xor     eax, eax  

>   00458F69  |.  8945 9C          mov     dword ptr [ebp-64], eax  

>   00458F6C  |.  8D4D 9C          lea     ecx, dword ptr [ebp-64]  

>   00458F6F  |.  FF43 1C          inc     dword ptr [ebx+1C]  

>   00458F72  |.  8D87 AA000000    lea     eax, dword ptr [edi+AA]  

>   00458F78  |.  E8 DBE80900      call    004F7858  

>   00458F7D  |.  8D55 9C          lea     edx, dword ptr [ebp-64]  

>   00458F80  |.  52               push    edx  
```



```
00458F81 |. 8D97 DE000000      lea     edx, dword ptr [edi+DE]
00458F87 |. 8D45 98           lea     eax, dword ptr [ebp-68]
00458F8A |. E8 8DE00900       call    004F701C
00458F8F |. FF43 1C          inc     dword ptr [ebx+1C]
00458F92 |. 33C0             xor     eax, eax
00458F94 |. 8945 94          mov     dword ptr [ebp-6C], eax
00458F97 |. 8D55 98          lea     edx, dword ptr [ebp-68]
00458F9A |. FF43 1C          inc     dword ptr [ebx+1C]
00458F9D |. 8D4D 94          lea     ecx, dword ptr [ebp-6C]
00458FA0 |. 58               pop     eax
00458FA1 |. E8 5EE20900      call    004F7204
00458FA6 |. 8D45 94          lea     eax, dword ptr [ebp-6C]
00458FA9 |. 33D2             xor     edx, edx
00458FAB |. 8955 88          mov     dword ptr [ebp-78], edx
00458FAE |. 8D4D 88          lea     ecx, dword ptr [ebp-78]
00458FB1 |. FF43 1C          inc     dword ptr [ebx+1C]
00458FB4 |. 5A               pop     edx
00458FB5 |. E8 4AE20900      call    004F7204
00458FBA |. 8D45 88          lea     eax, dword ptr [ebp-78]
00458FBD |. 50               push    eax
00458FBE |. 8D97 E6000000    lea     edx, dword ptr [edi+E6]
00458FC4 |. 8D45 84          lea     eax, dword ptr [ebp-7C]
00458FC7 |. E8 50E00900      call    004F701C
00458FCC |. FF43 1C          inc     dword ptr [ebx+1C]
00458FCF |. 33C0             xor     eax, eax
00458FD1 |. 8945 80          mov     dword ptr [ebp-80], eax
00458FD4 |. 8D55 84          lea     edx, dword ptr [ebp-7C]
00458FD7 |. FF43 1C          inc     dword ptr [ebx+1C]
00458FDA |. 8D4D 80          lea     ecx, dword ptr [ebp-80]
00458FDD |. 58               pop     eax
00458FDE |. E8 21E20900      call    004F7204
00458FE3 |. 8D45 80          lea     eax, dword ptr [ebp-80]
00458FE6 |. 33D2             xor     edx, edx
00458FE8 |. 8995 7CFFFFFF    mov     dword ptr [ebp-84], edx
00458FEE |. 8D8D 7CFFFFFF    lea     ecx, dword ptr [ebp-84]
00458FF4 |. FF43 1C          inc     dword ptr [ebx+1C]
00458FF7 |. 8D55 F8          lea     edx, dword ptr [ebp-8]
00458FFA |. E8 05E20900      call    004F7204
00458FFF |. 8D85 7CFFFFFF    lea     eax, dword ptr [ebp-84]
00459005 |. 50               push    eax
00459006 |. 8D97 F0000000    lea     edx, dword ptr [edi+F0]
0045900C |. 8D85 78FFFFFF    lea     eax, dword ptr [ebp-88]
00459012 |. E8 05E00900      call    004F701C
00459017 |. FF43 1C          inc     dword ptr [ebx+1C]
0045901A |. 33C0             xor     eax, eax
0045901C |. 8985 74FFFFFF    mov     dword ptr [ebp-8C], eax
00459022 |. 8D95 78FFFFFF    lea     edx, dword ptr [ebp-88]
00459028 |. FF43 1C          inc     dword ptr [ebx+1C]
0045902B |. 8D8D 74FFFFFF    lea     ecx, dword ptr [ebp-8C]
00459031 |. 58               pop     eax
00459032 |. E8 CDE10900      call    004F7204
00459037 |. 8D95 74FFFFFF    lea     edx, dword ptr [ebp-8C]
0045903D |. 8D45 F4          lea     eax, dword ptr [ebp-C]
00459040 |. E8 ABE10900      call    004F71F0
00459045 |. FF4B 1C          dec     dword ptr [ebx+1C]
00459048 |. 8D85 74FFFFFF    lea     eax, dword ptr [ebp-8C]
0045904E |. BA 02000000      mov     edx, 2
00459053 |. E8 68E10900      call    004F71C0
00459058 |. FF4B 1C          dec     dword ptr [ebx+1C]
0045905B |. 8D85 78FFFFFF    lea     eax, dword ptr [ebp-88]
00459061 |. BA 02000000      mov     edx, 2
00459066 |. E8 55E10900      call    004F71C0
0045906B |. FF4B 1C          dec     dword ptr [ebx+1C]
0045906E |. 8D85 7CFFFFFF    lea     eax, dword ptr [ebp-84]
00459074 |. BA 02000000      mov     edx, 2
00459079 |. E8 42E10900      call    004F71C0
0045907E |. FF4B 1C          dec     dword ptr [ebx+1C]
00459081 |. 8D45 80          lea     eax, dword ptr [ebp-80]
00459084 |. BA 02000000      mov     edx, 2
00459089 |. E8 32E10900      call    004F71C0
0045908E |. FF4B 1C          dec     dword ptr [ebx+1C]
00459091 |. 8D45 84          lea     eax, dword ptr [ebp-7C]
00459094 |. BA 02000000      mov     edx, 2
00459099 |. E8 22E10900      call    004F71C0
0045909E |. FF4B 1C          dec     dword ptr [ebx+1C]
004590A1 |. 8D45 88          lea     eax, dword ptr [ebp-78]
004590A4 |. BA 02000000      mov     edx, 2
004590A9 |. E8 12E10900      call    004F71C0
004590AE |. FF4B 1C          dec     dword ptr [ebx+1C]
004590B1 |. 8D45 8C          lea     eax, dword ptr [ebp-74]
004590B4 |. BA 02000000      mov     edx, 2
004590B9 |. E8 02E10900      call    004F71C0
004590BE |. FF4B 1C          dec     dword ptr [ebx+1C]
004590C1 |. 8D45 90          lea     eax, dword ptr [ebp-70]
004590C4 |. BA 02000000      mov     edx, 2
004590C9 |. E8 F2E00900      call    004F71C0
004590CE |. FF4B 1C          dec     dword ptr [ebx+1C]
004590D1 |. 8D45 94          lea     eax, dword ptr [ebp-6C]
004590D4 |. BA 02000000      mov     edx, 2
004590D9 |. E8 E2E00900      call    004F71C0
004590DE |. FF4B 1C          dec     dword ptr [ebx+1C]
004590E1 |. 8D45 98          lea     eax, dword ptr [ebp-68]
004590E4 |. BA 02000000      mov     edx, 2
004590E9 |. E8 D2E00900      call    004F71C0
004590EE |. FF4B 1C          dec     dword ptr [ebx+1C]
004590F1 |. 8D45 9C          lea     eax, dword ptr [ebp-64]
004590F4 |. BA 02000000      mov     edx, 2
004590F9 |. E8 C2E00900      call    004F71C0
004590FE |. FF4B 1C          dec     dword ptr [ebx+1C]
00459101 |. 8D45 A0          lea     eax, dword ptr [ebp-60]
00459104 |. BA 02000000      mov     edx, 2
00459109 |. E8 B2E00900      call    004F71C0
0045910E |. FF4B 1C          dec     dword ptr [ebx+1C]
00459111 |. 8D45 A4          lea     eax, dword ptr [ebp-5C]
00459114 |. BA 02000000      mov     edx, 2
00459119 |. E8 A2E00900      call    004F71C0
0045911E |. E9 D5020000      jmp     004593F8
00459123 |> 83BD E8FEFFFF 04  cmp     dword ptr [ebp-118], 4            ;  企业版注册识别为 4
0045912A |. 0F85 8A020000    jnz     004593BA
00459130 |. 66:C743 10 6800  mov     word ptr [ebx+10], 68
00459136 |. 33C9             xor     ecx, ecx
00459138 |. 898D 5CFFFFFF    mov     dword ptr [ebp-A4], ecx
0045913E |. 8D95 5CFFFFFF    lea     edx, dword ptr [ebp-A4]
00459144 |. FF43 1C          inc     dword ptr [ebx+1C]
00459147 |. 8B86 F0020000    mov     eax, dword ptr [esi+2F0]
0045914D |. E8 902F0B00      call    <jmp.&VCL50.Controls::TControl::>
00459152 |. 8D85 5CFFFFFF    lea     eax, dword ptr [ebp-A4]
00459158 |. 33D2             xor     edx, edx
0045915A |. 8995 58FFFFFF    mov     dword ptr [ebp-A8], edx
00459160 |. 8D95 58FFFFFF    lea     edx, dword ptr [ebp-A8]
00459166 |. FF43 1C          inc     dword ptr [ebx+1C]
```



```
00459169  |. E8 8AE30900        call    004F74F8
0045916E  |. 8D85 58FFFFFF      lea     eax, dword ptr [ebp-A8]
00459174  |. 33C9               xor     ecx, ecx
00459176  |. 50                 push    eax
00459177  |. 898D 70FFFFFF      mov     dword ptr [ebp-90], ecx
0045917D  |. FF43 1C            inc     dword ptr [ebx+1C]
00459180  |. 8D95 70FFFFFF      lea     edx, dword ptr [ebp-90]
00459186  |. 8B86 EC020000      mov     eax, dword ptr [esi+2EC]
0045918C  |. E8 512F0B00        call    <jmp.&VCL50.Controls::TControl::>
00459191  |. 8D85 70FFFFFF      lea     eax, dword ptr [ebp-90]
00459197  |. 33D2               xor     edx, edx
00459199  |. 8995 6CFFFFFF      mov     dword ptr [ebp-94], edx
0045919F  |. 8D95 6CFFFFFF      lea     edx, dword ptr [ebp-94]
004591A5  |. FF43 1C            inc     dword ptr [ebx+1C]
004591A8  |. E8 4BE30900        call    004F74F8
004591AD  |. 8D95 6CFFFFFF      lea     edx, dword ptr [ebp-94]
004591B3  |. 33C0               xor     eax, eax
004591B5  |. 8985 68FFFFFF      mov     dword ptr [ebp-98], eax
004591BB  |. 8D8D 68FFFFFF      lea     ecx, dword ptr [ebp-98]
004591C1  |. FF43 1C            inc     dword ptr [ebx+1C]
004591C4  |. 8D87 F8000000      lea     eax, dword ptr [edi+F8]
004591CA  |. E8 89E60900        call    004F7858
004591CF  |. 8D95 68FFFFFF      lea     edx, dword ptr [ebp-98]
004591D5  |. 52                 push    edx
004591D6  |. 8D97 2C010000      lea     edx, dword ptr [edi+12C]
004591DC  |. 8D85 64FFFFFF      lea     eax, dword ptr [ebp-9C]
004591E2  |. E8 35DE0900        call    004F701C
004591E7  |. FF43 1C            inc     dword ptr [ebx+1C]
004591EA  |. 33C0               xor     eax, eax
004591EC  |. 8985 60FFFFFF      mov     dword ptr [ebp-A0], eax
004591F2  |. 8D95 64FFFFFF      lea     edx, dword ptr [ebp-9C]
004591F8  |. FF43 1C            inc     dword ptr [ebx+1C]
004591FB  |. 8D8D 60FFFFFF      lea     ecx, dword ptr [ebp-A0]
00459201  |. 58                 pop     eax
00459202  |. E8 FDDF0900        call    004F7204
00459207  |. 8D85 60FFFFFF      lea     eax, dword ptr [ebp-A0]
0045920D  |. 33D2               xor     edx, edx
0045920F  |. 8995 54FFFFFF      mov     dword ptr [ebp-AC], edx
00459215  |. 8D8D 54FFFFFF      lea     ecx, dword ptr [ebp-AC]
0045921B  |. FF43 1C            inc     dword ptr [ebx+1C]
0045921E  |. 5A                 pop     edx
0045921F  |. E8 E0DF0900        call    004F7204
00459224  |. 8D85 54FFFFFF      lea     eax, dword ptr [ebp-AC]
0045922A  |. 50                 push    eax
0045922B  |. 8D97 34010000      lea     edx, dword ptr [edi+134]
00459231  |. 8D85 50FFFFFF      lea     eax, dword ptr [ebp-B0]
00459237  |. E8 E0DD0900        call    004F701C
0045923C  |. FF43 1C            inc     dword ptr [ebx+1C]
0045923F  |. 33C0               xor     eax, eax
00459241  |. 8985 4CFFFFFF      mov     dword ptr [ebp-B4], eax
00459247  |. 8D95 50FFFFFF      lea     edx, dword ptr [ebp-B0]
0045924D  |. FF43 1C            inc     dword ptr [ebx+1C]
00459250  |. 8D8D 4CFFFFFF      lea     ecx, dword ptr [ebp-B4]
00459256  |. 58                 pop     eax
00459257  |. E8 A8DF0900        call    004F7204
0045925C  |. 8D85 4CFFFFFF      lea     eax, dword ptr [ebp-B4]
00459262  |. 33D2               xor     edx, edx
00459264  |. 8995 48FFFFFF      mov     dword ptr [ebp-B8], edx
0045926A  |. 8D8D 48FFFFFF      lea     ecx, dword ptr [ebp-B8]
00459270  |. FF43 1C            inc     dword ptr [ebx+1C]
00459273  |. 8D55 F8            lea     edx, dword ptr [ebp-8]
00459276  |. E8 89DF0900        call    004F7204
0045927B  |. 8D85 48FFFFFF      lea     eax, dword ptr [ebp-B8]
00459281  |. 50                 push    eax
00459282  |. 8D97 3E010000      lea     edx, dword ptr [edi+13E]
00459288  |. 8D85 44FFFFFF      lea     eax, dword ptr [ebp-BC]
0045928E  |. E8 89DD0900        call    004F701C
00459293  |. FF43 1C            inc     dword ptr [ebx+1C]
00459296  |. 33C0               xor     eax, eax
00459298  |. 8985 40FFFFFF      mov     dword ptr [ebp-C0], eax
0045929E  |. 8D95 44FFFFFF      lea     edx, dword ptr [ebp-BC]
004592A4  |. FF43 1C            inc     dword ptr [ebx+1C]
004592A7  |. 8D8D 40FFFFFF      lea     ecx, dword ptr [ebp-C0]
004592AD  |. 58                 pop     eax
004592AE  |. E8 51DF0900        call    004F7204
004592B3  |. 8D95 40FFFFFF      lea     edx, dword ptr [ebp-C0]
004592B9  |. 8D45 F4            lea     eax, dword ptr [ebp-C]
004592BC  |. E8 2FDF0900        call    004F71F0
004592C1  |. FF4B 1C            dec     dword ptr [ebx+1C]
004592C4  |. 8D85 40FFFFFF      lea     eax, dword ptr [ebp-C0]
004592CA  |. BA 02000000        mov     edx, 2
004592CF  |. E8 ECDE0900        call    004F71C0
004592D4  |. FF4B 1C            dec     dword ptr [ebx+1C]
004592D7  |. 8D85 44FFFFFF      lea     eax, dword ptr [ebp-BC]
004592DD  |. BA 02000000        mov     edx, 2
004592E2  |. E8 D9DE0900        call    004F71C0
004592E7  |. FF4B 1C            dec     dword ptr [ebx+1C]
004592EA  |. 8D85 48FFFFFF      lea     eax, dword ptr [ebp-B8]
004592F0  |. BA 02000000        mov     edx, 2
004592F5  |. E8 C6DE0900        call    004F71C0
004592FA  |. FF4B 1C            dec     dword ptr [ebx+1C]
004592FD  |. 8D85 4CFFFFFF      lea     eax, dword ptr [ebp-B4]
00459303  |. BA 02000000        mov     edx, 2
00459308  |. E8 B3DE0900        call    004F71C0
0045930D  |. FF4B 1C            dec     dword ptr [ebx+1C]
00459310  |. 8D85 50FFFFFF      lea     eax, dword ptr [ebp-B0]
00459316  |. BA 02000000        mov     edx, 2
0045931B  |. E8 A0DE0900        call    004F71C0
00459320  |. FF4B 1C            dec     dword ptr [ebx+1C]
00459323  |. 8D85 54FFFFFF      lea     eax, dword ptr [ebp-AC]
00459329  |. BA 02000000        mov     edx, 2
0045932E  |. E8 8DDE0900        call    004F71C0
00459333  |. FF4B 1C            dec     dword ptr [ebx+1C]
00459336  |. 8D85 58FFFFFF      lea     eax, dword ptr [ebp-A8]
0045933C  |. BA 02000000        mov     edx, 2
00459341  |. E8 7ADE0900        call    004F71C0
00459346  |. FF4B 1C            dec     dword ptr [ebx+1C]
00459349  |. 8D85 5CFFFFFF      lea     eax, dword ptr [ebp-A4]
0045934F  |. BA 02000000        mov     edx, 2
00459354  |. E8 67DE0900        call    004F71C0
00459359  |. FF4B 1C            dec     dword ptr [ebx+1C]
0045935C  |. 8D85 60FFFFFF      lea     eax, dword ptr [ebp-A0]
00459362  |. BA 02000000        mov     edx, 2
00459367  |. E8 54DE0900        call    004F71C0
0045936C  |. FF4B 1C            dec     dword ptr [ebx+1C]
0045936F  |. 8D85 64FFFFFF      lea     eax, dword ptr [ebp-9C]
00459375  |. BA 02000000        mov     edx, 2
0045937A  |. E8 41DE0900        call    004F71C0
0045937F  |. FF4B 1C            dec     dword ptr [ebx+1C]
00459382  |. 8D85 68FFFFFF      lea     eax, dword ptr [ebp-98]
00459388  |. BA 02000000        mov     edx, 2
```



```
  0045938D  |.  E8 2EDE0900        call    004F71C0  
  00459392  |.  FF4B 1C           dec     dword ptr [ebx+1C]  
  00459395  |.  8D85 6CFFFFFF     lea     eax, dword ptr [ebp-94]  
  0045939B  |.  BA 02000000       mov     edx, 2  
  004593A0  |.  E8 1BDE0900       call    004F71C0  
  004593A5  |.  FF4B 1C           dec     dword ptr [ebx+1C]  
  004593A8  |.  8D85 70FFFFFF     lea     eax, dword ptr [ebp-90]  
  004593AE  |.  BA 02000000       mov     edx, 2  
  004593B3  |.  E8 08DE0900       call    004F71C0  
  004593B8  |.  EB 3E             jmp     short 004593F8  
  004593BA  |>  FF4B 1C           dec     dword ptr [ebx+1C]  
  004593BD  |.  8D45 F4           lea     eax, dword ptr [ebp-C]  
  004593C0  |.  BA 02000000       mov     edx, 2  
  004593C5  |.  E8 F6DD0900       call    004F71C0  
  004593CA  |.  FF4B 1C           dec     dword ptr [ebx+1C]  
  004593CD  |.  8D45 F8           lea     eax, dword ptr [ebp-8]  
  004593D0  |.  BA 02000000       mov     edx, 2  
  004593D5  |.  E8 E6DD0900       call    004F71C0  
  004593DA  |.  FF4B 1C           dec     dword ptr [ebx+1C]  
  004593DD  |.  8D45 FC           lea     eax, dword ptr [ebp-4]  
  004593E0  |.  BA 02000000       mov     edx, 2  
  004593E5  |.  E8 D6DD0900       call    004F71C0  
  004593EA  |.  8B0B             mov     ecx, dword ptr [ebx]  
  004593EC  |.  64:890D 00000000  mov     dword ptr fs:[0], ecx  
  004593F3  |.  E9 6D030000      jmp     00459765                        ;  直接返回失败信息  
  004593F8  |>  66:C743 10 7400   mov     word ptr [ebx+10], 74  
  004593FE  |.  33C0             xor     eax, eax  
  00459400  |.  B2 01             mov     dl, 1  
  00459402  |.  8945 F0           mov     dword ptr [ebp-10], eax  
  00459405  |.  FF43 1C           inc     dword ptr [ebx+1C]  
  00459408  |.  66:C743 10 0800   mov     word ptr [ebx+10], 8  
  0045940E  |.  A1 20E75500       mov     eax, dword ptr [<&VCL50.Registry>  
  00459413  |.  E8 52300B00       call    <jmp.&VCL50.Registry::TRegistry:>;  准备写入注册表  
  00459418  |.  8985 DCFEFFFF     mov     dword ptr [ebp-124], eax  
  0045941E  |.  8D85 3CFFFFFF     lea     eax, dword ptr [ebp-C4]  
  00459424  |.  66:C743 10 8000   mov     word ptr [ebx+10], 80  
  0045942A  |.  8D97 46010000     lea     edx, dword ptr [edi+146]  
  00459430  |.  E8 E7DB0900       call    004F701C  
  00459435  |.  FF43 1C           inc     dword ptr [ebx+1C]  
  00459438  |.  33C9             xor     ecx, ecx  
  0045943A  |.  898D 38FFFFFF     mov     dword ptr [ebp-C8], ecx  
  00459440  |.  8D8D 38FFFFFF     lea     ecx, dword ptr [ebp-C8]  
  00459446  |.  FF43 1C           inc     dword ptr [ebx+1C]  
  00459449  |.  8D95 3CFFFFFF     lea     edx, dword ptr [ebp-C4]  
  0045944F  |.  B8 80945500       mov     eax, 00559480  
  00459454  |.  E8 ABDD0900       call    004F7204                        ;  写入注册表地址  
  00459459  |.  8D95 38FFFFFF     lea     edx, dword ptr [ebp-C8]  
  0045945F  |.  8B12             mov     edx, dword ptr [edx]           ;  ASCII "Software\\Aoqun\\Stock Expert\\2.0\\Proxy"  
  00459461  |.  33C9             xor     ecx, ecx  
  00459463  |.  8B85 DCFEFFFF     mov     eax, dword ptr [ebp-124]  
  00459469  |.  E8 E42F0B00       call    <jmp.&VCL50.Registry::TRegistry:>;  打开注册表  
  0045946E  |.  8885 DBFEFFFF     mov     byte ptr [ebp-125], al  
  00459474  |.  FF4B 1C           dec     dword ptr [ebx+1C]  
  00459477  |.  8D85 38FFFFFF     lea     eax, dword ptr [ebp-C8]  
  0045947D  |.  BA 02000000       mov     edx, 2  
  00459482  |.  E8 39DD0900       call    004F71C0  
  00459487  |.  FF4B 1C           dec     dword ptr [ebx+1C]  
  0045948A  |.  8D85 3CFFFFFF     lea     eax, dword ptr [ebp-C4]  
  00459490  |.  BA 02000000       mov     edx, 2  
  00459495  |.  E8 26DD0900       call    004F71C0  
  0045949A  |.  66:C743 10 0800   mov     word ptr [ebx+10], 8  
  004594A0  |.  80BD DBFEFFFF 00  cmp     byte ptr [ebp-125], 0  
  004594A7  |.  0F84 F1000000     je      0045959E  
  004594AD  |.  66:C743 10 8C00   mov     word ptr [ebx+10], 8C  
  004594B3  |.  8D97 4D010000     lea     edx, dword ptr [edi+14D]       ;  UseProxy  
  004594B9  |.  8D85 34FFFFFF     lea     eax, dword ptr [ebp-CC]  
  004594BF  |.  E8 58DB0900       call    004F701C                        ;  写入注册版本类型标识  
  004594C4  |.  FF43 1C           inc     dword ptr [ebx+1C]  
  004594C7  |.  8B10             mov     edx, dword ptr [eax]  
  004594C9  |.  8B85 DCFEFFFF     mov     eax, dword ptr [ebp-124]  
  004594CF  |.  E8 3C2F0B00       call    <jmp.&VCL50.Registry::TRegistry:>;  注册表数据类型：DWORD 值  
  004594D4  |.  8885 E7FEFFFF     mov     byte ptr [ebp-119], al  
  004594DA  |.  FF4B 1C           dec     dword ptr [ebx+1C]  
  004594DD  |.  8D85 34FFFFFF     lea     eax, dword ptr [ebp-CC]  
  004594E3  |.  BA 02000000       mov     edx, 2  
  004594E8  |.  E8 D3DC0900       call    004F71C0                        ;  ProxyHost  
  004594ED  |.  66:C743 10 9800   mov     word ptr [ebx+10], 98  
  004594F3  |.  8D97 56010000     lea     edx, dword ptr [edi+156]  
  004594F9  |.  8D85 30FFFFFF     lea     eax, dword ptr [ebp-D0]  
  004594FF  |.  E8 18DB0900       call    004F701C  
  00459504  |.  FF43 1C           inc     dword ptr [ebx+1C]  
  00459507  |.  8B10             mov     edx, dword ptr [eax]  
  00459509  |.  33C0             xor     eax, eax  
  0045950B  |.  8985 2CFFFFFF     mov     dword ptr [ebp-D4], eax  
  00459511  |.  8D8D 2CFFFFFF     lea     ecx, dword ptr [ebp-D4]  
  00459517  |.  FF43 1C           inc     dword ptr [ebx+1C]  
  0045951A  |.  8B85 DCFEFFFF     mov     eax, dword ptr [ebp-124]  
  00459520  |.  E8 032F0B00       call    <jmp.&VCL50.Registry::TRegistry:>;  注册表数据类型：字符串值  
  00459525  |.  8D95 2CFFFFFF     lea     edx, dword ptr [ebp-D4]  
  0045952B  |.  8D45 F0           lea     eax, dword ptr [ebp-10]  
  0045952E  |.  E8 BDDC0900       call    004F71F0  
  00459533  |.  FF4B 1C           dec     dword ptr [ebx+1C]  
  00459536  |.  8D85 2CFFFFFF     lea     eax, dword ptr [ebp-D4]  
  0045953C  |.  BA 02000000       mov     edx, 2  
  00459541  |.  E8 7ADC0900       call    004F71C0  
  00459546  |.  FF4B 1C           dec     dword ptr [ebx+1C]  
  00459549  |.  8D85 30FFFFFF     lea     eax, dword ptr [ebp-D0]  
  0045954F  |.  BA 02000000       mov     edx, 2  
  00459554  |.  E8 67DC0900       call    004F71C0                        ;  ProxyHostPort  
  00459559  |.  66:C743 10 A400   mov     word ptr [ebx+10], 0A4  
  0045955F  |.  8D97 60010000     lea     edx, dword ptr [edi+160]  
  00459565  |.  8D85 28FFFFFF     lea     eax, dword ptr [ebp-D8]  
  0045956B  |.  E8 ACDA0900       call    004F701C  
  00459570  |.  FF43 1C           inc     dword ptr [ebx+1C]  
  00459573  |.  8B10             mov     edx, dword ptr [eax]  
  00459575  |.  8B85 DCFEFFFF     mov     eax, dword ptr [ebp-124]  
  0045957B  |.  E8 9C2E0B00       call    <jmp.&VCL50.Registry::TRegistry:>;  注册表数据类型：DWORD 值  
  00459580  |.  8985 E0FEFFFF     mov     dword ptr [ebp-120], eax  
  00459586  |.  FF4B 1C           dec     dword ptr [ebx+1C]  
  00459589  |.  8D85 28FFFFFF     lea     eax, dword ptr [ebp-D8]  
  0045958F  |.  BA 02000000       mov     edx, 2  
  00459594  |.  E8 27DC0900       call    004F71C0  
  00459599  |.  E9 9C000000       jmp     0045963A  
```



```
0045959E  |   A1 80DF5500            mov     eax, dword ptr [<&VCL50.Forms::A>
004595A3  |.  6A 30                  push    30
004595A5  |.  8D8F 7D010000         lea     ecx, dword ptr [edi+17D]
004595AB  |.  8D97 6E010000         lea     edx, dword ptr [edi+16E]
004595B1  |.  8B00                  mov     eax, dword ptr [eax]
004595B3  |.  E8 10DA0900          call    004F6FC8
004595B8  |.  8BB5 DCFEFFFF        mov     esi, dword ptr [ebp-124]
004595BE  |.  89B5 20FFFFFF       mov     dword ptr [ebp-E0], esi
004595C4  |.  85F6                  test    esi, esi
004595C6  |.  74 24                  je      short 004595EC
004595C8  |.  8B06                  mov     eax, dword ptr [esi]
004595CA  |.  8985 24FFFFFF       mov     dword ptr [ebp-DC], eax
004595D0  |.  66:C743 10 BC00     mov     word ptr [ebx+10], 0BC
004595D6  |.  BA 03000000        mov     edx, 3
004595DB  |.  8B85 20FFFFFF       mov     eax, dword ptr [ebp-E0]
004595E1  |.  8B08                  mov     ecx, dword ptr [eax]
004595E3  |.  FF51 FC              call    dword ptr [ecx-4]
004595E6  |.  66:C743 10 B000     mov     word ptr [ebx+10], 0B0
004595EC  |   FF4B 1C              dec     dword ptr [ebx+1C]
004595EF  |.  8D45 F0              lea     eax, dword ptr [ebp-10]
004595F2  |.  BA 02000000        mov     edx, 2
004595F7  |.  E8 C4DB0900        call    004F71C0
004595FC  |.  FF4B 1C              dec     dword ptr [ebx+1C]
004595FF  |.  8D45 F4              lea     eax, dword ptr [ebp-C]
00459602  |.  BA 02000000        mov     edx, 2
00459607  |.  E8 B4DB0900        call    004F71C0
0045960C  |.  FF4B 1C              dec     dword ptr [ebx+1C]
0045960F  |.  8D45 F8              lea     eax, dword ptr [ebp-8]
00459612  |.  BA 02000000        mov     edx, 2
00459617  |.  E8 A4DB0900        call    004F71C0
0045961C  |.  FF4B 1C              dec     dword ptr [ebx+1C]
0045961F  |.  8D45 FC              lea     eax, dword ptr [ebp-4]
00459622  |.  BA 02000000        mov     edx, 2
00459627  |.  E8 94DB0900        call    004F71C0
0045962C  |.  8B0B                  mov     ecx, dword ptr [ebx]
0045962E  |.  64:890D 00000000    mov     dword ptr fs:[0], ecx
00459635  |.  E9 2B010000        jmp     00459765
0045963A  |   8B85 DCFEFFFF        mov     eax, dword ptr [ebp-124]
00459640  |.  8985 18FFFFFF       mov     dword ptr [ebp-E8], eax
00459646  |.  83BD 18FFFFFF 00    cmp     dword ptr [ebp-E8], 0
0045964D  |.  74 2A                  je      short 00459679
0045964F  |.  8B95 18FFFFFF       mov     edx, dword ptr [ebp-E8]
00459655  |.  8B0A                  mov     ecx, dword ptr [edx]
00459657  |.  898D 1CFFFFFF       mov     dword ptr [ebp-E4], ecx
0045965D  |.  66:C743 10 D400     mov     word ptr [ebx+10], 0D4
00459663  |.  BA 03000000        mov     edx, 3
00459668  |.  8B85 18FFFFFF       mov     eax, dword ptr [ebp-E8]
0045966E  |.  8B08                  mov     ecx, dword ptr [eax]
00459670  |.  FF51 FC              call    dword ptr [ecx-4]
00459673  |.  66:C743 10 C800     mov     word ptr [ebx+10], 0C8
00459679  |   33D2                  xor     edx, edx
0045967B  |.  8B86 D0020000      mov     eax, dword ptr [esi+2D0]
00459681  |.  8B08                  mov     ecx, dword ptr [eax]
00459683  |.  FF51 5C              call    dword ptr [ecx+5C]
00459686  |.  33D2                  xor     edx, edx
00459688  |.  8B86 D8020000      mov     eax, dword ptr [esi+2D8]
0045968E  |.  8B08                  mov     ecx, dword ptr [eax]
00459690  |.  FF51 5C              call    dword ptr [ecx+5C]
00459693  |.  8B86 FC020000      mov     eax, dword ptr [esi+2FC]
00459699  |.  05 08020000        add     eax, 208
0045969E  |.  8B00                  mov     eax, dword ptr [eax]
004596A0  |.  8B10                  mov     edx, dword ptr [eax]
004596A2  |.  FF52 40              call    dword ptr [edx+40]
004596A5  |.  8B8E FC020000      mov     ecx, dword ptr [esi+2FC]
004596AB  |.  8D97 82010000      lea     edx, dword ptr [edi+182]
004596B1  |.  81C1 08020000      add     ecx, 208
004596B7  |.  8D85 14FFFFFF      lea     eax, dword ptr [ebp-EC]
004596BD  |.  898D D4FEFFFF      mov     dword ptr [ebp-12C], ecx
004596C3  |.  66:C743 10 E000     mov     word ptr [ebx+10], 0E0
004596C9  |.  E8 4ED90900        call    004F701C
004596CE  |.  FF43 1C              inc     dword ptr [ebx+1C]
004596D1  |.  8B10                  mov     edx, dword ptr [eax]
004596D3  |.  8B85 D4FEFFFF      mov     eax, dword ptr [ebp-12C]
004596D9  |.  8B00                  mov     eax, dword ptr [eax]
004596DB  |.  8B08                  mov     ecx, dword ptr [eax]
004596DD  |.  FF51 34              call    dword ptr [ecx+34]
004596E0  |.  FF4B 1C              dec     dword ptr [ebx+1C]
004596E3  |.  8D85 14FFFFFF      lea     eax, dword ptr [ebp-EC]
004596E9  |.  BA 02000000        mov     edx, 2
004596EE  |.  E8 CDDA0900        call    004F71C0
004596F3  |.  80BD E7FEFFFF 00   cmp     byte ptr [ebp-119], 0
004596FA  |.  74 16                  je      short 00459712
004596FC  |.  8B8D E0FEFFFF      mov     ecx, dword ptr [ebp-120]
00459702  |.  51                      push    ecx
00459703  |.  8B4D F0              mov     ecx, dword ptr [ebp-10]
00459706  |.  8B55 F4              mov     edx, dword ptr [ebp-C]
00459709  |.  8BC6                  mov     eax, esi
0045970B  |.  E8 88100000        call    0045A798
00459710  |.  EB 0A                  jmp     short 0045971C
00459712  |   8B55 F4              mov     edx, dword ptr [ebp-C]
00459715  |.  8BC6                  mov     eax, esi
00459717  |.  E8 3C0F0000        call    0045A658
0045971C  |   FF4B 1C              dec     dword ptr [ebx+1C]
0045971F  |.  8D45 F0              lea     eax, dword ptr [ebp-10]
00459722  |.  BA 02000000        mov     edx, 2
00459727  |.  E8 94DA0900        call    004F71C0
0045972C  |.  FF4B 1C              dec     dword ptr [ebx+1C]
0045972F  |.  8D45 F4              lea     eax, dword ptr [ebp-C]
00459732  |.  BA 02000000        mov     edx, 2
00459737  |.  E8 84DA0900        call    004F71C0
0045973C  |.  FF4B 1C              dec     dword ptr [ebx+1C]
0045973F  |.  8D45 F8              lea     eax, dword ptr [ebp-8]
00459742  |.  BA 02000000        mov     edx, 2
00459747  |.  E8 74DA0900        call    004F71C0
0045974C  |.  FF4B 1C              dec     dword ptr [ebx+1C]
0045974F  |.  8D45 FC              lea     eax, dword ptr [ebp-4]
00459752  |.  BA 02000000        mov     edx, 2
00459757  |.  E8 64DA0900        call    004F71C0
0045975C  |.  8B0B                  mov     ecx, dword ptr [ebx]
0045975E  |.  64:890D 00000000    mov     dword ptr fs:[0], ecx
00459765  |   5F                      pop     edi
00459766  |.  5E                      pop     esi
00459767  |.  5B                      pop     ebx
00459768  |.  8BE5                  mov     esp, ebp
0045976A  |.  5D                      pop     ebp
0045976B  \.  C3                      ret

CALL 进后，程序就开始关键的注册算法验证了：

004252C4  /   55                      push    ebp                     ;  跟进到这里（算法开始）
004252C5  |.  8BEC                  mov     ebp, esp
004252C7  |.  83C4 E0              add     esp, -20
004252CA  |.  53                      push    ebx
```



```
 004252CB  |.  56                      push    esi  
 004252CC  |.  57                      push    edi  
 004252CD  |.  8B7D 08                mov     edi, dword ptr [ebp+8]  
 004252D0  |.  57                      push    edi                         ; /s  
 004252D1  |.  E8 7C760E00            call    <jmp.&CC3250MT._strlen>      ; \_strlen  
 004252D6  |.  59                      pop     ecx  
 004252D7  |.  85C0                   test    eax, eax  
 004252D9  |.  75 07                  jnz     short 004252E2  
 004252DB  |.  33C0                   xor     eax, eax  
 004252DD  |.  E9 26010000            jmp     00425408  
 004252E2  |>  57                      push    edi                         ; /s  
 004252E3  |.  E8 6A760E00            call    <jmp.&CC3250MT._strlen>      ; \_strlen  
 004252E8  |.  59                      pop     ecx  
 004252E9  |.  83F8 10                cmp     eax, 10  
 004252EC  |.  74 0A                  je      short 004252F8  
 004252EE  |.  B8 09000000            mov     eax, 9  
 004252F3  |.  E9 10010000            jmp     00425408  
 004252F8  |>  33F6                   xor     esi, esi  
 004252FA  |.  8BDF                   mov     ebx, edi  
 004252FC  |>  0FBE03                 /movsx   eax, byte ptr [ebx]         ;  依次传入注册密码的 ASCII 值  
 004252FF  |.  50                     |push     eax                        ; /c  
 00425300  |.  E8 0B760E00            |call     <jmp.&CC3250MT._isxdigit>   ; \_isxdigit  
 00425305  |.  59                     |pop      ecx  
 00425306  |.  85C0                   |test     eax, eax  
 00425308  |.  75 0A                  |jnz      short 00425314  
 0042530A  |.  B8 09000000            |mov      eax, 9  
 0042530F  |.  E9 F4000000            |jmp      00425408  
 00425314  |>  46                     |inc      esi  
 00425315  |.  43                     |inc      ebx  
 00425316  |.  83FE 10                |cmp      esi, 10  
 00425319  |.^ 7C E1                  \jl      short 004252FC  
 0042531B  |.  8D55 FC                lea     edx, dword ptr [ebp-4]  
 0042531E  |.  52                      push    edx  
 0042531F  |.  8D4D F8                lea     ecx, dword ptr [ebp-8]  
 00425322  |.  51                      push    ecx  
 00425323  |.  8D45 F4                lea     eax, dword ptr [ebp-C]  
 00425326  |.  50                      push    eax  
 00425327  |.  8D55 F0                lea     edx, dword ptr [ebp-10]  
 0042532A  |.  52                      push    edx  
 0042532B  |.  8D4D EC                lea     ecx, dword ptr [ebp-14]  
 0042532E  |.  51                      push    ecx  
 0042532F  |.  8D45 E8                lea     eax, dword ptr [ebp-18]  
 00425332  |.  50                      push    eax  
 00425333  |.  8D55 E4                lea     edx, dword ptr [ebp-1C]  
 00425336  |.  52                      push    edx  
 00425337  |.  8D4D E0                lea     ecx, dword ptr [ebp-20]  
 0042533A  |.  51                      push    ecx  
 0042533B  |.  68 2A9A5100            push    00519A2A                    ; |%02X%02X%02X%02X%02X%02X%02X%02X  
 00425340  |.  57                      push    edi                         ; |ASCII "1234567890123456"  
 00425341  |.  E8 F4750E00            call    <jmp.&CC3250MT._sscanf>      ; \_sscanf  
 00425346  |.  8B45 FC                mov     eax, dword ptr [ebp-4]  
 00425349  |.  8B55 F8                mov     edx, dword ptr [ebp-8]  
 0042534C  |.  8B4D F4                mov     ecx, dword ptr [ebp-C]  
 0042534F  |.  8B5D F0                mov     ebx, dword ptr [ebp-10]  
 00425352  |.  F7D3                   not     ebx                         ;  ebx=00000090  
 00425354  |.  F7D1                   not     ecx                         ;  ecx=00000012  
 00425356  |.  F7D2                   not     edx                         ;  edx=00000034  
 00425358  |.  F7D0                   not     eax                         ;  eax=00000056  
 0042535A  |.  25 FF000000            and     eax, 0FF                    ;  0FF and eax = FFFFFFFFA9  
 0042535F  |.  81E2 FF000000          and     edx, 0FF                    ;  0FF and edx = FFFFFFFFCB  
 00425365  |.  81E1 FF000000          and     ecx, 0FF                    ;  0FF and ecx = FFFFFFFFED  
 0042536B  |.  81E3 FF000000          and     ebx, 0FF                    ;  0FF and ebx = FFFFFFFF6F  
 00425371  |.  335D EC                xor     ebx, dword ptr [ebp-14]     ;  EBX 与上面的值异或,ebx=00000078  
 00425374  |.  334D E8                xor     ecx, dword ptr [ebp-18]     ;  ECX 与上面的值异或,ecx=00000056  
 00425377  |.  3355 E4                xor     edx, dword ptr [ebp-1C]     ;  EDX 与上面的值异或,edx=00000034  
 0042537A  |.  3345 E0                xor     eax, dword ptr [ebp-20]     ;  EAX 与上面的值异或,eax=00000012  
 0042537D  |.  25 FF000000            and     eax, 0FF                    ;  0FF and eax = 000000BB  
 00425382  |.  81E2 FF000000          and     edx, 0FF                    ;  0FF and edx = 000000FF  
 00425388  |.  81E1 FF000000          and     ecx, 0FF                    ;  0FF and edx = 000000BB  
 0042538E  |.  81E3 FF000000          and     ebx, 0FF                    ;  0FF and ebx = 00000017  
 00425394  |.  83C4 28                add     esp, 28                     ;  下面开始例举注册环境  
 00425397  |.  83F8 32                cmp     eax, 32                     ;  注册环境①（高级版）成功注册后，开始连接网络  
 0042539A  |.  75 16                  jnz     short 004253B2  
 0042539C  |.  83FA 31                cmp     edx, 31  
 0042539F  |.  75 11                  jnz     short 004253B2  
 004253A1  |.  83F9 30                cmp     ecx, 30  
 004253A4  |.  75 0C                  jnz     short 004253B2  
 004253A6  |.  83FB 2F                cmp     ebx, 2F  
 004253A9  |.  75 07                  jnz     short 004253B2  
 004253AB  |.  B8 02000000            mov     eax, 2  
 004253B0  |.  EB 56                  jmp     short 00425408  
 004253B2  |>  83F8 33                cmp     eax, 33                     ;  注册环境②（普及版）成功注册后，显示“试用版”  
 004253B5  |.  75 16                  jnz     short 004253CD  
 004253B7  |.  83FA 34                cmp     edx, 34  
 004253BA  |.  75 11                  jnz     short 004253CD  
 004253BC  |.  83F9 35                cmp     ecx, 35  
 004253BF  |.  75 0C                  jnz     short 004253CD  
 004253C1  |.  83FB 36                cmp     ebx, 36  
 004253C4  |.  75 07                  jnz     short 004253CD  
 004253C6  |.  B8 01000000            mov     eax, 1  
 004253CB  |.  EB 3B                  jmp     short 00425408  
 004253CD  |>  83F8 2E                cmp     eax, 2E                     ;  注册环境③（超级版）成功注册后，开始连网  
 004253D0  |.  75 16                  jnz     short 004253E8  
 004253D2  |.  83FA 2D                cmp     edx, 2D  
 004253D5  |.  75 11                  jnz     short 004253E8  
 004253D7  |.  83F9 2C                cmp     ecx, 2C  
 004253DA  |.  75 0C                  jnz     short 004253E8  
 004253DC  |.  83FB 2B                cmp     ebx, 2B  
 004253DF  |.  75 07                  jnz     short 004253E8  
 004253E1  |.  B8 03000000            mov     eax, 3  
 004253E6  |.  EB 20                  jmp     short 00425408  
 004253E8  |>  83F8 31                cmp     eax, 31                     ;  注册环境④（企业版）成功注册后，开始连网  
 004253EB  |.  75 16                  jnz     short 00425403  
 004253ED  |.  83FA 32                cmp     edx, 32  
 004253F0  |.  75 11                  jnz     short 00425403  
 004253F2  |.  83F9 33                cmp     ecx, 33  
 004253F5  |.  75 0C                  jnz     short 00425403  
 004253F7  |.  83FB 34                cmp     ebx, 34  
 004253FA  |.  75 07                  jnz     short 00425403  
 004253FC  |.  B8 04000000            mov     eax, 4  
```



```
  00425401  |.  EB 05             jmp     short 00425408  

  00425403  |>  B8 09000000       mov     eax, 9                        ;  注册环境⑤注册失败（未注册版）  

  00425408  |>  5F               pop     edi  

  00425409  |.  5E               pop     esi  

  0042540A  |.  5B               pop     ebx  

  0042540B  |.  8BE5             mov     esp, ebp  

  0042540D  |.  5D               pop     ebp  

  0042540E  \.  C3               retn  

```

--------------------------------------------------------------------------------

**【经验总结】**

到这里算法应该看懂了吧，整个程序算法并没有用到用户名和邮箱地址，而且注册码计算中所使用到的 `Xor` 跟 `Not` 都是可逆运

算，自己想怎么写 `KEYGEN` 就怎么写吧

这里提供一组通用密码：

* 注册环境①（高级版）：`5457564999999999`
* 注册环境②（普及版）：`5552535099999999`
* 注册环境③（超级版）：`4142434490909090`
* 注册环境④（企业版）：`5754555299999999`

不过，这个软件虽然是 `2006-9-3` 的软件不算新，也不算老，但是注册后登陆一直接受不到数据，郁闷 ing...

--------------------------------------------------------------------------------

**【版权声明】**: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

`2007` 年 `02` 月 `10` 日 `PM 11:33:45`

`4dd466d0e1e0cc123afbe504da2462ca`
