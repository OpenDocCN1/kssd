
```assembly
00409D3A   |.   8D8C24 900D0000    lea ecx,dword ptr ss:[esp+D90]        //ecx 指向注册码  

00409D41   |.   50                 push eax             //用户名入桟  

00409D42   |.   51                 push ecx             //注册码入桟  

00409D43   |.   E8 28F9FFFF        call CCProxy.00409670           //关键的 call  

00409D48   |.   83C4 08            add esp,8              //平衡堆栈  

00409D4B   |.   A3 D4F65100        mov dword ptr ds:[51F6D4],eax        //将返回值保存到地址 51F6D4，注册标志  

00409D50   |.   85C0               test eax,eax             //判断，eax=1 注册，eax=0 没注册  

00409D52   |.   0F84 7B010000      je CCProxy.00409ED3          //作者为了防止爆破，在很多地方进行  

        //这种判断，还用了 timer，但始终用 call CCProxy.00409670，所以爆破很简单，进入这个 call  

        //直接修改为 mov eax,1,ret。所以编程一定不要规范，呵呵。  

        //好了，让我们进入 call CCProxy.00409670，精彩的算法还在后面。  

00409D58   |.   80BC24 91010000 3>cmp byte ptr ss:[esp+191],30           

00409D60   |.   0F85 6D010000      jnz CCProxy.00409ED3  

00409D66   |.   8A15 B03A4700      mov dl,byte ptr ds:[473AB0]  

00409D6C   |.   B9 41000000        mov ecx,41  

00409D71   |.   33C0               xor eax,eax  

00409D73   |.   8DBC24 85000000    lea edi,dword ptr ss:[esp+85]  

00409D7A   |.   889424 84000000    mov byte ptr ss:[esp+84],dl  

00409D81   |.   68 04010000        push 104                              ; /BufSize = 104 (260.)  

00409D86   |.   F3:AB              rep stos dword ptr es:[edi]           ; |  

00409D88   |.   8D8424 88000000    lea eax,dword ptr ss:[esp+88]         ; |  

00409D8F   |.   C68424 9C010000 0>mov byte ptr ss:[esp+19C],0           ; |  

00409D97   |.   50                 push eax                              ; |Buffer  

00409D98   |.   FF15 6CE24500      call dword ptr ds:[<&KERNEL32.GetWindowsDir>; \\GetWindowsDirectoryA  

00409D9E   |.   BF D8F04600        mov edi,CCProxy.0046F0D8             ;  ASCII "\\Win.ini"  

00409DA3   |.   83C9 FF            or ecx,FFFFFFFF  

00409DA6   |.   33C0               xor eax,eax  

00409DA8   |.   8D9424 84000000    lea edx,dword ptr ss:[esp+84]  

00409DAF   |.   F2:AE              repne scas byte ptr es:[edi]  

00409DB1   |.   F7D1               not ecx  

00409DB3   |.   2BF9               sub edi,ecx  

00409DB5   |.   8BF7               mov esi,edi  

00409DB7   |.   8BE9               mov ebp,ecx  

00409DB9   |.   8BFA               mov edi,edx  

00409DBB   |.   83C9 FF            or ecx,FFFFFFFF  

00409DBE   |.   F2:AE              repne scas byte ptr es:[edi]  

00409DC0   |.   8BCD               mov ecx,ebp  

00409DC2   |.   4F                 dec edi  

00409DC3   |.   C1E9 02            shr ecx,2  

00409DC6   |.   F3:A5              rep movs dword ptr es:[edi],dword ptr ds:[e>  

00409DC8   |.   8BCD               mov ecx,ebp  

00409DCA   |.   8D8424 84000000    lea eax,dword ptr ss:[esp+84]  

00409DD1   |.   83E1 03            and ecx,3  

00409DD4   |.   50                 push eax                              ; /IniFileName  

00409DD5   |.   F3:A4              rep movs byte ptr es:[edi],byte ptr ds:[esi\>; |  

00409DD7   |.   8D8C24 90010000    lea ecx,dword ptr ss:[esp+190]       ; |  

00409DDE   |.   6A 00              push 0                               ; |Default = 0  

00409DE0   |.   51                 push ecx                             ; |Key  

00409DE1   |.   68 D0F04600        push CCProxy.0046F0D0               ; |Section = "Product"  

00409DE6   |.   FF15 70E24500      call dword ptr ds:[<&KERNEL32.GetPrivatePro>; \\GetPrivateProfileIntA  

00409DEC   |.   8D5424 10          lea edx,dword ptr ss:[esp+10]  

00409DF0   |.   8BE8               mov ebp,eax  

00409DF2   |.   52                 push edx                              ; /pSystemTime  

00409DF3   |.   FF15 94E24500      call dword ptr ds:[<&KERNEL32.GetSystemTime>; \\GetSystemTime  

00409DF9   |.   8D4424 10          lea eax,dword ptr ss:[esp+10]  

00409DFD   |.   6A FF              push -1                             ; /Arg2 = FFFFFFFF  

00409DFF   |.   50                 push eax                             ; |Arg1  

00409E00   |.   8D4C24 14          lea ecx,dword ptr ss:[esp+14]         ; |  

00409E04   |.   E8 D1150400        call CCProxy.0044B3DA               ; \\CCProxy.0044B3DA  

00409E09   |.   6A 00              push 0  

00409E0B   |.   8D4C24 10          lea ecx,dword ptr ss:[esp+10]  

00409E0F   |.   E8 71160400        call CCProxy.0044B485  

00409E14   |.   8B70 14            mov esi,dword ptr ds:[eax+14]  

00409E17   |.   6A 00              push 0  

00409E19   |.   8D4C24 10          lea ecx,dword ptr ss:[esp+10]  

00409E1D   |.   81C6 6C070000      add esi,76C  

00409E23   |.   E8 5D160400        call CCProxy.0044B485  

00409E28   |.   8B78 10            mov edi,dword ptr ds:[eax+10]  

00409E2B   |.   6A 00              push 0  

00409E2D   |.   8D4C24 10          lea ecx,dword ptr ss:[esp+10]  

00409E31   |.   47                 inc edi  

00409E32   |.   E8 4E160400        call CCProxy.0044B485  

00409E37   |.   8D14F6             lea edx,dword ptr ds:[esi+esi*8]  

00409E3A   |.   8BC8               mov ecx,eax  

00409E3C   |.   8B49 0C            mov ecx,dword ptr ds:[ecx+C]  

00409E3F   |.   8D04D6             lea eax,dword ptr ds:[esi+edx*8]  

00409E42   |.   8BD7               mov edx,edi  

00409E44   |.   8D0480             lea eax,dword ptr ds:[eax+eax*4]  

00409E47   |.   C1E2 04            shl edx,4  

00409E4A   |.   03C8               add ecx,eax  

00409E4C   |.   2BD7               sub edx,edi  

00409E4E   |.   85ED               test ebp,ebp  

00409E50   |.   8D1451             lea edx,dword ptr ds:[ecx+edx*2]  

00409E53   |.   75 5A              jnz short CCProxy.00409EAF  

00409E55   |.   A0 B03A4700        mov al,byte ptr ds:[473AB0]  

00409E5A   |.   B9 FF000000        mov ecx,0FF  

00409E5F   |.   888424 90090000    mov byte ptr ss:[esp+990],al  

00409E66   |.   33C0               xor eax,eax  

00409E68   |.   8DBC24 91090000    lea edi,dword ptr ss:[esp+991]  

00409E6F   |.   52                 push edx  

00409E70   |.   F3:AB              rep stos dword ptr es:[edi]  

00409E72   |.   66:AB              stos word ptr es:[edi]  

00409E74   |.   8D8C24 94090000    lea ecx,dword ptr ss:[esp+994]  

00409E7B   |.   68 50E14600        push CCProxy.0046E150               ;  ASCII "%d"  

00409E80   |.   51                 push ecx  

00409E81   |.   AA                 stos byte ptr es:[edi]  

00409E82   |.   E8 3D110300        call CCProxy.0043AFC4  

00409E87   |.   83C4 0C            add esp,0C  

00409E8A   |.   8D9424 84000000    lea edx,dword ptr ss:[esp+84]  

00409E91   |.   8D8424 90090000    lea eax,dword ptr ss:[esp+990]  

00409E98   |.   8D8C24 8C010000    lea ecx,dword ptr ss:[esp+18C]  

00409E9F   |.   52                 push edx                              ; /FileName  

00409EA0   |.   50                 push eax                             ; |String  

00409EA1   |.   51                 push ecx                             ; |Key  

00409EA2   |.   68 D0F04600        push CCProxy.0046F0D0               ; |Section = "Product"  

00409EA7   |.   FF15 88E24500      call dword ptr ds:[<&KERNEL32.WritePrivateP>; \\WritePrivateProfileStringA  

00409EAD   |.   EB 24              jmp short CCProxy.00409ED3  

00409EAF   |>  8BF2               mov esi,edx  

00409EB1   |.   2BF5               sub esi,ebp  

00409EB3   |.   78 14              js short CCProxy.00409EC9  

00409EB5   |.   8D9424 96010000    lea edx,dword ptr ss:[esp+196]  

00409EBC   |.   52                 push edx  

00409EBD   |.   E8 C71E0300        call CCProxy.0043BD89  

00409EC2   |.   83C4 04            add esp,4  

00409EC5   |.   3BF0               cmp esi,eax  

00409EC7   |.   7C 0A              jl short CCProxy.00409ED3  

00409EC9   |>  C705 D4F65100 000>mov dword ptr ds:[51F6D4],0  

00409ED3   |>  B0 43              mov al,43  

00409ED5   |.   BF 4CE14600        mov edi,CCProxy.0046E14C  
```
