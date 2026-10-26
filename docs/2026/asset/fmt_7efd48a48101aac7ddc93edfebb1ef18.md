
## 代码片段

```
00581FAD    59                    pop ecx

00581FAE    59                    pop ecx

00581FAF    64:8910               mov dword ptr fs:[eax],edx

00581FB2    EB 0A                 jmp short dumped\_.00581FBE

00581FB4  ^ E9 731DE8FF           jmp dumped\_.00403D2C

00581FB9    E8 D620E8FF           call dumped\_.00404094

00581FBE    8B45 F8               mov eax,dword ptr ss:[ebp-8]           ; 清零，eax=00000000

00581FC1    E8 8618E8FF           call dumped\_.0040384C

00581FC6    33C0                  xor eax,eax                            ; 异或清零，eax=00000000

00581FC8    5A                    pop edx

00581FC9    59                    pop ecx

00581FCA    59                    pop ecx

00581FCB    64:8910               mov dword ptr fs:[eax],edx

00581FCE    68 F01F5800           push dumped\_.00581FF0

00581FD3    8D45 EC               lea eax,dword ptr ss:[ebp-14]

00581FD6    BA 03000000           mov edx,3

00581FDB    E8 8026E8FF           call dumped\_.00404660

00581FE0    8D45 FC               lea eax,dword ptr ss:[ebp-4]           ; 清零，eax=00000000

00581FE3    E8 5426E8FF           call dumped\_.0040463C

00581FE8    C3                    retn

00581FE9  ^ E9 F21FE8FF           jmp dumped\_.00403FE0

00581FEE  ^ EB E3                 jmp short dumped\_.00581FD3

00581FF0    5F                    pop edi                                ; dumped\_.00470850

00581FF1    5E                    pop esi

00581FF2    5B                    pop ebx

00581FF3    8BE5                  mov esp,ebp

00581FF5    5D                    pop ebp

00581FF6    C3                    retn                                   ; 返回程序并执行命令
```

返回到这里：(★★★★★)

```
00584BAA    E8 19F9E7FF           call dumped\_.004044C8                  ; 程序到这里，就已经执行该命令了，Game Over ~
00584BAF    E8 E0D4FFFF           call dumped\_.00582094
00584BB4    84C0                  test al,al
......
```

## 程序“自杀”原因 | 批处理文件内容

```batch
:loop

if exist "D:\文章试验品\图章制作系统\dumped\_.exe" del "D:\文章试验品\图章制作系统\dumped\_.exe"

if exist "D:\文章试验品\图章制作系统\dumped\_.exe" goto loop

if not exist "D:\文章试验品\图章制作系统\dumped\_.exe" del "c:\autoexec1.bat"
```

--------------------------------------------------------------------------------------------

## 本章总结

作者同样采用CRC冗余代码校验方式，检测程序是否已遭受破解，狠心的是在检测程序完整性失败完后(非脱壳校验失败)，调用“autoexec.bat”批处理命令以及系统配置文件“system.ini”，在后台随机删除一个系统文件，从而达到程序避免遭受破解的可能性，由之加大了对破解者机器的威胁，而检测是时时刻刻存在的，所以一定要分析完后再做修改！

提醒一点：在脱壳未去校验前，千万请勿对程序作任何代码修改，避免不必要的事件发生！

--------------------------------------------------------------------------------------------

版权所有（C）2005 KuNgBiM[DFCG]         Copyright (C) 2005 KuNgBiM[DFCG]

--------------------------------------------------------------------------------------------

     UnPacked & Cracked By KuNgBiM[DFCG]

                2005-08-01

                23:09:18 PM

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
