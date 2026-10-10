

# EmbedPE 1.13 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6927.htm`](https://www.pediy.com/kssd/pediy06/pediy6927.htm)

【脱文作者】 simonzh2000  
  
【使用工具】 Ollydbg1.10, LordPE  
  
【破解平台】 Win2000SP4 English  
  
【软件名称】 EmbedPE 1.13 主程序  

【实例下载】 点击此处下载  
  
【软件简介】 Cyclotron 兄弟的壳, 花指令变态的多, AntiTrace 手段也很多, 花了几天工夫, 仔细分析了一下, 去掉了所有的花指令.  
  
1. SetUnhandleExceptionFilter  
2. DRx 解码(只用了 DR3)  
3. SEH 死循环  
4. 花指令  
   
好象对 VC 程序, 不能使用高级选项.   
另外, 如果被加壳程序本身处理 INT3 异常的话, 可能会出错.  
  
【加壳方式】 自己  
  
【作者声明】 本笔记只用于学习交流, 初学 Crack，只是感兴趣技术，没有其他目的, 如有不妥之处, 请 Cyclotron 兄弟谅解.  
  
压缩包内有全文, 脱壳后程序.

进入 OD 后, 停在这. 不忽略异常.  
  
这个壳有很多花指令.  
  
  
00422000 >  83EC 50         SUB ESP,50  
00422003    60              PUSHAD  
00422004    68 5DB9525A     PUSH 5A52B95D  
00422009    E8 2F990000     CALL EmbedPE.0042B93D                   ; F7  
0042200E  
  
0042B93D   /EB 27           JMP SHORT EmbedPE.0042B966              ; 下面开始花指令折磨,            
  
\=========================================================================================================================  
  
0042B944    832C24 04       SUB DWORD PTR SS:\[ESP\],4                ; \[ESP\]=42200E-4=42200A, 一大段程序就这句有用, 晕啊  
0042B948    EB 4A           JMP SHORT EmbedPE.0042B994  
  
0042B951    59              POP ECX                                 ; ECX = 101  
0042B952  ^ 79 F0           JNS SHORT EmbedPE.0042B944  
0042B954    EB 01           JMP SHORT EmbedPE.0042B957  
0042B956    DF  
0042B957  ^\\78 EB           JS SHORT EmbedPE.0042B944               ; 都是跳到 0042B944,  
  
0042B95A    59              POP ECX                                 ; ECX = 0042B96C  
0042B95B    9C              PUSHFD  
0042B95C    81C1 E5FFFFFF   ADD ECX,-1B                               
0042B962    9D              POPFD  
0042B963    FFE1            JMP ECX                                 ; ECX = 0042B951  
  
0042B966    51              PUSH ECX                                ; ECX\=101h  
0042B967    E8 EEFFFFFF     CALL EmbedPE.0042B95A                   ; 返回地址 0042B96C  
0042B96C  
    
\===========================================================================================================================  
     
0042B973    8B3C24          MOV EDI,DWORD PTR SS:\[ESP\]               ; EmbedPE.0042200A  
0042B976   /EB 4A           JMP SHORT EmbedPE.0042B9C2  
  
  
0042B97F    59              POP ECX                                  ; ECX = 101  
0042B980    7A F1           JPE SHORT EmbedPE.0042B973  
0042B982    EB 01           JMP SHORT EmbedPE.0042B985  
0042B984  
0042B985   \\7B EC           JPO SHORT EmbedPE.0042B973              ; 都是跳到 0042B973,   
  
0042B988    59              POP ECX                                  ; ECX = 0042B99A  
0042B989    9C              PUSHFD  
0042B98A    81C1 E5FFFFFF   ADD ECX,-1B  
0042B990    9D              POPFD  
0042B991    FFE1            JMP ECX                                  ; ECX = 0042B97F  
  
0042B994    51              PUSH ECX  
0042B995    E8 EEFFFFFF     CALL EmbedPE.0042B988  
0042B99A  
  
\============================================================================================================================  
  
  
上面是二段花指令, 很有规律, 把每一段的关键一句找出来, 就知道程序在干什么了, 下面只记录关键语句.  
  
  
  
  
  
(一) 解码壳的 Loader.  
  
0042B944    832C24 04       SUB DWORD PTR SS:\[ESP\],4                ; \[12FF4C\]=42200E-4=42200A  
0042B973    8B3C24          MOV EDI,DWORD PTR SS:\[ESP\]                
0042B9A1    83C7 04         ADD EDI,4                               ; EDI = 0042200E  
0042B9CF    B9 2F990000     MOV ECX,992F                              
0042B9FD    C1E9 02         SHR ECX,2                                 
0042BA2B    49              DEC ECX                                 ; ECX = 264A  
0042BA55    8B4424 04       MOV EAX,DWORD PTR SS:\[ESP+4\]            ; EAX = 5A52B95D  
  
0042BA82    C00F 8D         ROR BYTE PTR DS:\[EDI\],8D  
0042BAB0    3107            XOR DWORD PTR DS:\[EDI\],EAX  
0042BADB    66:C107 37      ROL WORD PTR DS:\[EDI\],37  
0042BB0A    66:314F 02      XOR WORD PTR DS:\[EDI+2\],CX  
0042BB37    83C7 04         ADD EDI,4  
0042BB65    49              DEC ECX  
0042BB91  ^\\0F85 0CFFFFFF   JNZ EmbedPE.0042BAA3                   ; 实际上到 42BA82  
                                                                   ; ECX\=0, Loader 解码结束  
  
  
过了 42BB91, 以下程序都可以下普通断点,  API 可以下硬件断点, 这样可以省很多工夫  
  
  
  
(二) SEH 死循环和 LOCK CMPXCHG8B 非法指令阻止跟踪  
  
0042BB97    EB 47           JMP SHORT EmbedPE.0042BBE0             ; 花指令又开始了, 晕啊  
  
              
0042BBC0    33C0            XOR EAX,EAX  
0042BBED    8B3C24          MOV EDI,DWORD PTR SS:\[ESP\]               ; EmbedPE.0042200A  
0042BC19    C607 E8         MOV BYTE PTR DS:\[EDI\],0E8  
0042BC47    8947 01         MOV DWORD PTR DS:\[EDI+1\],EAX             ; EAX = 0  
0042BC75    C2 0400         RETN 4                                   ; \[ESP\]=42200A  
  
0042200A    E8 00000000     CALL EmbedPE.0042200F  
0042200F    5D              POP EBP                                  ; EmbedPE.0042200F  
00422010    EB 2B           JMP SHORT EmbedPE.0042203D  
  
00422019    81ED 0F104000   SUB EBP,EmbedPE.0040100F                 ; EBP = 21000  
00422048    B8 00104000     MOV EAX,EmbedPE.00401000  
00422078    03C5            ADD EAX,EBP                               
004220A5    2B85 2E9E4000   SUB EAX,DWORD PTR SS:\[EBP+409E2E\]        ; EAX = 400000  
004220D6    8985 3A9E4000   MOV DWORD PTR SS:\[EBP+409E3A\],EAX        ; EmbedPE.00400000  
  
  
00422105    33C0            XOR EAX,EAX                              ; EmbedPE.00400000  
00422130    BA A48E4000     MOV EDX,EmbedPE.00408EA4  
00422160    03D5            ADD EDX,EBP  
0042218D    52              PUSH EDX                                 ; EmbedPE.00429EA4  
004221B9    64:FF30         PUSH DWORD PTR FS:\[EAX\]  
004221E7    64:8920         MOV DWORD PTR FS:\[EAX\],ESP               ; SEH 建立  
  
  
                                                                     ; 下面开始搞花招  
00422215    64:FF30         PUSH DWORD PTR FS:\[EAX\]  
00422241    BA 758B4000     MOV EDX,EmbedPE.00408B75  
00422271    03D5            ADD EDX,EBP  
0042229C    33C0            XOR EAX,EAX  
004222C7    83EC 08         SUB ESP,8  
004222F5    52              PUSH EDX                                 ; EmbedPE.00429B75  
00422321    64:FF30         PUSH DWORD PTR FS:\[EAX\]  
0042234F    64:8920         MOV DWORD PTR FS:\[EAX\],ESP  
0042237B    83C4 10         ADD ESP,10  
004223A9    52              PUSH EDX  
004223D5    64:FF30         PUSH DWORD PTR FS:\[EAX\]  
00422403    64:8920         MOV DWORD PTR FS:\[EAX\],ESP  
00422431    52              PUSH EDX                                 ; EmbedPE.00429B75  
0042245D    64:FF30         PUSH DWORD PTR FS:\[EAX\]  
0042248B    64:8920         MOV DWORD PTR FS:\[EAX\],ESP  
  
  
  
到这里, 看一下 Stack, SEH 死循环, OD 处理会有问题, 改变 \[12FF40\] 为 12FF4C  
  
0012FF38  /0012FF40  Pointer to next SEH record  
0012FF3C  |00429B75  SE handler  
0012FF40  \\0012FF38  Pointer to next SEH record  
0012FF44   00429B75  SE handler  
0012FF48   0012FF4C  
0012FF4C   0012FFE0  
0012FF50   00429EA4  EmbedPE.00429EA4  
  
  
  
继续花指令,   
  
  
004224B9    F0:0FC7C8       LOCK CMPXCHG8B EAX                       ; 非法指令异常, 429B75 下断, Shift+F9  
  
  
// SEH   
00429B75    55              PUSH EBP                                 ; 断下后, 取消断点  
00429B76    8BEC            MOV EBP,ESP  
00429B78    56              PUSH ESI  
00429B79    57              PUSH EDI  
00429B81    8B75 08         MOV ESI,DWORD PTR SS:\[EBP+8\]             ; pEXCEPTION\_RECORD  
00429BAF    813E 1E0000C0   CMP DWORD PTR DS:\[ESI\],C000001E          ; 异常码  
00429BE0   /0F85 4E020000   JNZ EmbedPE.00429E34  
00429C0F    8B75 10         MOV ESI,DWORD PTR SS:\[EBP+10\]            ; pContext  
00429C3B    8B86 C4000000   MOV EAX,DWORD PTR DS:\[ESI+C4\]            ; regESP=12FF38  
00429C6A    83C0 4C         ADD EAX,4C  
00429C96    8946 10         MOV DWORD PTR DS:\[ESI+10\],EAX            ; regDr3=12FF38+4C=12FF84, 这个很关键, 以后要用  
00429CC4    C746 04 0000000>MOV DWORD PTR DS:\[ESI+4\],0               ; regDr0  
00429CF6    C746 08 0000000>MOV DWORD PTR DS:\[ESI+8\],0               ; regDr1  
00429D26    C746 0C 0000000>MOV DWORD PTR DS:\[ESI+C\],0               ; regDr2  
00429D58    8BBE B8000000   MOV EDI,DWORD PTR DS:\[ESI+B8\]            ; regEIP=4224B9  
00429D87    8137 609F5758   XOR DWORD PTR DS:\[EDI\],58579F60          ; 把 4224B9 变成 90909090  
00429DB6    B8 00000000     MOV EAX,0                                ; ExceptionContinueExecution,已经复 CONTEXT,可从异常发生处继续执行  
00429DE6   /EB 7D           JMP SHORT EmbedPE.00429E65  
00429E41    5F              POP EDI                                  ; EmbedPE.0042200A  
00429E42    5E              POP ESI  
00429E43    C9              LEAVE  
00429E44    C2 1000         RETN 10  
  
  
异常处理结束, 4224B9 下断, F9  
  
004224B9    90              NOP  
004224BA    90              NOP  
004224BB    90              NOP  
004224BC    90              NOP  
004224BD    EB 4A           JMP SHORT EmbedPE.00422509  
  
  
004224E8    83C4 10         ADD ESP,10  
00422516    64:8F05 0000000>POP DWORD PTR FS:\[0\]                     ; 0012FF4C, 恢复 SEH 链  
  
  
  
  
  
  
(三) 关键的第二个异常处理程序 429EA4  
   
这段异常处理程序以后会经常用到, 很重要  
  
00429EA4    55              PUSH EBP                                 ; SEH handler  
00429EA5    8BEC            MOV EBP,ESP  
00429EA7    53              PUSH EBX  
00429EA8    56              PUSH ESI  
00429EA9    57              PUSH EDI  
  
  
00429EB1    8B75 08         MOV ESI,DWORD PTR SS:\[EBP+8\]             ; pEXCEPTION\_RECORD            
00429EDF    813E 1E0000C0   CMP DWORD PTR DS:\[ESI\],C000001E          ; 异常码  
00429F10   /0F85 4E020000   JNZ EmbedPE.00429E34  
  
00429F41    8B75 10         MOV ESI,DWORD PTR SS:\[EBP+10\]            ; pContext  
00429F6D    8B86 B0000000   MOV EAX,DWORD PTR DS:\[ESI+B0\]            ; regEAX  
00429F9E    A9 00000080     TEST EAX,80000000  
00429FCE   /0F85 2E060000   JNZ EmbedPE.0042A602                     ; 实际上到 42A5DE  
  
  
             // EAX = 0000000X, 计算 EBX 处函数地址并保存到 Stack  
  
00429FFD    8B9E B4000000   MOV EBX,DWORD PTR DS:\[ESI+B4\]            ; regEBP  
  
0042A02E    FFB6 A4000000   PUSH DWORD PTR DS:\[ESI+A4\]               ; regEBX  
0042A05F    FFB3 429E4000   PUSH DWORD PTR DS:\[EBX+409E42\]           ; KERNEL32.7C570000  
0042A090    B8 26944000     MOV EAX,EmbedPE.00409426  
0042A0BE    03C3            ADD EAX,EBX                              ; EAX = 42A426  
0042A0EB    50              PUSH EAX                                 ; EmbedPE.0042A426, GetProcAddress 的返回地址  
  
0042A117    55              PUSH EBP                                 ; GetProcAddress 的 第一句在这里执行  
0042A143    8B83 E8AC4000   MOV EAX,DWORD PTR DS:\[EBX+40ACE8\]        ; KERNEL32.GetProcAddress  
0042A174    83C0 01         ADD EAX,1  
0042A1A2    8038 CC         CMP BYTE PTR DS:\[EAX\],0CC                ; 前 7 字节都不能下断点  
0042A1D0   /0F84 50020000   JE EmbedPE.0042A426  
0042A201    8078 01 CC      CMP BYTE PTR DS:\[EAX+1\],0CC  
0042A230   /0F84 F0010000   JE EmbedPE.0042A426  
0042A25F    8078 02 CC      CMP BYTE PTR DS:\[EAX+2\],0CC  
0042A28E   /0F84 92010000   JE EmbedPE.0042A426  
0042A2BD    8078 03 CC      CMP BYTE PTR DS:\[EAX+3\],0CC  
0042A2EC   /0F84 34010000   JE EmbedPE.0042A426  
0042A31D    8078 04 CC      CMP BYTE PTR DS:\[EAX+4\],0CC  
0042A34C   /0F84 D4000000   JE EmbedPE.0042A426  
0042A37B    8078 05 CC      CMP BYTE PTR DS:\[EAX+5\],0CC  
0042A3AA   /74 7A           JE SHORT EmbedPE.0042A426  
0042A3D7  - FFE0            JMP EAX                                  ; KERNEL32.7C590C60  
  
  
0042A402    8BBE B0000000   MOV EDI,DWORD PTR DS:\[ESI+B0\]            ; regEAX, 表示第 X 个函数  
0042A431    C1E7 02         SHL EDI,2                                ; 4 个字节一个地址  
0042A45D    037E 10         ADD EDI,DWORD PTR DS:\[ESI+10\]            ; + regDr3  
0042A48B    8907            MOV DWORD PTR DS:\[EDI\],EAX               ; 保存 GetProcAddress  得到的函数地址  
  
        这里很特别, 在 OD 里 regDr3 = 0, EDI\=0, 不能继续, 但实际上我们在前面给 regDr3   
        赋了一个值 12FF84,   把 EDI 改成 12FF84, OK 了.     
        道理也很简单, OD 是根据我们的要求来设置 DRx 的 , 如果你没下硬件断点, Dr3 当然等于 0.  
        可以在 12FF84 设一硬件断点, 这样就不用手动修改了.  
  
  
0042A4B8    C746 04 0000000>MOV DWORD PTR DS:\[ESI+4\],0  
0042A4EA    C746 08 0000000>MOV DWORD PTR DS:\[ESI+8\],0  
0042A51A    C746 0C 0000000>MOV DWORD PTR DS:\[ESI+C\],0  
0042A54C    8BBE B8000000   MOV EDI,DWORD PTR DS:\[ESI+B8\]            ; regEIP=00422A7C  
0042A57D    8137 609F5758   XOR DWORD PTR DS:\[EDI\],58579F60          ; 90909090  
0042A5AE   /E9 5A020000     JMP EmbedPE.0042A80D                     ; 实际到 42A7EA  
  
  
            // EAX = 8000000X, 取出函数地址到 EAX  
  
0042A5DE    8BBE B0000000   MOV EDI,DWORD PTR DS:\[ESI+B0\]            ; regEAX  
0042A60D    C1E7 02         SHL EDI,2  
0042A63B    037E 10         ADD EDI,DWORD PTR DS:\[ESI+10\]            ; + regDr3  
0042A669    8B07            MOV EAX,DWORD PTR DS:\[EDI\]               ; 取函数地址到 regEAX  
0042A694    8986 B0000000   MOV DWORD PTR DS:\[ESI+B0\],EAX            ;   
0042A6C5    C746 04 0000000>MOV DWORD PTR DS:\[ESI+4\],0  
0042A6F7    C746 08 0000000>MOV DWORD PTR DS:\[ESI+8\],0  
0042A727    C746 0C 0000000>MOV DWORD PTR DS:\[ESI+C\],0  
0042A759    8BBE B8000000   MOV EDI,DWORD PTR DS:\[ESI+B8\]            ; regEIP  
0042A788    8137 7BDA5758   XOR DWORD PTR DS:\[EDI\],5857DA7B          ; MOV EDX,EBP (不是 NOP)  
0042A7B7    C786 A8000000 0>MOV DWORD PTR DS:\[ESI+A8\],0              ; regEDX  
  
0042A7EA    B8 00000000     MOV EAX,0  
0042A87A    5F              POP EDI                                  ; EmbedPE.0042200A  
0042A87B    5E              POP ESI  
0042A87C    5B              POP EBX  
0042A87D    C9              LEAVE  
0042A87E    C2 1000         RETN 10  
  
异常处理完毕后从异常发生处继续  
  
  
  
  
  
(四) 原程序 PE Section 解码, 注意异常处理中 Dr3 的使用  
  
尽管程序对 INT3 断点有很多校验, 但对硬件执行断点却不检查, 多利用吧  
  
00422546    B1 CC           MOV CL,0CC  
00422571    8D85 CC9D4000   LEA EAX,DWORD PTR SS:\[EBP+409DCC\]        ; 42ADCC, 看下面  
  
      0042ADCC  6B 65 72 6E 65 6C 33 32 2E 64 6C 6C 00 56 69 72  kernel32.dll.Vir  
      0042ADDC  74 75 61 6C 41 6C 6C 6F 63 00 56 69 72 74 75 61  tualAlloc.Virtua  
      0042ADEC  6C 46 72 65 65 00 56 69 72 74 75 61 6C 50 72 6F  lFree.VirtualPro  
      0042ADFC  74 65 63 74 00 53 65 74 55 6E 68 61 6E 64 6C 65  tect.SetUnhandle  
      0042AE0C  64 45 78 63 65 70 74 69 6F 6E 46 69 6C 74 65 72  dExceptionFilter  
  
004225A2    50              PUSH EAX                                 ; EmbedPE.0042ADCC  
004225CC    8B85 ECAC4000   MOV EAX,DWORD PTR SS:\[EBP+40ACEC\]        ; KERNEL32.GetModuleHandleA  
004225FB    BA E4194000     MOV EDX,EmbedPE.004019E4  
00422629    03D5            ADD EDX,EBP  
00422654    3A08            CMP CL,BYTE PTR DS:\[EAX\]                 ; 断点检查, 前 5 字节  
00422681   /75 75           JNZ SHORT EmbedPE.004226F8  
004226D7    3A48 01         CMP CL,BYTE PTR DS:\[EAX+1\]  
00422705   /75 75           JNZ SHORT EmbedPE.0042277C  
0042275B    3A48 02         CMP CL,BYTE PTR DS:\[EAX+2\]  
00422789   /75 75           JNZ SHORT EmbedPE.00422800  
004227DF    3A48 03         CMP CL,BYTE PTR DS:\[EAX+3\]  
0042280D   /75 75           JNZ SHORT EmbedPE.00422884  
00422863    3A48 04         CMP CL,BYTE PTR DS:\[EAX+4\]  
00422891   /75 75           JNZ SHORT EmbedPE.00422908  
004228E7    3A48 05         CMP CL,BYTE PTR DS:\[EAX+5\]  
00422915   /75 75           JNZ SHORT EmbedPE.0042298C  
  
0042296D    52              PUSH EDX                                 ; EmbedPE.004229E4, 返回地址  
00422999  - FFE0            JMP EAX                                  ; KERNEL32.GetModuleHandleA  
004229C4    8BD0            MOV EDX,EAX                              ; KERNEL32.7C570000  
004229F1    8985 429E4000   MOV DWORD PTR SS:\[EBP+409E42\],EAX        ; KERNEL32.7C570000  
  
  
00422A20    33C0            XOR EAX,EAX                              ; EAX\=0000000X , 保存 EBX 指向的函数地址                                                             
00422A4B    8D9D D99D4000   LEA EBX,DWORD PTR SS:\[EBP+409DD9\]        ; "VirtualAlloc"  
  
00422A7C    F0:0FC7C8       LOCK CMPXCHG8B EAX                       ; 异常计算第一个函数地址到 Stack  
  
00422A7C    90              NOP  
00422A7D    90              NOP  
00422A7E    90              NOP  
00422A7F    90              NOP  
00422A80    EB 4A           JMP SHORT EmbedPE.00422ACC  
  
00422AA9    B8 01000000     MOV EAX,1                              ; 第二个函数了  
00422AD7    8D9D E69D4000   LEA EBX,DWORD PTR SS:\[EBP+409DE6\]      ; "VirtualFree"  
  
00422B08    F0:0FC7C8       LOCK CMPXCHG8B EAX                       ; 异常计算第二个函数地址到 Stack  
  
00422B08   90               NOP  
00422B09   90               NOP  
00422B0A   90               NOP  
00422B0B   90               NOP  
00422B0C   /EB 4B           JMP SHORT EmbedPE.00422B59  
  
00422B35    8DB5 E89B4000   LEA ESI,DWORD PTR SS:\[EBP+409BE8\]      ; 42ABE8  
  
00422B64    B8 00000080     MOV EAX,80000000                       ; 有点不一样了, 80000000  
00422B94    F0:0FC7C8       LOCK CMPXCHG8B EAX                     ; 异常取出第一个函数地址到 EAX  
  
00422B94    8BD5            MOV EDX,EBP  
00422B96    90              NOP  
00422B97    90              NOP  
00422B98    EB 4A           JMP SHORT EmbedPE.00422BE4  
  
00422BC1    BF 941B4000     MOV EDI,EmbedPE.00401B94  
00422BF1    03FD            ADD EDI,EBP                              ; 422B94  
00422C1E    8137 7BDA5758   XOR DWORD PTR DS:\[EDI\],5857DA7B          ; 又变成了 Lock 指令  
00422C4D    81C2 1F214000   ADD EDX,EmbedPE.0040211F                 ; 42311F  
  
00422C7E    B3 CC           MOV BL,0CC  
00422CA9    8B4E 04         MOV ECX,DWORD PTR DS:\[ESI+4\]             ; \[42ABE8+4\]=11400  
00422CD7    6A 04           PUSH 4  
00422D04    68 00100000     PUSH 1000  
00422D34    51              PUSH ECX                                 ; 长度  
00422D60    6A 00           PUSH 0  
00422D8B    3A18            CMP BL,BYTE PTR DS:\[EAX\]  
00422DB8   /75 75           JNZ SHORT EmbedPE.00422E2F  
00422E0E    3A58 01         CMP BL,BYTE PTR DS:\[EAX+1\]  
00422E3C   /75 75           JNZ SHORT EmbedPE.00422EB3  
00422E92    3A58 02         CMP BL,BYTE PTR DS:\[EAX+2\]  
00422EC0   /75 75           JNZ SHORT EmbedPE.00422F37  
00422F16    3A58 03         CMP BL,BYTE PTR DS:\[EAX+3\]  
00422F44   /75 75           JNZ SHORT EmbedPE.00422FBB  
00422F9A    3A58 04         CMP BL,BYTE PTR DS:\[EAX+4\]  
00422FXX   /75 75           JNZ SHORT EmbedPE.0042303F  
0042301E    3A58 05         CMP BL,BYTE PTR DS:\[EAX+5\]  
0042304C   /75 75           JNZ SHORT EmbedPE.004230C3  
004230A4    52              PUSH EDX                                 ; EmbedPE.0042311F, 返回地址  
004230D0  - FFE0            JMP EAX                                  ; KERNEL32.VirtualAlloc  
  
  
        0012FF38   0042311F  /CALL to VirtualAlloc  
        0012FF3C   00000000  |Address = NULL  
        0012FF40   00011400  |Size = 11400 (70656.)  
        0012FF44   00001000  |AllocationType = MEM\_COMMIT  
        0012FF48   00000004  \\Protect = PAGE\_READWRITE  
0042311F  
004230FB    8985 E49B4000   MOV DWORD PTR SS:\[EBP+409BE4\],EAX         ; \[42ABE4\]= 340000  
0042312A    8B1E            MOV EBX,DWORD PTR DS:\[ESI\]                ; \[42ABE8\]=1000  
  
00423157    039D 3A9E4000   ADD EBX,DWORD PTR SS:\[EBP+409E3A\]        ; EmbedPE.00400000  
  
00423188    50              PUSH EAX  
004231B4    53              PUSH EBX                                 ; EmbedPE.00401000  
004231E0    E8 85550000     CALL EmbedPE.0042876A                    ; 区段解码  
0042320E    83C4 08         ADD ESP,8  
  
0042323C    56              PUSH ESI                                 ; EmbedPE.0042ABE8  
00423268    8BC8            MOV ECX,EAX                              ; 11400  
00423295    8B3E            MOV EDI,DWORD PTR DS:\[ESI\]               ; 1000  
004232C0    03BD 3A9E4000   ADD EDI,DWORD PTR SS:\[EBP+409E3A\]        ; EmbedPE.00400000  
004232EF    8BB5 E49B4000   MOV ESI,DWORD PTR SS:\[EBP+409BE4\]        ; 340000  
00423320    F3:A4           REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[ESI\]   
0042334D    5E              POP ESI                                  ; EmbedPE.0042ABE8  
  
00423377    B8 01000080     MOV EAX,80000001  
004233A7    F0:0FC7C8       LOCK CMPXCHG8B EAX                       ; 异常取出第二个函数地址  
  
004233A7    8BD5            MOV EDX,EBP  
004233A9    90              NOP  
004233AA    90              NOP  
004233AB    EB 4A           JMP SHORT EmbedPE.004233F7  
  
  
  
004233D4    BF A7234000     MOV EDI,EmbedPE.004023A7  
00423404    03FD            ADD EDI,EBP  
00423431    8137 7BDA5758   XOR DWORD PTR DS:\[EDI\],5857DA7B          ; 4233A7  
00423462    81C2 07294000   ADD EDX,EmbedPE.00402907  
00423493    B3 CC           MOV BL,0CC  
004234EF    68 00800000     PUSH 8000  
0042351F    6A 00           PUSH 0  
00423XXX    XXXX            PUSH 340000  
00423576    3A18            CMP BL,BYTE PTR DS:\[EAX\]  
004235F9    3A58 01         CMP BL,BYTE PTR DS:\[EAX+1\]  
0042367D    3A58 02         CMP BL,BYTE PTR DS:\[EAX+2\]  
00423701    3A58 03         CMP BL,BYTE PTR DS:\[EAX+3\]  
00423785    3A58 04         CMP BL,BYTE PTR DS:\[EAX+4\]  
00423809    3A58 05         CMP BL,BYTE PTR DS:\[EAX+5\]  
0042388F    52              PUSH EDX                                 ; EmbedPE.00423907  
004238BB  - FFE0            JMP EAX                                  ; KERNEL32.VirtualFree  
  
  
004238E6    83C6 08         ADD ESI,8                                   ; 下一个区段  
00423912    833E 00         CMP DWORD PTR DS:\[ESI\],0  
00423940  ^\\0F85 41F2FFFF   JNZ EmbedPE.00422B87                        ; 还有区段没解码, 继续上述过程  
  
00423946   /EB 4A           JMP SHORT EmbedPE.00423992                  ; 解码完毕到这里  
  
  
  
  
  
(五) 处理 PE 头  
  
0042396F    B8 00000080     MOV EAX,80000000  
0042399F    F0:0FC7C8       LOCK CMPXCHG8B EAX                       ; 异常取出第一个函数地址  
  
004239CC    BF 9F294000     MOV EDI,EmbedPE.0040299F  
004239FC    03FD            ADD EDI,EBP  
00423A29    8137 7BDA5758   XOR DWORD PTR DS:\[EDI\],5857DA7B  
  
00423A58    81C2 032F4000   ADD EDX,EmbedPE.00402F03  
00423A89    6A 04           PUSH 4  
00423AB6    68 00100000     PUSH 1000  
00423AE6    6A 20           PUSH 20  
00423B13    6A 00           PUSH 0  
00423B3E    B3 CC           MOV BL,0CC  
00423B69    3A18            CMP BL,BYTE PTR DS:\[EAX\]  
00423BEE    3A58 01         CMP BL,BYTE PTR DS:\[EAX+1\]  
00423C72    3A58 02         CMP BL,BYTE PTR DS:\[EAX+2\]  
00423CF8    3A58 03         CMP BL,BYTE PTR DS:\[EAX+3\]  
00423D7C    3A58 04         CMP BL,BYTE PTR DS:\[EAX+4\]  
00423E02    3A58 05         CMP BL,BYTE PTR DS:\[EAX+5\]  
00423E88    52              PUSH EDX                                 ; EmbedPE.00423F03  
00423EB4  - FFE0            JMP EAX                                  ; KERNEL32.VirtualAlloc, 分配 1000h 空间  
  
  
00423EDF    8985 3E9E4000   MOV DWORD PTR SS:\[EBP+409E3E\],EAX  
00423F0E    8BF4            MOV ESI,ESP  
00423F39    83C6 08         ADD ESI,8                                
00423F65    8BF8            MOV EDI,EAX  
00423F90    B9 08000000     MOV ECX,8  
00423FC0    F3:A5           REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]   ; 保留 Stack 里 8 DWORD, 最后恢复用  
  
00423FEB    33C0            XOR EAX,EAX  
00424016    8D9D F29D4000   LEA EBX,DWORD PTR SS:\[EBP+409DF2\]        ; "VirtualProtect"  
  
00424047    F0:0FC7C8       LOCK CMPXCHG8B EAX                       ; 计算 API　地址到 Stack  
0042404B    90              NOP  
  
00424074    B8 00000080     MOV EAX,80000000  
004240A4    F0:0FC7C8       LOCK CMPXCHG8B EAX                       ; 异常取出第一个函数地址  
  
004240A4    8BD5            MOV EDX,EBP  
004240A6    90              NOP  
004240A7    90              NOP  
004240A8    EB 4D           JMP SHORT EmbedPE.004240F7  
  
004240D3    81C2 D9354000   ADD EDX,EmbedPE.004035D9  
  
00424104    6A 00           PUSH 0  
  
00424131    6A 01           PUSH 1  
0042415E    68 870B0000     PUSH 0B87  
0042418E    BB 00B04000     MOV EBX,EmbedPE.0040B000  
004241BC    03DD            ADD EBX,EBP  
  
004241E9    53              PUSH EBX                                 ; EmbedPE.0042C000  
  
  
00424213    B3 CC           MOV BL,0CC  
0042423E    3A18            CMP BL,BYTE PTR DS:\[EAX\]  
004242C3    3A58 01         CMP BL,BYTE PTR DS:\[EAX+1\]  
00424347    3A58 02         CMP BL,BYTE PTR DS:\[EAX+2\]  
004243CD    3A58 03         CMP BL,BYTE PTR DS:\[EAX+3\]  
00424451    3A58 04         CMP BL,BYTE PTR DS:\[EAX+4\]  
004244D7    3A58 05         CMP BL,BYTE PTR DS:\[EAX+5\]  
0042455D    52              PUSH EDX                                  ; EmbedPE.004245D9  
00424589  - FFE0            JMP EAX                                   ; KERNEL32.VirtualProtect  
  
        0012FF38   004245D9  /CALL to VirtualProtect  
        0012FF3C   0042C000  |Address = EmbedPE.0042C000  
        0012FF40   00000B87  |Size = B87 (2951.)  
        0012FF44   00000001  |NewProtect = PAGE\_NOACCESS  
        0012FF48   00000000  \\pOldProtect = NULL  
  
  
004245B4    F685 219E4000 4>TEST BYTE PTR SS:\[EBP+409E21\],40         ; F8 ????  
004245E6   /0F84 940B0000   JE EmbedPE.00425180  
00424615    83BD 229E4000 0>CMP DWORD PTR SS:\[EBP+409E22\],1          ; 01 ????  
00424647   /0F85 330B0000   JNZ EmbedPE.00425180  
  
  
00424676    B8 00000080     MOV EAX,80000000  
004246A6    F0:0FC7C8       LOCK CMPXCHG8B EAX                    ; 异常取出第一个函数地址  
  
  
004246D5    81C2 E73B4000   ADD EDX,EmbedPE.00403BE7  
00424704    8D8D 1D9E4000   LEA ECX,DWORD PTR SS:\[EBP+409E1D\]  
  
  
  
  
00424735    51              PUSH ECX                              ; EmbedPE.0042AE1D  
00424761    6A 04           PUSH 4  
0042478E    6A 08           PUSH 8  
004247BB    8B9D D49B4000   MOV EBX,DWORD PTR SS:\[EBP+409BD4\]  
004247EA    039D 3A9E4000   ADD EBX,DWORD PTR SS:\[EBP+409E3A\]                        ; EmbedPE.00400000  
0042481B    53              PUSH EBX                                                 ; EmbedPE.00400148  
00424845    8038 CC         CMP BYTE PTR DS:\[EAX\],0CC  
00424XXX    8078 01 CC      CMP BYTE PTR DS:\[EAX+1\],0CC  
00424950    8078 02 CC      CMP BYTE PTR DS:\[EAX+2\],0CC  
004249D7    8078 03 CC      CMP BYTE PTR DS:\[EAX+3\],0CC  
00424A5C    8078 04 CC      CMP BYTE PTR DS:\[EAX+4\],0CC  
00424AE3    8078 05 CC      CMP BYTE PTR DS:\[EAX+5\],0CC  
00424B6A    52              PUSH EDX                                                 ; EmbedPE.00424BE7  
00424B96  - FFE0            JMP EAX                                                  ; KERNEL32.VirtualProtect  
  
  
  
      0012FF38   00424BE7  /CALL to VirtualProtect  
      0012FF3C   00400148  |Address = EmbedPE.00400148             ; Directory 的　Resource   
      0012FF40   00000008  |Size = 8  
      0012FF44   00000004  |NewProtect = PAGE\_READWRITE  
      0012FF48   0042AE1D  \\pOldProtect = EmbedPE.0042AE1D  
  
  
00424BC3    8B85 DC9B4000   MOV EAX,DWORD PTR SS:\[EBP+409BDC\]                          
00424BF2    8903            MOV DWORD PTR DS:\[EBX\],EAX                               ; \[400148\]=1F000, 资源真正的 RVA  
00424C1F    8B85 E09B4000   MOV EAX,DWORD PTR SS:\[EBP+409BE0\]                        ; 22D8  
00424C4E    8943 04         MOV DWORD PTR DS:\[EBX+4\],EAX                             ; \[40014C\]=22D8, 资源真正的长度  
  
  
00424C7A    B8 00000080     MOV EAX,80000000  
00424CAA    F0:0FC7C8       LOCK CMPXCHG8B EAX                                       ; 异常取出第一个函数地址  
00424CD9    81C2 80414000   ADD EDX,EmbedPE.00404180  
  
00424D0A    6A 00           PUSH 0  
00424D37    51              PUSH ECX  
00424D63    6A 08           PUSH 8  
00424D90    53              PUSH EBX                                                 ; EmbedPE.00400148  
00424DBA    B3 CC           MOV BL,0CC  
00424DE5    3A18            CMP BL,BYTE PTR DS:\[EAX\]  
00424E6A    3A58 01         CMP BL,BYTE PTR DS:\[EAX+1\]  
00424EEE    3A58 02         CMP BL,BYTE PTR DS:\[EAX+2\]  
00424F74    3A58 03         CMP BL,BYTE PTR DS:\[EAX+3\]  
00424FF8    3A58 04         CMP BL,BYTE PTR DS:\[EAX+4\]  
0042507E    3A58 05         CMP BL,BYTE PTR DS:\[EAX+5\]  
00425104    52              PUSH EDX                                                 ; EmbedPE.00425180  
00425130  - FFE0            JMP EAX                                                  ; KERNEL32.VirtualProtect  
  
            0012FF38   00425180  /CALL to VirtualProtect  
            0012FF3C   00400148  |Address = EmbedPE.00400148  
            0012FF40   00000008  |Size = 8  
            0012FF44   00000101  |NewProtect = PAGE\_NOACCESS|PAGE\_GUARD  
            0012FF48   00000000  \\pOldProtect = NULL  
  
  
  
0042515B    F685 219E4000 8>TEST BYTE PTR SS:\[EBP+409E21\],80  
0042518D   /0F84 CA0A0000   JE EmbedPE.00425C5D  
  
  
004251BC    B8 00000080     MOV EAX,80000000  
004251EC    F0:0FC7C8       LOCK CMPXCHG8B EAX                                      ; 异常取出第一个函数地址  
  
0042521B    81C2 4D474000   ADD EDX,EmbedPE.0040474D  
0042524A    8D8D 1D9E4000   LEA ECX,DWORD PTR SS:\[EBP+409E1D\]  
0042527B    51              PUSH ECX                                                 ; EmbedPE.0042AE1D  
004252A7    6A 04           PUSH 4  
004252D4    6A 02           PUSH 2  
00425301    8B9D D89B4000   MOV EBX,DWORD PTR SS:\[EBP+409BD8\]  
00425330    039D 3A9E4000   ADD EBX,DWORD PTR SS:\[EBP+409E3A\]                        ; EmbedPE.00400000  
00425361    53              PUSH EBX                                                 ; EmbedPE.004000C6  
0042538B    B1 CC           MOV CL,0CC  
  
  
      0012FF38   0042574D  /CALL to VirtualProtect  
      0012FF3C   004000C6  |Address = EmbedPE.004000C6             ; section number  
      0012FF40   00000002  |Size = 2  
      0012FF44   00000004  |NewProtect = PAGE\_READWRITE  
      0012FF48   0042AE1D  \\pOldProtect = EmbedPE.0042AE1D  
  
  
  
0042572C    66:0923         OR WORD PTR DS:\[EBX\],SP                                  ; 破坏区段数, 这句 NOP 掉 ============================NOP  
  
00425758    B8 00000080     MOV EAX,80000000  
00425788    F0:0FC7C8       LOCK CMPXCHG8B EAX                                       ; 异常取出第一个函数地址  
  
      0012FF38   00425C5D  /CALL to VirtualProtect  
      0012FF3C   004000C6  |Address = EmbedPE.004000C6  
      0012FF40   00000002  |Size = 2  
      0012FF44   00000101  |NewProtect = PAGE\_NOACCESS|PAGE\_GUARD  
      0012FF48   00000000  \\pOldProtect = NULL  
  
  
  
  
00425C39    8B95 3A9E4000   MOV EDX,DWORD PTR SS:\[EBP+409E3A\]                        ; EmbedPE.00400000  
00425C6A    8B85 2A9E4000   MOV EAX,DWORD PTR SS:\[EBP+409E2A\]                        ; EmbedPE.00400000  
00425C99    2BD0            SUB EDX,EAX                                              ; EmbedPE.00400000  
00425CC6   /0F84 C8040000   JE EmbedPE.00426194                                      ; 重定位处理?  
  
  
  
  
(六)  一个简单的花招后,  调用 SetUnhandledExceptionFilter    
  
  
00426170    8B85 C89D4000   MOV EAX,DWORD PTR SS:\[EBP+409DC8\]        ; D000  
004261A1    0385 3A9E4000   ADD EAX,DWORD PTR SS:\[EBP+409E3A\]        ; EmbedPE.00400000  
004261D0    B9 00010000     MOV ECX,100  
004261FE    B3 C3           MOV BL,0C3                               ; RET  
00426229    8BF0            MOV ESI,EAX                              ; EmbedPE.0040D000  
  
00426256    83C6 02         ADD ESI,2  
00426284    861E            XCHG BYTE PTR DS:\[ESI\],BL  
004262B1    FFD6            CALL ESI                                 ; EmbedPE.0040D002,   
004262DC    861E            XCHG BYTE PTR DS:\[ESI\],BL  
00426309    49              DEC ECX  
00426335  ^\\0F85 3CFFFFFF   JNZ EmbedPE.00426277                      ; 实际上到 426256  
  
  
00426364    F685 219E4000 2>TEST BYTE PTR SS:\[EBP+409E21\],21  
00426396   /0F84 17060000   JE EmbedPE.004269B3  
  
004263C5    33C0            XOR EAX,EAX                              ; EmbedPE.0040D000  
004263F0    8D9D 019E4000   LEA EBX,DWORD PTR SS:\[EBP+409E01\]        ; "SetUnhandledExceptionFilter"  
00426421    F0:0FC7C8       LOCK CMPXCHG8B EAX                       ; 异常计算该函数地址  
  
0042644E    B8 00000080     MOV EAX,80000000  
0042647E    F0:0FC7C8       LOCK CMPXCHG8B EAX                       ; 异常取出该函数地址到 EAX  
  
004264AB    BF 7E544000     MOV EDI,EmbedPE.0040547E  
004264DB    03FD            ADD EDI,EBP  
00426508    8137 7BDA5758   XOR DWORD PTR DS:\[EDI\],5857DA7B  
00426539    81C2 B3594000   ADD EDX,EmbedPE.004059B3  
00426568    BB E77C4000     MOV EBX,EmbedPE.00407CE7  
00426596    03DD            ADD EBX,EBP  
004265C3    53              PUSH EBX                                 ; EmbedPE.00428CE7  
  
      0012FF44   004269B3  /CALL to SetUnhandledExceptionFilter  
      0012FF48   00428CE7  \\pTopLevelFilter = EmbedPE.00428CE7  
  
  
  
(七)  恢复 IAT  
  
  
0042698E    F685 219E4000 0>TEST BYTE PTR SS:\[EBP+409E21\],2  
004269C0   /0F84 A5030000   JE EmbedPE.00426D6B  
  
  
00426D47    8B95 3A9E4000   MOV EDX,DWORD PTR SS:\[EBP+409E3A\]        ; EmbedPE.00400000  
00426D76    8BB5 4E9E4000   MOV ESI,DWORD PTR SS:\[EBP+409E4E\]        ; 130F0  
00426DA5    03F2            ADD ESI,EDX                              ; EmbedPE.00400000  
  
  
  
  
00426DD2    8B46 0C         MOV EAX,DWORD PTR DS:\[ESI+C\]             ; 1333C  
00426DFE    85C0            TEST EAX,EAX  
00426E2B   /0F84 3E150000   JE EmbedPE.0042836F                      ; // 大循环出口, 所有 DLL 都处理完了  
  
00426E5A    03C2            ADD EAX,EDX                              ; EmbedPE.00400000  
00426E87    8BD8            MOV EBX,EAX                              ; EmbedPE.0041333C = "kernel32.dll"  
00426EB4    50              PUSH EAX                                 ; EmbedPE.0041333C  
00426EDE    8B85 ECAC4000   MOV EAX,DWORD PTR SS:\[EBP+40ACEC\]        ; KERNEL32.GetModuleHandleA  
00426F0D    BA 27634000     MOV EDX,EmbedPE.00406327  
00426F3B    03D5            ADD EDX,EBP  
  
00426F66    B1 CC           MOV CL,0CC  
00426F91    3A08            CMP CL,BYTE PTR DS:\[EAX\]  
00427xxx    3A48 01         CMP CL,BYTE PTR DS:\[EAX+5\]  
004272B0    52              PUSH EDX                                 ; EmbedPE.00427327  
004272DC  - FFE0            JMP EAX                                  ; KERNEL32.GetModuleHandleA  
00427307    85C0            TEST EAX,EAX                             ; KERNEL32.7C570000  
00427334   /0F85 A4040000   JNZ EmbedPE.004277DE                     ; DLL 已经加载跳到 004277BA, 否则要加载  
  
00427365    53              PUSH EBX                                             ; EmbedPE.00413468  
0042738F    8B85 F0AC4000   MOV EAX,DWORD PTR SS:\[EBP+40ACF0\]                    ; KERNEL32.LoadLibraryA  
004273BE    BA DE674000     MOV EDX,EmbedPE.004067DE  
004273EC    03D5            ADD EDX,EBP  
00427417    B1 CC           MOV CL,0CC  
00427761    52              PUSH EDX                                             ; EmbedPE.004277DE  
0042778D  - FFE0            JMP EAX                                              ; KERNEL32.LoadLibraryA  
  
  
  
004277BA    8985 469E4000   MOV DWORD PTR SS:\[EBP+409E46\],EAX        ; KERNEL32.7C570000  
004277E9    C785 529E4000 0>MOV DWORD PTR SS:\[EBP+409E52\],0  
0042781E    8B95 3A9E4000   MOV EDX,DWORD PTR SS:\[EBP+409E3A\]        ; EmbedPE.00400000  
0042784F    8B06            MOV EAX,DWORD PTR DS:\[ESI\]  
0042787A    85C0            TEST EAX,EAX  
004278A7   /75 75           JNZ SHORT EmbedPE.0042791E  
004278FE    03C2            ADD EAX,EDX                              ; EmbedPE.00400000  
00427929    0385 529E4000   ADD EAX,DWORD PTR SS:\[EBP+409E52\]  
00427958    8B18            MOV EBX,DWORD PTR DS:\[EAX\]  
00427985    8B7E 10         MOV EDI,DWORD PTR DS:\[ESI+10\]  
004279B3    03FA            ADD EDI,EDX                              ; EmbedPE.00400000  
004279E0    03BD 529E4000   ADD EDI,DWORD PTR SS:\[EBP+409E52\]  
00427A0F    85DB            TEST EBX,EBX  
00427A3C   /0F84 CB080000   JE EmbedPE.0042830D                      ; // 小循环出口, 一个 DLL 的函数处理完了  
00427A6D    53              PUSH EBX  
00427A97    F7C3 00000080   TEST EBX,80000000  
00427AC8   /0F85 D2000000   JNZ EmbedPE.00427BA0  
00427AF7    03DA            ADD EBX,EDX                              ; EmbedPE.00400000  
00427B24    43              INC EBX                                  ; EmbedPE.00413274  
00427B50    43              INC EBX  
00427B7C    81E3 FFFFFF0F   AND EBX,0FFFFFFF  
00427BAD    53              PUSH EBX                                 ; EmbedPE.00413276  "CreateMutexA"  
00427BD9    FFB5 469E4000   PUSH DWORD PTR SS:\[EBP+409E46\]           ; KERNEL32.7C570000  
00427C08    8B85 E8AC4000   MOV EAX,DWORD PTR SS:\[EBP+40ACE8\]        ; KERNEL32.GetProcAddress  
00427C37    BA 53704000     MOV EDX,EmbedPE.00407053  
00427C65    03D5            ADD EDX,EBP  
00427C90    B1 CC           MOV CL,0CC  
00427CBB    3A08            CMP CL,BYTE PTR DS:\[EAX\]  
00427XXX    3A08            CMP CL,BYTE PTR DS:\[EAX+5\]  
00428006  - FFE0            JMP EAX                                  ; KERNEL32.GetProcAddress  
00428033    8907            MOV DWORD PTR DS:\[EDI\],EAX               ; 写入 API 地址, 这句 dump 时可以 NOP 掉.  
                 
0042805E    8385 529E4000 0>ADD DWORD PTR SS:\[EBP+409E52\],4  
00428090    5B              POP EBX                                  ; EmbedPE.00413276  
004280BA    F7C3 00000080   TEST EBX,80000000  
004280EB   /0F85 F0010000   JNZ EmbedPE.004282E1  
0042811C    F685 219E4000 0>TEST BYTE PTR SS:\[EBP+409E21\],8  
0042814E   /0F84 8D010000   JE EmbedPE.004282E1  
0042817F    83C3 02         ADD EBX,2  
004281AD    039D 3A9E4000   ADD EBX,DWORD PTR SS:\[EBP+409E3A\]        ; EmbedPE.00400000  
  
004281DC    33C0            XOR EAX,EAX                                
00428207    8903            MOV DWORD PTR DS:\[EBX\],EAX               ; //破坏, NOP 掉这句=============================================NOP  
00428234    43              INC EBX                                  ; EmbedPE.00413276  
00428260    3903            CMP DWORD PTR DS:\[EBX\],EAX  
0042828D  ^\\0F85 69FFFFFF   JNZ EmbedPE.004281FC                     ; 实际到 4281DC  
  
004282BE  ^\\E9 7FF5FFFF     JMP EmbedPE.00427842                     ; // 小循环, 实际到 42781E                  
  
  
0042830D    51              PUSH ECX                                 ; // 小循环结束到这里  
0042830E    E8 EEFFFFFF     CALL EmbedPE.00428301  
  
  
004282EC    83C6 14         ADD ESI,14  
0042831A  ^\\E9 D4EAFFFF     JMP EmbedPE.00426DF3                     ; // 大循环, 实际到 426DD2  
  
  
  
0042836F    51              PUSH ECX                                 ; // 大循环结束到这里  
00428370    E8 EEFFFFFF     CALL EmbedPE.00428363  
  
  
  
  
(八)  小花招之后, 从分配的空间里恢复 STACK, 跳到 OEP  
  
0042834A    64:8F05 0000000>POP DWORD PTR FS:\[0\]                     ; 恢复 SEH  
0042837A    83C4 04         ADD ESP,4  
004283A8    8B85 C89D4000   MOV EAX,DWORD PTR SS:\[EBP+409DC8\]  
004283D9    0385 3A9E4000   ADD EAX,DWORD PTR SS:\[EBP+409E3A\]        ; EmbedPE.00400000  
00428408    B9 00010000     MOV ECX,100  
00428436    B3 C3           MOV BL,0C3  
00428461    8BF0            MOV ESI,EAX                              ; EmbedPE.0040D000  
0042848E    83C6 02         ADD ESI,2  
004284BC    861E            XCHG BYTE PTR DS:\[ESI\],BL  
004284E9    FFD6            CALL ESI                                 ; EmbedPE.0040D002  
00428514    861E            XCHG BYTE PTR DS:\[ESI\],BL  
00428541    49              DEC ECX  
0042856D  ^\\0F85 3CFFFFFF   JNZ EmbedPE.004284AF  
0042859E    8B85 269E4000   MOV EAX,DWORD PTR SS:\[EBP+409E26\]  
004285CF    0385 3A9E4000   ADD EAX,DWORD PTR SS:\[EBP+409E3A\]        ; EmbedPE.00400000  
00428604    8BB5 3E9E4000   MOV ESI,DWORD PTR SS:\[EBP+409E3E\]  
00428633    8BFC            MOV EDI,ESP  
00428660    B9 08000000     MOV ECX,8  
00428690    F3:A5           REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]      ; // 恢复 STACK  
004286BB    83EC 20         SUB ESP,20  
004286E9    894424 1C       MOV DWORD PTR SS:\[ESP+1C\],EAX                        ; EmbedPE.0040DF11  
00428718    61              POPAD  
00428744  - FFE0            JMP EAX                                              ; EmbedPE.0040DF11  
  
  
  
(九) 最终异常处理函数实现 API 调用  
  
  
上面两处 NOP , 到 OEP 后, 我们去 413000 看看, 完整的 IAT 就在这里, 连 ImportREC 都不要用了.   
  
就这么简单吗? 当然不是, F7 走, 40DF5C 我们就遇上了麻烦.  
  
0040DF11    68 6A404100     PUSH EmbedPE.0041406A         ; ASCII " EmbedPE v1.13 "  
0040DF16    6A 01           PUSH 1  
0040DF18    6A 00           PUSH 0  
0040DF1A    E8 3D000000     CALL EmbedPE.0040DF5C         ; // F7  
0040DF1F    E8 50000000     CALL EmbedPE.0040DF74  
0040DF24    3D B7000000     CMP EAX,0B7  
0040DF29    74 1D           JE SHORT EmbedPE.0040DF48  
0040DF2B    6A 00           PUSH 0  
0040DF2D    E8 48000000     CALL EmbedPE.0040DF7A  
0040DF32    A3 14404100     MOV DWORD PTR DS:\[414014\],EAX  
0040DF37    6A 00           PUSH 0  
0040DF39    68 97D94000     PUSH EmbedPE.0040D997  
0040DF3E    6A 00           PUSH 0  
0040DF40    6A 67           PUSH 67  
0040DF42    50              PUSH EAX  
0040DF43    E8 6E000000     CALL EmbedPE.0040DFB6  
0040DF48    6A 00           PUSH 0  
0040DF4A    E8 19000000     CALL EmbedPE.0040DF68  
0040DF4F    CC              INT3  
0040DF50    CC              INT3  
0040DF51    90              NOP  
0040DF52    25 600C45CC     AND EAX,CC450C60  
0040DF57    90              NOP  
0040DF58    65:60           PUSHAD  
0040DF5A    0C 45           OR AL,45  
  
0040DF5C    CC              INT3                           // 这个就是 JMP　［ＸＸＸＸ］　的变形  
0040DF5D    90              NOP  
0040DF5E    49              DEC ECX  
0040DF5F    60              PUSHAD  
0040DF60    0C 45           OR AL,45  
  
0040DF62    CC              INT3  
0040DF63    90              NOP  
0040DF64    61              POPAD  
0040DF65    60              PUSHAD  
0040DF66    0C 45           OR AL,45  
  
  
如果不使用 插件, OD 将不能继续, 使用插件后, 下断 428CE7, 可以看到是怎样到 API 的  
注意最终异常处理例程的入栈参数是 \*EXCEPTION\_POINTERS    
  
 EXCEPTION\_POINTERS STRUCT   
              pExceptionRecord  DWORD      ?                
              pContextRecord    DWORD      ?   
 EXCEPTION\_POINTERS ENDS   
  
 在 call xHandler 之前,堆栈结构如下:   
            esp    ->   \*EXCEPTION\_POINTERS   
  
  
  
00428CE7    55              PUSH EBP  
00428CE8    8BEC            MOV EBP,ESP  
00428CEA    53              PUSH EBX  
00428CEB    51              PUSH ECX  
00428CEC    52              PUSH EDX  
00428CED    56              PUSH ESI  
00428CEE    57              PUSH EDI  
  
00428CF8    8B7D 08         MOV EDI,DWORD PTR SS:\[EBP+8\]           ; pEXCEPTION\_POINTERS  
00428D24    8B37            MOV ESI,DWORD PTR DS:\[EDI\]             ; pExceptionRecord  
00428D4F    813E 03000080   CMP DWORD PTR DS:\[ESI\],80000003        ; 异常码  
00428D80   /0F85 B50D0000   JNZ EmbedPE.00429B3B  
00428DB1    8B77 04         MOV ESI,DWORD PTR DS:\[EDI+4\]           ; pContextRecord  
  
00428DD8    E8 00000000     CALL EmbedPE.00428DDD  
00428DDD    5B              POP EBX  
00428DDE    81EB DD7D4000   SUB EBX,EmbedPE.00407DDD               ; EBX = 21000  
  
00428DEB    BF A8984000     MOV EDI,EmbedPE.004098A8  
00428E19    03FB            ADD EDI,EBX                            ; EDI = 42A8A8, 一个 TABLE, 见下面  
  
00428E46    8B07            MOV EAX,DWORD PTR DS:\[EDI\]             ; 下面开始循环, 从 TABLE 里找             
00428E73    85C0            TEST EAX,EAX              
00428EA0   /0F84 950C0000   JE EmbedPE.00429B3B                     ; Table 结束了  
00428ECF    8B96 B8000000   MOV EDX,DWORD PTR DS:\[ESI+B8\]           ; regEIP = 40DF5C, 异常发生地址  
00428F00    2B93 3A9E4000   SUB EDX,DWORD PTR DS:\[EBX+409E3A\]       ; EmbedPE.00400000  
  
00428F31    50              PUSH EAX  
  
00428F5B    8BC2            MOV EAX,EDX                               
00428F88    E8 21F9FFFF     CALL EmbedPE.004288AE                   ; 变换 EAX, 见下面  
00428FB8    8BD0            MOV EDX,EAX                             ; 45391B13  
  
00428FE5    58              POP EAX  
  
0042900F    3BC2            CMP EAX,EDX  
0042903C   /0F84 AB000000   JE EmbedPE.004290ED                     ; 找到了跳出循环, 实际到 4090C9  
0042906D    83C7 08         ADD EDI,8                               ; 不对, 下一个, 每项 8 字节  
0042909B  ^\\E9 C6FDFFFF     JMP EmbedPE.00428E66                    ; 实际到 428E46,  循环  
  
  
004290C9    8B86 C4000000   MOV EAX,DWORD PTR DS:\[ESI+C4\]           ; regESP  
004290FA    56              PUSH ESI  
00429126    57              PUSH EDI                                ; EmbedPE.0042A8C0,  在 Table 找到的位置  
  
00429150    8BF0            MOV ESI,EAX                             ; 异常发生时的 Stack  
0042917B    8BFC            MOV EDI,ESP  
004291A6    83EF 40         SUB EDI,40                              ; 12F81C  
004291D4    B9 40000000     MOV ECX,40  
00429204    F3:A4           REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[ESI\] ; 把异常发生时 Stack 取出来, API 的参数  
  
00429231    5F              POP EDI                                 ; EmbedPE.0042A8C0  
0042925D    5E              POP ESI                                 ; 0012FCA0  
  
  
00429289    83EC 48         SUB ESP,48                              ; 12F81C  
004292B5    8BD4            MOV EDX,ESP  
  
004292E0    8B47 04         MOV EAX,DWORD PTR DS:\[EDI+4\]            ; Table + 4  
0042930E    F683 219E4000 2>TEST BYTE PTR DS:\[EBX+409E21\],20        ; \[42AE21\]  
00429340   /0F84 B3050000   JE EmbedPE.004298F9  
  
0042936F    A9 00000080     TEST EAX,80000000  
0042939F   /0F85 33020000   JNZ EmbedPE.004295D8  
  
004295B4    8F86 B8000000   POP DWORD PTR DS:\[ESI+B8\]               ; EmbedPE.0040DF1F, 修改 regEIP, API 的返回地址  
004295E3    25 FFFFFF7F     AND EAX,7FFFFFFF                        ; 8041300C  
  
00429613    879E A4000000   XCHG DWORD PTR DS:\[ESI+A4\],EBX          ; regEBX <-> EBX, API 参数  
00429644    878E AC000000   XCHG DWORD PTR DS:\[ESI+AC\],ECX          ; regECX <-> ECX, API 参数  
00429675    8796 A8000000   XCHG DWORD PTR DS:\[ESI+A8\],EDX          ; regEDX <-> EDX, API 参数  
004296A6    FF10            CALL DWORD PTR DS:\[EAX\]                 ; KERNEL32.CreateMutexA  
  
004296D3    879E A4000000   XCHG DWORD PTR DS:\[ESI+A4\],EBX          ; API 结果  
00429704    878E AC000000   XCHG DWORD PTR DS:\[ESI+AC\],ECX          ; API 结果  
00429735    8796 A8000000   XCHG DWORD PTR DS:\[ESI+A8\],EDX          ; API 结果  
00429764    8986 B0000000   MOV DWORD PTR DS:\[ESI+B0\],EAX           ; API 结果  
  
  
00429793    2BD4            SUB EDX,ESP                             ; 这个 API 有几个参数  
  
004297BE    2996 C4000000   SUB DWORD PTR DS:\[ESI+C4\],EDX           ; regESP, 修正 Stack  
  
004297EF    03D4            ADD EDX,ESP  
0042981A    83C2 48         ADD EDX,48  
  
00429848    8BE2            MOV ESP,EDX  
  
00429875    B8 FFFFFFFF     MOV EAX,-1                              ; EXCEPTION\_CONTINUE\_EXECUTION  
00429B48    5F              POP EDI  
00429B49    5E              POP ESI  
00429B4A    5A              POP EDX  
00429B4B    59              POP ECX  
00429B4C    5B              POP EBX  
00429B4D    C9              LEAVE  
00429B4E    C2 0400         RETN 4  
  
  
  
// 对 EAX 做一变换  
004288AE    53              PUSH EBX  
004288AF    51              PUSH ECX  
004288B0    52              PUSH EDX  
004288B1    56              PUSH ESI  
  
004288B2    E8 00000000     CALL EmbedPE.004288B7  
004288B7    5B              POP EBX  
004288B8    81EB B7784000   SUB EBX,EmbedPE.004078B7  
004288BE    81C3 E7784000   ADD EBX,EmbedPE.004078E7    // EBX = 4288E7  
  
004288C4    8BC8            MOV ECX,EAX  
004288C6    33D2            XOR EDX,EDX  
004288C8    83C8 FF         OR EAX,FFFFFFFF  
004288CB    BE 04000000     MOV ESI,4  
  
004288D0    C1C1 08         ROL ECX,8  
004288D3    8AD1            MOV DL,CL  
004288D5    32D0            XOR DL,AL  
004288D7    C1E8 08         SHR EAX,8  
004288DA    330493          XOR EAX,DWORD PTR DS:\[EBX+EDX\*4\]  
004288DD    4E              DEC ESI  
004288DE  ^ 75 F0           JNZ SHORT EmbedPE.004288D0  
  
004288E0    F7D0            NOT EAX  
004288E2    5E              POP ESI  
004288E3    5A              POP EDX  
004288E4    59              POP ECX  
004288E5    5B              POP EBX  
004288E6    C3              RETN  
  
  
0042A8A8  38 57 8F 4C 60 30 41 80 93 4A EE 8D 14 30 41 80  8W 廘\`0A€揓顛 0A€  
0042A8B8  0D F2 EC A5 20 30 41 80 13 1B 39 45 0C 30 41 80  .蜢?0A€9E.0A€  
0042A8C8  B8 06 58 84 24 30 41 80 A6 EF 8D 64 10 30 41 80  ?X?0A€︼峝 0A€  
0042A8D8  EC 85 5C CA 3C 30 41 80 C7 C9 EA C3 28 30 41 80  靺\\?0A€巧昝(0A€  
0042A8E8  E9 B3 8C 70 18 30 41 80 D9 20 3F 23 40 30 41 80  槌宲 0A€??#@0A€  
0042A8F8  EE 9E 34 97 1C 30 41 80 A4 F4 E5 39 2C 30 41 80  顬 4?0A€?,0A€  
0042A908  BA 1D 30 D9 30 30 41 80 3D 61 5F F6 38 30 41 80  ?0?0A€=a\_?0A€  
0042A918  6B F9 33 E5 A8 30 41 80 3A 4C E7 11 84 30 41 80  k?濞 0A€:L??A€  
0042A928  40 B5 85 EC 90 30 41 80 75 10 E6 05 B8 30 41 80  @祬鞇 0A€u??A€  
0042A938  4E 4C 5B B2 AC 30 41 80 50 A5 8E 52 B0 30 41 80  NL\[铂 0A€PR?A€  
0042A948  8F B8 53 30 34 30 41 80 81 41 8D 6E 98 30 41 80  徃 S040A€丄峮?A€  
0042A958  06 3D E2 41 A0 30 41 80 01 10 5A A6 A4 30 41 80  =釧?A€ZΔ0A€  
0042A968  B4 E4 EE 87 BC 30 41 80 7B E9 38 5B 9C 30 41 80  翠顕?A€{?\[?A€  
0042A978  FC 95 57 74 88 30 41 80 9F A8 58 8E 94 30 41 80  鼤 Wt?A€煥 X 帞 0A€  
0042A988  E2 7C 82 94 8C 30 41 80 94 3C 8D 62 B4 30 41 80  鈢倲?A€?峛?A€  
0042A998  D7 D9 E1 7D 80 30 41 80 E9 E8 57 78 C0 30 41 80  踪醹€0A€殍 Wx?A€  
0042A9A8  EE C5 EF 9F 7C 30 41 80 BF 70 3B 6B 00 30 41 80  钆餆|0A€縫;k.0A€  
0042A9B8  A1 99 EE 8B 04 30 41 80 0A 84 8F 4A 6C 30 41 80  顙 0A€.剰 Jl0A€  
0042A9C8  14 6D 5A AA 74 30 41 80 64 2C 8D 7E 4C 30 41 80  mZ 猼 0A€d,崀 L0A€  
0042A9D8  35 99 59 8A 54 30 41 80 45 D8 8E 5E 68 30 41 80  5 橸奣 0A€E 貛^h0A€  
0042A9E8  D1 D8 39 5F 58 30 41 80 7A C5 58 9E 50 30 41 80  沿 9\_X0A€z 臱濸 0A€  
0042A9F8  CF 31 EC BF 48 30 41 80 4F 60 3B 77 5C 30 41 80  ?炜 H0A€O\`;w\\0A€  
0042AA08  3F 21 EC A3 70 30 41 80 E4 7D 5A B6 44 30 41 80  ?!欤 p0A€鋧 Z 禗 0A€  
0042AA18  F3 40 E9 EF C8 30 41 80 BB 31 50 1C D0 30 41 80  驚轱?A€?P?A€  
0042AA28  9E 84 38 4B DC 30 41 80 90 7D E6 15 D8 30 41 80  瀯 8K?A€恾??A€  
0042AA38  8E 94 33 F5 D4 30 41 80 D8 0C 5F E6 E4 30 41 80  帞 3 踉 0A€?\_ 驿 0A€  
0042AA48  99 A9 80 AC CC 30 41 80 6A D5 53 20 E8 30 41 80  櫓€0A€j 誗 ?A€  
0042AA58  ED A9 3C 0F E0 30 41 80                          愆<?A€..  
  
  
  
  
// 根据上面可写一段修复代码, 如下, 重新来过, 把 428033 这句也 NOP 掉.  
// 到 OEP 后, 修复代码放到 860000  
  
60 B8 00 10 40 00 80 38 CC 75 49 50 40 80 38 90 75 41 40 8B 00 25 00 FF FF FF 3D 00 60 0C 45 75  
32 8B 04 24 2D 00 00 40 00 E8 80 88 BC FF BF A8 A8 42 00 39 07 74 05 83 C7 08 EB F7 8B 47 04 25  
FF FF FF 00 8B 1C 24 66 C7 03 FF 25 89 43 02 83 04 24 05 58 40 3D 00 23 41 00 7C AA 61  
  
  
  
00860000    60              PUSHAD  
00860001    B8 00104000     MOV EAX,401000                     ; // 搜索从 401000 开始                  
00860006    8038 CC         CMP BYTE PTR DS:\[EAX\],0CC          ; // INT3  
00860009    75 49           JNZ SHORT 00860054  
  
0086000B    50              PUSH EAX  
0086000C    40              INC EAX  
0086000D    8038 90         CMP BYTE PTR DS:\[EAX\],90           ; // INT3 跟 NOP   
00860010    75 41           JNZ SHORT 00860053  
  
00860012    40              INC EAX  
00860013    8B00            MOV EAX,DWORD PTR DS:\[EAX\]  
00860015    25 00FFFFFF     AND EAX,FFFFFF00  
0086001A    3D 00600C45     CMP EAX,450C6000                   ;  // NOP 后面还有 XX, 60, 0C, 45             
0086001F    75 32           JNZ SHORT 00860053  
  
00860021    8B0424          MOV EAX,DWORD PTR SS:\[ESP\]         ; // 异常发生处  
00860024    2D 00004000     SUB EAX,400000  
00860029    E8 8088BCFF     CALL EmbedPE.004288AE              ; // 变换  
  
0086002E    BF A8A84200     MOV EDI,42A8A8                     ; // 查表  
00860033    3907            CMP DWORD PTR DS:\[EDI\],EAX  
00860035    74 05           JE SHORT 0086003C  
00860037    83C7 08         ADD EDI,8  
0086003A  ^ EB F7           JMP SHORT 00860033  
  
0086003C    8B47 04         MOV EAX,DWORD PTR DS:\[EDI+4\]  
0086003F    25 FFFFFF00     AND EAX,0FFFFFF  
00860044    8B1C24          MOV EBX,DWORD PTR SS:\[ESP\]        ; // 异常发生处  
00860047    66:C703 FF25    MOV WORD PTR DS:\[EBX\],25FF  
0086004C    8943 02         MOV DWORD PTR DS:\[EBX+2\],EAX      ; // JMP \[XXXXXXXX\]  
0086004F    830424 06       ADD DWORD PTR SS:\[ESP\],5  
  
00860053    58              POP EAX  
00860054    40              INC EAX  
00860055    3D 00234100     CMP EAX,412300                   ; // 搜索到 412300 结束  
0086005A  ^ 7C AA           JL SHORT 00860006  
0086005C    61              POPAD  
  
  
  
DUMP 后, LordPE 改一下 IAT 就可以了,  不需要 ImportREC.  
  
RVA = 4130F0, size = 78  
  
  
最后一个区段可删除, 优化一下.

 4dd466d0e1e0cc123afbe504da2462ca