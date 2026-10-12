
## 循环分析（Loop 3 & 4）

```
0050224C  |.  33F6         |xor     esi, esi                     ; //若相等则 ESI 清零，进行第二轮交换
0050224E  |>  47          |inc     edi                          ; //EDI+1
0050224F  |.  3BFD        |cmp     edi, ebp                     ; //标志位与用户名长度比较
00502251  |.^ 72 DF       \jb     short 00502232               ; //此循环（循环 3）功能同循环 1，从固定值的第 11 位开始替换
00502253  |>  33FF        xor     edi, edi                       ; //EDI 清零
00502255  |.  33C9        xor     ecx, ecx                       ; //ECX 清零
00502257  |.  85ED        test    ebp, ebp                       ; //用户名是否为空
00502259  |.  76 26       jbe     short 00502281                 ; //为空则跳
0050225B  |>  8A9F 9F087300 /mov   bl, byte ptr [edi+73089F]     ; //依次取固定值送给 BL，从固定值第 16 位开始取值
00502261  |.  8BF5        |mov     esi, ebp                       ; //用户名长度送给 ESI
00502263  |.  2BF1        |sub     esi, ecx                       ; //ESI-EBP，EBP 初始值为零
00502265  |.  4E          |dec     esi                            ; //再-1，这三行代码用来从倒数第一位依次往前取值
00502266  |.  8A0416      |mov     al, byte ptr [esi+edx]         ; //依次取循环 3 后的用户名 ASCII 值送给 AL，从后往前取值
00502269  |.  32D8        |xor     bl, al                         ; //ASCII 值与固定值异或运算
0050226B  |.  47          |inc     edi                            ; //EDI+1，用于比较是否到了第五位以便进行第二轮交换
0050226C  |.  881C16      |mov     byte ptr [esi+edx], bl
0050226F  |.  8887 9E087300 |mov    byte ptr [edi+73089E], al     ; //这两行代码进行值的交换
00502275  |.  83FF 05     |cmp     edi, 5                         ; //EDI 与 5 比较
00502278  |.  75 02       |jnz     short 0050227C                 ; //不等则跳
0050227A  |.  33FF        |xor     edi, edi                       ; //若相等则 EDI 清零，进行第二轮交换
0050227C  |>  41          |inc     ecx                            ; //ECX+1
0050227D  |.  3BCD        |cmp     ecx, ebp                       ; //与长度比较
0050227F  |.^ 72 DA       \jb     short 0050225B                 ; //此循环（循环 4）功能同循环 2，从固定值第 16 位开始交换
00502281  |>  8B7C24 18   mov     edi, dword ptr [esp+18]        ; //堆栈地址 F5A0 送给 EDI
00502285  |.  33C0        xor     eax, eax                       ; //EAX 清零
00502287  |.  85ED        test    ebp, ebp                       ; //较验用户名是否为空
00502289  |.  C707 00000000 mov    dword ptr [edi], 0            ; //堆栈清零，用于存储 4 次循环后的前 4 位值的累加值
0050228F  |.  76 17       jbe     short 005022A8                 ; //为空则跳
00502291  |>  8BC8        /mov     ecx, eax                       ; //ECX=EAX（标志位）
00502293  |.  83E1 03     |and     ecx, 3                         ; //ECX AND 3，（4 位一循环）
00502296  |.  8A1C39      |mov     bl, byte ptr [ecx+edi]         ; //依次送每一位累加值给 BL
00502299  |.  8D3439      |lea     esi, dword ptr [ecx+edi]       ; //存储地址送给 ESI
0050229C  |.  8A0C10      |mov     cl, byte ptr [eax+edx]         ; //依次取用户名 4 次循环交换后的值给 CL
0050229F  |.  02D9        |add     bl, cl                         ; //累加
005022A1  |.  40          |inc     eax                            ; //EAX+1，用于取下一位
005022A2  |.  3BC5        |cmp     eax, ebp                       ; //与用户名比较
005022A4  |.  881E        |mov     byte ptr [esi], bl             ; //相加值送入堆栈中
005022A6  |.^ 72 E9       \jb     short 00502291                 ; //未完继续（4 位一循环，对前面几次循环的值进行累加）
005022A8  |>  5F          pop     edi
005022A9  |.  5E          pop     esi
005022AA  |.  5D          pop     ebp
005022AB  |.  5B          pop     ebx
005022AC  \.  C3          retn
```

## 三、由 `00566C02` 调用 `call 00592283` ; //注册码转换为十六进制值函数

```
00592283  /$  53          push    ebx                      ; //这个函数用于将注册码转换为十六进制值
00592284  |.  55          push    ebp
00592285  |.  56          push    esi
00592286  |.  57          push    edi
00592287  |.  8B7C24 14   mov     edi, dword ptr [esp+14]  ; //注册码送给 EDI
0059228B  |>  833D 8C727100>/cmp    dword ptr [71728C], 1   ; //[71728C]中的值（初值为 1）与 1 比较
00592292  |.  7E 0F       |jle     short 005922A3           ; //小于等于则跳
00592294  |.  0FB607      |movzx   eax, byte ptr [edi]
00592297  |.  6A 08       |push    8
00592299  |.  50          |push    eax
0059229A  |.  E8 7C240000 |call    0059471B
0059229F  |.  59          |pop     ecx
005922A0  |.  59          |pop     ecx
005922A1  |.  EB 0F       |jmp     short 005922B2
005922A3  |>  0FB607      |movzx   eax, byte ptr [edi]      ; //依次取注册码 ASCII 值送给 EAX
005922A6  |.  8B0D 80707100 |mov   ecx, dword ptr [717080]  ; //[717080]中的值（字符表地址）送给 ECX
005922AC  |.  8A0441      |mov     al, byte ptr [ecx+eax*2]  ; //取字符表中的值送给 AL
005922AF  |.  83E0 08     |and     eax, 8                     ; //EAX AND 8（无效地址：709C-90A4、70CA）
005922B2  |>  85C0        |test    eax, eax                   ; //是否为空（无效 ASCII 值为 9-D、20）
005922B4  |.  74 03       |je      short 005922B9             ; //为空则跳（只要第一个为有效字符就跳出循环，否则清除）
005922B6  |.  47          |inc     edi                        ; //取下一位
005922B7  |.^ EB D2       \jmp    short 0059228B             ; //这一个循环是较验注册码字符的有效性，把开始处的无效字符清除
005922B9  |>  0FB637      movzx   esi, byte ptr [edi]        ; //取注册码第一位 ASCII 值送给 ESI
005922BC  |.  47          inc     edi                         ; //EDI 加 1
005922BD  |.  83FE 2D     cmp     esi, 2D                     ; //ASCII 值与 2D（-）比较
005922C0  |.  8BEE        mov     ebp, esi                    ; //ASCII 值送给 EBP
005922C2  |.  74 05       je      short 005922C9              ; //相等则跳
005922C4  |.  83FE 2B     cmp     esi, 2B                     ; //ASCII 值与 2B（+）比较
005922C7  |.  75 04       jnz     short 005922CD              ; //不等则跳
005922C9  |>  0FB637      movzx   esi, byte ptr [edi]         ; //若相等，则 ASCII 值送给 ESI
005922CC  |.  47          inc     edi                         ; //EDI+1，取下一位
005922CD  |>  33DB        xor     ebx, ebx                    ; //不为（+）号跳来此处，EBX 清零
005922CF  |>  833D 8C727100>/cmp    dword ptr [71728C], 1     ; //[71728C]中的值（初值为 1）与 1 比较
005922D6  |.  7E 0C       |jle     short 005922E4             ; //小于等于则跳
005922D8  |.  6A 04       |push    4
005922DA  |.  56          |push    esi
005922DB  |.  E8 3B240000 |call    0059471B
005922E0  |.  59          |pop     ecx
005922E1  |.  59          |pop     ecx
005922E2  |.  EB 0B       |jmp     short 005922EF
005922E4  |>  A1 80707100  |mov     eax, dword ptr [717080]   ; //[717080]中的值（固定值）送给 EAX
005922E9  |.  8A0470      |mov     al, byte ptr [eax+esi*2]   ; //字符表中的值送给 AL
005922EC  |.  83E0 04     |and     eax, 4                     ; //EAX AND 4（有效字符表：70EA-70FC）
005922EF  |>  85C0        |test    eax, eax                    ; //是否为空（有效 ASCII 值：30-39，说明注册码必为数字）
005922F1  |.  74 0D       |je      short 00592300             ; //为空则跳，跳了就挂了
005922F3  |.  8D049B      |lea     eax, dword ptr [ebx+ebx*4] ; //EAX=EBX*5（EBX 初始值为 0）
005922F6  |.  8D5C46 D0   |lea     ebx, dword ptr [esi+eax*2-30] ; //EBX=ESI+EAX*2-30 （ESI 为注册码的 ASCII 值）
005922FA  |.  0FB637      |movzx   esi, byte ptr [edi]        ; //注册码 ASCII 值送给 ESI
005922FD  |.  47          |inc     edi                        ; //EDI+1，取下一位注册码
005922FE  |.^ EB CF       \jmp    short 005922CF             ; //字符串有效性较验的循环
00592300  |>  83FD 2D     cmp     ebp, 2D                     ; //ASCII 值与 2D（-）比较
00592303  |.  8BC3        mov     eax, ebx                    ; //EAX=EBX=0
```
