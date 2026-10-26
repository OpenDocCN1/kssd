
```
00C2A6FD     6A 01            push 1

00C2A6FF     58               pop eax

00C2A700     85C0             test eax,eax

00C2A702     0F84 7B020000    je 00C2A983

00C2A708     83A5 94E7FFFF 00 and dword ptr ss:[ebp-186C],0                    ; 将上一次存放的iat清空

00C2A70F     8B85 94ECFFFF    mov eax,dword ptr ss:[ebp-136C]                  ; 取得iat函数名

00C2A715     0FBE00           movsx eax,byte ptr ds:[eax]

00C2A718     85C0             test eax,eax

00C2A71A     0F85 0C010000    jnz 00C2A82C                                     ; 判断函数名首字母是否为0

00C2A720     C785 44E7FFFF B2>mov dword ptr ss:[ebp-18BC],0C162B2

00C2A72A     C785 48E7FFFF 77>mov dword ptr ss:[ebp-18B8],0C16377

00C2A734     C785 4CE7FFFF 9A>mov dword ptr ss:[ebp-18B4],0C1639A

00C2A73E     C785 50E7FFFF D8>mov dword ptr ss:[ebp-18B0],0C162D8

00C2A748     C785 54E7FFFF 15>mov dword ptr ss:[ebp-18AC],0C16315

00C2A752     C785 58E7FFFF 1A>mov dword ptr ss:[ebp-18A8],0C1631A

00C2A75C     C785 5CE7FFFF 1F>mov dword ptr ss:[ebp-18A4],0C1631F

00C2A766     C785 60E7FFFF 24>mov dword ptr ss:[ebp-18A0],0C16324

00C2A770     C785 64E7FFFF 4A>mov dword ptr ss:[ebp-189C],0C1634A

00C2A77A     C785 68E7FFFF 70>mov dword ptr ss:[ebp-1898],0C16370

00C2A784     C785 6CE7FFFF B2>mov dword ptr ss:[ebp-1894],0C162B2

00C2A78E     C785 70E7FFFF 77>mov dword ptr ss:[ebp-1890],0C16377

00C2A798     C785 74E7FFFF 9A>mov dword ptr ss:[ebp-188C],0C1639A

00C2A7A2     C785 78E7FFFF D8>mov dword ptr ss:[ebp-1888],0C162D8

00C2A7AC     C785 7CE7FFFF AD>mov dword ptr ss:[ebp-1884],0C163AD

00C2A7B6     C785 80E7FFFF B2>mov dword ptr ss:[ebp-1880],0C163B2

00C2A7C0     C785 84E7FFFF B7>mov dword ptr ss:[ebp-187C],0C163B7

00C2A7CA     C785 88E7FFFF 24>mov dword ptr ss:[ebp-1878],0C16324

00C2A7D4     C785 8CE7FFFF 4A>mov dword ptr ss:[ebp-1874],0C1634A

00C2A7DE     C785 90E7FFFF BC>mov dword ptr ss:[ebp-1870],0C163BC

00C2A7E8     8D8D 68ECFFFF    lea ecx,dword ptr ss:[ebp-1398]

00C2A7EE     E8 4D68FEFF      call 00C11040

00C2A7F3     0FB6C0           movzx eax,al

00C2A7F6     99               cdq

00C2A7F7     6A 14            push 14

00C2A7F9     59               pop ecx

00C2A7FA     F7F9             idiv ecx

00C2A7FC     8B85 A8E9FFFF    mov eax,dword ptr ss:[ebp-1658]

00C2A802     8B8C95 44E7FFFF  mov ecx,dword ptr ss:[ebp+edx*4-18BC]

00C2A809     8908             mov dword ptr ds:[eax],ecx

00C2A80B     8B85 A8E9FFFF    mov eax,dword ptr ss:[ebp-1658]

00C2A811     83C0 04          add eax,4

00C2A814     8985 A8E9FFFF    mov dword ptr ss:[ebp-1658],eax

00C2A81A     8B85 94ECFFFF    mov eax,dword ptr ss:[ebp-136C]

00C2A820     40               inc eax

00C2A821     8985 94ECFFFF    mov dword ptr ss:[ebp-136C],eax

00C2A827     E9 57010000      jmp 00C2A983

00C2A82C     8B85 94ECFFFF    mov eax,dword ptr ss:[ebp-136C]

00C2A832     0FB600           movzx eax,byte ptr ds:[eax]

00C2A835     3D FF000000      cmp eax,0FF                                      ; 再一次判断iat首字母是否为0ff

00C2A83A     0F85 A7000000    jnz 00C2A8E7

00C2A840     8B85 94ECFFFF    mov eax,dword ptr ss:[ebp-136C]

00C2A846     40               inc eax

00C2A847     8985 94ECFFFF    mov dword ptr ss:[ebp-136C],eax

00C2A84D     8B85 94ECFFFF    mov eax,dword ptr ss:[ebp-136C]

00C2A853     66:8B00          mov ax,word ptr ds:[eax]

00C2A856     66:8985 40E7FFFF mov word ptr ss:[ebp-18C0],ax

00C2A85D     8B85 94ECFFFF    mov eax,dword ptr ss:[ebp-136C]

00C2A863     40               inc eax

00C2A864     40               inc eax

00C2A865     8985 94ECFFFF    mov dword ptr ss:[ebp-136C],eax

00C2A86B     0FB785 40E7FFFF  movzx eax,word ptr ss:[ebp-18C0]

00C2A872     50               push eax

00C2A873     FFB5 BCE9FFFF    push dword ptr ss:[ebp-1644]

00C2A879     E8 A0B1FEFF      call 00C15A1E

00C2A87E     8985 94E7FFFF    mov dword ptr ss:[ebp-186C],eax

00C2A884     83BD 94E7FFFF 00 cmp dword ptr ss:[ebp-186C],0

00C2A88B     75 58            jnz short 00C2A8E5

00C2A88D     FF15 C050C300    call dword ptr ds:[C350C0]                       ; ntdll.RtlGetLastWin32Error

00C2A893     83F8 32          cmp eax,32

00C2A896     75 0A            jnz short 00C2A8A2

00C2A898     C785 94E7FFFF CA>mov dword ptr ss:[ebp-186C],0C158CA

00C2A8A2     83BD 94E7FFFF 00 cmp dword ptr ss:[ebp-186C],0

00C2A8A9     75 3A            jnz short 00C2A8E5

00C2A8AB     8B45 08          mov eax,dword ptr ss:[ebp+8]

00C2A8AE     8B00             mov eax,dword ptr ds:[eax]

00C2A8B0     C700 03000000    mov dword ptr ds:[eax],3

00C2A8B6     FF15 C050C300    call dword ptr ds:[C350C0]                       ; ntdll.RtlGetLastWin32Error

00C2A8BC     50               push eax

00C2A8BD     0FB785 40E7FFFF  movzx eax,word ptr ss:[ebp-18C0]

00C2A8C4     50               push eax

00C2A8C5     FFB5 A0E8FFFF    push dword ptr ss:[ebp-1760]

00C2A8CB     68 B090C300      push 0C390B0                                     ; ASCII "File "%s", ordinal %d (error %d)"

00C2A8D0     8B45 08          mov eax,dword ptr ss:[ebp+8]

00C2A8D3     FF70 04          push dword ptr ds:[eax+4]

00C2A8D6     E8 D13F0000      call 00C2E8AC

00C2A8DB     83C4 14          add esp,14

00C2A8DE     33C0             xor eax,eax

00C2A8E0     E9 A00C0000      jmp 00C2B585

00C2A8E5     EB 7A            jmp short 00C2A961

00C2A8E7     8B85 94ECFFFF    mov eax,dword ptr ss:[ebp-136C]

00C2A8ED     8985 3CE7FFFF    mov dword ptr ss:[ebp-18C4],eax

00C2A8F3     6A 00            push 0

00C2A8F5     FFB5 94ECFFFF    push dword ptr ss:[ebp-136C]

00C2A8FB     E8 F03E0000      call 00C2E7F0                                    ; 取得下个函数名

00C2A900     59               pop ecx

00C2A901     59               pop ecx

00C2A902     40               inc eax

00C2A903     8985 94ECFFFF    mov dword ptr ss:[ebp-136C],eax                  ; 将下个函数名保存

00C2A909     FFB5 3CE7FFFF    push dword ptr ss:[ebp-18C4]                     ; 可以dd eax,查看iat的函数列表

00C2A90F     FFB5 BCE9FFFF    push dword ptr ss:[ebp-1644]

00C2A915     E8 04B1FEFF      call 00C15A1E                                    ; 关键函数，取得iat地址

00C2A91A     8985 94E7FFFF    mov dword ptr ss:[ebp-186C],eax

00C2A920     83BD 94E7FFFF 00 cmp dword ptr ss:[ebp-186C],0

00C2A927     75 38            jnz short 00C2A961

00C2A929     8B45 08          mov eax,dword ptr ss:[ebp+8]

00C2A92C     8B00             mov eax,dword ptr ds:[eax]

00C2A92E     C700 03000000    mov dword ptr ds:[eax],3

00C2A934     FF15 C050C300    call dword ptr ds:[C350C0]                       ; ntdll.RtlGetLastWin32Error

00C2A93A     50               push eax

00C2A93B     FFB5 3CE7FFFF    push dword ptr ss:[ebp-18C4]

00C2A941     FFB5 A0E8FFFF    push dword ptr ss:[ebp-1760]

00C2A947     68 8C90C300      push 0C3908C                                     ; ASCII "File "%s", function "%s" (error %d)"

00C2A94C     8B45 08          mov eax,dword ptr ss:[ebp+8]

00C2A94F     FF70 04          push dword ptr ds:[eax+4]

00C2A952     E8 553F0000      call 00C2E8AC

00C2A957     83C4 14          add esp,14

00C2A95A     33C0             xor eax,eax

00C2A95C     E9 240C0000      jmp 00C2B585

00C2A961     8B85 A8E9FFFF    mov eax,dword ptr ss:[ebp-1658]

00C2A967     8B8D 94E7FFFF    mov ecx,dword ptr ss:[ebp-186C]

00C2A96D     8908             mov dword ptr ds:[eax],ecx                       ; iat写入

00C2A96F     8B85 A8E9FFFF    mov eax,dword ptr ss:[ebp-1658]                  ; ====>硬件断点

00C2A975     83C0 04          add eax,4

00C2A978     8985 A8E9FFFF    mov dword ptr ss:[ebp-1658],eax

00C2A97E   ^ E9 7AFDFFFF      jmp 00C2A6FD
```

需进入关键函数，继续分析

`f7`步入后会有比较多的`je` `je` `jump`, 可以直接在`jump`下行`f4`。

一直`f8`单步向下跟，遇到`seh`断续`f8`.跟到有好几个`nop`语句的地方
