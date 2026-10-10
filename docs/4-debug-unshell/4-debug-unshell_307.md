

# ASPack 2.12 - Alexey Solodovnikov 脱壳分析

> 原文：[`www.pediy.com/kssd/pediy11/114576.html`](https://www.pediy.com/kssd/pediy11/114576.html)

【前言】没有其他目的，只是作个学习笔记存档，兼与与我这样的菜鸟交流。错误之处，欢迎批评指正。  
【软件】加了 ASPack 2.12 壳的 KillBox.exe，一个小工具程序，由天草老师教程中提供。  
【工具】OD、PEiD、ImportREC  
用 PEiD 查一下，是 ASPack 2.12 - Alexey Solodovnikov 壳，下面 OD 载入，开始分析。  
  
程序入口：  
00424001 K>  60                       pushad                                        ; 程序入口(EP)  
00424002     E8 03000000              call KillBox.0042400A                         ; =======================  
00424007   - E9 EB045D45              jmp 459F44F7                                  ; 这 4 行都是花指令  
0042400C     55                       push ebp                                      ;  
0042400D     C3                       retn                                          ; =======================  
0042400E     E8 01000000              call KillBox.00424014                         ; 目标地址隐藏了，单步进去  
00424013     EB 5D                    jmp short KillBox.00424072                    ; EB 花指令，隐藏了上面 call 的目标地址  
  
单步进去后是这样的：  
00424014     5D                       pop ebp                                       ; 代码重定位。弹出返回地址 ebp=00424013  
00424015     BB EDFFFFFF              mov ebx,-13                                   ; 偏移量为 13h 字节  
0042401A     03DD                     add ebx,ebp                                   ; 00424013-13=00424000 壳代码段起始位置  
0042401C     81EB 00400200            sub ebx,24000                                 ; 00424000-24000=00400000(ImageBase)  
00424022     83BD 22040000 00         cmp dword ptr \[ebp+422\],0                     ; \[ebp+422\]=\[00424435\]=00000000  
00424029     899D 22040000            mov dword ptr \[ebp+422\],ebx                   ; \[ebp+422\]存入 00400000(ImageBase)  
0042402F     0F85 65030000            jnz KillBox.0042439A                          ; 初始为 0，进行解压、填充 IAT 处理，完成后再来这里判断，发现是 00400000 就去 OEP）  
00424035     8D85 2E040000            lea eax,dword ptr \[ebp+42E\]                   ; ASCII "kernel32.dll"  
0042403B     50                       push eax  
0042403C     FF95 4D0F0000            call dword ptr \[ebp+F4D\]                      ; GetModuleHandleA 获取 kernel32.dll 句柄  
00424042     8985 26040000            mov dword ptr \[ebp+426\],eax                   ; 保存句柄  
00424048     8BF8                     mov edi,eax  
0042404A     8D5D 5E                  lea ebx,dword ptr \[ebp+5E\]                    ; ASCII "VirtualAlloc"  
0042404D     53                       push ebx  
0042404E     50                       push eax                                      ; kernel32.dll 句柄  
0042404F     FF95 490F0000            call dword ptr \[ebp+F49\]                      ; GetProcAddress 获取 kernel32.VirtualAlloc 函数地址  
00424055     8985 4D050000            mov dword ptr \[ebp+54D\],eax                   ; 保存函数地址  
0042405B     8D5D 6B                  lea ebx,dword ptr \[ebp+6B\]                    ; ASCII "VirtualFree"  
0042405E     53                       push ebx  
0042405F     57                       push edi  
00424060     FF95 490F0000            call dword ptr \[ebp+F49\]                      ; GetProcAddress 获取 kernel32.VirtualFree 函数地址  
00424066     8985 51050000            mov dword ptr \[ebp+551\],eax                   ; 保存函数地址  
0042406C     8D45 77                  lea eax,dword ptr \[ebp+77\]                    ; 地址=0042408A  
0042406F     FFE0                     jmp eax                                       ; 目标地址 0042408A 又藏花指令的后面了  
  
跳到了这里，看到上面的两个函数，猜到下面可能是开始解压代码了。  
0042408A     8B9D 31050000            mov ebx,dword ptr \[ebp+531\]                   ; ss:\[00424544\]=00000000  
00424090     0BDB                     or ebx,ebx  
00424092    /74 0A                    je short KillBox.0042409E                     ; 跳  
00424094    |8B03                     mov eax,dword ptr \[ebx\]  
00424096    |8785 35050000            xchg dword ptr \[ebp+535\],eax  
0042409C    |8903                     mov dword ptr \[ebx\],eax  
0042409E    \\8DB5 69050000            lea esi,dword ptr \[ebp+569\]                   ; 地址=0042457C。这是什么地址？数据窗口看看，是一组数据：00001000、00020000、00021000、00001000、000230E8、00000F18，应该与解压区段有关系，这里暂称其为信息表吧。  
004240A4     833E 00                  cmp dword ptr \[esi\],0                         ; 信息表第 1 项是否为 0？（ds:\[0042457C\]=00001000）  
004240A7     0F84 21010000            je KillBox.004241CE                           ; 非 0 不跳  
004240AD     6A 04                    push 4                                        ; flProtect＝PAGE\_READWRITE  
004240AF     68 00100000              push 1000                                     ; flAllocationType＝MEM\_COMMIT 提交物理内存  
004240B4     68 00180000              push 1800                                     ; dwSize=1800h 字节  
004240B9     6A 00                    push 0                                        ; lpAddress=NULL，不指定起始地址  
004240BB     FF95 4D050000            call dword ptr \[ebp+54D\]                      ; VirtualAlloc 申请第一块内存  
004240C1     8985 56010000            mov dword ptr \[ebp+156\],eax                   ; 保存申请到的内存起始地址。我这里是 00D50000  
  
这里开始一个大循环，各段解压：  
004240C7     8B46 04                  mov eax,dword ptr \[esi+4\]                     ; 信息表中取出第 2 项：00020000。从下面知道这是 VirtualAlloc 的 dwSize，也就是要解压的区段的大小了。  
004240CA     05 0E010000              add eax,10E                                   ; 每个大小还要加上 10Eh 字节，为什么？  
004240CF     6A 04                    push 4                                        ; /flProtect＝PAGE\_READWRITE  
004240D1     68 00100000              push 1000                                     ; |flAllocationType＝MEM\_COMMIT 提交物理内存  
004240D6     50                       push eax                                      ; |dwSize  
004240D7     6A 00                    push 0                                        ; |  
004240D9     FF95 4D050000            call dword ptr \[ebp+54D\]                      ; \\VirtualAlloc 申请第二块内存，且称其为缓冲区吧  
004240DF     8985 52010000            mov dword ptr \[ebp+152\],eax                   ; 保存申请到的内存起始地址。我这里是 00D60000  
004240E5     56                       push esi                                      ; 信息表地址入栈保存  
004240E6     8B1E                     mov ebx,dword ptr \[esi\]                       ; 取出信息表第 1 项：00001000  
004240E8     039D 22040000            add ebx,dword ptr \[ebp+422\]                   ; RVA+ImageBase  
到这里明白了，信息表第 1 项是待解压的代码段起始地址的 RVA，而第 2 项是大小，在内存窗口看看，果然是.text 段的参数。再看剩下的两组数据：  
00021000、00001000 和 000230E8、00000F18，分别是.data 段的 RVA 和.rsrc 段偏移 E8 字节的 RVA，这里分别只解压 1000h 和 0F18h 字节。为什么会这样呢？代码段全部解压，数据段解压一半，而资源段只选择性的解压，大概是因为资源段不好被全部压缩，否则在 WINDOWS 下怎么能看到图标之类的信息呢？  
  
004240EE     FFB5 56010000            push dword ptr \[ebp+156\]                      ; 申请的第一块内存地址  
004240F4     FF76 04                  push dword ptr \[esi+4\]                        ; 信息表中取出的该段待解压数据大小  
004240F7     50                       push eax                                      ; 缓冲区地址  
004240F8     53                       push ebx                                      ; 待解压的起始地址  
004240F9     E8 6E050000              call KillBox.0042466C                         ; 解压到缓冲区  
004240FE     B3 01                    mov bl,0                                      ; 初始指令为：mov bl,0，00424105 处指令将这里改为 mov bl,1  
00424100     80FB 00                  cmp bl,0  
00424103     75 5E                    jnz short KillBox.00424163                    ; 初始时 bl＝0，所以不跳，先对解压到缓冲区中的代码中的 call 和 jmp 指令进行解密，再将缓冲区的数据（指令）复制到目标区段（代码段）。  
00424105     FE85 EC000000            inc byte ptr \[ebp+EC\]                         ; 这里有玄机，将 004240FE 处指令改为“mov bl,1”，所以处理完代码段后对其他区段解压时，再来判断就跳了，直接复制解压后的数据到目标区段。  
0042410B     8B3E                     mov edi,dword ptr \[esi\]                       ; ds:\[0042457C\]=00001000  
0042410D     03BD 22040000            add edi,dword ptr \[ebp+422\]                   ; 1000h+ImageBase=代码段首地址  
00424113     FF37                     push dword ptr \[edi\]                          ; 00401000 处 4 字节代码入栈保存  
00424115     C607 C3                  mov byte ptr \[edi\],0C3                        ; 将 00401000 处 1 字节改为 C3（retn）  
00424118     FFD7                     call edi                                      ; edi=00401000 (retn)“到此一游”  
  {  
  00401000     C3                       retn  
  }  
0042411A     8F07                     pop dword ptr \[edi\]                           ; 恢复 00401000 处 4 字节代码  
0042411C     50                       push eax  
0042411D     51                       push ecx  
0042411E     56                       push esi  
0042411F     53                       push ebx  
00424120     8BC8                     mov ecx,eax  
00424122     83E9 06                  sub ecx,6                                     ; ecx=00020000-6＝0001FFFA  
00424125     8BB5 52010000            mov esi,dword ptr \[ebp+152\]                   ; 缓冲区  
0042412B     33DB                     xor ebx,ebx                                   ; 初始化计数器  
  
下面开始修改解压后的代码，将 E804 和 E904 开始的 call 和 jmp 目标地址解密：  
0042412D /  /0BC9                     or ecx,ecx                                    ; 查找范围字节数。初始＝0001FFFA  
0042412F |  |74 2E                    je short KillBox.0042415F                     ; 为 0 则跳  
00424131 |  |78 2C                    js short KillBox.0042415F                     ; 为负数（即超出了搜索范围）则跳  
00424133 |  |AC                       lods byte ptr \[esi\]                           ; 从缓冲区查找特征字符（E8、E9）  
00424134 |  |3C E8                    cmp al,0E8                                    ; 是否为 call 指令  
00424136 |  |74 0A                    je short KillBox.00424142                     ; 为 call 指令则去解密  
00424138 |/ |EB 00                    jmp short KillBox.0042413A  
0042413A || |3C E9                    cmp al,0E9                                    ; 是否为 jmp 指令  
0042413C ||/|74 04                    je short KillBox.00424142                     ; 为 jmp 指令则则去解密  
0042413E ||||43                       inc ebx                                       ; 计数器加 1  
0042413F ||||49                       dec ecx                                       ; 查找范围字节数－1  
00424140 |||\\EB EB                    jmp short KillBox.0042412D  
00424142 |\\\\ 8B06                     mov eax,dword ptr \[esi\]                       ; E8 或 E9 后面再取一字符，即 jmp XXXXXXXX 指令的目标地址最低位字节。  
00424144 |   EB 00                    jmp short KillBox.00424146  
00424146 |   803E 04                  cmp byte ptr \[esi\],4                          ; 判断是否“E904”或“E804”（00D602EC 处才有 E904，00D60DE5 处才有 E804）  
00424149 | ^ 75 F3                    jnz short KillBox.0042413E                    ; 不是继续循环搜索  
0042414B |   24 00                    and al,0                                      ; 对 jmp 和 call 的目标地址进行解密  
0042414D |   C1C0 18                  rol eax,18  
00424150 |   2BC3                     sub eax,ebx  
00424152 |   8906                     mov dword ptr \[esi\],eax                       ; 修改 jmp 和 call 目标地址  
00424154 |   83C3 05                  add ebx,5                                     ; 计数器加 5（jmp 指令共 5 字节）  
00424157 |   83C6 04                  add esi,4                                     ; 指针后移 4 字节，即指向 jmp 指令后面的一条指令处  
0042415A |   83E9 05                  sub ecx,5                                     ; 查找范围字节数－5  
0042415D \\ ^ EB CE                    jmp short KillBox.0042412D  
  
修改完毕后跳到这里：  
0042415F     5B                       pop ebx  
00424160     5E                       pop esi  
00424161     59                       pop ecx  
00424162     58                       pop eax  
00424163     EB 08                    jmp short KillBox.0042416D                    ; 解压其他区段时是直接跳到这里  
  
跳到了这里，将缓冲区的代码（数据）复制到目标区段：  
0042416D     8BC8                     mov ecx,eax                                   ; eax=解压数据大小  
0042416F     8B3E                     mov edi,dword ptr \[esi\]                       ; 本区段待解压数据的起始地址 RVA  
00424171     03BD 22040000            add edi,dword ptr \[ebp+422\]                   ; RVA+ImageBase=目标地址  
00424177     8BB5 52010000            mov esi,dword ptr \[ebp+152\]                   ; 缓冲区  
0042417D     C1F9 02                  sar ecx,2  
00424180     F3:A5                    rep movs dword ptr es:\[edi\],dword ptr \[esi\]   ; 复制  
00424182     8BC8                     mov ecx,eax  
00424184     83E1 03                  and ecx,3  
00424187     F3:A4                    rep movs byte ptr es:\[edi\],byte ptr \[esi\]  
00424189     5E                       pop esi  
0042418A     68 00800000              push 8000  
0042418F     6A 00                    push 0  
00424191     FFB5 52010000            push dword ptr \[ebp+152\]                      ; 缓冲区  
00424197     FF95 51050000            call dword ptr \[ebp+551\]                      ; VirtualFree 释放缓冲区  
  
0042419D     83C6 08                  add esi,8                                     ; 信息表中指向下一个待解压的区段  
004241A0     833E 00                  cmp dword ptr \[esi\],0                         ; 是否还有待解压的段？  
004241A3   ^ 0F85 1EFFFFFF            jnz KillBox.004240C7                          ; 有则循环继续解压  
  
至此解压全部完成，接下来清理内存：  
004241A9     68 00800000              push 8000  
004241AE     6A 00                    push 0  
004241B0     FFB5 56010000            push dword ptr \[ebp+156\]                      ; 申请的第一块内存地址  
004241B6     FF95 51050000            call dword ptr \[ebp+551\]                      ; VirtualFree 释放第一块内存  
004241BC     8B9D 31050000            mov ebx,dword ptr \[ebp+531\]                   ; ss:\[00424544\]=00000000  
004241C2     0BDB                     or ebx,ebx  
004241C4     74 08                    je short KillBox.004241CE                     ; 跳  
004241C6     8B03                     mov eax,dword ptr \[ebx\]  
004241C8     8785 35050000            xchg dword ptr \[ebp+535\],eax  
004241CE     8B95 22040000            mov edx,dword ptr \[ebp+422\]                   ; ImageBase  
004241D4     8B85 2D050000            mov eax,dword ptr \[ebp+52D\]                   ; ImageBase  
004241DA     2BD0                     sub edx,eax  
004241DC     74 79                    je short KillBox.00424257                     ; 相等，跳  
  
至此，程序解压完毕，跳到了这里，开始填充 IAT。  
00424257     8B95 22040000            mov edx,dword ptr \[ebp+422\]                   ; ImageBase  
0042425D     8BB5 41050000            mov esi,dword ptr \[ebp+541\]                   ; ss:\[00424554\]=00000000  
00424263     0BF6                     or esi,esi  
00424265    /74 11                    je short KillBox.00424278                     ; 跳  
00424267    |03F2                     add esi,edx  
00424269    |AD                       lods dword ptr \[esi\]  
0042426A    |0BC0                     or eax,eax  
0042426C    |74 0A                    je short KillBox.00424278  
0042426E    |03C2                     add eax,edx  
00424270    |8BF8                     mov edi,eax  
00424272    |66:AD                    lods word ptr \[esi\]  
00424274    |66:AB                    stos word ptr es:\[edi\]  
00424276   ^|EB F1                    jmp short KillBox.00424269  
00424278    \\BE B4F80100              mov esi,1F8B4                                 ; 跳到这里  
0042427D     8B95 22040000            mov edx,dword ptr \[ebp+422\]                   ; ImageBase  
00424283     03F2                     add esi,edx                                   ; 1F8B4+ImageBase 定位到输入表  
00424285     8B46 0C                  mov eax,dword ptr \[esi+C\]                     ; 取 IID 的 NAME 成员  
00424288     85C0                     test eax,eax                                  ; NAME 成员是否为 0  
0042428A     0F84 0A010000            je KillBox.0042439A                           ; 是，则跳，即 IAT 填充完毕就跳出循环。  
00424290     03C2                     add eax,edx                                   ; 否，则定位 dll，1FB94+ImageBase  
00424292     8BD8                     mov ebx,eax                                   ; ASCII "MSVBVM60.DLL"（VB 程序）  
00424294     50                       push eax  
00424295     FF95 4D0F0000            call dword ptr \[ebp+F4D\]                      ; GetModuleHandleA 获取 dll 句柄  
0042429B     85C0                     test eax,eax  
0042429D     75 07                    jnz short KillBox.004242A6                    ; 成功则跳  
0042429F     53                       push ebx                                      ; 否则 LoadLibraryA 动态调入 dll 库  
004242A0     FF95 510F0000            call dword ptr \[ebp+F51\]  
004242A6     8985 45050000            mov dword ptr \[ebp+545\],eax                   ; 保存 dll 句柄  
004242AC     C785 49050000 00000000   mov dword ptr \[ebp+549\],0                     ; 初始 IAT 首地址偏移量为 0，以后每填充完一个就+4  
004242B6     8B95 22040000            mov edx,dword ptr \[ebp+422\]                   ; ImageBase  
004242BC     8B06                     mov eax,dword ptr \[esi\]                       ; \[esi\]处是输入表 MSVBVM60.DLL 的 IID，这里取 OriginalFirstThunk  
004242BE     85C0                     test eax,eax                                  ; OriginalFirstThunk 是否为 0？  
004242C0    /75 03                    jnz short KillBox.004242C5                    ; 否，则跳去找函数名（本例非 0）  
004242C2    |8B46 10                  mov eax,dword ptr \[esi+10\]                    ; 是，则取 FirstThunk(＝00001000)  
004242C5    \\03C2                     add eax,edx                                   ; OriginalFirstThunk+ImageBase 定位到 IMAGE\_THUNK\_DATA 数组首地址  
004242C7     0385 49050000            add eax,dword ptr \[ebp+549\]                   ; IMAGE\_THUNK\_DATA 数组首地址+偏移量(N\*4 字节)，定位到待填充函数的 IMAGE\_THUNK\_DATA 地址  
004242CD     8B18                     mov ebx,dword ptr \[eax\]                       ; 取 IMAGE\_THUNK\_DATA  
004242CF     8B7E 10                  mov edi,dword ptr \[esi+10\]                    ; 取 FirstThunk  
004242D2     03FA                     add edi,edx                                   ; FirstThunk+ImageBase 定位到 IAT 首地址  
004242D4     03BD 49050000            add edi,dword ptr \[ebp+549\]                   ; IAT 首地址+偏移量(N\*4 字节)，定位到本次待填充的 IAT 目标地址  
004242DA     85DB                     test ebx,ebx                                  ; IMAGE\_THUNK\_DATA 数组是否已到末尾？  
004242DC     0F84 A2000000            je KillBox.00424384                           ; 是，则结束  
004242E2     F7C3 00000080            test ebx,80000000                             ; 否，则判断该函数 IMAGE\_IMPORT\_BY\_NAME 还是 BY\_INDEX 方式导入。（在填充了部分函数后，我们可以看到本例是 VB 语言写的程序，以"rtc"开头的函数都是以序号导入的方式，其他函数的则是 BY\_NAME 方式）  
004242E8     75 04                    jnz short KillBox.004242EE                    ; by\_INDEX 方式则跳。  
004242EA     03DA                     add ebx,edx                                   ; by\_NAME 方式则+ImageBase 定位函数名  
004242EC     43                       inc ebx  
004242ED     43                       inc ebx                                       ; 后移 2 位跳过成员 Hint 定位到函数名地址  
004242EE     53                       push ebx                                      ; by\_NAME 方式时为函数名地址，by\_INDEX 方式是时为 IMAGE\_THUNK\_DATA  
004242EF     81E3 FFFFFF7F            and ebx,7FFFFFFF                              ; 取序号导入的函数序号(按名称导入的不影响)  
004242F5     53                       push ebx                                      ; 函数名地址或函数序号  
004242F6     FFB5 45050000            push dword ptr \[ebp+545\]                      ; dll 句柄  
004242FC     FF95 490F0000            call dword ptr \[ebp+F49\]                      ; GetProcAddress 获取函数地址  
00424302     85C0                     test eax,eax  
00424304     5B                       pop ebx  
00424305     75 6F                    jnz short KillBox.00424376                    ; 成功则跳  
  
00424376     8907                     mov dword ptr \[edi\],eax                       ; 跳到了这里，填充 IAT  
00424378     8385 49050000 04         add dword ptr \[ebp+549\],4                     ; 距首地址偏移量+4，准备处理下一个函数  
0042437F   ^ E9 32FFFFFF              jmp KillBox.004242B6                          ; 循环处理  
  
一个 DLL 的 IAT 填充完毕后跳转到这里：  
00424384     8906                     mov dword ptr \[esi\],eax                       ; OriginalFirstThunk 成员指向 00000000  
00424386     8946 0C                  mov dword ptr \[esi+C\],eax                     ; NAME 成员指向 00000000  
00424389     8946 10                  mov dword ptr \[esi+10\],eax                    ; FirstThunk 成员指向 00000000  
0042438C     83C6 14                  add esi,14                                    ; 指针移到下一个 IID  
0042438F     8B95 22040000            mov edx,dword ptr \[ebp+422\]                   ; ImageBase  
00424395   ^ E9 EBFEFFFF              jmp KillBox.00424285  
  
全部 IAT 填充完毕后跳到这里，下面开始处理 OEP：  
0042439A     B8 60230000              mov eax,2360                                  ; OEP 的 RVA  
0042439F     50                       push eax  
004243A0     0385 22040000            add eax,dword ptr \[ebp+422\]                   ; RVA+ImageBase = OEP  
004243A6     59                       pop ecx                                       ; 弹出 OEP 的 RVA  
004243A7     0BC9                     or ecx,ecx                                    ; OEP 的 RVA 是否为 0  
004243A9     8985 A8030000            mov dword ptr \[ebp+3A8\],eax                   ; 修改程序代码，将 OEP 写入压栈指令  
  
修改程序代码之前下面的几行代码是这样的：  
004243AF     61                       popad  
004243B0     75 08                    jnz short KillBox.004243BA                    ; OEP 的 RVA 不为 0 则跳，否则 game over  
004243B2     B8 01000000              mov eax,1  
004243B7     C2 0C00                  retn 0C  
004243BA     68 00000000              push 0   
004243BF     C3                       retn  
  
修改程序代码后是这样的：  
004243AF     61                       popad  
004243B0     75 08                    jnz short KillBox.004243BA                    ; OEP 的 RVA 不为 0 则跳，否则 game over  
004243B2     B8 01000000              mov eax,1  
004243B7     C2 0C00                  retn 0C  
004243BA     68 60234000              push KillBox.00402360                         ; 这里被修改成了 push 00402360  
004243BF     C3                       retn                                          ; 前往 OEP  
  
到达 OEP：  
00402360     68 2C4D4000              push KillBox.00404D2C                         ; ASCII "VB5!6&\*vb6chs.dll"  
00402365     E8 EEFFFFFF              call KillBox.00402358  
0040236A     0000                     add byte ptr \[eax\],al  
0040236C     0000                     add byte ptr \[eax\],al  
0040236E     0000                     add byte ptr \[eax\],al  
00402370     3000                     xor byte ptr \[eax\],al  
00402372     0000                     add byte ptr \[eax\],al  
00402374     3800                     cmp byte ptr \[eax\],al  
00402376     0000                     add byte ptr \[eax\],al  
00402378     0000                     add byte ptr \[eax\],al  
0040237A     0000                     add byte ptr \[eax\],al  
0040237C     4F                       dec edi  
0040237D     C2 F150                  retn 50F1  
  
现在可以用 OD 的 OllyDump 插件脱壳了，用方式一脱后可运行；用方式二脱后不能运行，原因是方式二没能正确识别以序号方式导入的函数，均成为无效函数，需要修复输入表（用 ImportREC 即可）。  
  
  
小结：  
一、壳的工作流程是：  
1、根据壳中的伪输入表，动态调入壳运行所需的函数。  
2、解压。将原程序的各个段依次解压到缓冲区，再从缓冲区中复制到目标区段。如果是代码段，则在缓冲区中将所有 E804 和 E904 开头的 call 和 jmp 指令解密出正确的目标地址，再行复制。  
3、填充 IAT。保留了输入表，根据输入表动态加载 dll 库，再根据 OriginalFirstThunk 迭代搜索函数名，动态获取函数地址再填入 IAT，填充完一个 dll 的函数后，却又把相应的 IID 的 OriginalFirstThunk、Name、FirstThunk 三个成员全指向了 0。  
4、转往 OEP。壳通过自修改代码，以 push＋retn 的方式跳往 OEP。  
  
二、程序保护方面  
这个壳也是一个早期的压缩壳，所以在保护方法也没采取什么措施。下面是一些简单的措施：  
1、采用了一些花指令干扰静态调试。  
2、对部分 call 和 jmp 的目标地址似乎是加密了，不过解压后在缓冲区中又解密并复制到代码段中。  
3、两次用了 SMC 技术，第一次却不是出于保护目的，而是为了控制正常流程（将 mov bl,0 改为 mov bl,1），使程序除代码段之外不去执行额外的修改 call 和 jmp 目标地址的工作；第二次倒是起到干扰静态调试的作用（将 push 0 改为 push 00402360）。

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || KillBox.rar  
 |