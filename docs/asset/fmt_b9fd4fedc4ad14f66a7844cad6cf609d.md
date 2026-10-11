
```
00455543  |. >FSTP QWORD PTR SS:\[ESP\]               ; |Arg1 (8-byte)   

00455546  |. >CALL WebCatch.00549AE8                ; \\WebCatch.00549AE8   

0045554B  |. >ADD ESP,8   

0045554E  |. >ADD ESP,-8                            ; /   

00455551  |. >FSTP QWORD PTR SS:\[ESP\]               ; |Arg1 (8-byte)   

00455554  |. >CALL WebCatch.00548698                ; \\WebCatch.00548698   

00455559  |. >ADD ESP,8   

0045555C  |. >FMUL DWORD PTR DS:\[455944\]            ；浮点运算刚刚取得绝对值 \* 1381   

00455562  |. >CALL WebCatch.005486C0   

00455567  |. >MOV ECX,1A   

0045556C  |. >CDQ   

0045556D  |. >IDIV ECX                              ；除以 1ah 的余数存入 dx   

0045556F  |. >ADD DL,41                             ；加上 41h   

00455572  |. >MOV BYTE PTR SS:\[EBP\-38\],DL           ；保存结果，并于假码的第 21-25 位进行比较   

00455575  |. >PUSH 5                                ; /Arg3 = 00000005   

00455577  |. >PUSH WebCatch.005CA5C7                ; |Arg2 = 005CA5C7 ASCII "4444455555"   

0045557C  |. >LEA EAX,DWORD PTR SS:\[EBP\-3C\]         ; |   

0045557F  |. >PUSH EAX                              ; |Arg1   

00455580  |. >CALL WebCatch.0054341C                ; \\WebCatch.0054341C   

00455585  |. >ADD ESP,0C   

00455588  |. >TEST EAX,EAX   

0045558A  |. >JE SHORT WebCatch.00455590   

0045558C  |. >MOV DL,5   

0045558E  |. >JMP SHORT WebCatch.00455592   

00455590  |> >XOR EDX,EDX   

00455592  |> >TEST DL,DL   

00455594  |. >JE SHORT WebCatch.004555B9   

00455596  |. >XOR EAX,EAX   

00455598  |. >MOV EDX,2   

0045559D  |. >PUSH EAX   

0045559E  |. >LEA EAX,DWORD PTR SS:\[EBP\-4\]   

004555A1  |. >DEC DWORD PTR SS:\[EBP\-10\]   

004555A4  |. >CALL WebCatch.0055053C   

004555A9  |. >POP EAX   

004555AA  |. >MOV EDX,DWORD PTR SS:\[EBP\-2C\]   

004555AD  |. >MOV DWORD PTR FS:\[0\],EDX   

004555B4  |. >JMP WebCatch.0045592D   

004555B9  |> >MOVSX ECX,BYTE PTR DS:\[EDI+3\]         ；取去‘-’后注册码的第 14 位           

004555BD  |. >MOVSX EAX,BYTE PTR DS:\[EDI+4\]         ；取去‘-’后注册码的第 15 位   

004555C1  |. >MOV EBX,ECX   

004555C3  |. >MOVSX EDX,BYTE PTR DS:\[ESI\]           ；取去‘-’后注册码的第 1 位   

004555C6  |. >MOVSX ECX,BYTE PTR DS:\[5CA5BD\]        ；取去‘-’后注册码的第 6 位   

004555CD  |. >ADD EBX,EBX   

004555CF  |. >ADD EDX,ECX   

004555D1  |. >MOVSX ECX,BYTE PTR DS:\[5CA5C7\]        ；取去‘-’后注册码的第 16 位   

004555D8  |. >LEA EBX,DWORD PTR DS:\[EBX+EBX\*4\]   

004555DB  |. >ADD EBX,EAX                           ；此处就算结果以后 455615 处使用   

004555DD  |. >MOVSX EAX,BYTE PTR DS:\[EDI\]           ；取去‘-’后注册码的第 11 位   

004555E0  |. >ADD EDX,EAX   

004555E2  |. >ADD EBX,-210   

004555E8  |. >ADD EDX,ECX   

004555EA  |. >ADD EDX,6   

004555ED  |. >ADD ESP,-8   

004555F0  |. >MOV DWORD PTR SS:\[EBP\-5C\],EDX         ; |   

004555F3  |. >FILD DWORD PTR SS:\[EBP\-5C\]            ; |   

004555F6  |. >FSTP QWORD PTR SS:\[ESP\]               ; |   

004555F9  |. >CALL WebCatch.00548560                ; \\WebCatch.00548560  求出 cos 值   

004555FE  |. >ADD ESP,8   

00455601  |. >ADD ESP,-8                            ; /   

00455604  |. >FSTP QWORD PTR SS:\[ESP\]               ; |Arg1 (8-byte)   

00455607  |. >CALL WebCatch.00548698                ; \\WebCatch.00548698  取决对值   

0045560C  |. >ADD ESP,8   

0045560F  |. >FMUL DWORD PTR DS:\[455948\]            ；浮点运算刚刚取得绝对值 \* 1016   

00455615  |. >MOV DWORD PTR SS:\[EBP\-60\],EBX   

00455618  |. >FILD DWORD PTR SS:\[EBP\-60\]   

0045561B  |. >FMULP ST(1),ST                        ；相乘   

0045561D  |. >CALL WebCatch.005486C0   

00455622  |. >MOV ECX,1A   

00455627  |. >CDQ   

00455628  |. >IDIV ECX                              ；除以 1ah 的余数存入 dx   

0045562A  |. >ADD DL,41                             ；保存结果   

0045562D  |. >MOV BYTE PTR SS:\[EBP\-44\],DL   

00455630  |. >MOVSX EAX,BYTE PTR DS:\[ESI+1\]         ；取去‘-’后注册码的第 2 位   

00455634  |. >MOVSX EDX,BYTE PTR DS:\[5CA5BE\]        ；取去‘-’后注册码的第 7 位   

0045563B  |. >ADD EAX,EDX   

0045563D  |. >MOVSX ECX,BYTE PTR DS:\[EDI+1\]         ；取去‘-’后注册码的第 17 位   

00455641  |. >ADD EAX,ECX   

00455643  |. >MOVSX EDX,BYTE PTR DS:\[5CA5C8\]        ；取去‘-’后注册码的第 12 位，下面的计算同上   

0045564A  |. >ADD EAX,EDX   

0045564C  |. >ADD EAX,7   

0045564F  |. >MOV DWORD PTR SS:\[EBP\-64\],EAX   

00455652  |. >FILD DWORD PTR SS:\[EBP\-64\]   

00455655  |. >ADD ESP,-8                            ; /   

00455658  |. >FSTP QWORD PTR SS:\[ESP\]               ; |Arg1 (8-byte)   

0045565B  |. >CALL WebCatch.00548560                ; \\WebCatch.00548560   

00455660  |. >ADD ESP,8   

00455663  |. >ADD ESP,-8                            ; /   

00455666  |. >FSTP QWORD PTR SS:\[ESP\]               ; |Arg1 (8-byte)   

00455669  |. >CALL WebCatch.00548698                ; \\WebCatch.00548698   

0045566E  |. >ADD ESP,8   

00455671  |. >FMUL DWORD PTR DS:\[45594C\]            ；浮点运算刚刚取得绝对值 \* 1207   

00455677  |. >MOV DWORD PTR SS:\[EBP\-68\],EBX   

0045567A  |. >FILD DWORD PTR SS:\[EBP\-68\]   

0045567D  |. >FMULP ST(1),ST   

0045567F  |. >CALL WebCatch.005486C0   

00455684  |. >MOV ECX,1A   

00455689  |. >CDQ   

0045568A  |. >IDIV ECX   

0045568C  |. >ADD DL,41   

0045568F  |. >MOV BYTE PTR SS:\[EBP\-43\],DL            ；保存结果   

00455692  |. >MOVSX EAX,BYTE PTR DS:\[ESI+2\]          ；取去‘-’后注册码的第 3 位   

00455696  |. >MOVSX EDX,BYTE PTR DS:\[5CA5BF\]         ；取去‘-’后注册码的第 8 位   

0045569D  |. >ADD EAX,EDX   

0045569F  |. >MOVSX ECX,BYTE PTR DS:\[EDI+2\]          ；取去‘-’后注册码的第 18 位   

004556A3  |. >ADD EAX,ECX   

004556A5  |. >MOVSX EDX,BYTE PTR DS:\[5CA5C9\]         ；取去‘-’后注册码的第 13 位 ，下面的计算同上   

004556AC  |. >ADD EAX,EDX   

004556AE  |. >ADD EAX,8   

004556B1  |. >MOV DWORD PTR SS:\[EBP\-6C\],EAX   

004556B4  |. >FILD DWORD PTR SS:\[EBP\-6C\]   

004556B7  |. >ADD ESP,-8                            ; /   

004556BA  |. >FSTP QWORD PTR SS:\[ESP\]               ; |Arg1 (8-byte)   

004556BD  |. >CALL WebCatch.00548560                ; \\WebCatch.00548560   

004556C2  |. >ADD ESP,8   

004556C5  |. >ADD ESP,-8                            ; /   

004556C8  |. >FSTP QWORD PTR SS:\[ESP\]               ; |Arg1 (8-byte)   

004556CB  |. >CALL WebCatch.00548698                ; \\WebCatch.00548698   

004556D0  |. >ADD ESP,8   

004556D3  |. >FMUL DWORD PTR DS:\[455950\]            ；浮点运算刚刚取得绝对值 \* 1084   

004556D9  |. >MOV DWORD PTR SS:\[EBP\-70\],EBX   

004556DC  |. >FILD DWORD PTR SS:\[EBP\-70\]   

004556DF  |. >FMULP ST(1),ST   

004556E1  |. >CALL WebCatch.005486C0   

004556E6  |. >MOV ECX,1A   

004556EB  |. >CDQ   

004556EC  |. >IDIV ECX   

004556EE  |. >ADD DL,41   

004556F1  |. >MOV BYTE PTR SS:\[EBP\-42\],DL            ；保存结果   

004556F4  |. >MOVSX EAX,BYTE PTR DS:\[ESI+3\]          ；取去‘-’后注册码的第 4 位   

004556F8  |. >MOVSX EDX,BYTE PTR DS:\[5CA5C0\]         ；取去‘-’后注册码的第 9 位   

004556FF  |. >ADD EAX,EDX   

00455701  |. >MOVSX ECX,BYTE PTR DS:\[EDI+3\]          ；取去‘-’后注册码的第 19 位     

00455705  |. >ADD EAX,ECX   

00455707  |. >MOVSX EDX,BYTE PTR DS:\[5CA5CA\]         ；取去‘-’后注册码的第 13 位 ，下面的计算同上   

0045570E  |. >ADD EAX,EDX   

00455710  |. >ADD EAX,9   

00455713  |. >MOV DWORD PTR SS:\[EBP\-74\],EAX   

00455716  |. >FILD DWORD PTR SS:\[EBP\-74\]   

00455719  |. >ADD ESP,-8                            ; /   

0045571C  |. >FSTP QWORD PTR SS:\[ESP\]               ; |Arg1 (8-byte)   

0045571F  |. >CALL WebCatch.00548560                ; \\WebCatch.00548560   

00455724  |. >ADD ESP,8   

00455727  |. >ADD ESP,-8                            ; /   

0045572A  |. >FSTP QWORD PTR SS:\[ESP\]               ; |Arg1 (8-byte)   

0045572D  |. >CALL WebCatch.00548698                ; \\WebCatch.00548698   

00455732  |. >ADD ESP,8   

00455735  |. >FMUL DWORD PTR DS:\[455954\]            ；浮点运算刚刚取得绝对值 \* 1059   

0045573B  |. >MOV DWORD PTR SS:\[EBP\-78\],EBX   

0045573E  |. >FILD DWORD PTR SS:\[EBP\-78\]   

00455741  |. >FMULP ST(1),ST   

00455743  |. >CALL WebCatch.005486C0   

00455748  |. >MOV ECX,1A   

0045574D  |. >CDQ   

0045574E  |. >IDIV ECX   

00455750  |. >ADD DL,41   
```
