
# 带壳分析易语言+简单浮点算法 [靓仔系统保护专家]

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-351.htm`](https://www.pediy.com/kssd/pediy07/pediy7-351.htm)

**【破文名称】** 带壳分析易语言+简单浮点算法 靓仔系统保护专家

**【软件名称】** 靓仔系统保护专家 1.34 版!

**【软件介绍】** 靓仔系统保护专家是一款小巧而功能强大的系统保护软件，可以保护您的系统安全，支持现今所有的 windows 操作系统，本软件有三大功能模块！！系统安全锁功能：能够强制锁定计算机并支持远程解锁（通过因特网对目标计算机进行远程解锁），初始密码为空，建议用户使用后马上更改密码，文件加密功能：用户可采用强大的 RC4 或 DES 算法对任何类型的文件进行加密！！伪装文件夹：可以让你分不清文件夹的真伪,让您文件夹里的数据更加安全!

**【软件地址】** [`www4.skycn.com/soft/15386.html`](http://www4.skycn.com/soft/15386.html)

**【破文作者】** KiLlL\[DFCG\]

**【破解时间】** 2005-7-13

**【简要说明】** 可以看作简单的易语言入门 CRACKME

## 破解过程

`peid` 查壳，知道是 ASPack 2.12 -&gt; Alexey Solodovnikov \[覆盖\]，对于我壳盲来说，先用 `peid` 的 `general unpack` 试试，可以脱，但是脱了运行提示：文件数据错误！这个不好玩！如果对易语言有点了解，估计可以知道是易语言编写的。

各位大哥，小弟不知如何脱壳，只有硬着头皮来带壳分析了

唯一的好处就是知道了语言，易语言的弱点就是下断了，且看：

`od` 载入后，`ALT+M`,看看是不是有 `ecode`？运行，提示是否注册，点注册，弹出注册窗口。

好，输入假码 `123` 之后进行 `alt+m`，盯准 `ecode` 狠狠按下 `F2`，点注册，ok，断了，成功了一半：

```
00413C08    55               push ebp                       ; 程序断下了
00413C09    8BEC             mov ebp,esp
00413C0B    81EC 34000000    sub esp,34
00413C11    6A FF            push -1
00413C13    6A 0A            push 0A
00413C15    68 DB000116      push 160100DB
00413C1A    68 D8000152      push 520100D8
00413C1F    E8 2F100000      call lzbhzj.00414C53
00413C24    83C4 10          add esp,10
00413C27    8945 FC          mov dword ptr ss:\[ebp\-4\],eax
省略一些
00413CCC    83C4 04          add esp,4
00413CCF    837D F8 00       cmp dword ptr ss:\[ebp\-8\],0
00413CD3    0F84 D9040000    je lzbhzj.004141B2
00413CD9    6A FF            push -1
00413CDB    6A 08            push 8
00413CDD    68 DD000116      push 160100DD
00413CE2    68 D8000152      push 520100D8
00413CE7    E8 670F0000      call lzbhzj.00414C53
00413CEC    83C4 10          add esp,10
00413CEF    8945 FC          mov dword ptr ss:\[ebp\-4\],eax     ; 读取假码 123
00413CF2    68 CA914000      push lzbhzj.004091CA
00413CF7    FF75 FC          push dword ptr ss:\[ebp\-4\]
00413CFA    E8 E5B1FFFF      call lzbhzj.0040EEE4
00413CFF    83C4 08          add esp,8
00413D02    83F8 00          cmp eax,0
00413D05    B8 00000000      mov eax,0
00413D0A    0F94C0           sete al
00413D0D    8945 F8          mov dword ptr ss:\[ebp\-8\],eax
00413D10    8B5D FC          mov ebx,dword ptr ss:\[ebp\-4\]
00413D13    85DB             test ebx,ebx                     ; 判断是否为空
00413D15    74 09            je short lzbhzj.00413D20
00413D17    53               push ebx
00413D18    E8 300F0000      call lzbhzj.00414C4D
00413D1D    83C4 04          add esp,4
00413D20    837D F8 00       cmp dword ptr ss:\[ebp\-8\],0
00413D24    0F84 61000000    je lzbhzj.00413D8B
00413D2A    68 04000080      push 80000004
00413D2F    6A 00            push 0
00413D31    68 D3A24000      push lzbhzj.0040A2D3
00413D36    68 01030080      push 80000301
00413D3B    6A 00            push 0
00413D3D    68 40000000      push 40
00413D42    68 04000080      push 80000004
00413D47    6A 00            push 0
00413D49    68 DAA24000      push lzbhzj.0040A2DA
00413D4E    68 03000000      push 3
00413D53    BB 00030000      mov ebx,300
00413D58    E8 E40E0000      call lzbhzj.00414C41
00413D5D    83C4 28          add esp,28
00413D60    68 05000100      push 10005
00413D65    68 DD000116      push 160100DD
00413D6A    68 D8000152      push 520100D8
00413D6F    68 01000000      push 1
00413D74    BB 64030000      mov ebx,364
00413D79    E8 C30E0000      call lzbhzj.00414C41
00413D7E    83C4 10          add esp,10
00413D81    E9 2C040000      jmp lzbhzj.004141B2
00413D86    E9 22040000      jmp lzbhzj.004141AD
00413D8B    DB05 200DD600    fild dword ptr ds:\[D60D20\]       ; 机器码 1347647860
00413D91    DD5D F8          fstp qword ptr ss:\[ebp\-8\]
00413D94    DD45 F8          fld qword ptr ss:\[ebp\-8\]
00413D97    DC35 1C924000    fdiv qword ptr ds:\[40921C\]       ; /67 固定数
00413D9D    DD5D F8          fstp qword ptr ss:\[ebp\-8\]        ; st=20114147.164179105280
00413DA0    DD45 F8          fld qword ptr ss:\[ebp\-8\]
00413DA3    DC05 24924000    fadd qword ptr ds:\[409224\]       ; 加上固定数 8762454
00413DA9    DD5D F0          fstp qword ptr ss:\[ebp\-10\]
00413DAC    DD45 F0          fld qword ptr ss:\[ebp\-10\]        ; st=28876601.164179107840
00413DAF    DC0D 1C924000    fmul qword ptr ds:\[40921C\]       ; \*67 固定数
00413DB5    DD5D E8          fstp qword ptr ss:\[ebp\-18\]
00413DB8    6A 00            push 0
00413DBA    6A 00            push 0
00413DBC    6A 00            push 0
00413DBE    68 01060080      push 80000601
00413DC3    FF75 EC          push dword ptr ss:\[ebp\-14\]
00413DC6    FF75 E8          push dword ptr ss:\[ebp\-18\]
00413DC9    68 02000000      push 2
00413DCE    BB 68000000      mov ebx,68
00413DD3    E8 690E0000      call lzbhzj.00414C41
00413DD8    83C4 1C          add esp,1C
00413DDB    8945 D8          mov dword ptr ss:\[ebp\-28\],eax
00413DDE    8955 DC          mov dword ptr ss:\[ebp\-24\],edx     ; 1104991338
00413DE1    DD45 D8          fld qword ptr ss:\[ebp\-28\]         ; 计算结果 1934732278.000000
00413DE4    E8 98B1FFFF      call lzbhzj.0040EF81
00413DE9    A3 240DD600      mov dword ptr ds:\[D60D24\],eax     ; 转成十六进制 eax=7351ABF6
00413DEE    DB05 200DD600    fild dword ptr ds:\[D60D20\]         ; 机器码 1104991338
00413DF4    DD5D F8          fstp qword ptr ss:\[ebp\-8\]
00413DF7    DD45 F8          fld qword ptr ss:\[ebp\-8\]
00413DFA    DC35 1C924000    fdiv qword ptr ds:\[40921C\]       ; /67
00413E00    DD5D F8          fstp qword ptr ss:\[ebp\-8\]        ; st=20114147.164179105280
00413E03    DD45 F8          fld qword ptr ss:\[ebp\-8\]
00413E06    DC05 24924000    fadd qword ptr ds:\[409224\]       ; 加上固定数 8762454
00413E0C    DD5D F0          fstp qword ptr ss:\[ebp\-10\]
00413E0F    DD45 F0          fld qword ptr ss:\[ebp\-10\]
00413E12    DC0D 1C924000    fmul qword ptr ds:\[40921C\]       ; 再乘以 67
00413E18    DD5D E8          fstp qword ptr ss:\[ebp\-18\]       ; 结果：1934732278
00413E1B    6A 00            push 0                               ; 怎么又计算了一次？BUG
00413E1D    6A 00            push 0
00413E1F    6A 00            push 0
00413E21    68 01060080      push 80000601
00413E26    FF75 EC          push dword ptr ss:\[ebp\-14\]
00413E29    FF75 E8          push dword ptr ss:\[ebp\-18\]
00413E2C    68 02000000      push 2
00413E31    BB 68000000      mov ebx,68
00413E36    E8 060E0000      call lzbhzj.00414C41
00413E3B    83C4 1C          add esp,1C
00413E3E    8945 E0          mov dword ptr ss:\[ebp\-20\],eax
00413E41    8955 E4          mov dword ptr ss:\[ebp\-1C\],edx
00413E44    6A FF            push -1
00413E46    6A 08            push 8
00413E48    68 DD000116      push 160100DD
00413E4D    68 D8000152      push 520100D8
00413E52    E8 FC0D0000      call lzbhzj.00414C53
00413E57    83C4 10          add esp,10
00413E5A    8945 DC          mov dword ptr ss:\[ebp\-24\],eax     ; 假码 123
00413E5D    68 04000080      push 80000004
00413E62    6A 00            push 0
00413E64    8B45 DC          mov eax,dword ptr ss:\[ebp\-24\]
00413E67    85C0             test eax,eax
00413E69    75 05            jnz short lzbhzj.00413E70
00413E6B    B8 CA914000      mov eax,lzbhzj.004091CA
00413E70    50               push eax
00413E71    68 01000000      push 1
00413E76    BB 78010000      mov ebx,178
00413E7B    E8 C10D0000      call lzbhzj.00414C41
00413E80    83C4 10          add esp,10
00413E83    8945 D8          mov dword ptr ss:\[ebp\-28\],eax
00413E86    8B5D DC          mov ebx,dword ptr ss:\[ebp\-24\]
00413E89    85DB             test ebx,ebx
00413E8B    74 09            je short lzbhzj.00413E96
00413E8D    53               push ebx
00413E8E    E8 BA0D0000      call lzbhzj.00414C4D
00413E93    83C4 04          add esp,4
```
