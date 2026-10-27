
```
010D505D ; int __stdcall GetFunctionAddress(int BaseAddress,int CryptCode)
010D505D GetFunctionAddress proc near
010D505D FunctionAddress= dword ptr -4
010D505D BaseAddress= dword ptr  4
010D505D CryptCode= dword ptr  8
010D505D         pusha                                     ; 1
010D505E         push    eax
010D505F         push    edx
010D5060         push    ecx
010D5061         push    ecx
010D5062         pusha
010D5063         popa
010D5064         sidt    qword ptr [esp-2]
010D5069         pop     ecx
010D506A         jmp     loc_10D507E
010D506F         db 11h, 0AAh, 58h, 8Eh, 9Bh, 0E4h, 0CFh, 16h, 67h, 0C0h, 0DDh
010D506F         db 0F7h, 0F8h, 13h, 0C5h
010D507E loc_10D507E:
010D507E         pop     ecx
010D507F         rdtsc
010D5081         jnb     loc_10D5089
010D5087         pusha
010D5088         popa
010D5089 loc_10D5089:
010D5089         pop     edx
010D508A         pop     eax
010D508B         mov     eax, 0                            ; 2
010D5090         pusha
010D5091         movsx   ecx, cx
010D5094         add     [ebp+153E2D91h], edi
010D509A         popa
010D509B         mov     edx, [esp+20h+CryptCode]          ; 3  CryptCode 函数名加密代码
010D509F         stc
010D50A0         mov     ss:(dwFunctionsCount+1531F38Ah - ThemidaSeg)[ebp], eax ; 4
010D50A6         pusha
010D50A7         mov     edi, [ebp+153E23DDh]
010D50AD         mov     edx, [ebp+153E2DB9h]
010D50B3         popa
010D50B4         push    edx
010D50B5         push    edx
010D50B6         mov     [ebp+153E11CDh], edx
010D50BC         sidt    qword ptr [esp-2]
010D50C1         pop     edx
010D50C2         push    eax
010D50C3         push    edx
010D50C4         stc
010D50C5         rdtsc
010D50C7 loc_10D50C7:
010D50C7         sub     [ebp+153E04B5h], edx
010D50CD         pop     edx
010D50CE         pop     eax
010D50CF         pop     edx
010D50D0         mov     esi, IMAGE_DOS_HEADER.e_lfanew    ; 5 PE 文件头的偏移量
010D50D5         push    eax
010D50D6         push    edx
010D50D7         jle     loc_10D50E3
010D50DD         jle     $+6
010D50E3 loc_10D50E3:
010D50E3         rdtsc
010D50E5         jmp     loc_10D50FB
010D50EA         db 6Fh, 95h, 0D0h, 51h, 69h, 31h, 0ECh, 0CEh, 9Fh, 65h, 0FEh, 2Bh
010D50EA         db 0B8h, 0Ch, 0DEh, 40h, 18h
010D50FB loc_10D50FB:
010D50FB         pop     edx
010D50FC         pop     eax
010D50FD         add     esi, [esp+20h+BaseAddress]        ; 6 DosHeader.e_lfanew + BaseAddress
010D5101         mov     [ebp+153E0021h], edi
010D5107         pusha
010D5108         call    loc_10D5119
010D510D         db 2 dup(53h), 0AFh, 7Ah, 0B8h, 0A5h, 8Ah, 20h, 58h, 84h, 8Fh
010D510D         db 91h
010D5119 loc_10D5119:
010D5119         jb      $+6
010D511F         pop     eax
010D5120         mov     [ebp+153E029Dh], ebx
010D5126         popa
010D5127         lodsw                                     ; 7 获取PE头的偏移量
010D5129         sub     [ebp+153E271Dh], ebx
010D512F         add     eax, [esp+20h+BaseAddress]        ; 8 加上基地址得到 PE 头的起始位置
010D5133         jmp     loc_10D5142
010D5138         db 3Dh, 0BCh, 69h, 0B6h, 4Bh, 0ADh, 0AEh, 0E2h, 79h, 2Bh
010D5142 loc_10D5142:
010D5142         nop
010D5143         jle     loc_10D5157
010D5149         pusha
010D514A         jg      $+6
010D5150         jl      $+6
010D5156         popa
010D5157 loc_10D5157:
010D5157         push    eax                               ; 9
010D5158         jnp     loc_10D5164
010D515E         pusha
010D515F         mov     esi, edx
010D5161         mov     ah, dl
010D5163         popa
010D5164 loc_10D5164:
010D5164         nop
010D5165         pusha
010D5166         clc
010D5167         call    loc_10D5171
010D516C         db 0D2h, 1, 94h, 0ECh, 0E3h
010D5171 loc_10D5171:
010D5171         xor     eax, [ebp+153E2071h]
010D5177         pop     ecx
010D5178         popa
010D5179         mov     eax, [eax+IMAGE_NT_HEADERS.OptionalHeader.DirectoryExport.VirtualAddress] ; 10
010D517C         jnb     loc_10D5190
010D5182         pusha
010D5183         push    eax
010D5184         push    edx
010D5185         rdtsc
010D5187         pop     edx
010D5188         pop     eax
010D5189         or      [ebp+153E1569h], edx
010D518F         popa
010D5190 loc_10D5190:
010D5190         stc
010D5191         add     eax, [esp+24h+BaseAddress]        ; 11, Export.VirtualAddress + BaseAddress
010D5195         sub     [ebp+153E1965h], edi
010D519B         mov     eax, [eax+IMAGE_EXPORT_DIRECTORY.NumberOfNames] ; 12
010D519E         jb      loc_10D51AA
010D51A4         mov     [ebp+153E0D35h], ecx
010D51AA loc_10D51AA:
010D51AA         inc     eax                               ; 13
010D51AB         jbe     loc_10D51BE
010D51B1         push    eax
010D51B2         push    edx
010D51B3         cmc
010D51B4         rdtsc
010D51B6         push    eax
010D51B7         push    edx
010D51B8         rdtsc
010D51BA         pop     edx
010D51BB         pop     eax
010D51BC         pop     edx
010D51BD         pop     eax
010D51BE loc_10D51BE:                                      ; 14
010D51BE         mov     ss:(dwNumeberOfName+1531F38Ah - ThemidaSeg)[ebp], eax
010D51C4         sub     [ebp+153E199Dh], esi
010D51CA         pop     eax                               ; 15
010D51CB         pusha
010D51CC         mov     eax, [ebp+153E0181h]
010D51D2         jmp     loc_10D51E2
010D51D7         db 1Fh, 5Ch, 94h, 99h, 29h, 91h, 49h, 0Dh, 0B6h, 97h, 8Eh
010D51E2 loc_10D51E2:
010D51E2         popa
010D51E3         mov     esi, [eax+IMAGE_NT_HEADERS.OptionalHeader.DirectoryExport.VirtualAddress] ; 16
010D51E6         jb      loc_10D51F2
010D51EC         or      [ebp+153E2ED9h], ebx
010D51F2 loc_10D51F2:
010D51F2         add     esi, IMAGE_EXPORT_DIRECTORY.AddressOfFunctions ; 17
010D51F5         jnp     loc_10D5210
010D51FB         push    eax
010D51FC         push    edx
010D51FD         push    esi
010D51FE         push    esi
010D51FF         sidt    qword ptr [esp-2]
010D5204         pop     esi
010D5205         pop     esi
010D5206         rdtsc
010D5208         sub     [ebp+153E2869h], edi
010D520E         pop     edx
010D520F         pop     eax
010D5210 loc_10D5210:                                      ; 18, AddressOfFunctions + BaseAddress
010D5210         add     esi, [esp+20h+BaseAddress]
010D5214         jmp     loc_10D521F
010D5219         db 88h, 0C1h, 0A4h, 6Ah, 85h, 17h
010D521F loc_10D521F:
010D521F         clc
010D5220         lea     edi, (dwAddressOfFunctions+1531F38Ah - ThemidaSeg)[ebp] ; 19
010D5220                                                   ; 取 AddressOfFunctions 变量地址
010D5226         pusha
010D5227         and     eax, [ebp+153E2411h]
010D522D         mov     esi, [ebp+153E2285h]
010D5233         popa
010D5234         lodsd                                     ; 20 取AddressOfFunctions 的偏移量
010D5235         mov     [ebp+153E15F9h], eax
010D523B         add     eax, [esp+20h+BaseAddress]        ; 21 + BaseAddress
010D523F         jmp     loc_10D524F
010D5244         db 3Ch, 69h, 36h, 4, 0A3h, 0C9h, 88h, 19h, 0CBh, 3Fh, 7Fh
010D524F loc_10D524F:
010D524F         stosd                                     ; 22 保存 AddressOfFunctions
010D5250         jmp     loc_10D5267
010D5255         db 0ABh, 37h, 2 dup(50h), 53h, 9Ah, 68h, 0C4h, 36h, 69h, 0B8h
010D5255         db 2Ah, 7Ah, 0F5h, 52h, 0E6h, 33h, 0E7h
010D5267 loc_10D5267:
010D5267         jmp     loc_10D5271
010D526C         db 62h, 0CCh, 5Dh, 5Ah, 0BAh
010D5271 loc_10D5271:
010D5271         lea     edi, (dwAddressOfNames+1531F38Ah - ThemidaSeg)[ebp] ; 23
010D5271                                                   ; 取保存 AddressOfNames 变量地址
010D5277         mov     [ebp+153E2219h], edi
010D527D         lodsd                                     ; 24 取 AddressOfNames 的偏移量
010D527E         pusha
010D527F         mov     ebx, [ebp+153E044Dh]
010D5285         mov     ecx, [ebp+153E1A51h]
010D528B         popa
010D528C         add     eax, [esp+20h+BaseAddress]        ; 25 + BaseAddress
010D5290         pusha
010D5291         or      [ebp+153E133Dh], ecx
010D5297         cmc
010D5298         popa
010D5299         push    eax                               ; 26
010D529A         push    eax
010D529B         push    edx
010D529C         jmp     loc_10D52B4
010D52A1         db 4Dh, 0AEh, 42h, 0B7h, 0AEh, 0E4h, 0B0h, 3Ah, 0CCh, 15h, 0EEh
010D52A1         db 31h, 6Bh, 0F2h, 4Eh, 96h, 0E6h, 3Fh, 0A0h
010D52B4 loc_10D52B4:
010D52B4         rdtsc
010D52B6         push    eax
010D52B7         push    edx
010D52B8         jmp     loc_10D52C8
010D52BD         db 0B8h, 21h, 0F6h, 3Ah, 0D2h, 0D3h, 0ADh, 66h, 0D6h, 0F6h, 0D1h
010D52C8 loc_10D52C8:
010D52C8         rdtsc
010D52CA         pusha
010D52CB         popa
010D52CC         pop     edx
010D52CD         pop     eax
010D52CE         pop     edx
010D52CF         pop     eax
010D52D0         mov     [esp], eax
010D52D3         mov     [ebp+153E1895h], esi
010D52D9         stosd                                     ; 27 保存 AddressOfNames 地址
010D52DA         jnz     loc_10D52F9
010D52E0         jmp     loc_10D52F9
010D52E5         db 9Eh, 3Ah, 2Ah, 0E4h, 0BCh, 46h, 8Bh, 5, 7Dh, 0C5h, 9, 0C0h
010D52E5         db 0AFh, 61h, 0A4h, 0Ah, 0FAh, 84h, 0A1h, 0D4h
010D52F9 loc_10D52F9:
010D52F9         lea     edi, (dwAddressOfNamesOrdinals+1531F38Ah - ThemidaSeg)[ebp] ; 28
010D52F9                                                   ; 取 AddressOfNameOrdinals 变量地址
010D52FF         stc
010D5300         cld                                       ; 29
010D5301         lodsd                                     ; 30 读 AddressNameOrdinals 的偏移量
010D5302         push    eax
010D5303         push    edx
010D5304         jnz     loc_10D5320
010D530A         jmp     loc_10D5320
010D530F         db 6Ch, 29h, 0AEh, 0E6h, 87h, 64h, 49h, 7Ah, 92h, 0Ch, 38h, 5
010D530F         db 0CCh, 0CEh, 3Fh, 18h, 0F2h
010D5320 loc_10D5320:
010D5320         rdtsc
010D5322         cld
010D5323         pop     edx
010D5324         pop     eax
010D5325         pusha
010D5326         and     di, 44F4h
010D532B         mov     al, 7Ch
010D532D         popa
010D532E         add     eax, [esp+24h+BaseAddress]        ; 31, + BaseAddress
010D5332         jnz     loc_10D534D
010D5338         jmp     loc_10D534D
010D533D         db 3Eh, 3Bh, 53h, 57h, 1Ch, 0C4h, 0EAh, 0F4h, 5Ah, 0CBh, 86h, 67h
010D533D         db 70h, 0BCh, 0D2h, 0B5h
010D534D loc_10D534D:
010D534D         cld
010D534E         stosd                                     ; 32 保存 AddressOfNameOrdinals 地址
010D534F         clc
010D5350         pop     esi                               ; 33
010D5351         pusha
010D5352         push    edi
010D5353         push    edi
010D5354         mov     edi, [ebp+153E154Dh]
010D535A         sidt    qword ptr [esp-2]
010D535F         pop     edi
010D5360         sub     [ebp+153E17B1h], edi
010D5366         pop     edi
010D5367         popa
010D5368         jmp     LoopNextName
010D536D         db 0E8h, 0DCh, 0B9h, 0F5h, 0DFh, 0FBh, 0D8h, 8, 0Fh, 24h, 79h
010D536D         db 0D5h, 0A3h, 0A8h
010D537B;下面代码搜索第一字母相匹配的函数名
010D537B LoopNextName:
010D537B         dec     ss:(dwNumeberOfName+1531F38Ah - ThemidaSeg)[ebp]    ; 34
010D537B                                                   ; 计数器减 1
010D5381         cmp     ss:(dwNumeberOfName+1531F38Ah - ThemidaSeg)[ebp], 0 ; 35
010D5381                                                   ; 如果为 0 则失败
010D5388         jnz     short IfNumberOfNameNotZero       ; 36
010D538A         mov     [esp+20h+FunctionAddress], 0      ; 37 如果没有找到，eax 中返回 0
010D5392         lea     esi, (NoFind_CC+1531F38Ah - ThemidaSeg)[ebp] ; 38
010D5398         jmp     esi                               ; 39 没有要求的函数跳转
010D539A IfNumberOfNameNotZero:                            ; 40
010D539A         push    esi
010D539B         lodsd                                     ; 41, 取函数名的偏移地址
010D539C         add     eax, [esp+24h+BaseAddress]        ; 42, + BaseAddress
010D53A0         xchg    eax, edi                          ; 43
010D53A1         mov     ebx, edi                          ; 44
010D53A3         mov     al, ss:(APIFristChar+1531F38Ah - ThemidaSeg)[ebp] ; 45
010D53A3                                                   ; 查找 API 函数名第一个字母
010D53A9         test    al, al                            ; 46 如果为 0 为无效函数名
010D53AB         jz      short loc_10D53B1                 ; 47
010D53AD         cmp     al, [edi]                         ; 48, 比较函数的第一字母
010D53AF         jnz     short loc_10D53FD                 ; 49
010D53B1 loc_10D53B1:                                      ; 50
010D53B1         push    edi
010D53B2         xor     al, al                            ; 51
010D53B4 loc_10D53B4:
010D53B4         scasb                                     ; 52 字符串以 0 结尾
010D53B5         jnz     short loc_10D53B4                 ; 53
010D53B7         pop     esi                               ; 54
010D53B8         sub     edi, ebx                          ; 55, 得到函数名的长度
010D53BA         push    edx                               ; 56
010D53BB         cld                                       ; 57
010D53BC         xor     ecx, ecx                          ; 58
010D53BE         dec     ecx                               ; 59
010D53BF         mov     edx, ecx                          ; 60
010D53C1 ;下面的代码将API函数名加密，然后比较，如相同则是需要的函数
010D53C1 loc_10D53C1:                                      ; 61
010D53C1         xor     eax, eax
010D53C3         xor     ebx, ebx                          ; 62
010D53C5         lodsb                                     ; 63
010D53C6         xor     al, cl                            ; 64
010D53C8         mov     cl, ch                            ; 65
010D53CA         mov     ch, dl                            ; 66
010D53CC         mov     dl, dh                            ; 67
010D53CE         mov     dh, 8                             ; 68
010D53D0 loc_10D53D0:                                      ; 69
010D53D0         shr     bx, 1
010D53D3         rcr     ax, 1                             ; 70
010D53D6         jnb     short loc_10D53E1                 ; 71
010D53D8         xor     ax, 5041h                         ; 72
010D53DC         xor     bx, 5449h                         ; 73
010D53E1 loc_10D53E1:                                      ; 74
010D53E1         dec     dh
010D53E3         jnz     short loc_10D53D0                 ; 75
010D53E5         xor     ecx, eax                          ; 76
010D53E7         xor     edx, ebx                          ; 77
010D53E9         dec     edi                               ; 78
010D53EA         jnz     short loc_10D53C1                 ; 79
010D53EC         not     edx                               ; 80
010D53EE         not     ecx                               ; 81
010D53F0         mov     eax, edx                          ; 82
010D53F2         rol     eax, 10h                          ; 83
010D53F5         mov     ax, cx                            ; 84
010D53F8         pop     edx                               ; 85
010D53F9         cmp     edx, eax                          ; 86
010D53FB         jz      short IfEqu                       ; 87
010D53FD loc_10D53FD:                                      ; 88
010D53FD         pop     esi
010D53FE         add     esi, 4                            ; 89
010D5401         inc     ss:(dwFunctionsCount+1531F38Ah - ThemidaSeg)[ebp] ; 90
010D5407         jmp     LoopNextName                      ; 91
010D540C IfEqu:                                            ; 92
010D540C         pop     esi
010D540D         push    eax
010D540E         push    edx
010D540F         mov     [ebp+153E13DDh], edi
010D5415         rdtsc
010D5417         jmp     loc_10D5424
010D541C         db 0CEh, 34h, 14h, 87h, 0F2h, 52h, 0EDh, 67h
010D5424 loc_10D5424:
010D5424         pop     edx
010D5425         pop     eax
010D5426         sub     [ebp+153E1F45h], edi
010D542C         mov     eax, ss:(dwFunctionsCount+1531F38Ah - ThemidaSeg)[ebp] ; 93
010D542C                                                   ; FunctionsCount 中的数即导出函数的编号
010D5432         pusha
010D5433         push    esi
010D5434         sbb     eax, 11C02E28h
010D543A         pop     ecx
010D543B         and     edi, [ebp+153E1B69h]
010D5441         popa
010D5442         shl     eax, 1                            ; 94, 导出函数编号数组的每个编号占两个字节，
010D5442                                                   ; 所以这里将这个 FunctionsCount 乘 2
010D5444         push    eax
010D5445         push    edx
010D5446         sub     [ebp+153E14E5h], edi
010D544C         rdtsc
010D544E         jnp     loc_10D5455
010D5454         stc
010D5455 loc_10D5455:
010D5455         pop     edx
010D5456         pop     eax
010D5457         stc
010D5458         add     eax, ss:(dwAddressOfNamesOrdinals+1531F38Ah - ThemidaSeg)[ebp] ;
010D5458                                                   ; 95, 加上导出函数编号数组的基地址
010D545E         jmp     loc_10D546D
010D5463         db 12h, 6Eh, 0C2h, 0C5h, 58h, 0F3h, 5Ch, 28h, 4Fh, 5Ch
010D546D loc_10D546D:
010D546D         sub     esi, esi                          ; 96
010D546F         jmp     loc_10D5480
010D5474 byte_10D5474 db 0D8h, 14h, 5Bh, 3Dh, 52h, 7Eh, 24h, 7Eh, 6, 4Eh, 22h, 0E3h
010D5480 loc_10D5480:
010D5480         xchg    eax, esi                          ; 97 esi 中为导出函数编号数组的基地址
010D5481         mov     [ebp+153E26EDh], ebx
010D5487         push    eax
010D5488         push    edx
010D5489         mov     [ebp+153E222Dh], eax
010D548F         rdtsc
010D5491         jnp     loc_10D5498
010D5497         stc
010D5498 loc_10D5498:
010D5498         pop     edx
010D5499         pop     eax
010D549A         lodsw                                     ; 98, 取出API函数的编号 16位
010D549C         cld
010D549D         shl     eax, 2                            ; 99, 编号乘 4
010D54A0         pusha
010D54A1         mov     ah, 3Fh
010D54A3         adc     eax, 795FFC2Eh
010D54A9         popa
010D54AA         add     eax, ss:(dwAddressOfFunctions+1531F38Ah - ThemidaSeg)[ebp] ;
010D54AA                                                   ; 100, 在AddressOfFunctions 数组中定位到
010D54AA                                                   ; API 函数偏移量的地址
010D54B0         add     [ebp+153E1839h], edi
010D54B6         xchg    eax, esi                          ; 101, 将地址交换到 esi 寄存器中准备 Load
010D54B7         jnz     loc_10D54BE
010D54BD         stc
010D54BE loc_10D54BE:
010D54BE         lodsd                                      ; 102 取API函数的偏移地址
010D54BF         mov     [ebp+153E058Dh], edx
010D54C5         mov     ecx, [esp+24h]
010D54C9         jmp     loc_10D54D3
010D54CE         db 0ADh, 35h, 31h, 0, 0Dh
010D54D3 loc_10D54D3:
010D54D3         add     eax, [esp+20h+BaseAddress]         ; 103 + BaseAddress
010D54D3                                                    ; eax 中既是 API 函数的调用地址
010D54D7         push    edi
010D54D8         push    edi
010D54D9         pusha
010D54DA         and     di, 0AFBBh
010D54DF         call    loc_10D54F7
010D54E4         db 0E0h, 96h, 90h, 81h, 0F4h, 0Bh, 0D2h, 1Ah, 22h, 0F7h, 77h, 65h
010D54E4         db 6, 0BBh, 88h, 4Bh, 8Fh, 7Eh, 5Fh
010D54F7 loc_10D54F7:
010D54F7         pop     edx
010D54F8         popa
010D54F9         sidt    qword ptr [esp-2]
010D54FE         pop     edi
010D54FF         jmp     loc_10D550B
010D5504         db 6Dh, 46h, 1Ah, 3Eh, 0C4h, 0BCh, 0A6h
010D550B loc_10D550B:
010D550B         pop     edi
010D550C         mov     [esp+20h+FunctionAddress], eax    ; 104, 保存结果，即函数调用地址
010D550C                                                   ; 这里将结果保存在入口pushad的第
010D550C                                                   ; 一个压入堆栈的寄存器，即eax，程序
010D550C                                                   ; 在正常退出时，popad 将结果弹人
010D550C                                                   ; eax 寄存器。
010D5510         pusha
010D5511         jge     loc_10D552D
010D5517         jmp     loc_10D552D
010D551C         db 7Fh, 0Fh, 35h, 9Bh, 5, 0CDh, 4, 0B0h, 1Ah, 78h, 0EAh, 60h, 0E3h
010D551C         db 0A2h, 75h, 0B9h, 0BCh
010D552D loc_10D552D:
010D552D         mov     [ebp+153E2829h], eax
010D5533         popa
010D5534         xchg    eax, esi                          ; 105, 将函数起始地址转入 esi
010D5534                                                   ; 准备检测是否有debug断点
010D5535         or      [ebp+153E1F99h], edx
010D553B ;下面这段代码是用来检测API函数的入口是否有断点，Debug 断点中断是一个非常特殊的代码，0CCh，
010D553B ;它的二进制码为 11001100b 该段代码利用 1100 的特征，循环左移，四个循环为一个组合，检测是
010D553B ;否为 1100b，这段代码的构思非常巧妙。
010D553B         lodsb                                     ; 106, 取入口地址的第一个字节
010D553C         push    eax
010D553D         push    eax
010D553E         cmc
010D553F         sidt    qword ptr [esp-2]
010D5544         pop     eax
010D5545         xor     [ebp+153E1D01h], ecx
010D554B         pop     eax
010D554C         pusha
010D554D         push    esi
010D554E         mov     [ebp+153E162Dh], eax
010D5554         pop     ecx
010D5555         mov     [ebp+153E0331h], edx
010D555B         popa
010D555C         xor     cl, cl                            ; 107, 计数器清零
010D555E         mov     [ebp+153E23FDh], eax
010D5564         inc     cl                                ; 108
010D5566         pusha
010D5567         cld
010D5568         add     ecx, [ebp+153E15C9h]
010D556E         popa
010D556F LoopCheckINT3:
010D556F         rcl     al, 1                      ; 109 1001100b 如果第 7 位是 0，表示不是断点
010D5571         jnb     NoFind_CC                  ; 110 没有断点跳转，下同
010D5577         mov     [ebp+153E1ABDh], edx
010D557D         push    eax
010D557E         push    eax
010D557F         clc
010D5580         sidt    qword ptr [esp-2]
010D5585         pop     eax
010D5586         mov     [ebp+153E162Dh], eax
010D558C         pop     eax
010D558D         rcl     al, 1                      ; 111 001100b  如果第 6 位是 0，表示不是断点
010D558F         jnb     NoFind_CC                  ; 112
010D5595         mov     ss:(dword_10D4733+1531F38Ah - ThemidaSeg)[ebp], ebx
010D559B         rcl     al, 1                      ; 113 01100b   如果第 5 位是 1，表示不是断点
010D559D         jb      NoFind_CC                  ; 114
010D55A3         cld
010D55A4         rcl     al, 1                      ; 115 1100b    如果第 4 位是 1，表示不是断点
010D55A6         jb      NoFind_CC                  ; 116
010D55AC ;下面为发现了断点中断的特殊代码 0CCh，程序从这转移，提示发现 Debug
010D55AC         push    esi
010D55AD         push    esi
010D55AE         jmp     loc_10D55BD
010D55B3         db 4, 38h, 0CDh, 79h, 8Ch, 0FBh, 77h, 9, 82h, 9Fh
010D55BD loc_10D55BD:
010D55BD         sidt    qword ptr [esp-2]
010D55C2         pop     esi
010D55C3         push    eax
010D55C4         push    edx
010D55C5         cld
010D55C6         rdtsc
010D55C8         push    eax
010D55C9         push    edx
010D55CA         rdtsc
010D55CC         pop     edx
010D55CD         pop     eax
010D55CE         pop     edx
010D55CF         pop     eax
010D55D0         pop     esi
010D55D1         dec     cl                                ; 117 计数器减一
010D55D3         jns     LoopCheckINT3                     ; 118 循环检测低位
010D55D9         push    esi
010D55DA         push    esi
010D55DB         jmp     loc_10D55EA
010D55E0         db 0ECh, 0E2h, 7Fh, 0E5h, 4Fh, 53h, 6Bh, 64h, 0EDh, 39h
010D55EA loc_10D55EA:
010D55EA         sidt    qword ptr [esp-2]
010D55EF         pop     esi
010D55F0         push    edx
010D55F1         push    edx
010D55F2         push    ecx
010D55F3         push    ecx
010D55F4         sidt    qword ptr [esp-2]
010D55F9         pop     ecx
010D55FA         pop     ecx
010D55FB         sidt    qword ptr [esp-2]
010D5600         pop     edx
010D5601         jp      $+6
010D5607         pop     edx
010D5608         pop     esi
010D5609         add     esp, 20h                          ; 119 堆栈平衡
010D560C         push    eax
010D560D         push    edx
010D560E         or      [ebp+153E2499h], edi
010D5614         rdtsc
010D5616         push    edi
010D5617         push    edi
010D5618         jbe     $+6
010D561E         sidt    qword ptr [esp-2]
010D5623         pop     edi
010D5624         clc
010D5625         pop     edi
010D5626         pop     edx
010D5627         pop     eax
010D5628         mov     [esp+FunctionAddress], 0          ; 120 发现debug将函数调用地址置 0
010D5630         sub     [ebp+153E13C9h], edi
010D5636         mov     eax, 1                            ; 121 发现debug设标志
010D563B         clc
010D563C         lea     ecx, (FindDebug+1531F38Ah - ThemidaSeg)[ebp] ; 122
010D5642         mov     ss:(FindDebugFlag+1531F38Ah - ThemidaSeg)[ebp], eax ; 123
010D5648 ;从这里进入发现 Debug 的处理
010D5648         jmp     ecx                               ; 124 转移到处理发现 debug 的程序
010D564A         db 89h, 95h, 51h, 21h, 3Eh, 15h, 0E9h, 0Dh, 3 dup(0), 12h, 0E3h
010D564A         db 94h, 0F6h, 77h, 9Ah, 4, 17h, 6Fh, 0C6h, 0BBh, 48h, 53h
010D5662 NoFind_CC:                                        ; 125, 清除这个API首字母
010D5662         mov     ss:(dwAPIFristChar+1531F38Ah - ThemidaSeg)[ebp], 0
010D5669         pusha
010D566A         mov     edi, [ebp+153E1BC5h]
010D5670         jmp     loc_10D567F
010D5675         db 0ECh, 22h, 9, 87h, 51h, 0D6h, 7Fh, 49h, 0F3h, 0FFh
010D567F loc_10D567F:
010D567F         popa
010D5680         popa
010D5681         pusha
010D5682         jmp     loc_10D5695
010D5687         db 1Ch, 36h, 91h, 0B8h, 82h, 0B8h, 1Dh, 75h, 8Ah, 90h, 15h, 4Ah
010D5687         db 19h, 65h
010D5695 loc_10D5695:
010D5695         movsx   edx, bx
010D5698         popa
010D5699         pusha
010D569A         sub     [ebp+153E265Dh], esi
010D56A0         mov     bh, dl
010D56A2         popa                                      ; 126
010D56A3         retn    8                                 ; 127 平衡入口参数
010D56A3 GetFunctionAddress endp
```
