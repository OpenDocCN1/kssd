
# ufony

> 原文：[`www.pediy.com/kssd/pediy06/pediy6526.htm`](https://www.pediy.com/kssd/pediy06/pediy6526.htm)

ufony 破解记录

`kongfoo` / `2004.6.6`

由于要将 2 首 WMA 转成 MP3，找到了这个软件。不注册只能转换 60 秒的音乐，哪有这样做限制的？气愤中。。。

`PEiD` 查壳 `tElock` 0.98b1，这个壳不错啊。想一下搞音频转换算法是少不了的，这个软件好像不易对付呢。。。

目的在于搞掂限制，尽快脱壳。

直接查 `OEP`：`4135e4`。`LordPE` dump full，`ImpREC` 有一堆函数无效，用 `tElock 98` / `tElock` 插件都不行。`trace level2` 找到了几个。`trace level3` 又找到几个。最后剩 6 个，`OD` attach 上去分析一下。

第 1 个 `3e0011`：

```
003E0011    B8 8DC44300     MOV EAX,43C48D
003E0016    90              NOP
003E0017    FF30            PUSH DWORD PTR DS:[EAX]
003E0019    C3              RETN
```

`43c48d` 处是 0，哪么这个项可以 cut 掉了。跟着的 3 个和 `3e0011` 的代码一样，都 cut 掉。剩下 2 个指向程序本身空间，不管。`fix dump`。脱出来可以运行。

`OD` 载入脱壳后的程序，倒，程序自带的 DLL 都上了壳。。。

注册窗口点确定后什么反应都没有，果然没有估计错误。由于 DLL 的壳（都是 `tElock`）没脱去，直接在 `OD` 运行很烦，所以用 `attach` 的办法。下断 `GetWindowTextA`。没断到，一狠心，把 `hlclass` 的函数全断了。光下断就用了近 10 分钟:(

这一招可谓必杀:)点确定后断下：

```
0040BE31    FF15 BCE24300   CALL DWORD PTR DS:[<&hlclass.??0CRegistr>; hlclass.??0CRegistry@@QAE@XZ  ==这里断下
0040BE37    8D4C24 44       LEA ECX,DWORD PTR SS:[ESP+44]
0040BE3B    C64424 64 02    MOV BYTE PTR SS:[ESP+64],2
0040BE40    FF15 2CE14300   CALL DWORD PTR DS:[<&hlclass.??0CString@>; hlclass.??0CString@@QAE@XZ
0040BE46    8D4C24 4C       LEA ECX,DWORD PTR SS:[ESP+4C]
0040BE4A    C64424 64 03    MOV BYTE PTR SS:[ESP+64],3
0040BE4F    FF15 B8E24300   CALL DWORD PTR DS:[<&hlclass.??0CMenu@@Q>; hlclass.??0CMenu@@QAE@XZ
0040BE55    8B3D F0E04300   MOV EDI,DWORD PTR DS:[<&hlclass.?IsEmpty>; hlclass.?IsEmpty@CString@@QAEHXZ
0040BE5B    8D4C24 20       LEA ECX,DWORD PTR SS:[ESP+20]
0040BE5F    C64424 64 04    MOV BYTE PTR SS:[ESP+64],4
0040BE64    FFD7            CALL EDI
0040BE66    85C0            TEST EAX,EAX
0040BE68    0F85 9F000000   JNZ dumped\_.0040BF0D
0040BE6E    8D4C24 18       LEA ECX,DWORD PTR SS:[ESP+18]
0040BE72    FFD7            CALL EDI
0040BE74    85C0            TEST EAX,EAX
0040BE76    0F85 91000000   JNZ dumped\_.0040BF0D
0040BE7C    8B3D C4E04300   MOV EDI,DWORD PTR DS:[<&hlclass.??0CStri>; hlclass.??0CString@@QAE@PAD@Z
0040BE82    83EC 08         SUB ESP,8
0040BE85    8BCC            MOV ECX,ESP
0040BE87    896424 18       MOV DWORD PTR SS:[ESP+18],ESP
0040BE8B    68 34EB4100     PUSH dumped\_.0041EB34                     ; ASCII "Software\\Ufony\\Registration"
0040BE90    FFD7            CALL EDI
0040BE92    68 01000080     PUSH 80000001
0040BE97    8D4C24 34       LEA ECX,DWORD PTR SS:[ESP+34]
0040BE9B    FF15 4CE44300   CALL DWORD PTR DS:[<&hlclass.?CreateKey@>; hlclass.?CreateKey@CRegistry@@QAEJPAUHKEY\_\_@@VCString@@@Z
0040BEA1    8B2D B8E34300   MOV EBP,DWORD PTR DS:[<&hlclass.??0CStri>; hlclass.??0CString@@QAE@AAV0@@Z
0040BEA7    83EC 08         SUB ESP,8
0040BEAA    8D5424 28       LEA EDX,DWORD PTR SS:[ESP+28]
0040BEAE    8BCC            MOV ECX,ESP
0040BEB0    896424 18       MOV DWORD PTR SS:[ESP+18],ESP
0040BEB4    52              PUSH EDX
0040BEB5    FFD5            CALL EBP
0040BEB7    83EC 08         SUB ESP,8
0040BEBA    8BCC            MOV ECX,ESP
0040BEBC    896424 24       MOV DWORD PTR SS:[ESP+24],ESP
0040BEC0    68 28EB4100     PUSH dumped\_.0041EB28                     ; ASCII "UserName"
0040BEC5    C64424 78 05    MOV BYTE PTR SS:[ESP+78],5
0040BECA    FFD7            CALL EDI
0040BECC    8B1D 48E44300   MOV EBX,DWORD PTR DS:[<&hlclass.?SetValu>; hlclass.?SetValue@CRegistry@@QAEJVCString@@0@Z
0040BED2    8D4C24 38       LEA ECX,DWORD PTR SS:[ESP+38]
0040BED6    C64424 74 04    MOV BYTE PTR SS:[ESP+74],4
0040BEDB    FFD3            CALL EBX
0040BEDD    83EC 08         SUB ESP,8
0040BEE0    8D4424 20       LEA EAX,DWORD PTR SS:[ESP+20]
0040BEE4    8BCC            MOV ECX,ESP
0040BEE6    896424 1C       MOV DWORD PTR SS:[ESP+1C],ESP
0040BEEA    50              PUSH EAX
0040BEEB    FFD5            CALL EBP
0040BEED    83EC 08         SUB ESP,8
0040BEF0    8BCC            MOV ECX,ESP
0040BEF2    896424 20       MOV DWORD PTR SS:[ESP+20],ESP
0040BEF6    68 20EB4100     PUSH dumped\_.0041EB20                     ; ASCII "RegCode"
0040BEFB    C64424 78 06    MOV BYTE PTR SS:[ESP+78],6
0040BF00    FFD7            CALL EDI
0040BF02    8D4C24 38       LEA ECX,DWORD PTR SS:[ESP+38]
0040BF06    C64424 74 04    MOV BYTE PTR SS:[ESP+74],4
0040BF0B    FFD3            CALL EBX
0040BF0D    8B4C24 70       MOV ECX,DWORD PTR SS:[ESP+70]
0040BF11    8B5424 6C       MOV EDX,DWORD PTR SS:[ESP+6C]
0040BF15    51              PUSH ECX
0040BF16    52              PUSH EDX
0040BF17    8BCE            MOV ECX,ESI
0040BF19    FF15 20E44300   CALL DWORD PTR DS:[<&hlclass.?OnCancelCl>; hlclass.?OnOKClick@CDialog@@QAEJIJ@Z
0040BF1F    8D4C24 4C       LEA ECX,DWORD PTR SS:[ESP+4C]
0040BF23    C64424 64 08    MOV BYTE PTR SS:[ESP+64],8
0040BF28    FF15 98E24300   CALL DWORD PTR DS:[<&hlclass.??1CMenu@@U>; hlclass.??1CMenu@@UAE@XZ
0040BF2E    8B35 9CE04300   MOV ESI,DWORD PTR DS:[<&hlclass.??1CStri>; hlclass.??1CString@@QAE@XZ
0040BF34    8D4C24 44       LEA ECX,DWORD PTR SS:[ESP+44]
0040BF38    C64424 64 07    MOV BYTE PTR SS:[ESP+64],7
0040BF3D    FFD6            CALL ESI
0040BF3F    8D4C24 28       LEA ECX,DWORD PTR SS:[ESP+28]
0040BF43    C64424 64 01    MOV BYTE PTR SS:[ESP+64],1
0040BF48    FF15 94E24300   CALL DWORD PTR DS:[<&hlclass.??1CRegistr>; hlclass.??1CRegistry@@QAE@XZ
0040BF4E    8D4C24 18       LEA ECX,DWORD PTR SS:[ESP+18]
0040BF52    C64424 64 00    MOV BYTE PTR SS:[ESP+64],0
0040BF57    FFD6            CALL ESI
0040BF59    8D4C24 20       LEA ECX,DWORD PTR SS:[ESP+20]
0040BF5D    C74424 64 FFFFF>MOV DWORD PTR SS:[ESP+64],-1
0040BF65    FFD6            CALL ESI
0040BF67    8B4C24 5C       MOV ECX,DWORD PTR SS:[ESP+5C]
0040BF6B    5F              POP EDI
0040BF6C    5E              POP ESI
0040BF6D    5D              POP EBP
0040BF6E    33C0            XOR EAX,EAX
0040BF70    64:890D 0000000>MOV DWORD PTR FS:[0],ECX
0040BF77    5B              POP EBX
0040BF78    83C4 58         ADD ESP,58
0040BF7B    C2 0800         RETN 8
```

只是将用户名和 sn 直接放到注册表。。。在调用转换窗口时会弹出限制信息窗口，应该会判断是否注册，下断 `hlclass` 所有注册表操作函数。断下：

```
0040A612    FF15 BCE24300   CALL DWORD PTR DS:[<&hlclass.??0CRegistr>; hlclass.??0CRegistry@@QAE@XZ

0040A64E    E8 CD660000     CALL dumped\_.00410D20
0040A653    85C0            TEST EAX,EAX
0040A655   /74 41           JZ SHORT dumped\_.0040A698 ==令这个跳不跳就可以跳过限制信息窗口，但只能转换 1 分钟的限制还在。
```

很明显 `410d20` 就是判断是否注册的函数了。
