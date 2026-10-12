
# 赢家股票盘后分析 1.2

> 原文：[`www.pediy.com/kssd/pediy03/forum1132.htm`](https://www.pediy.com/kssd/pediy03/forum1132.htm)

赢家股票盘后分析 1.2

by 6767 `[BCG]`

工具：`SoftIce`，`IceDump` (隐藏 `SI`)，`RegMon`，`Wdasm`

d/l：

简述：用江恩（上世纪初的一位投资大师）理论对股票趋势作分析。

用 `RegMon` 监视它的运行发现对这两个键有读写操作：

`HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\excel\"Gc_id"`

`HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\excel\"sex"`

其中 `"Gc_id"` 每次运行会加 5，到 100 时你就要注册了；`"sex"` 存放机器码。

这里面很有意思：当你直接改 `Gc_id` 的值，使它大于 100，重新运行时要求注册。如果 `sex` 的值是正确的机器码，你若注册成功，那就是真的成功了；如果 `sex` 的值不是正确的机器码（比如人为改动），注册成功后你可以运行，当你退出程序后，你会发现 `Gc_id` 的值比 `sex` 值大 10101，再次运行，提示不正确注册码。这时，`Gc_id` 的值变为 100，`sex` 的值变为与机器码相关的一个数。再次运行，出现注册窗口，窗口的标题就是你的机器码，记下这个数值。

注册过程：安装后运行一次，改变 `Gc_id` 的值，使它大于 100。再运行，到注册窗口了。随便填入些信息，在 `SI` 中下断点 `Bpx hmemcpy`。中断下来后跟踪，N 次 `F12` 到下面

`* Possible StringData Ref from Code Obj ->"非注册软件序列号："`

```
|
:0050373E 68343F5000               push 00503F34
:00503743 FFB3AC040000             push dword ptr [ebx+000004AC]
* Possible StringData Ref from Code Obj ->",请输入注册码："
|
:00503749 68503F5000               push 00503F50
:0050374E 8D45A0                   lea eax, dword ptr [ebp-60]
:00503751 BA03000000               mov edx, 00000003
:00503756 E80D09F0FF               call 00404068
:0050375B 8B45A0                   mov eax, dword ptr [ebp-60]
:0050375E 8D4DF0                   lea ecx, dword ptr [ebp-10]
:00503761 BA683F5000               mov edx, 00503F68
:00503766 E8B940F5FF               call 00457824
:0050376B 84C0                     test al, al             <- 返回处
:0050376D 0F840B010000             je 0050387E             <- 不会跳
:00503773 8BC3                     mov eax, ebx
:00503775 E8921A0000               call 0050520C
:0050377A 3C01                     cmp al, 01
:0050377C 750C                     jne 0050378A
:0050377E A1C0905000               mov eax, dword ptr [005090C0]
:00503783 8B00                     mov eax, dword ptr [eax]
:00503785 E806DAF4FF               call 00451190
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0050377C(C)
|
:0050378A 6A00                     push 00000000
:0050378C 6800E1F505               push 05F5E100             <- 0x5f5e100=100,000,000
:00503791 8B45F8                   mov eax, dword ptr [ebp-08]   <- eax=机器号码，显示在标题中
:00503794 03C0                     add eax, eax              <- eax*=2
:00503796 8945A4                   mov dword ptr [ebp-5C], eax
:00503799 DB45A4                   fild dword ptr [ebp-5C]   <-
:0050379C DB2D6C3F5000             fld tbyte ptr [00503F6C]    <- 3.14152696
:005037A2 DEC9                     fmulp st(1), st(0)        <- st(0)*=st(1)
:005037A4 D835783F5000             fdiv dword ptr [00503F78]   <- st(0)/=360
:005037AA E80DF3EFFF               call 00402ABC             <- st(0)=sin(st(0))

>>>>>>>>>>>>>>
|
:00402ABC D9FE                     fsin
:00402ABE DFE0                     fstsw ax
:00402AC0 9E                       sahf
:00402AC1 7A01                     jpe 00402AC4
:00402AC3 C3                       ret
:00402AC4 DDD8                     fstp st(0)
:00402AC6 D9EE                     fldz
:00402AC8 C3                       ret
<<<<<<<<<<<<<<<<

:005037AF DB2D7C3F5000             fld tbyte ptr [00503F7C]    <- st(0)=1.01
:005037B5 DEC1                     faddp st(1), st(0)          <- st(0)+=st(1)
:005037B7 6945F883000000           imul eax, dword ptr [ebp-08], 00000083<- eax=机器号码*131
:005037BE 89459C                   mov dword ptr [ebp-64], eax
:005037C1 DB459C                   fild dword ptr [ebp-64]
:005037C4 DEC9                     fmulp st(1), st(0)          <- st(0)*=st(1)
:005037C6 E81DF3EFFF               call 00402AE8               <- eax=st(0)

>>>>>>>>>>>>>>>
|
:00402AE8 83EC08                   sub esp, 00000008
:00402AEB DF3C24                   fistp qword ptr [esp]
:00402AEE 9B                       wait
:00402AEF 58                       pop eax
:00402AF0 5A                       pop edx
:00402AF1 C3                       ret
<<<<<<<<<<<<<

:005037CB E8CD39F0FF               call 0040719D               <- eax%=100,000,000
:005037D0 52                       push edx
:005037D1 50                       push eax
:005037D2 8D45EC                   lea eax, dword ptr [ebp-14]
:005037D5 E80260F0FF               call 004097DC               <- 将 eax 值存放于[ebp-14]
:005037DA 8B45EC                   mov eax, dword ptr [ebp-14]   <- 正确的注册码
:005037DD 8B55F0                   mov edx, dword ptr [ebp-10]   <- 输入的注册码
:005037E0 E8D308F0FF               call 004040B8               <- 比较
:005037E5 7563                     jne 0050384A                <- 相同则不跳走
:005037E7 8BC3                     mov eax, ebx
:005037E9 E81E1A0000               call 0050520C
:005037EE 3C01                     cmp al, 01
:005037F0 750C                     jne 005037FE
:005037F2 A1C0905000               mov eax, dword ptr [005090C0]
:005037F7 8B00                     mov eax, dword ptr [eax]
......
:00503841 8BC6                     mov eax, esi
:00503843 E818F8EFFF               call 00403060
:00503848 EB7D                     jmp 005038C7
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:005037E5(C)
|
:0050384A 6A00                     push 00000000
* Possible StringData Ref from Code Obj ->"警告框"
|
:0050384C B9883F5000               mov ecx, 00503F88
* Possible StringData Ref from Code Obj ->"软件未注册!请与作者联系!"
|
:00503851 BA903F5000               mov edx, 00503F90
:00503856 A1C0905000               mov eax, dword ptr [005090C0]
:0050385B 8B00                     mov eax, dword ptr [eax]
:0050385D E8D2D9F4FF               call 00451234
:00503862 8BC6                     mov eax, esi
:00503864 E8B729F6FF               call 00466220
:00503869 8BC6                     mov eax, esi
:0050386B E8F0F7EFFF               call 00403060
:00503870 A1C0905000               mov eax, dword ptr [005090C0]
:00503875 8B00                     mov eax, dword ptr [eax]
:00503877 E814D9F4FF               call 00451190
:0050387C EB49                     jmp 005038C7
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0050376D(C)
|
:0050387E 8BC3                     mov eax, ebx
:00503880 E887190000               call 0050520C
:00503885 3C01                     cmp al, 01
:00503887 750C                     jne 00503895
:00503889 A1C0905000               mov eax, dword ptr [005090C0]
:0050388E 8B00                     mov eax, dword ptr [eax]
:00503890 E8FBD8F4FF               call 00451190
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00503887(C)
|
:00503895 6A00                     push 00000000
* Possible StringData Ref from Code Obj ->"警告框"
|
:00503897 B9883F5000               mov ecx, 00503F88
* Possible StringData Ref from Code Obj ->"请与作者联系，使用注册软件!"
|
:0050389C BAAC3F5000               mov edx, 00503FAC
:005038A1 A1C0905000               mov eax, dword ptr [005090C0]
:005038A6 8B00                     mov eax, dword ptr [eax]
:005038A8 E887D9F4FF               call 00451234
:005038AD 8BC6                     mov eax, esi
:005038AF E86C29F6FF               call 00466220
```



```
:005038B4 8BC6                    mov eax, esi
:005038B6 E8A5F7EFFF              call 00403060
:005038BB A1C0905000              mov eax, dword ptr [005090C0]
:005038C0 8B00                    mov eax, dword ptr [eax]
:005038C2 E8C9D8F4FF              call 00451190
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:00503708(C), :00503848(U), :0050387C(U)
:005038C7 8B45F8                  mov eax, dword ptr [ebp-08]
:005038CA B91D57CA00              mov ecx, 00CA571D
:005038CF 99                      cdq
:005038D0 F7F9                    idiv ecx
:005038D2 3BFA                    cmp edi, edx
:005038D4 0F8528030000            jne 00503C02
:005038DA 8BC3                    mov eax, ebx
```

在`005037DD`处“d eax”，看到的是你的注册码，记下来，`bc` *清断点。重新运行，输入正确的注册码，注册成功。

不知为什么，对程序中的某个地址下`Bpx`拦不住（`TRW`能但没办法看到浮点寄存器）。

```c
#include <stdio.h>

#include <math.h>

main()
{
    unsigned long aa;
    double bb;

    printf("--Stk 1.20--by FarProcCall\n\nPlease input your Code\t:");

    scanf("%lu",&aa);

    bb=aa;

    bb*=2;

    bb*=3.14152696;

    bb/=360;

    bb=sin(bb);

    bb+=1.01;

    aa*=131;

    bb*=aa;

    aa=bb;

    bb-=aa;

    if (bb>0.499999)
        aa++;

    aa%=100000000;

    printf("\nYou RegistrationCode is\t:%lu \n\n",aa);

    getch();
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
