
```
00408348     |.  75 39           |jnz short avprp.00408383  
0040834A     |.  66:837D C2 06   |cmp word ptr ss:[ebp-3E],6  
0040834F     |.  75 32           |jnz short avprp.00408383  
00408351     |.  66:837D C0 00   |cmp word ptr ss:[ebp-40],0  
00408356     |.  75 2B           |jnz short avprp.00408383  
00408358     |.  66:837D C6 57   |cmp word ptr ss:[ebp-3A],57  
0040835D     |.  75 24           |jnz short avprp.00408383  
0040835F     |.  66:837D C4 0E   |cmp word ptr ss:[ebp-3C],0E  
00408364     |.  75 1D           |jnz short avprp.00408383  
00408366     |.  8B85 1CFDFFFF   |mov eax,dword ptr ss:[ebp-2E4]  
0040836C     |.  50              |push eax                    ; /Arg2  
0040836D     |.  8B95 6CFFFFFF   |mov edx,dword ptr ss:[ebp-94]      ; |  
00408373     |.  52              |push edx                    ; |Arg1  
00408374     |.  E8 B3050000     |call avprp.0040892C         ; \avprp.0040892C  
00408379     |.  83C4 08         |add esp,8  
0040837C     |.  8BF8           |mov edi,eax  
0040837E     |.  E9 92010000     |jmp avprp.00408515  
00408383     |>  81FB 00202600   |cmp ebx,262000               ;跳到这里，比较后再 跳  
00408389     |.  75 39           |jnz short avprp.004083C4  
0040838B     |.  66:837D C2 06   |cmp word ptr ss:[ebp-3E],6  
00408390     |.  75 32           |jnz short avprp.004083C4  
00408392     |.  66:837D C0 00   |cmp word ptr ss:[ebp-40],0  
00408397     |.  75 2B           |jnz short avprp.004083C4  
00408399     |.  66:837D C6 59   |cmp word ptr ss:[ebp-3A],59  
0040839E     |.  75 24           |jnz short avprp.004083C4  
004083A0     |.  66:837D C4 43   |cmp word ptr ss:[ebp-3C],43  
004083A5     |.  75 1D           |jnz short avprp.004083C4  
004083A7     |.  8B85 1CFDFFFF   |mov eax,dword ptr ss:[ebp-2E4]  
004083AD     |.  50              |push eax                    ; /Arg2  
004083AE     |.  8B95 6CFFFFFF   |mov edx,dword ptr ss:[ebp-94]      ; |  
004083B4     |.  52              |push edx                    ; |Arg1  
004083B5     |.  E8 9A060000     |call avprp.00408A54         ; \avprp.00408A54  
004083BA     |.  83C4 08         |add esp,8  
004083BD     |.  8BF8           |mov edi,eax  
004083BF     |.  E9 51010000     |jmp avprp.00408515  
004083C4     |>  81FB 00202600   |cmp ebx,262000                ;跳到这里  
004083CA     |.  75 39           |jnz short avprp.00408405  
004083CC     |.  66:837D C2 06   |cmp word ptr ss:[ebp-3E],6  
004083D1     |.  75 32           |jnz short avprp.00408405  
004083D3     |.  66:837D C0 03   |cmp word ptr ss:[ebp-40],3  
004083D8     |.  75 2B           |jnz short avprp.00408405  
004083DA     |.  66:837D C6 5B   |cmp word ptr ss:[ebp-3A],5B  
004083DF     |.  75 24           |jnz short avprp.00408405  
004083E1     |.  66:837D C4 08   |cmp word ptr ss:[ebp-3C],8  
004083E6     |.  75 1D           |jnz short avprp.00408405  
004083E8     |.  8B85 1CFDFFFF   |mov eax,dword ptr ss:[ebp-2E4]  
004083EE     |.  50              |push eax                    ; /Arg2  
004083EF     |.  8B95 6CFFFFFF   |mov edx,dword ptr ss:[ebp-94]      ; |  
004083F5     |.  52              |push edx                    ; |Arg1  
004083F6     |.  E8 81070000     |call avprp.00408B7C         ; \avprp.00408B7C  
004083FB     |.  83C4 08         |add esp,8  
004083FE     |.  8BF8           |mov edi,eax  
00408400     |.  E9 10010000     |jmp avprp.00408515  
00408405     |>  81FB 10692400   |cmp ebx,246910                ;跳到这里，大小不一 样，跳  
0040840B     |.  75 39           |jnz short avprp.00408446  
0040840D     |.  66:837D C2 06   |cmp word ptr ss:[ebp-3E],6  
00408412     |.  75 32           |jnz short avprp.00408446  
00408414     |.  66:837D C0 04   |cmp word ptr ss:[ebp-40],4  
00408419     |.  75 2B           |jnz short avprp.00408446  
0040841B     |.  66:837D C6 61   |cmp word ptr ss:[ebp-3A],61  
00408420     |.  75 24           |jnz short avprp.00408446  
00408422     |.  66:837D C4 3B   |cmp word ptr ss:[ebp-3C],3B  
00408427     |.  75 1D           |jnz short avprp.00408446  
00408429     |.  8B85 1CFDFFFF   |mov eax,dword ptr ss:[ebp-2E4]  
0040842F     |.  50              |push eax                    ; /Arg2  
00408430     |.  8B95 6CFFFFFF   |mov edx,dword ptr ss:[ebp-94]      ; |  
00408436     |.  52              |push edx                    ; |Arg1  
00408437     |.  E8 A0020000     |call avprp.004086DC         ; \avprp.004086DC  
0040843C     |.  83C4 08         |add esp,8  
0040843F     |.  8BF8           |mov edi,eax  
00408441     |.  E9 CF000000     |jmp avprp.00408515  
00408446     |>  81FB 10630B00   |cmp ebx,0B6310                ;跳。  
0040844C     |.  75 39           |jnz short avprp.00408487  
0040844E     |.  66:837D C2 05   |cmp word ptr ss:[ebp-3E],5  
00408453     |.  75 32           |jnz short avprp.00408487  
00408455     |.  66:837D C0 00   |cmp word ptr ss:[ebp-40],0  
0040845A     |.  75 2B           |jnz short avprp.00408487  
0040845C     |.  66:837D C6 29   |cmp word ptr ss:[ebp-3A],29  
00408461     |.  75 24           |jnz short avprp.00408487  
00408463     |.  66:837D C4 15   |cmp word ptr ss:[ebp-3C],15  
00408468     |.  75 1D           |jnz short avprp.00408487  
0040846A     |.  8B85 1CFDFFFF   |mov eax,dword ptr ss:[ebp-2E4]  
00408470     |.  50              |push eax                    ; /Arg2  
00408471     |.  8B95 6CFFFFFF   |mov edx,dword ptr ss:[ebp-94]      ; |  
00408477     |.  52              |push edx                    ; |Arg1  
00408478     |.  E8 630B0000     |call avprp.00408FE0         ; \avprp.00408FE0  
0040847D     |.  83C4 08         |add esp,8  
00408480     |.  8BF8           |mov edi,eax  
00408482     |.  E9 8E000000     |jmp avprp.00408515  
00408487     |>  81FB 105F0B00   |cmp ebx,0B5F10                ;跳  
0040848D     |.  75 36           |jnz short avprp.004084C5  
0040848F     |.  66:837D C2 05   |cmp word ptr ss:[ebp-3E],5  
00408494     |.  75 2F           |jnz short avprp.004084C5  
00408496     |.  66:837D C0 00   |cmp word ptr ss:[ebp-40],0  
0040849B     |.  75 28           |jnz short avprp.004084C5  
0040849D     |.  66:837D C6 23   |cmp word ptr ss:[ebp-3A],23  
004084A2     |.  75 21           |jnz short avprp.004084C5  
004084A4     |.  66:837D C4 08   |cmp word ptr ss:[ebp-3C],8  
004084A9     |.  75 1A           |jnz short avprp.004084C5  
004084AB     |.  8B85 1CFDFFFF   |mov eax,dword ptr ss:[ebp-2E4]  
004084B1     |.  50              |push eax                    ; /Arg2  
004084B2     |.  8B95 6CFFFFFF   |mov edx,dword ptr ss:[ebp-94]      ; |  
004084B8     |.  52              |push edx                    ; |Arg1  
004084B9     |.  E8 AE0B0000     |call avprp.0040906C         ; \avprp.0040906C  
004084BE     |.  83C4 08         |add esp,8  
004084C1     |.  8BF8           |mov edi,eax  
004084C3     |.  EB 50           |jmp short avprp.00408515  
004084C5     >\66:837D C2 06   |cmp word ptr ss:[ebp-3E],6       ;  跳到了这里，停了 。获得版本号  
004084CA     |.  75 1B           |jnz short avprp.004084E7  
004084CC     |.  53              |push ebx                    ; /文件大小 263000  
004084CD     |.  8B85 1CFDFFFF   |mov eax,dword ptr ss:[ebp-2E4]     ; |Stack ss: [0167FCD4]=65000000  
004084D3     |.  50              |push eax                    ; |Arg2  
004084D4     |.  8B95 6CFFFFFF   |mov edx,dword ptr ss:[ebp-94]      ; |  
004084DA     |.  52              |push edx                    ; |Arg1  
004084DB     |.  E8 60080000     |call avprp.00408D40         ; \关键函数。  
004084E0     |.  83C4 0C         |add esp,0C  
```
