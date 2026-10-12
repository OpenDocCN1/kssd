
# WinBowl Version 3.2

> 原文：[`www.pediy.com/kssd/pediy05/pediy50323.htm`](https://www.pediy.com/kssd/pediy05/pediy50323.htm)

**破解作者**：`master`

**软件名称**：`WinBowl Version 3.2`

**软件简介**：一款打保龄球的休闲娱乐游戏

**下载地址**：`http://members.aol.com/revtjb/wb32.exe`

**破解工具**：`TRW2000` & `W32Dasm`

**破解过程**：用 `TRW` 载入 `Winbwl32.exe`；点注册随便填入注册信息；激活 `TRW` 下 `bpx hmemcpy`；点 `0K`；拦后下 `pmodule`；`F10` 来到：

```
:0040D8ED 57                       push edi

:0040D8EE E8AD590100               call 004232A0

:0040D8F3 83C404                   add esp, 00000004

:0040D8F6 85C0                     test eax, eax<-----------------测试输入的注册码是否为空

:0040D8F8 7536                     jne 0040D930<------------------不为空则跳

:0040D8FA 6800010000               push 00000100

:0040D8FF BFC0D34300               mov edi, 0043D3C0

:0040D904 57                       push edi
```

跳到此处：

```
:0040D930 8D45E0                   lea eax, dword ptr [ebp-20]<---EAX 为输入的假码

:0040D933 50                       push eax

:0040D934 E897800100               call 004259D0<-----------------处理假注册码，将其转化为一个十六进制数

:0040D939 668BD8                   mov bx, ax<--------------------在此处下? EAX 显示假注册码的十六进制数

:0040D93C 83C404                   add esp, 00000004

:0040D93F 53                       push ebx

:0040D940 E893FAFFFF               call 0040D3D8

:0040D945 83C404                   add esp, 00000004

:0040D948 57                       push edi

* Possible Reference to String Resource ID=00032: "Unregistered Shareware"

:0040D949 6A20                     push 00000020

* Possible Reference to String Resource ID=00013: "Don't give up!"

:0040D94B 6A0D                     push 0000000D

:0040D94D BF07040000               mov edi, 00000407

:0040D952 57                       push edi

:0040D953 FF7508                   push [ebp+08]

:0040D956 FFD6                     call esi

:0040D958 50                       push eax

* Reference To: USER32.SendMessageA, Ord:01C6h

:0040D959 FF15BCF64300             Call dword ptr [0043F6BC]

:0040D95F 8D45E0                   lea eax, dword ptr [ebp-20]<----EAX 为输入的注册名

:0040D962 50                       push eax

:0040D963 E838590100               call 004232A0

:0040D968 83C404                   add esp, 00000004

:0040D96B 85C0                     test eax, eax<------------------测试注册名是否为空

:0040D96D 7536                     jne 0040D9A5<-------------------不为空则跳

:0040D96F 6800010000               push 00000100

:0040D974 BBC0D34300               mov ebx, 0043D3C0

:0040D979 53                       push ebx

:0040D97A A11CD84300               mov eax, dword ptr [0043D81C]

* Possible Reference to String Resource ID=00039: "You must enter your name."

:0040D97F 6A27                     push 00000027

:0040D981 50                       push eax

* Reference To: USER32.LoadStringA, Ord:0177h

:0040D982 FF15E4F64300             Call dword ptr [0043F6E4]

:0040D988 53                       push ebx

:0040D989 6A00                     push 00000000

:0040D98B E882B60000               call 00419012

:0040D990 83C408                   add esp, 00000008

:0040D993 57                       push edi

:0040D994 FF7508                   push [ebp+08]

:0040D997 FFD6                     call esi

:0040D999 50                       push eax

* Reference To: USER32.SetFocus, Ord:01E1h

:0040D99A FF1594F74300             Call dword ptr [0043F794]

:0040D9A0 E9DF010000               jmp 0040DB84
```

跳到这里：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040D96D(C)
|

:0040D9A5 8D45E0                   lea eax, dword ptr [ebp-20]

:0040D9A8 50                       push eax

:0040D9A9 E86AFAFFFF               call 0040D418

:0040D9AE 83C404                   add esp, 00000004

:0040D9B1 E8C4FAFFFF               call 0040D47A<--------计算注册码关键处,跟进

:0040D9B6 85C0                     test eax, eax<--------测试注册标记

:0040D9B8 0F840B010000             je 0040DAC9<----------注册失败则跳转

:0040D9BE 53                       push ebx

:0040D9BF 8D45E0                   lea eax, dword ptr [ebp-20]

:0040D9C2 50                       push eax

....................

....................
```

显示成功注册对话框：

```
* Possible Reference to String Resource ID=00043: "Your game has now been registered. Save your code - it can b"

:0040DA27 6A2B                     push 0000002B

:0040DA29 50                       push eax
```

下面将假注册码处理为一个十六进制处,由 `0040D934 E897800100 call 004259D0` 到此：

```
============================================================================

:004259D0 8B442404                 mov eax, dword ptr [esp+04]

:004259D4 50                       push eax

:004259D5 E846FFFFFF               call 00425920<--------想 CALL 就进去

:004259DA 83C404                   add esp, 00000004

:004259DD C3                       ret

CALL 到此：

:00425920 53                       push ebx

:00425921 56                       push esi

:00425922 8B74240C                 mov esi, dword ptr [esp+0C]

:00425926 57                       push edi

:00425927 55                       push ebp

.....................

.....................

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0042596A(C)
|

:00425971 33DB                     xor ebx, ebx

:00425973 8A1E                     mov bl, byte ptr [esi]<----取一位注册码存入 BL

:00425975 46                       inc esi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0042596F(C)
|

:00425976 33ED                     xor ebp, ebp<--------------累加器清零

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004259AF(U)
|

:00425978 833DEC6E430001           cmp dword ptr [00436EEC], 00000001

:0042597F 7E0D                     jle 0042598E

..............

..............

:0042598E 8B0DE06C4300             mov ecx, dword ptr [00436CE0]

:00425994 33C0                     xor eax, eax

:00425996 668B0459                 mov ax, word ptr [ecx+2*ebx]

:0042599A 83E004                   and eax, 00000004

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0042598C(U)
|

:0042599D 85C0                     test eax, eax

:0042599F 7410                     je 004259B1<-----------------------处理完毕跳出去

:004259A1 8D44AD00                 lea eax, dword ptr [ebp+4*ebp]<----计算处理注册码

:004259A5 46                       inc esi<---------------------------注册码移一位

:004259A6 8D6C43D0                 lea ebp, dword ptr [ebx+2*eax-30]<-计算处理注册码

:004259AA 33DB                     xor ebx, ebx<----------------------清空 EBX

:004259AC 8A5EFF                   mov bl, byte ptr [esi-01]<---------取下一位注册码

:004259AF EBC7                     jmp 00425978

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0042599F(C)
|

:004259B1 8BC5                     mov eax, ebp<----------------------EBP 为假注册码的十六进制数

..............

:004259BE C3                       ret
```

下面处理注册名及比较处，由 `0040D9B1 E8C4FAFFFF call 0040D47A` 到此：

```
================================================================

:0040D47A 66C70508BC43000000        mov word ptr [0043BC08], 0000

:0040D483 56                       push esi

:0040D484 57                       push edi

:0040D485 E893020000               call 0040D71D

:0040D48A 668B1508BC4300           mov dx, word ptr [0043BC08]<--计算一个值 X 开始处

:0040D491 B9FF000000               mov ecx, 000000FF

:0040D496 668BC2                   mov ax, dx

:0040D499 66C1EA08                 shr dx, 08
```
