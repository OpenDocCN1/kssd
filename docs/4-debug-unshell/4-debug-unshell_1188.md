

# Yoda's Crypter v1.2 脱壳分析

> 原文：[`www.pediy.com/kssd/pediy11/111252.html`](https://www.pediy.com/kssd/pediy11/111252.html)

【前言】时断时续的学习破解，这是我在本坛贴出的第二篇学习笔记，比前一篇稍有了点进步，与刚学破解的朋友们交流，大牛们看了别笑话水平太低，给我指点就好了![](img/234a76e67df951b6551d305c177a8b78.jpg "扮鬼脸")  
【软件】加了 yoda's cryptor 1.2 壳的 win98 记事本程序，由天草老师教程中提供。  
【工具】OD、PEiD、lordPE、ImportREC  
  
用 PEiD 查一下，是 yoda's cryptor 1.2 壳，下面 OD 载入，开始分析。  
  
首先是壳引导部分，功能是解密壳主体部分代码，解密完成后就将控制权交给主体代码。  
0040D060 Y>  60                pushad                                   ; 程序入口，载入时停在此处  
0040D061     E8 00000000       call Yoda's\_C.0040D066  
0040D066     5D                pop ebp                                  ; 这两行是壳代码自定位  
0040D067     81ED F31D4000     sub ebp,Yoda's\_C.00401DF3                ; 计算偏移量，用于取加壳时保存的信息的重定位  
0040D06D     B9 7B090000       mov ecx,97B                              ; 要解密代码的字节数为 97Bh  
0040D072     8DBD 3B1E4000     lea edi,dword ptr ss:\[ebp+401E3B\]        ; 被解密代码的起始位置，解密完成全就被执行  
0040D078     8BF7              mov esi,edi  
0040D07A  /  AC                lods byte ptr ds:\[esi\]                   ; 逐字节取出进行解密，以下就是解密代码  
0040D07B  |  2AC1              sub al,cl  
0040D07D  |  EB 01             jmp short Yoda's\_C.0040D080  
0040D07F  |  C2 348F           retn 8F34        ; C2 花指令，可将 C2 改为 90（nop）。以下几处同此。  
  
0040D080  |  34 8F             xor al,8F  
0040D082  |  F9                stc  
0040D083  |  02C1              add al,cl  
0040D085  |  F8                clc  
0040D086  |  34 AE             xor al,0AE  
0040D088  |  2C E9             sub al,0E9  
0040D08A  |  FEC8              dec al  
0040D08C  |  02C1              add al,cl  
0040D08E  |  FEC8              dec al  
0040D090  |  04 CD             add al,0CD  
0040D092  |  90                nop  
0040D093  |  EB 01             jmp short Yoda's\_C.0040D096  
0040D095  |  C2 2AC1           retn 0C12A        ; 又是 C2 花指令  
  
0040D096  |  2AC1              sub al,cl  
0040D098  |  C0C8 5E           ror al,5E  
0040D09B  |  FEC8              dec al  
0040D09D  |  2AC1              sub al,cl  
0040D09F  |  04 7D             add al,7D  
0040D0A1  |  EB 01             jmp short Yoda's\_C.0040D0A4  
0040D0A3  |  C2 2AC1           retn 0C12A        ; 依然是 C2 花指令  
  
0040D0A4  |  2AC1              sub al,cl  
0040D0A6  |  34 58             xor al,58  
0040D0A8  |  C0C0 B1           rol al,0B1  
0040D0AB  |  AA                stos byte ptr es:\[edi\]  
0040D0AC  \\^ E2 CC             loopd short Yoda's\_C.0040D07A  
  
因为有代码自解密，所以会给静态调试带来干扰，但分析上面所有解密方法，可以看出都是对 al 进行运算，另两个参数就是 cl（剩余字节数的）和 edi（代码地址），如果用 IDA 来调试的话，也很容易能写出脚本函数来解密这段代码。解密完成后，我们在数据窗口看到被解密的末尾赫然出现了“IsDebuggerPresent”，这里埋下伏笔。  
  
以下就是解密后的代码，位置是紧接上面的代码之后，解密完成后就开始执行之。  
  
0040D0AE     8B4424 20                mov eax,dword ptr ss:\[esp+20\]        ; kernel32.7C817077  
0040D0B2     40                       inc eax                              ; 程序出口点--kernel32.ExitThread  
0040D0B3     78 0A                    js short Yoda's\_C.0040D0BF  
0040D0B5     C785 78254000 01000000   mov dword ptr ss:\[ebp+402578\],1      ; 设置标志，后面会用到。  
0040D0BF     8D85 ED1D4000            lea eax,dword ptr ss:\[ebp+401DED\]    ; 地址=0040D060，程序入口点  
0040D0C5     B9 2A060000              mov ecx,62A                          ; 自校验的代码字节数  
0040D0CA     E8 41020000              call Yoda's\_C.0040D310               ; 壳代码自校验  
  {  
  0040D310     8BF8                     mov edi,eax                  ; 从入口点 0040D060 起 62Ah 字节纳入校验  
  0040D312     33C0                     xor eax,eax  
  0040D314     33DB                     xor ebx,ebx  
  0040D316     33D2                     xor edx,edx  
  0040D318     8A07                     mov al,byte ptr ds:\[edi\]  
  0040D31A     F7E2                     mul edx  
  0040D31C     03D8                     add ebx,eax  
  0040D31E     42                       inc edx  
  0040D31F     47                       inc edi  
  0040D320   ^ E2 F6                    loopd short Yoda's\_C.0040D318  
  }  
这个函数校验从壳入口起 62A 字节，实际上第 1 字节不影响结果，因这它乘 0 还是=0。  
  
0040D0CF     8985 74254000            mov dword ptr ss:\[ebp+402574\],eax    ; 保存校验和（0002494C），后面会用到。  
  
我们可以看到\[ebp+402574\]上下是一张信息表，其中存放了加壳前原程序的一些数据信息，如\[0040D7DB\]处就是存放的原程序 OEP－－004010CC，以及\[0040D7DB\]处的 00400000，加壳后的程序运行时用这些数据进行一系列处理。  
  
0040D0D5     8B85 6C254000            mov eax,dword ptr ss:\[ebp+40256C\]    ; ss:\[0040D7DF\]=0000003C  
0040D0DB     83E0 01                  and eax,1  
0040D0DE     74 40                    je short Yoda's\_C.0040D120           ; 偶数，跳  
  
0040D120     8B85 64254000            mov eax,dword ptr ss:\[ebp+402564\]    ; 取 ImageBaseAddress  
0040D126     0340 3C                  add eax,dword ptr ds:\[eax+3C\]        ; 定位 PE Header  
0040D129     05 80000000              add eax,80                           ; 取输入表数据目录  
0040D12E     8B08                     mov ecx,dword ptr ds:\[eax\]           ; 取输入表 RVA（加壳时已经被改指向壳中的新位置）  
0040D130     038D 64254000            add ecx,dword ptr ss:\[ebp+402564\]    ; +ImageBaseAddress 定位输入表（位于壳中，只有 1 个 IID）  
0040D136     83C1 10                  add ecx,10                           ; FirstThunk  
0040D139     8B01                     mov eax,dword ptr ds:\[ecx\]           ;   
0040D13B     0385 64254000            add eax,dword ptr ss:\[ebp+402564\]    ; 函数指针  
0040D141     8B18                     mov ebx,dword ptr ds:\[eax\]           ; 函数 kernel32.LoadLibraryA 地址  
0040D143     899D F0264000            mov dword ptr ss:\[ebp+4026F0\],ebx    ; 保存 LoadLibraryA 函数地址  
0040D149     83C0 04                  add eax,4                            ; 下一个函数  
0040D14C     8B18                     mov ebx,dword ptr ds:\[eax\]           ; kernel32.GetProcAddress  
0040D14E     899D F4264000            mov dword ptr ss:\[ebp+4026F4\],ebx    ; 保存 GetProcAddress 函数地址  
0040D154     8D85 F8264000            lea eax,dword ptr ss:\[ebp+4026F8\]    ; ASCII "Kernel32.dll"  
0040D15A     50                       push eax  
0040D15B     FF95 F0264000            call dword ptr ss:\[ebp+4026F0\]       ; 动态调入 Kernel32.dll  
0040D161     8BF0                     mov esi,eax                          ; Kernel32.dll 句柄  
0040D163     8985 05274000            mov dword ptr ss:\[ebp+402705\],eax    ; 保存句柄  
0040D169     8D85 09274000            lea eax,dword ptr ss:\[ebp+402709\]    ; ASCII "GetModuleHandleA"  
0040D16F     E8 96000000              call Yoda's\_C.0040D20A               ; 简单封装的 GetProcAddress 函数。获取 kernel32.GetModuleHandleA 地址  
  {  
  0040D20A     50                       push eax  
  0040D20B     56                       push esi  
  0040D20C     FF95 F4264000            call dword ptr ss:\[ebp+4026F4\] ; kernel32.GetProcAddress  
  }  
0040D174     8985 1A274000            mov dword ptr ss:\[ebp+40271A\],eax    ; 保存函数地址  
0040D17A     8D85 1E274000            lea eax,dword ptr ss:\[ebp+40271E\]    ; ASCII "VirtualProtect"  
0040D180     E8 85000000              call Yoda's\_C.0040D20A               ; 获取 kernel32.VirtualProtect 地址  
0040D185     8985 2D274000            mov dword ptr ss:\[ebp+40272D\],eax    ; 保存函数地址  
0040D18B     8D85 31274000            lea eax,dword ptr ss:\[ebp+402731\]  
0040D191     E8 74000000              call Yoda's\_C.0040D20A               ; 获取 kernel32.GetModuleFileNameA 地址  
0040D196     8985 44274000            mov dword ptr ss:\[ebp+402744\],eax    ; 保存函数地址  
0040D19C     8D85 48274000            lea eax,dword ptr ss:\[ebp+402748\]  
0040D1A2     E8 63000000              call Yoda's\_C.0040D20A               ; 获取 kernel32.CreateFileA 地址  
0040D1A7     8985 54274000            mov dword ptr ss:\[ebp+402754\],eax    ; 保存函数地址  
0040D1AD     8D85 58274000            lea eax,dword ptr ss:\[ebp+402758\]  
0040D1B3     E8 52000000              call Yoda's\_C.0040D20A               ; 获取 kernel32.GlobalAlloc 地址  
0040D1B8     8985 64274000            mov dword ptr ss:\[ebp+402764\],eax    ; 保存函数地址  
0040D1BE     8D85 68274000            lea eax,dword ptr ss:\[ebp+402768\]  
0040D1C4     E8 41000000              call Yoda's\_C.0040D20A               ; 获取 kernel32.GlobalFree 地址  
0040D1C9     8985 73274000            mov dword ptr ss:\[ebp+402773\],eax    ; 保存函数地址  
0040D1CF     8D85 77274000            lea eax,dword ptr ss:\[ebp+402777\]  
0040D1D5     E8 30000000              call Yoda's\_C.0040D20A               ; 获取 kernel32.ReadFile 地址  
0040D1DA     8985 80274000            mov dword ptr ss:\[ebp+402780\],eax    ; 保存函数地址  
0040D1E0     8D85 84274000            lea eax,dword ptr ss:\[ebp+402784\]  
0040D1E6     E8 1F000000              call Yoda's\_C.0040D20A               ; 获取 kernel32.GetFileSize 地址  
0040D1EB     8985 90274000            mov dword ptr ss:\[ebp+402790\],eax    ; 保存函数地址  
0040D1F1     8D85 94274000            lea eax,dword ptr ss:\[ebp+402794\]  
0040D1F7     E8 0E000000              call Yoda's\_C.0040D20A               ; 获取 kernel32.CloseHandle 地址  
0040D1FC     8985 A0274000            mov dword ptr ss:\[ebp+4027A0\],eax    ; 保存函数地址  
0040D202     8D85 A01F4000            lea eax,dword ptr ss:\[ebp+401FA0\]    ; 取上面执行过的子函数之后的代码的地址（0040D213）。  
0040D208     50                       push eax  
0040D209     C3                       retn                                 ; 用 push+retn 的形式跳转  
\-------------------------------------------------  
0040D20A     50                       push eax  
0040D20B     56                       push esi  
0040D20C     FF95 F4264000            call dword ptr ss:\[ebp+4026F4\]     ; kernel32.GetProcAddress  
\-------------------------------------------------  
上面 3 行就是上面执行过的子函数，这种将子函数嵌入主程序代码中并用 push+retn 的形式跳转的做法，下面还有几处，我不再把子函数再一一列出了，看到用 push+retn 的形式跳转的形式就明白下面是还有一段已调用过的子函数的。  
  
用 push+retn 的形式跳转到了这里：  
0040D213     F785 6C254000 10000000   test dword ptr ss:\[ebp+40256C\],10     ; 3C  
0040D21D     74 37                    je short Yoda's\_C.0040D256            ; 不跳  
0040D21F     64:FF35 30000000         push dword ptr fs:\[30\]                ; push+pop 获得 PEB 首地址  
0040D226     58                       pop eax  
0040D227     85C0                     test eax,eax  
0040D229     78 0F                    js short Yoda's\_C.0040D23A            ; 如果是 windows9x 就跳，NT 则不跳  
0040D22B     8B40 0C                  mov eax,dword ptr ds:\[eax+C\]          ; 进程 PEB 的双向链表（PEB\_LDR\_DATA）  
0040D22E     8B40 0C                  mov eax,dword ptr ds:\[eax+C\]          ; InLoadOrderModuleList  
0040D231     C740 20 00100000         mov dword ptr ds:\[eax+20\],1000        ; 修改 InLoadOrderLinks.SizeOfImage 大小，原为 0F000，做点小手脚反脱壳  
0040D238     EB 1C                    jmp short Yoda's\_C.0040D256           ; 跳  
0040D23A     6A 00                    push 0  
0040D23C     FF95 1A274000            call dword ptr ss:\[ebp+40271A\]  
0040D242     85D2                     test edx,edx  
0040D244     79 10                    jns short Yoda's\_C.0040D256  
0040D246     837A 08 FF               cmp dword ptr ds:\[edx+8\],-1  
0040D24A     75 0A                    jnz short Yoda's\_C.0040D256  
0040D24C     8B52 04                  mov edx,dword ptr ds:\[edx+4\]  
0040D24F     C742 50 00100000         mov dword ptr ds:\[edx+50\],1000  
  
0040D256     8BBD 64254000            mov edi,dword ptr ss:\[ebp+402564\]     ; 跳到这里。ImageBaseAddress  
0040D25C     037F 3C                  add edi,dword ptr ds:\[edi+3C\]         ; 定位 PE 头  
0040D25F     8BB5 64254000            mov esi,dword ptr ss:\[ebp+402564\]     ; ImageBaseAddress  
0040D265     8B4F 54                  mov ecx,dword ptr ds:\[edi+54\]         ; SizeOfHeaders  
0040D268     8D85 D2274000            lea eax,dword ptr ss:\[ebp+4027D2\]     ; 地址=0040DA45，文件末尾的空白处。这个壳把文件末尾的空白区域作为缓冲区（暂且称为 buffer 吧），保存一些临时数据。  
0040D26E     50                       push eax                              ; lpflOldProtect  
0040D26F     6A 04                    push 4  
0040D271     51                       push ecx                              ; size=1000  
0040D272     FFB5 64254000            push dword ptr ss:\[ebp+402564\]        ; lpAddress=00400000  
0040D278     FF95 2D274000            call dword ptr ss:\[ebp+40272D\]        ; kernel32.VirtualProtect。文件头内存页的只读属性改为可读写  
0040D27E     F785 6C254000 08000000   test dword ptr ss:\[ebp+40256C\],8      ; 3C  
0040D288     0F84 A7000000            je Yoda's\_C.0040D335  
0040D28E     68 04010000              push 104  
0040D293     8DBD D2274000            lea edi,dword ptr ss:\[ebp+4027D2\]     ; buffer  
0040D299     57                       push edi  
0040D29A     6A 00                    push 0  
0040D29C     FF95 44274000            call dword ptr ss:\[ebp+402744\]        ; GetModuleFileNameA。获取文件名  
0040D2A2     6A 00                    push 0                                ; hTemplateFile  
0040D2A4     68 80000000              push 80                               ; dwFlagsAndAttributes  
0040D2A9     6A 03                    push 3                                ; dwCreationDisposition  
0040D2AB     6A 00                    push 0                                ; lpSecurityAttributes  
0040D2AD     6A 01                    push 1                                ; dwShareMode  
0040D2AF     68 00000080              push 80000000                         ; dwDesiredAccess  
0040D2B4     57                       push edi                              ; lpFileName  
0040D2B5     FF95 54274000            call dword ptr ss:\[ebp+402754\]        ; kernel32.CreateFileA，打开磁盘文件并获取文件句柄。  
0040D2BB     83F8 FF                  cmp eax,-1  
0040D2BE     75 04                    jnz short Yoda's\_C.0040D2C4           ; 成功则跳  
0040D2C0     33C0                     xor eax,eax  
0040D2C2     EB 71                    jmp short Yoda's\_C.0040D335  
  
0040D2C4     8BF8                     mov edi,eax                           ; 打开的文件句柄  
0040D2C6     6A 00                    push 0  
0040D2C8     57                       push edi  
0040D2C9     FF95 90274000            call dword ptr ss:\[ebp+402790\]        ; GetFileSize 获取文件大小  
0040D2CF     83E8 05                  sub eax,5                             ; 文件大小 DA46-5=DA41 字节。后面我们可以知道，DA41 之后的 5 个字节是 DA41 字节的校验和（4 字节）及 0  
0040D2D2     96                       xchg eax,esi  
0040D2D3     56                       push esi  
0040D2D4     6A 40                    push 40  
0040D2D6     FF95 64274000            call dword ptr ss:\[ebp+402764\]        ; GlobalAlloc 申请一个比原文件小 5 字节的内存空间  
0040D2DC     0BC0                     or eax,eax  
0040D2DE     75 02                    jnz short Yoda's\_C.0040D2E2           ; 成功则跳  
0040D2E0     EB 4A                    jmp short Yoda's\_C.0040D32C  
0040D2E2     93                       xchg eax,ebx  
0040D2E3     6A 00                    push 0                                ; lpOverlapped  
0040D2E5     8D85 D2274000            lea eax,dword ptr ss:\[ebp+4027D2\]     ; buffer，现用来存放从文件读取的字节数  
0040D2EB     50                       push eax                              ; lpNumberOfBytesRead  
0040D2EC     56                       push esi                              ; nNumberOfBytesToRead  
0040D2ED     53                       push ebx                              ; lpBuffer  
0040D2EE     57                       push edi                              ; hFile  
0040D2EF     FF95 80274000            call dword ptr ss:\[ebp+402780\]        ; ReadFile 读取文件  
0040D2F5     8BC3                     mov eax,ebx                           ; lpBuffer  
0040D2F7     8BCE                     mov ecx,esi                           ; nNumberOfBytesToRead  
0040D2F9     53                       push ebx  
0040D2FA     57                       push edi  
0040D2FB     E8 10000000              call Yoda's\_C.0040D310                ; 读入的文件内容校验和，同前面壳代码自校验是同一函数。  
  {  
  0040D310     8BF8                     mov edi,eax                   ; lpBuffer（共 DA41 字节）  
  0040D312     33C0                     xor eax,eax  
  0040D314     33DB                     xor ebx,ebx  
  0040D316     33D2                     xor edx,edx  
  0040D318     8A07                     mov al,byte ptr ds:\[edi\]  
  0040D31A     F7E2                     mul edx                       ; 同样是第 1 个字节不影响结果  
  0040D31C     03D8                     add ebx,eax  
  0040D31E     42                       inc edx  
  0040D31F     47                       inc edi  
  0040D320   ^ E2 F6                    loopd short Yoda's\_C.0040D318  
  0040D322     93                       xchg eax,ebx  
  0040D323     C3                       retn  
  }  
0040D300     8985 70254000            mov dword ptr ss:\[ebp+402570\],eax     ; 保存文件校验和（00504875）  
0040D306     5F                       pop edi  
0040D307     5B                       pop ebx  
0040D308     8D85 B1204000            lea eax,dword ptr ss:\[ebp+4020B1\]     ; 修改地址跳过上面已执行的子函数代码  
0040D30E     50                       push eax  
0040D30F     C3                       retn                                  ; 用 push+retn 的形式跳转  
  
跳到了这里:  
0040D324     53                       push ebx    
0040D325     FF95 73274000            call dword ptr ss:\[ebp+402773\]        ; 释放内存  
0040D32B     96                       xchg eax,esi  
0040D32C     50                       push eax  
0040D32D     57                       push edi  
0040D32E     FF95 A0274000            call dword ptr ss:\[ebp+4027A0\]        ; 关闭打开的文件  
0040D334     58                       pop eax  
0040D335     8B85 64254000            mov eax,dword ptr ss:\[ebp+402564\]     ; ImageBaseAddress  
0040D33B     BB 01000000              mov ebx,1                             ; 区段计数器  
0040D340     E8 08000000              call Yoda's\_C.0040D34D                ; 解密区段数据  
  {  
  0040D34D     8BF8                     mov edi,eax  
  0040D34F     037F 3C                  add edi,dword ptr ds:\[edi+3C\]         ; 定位 PE 头  
  0040D352     8BF7                     mov esi,edi  
  0040D354     81C6 F8000000            add esi,0F8                           ; 定位区段表  
  0040D35A     33D2                     xor edx,edx  
  0040D35C     813E 72737263            cmp dword ptr ds:\[esi\],63727372       ; 区段名前 4 个字符是否'rsrc'，是则跳过该区段，否则继续判断  
  0040D362     75 05                    jnz short Yoda's\_C.0040D369  
  0040D364     E9 9E000000              jmp Yoda's\_C.0040D407  
  0040D369     813E 2E727372            cmp dword ptr ds:\[esi\],7273722E       ; 是否'.rsr'，是则跳过该区段，否则继续判断  
  0040D36F     75 05                    jnz short Yoda's\_C.0040D376  
  0040D371     E9 91000000              jmp Yoda's\_C.0040D407  
  0040D376     813E 72656C6F            cmp dword ptr ds:\[esi\],6F6C6572       ; 是否'relo'，是则跳过该区段，否则继续判断  
  0040D37C     75 05                    jnz short Yoda's\_C.0040D383  
  0040D37E     E9 84000000              jmp Yoda's\_C.0040D407  
  0040D383     813E 2E72656C            cmp dword ptr ds:\[esi\],6C65722E       ; 是否'.rel'，是则跳过该区段，否则继续判断  
  0040D389     75 02                    jnz short Yoda's\_C.0040D38D  
  0040D38B     EB 7A                    jmp short Yoda's\_C.0040D407  
  0040D38D     813E 79430000            cmp dword ptr ds:\[esi\],4379           ; 是否'yC'，即 yada 加壳的段名标志，是则跳过该区段，否则继续判断  
  0040D393     75 02                    jnz short Yoda's\_C.0040D397  
  0040D395     EB 70                    jmp short Yoda's\_C.0040D407  
  0040D397     813E 2E656461            cmp dword ptr ds:\[esi\],6164652E       ; 是否'.eda'，是则跳过该区段，否则继续判断  
  0040D39D     75 02                    jnz short Yoda's\_C.0040D3A1  
  0040D39F     EB 66                    jmp short Yoda's\_C.0040D407  
  0040D3A1     837E 14 00               cmp dword ptr ds:\[esi+14\],0           ; 该区段在文件中的偏移量否为 0，是则跳过该区段，否则继续判断  
  0040D3A5     74 06                    je short Yoda's\_C.0040D3AD  
  0040D3A7     837E 10 00               cmp dword ptr ds:\[esi+10\],0           ; 该区段大小是否为 0，是则跳过该区段，否则解密该段  
  0040D3AB     75 02                    jnz short Yoda's\_C.0040D3AF  
  0040D3AD     EB 58                    jmp short Yoda's\_C.0040D407  
  
  0040D3AF     60                       pushad                                ; 保护环境，开始解密该段  
  0040D3B0     8B4E 10                  mov ecx,dword ptr ds:\[esi+10\]         ; 区段大小  
  0040D3B3     0BDB                     or ebx,ebx  
  0040D3B5    /75 0C                    jnz short Yoda's\_C.0040D3C3           ; 计数器不为 0 则跳  
  0040D3B7    |8B76 14                  mov esi,dword ptr ds:\[esi+14\]  
  0040D3BA    |03F0                     add esi,eax  
  0040D3BC    |E8 8BF9FFFF              call Yoda's\_C.0040CD4C  
  0040D3C1    |EB 0A                    jmp short Yoda's\_C.0040D3CD  
  0040D3C3    \\8B76 0C                  mov esi,dword ptr ds:\[esi+C\]          ; 取区段的 RVA 地址  
  0040D3C6     03F0                     add esi,eax                           ; 区段的地址  
  0040D3C8     E8 02000000              call Yoda's\_C.0040D3CF                ; 解密  
    {  
    0040D3CF     8BFE                     mov edi,esi                   ; 段起始地址  
    0040D3D1     AC                       lods byte ptr ds:\[esi\]        ; 取一个字节进行解密  
    0040D3D2     90                       nop  
    0040D3D3     EB 01                    jmp short Yoda's\_C.0040D3D6  
    0040D3D5     C2 348F                  retn 8F34                     ; 又是可恶的 C2，后面的这种代码我就不列出来了  
  
    0040D3D6     34 8F                    xor al,8F  
    0040D3D8     F9                       stc  
    0040D3D9     02C1                     add al,cl  
    0040D3DB     F8                       clc  
    0040D3DC     34 AE                    xor al,0AE  
    0040D3DE     2C E9                    sub al,0E9  
    0040D3E0     FEC8                     dec al  
    0040D3E2     02C1                     add al,cl  
    0040D3E4     FEC8                     dec al  
    0040D3E6     04 CD                    add al,0CD  
    0040D3E8     90                       nop  
    0040D3E9     EB 01                    jmp short Yoda's\_C.0040D3EC  
  
    0040D3EC     2AC1                     sub al,cl  
    0040D3EE     C0C8 5E                  ror al,5E  
    0040D3F1     FEC8                     dec al  
    0040D3F3     2AC1                     sub al,cl  
    0040D3F5     04 7D                    add al,7D  
    0040D3F7     EB 01                    jmp short Yoda's\_C.0040D3FA  
  
    0040D3FA     2AC1                     sub al,cl  
    0040D3FC     34 58                    xor al,58  
    0040D3FE     C0C0 B1                  rol al,0B1  
    0040D401     90                       nop  
    0040D402     AA                       stos byte ptr es:\[edi\]       ; 回写解密后的字节  
    0040D403   ^ E2 CC                    loopd short Yoda's\_C.0040D3D1  
    0040D405     C3                       retn  
    }  
    我们可以看到，这个解密方法与壳引导代码中解密壳代码的方法是完全相同的，只不过这里封装成了函数。  
  
  0040D3CD     EB 37                    jmp short Yoda's\_C.0040D406  
  
  0040D406     61                       popad                         ; 恢复环境  
  0040D407     83C6 28                  add esi,28                    ; 处理下一区段  
  0040D40A     42                       inc edx                       ; 区段计数器+1  
  0040D40B     66:3B57 06               cmp dx,word ptr ds:\[edi+6\]    ; 区段是否处理完毕  
  0040D40F   ^ 0F85 47FFFFFF            jnz Yoda's\_C.0040D35C         ; 否则循环处理  
  0040D415     C3                       retn  
  }  
0040D345     8D85 A3214000            lea eax,dword ptr ss:\[ebp+4021A3\]     ; 取上面执行过的子函数之后的代码的地址（0040D416）。  
0040D34B     50                       push eax                              ; 压入地址  
0040D34C     C3                       retn                                  ; 用 retn 的形式跳转  
  
跳到了这里：  
0040D416     8B9D 64254000            mov ebx,dword ptr ss:\[ebp+402564\]     ; 信息表中取 ImageBaseAddress  
0040D41C     039D 68254000            add ebx,dword ptr ss:\[ebp+402568\]     ; 00400000+000010CC=004010CC（OEP，这里先假装不知，因为碰到不熟悉的带壳程序时，就只能猜它可能是，它正好落在程序的代码段，到后面根据情况再确定是不是）  
0040D422     C1CB 07                  ror ebx,7                             ; 这里要做什么！注意，004010CC 被加密成了 98008021，更值得怀疑是 OEP！先留下一个悬念  
0040D425     895C24 10                mov dword ptr ss:\[esp+10\],ebx         ; 保存在堆栈 0012FFB4 中，这个地址的下一个地址的内容是 ntdll.KiFastSystemCallRet，这些特殊地址都要收起注意  
0040D429     8D9D 99244000            lea ebx,dword ptr ss:\[ebp+402499\]     ; 信息表中取什么地址？0040D70C，再留下一个悬念  
0040D42F     895C24 1C                mov dword ptr ss:\[esp+1C\],ebx         ; 保存在 0012FFC0，这个地址的下一个内容是“返回到 kernel32.7C817077”  
0040D433     8BBD 64254000            mov edi,dword ptr ss:\[ebp+402564\]     ; 信息表中取 ImageBaseAddress  
0040D439     037F 3C                  add edi,dword ptr ds:\[edi+3C\]         ; 定位 PE 头  
0040D43C     8B9F C0000000            mov ebx,dword ptr ds:\[edi+C0\]         ; 取线程本地存储器（TLS）这又是要做什么？这几行代码令人生疑！  
0040D442     83FB 00                  cmp ebx,0                             ; 本例为 0。如果非 0，则指向一个 IMAGE\_TLS\_DIRECTORY 结构，TLS 是程序最先开始执行的地方，比 EP 更早执行  
0040D445     74 0F                    je short Yoda's\_C.0040D456  
0040D447     039D 64254000            add ebx,dword ptr ss:\[ebp+402564\]  
0040D44D     8B43 08                  mov eax,dword ptr ds:\[ebx+8\]  
0040D450     C700 00000000            mov dword ptr ds:\[eax\],0              ; 将 TLS 索引清 0。本例中未执行到这里，因为本就是 0  
0040D456     8B85 70254000            mov eax,dword ptr ss:\[ebp+402570\]     ; 取出之前保存的文件校验和（文件大小-5 字节的，00504875），晕 S，隔这么远的地方来检验！  
0040D45C     0BC0                     or eax,eax  
0040D45E     74 0D                    je short Yoda's\_C.0040D46D  
0040D460     3B85 CE274000            cmp eax,dword ptr ss:\[ebp+4027CE\]     ; ss:\[0040DA41\]=00504875，这是加壳时保存的文件校验和，在此验证文件是否被改动过  
0040D466     74 05                    je short Yoda's\_C.0040D46D  
0040D468     E9 AF010000              jmp Yoda's\_C.0040D61C                 ; 如果改动了就 game over 了  
  
下面开始处理输入表了：  
0040D46D     8DB5 7C254000            lea esi,dword ptr ss:\[ebp+40257C\]     ; 地址=0040D7EF，这又是什么地址？从后面的处理可以看出，这个 esi 处是一个变形了的输入表结构，每个结构对应一个 dll，并且每个结构压缩成 3 个成员：第一个是成员 Name，第二个 FirstThunk，第三个是 OriginalFirstThunk。加壳时，对 Name 指向的 dll 名称都加了密，FirstThunk 指向的函数地址都被篡改。  
0040D473     F785 6C254000 20000000   test dword ptr ss:\[ebp+40256C\],20     ; 3C  
0040D47D     74 49                    je short Yoda's\_C.0040D4C8  
0040D47F     56                       push esi  
0040D480     8DBD D2274000            lea edi,dword ptr ss:\[ebp+4027D2\]     ; buffer  
0040D486     33C9                     xor ecx,ecx  
0040D488    /EB 17                    jmp short Yoda's\_C.0040D4A1           ; 这个循环象是在计算 API 个数  
0040D48A    |8B56 04                  mov edx,dword ptr ds:\[esi+4\]          ; ds:\[0040D7F3\]=000063F0，这是什么地址？IAT？  
0040D48D    |0395 64254000            add edx,dword ptr ss:\[ebp+402564\]     ; 偏移地址+ImageBaseAddress  
0040D493    |EB 04                    jmp short Yoda's\_C.0040D499  
0040D495    |41                       inc ecx                               ; 计数器+1  
0040D496    |83C2 04                  add edx,4  
0040D499    |833A 00                  cmp dword ptr ds:\[edx\],0  
0040D49C   ^|75 F7                    jnz short Yoda's\_C.0040D495           ; 象是在遍历 IAT  
0040D49E    |83C6 0C                  add esi,0C  
0040D4A1    \\837E 04 00               cmp dword ptr ds:\[esi+4\],0  
0040D4A5   ^ 75 E3                    jnz short Yoda's\_C.0040D48A           ; 下一个 DLL？  
0040D4A7     33D2                     xor edx,edx  
0040D4A9     B8 05000000              mov eax,5  
0040D4AE     F7E1                     mul ecx  
0040D4B0     50                       push eax                              ; 8Ah\*5=2B2h，为何要\*5？后面揭秘  
0040D4B1     6A 00                    push 0  
0040D4B3     FF95 64274000            call dword ptr ss:\[ebp+402764\]        ; GlobalAlloc 申请内存（堆），大小为 2B2h 字节  
0040D4B9     0BC0                     or eax,eax                            ; 我这里申请到的堆地址=001431E0  
0040D4BB     75 05                    jnz short Yoda's\_C.0040D4C2           ; 成功则跳，失败则退出  
0040D4BD     83C4 04                  add esp,4  
0040D4C0     61                       popad  
0040D4C1     C3                       retn  
0040D4C2     8907                     mov dword ptr ds:\[edi\],eax            ; \[edi\]=\[0040DA45\]，即保存堆地址到 buffer  
0040D4C4     8947 04                  mov dword ptr ds:\[edi+4\],eax          ; 连续保存 2 份  
0040D4C7     5E                       pop esi                               ; 0040D7EF，弹出变形的输入表首地址  
0040D4C8     E9 42010000              jmp Yoda's\_C.0040D60F                 ; 跳到循环条件判断处，之间为循环体  
0040D4CD  /  8B1E                     mov ebx,dword ptr ds:\[esi\]            ; 刚刚遍历的地址，通过一轮循环后知道这是 IAT 的 RVA  
0040D4CF  |  039D 64254000            add ebx,dword ptr ss:\[ebp+402564\]     ; +ImageBaseAddress，定位到 IAT  
0040D4D5  |  8BC3                     mov eax,ebx  
0040D4D7  |  E8 08000000              call Yoda's\_C.0040D4E4                ; 字符串解密函数。这里是将 dll 文件名解密  
  {  
  0040D4E4     56                       push esi  
  0040D4E5     57                       push edi  
  0040D4E6     8BF0                     mov esi,eax  
  0040D4E8     8BF8                     mov edi,eax                   ; 将源地址与目标地址指向同一地址，即直接解密后覆盖  
  0040D4EA     AC                       lods byte ptr ds:\[esi\]        ; ds:\[esi\]=\[0040658A\]，注意，这个地址是在.idata 段，不是在壳中  
  0040D4EB     C0C8 04                  ror al,4                      ; 简单地将每字节循环右移 4 位解密  
  0040D4EE     AA                       stos byte ptr es:\[edi\]        ; 果然与解密输入表有关，首先解密出了"SHELL32.DLL"字符串（IID 的 NAME 成员），并回写  
  0040D4EF     803F 00                  cmp byte ptr ds:\[edi\],0  
  0040D4F2   ^ 75 F6                    jnz short Yoda's\_C.0040D4EA  
  0040D4F4     5F                       pop edi  
  0040D4F5     5E                       pop esi  
  0040D4F6     C3                       retn  
  }  
0040D4DC  |  8D85 84224000            lea eax,dword ptr ss:\[ebp+402284\]     ; 取上面执行过的子函数之后的代码的地址（0040D4F7）。  
0040D4E2  |  50                       push eax                              ; 压入地址  
0040D4E3  |  C3                       retn                                  ; 用 push+retn 的形式跳转  
  
跳到了这里:  
0040D4F7  |  53                       push ebx                              ; ASCII "SHELL32.dll"  
0040D4F8  |  FF95 F0264000            call dword ptr ss:\[ebp+4026F0\]        ; LoadLibraryA 调入动态库 SHELL32.dll  
0040D4FE  |  85C0                     test eax,eax  
0040D500  |  0F84 16010000            je Yoda's\_C.0040D61C                  ; 调入失败则 game over  
0040D506  |  50                       push eax  
0040D507  |  F785 6C254000 04000000   test dword ptr ss:\[ebp+40256C\],4      ; 3C，若=4 则不清除 shell32.dll 字符串，否则清除  
0040D511  |  74 0E                    je short Yoda's\_C.0040D521  
0040D513  |  8D85 AE224000            lea eax,dword ptr ss:\[ebp+4022AE\]     ; 地址=0040D521  
0040D519  |  50                       push eax                              ; 这里用 push+jmp 的形式调用子函数  
0040D51A  |  8BC3                     mov eax,ebx                           ; ebx=0040658A, ASCII "SHELL32.dll"  
0040D51C  |  E9 48020000              jmp Yoda's\_C.0040D769                 ; 跳入子函数。功能是将 eax 处的字符串（这里是 dll 文件名）擦除  
  {  
  0040D769    /EB 04                    jmp short Yoda's\_C.0040D76F  
  0040D76B    |C600 00                  mov byte ptr ds:\[eax\],0  
  0040D76E    |40                       inc eax  
  0040D76F    \\8038 00                  cmp byte ptr ds:\[eax\],0       ; 把刚刚解密出来的 shell32.dll 字符串擦除了  
  0040D772   ^ 75 F7                    jnz short Yoda's\_C.0040D76B  
  0040D774     C3                       retn  
  }  
0040D521  |  5B                       pop ebx                               ; 弹出 dll 句柄  
0040D522  |  8B4E 08                  mov ecx,dword ptr ds:\[esi+8\]          ; 从后面可以看出，这是变形输入表的 OriginalFirstThunk  
0040D525  |  0BC9                     or ecx,ecx  
0040D527  |  75 03                    jnz short Yoda's\_C.0040D52C           ; 不为 0 则进行处理，为 0 则找 FirstThunk。本例非 0  
0040D529  |  8B4E 04                  mov ecx,dword ptr ds:\[esi+4\]          ; 从后面可以看出，这是变形输入表的 FirstThunk  
0040D52C  |  038D 64254000            add ecx,dword ptr ss:\[ebp+402564\]     ; RVA+ImageBaseAddress 定位 OriginalFirstThunk  
0040D532  |  8B56 04                  mov edx,dword ptr ds:\[esi+4\]          ; FirstThunk  
0040D535     0395 64254000            add edx,dword ptr ss:\[ebp+402564\]     ; RVA+ImageBaseAddress 定位  
0040D53B  | /E9 C3000000              jmp Yoda's\_C.0040D603                 ; 这里是一个大循环  
0040D540  | |F701 00000080            test dword ptr ds:\[ecx\],80000000      ; 从这个特征值几乎可以确定\[ecx\]就是一个 IMAGE\_THUNK\_DATA 了，判断是 by\_NAME 还是 by\_INDEX  
0040D546  | |75 4B                    jnz short Yoda's\_C.0040D593           ; 若 IMAGE\_ORDINAL\_FLAG32 则转向 by\_Index 方式处理  
0040D548  | |8B01                     mov eax,dword ptr ds:\[ecx\]            ; by\_Name 的处理  
0040D54A  | |83C0 02                  add eax,2                             ; 开始看到这里还以为是搞什么名堂，等循环了一遍才知道，eax+2 是跳过 IMAGE\_IMPORT\_BY\_NAME 结构的成员 Hint，定位到 Name 成员，以解密出函数名。而这个 eax 来自\[ecx\] ，ecx 来自\[esi+8\]，因此，\[esi+8\]就是 OriginalFirstThunk 了！  
0040D54D  | |0385 64254000            add eax,dword ptr ss:\[ebp+402564\]     ; RVA+ImageBaseAddress 定位待解密函数名称字符串的地址  
0040D553  | |50                       push eax                              ; 保存函数名地址  
0040D554  | |E8 8BFFFFFF              call Yoda's\_C.0040D4E4                ; 就是前面用过的那个字符串解密函数，在这里是解密函数名  
  {  
  0040D4E4     56                       push esi  
  0040D4E5     57                       push edi  
  0040D4E6     8BF0                     mov esi,eax  
  0040D4E8     8BF8                     mov edi,eax  
  0040D4EA    /AC                       lods byte ptr ds:\[esi\]  
  0040D4EB    |C0C8 04                  ror al,4                      ; 简单地将每字节循环右移 4 位  
  0040D4EE    |AA                       stos byte ptr es:\[edi\]        ; 这里将解密后的字符回写  
  0040D4EF    |803F 00                  cmp byte ptr ds:\[edi\],0  
  0040D4F2   ^\\75 F6                    jnz short Yoda's\_C.0040D4EA   ; 没到字符串末尾就继续解密  
  0040D4F4     5F                       pop edi  
  0040D4F5     5E                       pop esi  
  0040D4F6     C3                       retn  
  }  
0040D559  | |58                       pop eax                               ; 此时弹出的是解密后的函数名  
0040D55A  | |8BF8                     mov edi,eax                           ; 地址保存  
0040D55C  | |52                       push edx                              ; IAT 待填充的地址  
0040D55D  | |51                       push ecx                              ; FirstThunk  
0040D55E  | |50                       push eax                              ; 参数 2：lpProcName   
0040D55F  | |53                       push ebx                              ; 参数 1：hModule  
0040D560  | |FF95 F4264000            call dword ptr ss:\[ebp+4026F4\]        ; kernel32.GetProcAddress  
0040D566  | |0BC0                     or eax,eax   
0040D568  | |75 07                    jnz short Yoda's\_C.0040D571           ; 成功则跳，否则 game over  
0040D56A  | |59                       pop ecx  
0040D56B  | |5A                       pop edx  
0040D56C  | |E9 AB000000              jmp Yoda's\_C.0040D61C  
0040D571  | |59                       pop ecx                               ; FirstThunk  
0040D572  | |5A                       pop edx                               ; IAT 待填充的地址  
0040D573  | |60                       pushad  
0040D574  | |F785 6C254000 04000000   test dword ptr ss:\[ebp+40256C\],4      ; 3C  
0040D57E  | |74 0E                    je short Yoda's\_C.0040D58E  
0040D580  | |8D85 1B234000            lea eax,dword ptr ss:\[ebp+40231B\]     ; 取下面调用子函数后返回地址：0040D58E  
0040D586  | |50                       push eax                              ; 用 push+jmp 的形式调用子函数，这里压入子程序的返回地址  
0040D587  | |8BC7                     mov eax,edi                           ; edi=函数名地址  
0040D589  | |E9 DB010000              jmp Yoda's\_C.0040D769                 ; 跳入子函数。功能是将 eax 处的字符串（这里是函数名）擦除，与上面擦除 dll 名字是同一函数  
  {  
  0040D769    /EB 04                    jmp short Yoda's\_C.0040D76F  
  0040D76B    |C600 00                  mov byte ptr ds:\[eax\],0  
  0040D76E    |40                       inc eax  
  0040D76F    \\8038 00                  cmp byte ptr ds:\[eax\],0       ; 把刚刚解密出来的函数名称擦除了，即把 IMAGE\_IMPORT\_BY\_NAME 的 Name 成员清 0，只保留了 Hint 成员  
  0040D772   ^ 75 F7                    jnz short Yoda's\_C.0040D76B  
  0040D774     C3                       retn  
  }  
0040D58E  | |61                       popad                                 ; 调用子函数后返回到这里  
0040D58F  | |8902                     mov dword ptr ds:\[edx\],eax            ; 填充 IAT(by\_Name 的处理)  
0040D591  | |EB 19                    jmp short Yoda's\_C.0040D5AC           ; 跳  
0040D593  | |52                       push edx                              ; by\_Index 的处理（本例中未执行到）  
0040D594  | |51                       push ecx  
0040D595  | |8B01                     mov eax,dword ptr ds:\[ecx\]  
0040D597  | |2D 00000080              sub eax,80000000  
0040D59C  | |50                       push eax  
0040D59D  | |53                       push ebx  
0040D59E  | |FF95 F4264000            call dword ptr ss:\[ebp+4026F4\]        ; kernel32.GetProcAddress  
0040D5A4  | |85C0                     test eax,eax  
0040D5A6  | |74 74                    je short Yoda's\_C.0040D61C  
0040D5A8  | |59                       pop ecx  
0040D5A9  | |5A                       pop edx  
0040D5AA  | |8902                     mov dword ptr ds:\[edx\],eax            ; 填充 IAT  
  
跳到了这里：  
0040D5AC  | |F785 6C254000 20000000   test dword ptr ss:\[ebp+40256C\],20     ; 3C　两种方式分别填充 IAT 后都转到这里  
0040D5B6  | |74 45                    je short Yoda's\_C.0040D5FD            ; =20 则跳到下一个  
0040D5B8  | |83BD 78254000 00         cmp dword ptr ss:\[ebp+402578\],0       ; 1  
0040D5BF  | |74 14                    je short Yoda's\_C.0040D5D5  
0040D5C1  | |81FB 00000070            cmp ebx,70000000                      ; dll 句柄(ebx=7D590000 (shell32.7D590000))  
0040D5C7  | |72 08                    jb short Yoda's\_C.0040D5D1  
0040D5C9  | |81FB FFFFFF77            cmp ebx,77FFFFFF                      ; user32.wsprintfA 的句柄=77D10000  
0040D5CF /| |76 0E                    jbe short Yoda's\_C.0040D5DF           ; user32.dll 的函数在此跳  
0040D5D1 ||/|EB 2A                    jmp short Yoda's\_C.0040D5FD           ; 跳  
0040D5D3 ||||EB 0A                    jmp short Yoda's\_C.0040D5DF  
0040D5D5 ||||81FB 00000080            cmp ebx,80000000  
0040D5DB ||||73 02                    jnb short Yoda's\_C.0040D5DF  
0040D5DD ||||EB 1E                    jmp short Yoda's\_C.0040D5FD  
0040D5DF \\|||57                       push edi                              ; 如果我们在处理完第 1 个 dll 后在 0040D619 处 F4 自动处理完剩余 dll 的函数，则看不到这里的关键处理，所以要有耐心，把隐藏在中间的步骤看清楚。  
0040D5E0  |||56                       push esi  
0040D5E1  |||8DBD D2274000            lea edi,dword ptr ss:\[ebp+4027D2\]     ; 地址=0040DA45，buffer，此时保存的是堆地址  
0040D5E7  |||8B77 04                  mov esi,dword ptr ds:\[edi+4\]          ; buffer+4 处是前面保存的第 2 份堆地址 001431E0  
0040D5EA  |||8932                     mov dword ptr ds:\[edx\],esi            ; 将刚刚填充的函数地址又改为申请的堆地址  
0040D5EC  |||2BC6                     sub eax,esi                           ; 从下面几行得知此 2 句 sub 是计算跳转的相对距离  
0040D5EE  |||83E8 05                  sub eax,5                             ; -5 是因为 jmp 指令 E9XXXXXXXX 本身占 5 字节  
0040D5F1  |||C606 E9                  mov byte ptr ds:\[esi\],0E9             ; 前面申请的堆在这里派上用场。0E9 是 jmp 指令  
0040D5F4  |||8946 01                  mov dword ptr ds:\[esi+1\],eax          ; 填上 jmp 的目标地址（相对距离）。对 IAT 做手脚，这样，调用函数就成了对堆中某地址的调用，而堆中的该地址是 5 个字节的 jmp 指令到正确的函数地址。  
0040D5F7  |||8347 04 05               add dword ptr ds:\[edi+4\],5            ; buffer+4 处的内容改为指向堆地址第 6 字节处，即堆中已填充 jmp 指令的后面，准备下一轮填充下一个函数的 jmp 指令，这样循环后移填充直到所有函数处理完毕。这里就明白了前面为什么申请的堆空间大小＝API 个数\*5 字节。实际上没用完，因为已经有部分函数地址被正确填充到了 IAT，这里填充的只是函数地址位于 70000000 和 77FFFFFF 之间的函数。  
0040D5FB  |||5E                       pop esi  
0040D5FC  |||5F                       pop edi  
0040D5FD  |\\|83C1 04                  add ecx,4                             ; 现在可以知道是取下一个函数指针  
0040D600  | |83C2 04                  add edx,4                             ; IAT 下移一项  
0040D603  | \\8339 00                  cmp dword ptr ds:\[ecx\],0              ; 函数指针是否为空  
0040D606  |^ 0F85 34FFFFFF            jnz Yoda's\_C.0040D540                 ; 不为空就继续循环处理  
0040D60C  |  83C6 0C                  add esi,0C                            ; 否则取下一个 IID  
0040D60F  |  837E 04 00               cmp dword ptr ds:\[esi+4\],0            ; 输入表是否已到末尾  
0040D613  \\^ 0F85 B4FEFFFF            jnz Yoda's\_C.0040D4CD                 ; 否，则返回去继续处理下一个 dll 的函数  
0040D619     33C0                     xor eax,eax  
0040D61B     40                       inc eax  
0040D61C     83F8 01                  cmp eax,1                             ; 正常填充完毕 eax 才等于 1  
0040D61F     74 02                    je short Yoda's\_C.0040D623            ; 正常填充完毕则跳  
0040D621     61                       popad  
0040D622     C3                       retn                                  ; 否则 game over  
  
以上代码完成了 IAT 填充，开始进行下一步工作  
  
跳到了这里：  
0040D623     F785 6C254000 02000000   test dword ptr ss:\[ebp+40256C\],2      ; 3C  
0040D62D    /74 18                    je short Yoda's\_C.0040D647            ; 跳  
0040D62F    |8BBD 64254000            mov edi,dword ptr ss:\[ebp+402564\]  
0040D635    |037F 3C                  add edi,dword ptr ds:\[edi+3C\]  
0040D638    |8B4F 54                  mov ecx,dword ptr ds:\[edi+54\]  
0040D63B    |8BB5 64254000            mov esi,dword ptr ss:\[ebp+402564\]  
0040D641    |C606 00                  mov byte ptr ds:\[esi\],0  
0040D644    |46                       inc esi  
0040D645   ^|E2 FA                    loopd short Yoda's\_C.0040D641  
0040D647    \\8D85 ED1D4000            lea eax,dword ptr ss:\[ebp+401DED\]     ; 地址=0040D060 (offset Yoda's\_C.<ModuleEntryPoint>)  
0040D64D     B9 2A060000              mov ecx,62A  
0040D652     EB 01                    jmp short Yoda's\_C.0040D655           ; 跳  
0040D654   - E9 E8B6FCFF              jmp 003D8D41                          ; 这次没用 C2 而是用 E9 花了  
0040D659     FFEB                     jmp far ebx                           ; 非法使用寄存器（用 FF 隐藏了调用子函数后返回的地址）  
0040D65B     01C7                     add edi,eax  
  
  
拨开迷雾见太阳，跳到了这里：  
0040D655     E8 B6FCFFFF              call Yoda's\_C.0040D310                ; 原来是隐藏了这个 call，这地址好熟悉，对了，就是前面执行过 2 次的校验和函数！  
  {  
  0040D310     8BF8                     mov edi,eax  
  0040D312     33C0                     xor eax,eax  
  0040D314     33DB                     xor ebx,ebx  
  0040D316     33D2                     xor edx,edx  
  0040D318     8A07                     mov al,byte ptr ds:\[edi\]      ; 又来自校验壳代码  
  0040D31A     F7E2                     mul edx  
  0040D31C     03D8                     add ebx,eax  
  0040D31E     42                       inc edx  
  0040D31F     47                       inc edi  
  0040D320   ^ E2 F6                    loopd short Yoda's\_C.0040D318  
  0040D322     93                       xchg eax,ebx  
  0040D323     C3                       retn  
  }  
0040D65A    /EB 01                    jmp short Yoda's\_C.0040D65D           ; 呵呵，这个指令就是被拆开隐藏在 0040D659 和 0040D65B 那两句代码中的。  
0040D65C    |C7                       ???                                   ; 未知命令  
0040D65D    \\8B9D 74254000            mov ebx,dword ptr ss:\[ebp+402574\]     ; ss:\[ebp+402574\]=0002494C，这是什么东东？对了，就是前面保存的 62Ah 字节壳代码的校验和  
0040D663     33C3                     xor eax,ebx                           ; 这里来比较文件是否被改动过  
0040D665    /74 08                    je short Yoda's\_C.0040D66F            ; 校验相符则跳  
0040D667    |EB 01                    jmp short Yoda's\_C.0040D66A           ; 否则？game over？呵呵目标地址又藏起来了，懒得理它！  
  
跳到了这里：  
0040D66F     8DBD 17244000            lea edi,dword ptr ss:\[ebp+402417\]     ; 地址=0040D68A，刚刚正是自校验从壳入口到这个地址之间代码的校验和，40D060+62A=40D68A  
0040D675     8BF7                     mov esi,edi  
0040D677     B9 DF000000              mov ecx,0DF                           ; 0040D68A+DF=0040D769  
0040D67C     33DB                     xor ebx,ebx  
0040D67E     AC                       lods byte ptr ds:\[esi\]                ; esi=0040D68A  
0040D67F     34 79                    xor al,79  
0040D681     2AC3                     sub al,bl  
0040D683     C0C0 02                  rol al,2  
0040D686     AA                       stos byte ptr es:\[edi\]                ; 又是解密代码。这次是将什么代码解密出来呢？edi=0040D68A 往下看  
0040D687     43                       inc ebx  
0040D688   ^ E2 F4                    loopd short Yoda's\_C.0040D67E  
0040D68A     1A1B                     sbb bl,byte ptr ds:\[ebx\]              ; edi 指向的是解密紧接着的这段代码  
。。。。。。  
  
  
解密之后就出现下面的代码了：  
0040D68A     8D85 A4274000            lea eax,dword ptr ss:\[ebp+4027A4\]     ; "IsDebuggerPresent"出现了！  
0040D690     50                       push eax  
0040D691     FFB5 05274000            push dword ptr ss:\[ebp+402705\]        ; Kernel32.dll 句柄  
0040D697     FF95 F4264000            call dword ptr ss:\[ebp+4026F4\]        ; GetProcAddress 获取 IsDebuggerPresent 函数地址  
0040D69D     0BC0                     or eax,eax                            ; eax=7C813133 (kernel32.IsDebuggerPresent)  
0040D69F    /74 08                    je short Yoda's\_C.0040D6A9            ; 敢不敢进雷区？先试一趟好了  
0040D6A1    |FFD0                     call eax                              ; kernel32.IsDebuggerPresent  
0040D6A3    |0BC0                     or eax,eax                            ; eax=00000000，也没怎么着嘛，原来我隐藏了 OD，呵呵。  
0040D6A5    |74 02                    je short Yoda's\_C.0040D6A9            ; 顺利过这一道关！看来要接近尾声了  
0040D6A7    |61                       popad  
0040D6A8    |C3                       retn  
0040D6A9    \\F785 6C254000 01000000   test dword ptr ss:\[ebp+40256C\],1      ; 3C    
0040D6B3    /74 4F                    je short Yoda's\_C.0040D704            ; 跳  
  
跳到了这里：  
0040D704     8D85 CB244000            lea eax,dword ptr ss:\[ebp+4024CB\]     ; 地址=0040D73E，又是这种 push+retn 方式跳过子函数，没新意  
0040D70A     50                       push eax  
0040D70B     C3                       retn                                  ; 转移战场  
  
转移到了这里：  
0040D73E     32C0                     xor al,al  
0040D740     8DBD ED1D4000            lea edi,dword ptr ss:\[ebp+401DED\]     ; 地址=0040D060 (offset Yoda's\_C.<ModuleEntryPoint>)  
0040D746     B9 AC060000              mov ecx,6AC                           ; 40D060+6AC=0040D70C ! 即刚刚才转移前的地点  
0040D74B    /AA                       stos byte ptr es:\[edi\]                ; 自毁壳代码！把转移之前的代码全部清除！  
0040D74C   ^\\E2 FD                    loopd short Yoda's\_C.0040D74B  
0040D74E     8DBD F6244000            lea edi,dword ptr ss:\[ebp+4024F6\]     ; 地址=0040D769  
0040D754     B9 C0020000              mov ecx,2C0                           ; 0040D769+2C0=0040DA29  
0040D759     AA                       stos byte ptr es:\[edi\]                ; 将 0040D769 之后 2C0 字节的内容全部清 0  
0040D75A   ^ E2 FD                    loopd short Yoda's\_C.0040D759  
0040D75C     61                       popad  
0040D75D     50                       push eax                              ; 0040D70C，前面的第二个悬念终于解开了：之前保存这个数就是为了 popad 后 eax 能很隐蔽地获得该值。同样也看到了 ebx 被改成了 98008021 这个神秘数据，第一个悬念也初露端倪  
0040D75E     33C0                     xor eax,eax  
0040D760     64:FF30                  push dword ptr fs:\[eax\]  
0040D763     64:8920                  mov dword ptr fs:\[eax\],esp            ; 原来是安装 SEH，0040D70C 就是回调函数的入口，也难怪自清除壳代码时保留了那部分代码，赶紧去下断。  
0040D766     EB 01                    jmp short Yoda's\_C.0040D769  
0040D769     0000                     add byte ptr ds:\[eax\],al              ; 写 0 地址人为触发异常  
  
进入到了系统领空，Alt+F9 到断点处：  
0040D70F     57                       push edi  
0040D710     8B45 10                  mov eax,dword ptr ss:\[ebp+10\]         ; CONTEXT  
0040D713     8BB8 C4000000            mov edi,dword ptr ds:\[eax+C4\]         ; regEsp  
0040D719     FF37                     push dword ptr ds:\[edi\]               ; 下一个 SEH 记录指针  
0040D71B     33FF                     xor edi,edi  
0040D71D     64:8F07                  pop dword ptr fs:\[edi\]                ; 恢复下一个 SEH  
0040D720     8380 C4000000 08         add dword ptr ds:\[eax+C4\],8  
0040D727     8BB8 A4000000            mov edi,dword ptr ds:\[eax+A4\]         ; regEbx=98008021  
0040D72D     C1C7 07                  rol edi,7                             ; 解密后=004010CC  
0040D730     89B8 B8000000            mov dword ptr ds:\[eax+B8\],edi         ; regEip 修改变 004010CC，彻底解开了第一个悬念，这就是 OEP，通过这种隐蔽的方式跳转到 OEP。004010CC 处下断  
0040D736     B8 00000000              mov eax,0  
0040D73B     5F                       pop edi  
0040D73C     C9                       leave  
0040D73D     C3                       retn  
  
处理完回调函数后，进入了系统领空，Alt+F9 回到用户领空：  
004010CC     55                       push ebp                              ; OEP  
004010CD     8BEC                     mov ebp,esp  
004010CF     83EC 44                  sub esp,44  
004010D2     56                       push esi  
004010D3     FF15 E0634000            call dword ptr ds:\[4063E0\]            ; kernel32.GetCommandLineA  
004010D9     8BF0                     mov esi,eax  
004010DB     8A00                     mov al,byte ptr ds:\[eax\]  
004010DD     3C 22                    cmp al,22  
004010DF     75 13                    jnz short Yoda's\_C.004010F4  
  
到达 OEP，取消断点，用 OD 的 OllyDump 插件脱壳，可直接运行；如果用 LordPE 脱壳，则要修正映像大小，因为壳将映像大小由 F000 改成了 1000，所以要修正回来，并且需要修复输入表（用 ImportREC 即可）。  
  
  
　　小结：  
　　这个壳是一个比较早期的壳，做了一些反调试和反脱壳措施，但强度很弱。具体做了以下几点：  
　　1、反调试方面，一是用了简单的 C2 和 E9 来混淆代码，二是用了一些简单的技巧来隐藏指令的跳转，如用 push+retn 的形式转移流程；三是做壳代码动态解密，边执行边解密，而不是一次性将代码全部解密；四是用了一个 IsDebuggerPresent 函数检测是否被调试，很轻易的能被避开检测。  
　　2、防程序修改方面，做了两次代码自校验，而且故意将计算与比较放在比较远的位置，但对于 DA41 字节的校验，只要在内存中修改而不保存，或保存文件的同时将校验和也修改则检测不出改动过；而对于 62A 字节的校验，两次计算都是检测同一段内存中的代码，只要在第一次计算之前修改，也检测不出来。  
　　3、在跳往 OEP 方面，做的比较隐蔽。一是很早就将 OEP 加密，再通过修改堆栈的方法，使得 popad 时很隐蔽的就传给了寄存器 ebx，并且是在“暗桩”（SEH 回调函数）中用 1 个指令解密就立即修改 EIP 指针，如果稍不注意就会跑飞了。  
　　4、输入表处理方面，加壳时破坏了原输入表，将输入表指向了一个壳中的伪输入表，伪输入表中只保留了 1 个 IID 且除 NAME 和 FIRSTTHUNK 外的其他 3 个成员为空，IAT 只有两个函数，即 kernel32.LoadLibraryA 和 kernel32.GetProcAddress，“KERNEL32.dll”被改成了“KeRnEl32.dLl”（API 对大小写敏感）导致无效，壳在运行时是从另外的地方获得“Kernel32.dll”这个名字再通过 LoadLibraryA 动态调入 Kernel32.dll，然后通过 GetProcAddress 获得壳代码运行时需要的一些函数。而对于源程序运行时需要的输入表，则是在壳的另外一个地方（地址=0040D7EF）保存了一张真正的输入表，但也是变形过了的，每个 IID 对应一个 dll，并且压缩成 3 个成员：第一个是成员 Name，第二个 FirstThunk，第三个是 OriginalFirstThunk。加壳时，对只是对源程序的 dll 名称和函数名称加密，并没有删除或转移位置，壳中变形的输入表分别指向这些加密了的名称，FirstThunk 指向的函数地址都被篡改。壳在运行时，先根据变形输入表找到加密的 dll，解密后动态调入 dll，获得句柄，随即根据判断标志将 dll 名称擦除。在填充 IAT 时，把部分函数地址正确填充，而函数地址在 70000000 和 77FFFFFF 之间的函数都作了处理，IAT 中填充的是壳申请的堆地址，而将跳转到正确函数地址的 jmp 指令放入相应的堆地址中，用 lordPE 脱壳时堆是无法 dump 出来的，因此，用 lordPE 脱壳后的程序运行时会报内存读写冲突的错误信息，需要修复输入表，而 OD 的 OllyDump 插件则能很好地解决这个问题。  
  
　　附上带壳的记事本程序：

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || Yoda's Crypter v1.2.98.E.rar  
 |