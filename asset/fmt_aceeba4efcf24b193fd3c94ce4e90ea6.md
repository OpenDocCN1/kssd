
```
0046B560    68 C4D94600               push NOTEPAD.0046D9C4                 ; ASCII "LICENSE"

0046B565    E8 F6B0F9FF               call NOTEPAD.00406660                ; jmp 到 kernel32.SetEnvironmentVariableA

0046B56A    68 CCD94600               push NOTEPAD.0046D9CC                ; ASCII "TRUE"

0046B56F    68 D4D94600               push NOTEPAD.0046D9D4                ; ASCII "PROTECTED"

0046B574    E8 E7B0F9FF               call NOTEPAD.00406660                ; jmp 到 kernel32.SetEnvironmentVariableA

0046B579    8B45 94                   mov eax,dword ptr ss:[ebp-6C]        ; ASCII "A512-A7C1"

0046B57C    E8 DF8AF9FF               call NOTEPAD.00404060

0046B581    50                        push eax

0046B582    68 E0D94600               push NOTEPAD.0046D9E0                ; ASCII "REGNUMBER"

0046B587    E8 D4B0F9FF               call NOTEPAD.00406660                ; jmp 到 kernel32.SetEnvironmentVariableA

0046B58C    33C0                      xor eax,eax

0046B58E    5A                        pop edx

0046B58F    59                        pop ecx

0046B590    59                        pop ecx

0046B591    64:8910                   mov dword ptr fs:[eax],edx

0046B594    EB 0A                     jmp short NOTEPAD.0046B5A0

0046B596  ^ E9 B97EF9FF               jmp NOTEPAD.00403454

0046B59B    E8 6481F9FF               call NOTEPAD.00403704

0046B5A0    8D85 70FFFFFF             lea eax,dword ptr ss:[ebp-90]

0046B5A6    BA 90EC9900               mov edx,NOTEPAD.0099EC90

0046B5AB    E8 9088F9FF               call NOTEPAD.00403E40

0046B5B0    33C0                      xor eax,eax

0046B5B2    55                        push ebp

0046B5B3    68 FFB54600               push NOTEPAD.0046B5FF

0046B5B8    64:FF30                   push dword ptr fs:[eax]

0046B5BB    64:8920                   mov dword ptr fs:[eax],esp

0046B5BE    83BD 70FFFFFF 00          cmp dword ptr ss:[ebp-90],0          ; 用户名长度是否等于 0

0046B5C5    74 16                     je short NOTEPAD.0046B5DD

0046B5C7    8B85 70FFFFFF             mov eax,dword ptr ss:[ebp-90]

0046B5CD    E8 8E8AF9FF               call NOTEPAD.00404060

0046B5D2    50                        push eax

0046B5D3    68 ECD94600               push NOTEPAD.0046D9EC                ; ASCII "USERNAME"

0046B5D8    E8 83B0F9FF               call NOTEPAD.00406660                ; jmp 到 kernel32.SetEnvironmentVariableA

0046B5DD    83BD 70FFFFFF 00          cmp dword ptr ss:[ebp-90],0          ; 用户名长度等于 0，那么判断为未授权版本

0046B5E4    75 0F                     jnz short NOTEPAD.0046B5F5

0046B5E6    68 F8D94600               push NOTEPAD.0046D9F8                ; ASCII "NOT REGISTERED"

0046B5EB    68 ECD94600               push NOTEPAD.0046D9EC                ; ASCII "USERNAME"

0046B5F0    E8 6BB0F9FF               call NOTEPAD.00406660                ; jmp 到 kernel32.SetEnvironmentVariableA

0046B5F5    33C0                      xor eax,eax

0046B5F7    5A                        pop edx

0046B5F8    59                        pop ecx

0046B5F9    59                        pop ecx

0046B5FA    64:8910                   mov dword ptr fs:[eax],edx

0046B5FD    EB 0A                     jmp short NOTEPAD.0046B609

0046B5FF  ^ E9 507EF9FF               jmp NOTEPAD.00403454

0046B604    E8 FB80F9FF               call NOTEPAD.00403704

0046B609    83BD 70FFFFFF 00          cmp dword ptr ss:[ebp-90],0          ; 此处同样是作注册标志判断

0046B610    74 0F                     je short NOTEPAD.0046B621

0046B612    A1 A02C4700               mov eax,dword ptr ds:[472CA0]

0046B617    BA 4CD94600               mov edx,NOTEPAD.0046D94C            ; ASCII "REGISTERED"

0046B61C    E8 5386F9FF               call NOTEPAD.00403C74

0046B621    83BD 70FFFFFF 00          cmp dword ptr ss:[ebp-90],0          ; 同上

0046B628    75 0F                     jnz short NOTEPAD.0046B639

0046B62A    A1 A02C4700               mov eax,dword ptr ds:[472CA0]

0046B62F    BA 10DA4600               mov edx,NOTEPAD.0046DA10            ; ASCII "NOTREGISTERED"

0046B634    E8 3B86F9FF               call NOTEPAD.00403C74

0046B639    B9 F4D54600               mov ecx,NOTEPAD.0046D5F4            ; ASCII "netctrl.ini"

0046B63E    B2 01                     mov dl,1

0046B640    A1 60134500               mov eax,dword ptr ds:[451360]

0046B645    E8 BE5DFEFF               call NOTEPAD.00451408               ; 授权文件信息检测

0046B64A    A3 20F09900               mov dword ptr ds:[99F020],eax

0046B64F    6A 00                     push 0

0046B651    8D85 34FFFFFF             lea eax,dword ptr ss:[ebp-CC]

0046B657    50                        push eax

0046B658    8B15 7C2E4700             mov edx,dword ptr ds:[472E7C]        ; ASCII "NOTEPAD.LIC"

0046B65E    8B12                      mov edx,dword ptr ds:[edx]

0046B660    B9 28DA4600               mov ecx,NOTEPAD.0046DA28            ; ASCII "Serial"

0046B665    A1 20F09900               mov eax,dword ptr ds:[99F020]

0046B66A    8B18                      mov ebx,dword ptr ds:[eax]

0046B66C    FF13                      call dword ptr ds:[ebx]

0046B66E    B2 01                     mov dl,1

0046B670    A1 20F09900               mov eax,dword ptr ds:[99F020]

0046B675    8B08                      mov ecx,dword ptr ds:[eax]

0046B677    FF51 FC                   call dword ptr ds:[ecx-4]

0046B67A    80BD 1FFFFFFF 01          cmp byte ptr ss:[ebp-E1],1           ; 小于 1 则表明没有授权文件

0046B681    75 0C                     jnz short NOTEPAD.0046B68F

0046B683    C685 3FFFFFFF 01          mov byte ptr ss:[ebp-C1],1

0046B68A    E9 80150000               jmp NOTEPAD.0046CC0F

0046B68F    8D85 84DEF9FF             lea eax,dword ptr ss:[ebp+FFF9DE84]

0046B695    BA 15CD9900               mov edx,NOTEPAD.0099CD15

0046B69A    E8 A187F9FF               call NOTEPAD.00403E40               ; 难道是授权码？不会吧

0046B69F    8B95 84DEF9FF             mov edx,dword ptr ss:[ebp+FFF9DE84]  ; ASCII "FD41-3CFC-81AE-BBA3-D3B2"

0046B6A5    8B45 DC                   mov eax,dword ptr ss:[ebp-24]

0046B6A8    E8 FF88F9FF               call NOTEPAD.00403FAC

0046B6AD    75 20                     jnz short NOTEPAD.0046B6CF

0046B6AF    C605 58F09900 01          mov byte ptr ds:[99F058],1

0046B6B6    803D 48F09900 00          cmp byte ptr ds:[99F048],0

0046B6BD    0F85 4C150000             jnz NOTEPAD.0046CC0F

0046B6C3    C685 3FFFFFFF 01          mov byte ptr ss:[ebp-C1],1

0046B6CA    E9 40150000               jmp NOTEPAD.0046CC0F

0046B6CF    803D 99ED9900 01          cmp byte ptr ds:[99ED99],1           ; 小于 1 则表明没有授权码

0046B6D6    0F85 B4000000             jnz NOTEPAD.0046B790

0046B6DC    E8 5BE0F9FF               call NOTEPAD.0040973C

0046B6E1    DC1D 91ED9900             fcomp qword ptr ds:[99ED91]

0046B6E7    DFE0                      fstsw ax

0046B6E9    9E                        sahf

0046B6EA    0F82 A0000000             jb NOTEPAD.0046B790

0046B6F0    C605 07CD9900 01          mov byte ptr ds:[99CD07],1

0046B6F7    8D85 6CDEF9FF             lea eax,dword ptr ss:[ebp+FFF9DE6C]

0046B6FD    E8 A688FEFF               call NOTEPAD.00453FA8

0046B702    FFB5 6CDEF9FF             push dword ptr ss:[ebp+FFF9DE6C]

0046B708    68 9CD84600               push NOTEPAD.0046D89C

0046B70D    FF75 E0                   push dword ptr ss:[ebp-20]

0046B710    8D85 84DEF9FF             lea eax,dword ptr ss:[ebp+FFF9DE84]

0046B716    BA 03000000               mov edx,3

0046B71B    E8 3C88F9FF               call NOTEPAD.00403F5C

0046B720    8B95 84DEF9FF             mov edx,dword ptr ss:[ebp+FFF9DE84]

0046B726    8B45 FC                   mov eax,dword ptr ss:[ebp-4]

0046B729    E8 DEBBFFFF               call NOTEPAD.0046730C
```
