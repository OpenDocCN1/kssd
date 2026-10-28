
# [原创]菜鸟对一病毒的详细分析~~~（愚人节快乐）

**文章标题：** 对一款木马的详细分析  
**作者：** 曹无咎  
**工具：** OD，PEid  
**时间：** 2011.4.1  
**样本：** 见附件  
**说明：** 菜鸟对一款木马的分析，失误之处还请大家指出

这是在 52pojie 样本区下载的病毒样本，之前也分析过一个启发小马，趁现在还有空闲时间，抓紧分析分析。

（宿舍的兄弟都在准备考研，我一个人在宿舍很是寂寞，嘿嘿~~）

## 第一步：查壳

先用 PEid 查壳，发现是款压缩壳：WinUpack 0.32-0.39 final by Dwing，这个壳脱壳很简单，单步即可脱之，我们来到 `OEP`：

```
0041EAE0  /.  55            push    ebp
0041EAE1  |.  8BEC          mov     ebp, esp
0041EAE3  |.  81EC 18040000 sub     esp, 418
```

到达 `OEP` 后，我们就可以单步 F8 分析这个病毒到底做了哪些事情。

## 第二步：OD 载入分析

OD 载入：

```
0041EAE0  /.  55            push    ebp
0041EAE1  |.  8BEC          mov     ebp, esp
0041EAE3  |.  81EC 18040000 sub     esp, 418
0041EAE9  |.  53            push    ebx
0041EAEA  |.  56            push    esi
0041EAEB  |.  57            push    edi
0041EAEC  |.  60            pushad
0041EAED  |.  83C0 12       add     eax, 12
0041EAF0  |.  83C3 13       add     ebx, 13
0041EAF3  |.  83C1 14       add     ecx, 14
0041EAF6  |.  61            popad
0041EAF7  |.  90            nop
0041EAF8  |.  90            nop
0041EAF9  |.  03DA          add     ebx, edx
0041EAFB  |.  90            nop
0041EAFC  |.  90            nop
0041EAFD  |.  03DA          add     ebx, edx
0041EAFF  |.  90            nop
0041EB00  |.  F7D2          not     edx
0041EB02  |.  90            nop
0041EB03  |.  90            nop
0041EB04  |.  90            nop
0041EB05  |.  42            inc     edx
0041EB06  |.  90            nop
0041EB07  |.  90            nop
0041EB08  |.  03DA          add     ebx, edx
0041EB0A  |.  90            nop
0041EB0B  |.  90            nop
0041EB0C  |.  F7DA          neg     edx
0041EB0E  |.  90            nop
0041EB0F  |.  90            nop
0041EB10  |.  60            pushad
0041EB11  |.  33C0          xor     eax, eax
0041EB13  |>  90            /nop
0041EB14  |.  83EC 0C       |sub     esp, 0C
0041EB17  |.  83C0 0C       |add     eax, 0C
0041EB1A  |.  36:833C24 00  |cmp     dword ptr [esp], 0
0041EB1F  |.^ 74 F2         \je      short 0041EB13
0041EB21  |.  03E0          add     esp, eax
0041EB23  |.  61            popad
0041EB24  |.  90            nop
0041EB25  |.  90            nop
0041EB26  |.  03DA          add     ebx, edx
0041EB28  |.  90            nop
0041EB29  |.  90            nop
0041EB2A  |.  03DA          add     ebx, edx
0041EB2C  |.  90            nop
0041EB2D  |.  F7D2          not     edx
0041EB2F  |.  90            nop
0041EB30  |.  90            nop
0041EB31  |.  90            nop
0041EB32  |.  42            inc     edx
0041EB33  |.  90            nop
0041EB34  |.  90            nop
0041EB35  |.  03DA          add     ebx, edx
0041EB37  |.  90            nop
0041EB38  |.  90            nop
0041EB39  |.  F7DA          neg     edx
0041EB3B  |.  90            nop
0041EB3C  |.  90            nop
0041EB3D  |.  60            pushad
0041EB3E  |.  81E9 99000000 sub     ecx, 99
0041EB44  |.  90            nop
0041EB45  |.  35 99000000   xor     eax, 99
0041EB4A  |.  90            nop
0041EB4B  |.  2D 99000000   sub     eax, 99
0041EB50  |.  61            popad
0041EB51  |.  90            nop
0041EB52  |.  90            nop
0041EB53  |.  03D3          add     edx, ebx
0041EB55  |.  90            nop
0041EB56  |.  90            nop
0041EB57  |.  03D3          add     edx, ebx
0041EB59  |.  90            nop
0041EB5A  |.  F7D3          not     ebx
0041EB5C  |.  90            nop
0041EB5D  |.  90            nop
0041EB5E  |.  90            nop
0041EB5F  |.  43            inc     ebx
0041EB60  |.  90            nop
0041EB61  |.  90            nop
0041EB62  |.  03D3          add     edx, ebx
0041EB64  |.  90            nop
0041EB65  |.  90            nop
0041EB66  |.  F7DB          neg     ebx
0041EB68  |.  90            nop
0041EB69  |.  90            nop
```

这段代码是在进行重定位，确定 `kernel.dll` 的地址，这个编码有点奇怪~~~

继续向下分析，主程序会创建一个互斥量 `mlgbcao`（名字有点不和谐），用来防止重复启动：

```
0041EB6A  |.  68 D4CE4100   push    0041CED4                         ;  ASCII "MLGBCAO."
0041EB6F  |.  6A 00         push    0
0041EB71  |.  6A 00         push    0
0041EB73  |.  FF15 5C104000 call    dword ptr [40105C]               kernel.CreateMutexA
0041EB79  |.  FF15 18104000 call    dword ptr [401018]               获取最近一次一异常
0041EB7F  |.  3D B7000000   cmp     eax, 0B7                         eax与0B7h作比较
0041EB84  |.  75 08         jnz     short 0041EB8E                   不等则跳，这里跳走了，我们改标志位不让他跳
0041EB86  |.  6A 00         push    0
0041EB88  |.  FF15 58104000 call    dword ptr [401058]               exitprocess，退出函数，一定要跳走
```

接下来，程序会继续定位函数地址，获取现在正在活动的窗口的窗口句柄并显示窗口：

```
0041EB8E    90              nop
0041EB8F    90              nop
0041EB90    03C8            add ecx,eax
0041EB92    90              nop
0041EB93    90              nop
0041EB94    03C8            add ecx,eax
0041EB96    90              nop
0041EB97    F7D0            not eax
0041EB99    90              nop
0041EB9A    90              nop
0041EB9B    90              nop
0041EB9C    40              inc eax
0041EB9D    90              nop
0041EB9E    90              nop
0041EB9F    03C8            add ecx,eax
0041EBA1    90              nop
0041EBA2    90              nop
0041EBA3    F7D8            neg eax
0041EBA5    90              nop
0041EBA6    90              nop
0041EBA7    B9 3F000000     mov ecx,3F
0041EBAC    33C0            xor eax,eax
0041EBAE    8DBD E8FBFFFF   lea edi,dword ptr ss:[ebp-418]
0041EBB4    F3:AB           rep stos dword ptr es:[edi]
0041EBB6    66:AB           stos word ptr es:[edi]
0041EBB8    AA              stos byte ptr es:[edi]
0041EBB9    FF15 C4104000   call dword ptr ds:[<&USER32.GetActiveWindow>]    ; user32.GetActiveWindow
0041EBBF    90              nop
0041EBC0    90              nop
0041EBC1    03C8            add ecx,eax
0041EBC3    90              nop
0041EBC4    90              nop
0041EBC5    03C8            add ecx,eax
0041EBC7    90              nop
0041EBC8    F7D0            not eax
0041EBCA    90              nop
0041EBCB    90              nop
0041EBCC    90              nop
0041EBCD    40              inc eax
0041EBCE    90              nop
0041EBCF    90              nop
0041EBD0    03C8            add ecx,eax
0041EBD2    90              nop
0041EBD3    90              nop
0041EBD4    F7D8            neg eax
0041EBD6    90              nop
0041EBD7    90              nop
0041EBD8    60              pushad
0041EBD9    81E9 99000000   sub ecx,99
0041EBDF    90              nop
0041EBE0    35 99000000     xor eax,99
0041EBE5    90              nop
0041EBE6    2D 99000000     sub eax,99
0041EBEB    61              popad
0041EBEC    90              nop
0041EBED    90              nop
0041EBEE    03D3            add edx,ebx
0041EBF0    90              nop
0041EBF1    90              nop
0041EBF2    03D3            add edx,ebx
0041EBF4    90              nop
0041EBF5    F7D3            not ebx
0041EBF7    90              nop
0041EBF8    90              nop
0041EBF9    90              nop
0041EBFA    43              inc ebx
0041EBFB    90              nop
0041EBFC    90              nop
0041EBFD    03D3            add edx,ebx
0041EBFF    90              nop
0041EC00    90              nop
0041EC01    F7DB            neg ebx
0041EC03    90              nop
0041EC04    90              nop
0041EC05    FF15 C4104000   call dword ptr ds:[<&USER32.GetActiveWindow>]    ; user32.GetActiveWindow
0041EC0B    6A 05           push 5
0041EC0D    50              push eax
0041EC0E    FF15 B8104000   call dword ptr ds:[<&USER32.ShowWindow>]         ; user32.ShowWindow
```

ShowWindow：

```
0012FB94   00000000  |hWnd = NULL
0012FB98   00000005  \ShowState = SW_SHOW
```

继续单步向下走：

```
0041EC14    60              pushad
0041EC15    81E9 99000000   sub ecx,99
0041EC1B    90              nop
0041EC1C    35 99000000     xor eax,99
0041EC21    90              nop
0041EC22    2D 99000000     sub eax,99
0041EC27    61              popad
0041EC28    68 F09AB86F     push 6FB89AF0
0041EC2D    6A 01           push 1
```
