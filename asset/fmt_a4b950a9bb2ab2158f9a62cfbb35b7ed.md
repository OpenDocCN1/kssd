
```
00477CFD   |.  5A                 |pop edx                                         ;  弹出机器码[I]  

00477CFE   |.  32D0               |xor dl,al                                       ;  机器码[I]和机器码长度 XOR  

00477D00   |.  32D3               |xor dl,bl                                       ;  和 BL XOR  

00477D02   |.  8816               |mov byte ptr ds:[esi],dl                          ;  保存取名为 T1[]  

00477D04   |.  43                 |inc ebx                                         ;  EBX++  

00477D05   |.  46                 |inc esi                                         ;  ESI++  

00477D06   |.  83FB 0A            |cmp ebx,0A  

00477D09   |.^ 75 D1             \\jnz short 999.00477CDC  

00477D0B   |.  8B45 FC           mov eax,dword ptr ss:[ebp-4]                       ;  机器码  

00477D0E   |.  E8 11C1F8FF        call 999.00403E24                                 ;  机器码长度  

00477D13   |.  8BF0               mov esi,eax  

00477D15   |.  85F6               test esi,esi  

00477D17   |.  7E 2A              jle short 999.00477D43  

00477D19   |.  BB 01000000        mov ebx,1                                         ;  EBX=1  

00477D1E   |>  8B45 FC            /mov eax,dword ptr ss:[ebp-4]                    ;  机器码  

00477D21   |.  E8 FEC0F8FF        |call 999.00403E24                                ;  机器码长度  

00477D26   |.  2BC3               |sub eax,ebx                                     ;  EAX-EBX  

00477D28   |.  8B55 FC            |mov edx,dword ptr ss:[ebp-4]                    ;  机器码  

00477D2B   |.  8A0C02             |mov cl,byte ptr ds:[edx+eax]                    ;  CL=机器码[9-I]  

00477D2E   |.  8BC3               |mov eax,ebx  

00477D30   |.  48                 |dec eax  

00477D31   |.  51                 |push ecx  

00477D32   |.  B9 09000000        |mov ecx,9  

00477D37   |.  99                 |cdq  

00477D38   |.  F7F9               |idiv ecx  

00477D3A   |.  59                 |pop ecx  

00477D3B   |.  304C15 EF          |xor byte ptr ss:[ebp+edx-11],cl                 ;  TI[I]^CL  

00477D3F   |.  43                 |inc ebx  

00477D40   |.  4E                 |dec esi  

00477D41   |.^ 75 DB             \\jnz short 999.00477D1E  

00477D43   |>  837D F8 00         cmp dword ptr ss:[ebp-8],0  

00477D47   |.  74 39              je short 999.00477D82  

00477D49   |.  BB 01000000        mov ebx,1  

00477D4E   |.  8D75 EF            lea esi,dword ptr ss:[ebp-11]  

00477D51   |>  8B45 F8            /mov eax,dword ptr ss:[ebp-8]  

00477D54   |.  E8 CBC0F8FF        |call 999.00403E24  

00477D59   |.  50                 |push eax  

00477D5A   |.  8BC3               |mov eax,ebx  

00477D5C   |.  48                 |dec eax  

00477D5D   |.  5A                 |pop edx  

00477D5E   |.  8BCA               |mov ecx,edx  

00477D60   |.  99                 |cdq  

00477D61   |.  F7F9               |idiv ecx  

00477D63   |.  8B45 F8            |mov eax,dword ptr ss:[ebp-8]  

00477D66   |.  8A0410             |mov al,byte ptr ds:[eax+edx]  

00477D69   |.  3206               |xor al,byte ptr ds:[esi]  

00477D6B   |.  50                 |push eax  

00477D6C   |.  8B45 F8            |mov eax,dword ptr ss:[ebp-8]  

00477D6F   |.  E8 B0C0F8FF        |call 999.00403E24  

00477D74   |.  5A                 |pop edx  

00477D75   |.  32D0               |xor dl,al  

00477D77   |.  32D3               |xor dl,bl  

00477D79   |.  8816               |mov byte ptr ds:[esi],dl  

00477D7B   |.  43                 |inc ebx  

00477D7C   |.  46                 |inc esi  

00477D7D   |.  83FB 0A            |cmp ebx,0A  

00477D80   |.^ 75 CF             \\jnz short 999.00477D51  

00477D82   |>  8D45 E8            lea eax,dword ptr ss:[ebp-18]                     ;  跳到这里  

00477D85   |.  E8 1ABEF8FF        call 999.00403BA4  

00477D8A   |.  BB 09000000        mov ebx,9                                         ;  ebx=9  

00477D8F   |.  8D75 EF            lea esi,dword ptr ss:[ebp-11]                     ;  esi=t1[]  

00477D92   |>  8D45 E4            /lea eax,dword ptr ss:[ebp-1C]  

00477D95   |.  8A16               |mov dl,byte ptr ds:[esi]                          ;  dl=t1[i]  

00477D97   |.  E8 B0BFF8FF        |call 999.00403D4C  

00477D9C   |.  8B55 E4            |mov edx,dword ptr ss:[ebp-1C]  

00477D9F   |.  8D45 E8            |lea eax,dword ptr ss:[ebp-18]  

00477DA2   |.  E8 85C0F8FF        |call 999.00403E2C  

00477DA7   |.  46                 |inc esi  

00477DA8   |.  4B                 |dec ebx  

00477DA9   |.^ 75 E7             \\jnz short 999.00477D92                            ;  这个循环没有什么作用  

00477DAB   |.  8D55 E0            lea edx,dword ptr ss:[ebp-20]  

00477DAE   |.  8B45 E8            mov eax,dword ptr ss:[ebp-18]                     ;  eax=t1[]  

00477DB1   |.  E8 9AFDFFFF        call 999.00477B50                                  ;  关键算法  

00477DB6   |.  8B55 E0            mov edx,dword ptr ss:[ebp-20]                     ;  真码  

00477DB9   |.  8BC7               mov eax,edi  

00477DBB   |.  B9 FF000000        mov ecx,0FF  

00477DC0   |.  E8 3BC0F8FF        call 999.00403E00                                  ;  比较注册码  

00477DC5   |.  33C0               xor eax,eax  

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>.00477DB1       call 999.00477B50     ;  关键算法  

00477B50   /$  55                 push ebp  

00477B51   |.  8BEC               mov ebp,esp  

00477B53   |.  83C4 F0            add esp,-10  

00477B56   |.  53                 push ebx  

00477B57   |.  56                 push esi  

00477B58   |.  57                 push edi  

00477B59   |.  33C9               xor ecx,ecx  

00477B5B   |.  894D F0            mov dword ptr ss:[ebp-10],ecx  

00477B5E   |.  8BFA               mov edi,edx  

00477B60   |.  8945 FC            mov dword ptr ss:[ebp-4],eax  

00477B63   |.  8B45 FC            mov eax,dword ptr ss:[ebp-4]  

00477B66   |.  E8 6DC4F8FF        call 999.00403FD8  

00477B6B   |.  33C0               xor eax,eax  

00477B6D   |.  55                 push ebp  

00477B6E   |.  68 847C4700        push 999.00477C84  

00477B73   |.  64:FF30            push dword ptr fs:[eax]  

00477B76   |.  64:8920            mov dword ptr fs:[eax],esp  

00477B79   |.  8BC7               mov eax,edi  

00477B7B   |.  E8 24C0F8FF        call 999.00403BA4  

00477B80   |.  E9 D7000000        jmp 999.00477C5C  

00477B85   |>  8B45 FC            /mov eax,dword ptr ss:[ebp-4]                     ;  eax=t1[]  

00477B88   |.  E8 97C2F8FF        |call 999.00403E24                                ;  求长度  

00477B8D   |.  8BC8               |mov ecx,eax  

00477B8F   |.  8BC1               |mov eax,ecx  

00477B91   |.  BB 03000000        |mov ebx,3                                         ;  ebx=3  

00477B96   |.  99                 |cdq  

00477B97   |.  F7FB               |idiv ebx  

00477B99   |.  85C0               |test eax,eax  

00477B9B   |.  7E 07              |jle short 999.00477BA4  

00477B9D   |.  BB 03000000        |mov ebx,3                                         ;  ebx=3  

00477BA2   |.  EB 02              |jmp short 999.00477BA6  

00477BA4   |>  8BD9               |mov ebx,ecx  

00477BA6   |>  8D45 F9            |lea eax,dword ptr ss:[ebp-7]  

00477BA9   |.  33C9               |xor ecx,ecx  

00477BAB   |.  BA 03000000        |mov edx,3                                         ;  edx=3  

00477BB0   |.  E8 B3AFF8FF        |call 999.00402B68  

00477BB5   |.  8D45 F5            |lea eax,dword ptr ss:[ebp-B]  

00477BB8   |.  B9 40000000        |mov ecx,40  

00477BBD   |.  BA 04000000        |mov edx,4  

00477BC2   |.  E8 A1AFF8FF        |call 999.00402B68  

00477BC7   |.  8D45 FC            |lea eax,dword ptr ss:[ebp-4]  

00477BCA   |.  E8 25C4F8FF        |call 999.00403FF4  

00477BCF   |.  8D55 F9            |lea edx,dword ptr ss:[ebp-7]  

00477BD2   |.  8BCB               |mov ecx,ebx  

00477BD4   |.  E8 B7ACF8FF        |call 999.00402890  

00477BD9   |.  83FB 03            |cmp ebx,3  
```
