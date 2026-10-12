
```
0045781F   /$   55              PUSH EBP
00457820   |.   8BEC            MOV EBP,ESP
00457822   |.   83EC 10         SUB ESP,10
00457825   |.   894D F0         MOV DWORD PTR SS:\[EBP-10\],ECX
00457828   |.   68 94DC4700     PUSH NewsjRpg.0047DC94                     ; /r
0045782D   |.   8B45 0C         MOV EAX,DWORD PTR SS:\[EBP+C\]             ; |
00457830   |.   50              PUSH EAX                                    ; |path
00457831   |.   FF15 40E74600   CALL DWORD PTR DS:\[<&MSVCRT.fopen>\]      ; \\fopen
00457837   |.   83C4 08         ADD ESP,8
0045783A   |.   8945 F8         MOV DWORD PTR SS:\[EBP-8\],EAX
0045783D   |.   837D F8 00      CMP DWORD PTR SS:\[EBP-8\],0
00457841   |.   75 1D           JNZ SHORT NewsjRpg.00457860
00457843   |.   68 98DC4700     PUSH NewsjRpg.0047DC98                     ; /r
00457848   |.   8B4D 0C         MOV ECX,DWORD PTR SS:\[EBP+C\]             ; |
0045784B   |.   51              PUSH ECX                                    ; |path
0045784C   |.   FF15 40E74600   CALL DWORD PTR DS:\[<&MSVCRT.fopen>\]      ; \\fopen
00457852   |.   83C4 08         ADD ESP,8
00457855   |.   85C0            TEST EAX,EAX
00457857   |.   75 07           JNZ SHORT NewsjRpg.00457860
00457859   |.   33C0            XOR EAX,EAX
0045785B   |.   E9 D4000000     JMP NewsjRpg.00457934
00457860   |>   8B55 08         MOV EDX,DWORD PTR SS:\[EBP+8\]
00457863   |.   52              PUSH EDX
00457864   |.   68 9CDC4700     PUSH NewsjRpg.0047DC9C                     ; |%d;
00457869   |.   8B45 F8         MOV EAX,DWORD PTR SS:\[EBP-8\]             ; |
0045786C   |.   50              PUSH EAX                                    ; |stream
0045786D   |.   FF15 B0E64600   CALL DWORD PTR DS:\[<&MSVCRT.fscanf>\]     ; \\fscanf
00457873   |.   83C4 0C         ADD ESP,0C
00457876   |.   8B4D 08         MOV ECX,DWORD PTR SS:\[EBP+8\]
00457879   |.   83C1 04         ADD ECX,4
0045787C   |.   51              PUSH ECX
0045787D   |.   68 A0DC4700     PUSH NewsjRpg.0047DCA0                     ; |%d;
00457882   |.   8B55 F8         MOV EDX,DWORD PTR SS:\[EBP-8\]             ; |
00457885   |.   52              PUSH EDX                                    ; |stream
00457886   |.   FF15 B0E64600   CALL DWORD PTR DS:\[<&MSVCRT.fscanf>\]     ; \\fscanf
0045788C   |.   83C4 0C         ADD ESP,0C
0045788F   |.   8B45 08         MOV EAX,DWORD PTR SS:\[EBP+8\]
00457892   |.   83C0 08         ADD EAX,8
00457895   |.   50              PUSH EAX
00457896   |.   68 A4DC4700     PUSH NewsjRpg.0047DCA4                     ; |%d;
0045789B   |.   8B4D F8         MOV ECX,DWORD PTR SS:\[EBP-8\]             ; |
0045789E   |.   51              PUSH ECX                                    ; |stream
0045789F   |.   FF15 B0E64600   CALL DWORD PTR DS:\[<&MSVCRT.fscanf>\]     ; \\fscanf
004578A5   |.   83C4 0C         ADD ESP,0C
004578A8   |.   C745 FC 00000>  MOV DWORD PTR SS:\[EBP-4\],0
004578AF   |.   EB 09           JMP SHORT NewsjRpg.004578BA
004578B1   |>   8B55 FC         /MOV EDX,DWORD PTR SS:\[EBP-4\]
004578B4   |.   83C2 01         |ADD EDX,1
004578B7   |.   8955 FC         |MOV DWORD PTR SS:\[EBP-4\],EDX
004578BA   |>   837D FC 06      CMP DWORD PTR SS:\[EBP-4\],6               ;大循环 6 次
004578BE   |.   7D 62           |JGE SHORT NewsjRpg.00457922
004578C0   |.   8B45 FC         |MOV EAX,DWORD PTR SS:\[EBP-4\]
004578C3   |.   8B4D 08         |MOV ECX,DWORD PTR SS:\[EBP+8\]
004578C6   |.   8D5481 0C       |LEA EDX,DWORD PTR DS:\[ECX+EAX\*4+C\]
004578CA   |.   52              |PUSH EDX
004578CB   |.   68 A8DC4700     |PUSH NewsjRpg.0047DCA8                    ; |%d;
004578D0   |.   8B45 F8         |MOV EAX,DWORD PTR SS:\[EBP-8\]           ; |
004578D3   |.   50              |PUSH EAX                                  ; |stream
004578D4   |.   FF15 B0E64600   |CALL DWORD PTR DS:\[<&MSVCRT.fscanf>\]    ; \\fscanf
004578DA   |.   83C4 0C         |ADD ESP,0C
004578DD   |.   C745 F4 00000>  |MOV DWORD PTR SS:\[EBP-C\],0
004578E4   |.   EB 09           |JMP SHORT NewsjRpg.004578EF
004578E6   |>   8B4D F4         |/MOV ECX,DWORD PTR SS:\[EBP-C\]
004578E9   |.   83C1 01         ||ADD ECX,1
004578EC   |.   894D F4         ||MOV DWORD PTR SS:\[EBP-C\],ECX
004578EF   |>   837D F4 3C      | CMP DWORD PTR SS:\[EBP-C\],3C            ；小循环 3C=60 次
004578F3   |.   7D 2B           ||JGE SHORT NewsjRpg.00457920
004578F5   |.   8B55 FC         ||MOV EDX,DWORD PTR SS:\[EBP-4\]
004578F8   |.   69D2 F0000000   ||IMUL EDX,EDX,0F0
004578FE   |.   8B45 08         ||MOV EAX,DWORD PTR SS:\[EBP+8\]
00457901   |.   8D4C10 24       ||LEA ECX,DWORD PTR DS:\[EAX+EDX+24\]
00457905   |.   8B55 F4         ||MOV EDX,DWORD PTR SS:\[EBP-C\]
00457908   |.   8D0491          ||LEA EAX,DWORD PTR DS:\[ECX+EDX\*4\]
0045790B   |.   50              ||PUSH EAX
0045790C   |.   68 ACDC4700     ||PUSH NewsjRpg.0047DCAC                   ; |%d;
00457911   |.   8B4D F8         ||MOV ECX,DWORD PTR SS:\[EBP-8\]           ; |
00457914   |.   51              ||PUSH ECX                                  ; |stream
00457915   |.   FF15 B0E64600   ||CALL DWORD PTR DS:\[<&MSVCRT.fscanf>\]   ; \\fscanf
0045791B   |.   83C4 0C         ||ADD ESP,0C
0045791E   |.^  EB C6          |\\JMP SHORT NewsjRpg.004578E6
00457920   |>^ EB 8F          \\JMP SHORT NewsjRpg.004578B1
00457922   |>   8B55 F8         MOV EDX,DWORD PTR SS:\[EBP-8\]
00457925   |.   52              PUSH EDX                                    ; /stream
00457926   |.   FF15 4CE74600   CALL DWORD PTR DS:\[<&MSVCRT.fclose>\]     ; \\fclose
0045792C   |.   83C4 04         ADD ESP,4
0045792F   |.   B8 01000000     MOV EAX,1
00457934   |>   8BE5            MOV ESP,EBP
00457936   |.   5D              POP EBP
00457937   \\.  C2 0800         RETN 8
```

调试发现，它是先取出头三个数字，从第四个数字开始是 6 个牌局情况，结构为牌的个数+60 个牌面数据，其中 1,2,3,4 牌局分别是如下图位置的四家所分发的牌，5 和 6 暂时不知道什么用处，另外这个牌面数据与帮助里的规则不同，见下面代码，将数据进行了转换

![](img/5e8d55b33adc307c352f8e92cde42b78.jpg)

代码:
