

# Vbox 4.2 实例 Insta3De(《加密与解密》书 13.2.2 )

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-479.htm`](https://www.pediy.com/kssd/pediy09/pediy09-479.htm)

> 【软件名称】Vbox 4.2 实例 Insta3De(《加密与解密》书 13.2.2 )  
> 【应用平台】Win2000 SP4  
> 【作者邮箱】chubing6143@sina.com  
> 【使用工具】OllyDbg1.10  
> 【软件限制】VBOX4.20  
> 【破解工具】OllyDbg v1.10  
>   
> 【破解过程】  
> 写在前面的话:  
> 《加密与解密》一书中讲解的时利用 TRW2000 如何去除该例子的保护,然而我用 OD 脱壳时,遇到了不少困难(先说明,我是在 Win2K 平台下脱壳的,我没有试其他系统),因此将过程写出来,希望对我等菜鸟有一定的帮助.  
>   
> 按照书上讲解,我首先利用 OD 加载程序,然后,F9 运行,然后弹出 VBOX 保护界面,在 OD 中下断 BP GetProcAddress,此时想去点击"Try"按钮,但是不能够了,因为总是被 OD 拦截下来了.  
>   
> 0012DCFC    3B055C79  /CALL 到 GetProcAddress 来自 IMSC40A.3B055C73  
> 0012DD00    796D0000  |hModule = 796D0000 (ADVAPI32)  
> 0012DD04    3B034248  \\ProcNameOrOrdinal = "CheckTokenMembership"  
>   
> 作为菜鸟这下可完蛋了.我想到一个办法,不是不能点击"Try"按钮吗?我先对"Try"按钮下 WM\_LBUTTONUP 消息断点,然后被中断之后再下断 BP GetProcAddress 不就行了嘛!菜鸟也能变通了吧!  
>   
> F9 运行,然后弹出 VBOX 保护界面,对"Try"按钮下 WM\_LBUTTONUP 消息断点,点击 Try 按钮,  
> 程序被中断下来了:  
> 77E062C3 \[>  55             push ebp  
> 77E062C4     8BEC           mov ebp,esp  
> 77E062C6     8B4D 08        mov ecx,dword ptr ss:\[ebp+8\]  
> 77E062C9     56             push esi  
> 77E062CA     57             push edi  
> 77E062CB     E8 00CDFEFF    call USER32.77DF2FD0  
> 堆栈显示：  
> 0012E32C    77DF2CA8  /CALL 到 Assumed WinProc 来自 USER32.77DF2CA5  
> 0012E330    003801EA  |hWnd = 003801EA ('&Try',class='Button',parent=00BB01CE)  
> 0012E334    00000202  |Message = WM\_LBUTTONUP  
> 0012E338    00000000  |Keys = 0  
> 0012E33C    000B0036  \\X = 54. Y = 11.  
>   
> 此时,再在 OD 中下断 BP GetProcAddress,F9 运行一下:  
> 程序被中断下来,  
> 77E6E6A9 K>  55             push ebp                      ; 中断在此处  
> 77E6E6AA     8BEC           mov ebp,esp  
> 77E6E6AC     51             push ecx  
> 77E6E6AD     51             push ecx  
> 77E6E6AE     53             push ebx  
> 77E6E6AF     57             push edi  
> 77E6E6B0     8B7D 0C        mov edi,dword ptr ss:\[ebp+C\]  
> 77E6E6B3     BB FFFF0000    mov ebx,0FFFF  
> 77E6E6B8     3BFB           cmp edi,ebx  
> 77E6E6BA     0F86 D3EB0000  jbe KERNEL32.77E7D293  
> 77E6E6C0     8D45 F8        lea eax,dword ptr ss:\[ebp-8\]  
> 77E6E6C3     57             push edi  
> 77E6E6C4     50             push eax  
> 77E6E6C5     FF15 2413E677  call dword ptr ds:\[<&NTDLL.Rt>; ntdll.RtlInitString  
>   
> F2 取消断点,Ctrl+F9  
> 程序运行到下面代码处:  
> 77E6E706     C2 0800        retn 8  
> 77E6E709   - FF25 2013E677  jmp dword ptr ds:\[<&NTDLL.Ldr>; ntdll.LdrGetProcedureAddress  
>   
> F8 运行,程序运行到下面代码:  
> 08F74EB1     FF15 5458FE08  call dword ptr ds:\[8FE5854\]   ; KERNEL32.GetProcAddress  
> 08F74EB7     3BC3           cmp eax,ebx                              ; 中断在这儿  
>   
> 运行到 08F74EB7 处后 Ctrl+F9 运行,等待时间稍长,运行到下面代码处：  
> 08F7502B     C3             retn                                     ; 中断在这儿  
> 08F7502C     55             push ebp  
> 08F7502D     8BEC           mov ebp,esp  
>   
> F8 运行,程序来到下面的位置：  
> 08F71C95     E8 772F0000    call vboxt420.08F74C11  
> 08F71C9A     8B75 08        mov esi,dword ptr ss:\[ebp+8\]  
> 08F71C9D     83C4 20        add esp,20  
> 08F71CA0     837E 10 00     cmp dword ptr ds:\[esi+10\],0  
> 08F71CA4     74 5F          je short vboxt420.08F71D05  
> 08F71CA6     8D8D 38FFFFFF  lea ecx,dword ptr ss:\[ebp-C8\]  
> 08F71CAC     E8 1FB70100    call vboxt420.08F8D3D0  
>   
> Ctrl+F9 运行,运行到下面代码处：  
> 08F73920     C2 1000        retn 10                       ; 中断在这儿  
> 08F73923     8B0D 58F8FD08  mov ecx,dword ptr ds:\[8FDF858>  
> 08F73929     85C9           test ecx,ecx  
> 08F7392B     74 05          je short vboxt420.08F73932  
>   
> F8 运行,程序来到下面的位置：  
> 08F5049A     FFD0           call eax  
> 08F5049C     B8 01000000    mov eax,1  
> 08F504A1     5F             pop edi  
>   
> Ctrl+F9 运行,运行到下面代码处：  
> 08F504AB     C3             retn                           ; 中断在这儿  
>   
> F8 运行,程序来到下面的位置：  
> 08F50248     E8 9A000000    call 08F502E7  
> 08F5024D     83C4 04        add esp,4  
> 08F50250     85C0           test eax,eax  
> 08F50252     75 0A          jnz short 08F5025E  
>   
> Ctrl+F9 运行,运行到下面代码处：  
> 08F502C8     C2 0C00        retn 0C                        ; 中断在这儿  
>   
> F8 运行,程序来到下面的位置：  
> 08F5011C     E8 06000000    call 08F50127  
> 08F50121     5F             pop edi                       ; 08DF00C8  
>   
> Ctrl+F9 运行,运行到下面代码处：  
> 08F50124     C2 0400        retn 4                        ; 中断在这儿，我个人总结 VBOX 保护中碰到 retn 4 的时候  
>                                                           ; 其实就要返回关键地方了，而且破解 VBOX4.3 与 4.2 保护时经过的 RET 次数都一样  
>   
> F8 运行,程序来到下面的位置：  
> 08F20454     FFD6           call esi                      ; 这就使书上讲的关键的 CALL ESI 了，设断点  
> 08F20456     5B             pop ebx                       ; Insta3De.00692048  
> 08F20457     85C0           test eax,eax  
> 08F20459     74 2E          je short 08F20489  
> 08F2045B     68 00800000    push 8000  
> 08F20460     6A 00          push 0  
> 08F20462     56             push esi  
> 08F20463     8B75 08        mov esi,dword ptr ss:\[ebp+8\]  
> 08F20466     FF56 78        call dword ptr ds:\[esi+78\]  
> 08F20469     8B45 0C        mov eax,dword ptr ss:\[ebp+C\]  
> 08F2046C     68 00800000    push 8000  
> 08F20471     6A 00          push 0  
> 08F20473     50             push eax  
>   
>   
>   
> 在 08F20454 处 F2,然后 F9 运行,程序再次中断在此处,然后 F2 取消断点,此时 F7 进入该 CALL,然后一路 F8  
>   
> 0FC60000     0BC0           or eax,eax  
> 0FC60002     74 00          je short 0FC60004  
> 0FC60004     33C4           xor eax,esp  
> 0FC60006     8CE0           mov ax,fs  
> 0FC60008     2BC4           sub eax,esp  
> 0FC6000A     2BC5           sub eax,ebp  
> 0FC6000C     33C5           xor eax,ebp  
> 0FC6000E     90             nop  
> 0FC6000F     0BC5           or eax,ebp  
> 0FC60011     8CE0           mov ax,fs  
> 0FC60013     F7D0           not eax  
> 0FC60015     86E0           xchg al,ah  
> 0FC60017     32E0           xor ah,al  
> 0FC60019     0BC5           or eax,ebp  
> 0FC6001B     EB 62          jmp short 0FC6007F  
> ......  
> 0FC6007F     55             push ebp  
> 0FC60080     8BEC           mov ebp,esp  
> 0FC60082     83C4 F8        add esp,-8  
> 0FC60085     56             push esi  
> 0FC60086     57             push edi  
> 0FC60087     EB 08          jmp short 0FC60091  
> 0FC60089     0000           add byte ptr ds:\[eax\],al  
> 0FC6008B     0000           add byte ptr ds:\[eax\],al  
> 0FC6008D     0000           add byte ptr ds:\[eax\],al  
> 0FC6008F     0000           add byte ptr ds:\[eax\],al  
> 0FC60091     EB 2F          jmp short 0FC600C2  
> ......  
> 0FC600C2    /EB 24          jmp short 0FC600E8  
> ......  
> 0FC600E8     8B75 08        mov esi,dword ptr ss:\[ebp+8\]  ; Insta3De.00692048  
> 0FC600EB     8B86 00010000  mov eax,dword ptr ds:\[esi+100>  
> 0FC600F1     8BD8           mov ebx,eax  
> 0FC600F3     53             push ebx  
> 0FC600F4     81C3 45000000  add ebx,45  
> 0FC600FA     895D FC        mov dword ptr ss:\[ebp-4\],ebx  
> 0FC600FD     5B             pop ebx  
> 0FC600FE     8BFB           mov edi,ebx  
> 0FC60100     81C7 28000000  add edi,28  
> 0FC60106     897D F8        mov dword ptr ss:\[ebp-8\],edi  
> 0FC60109     FF75 F8        push dword ptr ss:\[ebp-8\]  
> 0FC6010C     FF75 FC        push dword ptr ss:\[ebp-4\]  
> 0FC6010F     FF75 08        push dword ptr ss:\[ebp+8\]  
> 0FC60112     E8 08000000    call 0FC6011F                 ; 短距离 CALL,实际上就是跳转了,F7 跟进去,如果 F8 的话就会运行  
> ......  
> 0FC8011F     8B4C24 0C      mov ecx,dword ptr ss:\[esp+C\]  
> 0FC80123     83EC 10        sub esp,10  
> 0FC80126     8A01           mov al,byte ptr ds:\[ecx\]  
> 0FC80128     53             push ebx  
> 0FC80129     55             push ebp  
> 0FC8012A     56             push esi  
> 0FC8012B     84C0           test al,al  
> 0FC8012D     57             push edi  
> 0FC8012E     74 0C          je short 0FC8013C  
> 0FC80130     F6D0           not al  
> 0FC80132     8801           mov byte ptr ds:\[ecx\],al  
> 0FC80134     8A41 01        mov al,byte ptr ds:\[ecx+1\]  
> 0FC80137     41             inc ecx  
> 0FC80138     84C0           test al,al  
> 0FC8013A   ^ 75 F4          jnz short 0FC80130  
> 0FC8013C     8B6C24 24      mov ebp,dword ptr ss:\[esp+24\]  
> 0FC80140     8BBD D4000000  mov edi,dword ptr ss:\[ebp+D4\]  
> 0FC80146     8BB5 DC000000  mov esi,dword ptr ss:\[ebp+DC\]  
> 0FC8014C     897424 10      mov dword ptr ss:\[esp+10\],esi  
> 0FC80150     66:8B07        mov ax,word ptr ds:\[edi\]  
> 0FC80153     66:35 F80A     xor ax,0AF8  
> 0FC80157     66:3D B550     cmp ax,50B5  
> 0FC8015B     74 04          je short 0FC80161  
> 0FC8015D     33C0           xor eax,eax  
> 0FC8015F     EB 23          jmp short 0FC80184  
> 0FC80161     8B47 3C        mov eax,dword ptr ds:\[edi+3C\]  
> 0FC80164     33C9           xor ecx,ecx  
> 0FC80166     03C7           add eax,edi  
> 0FC80168     66:8B08        mov cx,word ptr ds:\[eax\]  
> 0FC8016B     F7D1           not ecx  
> 0FC8016D     81F1 E44B0000  xor ecx,4BE4  
> 0FC80173     81F9 4BF1FFFF  cmp ecx,-0EB5  
> 0FC80179     74 04          je short 0FC8017F  
> 0FC8017B     33C0           xor eax,eax  
> 0FC8017D     EB 05          jmp short 0FC80184  
> 0FC8017F     8B40 78        mov eax,dword ptr ds:\[eax+78\]  
> 0FC80182     03C7           add eax,edi  
> 0FC80184     85C0           test eax,eax  
> 0FC80186     0F84 93000000  je 0FC8021F  
> 0FC8018C     8B50 1C        mov edx,dword ptr ds:\[eax+1C\]  
> 0FC8018F     8B48 20        mov ecx,dword ptr ds:\[eax+20\]  
> 0FC80192     8B58 24        mov ebx,dword ptr ds:\[eax+24\]  
> 0FC80195     03D7           add edx,edi  
> 0FC80197     03CF           add ecx,edi  
> 0FC80199     03DF           add ebx,edi  
> 0FC8019B     F7C6 00000080  test esi,80000000  
> 0FC801A1     895424 1C      mov dword ptr ss:\[esp+1C\],edx  
> 0FC801A5     895C24 18      mov dword ptr ss:\[esp+18\],ebx  
> 0FC801A9     74 17          je short 0FC801C2  
> 0FC801AB     81E6 FFFFFF7F  and esi,7FFFFFFF  
> 0FC801B1     4E             dec esi  
> 0FC801B2     8BCE           mov ecx,esi  
> 0FC801B4     8B70 14        mov esi,dword ptr ds:\[eax+14\]  
> 0FC801B7     3BCE           cmp ecx,esi  
> 0FC801B9     73 64          jnb short 0FC8021F  
> 0FC801BB     8B048A         mov eax,dword ptr ds:\[edx+ecx>  
> 0FC801BE     03C7           add eax,edi  
> 0FC801C0     EB 5F          jmp short 0FC80221  
> 0FC801C2     8B40 18        mov eax,dword ptr ds:\[eax+18\]  
> 0FC801C5     33D2           xor edx,edx  
> 0FC801C7     85C0           test eax,eax  
> 0FC801C9     894424 14      mov dword ptr ss:\[esp+14\],eax  
> 0FC801CD     76 50          jbe short 0FC8021F  
> 0FC801CF     8BE9           mov ebp,ecx  
> 0FC801D1     EB 04          jmp short 0FC801D7  
> 0FC801D3     8B7424 10      mov esi,dword ptr ss:\[esp+10\]  
> 0FC801D7     8B4D 00        mov ecx,dword ptr ss:\[ebp\]  
> 0FC801DA     8BC7           mov eax,edi  
> 0FC801DC     03C1           add eax,ecx  
> 0FC801DE     8A18           mov bl,byte ptr ds:\[eax\]  
> 0FC801E0     8ACB           mov cl,bl  
> 0FC801E2     3A1E           cmp bl,byte ptr ds:\[esi\]  
> 0FC801E4     75 1C          jnz short 0FC80202  
> 0FC801E6     84C9           test cl,cl  
> 0FC801E8     74 14          je short 0FC801FE  
> 0FC801EA     8A58 01        mov bl,byte ptr ds:\[eax+1\]  
> 0FC801ED     8ACB           mov cl,bl  
> 0FC801EF     3A5E 01        cmp bl,byte ptr ds:\[esi+1\]  
> 0FC801F2     75 0E          jnz short 0FC80202  
> 0FC801F4     83C0 02        add eax,2  
> 0FC801F7     83C6 02        add esi,2  
> 0FC801FA     84C9           test cl,cl  
> 0FC801FC   ^ 75 E0          jnz short 0FC801DE  
> 0FC801FE     33C0           xor eax,eax  
> 0FC80200     EB 05          jmp short 0FC80207  
> 0FC80202     1BC0           sbb eax,eax  
> 0FC80204     83D8 FF        sbb eax,-1  
> 0FC80207     85C0           test eax,eax  
> 0FC80209     0F84 C1000000  je 0FC802D0  
> 0FC8020F     8B4424 14      mov eax,dword ptr ss:\[esp+14\]  
> 0FC80213     42             inc edx  
> 0FC80214     83C5 04        add ebp,4  
> 0FC80217     3BD0           cmp edx,eax  
> 0FC80219   ^ 72 B8          jb short 0FC801D3             ; 在这里循环,向上搜索大跳转,发现 0FC801C0    /EB 5F          jmp short 0FC80221  
> 0FC8021B     8B6C24 24      mov ebp,dword ptr ss:\[esp+24\]  
> 0FC8021F     33C0           xor eax,eax  
> 0FC80221     8B4C24 2C      mov ecx,dword ptr ss:\[esp+2C\] ; 此处 F4  
> 0FC80225     51             push ecx  
> 0FC80226     57             push edi  
> 0FC80227     FFD0           call eax                      ; EAX 对应 KERNEL32.GetProcAddress  
> 0FC80229     85C0           test eax,eax  
> 0FC8022B     8985 9C000000  mov dword ptr ss:\[ebp+9C\],eax  
> 0FC80231     75 0A          jnz short 0FC8023D  
> 0FC80233     5F             pop edi  
> 0FC80234     5E             pop esi  
> 0FC80235     5D             pop ebp  
> 0FC80236     5B             pop ebx  
> 0FC80237     83C4 10        add esp,10  
> 0FC8023A     C2 0C00        retn 0C  
> 0FC8023D     8D5424 24      lea edx,dword ptr ss:\[esp+24\]  
> 0FC80241     52             push edx  
> 0FC80242     55             push ebp  
> 0FC80243     E8 A7000000    call 0FC802EF                 ; 此处 F7 进去  
> ......  
> 0FC802EF     55             push ebp                      ; Insta3De.00692048  
> 0FC802F0     8BEC           mov ebp,esp  
> 0FC802F2     83EC 14        sub esp,14  
> 0FC802F5     53             push ebx  
> 0FC802F6     56             push esi  
> 0FC802F7     57             push edi  
> 0FC802F8     8B45 08        mov eax,dword ptr ss:\[ebp+8\]  
> 0FC802FB     8B88 20010000  mov ecx,dword ptr ds:\[eax+120>  
> 0FC80301     894D F8        mov dword ptr ss:\[ebp-8\],ecx  
> 0FC80304     8B55 08        mov edx,dword ptr ss:\[ebp+8\]  
> 0FC80307     8B82 24010000  mov eax,dword ptr ds:\[edx+124>  
> 0FC8030D     8945 FC        mov dword ptr ss:\[ebp-4\],eax  
> 0FC80310     8B4D FC        mov ecx,dword ptr ss:\[ebp-4\]  
> 0FC80313     8B11           mov edx,dword ptr ds:\[ecx\]  
> 0FC80315     83E2 01        and edx,1  
> 0FC80318     85D2           test edx,edx  
> 0FC8031A     74 18          je short 0FC80334  
> 0FC8031C     8B45 FC        mov eax,dword ptr ss:\[ebp-4\]  
> 0FC8031F     8B08           mov ecx,dword ptr ds:\[eax\]  
> 0FC80321     83E1 02        and ecx,2  
> 0FC80324     85C9           test ecx,ecx  
> 0FC80326     74 0C          je short 0FC80334  
> 0FC80328     8B55 FC        mov edx,dword ptr ss:\[ebp-4\]  
> 0FC8032B     8B02           mov eax,dword ptr ds:\[edx\]  
> 0FC8032D     83E0 04        and eax,4  
> 0FC80330     85C0           test eax,eax  
> 0FC80332     75 04          jnz short 0FC80338  
> 0FC80334     33C0           xor eax,eax  
> 0FC80336     EB 6E          jmp short 0FC803A6  
> 0FC80338     8B4D F8        mov ecx,dword ptr ss:\[ebp-8\]  
> 0FC8033B     8179 0C C70000>cmp dword ptr ds:\[ecx+C\],0C7  
> 0FC80342     75 41          jnz short 0FC80385  
> 0FC80344     8B55 F8        mov edx,dword ptr ss:\[ebp-8\]  
> 0FC80347     8B45 08        mov eax,dword ptr ss:\[ebp+8\]  
> 0FC8034A     8B4A 14        mov ecx,dword ptr ds:\[edx+14\]  
> 0FC8034D     3B48 70        cmp ecx,dword ptr ds:\[eax+70\]  
> 0FC80350     75 0B          jnz short 0FC8035D  
> 0FC80352     8B55 0C        mov edx,dword ptr ss:\[ebp+C\]  
> 0FC80355     C702 00000000  mov dword ptr ds:\[edx\],0  
> 0FC8035B     EB 26          jmp short 0FC80383  
> 0FC8035D     8B45 F8        mov eax,dword ptr ss:\[ebp-8\]  
> 0FC80360     8B48 14        mov ecx,dword ptr ds:\[eax+14\]  
> 0FC80363     894D F4        mov dword ptr ss:\[ebp-C\],ecx  
> 0FC80366     8B55 08        mov edx,dword ptr ss:\[ebp+8\]  
> 0FC80369     8B42 5C        mov eax,dword ptr ds:\[edx+5C\]  
> 0FC8036C     50             push eax  
> 0FC8036D     8B4D 08        mov ecx,dword ptr ss:\[ebp+8\]  
> 0FC80370     8B51 58        mov edx,dword ptr ds:\[ecx+58\]  
> 0FC80373     52             push edx  
> 0FC80374     8B45 F8        mov eax,dword ptr ss:\[ebp-8\]  
> 0FC80377     8B48 04        mov ecx,dword ptr ds:\[eax+4\]  
> 0FC8037A     51             push ecx  
> 0FC8037B     FF55 F4        call dword ptr ss:\[ebp-C\]  
> 0FC8037E     8B55 0C        mov edx,dword ptr ss:\[ebp+C\]  
> 0FC80381     8902           mov dword ptr ds:\[edx\],eax  
> 0FC80383     EB 1C          jmp short 0FC803A1  
> 0FC80385     8B45 F8        mov eax,dword ptr ss:\[ebp-8\]  
> 0FC80388     8B48 14        mov ecx,dword ptr ds:\[eax+14\]  
> 0FC8038B     894D EC        mov dword ptr ss:\[ebp-14\],ecx  
> 0FC8038E     8B5D EC        mov ebx,dword ptr ss:\[ebp-14\]  
> 0FC80391     FFE3           jmp ebx                       ; 这里会跳到完全解压的主程序，就是 OEP 了。  
>   
> F8 一下,光明世界就在你的面前:  
> 004E8E90   /.  55           push ebp  
> 004E8E91   |.  8BEC         mov ebp,esp  
> 004E8E93   |.  6A FF        push -1  
> 004E8E95   |.  68 D8285B00  push Insta3De.005B28D8  
> 004E8E9A   |.  68 50E14E00  push Insta3De.004EE150        ;  SE handler installation  
> 004E8E9F   |.  64:A1 000000>mov eax,dword ptr fs:\[0\]  
> 004E8EA5   |.  50           push eax  
> 004E8EA6   |.  64:8925 0000>mov dword ptr fs:\[0\],esp  
> 004E8EAD   |.  83C4 A4      add esp,-5C  
> 004E8EB0   |.  53           push ebx  
> 004E8EB1   |.  56           push esi  
> 004E8EB2   |.  57           push edi  
> 004E8EB3   |.  8965 E8      mov dword ptr ss:\[ebp-18\],esp  
> 004E8EB6   |.  FF15 68F6590>call dword ptr ds:\[59F668\]    ;  KERNEL32.GetVersion  
> 004E8EBC   |.  A3 E0875F00  mov dword ptr ds:\[5F87E0\],eax  
> 004E8EC1   |.  A1 E0875F00  mov eax,dword ptr ds:\[5F87E0\]  
> 004E8EC6   |.  C1E8 08      shr eax,8  
> 004E8EC9   |.  25 FF000000  and eax,0FF  
> 004E8ECE   |.  A3 EC875F00  mov dword ptr ds:\[5F87EC\],eax  
> 004E8ED3   |.  8B0D E0875F0>mov ecx,dword ptr ds:\[5F87E0\]  
> 004E8ED9   |.  81E1 FF00000>and ecx,0FF  
>   
> 脱壳完成了.然后用 ImprtRec 修复一下即大功告成.菜鸟累了,该喝一杯去了!!!  
>   
> 【总结】我按照书上讲解将 VBOX 几个例子在 OD 下都脱壳了一遍,觉得有很多共同点:   
>   1 寻找 VBOX 关键的 CALL ESI 或 CALL EAX 等时一般前面都是 RETN 4 了.  
>   2 破解 VBOX4.3 与 4.2 保护时经过的 RET 次数都一样    
>   3 对于 VBOX4.3 如果利用 OD 直接脱壳，重建输入表有两种方式：  
>     利用方式 1 脱壳得到的文件可以直接运行，但是点击运行时时可能会出现内存不可以访问的错误。  
>     利用方式 2 脱壳得到的文件不能运行，但是利用 ImportRec 进行修复之后程序能够完全正常运行。  
>   4 短距离 CALL 一定要 F7 跟进去了. 

 4dd466d0e1e0cc123afbe504da2462ca