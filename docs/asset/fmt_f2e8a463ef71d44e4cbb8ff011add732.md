
# WinImage V 8.0.8000

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-318.htm`](https://www.pediy.com/kssd/pediy08/pediy8-318.htm)

## 基本信息

- **【破文标题】** WinImage V 8.0.8000 序列号算法分析
- **【破文作者】** Fahrenheit
- **【作者邮箱】** `fahrenheit871116@163.com`
- **【破解工具】** `PEiD`, `OllyDebugger`
- **【破解平台】** `Windows XP 32-BIT EDITION`
- **【软件名称】** `WinImage V 8.0.8000`
- **【保护方式】** 注册码
- **【破解声明】** 请各位指教!!!@\_@

---

目标是 `WinImage V 8.0.8000`（English Version）。本打算用爆破法解除软件的时间限制，因为他只有 30 天的使用期限。

### 初步分析

查壳无壳。`Microsoft Visual C++ 7.0` 编写。`OllyDebugger` 下打开 `WinImage`，直接在 `GetSystemTime()` 和 `GetLocalTime()` 两个函数下断点，中断后，找到获取日期的地方，但比来比去，想想还是解除软件的序列号比较好，完全破解总比暴破来的安全，那就先破解序列号！

输入序列号有出错提示信息，“`Wrong Registration Key !`”，在参考文本串中搜寻，没有结果，看来问题有点棘手。搜寻数次仍然没有结果，看来老方法是行不通了。突然间想到，这是一个镜像制作软件，大多地方，都是选择输入，很少有文本输入的地方，这样就可以从程序中的文本输入的函数入手。在 Windows 的 API 函数表里一查，有 `GetDlgItemInt()`『获得指定输入框整数值』和 `GetDlgItemText()`『获得指定输入框输入字符串』和 `GetDlgItemTextA()`『获得指定输入框输入字符串』。前者不会有太大用处，所以直接在后两个函数出下了断点。再次输入序列号，果然，拦截成功！

输入序列号的信息窗口函数如下：

```asm
0044549D                 /.  55               push ebp
0044549E                 |.  8BEC             mov ebp,esp
004454A0                 |.  8B45 0C          mov eax,dword ptr ss:[ebp+C]
004454A3                 |.  2D 10010000      sub eax,110                     ;  Switch (cases 110..111)
004454A8                 |.  56               push esi
004454A9                 |.  0F84 13010000    je winimage.004455C2
004454AF                 |.  48               dec eax
004454B0                 |.  75 2C            jnz short winimage.004454DE
004454B2                 |.  0FB745 10        movzx eax,word ptr ss:[ebp+10]  ;  Case 111 of switch 004454A3
004454B6                 |.  48               dec eax                         ;  Switch (cases 1..819)
004454B7                 |.  74 3C            je short winimage.004454F5
004454B9                 |.  48               dec eax
004454BA                 |.  74 29            je short winimage.004454E5
004454BC                 |.  2D 17080000      sub eax,817
004454C1                 |.  75 1B            jnz short winimage.004454DE
004454C3                 |.  68 54D74700      push winimage.0047D754          ;  ASCII "::/registration.html"; Case 819 of switch 004454B6
004454C8                 |.  6A 20            push 20
004454CA                 |.  6A 01            push 1
004454CC                 |.  FF75 08          push dword ptr ss:[ebp+8]       ;  /hWnd
004454CF                 |.  FF15 10354700    call dword ptr ds:[<&USER32.GetParent>>; \GetParent
004454D5                 |.  50               push eax
004454D6                 |.  E8 189B0000      call winimage.0044EFF3
004454DB                 |.  83C4 10          add esp,10
004454DE                 |>  33C0             xor eax,eax                     ;  Default case of switch 004454B6
004454E0                 |.  E9 16010000      jmp winimage.004455FB
004454E5                 |>  6A 00            push 0                          ;  /Result = 0; Case 2 of switch 004454B6
004454E7                 |.  FF75 08          push dword ptr ss:[ebp+8]       ;  |hWnd
004454EA                 |.  FF15 38354700    call dword ptr ds:[<&USER32.EndDialog>>; \EndDialog
004454F0                 |.  E9 03010000      jmp winimage.004455F8
004454F5                 |>  8B35 6C354700    mov esi,dword ptr ds:[<&USER32.GetDlgI>;  USER32.GetDlgItemTextA; Case 1 of switch 004454B6
004454FB                 |.  53               push ebx
004454FC                 |.  57               push edi                        ;  --->  load name and sn
004454FD    1. ----->     |.  68 01010000      push 101                        ;  /Count = 101 (257.)
00445502                 |.  BF B8DE4800      mov edi,winimage.0048DEB8       ;  |ASCII "sadf"
00445507                 |.  57               push edi                        ;  |Buffer => winimage.0048DEB8
00445508                 |.  68 16080000      push 816                        ;  |ControlID = 816 (2070.)
0044550D                 |.  FF75 08          push dword ptr ss:[ebp+8]       ;  |hWnd
00445510                 |.  FFD6             call esi                        ;  \GetDlgItemTextA
00445512    2. ----->     |.  6A 7F            push 7F                         ;  /Count = 7F (127.)
00445514                 |.  BB 38E24800      mov ebx,winimage.0048E238       ;  |ASCII "dffdf"
00445519                 |.  53               push ebx                        ;  |Buffer => winimage.0048E238
0044551A                 |.  68 17080000      push 817                        ;  |ControlID = 817 (2071.)
0044551F                 |.  FF75 08          push dword ptr ss:[ebp+8]       ;  |hWnd
00445522                 |.  FFD6             call esi                        ;  \GetDlgItemTextA
00445524                 |.  57               push edi
00445525                 |.  BF 70DB4800      mov edi,winimage.0048DB70
0044552A                 |.  8BC3             mov eax,ebx
0044552C    3. ----->     |.  E8 2E720000      call winimage.0044C75F          ;  VERY SIMILAR
00445531                 |.  59               pop ecx
00445532                 |.  8B0D 70DB4800    mov ecx,dword ptr ds:[48DB70]
00445538                 |.  33D2             xor edx,edx
0044553A                 |.  3BC2             cmp eax,edx
0044553C                 |.  5F               pop edi
0044553D                 |.  A3 F8E54800      mov dword ptr ds:[48E5F8],eax
00445542                 |.  5B               pop ebx
00445543                 |.  74 06            je short winimage.0044554B
00445545                 |.  890D D4D74800    mov dword ptr ds:[48D7D4],ecx
0044554B                 |>  3915 D4D74800    cmp dword ptr ds:[48D7D4],edx
00445551                 |.  890D 90DE4800    mov dword ptr ds:[48DE90],ecx
00445557                 |.  75 05            jnz short winimage.0044555E
00445559                 |.  A3 90DE4800      mov dword ptr ds:[48DE90],eax
0044555E                 |>  33F6             xor esi,esi
00445560                 |.  46               inc esi
00445561                 |.  3BC2             cmp eax,edx
00445563    4. ----->     |.  75 29            jnz short winimage.0044558E    ------|   ;   -----> If Right, Then Jump
00445565                 |.  68 10200000      push 2010                      |
0044556A                 |.  68 2D040000      push 42D                       |
0044556F    5. ----->     |.  8935 F0E24800    mov dword ptr ds:[48E2F0],esi   |
00445575                 |.  8935 88DE4800    mov dword ptr ds:[48DE88],esi   |
0044557B                 |.  8815 38E24800    mov byte ptr ds:[48E238],dl     |
00445581                 |.  8815 B8DE4800    mov byte ptr ds:[48DEB8],dl     |
00445587                 |.  68 2B040000      push 42B                       |
0044558C                 |.  EB 1B            jmp short winimage.004455A9     |
0044558E                 |>  68 40200000      push 2040                <------|
00445593                 |.  68 2D040000      push 42D
00445598                 |.  8915 F0E24800    mov dword ptr ds:[48E2F0],edx
0044559E                 |.  8915 88DE4800    mov dword ptr ds:[48DE88],edx
004455A4                 |.  68 2A040000      push 42A
004455A9                 |>  FF75 08          push dword ptr ss:[ebp+8]
```
