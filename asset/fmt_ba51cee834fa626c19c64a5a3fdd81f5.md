
0044028D       .  E8 BEF3FFFF              call unpack.0043F650                   ; \在 CODE 节里搜索特征码  

00440292       .  84C0                      test al,al  

00440294        74 07                       je short unpack.0044029D  

00440296       .  68 5C634000               push unpack.0040635C                   ;  ASCII "Borland Delphi 6.0 - 7.0"  

0044029B       .  EB 37                    jmp short unpack.004402D4  

0044029D       >  6A 2D                    push 2D  

0044029F       .  8D5424 6C                lea edx,dword ptr ss:\[esp+6C\]  

004402A3       .  52                        push edx  

004402A4       .  8D8C24 90010000          lea ecx,dword ptr ss:\[esp+190\]  

004402AB       .  E8 9070FFFF              call unpack.00437340  

004402B0       .  8B0E                      mov ecx,dword ptr ds:\[esi\]  

004402B2       .  8D4424 10                lea eax,dword ptr ss:\[esp+10\]  

004402B6       .  50                        push eax                                 ; /Arg3  

004402B7       .  81C1 00040000            add ecx,400                              ; |  

004402BD       .  57                        push edi                                 ; |Arg2  

004402BE       .  51                        push ecx                                 ; |Arg1  

004402BF       .  8D8C24 94010000          lea ecx,dword ptr ss:\[esp+194\]         ; |  

004402C6       .  E8 85F3FFFF              call unpack.0043F650                   ; \在 CODE 节里搜索特征码  

004402CB       .  84C0                      test al,al  

004402CD       .  74 24                    je short unpack.004402F3  

004402CF       .  68 48634000              push unpack.00406348                   ;  ASCII "Borland Delphi 2.0"  

004402D4       >  8BB424 98050000          mov esi,dword ptr ss:\[esp+598\]  

004402DB       .  8D4E 04                 lea ecx,dword ptr ds:\[esi+4\]  

004402DE       .  E8 BD73FFFF              call unpack.004376A0  

004402E3       .  5F                        pop edi  

004402E4       .  C606 01                   mov byte ptr ds:\[esi\],1  

004402E7       .  5E                        pop esi  

004402E8       .  5D                        pop ebp  

004402E9       .  B0 01                     mov al,1  

004402EB       .  5B                        pop ebx  

004402EC       .  81C4 80050000            add esp,580  

004402F2       .  C3                        retn  

; 在 CODE 节里搜索特征码，  

; 特征码库定义为一个 WORD 数组，  

; 按 CODE 节里的数据一一和对应版本的特征码比较。  

; 其中，特征码中的 00 AB 是通配符。  

0043F650     /$  83EC 08                   sub esp,8                              ;  在 CODE 节里搜索特征码  

0043F653     |.  8B5424 10                mov edx,dword ptr ss:\[esp+10\]  

0043F657     |.  57                       push edi  

0043F658     |.  8B79 04                 mov edi,dword ptr ds:\[ecx+4\]  

0043F65B     |.  3BD7                     cmp edx,edi  

0043F65D     |.  894C24 04               mov dword ptr ss:\[esp+4\],ecx  

0043F661     |.  7D 09                    jge short unpack.0043F66C  

0043F663     |.  32C0                     xor al,al  

0043F665     |.  5F                       pop edi  

0043F666     |.  83C4 08                 add esp,8  

0043F669     |.  C2 0C00                  retn 0C  

0043F66C     |>  53                       push ebx  

0043F66D     |.  55                       push ebp  

0043F66E     |.  8B6C24 18               mov ebp,dword ptr ss:\[esp+18\]           ;  ebp = CODE 节首地址  

0043F672     |.  8BC2                     mov eax,edx  

0043F674     |.  2BC7                     sub eax,edi  

0043F676     |.  56                       push esi  

0043F677     |.  33F6                     xor esi,esi  

0043F679     |.  85C0                     test eax,eax  

0043F67B     |.  894424 14               mov dword ptr ss:\[esp+14\],eax  

0043F67F     |.  7E 5F                    jle short unpack.0043F6E0  

0043F681     |>  33C0                     xor eax,eax  

0043F683     |.  85FF                     test edi,edi  

0043F685     |.  7E 26                    jle short unpack.0043F6AD  

0043F687     |.  8B19                     mov ebx,dword ptr ds:\[ecx\]            ;  ebx = 特征码首地址  

0043F689     |.  8DA424 00000000         lea esp,dword ptr ss:\[esp\]  

0043F690     |>  66:8B13                  /mov dx,word ptr ds:\[ebx\]  

0043F693     |.  F6C6 FF                  |test dh,0FF  

0043F696     |.  75 0D                    |jnz short unpack.0043F6A5  

0043F698     |.  8D0C30                  |lea ecx,dword ptr ds:\[eax+esi\]  

0043F69B        66:0FB60C29             movzx cx,byte ptr ds:\[ecx+ebp\]  

0043F6A0        66:3BCA                 cmp cx,dx  

0043F6A3        75 1E                    jnz short unpack.0043F6C3  

0043F6A5     |>  40                       |inc eax  

0043F6A6     |.  83C3 02                |add ebx,2  

0043F6A9     |.  3BC7                    |cmp eax,edi  

0043F6AB     |.^ 7C E3                   \jl short unpack.0043F690  

0043F6AD     |>  8B4424 24               mov eax,dword ptr ss:\[esp+24\]  

0043F6B1     |.  85C0                    test eax,eax  

0043F6B3     |.  74 02                   je short unpack.0043F6B7  

0043F6B5     |.  8930                    mov dword ptr ds:\[eax\],esi  

0043F6B7     |>  5E                       pop esi  

0043F6B8     |.  5D                       pop ebp  

0043F6B9     |.  5B                       pop ebx  

0043F6BA     |.  B0 01                    mov al,1  

0043F6BC     |.  5F                       pop edi  

0043F6BD     |.  83C4 08                 add esp,8  

0043F6C0     |.  C2 0C00                  retn 0C  

0043F6C3     |>  8B4C24 10               mov ecx,dword ptr ss:\[esp+10\]  

0043F6C7     |.  8D1437                  lea edx,dword ptr ds:\[edi+esi\]  

0043F6CA     |.  0FB6042A               movzx eax,byte ptr ds:\[edx+ebp\]  

0043F6CE     |.  8B5C81 08              mov ebx,dword ptr ds:\[ecx+eax\*4+8\]  

0043F6D2     |.  8B4424 14             mov eax,dword ptr ss:\[esp+14\]  

0043F6D6     |.  03F3                   add esi,ebx  

0043F6D8     |.  3BF0                   cmp esi,eax  

0043F6DA     |.^ 7C A5                  jl short unpack.0043F681  

0043F6DC     |.  8B5424 20             mov edx,dword ptr ss:\[esp+20\]  

0043F6E0     |>  3BF0                   cmp esi,eax  

0043F6E2     |.  75 10                  jnz short unpack.0043F6F4  

0043F6E4     |.  2BD6                   sub edx,esi  

0043F6E6     |.  52                      push edx  

0043F6E7     |.  8D142E                lea edx,dword ptr ds:\[esi+ebp\]  

0043F6EA     |.  52                      push edx  

0043F6EB     |.  E8 D07CFFFF           call unpack.004373C0  

0043F6F0     |.  84C0                   test al,al  

0043F6F2     |.^ 75 B9                  jnz short unpack.0043F6AD  

0043F6F4     |>  5E                      pop esi  

0043F6F5     |.  5D                      pop ebp  

0043F6F6     |.  5B                      pop ebx  

0043F6F7     |.  32C0                    xor al,al  

0043F6F9     |.  5F                      pop edi  

0043F6FA     |.  83C4 08                add esp,8  

0043F6FD     \\.  C2 0C00                retn 0C  

Delphi 各个版本的特征码如下所示：  

;Borland  Delphi 3.0  

01BBF9BC  50 00 6A 00 00 00 E8 00 00 AB 00 AB FF 00 FF 00  P.j...?.??.

01BBF9CC  BA 00 00 AB 00 AB 00 AB 00 AB 52 00 89 00 05 00  ?.???玆.?.  

01BBF9DC  00 AB 00 AB 00 AB 00 AB 89 00 42 00 04 00 E8 00  .???珘.B..?  
