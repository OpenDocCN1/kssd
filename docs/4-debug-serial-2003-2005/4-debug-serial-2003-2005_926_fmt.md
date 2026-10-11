
# 图章制作系统 1.0

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-233.htm`](https://www.pediy.com/kssd/pediy07/pediy7-233.htm)

日期：2005 年 5 月 15 日 破解人：`yijun[PYG]`

【软件名称】：图章制作系统 软件版本：1.0
【软件大小】：252KB
【软件简介】：制作各种图章
【软件限制】：NAG
【破解声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！
【破解工具】：`OD`, `PEID`

### 破解过程

`PEID` 查知道该软件加了 `ASPack 2.12 -> Alexey Solodovnikov` 壳，`OD` 轻松搞定~~~再查知道该软件是 `Borland Delphi 6.0 - 7.0` 编写~~~~

`OD` 载入，通过字符串查找很容易来到这里：

```
0046ADAC    55             push ebp  //在此下断
0046ADAD    68 A0AE4600    push Unpacked.0046AEA0
0046ADB2    64:FF30        push dword ptr fs:[eax]
0046ADB5    64:8920        mov dword ptr fs:[eax],esp
0046ADB8    8D55 F8        lea edx,dword ptr ss:[ebp-8]
0046ADBB    8BB3 00030000 mov esi,dword ptr ds:[ebx+300]
0046ADC1    8BC6           mov eax,esi
0046ADC3    E8 1859FDFF    call Unpacked.004406E0                ; 取注册码，长度送 EAX
0046ADC8    8B45 F8        mov eax,dword ptr ss:[ebp-8]         ; 注册码送 EAX
0046ADCB    8D55 FC        lea edx,dword ptr ss:[ebp-4]
0046ADCE    E8 45D9F9FF    call Unpacked.00408718
0046ADD3    8B55 FC        mov edx,dword ptr ss:[ebp-4]         ; 注册码送 EDX
0046ADD6    8BC6           mov eax,esi
0046ADD8    E8 3359FDFF    call Unpacked.00440710
0046ADDD    8D55 F4        lea edx,dword ptr ss:[ebp-C]
0046ADE0    8B83 00030000 mov eax,dword ptr ds:[ebx+300]
0046ADE6    E8 F558FDFF    call Unpacked.004406E0
0046ADEB    837D F4 00     cmp dword ptr ss:[ebp-C],0           ; 注册码是否为空
0046ADEF    0F84 88000000 je Unpacked.0046AE7D                  ; 是就跳
0046ADF5    B9 B8AE4600    mov ecx,Unpacked.0046AEB8             ; 否则"HsjSoft.ini"送 ECX
0046ADFA    B2 01          mov dl,1                               ; DL 置 1
0046ADFC    A1 085D4600    mov eax,dword ptr ds:[465D08]
0046AE01    E8 B2AFFFFF    call Unpacked.00465DB8
0046AE06    8BF0           mov esi,eax
0046AE08    8D55 F0        lea edx,dword ptr ss:[ebp-10]
0046AE0B    8B83 00030000 mov eax,dword ptr ds:[ebx+300]
0046AE11    E8 CA58FDFF    call Unpacked.004406E0
0046AE16    8B45 F0        mov eax,dword ptr ss:[ebp-10]         ; 注册码送 EAX
0046AE19    50             push eax
0046AE1A    B9 CCAE4600    mov ecx,Unpacked.0046AECC             ; "reg_code"送 ECX
0046AE1F    8B93 10030000 mov edx,dword ptr ds:[ebx+310]
0046AE25    8BC6           mov eax,esi
0046AE27    8B38           mov edi,dword ptr ds:[eax]
0046AE29    FF57 04        call dword ptr ds:[edi+4]
0046AE2C    8BC6           mov eax,esi
0046AE2E    E8 4587F9FF    call Unpacked.00403578
0046AE33    8B83 10030000 mov eax,dword ptr ds:[ebx+310]
0046AE39    E8 B60B0000    call Unpacked.0046B9F4                ; 关键 CALL，跟进~~~~~~~~
0046AE3E    84C0           test al,al                             ; AL 是否为 0，（如果刚才比较条件为真则 AL 为 1~~~~~~~）
0046AE40    75 1B          jnz short Unpacked.0046AE5D            ; 不就跳，不跳就挂~~~~~~
0046AE42    6A 00          push 0
0046AE44    68 D8AE4600    push Unpacked.0046AED8
0046AE49    68 E0AE4600    push Unpacked.0046AEE0                ; 错误
0046AE4E    8BC3           mov eax,ebx
0046AE50    E8 A3BFFDFF    call Unpacked.00446DF8
0046AE55    50             push eax
0046AE56    E8 DDC2F9FF    call <jmp.&user32.MessageBoxA>
0046AE5B    EB 20          jmp short Unpacked.0046AE7D
0046AE5D    6A 00          push 0
0046AE5F    68 F8AE4600    push Unpacked.0046AEF8
0046AE64    68 00AF4600    push Unpacked.0046AF00                ; 成功
0046AE69    8BC3           mov eax,ebx
0046AE6B    E8 88BFFDFF    call Unpacked.00446DF8
0046AE70    50             push eax
0046AE71    E8 C2C2F9FF    call <jmp.&user32.MessageBoxA>
0046AE76    8BC3           mov eax,ebx
0046AE78    E8 7F21FFFF    call Unpacked.0045CFFC
0046AE7D    33C0           xor eax,eax
0046AE7F    5A             pop edx
0046AE80    59             pop ecx
0046AE81    59             pop ecx
0046AE82    64:8910       mov dword ptr fs:[eax],edx
0046AE85    68 A7AE4600    push Unpacked.0046AEA7
0046AE8A    8D45 F0        lea eax,dword ptr ss:[ebp-10]
0046AE8D    BA 03000000    mov edx,3
0046AE92    E8 F594F9FF    call Unpacked.0040438C
0046AE97    8D45 FC        lea eax,dword ptr ss:[ebp-4]
0046AE9A    E8 C994F9FF    call Unpacked.00404368
0046AE9F    C3             retn
0046AEA0    ^ E9 678EF9FF jmp Unpacked.00403D0C
0046AEA5    ^ EB E3        jmp short Unpacked.0046AE8A
```

跟进 `0046AE39` 处 `CALL` 来到：

```
0046B9F4    55             push ebp
0046B9F5    8BEC           mov ebp,esp
0046B9F7    33C9           xor ecx,ecx
0046B9F9    51             push ecx
0046B9FA    51             push ecx
0046B9FB    51             push ecx
0046B9FC    51             push ecx
0046B9FD    51             push ecx
0046B9FE    53             push ebx
0046B9FF    56             push esi
0046BA00    8945 FC        mov dword ptr ss:[ebp-4],eax
0046BA03    8B45 FC        mov eax,dword ptr ss:[ebp-4]
0046BA06    E8 FD8DF9FF    call Unpacked.00404808
0046BA0B    33C0           xor eax,eax
0046BA0D    55             push ebp
0046BA0E    68 91BA4600    push Unpacked.0046BA91
0046BA13    64:FF30        push dword ptr fs:[eax]
0046BA16    64:8920        mov dword ptr fs:[eax],esp
0046BA19    8D55 F4        lea edx,dword ptr ss:[ebp-C]
0046BA1C    8B45 FC        mov eax,dword ptr ss:[ebp-4]
0046BA1F    E8 DCFDFFFF    call Unpacked.0046B800
0046BA24    8D55 F8        lea edx,dword ptr ss:[ebp-8]
0046BA27    8B45 F4        mov eax,dword ptr ss:[ebp-C]          ; 机器码送 EAX
0046BA2A    E8 D9FEFFFF    call Unpacked.0046B908  //跟进
0046BA2F    B9 A8BA4600    mov ecx,Unpacked.0046BAA8             ; ASCII "HsjSoft.ini"//返回到这里
0046BA34    B2 01          mov dl,1                               ; DL 置 1
0046BA36    A1 085D4600    mov eax,dword ptr ds:[465D08]
0046BA3B    E8 78A3FFFF    call Unpacked.00465DB8
0046BA40    8BD8           mov ebx,eax
0046BA42    6A 00          push 0
0046BA44    8D45 EC        lea eax,dword ptr ss:[ebp-14]
0046BA47    50             push eax
0046BA48    B9 BCBA4600    mov ecx,Unpacked.0046BABC             ; ASCII "reg_code"
0046BA4D    8B55 FC        mov edx,dword ptr ss:[ebp-4]
0046BA50    8BC3           mov eax,ebx
0046BA52    8B30           mov esi,dword ptr ds:[eax]
0046BA54    FF16           call dword ptr ds:[esi]
0046BA56    8B45 EC        mov eax,dword ptr ss:[ebp-14]         ; 假码送 EAX
0046BA59    8D55 F0        lea edx,dword ptr ss:[ebp-10]
0046BA5C    E8 B7CCF9FF    call Unpacked.00408718
0046BA61    8BC3           mov eax,ebx
0046BA63    E8 107BF9FF    call Unpacked.00403578
0046BA68    8B45 F8        mov eax,dword ptr ss:[ebp-8]          ; 真码送 EAX
0046BA6B    8B55 F0        mov edx,dword ptr ss:[ebp-10]         ; 假码送 EDX
0046BA6E    E8 F18CF9FF    call Unpacked.00404764                ; 将假码和真码比较~~
0046BA73    0F94C3         sete bl                               ; 置 BL 值
0046BA76    33C0           xor eax,eax
0046BA78    5A             pop edx
0046BA79    59             pop ecx
0046BA7A    59             pop ecx
0046BA7B    64:8910        mov dword ptr fs:[eax],edx
0046BA7E    68 98BA4600    push Unpacked.0046BA98
0046BA83    8D45 EC        lea eax,dword ptr ss:[ebp-14]
0046BA86    BA 05000000    mov edx,5
```



```
0046BA8B    E8 FC88F9FF     call Unpacked.0040438C  
0046BA90    C3             retn  
```

****************************************************************

跟进 `0046BA2A` 处 `CALL` 来到：

```
0046B908    55             push ebp  
0046B909    8BEC           mov ebp,esp  
0046B90B    83C4 DC        add esp,-24  
0046B90E    53             push ebx  
0046B90F    56             push esi  
0046B910    33C9           xor ecx,ecx  
0046B912    894D DC        mov dword ptr ss:[ebp-24],ecx  
0046B915    894D E0        mov dword ptr ss:[ebp-20],ecx  
0046B918    894D F8        mov dword ptr ss:[ebp-8],ecx  
0046B91B    894D F4        mov dword ptr ss:[ebp-C],ecx  
0046B91E    8BF2           mov esi,edx  
0046B920    8945 FC        mov dword ptr ss:[ebp-4],eax  
0046B923    8B45 FC        mov eax,dword ptr ss:[ebp-4]  
0046B926    E8 DD8EF9FF    call Unpacked.00404808  
0046B92B    33C0           xor eax,eax  
0046B92D    55             push ebp  
0046B92E    68 CEB94600    push Unpacked.0046B9CE  
0046B933    64:FF30        push dword ptr fs:[eax]  
0046B936    64:8920        mov dword ptr fs:[eax],esp  
0046B939    8BC6           mov eax,esi  
0046B93B    E8 288AF9FF    call Unpacked.00404368  
0046B940    8D45 E0        lea eax,dword ptr ss:[ebp-20]  
0046B943    8B4D FC        mov ecx,dword ptr ss:[ebp-4]      ; 机器码送 ECX  
0046B946    BA E4B94600    mov edx,Unpacked.0046B9E4  
0046B94B    E8 1C8DF9FF    call Unpacked.0040466C  
0046B950    8B45 E0        mov eax,dword ptr ss:[ebp-20]  
0046B953    8D55 E4        lea edx,dword ptr ss:[ebp-1C]  
0046B956    E8 EDBCAFFFF   call Unpacked.00467648  
0046B95B    8D45 E4        lea eax,dword ptr ss:[ebp-1C]  
0046B95E    8D55 F8        lea edx,dword ptr ss:[ebp-8]  
0046B961    E8 56BDFFFF    call Unpacked.004676BC            ; 跟进~~~~~~~~，最终得一字符串 191bbf78cd24da75f4da7d784915708e  
0046B966    8D45 F4        lea eax,dword ptr ss:[ebp-C]  //返回到这里~~~~~  
0046B969    E8 FA89F9FF    call Unpacked.00404368  
0046B96E    8B45 F8        mov eax,dword ptr ss:[ebp-8]      ; 191bbf78cd24da75f4da7d784915708e 送 EAX  
0046B971    E8 AA8CF9FF    call Unpacked.00404620            ; 测试 191bbf78cd24da75f4da7d784915708e 是否为空，如果不是则长度送 EAX  
0046B976    8BD8           mov ebx,eax                       ; EAX 送 EBX  
0046B978    83FB 01        cmp ebx,1                         ; EBX 和 1 比较  
0046B97B    7C 1F          jl short Unpacked.0046B99C         ; 小于就跳  
0046B97D    8D45 DC        lea eax,dword ptr ss:[ebp-24]     ; 否则，依次计算  
0046B980    8B55 F8        mov edx,dword ptr ss:[ebp-8]      ; 191bbf78cd24da75f4da7d784915708e 送 EDX  
0046B983    8A541A FF     mov dl,byte ptr ds:[edx+ebx-1]    ; [edx+ebx-1]送 DL  
0046B987    E8 BC8BF9FF   call Unpacked.00404548  
0046B98C    8B55 DC        mov edx,dword ptr ss:[ebp-24]  
0046B98F    8D45 F4        lea eax,dword ptr ss:[ebp-C]      ; [ebp-C]送 EAX  
0046B992    E8 918CF9FF    call Unpacked.00404628  
0046B997    4B             dec ebx                             ; EBX 减一  
0046B998    85DB           test ebx,ebx                        ; 是否为 0  
0046B99A    75 E1          jnz short Unpacked.0046B97D         ; 不为 0 就继续，以上这段循环就是将刚才得到的字符串倒序，得：e807519487d7ad4f57ad42dc87fbb191（真码）。  
0046B99C    8BC6           mov eax,esi  
0046B99E    8B55 F4        mov edx,dword ptr ss:[ebp-C]      ; e807519487d7ad4f57ad42dc87fbb191（真码）送 EDX  
0046B9A1    E8 168AF9FF    call Unpacked.004043BC  
0046B9A6    33C0           xor eax,eax  
0046B9A8    5A             pop edx  
0046B9A9    59             pop ecx  
0046B9AA    59             pop ecx  
0046B9AB    64:8910        mov dword ptr fs:[eax],edx  
0046B9AE    68 D5B94600    push Unpacked.0046B9D5  
0046B9B3    8D45 DC        lea eax,dword ptr ss:[ebp-24]  
0046B9B6    BA 02000000    mov edx,2  
0046B9BB    E8 CC89F9FF    call Unpacked.0040438C  
0046B9C0    8D45 F4        lea eax,dword ptr ss:[ebp-C]  
0046B9C3    BA 03000000    mov edx,3  
0046B9C8    E8 BF89F9FF    call Unpacked.0040438C  
0046B9CD    C3             retn  
```

****************************************************************

跟进 `0046B961` 处 `CALL` 来到：

```
004676BC    55             push ebp  
004676BD    8BEC           mov ebp,esp  
004676BF    83C4 E8        add esp,-18  
004676C2    53             push ebx  
004676C3    56             push esi  
004676C4    57             push edi  
004676C5    33C9           xor ecx,ecx  
004676C7    894D EC        mov dword ptr ss:[ebp-14],ecx  
004676CA    894D E8        mov dword ptr ss:[ebp-18],ecx  
004676CD    8BF0           mov esi,eax  
004676CF    8D7D F0        lea edi,dword ptr ss:[ebp-10]  
004676D2    A5             movs dword ptr es:[edi],dword ptr ds:[esi>  
004676D3    A5             movs dword ptr es:[edi],dword ptr ds:[esi>  
004676D4    A5             movs dword ptr es:[edi],dword ptr ds:[esi>  
004676D5    A5             movs dword ptr es:[edi],dword ptr ds:[esi>  
004676D6    8BFA           mov edi,edx  
004676D8    33C0           xor eax,eax  
004676DA    55             push ebp  
004676DB    68 57774600    push Unpacked.00467757  
004676E0    64:FF30        push dword ptr fs:[eax]  
004676E3    64:8920        mov dword ptr fs:[eax],esp  
004676E6    8BC7           mov eax,edi  
004676E8    E8 7BCCF9FF    call Unpacked.00404368  
004676ED    B3 10          mov bl,10  
004676EF    8D75 F0        lea esi,dword ptr ss:[ebp-10]  
004676F2    FF37           push dword ptr ds:[edi]             ; *****循环*****  
004676F4    8D45 EC        lea eax,dword ptr ss:[ebp-14]  
004676F7    33D2           xor edx,edx  
004676F9    8A16           mov dl,byte ptr ds:[esi]            ; [esi]送 DL  
004676FB    C1EA 04        shr edx,4                           ; EDX 右移 4 位  
004676FE    83E2 0F        and edx,0F                          ; 和 0F 与  
00467701    8A92 E4FF4700  mov dl,byte ptr ds:[edx+47FFE4]     ; [edx+47FFE4]送 DL  
00467707    E8 3CCEF9FF    call Unpacked.00404548  
0046770C    FF75 EC        push dword ptr ss:[ebp-14]  
0046770F    8D45 E8        lea eax,dword ptr ss:[ebp-18]  
00467712    8A16           mov dl,byte ptr ds:[esi]            ; [esi]送 DL  
00467714    80E2 0F        and dl,0F                           ; DL 和 0F 与  
00467717    81E2 FF000000  and edx,0FF                         ; EDX 和 0FF 与  
0046771D    8A92 E4FF4700  mov dl,byte ptr ds:[edx+47FFE4]     ; [edx+47FFE4]送 DL  
00467723    E8 20CEF9FF    call Unpacked.00404548  
00467728    FF75 E8        push dword ptr ss:[ebp-18]  
0046772B    8BC7           mov eax,edi                         ; EDI 送 EAX  
0046772D    BA 03000000    mov edx,3  
00467732    E8 A9CFF9FF    call Unpacked.004046E0  
00467737    46             inc esi  
00467738    FECB           dec bl  
0046773A    75 B6          jnz short Unpacked.004676F2         ; BL 不为 0 就继续，循环完后得字符串 191bbf78cd24da75f4da7d784915708e  
0046773C    33C0           xor eax,eax  
0046773E    5A             pop edx  
0046773F    59             pop ecx  
00467740    59             pop ecx  
00467741    64:8910        mov dword ptr fs:[eax],edx  
00467744    68 5E774600    push Unpacked.0046775E  
00467749    8D45 E8        lea eax,dword ptr ss:[ebp-18]  
0046774C    BA 02000000    mov edx,2  
00467751    E8 36CCF9FF    call Unpacked.0040438C  
00467756    C3             retn  //返回  
```

———————————————————————————————————————————

**Crack 总结**：

将机器码通过一系列运算得一字符串，再将此字符串倒序即为注册码^-^



机器码：`bbba33de56c4cf31d9063cc074248d4b`

注册码：`e807519487d7ad4f57ad42dc87fbb191`

`4dd466d0e1e0cc123afbe504da2462ca`
