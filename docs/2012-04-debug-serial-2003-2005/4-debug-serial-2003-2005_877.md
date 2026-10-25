
# 简易日记本 1.9 注册码算法初步分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-336.htm`](https://www.pediy.com/kssd/pediy07/pediy7-336.htm)

简易日记本 1.9 注册码算法初步分析

日期：2005 年 7 月 19 日　　破解人：Baby2008

---

『软件名称』：简易日记本 1.9

『软件大小』：2.20MB

『下载地址』：`http://try.nease.net/cdiary.html`

『软件介绍』：简易日记本是一款用于在电脑上写日记和收集资料的软件，支持常用的文字编辑排版功能，和 `Word2000` 操作类似，简单易用。

『保护方式』：注册码保护,试用时间限制

『破解声明』：初学 `Crack`，只是感兴趣,失误之处敬请诸位大侠赐教!

『破解工具』：`OllyDbg.V1.10` 聆风听雨汉化第二版、`PeID 0.93`

『破解过程』：

不知是啥壳，直接用 `PEID` 插件直接给脱了，查找关键字符串“REGSN”来到:

---

```
0063CAB2     6A 00              push 0
0063CAB4     8D45 E0           lea eax,dword ptr ss:[ebp-20]
0063CAB7     50                push eax
0063CAB8     B9 B4D66300       mov ecx,bdiary_e.0063D6B4                ; ASCII "REGSN"
0063CABD     BA 6CD66300       mov edx,bdiary_e.0063D66C                ; ASCII "bdiary"
0063CAC2     8BC3              mov eax,ebx
0063CAC4     8B30              mov esi,dword ptr ds:[eax]
0063CAC6     FF16              call dword ptr ds:[esi]
0063CAC8     837D E0 00        cmp dword ptr ss:[ebp-20],0              ; 试炼码
0063CACC     75 0D             jnz short bdiary_e.0063CADB
0063CACE     8D45 E0           lea eax,dword ptr ss:[ebp-20]
0063CAD1     BA C4D66300       mov edx,bdiary_e.0063D6C4                ; ASCII "d343a609a3140"
0063CAD6 <>  E8 E181DCFF       call bdiary_e.00404CBC                   ; ->System.@LStrLAsg(void;void;void;void);
0063CADB     8D85 50FFFFFF     lea eax,dword ptr ss:[ebp-B0]
0063CAE1     50                push eax
0063CAE2     8B45 E0           mov eax,dword ptr ss:[ebp-20]            ; 试炼码
0063CAE5 <>  E8 F283DCFF       call bdiary_e.00404EDC                   ; ->System.@LStrLen(String):Integer;<+>
0063CAEA     8BC8              mov ecx,eax
0063CAEC     83E9 02           sub ecx,2                                ; Length(试炼码)-2
0063CAEF     BA 01000000       mov edx,1                                ; 1
0063CAF4     8B45 E0           mov eax,dword ptr ss:[ebp-20]
0063CAF7 <>  E8 3886DCFF       call bdiary_e.00405134                   ; ->System.@LStrCopy;
0063CAFC     8B85 50FFFFFF     mov eax,dword ptr ss:[ebp-B0]            ; 试炼码(长度-2)字符串
0063CB02     8D55 D8           lea edx,dword ptr ss:[ebp-28]
0063CB05     E8 B2D6FEFF       call bdiary_e.0062A1BC                   ; 试炼码处理,重要!
0063CB0A     6A 00             push 0
0063CB0C     8D85 4CFFFFFF     lea eax,dword ptr ss:[ebp-B4]
0063CB12     50                push eax
0063CB13     B9 DCD66300       mov ecx,bdiary_e.0063D6DC                ; ASCII "REGNAME"
0063CB18     BA 6CD66300       mov edx,bdiary_e.0063D66C                ; ASCII "bdiary"
0063CB1D     8BC3              mov eax,ebx
0063CB1F     8B30              mov esi,dword ptr ds:[eax]
0063CB21     FF16              call dword ptr ds:[esi]
0063CB23     8B95 4CFFFFFF     mov edx,dword ptr ss:[ebp-B4]            ; 注册名
0063CB29     A1 28C96800       mov eax,dword ptr ds:[68C928]
……省略 N 行界面处理代码…………
0063CE4E <>  8B80 04030000     mov eax,dword ptr ds:[eax+304]           ; *P_status:N.A.
0063CE54 <>  E8 3389E1FF       call bdiary_e.0045578C                   ; ->Controls.TControl.SetVisible(TControl;Boolean);
0063CE59     8D45 D0           lea eax,dword ptr ss:[ebp-30]            ; 保存试炼码第 1 位 30 位置重要
0063CE5C     50                push eax
0063CE5D     B9 01000000       mov ecx,1
0063CE62     BA 01000000       mov edx,1
0063CE67     8B45 E0           mov eax,dword ptr ss:[ebp-20]            ; 试炼码
0063CE6A <>  E8 C582DCFF       call bdiary_e.00405134                   ; ->System.@LStrCopy;
0063CE6F     8D45 CC           lea eax,dword ptr ss:[ebp-34]            ; 重要
0063CE72     50                push eax
0063CE73     8B45 E0           mov eax,dword ptr ss:[ebp-20]            ; 试炼码
0063CE76 <>  E8 6180DCFF       call bdiary_e.00404EDC                   ; ->System.@LStrLen(String):Integer;<+>
0063CE7B     8BD0              mov edx,eax
0063CE7D     4A                dec edx
0063CE7E     B9 01000000       mov ecx,1
0063CE83     8B45 E0           mov eax,dword ptr ss:[ebp-20]            ; 试炼码第长度-1 位,重要
0063CE86 <>  E8 A982DCFF       call bdiary_e.00405134                   ; ->System.@LStrCopy;
0063CE8B     8BC3              mov eax,ebx
0063CE8D <>  E8 A26EDCFF       call bdiary_e.00403D34                   ; ->System.TObject.Free(TObject);
……省略 N 行界面处理代码(皮肤处理)…………
0063D1B1     64:FF30           push dword ptr fs:[eax]
0063D1B4     64:8920           mov dword ptr fs:[eax],esp
0063D1B7     837D C8 00        cmp dword ptr ss:[ebp-38],0              ; 下面取磁盘空间做机器码
0063D1BB     75 17             jnz short bdiary_e.0063D1D4
0063D1BD     6A 00             push 0
0063D1BF     68 E8030000       push 3E8
0063D1C4     B0 03             mov al,3
0063D1C6 <>  E8 E1D0DCFF       call bdiary_e.0040A2AC                   ; ->SysUtils.DiskSize(Byte):Int64;
0063D1CB <>  E8 388CDCFF       call bdiary_e.00405E08                   ; ->System.@_lldiv;
0063D1D0     8BD8              mov ebx,eax
0063D1D2     EB 1F             jmp short bdiary_e.0063D1F3
0063D1D4     8B45 C8           mov eax,dword ptr ss:[ebp-38]
0063D1D7 <>  E8 C4CADCFF       call bdiary_e.00409CA0                   ; ->SysUtils.StrToInt(AnsiString):Integer;
0063D1DC     8BD8              mov ebx,eax
0063D1DE     6A 00             push 0
0063D1E0     68 E8030000       push 3E8
0063D1E5     B0 03             mov al,3
0063D1E7 <>  E8 C0D0DCFF       call bdiary_e.0040A2AC                   ; ->SysUtils.DiskSize(Byte):Int64;
0063D1EC <>  E8 178CDCFF       call bdiary_e.00405E08                   ; ->System.@_lldiv;
0063D1F1     03D8              add ebx,eax
0063D1F3     8BC3              mov eax,ebx
0063D1F5     33D2              xor edx,edx
0063D1F7     52                push edx
0063D1F8     50                push eax
0063D1F9     8D85 44FFFFFF     lea eax,dword ptr ss:[ebp-BC]
0063D1FF <>  E8 2CCADCFF       call bdiary_e.00409C30                   ; ->SysUtils.IntToStr(Int64):AnsiString;overload;
0063D204     8B95 44FFFFFF     mov edx,dword ptr ss:[ebp-BC]            ; 机器码
0063D20A     A1 E4C86800       mov eax,dword ptr ds:[68C8E4]
0063D20F <>  E8 647ADCFF       call bdiary_e.00404C78                   ; ->System.@LStrAsg(void;void;void;void);
0063D214     8D95 40FFFFFF     lea edx,dword ptr ss:[ebp-C0]
0063D21A     8BC3              mov eax,ebx                                ; EBX 值为机器码的 16 进制数
0063D21C     E8 87D4FEFF       call bdiary_e.0062A6A8                   ; 机器码处理,重要!
0063D221     8B85 40FFFFFF     mov eax,dword ptr ss:[ebp-C0]            ; (字符插入结果)
0063D227     8D55 DC           lea edx,dword ptr ss:[ebp-24]
0063D22A     E8 D5D5FEFF       call bdiary_e.0062A804                   ; 结果再次处理,重要!
0063D22F     8D55 D4           lea edx,dword ptr ss:[ebp-2C]            ; 首次处理结果，没啥用处了
0063D232     8B45 DC           mov eax,dword ptr ss:[ebp-24]            ; 再次处理结果(最终结果)
0063D235     E8 82CFFEFF       call bdiary_e.0062A1BC                   ; 同上试炼码编码过程
0063D23A     33C0              xor eax,eax
0063D23C     5A                pop edx
0063D23D     59                pop ecx
0063D23E     59                pop ecx
0063D23F     64:8910           mov dword ptr fs:[eax],edx
0063D242     EB 0A             jmp short bdiary_e.0063D24E
```



```
0063D244  <>^  E9 CB6FDCFF        jmp bdiary_e.00404214                 ; ->System.@HandleAnyException;  
0063D249  <>   E8 F273DCFF        call bdiary_e.00404640                ; ->System.@DoneExcept;  
……省略 N 行界面处理代码…………  
0063D4FE    68 3FD56300        push <bdiary_e.->System.@HandleAnyExcept>  
0063D503    64:FF30            push dword ptr fs:[eax]  
0063D506    64:8920            mov dword ptr fs:[eax],esp  
0063D509    8B45 D8            mov eax,dword ptr ss:[ebp-28]            ; 试炼码长度-2 计算结果  
0063D50C    8B55 D4            mov edx,dword ptr ss:[ebp-2C]            ; 机器码计算结果  
0063D50F  <>   E8 0C7BDCFF      call bdiary_e.00405020                  ; ->System.@LStrCmp;  
0063D514    75 17              jnz short bdiary_e.0063D52D             ; 爆破 1  
0063D516    8B45 D0            mov eax,dword ptr ss:[ebp-30]            ; 试炼码第 1 位  
0063D519    8B55 CC            mov edx,dword ptr ss:[ebp-34]            ; 试炼码第 5 位  
0063D51C  <>   E8 FF7ADCFF     call bdiary_e.00405020                  ; ->System.@LStrCmp;  
0063D521    75 0A              jnz short bdiary_e.0063D52D             ; 爆破 2  
0063D523    A1 B8C16800        mov eax,dword ptr ds:[68C1B8]  
0063D528    C600 01            mov byte ptr ds:[eax],1                 ; 置注册成功标志  
0063D52B    EB 08              jmp short bdiary_e.0063D535  
```

在 `0063CB05 E8 B2D6FEFF call bdiary_e.0062A1BC` 处跟进:

```
0062A1BC    55                 push ebp  
0062A1BD    8BEC               mov ebp,esp  
0062A1BF    33C9               xor ecx,ecx  
0062A1C1    51                 push ecx  
0062A1C2    51                 push ecx  
0062A1C3    51                 push ecx  
0062A1C4    51                 push ecx  
0062A1C5    51                 push ecx  
0062A1C6    53                 push ebx  
0062A1C7    56                 push esi  
0062A1C8    8BF2               mov esi,edx  
0062A1CA    8945 FC            mov dword ptr ss:[ebp-4],eax  
0062A1CD    8B45 FC            mov eax,dword ptr ss:[ebp-4]  
0062A1D0    E8 EFAEDDFF        call bdiary_e.004050C4  
0062A1D5    33C0               xor eax,eax  
0062A1D7    55                 push ebp  
0062A1D8    68 75A26200        push bdiary_e.0062A275  
0062A1DD    64:FF30            push dword ptr fs:[eax]  
0062A1E0    64:8920            mov dword ptr fs:[eax],esp  
0062A1E3    BB 01000000        mov ebx,1                       ; EBX=1  
0062A1E8    8D45 F8            lea eax,dword ptr ss:[ebp-8]      ; 常数字符串 1  
0062A1EB    BA 8CA26200        mov edx,bdiary_e.0062A28C        ; ASCII  
"YqhNrd8K1JOsbfZVUatRgB3McDiExTSu5GjvPekL2w7AXWzlIyHmQn94Co6Fp0#S_"  
0062A1F0    E8 C7AADDFF        call bdiary_e.00404CBC  
0062A1F5    8D45 F4            lea eax,dword ptr ss:[ebp-C]      ; 常数字符串 2  
0062A1F8    BA D8A26200        mov edx,bdiary_e.0062A2D8        ; ASCII  
"_$#abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"  
0062A1FD    E8 BAAADDFF        call bdiary_e.00404CBC  
0062A202    8D45 F0            lea eax,dword ptr ss:[ebp-10]     ; 保存常数字符串 1 的首字符  
0062A205    50                 push eax  
0062A206    B9 01000000        mov ecx,1                       ; 1  
0062A20B    8BD3               mov edx,ebx                     ; 1  
0062A20D    8B45 F8            mov eax,dword ptr ss:[ebp-8]      ; 常数字符串 1'YqhNr……'  
0062A210    E8 1FAFDDFF        call bdiary_e.00405134          ; System.@LStrCopy;  
0062A215    8D45 F8            lea eax,dword ptr ss:[ebp-8]  
0062A218    8B55 F8            mov edx,dword ptr ss:[ebp-8]  
0062A21B    E8 C4ACDDFF        call bdiary_e.00404EE4          ; System.@LStrCat;  
0062A220    8D45 F8            lea eax,dword ptr ss:[ebp-8]      ; 常数字符串 1 自相加  
0062A223    50                 push eax  
0062A224    B9 41000000        mov ecx,41                      ; 41  
0062A229    8BD3               mov edx,ebx                     ; 1  
0062A22B    8B45 F8            mov eax,dword ptr ss:[ebp-8]  
0062A22E    E8 01AFDDFF        call bdiary_e.00405134          ; System.@LStrCopy;  
0062A233    8D45 EC            lea eax,dword ptr ss:[ebp-14]  
0062A236    50                 push eax  
0062A237    8B4D F4            mov ecx,dword ptr ss:[ebp-C]      ; 常数字符串 2  
0062A23A    8B55 F8            mov edx,dword ptr ss:[ebp-8]      ; 常数字符串 1  
0062A23D    8B45 FC            mov eax,dword ptr ss:[ebp-4]      ; 待编码字符串  
0062A240    E8 53020000        call bdiary_e.0062A498          ; 重要函数，需要分析(字符串编码?)  
0062A245    8B55 EC            mov edx,dword ptr ss:[ebp-14]    ; 编码后字符串  
0062A248    8D45 F0            lea eax,dword ptr ss:[ebp-10]    ; 常数字符串 1 的首字符  
0062A24B    E8 94ACDDFF        call bdiary_e.00404EE4          ; System.@LStrCat;  
0062A250    8BC6               mov eax,esi  
0062A252    8B55 F0            mov edx,dword ptr ss:[ebp-10]    ; 连接结果  
0062A255    E8 1EAADDFF        call bdiary_e.00404C78  
0062A25A    33C0               xor eax,eax  
0062A25C    5A                 pop edx  
0062A25D    59                 pop ecx  
0062A25E    59                 pop ecx  
0062A25F    64:8910            mov dword ptr fs:[eax],edx  
0062A262    68 7CA26200        push bdiary_e.0062A27C  
0062A267    8D45 EC            lea eax,dword ptr ss:[ebp-14]  
0062A26A    BA 05000000        mov edx,5  
0062A26F    E8 D4A9DDFF        call bdiary_e.00404C48  
0062A274    C3                 retn  
0062A275   ^ E9 4EA2DDFF       jmp bdiary_e.004044C8  
0062A27A   ^ EB EB            jmp short bdiary_e.0062A267  
0062A27C    5E                 pop esi  
0062A27D    5B                 pop ebx  
0062A27E    8BE5               mov esp,ebp  
0062A280    5D                 pop ebp  
0062A281    C3                 retn  
```

在 `0062A240 E8 53020000 call bdiary_e.0062A498 ; 重要函数，需要分析(字符串编码?)` 处继续跟进：

```
0062A498    55                 push ebp  
0062A499    8BEC               mov ebp,esp  
0062A49B    83C4 F0            add esp,-10  
0062A49E    53                 push ebx  
0062A49F    56                 push esi  
0062A4A0    57                 push edi  
0062A4A1    33DB               xor ebx,ebx  
0062A4A3    895D F0            mov dword ptr ss:[ebp-10],ebx  
0062A4A6    894D F4            mov dword ptr ss:[ebp-C],ecx      ; 常数字符串 2  
0062A4A9    8955 F8            mov dword ptr ss:[ebp-8],edx      ; 常数字符串 1  
0062A4AC    8945 FC            mov dword ptr ss:[ebp-4],eax      ; 待处理字符串,Str  
0062A4AF    8B45 FC            mov eax,dword ptr ss:[ebp-4]  
0062A4B2    E8 0DACDDFF        call bdiary_e.004050C4  
0062A4B7    8B45 F8            mov eax,dword ptr ss:[ebp-8]  
0062A4BA    E8 05ACDDFF        call bdiary_e.004050C4  
0062A4BF    8B45 F4            mov eax,dword ptr ss:[ebp-C]  
0062A4C2    E8 FDABDDFF        call bdiary_e.004050C4  
0062A4C7    33C0               xor eax,eax  
0062A4C9    55                 push ebp  
0062A4CA    68 49A56200        push bdiary_e.0062A549  
0062A4CF    64:FF30            push dword ptr fs:[eax]  
```



```
0062A4D2     64:8920            mov dword ptr fs:[eax],esp

0062A4D5     8B45 08            mov eax,dword ptr ss:[ebp+8]

0062A4D8     8B55 FC            mov edx,dword ptr ss:[ebp-4]             ; 待处理字符串

0062A4DB     E8 98A7DDFF        call bdiary_e.00404C78

0062A4E0     8B45 08            mov eax,dword ptr ss:[ebp+8]

0062A4E3     8B00               mov eax,dword ptr ds:[eax]               ; 同上

0062A4E5     E8 F2A9DDFF        call bdiary_e.00404EDC                   ; Length

0062A4EA     8BF8               mov edi,eax

0062A4EC     85FF               test edi,edi

0062A4EE     7E 3E              jle short bdiary_e.0062A52E             ; 长度不能<=0

0062A4F0     BB 01000000        mov ebx,1                                ; i:=1

0062A4F5     8D45 F0            lea eax,dword ptr ss:[ebp-10]

0062A4F8     8B55 08            mov edx,dword ptr ss:[ebp+8]

0062A4FB     8B12               mov edx,dword ptr ds:[edx]               ; Str

0062A4FD     8A541A FF          mov dl,byte ptr ds:[edx+ebx-1]           ; Str[i]

0062A501     E8 FEA8DDFF        call bdiary_e.00404E04                   ; Char

0062A506     8B45 F0            mov eax,dword ptr ss:[ebp-10]

0062A509     8B55 F8            mov edx,dword ptr ss:[ebp-8]             ; 常数字符串 1 长度$41

0062A50C     E8 07ADDDFF        call bdiary_e.00405218                   ; System.@LStrPos;

0062A511     8BF0               mov esi,eax                               ; Str[i]在固定字符串 1 中的位置值

0062A513     85F6               test esi,esi

0062A515     7E 13              jle short bdiary_e.0062A52A

0062A517     8B45 08            mov eax,dword ptr ss:[ebp+8]

0062A51A     E8 0DACDDFF        call bdiary_e.0040512C

0062A51F     8B55 F4            mov edx,dword ptr ss:[ebp-C]             ; 固定字符串 2

0062A522     8A5432 FF          mov dl,byte ptr ds:[edx+esi-1]           ; 位置值在常数字符串 2 中的字符

0062A526     885418 FF          mov byte ptr ds:[eax+ebx-1],dl

0062A52A     43                 inc ebx

0062A52B     4F                 dec edi

0062A52C   ^ 75 C7              jnz short bdiary_e.0062A4F5             ; 循环编码

0062A52E     33C0               xor eax,eax

0062A530     5A                 pop edx

0062A531     59                 pop ecx

0062A532     59                 pop ecx

0062A533     64:8910            mov dword ptr fs:[eax],edx

0062A536     68 50A56200        push bdiary_e.0062A550

0062A53B     8D45 F0            lea eax,dword ptr ss:[ebp-10]

0062A53E     BA 04000000        mov edx,4

0062A543     E8 00A7DDFF        call bdiary_e.00404C48

0062A548     C3                 retn

0062A549   ^ E9 7A9FDDFF        jmp bdiary_e.004044C8

0062A54E   ^ EB EB              jmp short bdiary_e.0062A53B

0062A550     5F                 pop edi

0062A551     5E                 pop esi

0062A552     5B                 pop ebx

0062A553     8BE5               mov esp,ebp

0062A555     5D                 pop ebp

0062A556     C2 0400            retn 4
```

\-------------------------------------------------------

这个函数估计是作者自己写的一个字符串编码函数吧?用于处理注册码和机器码,下面还有一处用到。

在 `0063D21C` 处 跟进 `call bdiary_e.0062A6A8` ; 机器码处理,重要!

\-------------------------------------------------------

```
0062A6A8     55                 push ebp

0062A6A9     8BEC               mov ebp,esp

0062A6AB     33C9               xor ecx,ecx

0062A6AD     51                 push ecx

0062A6AE     51                 push ecx

0062A6AF     51                 push ecx

0062A6B0     51                 push ecx

0062A6B1     51                 push ecx

0062A6B2     51                 push ecx

0062A6B3     53                 push ebx

0062A6B4     56                 push esi

0062A6B5     8BF2               mov esi,edx

0062A6B7     8BD8               mov ebx,eax                               ; 机器码 16 进制数

0062A6B9     33C0               xor eax,eax

0062A6BB     55                 push ebp

0062A6BC     68 F4A76200        push bdiary_e.0062A7F4

0062A6C1     64:FF30            push dword ptr fs:[eax]

0062A6C4     64:8920            mov dword ptr fs:[eax],esp

0062A6C7     81F3 D9206F7E      xor ebx,7E6F20D9                           ; 机器码 16 进制 Xor 7E6F20D9

0062A6CD     8BC3               mov eax,ebx

0062A6CF     33D2               xor edx,edx

0062A6D1     52                 push edx

0062A6D2     50                 push eax

0062A6D3     8D45 FC            lea eax,dword ptr ss:[ebp-4]

0062A6D6     E8 55F5DDFF        call bdiary_e.00409C30                   ; IntToStr(Int64)

0062A6DB     8B45 FC            mov eax,dword ptr ss:[ebp-4]             ; IntToStr(机器码 16 进制 Xor 7E6F20D9),记为 M

0062A6DE     0FB600             movzx eax,byte ptr ds:[eax]              ; M[1]

0062A6E1     8B55 FC            mov edx,dword ptr ss:[ebp-4]

0062A6E4     0FB652 01          movzx edx,byte ptr ds:[edx+1]            ; M[2]

0062A6E8     03C2               add eax,edx                               ; M[1]+M[2]

0062A6EA     B9 05000000        mov ecx,5

0062A6EF     99                 cdq

0062A6F0     F7F9               idiv ecx

0062A6F2     80C2 61            add dl,61

0062A6F5     8855 F8            mov byte ptr ss:[ebp-8],dl               ; (M[1]+M[2]) Mod 5 + 61,记为 Char1

0062A6F8     8B45 FC            mov eax,dword ptr ss:[ebp-4]             ; M

0062A6FB     0FB640 02          movzx eax,byte ptr ds:[eax+2]            ; M[3]

0062A6FF     8B55 FC            mov edx,dword ptr ss:[ebp-4]             ; M

0062A702     0FB652 03          movzx edx,byte ptr ds:[edx+3]            ; M[4]

0062A706     03C2               add eax,edx

0062A708     B9 05000000        mov ecx,5

0062A70D     99                 cdq

0062A70E     F7F9               idiv ecx

0062A710     80C2 61            add dl,61

0062A713     8855 F9            mov byte ptr ss:[ebp-7],dl               ; (M[3]+M[4]) Mod 5 + 61，记为 Char2

0062A716     8B45 FC            mov eax,dword ptr ss:[ebp-4]             ; M

0062A719     0FB640 04          movzx eax,byte ptr ds:[eax+4]            ; M[5]

0062A71D     8B55 FC            mov edx,dword ptr ss:[ebp-4]             ; M

0062A720     0FB652 05          movzx edx,byte ptr ds:[edx+5]            ; M[6]

0062A724     03C2               add eax,edx

0062A726     B9 05000000        mov ecx,5

0062A72B     99                 cdq

0062A72C     F7F9               idiv ecx

0062A72E     80C2 61            add dl,61

0062A731     8855 FA            mov byte ptr ss:[ebp-6],dl               ; (M[5]+M[6]) Mod 5 + 61,记为 Char3

0062A734     8B45 FC            mov eax,dword ptr ss:[ebp-4]             ; M

0062A737     0FB640 06          movzx eax,byte ptr ds:[eax+6]            ; M[7]

0062A73B     8B55 FC            mov edx,dword ptr ss:[ebp-4]             ; M

0062A73E     0FB652 07          movzx edx,byte ptr ds:[edx+7]            ; M[8]

0062A742     03C2               add eax,edx                               ; M[7]+M[8]

0062A744     8B55 FC            mov edx,dword ptr ss:[ebp-4]             ; M

0062A747     0FB652 08          movzx edx,byte ptr ds:[edx+8]            ; M[9]

0062A74B     03C2               add eax,edx

0062A74D     B9 05000000        mov ecx,5

0062A752     99                 cdq

0062A753     F7F9               idiv ecx

0062A755     80C2 61            add dl,61

0062A758     8855 FB            mov byte ptr ss:[ebp-5],dl               ; (M[7]+M[8]+M[9]) Mod 5 + 61,记为 Char4

0062A75B     8D45 F4            lea eax,dword ptr ss:[ebp-C]
```



```
0062A75E      8A55 F8                  mov dl,byte ptr ss:[ebp-8]           ; Char1
0062A761      E8 9EA6DDFF             call bdiary_e.00404E04               ; Char 函数
0062A766      8B45 F4                 mov eax,dword ptr ss:[ebp-C]          ; Char1
0062A769      8D55 FC                 lea edx,dword ptr ss:[ebp-4]         ; M
0062A76C      B9 01000000            mov ecx,1                            ; 1
0062A771      E8 46AADDFF            call bdiary_e.004051BC               ; StrInsert;
0062A776      8D45 F0                 lea eax,dword ptr ss:[ebp-10]
0062A779      8A55 FB                 mov dl,byte ptr ss:[ebp-5]          ; Char4
0062A77C      E8 83A6DDFF            call bdiary_e.00404E04               ; Char 函数
0062A781      8B45 F0                 mov eax,dword ptr ss:[ebp-10]
0062A784      8D55 FC                 lea edx,dword ptr ss:[ebp-4]
0062A787      B9 03000000            mov ecx,3                            ; 3
0062A78C      E8 2BAADDFF            call bdiary_e.004051BC               ; StrInsert;
0062A791      8D45 EC                 lea eax,dword ptr ss:[ebp-14]
0062A794      8A55 F9                 mov dl,byte ptr ss:[ebp-7]          ; Char2
0062A797      E8 68A6DDFF            call bdiary_e.00404E04               ; Char 函数
0062A79C      8B45 EC                 mov eax,dword ptr ss:[ebp-14]
0062A79F      8D55 FC                 lea edx,dword ptr ss:[ebp-4]
0062A7A2      B9 05000000            mov ecx,5                            ; 5
0062A7A7      E8 10AADDFF            call bdiary_e.004051BC               ; StrInsert;
0062A7AC      8D45 E8                 lea eax,dword ptr ss:[ebp-18]
0062A7AF      8A55 FA                 mov dl,byte ptr ss:[ebp-6]          ; Char3
0062A7B2      E8 4DA6DDFF            call bdiary_e.00404E04               ; Char 函数
0062A7B7      8B45 E8                 mov eax,dword ptr ss:[ebp-18]
0062A7BA      8D55 FC                 lea edx,dword ptr ss:[ebp-4]
0062A7BD      B9 09000000            mov ecx,9                            ; 9
0062A7C2      E8 F5A9DDFF            call bdiary_e.004051BC               ; StrInsert;
0062A7C7      8BC6                    mov eax,esi                          ; 在 1，3，5，9 位插入字符
0062A7C9      8B55 FC                 mov edx,dword ptr ss:[ebp-4]         ; 插入结果
0062A7CC      E8 A7A4DDFF            call bdiary_e.00404C78
0062A7D1      33C0                    xor eax,eax
0062A7D3      5A                      pop edx
0062A7D4      59                      pop ecx
0062A7D5      59                      pop ecx
0062A7D6      64:8910                 mov dword ptr fs:[eax],edx
0062A7D9      68 FBA76200            push bdiary_e.0062A7FB
0062A7DE      8D45 E8                 lea eax,dword ptr ss:[ebp-18]
0062A7E1      BA 04000000            mov edx,4
0062A7E6      E8 5DA4DDFF            call bdiary_e.00404C48
0062A7EB      8D45 FC                 lea eax,dword ptr ss:[ebp-4]
0062A7EE      E8 31A4DDFF            call bdiary_e.00404C24
0062A7F3      C3                      retn
0062A7F4    ^ E9 CF9CDDFF             jmp bdiary_e.004044C8
0062A7F9    ^ EB E3                   jmp short bdiary_e.0062A7DE
0062A7FB      5E                      pop esi
0062A7FC      5B                      pop ebx
0062A7FD      8BE5                    mov esp,ebp
0062A7FF      5D                      pop ebp
0062A800      C3                      retn
```

函数完成机器码的首次变换，后来发现这是作者设计的一个玩笑……，继续

在 `0063D22A` 处  跟进 `call bdiary_e.0062A804`；结果再次处理,重要!

```
0062A804      55                      push ebp
0062A805      8BEC                    mov ebp,esp
0062A807      83C4 C0                 add esp,-40
0062A80A      53                      push ebx
0062A80B      56                      push esi
0062A80C      33C9                    xor ecx,ecx
0062A80E      894D C0                 mov dword ptr ss:[ebp-40],ecx
0062A811      894D C4                 mov dword ptr ss:[ebp-3C],ecx
0062A814      894D C8                 mov dword ptr ss:[ebp-38],ecx
0062A817      894D F4                 mov dword ptr ss:[ebp-C],ecx
0062A81A      894D F0                 mov dword ptr ss:[ebp-10],ecx
0062A81D      8955 F8                 mov dword ptr ss:[ebp-8],edx
0062A820      8945 FC                 mov dword ptr ss:[ebp-4],eax       ; 机器码首次处理结果 Code
0062A823      8B45 FC                 mov eax,dword ptr ss:[ebp-4]
0062A826      E8 99A8DDFF            call bdiary_e.004050C4
0062A82B      8D75 DC                 lea esi,dword ptr ss:[ebp-24]
0062A82E      33C0                    xor eax,eax
0062A830      55                      push ebp
0062A831      68 2FAB6200            push bdiary_e.0062AB2F
0062A836      64:FF30                 push dword ptr fs:[eax]
0062A839      64:8920                 mov dword ptr fs:[eax],esp
0062A83C      8D45 F4                 lea eax,dword ptr ss:[ebp-C]
0062A83F      8B55 FC                 mov edx,dword ptr ss:[ebp-4]
0062A842      E8 75A4DDFF            call bdiary_e.00404CBC
0062A847      8D45 F4                 lea eax,dword ptr ss:[ebp-C]       ; 指向 Code
0062A84A      B9 01000000            mov ecx,1                            ; 1
0062A84F      BA 01000000            mov edx,1                            ; 1
0062A854      E8 1BA9DDFF            call bdiary_e.00405174               ; StrDelete
0062A859      8D45 F4                 lea eax,dword ptr ss:[ebp-C]       ; Code
0062A85C      B9 01000000            mov ecx,1                            ; 1
0062A861      BA 02000000            mov edx,2                            ; 2
0062A866      E8 09A9DDFF            call bdiary_e.00405174               ; StrDelete
0062A86B      8D45 F4                 lea eax,dword ptr ss:[ebp-C]       ; Code
0062A86E      B9 01000000            mov ecx,1                            ; 1
0062A873      BA 03000000            mov edx,3                            ; 3
0062A878      E8 F7A8DDFF            call bdiary_e.00405174               ; StrDelete
0062A87D      8D45 F4                 lea eax,dword ptr ss:[ebp-C]       ; Code
0062A880      B9 01000000            mov ecx,1                            ; 1
0062A885      BA 06000000            mov edx,6                            ; 6
0062A88A      E8 E5A8DDFF            call bdiary_e.00405174               ; StrDelete
0062A88F      8B45 F4                 mov eax,dword ptr ss:[ebp-C]       ; 机器码回到原始状态?作者真是坏人!
```

到这里为止，发现 `call bdiary_e.0062A6A8` 函数对机器码首次处理结果被还原了,所以说是作者跟我们开了个玩笑^\_^

```
0062A892      E8 7DF4DDFF            call bdiary_e.00409D14               ; StrToInt64(AnsiString)
0062A897      8BD8                    mov ebx,eax
0062A899      F7D3                    not ebx                              ; Not
0062A89B      81F3 22211276          xor ebx,76122122                     ; XOR ,EBX 重要
0062A8A1      8BC3                    mov eax,ebx
0062A8A3      25 000000FF            and eax,FF000000                     ; AND
0062A8A8      C1E8 18                shr eax,18                           ; Shr
0062A8AB      8906                    mov dword ptr ds:[esi],eax           ; 保存 1
0062A8AD      8BC3                    mov eax,ebx
0062A8AF      25 0000FF00            and eax,0FF0000
0062A8B4      C1E8 10                shr eax,10
0062A8B7      8946 04                mov dword ptr ds:[esi+4],eax         ; 保存 2
0062A8BA      8BC3                    mov eax,ebx
0062A8BC      25 00FF0000            and eax,0FF00
0062A8C1      C1E8 08                shr eax,8
0062A8C4      8946 08                mov dword ptr ds:[esi+8],eax         ; 保存 3
0062A8C7      8BC3                    mov eax,ebx
0062A8C9      25 FF000000            and eax,0FF
```



```asm
0062A8CE     8946 0C              mov dword ptr ds:[esi+C],eax           ; 保存 4,分成 4 步将 EBX 拆分成 4Bit  

0062A8D1     8B16                 mov edx,dword ptr ds:[esi]  

0062A8D3     81E2 C0000000        and edx,0C0  

0062A8D9     8B4E 04             mov ecx,dword ptr ds:[esi+4]           ; 以下是简单的数据操作，看起来很烦人  

0062A8DC     81E1 C0000000        and ecx,0C0  

0062A8E2     C1E9 02              shr ecx,2  

0062A8E5     03D1                 add edx,ecx  

0062A8E7     8B4E 08             mov ecx,dword ptr ds:[esi+8]  

0062A8EA     81E1 C0000000        and ecx,0C0  

0062A8F0     C1E9 04              shr ecx,4  

0062A8F3     03D1                 add edx,ecx  

0062A8F5     25 C0000000          and eax,0C0  

0062A8FA     C1E8 06              shr eax,6  

0062A8FD     03D0                 add edx,eax  

0062A8FF     8955 CC             mov dword ptr ss:[ebp-34],edx  

0062A902     8B06                 mov eax,dword ptr ds:[esi]  

0062A904     83E0 30              and eax,30  

0062A907     C1E0 02              shl eax,2  

0062A90A     8B56 04             mov edx,dword ptr ds:[esi+4]  

0062A90D     83E2 30              and edx,30  

0062A910     03C2                 add eax,edx  

0062A912     8B56 08             mov edx,dword ptr ds:[esi+8]  

0062A915     83E2 30              and edx,30  

0062A918     C1EA 02              shr edx,2  

0062A91B     03C2                 add eax,edx  

0062A91D     8B56 0C             mov edx,dword ptr ds:[esi+C]  

0062A920     83E2 30              and edx,30  

0062A923     C1EA 04              shr edx,4  

0062A926     03C2                 add eax,edx  

0062A928     8945 D0              mov dword ptr ss:[ebp-30],eax  

0062A92B     8B06                 mov eax,dword ptr ds:[esi]  

0062A92D     83E0 0C              and eax,0C  

0062A930     C1E0 04              shl eax,4  

0062A933     8B56 04             mov edx,dword ptr ds:[esi+4]  

0062A936     83E2 0C              and edx,0C  

0062A939     C1E2 02              shl edx,2  

0062A93C     03C2                 add eax,edx  

0062A93E     8B56 08             mov edx,dword ptr ds:[esi+8]  

0062A941     83E2 0C              and edx,0C  

0062A944     03C2                 add eax,edx  

0062A946     8B56 0C             mov edx,dword ptr ds:[esi+C]  

0062A949     83E2 0C              and edx,0C  

0062A94C     C1EA 02              shr edx,2  

0062A94F     03C2                 add eax,edx  

0062A951     8945 D4              mov dword ptr ss:[ebp-2C],eax  

0062A954     8B06                 mov eax,dword ptr ds:[esi]  

0062A956     83E0 03              and eax,3  

0062A959     C1E0 06              shl eax,6  

0062A95C     8B56 04             mov edx,dword ptr ds:[esi+4]  

0062A95F     83E2 03              and edx,3  

0062A962     C1E2 04              shl edx,4  

0062A965     03C2                 add eax,edx  

0062A967     8B56 08             mov edx,dword ptr ds:[esi+8]  

0062A96A     83E2 03              and edx,3  

0062A96D     C1E2 02              shl edx,2  

0062A970     03C2                 add eax,edx  

0062A972     8B56 0C             mov edx,dword ptr ds:[esi+C]  

0062A975     83E2 03              and edx,3  

0062A978     03C2                 add eax,edx  

0062A97A     8945 D8              mov dword ptr ss:[ebp-28],eax  

0062A97D     8B5D CC             mov ebx,dword ptr ss:[ebp-34]  

0062A980     C1E3 18              shl ebx,18  

0062A983     8B45 D0              mov eax,dword ptr ss:[ebp-30]  

0062A986     C1E0 10              shl eax,10  

0062A989     03D8                 add ebx,eax  

0062A98B     8B45 D4              mov eax,dword ptr ss:[ebp-2C]  

0062A98E     C1E0 08              shl eax,8  

0062A991     03D8                 add ebx,eax  

0062A993     035D D8              add ebx,dword ptr ss:[ebp-28]  

0062A996     8BC3                 mov eax,ebx  

0062A998     25 000000FF          and eax,FF000000  

0062A99D     C1E8 18              shr eax,18  

0062A9A0     8906                 mov dword ptr ds:[esi],eax  

0062A9A2     8BC3                 mov eax,ebx  

0062A9A4     25 0000FF00          and eax,0FF0000  

0062A9A9     C1E8 10              shr eax,10  

0062A9AC     8946 04              mov dword ptr ds:[esi+4],eax  

0062A9AF     8BC3                 mov eax,ebx  

0062A9B1     25 00FF0000          and eax,0FF00  

0062A9B6     C1E8 08              shr eax,8  

0062A9B9     8946 08              mov dword ptr ds:[esi+8],eax  

0062A9BC     81E3 FF000000        and ebx,0FF  

0062A9C2     895E 0C              mov dword ptr ds:[esi+C],ebx  

0062A9C5     8B06                 mov eax,dword ptr ds:[esi]  

0062A9C7     8BD0                 mov edx,eax  

0062A9C9     81E2 F0000000        and edx,0F0  

0062A9CF     C1EA 04              shr edx,4  

0062A9D2     83E0 0F              and eax,0F  

0062A9D5     C1E0 04              shl eax,4  

0062A9D8     03D0                 add edx,eax  

0062A9DA     8916                 mov dword ptr ds:[esi],edx  

0062A9DC     8B46 04              mov eax,dword ptr ds:[esi+4]  

0062A9DF     8BD0                 mov edx,eax  

0062A9E1     81E2 F0000000        and edx,0F0  

0062A9E7     C1EA 04              shr edx,4  

0062A9EA     83E0 0F              and eax,0F  

0062A9ED     C1E0 04              shl eax,4  

0062A9F0     03D0                 add edx,eax  

0062A9F2     8956 04              mov dword ptr ds:[esi+4],edx  

0062A9F5     8B46 08              mov eax,dword ptr ds:[esi+8]  

0062A9F8     8BD0                 mov edx,eax  

0062A9FA     81E2 F0000000        and edx,0F0  

0062AA00     C1EA 04              shr edx,4  

0062AA03     83E0 0F              and eax,0F  

0062AA06     C1E0 04              shl eax,4  

0062AA09     03D0                 add edx,eax  

0062AA0B     8956 08              mov dword ptr ds:[esi+8],edx  

0062AA0E     8B46 0C              mov eax,dword ptr ds:[esi+C]  

0062AA11     8BD0                 mov edx,eax  

0062AA13     81E2 F0000000        and edx,0F0  

0062AA19     C1EA 04              shr edx,4  

0062AA1C     83E0 0F              and eax,0F  

0062AA1F     C1E0 04              shl eax,4  

0062AA22     03D0                 add edx,eax  

0062AA24     8956 0C              mov dword ptr ds:[esi+C],edx  

0062AA27     8B5E 04              mov ebx,dword ptr ds:[esi+4]  

0062AA2A     C1E3 18              shl ebx,18  

0062AA2D     8B06                 mov eax,dword ptr ds:[esi]  

0062AA2F     C1E0 10              shl eax,10  

0062AA32     03D8                 add ebx,eax  

0062AA34     C1E2 08              shl edx,8  

0062AA37     03DA                 add ebx,edx  

0062AA39     035E 08              add ebx,dword ptr ds:[esi+8]  

0062AA3C     8BC3                 mov eax,ebx  

0062AA3E     33D2                 xor edx,edx  

0062AA40     52                   push edx  

0062AA41     50                   push eax  

0062AA42     8D45 F0              lea eax,dword ptr ss:[ebp-10]  

0062AA45     E8 E6F1DDFF          call bdiary_e.00409C30                ; SysUtils.IntToStr  

0062AA4A     8B45 F0              mov eax,dword ptr ss:[ebp-10]         ; 终于产生结果了，晕……  

0062AA4D     0FB600               movzx eax,byte ptr ds:[eax]          ; 继续处理,我的机器码计算结果:3158163090  

0062AA50     8B55 F0              mov edx,dword ptr ss:[ebp-10]  

0062AA53     0FB652 01            movzx edx,byte ptr ds:[edx+1]  

0062AA57     03C2                 add eax,edx  

0062AA59     B9 05000000          mov ecx,5  

0062AA5E     99                   cdq  

0062AA5F     F7F9                 idiv ecx  

0062AA61     80C2 61              add dl,61  

0062AA64     8855 ED              mov byte ptr ss:[ebp-13],dl  

0062AA67     8B45 F0              mov eax,dword ptr ss:[ebp-10]  

0062AA6A     0FB640 02            movzx eax,byte ptr ds:[eax+2]  

0062AA6E     8B55 F0              mov edx,dword ptr ss:[ebp-10]  

0062AA71     0FB652 03            movzx edx,byte ptr ds:[edx+3]  

0062AA75     03C2                 add eax,edx  

0062AA77     B9 05000000          mov ecx,5
```



# 汇编代码分析

## 机器码变换与字符插入

```asm
0062AA7C     99                     cdq  
0062AA7D     F7F9                   idiv ecx  
0062AA7F     80C2 61               add dl,61  
0062AA82     8855 EE               mov byte ptr ss:[ebp-12],dl  
0062AA85     8B45 F0               mov eax,dword ptr ss:[ebp-10]  
0062AA88     0FB640 04             movzx eax,byte ptr ds:[eax+4]  
0062AA8C     8B55 F0               mov edx,dword ptr ss:[ebp-10]  
0062AA8F     0FB652 05             movzx edx,byte ptr ds:[edx+5]  
0062AA93     03C2                   add eax,edx  
0062AA95     B9 05000000           mov ecx,5  
0062AA9A     99                     cdq  
0062AA9B     F7F9                   idiv ecx  
0062AA9D     80C2 61               add dl,61  
0062AAA0     8855 EF               mov byte ptr ss:[ebp-11],dl  
0062AAA3     8D45 C8               lea eax,dword ptr ss:[ebp-38]  
0062AAA6     8A55 ED               mov dl,byte ptr ss:[ebp-13]  
0062AAA9     E8 56A3DDFF           call bdiary_e.00404E04  
0062AAAE     8B45 C8               mov eax,dword ptr ss:[ebp-38]  
0062AAB1     8D55 F0               lea edx,dword ptr ss:[ebp-10]  
0062AAB4     B9 01000000           mov ecx,1  
0062AAB9     E8 FEA6DDFF           call bdiary_e.004051BC             ; LStrInsert  
0062AABE     8D45 C4               lea eax,dword ptr ss:[ebp-3C]  
0062AAC1     8A55 EE               mov dl,byte ptr ss:[ebp-12]  
0062AAC4     E8 3BA3DDFF           call bdiary_e.00404E04  
0062AAC9     8B45 C4               mov eax,dword ptr ss:[ebp-3C]  
0062AACC     8D55 F0               lea edx,dword ptr ss:[ebp-10]  
0062AACF     B9 05000000           mov ecx,5  
0062AAD4     E8 E3A6DDFF           call bdiary_e.004051BC             ; LStrInsert  
0062AAD9     8D45 C0               lea eax,dword ptr ss:[ebp-40]  
0062AADC     8A55 EF               mov dl,byte ptr ss:[ebp-11]  
0062AADF     E8 20A3DDFF           call bdiary_e.00404E04  
0062AAE4     8B45 C0               mov eax,dword ptr ss:[ebp-40]  
0062AAE7     8D55 F0               lea edx,dword ptr ss:[ebp-10]  
0062AAEA     B9 09000000           mov ecx,9  
0062AAEF     E8 C8A6DDFF           call bdiary_e.004051BC             ; LStrInsert,又是插入字符  
0062AAF4     8B45 F8               mov eax,dword ptr ss:[ebp-8]  
0062AAF7     8B55 F0               mov edx,dword ptr ss:[ebp-10]      ; 最终结果出来了  
0062AAFA     E8 79A1DDFF           call bdiary_e.00404C78  
0062AAFF     33C0                   xor eax,eax  
0062AB01     5A                     pop edx  
0062AB02     59                     pop ecx  
0062AB03     59                     pop ecx  
0062AB04     64:8910               mov dword ptr fs:[eax],edx  
0062AB07     68 36AB6200           push bdiary_e.0062AB36  
0062AB0C     8D45 C0               lea eax,dword ptr ss:[ebp-40]  
0062AB0F     BA 03000000           mov edx,3  
0062AB14     E8 2FA1DDFF           call bdiary_e.00404C48  
0062AB19     8D45 F0               lea eax,dword ptr ss:[ebp-10]  
0062AB1C     BA 02000000           mov edx,2  
0062AB21     E8 22A1DDFF           call bdiary_e.00404C48  
0062AB26     8D45 FC               lea eax,dword ptr ss:[ebp-4]  
0062AB29     E8 F6A0DDFF           call bdiary_e.00404C24  
0062AB2E     C3                     retn  
0062AB2F   ^ E9 9499DDFF           jmp bdiary_e.004044C8  
0062AB34   ^ EB D6                jmp short bdiary_e.0062AB0C  
0062AB36     5E                     pop esi  
0062AB37     5B                     pop ebx  
0062AB38     8BE5                   mov esp,ebp  
0062AB3A     5D                     pop ebp  
0062AB3B     C3                     retn  
```

上面这段代码时间上完成了机器码的变换过程，算法简单，但是代码量比较多，让人看了心烦，我可没这个耐心，呵呵！

分析到此，所有的验证计算过程全部已经完成，接下来开始比较了：

## 注册验证与爆破

```asm
0063D4FE     68 3FD56300           push <bdiary_e.->System.@HandleAnyExcept>  
0063D503     64:FF30               push dword ptr fs:[eax]  
0063D506     64:8920               mov dword ptr fs:[eax],esp  
0063D509     8B45 D8               mov eax,dword ptr ss:[ebp-28]        ; 试炼码长度-2 计算结果  
0063D50C     8B55 D4               mov edx,dword ptr ss:[ebp-2C]        ; 机器码计算结果  
0063D50F <>  E8 0C7BDCFF           call bdiary_e.00405020               ; ->System.@LStrCmp;  
0063D514     75 17                 jnz short bdiary_e.0063D52D         ; 爆破 1，改为 JE  
0063D516     8B45 D0               mov eax,dword ptr ss:[ebp-30]        ; 试炼码第 1 位  
0063D519     8B55 CC               mov edx,dword ptr ss:[ebp-34]        ; 试炼码第 5 位  
0063D51C <>  E8 FF7ADCFF           call bdiary_e.00405020               ; ->System.@LStrCmp;  
0063D521     75 0A                 jnz short bdiary_e.0063D52D         ; 爆破 2，改为 JE  
0063D523     A1 B8C16800           mov eax,dword ptr ds:[68C1B8]  
0063D528     C600 01                mov byte ptr ds:[eax],1             ; 置注册成功标志  
0063D52B     EB 08                 jmp short bdiary_e.0063D535  
```

## 算法总结

注册验证基本上采用 `F1(注册码)=F2(机器码)` 的验证方式，没法做内存注册机，对机器码变换运算过程比较烦，但涉及的计算过程还是比较简单的，有耐心的朋友可以作息看看，我性子急 `^\_^`，明白了大致验证过程就想收工了。

<完>
