
```
00575B70    64:FF30         push dword ptr fs:[eax]  
00575B73    64:8920         mov dword ptr fs:[eax],esp  
00575B76    C645 FF 00      mov byte ptr ss:[ebp-1],0  
00575B7A    B8 E0515700     mov eax,ETDirLoc.005751E0  
00575B7F    E8 64F8FFFF     call ETDirLoc.005753E8  
00575B84    84C0            test al,al  
00575B86    74 0C           je short ETDirLoc.00575B94  
00575B88    A1 2CD15900     mov eax,dword ptr ds:[59D12C]  
00575B8D    8B00            mov eax,dword ptr ds:[eax]  
00575B8F    E8 5CF1FCFF     call ETDirLoc.00544CF0  
00575B94    B8 944F5700     mov eax,ETDirLoc.00574F94  
00575B99    E8 4AF8FFFF     call ETDirLoc.005753E8  
00575B9E    84C0            test al,al  
00575BA0    74 0C           je short ETDirLoc.00575BAE  
00575BA2    A1 2CD15900     mov eax,dword ptr ds:[59D12C]  
00575BA7    8B00            mov eax,dword ptr ds:[eax]  
00575BA9    E8 42F1FCFF     call ETDirLoc.00544CF0  
00575BAE    B8 A8505700     mov eax,ETDirLoc.005750A8  
00575BB3    E8 30F8FFFF     call ETDirLoc.005753E8  
00575BB8    84C0            test al,al  
00575BBA    74 0C           je short ETDirLoc.00575BC8  
00575BBC    A1 2CD15900     mov eax,dword ptr ds:[59D12C]  
00575BC1    8B00            mov eax,dword ptr ds:[eax]  
00575BC3    E8 28F1FCFF     call ETDirLoc.00544CF0  
00575BC8    8D45 F4         lea eax,dword ptr ss:[ebp-C]  
00575BCB    E8 BCFAFFFF     call ETDirLoc.0057568C                   ; 算法 call(2)  
00575BD0    8D4D F0         lea ecx,dword ptr ss:[ebp-10]  
00575BD3    8BD7            mov edx,edi                               ; edi=tigerisme  
00575BD5    A1 68EC5900     mov eax,dword ptr ds:[59EC68]  
00575BDA    E8 BDFCFFFF     call ETDirLoc.0057589C  
00575BDF    8B45 F0         mov eax,dword ptr ss:[ebp-10]  
00575BE2    8BD6            mov edx,esi  
00575BE4    E8 87F1F8FF     call ETDirLoc.00504D70                   ; 由算法 call（2）得出真正注册码“O8L708C1IOYFV086YK9S”  
00575BE9    0F85 A0000000   jnz ETDirLoc.00575C8F  
00575BEF    B2 01           mov dl,1  
00575BF1    A1 FC465700     mov eax,dword ptr ds:[5746FC]  
00575BF6    E8 01ECFFFF     call ETDirLoc.005747FC  
00575BFB    8BD8            mov ebx,eax  
00575BFD    BA 00000080     mov edx,80000000  
00575C02    8BC3            mov eax,ebx  
00575C04    E8 93ECFFFF     call ETDirLoc.0057489C  
00575C09    8D45 F8         lea eax,dword ptr ss:[ebp-8]  
00575C0C    50              push eax  
00575C0D    B9 01000000     mov ecx,1  
00575C12    BA 12000000     mov edx,12  
00575C17    A1 68EC5900     mov eax,dword ptr ds:[59EC68]  
00575C1C    E8 B7FBFFFF     call ETDirLoc.005757D8  
00575C21    B1 01           mov cl,1  
00575C23    8B55 F8         mov edx,dword ptr ss:[ebp-8]  
00575C26    8BC3            mov eax,ebx  
00575C28    E8 D7ECFFFF     call ETDirLoc.00574904  
00575C2D    84C0            test al,al  
00575C2F    74 50           je short ETDirLoc.00575C81  
00575C31    8D55 EC         lea edx,dword ptr ss:[ebp-14]  
00575C34    8BC6            mov eax,esi  
00575C36    E8 11FBFFFF     call ETDirLoc.0057574C  
00575C3B    8B45 EC         mov eax,dword ptr ss:[ebp-14]  
00575C3E    50              push eax  
00575C3F    8D55 E8         lea edx,dword ptr ss:[ebp-18]  
00575C42    B8 C45C5700     mov eax,ETDirLoc.00575CC4                ; ASCII "cpsn"  
00575C47    E8 00FBFFFF     call ETDirLoc.0057574C  
00575C4C    8B55 E8         mov edx,dword ptr ss:[ebp-18]  
00575C4F    8BC3            mov eax,ebx  
00575C51    59              pop ecx  
00575C52    E8 49F0FFFF     call ETDirLoc.00574CA0  
00575C57    8D55 E4         lea edx,dword ptr ss:[ebp-1C]  
00575C5A    8BC7            mov eax,edi  
00575C5C    E8 EBFAFFFF     call ETDirLoc.0057574C  
00575C61    8B45 E4         mov eax,dword ptr ss:[ebp-1C]  
00575C64    50              push eax  
00575C65    8D55 E0         lea edx,dword ptr ss:[ebp-20]  
00575C68    B8 D45C5700     mov eax,ETDirLoc.00575CD4                ; ASCII "user"  
00575C6D    E8 DAFAFFFF     call ETDirLoc.0057574C  
00575C72    8B55 E0         mov edx,dword ptr ss:[ebp-20]  
00575C75    8BC3            mov eax,ebx  
00575C77    59              pop ecx  
00575C78    E8 23F0FFFF     call ETDirLoc.00574CA0  
00575C7D    C645 FF 01      mov byte ptr ss:[ebp-1],1  
00575C81    8BC3            mov eax,ebx  
00575C83    E8 E4EBFFFF     call ETDirLoc.0057486C  
00575C88    8BC3            mov eax,ebx  
00575C8A    E8 F1DEF8FF     call ETDirLoc.00503B80  
00575C8F    33C0            xor eax,eax  
00575C91    5A              pop edx  
00575C92    59              pop ecx  
00575C93    59              pop ecx  
00575C94    64:8910         mov dword ptr fs:[eax],edx  
00575C97    68 B15C5700     push ETDirLoc.00575CB1  
00575C9C    8D45 E0         lea eax,dword ptr ss:[ebp-20]  
00575C9F    BA 07000000     mov edx,7  
00575CA4    E8 EFECF8FF     call ETDirLoc.00504998  
00575CA9    C3              retn  
00575CAA  ^ E9 65E6F8FF     jmp ETDirLoc.00504314  
00575CAF  ^ EB EB           jmp short ETDirLoc.00575C9C  
00575CB1    8A45 FF         mov al,byte ptr ss:[ebp-1]  
00575CB4    5F              pop edi  
00575CB5    5E              pop esi  
00575CB6    5B              pop ebx  
00575CB7    8BE5            mov esp,ebp  
00575CB9    5D              pop ebp  
00575CBA    C3              retn  
```

`****************************************************************`

### `算法 call(2)`

```
0057568C    55              push ebp  
0057568D    8BEC            mov ebp,esp  
0057568F    33C9            xor ecx,ecx  
00575691    51              push ecx  
00575692    51              push ecx  
00575693    51              push ecx  
00575694    51              push ecx  
00575695    51              push ecx  
00575696    53              push ebx  
00575697    8BD8            mov ebx,eax  
00575699    33C0            xor eax,eax  
0057569B    55              push ebp  
0057569C    68 3D575700     push ETDirLoc.0057573D  
005756A1    64:FF30         push dword ptr fs:[eax]  
005756A4    64:8920         mov dword ptr fs:[eax],esp  
005756A7    8D45 FC         lea eax,dword ptr ss:[ebp-4]  
005756AA    E8 5DFEFFFF     call ETDirLoc.0057550C  
005756AF    8D55 F8         lea edx,dword ptr ss:[ebp-8]  
005756B2    8B45 FC         mov eax,dword ptr ss:[ebp-4]         ; WD-WCAD19885326=ebp-4 送 eax  
005756B5    E8 6238F9FF     call ETDirLoc.00508F1C  
005756BA    8B55 F8         mov edx,dword ptr ss:[ebp-8]         ; ebp-8=WD-WCAD19885326，送 edx  
005756BD    8D45 FC         lea eax,dword ptr ss:[ebp-4]  
005756C0    E8 47F3F8FF     call ETDirLoc.00504A0C  
005756C5    837D FC 00      cmp dword ptr ss:[ebp-4],0           ; WD-WCAD19885326 与 0 比较  
005756C9    74 22           je short ETDirLoc.005756ED  
005756CB    8D45 F4         lea eax,dword ptr ss:[ebp-C]  
005756CE    50              push eax  
005756CF    33C9            xor ecx,ecx  
005756D1    BA 12000000     mov edx,12  
005756D6    8B45 FC         mov eax,dword ptr ss:[ebp-4]         ; WD-WCAD19885326=ebp-4 送 eax  
005756D9    E8 FA000000     call ETDirLoc.005757D8               ; 算法 call（3）  
005756DE    8B55 F4         mov edx,dword ptr ss:[ebp-C]         ; 由前个 call 得出“14121011813951089292015”  
005756E1    B8 68EC5900     mov eax,ETDirLoc.0059EC68  
005756E6    E8 DDF2F8FF     call ETDirLoc.005049C8  
005756EB    EB 28           jmp short ETDirLoc.00575715  
005756ED    8D45 F0         lea eax,dword ptr ss:[ebp-10]  
005756F0    50              push eax  
005756F1    8D45 EC         lea eax,dword ptr ss:[ebp-14]  
005756F4    E8 2FFDFFFF     call ETDirLoc.00575428  
005756F9    8B45 EC         mov eax,dword ptr ss:[ebp-14]  
005756FC    33C9            xor ecx,ecx  
005756FE    BA 12000000     mov edx,12  
```
