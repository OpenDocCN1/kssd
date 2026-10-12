
# ssreader 360

> 原文：[`www.pediy.com/kssd/pediy04/kanxue328.htm`](https://www.pediy.com/kssd/pediy04/kanxue328.htm)

`ssreader 360` 正式版 完美破解版本

按如下方法修改, 可不用读书卡。

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040C451(C)

|

:0040C46E 6A24  push 00000024  -----> 改成 `jmp 40c498`

\* Possible StringData Ref from Data Obj ->"提示"

* Possible StringData Ref from Data Obj ->"专为读书卡会员服务，请进行读书卡注册。 是否现?

* ->"谧⒉幔?

:0040C470 68C9D55600  push 0056D5C9

:0040C475 6893D55600  push 0056D593

:0040C47A 8B45A8  mov eax, dword ptr [ebp-58]

:0040C47D E856FE1000  call 0051C2D8

:0040C482 50  push eax

:0040C483 E816F41500  call 0056B89E  `USER32!MessageBoxA`

:0040C488 83F806  cmp eax, 00000006

:0040C48B 750B  jne 0040C498

:0040C48D 8B55A8  mov edx, dword ptr [ebp-58]

:0040C490 8B45A8  mov eax, dword ptr [ebp-58]

:0040C493 E888E4FFFF  call 0040A920

0167:0041F912 0F84A4000000  `JZ NEAR 0041F9BC`  -----------------> 此行必须跳走才行，改成下面两行

0167:0041F912 E9A5000000  `JMP 0041F9BC`  <---- 上面一行要改成这样

0167:0041F917 90  `NOP`

0167:0041F918 8B0D1C015A00  `MOV ECX,[005A011C]`

0167:0041F91E 8B01  `MOV EAX,[ECX]`

0167:0041F920 E80FCBFEFF  `CALL 0040C434`

0167:0041F925 84C0  `TEST AL,AL`

0167:0041F927 0F858F000000  `JNZ NEAR 0041F9BC`

0167:0041F92D FF4DD8  `DEC DWORD [EBP-28]`

0167:0041F930 8D45F8  `LEA EAX,[EBP-08]`

0167:0041F912 E9A5000000  `JMP 0041F9BC`

0167:0041F917 90  `NOP`

0167:0040B770 E8BF0C0000  `CALL 0040C434`

0167:0040B775 84C0  `TEST AL,AL`

0167:0040B777 740E  `JZ 0040B787`  -------------------> 改成 `nop`

0167:0040B779 B001  `MOV AL,01`

0167:0040B77B 8B55D4  `MOV EDX,[EBP-2C]`

0167:0040B77E 64891500000000  `MOV [FS:00],EDX`

0167:0040B785 EB0C  `JMP SHORT 0040B793`

0167:0040B787 33C0  `XOR EAX,EAX`

0167:0040B789 8B55D4  `MOV EDX,[EBP-2C]`

0167:0040B78C 64891500000000  `MOV [FS:00],EDX`

## pdg2.dll 文字识别限制的解除

可进行整页文字识别。

:1000C067 89550C  `mov dword ptr [ebp+0C], edx`

:1000C06A 53  `push ebx`

:1000C06B E88F6C0200  `call 10032CFF`

:1000C070 59  `pop ecx`

:1000C071 59  `pop ecx`

:1000C072 8B4D0C  `mov ecx, dword ptr [ebp+0C]`

:1000C075 6A04  `push 00000004`  ----------------->改成 `push 1` (6a04 改成 6a01)

:1000C077 0FAFC8  `imul ecx, eax`

:1000C07A 8B45FC  `mov eax, dword ptr [ebp-04]`

:1000C07D 5B  `pop ebx`

:1000C07E 0FAF45F8  `imul eax, dword ptr [ebp-08]`

:1000C082 99  `cdq`

:1000C083 F7FB  `idiv ebx`

:1000C085 3BC8  `cmp ecx, eax`

:1000C087 7E6F  `jle 1000C0F8`

:1000C089 8B86CC1E0000  `mov eax, dword ptr [esi+00001ECC]`

就这样,把上面几处修改之后用用看,爽吧!

我希望大家只是自己悄悄使用就是了,别大面积扩散,因为我们应该感谢超星公司,它给我们提供了一个非常丰富的资料宝库!所以,

让那些有钱的人支持一下次超星公司吧!你认为？

有问题请与我联系: `wjt@sina.com` `4dd466d0e1e0cc123afbe504da2462ca`
