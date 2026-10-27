
```
004BE4B7   |.  894424 08         mov dword ptr ss:[esp+8],eax              ;  存储把原来78563412换成12345678

004BE4BB   |.  83BB 90000000 0C  cmp dword ptr ds:[ebx+90],0C              ;  上面把密码反过来了

004BE4C2   |.  0F8E 28010000     jle 12\_.004BE5F0

004BE4C8   |.  8B93 8C000000     mov edx,dword ptr ds:[ebx+8C]             ;  固定值11

004BE4CE   |.  8B43 4C           mov eax,dword ptr ds:[ebx+4C]             ;  固定值BC62FF29

004BE4D1   |.  034424 04         add eax,dword ptr ss:[esp+4]              ;  固定值BC62FF29与输入码的后半部分90ABCDEF相加

004BE4D5   |.  E8 8EF0FFFF       call 12\_.004BD568

004BE4DA   |.  8BD0              mov edx,eax

004BE4DC   |.  C1EA 18           shr edx,18

004BE4DF   |.  8B1496            mov edx,dword ptr ds:[esi+edx\*4]

004BE4E2   |.  8BC8              mov ecx,eax

004BE4E4   |.  C1E9 10           shr ecx,10

004BE4E7   |.  81E1 FF000000     and ecx,0FF

004BE4ED   |.  33148F            xor edx,dword ptr ds:[edi+ecx\*4]

004BE4F0   |.  8BC8              mov ecx,eax

004BE4F2   |.  C1E9 08           shr ecx,8

004BE4F5   |.  81E1 FF000000     and ecx,0FF

004BE4FB   |.  2B148D 48B25500   sub edx,dword ptr ds:[ecx\*4+55B248]

004BE502   |.  25 FF000000       and eax,0FF

004BE507   |.  031485 48B65500   add edx,dword ptr ds:[eax\*4+55B648]

004BE50E   |.  315424 08         xor dword ptr ss:[esp+8],edx              ;  存储了39BDCFAE  真实的前8位密码53686CE7

004BE512   |.  8B93 88000000     mov edx,dword ptr ds:[ebx+88]             ;  固定值11

004BE518   |.  8B43 48           mov eax,dword ptr ds:[ebx+48]             ;  固定值D1FA7D9E

004BE51B   |.  2B4424 08         sub eax,dword ptr ss:[esp+8]

004BE51F   |.  E8 44F0FFFF       call 12\_.004BD568

004BE524   |.  8BD0              mov edx,eax

004BE526   |.  C1EA 18           shr edx,18

004BE529   |.  8B1496            mov edx,dword ptr ds:[esi+edx\*4]

004BE52C   |.  8BC8              mov ecx,eax

004BE52E   |.  C1E9 10           shr ecx,10

004BE531   |.  81E1 FF000000     and ecx,0FF

004BE537   |.  03148F            add edx,dword ptr ds:[edi+ecx\*4]

004BE53A   |.  8BC8              mov ecx,eax

004BE53C   |.  C1E9 08           shr ecx,8

004BE53F   |.  81E1 FF000000     and ecx,0FF

004BE545   |.  33148D 48B25500   xor edx,dword ptr ds:[ecx\*4+55B248]

004BE54C   |.  25 FF000000       and eax,0FF

004BE551   |.  2B1485 48B65500   sub edx,dword ptr ds:[eax\*4+55B648]

004BE558   |.  315424 04         xor dword ptr ss:[esp+4],edx              ;  存储了B316C2A2真实的后8位密码B63E5BCA

004BE55C   |.  8B93 84000000     mov edx,dword ptr ds:[ebx+84]

004BE562   |.  8B43 44           mov eax,dword ptr ds:[ebx+44]

004BE565   |.  334424 04         xor eax,dword ptr ss:[esp+4]

004BE569   |.  E8 FAEFFFFF       call 12\_.004BD568

004BE56E   |.  8BD0              mov edx,eax

004BE570   |.  C1EA 18           shr edx,18

004BE573   |.  8B1496            mov edx,dword ptr ds:[esi+edx\*4]

004BE576   |.  8BC8              mov ecx,eax

004BE578   |.  C1E9 10           shr ecx,10

004BE57B   |.  81E1 FF000000     and ecx,0FF

004BE581   |.  2B148F            sub edx,dword ptr ds:[edi+ecx\*4]

004BE584   |.  8BC8              mov ecx,eax

004BE586   |.  C1E9 08           shr ecx,8

004BE589   |.  81E1 FF000000     and ecx,0FF

004BE58F   |.  03148D 48B25500   add edx,dword ptr ds:[ecx\*4+55B248]

004BE596   |.  25 FF000000       and eax,0FF

004BE59B   |.  331485 48B65500   xor edx,dword ptr ds:[eax\*4+55B648]

004BE5A2   |.  315424 08         xor dword ptr ss:[esp+8],edx              ;  存储了B217DF05

004BE5A6   |.  8B93 80000000     mov edx,dword ptr ds:[ebx+80]

004BE5AC   |.  8B43 40           mov eax,dword ptr ds:[ebx+40]

004BE5AF   |.  034424 08         add eax,dword ptr ss:[esp+8]

004BE5B3   |.  E8 B0EFFFFF       call 12\_.004BD568

004BE5B8   |.  8BD0              mov edx,eax

004BE5BA   |.  C1EA 18           shr edx,18

004BE5BD   |.  8B1496            mov edx,dword ptr ds:[esi+edx\*4]

004BE5C0   |.  8BC8              mov ecx,eax

004BE5C2   |.  C1E9 10           shr ecx,10

004BE5C5   |.  81E1 FF000000     and ecx,0FF

004BE5CB   |.  33148F            xor edx,dword ptr ds:[edi+ecx\*4]

004BE5CE   |.  8BC8              mov ecx,eax

004BE5D0   |.  C1E9 08           shr ecx,8

004BE5D3   |.  81E1 FF000000     and ecx,0FF

004BE5D9   |.  2B148D 48B25500   sub edx,dword ptr ds:[ecx\*4+55B248]

004BE5E0   |.  25 FF000000       and eax,0FF

004BE5E5   |.  031485 48B65500   add edx,dword ptr ds:[eax\*4+55B648]

004BE5EC   |.  315424 04         xor dword ptr ss:[esp+4],edx              ;  存储了10E19E3A

004BE5F0   |>  8B53 7C           mov edx,dword ptr ds:[ebx+7C]

004BE5F3   |.  8B43 3C           mov eax,dword ptr ds:[ebx+3C]

004BE5F6   |.  2B4424 04         sub eax,dword ptr ss:[esp+4]

004BE5FA   |.  E8 69EFFFFF       call 12\_.004BD568

004BE5FF   |.  8BD0              mov edx,eax

004BE601   |.  C1EA 18           shr edx,18

004BE604   |.  8B1496            mov edx,dword ptr ds:[esi+edx\*4]

004BE607   |.  8BC8              mov ecx,eax

004BE609   |.  C1E9 10           shr ecx,10

004BE60C   |.  81E1 FF000000     and ecx,0FF

004BE612   |.  03148F            add edx,dword ptr ds:[edi+ecx\*4]

004BE615   |.  8BC8              mov ecx,eax

004BE617   |.  C1E9 08           shr ecx,8

004BE61A   |.  81E1 FF000000     and ecx,0FF

004BE620   |.  33148D 48B25500   xor edx,dword ptr ds:[ecx\*4+55B248]

004BE627   |.  25 FF000000       and eax,0FF

004BE62C   |.  2B1485 48B65500   sub edx,dword ptr ds:[eax\*4+55B648]

004BE633   |.  315424 08         xor dword ptr ss:[esp+8],edx              ;  存储了A5F8B413

004BE637   |.  8B53 78           mov edx,dword ptr ds:[ebx+78]

004BE63A   |.  8B43 38           mov eax,dword ptr ds:[ebx+38]

004BE63D   |.  334424 08         xor eax,dword ptr ss:[esp+8]

004BE641   |.  E8 22EFFFFF       call 12\_.004BD568

004BE646   |.  8BD0              mov edx,eax

004BE648   |.  C1EA 18           shr edx,18

004BE64B   |.  8B1496            mov edx,dword ptr ds:[esi+edx\*4]

004BE64E   |.  8BC8              mov ecx,eax

004BE650   |.  C1E9 10           shr ecx,10

004BE653   |.  81E1 FF000000     and ecx,0FF

004BE659   |.  2B148F            sub edx,dword ptr ds:[edi+ecx\*4]

004BE65C   |.  8BC8              mov ecx,eax

004BE65E   |.  C1E9 08           shr ecx,8

004BE661   |.  81E1 FF000000     and ecx,0FF

004BE667   |.  03148D 48B25500   add edx,dword ptr ds:[ecx\*4+55B248]

004BE66E   |.  25 FF000000       and eax,0FF

004BE673   |.  331485 48B65500   xor edx,dword ptr ds:[eax\*4+55B648]

004BE67A   |.  315424 04         xor dword ptr ss:[esp+4],edx              ;  存储了EAC77CE9

004BE67E   |.  8B53 74           mov edx,dword ptr ds:[ebx+74]

004BE681   |.  8B43 34           mov eax,dword ptr ds:[ebx+34]

004BE684   |.  034424 04         add eax,dword ptr ss:[esp+4]

004BE688   |.  E8 DBEEFFFF       call 12\_.004BD568

004BE68D   |.  8BD0              mov edx,eax

004BE68F   |.  C1EA 18           shr edx,18

004BE692   |.  8B1496            mov edx,dword ptr ds:[esi+edx\*4]

004BE695   |.  8BC8              mov ecx,eax

004BE697   |.  C1E9 10           shr ecx,10

004BE69A   |.  81E1 FF000000     and ecx,0FF

004BE6A0   |.  33148F            xor edx,dword ptr ds:[edi+ecx\*4]

004BE6A3   |.  8BC8              mov ecx,eax

004BE6A5   |.  C1E9 08           shr ecx,8

004BE6A8   |.  81E1 FF000000     and ecx,0FF

004BE6AE   |.  2B148D 48B25500   sub edx,dword ptr ds:[ecx\*4+55B248]

004BE6B5   |.  25 FF000000       and eax,0FF

004BE6BA   |.  031485 48B65500   add edx,dword ptr ds:[eax\*4+55B648]

004BE6C1   |.  315424 08         xor dword ptr ss:[esp+8],edx              ;  存储了E423B09C

004BE6C5   |.  8B53 70           mov edx,dword ptr ds:[ebx+70]

004BE6C8   |.  8B43 30           mov eax,dword ptr ds:[ebx+30]

004BE6CB   |.  2B4424 08         sub eax,dword ptr ss:[esp+8]

004BE6CF   |.  E8 94EEFFFF       call 12\_.004BD568

004BE6D4   |.  8BD0              mov edx,eax

004BE6D6   |.  C1EA 18           shr edx,18

004BE6D9   |.  8B1496            mov edx,dword ptr ds:[esi+edx\*4]
```
