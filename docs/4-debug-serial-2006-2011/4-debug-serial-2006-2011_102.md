
# Panorado 3.3.1.119

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-475.htm`](https://www.pediy.com/kssd/pediy08/pediy8-475.htm)

**【破文标题】** 菜鸟 maomaoma 的算法练习破文五  
**【破文作者】** maomaoma  
**【作者邮箱】**  
**【作者主页】** 无  
**【破解工具】** `OD`、`PEiD`  
**【破解平台】** `winxp`  
**【软件名称】** `Panorado 3.3.1.119`  
**【软件大小】** `1808KB`  
**【原版下载】** `http://ks.onlinedown.net/soft/45759.htm`  
**【保护方式】** 无  
**【软件简介】** 一个特别为非常大的图片和全景图片而设计的图片查看器。它支持 360 度的查看、浏览、缩略图查看、全屏模式、幻灯片播放和打印功能。  
**【破解声明】** 我是菜鸟，学写破文，还请大侠多多指教：）

------------------------------------------------------------------------

### 【破解过程】

1. `PEiD` 查主程序无壳，`Microsoft Visual C++ 6.0` 编译
2. `OD` 载入，`ctrl+N` 在 `USER32.GetWindowTextW` 下断点，`F9` 运行，输入 e-mail 地址：`123@1.com`，及假码：`12345678`，点注册，程序断下

```asm
00445770   $   53              push    ebx
00445771   |.  55              push    ebp
00445772   |.  56              push    esi
00445773   |.  57              push    edi
00445774   |.  E8 77D3FFFF     call    00442AF0
00445779   |.  85C0            test    eax, eax
0044577B   |.  74 1A           je      short 00445797
0044577D   |.  8B4424 1C       mov     eax, [esp+1C]
00445781   |.  8B4C24 18       mov     ecx, [esp+18]
00445785   |.  8B5424 14       mov     edx, [esp+14]
00445789   |.  50              push    eax               ; /Count
0044578A   |.  51              push    ecx               ; |Buffer
0044578B   |.  52              push    edx               ; |hWnd
0044578C   |.  FF15 8C354D00   call    [<&USER32.GetWindowTextW>]       ; \GetWindowTextW \\OD 断在此处
00445792   |.  5F              pop     edi
00445793   |.  5E              pop     esi
00445794   |.  5D              pop     ebp
00445795   |.  5B              pop     ebx
00445796   |.  C3              retn
```

`F8` 单步至

```asm
00425B82    .   8D4424 18       lea     eax, [esp+18]
00425B86    .   50              push    eax
00425B87    .   68 9E070000     push    79E
00425B8C    .   57              push    edi
00425B8D    .   E8 EEFE0100     call    00445A80                  ;
00425B92    .   8D4C24 38       lea     ecx, [esp+38]
00425B96    .   51              push    ecx
```

向上至

```asm
00425B69    > \ 8B7C24 18       mov     edi, [esp+18]             ;  Case 1 of switch 00425B11
00425B6D    .   8D5424 08       lea     edx, [esp+8]
00425B71    .   52              push    edx
00425B72    .   68 D9040000     push    4D9
00425B77    .   57              push    edi
00425B78    .   BE 05400080     mov     esi, 80004005
00425B7D    .   E8 FEFE0100     call    00445A80                  ;
00425B82    .   8D4424 18       lea     eax, [esp+18]
00425B86    .   50              push    eax
```

在

```asm
00425B69    > \ 8B7C24 18       mov     edi, [esp+18]
```

处重新下断点，`ctrl+F2` 重新载入程序，`F9` 运行，输入

e-mail 地址：`123@1.com`，及假码：`12345678`，点注册，程序断下

```asm
00425B69    > \ 8B7C24 18       mov     edi, [esp+18]             ;  Case 1 of switch 00425B11  \\OD 断在此处
00425B6D    .   8D5424 08       lea     edx, [esp+8]
00425B71    .   52              push    edx
00425B72    .   68 D9040000     push    4D9
00425B77    .   57              push    edi
00425B78    .   BE 05400080     mov     esi, 80004005
00425B7D    .   E8 FEFE0100     call    00445A80                  ;  取 e-mail 地址
00425B82    .   8D4424 18       lea     eax, [esp+18]
00425B86    .   50              push    eax
00425B87    .   68 9E070000     push    79E
00425B8C    .   57              push    edi
00425B8D    .   E8 EEFE0100     call    00445A80                  ;  取假码
00425B92    .   8D4C24 38       lea     ecx, [esp+38]
00425B96    .   51              push    ecx
00425B97    .   8D5424 38       lea     edx, [esp+38]
00425B9B    .   52              push    edx
00425B9C    .   8B5424 2C       mov     edx, [esp+2C]
00425BA0    .   8D4424 38       lea     eax, [esp+38]
00425BA4    .   50              push    eax
00425BA5    .   8D4C24 48       lea     ecx, [esp+48]
00425BA9    .   51              push    ecx
00425BAA    .   52              push    edx
00425BAB    .   E8 C0030000     call    00425F70                  ;  注册码合法性验证 call，跟进（1）
00425BB0    .   83C4 2C         add     esp, 2C
00425BB3    .   85C0            test    eax, eax
00425BB5    .   0F84 E5000000   je      00425CA0
00425BBB    .   8B4C24 08       mov     ecx, [esp+8]
00425BBF    .   8D4424 10       lea     eax, [esp+10]
00425BC3    .   50              push    eax
00425BC4    .   51              push    ecx
00425BC5    .   E8 36F7FFFF     call    00425300                  ;  e-mail 地址验证及变换 call,跟进（2）
00425BCA    .   83C4 08         add     esp, 8
00425BCD    .   85C0            test    eax, eax
00425BCF    .   0F84 CB000000   je      00425CA0
00425BD5    .   8B4424 24       mov     eax, [esp+24]
00425BD9    .   394424 10       cmp     [esp+10], eax             ;  注册码第一部分应为 e-mail 变换结果（我这里是 0CECD864）
00425BDD    .   0F85 BD000000   jnz     00425CA0                  ;  不等则跳
00425BE3    .   8B5424 1C       mov     edx, [esp+1C]             ;  假码第三部分入 edx
00425BE7    .   8B4C24 18       mov     ecx, [esp+18]             ;  假码第二部分入 ecx
00425BEB    .   52              push    edx
00425BEC    .   51              push    ecx
00425BED    .   50              push    eax
00425BEE    .   E8 6DF7FFFF     call    00425360                  ;  注册码第四部分验证 call，返回值作为真码保留在 eax 中，跟进（3）
00425BF3    .   8B4C24 2C       mov     ecx, [esp+2C]             ;  假码第四部分入 ecx
00425BF7    .   83C4 0C         add     esp, 0C
00425BFA    .   3BC1            cmp     eax, ecx
00425BFC    .   0F85 9E000000   jnz     00425CA0
00425C02    .   8B4424 18       mov     eax, [esp+18]             ;  假码第二部分入 eax
00425C06    .   3D 1AA9E111     cmp     eax, 11E1A91A             ;  注册码第二部分为 11E1A91A
00425C0B    .   74 15           je      short 00425C22
00425C0D    .   3D 00000502     cmp     eax, 2050000              ;  注册码第二部分或为 2050000
00425C12    .   74 0E           je      short 00425C22
00425C14    .   3D 00000003     cmp     eax, 3000000              ;  注册码第二部分或为 3000000
00425C19    .   74 07           je      short 00425C22
00425C1B    .   3D 00000303     cmp     eax, 3030000              ;  注册码第二部分或为 3030000
00425C20    .   75 7E           jnz     short 00425CA0
00425C22    >   8B15 C4615000   mov     edx, [5061C4]
00425C28    .   8B4424 08       mov     eax, [esp+8]
00425C2C    .   6A 40           push    40
00425C2E    .   83C2 04         add     edx, 4
00425C31    .   52              push    edx
00425C32    .   50              push    eax
00425C33    .   E8 48D70100     call    00443380
00425C38    .   8D4C24 14       lea     ecx, [esp+14]
00425C3C    .   51              push    ecx
00425C3D    .   E8 9ED70100     call    004433E0
00425C42    .   8D5424 1C       lea     edx, [esp+1C]
00425C46    .   52              push    edx
00425C47    .   E8 94D70100     call    004433E0
00425C4C    .   A1 C4615000     mov     eax, [5061C4]
00425C51    .   8B4C24 2C       mov     ecx, [esp+2C]
00425C55    .   8988 84000000   mov     [eax+84], ecx
00425C5B    .   8B15 C4615000   mov     edx, [5061C4]
00425C61    .   8B4424 30       mov     eax, [esp+30]
00425C65    .   8982 88000000   mov     [edx+88], eax
00425C6B    .   8B0D C4615000   mov     ecx, [5061C4]
00425C71    .   8B5424 34       mov     edx, [esp+34]
00425C75    .   83C4 14         add     esp, 14
00425C78    .   8991 8C000000   mov     [ecx+8C], edx
00425C7E    .   E8 BDF8FFFF     call    00425540
00425C83    .   8B0D C4615000   mov     ecx, [5061C4]
00425C89    .   8981 90000000   mov     [ecx+90], eax
00425C8F    .   8B15 C4615000   mov     edx, [5061C4]
00425C95    .   66:C782 94000>mov      word ptr [edx+94], 0
00425C9E    .   33F6            xor     esi, esi
00425CA0    >   8D4424 08       lea     eax, [esp+8]
00425CA4    .   50              push    eax
00425CA5    .   E8 36D70100     call    004433E0
00425CAA    .   8D4C24 10       lea     ecx, [esp+10]
00425CAE    .   51              push    ecx
00425CAF    .   E8 2CD70100     call    004433E0
```



## 00425CB4 - 00425CF8

```asm
00425CB4   .  83C4 08       add     esp, 8  
00425CB7   .  85F6        test    esi, esi  
00425CB9   .  7D 40       jge     short 00425CFB  
00425CBB   .  68 E8030000 push    3E8                       ; /Timeout = 1000. ms  
00425CC0   .  FF15 A4314D00 call   [<&KERNEL32.Sleep>]       ; \Sleep  
00425CC6   .  8B15 F8625100 mov    edx, [5162F8]  
00425CCC   .  68 4A560000 push    564A  
00425CD1   .  6A 00       push    0  
00425CD3   .  6A 10       push    10  
00425CD5   .  52         push    edx  
00425CD6   .  E8 15160200 call   004472F0  
00425CDB   .  83C4 10     add     esp, 10  
00425CDE   .  68 D9040000 push    4D9                       ; /ControlID = 4D9 (1241.)  
00425CE3   .  57         push    edi                       ; |hWnd  
00425CE4   .  FF15 D4364D00 call   [<&USER32.GetDlgItem>]  ; \GetDlgItem  
00425CEA   .  50         push    eax                       ; /hWnd  
00425CEB   .  FF15 70364D00 call   [<&USER32.SetFocus>]    ; \SetFocus  
00425CF1   .  5F         pop     edi  
00425CF2   .  33C0      xor     eax, eax  
00425CF4   .  5E         pop     esi  
00425CF5   .  83C4 0C   add     esp, 0C  
00425CF8   .  C2 1000   retn    10  
```

## 00425CFB - 00425D2F

```asm
00425CFB   >  8B0D C4615000 mov    ecx, [5061C4]  
00425D01   .  E8 7AF0FFFF call   00424D80  
00425D06   .  85C0      test    eax, eax  
00425D08   .  7C 40     jl      short 00425D4A  
00425D0A   .  E8 A1F3FFFF call   004250B0  
00425D0F   .  8B0D F8625100 mov    ecx, [5162F8]  
00425D15   .  8BF0      mov     esi, eax  
00425D17   .  E8 64380000 call   00429580  
00425D1C   .  85F6      test    esi, esi  
00425D1E   .  7C 2A     jl      short 00425D4A  
00425D20   .  A1 F8625100 mov    eax, [5162F8]  
00425D25   .  68 4B560000 push    564B  
00425D2A   .  6A 00     push    0  
00425D2C   .  6A 40     push    40  
00425D2E   .  50        push    eax  
00425D2F   .  E8 BC150200 call   004472F0                   ;  弹出注册成功对话框  
```

### （1）注册码合法性验证 call

```asm
00425F70  /$  8B4424 14    mov     eax, [esp+14]  
00425F74  |.  8B4C24 10    mov     ecx, [esp+10]  
00425F78  |.  8B5424 0C    mov     edx, [esp+C]  
00425F7C  |.  53          push    ebx  
00425F7D  |.  55          push    ebp  
00425F7E  |.  56          push    esi  
00425F7F  |.  8B7424 10    mov     esi, [esp+10]  
00425F83  |.  C700 00000000 mov    dword ptr [eax], 0  
00425F89  |.  8B4424 14    mov     eax, [esp+14]  
00425F8D  |.  C701 00000000 mov    dword ptr [ecx], 0  
00425F93  |.  57          push    edi  
00425F94  |.  C702 00000000 mov    dword ptr [edx], 0  
00425F9A  |.  56          push    esi  
00425F9B  |.  C700 00000000 mov    dword ptr [eax], 0  
00425FA1  |.  E8 DAD10100 call    00443180                   ;  取假码第一部分八位  
00425FA6  |.  8B3D A4334D00 mov    edi, [<&MSVCRT.wcschr>]     ;  MSVCRT.wcschr  
00425FAC  |.  6A 2D       push    2D                          ; /w = 002D  ('-')  \\ "-"连接  
00425FAE  |.  56          push    esi                         ; |wstr  
00425FAF  |.  894424 20   mov     [esp+20], eax               ; |  
00425FB3  |.  FFD7        call    edi                         ; \wcschr  
00425FB5  |.  8BF0        mov     esi, eax  
00425FB7  |.  83C4 0C     add     esp, 0C  
00425FBA  |.  85F6        test    esi, esi  
00425FBC  |.  75 05       jnz     short 00425FC3  
00425FBE  |.  5F          pop     edi  
00425FBF  |.  5E          pop     esi  
00425FC0  |.  5D          pop     ebp  
00425FC1  |.  5B          pop     ebx  
00425FC2  |.  C3          retn  
00425FC3  |>  83C6 02     add     esi, 2  
00425FC6  |.  56          push    esi  
00425FC7  |.  E8 B4D10100 call    00443180                   ;  取假码第二部分八位  
00425FCC  |.  6A 2D       push    2D                         \\ "-"连接  
00425FCE  |.  56          push    esi  
00425FCF  |.  8BE8        mov     ebp, eax  
00425FD1  |.  FFD7        call    edi  
00425FD3  |.  8BF0        mov     esi, eax  
00425FD5  |.  83C4 0C     add     esp, 0C  
00425FD8  |.  85F6        test    esi, esi  
00425FDA  |.  75 05       jnz     short 00425FE1  
00425FDC  |.  5F          pop     edi  
00425FDD  |.  5E          pop     esi  
00425FDE  |.  5D          pop     ebp  
00425FDF  |.  5B          pop     ebx  
00425FE0  |.  C3          retn  
00425FE1  |>  83C6 02     add     esi, 2  
00425FE4  |.  56          push    esi  
00425FE5  |.  E8 96D10100 call    00443180                   ;  取假码第三部分八位  
00425FEA  |.  6A 2D       push    2D                         \\ "-"连接  
00425FEC  |.  56          push    esi  
00425FED  |.  8BD8        mov     ebx, eax  
00425FEF  |.  FFD7        call    edi  
00425FF1  |.  8BF0        mov     esi, eax  
00425FF3  |.  83C4 0C     add     esp, 0C  
00425FF6  |.  85F6        test    esi, esi  
00425FF8  |.  75 05       jnz     short 00425FFF  
00425FFA  |.  5F          pop     edi  
00425FFB  |.  5E          pop     esi  
00425FFC  |.  5D          pop     ebp  
00425FFD  |.  5B          pop     ebx  
00425FFE  |.  C3          retn  
00425FFF  |>  83C6 02     add     esi, 2  
00426002  |.  56          push    esi  
00426003  |.  E8 78D10100 call    00443180                   ;  取假码第四部分八位  
00426008  |.  83C4 04     add     esp, 4  
0042600B  |.  66:837E 10 00 cmp   word ptr [esi+10], 0  
00426010  |.  74 07       je      short 00426019  
00426012  |.  5F          pop     edi  
00426013  |.  5E          pop     esi  
00426014  |.  5D          pop     ebp  
00426015  |.  33C0        xor     eax, eax  
00426017  |.  5B          pop     ebx  
00426018  |.  C3          retn  
00426019  |>  8B4C24 14   mov     ecx, [esp+14]  
0042601D  |.  8B5424 18   mov     edx, [esp+18]  
00426021  |.  890A        mov     [edx], ecx  
00426023  |.  8B4C24 1C   mov     ecx, [esp+1C]  
00426027  |.  8B5424 20   mov     edx, [esp+20]  
0042602B  |.  5F          pop     edi  
0042602C  |.  8929        mov     [ecx], ebp  
0042602E  |.  8B4C24 20   mov     ecx, [esp+20]  
00426032  |.  5E          pop     esi  
00426033  |.  891A        mov     [edx], ebx  
00426035  |.  5D          pop     ebp  
00426036  |.  8901        mov     [ecx], eax  
00426038  |.  B8 01000000 mov     eax, 1  
0042603D  |.  5B          pop     ebx  
0042603E  \\.  C3         retn  
0042603F      90          nop  
00426040   .  E9 0B000000 jmp     00426050  
```

### （2）e-mail 地址验证及变换 call

```asm
00425300  /$  56          push    esi  
00425301  |.  8B7424 08   mov     esi, [esp+8]  
00425305  |.  57          push    edi  
00425306  |.  8B7C24 10   mov     edi, [esp+10]  
0042530A  |.  33D2        xor     edx, edx  
0042530C  |.  33C0        xor     eax, eax  
0042530E  |.  8917        mov     [edi], edx  
00425310  |.  66:8B06     mov     ax, [esi]  
00425313  |.  66:85C0     test    ax, ax  
00425316  |.  B9 01000000 mov     ecx, 1  
0042531B  |.  74 32       je      short 0042534F  
0042531D  |>  66:3D 2000  /cmp    ax, 20                      ;  与空格比较  
00425321  |.  74 1B       |je     short 0042533E  
00425323  |.  66:3D 0900  |cmp    ax, 9  
00425327  |.  74 15       |je     short 0042533E  
00425329  |.  25 FFFF0000 |and    eax, 0FFFF  
0042532E  |.  0FAFC1      |imul   eax, ecx                    ;  eax=eax*ecx  
00425331  |.  35 82945D41 |xor    eax, 415D9482               ;  eax=eax xor 415D9482  
00425336  |.  42         |inc    edx                          ;  循环计数  
00425337  |.  83FA 1E   |cmp    edx, 1E                       ;  跟 1E 比较  
```



```
0042533A  .  8BC8            |mov     ecx, eax                        ;  eax 结果入 ecx  
0042533C  .  7D 0C           |jge     short 0042534A  
0042533E  >  66:8B46 02      |mov     ax, [esi+2]                      ;  下一字符入 ax  
00425342  .  83C6 02         |add     esi, 2  
00425345  .  66:85C0         |test    ax, ax  
00425348  .^ 75 D3           \jnz     short 0042531D  
0042534A  >  83FA 08         |cmp     edx, 8                            ;  e-mail 地址长度要大于等于 8 位  
0042534D  .  7D 05           |jge     short 00425354  
0042534F  >  5F              |pop     edi  
00425350  .  33C0           |xor     eax, eax  
00425352  .  5E              |pop     esi  
00425353  .  C3              |retn  
00425354  >  890F            |mov     [edi], ecx  
00425356  .  5F              |pop     edi  
00425357  .  B8 01000000     |mov     eax, 1  
0042535C  .  5E              |pop     esi  
0042535D  \.  C3             |retn  
```

### （3）注册码第四部分验证

```
00425360  /$  8B4424 04      |mov     eax, [esp+4]                      ;  真码第一部分入 eax  
00425364  |.  8B4C24 08      |mov     ecx, [esp+8]                      ;  假码第二部分入 ecx  
00425368  |.  56              |push    esi  
00425369  |.  8B7424 10      |mov     esi, [esp+10]                    ;  假码第三部分入 esi  
0042536D  |.  35 21098B43     |xor     eax, 438B0921                   ;  eax=eax xor 438B0921  
00425372  |.  81F6 377F5583   |xor     esi, 83557F37                   ;  esi=esi xor 83557F37  
00425378  |.  0FAFF0         |imul    esi, eax                        ;  esi=esi*eax  
0042537B  |.  81F1 44709C63  |xor     ecx, 639C7044                   ;  ecx=ecx xor 639C7044  
00425381  |.  0FAFF1         |imul    esi, ecx                        ;  esi=esi*ecx  
00425384  |.  33D2           |xor     edx, edx  
00425386  |.  8BC6           |mov     eax, esi  
00425388  |.  B9 1F000000     |mov     ecx, 1F  
0042538D  |.  F7F1           |div     ecx  
0042538F  |.  8BCA           |mov     ecx, edx  
00425391  |.  D3C6           |rol     esi, cl                         ;  esi 左移位 cl 次  
00425393  |.  33D2           |xor     edx, edx  
00425395  |.  B9 1F000000     |mov     ecx, 1F  
0042539A  |.  81F6 7ACD9642  |xor     esi, 4296CD7A                   ;  esi=esi xor 4296CD7A  
004253A0  |.  8BC6           |mov     eax, esi  
004253A2  |.  F7F1           |div     ecx  
004253A4  |.  8BC6           |mov     eax, esi  
004253A6  |.  5E              |pop     esi  
004253A7  |.  8BCA           |mov     ecx, edx  
004253A9  |.  D3C0           |rol     eax, cl                         ;  eax 左移位 cl 次  
004253AB  \.  C3             |retn  
```

------------------------------------------------------------------------

## 【破解总结】

1. `e-mail` 地址长度要大于等于八位；
2. 注册码分四部分，每部分为八位，中间由`-`连接；
3. 注册码第一部分与 `e-mail` 地址有关，由其转换结果决定；
4. 注册码第二部分为固定值，可为 `11E1A91A`、`2050000`、`3000000`、`3030000` 其中之一；
5. 注册码第三部分任意；
6. 注册码第四部分由注册码第一部分、注册码第二部分、注册码第三部分计算而来；
7. 提供一组可用注册码：

   `e-mail`：`123@1.com`
   注册码：`0CECD864-11E1A91A-78787878-B40EF817`

8. 注册信息保存于注册表。

------------------------------------------------------------------------

## 【版权声明】

本文系作者原创, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
