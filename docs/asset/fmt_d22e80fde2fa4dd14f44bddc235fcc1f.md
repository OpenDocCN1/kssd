
```
003842B2    3D 45454545         cmp eax,45454545

003842B7    0F85 F4000000       jnz 003843B1

003842BD    8B4424 2C           mov eax,dword ptr ss:[esp+2C]

003842C1    3D 00000080         cmp eax,80000000

003842C6    0F83 E5000000       jnb 003843B1

003842CC    8A08                mov cl,byte ptr ds:[eax]

003842CE    80F9 01             cmp cl,1

003842D1    75 5F               jnz short 00384332

003842D3    66:8B48 02          mov cx,word ptr ds:[eax+2]

003842D7    66:0BC9             or cx,cx

003842DA    75 19               jnz short 003842F5

003842DC    8B70 04             mov esi,dword ptr ds:[eax+4]

003842DF    83AD D4414100 08    sub dword ptr ss:[ebp+4141D4],8

003842E6    8BBD D4414100        mov edi,dword ptr ss:[ebp+4141D4]

003842EC    B9 02000000         mov ecx,2

003842F1    F3:A5               rep movs dword ptr es:[edi],dword ptr>

003842F3    EB 36               jmp short 0038432B

003842F5    66:83F9 01          cmp cx,1

003842F9    75 17               jnz short 00384312

003842FB    8B40 04             mov eax,dword ptr ds:[eax+4]

003842FE    DB00                fild dword ptr ds:[eax]

00384300    83AD D4414100 08    sub dword ptr ss:[ebp+4141D4],8

00384307    8B85 D4414100        mov eax,dword ptr ss:[ebp+4141D4]

0038430D    DD18                fstp qword ptr ds:[eax]

0038430F    9B                  wait

00384310    EB 19               jmp short 0038432B

00384312    66:83F9 02          cmp cx,2

00384316    75 13               jnz short 0038432B

00384318    DB40 04             fild dword ptr ds:[eax+4]

0038431B    83AD D4414100 08    sub dword ptr ss:[ebp+4141D4],8

00384322    8B85 D4414100        mov eax,dword ptr ss:[ebp+4141D4]

00384328    DD18                fstp qword ptr ds:[eax]

0038432A    9B                  wait

0038432B    61                  popad

0038432C    58                  pop eax

0038432D    C2 0800             retn 8

00384330    EB 7F               jmp short 003843B1

00384332    80F9 02             cmp cl,2

00384335    75 44               jnz short 0038437B

00384337    66:8B48 02          mov cx,word ptr ds:[eax+2]

0038433B    66:0BC9             or cx,cx

0038433E    75 19               jnz short 00384359

00384340    8B78 04             mov edi,dword ptr ds:[eax+4]

00384343    8BB5 D4414100        mov esi,dword ptr ss:[ebp+4141D4]

00384349    B9 02000000         mov ecx,2

0038434E    F3:A5               rep movs dword ptr es:[edi],dword ptr>

00384350    8385 D4414100 08    add dword ptr ss:[ebp+4141D4],8

00384357    EB 1B               jmp short 00384374

00384359    66:83F9 01          cmp cx,1

0038435D    75 15               jnz short 00384374

0038435F    8B8D D4414100        mov ecx,dword ptr ss:[ebp+4141D4]

00384365    DD01                fld qword ptr ds:[ecx]

00384367    8B40 04             mov eax,dword ptr ds:[eax+4]

0038436A    DB18                fistp dword ptr ds:[eax]

0038436C    9B                  wait

0038436D    8385 D4414100 08    add dword ptr ss:[ebp+4141D4],8

00384374    61                  popad

00384375    58                  pop eax

00384376    C2 0800             retn 8

00384379    EB 36               jmp short 003843B1

0038437B    80F9 03             cmp cl,3

0038437E    75 17               jnz short 00384397

00384380    8B48 02             mov ecx,dword ptr ds:[eax+2]

00384383    FF71 08             push dword ptr ds:[ecx+8]

00384386    FF71 04             push dword ptr ds:[ecx+4]

00384389    FF31                push dword ptr ds:[ecx]

0038438B    E8 F9ECFFFF         call 00383089

00384390    61                  popad

00384391    58                  pop eax

00384392    C2 0800             retn 8

00384395    EB 1A               jmp short 003843B1

00384397    80F9 04             cmp cl,4

0038439A    75 15               jnz short 003843B1

0038439C    8B48 02             mov ecx,dword ptr ds:[eax+2]

0038439F    FF71 08             push dword ptr ds:[ecx+8]

003843A2    FF71 04             push dword ptr ds:[ecx+4]

003843A5    FF31                push dword ptr ds:[ecx]

003843A7    E8 7EECFFFF         call 0038302A

003843AC    61                  popad

003843AD    58                  pop eax

003843AE    C2 0800             retn 8

003843B1    8B85 30414100        mov eax,dword ptr ss:[ebp+414130]    //ss:[003852DD]

=00384FB0

eax=77E40000 (kernel32.77E40000)

003843B7    894424 20           mov dword ptr ss:[esp+20],eax

003843BB    61                  popad

003843BC    83C4 04             add esp,4

003843BF    FF6424 FC           jmp dword ptr ss:[esp-4]

003843C3    EB 03               jmp short 003843C8

003843C5    CD20 E9E94D04       vxdjump 44DE9E9

003843CB    0000                add byte ptr ds:[eax],al

003843CD    83F8 07             cmp eax,7         //第七个 4575be<jmp.&kernel32.GetVersion>

003843D0    75 2A               jnz short 003843FC

003843D2    50                  push eax

003843D3    60                  pushad

003843D4    E8 00000000         call 003843D9

003843D9    5D                  pop ebp

003843DA    81ED 2C324100       sub ebp,41322C

003843E0    8B85 E0454100        mov eax,dword ptr ss:[ebp+4145E0]

003843E6    894424 20           mov dword ptr ss:[esp+20],eax

003843EA    61                  popad

003843EB    58                  pop eax

003843EC    83C4 04             add esp,4

003843EF    FF6424 FC           jmp dword ptr ss:[esp-4]

003843F3    EB 02               jmp short 003843F7

003843F5    0F0F                ???                           ; 未知命令

003843F7    E9 1E040000         jmp 0038481A

003843FC    83F8 08             cmp eax,8          //第八个 jmp.&kernel32.GetModuleHandleA>

003843FF    75 53               jnz short 00384454

00384401    50                  push eax

00384402    60                  pushad

00384403    E8 00000000         call 00384408

00384408    5D                  pop ebp

00384409    81ED 5B324100       sub ebp,41325B

0038440F    8B4424 28           mov eax,dword ptr ss:[esp+28]

00384413    0BC0                or eax,eax

00384415    75 0F               jnz short 00384426

00384417    8B85 E4454100        mov eax,dword ptr ss:[ebp+4145E4]

0038441D    894424 20           mov dword ptr ss:[esp+20],eax

00384421    61                  popad

00384422    58                  pop eax

00384423    C2 0400             retn 4

00384426    8B85 3F434100        mov eax,dword ptr ss:[ebp+41433F]   //ss:[003854EC]=77E59F93

(kernel32.GetModuleHandleA)

eax=0040E678 (main.0040E678), ASCII "kernel32.dll"

0038442C    894424 20           mov dword ptr ss:[esp+20],eax

00384430    61                  popad

00384431    83C4 04             add esp,4

00384434    FF6424 FC           jmp dword ptr ss:[esp-4]

00384438    E8 04000000         call 00384441

0038443D    C783 EB0E58EB 02CD2>mov dword ptr ds:[ebx+EB580EEB],8320C>

00384447    C002 EB             rol byte ptr ds:[edx],0EB

0038444A    01E9                add ecx,ebp

0038444C    50                  push eax

0038444D    C3                  retn

0038444E    E8 E9C60300         call 003C0B3C

00384453    90                  nop

00384454    83F8 09             cmp eax,9          //第九个 没有用到

00384457    75 3A               jnz short 00384493

00384459    50                  push eax

0038445A    60                  pushad

0038445B    E8 00000000         call 00384460

00384460    5D                  pop ebp

00384461    81ED B3324100       sub ebp,4132B3

00384467    8B85 E8454100        mov eax,dword ptr ss:[ebp+4145E8]

0038446D    894424 20           mov dword ptr ss:[esp+20],eax
```
