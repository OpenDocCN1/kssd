
```
0043C081    5F                   POP EDI
0043C082    36:C702 1C00000>     MOV DWORD PTR SS:[EDX],1C
0043C089    8BD5                 MOV EDX,EBP
0043C08B    81C2 F9EB4000        ADD EDX,MyOpenGL.0040EBF9
0043C091    8B02                 MOV EAX,DWORD PTR DS:[EDX]
0043C093    50                   PUSH EAX
0043C094    6A 04                PUSH 4
0043C096    E8 14F5FFFF          CALL MyOpenGL.0043B5AF      kernel32.CreateToolhelp32Snapshot
0043C09B    8BF0                 MOV ESI,EAX
0043C09D    8BC5                 MOV EAX,EBP
0043C09F    05 7B0C4100          ADD EAX,MyOpenGL.00410C7B
0043C0A4    50                   PUSH EAX
0043C0A5    56                   PUSH ESI
0043C0A6    E8 2EF5FFFF          CALL MyOpenGL.0043B5D9      kernel32.Thread32First
0043C0AB    85C0                 TEST EAX,EAX
0043C0AD    0F84 80000000        JE MyOpenGL.0043C133
0043C0B3    8BD5                 MOV EDX,EBP
0043C0B5    81C2 7B0C4100        ADD EDX,MyOpenGL.00410C7B
0043C0BB    8D0A                 LEA ECX,DWORD PTR DS:[EDX]
0043C0BD    51                   PUSH ECX
0043C0BE    56                   PUSH ESI
0043C0BF    E8 1BF5FFFF          CALL MyOpenGL.0043B5DF      kernel32.Thread32Next
0043C0C4    85C0                 TEST EAX,EAX
0043C0C6    74 6B                JE SHORT MyOpenGL.0043C133
0043C0C8    56                   PUSH ESI
0043C0C9    8BD5                 MOV EDX,EBP
0043C0CB    81C2 7B0C4100        ADD EDX,MyOpenGL.00410C7B
0043C0D1    8B4A 0C              MOV ECX,DWORD PTR DS:[EDX+C]
0043C0D4    8BD5                 MOV EDX,EBP
0043C0D6    81C2 F9EB4000        ADD EDX,MyOpenGL.0040EBF9
0043C0DC    8B02                 MOV EAX,DWORD PTR DS:[EDX]
0043C0DE    3BC8                 CMP ECX,EAX
0043C0E0    75 3B                JNZ SHORT MyOpenGL.0043C11D
0043C0E2    8BD5                 MOV EDX,EBP
0043C0E4    81C2 7B0C4100        ADD EDX,MyOpenGL.00410C7B
0043C0EA    8B42 08              MOV EAX,DWORD PTR DS:[EDX+8]
0043C0ED    50                   PUSH EAX
0043C0EE    6A 00                PUSH 0
0043C0F0    6A 02                PUSH 2
0043C0F2    E8 EEF4FFFF          CALL MyOpenGL.0043B5E5       kernel32.OpenThread
0043C0F7    85C0                 TEST EAX,EAX
0043C0F9    74 22                JE SHORT MyOpenGL.0043C11D
0043C0FB    8BC8                 MOV ECX,EAX
0043C0FD    85DB                 TEST EBX,EBX
0043C0FF    74 0C                JE SHORT MyOpenGL.0043C10D
0043C101    51                   PUSH ECX
0043C102    8BC1                 MOV EAX,ECX
0043C104    50                   PUSH EAX
0043C105    E8 1DF5FFFF          CALL MyOpenGL.0043B627        kernel32.SuspendThread
0043C10A    59                   POP ECX
0043C10B    EB 0A                JMP SHORT MyOpenGL.0043C117
0043C10D    51                   PUSH ECX
0043C10E    8BC1                 MOV EAX,ECX
0043C110    50                   PUSH EAX
0043C111    E8 17F5FFFF          CALL MyOpenGL.0043B62D         kernel32.ResumeThread
0043C116    59                   POP ECX
0043C117    51                   PUSH ECX
0043C118    E8 86F4FFFF          CALL MyOpenGL.0043B5A3         kernel32.CloseHandle
0043C11D    5E                   POP ESI
0043C11E    8BD5                 MOV EDX,EBP
0043C120    81C2 7B0C4100        ADD EDX,MyOpenGL.00410C7B
0043C126    8D0A                 LEA ECX,DWORD PTR DS:[EDX]
0043C128    51                   PUSH ECX
0043C129    56                   PUSH ESI
0043C12A    E8 B0F4FFFF          CALL MyOpenGL.0043B5DF          kernel32.Thread32Next
0043C12F    85C0                 TEST EAX,EAX
0043C131   ^ 75 95                JNZ SHORT MyOpenGL.0043C0C8
0043C133    B8 55555555          MOV EAX,55555555
0043C138    8BD5                 MOV EDX,EBP
0043C13A    81C2 F9EB4000        ADD EDX,MyOpenGL.0040EBF9
0043C140    8902                 MOV DWORD PTR DS:[EDX],EAX
0043C142    C3                   RETN
```

多处用到的校验 `call`: //可以检测你下的一次性断点，也可以阻止你修改代码

```
0043BF57    55                   PUSH EBP
0043BF58    8BEC                 MOV EBP,ESP
0043BF5A    53                   PUSH EBX
0043BF5B    51                   PUSH ECX
0043BF5C    52                   PUSH EDX
0043BF5D    56                   PUSH ESI
0043BF5E    57                   PUSH EDI
0043BF5F    36:8B45 08           MOV EAX,DWORD PTR SS:[EBP+8]
0043BF63    36:8B4D 0C           MOV ECX,DWORD PTR SS:[EBP+C]
0043BF67    8BF8                 MOV EDI,EAX
0043BF69    33C0                 XOR EAX,EAX
0043BF6B    33DB                 XOR EBX,EBX
0043BF6D    33D2                 XOR EDX,EDX
0043BF6F    8A07                 MOV AL,BYTE PTR DS:[EDI]
0043BF71    F7E2                 MUL EDX
0043BF73    03D8                 ADD EBX,EAX
0043BF75    42                   INC EDX
0043BF76    47                   INC EDI
0043BF77   ^ E2 F6              LOOPD SHORT MyOpenGL.0043BF6F
0043BF79    93                   XCHG EAX,EBX
0043BF7A    5F                   POP EDI
0043BF7B    5E                   POP ESI
0043BF7C    5A                   POP EDX
0043BF7D    59                   POP ECX
0043BF7E    5B                   POP EBX
0043BF7F    8BE5                 MOV ESP,EBP
0043BF81    5D                   POP EBP
0043BF82    C3                   RETN
```

附上脱壳后的程序及源代码，最后感谢海浪轻风前辈给我一次练手的机会！

### 上传的附件

| ![图片](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | 程序及 `delphi` 源代码.rar |
| :--- | :--- |
| ![图片](img/12f2a5fbf8d3fd9a8bf145533ecd3281.jpg) | 投稿.txt |
