
## 二、算法分析

```
0040BB10              6A FF                push -1                                         ；按钮事件
0040BB12              68 034F4300          push TifImage.00434F03
0040BB17              64:A1 00000000       mov eax,dword ptr fs:[0]
0040BB1D              50                   push eax
0040BB1E              64:8925 00000000     mov dword ptr fs:[0],esp
0040BB25              81EC 18060000        sub esp,618
0040BB2B              A1 00FD4400          mov eax,dword ptr ds:[44FD00]
0040BB30              33C4                 xor eax,esp
0040BB32              56                   push esi
0040BB33              57                   push edi
0040BB34              6A 01                push 1
0040BB36              898424 20060000      mov dword ptr ss:[esp+620],eax
0040BB3D              8BF1                 mov esi,ecx
0040BB3F              E8 46C70100          call TifImage.0042828A                             ; 读取用户名
0040BB44              8B46 70              mov eax,dword ptr ds:[esi+70]                        返回处
0040BB47              8B48 F4              mov ecx,dword ptr ds:[eax-C]                       ; 用户名长度
0040BB4A              83F9 02              cmp ecx,2                                          ; 用户名长度是否大于等于2
0040BB4D              0F8D E5000000        jge TifImage.0040BC38                              ; 小于则OVER
0040BB53              E8 9FEE0100          call TifImage.0042A9F7
0040BB58              8B10                 mov edx,dword ptr ds:[eax]
0040BB5A              8BC8                 mov ecx,eax
0040BB5C              FF52 0C              call dword ptr ds:[edx+C]
0040BB5F              83C0 10              add eax,10
0040BB62              894424 10            mov dword ptr ss:[esp+10],eax
0040BB66              6A 67                push 67
0040BB68              8D4424 0C            lea eax,dword ptr ss:[esp+C]
0040BB6C              50                   push eax
0040BB6D              B9 800C4500          mov ecx,TifImage.00450C80
0040BB72              C78424 30060000 0000>mov dword ptr ss:[esp+630],0
0040BB7D              E8 3E71FFFF          call TifImage.00402CC0
0040BB82              50                   push eax
0040BB83              8D4C24 14            lea ecx,dword ptr ss:[esp+14]
0040BB87              C68424 2C060000 01   mov byte ptr ss:[esp+62C],1
0040BB8F              E8 DC69FFFF          call TifImage.00402570
0040BB94              8B4424 08            mov eax,dword ptr ss:[esp+8]
0040BB98              83C0 F0              add eax,-10
0040BB9B              C68424 28060000 00   mov byte ptr ss:[esp+628],0
0040BBA3              8D48 0C              lea ecx,dword ptr ds:[eax+C]
0040BBA6              83CA FF              or edx,FFFFFFFF
0040BBA9              F0:0FC111            lock xadd dword ptr ds:[ecx],edx
0040BBAD              4A                   dec edx
0040BBAE              85D2                 test edx,edx
0040BBB0              7F 08                jg short TifImage.0040BBBA
0040BBB2              8B08                 mov ecx,dword ptr ds:[eax]
0040BBB4              8B11                 mov edx,dword ptr ds:[ecx]
0040BBB6              50                   push eax
0040BBB7              FF52 04              call dword ptr ds:[edx+4]
0040BBBA              6A 69                push 69
0040BBBC              8D4424 0C            lea eax,dword ptr ss:[esp+C]
0040BBC0              50                   push eax
0040BBC1              B9 800C4500          mov ecx,TifImage.00450C80
0040BBC6              E8 F570FFFF          call TifImage.00402CC0
0040BBCB              8B00                 mov eax,dword ptr ds:[eax]
0040BBCD              8B7C24 10            mov edi,dword ptr ss:[esp+10]
0040BBD1              6A 00                push 0
0040BBD3              50                   push eax
0040BBD4              57                   push edi
0040BBD5              8BCE                 mov ecx,esi
0040BBD7              C68424 34060000 02   mov byte ptr ss:[esp+634],2
0040BBDF              E8 48C60100          call TifImage.0042822C
0040BBE4              8B4424 08            mov eax,dword ptr ss:[esp+8]
0040BBE8              83C0 F0              add eax,-10
0040BBEB              C68424 28060000 00   mov byte ptr ss:[esp+628],0
0040BBF3              8D48 0C              lea ecx,dword ptr ds:[eax+C]
0040BBF6              83CA FF              or edx,FFFFFFFF
0040BBF9              F0:0FC111            lock xadd dword ptr ds:[ecx],edx
0040BBFD              4A                   dec edx
0040BBFE              85D2                 test edx,edx
0040BC00              7F 08                jg short TifImage.0040BC0A
0040BC02              8B08                 mov ecx,dword ptr ds:[eax]
0040BC04              8B11                 mov edx,dword ptr ds:[ecx]
0040BC06              50                   push eax
0040BC07              FF52 04              call dword ptr ds:[edx+4]
0040BC0A              8D47 F0              lea eax,dword ptr ds:[edi-10]
0040BC0D              C78424 28060000 FFFF>mov dword ptr ss:[esp+628],-1
0040BC18              8D48 0C              lea ecx,dword ptr ds:[eax+C]
0040BC1B              83CA FF              or edx,FFFFFFFF
0040BC1E              F0:0FC111            lock xadd dword ptr ds:[ecx],edx
0040BC22              4A                   dec edx
0040BC23              85D2                 test edx,edx
0040BC25              0F8F 86030000        jg TifImage.0040BFB1
0040BC2B              8B08                 mov ecx,dword ptr ds:[eax]
0040BC2D              8B11                 mov edx,dword ptr ds:[ecx]
0040BC2F              50                   push eax
0040BC30              FF52 04              call dword ptr ds:[edx+4]
0040BC33              E9 79030000          jmp TifImage.0040BFB1
0040BC38              8B4E 74              mov ecx,dword ptr ds:[esi+74]
0040BC3B              8B49 F4              mov ecx,dword ptr ds:[ecx-C]                假码长度
0040BC3E              83F9 08              cmp ecx,8                                          ; 假码长度是否大于等于8位？
0040BC41              53                   push ebx
0040BC42              0F8D BE000000        jge TifImage.0040BD06                              ; 小于则OVER
0040BC48              E8 AAED0100          call TifImage.0042A9F7
0040BC4D              8B10                 mov edx,dword ptr ds:[eax]
0040BC4F              8BC8                 mov ecx,eax
0040BC51              FF52 0C              call dword ptr ds:[edx+C]
0040BC54              83C0 10              add eax,10
0040BC57              894424 14            mov dword ptr ss:[esp+14],eax
0040BC5B              6A 68                push 68
0040BC5D              8D4424 10            lea eax,dword ptr ss:[esp+10]
0040BC61              BB 03000000          mov ebx,3
0040BC66              50                   push eax
0040BC67              B9 800C4500          mov ecx,TifImage.00450C80
0040BC6C              899C24 34060000      mov dword ptr ss:[esp+634],ebx
0040BC73              E8 4870FFFF          call TifImage.00402CC0
0040BC78              50                   push eax
0040BC79              8D4C24 18            lea ecx,dword ptr ss:[esp+18]
0040BC7D              C68424 30060000 04   mov byte ptr ss:[esp+630],4
0040BC85              E8 E668FFFF          call TifImage.00402570
0040BC8A              8B4424 0C            mov eax,dword ptr ss:[esp+C]
0040BC8E              83C0 F0              add eax,-10
0040BC91              889C24 2C060000      mov byte ptr ss:[esp+62C],bl
0040BC98              8D48 0C              lea ecx,dword ptr ds:[eax+C]
0040BC9B              83CA FF              or edx,FFFFFFFF
0040BC9E              F0:0FC111            lock xadd dword ptr ds:[ecx],edx
0040BCA2              4A                   dec edx
0040BCA3              85D2                 test edx,edx
0040BCA5              7F 08                jg short TifImage.0040BCAF
0040BCA7              8B08                 mov ecx,dword ptr ds:[eax]
0040BCA9              8B11                 mov edx,dword ptr ds:[ecx]
0040BCAB              50                   push eax
0040BCAC              FF52 04              call dword ptr ds:[edx+4]
0040BCAF              6A 69                push 69
0040BCB1              8D4424 10            lea eax,dword ptr ss:[esp+10]
0040BCB5              50                   push eax
0040BCB6              B9 800C4500          mov ecx,TifImage.00450C80
0040BCBB              E8 0070FFFF          call TifImage.00402CC0
0040BCC0              8B00                 mov eax,dword ptr ds:[eax]
0040BCC2              8B7C24 14            mov edi,dword ptr ss:[esp+14]
0040BCC6              6A 00                push 0
0040BCC8              50                   push eax
0040BCC9              57                   push edi
0040BCCA              8BCE                 mov ecx,esi
0040BCCC              C68424 38060000 05   mov byte ptr ss:[esp+638],5
0040BCD4              E8 53C50100          call TifImage.0042822C
0040BCD9              8B4424 0C            mov eax,dword ptr ss:[esp+C]
0040BCDD              83C0 F0              add eax,-10
0040BCE0              889C24 2C060000      mov byte ptr ss:[esp+62C],bl
0040BCE7              8D48 0C              lea ecx,dword ptr ds:[eax+C]
0040BCEA              83CA FF              or edx,FFFFFFFF
0040BCED              F0:0FC111            lock xadd dword ptr ds:[ecx],edx
0040BCF1              4A                   dec edx
0040BCF2              85D2                 test edx,edx
0040BCF4              7F 08                jg short TifImage.0040BCFE
0040BCF6              8B08                 mov ecx,dword ptr ds:[eax]
0040BCF8              8B11                 mov edx,dword ptr ds:[ecx]
0040BCFA              50                   push eax
0040BCFB              FF52 04              call dword ptr ds:[edx+4]
0040BCFE              8D47 F0              lea eax,dword ptr ds:[edi-10]
0040BD01              E9 88020000          jmp TifImage.0040BF8E
0040BD06              8B3D B8624300        mov edi,dword ptr ds:[<&KERNEL32.lstrcpyW>]        ; kernel32.lstrcpyW
0040BD0C              55                   push ebp
0040BD0D              50                   push eax
0040BD0E              8D8424 28040000      lea eax,dword ptr ss:[esp+428]
0040BD15              50                   push eax
0040BD16              FFD7                 call edi
0040BD18              8B2D A4624300        mov ebp,dword ptr ds:[<&KERNEL32.WideCharToMultiBy>; kernel32.WideCharToMultiByte
0040BD1E              6A 00                push 0
0040BD20              6A 00                push 0
0040BD22              68 00020000          push 200
0040BD27              8D8C24 30020000      lea ecx,dword ptr ss:[esp+230]
0040BD2E              51                   push ecx
0040BD2F              6A FF                push -1
0040BD31              8D9424 38040000      lea edx,dword ptr ss:[esp+438]
0040BD38              52                   push edx
0040BD39              6A 00                push 0
0040BD3B              6A 00                push 0
0040BD3D              FFD5                 call ebp
0040BD3F              8B46 74              mov eax,dword ptr ds:[esi+74]
0040BD42              50                   push eax
0040BD43              8D8424 28040000      lea eax,dword ptr ss:[esp+428]
0040BD4A              50                   push eax
0040BD4B              FFD7                 call edi
0040BD4D              6A 00                push 0
0040BD4F              6A 00                push 0
0040BD51              68 00020000          push 200
0040BD56              8D4C24 30            lea ecx,dword ptr ss:[esp+30]
0040BD5A              51                   push ecx
0040BD5B              6A FF                push -1
0040BD5D              8D9424 38040000      lea edx,dword ptr ss:[esp+438]
0040BD64              52                   push edx
0040BD65              6A 00                push 0
0040BD67              6A 00                push 0
0040BD69              FFD5                 call ebp                                           ; kernel32.WideCharToMultiByte
0040BD6B              0FB68C24 24020000    movzx ecx,byte ptr ss:[esp+224]                    ; 用户名第1位ASCII值传送到ECX，设用户名数组name[i]
0040BD73              8BC1                 mov eax,ecx                                        ; eax=ecx=name[0]=71h
0040BD75              83C8 54              or eax,54                                          ; eax=eax or 54h
0040BD78              99                   cdq
0040BD79              BF 0A000000          mov edi,0A                                         ; edi=0Ah，即十进制10
0040BD7E              F7FF                 idiv edi                                           ; 我们只关注余数，后面计算也是。
                                                                                                 ;(name[0] or 54h) % 0Ah=7,保存在edx
0040BD80              0FB6BC24 25020000    movzx edi,byte ptr ss:[esp+225]                    ; 用户名第2位ASCII值传送到EDI
0040BD88              8BC7                 mov eax,edi                                        ; eax=edi=name[1]=69h
0040BD8A              83C8 49              or eax,49                                          ; eax=name[1] or 49h
0040BD8D              BD 0A000000          mov ebp,0A                                         ; ebp=0Ah
0040BD92              8ADA                 mov bl,dl                                          ;余数保存， bl=dl=7
0040BD94              99                   cdq
0040BD95              F7FD                 idiv ebp                                           ;(name[0] r49h) % 0Ah=5保存在edx
0040BD97              8BC1                 mov eax,ecx                                        ; eax=ecx=71h
0040BD99              83C8 46              or eax,46                                          ; eax=eax or 46h
0040BD9C              8BCD                 mov ecx,ebp                                          ecx=ebp=0Ah
0040BD9E              5D                   pop ebp
0040BD9F              885424 13            mov byte ptr ss:[esp+13],dl                        ; 余数保存到局部变量，[esp+13]=dl=5
0040BDA3              99                   cdq
0040BDA4              F7F9                 idiv ecx                                           ; name[0]or46h) % 0Ah=9保存在edx
0040BDA6              8BC7                 mov eax,edi                                        ; eax=edi=69
0040BDA8              83C8 46              or eax,46                                          ; eax=eax or 46h
0040BDAB              885424 1B            mov byte ptr ss:[esp+1B],dl                        ; 余数保存到局部变量，[esp+1B]=dl=9
0040BDAF              99                   cdq
0040BDB0              F7F9                 idiv ecx                                           ;(name[1]or46h) % 0Ah=1保存在edx
0040BDB2              8D8C24 20020000      lea ecx,dword ptr ss:[esp+220]                     ; 用户名指针保存到ECX
0040BDB9              33C0                 xor eax,eax
0040BDBB              8D79 01              lea edi,dword ptr ds:[ecx+1]
0040BDBE              885424 1A            mov byte ptr ss:[esp+1A],dl                        ; 余数保存到局部变量，[esp+1A]=dl=1
0040BDC2              8A11                 mov dl,byte ptr ds:[ecx]
0040BDC4              41                   inc ecx
0040BDC5              84D2                 test dl,dl
0040BDC7            ^ 75 F9                jnz short TifImage.0040BDC2
0040BDC9              2BCF                 sub ecx,edi
0040BDCB              33D2                 xor edx,edx
0040BDCD              85C9                 test ecx,ecx
0040BDCF              7E 0F                jle short TifImage.0040BDE0
0040BDD1              0FB6BC14 20020000    movzx edi,byte ptr ss:[esp+edx+220]                ; 循环取用户名第ASCII值传送到EDI
0040BDD9              03C7                 add eax,edi                                        ; EAX=EAX+EDI
0040BDDB              42                   inc edx
0040BDDC              3BD1                 cmp edx,ecx
0040BDDE            ^ 7C F1                jl short TifImage.0040BDD1                         ; 小循环计算用户名ASCII值之和，设为sum
0040BDE0              99                   cdq
0040BDE1              B9 0A000000          mov ecx,0A
0040BDE6              F7F9                 idiv ecx                                           ; sum % 0Ah=2,保存在edx
0040BDE8              8A4424 20            mov al,byte ptr ss:[esp+20]                        ; 假码首位ASIII值
0040BDEC              0FB6C8               movzx ecx,al
0040BDEF              0FB6FB               movzx edi,bl                                        ;bl= (name[0] or 54h) % 0Ah=7
0040BDF2              83E9 30              sub ecx,30
0040BDF5              3BF9                 cmp edi,ecx                                        ; 实际上等于判断：第1位是否为‘7’
0040BDF7              8A4C24 21            mov cl,byte ptr ss:[esp+21]                        ; 假码第2位ASIII值
0040BDFB              75 40                jnz short TifImage.0040BE3D
0040BDFD              0FB65C24 13          movzx ebx,byte ptr ss:[esp+13]                     ;[esp+13]=(name[0]or49h) % 0Ah=5       
0040BE02              0FB6F9               movzx edi,cl
0040BE05              83EF 30              sub edi,30
0040BE08              3BDF                 cmp ebx,edi                                        ; 第2位是否为‘5’
0040BE0A              75 31                jnz short TifImage.0040BE3D
0040BE0C              0FB67C24 22          movzx edi,byte ptr ss:[esp+22]                     ; 假码第3位ASIII值
0040BE11              0FB65C24 1B          movzx ebx,byte ptr ss:[esp+1B]                     [esp+1B]=(name[0] or 46h) % 0Ah=9,
0040BE16              83EF 30              sub edi,30
0040BE19              3BDF                 cmp ebx,edi                                        ; 第3位是否为‘9’
0040BE1B              75 20                jnz short TifImage.0040BE3D
0040BE1D              0FB67C24 23          movzx edi,byte ptr ss:[esp+23]                     ; 假码第4位ASIII值
0040BE22              0FB65C24 1A          movzx ebx,byte ptr ss:[esp+1A]                      esp+1A]=(name[1]or46h) % 0Ah=1
0040BE27              83EF 30              sub edi,30
0040BE2A              3BDF                 cmp ebx,edi                                        ; 第4位是否为‘1’
0040BE2C              75 0F                jnz short TifImage.0040BE3D
0040BE2E              0FB67C24 24          movzx edi,byte ptr ss:[esp+24]                     ; 假码第5位ASIII值
0040BE33              0FB6D2               movzx edx,dl                                       ;dl= sum % 0Ah=2,   
0040BE36              83EF 30              sub edi,30
0040BE39              3BD7                 cmp edx,edi                                        ; 第5位是否为‘2’
0040BE3B              74 57                je short TifImage.0040BE94                         ; 相等则OK,不等则进入固定注册码的验证
上面是对注册码前5位验证，都满足即可成功，否则进入另外固定码验证
以下是对一组固定注册码的验证
0040BE3D              3C 35                cmp al,35                                          ; 假码第1位是否为‘5’
0040BE3F              0F85 D6000000        jnz TifImage.0040BF1B
0040BE45              80F9 32              cmp cl,32                                          ; 假码第2位是否为‘2’
0040BE48              0F85 CD000000        jnz TifImage.0040BF1B
0040BE4E              8A4424 22            mov al,byte ptr ss:[esp+22]
0040BE52              B1 36                mov cl,36
0040BE54              3AC1                 cmp al,cl                                          ; 假码第3位是否为‘6’
0040BE56              0F85 BF000000        jnz TifImage.0040BF1B
0040BE5C              8A5424 23            mov dl,byte ptr ss:[esp+23]
0040BE60              B0 39                mov al,39
0040BE62              3AD0                 cmp dl,al                                          ; 假码第4位是否为‘9’
0040BE64              0F85 B1000000        jnz TifImage.0040BF1B
0040BE6A              807C24 24 37         cmp byte ptr ss:[esp+24],37                        ; 假码第5位是否为‘7’
0040BE6F              0F85 A6000000        jnz TifImage.0040BF1B
0040BE75              384C24 25            cmp byte ptr ss:[esp+25],cl                        ; 假码第6位是否为‘6’
0040BE79              0F85 9C000000        jnz TifImage.0040BF1B
0040BE7F              807C24 26 32         cmp byte ptr ss:[esp+26],32                        ; 假码第7位是否为‘2’
0040BE84              0F85 91000000        jnz TifImage.0040BF1B
0040BE8A              384424 27            cmp byte ptr ss:[esp+27],al                        ; 假码第8位是否为‘9’
0040BE8E              0F85 87000000        jnz TifImage.0040BF1B
0040BE94              6A 6A                push 6A
0040BE96              8D4424 10            lea eax,dword ptr ss:[esp+10]
0040BE9A              50                   push eax
0040BE9B              B9 800C4500          mov ecx,TifImage.00450C80
0040BEA0              E8 1B6EFFFF          call TifImage.00402CC0
0040BEA5              8B00                 mov eax,dword ptr ds:[eax]
0040BEA7              6A 00                push 0
0040BEA9              68 10844300          push TifImage.00438410                             ; ok
0040BEAE              50                   push eax
0040BEAF              8BCE                 mov ecx,esi
0040BEB1              C78424 38060000 0600>mov dword ptr ss:[esp+638],6
0040BEBC              E8 6BC30100          call TifImage.0042822C
0040BEC1              8D4C24 0C            lea ecx,dword ptr ss:[esp+C]
0040BEC5              C78424 2C060000 FFFF>mov dword ptr ss:[esp+62C],-1
0040BED0              E8 FB57FFFF          call TifImage.004016D0
0040BED5              8B7E 70              mov edi,dword ptr ds:[esi+70]
0040BED8              E8 CD470200          call TifImage.004306AA
0040BEDD              8B40 04              mov eax,dword ptr ds:[eax+4]
0040BEE0              57                   push edi
0040BEE1              68 847C4300          push TifImage.00437C84                             ; username
0040BEE6              68 EC744300          push TifImage.004374EC                             ; option
0040BEEB              8BC8                 mov ecx,eax
0040BEED              E8 25340200          call TifImage.0042F317
0040BEF2              8B7E 74              mov edi,dword ptr ds:[esi+74]
0040BEF5              E8 B0470200          call TifImage.004306AA
0040BEFA              8B40 04              mov eax,dword ptr ds:[eax+4]
0040BEFD              57                   push edi
0040BEFE              68 607C4300          push TifImage.00437C60                             ; registration_code
0040BF03              68 EC744300          push TifImage.004374EC                             ; option
0040BF08              8BC8                 mov ecx,eax
```
