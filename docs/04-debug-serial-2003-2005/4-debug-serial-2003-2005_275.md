
# SWF 探索者 XP V1.11.2002.326

> 原文：[`www.pediy.com/kssd/pediy05/pediy50027.htm`](https://www.pediy.com/kssd/pediy05/pediy50027.htm)

## SWF 探索者(SWFExplorer)XP V1.11.2002.326 破解过程

破解撰写：`leeyam[BCG]`

运行该程序，随意输入注册信息，提示需要重新启动软件验证注册码。

判断该程序先将输入的注册码存放某个位置，然后启动时调用！

用 `Language` 发现是用 `PECompact` 加的壳，用 `UnPECompact` 自动脱壳。再用 `PEditor` 载入脱壳后的程序，选择"sections"修改 `Pec1` 为 `.data`。然后利用 `W32Dasm` 反编，查找字串，发现程序会调用注册表，双击进入第一个调入：

```
* Possible StringData Ref from Data Obj ->"Software\SWFExplorer"
                                    |
:004B4CF4 BA804E4B00                mov edx, 004B4E80
:004B4CF9 A1F8ED4B00                mov eax, dword ptr [004BEDF8]
:004B4CFE E86DCEFBFF                call 00471B70
:004B4D03 8D4DF4                    lea ecx, dword ptr [ebp-0C]

* Possible StringData Ref from Data Obj ->"UserName"
                                    |
:004B4D06 BAA04E4B00                mov edx, 004B4EA0
:004B4D0B A1F8ED4B00                mov eax, dword ptr [004BEDF8]
:004B4D10 E823D0FBFF                call 00471D38
:004B4D15 8B55F4                    mov edx, dword ptr [ebp-0C]
:004B4D18 B8E8ED4B00                mov eax, 004BEDE8
:004B4D1D E8B2F8F4FF                call 004045D4
:004B4D22 8D4DF0                    lea ecx, dword ptr [ebp-10]

* Possible StringData Ref from Data Obj ->"RegCode"
                                    |
:004B4D25 BAB44E4B00                mov edx, 004B4EB4
:004B4D2A A1F8ED4B00                mov eax, dword ptr [004BEDF8]
:004B4D2F E804D0FBFF                call 00471D38
:004B4D34 8B55F0                    mov edx, dword ptr [ebp-10]
:004B4D37 B8ECED4B00                mov eax, 004BEDEC
:004B4D3C E893F8F4FF                call 004045D4
:004B4D41 A1F8ED4B00                mov eax, dword ptr [004BEDF8]
:004B4D46 E891CDFBFF                call 00471ADC
:004B4D4B B101                      mov cl, 01

* Possible StringData Ref from Data Obj ->"Software\SWFExplorer"
                                    |
:004B4D4D BA804E4B00                mov edx, 004B4E80
:004B4D52 A1F8ED4B00                mov eax, dword ptr [004BEDF8]
:004B4D57 E814CEFBFF                call 00471B70

* Possible StringData Ref from Data Obj ->"TrialDate"
                                    |
:004B4D5C BAC44E4B00                mov edx, 004B4EC4
:004B4D61 A1F8ED4B00                mov eax, dword ptr [004BEDF8]
:004B4D66 E891D1FBFF                call 00471EFC
:004B4D6B 84C0                      test al, al
:004B4D6D 751B                      jne 004B4D8A
:004B4D6F E87C5AF5FF                call 0040A7F0
:004B4D74 83C4F8                    add esp, FFFFFFF8
:004B4D77 DD1C24                    fstp qword ptr [esp]
:004B4D7A 9B                        wait

* Possible StringData Ref from Data Obj ->"TrialDate"
                                    |
:004B4D7B BAC44E4B00                mov edx, 004B4EC4
:004B4D80 A1F8ED4B00                mov eax, dword ptr [004BEDF8]
:004B4D85 E86ED0FBFF                call 00471DF8

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B4D6D(C)
|

* Possible StringData Ref from Data Obj ->"TrialDate"
                                    |
:004B4D8A BAC44E4B00                mov edx, 004B4EC4
:004B4D8F A1F8ED4B00                mov eax, dword ptr [004BEDF8]
:004B4D94 E873D0FBFF                call 00471E0C
:004B4D99 DD5DE8                    fstp qword ptr [ebp-18]
:004B4D9C 9B                        wait
:004B4D9D E84E5AF5FF                call 0040A7F0
:004B4DA2 DC5DE8                    fcomp qword ptr [ebp-18]
:004B4DA5 DFE0                      fstsw ax
:004B4DA7 9E                        sahf
:004B4DA8 7236                      jb 004B4DE0
:004B4DAA E8415AF5FF                call 0040A7F0
:004B4DAF 83C4F8                    add esp, FFFFFFF8
:004B4DB2 DD1C24                    fstp qword ptr [esp]
:004B4DB5 9B                        wait

* Possible StringData Ref from Data Obj ->"TrialDate"
                                    |
:004B4DB6 BAC44E4B00                mov edx, 004B4EC4
:004B4DBB A1F8ED4B00                mov eax, dword ptr [004BEDF8]
:004B4DC0 E847D0FBFF                call 00471E0C
:004B4DC5 83C4F8                    add esp, FFFFFFF8
:004B4DC8 DD1C24                    fstp qword ptr [esp]
:004B4DCB 9B                        wait
:004B4DCC E88F0CFFFF                call 004A5A60
:004B4DD1 BA1E000000                mov edx, 0000001E
:004B4DD6 2BD0                      sub edx, eax
:004B4DD8 8915F0ED4B00             mov dword ptr [004BEDF0], edx
:004B4DDE EB07                      jmp 004B4DE7

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B4DA8(C)
|

:004B4DE0 33C0                      xor eax, eax
:004B4DE2 A3F0ED4B00               mov dword ptr [004BEDF0], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B4DDE(U)
|

:004B4DE7 33C0                      xor eax, eax
:004B4DE9 5A                        pop edx
:004B4DEA 59                        pop ecx
:004B4DEB 59                        pop ecx
:004B4DEC 648910                    mov dword ptr fs:[eax], edx
:004B4DEF 68064E4B00                push 004B4E06

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B4E04(U)
|

:004B4DF4 A1F8ED4B00                mov eax, dword ptr [004BEDF8]
:004B4DF9 E816EAF4FF                call 00403814
:004B4DFE C3                        ret
:004B4DFF E9A4F1F4FF                jmp 00403FA8
:004B4E04 EBEE                      jmp 004B4DF4
:004B4E06 8D45E0                    lea eax, dword ptr [ebp-20]

* Possible StringData Ref from Data Obj ->"1.2.2002.326"
                                    |
:004B4E09 BAD84E4B00                mov edx, 004B4ED8
:004B4E0E 8A12                      mov dl, byte ptr [edx]
:004B4E10 E84BF9F4FF                call 00404760
:004B4E15 8B45E0                    mov eax, dword ptr [ebp-20]
:004B4E18 50                        push eax
:004B4E19 8D45E4                    lea eax, dword ptr [ebp-1C]
:004B4E1C 50                        push eax

* Possible StringData Ref from Data Obj ->"SWFExplorer"
                                    |
:004B4E1D B9F04E4B00                mov ecx, 004B4EF0

* Possible StringData Ref from Data Obj ->"Cloud Lee"
                                    |
:004B4E22 BA044F4B00                mov edx, 004B4F04
:004B4E27 A1E8ED4B00                mov eax, dword ptr [004BEDE8]
:004B4E2C E8DFF9FFFF                call 004B4810
:004B4E31 8B55E4                    mov edx, dword ptr [ebp-1C]…………………………调入真码
:004B4E34 A1ECED4B00                mov eax, dword ptr [004BEDEC]…………………………调入假码
:004B4E39 E83EFBF4FF                call 0040497C…………………………比较
:004B4E3E 0F9405E0ED4B00            sete byte ptr [004BEDE0]
```

看到这里眼前一亮，发现上面 `004B4E39` 的 `Call` 可疑，于是开始用 `TRW2000` 直接下中断 `bpx 4b4e39` 顺利拦截，`D edx` 看见真码。 `4dd466d0e1e0cc123afbe504da2462ca`
