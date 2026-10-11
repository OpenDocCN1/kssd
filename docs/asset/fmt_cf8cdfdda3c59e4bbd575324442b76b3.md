
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
