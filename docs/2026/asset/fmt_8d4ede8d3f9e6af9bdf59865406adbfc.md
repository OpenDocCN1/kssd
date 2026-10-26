
`0042411A`     `8F07`                     `pop dword ptr [edi]`                           ; 恢复`00401000`处4字节代码

`0042411C`     `50`                       `push eax`

`0042411D`     `51`                       `push ecx`

`0042411E`     `56`                       `push esi`

`0042411F`     `53`                       `push ebx`

`00424120`     `8BC8`                     `mov ecx,eax`

`00424122`     `83E9 06`                  `sub ecx,6`                                     ; `ecx=00020000-6＝0001FFFA`

`00424125`     `8BB5 52010000`            `mov esi,dword ptr [ebp+152]`                   ; 缓冲区

`0042412B`     `33DB`                     `xor ebx,ebx`                                   ; 初始化计数器

下面开始修改解压后的代码，将`E804`和`E904`开始的`call`和`jmp`目标地址解密：

```
0042412D /  /0BC9                     or ecx,ecx                                    ; 查找范围字节数。初始＝0001FFFA
0042412F |  |74 2E                    je short KillBox.0042415F                     ; 为0则跳
00424131 |  |78 2C                    js short KillBox.0042415F                     ; 为负数（即超出了搜索范围）则跳
00424133 |  |AC                       lods byte ptr [esi]                           ; 从缓冲区查找特征字符（E8、E9）
00424134 |  |3C E8                    cmp al,0E8                                    ; 是否为call指令
00424136 |  |74 0A                    je short KillBox.00424142                     ; 为call指令则去解密
00424138 |/ |EB 00                    jmp short KillBox.0042413A
0042413A || |3C E9                    cmp al,0E9                                    ; 是否为jmp指令
0042413C ||/|74 04                    je short KillBox.00424142                     ; 为jmp指令则则去解密
0042413E ||||43                       inc ebx                                       ; 计数器加1
0042413F ||||49                       dec ecx                                       ; 查找范围字节数－1
00424140 |||\EB EB                    jmp short KillBox.0042412D
00424142 |\\ 8B06                     mov eax,dword ptr [esi]                       ; E8或E9后面再取一字符，即jmp XXXXXXXX指令的目标地址最低位字节。
00424144 |   EB 00                    jmp short KillBox.00424146
00424146 |   803E 04                  cmp byte ptr [esi],4                          ; 判断是否“E904”或“E804”（00D602EC处才有E904，00D60DE5处才有E804）
00424149 | ^ 75 F3                    jnz short KillBox.0042413E                    ; 不是继续循环搜索
0042414B |   24 00                    and al,0                                      ; 对jmp和call的目标地址进行解密
0042414D |   C1C0 18                  rol eax,18
00424150 |   2BC3                     sub eax,ebx
00424152 |   8906                     mov dword ptr [esi],eax                       ; 修改jmp和call目标地址
00424154 |   83C3 05                  add ebx,5                                     ; 计数器加5（jmp指令共5字节）
00424157 |   83C6 04                  add esi,4                                     ; 指针后移4字节，即指向jmp指令后面的一条指令处
0042415A |   83E9 05                  sub ecx,5                                     ; 查找范围字节数－5
0042415D \ ^ EB CE                    jmp short KillBox.0042412D
```

修改完毕后跳到这里：

```
0042415F     5B                       pop ebx
00424160     5E                       pop esi
00424161     59                       pop ecx
00424162     58                       pop eax
00424163     EB 08                    jmp short KillBox.0042416D                    ; 解压其他区段时是直接跳到这里
```

跳到了这里，将缓冲区的代码（数据）复制到目标区段：

```
0042416D     8BC8                     mov ecx,eax                                   ; eax=解压数据大小
0042416F     8B3E                     mov edi,dword ptr [esi]                       ; 本区段待解压数据的起始地址RVA
00424171     03BD 22040000            add edi,dword ptr [ebp+422]                   ; RVA+ImageBase=目标地址
00424177     8BB5 52010000            mov esi,dword ptr [ebp+152]                   ; 缓冲区
0042417D     C1F9 02                  sar ecx,2
00424180     F3:A5                    rep movs dword ptr es:[edi],dword ptr [esi]   ; 复制
00424182     8BC8                     mov ecx,eax
00424184     83E1 03                  and ecx,3
00424187     F3:A4                    rep movs byte ptr es:[edi],byte ptr [esi]
00424189     5E                       pop esi
0042418A     68 00800000              push 8000
0042418F     6A 00                    push 0
00424191     FFB5 52010000            push dword ptr [ebp+152]                      ; 缓冲区
00424197     FF95 51050000            call dword ptr [ebp+551]                      ; VirtualFree释放缓冲区
0042419D     83C6 08                  add esi,8                                     ; 信息表中指向下一个待解压的区段
004241A0     833E 00                  cmp dword ptr [esi],0                         ; 是否还有待解压的段？
004241A3   ^ 0F85 1EFFFFFF            jnz KillBox.004240C7                          ; 有则循环继续解压
```

至此解压全部完成，接下来清理内存：

```
004241A9     68 00800000              push 8000
004241AE     6A 00                    push 0
004241B0     FFB5 56010000            push dword ptr [ebp+156]                      ; 申请的第一块内存地址
004241B6     FF95 51050000            call dword ptr [ebp+551]                      ; VirtualFree释放第一块内存
004241BC     8B9D 31050000            mov ebx,dword ptr [ebp+531]                   ; ss:[00424544]=00000000
004241C2     0BDB                     or ebx,ebx
004241C4     74 08                    je short KillBox.004241CE                     ; 跳
004241C6     8B03                     mov eax,dword ptr [ebx]
004241C8     8785 35050000            xchg dword ptr [ebp+535],eax
004241CE     8B95 22040000            mov edx,dword ptr [ebp+422]                   ; ImageBase
004241D4     8B85 2D050000            mov eax,dword ptr [ebp+52D]                   ; ImageBase
004241DA     2BD0                     sub edx,eax
004241DC     74 79                    je short KillBox.00424257                     ; 相等，跳
```

至此，程序解压完毕，跳到了这里，开始填充IAT。

```
00424257     8B95 22040000            mov edx,dword ptr [ebp+422]                   ; ImageBase
0042425D     8BB5 41050000            mov esi,dword ptr [ebp+541]                   ; ss:[00424554]=00000000
00424263     0BF6                     or esi,esi
00424265    /74 11                    je short KillBox.00424278                     ; 跳
00424267    |03F2                     add esi,edx
00424269    |AD                       lods dword ptr [esi]
0042426A    |0BC0                     or eax,eax
0042426C    |74 0A                    je short KillBox.00424278
0042426E    |03C2                     add eax,edx
00424270    |8BF8                     mov edi,eax
00424272    |66:AD                    lods word ptr [esi]
00424274    |66:AB                    stos word ptr es:[edi]
00424276   ^|EB F1                    jmp short KillBox.00424269
00424278    \BE B4F80100              mov esi,1F8B4                                 ; 跳到这里
0042427D     8B95 22040000            mov edx,dword ptr [ebp+422]                   ; ImageBase
00424283     03F2                     add esi,edx                                   ; 1F8B4+ImageBase定位到输入表
00424285     8B46 0C                  mov eax,dword ptr [esi+C]                     ; 取IID的NAME成员
00424288     85C0                     test eax,eax                                  ; NAME成员是否为0
0042428A     0F84 0A010000            je KillBox.0042439A                           ; 是，则跳，即IAT填充完毕就跳出循环。
00424290     03C2                     add eax,edx                                   ; 否，则定位dll，1FB94+ImageBase
00424292     8BD8                     mov ebx,eax                                   ; ASCII "MSVBVM60.DLL"（VB程序）
00424294     50                       push eax
00424295     FF95 4D0F0000            call dword ptr [ebp+F4D]                      ; GetModuleHandleA获取dll句柄
0042429B     85C0                     test eax,eax
0042429D     75 07                    jnz short KillBox.004242A6                    ; 成功则跳
0042429F     53                       push ebx                                      ; 否则LoadLibraryA动态调入dll库
004242A0     FF95 510F0000            call dword ptr [ebp+F51]
004242A6     8985 45050000            mov dword ptr [ebp+545],eax                   ; 保存dll句柄
```
