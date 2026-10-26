
```
0128704D Main     MOV EBX,442944                            ; EBX=00442944

01287052 Main     ADD EBX,EBP                               ; EBX=01287000

01287054 Main     SUB EBX,DWORD PTR SS:[EBP+442971]         ; EBX=01270000

0128705A Main     CMP DWORD PTR SS:[EBP+4430D8],0 ==================〉注意这里

01287061 Main     MOV DWORD PTR SS:[EBP+442E2F],EBX

01287067 Main     JNZ 012875AB  ===============================〉外壳处理完毕后会从这里跳转到最后处理部分！

0128706D Main     LEA EAX,DWORD PTR SS:[EBP+4430E0]         ; EAX=0128779C

01287073 Main     PUSH EAX

01287074 Main     CALL DWORD PTR SS:[EBP+4431EC]            ; EAX=77E60000, ECX=0012FFE0, EDX=77FCD348

0128707A Main     MOV DWORD PTR SS:[EBP+4430DC],EAX

01287080 Main     MOV EDI,EAX                               ; EDI=77E60000

01287082 Main     LEA EBX,DWORD PTR SS:[EBP+4430ED]         ; EBX=012877A9

01287088 Main     PUSH EBX

01287089 Main     PUSH EAX

0128708A Main     CALL DWORD PTR SS:[EBP+4431E8]            ; EAX=77E70EE2

01287090 Main     MOV DWORD PTR SS:[EBP+442979],EAX

01287096 Main     LEA EBX,DWORD PTR SS:[EBP+4430FA]         ; EBX=012877B6

0128709C Main     PUSH EBX

0128709D Main     PUSH EDI

0128709E Main     CALL DWORD PTR SS:[EBP+4431E8]            ; EAX=77E6D002

012870A4 Main     MOV DWORD PTR SS:[EBP+44297D],EAX

012870AA Main     MOV EAX,DWORD PTR SS:[EBP+442E2F]         ; EAX=01270000

012870B0 Main     MOV DWORD PTR SS:[EBP+4430D8],EAX

012870B6 Main     PUSH 4

012870B8 Main     PUSH 1000

012870BD Main     PUSH 546

012870C2 Main     PUSH 0

012870C4 Main     CALL DWORD PTR SS:[EBP+442979]            ; EAX=01290000, EDX=FFFFFFFF

012870CA Main     MOV DWORD PTR SS:[EBP+442975],EAX

012870D0 Main     LEA EBX,DWORD PTR SS:[EBP+442A45]         ; EBX=01287101

012870D6 Main     PUSH EAX

012870D7 Main     PUSH EBX

012870D8 Main     CALL 01287651                             ; EAX=00000546

012870DD Main     MOV ECX,EAX                               ; ECX=00000546

012870DF Main     LEA EDI,DWORD PTR SS:[EBP+442A45]         ; EDI=01287101

012870E5 Main     MOV ESI,DWORD PTR SS:[EBP+442975]         ; ESI=01290000

012870EB Main     REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]; ECX=00000000, ESI=01290546, EDI=01287647

012870ED Main     MOV EAX,DWORD PTR SS:[EBP+442975]         ; EAX=01290000

012870F3 Main     PUSH 8000

012870F8 Main     PUSH 0

012870FA Main     PUSH EAX

012870FB Main     CALL DWORD PTR SS:[EBP+44297D]            ; EAX=00000001, ECX=00010101

01287101 Main     LEA EAX,DWORD PTR SS:[EBP+442C51]         ; EAX=0128730D
```

（4）最后部分

```
=========================================================================================================================

012875AB   8B85 652A4400    MOV EAX,DWORD PTR SS:[EBP+442A65]

012875B1   50               PUSH EAX

012875B2   0385 D8304400    ADD EAX,DWORD PTR SS:[EBP+4430D8]

012875B8   5B               POP EBX

012875B9   0BDB             OR EBX,EBX

012875BB   8985 112F4400    MOV DWORD PTR SS:[EBP+442F11],EAX

012875C1   61               POPAD ==========================〉恢复入口点

012875C2   75 08            JNZ SHORT 012875CC

012875C4   B8 01000000      MOV EAX,1

012875C9   C2 0C00          RETN 0C

012875CC   68 A8132801      PUSH 12813A8 解压后程序的入口点！

012875D1   C3               RETN

==========================================================================================================================
```

到达入口点后剩下的工作！就是dump文件修复iat！

这个外壳的工作流程: 1开始部分代码解码生成========>2 aspack解码程序通过这个再解码生成===========〉3原始的程序！

好了这个外壳工作流程到此分析完了！

在这我提一下c32asm这个PE反编译软件！

这是一款有中国人自己开发的PE反编译工具

软件作者是：pll621

主要功能有：

支持解析符号，支持反汇编直接HEX编辑功能不过这个功能如果能像HIEW或OLLYDBG一样可以用汇编直接更改程序HEX代码的话就好了！支持数据解释器 支持多语言 支持插件 界面非常漂亮！

大老 写于凌晨 5:15 2003-6-26
