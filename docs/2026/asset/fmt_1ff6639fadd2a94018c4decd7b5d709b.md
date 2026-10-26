
```assembly
00437358   |.  75 0D                     |jnz short Client.00437367

0043735A   |.  8B48 04                   |mov ecx,dword ptr ds:[eax+4] //

0043735D   |.  83F9 2D                   |cmp ecx,2D

00437360   |.  74 14                     |je short Client.00437376 //youxi人物

00437362   |.  83F9 06                   |cmp ecx,6

00437365   |.  74 0F                     |je short Client.00437376 //静态咚咚？

00437367   |>  6A 00                     |push 0                                                     // /Arg4 = 00000000

00437369   |.  6A 00                     |push 0                                                     // |Arg3 = 00000000

0043736B   |.  6A 08                     |push 8                                                     // |Arg2 = 00000008

0043736D   |.  50                        |push eax                                                   // |Arg1 //指向对象的指针

0043736E   |.  E8 DDFCFFFF               |call Client.00437050                                       // \Client.00437050

{

        00437050   /$  55                        push ebp

        00437051   |.  8BEC                      mov ebp,esp

        00437053   |.  8B45 0C                   mov eax,dword ptr ss:[ebp+C]  // eax 0x08

        00437056   |.  56                        push esi

        00437057   |.  3D 16040000               cmp eax,416                                                 //  Switch (cases 0..44E)

        0043705C   |.  7F 77                     jg short Client.004370D5

        0043705E   |.  3D 11040000               cmp eax,411

        00437063   |.  0F8D 86000000             jge Client.004370EF

        00437069   |.  83F8 0A                   cmp eax,0A

        0043706C   |.  7F 48                     jg short Client.004370B6

        0043706E   |.  74 14                     je short Client.00437084

        00437070   |.  83F8 08                   cmp eax,8

        00437073   |.  77 48                     ja short Client.004370BD

        00437075   |.  33C9                      xor ecx,ecx

        00437077   |.  8A88 10714300             mov cl,byte ptr ds:[eax+437110] // cl = byte ptr [437118]  //索引

        0043707D   |.  FF248D 08714300           jmp dword ptr ds:[ecx\*4+437108] //跳到 8 那里

        00437084   |>  8B75 08                   mov esi,dword ptr ss:[ebp+8]                                //  Case A of switch 00437057

        00437087   |.  8B45 10                   mov eax,dword ptr ss:[ebp+10]

        0043708A   |.  57                        push edi

        0043708B   |.  8B7D 14                   mov edi,dword ptr ss:[ebp+14]

        0043708E   |.  8B16                      mov edx,dword ptr ds:[esi]

        00437090   |.  57                        push edi

        00437091   |.  50                        push eax

        00437092   |.  6A 0A                     push 0A

        00437094   |.  8BCE                      mov ecx,esi

        00437096   |.  FF52 04                   call dword ptr ds:[edx+4]

        00437099   |.  81FF FF000000             cmp edi,0FF

        0043709F   |.  5F                        pop edi

        004370A0   |.  75 5E                     jnz short Client.00437100

        004370A2   |.  85F6                      test esi,esi

        004370A4   |.  74 5A                     je short Client.00437100

        004370A6   |.  8B16                      mov edx,dword ptr ds:[esi]

        004370A8   |.  6A 01                     push 1

        004370AA   |.  8BCE                      mov ecx,esi

        004370AC   |.  FF12                      call dword ptr ds:[edx]

        004370AE   |.  B8 01000000               mov eax,1

        004370B3   |.  5E                        pop esi

        004370B4   |.  5D                        pop ebp

        004370B5   |.  C3                        retn

        004370B6   |>  3D F9030000               cmp eax,3F9

        004370BB   |.  74 32                     je short Client.004370EF

        004370BD   |>  8B4D 14                   mov ecx,dword ptr ss:[ebp+14]                               //  Default case of switch 00437057

        004370C0   |.  8B55 10                   mov edx,dword ptr ss:[ebp+10]

        004370C3   |.  51                        push ecx                                                    // /Arg4

        004370C4   |.  52                        push edx                                                    // |Arg3

        004370C5   |.  50                        push eax                                                    // |Arg2

        004370C6   |.  8B45 08                   mov eax,dword ptr ss:[ebp+8]                                // |

        004370C9   |.  50                        push eax                                                    // |Arg1

        004370CA   |.  E8 C1B2FFFF               call Client.00432390                                        // \Client.00432390

        004370CF   |.  83C4 10                   add esp,10

        004370D2   |.  5E                        pop esi

        004370D3   |.  5D                        pop ebp

        004370D4   |.  C3                        retn

        004370D5   |>  8D88 E7FBFFFF             lea ecx,dword ptr ds:[eax-419]

        004370DB   |.  83F9 35                   cmp ecx,35

        004370DE   |.^ 77 DD                     ja short Client.004370BD

        004370E0   |.  33D2                      xor edx,edx

        004370E2   |.  8A91 24714300             mov dl,byte ptr ds:[ecx+437124]

        004370E8   |.  FF2495 1C714300           jmp dword ptr ds:[edx\*4+43711C]                             //  Cases 0,1,2,3,6,7,8,3F9,411,412,413,414,415,416,419,42F,430,43A,44D,44E of sw>

        //调到这里

        004370EF   |>  8B75 14                   mov esi,dword ptr ss:[ebp+14]   //  0

        004370F2   |.  8B4D 08                   mov ecx,dword ptr ss:[ebp+8]    // 指向对象的指针

        004370F5   |.  56                        push esi  //

        004370F6   |.  8B75 10                   mov esi,dword ptr ss:[ebp+10]   // 0

        004370F9   |.  8B11                      mov edx,dword ptr ds:[ecx]      // edx 是指向这个处理对象数据的函数指针

        004370FB   |.  56                        push esi  // 0

        004370FC   |.  50                        push eax  // 8

        004370FD   |.  FF52 04                   call dword ptr ds:[edx+4]  // 调用这个函数

        {

                ecx = 指向对象的指针

                func(8,0,0)

        }

        00437100   |>  B8 01000000               mov eax,1

        00437105   |.  5E                        pop esi

        00437106   |.  5D                        pop ebp

        00437107   \.  C3                        retn

}

00437373   |.  83C4 10                   |add esp,10

00437376   |>  83C6 04                   |add esi,4

00437379   |.  81FE 88AC1E01             |cmp esi,Client.011EAC88

0043737F   |.^ 7C B5                     \jl short Client.00437336

00437381   |.  A1 745A1E01               mov eax,dword ptr ds:[11E5A74]

00437386   |.  85C0                      test eax,eax

00437388   |.  7E 2C                     jle short Client.004373B6

0043738A   |.  8BF0                      mov esi,eax
```
