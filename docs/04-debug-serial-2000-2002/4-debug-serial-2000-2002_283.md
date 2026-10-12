
# Don't Panic! 40

> 原文：[`www.pediy.com/kssd/pediy03/forum1272.htm`](https://www.pediy.com/kssd/pediy03/forum1272.htm)

软件名： `Don't Panic! v4.0`

下载地址： `http://www.panicware.com/downloads/DontPanic40.exe`

大小： 704KB

用途： `Don't Panic！`提供您使用电脑时的隐私，

时常有人在您身后晃来晃去，“不小心”就看到您的电脑屏幕吗？

`Don'tPanic！`让您以最简单的方法保障您的隐私。

让您按一下鼠标或键盘就能隐藏或关闭程序，再按一下就能把程序在便出来。

从此以后，不管是写信、上网、使用电脑软件都不用怕有别人靠近了！

工具： `trw2000 v1.23`

小弟初来乍到，请诸位老大多多关照，下面是破文。

运行 `Don't Panic!`，输入任意注册码，先别按确定，运行 `trw2000`，设断 `bpx hmemcpy`

或 `bpx getdlgitemtexta`，回到 `Don't Panic!`程序，按确定，被 `trw` 拦下，下 `pmodule` 来到程序领空，

按 `F10` 若干下，来到这里：

```
:004123C8 E8F3FeFFFF   call 004122C0------------------------- (1) 这两个 call 是计算核心，
:004123CD 8BF0         mov esi, eax                            一定要跟进去。
:004123CF 56           push esi
:004123D0 E8DBFDFFFF   call 004121B0------------------------- (2)
:004123D5 83C408       add esp, 00000008
:004123D8 85C0         test eax, eax\----------------- EAX 不为 0，则跳
:004123DA 7558         jne 00412434                            如果跳，就注册成功
\* Referenced by a CALL at Address:
|:004123C8
|
:004122C0 8B442404     mov eax, dword ptr [esp+04]             先进第(1)个 call
:004122C4 50           push eax
:004122C5 E844360000   call 0041590E-------------------------- 再跟进去,到 0041590e
:004122CA 83C404       add esp, 00000004
:004122CD C3           ret\---------------------------------- 回到 004123cd
\* Referenced by a CALL at Addresses:
|:004122C5 , :0041599D , :004186F9 , :00418727 , :00418752
|
:0041590E 53           push ebx
:0041590F 55           push ebp
:00415910 56           push esi
:00415911 57           push edi
:00415912 8B7C2414     mov edi, dword ptr [esp+14]----------- 这里下 d edi 可看见你的假注册码
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00415942(U)
|
:00415916 833DAC5C420001  cmp dword ptr [00425CAC], 00000001
:0041591D 7E0F           jle 0041592E-------------------------- 这里一定跳
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041591D(C)
|
:0041592E 0FB607         movzx eax, byte ptr [edi]
:00415931 8B0DA05A4200   mov ecx, dword ptr [00425AA0]
:00415937 8A0441         mov al, byte ptr [ecx+2*eax]
:0041593A 83E008         and eax, 00000008
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041592C(U)
|
:0041593D 85C0           test eax, eax
:0041593F 7403           je 00415944--------------------------- 跳
:00415941 47             inc edi
:00415942 EBD2           jmp 00415916
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041593F(C)
|
:00415944 0FB637         movzx esi, byte ptr [edi]
:00415947 47             inc edi
:00415948 83FE2D         cmp esi, 0000002D
:0041594B 8BEE           mov ebp, esi
:0041594D 7405           je 00415954
:0041594F 83FE2B         cmp esi, 0000002B
:00415952 7504           jne 00415958------------------------- 再跳
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00415952(C)
|
:00415958 33DB           xor ebx, ebx
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00415989(U)
|
:0041595A 833DAC5C420001  cmp dword ptr [00425CAC], 00000001
:00415961 7E0C           jle 0041596F------------------------- 再跳
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00415961(C)
|
:0041596F A1A05A4200     mov eax, dword ptr [00425AA0]
:00415974 8A0470         mov al, byte ptr [eax+2*esi]
:00415977 83E004         and eax, 00000004
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041596D(U)
|
:0041597A 85C0           test eax, eax
:0041597C 740D           je 0041598B
:0041597E 8D049B         lea eax, dword ptr [ebx+4*ebx]       这 5 行把你输入的假注册码
:00415981 8D5C46D0       lea ebx, dword ptr [esi+2*eax-30]    当成一个整数，以十六进制
:00415985 0FB637         movzx esi, byte ptr [edi]            的形式存在 ebx 中。
:00415988 47             inc edi
:00415989 EBCF           jmp 0041595A
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041597C(C)
|
:0041598B 83FD2D         cmp ebp, 0000002D
:0041598E 8BC3           mov eax, ebx\-------------------------- ebx 的值移到 eax 中
:00415990 7502           jne 00415994
:00415992 F7D8           neg eax
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00415990(C)
|
:00415994 5F             pop edi
:00415995 5E             pop esi
:00415996 5D             pop ebp
:00415997 5B             pop ebx
:00415998 C3             ret\----------------------------------- 这里又回到 004122ca,
然后又 ret 到 004123cd,
接着来到第(2)个 call
\* Referenced by a CALL at Addresses:
|:00405329 , :0040CC8A , :0040CCF9 , :0040DEFE , :004123D0    现在跟进第(2)个 call
|
:004121B0 8B442404       mov eax, dword ptr [esp+04]
:004121B4 68D135E2E1     push E1E235D1                            这两个数字干什么用的？
:004121B9 681953C633     push 33C65319
:004121BE 50             push eax
:004121BF E8DCFFFFFF     call 004121A0                              跟进去看，来到 004121a0
:004121C4 33D2           xor edx, edx
:004121C6 B9BB0B0000     mov ecx, 00000BBB
:004121CB F7F1           div ecx                                    把 EAX 中的值除与 BBB
:004121CD 83C40C         add esp, 0000000C                          再把余数给 EAX
:004121D0 8BC2           mov eax, edx
:004121D2 F7D8           neg eax
:004121D4 1BC0           sbb eax, eax
:004121D6 40             inc eax                                    如果余数为 0，这里将使
:004121D7 C3             ret                                          EAX 为 1，回到 004123d5
\* Referenced by a CALL at Address:
|:004121BF
|
:004121A0 8B442404       mov eax, dword ptr [esp+04]              原来是把 eax 中的值
:004121A4 8B4C240C       mov ecx, dword ptr [esp+0C]              与 E1E235D1 异或后再与
:004121A8 33C1           xor eax, ecx                               33C65319 相乘
:004121AA 0FAF442408     imul eax, dword ptr [esp+08]
:004121AF C3             ret\----------------------------------- 回到 004121c4
```

总结一下：程序把你输入的注册码当作一个 10 位的整数，并转换成 16 进制数，然后与 `E1E235D1` 异或后再乘 `33C65319`,得到的乘积除去 `BBB`，如果能整除，则注册成功。

好，现在可以写 `KeyGen` 了，用 C 语言写的 `KeyGen` 如下，多简单啊：

```c
#include <stdio.h>

#include <math.h>

void main()
{
    int x=0,y=0;
    for(int i=0; i<2147483647; i++)
    {
        x=i;
        x^=0xe1e235d1;
        x*=0x33c65319;
        x%=0xbbb;
        if (x==0)
            printf("i=%d\\n",i);
    }
}
```

运行 `KeyGen` ,随便抄一个注册码下来，`0011210112`，注册，成功！！！

`Don't Panic! v4.0` crack by `lancelot` `2001.6.26` `4dd466d0e1e0cc123afbe504da2462ca`
