

# Themida IAT 处理部分的简单分析

> 原文：[`www.pediy.com/kssd/pediy10/73227.html`](https://www.pediy.com/kssd/pediy10/73227.html)

Themida IAT 处理部分的简单分析  
下面类似的代码大家可以随便找个 themida 1.9.x 加密的文件,然后运行 fxyang 的脚本,运行到 370 行左右  
就到了下面代码的 范围  
  
(本文所列代码已经去除变形代码,有些代码使用等效代码代替)  
  
有不足之处,请大家批评指正  
  
D331F95: pDllBase  
D333185: NumberOfNames  
D3316A9: EndOfFuncAddr  
D330EB5: Base(输出表)  
D333671; AddressOfFunctions  
D3337E1: AddressOfNames  
D331739: AddressOfNameOrdinals  
  
  
038F6B60   mov     ebx, dword ptr \[ebp+D331F95\]  
038F6B83   mov     ecx, dword ptr \[ebx\]  
038F6BB8   cmp     ecx, 0  
038F6BBB   je      038FAA6B  
038F6BD1   push    eax  
038F6BFE   push    ecx  
038F6C15   pushad  
038F6C17   xor     eax, eax  
038F6C1A   mov     dword ptr \[ebp+D333725\], eax  
038F6C44   mov     esi, 3C                                ;熟悉的常数  
038F6C61   add     esi, dword ptr \[esp+20\]  
038F6C84   lods    word ptr \[esi\]  
038F6CAC   add     eax, dword ptr \[esp+20\]  
038F6CB6   push    dword ptr \[eax+78\]                        ；典型的寻找输出表  
038F6CAB   pop     esi  
038F6CD9   add     esi, dword ptr \[esp+20\]  
038F6CEA   push    dword ptr \[esi+18\]                        ；NumberOfNames？  
038F6CED   pop    edi  
038F6CF5   mov     dword ptr \[ebp+D333185\], edi  
038F6CFC   test    edi, edi  
038F6CFE   jnz     038F6D36  
038F6D36   push    ecx  
038F6D4F   mov    edx,    edi  
038F6D67   imul    edx, edx, 4  
038F6D7A   mov     dword ptr \[ebp+D3316A9\], edx  
038F6D8F   push    4  
038F6D98   push    1000  
038F6DA7   push    edx  
038F6DCD   push    0  
038F6DD9   call    dword ptr \[ebp+D33319D\]                ;VirutalAlloc?  
038F6DE6   mov     dword ptr \[ebp+D332CE9\], eax                ;保存分配的空间地址  
038F6E0C   mov     edx, eax  
038F6E1C   pop      ecx                                       
<< call 038FD38A  >>   
038FD38A   push    eax                                          
038FD3AB   mov     eax, dword ptr \[esi+10\]                ; Base  
038FD3B6   mov     dword ptr \[ebp+D330EB5\], eax  
038FD3E3   pop     eax  
038FD3F8   mov     ebx, dword ptr \[esi+1c\]                ; AddressOfFunctions  
038FD402   lea     edi, dword ptr \[ebp+D333671\]  
038FD421   lods    dword ptr \[esi\]                        << AddressOfFunctions  
038FD441   add     eax, dword ptr \[esp+24\]  
038FD463   stos    dword ptr es:\[edi\]  
038FD471   lea     edi, dword ptr \[ebp+D3337E1\]  
038FD478   lods    dword ptr \[esi\]                         << AddressOfNames  
038FD4AA   add     eax, dword ptr \[esp+24\]  
038FD4B8   push    eax  
038FD4CF   stos    dword ptr es:\[edi\]  
038FD4E9   lea     edi, dword ptr \[ebp+D331739\]  
038FD50C   lods    dword ptr \[esi\]                         << AddressOfNameOrdinals  
038FD53B   add     eax, dword ptr \[esp+28\]  
038FD55B   stos    dword ptr es:\[edi\]  
038FD586   pop     esi  
038F6E5A   push    esi  
038F6E7B   lods    dword ptr \[esi\]                        提取函数名  
038F6E87   add     eax, dword ptr \[esp+24\]  
038F6E96   xor     eax, edi  
038F6E98   xor     edi, eax  
038F6E9A   xor     eax, edi  
038F6EA0   mov     ebx, edi  
038F6EA4   push    edi  
038F6EC9   xor     al, al  
038F6ECC   rep scas    byte ptr es:\[edi\]                寻找字符串结尾  
038F6EE1   pop     esi  
038F6EFA   sub     edi, ebx  
038F6F18   push    edx  
038F6F3C   mov     edx, edi                                ;函数名长度  
038F6F54   mov     edi, dword ptr \[ebp+D331799\]  
038F6F5B   mov     ecx, -1  
038F6F78   xor     eax, eax                            开始处理函数名（crc32?）  
038F6F9B   mov     al, byte ptr \[esi\]                  
038F6F9F   xor     al, cl  
038F6FAE   inc     esi  
038F6FC0   mov     eax,   dword ptr \[edi+eax\*4\]  
038F6FCB   shr     ecx, 8  
038F6FCF   xor     ecx, eax  
038F6FEC   dec     edx  
038F6FED   jnz     038F6F78  
038F702B   mov     eax, ecx  
038F703B   not     eax  
038F7050   pop     edx  
038F7070   mov     dword ptr \[edx\],eax  
038F7098   add     edx, 4  
038F70C6   push    edx  
038F70CF   inc     dword ptr \[ebp+D333725\]  
038F70F0   mov     edx, dword ptr \[ebp+D333725\]  
038F7102   cmp     dword ptr \[ebp+D333185\], edx  
038F7108   je      038F7185  
038F7112   pop     edx  
038F713E   pop     esi  
038F715D   add     esi, 4  
038F7167   jmp     038F6E5A  
  
038F7197   pop     edx  
038F71AB   pop     esi  
038F71C1   popad  
038F71D5   pop     ecx  
038F71EF   pop     eax  
  
038F7202  mov     dword ptr \[ebp+D332401\], 0  
038F720C  mov     dword ptr \[ebp+D332D29\], 0  
038F7216  cmp     dword ptr \[ebp+D3E344A\], 0  
038F721D  je      short 038F7227  
038F721F  lea     ebx, dword ptr \[ebp+D3C15FF\]  
038F7225  call    ebx                                
  
038F7227  inc     dword ptr \[ebp+SelfCheckFlag\]  
038F7248  cmp     dword ptr \[ebp+SelfCheckFlag\], 64                  
038F724F  jb      038F7348                                ;貌似自校验，100 次循环检查一下  
                                                ；这里跳转就是不检查  
038F734A  mov     ecx, 020A27F1  
038F7371  mov     edx, 777F107C  
038F738A  lods    dword ptr \[esi\]                        ;又是处理加密表  
038F73A9  mov     dword ptr \[ebp+EnCryIAT\], esi  
038F73C4  mov     dword ptr \[esi-4\], 0  
038F73CD  cmp     eax, EEEEEEEE  
038F73D2  jnz     038F749A                        ；E 后面连接 D ，下一个输入文件  
038F73E9  cmp     dword ptr \[esi\], DDDDDDDD  
038F73EF  jnz     038F749A                                  
038F7413  mov     dword ptr \[esi\], 0                ；如果是 dddddddd 则清零，跳转  
038F7446  add     esi, 4  
038F745A  mov     dword ptr \[ebp+EnCryIAT\], esi  
038F748E   jmp     038FA981  
  
  
038F749B  mov     ebx, eax  
038F74B3  xor     eax, dword ptr \[ebp+D330881\]        ;与前一个组数据头异或  
038F74D2  ror     eax, 3  
038F74E9  sub     eax, edx                                ;上面给的固定值 777F107C  
038F7509  rol     eax, 10  
038F7523  xor     eax, ecx                         ;   
038F7526  mov     dword ptr \[ebp+D330881\], ebx  
038F752D  cmp     eax, 10000                       ; 计算结果是否小于 10000  
038F7532  jnb     038F77DE  
038F754F  cmp     dword ptr \[esi\], BBBBBBBB              ；下一个数据是否是 B  
038F7555  jnz     038F77DE  
038F7577  mov     dword ptr \[esi\], 0  
038F75B2  add     esi, 4  
038F75EF  mov     dword ptr \[ebp+ EnCryIAT\], esi  
038F7602  mov     ebx, dword ptr \[ebp+D331F95\]  
038F7621  mov     ecx, dword ptr \[ebx\]  
038F7646  mov     edx, eax  
038F7660  pushad  
038F766F  mov     eax, edx  
038F7680  sub     eax, dword ptr \[ebp+D330EB5\]  
038F7687  shl     eax, 2  
038F769C  add     eax, dword ptr \[ebp+D333671\]  
038F76A3  xchg    eax,esi  
038F76B8  lods    dword ptr \[esi\]  
038F76CF  add     eax, ecx  
038F76EA  mov     dword ptr \[esp+1C\], eax  
038F76FB  mov     byte ptr \[ebp+D3E3526\], 0  
038F770E  mov     dword ptr \[ebp+D3E352F\], 0  
038F7719  mov     edi, dword ptr \[ebp+D331F95\]  
038F7728  mov     edi, dword ptr \[edi\]  
038F7753  cmp     edi, dword ptr \[ebp+D3308AD\]  
038F7759  je      038F77B6  
038F7771  cmp     edi, dword ptr \[ebp+D333319\]  
038F7777  je      038F77B6  
038F778F  cmp     edi, dword ptr \[ebp+D333105\]  
038F7795  jnz     038F77C9  
038F77B6  mov     byte ptr \[ebp+D3E3526\], 1  
038F77C9  popad  
038F77D8  jmp     038F7D55  
  
038F77DE  push   ecx  
038F77ED  push    edx  
038F7824  xor     ecx, ecx                           
038F7834  mov     edx, dword ptr \[ebp+D332CE9\]                ; 加密的函数名 数组  
038F783B  cmp     eax, dword ptr \[edx\]                        ;循环找到和 eax 相同的数值  
038F783D  je      038F78EA                                                                          
038F785E  add     edx, 4  
038F785E  inc     ecx                                        ；记录序号  
038F787E  cmp     ecx, dword ptr \[ebp+D333185\]  
038F7884  jnz     038F783B  
  
038F78EA  mov     dword ptr \[ebp+D33725\], ecx                ;保存序号  
038F78FE  mov     byte ptr \[ebp+Flag526\], 0                ；flag 置 0  
038F7905  mov     dword ptr \[ebp+Flag52F\], 0                ；flag 置 0  
038F7918  mov     ecx, dword ptr \[ebp+D331F95\]                ;这里储存的是指针  
038F791E  mov     ecx, dword ptr \[ecx\]                                ;dll 文件基地址  
038F7920  cmp     ecx, dword ptr \[ebp+D3308AD\]    比较是否是 user32.dll  
038F7926  je      short 038F7938                                          
038F7928  cmp     ecx, dword ptr \[ebp+D333319\]     比较是否是 advapi32.dll  
038F792E  je      short 038F7938  
038F7930  cmp     ecx, dword ptr \[ebp+kernel32Base\]  比较是否是 Kernel32.dll  
038F7936  jnz     short 038F7964  
038F7938  mov     byte ptr \[ebp+Flag526\], 1                ；进入特殊 dll 标志  
038F793F  lea     edx, dword ptr \[ebp+D3E3533\]                ；4 个特殊函数？  
038F7945  lea     ecx, dword ptr \[ebp+D3E3543\]  
038F794B  jmp     short 038F7960  
038F794D  cmp     dword ptr \[edx\], eax                  
038F794F  jnz     short 038F795D  
038F7951  mov     dword ptr \[ebp+Flag52F\], 1                ；4 个特殊函数标志  
038F795B  jmp     short 038F7964  
038F795D  add     edx, 4  
038F7960  cmp     edx, ecx  
038F7962  jb      short 038F794D  
038F7979  pop     edx                  
038F797E  pop     ecx  
038F7986  push    esi  
038F79AA  mov     ebx, dword ptr \[ebp+D331F95\]                ；pCurDllBase  
038F79D2  mov     ecx, dword ptr \[ebx\]                  ;dll 文件基地址  
038F79E7  mov     eax, dword ptr \[ebp+D33725\]                ;上面找到的序号  
038F79FE  shl     eax, 1  
038F7A01  add     eax, dword ptr \[ebp+D331739\]     ; AddressOfOrdinals  
038F7A08  xor     esi, esi  
038F7A19  xchg           eax,esi  
038F7A2C  lods    word ptr \[esi\]  
038F7A49  shl     eax, 2                                                          
038F7A4D  add     eax, dword ptr \[ebp+D333671\]     ; AddressOfFunctions  
038F7A5B  xchg           eax,esi  
038F7A73  lods    dword ptr \[esi\]  
038F7A8F  add     eax, ecx                          ;前面是 Themida 自己的 GetProcAddressByKey  
038F7ADC  cmp     dword ptr \[eax\], 0E8               ；检查 api 函数第一个字节（dword 比较？）  
038F7AE2  jnz     038F7C7E  
038F7C8A  cmp     dword ptr \[eax\], 6C687069                ；iphlpapi 奇怪的检查，看不懂  
038F7C90  jnz     038F7D20  
038F7CA0  cmp     dword ptr \[eax+4\], 69706170  
038F7CA7  jnz     038F7D20  
  
038F7D23  pop     esi  
038F7D55  mov     ebx, dword ptr \[ebp+D330E11\]                ；标志位 E11  
038F7D80  dec     ebx  
038F7D81  je      038F7EA3                                ；检查 ebx 是否为 1  
038F7D8D  mov     ebx, dword ptr \[ebp+kernel32Base\]     ; kernel32.7C800000  
038F7DA7    sub     ebx, ecx                                 比较是否是 Kernel32.dll  
038F7DA9    je      038F7EA3                        ;fxyang 的脚本修改这里避开加密.新版已经无效  
038F7DD7    mov     ebx, dword ptr \[ebp+ user32\]       
038F7DE7    clc  
038F7DE8    sub     ebx, ecx                                比较是否是 user32.dll  
038F7DEA    je      038F7EA3                        ;fxyang 的脚本修改这里避开加密.新版已经无效  
  
038F7E01    mov     ebx, dword ptr \[ebp+ advapi32\]     ; 3844475  
038F7E14    sub     ebx, ecx                                 比较是否是 advapi32.dll  
038F7E16    je      038F7EA3                        ;fxyang 的脚本修改这里避开加密.新版已经无效  
  
//正常 dll 处理过程  
038F7E1C   cld  
038F7E1D   lea     ebx, dword ptr \[ebp+D3E9915\]     ; 38faa71  
038F7E39   call    ebx                              ;038FAA71  
038F7E48   mov     edi, eax  
038F7E55   cmc  
038F7E56   mov     dword ptr \[ebp+CurAPIFunc\], eax        ；存储当前正在处理的函数地址  
038F7E73   jmp     038FA189  
  
;特殊 dll 处理过程  
038F7EA3   lea     ebx, dword ptr \[ebp+D3E9915\]          
038F7EB3   call    ebx                              ;038FAA71  
038F7EC8   cmp     dword ptr \[ebp+D330E11\], 0            ；这个标志估计和加密 dll 有关  
038F7ECF   je      038F7F38  
  
038F7F38   cmp     eax, dword ptr \[ebp+D33001D\]     ; kernel32.ExitProcess  
038F7F3E   jnz     038F7FA0  
038F7FA0   cmp     eax, dword ptr \[ebp+D33001D\]     ; kernel32.ExitProcess  
038F7FA6   je      038F7E1D  
038F7FC7   cmp     dword ptr \[ebp+D3E342E\], 1  
038F7FCE   jnz     038F8035  
038F8035   xor     edi, edi                           
038F803E   cmp     dword ptr \[ebp+D33364D\], 0            ；貌似没有作用  
038F8045   je      038F93EC  
038F940C   cmp     eax, dword ptr \[ebp+D3321C9\]     ; USER32.wsprintfA  
038F9412   jnz     038F9469  
038F9469   cmp     eax, dword ptr \[ebp+D3327BD\]     ; kernel32.RaiseException  
038F946F   jnz     038F9555  
038F9555   cmp     eax, dword ptr \[ebp+D3E343E\]     ; ntdll.RtlEnterCriticalSection  
038F955B   je      038F95B2  
038F9599   cmp     eax, dword ptr \[ebp+D3E3442\]     ; ntdll.RtlLeaveCriticalSection  
038F959F   jnz     038F95B8  
038F95DF   mov     esi, eax  
038F967A   mov     dword ptr \[ebp+D33262D\], esi     ; kernel32.OutputDebugStringA  
038F9681   mov     dword ptr \[ebp+D3309A1\], esi  
038F97ED   mov     edi, dword ptr \[ebp+D3305A5\]  
038F9802   mov     dword ptr \[ebp+D332029\], 0  
038F983C   mov     dword ptr \[ebp+D3309A1\], esi     ; kernel32.OutputDebugStringA  
038F984C   lea     ebx, dword ptr \[ebp+D3EBEE8\]  
038F9864   call    ebx                                        ;结束以后 eax=0  
038F9881   or      eax, eax  
038F9883   je      038F98B1  
038F9890   jmp     038F9945                                <<这里跳出 寻找到 jmp 指令  
038F98B1   lea     ebx, dword ptr \[ebp+D3A365C\]  
038F98C9   call    ebx                              ; esi 移动到下一个指令  
038F98CB   jnb     038F983C  
038F98D4   mov     esi, dword ptr \[ebp+D3309A1\]  
038F98E0   jmp     038F98F5  
  
038F9945   mov     eax, dword ptr \[ebp+D33262D\]  
038F9952   mov     dword ptr \[ebp+D3309A1\], eax     ; kernel32.OutputDebugStringA  
  
038F999A   cmp     byte ptr \[esi\], 0E8  
038F999D   jnz     038F9A85  
038F99AE   cmp     dword ptr \[ebp+D330E21\], 0  
038F99B5   je      038F9A85  
038F99E3   mov     al, ??  
038F99FE   stos    byte ptr es:\[edi\]  
038F9A06   mov     eax, esi  
038F9A11   sub     eax, edi  
038F9A32   sub     eax, 4  
038F9A4E   stos    dword ptr es:\[edi\]  
038F9A63   jmp     038F9C9C  
  
038F9A85   lea     ebx, dword ptr \[ebp+D3EA5A3\]  
038F9A9E   call    ebx                              ;038FB6FF  
038F9AA1   lea     ebx, dword ptr \[ebp+D3EB8F9\]  
038F9AB6   call    ebx                              ;038FCA55  
038F9ABF   or      eax, eax  
038F9AC1   jnz     038F9AD7  
038F9AC8   jmp     038F9B05  
038F9AD7   add     dword ptr \[ebp+D3309A1\], 5  
038F9AEE   jmp     038F999A  
  
038F9B05   lea     ebx, dword ptr \[ebp+D3EBAC5\]  
038F9B17   call    ebx                              ;038FCC21  
038F9B36   or      eax, eax  
038F9B38   jnz     038F9B6C  
038F9B52   jmp     038F9B9C  
038F9B78   add     edx, ebx  
038F9 B80   jmp     038F9C9C  
  
038F9B9C   lea     ebx, dword ptr \[ebp+D3A365C\]  
038F9BB9   call    ebx                              ;038B47B8  
038F9BE5   mov     ecx, dword ptr \[ebp+D3309A1\]     ; kernel32.OutputDebugStringA  
038F9BFD   mov     dword ptr \[ebp+D3309A1\], esi     ; kernel32.7C859D7D  
038F9C1A   sub     ecx, esi  
038F9C39   not     ecx  
038F9C3D   inc     ecx  
038F9C5A   sub     esi, ecx  
038F9C66   rep     movs byte ptr es:\[edi\], byte ptr \[esi\]        <,将 api 函数体复制到 edi  
038F9C6F   jmp     038F999A  
  
038F9C9C   lea     ebx, dword ptr \[ebp+D3C15FF\]  
038F9CA3   call    ebx                                       ;   
038F9CBF   mov     eax, edi                                ；复制代码的末尾  
038F9CC9   sub     eax, dword ptr \[ebp+D3305A5\]                ；-代码头，得到长度  
038F9CE5   mov     dword ptr \[ebp+D331689\], eax                ；保存长度  
038F9CF9   mov     eax, dword ptr \[ebp+D3305A5\]  
038F9D00   push    edi  
038F9D00   push    eax  
038F9D39   lea     ecx, dword ptr \[ebp+D3C183C\]  
038F9D46   call    ecx                                    ;038D2998  
05AD0059   add     byte ptr \[eax\], al  
038F9D5B   push    eax  
038F9D72   push    edi  
038F9D7B   mov     eax, dword ptr \[ebp+D3305A5\]  
038F9D9C   push    eax  
038F9DB6   lea     ecx, dword ptr \[ebp+D3C1974\]  
038F9DD2   call    ecx                                      ;生成一份变态的代码，  
038F9DD7   mov     edx, eax                                ；代码结尾  
038F9E0A   mov     ecx, eax                                ；四处乱放  
038F9E11   sub     ecx, dword ptr \[ebp+D3326CD\]                又要计算长度了  
038F9E21   cmp     dword ptr \[ebp+D332A99\], 0  
038F9E28   je      038F9F0E  
038F9F0E   push    ecx  
038F9F44   mov    eax,ecx  
038F9F54   sub     eax, 1  
038F9F79   or      eax, 0FFF                                ;计算长度是否超出页面？  
038F9F8E   inc     eax  
038F9F96   mov     dword ptr \[ebp+D332475\], eax  
038F9FB6   add     dword ptr \[ebp+D33373D\], eax  
038F9FC9   mov     dword ptr \[ebp+D332A99\], 0  
038F9FDA   push    40  
038F9FE9   push    1000  
038F9FFC   push    ecx  
038FA012   push    0  
038FA01F   call    dword ptr \[ebp+D33319D\]                ;VirutalAlloc?  
038FA026   call    dword ptr \[ebp+D3303C1\]                ;03845981  
038FA03E   mov     dword ptr \[ebp+D3304A1\], eax                ；新分配的地址  
038FA04B   mov     dword ptr \[ebp+D332DD1\], eax  
038FA067   pop     ecx  
038FA07C   push    dword ptr \[ebp+D332DD1\]                新地址  
038FA094   push    dword ptr \[ebp+D3326CD\]                变态代码地址  
038FA0A1   push    edi                                        api 代码结束地址  
038FA0CD   push    dword ptr \[ebp+D3305A5\]                api 代码开始地址  
038FA0F7   lea     eax, dword ptr \[ebp+D3C3149\]  
038FA118   call    eax                                   ; 038D42A5 打算干什么呢？  
038FA11B   add     dword ptr \[ebp+D332A99\], ecx  
038FA154   mov     edi, dword ptr \[ebp+D332DD1\]  
038FA15B   mov     esi, dword ptr \[ebp+D3326CD\]  
038FA16B   rep     movs byte ptr es:\[edi\], byte ptr \[esi\] 移动代码到新的地址  
  
038FA189   mov     esi, dword ptr \[ebp+EnCryIAT\]                ；加密表地址  
038FA19D   lods    dword ptr \[esi\]  
038FA1B7   mov     dword ptr \[esi-4\], 0                                ；清零  
038FA1CE   mov     ecx, 2  
038FA1E9   or      ecx, ecx  
038FA1EB   jnz     038FA211  
038FA200   mov     dword ptr \[ebp+Flag52F\], 1  
038FA20A   ja      038FA211  
  
038FA211   cmp     byte ptr \[ebp+Flag526\], 0        ; 38f4682  
038FA218   je      038FA2B2  
038FA22A   cmp     dword ptr \[ebp+Flag52F\], 0  
038FA231   jnz     038FA2B2  
038FA255   call    dword ptr \[ebp+D3320C1\]     <<关键函数,这个函数就是处理加密 api 的 iat 的  
                                        函数很复杂,我是没精力看了,依靠各位了  
  
038FA271   mov     byte ptr \[ebp+Flag526\], 0  
038FA284  jmp     038FA393  
  
038FA2B2   rol     eax, 5  
038FA2CA  add     eax, 020A27F1                        ;对加密表的数据进行处理  
038FA2ED  add     eax, dword ptr \[ebp+D3319B9\]             ;00400000  
                                                                                                ;加上基地址  
038FA2F4  mov     ecx, dword ptr \[ebp+CurAPIFunc\]     ; IMM32.ImmCreateContext  
                                                                                                ；上面存储的函数地址  
038FA301  mov     dword ptr \[eax\], ecx                ;填入 iat  
038FA307  lods    dword ptr \[esi\]                        ;载入加密表的下一个数据  
038FA315  mov         dword ptr \[esi-4\],0                ;清零  
  
038FA330  mov     dword ptr \[ebp+EnCryIAT\], esi         ; 存储下个处理的加密表地址  
038FA35A  cmp     eax, -1                                ;比较数据是否是 FFFFFFFF  
038FA35D  jnz     038FA421  
038FA379  cmp     dword ptr \[esi\], DDDDDDDD  
038FA37F  jnz     038FA421  
  
038FA3AA  mov     dword ptr \[esi\], 0                ; 如果 FFFFFFFF 后面是 DDDDDD，  
038FA3E8  add     esi, 4                                                                                                  
038FA3FE   mov     dword ptr \[ebp+EnCryIAT\], esi        ；那么跳转到 202 这个地址  
038FA40B  jmp     038F7202  
  
038FA421  rol     eax, 3                                ;对加密表的数据进行处理  
038FA43B  add     eax, dword ptr \[ebp+D3319B9\]        ;加上基地址  
038FA449  cmp     dword ptr \[ebp+D330F15\], 1  
038FA450  je      038FA943  
038FA476  cmp     dword ptr \[esi\], AAAAAAAA                ;A 代表 JMP，否则为 Call  
038FA47C  jnz     038FA54F  
038FA488  add     esi, 4  
038FA4A5  mov     dword ptr \[esi-4\], 0  
038FA4F4  push    eax                                  ;eax,edi 交换  
038FA4F5  push    edi  
038FA4F6  pop     eax  
038FA4F7  pop          edi  
038FA52D  mov     al, 0E9  
038FA53F  jmp     038FA589  
  
038FA54F   xor     eax, edi  
038FA551   xor     edi, eax  
038FA553   xor     eax, edi  
038FA573   mov     al, 0E8  
  
038FA589  push    eax  
038FA5B0  cmp     dword ptr \[ebp+D330E11\], 1                ;检查 FlagE11 是否 1  
038FA5B7  je      038FA7E6  
038FA5CC  mov     eax,100  
038FA5EA  cmp     dword ptr \[ebp+D3E344A\], 0                ；标志位 Flag 44A  
038FA5F1  je      038FA633  
038FA5FD  jmp     038FA616  
038FA616  lea     ebx, dword ptr \[ebp+D3C4787\]        ;38faa71  
038FA630  call    ebx                                ; 返回后 eax 改变  
038FA633  cmp     byte ptr \[edi\], 90  
038FA636  je      038FA66A  
038FA66A  cmp     dword ptr \[ebp+D3E354F\], 0  
038FA671  je      038FA72E  
038FA741  mov     ebx, 0  
038FA766  cmp     eax, 50  
038FA769  jb      038FA7E6  
038FA78D  mov     al, 90  
038FA791  stos    byte ptr es:\[edi\]  
038FA79C  pop     eax                                ;上面计算的 E9  
038FA7D0  stos    byte ptr es:\[edi\]  
038FA8CC  mov     eax, dword ptr \[ebp+CurAPIFunc\]    ; IMM32.ImmIsUIMessageA  
038FA8DF  sub     eax, edi  
038FA921  sub     eax, 4                        ;3 行计算 jmp 地址  
038FA92B  stos    dword ptr es:\[edi\]                ;存入 jmp apiaddress  
038FA943  lods    dword ptr \[esi\]  
038FA954  mov     dword ptr \[esi-4\], 0  
038FA972  jmp     038FA330  
  
  
038FA981   mov     dword ptr \[ebp+D331BD5\], esi   
038FA991   push    edx  
038FA9A0   push    8000  
038FA9A6   push    0  
038FA9A9   push    dword ptr \[ebp+D332CE9\]                ；5af0000  
038FA9C5   call    dword ptr \[ebp+D331F49\]                ；VirtualFree ???  
038FA9DF   pop     edx  
038FA9FE   mov     ecx, dword ptr \[ebp+D331F95\]  
038FAA35   mov     dword ptr \[ecx\], 0  
038FAA46   add     ecx, 4  
038FAA50   mov     dword ptr \[ebp+D331F95\], ecx  
038FAA5D   jmp     038F6B60