
```asm
00401885  |.  8955 D8       |mov     dword ptr [ebp-28], edx         ；EDX-》EBP-28地址
00401888  |.  8B45 FC       |mov     eax, dword ptr [ebp-4]          ；固定字符串送入EAX
0040188B  |.  83C0 02       |add     eax, 2                          ；将固定字符串删去前两位。
0040188E  |.  8945 FC       |mov     dword ptr [ebp-4], eax          ；将新的字符串-》EBP-4地址
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
004018A7  |.  837D D8 18    |cmp     dword ptr [ebp-28], 18         ；比较EBP-28内容与18
004018AB  |.  7D 09         |jge     short 004018B6                 ；小于18就不跳
004018AD  |.  8B45 F8       |mov     eax, dword ptr [ebp-8]         ；将固定字符串---》EAX
004018B0  |.  83C0 01       |add     eax, 1                         ；删去固定字符串前一位。
004018B3  |.  8945 FC       |mov     dword ptr [ebp-4], eax         ；将新的字符串送EBP-4地址。
004018B6  |>^ E9 64FFFFFF   \jmp     0040181F                       ；循环生成一个字符串做为算法计算的一个KEY
004018BB  |>  6A 00         push    0                                ;
004018BD  |.  68 60354000   push    00403560                         ;
004018C2  |.  68 54354000   push    00403554                         ;
004018C7  |.  0FBE0D F64040>movsx   ecx, byte ptr [4040F6]           ; 取注册码的11位字符
004018CE  |.  83E9 55       sub     ecx, 55                          ; 将11位的字符的ASCALL值与85（16进制55）比较
004018D1  |.  F7D9          neg     ecx                              ; ECX求补
004018D3  |.  1BC9          sbb     ecx, ecx                         ; 有符号相减
004018D5  |.  41            inc     ecx                              ; ECX+1
004018D6  |.  51            push    ecx                              ; 将ECX压栈
004018D7  |.  FF15 00324000 call    dword ptr [<&USER32.MessageBoxA>>; 如果11位字符ASCALL值等于85，不弹出对话框。
004018DD  |.  8945 D8       mov     dword ptr [ebp-28], eax          ；不弹出对话框EAX变就为O，反之为1
004018E0  |.  837D D8 00    cmp     dword ptr [ebp-28], 0            ；比较EBP-28是否为0
004018E4  |.  74 15         je      short 004018FB                   ；相等则跳
004018E6  |.  FF15 20304000 call    dword ptr [<&KERNEL32.GetCurrent>; [GetCurrentProcess  ；取CM进程伪句柄
004018EC  |.  8945 D4       mov     dword ptr [ebp-2C], eax
004018EF  |.  6A 00         push    0                                ; /ExitCode = 0       ；句柄
004018F1  |.  8B55 D4       mov     edx, dword ptr [ebp-2C]          ; |
004018F4  |.  52            push    edx                              ; |hProcess
004018F5  |.  FF15 00304000 call    dword ptr [<&KERNEL32.TerminateP>; \TerminateProcess   ；结束进程
004018FB  |>  C745 D8 00000>mov     dword ptr [ebp-28], 0            ；初始化EBP-28的值为0
00401902  |.  C745 D0 05000>mov     dword ptr [ebp-30], 5            ；初始化EBP-30的值为5
00401909  |.  C745 CC 00000>mov     dword ptr [ebp-34], 0            ；初始化EBP-34的值为0
00401910  |>  837D D8 0C    /cmp     dword ptr [ebp-28], 0C          ；EBP-28的值是否等于12
00401914  |.  0F84 BB000000 |je      004019D5                        ；相等则跳走。
0040191A  |.  8B45 D0       |mov     eax, dword ptr [ebp-30]         ；5--》EAX
0040191D  |.  8D0C85 FCFFFF>|lea     ecx, dword ptr [eax\*4-4]        ；ECX=EAX\*4-4
00401924  |.  894D C8       |mov     dword ptr [ebp-38], ecx         ；ECX-》EBP-38地址
00401927  |>  8B55 D8       |/mov     edx, dword ptr [ebp-28]        ；EBP-28的值-EDX
0040192A  |.  0FBE82 EC4040>||movsx   eax, byte ptr [edx+4040EC]     ；依次取注册码各位字符的ASCALL值
00401931  |.  8B4D C8       ||mov     ecx, dword ptr [ebp-38]        ；EBP-38的值-》ECX
00401934  |.  0FBE540D DC   ||movsx   edx, byte ptr [ebp+ecx-24]     ；依次取上面得到的KEY指定位置上的字符的ASCALL值
00401939  |.  8B4D C8       ||mov     ecx, dword ptr [ebp-38]        ；将EBP-38的值-》ECX
0040193C  |.  83C1 01       ||add     ecx, 1                         ；ECX+1
0040193F  |.  894D C8       ||mov     dword ptr [ebp-38], ecx        ；ECX-》EBP-38地址
00401942  |.  3BC2          ||cmp     eax, edx                       ；注册码各位的字符与KEY指定位置上的字符比较
00401944  |.  74 13         ||je      short 00401959                 ；相等就跳走
00401946  |.  8B55 CC       ||mov     edx, dword ptr [ebp-34]        ；EBP-34的值-》EDX
00401949  |.  83C2 01       ||add     edx, 1                         ；EDX+1
0040194C  |.  8955 CC       ||mov     dword ptr [ebp-34], edx        ；EDX-》EBP-34地址
0040194F  |.  837D CC 04    ||cmp     dword ptr [ebp-34], 4          ；EBP-34的值与4比较
00401953  |.  7E 02         ||jle     short 00401957                 ；小于等于则跳。
00401955  |.  EB 02         ||jmp     short 00401959                 ；跳走
00401957  |>^ EB CE         |\jmp     short 00401927                 ；循环判断输入的注册码是否合法。
00401959  |>  837D CC 05    |cmp     dword ptr [ebp-34], 5           ；EBP-34的值与5比较
0040195D  |.  75 3F         |jnz     short 0040199E                  ；不等于则跳，反之就走向失败的地方。
0040195F  |.  C745 C4 78354>|mov     dword ptr [ebp-3C], 00403578    ;  abcdefghijklmn
00401966  |.  C745 C0 68354>|mov     dword ptr [ebp-40], 00403568    ;  opqrstuvwxyz
0040196D  |.  50            |push    eax
0040196E  |.  8D85 C4FFFFFF |lea     eax, dword ptr [ebp-3C]
00401974  |.  50            |push    eax
00401975  |.  FF75 C0       |push    dword ptr [ebp-40]
00401978  |.  58            |pop     eax
00401979  |.  58            |pop     eax
0040197A  |.  58            |pop     eax
0040197B  |.  8945 CC       |mov     dword ptr [ebp-34], eax
0040197E  |.  837D CC 00    |cmp     dword ptr [ebp-34], 0
00401982  |.  7E 07         |jle     short 0040198B
00401984  |.  E8 27FEFFFF   |call    004017B0                       ；错误CALL
00401989  |.  EB 0E         |jmp     short 00401999
0040198B  |>  50            |push    eax
0040198C  |.  8D85 C4FFFFFF |lea     eax, dword ptr [ebp-3C]
00401992  |.  50            |push    eax
00401993  |.  FF75 C0       |push    dword ptr [ebp-40]
00401996  |.  58            |pop     eax
00401997  |.  58            |pop     eax
00401998  |.  58            |pop     eax
00401999  |>  E8 12FEFFFF   |call    004017B0                      ；错误CALL
0040199E  |>  8B45 D8       |mov     eax, dword ptr [ebp-28]
004019A1  |.  83C0 02       |add     eax, 2
004019A4  |.  8945 D8       |mov     dword ptr [ebp-28], eax
004019A7  |.  8B4D D0       |mov     ecx, dword ptr [ebp-30]
004019AA  |.  83E9 01       |sub     ecx, 1
004019AD  |.  894D D0       |mov     dword ptr [ebp-30], ecx
004019B0  |.  90            |nop
004019B1  |.  90            |nop
004019B2  |.  90            |nop
004019B3  |.  90            |nop
004019B4  |.  90            |nop
004019B5  |.  90            |nop
004019B6  |.  90            |nop
004019B7  |.  90            |nop
004019B8  |.  90            |nop
004019B9  |.  90            |nop
004019BA  |.  90            |nop
004019BB  |.  90            |nop
004019BC  |.  837D D0 00    |cmp     dword ptr [ebp-30], 0           ；EBP-30的值与0比较
004019C0  |.  75 07         |jnz     short 004019C9                  ；不相等则跳走
004019C2  |.  C745 D0 06000>|mov     dword ptr [ebp-30], 6           ；6-》EBP-30地址
004019C9  |>  C745 CC 00000>|mov     dword ptr [ebp-34], 0           ；0-》EBP-34地址
004019D0  |.^ E9 3BFFFFFF   \jmp     00401910                        ；循环判断
004019D5  |>  E8 96FDFFFF   call    00401770                         ；成功CALL
004019DA  |.  5F            pop     edi
004019DB  |.  5E            pop     esi
004019DC  |.  5B            pop     ebx
004019DD  |.  8BE5          mov     esp, ebp
004019DF  |.  5D            pop     ebp
```
