
### 00409FF0 E8 BFF5FFFF call MultiDes.004095B0

`00409FF0    E8 BFF5FFFF     call MultiDes.004095B0`  
`00409FF5    8B00           mov eax,dword ptr ds:[eax]`  
`00409FF7    6A 40          push 40`  
`00409FF9    50            push eax`  
`00409FFA    68 9C874300   push MultiDes.0043879C           ; ASCII "Registration finished, thank for your registration!"`  
`00409FFF    8BCE          mov ecx,esi`  
`0040A001    C74424 38 07000> mov dword ptr ss:[esp+38],7`  
`0040A009    E8 FF2A0100   call MultiDes.0041CB0D`  
`0040A00E    8D4C24 1C     lea ecx,dword ptr ss:[esp+1C]`  
`0040A012    895C24 2C     mov dword ptr ss:[esp+2C],ebx`  
`0040A016    E8 CA490100   call MultiDes.0041E9E5`  
`0040A01B    57            push edi`  
`0040A01C    55            push ebp`  
`0040A01D    B9 88C64300   mov ecx,MultiDes.0043C688`  
`0040A022    E8 99F2FFFF   call MultiDes.004092C0`  
`0040A027    EB 45         jmp short MultiDes.0040A06E`  
`0040A029    83CF FF       or edi,FFFFFFFF`  
`0040A02C    8D4C24 10     lea ecx,dword ptr ss:[esp+10]`  
`0040A030    897C24 2C     mov dword ptr ss:[esp+2C],edi`  
`0040A034    E8 AC490100   call MultiDes.0041E9E5`  
`0040A039    8D5424 20     lea edx,dword ptr ss:[esp+20]`  
`0040A03D    B9 88C64300   mov ecx,MultiDes.0043C688`  
`0040A042    52            push edx`  
`0040A043    E8 68F5FFFF   call MultiDes.004095B0`  
`0040A048    8B00          mov eax,dword ptr ds:[eax]`  
`0040A04A    6A 10         push 10`  
`0040A04C    50            push eax`  
`0040A04D    68 60874300   push MultiDes.00438760         ; ASCII "Registration failed, please check the code and try again!"`  
`0040A052    8BCE          mov ecx,esi`  
`0040A054    C74424 38 08000> mov dword ptr ss:[esp+38],8`  
`0040A05C    E8 AC2A0100   call MultiDes.0041CB0D`  
`0040A061    8D4C24 20     lea ecx,dword ptr ss:[esp+20]     ; 返回到这里，向上找到可以处下断！`  
`0040A065    897C24 2C     mov dword ptr ss:[esp+2C],edi`  
`0040A069    E8 77490100   call MultiDes.0041E9E5`  
`0040A06E    8BCE          mov ecx,esi`  
`0040A070    E8 500D0100   call MultiDes.0041ADC5`  
`0040A075    8B4C24 24     mov ecx,dword ptr ss:[esp+24]`  
`0040A079    5F            pop edi`  
`0040A07A    5E            pop esi`  
`0040A07B    5D            pop ebp`  
`0040A07C    5B            pop ebx`  
`0040A07D    64:890D 0000000> mov dword ptr fs:[0],ecx`  
`0040A084    83C4 20       add esp,20`  
`0040A087    C3            retn                              ; 返回程序界面`

### ========================= 跟进 00409F43 E8 D8F4FFFF call MultiDes.00409420 =========================

`00409420    6A FF         push -1                            ; 跟进来到这里`  
`00409422    68 8FBB4200   push MultiDes.0042BB8F`  
`00409427    64:A1 00000000  mov eax,dword ptr fs:[0]`  
`0040942D    50            push eax`  
`0040942E    64:8925 0000000> mov dword ptr fs:[0],esp`  
`00409435    83EC 0C       sub esp,0C`  
`00409438    8B4424 20     mov eax,dword ptr ss:[esp+20]`  
`0040943C    53            push ebx`  
`0040943D    56            push esi`  
`0040943E    50            push eax`  
`0040943F    8D4C24 2C     lea ecx,dword ptr ss:[esp+2C]`  
`00409443    C74424 14 00000> mov dword ptr ss:[esp+14],0`  
`0040944B    E8 0A530100   call MultiDes.0041E75A`  
`00409450    BB 01000000   mov ebx,1`  
`00409455    8D4C24 28     lea ecx,dword ptr ss:[esp+28]`  
`00409459    895C24 1C     mov dword ptr ss:[esp+1C],ebx`  
`0040945D    E8 EDEF0000   call MultiDes.0041844F           ; 取用户名`  
`00409462    8D4C24 28     lea ecx,dword ptr ss:[esp+28]     ; ASCII "KuNgBiM"`  
`00409466    E8 98EF0000   call MultiDes.00418403`  
`0040946B    8D4C24 28     lea ecx,dword ptr ss:[esp+28]`  
`0040946F    E8 125A0100   call MultiDes.0041EE86           ; 用户名所有字符由小写转为大写`  
`00409474    6A 42         push 42                            ; ASCII "KUNGBIM"`  
`00409476    6A 2E         push 2E`  
`00409478    8D4C24 30     lea ecx,dword ptr ss:[esp+30]     ; 转换完毕后，重新赋值给 ecx`  
`0040947C    E8 34EB0000   call MultiDes.00417FB5`  
`00409481    6A 42         push 42`  
`00409483    6A 20         push 20`  
`00409485    8D4C24 30     lea ecx,dword ptr ss:[esp+30]`  
`00409489    E8 27EB0000   call MultiDes.00417FB5`  
`0040948E    8B4C24 28     mov ecx,dword ptr ss:[esp+28]     ; 取转换后的用户名，ASCII "KUNGBIM"`  
`00409492    8B41 F8       mov eax,dword ptr ds:[ecx-8]      ; 取用户名长度，ds:[009139B8]=00000007`  
`00409495    83F8 02       cmp eax,2                          ; 用户名长度与 2 比较`  
`00409498    7E 4C         jle short MultiDes.004094E6        ; 若用户名长度小于或等于 2 就跳向自定义用户名`  
`0040949A    8D5424 0C     lea edx,dword ptr ss:[esp+C]`  
`0040949E    6A 02         push 2                             ; （取用户名个数）2 入栈`  
`004094A0    52            push edx`  
`004094A1    8D4C24 30     lea ecx,dword ptr ss:[esp+30]     ; 取前 2 位转换后的用户名，ASCII "KUNGBIM"`  
`004094A5    E8 48EB0000   call MultiDes.00417FF2            ; 取固定字符串`  
`004094AA    68 44844300   push MultiDes.00438444            ; ASCII "wfeewwf3deda"`  
`004094AF    50            push eax`  
`004094B0    8D4424 10     lea eax,dword ptr ss:[esp+10]`  
`004094B4    C64424 24 02    mov byte ptr ss:[esp+24],2`  
`004094B9    50            push eax`  
`004094BA    E8 6F570100   call MultiDes.0041EC2E`  
`004094BF    50            push eax`  
`004094C0    8D4C24 2C     lea ecx,dword ptr ss:[esp+2C]`  
`004094C4    C64424 20 03    mov byte ptr ss:[esp+20],3`  
`004094C9    E8 04560100   call MultiDes.0041EAD2`  
`004094CE    8D4C24 08     lea ecx,dword ptr ss:[esp+8]`  
`004094D2    C64424 1C 02    mov byte ptr ss:[esp+1C],2`  
`004094D7    E8 09550100   call MultiDes.0041E9E5`  
`004094DC    885C24 1C     mov byte ptr ss:[esp+1C],bl`  
`004094E0    8D4C24 0C     lea ecx,dword ptr ss:[esp+C]`  
`004094E4    EB 58         jmp short MultiDes.0040953E`  
`004094E6    68 40844300   push MultiDes.00438440            ; 程序自定义用户名为“AA”来计算，ASCII "AA"`  
`004094EB    8D4C24 2C     lea ecx,dword ptr ss:[esp+2C]`  
`004094EF    E8 81580100   call MultiDes.0041ED75`  
`004094F4    8D4C24 08     lea ecx,dword ptr ss:[esp+8]`  
`004094F8    6A 02         push 2`  
`004094FA    51            push ecx`  
`004094FB    8D4C24 30     lea ecx,dword ptr ss:[esp+30]     ; 取自定义用户名，ASCII "AA"`  
`004094FF    E8 EEEA0000   call MultiDes.00417FF2            ; 取固定字符串`  
`00409504    68 44844300   push MultiDes.00438444            ; ASCII "wfeewwf3deda"`  
`00409509    8D5424 10     lea edx,dword ptr ss:[esp+10]`  
`0040950D    50            push eax`  
`0040950E    52            push edx`  
`0040950F    C64424 28 04    mov byte ptr ss:[esp+28],4`  
`00409514    E8 15570100   call MultiDes.0041EC2E`  
`00409519    50            push eax`  
`0040951A    8D4C24 2C     lea ecx,dword ptr ss:[esp+2C]`  
`0040951E    C64424 20 05    mov byte ptr ss:[esp+20],5`  
`00409523    E8 AA550100   call MultiDes.0041EAD2`  
`00409528    8D4C24 0C     lea ecx,dword ptr ss:[esp+C]`  
`0040952C    C64424 1C 04    mov byte ptr ss:[esp+1C],4`  
`00409531    E8 AF540100   call MultiDes.0041E9E5`  
`00409536    885C24 1C     mov byte ptr ss:[esp+1C],bl`  
`0040953A    8D4C24 08     lea ecx,dword ptr ss:[esp+8]`  
`0040953E    E8 A2540100   call MultiDes.0041E9E5`  
`00409543    8B7424 24     mov esi,dword ptr ss:[esp+24]`  
`00409547    8D4424 28     lea eax,dword ptr ss:[esp+28]`  
`0040954B    50            push eax`  
`0040954C    8BCE          mov ecx,esi`  
`0040954E    E8 07520100   call MultiDes.0041E75A`  
`00409553    895C24 10     mov dword ptr ss:[esp+10],ebx`  
`00409557    8D4C24 28     lea ecx,dword ptr ss:[esp+28]`  
`0040955B    C64424 1C 00    mov byte ptr ss:[esp+1C],0`  
`00409560    E8 80540100   call MultiDes.0041E9E5`  
`00409565    8B4C24 14     mov ecx,dword ptr ss:[esp+14]`  
`00409569    8BC6          mov eax,esi`
