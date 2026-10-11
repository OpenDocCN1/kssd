
# Audio Record Expert 2.0 2007 1006 破解分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-306.htm`](https://www.pediy.com/kssd/pediy09/pediy09-306.htm)

> 【文章标题】: Audio Record Expert 2.0 2007 1006 破解分析  
> 【文章作者】: `xss517`  
> 【软件名称】: Audio Record Expert 2.0 2007 1006  
> 【下载地址】: 自己搜索下载  
> 【编写语言】: Borland Delphi 6.0 - 7.0  
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

【详细过程】

`peid` 查看语言 Borland Delphi 6.0 - 7.0

`KANAL 2.9` 查看

`ADLER32` :: `000FF466` :: `00500066`
The reference is above.

`MD5` :: `0012EF6B` :: `0052FB6B`
The reference is above.

`ZLIB` `deflate` `[long]` :: `00142064` :: `00543A64`
Referenced at `00503596`
Referenced at `005036B5`
Referenced at `0050393E`

看来是运用了 `MD5` 算法，我对加密算法和加密壳一样都不太熟

用 `dede` 分析 `btnRegisterClick` `005368D0`

```
005368D0   /.  55                 push    ebp
005368D1   |.  8BEC               mov     ebp, esp
005368D3   |.  33C9               xor     ecx, ecx
005368D5   |.  51                 push    ecx
005368D6   |.  51                 push    ecx
005368D7   |.  51                 push    ecx
005368D8   |.  51                 push    ecx
005368D9   |.  51                 push    ecx
005368DA   |.  51                 push    ecx
005368DB   |.  51                 push    ecx
005368DC   |.  51                 push    ecx
005368DD   |.  53                 push    ebx
005368DE   |.  8BD8               mov     ebx, eax
005368E0   |.  33C0               xor     eax, eax
005368E2   |.  55                 push    ebp
005368E3   |.  68 3E6A5300        push    00536A3E
005368E8   |.  64:FF30          push    dword ptr fs:[eax]
005368EB   |.  64:8920          mov     dword ptr fs:[eax], esp
005368EE   |.  8D55 F8          lea     edx, [local.2]
005368F1   |.  8B83 54030000    mov     eax, dword ptr [ebx+354]
005368F7   |.  E8 9882F2FF      call    0045EB94
005368FC   |.  8B45 F8          mov     eax, [local.2]
005368FF   |.  8D55 FC          lea     edx, [local.1]
00536902   |.  E8 0D23EDFF      call    00408C14
00536907   |.  8B55 FC          mov     edx, [local.1]
0053690A   |.  8D83 50050000    lea     eax, dword ptr [ebx+550]
00536910   |.  E8 03DBECFF      call    00404418
00536915   |.  8D55 F0          lea     edx, [local.4]
00536918   |.  8B83 5C030000    mov     eax, dword ptr [ebx+35C]
0053691E   |.  E8 7182F2FF      call    0045EB94
00536923   |.  8B45 F0          mov     eax, [local.4]
00536926   |.  8D55 F4          lea     edx, [local.3]
00536929   |.  E8 E622EDFF      call    00408C14
0053692E   |.  8B55 F4          mov     edx, [local.3]
00536931   |.  8D83 54050000    lea     eax, dword ptr [ebx+554]
00536937   |.  E8 DCDAECFF      call    00404418
0053693C   |.  83BB 50050000 00 cmp     dword ptr [ebx+550], 0
00536943   |.  75 2F             jnz     short 00536974
00536945   |.  8D45 EC          lea     eax, [local.5]
00536948   |.  8B93 74050000    mov     edx, dword ptr [ebx+574]
0053694E   |.  E8 C5E2ECFF      call    00404C18
00536953   |.  8B55 EC          mov     edx, [local.5]
00536956   |.  8B83 80030000    mov     eax, dword ptr [ebx+380]
0053695C   |.  E8 AF67FAFF      call    004DD110
00536961   |.  8B83 54030000    mov     eax, dword ptr [ebx+354]
00536967   |.  8B10             mov     edx, dword ptr [eax]
00536969   |.  FF92 C4000000    call    dword ptr [edx+C4]
0053696F   |.  E9 8F000000      jmp     00536A03
00536974   |>  83BB 54050000 00 cmp     dword ptr [ebx+554], 0
0053697B   |.  75 2C             jnz     short 005369A9
0053697D   |.  8D45 E8          lea     eax, [local.6]
00536980   |.  8B93 78050000    mov     edx, dword ptr [ebx+578]
00536986   |.  E8 8DE2ECFF      call    00404C18
0053698B   |.  8B55 E8          mov     edx, [local.6]
0053698E   |.  8B83 80030000    mov     eax, dword ptr [ebx+380]
00536994   |.  E8 7767FAFF      call    004DD110
00536999   |.  8B83 5C030000    mov     eax, dword ptr [ebx+35C]
0053699F   |.  8B10             mov     edx, dword ptr [eax]
005369A1   |.  FF92 C4000000    call    dword ptr [edx+C4]
005369A7   |.  EB 5A             jmp     short 00536A03
005369A9   |>  8BC3               mov     eax, ebx
005369AB   |.  E8 9C000000      call    00536A4C               ;  关键 call f7 跟入
005369B0   |.  85C0               test    eax, eax
005369B2   |.  74 25             je      short 005369D9
005369B4   |.  8D45 E4          lea     eax, [local.7]
005369B7   |.  8B93 7C050000    mov     edx, dword ptr [ebx+57C]
005369BD   |.  E8 56E2ECFF      call    00404C18
005369C2   |.  8B55 E4          mov     edx, [local.7]
005369C5   |.  8B83 80030000    mov     eax, dword ptr [ebx+380]
005369CB   |.  E8 4067FAFF      call    004DD110
005369D0   |.  8BC3               mov     eax, ebx
005369D2   |.  E8 51030000      call    00536D28
005369D7   |.  EB 2A             jmp     short 00536A03
005369D9   |>  8D45 E0          lea     eax, [local.8]
005369DC   |.  8B93 80050000    mov     edx, dword ptr [ebx+580]
005369E2   |.  E8 31E2ECFF      call    00404C18
005369E7   |.  8B55 E0          mov     edx, [local.8]
005369EA   |.  8B83 80030000    mov     eax, dword ptr [ebx+380]
005369F0   |.  E8 1B67FAFF      call    004DD110
005369F5   |.  8B83 5C030000    mov     eax, dword ptr [ebx+35C]
005369FB   |.  8B10             mov     edx, dword ptr [eax]
005369FD   |.  FF92 C4000000    call    dword ptr [edx+C4]
00536A03   |>  33C0               xor     eax, eax
00536A05   |.  5A                 pop     edx
00536A06   |.  59                 pop     ecx
00536A07   |.  59                 pop     ecx
00536A08   |.  64:8910          mov     dword ptr fs:[eax], edx
00536A0B   |.  68 456A5300        push    00536A45
00536A10   |>  8D45 E0          lea     eax, [local.8]
00536A13   |.  BA 04000000        mov     edx, 4
00536A18   |.  E8 AFE0ECFF      call    00404ACC
00536A1D   |.  8D45 F0          lea     eax, [local.4]
00536A20   |.  E8 9FD9ECFF      call    004043C4
00536A25   |.  8D45 F4          lea     eax, [local.3]
00536A28   |.  E8 97D9ECFF      call    004043C4
00536A2D   |.  8D45 F8          lea     eax, [local.2]
00536A30   |.  E8 8FD9ECFF      call    004043C4
00536A35   |.  8D45 FC          lea     eax, [local.1]
00536A38   |.  E8 87D9ECFF      call    004043C4
00536A3D   \.  C3                 ret
```

```
00536A4C   /$  55                 push    ebp
00536A4D   |.  8BEC               mov     ebp, esp
00536A4F   |.  B9 06000000        mov     ecx, 6
00536A54   |>  6A 00              /push   0
00536A56   |.  6A 00              |push   0
00536A58   |.  49                 |dec     ecx
00536A59   |.^ 75 F9             \\jnz     short 00536A54
00536A5B   |.  51                 push    ecx
00536A5C   |.  53                 push    ebx
```



```
>   00536A5D    |.  56                   push    esi  

>   00536A5E    |.  57                   push    edi  

>   00536A5F    |.  8BD8                 mov     ebx, eax  

>   00536A61    |.  33C0                 xor     eax, eax  

>   00536A63    |.  55                   push    ebp  

>   00536A64    |.  68 0B6D5300          push    00536D0B  

>   00536A69    |.  64:FF30              push    dword ptr fs:[eax]  

>   00536A6C    |.  64:8920              mov     dword ptr fs:[eax], esp  

>   00536A6F    |.  8D45 FC              lea     eax, [local.1]  

>   00536A72    |.  8B93 50050000        mov     edx, dword ptr [ebx+550]  

>   00536A78    |.  E8 DFD9ECFF          call    0040445C  

>   00536A7D    |.  8D45 F8              lea     eax, [local.2]  

>   00536A80    |.  8B93 54050000        mov     edx, dword ptr [ebx+554]  

>   00536A86    |.  E8 D1D9ECFF          call    0040445C  

>   00536A8B    |.  8D45 FC              lea     eax, [local.1]  

>   00536A8E    |.  8B93 50050000        mov     edx, dword ptr [ebx+550]  

>   00536A94    |.  E8 C3D9ECFF          call    0040445C  

>   00536A99    |.  8D45 F8              lea     eax, [local.2]  

>   00536A9C    |.  8B93 54050000        mov     edx, dword ptr [ebx+554]  

>   00536AA2    |.  E8 B5D9ECFF          call    0040445C  

>   00536AA7    |.  8D45 FC              lea     eax, [local.1]  

>   00536AAA    |.  8B93 50050000        mov     edx, dword ptr [ebx+550]  

>   00536AB0    |.  E8 A7D9ECFF          call    0040445C  

>   00536AB5    |.  8D45 F8              lea     eax, [local.2]  

>   00536AB8    |.  8B93 54050000        mov     edx, dword ptr [ebx+554]  

>   00536ABE    |.  E8 99D9ECFF          call    0040445C  

>   00536AC3    |.  33F6                 xor     esi, esi  

>   00536AC5    |.  8D55 FC              lea     edx, [local.1]  

>   00536AC8    |.  8B83 50050000        mov     eax, dword ptr [ebx+550]  

>   00536ACE    |.  E8 4121EDFF          call    00408C14  

>   00536AD3    |.  8D55 F8              lea     edx, [local.2]  

>   00536AD6    |.  8B83 54050000        mov     eax, dword ptr [ebx+554]  

>   00536ADC    |.  E8 3321EDFF          call    00408C14  

>   00536AE1    |.  8B45 F8              mov     eax, [local.2]  

>   00536AE4    |.  E8 ABDBECFF          call    00404694  

>   00536AE9    |.  83F8 16              cmp     eax, 16                             ;  注册码长度 22 位  

>   00536AEC        0F85 FE010000        jnz     00536CF0  

>   00536AF2    |.  8B45 F8              mov     eax, [local.2]  

>   00536AF5    |.  E8 9ADBECFF          call    00404694  

>   00536AFA    |.  83F8 16              cmp     eax, 16  

>   00536AFD    |.  0F85 ED010000        jnz     00536CF0  

>   00536B03    |.  8B45 F8              mov     eax, [local.2]  

>   00536B06    |.  E8 89DBECFF          call    00404694  

>   00536B0B    |.  83F8 16              cmp     eax, 16  

>   00536B0E    |.  0F85 DC010000        jnz     00536CF0  

>   00536B14    |.  8B45 F8              mov     eax, [local.2]  

>   00536B17    |.  E8 78DBECFF          call    00404694  

>   00536B1C    |.  83F8 16              cmp     eax, 16  

>   00536B1F    |.  0F85 CB010000        jnz     00536CF0  

>   00536B25    |.  8D4D EC              lea     ecx, [local.5]  

>   00536B28    |.  BA 03000000          mov     edx, 3  

>   00536B2D    |.  8B83 54050000        mov     eax, dword ptr [ebx+554]  

>   00536B33    |.  E8 E416F0FF          call    0043821C  

>   00536B38    |.  8B45 EC              mov     eax, [local.5]                      ;  注册码前三位是 ERA  

>   00536B3B    |.  BA 246D5300          mov     edx, 00536D24                       ;  ASCII "ERA"  

>   00536B40    |.  E8 9BDCECFF          call    004047E0  

>   00536B45    |.  0F85 A5010000        jnz     00536CF0  

>   00536B4B    |.  8B45 F8              mov     eax, [local.2]  

>   00536B4E    |.  E8 41DBECFF          call    00404694  

>   00536B53    |.  83F8 16              cmp     eax, 16  

>   00536B56    |.  0F85 94010000        jnz     00536CF0  

>   00536B5C    |.  8D4D E8              lea     ecx, [local.6]  

>   00536B5F    |.  BA 03000000          mov     edx, 3  

>   00536B64    |.  8B83 54050000        mov     eax, dword ptr [ebx+554]  

>   00536B6A    |.  E8 AD16F0FF          call    0043821C  

>   00536B6F    |.  8B45 E8              mov     eax, [local.6]  

>   00536B72    |.  BA 246D5300          mov     edx, 00536D24                       ;  ASCII "ERA"  

>   00536B77    |.  E8 64DCECFF          call    004047E0  

>   00536B7C    |.  0F85 6E010000        jnz     00536CF0  

>   00536B82    |.  8B45 F8              mov     eax, [local.2]  

>   00536B85    |.  E8 0ADBECFF          call    00404694  

>   00536B8A    |.  83F8 16              cmp     eax, 16  

>   00536B8D    |.  0F85 5D010000        jnz     00536CF0  

>   00536B93    |.  8D4D E4              lea     ecx, [local.7]  

>   00536B96    |.  BA 03000000          mov     edx, 3  

>   00536B9B    |.  8B83 54050000        mov     eax, dword ptr [ebx+554]  

>   00536BA1    |.  E8 7616F0FF          call    0043821C  

>   00536BA6    |.  8B45 E4              mov     eax, [local.7]  

>   00536BA9    |.  BA 246D5300          mov     edx, 00536D24                       ;  ASCII "ERA"  

>   00536BAE    |.  E8 2DDCECFF          call    004047E0  

>   00536BB3    |.  0F85 37010000        jnz     00536CF0  

>   00536BB9    |.  8D4D F8              lea     ecx, [local.2]  

>   00536BBC    |.  BA 13000000          mov     edx, 13  

>   00536BC1    |.  8B83 54050000        mov     eax, dword ptr [ebx+554]  

>   00536BC7    |.  E8 C016F0FF          call    0043828C  

>   00536BCC    |.  837D FC 00           cmp     [local.1], 0  

>   00536BD0    |.  0F84 1A010000        je      00536CF0  

>   00536BD6    |.  837D F8 00           cmp     [local.2], 0                        ;  取后面的 19 位注册码  

>   00536BDA    |.  0F84 10010000        je      00536CF0  

>   00536BE0    |.  33FF                 xor     edi, edi  

>   00536BE2    |>  8B45 F8              /mov    eax, [local.2]  

>   00536BE5    |.  8A1C38               |mov    bl, byte ptr [eax+edi]  

>   00536BE8    |.  80FB 2D              |cmp    bl, 2D                              ;  去掉注册码里面-  

>   00536BEB    |.  74 15                |je     short 00536C02  

>   00536BED    |.  8D45 E0              |lea    eax, [local.8]  

>   00536BF0    |.  8BD3                 |mov    edx, ebx  

>   00536BF2    |.  E8 B5D9ECFF          |call   004045AC  

>   00536BF7    |.  8B55 E0              |mov    edx, [local.8]  

>   00536BFA    |.  8D45 F4              |lea    eax, [local.3]  

>   00536BFD    |.  E8 9ADAECFF          |call   0040469C  

>   00536C02    |>  47                   |inc     edi  

>   00536C03    |.  83FF 13              |cmp    edi, 13                             ;  循环到 19 位整个字符串结束  

>   00536C06    |.^ 75 DA                \jnz     short 00536BE2  

>   00536C08    |.  8D45 F8              lea     eax, [local.2]  

>   00536C0B    |.  8B55 F4              mov     edx, [local.3]                      ;  edx=ACEFA8EAE21AD3FE  

>   00536C0E    |.  E8 49D8ECFF          call    0040445C  

>   00536C13    |.  8D45 F4              lea     eax, [local.3]  

>   00536C16    |.  E8 A9D7ECFF          call    004043C4  

>   00536C1B    |.  8B45 F8              mov     eax, [local.2]                      ;  eax=ACEFA8EAE21AD3FE  

>   00536C1E    |.  E8 71DAECFF          call    00404694  

>   00536C23    |.  83F8 10              cmp     eax, 10                             ;  看是否字符串长度为 16 位，那就是三个“-”  

>   00536C26        0F85 C4000000        jnz     00536CF0  

>   00536C2C    |.  33FF                 xor     edi, edi  

>   00536C2E    |>  8BC7                 /mov    eax, edi  

>   00536C30    |.  25 01000080          |and     eax, 80000001  

>   00536C35    |.  79 05                |jns     short 00536C3C  

>   00536C37    |.  48                   |dec     eax  
```



## 汇编代码分析

```
  00536C38    |.  83C8 FE              |or      eax, FFFFFFFE  
  00536C3B    |.  40                   |inc     eax  
  00536C3C    |>  85C0                 |test    eax, eax  
  00536C3E    |.  75 1B                |jnz     short 00536C5B  
  00536C40    |.  8D45 DC              |lea     eax, [local.9]  
  00536C43    |.  8B55 F8              |mov     edx, [local.2]  
  00536C46    |.  8A143A               |mov     dl, byte ptr [edx+edi]  
  00536C49    |.  E8 5ED9ECFF          |call    004045AC  
  00536C4E    |.  8B55 DC              |mov     edx, [local.9]  
  00536C51    |.  8D45 F0              |lea     eax, [local.4]  
  00536C54    |.  E8 43DAECFF          |call    0040469C  
  00536C59    |.  EB 19                |jmp     short 00536C74  
  00536C5B    |>  8D45 D8              |lea     eax, [local.10]  
  00536C5E    |.  8B55 F8              |mov     edx, [local.2]  
  00536C61    |.  8A143A               |mov     dl, byte ptr [edx+edi]  
  00536C64    |.  E8 43D9ECFF          |call    004045AC  
  00536C69    |.  8B55 D8              |mov     edx, [local.10]  
  00536C6C    |.  8D45 F4              |lea     eax, [local.3]  
  00536C6F    |.  E8 28DAECFF          |call    0040469C  
  00536C74    |>  47                   |inc     edi  
  00536C75    |.  83FF 10              |cmp     edi, 10  
  00536C78    |.^ 75 B4               \jnz     short 00536C2E  
  00536C7A    |.  8D45 FC              lea     eax, [local.1]  
  00536C7D    |.  E8 42D7ECFF          call    004043C4  
  00536C82    |.  33FF                 xor     edi, edi  
  00536C84    |>  8D45 D4              /lea    eax, [local.11]                    ; 偶数位字符串取反操作 CF8A2A3E 变成 E3A2A8FC  
  00536C87    |.  BA 08000000          |mov     edx, 8  
  00536C8C    |.  2BD7                 |sub     edx, edi  
  00536C8E    |.  8B4D F4              |mov     ecx, [local.3]  
  00536C91    |.  8A5411 FF            |mov     dl, byte ptr [ecx+edx-1]  
  00536C95    |.  E8 12D9ECFF          |call    004045AC  
  00536C9A    |.  8B55 D4              |mov     edx, [local.11]  
  00536C9D    |.  8D45 FC              |lea     eax, [local.1]  
  00536CA0    |.  E8 F7D9ECFF          |call    0040469C  
  00536CA5    |.  47                   |inc     edi  
  00536CA6    |.  83FF 08              |cmp     edi, 8  
  00536CA9    |.^ 75 D9               \jnz     short 00536C84  
  00536CAB    |.  8D55 D0              lea     edx, [local.12]  
  00536CAE    |.  8B45 F0              mov     eax, [local.4]                    ; eax=AEAEE1DF 取奇数位 8 位  
  00536CB1    |.  E8 BE96FFFF          call    00530374                         ; 奇数位进入 MD5 运算  
  00536CB6    |.  8B55 D0              mov     edx, [local.12]                  ; edx=D7419D0CCAFCC752DBCBAEABE3A2A8FC MD5 运算结果  
  00536CB9    |.  8D45 F0              lea     eax, [local.4]  
  00536CBC    |.  E8 9BD7ECFF          call    0040445C  
  00536CC1    |.  8D4D CC              lea     ecx, [local.13]  
  00536CC4    |.  BA 08000000          mov     edx, 8                           ; 取 MD5 运算结果后 8 位 E3A2A8FC  
  00536CC9    |.  8B45 F0              mov     eax, [local.4]  
  00536CCC    |.  E8 3F16F0FF         call    00438310  
  00536CD1    |.  8B55 CC              mov     edx, [local.13]  
  00536CD4    |.  8D45 F0              lea     eax, [local.4]  
  00536CD7    |.  E8 80D7ECFF          call    0040445C  
  00536CDC    |.  8B45 FC              mov     eax, [local.1]                   ; 后八位与偶数位取反进行比较，相等就注册成功  
  00536CDF    |.  8B55 F0              mov     edx, [local.4]  
  00536CE2    |.  E8 F9DAECFF          call    004047E0  
  00536CE7    |.  75 05                jnz     short 00536CEE  
  00536CE9    |.  83CE FF              or      esi, FFFFFFFF  
  00536CEC    |.  EB 02                jmp     short 00536CF0  
  00536CEE    |>  33F6                 xor     esi, esi  
  00536CF0    |>  33C0                 xor     eax, eax  
  00536CF2    |.  5A                   pop     edx  
  00536CF3    |.  59                   pop     ecx  
  00536CF4    |.  59                   pop     ecx  
  00536CF5    |.  64:8910             mov     dword ptr fs:[eax], edx  
  00536CF8    |.  68 126D5300          push    00536D12  
  00536CFD    |>  8D45 CC              lea     eax, [local.13]  
  00536D00    |.  BA 0D000000          mov     edx, 0D  
  00536D05    |.  E8 DED6ECFF         call    004043E8  
  00536D0A    \\.  C3                  retn  
```

## 经验总结

* 注册码长度位 22 位，前三位为固定的 `ERA`，后面 19 位里面有 `-` 三个
* 然后取去掉 `-` 的 16 位长度的字符串，按偶数位和奇数位取成两个八位字符串
* 注册算法：偶数位字符串取反 = `MD5`(奇数位）的后面八位
* 虽然这个软件用到加密算法 `MD5`，并且还不是完全明码比较，不过注册流程太清楚，相对任意跟踪出注册码

一组 key：`ERA---ACEFA8EAE21AD3FE`

最后的注册码保存在 `strings.ini` 文件里面

```ini
[Common]
Name=zeromk2@crsky.com
Code=ERA---ACEFA8EAE21AD3FE
```

## 注册机代码

花了几分钟补了个注册机代码

`VB` 注册机源代码，其中 `MD5` 代码来自网络，被放在唯一的模块里面

```vb
Option Explicit

Private Sub Command1_Click()
Dim ser(15) As String
Dim i As Integer, j As Integer, key1 As String, key2 As String, key3 As String
For i = 0 To 15
    Randomize
    ser(i) = Chr(Int(Rnd * (90 - 65 + 1) + 65))
    'Print ser(i)
Next i
For i = 0 To 7
    key1 = key1 & ser(i * 2)
Next i
'key1 = "AEAEE1DF"
'Print key1
key1 = Md5_String_Calc(key1)
'Print key1
key2 = UCase(Mid(key1, (Len(key1) - 7), 8))
'Print key2
ser(1) = Mid(key2, 8, 1)
ser(3) = Mid(key2, 7, 1)
ser(5) = Mid(key2, 6, 1)
ser(7) = Mid(key2, 5, 1)
ser(9) = Mid(key2, 4, 1)
ser(11) = Mid(key2, 3, 1)
ser(13) = Mid(key2, 2, 1)
ser(15) = Mid(key2, 1, 1)
For i = 0 To 15
    key3 = key3 & ser(i)
Next i
Text1.Text = "ERA---" & key3
Clipboard.Clear
Clipboard.SetText Text1.Text
End Sub

Private Sub Command2_Click()
End
End Sub
```

### MD5 模块源代码

```vb
Option Explicit

Private Const OFFSET_4 = 4294967296#
Private Const MAXINT_4 = 2147483647
Private State(4) As Long
Private ByteCounter As Long
Private ByteBuffer(63) As Byte
Private Const S11 = 7
Private Const S12 = 12
Private Const S13 = 17
Private Const S14 = 22
Private Const S21 = 5
Private Const S22 = 9
Private Const S23 = 14
Private Const S24 = 20
Private Const S31 = 4
Private Const S32 = 11
Private Const S33 = 16
Private Const S34 = 23
Private Const S41 = 6
Private Const S42 = 10
Private Const S43 = 15
Private Const S44 = 21

Property Get RegisterA() As String
RegisterA = State(1)
End Property

Property Get RegisterB() As String
RegisterB = State(2)
End Property

Property Get RegisterC() As String
RegisterC = State(3)
End Property

Property Get RegisterD() As String
RegisterD = State(4)
End Property

Public Function Md5_String_Calc(SourceString As String) As String
MD5Init
MD5Update LenB(StrConv(SourceString, vbFromUnicode)), StringToArray(SourceString)
MD5Final
Md5_String_Calc = GetValues
End Function

Public Function Md5_File_Calc(InFile As String) As String
On Error GoTo errorhandler
GoSub begin

errorhandler:
Md5_File_Calc = ""
Exit Function

begin:
Dim FileO As Integer
FileO = FreeFile
Call FileLen(InFile)
Open InFile For Binary Access Read As #FileO
MD5Init
Do While Not EOF(FileO)
Get #FileO, , ByteBuffer
```



```vb
    If Loc(FileO) < LOF(FileO) Then
        ByteCounter = ByteCounter + 64
        MD5Transform ByteBuffer
    End If
    
    Loop
    
    ByteCounter = ByteCounter + (LOF(FileO) Mod 64)
    
    Close #FileO
    
    MD5Final
    
    Md5_File_Calc = GetValues
    
End Function

Private Function StringToArray(InString As String) As Byte()
    Dim i As Integer, bytBuffer() As Byte
    
    ReDim bytBuffer(LenB(StrConv(InString, vbFromUnicode)))
    
    bytBuffer = StrConv(InString, vbFromUnicode)
    
    StringToArray = bytBuffer
    
End Function

Public Function GetValues() As String
    GetValues = LongToString(State(1)) & LongToString(State(2)) & LongToString(State(3)) & LongToString(State(4))
End Function

Private Function LongToString(Num As Long) As String
    Dim A As Byte, B As Byte, C As Byte, d As Byte
    
    A = Num And &HFF&
    If A < 16 Then LongToString = "0" & Hex(A) Else LongToString = Hex(A)
    
    B = (Num And &HFF00&) \ 256
    If B < 16 Then LongToString = LongToString & "0" & Hex(B) Else LongToString = LongToString & Hex(B)
    
    C = (Num And &HFF0000) \ 65536
    If C < 16 Then LongToString = LongToString & "0" & Hex(C) Else LongToString = LongToString & Hex(C)
    
    If Num < 0 Then d = ((Num And &H7F000000) \ 16777216) Or &H80& Else d = (Num And &HFF000000) \ 16777216
    
    If d < 16 Then LongToString = LongToString & "0" & Hex(d) Else LongToString = LongToString & Hex(d)
    
End Function

Public Sub MD5Init()
    ByteCounter = 0
    
    State(1) = UnsignedToLong(1732584193#)
    State(2) = UnsignedToLong(4023233417#)
    State(3) = UnsignedToLong(2562383102#)
    State(4) = UnsignedToLong(271733878#)
    
End Sub

Public Sub MD5Final()
    Dim dblBits As Double, padding(72) As Byte, lngBytesBuffered As Long
    
    padding(0) = &H80
    
    dblBits = ByteCounter * 8
    
    lngBytesBuffered = ByteCounter Mod 64
    
    If lngBytesBuffered <= 56 Then MD5Update 56 - lngBytesBuffered, padding Else MD5Update 120 - ByteCounter, padding
    
    padding(0) = UnsignedToLong(dblBits) And &HFF&
    padding(1) = UnsignedToLong(dblBits) \ 256 And &HFF&
    padding(2) = UnsignedToLong(dblBits) \ 65536 And &HFF&
    padding(3) = UnsignedToLong(dblBits) \ 16777216 And &HFF&
    padding(4) = 0
    padding(5) = 0
    padding(6) = 0
    padding(7) = 0
    
    MD5Update 8, padding
    
End Sub

Public Sub MD5Update(InputLen As Long, InputBuffer() As Byte)
    Dim II As Integer, i As Integer, j As Integer, K As Integer, lngBufferedBytes As Long, lngBufferRemaining As Long, lngRem As Long
    
    lngBufferedBytes = ByteCounter Mod 64
    lngBufferRemaining = 64 - lngBufferedBytes
    ByteCounter = ByteCounter + InputLen
    
    If InputLen >= lngBufferRemaining Then
        For II = 0 To lngBufferRemaining - 1
            ByteBuffer(lngBufferedBytes + II) = InputBuffer(II)
        Next II
        MD5Transform ByteBuffer
        lngRem = (InputLen) Mod 64
        For i = lngBufferRemaining To InputLen - II - lngRem Step 64
            For j = 0 To 63
                ByteBuffer(j) = InputBuffer(i + j)
            Next j
            MD5Transform ByteBuffer
        Next i
        lngBufferedBytes = 0
    Else
        i = 0
    End If
    
    For K = 0 To InputLen - i - 1
        ByteBuffer(lngBufferedBytes + K) = InputBuffer(i + K)
    Next K
    
End Sub

Private Sub MD5Transform(Buffer() As Byte)
    Dim X(16) As Long, A As Long, B As Long, C As Long, d As Long
    
    A = State(1)
    B = State(2)
    C = State(3)
    d = State(4)
    
    Decode 64, X, Buffer
    
    FF A, B, C, d, X(0), S11, -680876936
    FF d, A, B, C, X(1), S12, -389564586
    FF C, d, A, B, X(2), S13, 606105819
    FF B, C, d, A, X(3), S14, -1044525330
    FF A, B, C, d, X(4), S11, -176418897
    FF d, A, B, C, X(5), S12, 1200080426
    FF C, d, A, B, X(6), S13, -1473231341
    FF B, C, d, A, X(7), S14, -45705983
    FF A, B, C, d, X(8), S11, 1770035416
    FF d, A, B, C, X(9), S12, -1958414417
    FF C, d, A, B, X(10), S13, -42063
    FF B, C, d, A, X(11), S14, -1990404162
    FF A, B, C, d, X(12), S11, 1804603682
    FF d, A, B, C, X(13), S12, -40341101
    FF C, d, A, B, X(14), S13, -1502002290
    FF B, C, d, A, X(15), S14, 1236535329
    
    GG A, B, C, d, X(1), S21, -165796510
    GG d, A, B, C, X(6), S22, -1069501632
    GG C, d, A, B, X(11), S23, 643717713
    GG B, C, d, A, X(0), S24, -373897302
    GG A, B, C, d, X(5), S21, -701558691
    GG d, A, B, C, X(10), S22, 38016083
    GG C, d, A, B, X(15), S23, -660478335
    GG B, C, d, A, X(4), S24, -405537848
    GG A, B, C, d, X(9), S21, 568446438
    GG d, A, B, C, X(14), S22, -1019803690
    GG C, d, A, B, X(3), S23, -187363961
    GG B, C, d, A, X(8), S24, 1163531501
    GG A, B, C, d, X(13), S21, -1444681467
    GG d, A, B, C, X(2), S22, -51403784
    GG C, d, A, B, X(7), S23, 1735328473
    GG B, C, d, A, X(12), S24, -1926607734
    
    HH A, B, C, d, X(5), S31, -378558
    HH d, A, B, C, X(8), S32, -2022574463
    HH C, d, A, B, X(11), S33, 1839030562
    HH B, C, d, A, X(14), S34, -35309556
    HH A, B, C, d, X(1), S31, -1530992060
    HH d, A, B, C, X(4), S32, 1272893353
    HH C, d, A, B, X(7), S33, -155497632
    HH B, C, d, A, X(10), S34, -1094730640
    HH A, B, C, d, X(13), S31, 681279174
    HH d, A, B, C, X(0), S32, -358537222
    HH C, d, A, B, X(3), S33, -722521979
    HH B, C, d, A, X(6), S34, 76029189
    HH A, B, C, d, X(9), S31, -640364487
    HH d, A, B, C, X(12), S32, -421815835
    HH C, d, A, B, X(15), S33, 530742520
    HH B, C, d, A, X(2), S34, -995338651
    
    II A, B, C, d, X(0), S41, -198630844
    II d, A, B, C, X(7), S42, 1126891415
    II C, d, A, B, X(14), S43, -1416354905
    II B, C, d, A, X(5), S44, -57434055
    II A, B, C, d, X(12), S41, 1700485571
    II d, A, B, C, X(3), S42, -1894986606
    II C, d, A, B, X(10), S43, -1051523
    II B, C, d, A, X(1), S44, -2054922799
    II A, B, C, d, X(8), S41, 1873313359
    II d, A, B, C, X(15), S42, -30611744
    II C, d, A, B, X(6), S43, -1560198380
    II B, C, d, A, X(13), S44, 1309151649
    II A, B, C, d, X(4), S41, -145523070
    II d, A, B, C, X(11), S42, -1120210379
    II C, d, A, B, X(2), S43, 718787259
    II B, C, d, A, X(9), S44, -343485551
    
    State(1) = LongOverflowAdd(State(1), A)
    State(2) = LongOverflowAdd(State(2), B)
    State(3) = LongOverflowAdd(State(3), C)
    State(4) = LongOverflowAdd(State(4), d)
    
End Sub

Private Sub Decode(Length As Integer, OutputBuffer() As Long, InputBuffer() As Byte)
    Dim intDblIndex As Integer, intByteIndex As Integer, dblSum As Double
    For intByteIndex = 0 To Length - 1 Step 4
        dblSum = InputBuffer(intByteIndex) + InputBuffer(intByteIndex + 1) * 256# + InputBuffer(intByteIndex + 2) * 65536# + InputBuffer(intByteIndex + 3) * 16777216#
        OutputBuffer(intDblIndex) = UnsignedToLong(dblSum)
        intDblIndex = intDblIndex + 1
    Next intByteIndex
End Sub

Private Function FF(A As Long, B As Long, C As Long, d As Long, X As Long, S As Long, ac As Long) As Long
    A = LongOverflowAdd4(A, (B And C) Or (Not (B) And d), X, ac)
    A = LongLeftRotate(A, S)
    A = LongOverflowAdd(A, B)
End Function

Private Function GG(A As Long, B As Long, C As Long, d As Long, X As Long, S As Long, ac As Long) As Long
    A = LongOverflowAdd4(A, (B And d) Or (C And Not (d)), X, ac)
    A = LongLeftRotate(A, S)
    A = LongOverflowAdd(A, B)
End Function

Private Function HH(A As Long, B As Long, C As Long, d As Long, X As Long, S As Long, ac As Long) As Long
    A = LongOverflowAdd4(A, B Xor C Xor d, X, ac)
    A = LongLeftRotate(A, S)
    A = LongOverflowAdd(A, B)
End Function

Private Function II(A As Long, B As Long, C As Long, d As Long, X As Long, S As Long, ac As Long) As Long
    A = LongOverflowAdd4(A, C Xor (B Or Not (d)), X, ac)
    A = LongLeftRotate(A, S)
    A = LongOverflowAdd(A, B)
End Function

Function LongLeftRotate(value As Long, Bits As Long) As Long
```



```vb
Dim lngSign As Long, lngI As Long

Bits = Bits Mod 32

If Bits = 0 Then LongLeftRotate = value: Exit Function

For lngI = 1 To Bits
    lngSign = value And &HC0000000
    value = (value And &H3FFFFFFF) * 2
    value = value Or ((lngSign < 0) And 1) Or (CBool(lngSign And &H40000000) And &H80000000)
Next

LongLeftRotate = value

End Function

Private Function LongOverflowAdd(Val1 As Long, Val2 As Long) As Long
    Dim lngHighWord As Long, lngLowWord As Long, lngOverflow As Long
    
    lngLowWord = (Val1 And &HFFFF&) + (Val2 And &HFFFF&)
    lngOverflow = lngLowWord \ 65536
    lngHighWord = (((Val1 And &HFFFF0000) \ 65536) + ((Val2 And &HFFFF0000) \ 65536) + lngOverflow) And &HFFFF&
    
    LongOverflowAdd = UnsignedToLong((lngHighWord * 65536#) + (lngLowWord And &HFFFF&))
    
End Function

Private Function LongOverflowAdd4(Val1 As Long, Val2 As Long, val3 As Long, val4 As Long) As Long
    Dim lngHighWord As Long, lngLowWord As Long, lngOverflow As Long
    
    lngLowWord = (Val1 And &HFFFF&) + (Val2 And &HFFFF&) + (val3 And &HFFFF&) + (val4 And &HFFFF&)
    lngOverflow = lngLowWord \ 65536
    lngHighWord = (((Val1 And &HFFFF0000) \ 65536) + ((Val2 And &HFFFF0000) \ 65536) + ((val3 And &HFFFF0000) \ 65536) + ((val4 And &HFFFF0000) \ 65536) + lngOverflow) And &HFFFF&
    
    LongOverflowAdd4 = UnsignedToLong((lngHighWord * 65536#) + (lngLowWord And &HFFFF&))
    
End Function

Private Function UnsignedToLong(value As Double) As Long
    If value < 0 Or value >= OFFSET_4 Then Error 6
    If value <= MAXINT_4 Then UnsignedToLong = value Else UnsignedToLong = value - OFFSET_4
End Function

Private Function LongToUnsigned(value As Long) As Double
    If value < 0 Then LongToUnsigned = value + OFFSET_4 Else LongToUnsigned = value
End Function
```

> 【版权声明】: 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!
> 2007 年 10 月 17 日 PM 09:23:17

`4dd466d0e1e0cc123afbe504da2462ca`
