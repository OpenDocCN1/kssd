
00A8874E 57 push `edi`

……F8

00A88768 33C0 xor `eax`, `eax`

00A8876A 8A46 07 mov `al`, `[esi+7]`

00A8876D 8B5483 40 mov `edx`, `[ebx+eax*4+40]`

00A88771 8BC7 mov `eax`, `edi`

00A88773 FFD2 call `edx`

如果 x 为内存操作数（`[00401235]`），该 call 求出 00401235，否则为 0

00A88775 894424 04 mov `[esp+4]`, `eax`

00A88779 33C0 xor `eax`, `eax`

00A8877B 8A46 05 mov `al`, `[esi+5]`

00A8877E 8B5483 40 mov `edx`, `[ebx+eax*4+40]`

00A8882 8BC7 mov `eax`, `edi`

00A88784 FFD2 call `edx`

如果 x 为寄存器，该 call 求出哪个寄存器（EXX），否则为大于 8 的值

(0=`eax`,1=`ecx`,2=`edx`,3=`ebx`,4=`esp`,5=`ebp`,6=`esi`,7=`edi`)

00A88786 8BD0 mov `edx`, `eax`

00A88788 80EA 08 sub `dl`, 8

00A8878B 0F92C2 setb `dl`

00A8878E 80FA 01 cmp `dl`, 1

00A88791 75 11 jnz short 00A887A4

若 x 不是寄存器跳 00A887A4(该地址接下来处理 y)

00A88793 8BC8 mov `ecx`, `eax`

00A88795 8B1424 mov `edx`, `[esp]`

00A88798 8BC3 mov `eax`, `ebx`

00A8879A E8 75020000 call 00A88A14

该 call 把 x EXX 中值取出

00A8879F 8BE8 mov `ebp`, `eax`

00A887A1 EB 01 jmp short 00A887A4

00A887A4 33C0 xor `eax`, `eax`

00A887A6 8A46 08 mov `al`, `[esi+8]`

00A887A9 8B5483 40 mov `edx`, `[ebx+eax*4+40]`

00A887AD 8BC7 mov `eax`, `edi`

00A887AF FFD2 call `edx`

如果 y 为内存操作数（`[00401235]`），该 call 求出 00401235，如果 y 为直接操作数（3D），该 call 求出 3D，否则为 0

00A887B1 894424 08 mov `[esp+8]`, `eax`

00A887B5 33C0 xor `eax`, `eax`

00A887B7 8A46 06 mov `al`, `[esi+6]`

00A887BA 8B5483 40 mov `edx`, `[ebx+eax*4+40]`

00A887BE 8BC7 mov `eax`, `edi`

00A887C0 FFD2 call `edx`

如果 y 为寄存器，该 call 求出哪个寄存器（EXX），否则为大于 8 的值

00A887C2 8BD0 mov `edx`, `eax`

00A887C4 80EA 08 sub `dl`, 8

00A887C7 0F92C2 setb `dl`

00A887CA 80FA 01 cmp `dl`, 1

00A887CD 75 13 jnz short 00A887E2 //

00A887CF 8BC8 mov `ecx`, `eax`

00A887D1 8B1424 mov `edx`, `[esp]`

00A887D4 8BC3 mov `eax`, `ebx`

00A887D6 E8 39020000 call 00A88A14

该 call 把 y EXX 中值取出

00A887DB 894424 0C mov `[esp+C]`, `eax`

00A887DF /EB 01 jmp short 00A887E2

00A887E2 036C24 04 add `ebp`, `[esp+4]`

00A887E2 036C24 04 add `ebp`, `[esp+4]`

00A887E6 8B4424 08 mov `eax`, `[esp+8]`

00A887EA 034424 0C add `eax`, `[esp+C]`

00A887EE 894424 10 mov `[esp+10]`, `eax`

00A887F2 EB 01 jmp short 00A887F5

00A887F5 33C0 xor `eax`, `eax`

00A887F7 8A46 09 mov `al`, `[esi+9]`

00A887FA 8B5483 40 mov `edx`, `[ebx+eax*4+40]`

00A887FE 8BC7 mov `eax`, `edi`

00A88800 FFD2 call `edx`

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

该 call 很有意思，返回后有 5 种情况：

若 `eax`=0： `cmp dword ptr [x], y`

若 `eax`=1： `cmp x,[ y]`

若 `eax`=2： `cmp byte ptr [x], y`

若 `eax`=3： `cmp x,byte ptr [ y]`

若 `eax`=4： `cmp x, y`

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

……

00A88862 8B5424 10 mov `edx`, `[esp+10]`

00A88866 8BC5 mov `eax`, `ebp`

00A88868 E8 CBFEFFFF call 00A88738

该 call 把比较 x,y，得到标志位

00A8886D 83C4 14 add `esp`, 14

00A88870 5D pop `ebp`

00A88871 5F pop `edi`

00A88872 5E pop `esi`

00A88873 5B pop `ebx`

00A88874 C3 retn

}

00A8897D 33C0 xor `eax`, `eax`

00A8897F 8A43 04 mov `al`, `[ebx+4]`

00A88982 8B55 F8 mov `edx`, `[ebp-8]`

00A88985 8B5482 40 mov `edx`, `[edx+eax*4+40]`

00A88989 8BC6 mov `eax`, `esi`

00A8898B FFD2 call `edx`

该 call 返回后是跳转类型：

`eax` = 0,1,2,3,4,5….分别对应机器码 70，71，72，73…..

00A8898D 8BD8 mov `ebx`, `eax`

00A8898F 8B4D 10 mov `ecx`, `[ebp+10]`

00A88992 8BD3 mov `edx`, `ebx`

00A88994 8B45 F8 mov `eax`, `[ebp-8]`

00A88997 E8 74FBFFFF call 00A88510

在 00A88937 call 00A88510 进行比较

00A8899C 84C0 test `al`, `al`

00A8899E 74 17 je short 00A889B7

00A8893E 处如果不跳的话可来到：

00A88949 0345 F4 add `eax`, `[ebp-C]`

00A8894C 8B55 F8 mov `edx`, `[ebp-8]`

00A8894F 0342 68 add `eax`, `[edx+68]`

00A88952 EB 7B jmp short 00A889CF

00A8894F 处 `eax` 就是原 jxx 跳后的地址。

跳的话可来到：

00A88957 8B45 F8 mov `eax`, `[ebp-8]`

00A8895A 8B40 18 mov `eax`, `[eax+18]`

00A8895D 03C7 add `eax`, `edi`

00A8895F 8B55 F8 mov `edx`, `[ebp-8]`

00A88962 0342 68 add `eax`, `[edx+68]`

00A88965 EB 68 jmp short 00A889CF

00A88962 处 `eax` 就是原 jxx 不跳后的地址。

}//第二层返回

}//第一层返回

具体修复方法相信大家看了后比我清楚，不多说了。。。。。。

`ASProtect.SKE.2.11` 到此已基本脱壳成功！

我的脱壳经历：

1. 找 OEP
2. 还原 IAT
3. 到达 OEP 后修复 `Advanced Import protection`
4. 到达 OEP 后修复 `stolen code` 里的 `call` 变形
5. `DUMP`

这其中还有一个要提的是：我脱的那个软件就是个贱！加壳时使用了次数限制，为此花了我两三个小时来解决它。大家遇到时小心。。。。。。

`Wak` 4.21 2006

4dd466d0e1e0cc123afbe504da2462ca
