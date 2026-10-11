
> `"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"`  

> `004146C8  |.  8DBC24 9000>lea edi,dword ptr ss:\[esp+90\]`  

> `004146CF  |.  C74424 10 0>mov dword ptr ss:\[esp+10\],5`  

> `004146D7  |.  F3:A5       rep movs dword ptr es:\[edi\],dword ptr ds:\[e>`  

> `004146D9  |.  A4          movs byte ptr es:\[edi\],byte ptr ds:\[esi\]`  

> `004146DA  |>  8B03        /mov eax,dword ptr ds:\[ebx\]     ; 取 DES 加密后的串“法文.c.”`  

> `004146DC  |.  8BC8        |mov ecx,eax                    ; 下面是对加密后的串进行 Base64 变换`  

> `004146DE  |.  C1E9 02     |shr ecx,2`  

> `004146E1  |.  83E1 3F     |and ecx,3F`  

> `004146E4  |.  7C 0E       |jl short Wordllk.004146F4`  

> `004146E6  |.  83F9 40     |cmp ecx,40`  

> `004146E9  |.  7D 09       |jge short Wordllk.004146F4`  

> `004146EB  |.  8A8C0C 5C01>|mov cl,byte ptr ss:\[esp+ecx+15C\]`  

> `004146F2  |.  EB 02       |jmp short Wordllk.004146F6`  

> `004146F4  |>  B1 3D       |mov cl,3D`  

> `004146F6  |>  0FBEF1      |movsx esi,cl`  

> `004146F9  |.  8BC8        |mov ecx,eax`  

> `004146FB  |.  8BD0        |mov edx,eax`  

> `004146FD  |.  C1E9 0C     |shr ecx,0C`  

> `00414700  |.  83E2 03     |and edx,3`  

> `00414703  |.  83E1 0F     |and ecx,0F`  

> `00414706  |.  C1E2 04     |shl edx,4`  

> `00414709  |.  0BCA        |or ecx,edx`  

> `0041470B  |.  7C 0E       |jl short Wordllk.0041471B`  

> `0041470D  |.  83F9 40     |cmp ecx,40`  

> `00414710  |.  7D 09       |jge short Wordllk.0041471B`  

> `00414712  |.  8A8C0C D400>|mov cl,byte ptr ss:\[esp+ecx+D4\]`  

> `00414719  |.  EB 02       |jmp short Wordllk.0041471D`  

> `0041471B  |>  B1 3D       |mov cl,3D`  

> `0041471D  |>  0FBED1      |movsx edx,cl`  

> `00414720  |.  8BC8        |mov ecx,eax`  

> `00414722  |.  8BF8        |mov edi,eax`  

> `00414724  |.  C1E9 08     |shr ecx,8`  

> `00414727  |.  83E1 0F     |and ecx,0F`  

> `0041472A  |.  C1EF 16     |shr edi,16`  

> `0041472D  |.  C1E1 02     |shl ecx,2`  

> `00414730  |.  83E7 03     |and edi,3`  

> `00414733  |.  0BCF        |or ecx,edi`  

> `00414735  |.  7C 0E       |jl short Wordllk.00414745`  

> `00414737  |.  83F9 40     |cmp ecx,40`  

> `0041473A  |.  7D 09       |jge short Wordllk.00414745`  

> `0041473C  |.  8A8C0C 1801>|mov cl,byte ptr ss:\[esp+ecx+118\]`  

> `00414743  |.  EB 02       |jmp short Wordllk.00414747`  

> `00414745  |>  B1 3D       |mov cl,3D`  

> `00414747  |>  C1E8 10     |shr eax,10`  

> `0041474A  |.  83E0 3F     |and eax,3F`  

> `0041474D  |.  0FBEC9      |movsx ecx,cl`  

> `00414750  |.  7C 0E       |jl short Wordllk.00414760`  

> `00414752  |.  83F8 40     |cmp eax,40`  

> `00414755  |.  7D 09       |jge short Wordllk.00414760`  

> `00414757  |.  8A8404 9000>|mov al,byte ptr ss:\[esp+eax+90\]`  

> `0041475E  |.  EB 02       |jmp short Wordllk.00414762`  

> `00414760  |>  B0 3D       |mov al,3D`  

> `00414762  |>  0FBEC0      |movsx eax,al`  

> `00414765  |.  C1E0 08     |shl eax,8`  

> `00414768  |.  0BC1        |or eax,ecx`  

> `0041476A  |.  83C5 04     |add ebp,4`  

> `0041476D  |.  C1E0 08     |shl eax,8`  

> `00414770  |.  0BC2        |or eax,edx`  

> `00414772  |.  83C3 03     |add ebx,3`  

> `00414775  |.  C1E0 08     |shl eax,8`  

> `00414778  |.  0BC6        |or eax,esi`  

> `0041477A  |.  8945 FC     |mov dword ptr ss:\[ebp-4\],eax`  

> `0041477D  |.  8B4424 10   |mov eax,dword ptr ss:\[esp+10\]`  

> `00414781  |.  48          |dec eax`  

> `00414782  |.  894424 10   |mov dword ptr ss:\[esp+10\],eax`  

> `00414786  |.^ 0F85 4EFFFF>\jnz Wordllk.004146DA`  

> `0041478C  |.  8A03        mov al,byte ptr ds:\[ebx\]`  

> `0041478E  |.  B9 10000000 mov ecx,10`  

> `00414793  |.  BE 80844300 mov esi,Wordllk.00438480                     ;  ASCII `  

>   

> `"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"`  

> `00414798  |.  8DBC24 9000>lea edi,dword ptr ss:\[esp+90\]`  

> `0041479F  |.  F3:A5       rep movs dword ptr es:\[edi\],dword ptr ds:\[e>`  

> `004147A1  |.  C74424 10 0>mov dword ptr ss:\[esp+10\],0`  

> `004147A9  |.  884424 10   mov byte ptr ss:\[esp+10\],al`  

> `004147AD  |.  8B5424 10   mov edx,dword ptr ss:\[esp+10\]`  

> `004147B1  |.  8BC2        mov eax,edx`  

> `004147B3  |.  C1E8 02     shr eax,2`  

> `004147B6  |.  83E0 3F     and eax,3F`  

> `004147B9  |.  A4          movs byte ptr es:\[edi\],byte ptr ds:\[esi\]`  

> `004147BA  |.  7C 0E       jl short Wordllk.004147CA`  

> `004147BC  |.  83F8 40     cmp eax,40`  

> `004147BF  |.  7D 09       jge short Wordllk.004147CA`  

> `004147C1  |.  8A8404 9000>mov al,byte ptr ss:\[esp+eax+90\]`  

> `004147C8  |.  EB 02       jmp short Wordllk.004147CC`  

> `004147CA  |>  B0 3D       mov al,3D`  

> `004147CC  |>  B9 10000000 mov ecx,10`  

> `004147D1  |.  BE 80844300 mov esi,Wordllk.00438480                     ;  ASCII `  

>   

> `"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"`  

> `004147D6  |.  8DBC24 9000>lea edi,dword ptr ss:\[esp+90\]`  

> `004147DD  |.  8845 00     mov byte ptr ss:\[ebp\],al`  

> `004147E0  |.  F3:A5       rep movs dword ptr es:\[edi\],dword ptr ds:\[e>`  

> `004147E2  |.  8BC2        mov eax,edx`  

> `004147E4  |.  83E2 03     and edx,3`  

> `004147E7  |.  C1E8 0C     shr eax,0C`  

> `004147EA  |.  83E0 0F     and eax,0F`  

> `004147ED  |.  C1E2 04     shl edx,4`  

> `004147F0  |.  0BC2        or eax,edx`  

> `004147F2  |.  A4          movs byte ptr es:\[edi\],byte ptr ds:\[esi\]`  

> `004147F3  |.  7C 0E       jl short Wordllk.00414803`  

> `004147F5  |.  83F8 40     cmp eax,40`  

> `004147F8  |.  7D 09       jge short Wordllk.00414803`  

> `004147FA  |.  8A8404 9000>mov al,byte ptr ss:\[esp+eax+90\]`  

> `00414801  |.  EB 02       jmp short Wordllk.00414805`  

> `00414803  |>  B0 3D       mov al,3D`  

> `00414805  |>  8845 01     mov byte ptr ss:\[ebp+1\],al`  

> `00414808  |.  C645 02 3D  mov byte ptr ss:\[ebp+2\],3D`  

> `0041480C  |.  C645 03 3D  mov byte ptr ss:\[ebp+3\],3D`  

> `00414810  |.  83C5 04     add ebp,4`  

> `00414813  |.  8D7C24 54   lea edi,dword ptr ss:\[esp+54\]               ;  Base64 结`  

>   

> `果"0Vy3qM7EupguY4e4n1suxQ=="`  

> `00414817  |.  83C9 FF     or ecx,FFFFFFFF`  

> `0041481A  |.  33C0        xor eax,eax`  

> `0041481C  |.  C645 00 00  mov byte ptr ss:\[ebp\],0`  

> `00414820  |.  F2:AE       repne scas byte ptr es:\[edi\]`  

> `00414822  |.  F7D1        not ecx`  

> `00414824  |.  49          dec ecx`  

> `00414825  |.  85C9        test ecx,ecx                             ;  ecx=24`  

> `00414827  |.  7E 29       jle short Wordllk.00414852                ;  依次取 Base64 变换后的每 1 位`  

> `00414829  |>  807C04 54 2>/cmp byte ptr ss:\[esp+eax+54\],2F            ;  如果是/`  

> `0041482E  |.  75 05       |jnz short Wordllk.00414835`  

> `00414830  |.  C64404 54 3>|mov byte ptr ss:\[esp+eax+54\],31            ;  替换为'1'`  

> `00414835  |>  807C04 54 2>|cmp byte ptr ss:\[esp+eax+54\],2B            ;  如果是+`  

> `0041483A  |.  75 05       |jnz short Wordllk.00414841`  

> `0041483C  |.  C64404 54 3>|mov byte ptr ss:\[esp+eax+54\],32            ;  替换为'2'`  

> `00414841  |>  807C04 54 3>|cmp byte ptr ss:\[esp+eax+54\],3D            ;  如果是=`  

> `00414846  |.  75 05       |jnz short Wordllk.0041484D`  

> `00414848  |.  C64404 54 0>|mov byte ptr ss:\[esp+eax+54\],0           ;  替换为 0`  

> `0041484D  |>  40          |inc eax`  

> `0041484E  |.  3BC1        |cmp eax,ecx`  

> `00414850  |.^ 7C D7       \jl short Wordllk.00414829`  

> `00414852  |>  8D4C24 54   lea ecx,dword ptr ss:\[esp+54\]               ;  ASCII `  

>   

> `"0Vy3qM7EupguY4e4n1suxQ"`  

> `00414856  |.  51          push ecx                              ; /s`  

> `00414857  |.  FF15 B0B942>call dword ptr ds:\[<&MSVCRT.\_strupr>\]     ; \\\_strupr 转换为大写`  

> `0041485D  |.  83C4 04     add esp,4`  

> `00414860  |.  8D5424 54   lea edx,dword ptr ss:\[esp+54\]               ;  ASCII `  

>   

> `"0VY3QM7EUPGUY4E4N1SUXQ"`  

> `00414864  |.  8D4C24 14   lea ecx,dword ptr ss:\[esp+14\]`  

> `00414868  |.  52          push edx`  

> `00414869  |.  E8 600D0100 call <jmp.&MFC42.#537>`  

> `0041486E  |.  6A 10       push 10`  

> `00414870  |.  8D4424 14   lea eax,dword ptr ss:\[esp+14\]`  
