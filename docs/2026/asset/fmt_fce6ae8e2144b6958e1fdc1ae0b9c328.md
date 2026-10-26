
```asm
00592EDB    2041 43         AND BYTE PTR DS:[ECX+43],AL
00592EDE    50              PUSH EAX
00592EDF    72 6F           JB SHORT 00592F50
00592EE1    74 65           JE SHORT 00592F48
00592EE3    637400 73       ARPL WORD PTR DS:[EAX+EAX+73],SI
00592EE7    74 65           JE SHORT 00592F4E
00592EE9    72 65           JB SHORT 00592F50
00592EEB    64:2041 43      AND BYTE PTR FS:[ECX+43],AL
00592EEF    50              PUSH EAX
00592EF0    72 6F           JB SHORT 00592F61
00592EF2    74 65           JE SHORT 00592F59
00592EF4    637421 00       ARPL WORD PTR DS:[ECX],SI
00592EF8 >  6A 00           PUSH 0                                   ; GOTO END改为jmp [addr] 的方式执行到fake OEP地址
00592EFA    83C4 10         ADD ESP,10
00592EFD    90              NOP
......
005930A9    E8 5290FFFF     CALL <sub\_GetEBP >
005930AE    8B85 4B814100   MOV EAX,DWORD PTR SS:[EBP+41814B]
005930B4    0385 46F84000   ADD EAX,DWORD PTR SS:[EBP+40F846]
005930BA    8985 4B814100   MOV DWORD PTR SS:[EBP+41814B],EAX        ; 要去执行的地址
005930C0    E8 3B90FFFF     CALL <sub\_GetEBP >
005930C5    C685 08814100 E>MOV BYTE PTR SS:[EBP+418108],0E8
005930CC    E8 2F90FFFF     CALL <sub\_GetEBP >
005930D1    C785 09814100 F>MOV DWORD PTR SS:[EBP+418109],25FF       ; 改成jmp [address]的方式
005930DB    8D85 4B814100   LEA EAX,DWORD PTR SS:[EBP+41814B]
005930E1    8985 0B814100   MOV DWORD PTR SS:[EBP+41810B],EAX
005930E7    E8 1490FFFF     CALL <sub\_GetEBP >
005930EC    8DBD B87E4100   LEA EDI,DWORD PTR SS:[EBP+417EB8]
005930F2    8D8D FD804100   LEA ECX,DWORD PTR SS:[EBP+4180FD]
005930F8    2BCF            SUB ECX,EDI
005930FA    C1E9 02         SHR ECX,2
005930FD >  E8 CC87FFFF     CALL <Get\_RND\_VALUE>                     ; Fill junk code
00593102    AB              STOS DWORD PTR ES:[EDI]
00593103  ^ E2 F8           LOOPD SHORT <sub\_fill\_Junk>              ; 循环填上垃圾代码，
00593105    61              POPAD
00593106    EB 01           JMP SHORT 00593109
00593108    90              NOP
00593109  - FF25 4B315900   JMP DWORD PTR DS:[59314B]                ; JMP to FAKE OEP
```

OK，分析篇到此结束，后面的脱壳篇基本上是体力活了。下次会脱出美女来的:-P。

再贴上acprotect 1.41的部分东西：

- `[EBP+40F76C]`：注册标志1，这里为1则表示没有注册提示
- `[EBP+414C63]`：注册标志2，这里为0则表示没有注册提示
- `[EBP+418520]`：`GetModuleHandleA`
- `[EBP+418524]`：`LoadLibraryA`
- `[EBP+402012]`：`hModule==kernel32.dll`
- `[EBP+41851C]`：`GetProcAddress`
- `[EBP+40FCFF]`：保存壳API的开始地址
- `[EBP+41851C]`：壳输入表起始位置
- `[EBP+40F907]`：程序输入表起始位置
- `[EBP+415AF7]`：重定位表的RVA
- `[EBP+415AFB]`：重定位表的SIZE
- `[EBP+401CCD]`：保存用了SDK地址大小列表的起始地址
- `[EBP+401B3D]`：保存用了SDK地址列表的起始地址
- `[EBP+4152A5]`：如果值为`C3`表示已经处理完输入表或输入表为空
- `[EBP+40F76D]`：保存可用次数或天数
- `[EBP+40FD5F]`：`UnhandledExceptionFilter`
- `[EBP+41814B]`：Stolen code后执行到该地址，记录RVA
- `[EBP+41816F]`：保存正确的CRC值

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
