

# 直面 MSLRH

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-502.htm`](https://www.pediy.com/kssd/pediy07/pediy7-502.htm)

\[**Ready:**:  
  首先隐藏 od, 在 GetCommandLineA 和 ZwQueryInformationProcess 的入口以及 GetProcAddress 的出口处（RETN 8）下断，  
并忽略所有异常。  
  鉴于 MSLRH 的主要手段是 rdtsc, 写一小段对它 clear/restore 代码如下：  
00468800   E8 00000000      CALL \[MSLRH\].00468805                    ;clear rdtsc 代码段入口  
00468805   5E               POP ESI                                    
00468806   83EE 10          SUB ESI,10                               ;定位保存下面申请的 memory 指针的地址  
00468809   AD               LODS DWORD PTR DS:\[ESI\]  
0046880A   85C0             TEST EAX,EAX  
0046880C   75 13            JNZ SHORT \[MSLRH\].00468821  
0046880E   6A 04            PUSH 4  
00468810   68 00100000      PUSH 1000  
00468815   68 00400000      PUSH 4000  
0046881A   6A 00            PUSH 0  
0046881C   E8 E90FA177      CALL kernel32.VirtualAlloc               ;申请 Memory 以保存修改了 rdtsc 的地址  
00468821   8946 FC          MOV DWORD PTR DS:\[ESI\-4\],EAX  
00468824   BF 00604500      MOV EDI,\[MSLRH\].<ModuleEntryPoint>  
00468829   B9 00200100      MOV ECX,12000  
0046882E   B0 0F            MOV AL,0F  
00468830   F2:AE            REPNE SCAS BYTE PTR ES:\[EDI\]             ;查找  
00468832   75 1B            JNZ SHORT \[MSLRH\].0046884F  
00468834   803F 31          CMP BYTE PTR DS:\[EDI\],31  
00468837  ⁷⁵ F7            JNZ SHORT \[MSLRH\].00468830  
00468839   8B16             MOV EDX,DWORD PTR DS:\[ESI\]               ;找到 rdtsc  
0046883B   8B5E FC          MOV EBX,DWORD PTR DS:\[ESI\-4\]  
0046883E   8D1C93           LEA EBX,DWORD PTR DS:\[EBX+EDX\*4\]  
00468841   FF06             INC DWORD PTR DS:\[ESI\]                   ;计数  
00468843   8D57 FF          LEA EDX,DWORD PTR DS:\[EDI\-1\]  
00468846   8913             MOV DWORD PTR DS:\[EBX\],EDX               ;保存地址  
00468848   66:C702 9090     MOV WORD PTR DS:\[EDX\],9090               ;patch  
0046884D  ^EB E1            JMP SHORT \[MSLRH\].00468830  
0046884F   C3               RETN  
;===============================================================================================  
00468850   E8 00000000      CALL \[MSLRH\].00468855                    ;Restore rdtsc 代码段入口  
00468855   5E               POP ESI                                    
00468856   83EE 60          SUB ESI,60                                 
00468859   AD               LODS DWORD PTR DS:\[ESI\]                  ;得到上面的 memory 指针的地址  
0046885A   8BC8             MOV ECX,EAX  
0046885C   AD               LODS DWORD PTR DS:\[ESI\]                  ;得到修改计数  
0046885D   91               XCHG EAX,ECX  
0046885E   8BF0             MOV ESI,EAX  
00468860   AD               LODS DWORD PTR DS:\[ESI\]                  ;得到修改过的地址  
00468861   66:C700 0F31     MOV WORD PTR DS:\[EAX\],310F               ;恢复  
00468866  ^E2 F8            LOOPD SHORT \[MSLRH\].00468860  
00468868   C3               RETN  
;===============================================================================================  
00468869   60               PUSHAD                                     
0046886A   E8 91FFFFFF      CALL \[MSLRH\].00468800                    ;call to Clear rdtsc  
0046886F   61               POPAD  
00468870  -E9 8BD7FEFF      JMP \[MSLRH\].<ModuleEntryPoint>           ;每次还要修改 jmp 地址（两次）  
  
00468877   60               PUSHAD   
00468878   E8 D3FFFFFF      CALL \[MSLRH\].00468850                    ;call to Restore rdtsc  
0046887D   61               POPAD  
0046887E  -E9 A0D8FEFF      JMP \[MSLRH\].00456123                     ;用时还要修改 jmp 地址（一次）  
;================================================================================================  
  
**Begin:**:  
光标定位于上面 00468869 Ctrl+ Gray \* ，F9 运行，停在 GetProcAddress 的出口点，去掉此处断点，F7 回到下面  
  
0045A5DE   8946 14          MOV DWORD PTR DS:\[ESI+14\],EAX            ; kernel32.OutputDebugStringA  
  
Ctrl + B 搜索 0F31，看到以下代码，在 0045A81B 下断，  
0045A80E   0F31             RDTSC                          
0045A810   2B0424           SUB EAX,DWORD PTR SS:\[ESP\]  
0045A813   83C4 04          ADD ESP,4  
0045A816   3D FFFFFF00      CMP EAX,0FFFFFF  
0045A81B  ⁰F87 D0B8FFFF    JA \[MSLRH\].004560F1                       ;这里下断，中断后转到下一指令（push esi）  
0045A821   56               PUSH ESI                                  ;这里执行  
  
F9 运行到上面断点处，光标移到下一条语句 PUSH ESI 上 Ctrl+ Gray \*，搜索 PUSH DWORD PTR FS:\[30\]，找到后在它下面的  
ADD DWORD PTR DS:\[EBX\],0C8 上下断，如下  
  
0045B1D0   75 28            JNZ SHORT \[MSLRH\].0045B1FA  
0045B1D2   64:FF35 30000000 PUSH DWORD PTR FS:\[30\]  
0045B1D9   58               POP EAX     
0045B1DA   0FB648 02        MOVZX ECX,BYTE PTR DS:\[EAX+2\]  
0045B1DE   884E 0C          MOV BYTE PTR DS:\[ESI+C\],CL  
0045B1E1   8B40 0C          MOV EAX,DWORD PTR DS:\[EAX+C\]  
0045B1E4   8B40 0C          MOV EAX,DWORD PTR DS:\[EAX+C\]  
0045B1E7   8D58 20          LEA EBX,DWORD PTR DS:\[EAX+20\]  
0045B1EA   8D48 18          LEA ECX,DWORD PTR DS:\[EAX+18\]  
0045B1ED   8103 C8000000    ADD DWORD PTR DS:\[EBX\],0C8               ;修改头部，此处跳过  
0045B1F3   B8 00000000      MOV EAX,0  
0045B1F8   0101             ADD DWORD PTR DS:\[ECX\],EAX  
0045B1FA   33C9             XOR ECX,ECX  
0045B1FC   E8 00000000      CALL \[MSLRH\].0045B201  
0045B201   5F               POP EDI                                    
0045B202   81C7 C1090000    ADD EDI,9C1                                 
0045B208   0FB60439         MOVZX EAX,BYTE PTR DS:\[ECX+EDI\]          /   
0045B20C   83F0 11          XOR EAX,11                               |动  
0045B20F   880439           MOV BYTE PTR DS:\[ECX+EDI\],AL             |态  
0045B212   41               INC ECX                                  |解  
0045B213   81F9 521D0000    CMP ECX,1D52                             |码  
0045B219  ⁷² ED            JB SHORT \[MSLRH\].0045B208                \\  
0045B21B   EB 05            JMP SHORT \[MSLRH\].0045B222               ;这里下断  
  
  
  中断后清除断点并跳过 0045B1ED 语句。从 0045B1FC---0045B219F9 是解码，在 0045B21B 下断，F9 停下后再次使用我们  
的 Clear rdtsc 代码（注意修改返回地址改成 0045B21B） go！程序中断在 OutputDebugStringA 入口！当然不要执行这  
个函数，跳到出口 F9，又在 GetCommandLineA 入口中断，单步执行返回到程序空间，看到：  
0045EDD7   40               INC EAX  
0045EDD8   33C9             XOR ECX,ECX  
0045EDDA   41               INC ECX  
0045EDDB   803C01 00        CMP BYTE PTR DS:\[ECX+EAX\],0  
0045EDDF   74 0C            JE SHORT \[MSLRH\].0045EDED  
0045EDE1   803C01 22        CMP BYTE PTR DS:\[ECX+EAX\],22  
0045EDE5  ⁷⁵ F3            JNZ SHORT \[MSLRH\].0045EDDA  
0045EDE7   C60401 00        MOV BYTE PTR DS:\[ECX+EAX\],0  
0045EDEB  ^EB ED            JMP SHORT \[MSLRH\].0045EDDA  
0045EDED   6A 00            PUSH 0  
0045EDEF   6A 00            PUSH 0  
0045EDF1   6A 03            PUSH 3  
0045EDF3   6A 00            PUSH 0  
0045EDF5   6A 00            PUSH 0                                   ;注意这里 ShareMode = 0  
0045EDF7   68 00000080      PUSH 80000000  
0045EDFC   50               PUSH EAX  
0045EDFD   FF56 1C          CALL DWORD PTR DS:\[ESI+1C\]               ; kernel32.CreateFileA  
0045EE00   EB 05            JMP SHORT \[MSLRH\].0045EE07  
  
  程序这里打开文件是为了 anti ImportREC，转到 0045EE00，F9，中断在 ZwQueryInformationProcess 入口，F8 逐步返  
回到程序中，  
0045F7CD   58               POP EAX  
0045F7CE   8846 0E          MOV BYTE PTR DS:\[ESI+E\],AL               ;跳过此句  
0045F7D1   EB 05            JMP SHORT \[MSLRH\].0045F7D8  
  
到 0045F7CE 后跳过。  
  
下面我们要面对 CRC ，应该恢复替换成 nop nop 的 rdtsc 了，作如下操作：  
1)找到这段代码的结束处，Ctrl + F  输入 or ebp,-1，找到后在其下面的 retn 上下断  
0046207B   83CD FF          OR EBP,FFFFFFFF  
0046207E   68 ADE29F00      PUSH 9FE2AD  
00462083   C3               RETN                                     ;这里下断  
  
2)跳到 00468877 执行恢复代码，返回地址 00456123 改成 0045F7D1  
  
Ok，F9，中断发生啦，看到：  
0046207E   68 ADE29F00      PUSH 004528C0  
00462083   C3               RETN                                     ;清除断点单步  
  
F7，看到 unpack 函数啦！在它的出口（jmp 00413D56）下断，中断后 F7 就“飞向光明之巅”。  
  
  
  
  
  
  
附 1：\[clear/restore 代码\]，有兴趣就把它粘到 00468800 试试，注意我这里 OS 是 xp sp1，如果是其他系统应该把  
      0046881C 处的 call VirtualAlloc 语句修正一下  
E8 00 00 00 00 5E 83 EE 10 AD 85 C0 75 13 6A 04 68 00 10 00 00 68 00 40 00 00 6A 00 E8 E9 0F A1  
77 89 46 FC BF 00 60 45 00 B9 00 20 01 00 B0 0F F2 AE 75 1B 80 3F 31 75 F7 8B 16 8B 5E FC 8D 1C  
93 FF 06 8D 57 FF 89 13 66 C7 02 90 90 EB E1 C3 E8 00 00 00 00 5E 83 EE 60 AD 8B C8 AD 91 8B F0  
AD 66 C7 00 0F 31 E2 F8 C3 60 E8 91 FF FF FF 61 E9 8B D7 FE FF 00 00 60 E8 D3 FF FF FF 61 E9 A0  
D8 FE FF  
  
附 2：有朋友提到 rdtsc 的问题，其实我也不太清楚，编了个小的演示程序，你可以：  
      1.直接执行  
      2.用 OD 加载 F9  
      3.OD 加载后单步  
      看看有啥区别  
  
附件:aaa.rar 

**PEiD Sign——MSLRH V0.3X**  
  
  
\[MSLRH V0.3X -> emadicius\]   
signature = 60 D1 CB 0F CA C1 CA E0 D1 CA 0F C8 EB 01 F1  
ep\_only = true 

 4dd466d0e1e0cc123afbe504da2462ca