
# Nokia 手机工具箱 v1.1.9

> 原文：[`www.pediy.com/kssd/pediy03/forum677.htm`](https://www.pediy.com/kssd/pediy03/forum677.htm)

请高手指点

软件名称：`Nktools`(手机工具箱) 繁体版 441KB

软件功能：Nokia 手机工具箱程序，`v1.1.9`，繁体中文未注册版。集合设置中文电话簿，中文简讯，备忘录，开工控模式，修改待机、分组图案，自定铃声，设置手机时间、闹铃时间，修改 `PIN1` 码为一体的 Nokia 手机程序。 支持手机连接线、电脑红外线。支持手机类型：`3210`、`3310`、`5110`、`5130`、`6110`、`6138`、`6150`、`8210`、`8250`、`8810`、`8850`。未注册版有功能限制，如工程模式、开机画面、铃声等不能使用，但可以用中文电本功能

注册形式：要求输入注册码，与手机的串号(`IMEI`)运算得出想应串号(`IMEI`) ，注册不成功不会出现出错信息，它会将注册码存放在软件目录的 `regcode.txt` 里，此类软件应该都是即时检测注册码的，而且每个功能都会检测注册码，所以如果要暴力的话很多地方要改，最好可以找到真注册码并写出注册机(具体是如何运算我还搞不清楚)

下载地址：`http://go.163.com/~nokiaz/software/NKToolSetup.exe`

不知大家有谁是用诺基亚的手机，`nokia` 只有 `3310` 和 `8250` 是可以用中文电话本的，但现在还有很多人都是用 `6110`、`6150`、`8210` 的，用了上面的软件就可以让这些手机都能用上中文电话本，而且还有很多其它功能，有兴趣的人试试吧，此软件要用红外线或者数据线。

此软件用 `upx 0.93` 压缩过，解压后反汇编，因为是繁体的所以我找不到有用的东西，但我找到个“`autoregister`”我设了中断后程序会自动中断，就算不作任何动作都会中断，所以我想程序是在即时检测注册码，但我跟下去却找不到关键地方(我只是个新手，功力 0.5 级)，以下的地方是我用 `trw200` 调试时找到的，我作了些注解，不知其它地方还有没有计算注册码的地方，在以下的过程中，我发现程序用我输入的假注册码和"060347221N”字符串作了运算，不知此字符串是不是程序将我的手机串号计算而来的，所以请大家帮验证一下，我只追到了程序的 `41A90C` 处到了后面我就晕掉了(我的汇编知识太菜了)，而且我也没追到“060347221N”是如何来的，所以请各位朋友帮个忙如果可以做个注册机出来就太好啦，实在不行也请帮忙在下面写一些注解吧!

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:0041A734(C)` |
|

```
:0041A739 C1FA02               sar edx, 02
:0041A73C 8955BC               mov dword ptr [ebp-44], edx
:0041A73F 33C9                 xor ecx, ecx
:0041A741 894DB0               mov dword ptr [ebp-50], ecx
:0041A744 8B45B0               mov eax, dword ptr [ebp-50]
:0041A747 3B45BC               cmp eax, dword ptr [ebp-44]
:0041A74A 0F8DC1000000         jnl 0041A811

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|
| :0041A80B(C)
|

:0041A750 8B55C8               mov edx, dword ptr [ebp-38]
:0041A753 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A756 8A048A               mov al, byte ptr [edx+4*ecx] <------假注册码首位数
:0041A759 04D0                 add al, D0
:0041A75B 8B55C8               mov edx, dword ptr [ebp-38]
:0041A75E 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A761 8A548A01             mov dl, byte ptr [edx+4*ecx+01]<----假注册码第二位
:0041A765 80C2D0               add dl, D0
:0041A768 C1E206               shl edx, 06
:0041A76B 0AC2                 or al, dl
:0041A76D 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A770 8D0C49               lea ecx, dword ptr [ecx+2*ecx]
:0041A773 8B55C0               mov edx, dword ptr [ebp-40]
:0041A776 88040A               mov byte ptr [edx+ecx], al <---结果写入此地址
:0041A779 8B45BC               mov eax, dword ptr [ebp-44]
:0041A77C 85C0                 test eax, eax
:0041A77E 7903                 jns 0041A783
:0041A780 83C003               add eax, 00000003

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|
| :0041A77E(C)
|

:0041A783 C1F802               sar eax, 02
:0041A786 8D0C40               lea ecx, dword ptr [eax+2*eax]
:0041A789 41                    inc ecx
:0041A78A 83C104               add ecx, 00000004
:0041A78D 894D9C               mov dword ptr [ebp-64], ecx
:0041A790 33C0                 xor eax, eax
:0041A792 8945A0               mov dword ptr [ebp-60], eax
:0041A795 DF6D9C               fild qword ptr [ebp-64]   -----|此处何解
:0041A798 DC0524AA4100         fadd qword ptr [0041AA24]     |是否浮点
:0041A79E DD5DA4               fstp qword ptr [ebp-5C]   -----|运算？？
:0041A7A1 8B55C8               mov edx, dword ptr [ebp-38]
:0041A7A4 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A7A7 33C0                 xor eax, eax
:0041A7A9 8A448A01             mov al, byte ptr [edx+4*ecx+01]
:0041A7AD 83C0D0               add eax, FFFFFFD0
:0041A7B0 C1F802               sar eax, 02
:0041A7B3 8B55C8               mov edx, dword ptr [ebp-38]
:0041A7B6 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A7B9 8A548A02             mov dl, byte ptr [edx+4*ecx+02]
:0041A7BD 80C2D0               add dl, D0
:0041A7C0 C1E204               shl edx, 04
:0041A7C3 0AC2                 or al, dl
:0041A7C5 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A7C8 8D0C49               lea ecx, dword ptr [ecx+2*ecx]
:0041A7CB 8B55C0               mov edx, dword ptr [ebp-40]
:0041A7CE 88440A01             mov byte ptr [edx+ecx+01], al
:0041A7D2 8B45C8               mov eax, dword ptr [ebp-38]
:0041A7D5 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A7D8 0FB6448802           movzx eax, byte ptr [eax+4*ecx+02]
:0041A7DD 83C0D0               add eax, FFFFFFD0
:0041A7E0 C1F804               sar eax, 04
:0041A7E3 8B55C8               mov edx, dword ptr [ebp-38]
:0041A7E6 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A7E9 8A548A03             mov dl, byte ptr [edx+4*ecx+03]
:0041A7ED 80C2D0               add dl, D0
:0041A7F0 C1E202               shl edx, 02
:0041A7F3 0AC2                 or al, dl
:0041A7F5 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A7F8 8D0C49               lea ecx, dword ptr [ecx+2*ecx]
:0041A7FB 8B55C0               mov edx, dword ptr [ebp-40]
:0041A7FE 88440A02             mov byte ptr [edx+ecx+02], al
:0041A802 FF45B0               inc [ebp-50]
:0041A805 8B45B0               mov eax, dword ptr [ebp-50]
:0041A808 3B45BC               cmp eax, dword ptr [ebp-44]
:0041A80B 0F8C3FFFFFFF         jl 0041A750

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|
| :0041A74A(C)
|

:0041A811 8B4DBC               mov ecx, dword ptr [ebp-44]
:0041A814 8D0C49               lea ecx, dword ptr [ecx+2*ecx]
:0041A817 894DBC               mov dword ptr [ebp-44], ecx
:0041A81A 8B45B8               mov eax, dword ptr [ebp-48]
:0041A81D 0345BC               add eax, dword ptr [ebp-44]
:0041A820 8945B4               mov dword ptr [ebp-4C], eax
:0041A823 8B55B4               mov edx, dword ptr [ebp-4C]
:0041A826 4A                   dec edx
:0041A827 8955B0               mov dword ptr [ebp-50], edx
:0041A82A 837DB000             cmp dword ptr [ebp-50], 00000000
:0041A82E 0F8CC4000000         jl 0041A8F8

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|
| :0041A8F2(C)
|

:0041A834 33C9                 xor ecx, ecx
:0041A836 894DAC               mov dword ptr [ebp-54], ecx
:0041A839 EB36                 jmp 0041A871

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|
| :0041A878(C)
|

:0041A83B 8B45AC               mov eax, dword ptr [ebp-54]
:0041A83E 0345B0               add eax, dword ptr [ebp-50]
:0041A841 99                   cdq
:0041A842 F77DB8               idiv [ebp-48]
:0041A845 8B4DC4               mov ecx, dword ptr [ebp-3C]
:0041A848 8B45C0               mov eax, dword ptr [ebp-40]
:0041A84B 8B5DAC               mov ebx, dword ptr [ebp-54]
:0041A84E 0FB6441801           movzx eax, byte ptr [eax+ebx+01]
:0041A853 0FAF45B0             imul eax, dword ptr [ebp-50]
:0041A857 8B5DC0               mov ebx, dword ptr [ebp-40]
:0041A85A 8B75AC               mov esi, dword ptr [ebp-54]
```
