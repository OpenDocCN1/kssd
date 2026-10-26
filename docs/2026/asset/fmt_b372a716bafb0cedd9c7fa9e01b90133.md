
## 修改前

```asm
004FA012       8945 F8                   mov dword ptr ss:[ebp-8],eax
004FA015       8B4D F8                   mov ecx,dword ptr ss:[ebp-8]
004FA018       894D FC                   mov dword ptr ss:[ebp-4],ecx
004FA01B       8B55 0C                   mov edx,dword ptr ss:[ebp+C]
004FA01E       52                        push edx                                           ;
/Arg3
004FA01F       8B45 FC                   mov eax,dword ptr ss:[ebp-4]                       ; |
004FA022       50                        push eax                                           ;
|Arg2
004FA023       8B4D 08                   mov ecx,dword ptr ss:[ebp+8]                       ; |
004FA026       51                        push ecx                                           ;
|Arg1
004FA027       E8 5D000000               call zy1.004FA089                                  ;
\zy1.004FA089
004FA02C       83C4 0C                   add esp,0C
004FA02F       8B55 0C                   mov edx,dword ptr ss:[ebp+C]
004FA032       52                        push edx                                           ;
/Arg3
004FA033       8B45 08                   mov eax,dword ptr ss:[ebp+8]                       ; |
004FA036       50                        push eax                                           ;
|Arg2
004FA037       8B4D FC                   mov ecx,dword ptr ss:[ebp-4]                       ; |
004FA03A       51                        push ecx                                           ;
|Arg1
004FA03B       E8 66020000               call zy1.004FA2A6                                  ;
\zy1.004FA2A6
004FA040       83C4 0C                   add esp,0C
004FA043       8B55 0C                   mov edx,dword ptr ss:[ebp+C]
004FA046       8B02                      mov eax,dword ptr ds:[edx]
004FA048       50                        push eax                                           ;
/Arg3
004FA049       8B4D FC                   mov ecx,dword ptr ss:[ebp-4]                       ; |
004FA04C       51                        push ecx                                           ;
|Arg2
004FA04D       8B55 08                   mov edx,dword ptr ss:[ebp+8]                       ; |
004FA050       52                        push edx                                           ;
|Arg1
004FA051       E8 40030000               call zy1.004FA396                                  ;
\zy1.004FA396
004FA056       83C4 0C                   add esp,0C
004FA059       8B45 10                   mov eax,dword ptr ss:[ebp+10]
004FA05C       50                        push eax                                           ;
/Arg4
004FA05D       8B4D 0C                   mov ecx,dword ptr ss:[ebp+C]                       ; |
004FA060       51                        push ecx                                           ;
|Arg3
004FA061       8B55 08                   mov edx,dword ptr ss:[ebp+8]                       ; |
004FA064       52                        push edx                                           ;
|Arg2
004FA065       8B45 FC                   mov eax,dword ptr ss:[ebp-4]                       ; |
004FA068       50                        push eax                                           ;
|Arg1
004FA069       E8 521A0000               call zy1.004FBAC0                                  ;
\zy1.004FBAC0
004FA06E       83C4 10                   add esp,10
004FA071       8B4D FC                   mov ecx,dword ptr ss:[ebp-4]
004FA074       894D F4                   mov dword ptr ss:[ebp-C],ecx
004FA077       8B55 F4                   mov edx,dword ptr ss:[ebp-C]
004FA07A       52                        push edx
004FA07B       E8 E48B0200               call zy1.00522C64
004FA080       83C4 04                   add esp,4
004FA083       B0 01                     mov al,1
004FA085       8BE5                      mov esp,ebp
004FA087       5D                        pop ebp
004FA088       C3                        retn
004FA089       55                        push ebp
004FA08A       8BEC                      mov ebp,esp
004FA08C   |.  83EC 28                   sub esp,28
004FA08F   |.  8B45 08                   mov eax,dword ptr ss:[ebp+8]
```

## 修改后

```asm
004FA000 z>    90                        nop
004FA001       90                        nop
004FA002       90                        nop
004FA003       90                        nop
004FA004       90                        nop
004FA005       90                        nop
004FA006       90                        nop
004FA007       90                        nop
004FA008       90                        nop
004FA009       90                        nop
004FA00A       90                        nop
004FA00B       90                        nop
004FA00C       90                        nop
004FA00D       90                        nop
004FA00E       90                        nop
004FA00F       90                        nop
004FA010       90                        nop
004FA011       90                        nop
004FA012       33C9                      xor ecx,ecx
004FA014       3E:8B048D 00905200        mov eax,dword ptr ds:[ecx\*4+529000]
004FA01C       83F8 00                   cmp eax,0
004FA01F       74 69                     je short zy1.004FA08A                              ; |
004FA021       0FB691 00915200           movzx edx,byte ptr ds:[ecx+529100]
004FA028       3E:8B1C8D 00925200        mov ebx,dword ptr ds:[ecx\*4+529200]
004FA030       0FB6B9 00935200           movzx edi,byte ptr ds:[ecx+529300]
004FA037       48                        dec eax                                            ; |
004FA038       41                        inc ecx
004FA039       8138 CC558BEC             cmp dword ptr ds:[eax],EC8B55CC
004FA03F     ^ 74 D3                     je short zy1.004FA014
004FA041       8078 FF CC                cmp byte ptr ds:[eax-1],0CC
004FA045     ^ 74 CD                     je short zy1.004FA014
004FA047       8078 01 CC                cmp byte ptr ds:[eax+1],0CC
004FA04B     ^ 74 C7                     je short zy1.004FA014
004FA04D       90                        nop                                                ; |
004FA04E       83FF 01                   cmp edi,1
004FA051       75 09                     jnz short zy1.004FA05C                             ;
\zy1.004FA396
004FA053       8810                      mov byte ptr ds:[eax],dl
004FA055       FECB                      dec bl
004FA057       8858 01                   mov byte ptr ds:[eax+1],bl
004FA05A       90                        nop
004FA05B       90                        nop
004FA05C       83FF 05                   cmp edi,5                                          ;
/Arg4
004FA05F       75 11                     jnz short zy1.004FA072
004FA061       C600 0F                   mov byte ptr ds:[eax],0F                           ; |
004FA064       80C2 10                   add dl,10                                          ;
|Arg2
004FA067       8850 01                   mov byte ptr ds:[eax+1],dl
004FA06A       83EB 05                   sub ebx,5
004FA06D       8958 02                   mov dword ptr ds:[eax+2],ebx
004FA070       90                        nop
004FA071       90                        nop
004FA072       83FF 04                   cmp edi,4
004FA075       75 10                     jnz short zy1.004FA087
004FA077       80FA EB                   cmp dl,0EB
004FA07A       75 0B                     jnz short zy1.004FA087
004FA07C       C600 E9                   mov byte ptr ds:[eax],0E9
004FA07F       83EB 04                   sub ebx,4
004FA082       8958 01                   mov dword ptr ds:[eax+1],ebx
004FA085       90                        nop
004FA086       90                        nop
004FA087     ^ EB 8B                     jmp short zy1.004FA014
004FA089       90                        nop
004FA08A       90                        nop
```
