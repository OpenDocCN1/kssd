
# BigInt Calculator 1.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue366.htm`](https://www.pediy.com/kssd/pediy04/kanxue366.htm)

URL `http://www.h108.com/stkman/soft/calcpro.zip` 专业版 1.0

作者：`stkman[CCG]` & `arbiter[CCG]`

简述：`BigInt Calculator` 是一个支持超大整数运算的计算器，可以对长度最多达 10000 位 10 进制的数据进行运算。

它不但支持普通的四则混合运算，而且具有位操作和表达式运算功能，计算器本身提供了 21 个存贮单元可供保存运算的中间结果，在表达式中可以引用这些存储单元。

`BigInt Calculator` 的一个显著优点是软件体积小，可执行文件不足 50K，纯绿色软件，不修改注册表。

`upx` 加的壳，脱壳很简单。

注册文件为 `calcpro.key`，如内容不对会退出。

\`~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

:00402C5E A0294F4100 \t\t\t\t\t\t mov al, byte ptr [00414F29]<==============比较重要！
:00402C63 84C0 \t\t\t\t\t\t\t\t test al, al
:00402C65 7516 \t\t\t\t\t\t\t\t jne 00402C7D
:00402C67 6810100000 \t\t\t\t\t\t push 00001010

\* Possible StringData Ref from Data Obj ->"Warning"

:00402C6C 682C294100 \t\t\t\t\t\t push 0041292C

\* Possible StringData Ref from Data Obj ->"THANK YOU FOR TRYING BIGINT CALCULATOR! "
->"This is a unregistered version,If "
->"you like it,please register it,Thanks. "
->" "

:00402C71 68C8294100 \t\t\t\t\t\t push 004129C8
:00402C76 52 \t\t\t\t\t\t\t\t push edx

\* Reference To: `USER32.MessageBoxA`, Ord:0000h

:00402C77 FF15F0104100 \t\t\t\t\t Call dword ptr [004110F0]

\`~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

:004030DC A0294F4100 \t\t\t\t\t\t mov al, byte ptr [00414F29]<==============比较重要！
:004030E1 84C0 \t\t\t\t\t\t\t\t test al, al
:004030E3 742D \t\t\t\t\t\t\t\t je 00403112
:004030E5 57 \t\t\t\t\t\t\t\t push edi
:004030E6 BF70434100 \t\t\t\t\t\t mov edi, 00414370
:004030EB 83C9FF \t\t\t\t\t\t\t\t or ecx, FFFFFFFF
:004030EE 33C0 \t\t\t\t\t\t\t\t xor eax, eax
:004030F0 F2 \t\t\t\t\t\t\t\t repnz
:004030F1 AE \t\t\t\t\t\t\t\t scasb
:004030F2 F7D1 \t\t\t\t\t\t\t\t not ecx
:004030F4 49 \t\t\t\t\t\t\t\t dec ecx
:004030F5 5F \t\t\t\t\t\t\t\t pop edi
:004030F6 741A \t\t\t\t\t\t\t\t je 00403112
:004030F8 8B4C2404 \t\t\t\t\t\t mov ecx, dword ptr [esp+04]
:004030FC 6870434100 \t\t\t\t\t\t push 00414370
:00403101 6837040000 \t\t\t\t\t\t push 00000437
:00403106 51 \t\t\t\t\t\t\t\t push ecx

\* Reference To: `USER32.SetDlgItemTextA`, Ord:0000h

:00403107 FF15EC104100 \t\t\t\t\t Call dword ptr [004110EC]
:0040310D 33C0 \t\t\t\t\t\t\t\t xor eax, eax
:0040310F C21000 \t\t\t\t\t\t\t ret 0010

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004030E3(C), :004030F6(C)

:00403112 8B542404 \t\t\t\t\t\t mov edx, dword ptr [esp+04]

\* Possible StringData Ref from Data Obj ->"UNREGISTERED"

:00403116 68802A4100 \t\t\t\t\t\t push 00412A80
:0040311B 6837040000 \t\t\t\t\t\t push 00000437
:00403120 52 \t\t\t\t\t\t\t\t push edx

\* Reference To: `USER32.SetDlgItemTextA`, Ord:0000h

:00403121 FF15EC104100 \t\t\t\t\t Call dword ptr [004110EC]

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004030BC(C), :004030C8(C)

:00403127 33C0 \t\t\t\t\t\t\t\t xor eax, eax
:00403129 C21000 \t\t\t\t\t\t\t ret 0010

\`~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

:004035FB E860550000 \t\t\t\t\t\t call 00408B60
:00403600 83C430 \t\t\t\t\t\t\t\t add esp, 00000030
:00403603 85C0 \t\t\t\t\t\t\t\t test eax, eax
:00403605 751B \t\t\t\t\t\t\t\t jne 00403622<======================仅改这里 741B 即可！
:00403607 A118514100 \t\t\t\t\t\t mov eax, dword ptr [00415118]
:0040360C 6870434100 \t\t\t\t\t\t push 00414370
:00403611 50 \t\t\t\t\t\t\t\t push eax
:00403612 E8F9770000 \t\t\t\t\t\t call 0040AE10
:00403617 83C408 \t\t\t\t\t\t\t\t add esp, 00000008
:0040361A 881D294F4100 \t\t\t\t\t mov byte ptr [00414F29], bl<==============比较重要！
:00403620 EB2C \t\t\t\t\t\t\t\t jmp 0040364E

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00403605(C)

:00403622 8D7C241C \t\t\t\t\t\t lea edi, dword ptr [esp+1C]
:00403626 83C9FF \t\t\t\t\t\t\t\t or ecx, FFFFFFFF
:00403629 33C0 \t\t\t\t\t\t\t\t xor eax, eax
:0040362B F2 \t\t\t\t\t\t\t\t repnz
:0040362C AE \t\t\t\t\t\t\t\t scasb
:0040362D F7D1 \t\t\t\t\t\t\t\t not ecx
:0040362F 2BF9 \t\t\t\t\t\t\t\t sub edi, ecx
:00403631 8BF7 \t\t\t\t\t\t\t\t mov esi, edi
:00403633 8BD1 \t\t\t\t\t\t\t\t mov edx, ecx

\* Possible StringData Ref from Data Obj ->"BigInt Calculator Pro 1.0 "

:00403635 BFA8284100 \t\t\t\t\t\t mov edi, 004128A8
:0040363A 83C9FF \t\t\t\t\t\t\t\t or ecx, FFFFFFFF
:0040363D F2 \t\t\t\t\t\t\t\t repnz
:0040363E AE \t\t\t\t\t\t\t\t scasb
:0040363F 8BCA \t\t\t\t\t\t\t\t mov ecx, edx
:00403641 4F \t\t\t\t\t\t\t\t dec edi
:00403642 C1E902 \t\t\t\t\t\t\t\t shr ecx, 02
:00403645 F3 \t\t\t\t\t\t\t\t repz
:00403646 A5 \t\t\t\t\t\t\t\t movsd
:00403647 8BCA \t\t\t\t\t\t\t\t mov ecx, edx
:00403649 83E103 \t\t\t\t\t\t\t\t and ecx, 00000003
:0040364C F3 \t\t\t\t\t\t\t\t repz
:0040364D A4 \t\t\t\t\t\t\t\t movsb

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00403620(U)

\* Possible StringData Ref from Data Obj ->"BigInt Calculator Pro 1.0 "

:0040364E 68A8284100 \t\t\t\t\t\t push 004128A8
:00403653 55 \t\t\t\t\t\t\t\t push ebp

\* Reference To: `USER32.SetWindowTextA`, Ord:0000h

:00403654 FF15DC104100 \t\t\t\t\t Call dword ptr [004110DC]
:0040365A A1582E4200 \t\t\t\t\t\t mov eax, dword ptr [00422E58]
:0040365F 8B0D882E4200 \t\t\t\t\t mov ecx, dword ptr [00422E88]
:00403665 5D \t\t\t\t\t\t\t\t pop ebp
:00403666 898834020000 \t\t\t\t\t mov dword ptr [eax+00000234], ecx
:0040366C 8B15582E4200 \t\t\t\t\t mov edx, dword ptr [00422E58]
:00403672 8BC3 \t\t\t\t\t\t\t\t mov eax, ebx
:00403674 899A28020000 \t\t\t\t\t mov dword ptr [edx+00000228], ebx
:0040367A 5B \t\t\t\t\t\t\t\t pop ebx
:0040367B 5F \t\t\t\t\t\t\t\t pop edi
:0040367C 5E \t\t\t\t\t\t\t\t pop esi
:0040367D 83C42C \t\t\t\t\t\t\t\t add esp, 0000002C
:00403680 C21000 \t\t\t\t\t\t\t ret 0010

\`~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 4dd466d0e1e0cc123afbe504da2462ca
