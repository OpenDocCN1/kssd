

# ACProtect 1.21

> 原文：[`www.pediy.com/kssd/pediy06/pediy6434.htm`](https://www.pediy.com/kssd/pediy06/pediy6434.htm)

ACProtect 1.21 专业版主程序的脱壳   
  
操作系统:WIN2K    
工具：OLLYDBG1.1，ImportREC，LordPE    
下载地址：  http://www.ultraprotect.com/acpr\_pro.exe    
目标程序：ACProtect 1.21 专业版主程序。    
  
脱壳过程:    
1.寻找 Stolen code 的变形及其伪 OEP。    
用 OLLYDBG1.1 加载后，停留在程序的入口：    
006D4000   pushad   
006D4001   dec ecx   
006D4002   sbb esi,B59B7C21   
006D4008   clc   
006D4009   mov ecx,ebx   
006D400B   dec ecx   
    用 IsDebug 1.4 插件去掉 Ollydbg 的调试器标志。忽略除了“int3 异常”之外的所有其他异常选项。   
查看堆栈 esp=12ffc4,根据此值来设置硬件访问断点：去转存窗口，Ctrl+G:0012FFC0(入口 esp-4),在 0012FFC0 处的 4 个字节上,下硬件访问－>DWord 断点。    
F9 开始运行,断下后再取消该硬件断点。    
006E56FF   popad   
006E5700   mov dword ptr ds:\[6D4E73\],ebx   
006E5706   push dword ptr ds:\[6D4E73\]   
006E570C   push eax   
006E570D   mov eax,ACProtec.006D4ECF   
006E5712   mov ebx,eax   
006E5714   pop eax                                       ; KERNEL32.77E887E7   
006E5715   mov dword ptr ds:\[ebx\],ecx   
006E5717   pop ebx                                       ; KERNEL32.77E887E7   
006E5718   push dword ptr ds:\[6D4ECF\]   
006E571E   mov dword ptr ss:\[esp\],esi   
006E5721   mov dword ptr ss:\[esp\],ebx   
006E5724   mov dword ptr ss:\[esp\],eax   
006E5727   push edx   
006E5728   pop dword ptr ds:\[6D4ECB\]                     ; KERNEL32.77E887E7   
006E572E   push dword ptr ds:\[6D4ECB\]   
006E5734   mov dword ptr ss:\[esp\],ecx   
006E5737   mov dword ptr ss:\[esp\],ACProtec.006D4F0F   
006E573E   pop dword ptr ds:\[6D4EEF\]                     ; KERNEL32.77E887E7   
006E5744   mov dword ptr ds:\[6D4E9F\],ebx   
006E574A   push dword ptr ds:\[6D4E9F\]   
006E5750   pushad   
006E5751   call ACProtec.006E1460   
006E5756   call ACProtec.006E575B   
006E575B   pop ebx                                       ; KERNEL32.77E887E7   
006E575C   sub ebx,dword ptr ss:\[ebp+4023EC\]   
006E5762   sub ebx,1175B   
006E5768   mov dword ptr ss:\[ebp+40D2AA\],ebx   
006E576E   popad   
006E576F   mov dword ptr ss:\[esp\],edi                    ; ACProtec.005D16A8   
006E5772   mov dword ptr ds:\[6D4E9B\],ACProtec.006D4EEF   
006E577C   push dword ptr ds:\[6D4E9B\]   
006E5782   pop dword ptr ds:\[6D4E97\]                     ; KERNEL32.77E887E7   
006E5788   mov edi,dword ptr ds:\[6D4E97\]   
006E578E   mov eax,dword ptr ds:\[edi\]   
006E5790   mov edi,dword ptr ss:\[esp\]                    ; KERNEL32.77E887E7   
006E5793   pop dword ptr ds:\[6D4EC7\]                     ; KERNEL32.77E887E7   
006E5799   mov dword ptr ds:\[eax\],ebp  //stolen code 第 1 行,push ebp 的变形    
006E579B   mov eax,dword ptr ss:\[esp\]                    ; KERNEL32.77E887E7   
006E579E   pop dword ptr ds:\[6D4EFB\]                     ; KERNEL32.77E887E7   
006E57A4   push dword ptr ds:\[6D4F0F\]   
006E57AA   mov dword ptr ds:\[6D4F0B\],esp   
006E57B0   push dword ptr ds:\[6D4F0B\]   
006E57B6   pop dword ptr ds:\[6D4EF7\]                     ; KERNEL32.77E887E7   
006E57BC   push eax   
006E57BD   nop   
006E57BE   nop   
006E57BF   pushad   
006E57C0   call ACProtec.006E1460   
006E57C5   call ACProtec.006E11FE   
006E57CA   mov dword ptr ss:\[ebp+40E205\],eax   
006E57D0   popad   
006E57D1   mov dword ptr ss:\[esp\],edx   
006E57D4   mov dword ptr ds:\[6D4E93\],ACProtec.006D4EDB   
006E57DE   mov edx,dword ptr ds:\[6D4E93\]   
006E57E4   mov dword ptr ds:\[edx\],ecx   
006E57E6   mov edx,dword ptr ss:\[esp\]                    ; KERNEL32.77E887E7   
006E57E9   pop dword ptr ds:\[6D4E8F\]                     ; KERNEL32.77E887E7   
006E57EF   push dword ptr ds:\[6D4EDB\]   
006E57F5   push ACProtec.006D4EF7   
006E57FA   pop dword ptr ds:\[6D4EC3\]                     ; KERNEL32.77E887E7   
006E5800   push dword ptr ds:\[6D4EC3\]   
006E5806   pop dword ptr ds:\[6D4EBF\]                     ; KERNEL32.77E887E7   
006E580C   push dword ptr ds:\[6D4EBF\]   
006E5812   pop ecx                                       ; KERNEL32.77E887E7   
006E5813   mov ebp,dword ptr ds:\[ecx\] //stolen code 第 2 行,mov ebp,esp 的变形    
006E5815   mov ecx,dword ptr ss:\[esp\]                    ; KERNEL32.77E887E7   
006E5818   pop dword ptr ds:\[6D4ED7\]                     ; KERNEL32.77E887E7   
006E581E   push edx   
006E581F   nop   
006E5820   nop   
006E5821   pushad   
006E5822   call ACProtec.006E1460   
006E5827   mov byte ptr ss:\[ebp+40D1D0\],0   
006E582E   popad   
006E582F   mov edx,ACProtec.006D4EBB   
006E5834   mov dword ptr ds:\[edx\],ebx   
006E5836   pop edx                                       ; KERNEL32.77E887E7   
006E5837   push dword ptr ds:\[6D4EBB\]   
006E583D   pop dword ptr ds:\[6D4EEB\]                     ; KERNEL32.77E887E7   
006E5843   push dword ptr ds:\[6D4EEB\]   
006E5849   mov dword ptr ss:\[esp\],eax   
006E584C   mov dword ptr ss:\[esp\],esi  //stolen code 第 3 行,push esi 的变形    
006E584F   push edx   
006E5850   mov edx,ACProtec.006D4EB7   
006E5855   mov dword ptr ds:\[edx\],ecx   
006E5857   pop edx                                       ; KERNEL32.77E887E7   
006E5858   push dword ptr ds:\[6D4EB7\]   
006E585E   push ecx   
006E585F   mov dword ptr ss:\[esp\],ACProtec.006D4EE7   
006E5866   pop dword ptr ds:\[6D4E8B\]                     ; KERNEL32.77E887E7   
006E586C   mov ecx,dword ptr ds:\[6D4E8B\]   
006E5872   mov dword ptr ds:\[ecx\],eax   
006E5874   mov ecx,dword ptr ss:\[esp\]                    ; KERNEL32.77E887E7   
006E5877   pop dword ptr ds:\[6D4EB3\]                     ; KERNEL32.77E887E7   
006E587D   nop   
006E587E   nop   
006E587F   pushad   
006E5880   call ACProtec.006E369A   
006E5885   popad   
006E5886   push dword ptr ds:\[6D4EE7\]   
006E588C   pop dword ptr ds:\[6D4F07\]                     ; KERNEL32.77E887E7   
006E5892   push dword ptr ds:\[6D4F07\]   
006E5898   push edi                                      ; ACProtec.005D16A8   
006E5899   mov edi,ACProtec.006D4EAF   
006E589E   mov dword ptr ds:\[edi\],esi   
006E58A0   pop edi                                       ; KERNEL32.77E887E7   
006E58A1   push dword ptr ds:\[6D4EAF\]   
006E58A7   mov dword ptr ss:\[esp\],edi                    ; ACProtec.005D16A8   
006E58AA   push ebx   
006E58AB   pop dword ptr ds:\[6D4EAB\]                     ; KERNEL32.77E887E7   
006E58B1   push dword ptr ds:\[6D4EAB\]   
006E58B7   mov dword ptr ds:\[6D4E87\],eax   
006E58BD   push dword ptr ds:\[6D4E87\] //stolen code 第 4 行,push eax 的变形        
006E58C3   mov dword ptr ds:\[6D4E83\],ACProtec.006D4EF3   
006E58CD   mov eax,dword ptr ds:\[6D4E83\]   
006E58D3   push ecx   
006E58D4   mov ecx,eax   
006E58D6   pushad   
006E58D7   call ACProtec.006E3437   
006E58DC   popad   
006E58DD   mov ebx,ecx   
006E58DF   pop ecx                                       ; KERNEL32.77E887E7   
006E58E0   pop dword ptr ds:\[6D4E7F\]                     ; KERNEL32.77E887E7   
006E58E6   mov eax,dword ptr ds:\[6D4E7F\]   
006E58EC   push edi                                      ; ACProtec.005D16A8   
006E58ED   mov edi,ACProtec.006D4EA7   
006E58F2   mov dword ptr ds:\[edi\],ebx   
006E58F4   pop edi                                       ; KERNEL32.77E887E7   
006E58F5   push edx   
006E58F6   mov edx,ACProtec.006D4EA7   
006E58FB   mov edi,dword ptr ds:\[edx\]   
006E58FD   pop edx                                       ; KERNEL32.77E887E7   
006E58FE   mov ebx,dword ptr ss:\[esp\]                    ; KERNEL32.77E887E7   
006E5901   pop dword ptr ds:\[6D4EA3\]                     ; KERNEL32.77E887E7   
006E5907   mov dword ptr ds:\[edi\],esi   
006E5909   pop dword ptr ds:\[6D4ED3\]                     ; KERNEL32.77E887E7   
006E590F   push dword ptr ds:\[6D4ED3\]   
006E5915   pop dword ptr ds:\[6D4E7B\]                     ; KERNEL32.77E887E7   
006E591B   mov edi,dword ptr ds:\[6D4E7B\]   
006E5921   push dword ptr ds:\[6D4EF3\]   
006E5927   mov dword ptr ss:\[esp\],ebx  //stolen code 第 5 行,push ebx 的变形          
006E592A   nop   
006E592B   nop   
006E592C   nop   
006E592D   pushad   
006E592E   call ACProtec.006E56B9   
006E5933   popad   
006E5934   mov dword ptr ds:\[6D4E77\],ebx   
006E593A   push dword ptr ds:\[6D4E77\]   
006E5940   mov dword ptr ss:\[esp\],eax   
006E5943   pop dword ptr ds:\[6D4EE3\]                     ; KERNEL32.77E887E7   
006E5949   push dword ptr ds:\[6D4EE3\]   
006E594F   mov dword ptr ss:\[esp\],edx   
006E5952   mov dword ptr ss:\[esp\],esi  //stolen code 第 6 行,push esi 的变形    
006E5955   push esi   
006E5956   mov dword ptr ss:\[esp\],ebx   
006E5959   mov dword ptr ss:\[esp\],ACProtec.004CBFF8   
006E5960   pop dword ptr ds:\[6D4EDF\]                     ; KERNEL32.77E887E7   
006E5966   push dword ptr ds:\[6D4EDF\]   
006E596C   pop dword ptr ds:\[6D4F03\]                     ; KERNEL32.77E887E7   
006E5972   push dword ptr ds:\[6D4F03\]   
006E5978   mov eax,dword ptr ss:\[esp\] //stolen code 第 7 行,mov eax,4cbff8 的变形    
006E597B   pop dword ptr ds:\[6D4EFF\]                     ; KERNEL32.77E887E7   
006E5981   nop   
006E5982   nop   
以上代码要一步一步按 F7 跟踪，注意执行前后各寄存器值的变化以及堆浅 esp 的变化，以便确认是附值还是 push，从而得出正确的 stolen code,并作好纪录。到 006e5981 后按 F9 继续运行。出现 int3 异常中断：   
006E2CFE   int3   
006E2CFF   nop   
006E2D00   pop dword ptr fs:\[0\]                          ; 0012FFE0   
006E2D06   add esp,4   
006E2D09   pushad   
006E2D0A   call ACProtec.006E2D0F   
006E2D0F   pop esi                                       ; 0012FFE0   
006E2D10   sub esi,6   
006E2D13   mov ecx,5B   
006E2D18   sub esi,ecx   
006E2D1A   mov edx,8C98BB7   
006E2D1F   shr ecx,2   
006E2D22   sub ecx,2   
006E2D25   cmp ecx,0   
006E2D28   jl short ACProtec.006E2D44   
006E2D2A   mov eax,dword ptr ds:\[esi+ecx\*4\]   
006E2D2D   mov ebx,dword ptr ds:\[esi+ecx\*4+4\]   
006E2D31   xor eax,ebx   
006E2D33   ror eax,12   
006E2D36   add eax,edx   
006E2D38   sub edx,7D69989C   
006E2D3E   mov dword ptr ds:\[esi+ecx\*4\],eax   
006E2D41   dec ecx   
006E2D42   jmp short ACProtec.006E2D25   
006E2D44   popad   
006E2D45   popad   
006E2D46   retn //在此处按 F2,然后按 shift-F9，停在此处再按 F2 取消，F7 走 1 步      
此时再次察看 ESP=12FF90,去转存窗口，Ctrl+G:0012FF90  到达 0012FF90 内存处,在 0012FF90 处的 4 个字节上,下硬件访问－>DWord 断点.F9 运行,断下。   
    由于执行以下代码包含许多 call,会影响程序代码和数据的变化，因此在这儿用 LordPE 完全 DUMP 进程。     
    接着前面继续分析 stolen code。以下仍然是 F7 单步跟踪，遇到 call 时再 F8。     
006ED0E3   push esi   
006ED0E4   mov dword ptr ss:\[esp\],ecx   
006ED0E7   push eax                                      ; ACProtec.004CBFF8   
006ED0E8   mov eax,ACProtec.00406EDC   
006ED0ED   mov ecx,eax                                   ; ACProtec.004CBFF8   
006ED0EF   pop eax                                       ; ACProtec.004CBFF8   
006ED0F0   push ecx   
006ED0F1   pop dword ptr ds:\[6D4F0F\]   
006ED0F7   pop dword ptr ds:\[6D4EFF\]                     ; ACProtec.004CBFF8   
006ED0FD   mov ecx,dword ptr ds:\[6D4EFF\]                 ; ACProtec.004CBFF8   
006ED103   call dword ptr ds:\[6D4F0F\]  //stolen code 第 8 行，call 406edc 的变形    
006ED109   mov eax,dword ptr ds:\[4F258C\]  //stolen code 第 9 行,没变形    
006ED10E   mov eax,dword ptr ds:\[eax\]     //stolen code 第 10 行，没变形    
006ED110   mov dword ptr ds:\[6D4EFB\],ACProtec.004622F8   
006ED11A   push dword ptr ds:\[6D4EFB\]   
006ED120   pop dword ptr ds:\[6D4F0B\]   
006ED126   call dword ptr ds:\[6D4F0B\]  //stolen code 第 11 行，call 4622f8 的变形    
006ED12C   mov dword ptr ds:\[6D4EF7\],edi                 ; ACProtec.005D16A8   
006ED132   nop   
006ED133   pushad   
006ED134   call ACProtec.006E1460   
006ED139   mov eax,dword ptr ss:\[ebp+41A27A\]   
006ED13F   add eax,dword ptr ss:\[ebp+40D2AA\]   
006ED145   mov dword ptr ss:\[ebp+41A27A\],eax             ; ACProtec.004CBFF8   
006ED14B   popad   
006ED14C   push dword ptr ds:\[6D4EF7\]   
006ED152   push eax                                      ; ACProtec.004CBFF8   
006ED153   mov eax,ACProtec.004F2354   
006ED158   mov edi,eax                                   ; ACProtec.004CBFF8   
006ED15A   pop eax                                       ; ACProtec.004CBFF8   
006ED15B   mov ecx,dword ptr ds:\[edi\]  //stolen code 第 12 行，mov ecx,\[4f2354\]    
  
的变形   
006ED15D   pop dword ptr ds:\[6D4EF3\]   
006ED163   mov edi,dword ptr ds:\[6D4EF3\]   
006ED169   mov eax,dword ptr ds:\[4F258C\]  //stolen code 第 13 行，没变形    
006ED16E   mov eax,dword ptr ds:\[eax\]     //stolen code 第 14 行，没变形    
006ED170   mov dword ptr ds:\[6D4EEF\],eax                 ; ACProtec.004CBFF8   
006ED176   push dword ptr ds:\[6D4EEF\]                    ; ACProtec.006D4F0F   
006ED17C   mov dword ptr ds:\[6D4EEB\],ACProtec.0047FB5C   
006ED186   mov eax,dword ptr ds:\[6D4EEB\]   
006ED18C   mov edx,dword ptr ds:\[eax\]  //stolen code 第 15 行，mov edx,\[47fb5c\]    
  
的变形   
006ED18E   pop dword ptr ds:\[6D4EE7\]   
006ED194   mov eax,dword ptr ds:\[6D4EE7\]   
006ED19A   push ebx   
006ED19B   nop   
006ED19C   pushad   
006ED19D   call ACProtec.006E1460   
006ED1A2   mov byte ptr ss:\[ebp+41A237\],0E8   
006ED1A9   popad   
006ED1AA   mov dword ptr ss:\[esp\],ACProtec.00462310   
006ED1B1   pop dword ptr ds:\[6D4F07\]   
006ED1B7   call dword ptr ds:\[6D4F07\]  //stolen code 第 16 行,call 462310 的变形    
006ED1BD   push dword ptr ds:\[4F228C\]                    ; ACProtec.004F45E8   
006ED1C3   pop dword ptr ds:\[6D4EE3\]   
006ED1C9   mov ecx,dword ptr ds:\[6D4EE3\]  //stolen code 第 17 行,mov     
  
ecx,\[4f228c\]的变形   
006ED1CF   mov eax,dword ptr ds:\[4F258C\]  //stolen code 第 18 行,没有变形    
006ED1D4   mov eax,dword ptr ds:\[eax\]     //stolen code 第 19 行,没有变形    
006ED1D6   push dword ptr ds:\[47F7CC\]                    ; ACProtec.0047F818   
006ED1DC   pop edx                 //stolen code 第 20 行,mov edx,\[47f7cc\]的变形    
006ED1DD   push ACProtec.00462310   
006ED1E2   pop dword ptr ds:\[6D4F03\]                     ; ACProtec.004CBFF8   
006ED1E8   call dword ptr ds:\[6D4F03\] //stolen code 第 21 行，call 462310 的变形    
006ED1EE   mov eax,dword ptr ds:\[4F258C\] //stolen code 第 22 行,没有变形    
006ED1F3   mov eax,dword ptr ds:\[eax\]    //stolen code 第 23 行,没有的变形    
006ED1F5   nop   
006ED1F6   nop   
006ED1F7   nop   
006ED1F8   nop   
006ED1F9   nop   
006ED1FA   pushad   
006ED1FB   call ACProtec.006E1460   
006ED200   mov dword ptr ss:\[ebp+41A238\],25FF   
006ED20A   lea eax,dword ptr ss:\[ebp+41A27A\]   
006ED210   mov dword ptr ss:\[ebp+41A23A\],eax             ; ACProtec.004CBFF8   
006ED216   call ACProtec.006E1460   
006ED21B   lea edi,dword ptr ss:\[ebp+419F20\]   
006ED221   lea ecx,dword ptr ss:\[ebp+41A22C\]   
006ED227   sub ecx,edi                                   ; ACProtec.005D16A8   
006ED229   shr ecx,2   
006ED22C   call ACProtec.006E11FE   
006ED231   stos dword ptr es:\[edi\]   
006ED232   loopd short ACProtec.006ED22C   
006ED234   popad   
006ED235   jmp short ACProtec.006ED238   
  
006ED238   jmp dword ptr ds:\[6ED27A\]  //飞向光明之巅！JMP 04cc263      
  
2.修复输入表    
为了还原加密了 IAT,随便在程序一个空地(偶是选择 6d4525 处)，写入以下代码并执行：   
006D4D25   pushad   
006D4D26   mov esi,ACProtec.006D4010   
006D4D2B   mov edi,dword ptr ds:\[esi+8\]   
006D4D2E   xor dword ptr ds:\[esi+1\],edi                  ; ACProtec.005D16A8   
006D4D31   add esi,0D   
006D4D34   cmp byte ptr ds:\[esi\],68   
006D4D37   je short ACProtec.006D4D2B   
006D4D39   popad   
运行 ImportREC，选择这个进程。把 OEP 改为 00CC263，点 IT AutoSearch，发现完全不对。于是回到 OLLYDBG,找到能正确显示 API 调用的代码，偶随便找到 4074a8 处，于是在 ImportREC 中把 OEP 修改为 0074a8,点 IT AutoSearch，点“Get Import”，用层次 1 即可修复全部无效函数数。FixDump！     
  
3.修复入口 Stolen code    
根据分析变形后 stolen code 得出程序的入口代码，并在 004cc210 开始处补上：   
push ebp    
mov ebp,esp    
sub esp,10   (变形后为：push esi；push eax；push ebx；push esi）    
mov eax,04cbff8    
call 406edc    
mov eax,\[4f528c\]    
mov eax,\[eax\]    
call 4622f8    
mov ecx,\[4f2354\]    
mov edx,\[47fb5c\]    
mov eax,\[4f258c\]    
mov eax,\[eax\]    
call 462310    
mov ecx,\[4f228c\]    
mov eax,\[4f258c\]    
mov eax,\[eax\]    
mov edx,\[47f7cc\]    
call 462310    
mov eax,\[4f258c\]    
mov eax,\[eax\]   
4.程序还有许多暗桩有待修复，这里只讨论脱壳。 

ACProtect V1.21 专业版主程序的修复   
  
    偶的《ACProtect V1.21 专业版主程序的脱壳》一文，想必大家都已经看过了。  
有人可能会问：为什么有 2 个函数修复不了？而且也没有去修复呢？这里我将要告诉大家为什么。   
  1.消除代码替换暗桩(即 ACPr 中的 Code Replace)   
  用 W32Dasm 打开 dump 出来的文件反汇编，然后搜索 call 006(壳的代码位于 006d4000 以上) 就会发现许多代码暗桩都是 call 006d540c,为此经过分析该段代码后修改如下：   
006D540C  push ebp   
006D540D  mov ebp,esp   
006D540F  pushad   
006D5410  mov esi,006D56E2   
006D5415  mov ebx,dword ptr ss:\[ebp+4\]   
006D5418  xor ecx,ecx   
006D541A  lods dword ptr ds:\[esi\]   
006D541B  add eax,00400000   
006D5420  cmp eax,ebx   
006D5422  nop   
006D5423  nop   
006D5424  je short Acp.006D542C   
006D5426  inc ecx   
006D5427  jmp 006D541A   
006D542C  mov edi,006D85C2   
006D5431  mov eax,0A   
006D5436  mul ecx   
006D5438  add edi,eax   
006D543A  mov ecx,0A   
006D543F  mov al,byte ptr ds:\[edi\]   
006D5441  xor al,6D   
006D5443  stos byte ptr es:\[edi\]   
006D5444  loopd short 006D543F   
006D5446  sub edi,0A   
006D5449  mov dword ptr ds:\[06d4000\],edi    
006D544F  sub edi,ebx   
006D5451  mov dword ptr ds:\[ebx\-4\],edi   
006D5454  popad   
006D5455  pop ebp   
006D5456  jmp dword ptr ds:\[06d4000\]   
偶本想写段代码直接把原来的代码还原到 CALL 的位置，因为长度都是 5 字节，但由于要用反汇编识别垃圾代码比较麻烦所以作摆。   
2.跨平台问题   
经过第一步修复,虽然在你 dump 时的操作系统下可以运行，但不能跨平台。比如偶的在 win2k 下运行没问题，但是到 win98 出错。于是再次进行跟踪分析，原来存在数据暗桩即共享壳中数据。原理是：在壳中对部分 API 函数取地址并在前面加标示识别，然后在主程序中通过搜索标示找到该部分函数的地址再进行调用。由于脱壳后，API 地址仍然是原来的没有随着系统的变化而变化所以出错。该段取 API 地址代码位于壳 06e369a 处，现修改如下:   
006E369A  call 006E1460   
006E369F  lea eax,dword ptr ss:\[ebp+40D5E7\]   
006E36A5  mov ebx,eax   
006E36A7  push eax   
006E36A8  push eax   
006E36A9  jmp 006E36C0   
006E36AB  nop   
006E36AC  nop   
006E36AD  nop   
006E36AE  nop   
006E36AF  nop   
006E36B0  nop   
006E36B1  nop   
006E36B2  nop   
006E36B3  nop   
006E36B4  nop   
006E36B5  nop   
006E36B6  nop   
006E36B7  nop   
006E36B8  nop   
006E36B9  nop   
006E36BA  nop   
006E36BB  nop   
006E36BC  nop   
006E36BD  nop   
006E36BE  nop   
006E36BF  nop   
006E36C0  pop eax                                          ;    
006E36C1  call dword ptr ds:\[691194\]    
006E36C7  jmp 006E36E0   
006E36C9  nop   
006E36CA  nop   
006E36CB  nop   
......   
006E36E0  or eax,eax   
006E36E2  jnz 006E3721   
006E36E4  retn   
006E36E5  nop   
006E36E6  nop   
006E36E7  nop   
......   
006E3721  mov ebx,eax   
006E3723  mov dword ptr ss:\[ebp+4023F0\],eax   
006E3729  mov eax,0040D5F4   
006E372E  mov edx,0040D763   
006E3733  call 006E148A   
006E3738  mov eax,0040D600   
006E373D  mov edx,0040D767   
006E3742  call 006E148A   
006E3747  mov eax,0040D60B   
006E374C  mov edx,0040D77B   
006E3751  call 006E148A   
006E3756  mov eax,0040D61F   
006E375B  mov edx,0040D77F   
006E3760  call 006E148A   
006E3765  mov eax,0040D638   
006E376A  mov edx,0040D783   
006E376F  call 006E148A   
006E3774  mov eax,0040D647   
006E3779  mov edx,0040D787   
006E377E  call 006E148A   
006E3783  mov eax,0040D655   
006E3788  mov edx,0040D78B   
006E378D  call 006E148A   
006E3792  mov eax,0040D661   
006E3797  mov edx,0040D78F   
006E379C  call 006E148A   
006E37A1  mov eax,Acp.0040D66D   
006E37A6  mov edx,Acp.0040D793   
006E37AB  call Acp.006E148A   
006E37B0  mov eax,Acp.0040D67E   
006E37B5  mov edx,Acp.0040D7AB   
006E37BA  call Acp.006E148A   
006E37BF  mov eax,Acp.0040D690   
006E37C4  mov edx,Acp.0040D7AF   
006E37C9  call Acp.006E148A   
006E37CE  mov eax,Acp.0040D69C   
006E37D3  mov edx,Acp.0040D7B3   
006E37D8  call Acp.006E148A   
006E37DD  mov eax,Acp.0040D6A5   
006E37E2  mov edx,Acp.0040D7B7   
006E37E7  call Acp.006E148A   
006E37EC  mov eax,Acp.0040D6AF   
006E37F1  mov edx,Acp.0040D7BB   
006E37F6  call Acp.006E148A   
006E37FB  mov eax,Acp.0040D6BB   
006E3800  mov edx,Acp.0040D7BF   
006E3805  call Acp.006E148A   
006E380A  mov eax,Acp.0040D6C8   
006E380F  mov edx,Acp.0040D7C3   
006E3814  call Acp.006E148A   
006E3819  mov eax,Acp.0040D6E1   
006E381E  mov edx,Acp.0040D7CB   
006E3823  call Acp.006E148A   
006E3828  mov eax,Acp.0040D6F2   
006E382D  mov edx,Acp.0040D7CF   
006E3832  call Acp.006E148A   
006E3837  mov eax,Acp.0040D703   
006E383C  mov edx,Acp.0040D7D3   
006E3841  call Acp.006E148A   
006E3846  mov eax,Acp.0040D803   
006E384B  mov edx,Acp.0040D7FF   
006E3850  call Acp.006E148A   
006E3855  lea eax,dword ptr ss:\[ebp+40D714\]   
006E385B  mov ebx,eax   
006E385D  push eax   
006E385E  push eax   
006E385F  nop   
006E3860  nop   
006E3861  nop   
006E3862  nop   
006E3863  nop   
006E3864  nop   
006E3865  nop   
006E3866  nop   
006E3867  nop   
006E3868  nop   
006E3869  nop   
006E386A  nop   
006E386B  nop   
006E386C  nop   
006E386D  nop   
006E386E  nop   
006E386F  nop   
006E3870  nop   
006E3871  nop   
006E3872  nop   
006E3873  nop   
006E3874  nop   
006E3875  nop   
006E3876  pop eax                                             
006E3877  call dword ptr ds:\[691194\]   
006E387D  jmp short Acp.006E3896   
006E387F  nop   
006E3880  nop   
006E3881  nop   
......   
006E3896  or eax,eax   
006E3898  jnz short Acp.006E38D7   
006E389A  retn   
006E389B  nop   
006E389C  nop   
006E389D  nop   
......   
006E38D7  mov ebx,eax   
006E38D9  mov dword ptr ss:\[ebp+4023F4\],eax   
006E38DF  mov eax,Acp.0040D71F   
006E38E4  mov edx,Acp.0040D76B   
006E38E9  call Acp.006E148A   
006E38EE  mov eax,Acp.0040D72B   
006E38F3  mov edx,Acp.0040D76F   
006E38F8  call Acp.006E148A   
006E38FD  mov eax,Acp.0040D73A   
006E3902  mov edx,Acp.0040D773   
006E3907  call Acp.006E148A   
006E390C  mov eax,Acp.0040D748   
006E3911  mov edx,Acp.0040D777   
006E3916  call Acp.006E148A   
006E391B  retn   
  
006E148A  push ebx   
006E148B  push eax   
006E148C  push edx   
006E148D  add eax,ebp   
006E148F  push eax   
006E1490  push ebx   
006E1491  push eax   
006E1492  mov eax,dword ptr ss:\[ebp+41A614\]   
006E1498  nop   
006E1499  nop   
006E149A  nop   
006E149B  nop   
006E149C  nop   
006E149D  nop   
006E149E  nop   
006E149F  nop   
006E14A0  nop   
006E14A1  nop   
006E14A2  nop   
006E14A3  nop   
006E14A4  nop   
006E14A5  nop   
006E14A6  nop   
006E14A7  nop   
006E14A8  nop   
006E14A9  pop eax                                          ;    
006E14AA  call dword ptr ds:\[691190\]   
006E14B0  jmp short Acp.006E14C9   
006E14B2  nop   
006E14B3  nop   
006E14B4  nop   
......   
006E14C9  pop edx                                             
006E14CA  mov dword ptr ss:\[ebp+edx\],eax   
006E14CE  pop eax                                          ;    
006E14CF  pop ebx                                          ;   
006E14D0  retn   
3.调用壳 SDK 的修复   
在 IAT 修复时有 2 个函数没有修复，这 2 个函数都是指向 MessageBoxA，看 ACPr 说明原来用作 SDK 接口。   
程序对该函数调用相应也是 2 处：00407830 和 00407838.都指向壳代码 006e084d.如果不修复直接指向原来的 USer32.dll 那么主程序总是显示未注册版而且对程序加壳后也显示未注册，很不爽。偶在 6d4010 处写入不分代码，然后修改 407830 和 407838 代码跳转到 006d4010：   
00407830:jmp 006d4010   
00407838:jmp 006d4010   
  
006D4010  cmp dword ptr ss:\[esp+4\],-1   
006D4015  jnz 006D4023   
006D4017  cmp dword ptr ss:\[esp+8\],0   
006D401C  je 006D4023   
006D401E  jmp 006E084D   
006D4023  jmp dword ptr ds:\[691588\]         ;JMP 到 user32.MessageBoxA   
这样处理后只能算基本可以了。但还是有点问题，由于要浪费很多时间就不进一步处理了。   
4.部分数据校验：   
主程序的入口必须是 2D4000   
IMAGESIZE 必须是：2F537A   
因为数据校验代码部分是动态还原后执行的，不便修改该部分代码。   
入口代码：   
006D4000:PUSHAD   
CALL 006E369A   
POPAD   
JMP 004CC210  4dd466d0e1e0cc123afbe504da2462ca