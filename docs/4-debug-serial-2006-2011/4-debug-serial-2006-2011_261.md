
# Ardamax Keylogger 2.71 破解分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-124.htm`](https://www.pediy.com/kssd/pediy09/pediy09-124.htm)
>
> └文章标题┐:Ardamax Keylogger 2.71 破解分析  
> └破文作者┐:-=>大菜<=-  
> └破解对象┐:Ardamax Keylogger 2.71  
> └下载地址┐:网上搜..  
> └对象大小┐:不知..  
> └加壳方式┐:没有  
> └保护方式┐:普通算法  
> └编写语言┐:peid 说"啥也没找到" - -!  
> └使用工具┐:OD  
> └破解平台┐:D 版 XP  
> └破解声明┐:  
> &nbsp;&nbsp;&nbsp;&nbsp;Crack 真帅\`\`哦呵呵呵  
> &nbsp;&nbsp;&nbsp;&nbsp;不过想想软件作者还真可怜,不过是国外的,我想偶在 Crack 时他大概在打喷嚏吧!我连软件怎么用有啥用都不清楚哇~~~  
> \----------------------------------------------------------------------------------  
> └破解过程┐:  
> 
> 所有 `messagebox` 都设置断点,一共 27 个!
>
> 输入"jiangwu55"->"121212"
>
> 断在:

```
00405E53  |.  6A 30            push    30
00405E55  |.  68 082D4600   push    00462D08                        ;  UNICODE "Ardamax Keylogger"
00405E5A  |.  68 D03A4600   push    00463AD0                        ;  UNICODE "No registration name entered."
00405E5F  |>  FF77 04         push    dword ptr [edi+4]             ; |hOwner
00405E62  |.  FF15 30B54500 call    dword ptr [<&USER32.MessageBoxW>>; \\MessageBoxW    ;断在这
00405E68  |.  E9 84000000   jmp     00405EF1
00405E6D  |>  51            push    ecx
00405E6E  |.  8D8D F0FBFFFF lea     ecx, dword ptr [ebp-410]
00405E74  |.  8BC4          mov     eax, esp
00405E76  |.  51            push    ecx
00405E77  |.  E8 B84C0100   call    0041AB34
00405E7C  |.  51            push    ecx
00405E7D  |.  8D8D F8FDFFFF lea     ecx, dword ptr [ebp-208]
00405E83  |.  8BC4          mov     eax, esp
00405E85  |.  51            push    ecx
00405E86  |.  E8 A94C0100   call    0041AB34
00405E8B  |.  E8 A1FEFFFF   call    00405D31
00405E90  |.  85C0          test    eax, eax
00405E92  |.  74 65         je      short 00405EF9
00405E94  |.  6A 40         push    40                              ; /Style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL
00405E96  |.  68 082D4600   push    00462D08                        ; |Title = "Ardamax Keylogger"
00405E9B  |.  68 103B4600   push    00463B10                        ; |Text = "Registration code is accepted. Thank you for registration!"
00405EA0  |.  FF77 04         push    dword ptr [edi+4]             ; |hOwner
00405EA3  |.  FF15 30B54500 call    dword ptr [<&USER32.MessageBoxW>>; \\MessageBoxW   ;注册成功
```

> 可以看到字符串了,向上看可以看到 `GetDlgItemText` 这个函数(不知为啥直接断这个函数不行),取消所有断点,重新断在 `405dfd` 处:

```
00405DFD  /$  55            push    ebp                               ;F2 重新载入后断在这
00405DFE  |.  8BEC          mov     ebp, esp
00405E00  |.  81EC 10040000 sub     esp, 410
00405E06  |.  66:837D 08 01 cmp     word ptr [ebp+8], 1
00405E0B  |.  53            push    ebx
00405E0C  |.  56            push    esi
00405E0D  |.  0F85 B1000000 jnz     00405EC4
00405E13  |.  8B35 64B54500 mov     esi, dword ptr [<&USER32.GetDlgI>;  USER32.GetDlgItemTextW
00405E19  |.  BB 04010000   mov     ebx, 104
00405E1E  |.  53            push    ebx                               ; /Count => 104 (260.)
00405E1F  |.  8D85 F0FBFFFF lea     eax, dword ptr [ebp-410]          ; |
00405E25  |.  50            push    eax                               ; |Buffer
00405E26  |.  68 07040000   push    407                               ; |ControlID = 407 (1031.)
00405E2B  |.  FF77 04         push    dword ptr [edi+4]             ; |hWnd
00405E2E  |.  FFD6          call    esi                               ; \\GetDlgItemTextW           ;取注册码
00405E30  |.  53            push    ebx                               ; /Count => 104 (260.)
00405E31  |.  8D85 F8FDFFFF lea     eax, dword ptr [ebp-208]          ; |
00405E37  |.  50            push    eax                               ; |Buffer
00405E38  |.  68 F7030000   push    3F7                               ; |ControlID = 3F7 (1015.)
00405E3D  |.  FF77 04         push    dword ptr [edi+4]             ; |hWnd
00405E40  |.  FFD6          call    esi                               ; \\GetDlgItemTextW           ;取注册名
```

```
00405E42  |.  8D85 F8FDFFFF lea     eax, dword ptr [ebp-208]
00405E48  |.  50            push    eax                               ; /String
00405E49  |.  FF15 00B14500 call    dword ptr [<&KERNEL32.lstrlenW>] ; \\lstrlenW                   ;取注册名长度
00405E4F  |.  85C0          test    eax, eax
00405E51  |.  75 1A         jnz     short 00405E6D                                        ;为空出错
*******************************************************************************************
00405E6D  |>  \\51          push    ecx                             ;上面不为空就跳到这
00405E6E  |.  8D8D F0FBFFFF lea     ecx, dword ptr [ebp-410]
00405E74  |.  8BC4          mov     eax, esp
00405E76  |.  51            push    ecx
00405E77  |.  E8 B84C0100   call    0041AB34
00405E7C  |.  51            push    ecx
00405E7D  |.  8D8D F8FDFFFF lea     ecx, dword ptr [ebp-208]
00405E83  |.  8BC4          mov     eax, esp
00405E85  |.  51            push    ecx
00405E86  |.  E8 A94C0100   call    0041AB34
00405E8B  |.  E8 A1FEFFFF   call    00405D31                         ;经典关键,下面有注册成功信息
00405E90  |.  85C0          test    eax, eax
00405E92  |.  74 65         je      short 00405EF9
*******************************************************************************************
```

> 关键 `call`:(节省篇幅,省略 n 条代码...)^-^

```
00405D31  /$  55            push    ebp                               ;跟进后的第一条句语
```

> `F8` 一直走....

```
00405D8D  |.  FF75 EC       push    dword ptr [ebp-14]     ; /Arg1 = 0012EAFC ASCII "jiangwu55"   ;注册名
00405D90  |.  8D9D 64FEFFFF lea     ebx, dword ptr [ebp-19C] ; |
00405D96  |.  E8 1DFFFFFF   call    00405CB8             ; \\IWM.00405CB8                    ;关键 call
```

> 因为步过之后可以看到注册码了&&

```
*******************************************************************************************
跟进后:
00405CB9  |.  8BEC          mov     ebp, esp
00405CBB  |.  83EC 0C       sub     esp, 0C
00405CBE  |.  56            push    esi
00405CBF  |.  8B35 38B14500 mov     esi, dword ptr [<&KERNEL32.lstrl>;  kernel32.lstrlenA
00405CC5  |.  57            push    edi
00405CC6  |.  BF 0C594600   mov     edi, 0046590C        ;  ASCII "207144FA6AD570E"    ;一串字符(下面说为字符串 a)
00405CCB  |.  57            push    edi                   ; /String => "207144FA6AD570E"
00405CCC  |.  FFD6          call    esi                   ; \\lstrlenA                   ;取其长度
00405CCE  |.  FF75 08       push    dword ptr [ebp+8]     ; /String                      ;注册名
00405CD1  |.  8945 F8       mov     dword ptr [ebp-8], eax; |
00405CD4  |.  FFD6          call    esi                   ; \\lstrlenA                   ;取注册名长度
00405CD6  |.  8BF0          mov     esi, eax
00405CD8  |.  85F6          test    esi, esi
00405CDA  |.  8975 F4       mov     dword ptr [ebp-C], esi
00405CDD  |.  75 04         jnz     short 00405CE3
00405CDF  |.  8803          mov     byte ptr [ebx], al
00405CE1  |.  EB 48         jmp     short 00405D2B
00405CE3  |>  57            push    edi                   ; /String2         ;字符串 a
00405CE4  |.  53            push    ebx                   ; |String1         ;一个内存地址
```



## 汇编代码分析

```assembly
00405CE5  |.  FF15 4CB14500 call    dword ptr [<KERNEL32.lstrcpyA>] ; \lstrcpyA        ;复制  

00405CEB  |.  8B45 F8      mov     eax, dword ptr [ebp-8]                      ;字符串 a 长度到 eax  

00405CEE  |.  3BF0        cmp     esi, eax                                      ;和注册名长度比较  

00405CF0  |.  8975 FC    mov     dword ptr [ebp-4], esi                        ;注册名长度传到 ebp-4  

00405CF3  |.  7F 03       jg      short 00405CF8                                ;大于就跳  

00405CF5  |.  8945 FC    mov     dword ptr [ebp-4], eax                        ;小于的话 ebp-4 就是字符串 a 长度  

00405CF8  |>  33F6        xor     esi, esi                                      ;字符串 a 长度大于注册名长度就清零  

00405CFA  |.  3975 FC     cmp     dword ptr [ebp-4], esi                      ;esi 作为计数器  
                                                  
    ebp-4 现在是注册名长度  

00405CFD  |.  7E 2C       jle     short 00405D2B  

00405CFF  |>  8BC6        /mov     eax, esi                                     ;本次循环计数器值到 eax  

00405D01  |.  99          |cdq                                                 ;双字扩展,没啥路用  

00405D02  |.  F77D F8     |idiv    dword ptr [ebp-8]                           ;本次循环计数器值除以字符串 a 长度  

00405D05  |.  8BC6        |mov     eax, esi                                     ;本次循环计数器值到 eax  

00405D07  |.  6A 19       |push    19                                           ;19 进栈  

00405D09  |.  5F          |pop     edi                                          ;弹出来  

00405D0A  |.  8D0C1A      |lea     ecx, dword ptr [edx+ebx]                     ;edx 是余数,ebx 是字符串 a 的地址  

                                                  
    第一次当然是整个字符串地址到 ecx 了  

00405D0D  |.  99          |cdq  

00405D0E  |.  F77D F4     |idiv    dword ptr [ebp-C]                           ;本次循环计数器值除以注册名长度  

00405D11  |.  8B45 08     |mov     eax, dword ptr [ebp+8]                      ;注册名到 eax  

00405D14  |.  0FB60402    |movzx   eax, byte ptr [edx+eax]                    ;每次循环取注册名各个字符  

00405D18  |.  0FB611      |movzx   edx, byte ptr [ecx]                         ;每次循环取字符串 a 各个字符  

00405D1B  |.  33C2        |xor     eax, edx                                     ;xor 异或  

00405D1D  |.  99          |cdq  

00405D1E  |.  F7FF        |idiv    edi                                          ;除以 19  

00405D20  |.  80C2 41     |add     dl, 41                                       ;余数加上 41  

00405D23  |.  46          |inc     esi                                          ;计数器加一  

00405D24  |.  3B75 FC     |cmp     esi, dword ptr [ebp-4]                     ;比看看算完没有  

00405D27  |.  8811        |mov     byte ptr [ecx], dl                           ;结果传回字符串 a 对应的位置  

00405D29  |.^ 7C D4       \jl     short 00405CFF                               ;没完跳上去  

00405D2B  |>  5F          pop     edi  

00405D2C  |.  5E          pop     esi  

00405D2D  |.  C9          leave  

00405D2E  \.  C2 0400     retn    4  
```

## 算法总结

根据比较字符串 `a` 和注册名的长度循环，前者大则循环 `15` 次，后者大则循环后者长度次。

在 `00405CFF` 到 `00405D02` 处代码和 `00405D18` 处是有联系的。

比如说注册名长度大于字符串 `a` 长度：

*   循环超过 `15` 次
*   到 `15` 次时原 `[ecx]` 值已经是新算出来的字符串了，而在 `00405D22` 处的 `idiv`，则为 `0xf % 0xf` 为 `0`
*   `00405D0A` 处又重新从新字符串第一个字符开始取了，而且还是一样传回原来的位置，刚开始偶还以为接在后面，我汗了

注册机可以算到注册名 `30` 位（我限制了⁰^）。

## WinSDK 注册机 WM_COMMAND 消息代码

下面是 WinSDK 注册机的 `WM_COMMAND` 消息代码：

```cpp
case WM_COMMAND:
    if (IDOK == LOWORD(wParam))
    {
        char name[100], str[100] = "207144FA6AD570E", key[100];
        int i, len = 0;
        GetDlgItemText(hwndDlg, IDC_EDIT_NAME, name, sizeof(name)); // 取 name
        memset(key, 0, sizeof(key)); // 全归 0, 免乱码
        if (name[0] != NULL) // 没输入的别点确定
        {       
            len = strlen(name);

            if (len <= 15) // 注册名长度小于字符串 a 长度的部分
            {
                for (i = len; i < 15; i++)
                    name[i] = name[i - len];
                for (i = 0; i < 15; i++)
                {
                    key[i] = ((name[i] ^ str[i]) % 0x19) + 0x41;
                }
                SetDlgItemText(hwndDlg, IDC_EDIT_SERIAL, key);
            }
            else // 字符串 a 长度小于注册名长度的部分
            {
                for (i = 0; i < len; i++)
                {
                    if (i > 14)
                        str[i - 15] = ((name[i] ^ str[i - 15]) % 0x19) + 0x41;
                    else
                        str[i] = ((name[i] ^ str[i]) % 0x19) + 0x41;
                }
                SetDlgItemText(hwndDlg, IDC_EDIT_SERIAL, str);
            }
        }
        else
        {
            SetDlgItemText(hwndDlg, IDC_EDIT_SERIAL, "Your Name???");
        }
    }
    return TRUE;
```

---

## 经验总结

Crack 一个软件也不难吧，呵呵至少这个不难->^-^

---

## 版权声明

本文原创于看雪软件安全论坛，转载请注明作者并保持文章的完整，谢谢!

2007 年 4 月 26 日 20:1:7

4dd466d0e1e0cc123afbe504da2462ca
