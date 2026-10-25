
```asm
0052A635    |.  33D2          xor edx,edx                                           ;  edx=0
0052A637    |.  52            push edx                                              ; /Arg2 => 00000000
0052A638    |.  50            push eax                                              ; |Arg1
0052A639    |.  8D45 FC       lea eax,dword ptr ss:[ebp-4]                          ; |
0052A63C    |.  E8 53F2EDFF   call yhds.00409894                                    ; \yhds.00409894
0052A641    |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]                          ;  确认码和0B25F1 异或结果转成10进制
0052A644    |.  0FB600        movzx eax,byte ptr ds:[eax]                           ;  eax=[eax]第1个数值
0052A647    |.  8B55 FC       mov edx,dword ptr ss:[ebp-4]
0052A64A    |.  0FB652 01     movzx edx,byte ptr ds:[edx+1]                         ;  edx=[eax+1]第2个数值
0052A64E    |.  03C2          add eax,edx                                           ;  EAX=eax+edx
0052A650    |.  B9 05000000   mov ecx,5                                             ;  ecx=5
0052A655    |.  99            cdq
0052A656    |.  F7F9          idiv ecx                                              ;  eax/ecx
0052A658    |.  80C2 34       add dl,34                                             ;  DL=eax%ecx+34
0052A65B    |.  8855 F8       mov byte ptr ss:[ebp-8],dl                            ;  [EBP-8]=DL
0052A65E    |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]
0052A661    |.  0FB640 02     movzx eax,byte ptr ds:[eax+2]                         ;  eax=[eax+2]第3个数值
0052A665    |.  8B55 FC       mov edx,dword ptr ss:[ebp-4]
0052A668    |.  0FB652 03     movzx edx,byte ptr ds:[edx+3]                         ;  edx=[eax+3]第4个数值
0052A66C    |.  03C2          add eax,edx                                           ;  EAX=eax+edx
0052A66E    |.  B9 05000000   mov ecx,5                                             ;  ecx=5
0052A673    |.  99            cdq
0052A674    |.  F7F9          idiv ecx                                              ;  eax/ecx
0052A676    |.  8BDA          mov ebx,edx
0052A678    |.  80C3 33       add bl,33                                             ;  BL=bl+33
0052A67B    |.  885D F9       mov byte ptr ss:[ebp-7],bl                            ;  [EBP-7]=BL
0052A67E    |.  8D45 F4       lea eax,dword ptr ss:[ebp-C]
0052A681    |.  8A55 F8       mov dl,byte ptr ss:[ebp-8]
0052A684    |.  E8 F3A5EDFF   call yhds.00404C7C
0052A689    |.  8B45 F4       mov eax,dword ptr ss:[ebp-C]
0052A68C    |.  8D55 FC       lea edx,dword ptr ss:[ebp-4]
0052A68F    |.  B9 1B000000   mov ecx,1B
0052A694    |.  E8 9BA9EDFF   call yhds.00405034
0052A699    |.  8D45 F0       lea eax,dword ptr ss:[ebp-10]
0052A69C    |.  8BD3          mov edx,ebx
0052A69E    |.  E8 D9A5EDFF   call yhds.00404C7C
0052A6A3    |.  8B45 F0       mov eax,dword ptr ss:[ebp-10]
0052A6A6    |.  8D55 FC       lea edx,dword ptr ss:[ebp-4]
0052A6A9    |.  B9 19000000   mov ecx,19
0052A6AE    |.  E8 81A9EDFF   call yhds.00405034                                    ;  把两个值连接到异或后的10进制数组后面
0052A6B3    |.  8BC6          mov eax,esi
0052A6B5    |.  8B55 FC       mov edx,dword ptr ss:[ebp-4]
0052A6B8    |.  E8 33A4EDFF   call yhds.00404AF0
0052A6BD    |.  33C0          xor eax,eax
0052A6BF    |.  5A            pop edx
0052A6C0    |.  59            pop ecx
0052A6C1    |.  59            pop ecx
0052A6C2    |.  64:8910       mov dword ptr fs:[eax],edx
0052A6C5    |.  68 E7A65200   push yhds.0052A6E7
0052A6CA    |>  8D45 F0       lea eax,dword ptr ss:[ebp-10]
0052A6CD    |.  BA 02000000   mov edx,2
0052A6D2    |.  E8 E9A3EDFF   call yhds.00404AC0
0052A6D7    |.  8D45 FC       lea eax,dword ptr ss:[ebp-4]
0052A6DA    |.  E8 BDA3EDFF   call yhds.00404A9C
====================================================================================call yhds.0052A6F0
0052A6F0    /$  55            push ebp
0052A6F1    |.  8BEC          mov ebp,esp
0052A6F3    |.  33C9          xor ecx,ecx
0052A6F5    |.  51            push ecx
0052A6F6    |.  51            push ecx
0052A6F7    |.  51            push ecx
0052A6F8    |.  51            push ecx
0052A6F9    |.  51            push ecx
0052A6FA    |.  51            push ecx
0052A6FB    |.  53            push ebx
0052A6FC    |.  56            push esi
0052A6FD    |.  8BF2          mov esi,edx
0052A6FF    |.  8BD8          mov ebx,eax                                           ;  EBX=EAX=刚形成的数值
0052A701    |.  33C0          xor eax,eax
0052A703    |.  55            push ebp
0052A704    |.  68 3CA85200   push yhds.0052A83C
0052A709    |.  64:FF30       push dword ptr fs:[eax]
0052A70C    |.  64:8920       mov dword ptr fs:[eax],esp
0052A70F    |.  81F3 8776FBDD xor ebx,DDFB7687                                      ;  EBX^DDFB7687
0052A715    |.  8BC3          mov eax,ebx                                           ;  EAX=EBX
0052A717    |.  33D2          xor edx,edx                                           ;  EDX=0
0052A719    |.  52            push edx                                              ; /Arg2 => 00000000
0052A71A    |.  50            push eax                                              ; |Arg1
0052A71B    |.  8D45 FC       lea eax,dword ptr ss:[ebp-4]                          ; |
0052A71E    |.  E8 71F1EDFF   call yhds.00409894                                    ; \结果转换为10进制
0052A723    |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]
0052A726    |.  0FB600        movzx eax,byte ptr ds:[eax]                           ;  EAX=第1位数值
0052A729    |.  8B55 FC       mov edx,dword ptr ss:[ebp-4]
0052A72C    |.  0FB652 01     movzx edx,byte ptr ds:[edx+1]                         ;  EDX=第2位数值
0052A730    |.  03C2          add eax,edx                                           ;  EAX=EAX+EDX
0052A732    |.  B9 05000000   mov ecx,5
0052A737    |.  99            cdq
0052A738    |.  F7F9          idiv ecx                                              ;  EAX/5
0052A73A    |.  80C2 66       add dl,66                                             ;  DL=EAX%5+66
0052A73D    |.  8855 F8       mov byte ptr ss:[ebp-8],dl                            ;  [EBP-8]=DL
0052A740    |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]
0052A743    |.  0FB640 02     movzx eax,byte ptr ds:[eax+2]                         ;  EAX=第3位数值
0052A747    |.  8B55 FC       mov edx,dword ptr ss:[ebp-4]
0052A74A    |.  0FB652 03     movzx edx,byte ptr ds:[edx+3]                         ;  EDX=第4位数值
0052A74E    |.  03C2          add eax,edx                                           ;  EAX=EAX+EDX
0052A750    |.  B9 05000000   mov ecx,5
0052A755    |.  99            cdq
0052A756    |.  F7F9          idiv ecx                                              ;  EAX/5
0052A758    |.  80C2 75       add dl,75                                             ;  DL=EAX%5+75
0052A75B    |.  8855 F9       mov byte ptr ss:[ebp-7],dl
0052A75E    |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]
0052A761    |.  0FB640 04     movzx eax,byte ptr ds:[eax+4]                         ;  EAX=第5位数值
0052A765    |.  8B55 FC       mov edx,dword ptr ss:[ebp-4]
0052A768    |.  0FB652 05     movzx edx,byte ptr ds:[edx+5]                         ;  EDX=第6位数值
0052A76C    |.  03C2          add eax,edx                                           ;  EAX=EAX+EDX
0052A76E    |.  B9 05000000   mov ecx,5
0052A773    |.  99            cdq
0052A774    |.  F7F9          idiv ecx                                              ;  EAX/5
0052A776    |.  80C2 7A       add dl,7A                                             ;  DL=EAX%5+7A
0052A779    |.  8855 FA       mov byte ptr ss:[ebp-6],dl
0052A77C    |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]
0052A77F    |.  0FB640 06     movzx eax,byte ptr ds:[eax+6]                         ;  EAX=第7位数值
0052A783    |.  8B55 FC       mov edx,dword ptr ss:[ebp-4]
0052A786    |.  0FB652 07     movzx edx,byte ptr ds:[edx+7]                         ;  EDX=第8位数值
```
