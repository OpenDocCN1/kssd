
## 软件算法 CALL 部分

> 004F55D8    55              PUSH EBP                    ; 软件算法 CALL 部分  
> 004F55D9    8BEC            MOV EBP,ESP  
> 004F55DB    83C4 E0         ADD ESP,-20  
> 004F55DE    53              PUSH EBX  
> 004F55DF    56              PUSH ESI  
> 004F55E0    57              PUSH EDI  
> 004F55E1    33DB            XOR EBX,EBX  
> 004F55E3    895D EC         MOV DWORD PTR SS:[EBP-14],EBX  
> 004F55E6    895D E8         MOV DWORD PTR SS:[EBP-18],EBX  
> 004F55E9    895D E4         MOV DWORD PTR SS:[EBP-1C],EBX  
> 004F55EC    895D E0         MOV DWORD PTR SS:[EBP-20],EBX  
> 004F55EF    8BF9            MOV EDI,ECX  
> 004F55F1    8955 FC         MOV DWORD PTR SS:[EBP-4],EDX  
> 004F55F4    8BF0            MOV ESI,EAX  
> 004F55F6    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]  
> 004F55F9    E8 2EF9F0FF     CALL folderlo.00404F2C  
> 004F55FE    33C0            XOR EAX,EAX  
> 004F5600    55              PUSH EBP  
> 004F5601    68 71574F00     PUSH folderlo.004F5771  
> 004F5606    64:FF30         PUSH DWORD PTR FS:[EAX]  
> 004F5609    64:8920         MOV DWORD PTR FS:[EAX],ESP  
> 004F560C    8D45 EC         LEA EAX,DWORD PTR SS:[EBP-14]  
> 004F560F    BA 88574F00     MOV EDX,folderlo.004F5788             ; 固定字符串:gf258369gf  
> 004F5614    E8 0BF5F0FF     CALL folderlo.00404B24  
> 004F5619    837D EC 00      CMP DWORD PTR SS:[EBP-14],0  
> 004F561D    75 0D           JNZ SHORT folderlo.004F562C  
> 004F561F    8D45 E0         LEA EAX,DWORD PTR SS:[EBP-20]  
> 004F5622    8B55 FC         MOV EDX,DWORD PTR SS:[EBP-4]  
> 004F5625    E8 FAF4F0FF     CALL folderlo.00404B24  
> 004F562A    EB 5D           JMP SHORT folderlo.004F5689  
> 004F562C    8B45 EC         MOV EAX,DWORD PTR SS:[EBP-14]  
> 004F562F    E8 10F7F0FF     CALL folderlo.00404D44  
> 004F5634    8BD8            MOV EBX,EAX                     ; 长度 A 放 EBX  
> 004F5636    8D45 E8         LEA EAX,DWORD PTR SS:[EBP-18]  
> 004F5639    50              PUSH EAX  
> 004F563A    8BCB            MOV ECX,EBX  
> 004F563C    D1F9            SAR ECX,1  
> 004F563E    79 03           JNS SHORT folderlo.004F5643  
> 004F5640    83D1 00         ADC ECX,0  
> 004F5643    BA 01000000     MOV EDX,1  
> 004F5648    8B45 EC         MOV EAX,DWORD PTR SS:[EBP-14]  
> 004F564B    E8 4CF9F0FF     CALL folderlo.00404F9C  
> 004F5650    8D45 E4         LEA EAX,DWORD PTR SS:[EBP-1C]  
> 004F5653    50              PUSH EAX  
> 004F5654    8BC3            MOV EAX,EBX  
> 004F5656    D1F8            SAR EAX,1  
> 004F5658    79 03           JNS SHORT folderlo.004F565D  
> 004F565A    83D0 00         ADC EAX,0  
> 004F565D    8BCB            MOV ECX,EBX  
> 004F565F    2BC8            SUB ECX,EAX  
> 004F5661    8BD3            MOV EDX,EBX  
> 004F5663    D1FA            SAR EDX,1  
> 004F5665    79 03           JNS SHORT folderlo.004F566A  
> 004F5667    83D2 00         ADC EDX,0  
> 004F566A    42              INC EDX  
> 004F566B    8B45 EC         MOV EAX,DWORD PTR SS:[EBP-14]  
> 004F566E    E8 29F9F0FF     CALL folderlo.00404F9C  
> 004F5673    FF75 E8         PUSH DWORD PTR SS:[EBP-18]        ; 固定值前 5 位进栈  
> 004F5676    FF75 FC         PUSH DWORD PTR SS:[EBP-4]         ; 用户名进栈  
> 004F5679    FF75 E4         PUSH DWORD PTR SS:[EBP-1C]        ; 固定值后 5 位进栈  
> 004F567C    8D45 E0         LEA EAX,DWORD PTR SS:[EBP-20]  
> 004F567F    BA 03000000     MOV EDX,3  
> 004F5684    E8 7BF7F0FF     CALL folderlo.00404E04  
> 004F5689    C745 F0 0000000>MOV DWORD PTR SS:[EBP-10],0     ; 拼接(将用户名连接在固定字符串中间)  
> 004F5690    C745 F4 0000000>MOV DWORD PTR SS:[EBP-C],0  
> 004F5697    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]  
> 004F569A    E8 A5F6F0FF     CALL folderlo.00404D44  
> 004F569F    3B46 4C         CMP EAX,DWORD PTR DS:[ESI+4C]  
> 004F56A2    7F 0D           JG SHORT folderlo.004F56B1       ; 用户名小于 63H  
> 004F56A4    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]  
> 004F56A7    E8 98F6F0FF     CALL folderlo.00404D44  
> 004F56AC    3B46 50         CMP EAX,DWORD PTR DS:[ESI+50]  
> 004F56AF    7D 0C           JGE SHORT folderlo.004F56BD      ; 用户名要大于 3  
> 004F56B1    8BC7            MOV EAX,EDI  
> 004F56B3    E8 D4F3F0FF     CALL folderlo.00404A8C  
> 004F56B8    E9 91000000     JMP folderlo.004F574E  
> 004F56BD    8B45 E0         MOV EAX,DWORD PTR SS:[EBP-20]  
> 004F56C0    E8 7FF6F0FF     CALL folderlo.00404D44  
> 004F56C5    8BD8            MOV EBX,EAX  
> 004F56C7    EB 37           JMP SHORT folderlo.004F5700  
> 004F56C9    8B46 68         MOV EAX,DWORD PTR DS:[ESI+68]   ; 算法从这里开始 赋给 EAX 初试值=06192A89  
> 004F56CC    8B56 6C         MOV EDX,DWORD PTR DS:[ESI+6C]  
> 004F56CF    0345 F0         ADD EAX,DWORD PTR SS:[EBP-10]   ; 加法  初试值=0  
> 004F56D2    1355 F4         ADC EDX,DWORD PTR SS:[EBP-C]    ; 带进位加法  
> 004F56D5    52              PUSH EDX  
> 004F56D6    50              PUSH EAX  
> 004F56D7    8B45 E0         MOV EAX,DWORD PTR SS:[EBP-20]   ; 将固定字符串地址放 EAX 中  
> 004F56DA    0FB64418 FF     MOVZX EAX,BYTE PTR DS:[EAX+EBX-1] ; 先扩展 后传送  
> 004F56DF    50              PUSH EAX  
> 004F56E0    B8 59040000     MOV EAX,459  
> 004F56E5    5A              POP EDX                     ; 将该字符的 ASCII 码数值放 EDX 中  
> 004F56E6    8BCA            MOV ECX,EDX                 ; 放 ECX 中当除数  
> 004F56E8    33D2            XOR EDX,EDX  
> 004F56EA    F7F1            DIV ECX  
> 004F56EC    8BC2            MOV EAX,EDX                 ; 将余数放 EAX 中  
> 004F56EE    33D2            XOR EDX,EDX                 ; EDX 清 0  
> 004F56F0    290424          SUB DWORD PTR SS:[ESP],EAX  ; ESP 中数据-余数  
> 004F56F3    195424 04       SBB DWORD PTR SS:[ESP+4],EDX ; 带借位的减法  
> 004F56F7    58              POP EAX  
> 004F56F8    5A              POP EDX  
> 004F56F9    8945 F0         MOV DWORD PTR SS:[EBP-10],EAX ; EAX 放 EBP-10  
> 004F56FC    8955 F4         MOV DWORD PTR SS:[EBP-C],EDX  
> 004F56FF    4B              DEC EBX                     ; EDC EBX EBX 的数值来自这里  
> 004F5700    8B45 E0         MOV EAX,DWORD PTR SS:[EBP-20] ; 将字符串地址放 EAX  
> 004F5703    E8 3CF6F0FF     CALL folderlo.00404D44      ; 测试下长度  
> 004F5708    3BD8            CMP EBX,EAX                 ; 长度比较  
> 004F570A    7F 04           JG SHORT folderlo.004F5710  
> 004F570C    85DB            TEST EBX,EBX  
> 004F570E  ^ 7F B9          JG SHORT folderlo.004F56C9   ; 大于 0 则继续循环  
> 004F5710    8B5E 60         MOV EBX,DWORD PTR DS:[ESI+60] ; [ESI+60] 这里是 A  
> 004F5713    85DB            TEST EBX,EBX  
> 004F5715    7F 11           JG SHORT folderlo.004F5728  
> 004F5717    FF75 F4         PUSH DWORD PTR SS:[EBP-C]  
> 004F571A    FF75 F0         PUSH DWORD PTR SS:[EBP-10]  
> 004F571D    8BD7            MOV EDX,EDI  
> 004F571F    33C0            XOR EAX,EAX  
> 004F5721    E8 8E40F1FF     CALL folderlo.004097B4  
> 004F5726    EB 26           JMP SHORT folderlo.004F574E  
> 004F5728    FF75 F4         PUSH DWORD PTR SS:[EBP-C]  
> 004F572B    FF75 F0         PUSH DWORD PTR SS:[EBP-10]  
> 004F572E    8BD7            MOV EDX,EDI  
> 004F5730    8BC3            MOV EAX,EBX                 ; EAX 和 EBX 都成为 A

## 函数返回与跳转

> 004F5A78    E8 0FF0F0FF     CALL folderlo.00404A8C  
> 004F5A7D    C3              RETN  
> 004F5A7E  ^ E9 A9E8F0FF     JMP folderlo.0040432C  
> 004F5A83  ^ EB E3          JMP SHORT folderlo.004F5A68  
> 004F5A85    8BC3            MOV EAX,EBX  
> 004F5A87    5B              POP EBX  
> 004F5A88    8BE5            MOV ESP,EBP  
> 004F5A8A    5D              POP EBP  
> 004F5A8B    C2 0400         RETN 4
