
# Active Desktop Album 3.1 算法分析

> 原文：[`www.pediy.com/kssd/pediy10/102670.html`](https://www.pediy.com/kssd/pediy10/102670.html)

## 基本信息

- **破文标题**：Active Desktop Album 3.1 算法分析
- **破文作者**：tianxj
- **作者邮箱**：tianxj_2007@126.com
- **作者主页**：WwW.ChiNaPYG.CoM
- **破解工具**：`PEiD`, `OD`, `DeDe`
- **破解平台**：Windows XP sp3
- **软件名称**：Active Desktop Album 3.1
- **软件大小**：2964KB
- **软件语言**：英文
- **软件类别**：国外软件/桌面制作
- **软件授权**：共享版
- **运行环境**：WinXP/2000/2003/Vista
- **更新时间**：2009-6-11
- **原版下载**：http://www.onlinedown.net/soft/86742.htm
- **保护方式**：注册码

### 软件简介
Active Desktop Album 是一款动态的在桌面上轮换显示照片或者其他图片的电子相册软件，程序可以自动的定时（每几分钟也可自己定义时间)来自动切换照片在桌面的显示，可以自由的编辑照片的显示方式和样式边框等。你可以将家人或者爱人的照片用这个程序将其显示在桌面上，工作上的时候随时都可以看到，相信那将是非常美妙的一件事情。

### 破解声明
我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

## 破解内容

### 一、对 `DesktopWatch.exe` 查壳为 `Borland Delphi 6.0 - 7.0`

### 二、用 `DeDe` 查找按钮事件就可以快速到达关键部位

代码：

```asm
00629E60  /$  55             push    ebp
00629E61  |.  8BEC           mov     ebp, esp
00629E63  |.  83C4 F4        add     esp, -0C
00629E66  |.  53             push    ebx
00629E67  |.  33C9           xor     ecx, ecx
00629E69  |.  894D F4        mov     dword ptr [ebp-C], ecx
00629E6C  |.  8955 F8        mov     dword ptr [ebp-8], edx
00629E6F  |.  8945 FC        mov     dword ptr [ebp-4], eax
00629E72  |.  8B45 FC        mov     eax, dword ptr [ebp-4]
00629E75  |.  E8 72ADDDFF    call    00404BEC
00629E7A  |.  8B45 F8        mov     eax, dword ptr [ebp-8]
00629E7D  |.  E8 6AADDDFF    call    00404BEC
00629E82  |.  33C0           xor     eax, eax
00629E84  |.  55             push    ebp
00629E85  |.  68 319F6200    push    00629F31
00629E8A  |.  64:FF30        push    dword ptr fs:[eax]
00629E8D  |.  64:8920        mov     dword ptr fs:[eax], esp
00629E90  |.  33DB           xor     ebx, ebx
00629E92  |.  8D55 F4        lea     edx, dword ptr [ebp-C]
00629E95  |.  8B45 F8        mov     eax, dword ptr [ebp-8]
00629E98  |.  E8 EBEFDDFF    call    00408E88
00629E9D  |.  837D F4 00     cmp     dword ptr [ebp-C], 0
00629EA1  |.  74 73          je      short 00629F16          ; //试炼码为空则跳
00629EA3  |.  8B55 F8        mov     edx, dword ptr [ebp-8]   ; //试炼码
00629EA6  |.  8B45 FC        mov     eax, dword ptr [ebp-4]   ; //用户名
00629EA9  |.  E8 4EFBFFFF    call    006299FC                ; //将用户名与"pb"相连字符串的 MD5 值与"56f9c69b5ad4bfd656c8dec9c6228223"作比较
00629EAE  |.  84C0           test    al, al
00629EB0  |.  74 04          je      short 00629EB6
00629EB2  |.  33DB           xor     ebx, ebx
00629EB4  |.  EB 60          jmp     short 00629F16
00629EB6  |>  8B55 F8        mov     edx, dword ptr [ebp-8]
00629EB9  |.  8B45 FC        mov     eax, dword ptr [ebp-4]
00629EBC  |.  E8 4BFAFFFF    call    0062990C                ; //将用户名与"tq"相连字符串的 MD5 值与"35e2233a9cbbd40e0da83ca5c185f975 作比较
00629EC1  |.  84C0           test    al, al
00629EC3  |.  74 04          je      short 00629EC9
00629EC5  |.  B3 01          mov     bl, 1
00629EC7  |.  EB 4D          jmp     short 00629F16          ;-------以上检验用户名注册码是否在黑名单
00629EC9  |>  8B55 F8        mov     edx, dword ptr [ebp-8]
00629ECC  |.  8B45 FC        mov     eax, dword ptr [ebp-4]
00629ECF  |.  E8 18FCFFFF    call    00629AEC                ; //注册码第 1 个字符必须是用户名与"gfhfghj"相连字符串的 MD5 值小写的第 1 个字符
00629ED4  |.  84C0           test    al, al
00629ED6  |.  74 3E          je      short 00629F16
00629ED8  |.  8B55 F8        mov     edx, dword ptr [ebp-8]
00629EDB  |.  8B45 FC        mov     eax, dword ptr [ebp-4]
00629EDE  |.  E8 B5FCFFFF    call    00629B98                ; //注册码第 2 个字符必须是用户名与"3425"相连字符串的 MD5 值小写的第 1 个字符
00629EE3  |.  84C0           test    al, al
00629EE5  |.  74 2F          je      short 00629F16
00629EE7  |.  8B55 F8        mov     edx, dword ptr [ebp-8]
00629EEA  |.  8B45 FC        mov     eax, dword ptr [ebp-4]
00629EED  |.  E8 56FDFFFF    call    00629C48                ; //注册码第 3 个字符必须是用户名与"rttrh"相连字符串的 MD5 值小写的第 1 个字符
00629EF2  |.  84C0           test    al, al
00629EF4  |.  74 20          je      short 00629F16
00629EF6  |.  8B55 F8        mov     edx, dword ptr [ebp-8]
00629EF9  |.  8B45 FC        mov     eax, dword ptr [ebp-4]
00629EFC  |.  E8 F7FDFFFF    call    00629CF8                ; //注册码第 4 个字符必须是用户名与"hnnn"相连字符串的 MD5 值小写的第 1 个字符
00629F01  |.  84C0           test    al, al
00629F03  |.  74 11          je      short 00629F16
00629F05  |.  8B55 F8        mov     edx, dword ptr [ebp-8]
00629F08  |.  8B45 FC        mov     eax, dword ptr [ebp-4]
00629F0B  |.  E8 98FEFFFF    call    00629DA8                ; //注册码第 5 个字符必须是用户名与"657y56yh8uhgj"相连字符串的 MD5 值小写的第 1 个字符
00629F10  |.  84C0           test    al, al
00629F12  |.  74 02          je      short 00629F16
00629F14  |.  B3 01          mov     bl, 1                   ; //标志位赋值
00629F16  |>  33C0           xor     eax, eax
00629F18  |.  5A             pop     edx
00629F19  |.  59             pop     ecx
00629F1A  |.  59             pop     ecx
00629F1B  |.  64:8910        mov     dword ptr fs:[eax], edx
00629F1E  |.  68 389F6200    push    00629F38
00629F23  |>  8D45 F4        lea     eax, dword ptr [ebp-C]
00629F26  |.  BA 03000000    mov     edx, 3
00629F2B  |.  E8 30A8DDFF    call    00404760
00629F30  \\.  C3             retn
00629F31   .^ E9 86A1DDFF    jmp     004040BC
00629F36   .^ EB EB          jmp     short 00629F23
00629F38   .  8BC3           mov     eax, ebx
00629F3A   .  5B             pop     ebx
00629F3B   .  8BE5          mov     esp, ebp
00629F3D   .  5D             pop     ebp
00629F3E   .  C3             retn
```

以 `call 00629AEC` 为例

代码：



```
00629AEC  /$  55          push    ebp
00629AED  |.  8BEC        mov     ebp, esp
00629AEF  |.  83C4 E0     add     esp, -20
00629AF2  |.  53          push    ebx
00629AF3  |.  33C9        xor     ecx, ecx
00629AF5  |.  894D E0     mov     dword ptr [ebp-20], ecx
00629AF8  |.  894D F4     mov     dword ptr [ebp-C], ecx
00629AFB  |.  8955 F8     mov     dword ptr [ebp-8], edx
00629AFE  |.  8945 FC     mov     dword ptr [ebp-4], eax
00629B01  |.  8B45 FC     mov     eax, dword ptr [ebp-4]           ;  //用户名
00629B04  |.  E8 E3B0DDFF call    00404BEC
00629B09  |.  8B45 F8     mov     eax, dword ptr [ebp-8]           ;  //试炼码
00629B0C  |.  E8 DBB0DDFF call    00404BEC
00629B11  |.  33C0        xor     eax, eax
00629B13  |.  55          push    ebp
00629B14  |.  68 7A9B6200 push    00629B7A
00629B19  |.  64:FF30     push    dword ptr fs:[eax]
00629B1C  |.  64:8920     mov     dword ptr fs:[eax], esp
00629B1F  |.  8D45 E0     lea     eax, dword ptr [ebp-20]
00629B22  |.  B9 909B6200 mov     ecx, 00629B90                   ;  ASCII "gfhfghj"
00629B27  |.  8B55 FC     mov     edx, dword ptr [ebp-4]           ;  //用户名
00629B2A  |.  E8 19AFDDFF call    00404A48
00629B2F  |.  8B45 E0     mov     eax, dword ptr [ebp-20]         ;  //用户名与"gfhfghj"相连
00629B32  |.  8D55 E4     lea     edx, dword ptr [ebp-1C]
00629B35  |.  E8 2EFCFFFF call    00629768                         ;  //标准 MD5 运算
00629B3A  |.  8D45 E4     lea     eax, dword ptr [ebp-1C]
00629B3D  |.  8D55 F4     lea     edx, dword ptr [ebp-C]
00629B40  |.  E8 97FCFFFF call    006297DC
00629B45  |.  8B45 F4     mov     eax, dword ptr [ebp-C]          ;  //MD5 值小写
00629B48  |.  8A00        mov     al, byte ptr [eax]              ;  //取 MD5 值小写的第 1 个字符
00629B4A  |.  8B55 F8     mov     edx, dword ptr [ebp-8]          ;  //试炼码
00629B4D  |.  3A02        cmp     al, byte ptr [edx]              ;  //将试炼码的第 1 个字符与 MD5 值小写的第 1 个字符比较
00629B4F  |.  75 04       jnz     short 00629B55                 ;  //关键跳转
00629B51  |.  B3 01       mov     bl, 1                          ;  //关键赋值
00629B53  |.  EB 02       jmp     short 00629B57
00629B55  |>  33DB        xor     ebx, ebx
00629B57  |>  33C0        xor     eax, eax
00629B59  |.  5A          pop     edx
00629B5A  |.  59          pop     ecx
00629B5B  |.  59          pop     ecx
00629B5C  |.  64:8910     mov     dword ptr fs:[eax], edx
00629B5F  |.  68 819B6200 push    00629B81
00629B64  |>  8D45 E0     lea     eax, dword ptr [ebp-20]
00629B67  |.  E8 D0ABDDFF call    0040473C
00629B6C  |.  8D45 F4     lea     eax, dword ptr [ebp-C]
00629B6F  |.  BA 03000000 mov     edx, 3
00629B74  |.  E8 E7ABDDFF call    00404760
00629B79  \\.  C3         retn
00629B7A   .^ E9 3DA5DDFF jmp     004040BC
00629B7F   .^ EB E3       jmp     short 00629B64
00629B81   .  8BC3        mov     eax, ebx
00629B83   .  5B          pop     ebx
00629B84   .  8BE5        mov     esp, ebp
00629B86   .  5D          pop     ebp
00629B87   .  C3          retn
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

## 破解总结

## 算法总结

1. 注册码第 1 个字符必须是用户名与`"gfhfghj"`相连字符串的 MD5 值小写的第 1 个字符
2. 注册码第 2 个字符必须是用户名与`"3425"`相连字符串的 MD5 值小写的第 1 个字符
3. 注册码第 3 个字符必须是用户名与`"rttrh"`相连字符串的 MD5 值小写的第 1 个字符
4. 注册码第 4 个字符必须是用户名与`"hnnn"`相连字符串的 MD5 值小写的第 1 个字符
5. 注册码第 5 个字符必须是用户名与`"657y56yh8uhgj"`相连字符串的 MD5 值小写的第 1 个字符

## 算法注册机

```vb
Private Sub Command1_Click()
    If Len(Text1.Text) = 0 Then
        Text2.Text = "输入有误，请重新输入！"
    Else
        Text2.Text = LCase(Mid(MD5(Text1.Text & "gfhfghj"), 1, 1) & Mid(MD5(Text1.Text & "3425"), 1, 1) & Mid(MD5(Text1.Text & "rttrh"), 1, 1) & Mid(MD5(Text1.Text & "hnnn"), 1, 1) & Mid(MD5(Text1.Text & "657y56yh8uhgj"), 1, 1))
    End If
End Sub
```

## 注册信息

保存在`[HKEY_CURRENT_USER\Software\Active Desktop Album]`

感谢飘云老大、猫老大、`Nisy` 老大以及很多前辈们的学习教程以及所有帮助过我的论坛兄弟姐妹们！谢谢

## 版权声明

破文是学习的手记，兴趣是成功的源泉；本破文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!
