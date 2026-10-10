

# 我也帖一下我的第 3 阶段第 1 题的解决方案

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-279.htm`](https://www.pediy.com/kssd/pediy09/pediy09-279.htm)

> 这个病毒的多态的，因此不太好处理，其会感染 exe 和 scr 文件，  
> 在 PE 头中会留下 love(evol)标志，可能会多次感染。  
>   
> 病毒入口，作用为解码:  
>   
> 
> 代码:
> 
> * * *
> 
> `00415000 > /EB 00           jmp     short 00415002              ; 多态   00415002   \FC              cld   00415003    87DB            xchg    ebx, ebx   00415005    F5              cmc   00415006    55              push    ebp   00415007    8BEC            mov     ebp, esp   00415009    E8 11000000     call    0041501F   0041500E    E8 B7000000     call    004150CA   00415013    EB 00           jmp     short 00415015   00415015    87DB            xchg    ebx, ebx   00415017    EB 00           jmp     short 00415019   00415019    90              nop   0041501A    E9 32000000     jmp     00415051   0041501F    67:64:FF36 0000 push    [dword fs:0]   00415025    67:64:8926 0000 mov     [fs:0], esp   0041502B    012D BA504100   add     [4150BA], ebp   00415031    31DB            xor     ebx, ebx   00415033    68 00000100     push    10000                            ; UNICODE "=::=::\"   00415038    68 00000080     push    80000000   0041503D    53              push    ebx   0041503E    53              push    ebx   0041503F    68 00040000     push    400   00415044    68 00000080     push    80000000   00415049    53              push    ebx   0041504A    53              push    ebx   0041504B    FF15 9C634000   call    [<&KERNEL32.GetModuleHandleA>]   ; kernel32.GetModuleHandleA   00415051    EB 00           jmp     short 00415053   00415053    F9              stc   00415054    F5              cmc   00415055    89DB            mov     ebx, ebx   00415057    29C0            sub     eax, eax   00415059    FEC8            dec     al   0041505B    08C0            or      al, al   0041505D    74 04           je      short 00415063   0041505F  ^ 75 F8           jnz     short 00415059   00415061    EB 67           jmp     short 004150CA   00415063    F9              stc   00415064    FC              cld   00415065    89DB            mov     ebx, ebx   00415067    F8              clc   00415068    29D2            sub     edx, edx   0041506A    29C9            sub     ecx, ecx   0041506C    EB 00           jmp     short 0041506E   0041506E    F8              clc   0041506F    B1 1F           mov     cl, 1F        ; 密钥, 会变   00415071    8D52 01         lea     edx, [edx+1]   00415074    49              dec     ecx   00415075  ^ 75 FA           jnz     short 00415071   00415077    F9              stc   00415078    E8 00000000     call    0041507D      ; 得到加密代码的位置   0041507D    5F              pop     edi          ; 得到加密代码的位置   0041507E    87DB            xchg    ebx, ebx   00415080    F8              clc   00415081    81C7 5A000000   add     edi, 5A        ; 得到加密代码的位置   00415087    57              push    edi   00415088    29F6            sub     esi, esi   0041508A    81F6 00290000   xor     esi, 2900           ; 加密代码的长度，可能会变, 2900 为正常，   00415090    EB 00           jmp     short 00415092      ; 如果大于 2900 则多余部份是被偷掉的代码，需要复制回去   00415092    89D2            mov     edx, edx   00415094    8607            xchg    [edi], al   00415096    89D2            mov     edx, edx   00415098    F5              cmc   00415099    89D2            mov     edx, edx   0041509B    87DB            xchg    ebx, ebx   0041509D    66:31D0         xor     ax, dx              ; 解码方法，可能会变   004150A0    8607            xchg    [edi], al   004150A2    FC              cld   004150A3    87DB            xchg    ebx, ebx   004150A5    90              nop   004150A6    EB 00           jmp     short 004150A8   004150A8    89D2            mov     edx, edx   004150AA    FC              cld   004150AB    90              nop   004150AC    90              nop   004150AD    89D2            mov     edx, edx   004150AF    47              inc     edi   004150B0    4E              dec     esi   004150B1    EB 00           jmp     short 004150B3   004150B3    FC              cld   004150B4    09F6            or      esi, esi   004150B6  ^ 75 DC           jnz     short 00415094   004150B8    5F              pop     edi                         ; 解码完成   004150B9    BD 00000000     mov     ebp, 0   004150BE    8B55 F8         mov     edx, [ebp-8]   004150C1    67:64:8916 0000 mov     [fs:0], edx   004150C7    C9              leave   004150C8    FFE7            jmp     edi                         ; 跳到解密后的代码去`
> 
> * * *
> 
> 解密后的代码：  
> 代码:
> 
> * * *
> 
> `004150D7    E8 00000000     call    004150DC                    ; 计算返回地址   004150DC    8B0424          mov     eax, [esp]                     004150DF    FC              cld   004150E0    8998 33300000   mov     [eax+3033], ebx   004150E6    8B88 EE280000   mov     ecx, [eax+28EE]    ;偏移 28EE+5 的地方是是否偷 call 的标志   004150EC    8B5C24 04       mov     ebx, [esp+4]   004150F0    81E1 00000080   and     ecx, 80000000   004150F6    74 2D           je      short 00415125   004150F8    59              pop     ecx   004150F9    89B0 37300000   mov     [eax+3037], esi   004150FF    57              push    edi   00415100    8F80 3B300000   pop     [dword eax+303B]   00415106    80B8 F2280000 E>cmp     [byte eax+28F2], 0E8  ; 偏移 28F2+5 的地方是被偷掉的 call   0041510D    75 0D           jnz     short 0041511C   0041510F    0398 F3280000   add     ebx, [eax+28F3]   00415115    8B5B 02         mov     ebx, [ebx+2]   00415118    FF33            push    [dword ebx]   0041511A    EB 08           jmp     short 00415124   0041511C    8B98 F4280000   mov     ebx, [eax+28F4]   00415122    FF33            push    [dword ebx]   00415124    5B              pop     ebx   00415125    55              push    ebp   00415126    8BE8            mov     ebp, eax   00415128    816C24 04 DC800>sub     [dword esp+4], 80DC         ; 计算被偷代码的地址   00415130    81ED 05103600   sub     ebp, 361005   00415136    8B7C24 04       mov     edi, [esp+4]   0041513A    8DB5 00393600   lea     esi, [ebp+363900]   00415140    B9 00000000     mov     ecx, 0                                  ; 恢复被偷掉的代码   00415145    F3:A4           rep     movs [byte es:edi], [byte esi]          ; 恢复被偷掉的代码   00415147    E8 B1000000     call    004151FD   0041514C    8BC8            mov     ecx, eax   0041514E    E8 AA000000     call    004151FD   00415153    2BC1            sub     eax, ecx   00415155    74 79           je      short 004151D0   00415157    3D 000C0000     cmp     eax, 0C00   0041515C    77 72           ja      short 004151D0   0041515E    81E3 00F0FFFF   and     ebx, FFFFF000                           ; 如果差值小于 0C00 则真正的病毒体就会运行起来了   00415164    817B 4E 5468697>cmp     [dword ebx+4E], 73696854   0041516B    74 08           je      short 00415175   0041516D    81EB 00010000   sub     ebx, 100   00415173  ^ 75 EF           jnz     short 00415164   00415175    8BC3            mov     eax, ebx   00415177    0343 3C         add     eax, [ebx+3C]   0041517A    66:8138 5045    cmp     [word eax], 4550        ; 定位 Kernel32.dll 的 PE 头   0041517F  ^ 75 EC           jnz     short 0041516D   00415181    8B50 78         mov     edx, [eax+78]   00415184    03D3            add     edx, ebx   00415186    8B72 20         mov     esi, [edx+20]   00415189    8B4A 18         mov     ecx, [edx+18]   0041518C    03F3            add     esi, ebx   0041518E    51              push    ecx   0041518F    AD              lods    [dword esi]   00415190    03C3            add     eax, ebx   00415192    66:8178 02 7450 cmp     [word eax+2], 5074   00415198    75 09           jnz     short 004151A3   0041519A    8178 05 6F63416>cmp     [dword eax+5], 6441636F    ; 取得 GetProcAddress 的地址   004151A1    74 05           je      short 004151A8   004151A3  ^ E2 EA           loopd   short 0041518F   004151A5    59              pop     ecx   004151A6    EB 28           jmp     short 004151D0   004151A8    290C24          sub     [esp], ecx   004151AB    8B72 24         mov     esi, [edx+24]   004151AE    59              pop     ecx   004151AF    03F3            add     esi, ebx   004151B1    0FB7044E        movzx   eax, [word esi+ecx*2]   004151B5    8B7A 1C         mov     edi, [edx+1C]   004151B8    03FB            add     edi, ebx   004151BA    8B3487          mov     esi, [edi+eax*4]   004151BD    03F3            add     esi, ebx   004151BF    8D85 40113600   lea     eax, [ebp+361140]   004151C5    66:8B50 E9      mov     dx, [eax-17]   004151C9    E8 34000000     call    00415202   004151CE    EB 47           jmp     short 00415217   004151D0    8B85 F3383600   mov     eax, [ebp+3638F3]   004151D6    25 00000080     and     eax, 80000000   004151DB    74 1E           je      short 004151FB   004151DD    8B7C24 04       mov     edi, [esp+4]   004151E1    8DB5 F7383600   lea     esi, [ebp+3638F7]   004151E7    A5              movs    [dword es:edi], [dword esi]    ; 恢复被偷掉的 call   004151E8    A4              movs    [byte es:edi], [byte esi]   004151E9    8BB5 3C403600   mov     esi, [ebp+36403C]   004151EF    8BBD 40403600   mov     edi, [ebp+364040]   004151F5    8B9D 38403600   mov     ebx, [ebp+364038]   004151FB    5D              pop     ebp   004151FC    C3              retn                                        ; 跳到 oep 或被偷掉的 call 的地方继续执行`
> 
> * * *
> 
>   
> 我们要写清除该病毒修复文件的专杀工具只需要重点关注以上代码就可以了。除此之外需要注意一下病毒会 hook 几个函数,   
> 在访问文件和创建进程的时候进行感染:  
>   
> 代码:
> 
> * * *
> 
> `7C956DDF ntdll.ZwCreateFile            E8 65BA6303     call    7FF92849   7C956DE4                               BA 0003FE7F     mov     edx, 7FFE0300   7C956DE9                               FF12            call    [edx]   7C956DEB                               C2 2C00         retn    2C      7C956E7F ntdll.ZwCreateProcess         E8 19BA6303     call    7FF9289D   7C956E84                               BA 0003FE7F     mov     edx, 7FFE0300   7C956E89                               FF12            call    [edx]   7C956E8B                               C2 2000         retn    20      7C956E8F ntdll.ZwCreateProcessEx       E8 16BA6303     call    7FF928AA   7C956E94                               BA 0003FE7F     mov     edx, 7FFE0300   7C956E99                               FF12            call    [edx]   7C956E9B                               C2 2400         retn    24      7C95730F ntdll.ZwOpenFile              E8 82B56303     call    7FF92896   7C957314                               BA 0003FE7F     mov     edx, 7FFE0300   7C957319                               FF12            call    [edx]   7C95731B                               C2 1800         retn    18`
> 
> * * *
> 
>   
> 病毒感染可分为三种情况：  
> 1. 修改 EntryPoint 使程序入口指向病毒体入口，病毒运行后会路到原始入口点(OEP)继续执行  
> 2. 覆盖 EntryPoint 处的代码，病毒运行后会恢复 EntryPoint 的原始代码，然后跳回去继续执行  
> 3. 修改 EntryPoint 下的某一个 call, 使其 call 到病毒体(可能会修改多个，多次感染的情况), 病毒体运行后会恢复这个 call，然后  
>    跳回这个 call 的原始地址去继续执行。  
>   
> 我们要写清除工具需要区分出这 3 种情况:  
> 1. 取得被加密代码的长度，如果长度大于$2900 则代表其是覆盖 oep 类型的(即上面说的第 2 种情况)  
> 2. 如果长度等于$2900 则代表其是修改 EntryPoint 类型的(即上面说的第 1 种情况)  
> 3. 解密代码，判断偏移 28EE+5 的地方是是否偷 call 的标志，如果是则代表是偷 call 类型的(即上面说的第 3 种情况)  
>   
> 分析清楚了病毒的形态及原理我们便可以写专杀工具了，程序基本思路如下：  
>   
> 1. 程序恢复内存中被 Hook 的 api：  
>     复制一份 ntdll.dll 到临时目录，重命名为 res.dll，加载，然后依次对各进程中的被 hook 的 api 进行检查, 如果发现前 5 个字节  
>     跟临时目录下的 ntdll.dll 的相应函数对应的头 5 个字节不一样，则代表被 hook 了，于是恢复之。  
>   
> 2. 扫描内存：  
>     取得进程列表，对进程所在路径下的文件进行扫描。如果发现被染毒则会提示用户，是否需要结束进程杀毒。(这样做比较人性化，  
>   得到用户的许可才杀进程，举个例，用户开的一个 Word 程序被染毒，而用户此时没有存盘，如果强杀进程，用户就要哭了。。。  
>   可能有其他类似情况)  
>   
> 3. 杀毒：  
>     查杀毒均采用 UNICODE 路径，这样可以避免一些路径太深(超过 260 字节)而查杀不到。  
>     3.1) 从 PE 头的偏移 8 的位置判断是否病毒标志 love(evol)，如果有则是病毒  
>     3.2) 如果扩展扫描模式打开的情况下会对代码进行分析，满足代码特征的就是病毒。  
>     3.3) 确定是病毒后就该清除病毒了, 在确认是病毒的情况下会有如下处理：  
>   3.3.1) 确定病毒类型，方法见上面  
>         3.3.2) 根据解密后的代码确定病毒体执行完后继续运行的地址，如果是第 1,2 种情况的病毒则返回地址是 OEP,第 3 种情况的  
>                返回地址是被偷的 Call, 这是最复杂的，因为病毒是多态的，因此要解析很多代码情况, 具体见程序代码  
>         3.3.2) 根据病毒类型来恢复代码，如果长度大于$2900，则把多出的部份恢复到 OEP 的位置，否则恢复被偷的 call  
>         3.3.3) 清除 Image.PNtHeaders.FileHeader.TimeDateStamp 的病毒标志  
>         3.3.4) 保存文件，重复 3.1(因为可能被多次感染)  
>   
> 4. 关于解密代码的 Key 的运算方法  
>     因为解密后的代码基本上来说是固定的，我们可以从前两个指令的字节码来推出解密用的 Key、步进和运算方法，具体见  
>     代码中的  TScanThread.GetDecryptKey, 解码 Key 定义如下：  
>     
>   TDecodeMethod = (METHOD\_XOR, METHOD\_SUB, METHOD\_ADD);  
>     
>   TDecryptKey = packed record  
>     Key: Byte;  
>     IncQty: Byte;  
>     DecodeMethod: TDecodeMethod;  
>   end;  
>   
> 程序源代码见附件 

 4dd466d0e1e0cc123afbe504da2462ca