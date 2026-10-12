
# 3gpConvert 2.0 算法分析（浮点）

> 原文：[`www.pediy.com/kssd/pediy10/62339.html`](https://www.pediy.com/kssd/pediy10/62339.html)

【文章标题】: `3gpConvert` 2.0 算法分析（浮点）

【文章作者】: 小子贼野

【作者主页】: `http://mayday.unpack.cn`

【作者 QQ 号】: 你猜

【下载地址】: `http://www.3gpconvert.com/`

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 【详细过程】

```
00401C60  /$  55                  push ebp
00401C61  |.  8BEC                mov ebp,esp
00401C63  |.  83E4 F8            and esp,FFFFFFF8
00401C66  |.  6A FF              push -1
00401C68  |.  68 AE4D4000        push 3gpConve.00404DAE              ;  SE 句柄安装
00401C6D  |.  64:A1 0000000>mov eax,dword ptr fs:[0]
00401C73  |.  50                  push eax
00401C74  |.  64:8925 00000>mov dword ptr fs:[0],esp
00401C7B  |.  81EC 18010000     sub esp,118
00401C81  |.  53                  push ebx
00401C82  |.  55                  push ebp
00401C83  |.  56                  push esi
00401C84  |.  8B35 40604000     mov esi,dword ptr ds:[<&KERNEL32.GetProc];  kernel32.GetProcAddress
00401C8A  |.  8BC6                mov eax,esi
00401C8C  |.  57                  push edi
00401C8D  |.  2D 00000070       sub eax,70000000
00401C92  |.  894C24 14         mov dword ptr ss:[esp+14],ecx
00401C96  |.  0F88 77030000     js 3gpConve.00402013
00401C9C  |.  68 0C914000       push 3gpConve.0040910C              ; /FileName = "kernel32.dll"
00401CA1  |.  FF15 30604000     call dword ptr ds:[<&KERNEL32.LoadLibrar>; \LoadLibraryA
00401CA7  |.  8BE8                mov ebp,eax
00401CA9  |.  68 00914000       push 3gpConve.00409100              ; /ProcNameOrOrdinal = "ReadFile"
00401CAE  |.  55                  push ebp                            ; |hModule
00401CAF  |.  FFD6                call esi                            ; \GetProcAddress
00401CB1  |.  68 F4904000       push 3gpConve.004090F4              ; /ProcNameOrOrdinal = "CreateFileA"
00401CB6  |.  55                  push ebp                            ; |hModule
00401CB7  |.  8BD8                mov ebx,eax                         ; |
00401CB9  |.  FFD6                call esi                            ; \GetProcAddress
00401CBB  |.  68 E8904000       push 3gpConve.004090E8              ; /ProcNameOrOrdinal = "WriteFile"
00401CC0  |.  55                  push ebp                            ; |hModule
00401CC1  |.  8BF8                mov edi,eax                         ; |
00401CC3  |.  FFD6                call esi                            ; \GetProcAddress
00401CC5  |.  8D4C24 10         lea ecx,dword ptr ss:[esp+10]
00401CC9  |.  E8 EE280000       call <jmp.&MFC42.#540>
00401CCE  |.  8B4C24 14         mov ecx,dword ptr ss:[esp+14]
00401CD2  |.  8D5424 10         lea edx,dword ptr ss:[esp+10]
00401CD6  |.  81C1 C4000000     add ecx,0C4
00401CDC  |.  C78424 300100>mov dword ptr ss:[esp+130],0
00401CE7  |.  51                  push ecx
00401CE8  |.  68 DC904000       push 3gpConve.004090DC              ;  ASCII "%s\erfv.dat"
00401CED  |.  52                  push edx
00401CEE  |.  E8 A5280000       call <jmp.&MFC42.#2818>
00401CF3  |.  8B4424 1C         mov eax,dword ptr ss:[esp+1C]
00401CF7  |.  83C4 0C           add esp,0C
00401CFA  |.  6A 00             push 0
00401CFC  |.  6A 00             push 0
00401CFE  |.  6A 03             push 3
00401D00  |.  6A 00             push 0
00401D02  |.  6A 00             push 0
00401D04  |.  68 00000080       push 80000000
00401D09  |.  50                  push eax
00401D0A  |.  FFD7                call edi
00401D0C  |.  8BF0                mov esi,eax
00401D0E  |.  83FE FF           cmp esi,-1
00401D11  |.  75 0C             jnz short 3gpConve.00401D1F
00401D13  |.  898424 300100>mov dword ptr ss:[esp+130],eax
00401D1A  |.  E9 EB020000       jmp 3gpConve.0040200A
00401D1F  |>  B9 08000000       mov ecx,8
00401D24  |.  33C0                xor eax,eax
00401D26  |.  8D7C24 64         lea edi,dword ptr ss:[esp+64]
00401D2A  |.  6A 00             push 0                                ; /Origin = FILE_BEGIN
00401D2C  |.  F3:AB             rep stos dword ptr es:[edi]          ; |
00401D2E  |.  66:AB             stos word ptr es:[edi]                ; |
00401D30  |.  AA                stos byte ptr es:[edi]                ; |
00401D31  |.  B9 08000000       mov ecx,8                             ; |
00401D36  |.  33C0                xor eax,eax                         ; |
00401D38  |.  8D7C24 44         lea edi,dword ptr ss:[esp+44]         ; |
00401D3C  |.  6A 00             push 0                                ; |pOffsetHi = NULL
00401D3E  |.  F3:AB             rep stos dword ptr es:[edi]          ; |
00401D40  |.  66:AB             stos word ptr es:[edi]                ; |
00401D42  |.  68 20030000       push 320                               ; |OffsetLo = 320 (800.)
00401D47  |.  56                  push esi                            ; |hFile
00401D48  |.  AA                stos byte ptr es:[edi]                ; |
00401D49  |.  FF15 48604000     call dword ptr ds:[<&KERNEL32.SetFilePoi>; \SetFilePointer
00401D4F  |.  8D4C24 24         lea ecx,dword ptr ss:[esp+24]
00401D53  |.  6A 00             push 0
00401D55  |.  51                  push ecx
00401D56  |.  8D5424 6C         lea edx,dword ptr ss:[esp+6C]
00401D5A  |.  6A 1E             push 1E
00401D5C  |.  52                  push edx
00401D5D  |.  56                  push esi
00401D5E  |.  FFD3                call ebx
00401D60  |.  8D4424 24         lea eax,dword ptr ss:[esp+24]
00401D64  |.  6A 00             push 0
00401D66  |.  50                  push eax
00401D67  |.  8D4C24 48         lea ecx,dword ptr ss:[esp+48]
00401D6B  |.  6A 1E             push 1E
00401D6D  |.  51                  push ecx
00401D6E  |.  56                  push esi
00401D6F  |.  FFD3                call ebx
00401D71  |.  8D5424 14         lea edx,dword ptr ss:[esp+14]
00401D75  |.  8D4424 1C         lea eax,dword ptr ss:[esp+1C]
00401D79  |.  52                  push edx                            ; /pLastWrite
00401D7A  |.  8D4C24 2C         lea ecx,dword ptr ss:[esp+2C]         ; |
00401D7E  |.  50                  push eax                            ; |pLastAccess
00401D7F  |.  51                  push ecx                            ; |pCreationTime
00401D80  |.  56                  push esi                            ; |hFile
00401D81  |.  FF15 44604000     call dword ptr ds:[<&KERNEL32.GetFileTim>; \GetFileTime
00401D87  |.  8D5424 1C         lea edx,dword ptr ss:[esp+1C]
00401D8B  |.  8D4424 30         lea eax,dword ptr ss:[esp+30]
00401D8F  |.  52                  push edx                            ; /pFileTime
00401D90  |.  50                  push eax                            ; |pSystemTime
00401D91  |.  66:C74424 38 >mov word ptr ss:[esp+38],7CF             ; |
00401D98  |.  66:C74424 3A >mov word ptr ss:[esp+3A],5               ; |
00401D9F  |.  66:C74424 3E >mov word ptr ss:[esp+3E],16               ; |
00401DA6  |.  FF15 28604000     call dword ptr ds:[<&KERNEL32.SystemTime>; \SystemTimeToFileTime
00401DAC  |.  8D4C24 14         lea ecx,dword ptr ss:[esp+14]
00401DB0  |.  8D5424 1C         lea edx,dword ptr ss:[esp+1C]
00401DB4  |.  51                  push ecx                            ; /pLastWrite
00401DB5  |.  8D4424 2C         lea eax,dword ptr ss:[esp+2C]         ; |
00401DB9  |.  52                  push edx                            ; |pLastAccess
00401DBA  |.  50                  push eax                            ; |pCreationTime
00401DBB  |.  56                  push esi                            ; |hFile
00401DBC  |.  FF15 70604000     call dword ptr ds:[<&KERNEL32.SetFileTim>; \SetFileTime
```



## 汇编代码分析

```
  00401DC2  |.  56                 push esi                         ; /hObject  
  00401DC3  |.  FF15 34604000    call dword ptr ds:[<&KERNEL32.CloseHandle>; \CloseHandle  
  00401DC9  |.  55                 push ebp                         ; /hLibModule  
  00401DCA  |.  FF15 38604000    call dword ptr ds:[<&KERNEL32.FreeLibrary>; \FreeLibrary  
  00401DD0  |.  83CB FF          or ebx,FFFFFFFF  
  00401DD3  |.  8D7C24 64        lea edi,dword ptr ss:[esp+64]  
  00401DD7  |.  8BCB             mov ecx,ebx  
  00401DD9  |.  33C0             xor eax,eax  
  00401DDB  |.  F2:AE            repne scas byte ptr es:[edi]  
  00401DDD  |.  F7D1             not ecx  
  00401DDF  |.  49               dec ecx  
  00401DE0  |.  83F9 01          cmp ecx,1                          ;  这里比较用户名填没填的  
  00401DE3  |.  0F82 1A020000    jb 3gpConve.00402003  
  00401DE9  |.  8D7C24 40        lea edi,dword ptr ss:[esp+40]  
  00401DED  |.  8BCB             mov ecx,ebx  
  00401DEF  |.  F2:AE            repne scas byte ptr es:[edi]  
  00401DF1  |.  F7D1             not ecx  
  00401DF3  |.  49               dec ecx  
  00401DF4  |.  83F9 01          cmp ecx,1  
  00401DF7  |.  0F82 06020000    jb 3gpConve.00402003               ;  这里比较注册码填没填的  
  00401DFD  |.  BF D0904000      mov edi,3gpConve.004090D0          ;  ASCII "3gp051128"  
  00401E02  |.  8BCB             mov ecx,ebx  
  00401E04  |.  F2:AE            repne scas byte ptr es:[edi]  
  00401E06  |.  F7D1             not ecx  
  00401E08  |.  49               dec ecx  
  00401E09  |.  8D7C24 64        lea edi,dword ptr ss:[esp+64]  
  00401E0D  |.  8BE9             mov ebp,ecx  
  00401E0F  |.  8BCB             mov ecx,ebx  
  00401E11  |.  F2:AE            repne scas byte ptr es:[edi]  
  00401E13  |.  F7D1             not ecx  
  00401E15  |.  49               dec ecx  
  00401E16  |.  C68424 300100>   mov byte ptr ss:[esp+130],1  
  00401E1E  |.  03E9             add ebp,ecx  
  00401E20  |.  8D4C24 14        lea ecx,dword ptr ss:[esp+14]  
  00401E24  |.  8BF3             mov esi,ebx  
  00401E26  |.  E8 91270000      call <jmp.&MFC42.#540>  
  00401E2B  |.  8D4C24 64        lea ecx,dword ptr ss:[esp+64]  
  00401E2F  |.  C68424 300100>   mov byte ptr ss:[esp+130],2  
  00401E37  |.  51               push ecx  
  00401E38  |.  8D4C24 18        lea ecx,dword ptr ss:[esp+18]  
  00401E3C  |.  E8 2B290000      call <jmp.&MFC42.#860>  
  00401E41  |.  68 D0904000      push 3gpConve.004090D0             ;  ASCII "3gp051128"  
  00401E46  |.  8D4C24 18        lea ecx,dword ptr ss:[esp+18]  
  00401E4A  |.  E8 17290000      call <jmp.&MFC42.#941>            ;  用户名+“3gp051128”  
  00401E4F  |.  8D4C24 14        lea ecx,dword ptr ss:[esp+14]  
  00401E53  |.  E8 08290000      call <jmp.&MFC42.#4204>           ;  转换成大写的  
  00401E58  |.  6A 00            push 0  
  00401E5A  |.  8D4C24 18        lea ecx,dword ptr ss:[esp+18]  
  00401E5E  |.  E8 6B270000      call <jmp.&MFC42.#2915>  
  00401E63  |.  33C9             xor ecx,ecx  
  00401E65  |.  8BF8             mov edi,eax  
  00401E67  |.  85ED             test ebp,ebp  
  00401E69  |.  7E 68            jle short 3gpConve.00401ED3  
  00401E6B  |>  8BC1             /mov eax,ecx  
  00401E6D  |.  BB 03000000      |mov ebx,3  
  00401E72  |.  99               |cdq  
  00401E73  |.  F7FB             |idiv ebx  
  00401E75  |.  46               |inc esi  
  00401E76  |.  85D2             |test edx,edx                    ;  Switch (cases 0..2)  
  00401E78  |.  75 16            |jnz short 3gpConve.00401E90  
  00401E7A  |.  8A0439           |mov al,byte ptr ds:[ecx+edi]      ;  第一位 Ascii; Case 0 of switch 00401E76  
  00401E7D  |.  0FBED0           |movsx edx,al                     ;  给 EDX  
  00401E80  |.  83EA 05          |sub edx,5                        ;  -5  
  00401E83  |.  83FA 41          |cmp edx,41  
  00401E86  |.  7E 04            |jle short 3gpConve.00401E8C  
  00401E88  |.  2C 05            |sub al,5  
  00401E8A  |.  EB 38            |jmp short 3gpConve.00401EC4  
  00401E8C  |>  04 05            |add al,5  
  00401E8E  |.  EB 34            |jmp short 3gpConve.00401EC4  
  00401E90  |>  83FA 01          |cmp edx,1  
  00401E93  |.  75 16            |jnz short 3gpConve.00401EAB  
  00401E95  |.  8A0439           |mov al,byte ptr ds:[ecx+edi]      ;  第二位 Ascii; Case 1 of switch 00401E76  
  00401E98  |.  0FBED0           |movsx edx,al                     ;  给 EDX  
  00401E9B  |.  83C2 07          |add edx,7                        ;  +7  
  00401E9E  |.  83FA 5A          |cmp edx,5A                       ;  和$5A 比较  
  00401EA1  |.  7D 04            |jge short 3gpConve.00401EA7       ;  大于或者等于就跳  
  00401EA3  |.  04 07            |add al,7                         ;  +7  
  00401EA5  |.  EB 1D            |jmp short 3gpConve.00401EC4  
  00401EA7  |>  2C 07            |sub al,7  
  00401EA9  |.  EB 19            |jmp short 3gpConve.00401EC4  
  00401EAB  |>  83FA 02          |cmp edx,2  
  00401EAE  |.  75 1B            |jnz short 3gpConve.00401ECB  
  00401EB0  |.  8A0439           |mov al,byte ptr ds:[ecx+edi]      ;  第三位 Ascii; Case 2 of switch 00401E76  
  00401EB3  |.  0FBED0           |movsx edx,al                     ;  给 EDX  
  00401EB6  |.  83EA 09          |sub edx,9                        ;  -9  
  00401EB9  |.  83FA 41          |cmp edx,41                       ;  和$41 比较  
  00401EBC  |.  7E 04            |jle short 3gpConve.00401EC2       ;  小于等于则跳  
  00401EBE  |.  2C 09            |sub al,9  
  00401EC0  |.  EB 02            |jmp short 3gpConve.00401EC4  
  00401EC2  |>  04 09            |add al,9                         ;  +9  
  00401EC4  |>  888434 880000>   |mov byte ptr ss:[esp+esi+88],al  
  00401ECB  |>  41               |inc ecx                          ;  Default case of switch 00401E76  
  00401ECC  |.  3BCD             |cmp ecx,ebp  
  00401ECE  |.^ 7C 9B            \jl short 3gpConve.00401E6B  
  00401ED0  |.  83CB FF          or ebx,FFFFFFFF  
  00401ED3  |>  DD05 C8674000    fld qword ptr ds:[4067C8]  
  00401ED9  |.  33C0             xor eax,eax  
  00401EDB  |.  85F6             test esi,esi  
  00401EDD  |.  7E 17            jle short 3gpConve.00401EF6  
  00401EDF  |>  0FBE8C04 8800>   /movsx ecx,byte ptr ss:[esp+eax+88]    ;  取新字符串的 Ascii  
  00401EE7  |.  894C24 1C        |mov dword ptr ss:[esp+1C],ecx  
  00401EEB  |.  40               |inc eax  
  00401EEC  |.  DB4424 1C        |fild dword ptr ss:[esp+1C]          ;  转换成浮点  
  00401EF0  |.  3BC6             |cmp eax,esi  
  00401EF2  |.  DEC1             |faddp st(1),st                     ;  加起来  
  00401EF4  |.^ 7C E9            \jl short 3gpConve.00401EDF          ;  循环  
```

这两个循环的意思就是逐个取新字符串的 `Ascii`，然后和固定数值比较，再做加减法运算，然后转换成字符串

连接起来，然后再取新字符串的 `Ascii`，加起来

```
  00401EF6  |>  D9C0             fld st  
  00401EF8  |.  D9FE             fsin                         ;  B=Sin(A)A 是上面循环后的值  
  00401EFA  |.  D9FF             fcos                         ;  C=Cos(B)  
  00401EFC  |.  D9FE             fsin                         ;  D=Sin(C)  
  00401EFE  |.  D9FF             fcos                         ;  E=Cos(D)  
  00401F00  |.  D9FE             fsin                         ;  F=Sin(E)  
```

数学函数的运算了



00401F02  |.  DD5424 1C       fst qword ptr ss:[esp+1C]
00401F06  |.  DC1D C8674000   fcomp qword ptr ds:[4067C8]
00401F0C  |.  DFE0            fstsw ax
00401F0E  |.  F6C4 01         test ah,1
00401F11  |.  74 23           je short 3gpConve.00401F36
00401F13  |>  DC0D C0674000  /fmul qword ptr ds:[4067C0]
00401F19  |.  D9C0            |fld st
00401F1B  |.  D9FE            |fsin
00401F1D  |.  D9FF            |fcos
00401F1F  |.  D9FE            |fsin
00401F21  |.  D9FF            |fcos
00401F23  |.  D9FE            |fsin
00401F25  |.  DD5424 1C       |fst qword ptr ss:[esp+1C]
00401F29  |.  DC1D C8674000   |fcomp qword ptr ds:[4067C8]
00401F2F  |.  DFE0            |fstsw ax
00401F31  |.  F6C4 01         |test ah,1
00401F34  |.^ 75 DD           \jnz short 3gpConve.00401F13
00401F36  |>  8B5424 20       mov edx,dword ptr ss:[esp+20]
00401F3A  |.  8B4424 1C       mov eax,dword ptr ss:[esp+1C]
00401F3E  |.  52              push edx
00401F3F  |.  50              push eax                              ; /<%.14f>
00401F40  |.  8D8C24 E00000>  lea ecx,dword ptr ss:[esp+E0]          ; |
00401F47  |.  68 C8904000     push 3gpConve.004090C8                ; |format = "%.14f"
00401F4C  |.  51              push ecx                              ; |s
00401F4D  |.  DDD8            fstp st                               ; |
00401F4F  |.  FF15 2C644000   call dword ptr ds:[<&MSVCRT.sprintf>] ; \sprintf
00401F55  |.  8DBC24 E80000>  lea edi,dword ptr ss:[esp+E8]         ; 上面是对字符串进行格式化，只取 14 位，由于那里是 8 所以要四舍五入，变成 9
00401F5C  |.  8BCB            mov ecx,ebx
00401F5E  |.  33C0            xor eax,eax
00401F60  |.  83C4 10         add esp,10
00401F63  |.  33D2            xor edx,edx
00401F65  |.  F2:AE           repne scas byte ptr es:[edi]
00401F67  |.  F7D1            not ecx
00401F69  |.  49              dec ecx
00401F6A  |.  83E9 02         sub ecx,2
00401F6D  |.  74 27           je short 3gpConve.00401F96
00401F6F  |>  8A8414 DA0000>  /mov al,byte ptr ss:[esp+edx+DA]      ; 取格式化后字符串的 Ascii
00401F76  |.  8DBC24 D80000>  |lea edi,dword ptr ss:[esp+D8]
00401F7D  |.  04 41           |add al,41                             ; +41
00401F7F  |.  8BCB            |mov ecx,ebx
00401F81  |.  888414 880000>  |mov byte ptr ss:[esp+edx+88],al      ; 转成字符
00401F88  |.  33C0            |xor eax,eax
00401F8A  |.  42              |inc edx
00401F8B  |.  F2:AE           |repne scas byte ptr es:[edi]
00401F8D  |.  F7D1            |not ecx
00401F8F  |.  83C1 FD         |add ecx,-3
00401F92  |.  3BD1            |cmp edx,ecx
00401F94  |.^ 72 D9           \jb short 3gpConve.00401F6F          ; 循环

这个循环是取格式化后字符串的 `Ascii`（不取“0.”的 `Ascii`），加 `41`，转成字符，连接起来，就是最终注册码

00401F96  |>  8D8C24 880000>  lea ecx,dword ptr ss:[esp+88]
00401F9D  |.  C68434 880000>  mov byte ptr ss:[esp+esi+88],0
00401FA5  |.  51              push ecx                              ; /StringOrChar
00401FA6  |.  FF15 68644000   call dword ptr ds:[<&USER32.CharUpperA>] ; \CharUpperA
00401FAC  |.  8D4C24 14       lea ecx,dword ptr ss:[esp+14]
00401FB0  |.  C68424 300100>  mov byte ptr ss:[esp+130],1
00401FB8  |.  E8 E1250000     call <jmp.&MFC42.#800>
00401FBD  |.  8D7C24 40       lea edi,dword ptr ss:[esp+40]
00401FC1  |.  8BCB            mov ecx,ebx
00401FC3  |.  33C0            xor eax,eax
00401FC5  |.  8DB424 880000>  lea esi,dword ptr ss:[esp+88]
00401FCC  |.  F2:AE           repne scas byte ptr es:[edi]
00401FCE  |.  F7D1            not ecx
00401FD0  |.  49              dec ecx
00401FD1  |.  8D7C24 40       lea edi,dword ptr ss:[esp+40]
00401FD5  |.  33D2            xor edx,edx
00401FD7  |.  899C24 300100>  mov dword ptr ss:[esp+130],ebx
00401FDE  |.  F3:A6           repe cmps byte ptr es:[edi],byte ptr ds:>
00401FE0  |.  8D4C24 10       lea ecx,dword ptr ss:[esp+10]
00401FE4  |.  75 28           jnz short 3gpConve.0040200E          ; 把这个跳该了，可以爆破
00401FE6  |.  E8 B3250000     call <jmp.&MFC42.#800>
00401FEB  |.  B0 01           mov al,1
00401FED  |.  8B8C24 280100>  mov ecx,dword ptr ss:[esp+128]
00401FF4  |.  64:890D 00000>  mov dword ptr fs:[0],ecx
00401FFB  |.  5F              pop edi
00401FFC  |.  5E              pop esi
00401FFD  |.  5D              pop ebp
00401FFE  |.  5B              pop ebx
00401FFF  |.  8BE5            mov esp,ebp
00402001  |.  5D              pop ebp
00402002  |.  C3              retn

### 算法总结

1. 将用户名和固定字符串连接并转成大写，逐个取 `Ascii`，然后和固定数值比较是否符合条件，再做加减法运算，然后转换成字符串连接起来，然后再取新字符串的 `Ascii`，加起来
2. 再进行一系列的数学函数运算，最后转成浮点小数
3. 循环取格式化后字符串的 `Ascii`（不取“0.”的 `Ascii`），加 `41`，转成字符，连接起来，就是最终注册码

---

### 经验总结

第一次分析浮点算法，虽然算法很简单，但是也挺累的，锻炼了自己的耐心，呵呵

---

### 版权声明

本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢！

2008 年 04 月 01 日 16:35:58
