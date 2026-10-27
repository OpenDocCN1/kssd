
```
004BE6DC   |.  8BC8              mov ecx,eax
004BE6DE   |.  C1E9 10           shr ecx,10
004BE6E1   |.  81E1 FF000000     and ecx,0FF
004BE6E7   |.  03148F            add edx,dword ptr ds:[edi+ecx*4]
004BE6EA   |.  8BC8              mov ecx,eax
004BE6EC   |.  C1E9 08           shr ecx,8
004BE6EF   |.  81E1 FF000000     and ecx,0FF
004BE6F5   |.  33148D 48B25500   xor edx,dword ptr ds:[ecx*4+55B248]
004BE6FC   |.  25 FF000000       and eax,0FF
004BE701   |.  2B1485 48B65500   sub edx,dword ptr ds:[eax*4+55B648]
004BE708   |.  315424 04         xor dword ptr ss:[esp+4],edx              ;  存储了AEAD0BA5
004BE70C   |.  8B53 6C           mov edx,dword ptr ds:[ebx+6C]
004BE70F   |.  8B43 2C           mov eax,dword ptr ds:[ebx+2C]
004BE712   |.  334424 04         xor eax,dword ptr ss:[esp+4]
004BE716   |.  E8 4DEEFFFF       call 12\_.004BD568
004BE71B   |.  8BD0              mov edx,eax
004BE71D   |.  C1EA 18           shr edx,18
004BE720   |.  8B1496            mov edx,dword ptr ds:[esi+edx*4]
004BE723   |.  8BC8              mov ecx,eax
004BE725   |.  C1E9 10           shr ecx,10
004BE728   |.  81E1 FF000000     and ecx,0FF
004BE72E   |.  2B148F            sub edx,dword ptr ds:[edi+ecx*4]
004BE731   |.  8BC8              mov ecx,eax
004BE733   |.  C1E9 08           shr ecx,8
004BE736   |.  81E1 FF000000     and ecx,0FF
004BE73C   |.  03148D 48B25500   add edx,dword ptr ds:[ecx*4+55B248]
004BE743   |.  25 FF000000       and eax,0FF
004BE748   |.  331485 48B65500   xor edx,dword ptr ds:[eax*4+55B648]
004BE74F   |.  315424 08         xor dword ptr ss:[esp+8],edx              ;  存储了7982365F
004BE753   |.  8B53 68           mov edx,dword ptr ds:[ebx+68]
004BE756   |.  8B43 28           mov eax,dword ptr ds:[ebx+28]
004BE759   |.  034424 08         add eax,dword ptr ss:[esp+8]
004BE75D   |.  E8 06EEFFFF       call 12\_.004BD568
004BE762   |.  8BD0              mov edx,eax
004BE764   |.  C1EA 18           shr edx,18
004BE767   |.  8B1496            mov edx,dword ptr ds:[esi+edx*4]
004BE76A   |.  8BC8              mov ecx,eax
004BE76C   |.  C1E9 10           shr ecx,10
004BE76F   |.  81E1 FF000000     and ecx,0FF
004BE775   |.  33148F            xor edx,dword ptr ds:[edi+ecx*4]
004BE778   |.  8BC8              mov ecx,eax
004BE77A   |.  C1E9 08           shr ecx,8
004BE77D   |.  81E1 FF000000     and ecx,0FF
004BE783   |.  2B148D 48B25500   sub edx,dword ptr ds:[ecx*4+55B248]
004BE78A   |.  25 FF000000       and eax,0FF
004BE78F   |.  031485 48B65500   add edx,dword ptr ds:[eax*4+55B648]
004BE796   |.  315424 04         xor dword ptr ss:[esp+4],edx              ;  存储了DFC61D90
004BE79A   |.  8B53 64           mov edx,dword ptr ds:[ebx+64]
004BE79D   |.  8B43 24           mov eax,dword ptr ds:[ebx+24]
004BE7A0   |.  2B4424 04         sub eax,dword ptr ss:[esp+4]
004BE7A4   |.  E8 BFEDFFFF       call 12\_.004BD568
004BE7A9   |.  8BD0              mov edx,eax
004BE7AB   |.  C1EA 18           shr edx,18
004BE7AE   |.  8B1496            mov edx,dword ptr ds:[esi+edx*4]
004BE7B1   |.  8BC8              mov ecx,eax
004BE7B3   |.  C1E9 10           shr ecx,10
004BE7B6   |.  81E1 FF000000     and ecx,0FF
004BE7BC   |.  03148F            add edx,dword ptr ds:[edi+ecx*4]
004BE7BF   |.  8BC8              mov ecx,eax
004BE7C1   |.  C1E9 08           shr ecx,8
004BE7C4   |.  81E1 FF000000     and ecx,0FF
004BE7CA   |.  33148D 48B25500   xor edx,dword ptr ds:[ecx*4+55B248]
004BE7D1   |.  25 FF000000       and eax,0FF
004BE7D6   |.  2B1485 48B65500   sub edx,dword ptr ds:[eax*4+55B648]
004BE7DD   |.  315424 08         xor dword ptr ss:[esp+8],edx              ;  存储了   61B1ACF7
004BE7E1   |.  8B53 60           mov edx,dword ptr ds:[ebx+60]
004BE7E4   |.  8B43 20           mov eax,dword ptr ds:[ebx+20]
004BE7E7   |.  334424 08         xor eax,dword ptr ss:[esp+8]
004BE7EB   |.  E8 78EDFFFF       call 12\_.004BD568
004BE7F0   |.  8BD0              mov edx,eax
004BE7F2   |.  C1EA 18           shr edx,18
004BE7F5   |.  8B1496            mov edx,dword ptr ds:[esi+edx*4]
004BE7F8   |.  8BC8              mov ecx,eax
004BE7FA   |.  C1E9 10           shr ecx,10
004BE7FD   |.  81E1 FF000000     and ecx,0FF
004BE803   |.  2B148F            sub edx,dword ptr ds:[edi+ecx*4]
004BE806   |.  8BC8              mov ecx,eax
004BE808   |.  C1E9 08           shr ecx,8
004BE80B   |.  81E1 FF000000     and ecx,0FF
004BE811   |.  03148D 48B25500   add edx,dword ptr ds:[ecx*4+55B248]
004BE818   |.  25 FF000000       and eax,0FF
004BE81D   |.  331485 48B65500   xor edx,dword ptr ds:[eax*4+55B648]
004BE824   |.  315424 04         xor dword ptr ss:[esp+4],edx              ;  存储了   0CF9CD3C
004BE828   |.  8B53 5C           mov edx,dword ptr ds:[ebx+5C]             ;  存储了   0CF9CD3C
004BE82B   |.  8B43 1C           mov eax,dword ptr ds:[ebx+1C]
004BE82E   |.  034424 04         add eax,dword ptr ss:[esp+4]
004BE832   |.  E8 31EDFFFF       call 12\_.004BD568
004BE837   |.  8BD0              mov edx,eax
004BE839   |.  C1EA 18           shr edx,18
004BE83C   |.  8B1496            mov edx,dword ptr ds:[esi+edx*4]
004BE83F   |.  8BC8              mov ecx,eax
004BE841   |.  C1E9 10           shr ecx,10
004BE844   |.  81E1 FF000000     and ecx,0FF
004BE84A   |.  33148F            xor edx,dword ptr ds:[edi+ecx*4]
004BE84D   |.  8BC8              mov ecx,eax
004BE84F   |.  C1E9 08           shr ecx,8
004BE852   |.  81E1 FF000000     and ecx,0FF
004BE858   |.  2B148D 48B25500   sub edx,dword ptr ds:[ecx*4+55B248]
004BE85F   |.  25 FF000000       and eax,0FF
004BE864   |.  031485 48B65500   add edx,dword ptr ds:[eax*4+55B648]
004BE86B   |.  315424 08         xor dword ptr ss:[esp+8],edx              ;  存储了1  结果为DB843288
004BE86F   |.  8B53 58           mov edx,dword ptr ds:[ebx+58]
004BE872   |.  8B43 18           mov eax,dword ptr ds:[ebx+18]
004BE875   |.  2B4424 08         sub eax,dword ptr ss:[esp+8]
004BE879   |.  E8 EAECFFFF       call 12\_.004BD568
004BE87E   |.  8BD0              mov edx,eax
004BE880   |.  C1EA 18           shr edx,18
004BE883   |.  8B1496            mov edx,dword ptr ds:[esi+edx*4]
004BE886   |.  8BC8              mov ecx,eax
004BE888   |.  C1E9 10           shr ecx,10
004BE88B   |.  81E1 FF000000     and ecx,0FF
004BE891   |.  03148F            add edx,dword ptr ds:[edi+ecx*4]
004BE894   |.  8BC8              mov ecx,eax
004BE896   |.  C1E9 08           shr ecx,8
004BE899   |.  81E1 FF000000     and ecx,0FF
004BE89F   |.  33148D 48B25500   xor edx,dword ptr ds:[ecx*4+55B248]
004BE8A6   |.  25 FF000000       and eax,0FF
004BE8AB   |.  2B1485 48B65500   sub edx,dword ptr ds:[eax*4+55B648]
004BE8B2   |.  315424 04         xor dword ptr ss:[esp+4],edx              ;  第三次下断改结果为7755FA5B  再与第一次断点处得到004BE86B得值DB843288
004BE8B6   |.  8B53 54           mov edx,dword ptr ds:[ebx+54]
004BE8B9   |.  8B43 14           mov eax,dword ptr ds:[ebx+14]
004BE8BC   |.  334424 04         xor eax,dword ptr ss:[esp+4]
004BE8C0   |.  E8 A3ECFFFF       call 12\_.004BD568
004BE8C5   |.  8BD0              mov edx,eax
004BE8C7   |.  C1EA 18           shr edx,18
004BE8CA   |.  8B1496            mov edx,dword ptr ds:[esi+edx*4]
004BE8CD   |.  8BC8              mov ecx,eax
004BE8CF   |.  C1E9 10           shr ecx,10
004BE8D2   |.  81E1 FF000000     and ecx,0FF
004BE8D8   |.  2B148F            sub edx,dword ptr ds:[edi+ecx*4]
004BE8DB   |.  8BC8              mov ecx,eax
004BE8DD   |.  C1E9 08           shr ecx,8
004BE8E0   |.  81E1 FF000000     and ecx,0FF
004BE8E6   |.  03148D 48B25500   add edx,dword ptr ds:[ecx*4+55B248]
004BE8ED   |.  25 FF000000       and eax,0FF
004BE8F2   |.  331485 48B65500   xor edx,dword ptr ds:[eax*4+55B648]
004BE8F9   |.  315424 08         xor dword ptr ss:[esp+8],edx              ;  第一次下断   改ESP+8=31353330
004BE8FD   |.  8B53 50           mov edx,dword ptr ds:[ebx+50]
004BE900   |.  8B43 10           mov eax,dword ptr ds:[ebx+10]
```
