

# duzaizhe 的旧 hying 修改壳脱壳过程

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-494.htm`](https://www.pediy.com/kssd/pediy07/pediy7-494.htm)

【脱壳作者】csjwaman\[DFCG\]  
  
【使用工具】OD 等传统工具  
  
【操作系统】winxp sp1  
  
【软件名称】duzaizhe 的旧 hying 修改壳加壳的 win98 记事本  
  
【加壳方式】旧 hying 修改壳, 附件:notepad.rar   
  
【脱壳声明】我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
  
\--------------------------------------------------------------------------------  
  
【脱壳过程】  
  
  
这个壳虽然可以用 forgot 的脱壳机自动脱壳，但为了学习脱壳技术，我还是手动跟踪了。在跟踪时碰到的第一个拦路虎就是 OD 的 BUG－－载入后一加载 OutputDebugStringA 函数就退出！后来在 forgot 的指点下才搞定了 BUG。下面我将脱壳过程写一下：  
  
  
  
OD 载入程序，忽略所有异常，隐藏 OD，bp VirtualAlloc，F9 运行断下：  
  
77E5AC72 >  55              PUSH EBP///断在函数入口。  
77E5AC73    8BEC            MOV EBP,ESP  
77E5AC75    FF75 14         PUSH DWORD PTR SS:\[EBP+14\]  
77E5AC78    FF75 10         PUSH DWORD PTR SS:\[EBP+10\]  
77E5AC7B    FF75 0C         PUSH DWORD PTR SS:\[EBP+C\]  
77E5AC7E    FF75 08         PUSH DWORD PTR SS:\[EBP+8\]  
77E5AC81    6A FF           PUSH -1  
  
堆栈区：  
  
0012FFB0   0040D3C9  指针到下一个 SEH 记录  
0012FFB4   00000000  SE 句柄  
0012FFB8   000056E2  |Size = 56E2 (22242.)////申请内存空间大小。  
0012FFBC   00001000  |AllocationType = MEM\_COMMIT  
0012FFC0   00000004  \\Protect = PAGE\_READWRITE  
  
ALT+F9 返回：  
  
0040D3C9    60              PUSHAD///返回到这里。此时 EAX=00370000  
0040D3CA    B9 04000000     MOV ECX,4  
0040D3CF    E8 1F000000     CALL notepad.0040D3F3  
0040D3D4  ^ EB FA           JMP SHORT notepad.0040D3D0  
0040D3D6    E8 16000000     CALL notepad.0040D3F1  
0040D3DB  - E9 EBF80000     JMP 0041CCCB  
0040D3E0    58              POP EAX  
0040D3E1    EB 09           JMP SHORT notepad.0040D3EC  
  
返回时 EAX=00370000，可见程序申请了一个首地址为 00370000，大小为 56E2H 的空间。后面解码时许多代码都往这里放，以抵抗 DUMP。现在往下找，一直来到：  
  
0040D5D5    E8 EEFFFFFF     CALL notepad.0040D5C8  
0040D5DA    13C9            ADC ECX,ECX  
0040D5DC    E8 E7FFFFFF     CALL notepad.0040D5C8  
0040D5E1  ^ 72 F2           JB SHORT notepad.0040D5D5  
0040D5E3    C3              RETN  
0040D5E4    2B7C24 28       SUB EDI,DWORD PTR SS:\[ESP+28\]  
0040D5E8    897C24 1C       MOV DWORD PTR SS:\[ESP+1C\],EDI  
0040D5EC    61              POPAD  
0040D5ED    C2 0800         RETN 8////F4 到这里。  
0040D5F0    E8 7007C702     CALL 0307DD65  
0040D5F5    C783 10C013EB 0>MOV DWORD PTR DS:\[EBX+EB13C010\],1B58730B  
0040D5FF    02CD            ADD CL,CH  
  
F4 到 40D5ED 后，CTRL＋G 来到：  
  
00372225    61                         POPAD  
00372226    300B                       XOR BYTE PTR DS:\[EBX\],CL////直接 F4 到这里！  
00372228    F8              　　　　　 CLC  
00372229    EB 01                      JMP SHORT 0037222C  
0037222B    E8 FFD0E800                CALL 011FF32F////这里有个花指令。  
00372230    0000                       ADD BYTE PTR DS:\[EAX\],AL  
  
取消花指令后：  
  
00372226    300B            XOR BYTE PTR DS:\[EBX\],CL////这句要 NOP 掉！  
00372228    F8              CLC/////如果上面一句不 NOP 掉，则这句会变成 NOP！导致后面解码错误。  
00372229    EB 01           JMP SHORT 0037222C  
0037222B    90              NOP  
0037222C    FFD0            CALL NEAR EAX////调用 kernel32.OutputDebugStringA 函数。  
0037222E    E8 00000000     CALL 00372233  
00372233    5B              POP EBX  
00372234    81EB D71A4000   SUB EBX,401AD7  
0037223A    56              PUSH ESI  
  
观察一个堆栈区：  
  
0012FFA0   003724F9  ASCII "%s%s%s%s%s%s"////参数。  
  
据 forgot 称，OD 有个 BUG，就是在调用 OutputDebugStringA 函数时，如果参数为"%s%s%s%s%s%s"就会自动退出。现在我们把参数改为“UUUUUUUUUUUU”。  
  
继续跟踪来到：  
  
00372479    C783 64FF3500 00000064    MOV DWORD PTR DS:\[EBX+35FF64\],64000000  
00372483    8925 00000000             MOV DWORD PTR DS:\[0\],ESP  
00372489    AD                        LODS DWORD PTR DS:\[ESI\]  
0037248A    CD 20                     INT 20  
0037248C    61                        POPAD  
0037248D    FFD0                      CALL NEAR EAX   ; kernel32.GetTickCount////获取当前时间标识。  
0037248F    8983 AA1D4000             MOV DWORD PTR DS:\[EBX+401DAA\],EAX////保存当前时间标识，为后面的比较作准备。  
00372495    60                        PUSHAD  
00372496    B9 04000000               MOV ECX,4  
0037249B    E8 1F000000               CALL 003724BF  
003724A0  ^ EB FA                     JMP SHORT 0037249C  
003724A2    E8 16000000               CALL 003724BD  
003724A7  - E9 EBF80000               JMP 00381D97  
003724AC    58                        POP EAX  
003724AD    EB 09                     JMP SHORT 003724B8  
003724AF    0F25                      ???                                      ; 未知命令  
  
  
00373773    308B 1F304000             XOR BYTE PTR DS:\[EBX+40301F\],CL  
00373779    0300                      ADD EAX,DWORD PTR DS:\[EAX\]  
0037377B    E8 00000000               CALL 00373780  
00373780    2D BA0000FE               SUB EAX,FE0000BA  
00373785    FFD0                      CALL NEAR EAX    ; kernel32.GetTickCount////获取当前时间标识。  
00373787    8B9B AA1D4000             MOV EBX,DWORD PTR DS:\[EBX+401DAA\]////取出先前保存的时间标识。  
0037378D    60                        PUSHAD  
0037378E    B9 04000000               MOV ECX,4  
00373793    E8 1F000000               CALL 003737B7  
00373798  ^ EB FA                     JMP SHORT 00373794  
0037379A    E8 16000000               CALL 003737B5  
0037379F  - E9 EBF80000               JMP 0038308F  
003737A4    58                        POP EAX  
003737A5    EB 09                     JMP SHORT 003737B0  
003737A7    0F25                      ???               ; 未知命令  
  
003737B9    D6                        SALC  
003737BA    61                        POPAD  
003737BB    2BC3                      SUB EAX,EBX/////两个时间标识相减了！  
003737BD    3D 80000000               CMP EAX,80  
003737C2    7F 15                     JG SHORT 003737D9////大于 80 则跳，跳则解码错误。这里不能跳！  
003737C4    5B                        POP EBX  
003737C5    EB 01                     JMP SHORT 003737C8  
003737C7    E8 EB01E858               CALL 591F39B7  
003737CC    EB 01                     JMP SHORT 003737CF  
003737CE    E8 61EB01E8               CALL E8392334  
003737D3    61                        POPAD  
003737D4    BB 3A240000               MOV EBX,243A  
003737D9    833C2B 00                 CMP DWORD PTR DS:\[EBX+EBP\],0  
003737DD    0F84 AC000000             JE 0037388F  
003737E3    53                        PUSH EBX  
003737E4    6A 04                     PUSH 4  
  
00373B59    8907            MOV DWORD PTR DS:\[EDI\],EAX     ; SHELL32.DragFinish  
00373B5B    5A              POP EDX  
00373B5C    0FB642 FF       MOVZX EAX,BYTE PTR DS:\[EDX-1\]  
00373B60    03D0            ADD EDX,EAX  
00373B62    42              INC EDX  
00373B63    60              PUSHAD  
00373B64    E8 03000000     CALL 00373B6C  
  
00373BB8    61              POPAD  
00373BB9    25 FFFFFF7F     AND EAX,7FFFFFFF////开始处理 IAT 了。从这里开始 PATH！修改为：  
  
00373BB9   /E9 42230000     JMP 00375F00  
  
00373BBE    8BDE            MOV EBX,ESI  
00373BC0    2BD8            SUB EBX,EAX  
00373BC2    8958 FC         MOV DWORD PTR DS:\[EAX-4\],EBX  
00373BC5    83C7 08         ADD EDI,8  
00373BC8    60              PUSHAD  
00373BC9    E8 16000000     CALL 00373BE4  
00373BCE    8B5C24 0C       MOV EBX,DWORD PTR SS:\[ESP+C\]  
00373BD2    8BA3 C4000000   MOV ESP,DWORD PTR DS:\[EBX+C4\]  
00373BD8    64:8F05 0000000>POP DWORD PTR FS:\[0\]  
00373BDF    83C4 04         ADD ESP,4  
  
375F00 处的 PATH 代码：  
  
  
00375F00    807F 03 00      CMP BYTE PTR DS:\[EDI+3\],0////比较最高位是否为 0。  
00375F04    75 11           JNZ SHORT 00375F17  
00375F06    8B5F 04         MOV EBX,DWORD PTR DS:\[EDI+4\]////正确的函数地址指针移入 EBX。  
00375F09    66:C740 FA FF15 MOV WORD PTR DS:\[EAX-6\],15FF////加入指令 CALL\[XXXXXXXX\]  
00375F0F    8958 FC         MOV DWORD PTR DS:\[EAX-4\],EBX  
00375F12  ^ E9 AEDCFFFF     JMP 00373BC5////跳回继续。  
00375F17    25 FFFFFF7F     AND EAX,7FFFFFFF////取消最高位。  
00375F1C    8B5F 04         MOV EBX,DWORD PTR DS:\[EDI+4\]  
00375F1F    66:C740 FA FF25 MOV WORD PTR DS:\[EAX-6\],25FF////加入指令 JMP\[XXXXXXXX\]  
00375F25    8958 FC         MOV DWORD PTR DS:\[EAX-4\],EBX  
00375F28  ^ E9 98DCFFFF     JMP 00373BC5////跳回继续。  
00375F2D    90              NOP  
  
二进制：  
  
80 7F 03 00 75 11 8B 5F 04 66 C7 40 FA FF 15 89 58 FC E9 AE DC FF FF 25 FF FF FF 7F 8B 5F 04 66 C7 40 FA FF 25 89 58 FC E9 98 DC FF FF 90  
  
这里就是表：  
  
00374A5A  65 79 41 00 00 00 00 00  eyA.....  
00374A62  5C 1A 40 00 F8 63 40 00  \\@.鴆@.  
00374A6A  23 15 40 00 FC 63 40 00  #@.點@.  
。。。。。。。  
  
00375092  BB 22 40 00 88 64 40 00  ?@.坉@.  
0037509A  FA 22 40 00 88 64 40 00  ?@.坉@.  
。。。。。。  
  
00375642  B5 1E 40 00 50 63 40 00  ?@.Pc@.  
0037564A  4C 44 40 00 50 63 40 00  LD@.Pc@.  
00375652  5A 44 40 00 50 63 40 00  ZD@.Pc@.  
0037565A  A4 44 40 00 50 63 40 00  @.Pc@.  
00375662  D2 26 40 00 54 63 40 00  ?@.Tc@.  
0037566A  CE 4F 40 80 08 65 40 00  蜲@€e@.///  
00375672  C8 4F 40 80 0C 65 40 00  萇@€.e@.///  
0037567A  C2 4F 40 80 10 65 40 00  翺@€e@.///  
00375682  BC 4F 40 80 14 65 40 00  糘@€e@.///首位为 80 的对应为 25FF。（即 JMP\[XXXXXXXX\]的形式）  
0037568A  B6 4F 40 80 18 65 40 00  禣@€e@.///  
00375692  B0 4F 40 80 1C 65 40 00  癘@€e@.///  
0037569A  D4 4F 40 80 20 65 40 00  設@€ e@.///  
003756A2  BC 24 40 00 E4 62 40 00  ?@.鋌@.  
003756AA  0A 25 40 00 E4 62 40 00  .%@.鋌@.  
003756B2  BF 26 40 00 E8 62 40 00  ?@.鑒@.  
003756BA  5F 24 40 00 EC 62 40 00  \_$@.靊@.  
003756C2  84 24 40 00 EC 62 40 00  ?@.靊@.  
003756CA  F3 26 40 00 F0 62 40 00  ?@.餬@.  
003756D2  42 25 40 00 F4 62 40 00  B%@.鬮@.  
  
  
继续跟踪来到：  
  
  
00373FB3    61              POPAD  
00373FB4    8997 B8000000   MOV DWORD PTR DS:\[EDI+B8\],EDX////EDX＝4010CC，异常回调处就是入口！  
00373FBA    5A              POP EDX  
00373FBB    33C0            XOR EAX,EAX  
00373FBD    8947 04         MOV DWORD PTR DS:\[EDI+4\],EAX      ///  
00373FC0    2147 08         AND DWORD PTR DS:\[EDI+8\],EAX      ///  
00373FC3    2147 0C         AND DWORD PTR DS:\[EDI+C\],EAX      ///调试寄存器清零！  
00373FC6    2147 10         AND DWORD PTR DS:\[EDI+10\],EAX     ///  
00373FC9    8167 14 F00FFFF>AND DWORD PTR DS:\[EDI+14\],FFFF0FF0///  
00373FD0    2147 18         AND DWORD PTR DS:\[EDI+18\],EAX     ///  
00373FD3    C707 17000100   MOV DWORD PTR DS:\[EDI\],10017  
00373FD9    B8 00000000     MOV EAX,0  
00373FDE    5F              POP EDI  
00373FDF    5E              POP ESI  
00373FE0    5B              POP EBX  
00373FE1    C9              LEAVE  
00373FE2    C3              RETN  
  
直接到 4010CC 的一个字节处下内存访问断点，断下几次后就会停在 4010CC 处：  
  
004010CC    55              PUSH EBP////入口！  
004010CD    8BEC            MOV EBP,ESP  
004010CF    83EC 44         SUB ESP,44  
004010D2    56              PUSH ESI  
004010D3    FF15 E4634000   CALL NEAR DWORD PTR DS:\[4063E4\]          ; kernel32.GetCommandLineA  
004010D9    8BF0            MOV ESI,EAX  
004010DB    8A00            MOV AL,BYTE PTR DS:\[EAX\]  
004010DD    3C 22           CMP AL,22  
004010DF    75 1B           JNZ SHORT notepad.004010FC  
004010E1    56              PUSH ESI  
004010E2    FF15 F4644000   CALL NEAR DWORD PTR DS:\[4064F4\]          ; User32.CharNextA  
004010E8    8BF0            MOV ESI,EAX  
004010EA    8A00            MOV AL,BYTE PTR DS:\[EAX\]  
004010EC    84C0            TEST AL,AL  
004010EE    74 04           JE SHORT notepad.004010F4  
004010F0    3C 22           CMP AL,22  
004010F2  ^ 75 ED           JNZ SHORT notepad.004010E1  
004010F4    803E 22         CMP BYTE PTR DS:\[ESI\],22  
004010F7    75 15           JNZ SHORT notepad.0040110E  
004010F9    46              INC ESI  
004010FA    EB 12           JMP SHORT notepad.0040110E  
004010FC    3C 20           CMP AL,20  
004010FE    7E 0E           JLE SHORT notepad.0040110E  
00401100    56              PUSH ESI  
00401101    FF15 F4644000   CALL NEAR DWORD PTR DS:\[4064F4\]          ; User32.CharNextA  
00401107    8038 20         CMP BYTE PTR DS:\[EAX\],20  
0040110A    8BF0            MOV ESI,EAX  
0040110C  ^ 7F F2           JG SHORT notepad.00401100  
0040110E    803E 00         CMP BYTE PTR DS:\[ESI\],0  
  
在入口处 DUMP，然后用 ImportREC 修复 IAT。OK 完工！  
  
  
  
\--------------------------------------------------------------------------------  
  
【版权声明】本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢! 

把 WINXP　SP1 下的 OS 也发一下吧，由于偷懒没有考虑其他系统的兼容性。  
   ////////////////////////////////////////////////////  
  //duzaizhe 的旧 hying 修改壳脱壳脚本，适用于 winxp sp1//  
 //        csjwaman\[DFCG\]  于 2005 年 2 月 19 日         //  
////////////////////////////////////////////////////  
  
  
  
dbh  
msg "请忽略所有异常!"  
gpa "VirtualAlloc","kernel32"  
bp $RESULT  
run  
bc $RESULT  
rtu  
bphws 372226,"x"  
run  
bphwc 372226  
mov \[372226\],#9090#  
mov \[3724F9\],#555555555555555555555555#  
go 3737C2  
mov \[372226\],#300B#  
mov !SF,1  
mov \[373BB9\],#E942230000#  
mov \[375F00\],#807F030075118B5F0466C740FAFF158958FCE9AEDCFFFF25FFFFFF7F8B5F0466C740FAFF258958FCE998DCFFFF90#  
bprm 4010CC,1  
run  
run  
bpmc  
msg "现在可以 DUMP 了，然后用 ImportREC 修复 IAT 即可!"  
ret 

你这么比较高位怎么行。  
  
mov eax, 值  
cdq  
test edx, edx  
jz ...  
这样，建议参考原壳代码 

我写得比较垃圾：  
  
; ------------------------------------------------------------------------  
; Rebuild imports, my hardest work brrrrr  
  
@@RebuildNewImports:  
      pushad  
      mov  ecx, ImportsProtectedFlag  
      test  ecx, ecx  
      mov  esi, MutatedImports  
      jz  \_\_xxxit\_nor  
        
  
; let's rebuild -\_0  
  
  
;FristThunk  00 01 02 03          ; we must patch this  
;LengthOfDllName  04  
;DllName    05 .. .. ..  
;  
;Null    00  
;NumberOfThunks  01 02 03 04  
;FakeThunkx  05 - Flag 0=Index, 1=String  
                ;06 07 08 09 index  
    ;06 xx xx xx string  
  
      mov  edi, LoaderStart    ; use old loader space for new IMPORT TABLE  
      add  edi, LoaderSize      ; skip loader,rsrc,othershit  
  
; state 1 - build stringz & fake-thunkx  
; recreate all of above, write new ptr in old  pos  
      push  esi  
\_\_x\_s\_1:  
      mov  eax, \[esi\]  
      test  eax, eax  
      jz  \_\_r\_end1  
  
      movzx  ecx, byte ptr\[esi+4\]    ; name len  
      inc  ecx  
  
      mov  eax, edi  
      sub  eax, FileBase      ; rva  
      add  esi, 4+1  
      push  esi  
      rep\_movsb  
      mov  ecx, esi  
      pop  esi  
      mov  \[esi\], eax  
        
      xchg  esi, ecx  
  
      lodsd  
      xchg  ecx, eax      ; # of thunkx  
  
\_\_r\_1\_big\_loop:  
      push  ecx  
  
      lodsb  
      test  al, al  
      jz  \_\_bd\_thunkx\_i      ; imported by index  
      mov  ecx, edi  
      sub  ecx, FileBase      ; 2 rva  
      mov  edx, esi  
      xor  eax, eax  
      stosw          ; no hint  
      @copysz  
      mov  \[edx-1\], ecx  
      jmp  \_\_r\_1\_big\_out  
  
\_\_bd\_thunkx\_i:  
      lodsd  
      or  eax, 80000000h      ; set MSB flag  
      mov  \[esi-5\], eax  
\_\_r\_1\_big\_out:  
      pop  ecx  
      loop  \_\_r\_1\_big\_loop  
  
      jmp  \_\_x\_s\_1  
\_\_r\_end1:  
      pop  esi  
  
      ;int  3  
      nop  
; state 2 - modify IID thunks array  
  
      push  esi  
  
\_\_x\_s\_2:  
      mov  eax, \[esi\]  
      test  eax, eax  
      jz  \_\_r\_end2  
      lodsd          ; 1st thunk array  
      xchg  ebx, eax  
      add  ebx, FileBase  
  
      movzx  ecx, byte ptr\[esi\]  
      inc  ecx        ; self  
      inc  ecx  
      add  esi, ecx  
  
      lodsd          ; # of thunkx  
      xchg  ecx, eax  
  
      mov  edx, esi      ; to put ptr here l8r  
        
      push  ebx  
  
\_\_make\_1st\_thunkx:  
      lodsd  
      mov  \[ebx\], eax  
      @endsz  
      add  ebx, 4  
      loop  \_\_make\_1st\_thunkx  
  
      pop  ebx  
  
      mov  \[edx\], ebx  
      mov  \[edx+4\], esi      ; we can place here next time  
  
  
      jmp  \_\_x\_s\_2  
\_\_r\_end2:  
  
      pop  esi  
  
        
; state 3 - final build IID structs , our imports back!  
  
      push  edi        ; IMPORTANT! save new Import Table VA  
  
      push  esi  
\_\_x\_s\_3:  
      mov  eax, \[esi\]  
      test  eax, eax  
      jz  \_\_r\_end3  
      lodsd  
      movzx  ecx, byte ptr\[esi\]  
      inc  esi  
  
      lodsd          ; name rva  
      xchg  edx, eax  
  
      sub  ecx, 4-1  
      add  esi, ecx        
        
      lodsd  
      lodsd          ; first thunk  
      sub  eax, FileBase      ; rva  
      xchg  ebx, eax  
  
      mov  ecx, edi      ; our NEW IMPORT ADDRESS  
  
      xor  eax, eax  
      stosd  
      dec  eax  
      stosd  
      stosd  
      xchg  eax, edx  
      stosd          ; name  
      xchg  eax, ebx  
      stosd          ; 1st thunk  
  
        
      lodsd          ; get done ptr  
      xchg  esi, eax  
      jmp  \_\_x\_s\_3  
  
\_\_r\_end3:  
      pop  esi  
  
; build a null IID for end  
      push  5  
      pop  ecx  
      xor  eax, eax  
\_\_bd\_null\_iid:  
      stosd  
      loop  \_\_bd\_null\_iid  
  
      ;int  3  
      ;nop  
; bound IT to target file  
  
      pop  edi  
      xchg  esi, edi  
      sub  esi, FileBase  
      jmp  \_\_xxxit\_do  
  
\_\_xxxit\_nor:  
      sub  esi, RealSymbiontStart  
\_\_xxxit\_do:  
      mov  edi, NtHeaderPtr      
      mov  \[edi+pe\_struc.pe\_importtablerva\], esi  
      xor  ecx, ecx  
      inc  ecx  
      mov  \[edi+pe\_struc.pe\_importtablesize\], ecx  
      jmp  \_\_xxxit\_exit  
  
\_\_xxxit\_exit:  
  
      popad  
      retn 

 4dd466d0e1e0cc123afbe504da2462ca