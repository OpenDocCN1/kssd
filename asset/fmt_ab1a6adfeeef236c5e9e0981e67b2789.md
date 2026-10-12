
```
00530CCA        .  8D45 F0         lea eax,dword ptr ss:[ebp-10]
00530CCD        .  50             push eax
00530CCE        .  56             push esi
00530CCF        .  E8 554CF5FF    call oC4PE.00485929             ;  验证函数 1
00530CD4        .  83C4 10        add esp,10
00530CD7        .  83E0 7F        and eax,7F
00530CDA        .  03F8           add edi,eax
00530CDC        .  8D45 DA       lea eax,dword ptr ss:[ebp-26]
00530CDF        .  50             push eax
00530CE0        .  8D45 DB       lea eax,dword ptr ss:[ebp-25]
00530CE3        .  50             push eax
00530CE4        .  56             push esi
00530CE5        .  E8 874DF5FF    call oC4PE.00485A71             ;  验证函数 2
00530CEA        .  83C4 0C        add esp,0C
00530CED        .  83E0 7F        and eax,7F
00530CF0        .  03F8           add edi,eax
00530CF2        .  56             push esi
00530CF3        .  E8 6B4EF5FF    call oC4PE.00485B63             ;  验证函数 3
00530CF8        .  59             pop ecx
00530CF9        .  83E0 7F        and eax,7F
00530CFC        .  03F8           add edi,eax
00530CFE        .  B9 1E000000    mov ecx,1E
00530D03        .  33D2           xor edx,edx
00530D05        .  8B45 E0        mov eax,dword ptr ss:[ebp-20]
00530D08        .  E8 83BAF4FF    call oC4PE.0047C790             ;  验证函 4
00530D0D        .  83E0 7F        and eax,7F
00530D10        .  03D8           add ebx,eax
00530D12        .  8B4D E8        mov ecx,dword ptr ss:[ebp-18]
00530D15        .  8B55 EC        mov edx,dword ptr ss:[ebp-14]
00530D18        .  8B45 F0        mov eax,dword ptr ss:[ebp-10]
00530D1B        .  E8 CC37F5FF    call oC4PE.004844EC             ;  验证函数 5
00530D20        .  B9 1E000000    mov ecx,1E
00530D25        .  33D2           xor edx,edx
00530D27        .  E8 64BAF4FF    call oC4PE.0047C790             ;  验证函数 6
00530D2C        .  83E0 7F        and eax,7F
00530D2F        .  03D8           add ebx,eax
00530D31        .  837D DC 02     cmp dword ptr ss:[ebp-24],2
00530D35        .  74 18          je short oC4PE.00530D4F         
00530D37        .  B9 9C0E5300    mov ecx,oC4PE.00530E9C
00530D3C        .  B2 01          mov dl,1
00530D3E        .  A1 7C414800    mov eax,dword ptr ds:[48417C]
00530D43        .  E8 8093EDFF    call oC4PE.0040A0C8
00530D48        .  E8 8B29EDFF    call oC4PE.004036D8
00530D4D        .  EB 38          jmp short oC4PE.00530D87
00530D4F        >  83FF 05       cmp edi,5
00530D52        .  74 05         je short oC4PE.00530D59         
00530D54        .  83FB 02       cmp ebx,2
00530D57        .  75 15         jnz short oC4PE.00530D6E
00530D59        >  33C9          xor ecx,ecx
00530D5B        .  B2 01         mov dl,1
00530D5D        .  A1 9C424800   mov eax,dword ptr ds:[48429C]
00530D62        .  E8 6193EDFF   call oC4PE.0040A0C8
00530D67        .  E8 6C29EDFF   call oC4PE.004036D8
```

由于过程与前面基本一样，我就不再赘述了。

根据验证函数 1，将序列号修改为`GP3EHA79-1ABB-12B55N7A`，

根据验证函数 2，将序列号修改为`GP3EHA79-2ABB-E2B55N7A`，

此时利用此序列号可以发现程序已经真正完成注册了。算是大功告成了！！！好累啊，休息了。

**【破解后话】**这个软件的破解其实从算法来说，并不复杂，但是需要耐心，而且需要不停的修改序列号，不停的调试。我就是准备放弃的时候，试了一下居然成功了。呵呵，那么在生活中其他事情也很有可能是“山穷水尽疑无路，柳暗花明又一村”。希望我的生活能够如此，看到这的看官们也能这样哦！！！

`4dd466d0e1e0cc123afbe504da2462ca`
