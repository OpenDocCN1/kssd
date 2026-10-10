
```
0040183C  |.  8B55 FC       |mov     edx, dword ptr [ebp-4]          ；将固定字符串送入 EDX  
0040183F  |.  0FBE02        |movsx   eax, byte ptr [edx]             ；依次取固定字符的 ACSCALL 值-》EAX  
00401842  |.  83C0 20       |add     eax, 20                         ；EAX 的值+32  
00401845  |.  3BC8          |cmp     ecx, eax                        ；ECX 与 EAX 比较  
00401847  |.  74 21         |je      short 0040186A                  ；相等则跳  
00401849  |.  68 6C414000   |push    0040416C                        ;  用户名        
0040184E  |.  E8 AD0B0000   |call    <jmp.&MSVCRT.strlen>            ;  取用户名长度，长度值---》EAX  
00401853  |.  83C4 04       |add     esp, 4  
00401856  |.  0FBE88 6B4140>|movsx   ecx, byte ptr [eax+40416B]      ；用户最后一位字符的 ASCALL 值-》ECX  
0040185D  |.  8B55 FC       |mov     edx, dword ptr [ebp-4]          ；将固定字符串送入 EDX  
00401860  |.  0FBE02        |movsx   eax, byte ptr [edx]             ；依次取固定字符的 ACSCALL 值-》EAX  
00401863  |.  83C0 20       |add     eax, 20                         ；EAX 的值+32  
00401866  |.  3BC8          |cmp     ecx, eax                        ；ECX 与 EAX 比较  
00401868  |.  75 09         |jnz     short 00401873                  ；不相等则跳  
0040186A  |>  8B4D FC       |mov     ecx, dword ptr [ebp-4]          ；将固定字符串送入 ECX  
0040186D  |.  83C1 01       |add     ecx, 1                          ；ECX 删去前一位字符  
00401870  |.  894D FC       |mov     dword ptr [ebp-4], ecx          ；将新的字符串-》EBP-4 地址  
00401873  |>  8B55 D8       |mov     edx, dword ptr [ebp-28]         ；EBP-28 地址--》EDX  
00401876  |.  8B45 FC       |mov     eax, dword ptr [ebp-4]          ；固定字符到 EAX  
00401879  |.  8A08          |mov     cl, byte ptr [eax]              ；依次取固定字符串指定位置的值送到 CL  
0040187B  |.  884C15 DC     |mov     byte ptr [ebp+edx-24], cl       ；保存字符，作用是得到一个字符串，算法计算要用到。  
0040187F  |.  8B55 D8       |mov     edx, dword ptr [ebp-28]         ；EBP-28 内容-》到 EDX  
00401882  |.  83C2 01       |add     edx, 1                          ；EDX+1  
00401885  |.  8955 D8       |mov     dword ptr [ebp-28], edx         ；EDX-》EBP-28 地址  
00401888  |.  8B45 FC       |mov     eax, dword ptr [ebp-4]          ；固定字符串送入 EAX  
0040188B  |.  83C0 02       |add     eax, 2                          ；将固定字符串删去前两位。  
0040188E  |.  8945 FC       |mov     dword ptr [ebp-4], eax          ；将新的字符串-》EBP-4 地址  
00401891  |.  90            |nop  
00401892  |.  90            |nop  
00401893  |.  90            |nop  
00401894  |.  90            |nop  
00401895  |.  90            |nop  
00401896  |.  90            |nop  
00401897  |.  90            |nop  
00401898  |.  90            |nop  
00401899  |.  90            |nop  
0040189A  |.  90            |nop  
0040189B  |.  90            |nop  
0040189C  |.  90            |nop  
0040189D  |.  8B4D FC       |mov     ecx, dword ptr [ebp-4]         ；将新的字符串-》ECX  
004018A0  |.  0FBE11        |movsx   edx, byte ptr [ecx]            ；取第一位字符串  
004018A3  |.  85D2          |test    edx, edx                       ；判断是否取成功  
004018A5  |.  75 0F         |jnz     short 004018B6                 ；成功就跳走。  
004018A7  |.  837D D8 18    |cmp     dword ptr [ebp-28], 18         ；比较 EBP-28 内容与 18  
004018AB  |.  7D 09         |jge     short 004018B6                 ；小于 18 就不跳  
004018AD  |.  8B45 F8       |mov     eax, dword ptr [ebp-8]         ；将固定字符串---》EAX  
004018B0  |.  83C0 01       |add     eax, 1                         ；删去固定字符串前一位。  
004018B3  |.  8945 FC       |mov     dword ptr [ebp-4], eax         ；将新的字符串送 EBP-4 地址。  
004018B6  |>^ E9 64FFFFFF  \\jmp     0040181F                       ；循环生成一个字符串做为算法计算的一个 KEY  
004018BB  |>  6A 00         push    0                                ;  
004018BD  |.  68 60354000   push    00403560                         ;  
004018C2  |.  68 54354000   push    00403554                         ;  
004018C7  |.  0FBE0D F64040>movsx   ecx, byte ptr [4040F6]           ;  取注册码的 11 位字符  
004018CE  |.  83E9 55       sub     ecx, 55                          ;  将 11 位的字符的 ASCALL 值与 85（16 进制 55）比较  
004018D1  |.  F7D9          neg     ecx                              ;  ECX 求补  
004018D3  |.  1BC9          sbb     ecx, ecx                         ;  有符号相减  
004018D5  |.  41            inc     ecx                              ;  ECX+1  
004018D6  |.  51            push    ecx                              ;  将 ECX 压栈  
004018D7  |.  FF15 00324000 call    dword ptr [<&USER32.MessageBoxA>>;  如果 11 位字符 ASCALL 值等于 85，不弹出对话框。  
004018DD  |.  8945 D8       mov     dword ptr [ebp-28], eax          ；不弹出对话框 EAX 变就为 O，反之为 1  
004018E0  |.  837D D8 00    cmp     dword ptr [ebp-28], 0            ；比较 EBP-28 是否为 0  
004018E4  |.  74 15         je      short 004018FB                   ；相等则跳  
004018E6  |.  FF15 20304000 call    dword ptr [<&KERNEL32.GetCurrent>;  \[GetCurrentProcess  ；取 CM 进程伪句柄  
004018EC  |.  8945 D4       mov     dword ptr [ebp-2C], eax  
004018EF  |.  6A 00         push    0                                ;  /ExitCode = 0       ；句柄  
004018F1  |.  8B55 D4       mov     edx, dword ptr [ebp-2C]          ;  |                    
004018F4  |.  52            push    edx                             ;  |hProcess  
004018F5  |.  FF15 00304000 call    dword ptr [<&KERNEL32.TerminateP>;  \\TerminateProcess   ；结束进程  
004018FB  |>  C745 D8 00000>mov     dword ptr [ebp-28], 0            ；初始化 EBP-28 的值为 0  
00401902  |.  C745 D0 05000>mov     dword ptr [ebp-30], 5            ；初始化 EBP-30 的值为 5  
00401909  |.  C745 CC 00000>mov     dword ptr [ebp-34], 0            ；初始化 EBP-34 的值为 0  
00401910  |>  837D D8 0C    /cmp     dword ptr [ebp-28], 0C          ；EBP-28 的值是否等于 12  
00401914  |.  0F84 BB000000 |je      004019D5                        ；相等则跳走。  
0040191A  |.  8B45 D0       |mov     eax, dword ptr [ebp-30]         ；5--》EAX  
0040191D  |.  8D0C85 FCFFFF>|lea     ecx, dword ptr [eax*4-4]        ；ECX=EAX*4-4  
00401924  |.  894D C8       |mov     dword ptr [ebp-38], ecx         ；ECX-》EBP-38 地址  
00401927  |>  8B55 D8       |/mov     edx, dword ptr [ebp-28]        ；EBP-28 的值-EDX  
0040192A  |.  0FBE82 EC4040>||movsx   eax, byte ptr [edx+4040EC]     ；依次取注册码各位字符的 ASCALL 值  
00401931  |.  8B4D C8       ||mov     ecx, dword ptr [ebp-38]        ；EBP-38 的值-》ECX  
00401934  |.  0FBE540D DC   ||movsx   edx, byte ptr [ebp+ecx-24]     ；依次取上面得到的 KEY 指定位置上的字符的 ASCALL 值  
00401939  |.  8B4D C8       ||mov     ecx, dword ptr [ebp-38]        ；将 EBP-38 的值-》ECX  
0040193C  |.  83C1 01       ||add     ecx, 1                         ；ECX+1  
0040193F  |.  894D C8       ||mov     dword ptr [ebp-38], ecx        ；ECX-》EBP-38 地址  
00401942  |.  3BC2          ||cmp     eax, edx                       ；注册码各位的字符与 KEY 指定位置上的字符比较  
00401944  |.  74 13         ||je      short 00401959                 ；相等就跳走  
00401946  |.  8B55 CC       ||mov     edx, dword ptr [ebp-34]        ；EBP-34 的值-》EDX  
00401949  |.  83C2 01       ||add     edx, 1                         ；EDX+1  
0040194C  |.  8955 CC       ||mov     dword ptr [ebp-34], edx        ；EDX-》EBP-34 地址  
0040194F  |.  837D CC 04    ||cmp     dword ptr [ebp-34], 4          ；EBP-34 的值与 4 比较  
00401953  |.  7E 02         ||jle     short 00401957                 ；小于等于则跳。  
00401955  |.  EB 02         ||jmp     short 00401959                 ；跳走  
00401957  |>^ EB CE         |\\jmp     short 00401927                 ；循环判断输入的注册码是否合法。  
00401959  |>  837D CC 05    |cmp     dword ptr [ebp-34], 5           ；EBP-34 的值与 5 比较  
0040195D  |.  75 3F         |jnz     short 0040199E                 ；不等于则跳，反之就走向失败的地方。  
0040195F  |.  C745 C4 78354>|mov     dword ptr [ebp-3C], 00403578    ;  abcdefghijklmn  
00401966  |.  C745 C0 68354>|mov     dword ptr [ebp-40], 00403568    ;  opqrstuvwxyz  
0040196D  |.  50            |push    eax  
0040196E  |.  8D85 C4FFFFFF |lea     eax, dword ptr [ebp-3C]  
00401974  |.  50            |push    eax  
```
