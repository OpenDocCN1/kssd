
```
00477D59   |.  50                |push eax  

00477D5A   |.  8BC3             |mov eax,ebx  

00477D5C   |.  48                |dec eax  

00477D5D   |.  5A                |pop edx  

00477D5E   |.  8BCA             |mov ecx,edx  

00477D60   |.  99                |cdq  

00477D61   |.  F7F9             |idiv ecx  

00477D63   |.  8B45 F8          |mov eax,dword ptr ss:[ebp-8]  

00477D66   |.  8A0410           |mov al,byte ptr ds:[eax+edx]                 ;  al=changshu[i]  

00477D69   |.  3206             |xor al,byte ptr ds:[esi]                     ;  al^esi[i]  

00477D6B   |.  50                |push eax  

00477D6C   |.  8B45 F8          |mov eax,dword ptr ss:[ebp-8]  

00477D6F   |.  E8 B0C0F8FF      |call unpacked.00403E24                         ;  求 dffy*dzs*v2005 长度  

00477D74   |.  5A                |pop edx  

00477D75   |.  32D0             |xor dl,al                                     ;  dl^al  

00477D77   |.  32D3             |xor dl,bl                                     ;  dl^bl  

00477D79   |.  8816             |mov byte ptr ds:[esi],dl  

00477D7B   |.  43                |inc ebx  

00477D7C   |.  46                |inc esi  

00477D7D   |.  83FB 0A          |cmp ebx,0A  

00477D80   |.^ 75 CF            \jnz short unpacked.00477D51  

00477D82   |>  8D45 E8          lea eax,dword ptr ss:[ebp-18]  

00477D85   |.  E8 1ABEF8FF      call unpacked.00403BA4  

00477D8A   |.  BB 09000000      mov ebx,9                                     ;  EBX=9  

00477D8F   |.  8D75 EF          lea esi,dword ptr ss:[ebp-11]  

00477D92   |>  8D45 E4          /lea eax,dword ptr ss:[ebp-1C]  

00477D95   |.  8A16             |mov dl,byte ptr ds:[esi]                     ;  DL=esi[i]  

00477D97   |.  E8 B0BFF8FF      |call unpacked.00403D4C  

00477D9C   |.  8B55 E4          |mov edx,dword ptr ss:[ebp-1C]  

00477D9F   |.  8D45 E8          |lea eax,dword ptr ss:[ebp-18]  

00477DA2   |.  E8 85C0F8FF      |call unpacked.00403E2C  

00477DA7   |.  46                |inc esi  

00477DA8   |.  4B                |dec ebx  

00477DA9   |.^ 75 E7            \jnz short unpacked.00477D92  

00477DAB   |.  8D55 E0          lea edx,dword ptr ss:[ebp-20]  

00477DAE   |.  8B45 E8          mov eax,dword ptr ss:[ebp-18]                 ;  ESX=RES[]  

00477DB1   |.  E8 9AFDFFFF      |call unpacked.00477B50                        ;  f7  

00477DB6   |.  8B55 E0          mov edx,dword ptr ss:[ebp-20]                 ;  EDX=真码  

\=============================================================================================================  

00477B50   /$  55                push ebp  

00477B51   |.  8BEC             mov ebp,esp  

00477B53   |.  83C4 F0          add esp,-10  

00477B56   |.  53                push ebx  

00477B57   |.  56                push esi  

00477B58   |.  57                push edi  

00477B59   |.  33C9             xor ecx,ecx  

00477B5B   |.  894D F0          mov dword ptr ss:[ebp-10],ecx  

00477B5E   |.  8BFA             mov edi,edx  

00477B60   |.  8945 FC          mov dword ptr ss:[ebp-4],eax  

00477B63   |.  8B45 FC          mov eax,dword ptr ss:[ebp-4]  

00477B66   |.  E8 6DC4F8FF      call unpacked.00403FD8  

00477B6B   |.  33C0             xor eax,eax  

00477B6D   |.  55                push ebp  

00477B6E   |.  68 847C4700      push unpacked.00477C84  

00477B73   |.  64:FF30          push dword ptr fs:[eax]  

00477B76   |.  64:8920          mov dword ptr fs:[eax],esp  

00477B79   |.  8BC7             mov eax,edi  

00477B7B   |.  E8 24C0F8FF      call unpacked.00403BA4  

00477B80   |.  E9 D7000000      jmp unpacked.00477C5C  

00477B85   |>  8B45 FC          /mov eax,dword ptr ss:[ebp-4]                 ;  EAX=RES[I]  

00477B88   |.  E8 97C2F8FF      |call unpacked.00403E24                        ;  求长度  

00477B8D   |.  8BC8             |mov ecx,eax  

00477B8F   |.  8BC1             |mov eax,ecx  

00477B91   |.  BB 03000000      |mov ebx,3                                     ;  EBX=3  

00477B96   |.  99                |cdq  

00477B97   |.  F7FB             |idiv ebx  

00477B99   |.  85C0             |test eax,eax  

00477B9B   |.  7E 07            |jle short unpacked.00477BA4  

00477B9D   |.  BB 03000000      |mov ebx,3                                     ;  EBX=3  

00477BA2   |.  EB 02            |jmp short unpacked.00477BA6  

00477BA4   |>  8BD9             |mov ebx,ecx  

00477BA6   |>  8D45 F9          |lea eax,dword ptr ss:[ebp-7]  

00477BA9   |.  33C9             |xor ecx,ecx                                   ;  ECX=0  

00477BAB   |.  BA 03000000      |mov edx,3                                     ;  EDX=3  

00477BB0   |.  E8 B3AFF8FF      |call unpacked.00402B68  

00477BB5   |.  8D45 F5          |lea eax,dword ptr ss:[ebp-B]  

00477BB8   |.  B9 40000000      |mov ecx,40  

00477BBD   |.  BA 04000000      |mov edx,4  

00477BC2   |.  E8 A1AFF8FF      |call unpacked.00402B68  

00477BC7   |.  8D45 FC          |lea eax,dword ptr ss:[ebp-4]                 ;  EAX=RES[I]  

00477BCA   |.  E8 25C4F8FF      |call unpacked.00403FF4  

00477BCF   |.  8D55 F9          |lea edx,dword ptr ss:[ebp-7]  

00477BD2   |.  8BCB             |mov ecx,ebx  

00477BD4   |.  E8 B7ACF8FF      |call unpacked.00402890  

00477BD9   |.  83FB 03          |cmp ebx,3  

00477BDC   |.  7C 08            |jl short unpacked.00477BE6  

00477BDE   |.  8A45 FB          |mov al,byte ptr ss:[ebp-5]                   ;  取值  

00477BE1   |.  24 3F            |and al,3F                                     ;  AL & 3F  

00477BE3   |.  8845 F8          |mov byte ptr ss:[ebp-8],al  

00477BE6   |>  83FB 02          |cmp ebx,2  

00477BE9   |.  7C 15            |jl short unpacked.00477C00  

00477BEB   |.  8A45 FA          |mov al,byte ptr ss:[ebp-6]  

00477BEE   |.  C1E0 02          |shl eax,2                                     ;  AL*4  

00477BF1   |.  33D2             |xor edx,edx                                   ;  EDX=0  

00477BF3   |.  8A55 FB          |mov dl,byte ptr ss:[ebp-5]  

00477BF6   |.  C1EA 06          |shr edx,6                                     ;  EDX 右移 6 次  

00477BF9   |.  0AC2             |or al,dl                                      ;  AL | DL  

00477BFB   |.  24 3F            |and al,3F                                     ;  AL & 3F  

00477BFD   |.  8845 F7          |mov byte ptr ss:[ebp-9],al  

00477C00   |>  8A45 F9          |mov al,byte ptr ss:[ebp-7]  

00477C03   |.  8BD0             |mov edx,eax                                   ;  EDX=EAX  

00477C05   |.  C1E2 04          |shl edx,4                                     ;  EDX 左移 4 次  

00477C08   |.  33C9             |xor ecx,ecx                                   ;  ECX=0  

00477C0A   |.  8A4D FA          |mov cl,byte ptr ss:[ebp-6]  

00477C0D   |.  C1E9 04          |shr ecx,4                                     ;  ECX 右移 4 次  

00477C10   |.  0AD1             |or dl,cl                                      ;  DL | CL  

00477C12   |.  80E2 3F          |and dl,3F                                     ;  DL & 3F  

00477C15   |.  8855 F6          |mov byte ptr ss:[ebp-A],dl  

00477C18   |.  25 FF000000      |and eax,0FF                                   ;  EAX & FF  

00477C1D   |.  C1E8 02          |shr eax,2                                     ;  EAX /4  

00477C20   |.  24 3F            |and al,3F                                     ;  AL & 3F  

00477C22   |.  8845 F5          |mov byte ptr ss:[ebp-B],al                    ;  注意  

00477C25   |.  8D45 FC          |lea eax,dword ptr ss:[ebp-4]  

00477C28   |.  8BCB             |mov ecx,ebx  

00477C2A   |.  BA 01000000      |mov edx,1  

00477C2F   |.  E8 38C4F8FF      |call unpacked.0040406C  
```
