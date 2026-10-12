
```
004EB1A5    .  8985 5CD1FFFF               mov dword ptr ss:[ebp-2EA4],eax  
004EB1AB    .  8B85 ECD6FFFF               mov eax,dword ptr ss:[ebp-2914]  
004EB1B1    .  8B00                         mov eax,dword ptr ds:[eax]  
004EB1B3    .  8985 60D1FFFF                mov dword ptr ss:[ebp-2EA0],eax  
004EB1B9    .  8B85 5CD1FFFF                mov eax,dword ptr ss:[ebp-2EA4]  
004EB1BF    .  8D0485 04000000              lea eax,dword ptr ds:[eax*4+4]  
004EB1C6    .  50                            push eax                                      ; /n  
004EB1C7    .  8B85 ECD6FFFF                mov eax,dword ptr ss:[ebp-2914]                ; |  
004EB1CD    .  83C0 04                      add eax,4                                      ; |  
004EB1D0    .  50                            push eax                                      ; |src  
004EB1D1    .  FFB5 ECD6FFFF                 push dword ptr ss:[ebp-2914]                 ; |dest  
004EB1D7    .  FF15 38794E00                call dword ptr ds:[<&msvcrt.memmove>]         ; \memmove  
004EB1DD    .  83C4 0C                      add esp,0C  
004EB1E0    .  8B85 5CD1FFFF                mov eax,dword ptr ss:[ebp-2EA4]  
004EB1E6    .  8B8D ECD6FFFF                mov ecx,dword ptr ss:[ebp-2914]  
004EB1EC    .  8B95 60D1FFFF                mov edx,dword ptr ss:[ebp-2EA0]  
004EB1F2    .  891481                      mov dword ptr ds:[ecx+eax*4],edx  
004EB1F5    .^ E9 47FFFFFF                   jmp un-Aspr.004EB141---》前面 dump 出来的代码 1 结束  
004EB1FA    >  90                            nop  
004EB1FB    .  90                            nop  
004EB1FC    .  90                            nop  
004EB1FD    .  90                            nop  
004EB1FE    .  90                            nop  
004EB1FF    .  90                            nop  
004EB200    .  90                            nop  
004EB201    .  90                            nop  
004EB202    .  90                            nop  
004EB203    .  90                            nop  
004EB204    .  90                            nop  
004EB205    .  B8 4C814D00                  mov eax,un-Aspr.004D814C  
004EB20A    .- E9 49D5FeFF                   jmp un-Aspr.004D8758  
004EB20F       90                            nop  
004EB210       90                            nop  
004EB211       90                            nop  
004EB212       90                            nop  
004EB213       90                            nop  
004EB214       90                            nop  
004EB215       90                            nop  
004EB216       90                            nop  
004EB217       90                            nop  
004EB218       90                            nop  
004EB219       90                            nop  
004EB21A       90                            nop  
004EB21B       90                            nop  
004EB21C       90                            nop  
004EB21D       90                            nop  
004EB21E       90                            nop  
004EB21F       90                            nop  
004EB220       90                            nop  
004EB221       90                            nop  
004EB222       90                            nop  
004EB223   /$  55                            push ebp\------》前面 dump 出来的代码 2 开始  
004EB224   |.  8BEC                       mov ebp,esp  
004EB226   |.  51                           push ecx  
004EB227   |.  8B45 08                      mov eax,dword ptr ss:[ebp+8]  
004EB22A   |.  53                           push ebx  
004EB22B   |.  B9 10270000                 mov ecx,2710  
004EB230   |.  56                           push esi  
004EB231   |.  99                           cdq  
004EB232   |.  8BF1                        mov esi,ecx  
004EB234   |.  57                           push edi  
004EB235   |.  F7FE                        idiv esi  
004EB237   |.  8B45 0C                     mov eax,dword ptr ss:[ebp+C]  
004EB23A   |.  8BF9                        mov edi,ecx  
004EB23C   |.  8BD9                        mov ebx,ecx  
004EB23E   |.  8955 FC                      mov dword ptr ss:[ebp-4],edx  
004EB241   |.  99                           cdq  
004EB242   |.  F7FE                        idiv esi  
004EB244   |.  8B45 08                     mov eax,dword ptr ss:[ebp+8]  
004EB247   |.  8BF2                        mov esi,edx  
004EB249   |.  99                           cdq  
004EB24A   |.  F7FF                        idiv edi  
004EB24C   |.  8BF8                        mov edi,eax  
004EB24E   |.  8B45 0C                     mov eax,dword ptr ss:[ebp+C]  
004EB251   |.  99                           cdq  
004EB252   |.  0FAFFE                      imul edi,esi  
004EB255   |.  F7FB                        idiv ebx  
004EB257   |.  0FAF75 FC                   imul esi,dword ptr ss:[ebp-4]  
004EB25B   |.  33D2                        xor edx,edx  
004EB25D   |.  0FAF45 FC                   imul eax,dword ptr ss:[ebp-4]  
004EB261   |.  03C7                        add eax,edi  
004EB263   |.  5F                           pop edi  
004EB264   |.  F7F1                        div ecx  
004EB266   |.  B9 00E1F505                mov ecx,5F5E100  
004EB26B   |.  8BC2                        mov eax,edx  
004EB26D   |.  33D2                        xor edx,edx  
004EB26F   |.  69C0 10270000             imul eax,eax,2710  
004EB275   |.  03C6                        add eax,esi  
004EB277   |.  5E                           pop esi  
004EB278   |.  F7F1                        div ecx  
004EB27A   |.  5B                           pop ebx  
004EB27B   |.  8BC2                        mov eax,edx  
004EB27D   |.  C9                           leave  
004EB27E   \.  C2 0800                     retn 8----》前面 dump 出来的代码 2 结束  
```

修改完毕，重新做了一遍，给文件减了一下肥，修改了一些地方，在 `w98` 和 `winxp` 下作了测试，运行正常，但如果在 `winxp` 下装了瑞星就会运行出错，不知为什么，还望各位朋友指教，谢谢您耐着性子看完，有些东西写的不是很清楚，还忘见谅，有些东西需要自己细细体会和琢磨。自己太菜，有不对的地方还望指教。

2004.10.3 下午 5.20

`4dd466d0e1e0cc123afbe504da2462ca`
