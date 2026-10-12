
# psde_demo_mj V1.4

> 原文：[`www.pediy.com/kssd/pediy06/pediy6525.htm`](https://www.pediy.com/kssd/pediy06/pediy6525.htm)

呵呵，看完前一篇了吗？来，第二篇

【破解作者】WXHing

【作者邮箱】`WXHing@163.com`

【使用工具】`OllyDbg1.09`

【破解平台】`Win9x/NT/2000/XP`

【软件名称】`psde_demo_mj` V1.4

【下载地址】中国共享软件注册中心

【软件简介】这个是一个用来演示如何用 PSDE 制作窗口游戏的软件，起本身就是一个小游戏，挺有意思的。

【软件大小】`272Kb`

【加壳方式】注册码保护

【破解声明】初学破解，妄谈破解，不当之处还请各位前辈指教！

--------------------------------------------------------------------------------

【破解内容】

首先用 `OLLYDBG` 装载程序，按 `F9` 运行，

输入用户名 `WXHing`

试练码 `987654321`

然后下断点 `bp GetDlgItem`，确定，拦住，按 `F8` 直到这里

```
00402E76    .   FFD3               call ebx                               ; \SendDlgItemMessageA          ;eax 返回用户名长度
00402E78    .   8B15 7C974000      mov edx, dword ptr ds:[40977C]                         ；从这里出来
00402E7E    .   52                 push edx                           ; /lParam => 12FA14
00402E7F    .   6A 10              push 10                           ; |wParam = 10
00402E81    .   6A 0D              push 0D                           ; |Message = WM_GETTEXT
00402E83    .   68 E9030000        push 3E9                          ; |ControlID = 3E9 (1001.)
00402E88    .   55                 push ebp                           ; |hWnd
00402E89    .   FFD3               call ebx                               ; \SendDlgItemMessageA
00402E8B    .   A1 88974000        mov eax, dword ptr ds:[409788]         ；获得用户名，eax 返回
00402E90    .   8038 00            cmp byte ptr ds:[eax], 0              ；用户名为空则出错
00402E93    .   0F84 38010000      je PSDE_DEM.00402FD1
00402E99    .   8B0D 7C974000      mov ecx, dword ptr ds:[40977C]        ；ecx 指向试练码
00402E9F    .   8039 00            cmp byte ptr ds:[ecx], 0              ；没输入则出错
00402EA2    .   0F84 29010000      je PSDE_DEM.00402FD1
00402EA8    .   50                 push eax                           ；用户名入栈
00402EA9    .   E8 22FeFFFF        call PSDE_DEM.00402CD0              ；关键，跟进
00402EAE    .   8B3D 7C974000      mov edi, dword ptr ds:[40977C]        ;edi 指向试练码
00402EB4    .   A1 88974000        mov eax, dword ptr ds:[409788]       ；eax 为真正的注册码
00402EB9    .   83C4 04            add esp, 4
00402EBC    .   8BF7               mov esi, edi
00402EBE    >   8A10               mov dl, byte ptr ds:[eax]
00402EC0    .   8ACA               mov cl, dl
00402EC2    .   3A16               cmp dl, byte ptr ds:[esi]          ；逐位比较注册码是否正确
00402EC4    .   75 1C              jnz short PSDE_DEM.00402EE2     ；不等则出错
00402EC6    .   84C9               test cl, cl
00402EC8    .   74 14              je short PSDE_DEM.00402EDE
00402ECA    .   8A50 01            mov dl, byte ptr ds:[eax+1]
00402ECD    .   8ACA               mov cl, dl
00402ECF    .   3A56 01            cmp dl, byte ptr ds:[esi+1]
00402ED2    .   75 0E              jnz short PSDE_DEM.00402EE2     ；不等则出错
00402ED4    .   83C0 02            add eax, 2
00402ED7    .   83C6 02            add esi, 2
00402EDA    .   84C9               test cl, cl
00402EDC    .   ^ 75 E0            jnz short PSDE_DEM.00402EBE
00402EDE    >   33C0               xor eax, eax
00402EE0    .   EB 05              jmp short PSDE_DEM.00402EE7
00402EE2    >   1BC0               sbb eax, eax
00402EE4    .   83D8 FF            sbb eax, -1
00402EE7    >   85C0               test eax, eax
00402EE9    .   0F84 8D000000      je PSDE_DEM.00402F7C
..............
..............
```

进入 `402EA9` 处的 `call`,

```
00402CD0   /$  53                 push ebx                               ；来到这里
00402CD1   |.   56                 push esi
00402CD2   |.   57                 push edi
00402CD3   |.   8B7C24 10          mov edi, dword ptr ss:[esp+10]        ；edi 指向用户名
00402CD7   |.   32DB               xor bl, bl                             ；bl 清零
00402CD9   |.   8BCF               mov ecx, edi                           ；ecx=edi
00402CDB   |.   8A07               mov al, byte ptr ds:[edi]             ;al 取用户名字符
00402CDD   |.   84C0               test al, al                            ；等于 0 吗
00402CDF   |.   74 0A              je short PSDE_DEM.00402CEB            ；为零则跳走
00402CE1   |>  02D8               /add bl, al                             ；bl=bl+al
00402CE3   |.   8A41 01            |mov al, byte ptr ds:[ecx+1]          ;al 指向下一字符
00402CE6   |.   41                 |inc ecx                               ；ecx 指向下一字符
00402CE7   |.   84C0               |test al, al                           ；取完了吗
00402CE9   |.^ 75 F6               \jnz short PSDE_DEM.00402CE1          ；没有则继续
00402CEB   |>  A1 78974000        mov eax, dword ptr ds:[409778]
00402CF0   |.   33F6               xor esi, esi
00402CF2   |.   A3 84974000        mov dword ptr ds:[409784], eax
00402CF7   |.   A1 74974000        mov eax, dword ptr ds:[409774]
00402CFC   |.   85C0               test eax, eax
00402CFE   |.   7E 2D              jle short PSDE_DEM.00402D2D
00402D00   |>  8A0C3E             /mov cl, byte ptr ds:[esi+edi]        ；取用户名字符
00402D03   |.   32CB               |xor cl, bl      ；cl 与 bl 异或，存入 cl,bl 为用户名字符 16 进制和低字节
00402D05   |.   51                 |push ecx                           ；ecx 入栈
00402D06   |.   E8 95FFFFFF        |call PSDE_DEM.00402CA0                ；关键，跟进
00402D0B   |.   83C4 04            |add esp, 4
00402D0E   |.   88043E             |mov byte ptr ds:[esi+edi], al
00402D11   |.   3C 0A              |cmp al, 0A                          ；al>=0A 吗
00402D13   |.   0FBEC0             |movsx eax, al                       ；扩展存入 eax
00402D16   |.   7D 05              |jge short PSDE_DEM.00402D1D         ;大于等于则跳
00402D18   |.   83C0 30            |add eax, 30                          ；eax=eax+30
00402D1B   |.   EB 03              |jmp short PSDE_DEM.00402D20
00402D1D   |>  83C0 41            |add eax, 41                           ;eax=eax+41
00402D20   |>  88043E             |mov byte ptr ds:[esi+edi], al        ;计算结果返回并存入 edi 指向的地址，esi 为计数器
00402D23   |.   A1 74974000        |mov eax, dword ptr ds:[409774]       ;eax=8
00402D28   |.   46                 |inc esi                               ;esi=esi+1
00402D29   |.   3BF0               |cmp esi, eax                           ;esi>8 吗？
00402D2B   |.^ 7C D3               \jl short PSDE_DEM.00402D00            ；小于则继续
00402D2D   |>  C60438 00          mov byte ptr ds:[eax+edi], 0
00402D31   |.   5F                 pop edi
00402D32   |.   5E                 pop esi
00402D33   |.   5B                 pop ebx
00402D34   \.   C3                 retn
```

进入 `402D06` 处的 `call`,

```
00402CA0   /$  0FBE4424 04        movsx eax, byte ptr ss:[esp+4]         ;eax 等于用户名字符与用户名字符和的异或值
00402CA5   |.   0305 84974000    add eax, dword ptr ds:[409784]         ；eax=eax+989681 ,[409784]初始值为 0x989881
00402CAB   |.   69C0 697DAE42    imul eax, eax, 42AE7D69               ；eax 等于 eax*42ae7d69 的低八位
00402CB1   |.   05 31D40000      add eax, 0D431                       ;eax=eax+0d431
00402CB6   |.   A3 84974000      mov dword ptr ds:[409784], eax         ；eax 计算后的值存入[409784]，下次循环时调用
00402CBB   |.   C1F8 10          sar eax, 10                           ；eax 右移 4 位
00402CBE   |.   83E0 0F          and eax, 0F                           ；eax=eax AND 0F
00402CC1   \.   C3               retn
```



以我的名字 `WXHing` 为例，字符 16 进制和为 0x57+0x58+0x48+0x69+0x6E+0x67=0xEB，取低 2 位为 0x35，

### 第一字符‘W’的变换

```
eax=57 xor 35 =62,
eax=62+[409784]=62+989681=9896E3
eax=9896E3*42AE7D69=27BEE7D48CBA1B,取低八位为 D48CBA1B
eax=D48CBA1B+0D431=D48D8E4C , [409784]=D48D8E4C
eax=D48D8E4C SAR 10=D48D
eax=D48D AND 0F=0D  > 0A
所以 eax=0D+41=4E,    转换为字符是'N'
```

### ‘X’的变换

```
eax=58 xor 35 =6D,
eax=6D+[409784]=62+D48D8E4C=D48D8EB9
eax=D48D8EB9*42AE7D69=375D5F23007DDEE1,取低八位为 007DDEE1
eax=007DDEE1+0D431=007EB312 , [409784]=007EB312
eax=007EB312 SAR 10=007E
eax=007E AND 0F=0E  > 0A
所以 eax=0D+41=4E,    转换为字符是'O'
```

### ‘H’的变换

```
eax=48 xor 35 =7D,
eax=7D+[409784]=7D+007EB312=7EB38F
eax=7EB38F*42AE7D69=2100A6FAD678A7,取低八位为 FAD678A7
eax=FAD678A7+0D431=FAD74CD8 , [409784]=FAD74CD8
eax=FAD74CD8 SAR 10=FAD7
eax=FAD7 AND 0F=07 < 0A
所以 eax=0D+30=37,    转换为字符是'7'
```

在下面就不分析了，`i`，`n`，`g`，分别转换为`9`、`6`、`6`，

因为注册码为 8 个字符，而用户名`WXHing`只有 6 个，所以要补上二个 0，分别转换为`N`、`8`

将以上字符连在一起就是注册码`NO7966N8`

收工！

--------------------------------------------------------------------------------

## 【破解总结】

这个软件的保护很脆弱，还是采用明码保护的，但是我主要是为了研究它的算法，所以才罗里罗嗦的写了那么一大篇。

最后还是非常非常的感谢您看完本文！

--------------------------------------------------------------------------------

## 【版权声明】

本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
