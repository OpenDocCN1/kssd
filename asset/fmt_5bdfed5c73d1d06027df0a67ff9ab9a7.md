
```
004147E0   |.  8D4C24 10     lea ecx,dword ptr ss:[esp+10]                       ;  第 14 位看有没有-，没有就失败  

004147E4   |.  E8 6F310100   call <jmp.&MFC42.#6662_CString::Find>  

004147E9   |.  3BC5          cmp eax,ebp  

004147EB   |.  0F84 4E020000 je Memory_B.00414A3F  

004147F1   |.  6A 0B         push 0B  

004147F3   |.  6A 2D         push 2D  

004147F5   |.  8D4C24 10     lea ecx,dword ptr ss:[esp+10]                       ;  看有没有-，没有就失败  

004147F9   |.  E8 5A310100   call <jmp.&MFC42.#6662_CString::Find>  

004147FE   |.  3BC5          cmp eax,ebp  

00414800   |.  0F84 39020000 je Memory_B.00414A3F  

00414806   |.  33DB          xor ebx,ebx                                       ;  EBX 清零  

00414808   |>  8AD3          /mov dl,bl                                          ;  DL=BL  

0041480A   |.  6A 01         |push 1  

0041480C   |.  80C2 61       |add dl,61                                         ;  DL+0X61(a)  

0041480F   |.  8D4C24 0C     |lea ecx,dword ptr ss:[esp+C]  

00414813   |.  885424 1C     |mov byte ptr ss:[esp+1C],dl                      ;  [esp+1c]=dl  

00414817   |.  8B4424 1C     |mov eax,dword ptr ss:[esp+1C]                    ;  eax=[esp+1c]  

0041481B   |.  50            |push eax                                          ;  看有没有 a,b,c,d..............z  

0041481C   |.  E8 37310100   |call <jmp.&MFC42.#6662_CString::Find>  

00414821   |.  3BC5          |cmp eax,ebp  

00414823   |.  0F85 16020000 |jnz Memory_B.00414A3F                             ;  看有没有小写字母没有继续找有就失败  

00414829   |.  43            |inc ebx                                           ;  EBX++  

0041482A   |.  83FB 1A       |cmp ebx,1A                                        ;  EBX 小于 1A=26D 吗  

0041482D   |.^ 7C D9         \jl short Memory_B.00414808  

0041482F   |.  33DB          xor ebx,ebx                                       ;  EBX 清零  

00414831   |>  8ACB          /mov cl,bl  

00414833   |.  6A 01         |push 1  

00414835   |.  80C1 41       |add cl,41  

00414838   |.  884C24 1C     |mov byte ptr ss:[esp+1C],cl  

0041483C   |.  8B5424 1C     |mov edx,dword ptr ss:[esp+1C]  

00414840   |.  52            |push edx  

00414841   |.  8D4C24 10     |lea ecx,dword ptr ss:[esp+10]  

00414845   |.  E8 0E310100   |call <jmp.&MFC42.#6662_CString::Find>           ;  看有没有 A,B,C,D..............Z  

0041484A   |.  3BC5          |cmp eax,ebp  

0041484C   |.  0F85 ED010000 |jnz Memory_B.00414A3F                             ;  看有没有大写字母没有继续找有就失败  

00414852   |.  43            |inc ebx  

00414853   |.  83FB 1A       |cmp ebx,1A  

00414856   |.^ 7C D9         \jl short Memory_B.00414831  

00414858   |.  56            push esi  

00414859   |.  57            push edi  

0041485A   |.  8D4C24 1C     lea ecx,dword ptr ss:[esp+1C]  

0041485E   |.  E8 AB2C0100   call <jmp.&MFC42.#540_CString::CString>  

00414863   |.  8D4C24 18     lea ecx,dword ptr ss:[esp+18]  

00414867   |.  C64424 50 02  mov byte ptr ss:[esp+50],2  

0041486C   |.  E8 9D2C0100   call <jmp.&MFC42.#540_CString::CString>  

00414871   |.  8D4C24 14     lea ecx,dword ptr ss:[esp+14]  

00414875   |.  C64424 50 03  mov byte ptr ss:[esp+50],3  

0041487A   |.  E8 8F2C0100   call <jmp.&MFC42.#540_CString::CString>  

0041487F   |.  8D4424 20     lea eax,dword ptr ss:[esp+20]  

00414883   |.  B3 04         mov bl,4                                          ;  BL=4  

00414885   |.  6A 05         push 5  

00414887   |.  50            push eax  

00414888   |.  8D4C24 18     lea ecx,dword ptr ss:[esp+18]                     ;  eCx=假码  

0041488C   |.  885C24 58     mov byte ptr ss:[esp+58],bl  

00414890   |.  E8 9D2C0100   call <jmp.&MFC42.#4129_CString::Left>  

00414895   |.  50            push eax                                          ;  EAX=前 5 位注册码  

00414896   |.  8D4C24 20     lea ecx,dword ptr ss:[esp+20]  

0041489A   |.  C64424 54 05  mov byte ptr ss:[esp+54],5  

0041489F   |.  E8 5E2C0100   call <jmp.&MFC42.#858_CString::operator=>  

004148A4   |.  8D4C24 20     lea ecx,dword ptr ss:[esp+20]  

004148A8   |.  885C24 50     mov byte ptr ss:[esp+50],bl  

004148AC   |.  E8 4B2C0100   call <jmp.&MFC42.#800_CString::~CString>  

004148B1   |.  8D4C24 20     lea ecx,dword ptr ss:[esp+20]  

004148B5   |.  6A 05         push 5  

004148B7   |.  51            push ecx  

004148B8   |.  8D4C24 18     lea ecx,dword ptr ss:[esp+18]  

004148BC   |.  E8 BF2F0100   call <jmp.&MFC42.#5710_CString::Right>  

004148C1   |.  50            push eax                                          ;  EAX=后 5 位注册码  

004148C2   |.  8D4C24 18     lea ecx,dword ptr ss:[esp+18]  

004148C6   |.  C64424 54 06  mov byte ptr ss:[esp+54],6  

004148CB   |.  E8 322C0100   call <jmp.&MFC42.#858_CString::operator=>  

004148D0   |.  8D4C24 20     lea ecx,dword ptr ss:[esp+20]  

004148D4   |.  885C24 50     mov byte ptr ss:[esp+50],bl  

004148D8   |.  E8 1F2C0100   call <jmp.&MFC42.#800_CString::~CString>  

004148DD   |.  6A 05         push 5  

004148DF   |.  8D5424 24     lea edx,dword ptr ss:[esp+24]  

004148E3   |.  6A 06         push 6  

004148E5   |.  52            push edx  

004148E6   |.  8D4C24 1C     lea ecx,dword ptr ss:[esp+1C]  

004148EA   |.  E8 63300100   call <jmp.&MFC42.#4278_CString::Mid>              ;  第 2 组首 5 位  

004148EF   |.  50            push eax  

004148F0   |.  8D4C24 1C     lea ecx,dword ptr ss:[esp+1C]  

004148F4   |.  C64424 54 07  mov byte ptr ss:[esp+54],7  

004148F9   |.  E8 042C0100   call <jmp.&MFC42.#858_CString::operator=>  

004148FE   |.  8D4C24 20     lea ecx,dword ptr ss:[esp+20]  

00414902   |.  885C24 50     mov byte ptr ss:[esp+50],bl  

00414906   |.  E8 F12B0100   call <jmp.&MFC42.#800_CString::~CString>  

0041490B   |.  8B7C24 1C     mov edi,dword ptr ss:[esp+1C]                     ;  EDI=前 5 位  

0041490F   |.  8BCD          mov ecx,ebp  

00414911   |.  33C0          xor eax,eax                                       ;  EAX=0  

00414913   |.  8D5424 24     lea edx,dword ptr ss:[esp+24]  

00414917   |.  F2:AE         repne scas byte ptr es:[edi]  

00414919   |.  F7D1          not ecx  

0041491B   |.  2BF9          sub edi,ecx                                       ;  EDI=前 5 位  

0041491D   |.  8BC1          mov eax,ecx  

0041491F   |.  8BF7          mov esi,edi                                       ;  ESI=EDI=前 5 位  

00414921   |.  8BFA          mov edi,edx  

00414923   |.  8D5424 30     lea edx,dword ptr ss:[esp+30]  

00414927   |.  C1E9 02       shr ecx,2                                         ;  ECX SHR 2  

0041492A   |.  F3:A5         rep movs dword ptr es:[edi],dword ptr ds:[esi]  

0041492C   |.  8BC8          mov ecx,eax                                       ;  ECX=EAX=6  

0041492E   |.  33C0          xor eax,eax                                       ;  EAX=0  

00414930   |.  83E1 03       and ecx,3                                         ;  ECX AND 3=2  

00414933   |.  F3:A4         rep movs byte ptr es:[edi],byte ptr ds:[esi]  

00414935   |.  8B7C24 18     mov edi,dword ptr ss:[esp+18]                     ;  第 2 组首 5 位  

00414939   |.  8BCD          mov ecx,ebp  

0041493B   |.  F2:AE         repne scas byte ptr es:[edi]  

0041493D   |.  F7D1          not ecx  

0041493F   |.  2BF9          sub edi,ecx  

00414941   |.  8BC1          mov eax,ecx  

00414943   |.  8BF7          mov esi,edi  

00414945   |.  8BFA          mov edi,edx  

00414947   |.  C1E9 02       shr ecx,2                                         ;  ECX SHR 2  

0041494A   |.  F3:A5         rep movs dword ptr es:[edi],dword ptr ds:[esi]  

0041494C   |.  8BC8          mov ecx,eax  

0041494E   |.  33C0          xor eax,eax                                       ;  EAX=0  

00414950   |.  83E1 03       and ecx,3                                         ;  ECX AND 3=2
```
