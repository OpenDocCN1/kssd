
# FlashSoft1.05

> 原文：`www.pediy.com/kssd/pediy03/forum866.htm`

## FlashSoft1.05 破解方法

先用 `W32Dasm` 反汇编 `DfBox.exe`，打开串式参考，找到了 `Msg_RegFalse` 和 `Msg_RegOk` 这个重要的信息，双击。

```text
* Possible StringData Ref from Code Obj -> "Messages"

|

:004D2088 BA7C214D00               mov edx, 004D217C
:004D208D A1449A4D00               mov eax, dword ptr [004D9A44]
:004D2092 E8E187FFFF               call 004CA878
:004D2097 8B45F0                   mov eax, dword ptr [ebp-10]
:004D209A 33C9                     xor ecx, ecx
:004D209C 33D2                     xor edx, edx
:004D209E E85D64F8FF               call 00458500
:004D20A3 8B55F8                   mov edx, dword ptr [ebp-08]
:004D20A6 8B45FC                   mov eax, dword ptr [ebp-04]
:004D20A9 E8E679FFFF               call 004C9A94              -->看来秘密在这里面
:004D20AE 84C0                     test al, al
:004D20B0 743C                     je 004D20EE               -->重要的信息
:004D20B2 8B55FC                   mov edx, dword ptr [ebp-04]
:004D20B5 A1409A4D00               mov eax, dword ptr [004D9A40]
:004D20BA E8F184FFFF               call 004CA5B0
:004D20BF 8B55F8                   mov edx, dword ptr [ebp-08]
:004D20C2 A1409A4D00               mov eax, dword ptr [004D9A40]
:004D20C7 E8A484FFFF               call 004CA570
:004D20CC 8D45EC                   lea eax, dword ptr [ebp-14]
:004D20CF 50                       push eax

* Possible StringData Ref from Code Obj -> "Msg_RegOk"       -->就是刚才找到的

|

:004D20D0 B9AC214D00               mov ecx, 004D21AC

* Possible StringData Ref from Code Obj -> "Messages"

|

:004D20D5 BA7C214D00               mov edx, 004D217C
:004D20DA A1449A4D00               mov eax, dword ptr [004D9A44]
:004D20DF E89487FFFF               call 004CA878
:004D20E4 8B45EC                   mov eax, dword ptr [ebp-14]
:004D20E7 E8F860F8FF               call 004581E4
:004D20EC EB42                     jmp 004D2130

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :004D20B0(C)

|

:004D20EE 8D45E8                   lea eax, dword ptr [ebp-18]
:004D20F1 50                       push eax

* Possible StringData Ref from Code Obj -> "Msg_RegFalse"      -->就是刚才找到的

|

:004D20F2 B9C0214D00               mov ecx, 004D21C0

\---------------------------------------------------------------
\---------------------------------------------------------------

* Referenced by a CALL at Addresses:

| :004D20A9  , :004D3AA3

|

:004C9A94 55                       push ebp
:004C9A95 8BEC                     mov ebp, esp
:004C9A97 83C4F4                   add esp, FFFFFFF4
:004C9A9A 53                       push ebx
:004C9A9B 33C9                     xor ecx, ecx
:004C9A9D 894DF4                   mov dword ptr [ebp-0C], ecx
:004C9AA0 8955F8                   mov dword ptr [ebp-08], edx
:004C9AA3 8945FC                   mov dword ptr [ebp-04], eax
:004C9AA6 8B45FC                   mov eax, dword ptr [ebp-04]
:004C9AA9 E876A7F3FF               call 00404224
:004C9AAE 8B45F8                   mov eax, dword ptr [ebp-08]
:004C9AB1 E86EA7F3FF               call 00404224
:004C9AB6 33C0                     xor eax, eax
:004C9AB8 55                       push ebp
:004C9AB9 68FD9A4C00               push 004C9AFD
:004C9ABE 64FF30                   push dword ptr fs:[eax]
:004C9AC1 648920                   mov dword ptr fs:[eax], esp
:004C9AC4 8D55F4                   lea edx, dword ptr [ebp-0C]
:004C9AC7 8B45FC                   mov eax, dword ptr [ebp-04]
:004C9ACA E83D000000               call 004C9B0C
:004C9ACF 8B45F4                   mov eax, dword ptr [ebp-0C]
:004C9AD2 8B55F8                   mov edx, dword ptr [ebp-08]
:004C9AD5 E8A6A6F3FF               call 00404180
:004C9ADA 7504                     jne 004C9AE0
:004C9ADC B301                     mov bl, 01
:004C9ADE EB02                     jmp 004C9AE2

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :004C9ADA(C)

|

:004C9AE0 33DB                     xor ebx, ebx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :004C9ADE(U)

|

:004C9AE2 33C0                     xor eax, eax
:004C9AE4 5A                       pop edx
:004C9AE5 59                       pop ecx
:004C9AE6 59                       pop ecx
:004C9AE7 648910                   mov dword ptr fs:[eax], edx
:004C9AEA 68049B4C00               push 004C9B04

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :004C9B02(U)

|

:004C9AEF 8D45F4                   lea eax, dword ptr [ebp-0C]
:004C9AF2 BA03000000               mov edx, 00000003
:004C9AF7 E818A3F3FF               call 00403E14          -->这是个重要的 Call，按 F8 进入
:004C9AFC C3                       ret

\---------------------------------------------------------------
\---------------------------------------------------------------

* Referenced by a CALL at Addresses:

| :0040495F  , :00405672  , :004059FB  , :0040BA2B  , :0040C0C4
| :0040C655  , :0040C98C  , :0040CDF6  , :0040D2A1  , :0040D487
| :0040DD81  , :0040DF51  , :00413004  , :00413613  , :00413792
| :004138D9  , :00413A9D  , :00415E43  , :00416A9E  , :00416DAC
| :0041732A  , :0041881C  , :00418C5E  , :0041BD77  , :0041E60F
| :0041F17A  , :0041FC08  , :00429BAE  , :0042CEC0  , :0042DB84
| :0042DF6F  , :0043519B  , :0043AB59  , :0043CAE0  , :0043F805
| :004417A7  , :004418F9  , :004447D4  , :00444C67  , :00444C74
| :004458A3  , :0044685F  , :004493E0  , :00452051  , :004529B8
| :00457256  , :00457397  , :004576BB  , :00457790  , :004580B5
| :004584E9  , :00460516  , :0046060E  , :004606CA  , :004607AC
| :0046084A  , :0046092C  , :004612E9  , :0046489B  , :004689A0
| :00468C57  , :0046C54B  , :004727D8  , :00472AAB  , :00472C84
| :00472FAD  , :0047326B  , :004735D5  , :00473B67  , :0047A391
| :0047A8E3  , :0047A8F0  , :0047AD17  , :0047B779  , :0047BE5F
| :0047BF28  , :0047C007  , :0047C5F1  , :0047C97B  , :0047CF4B
| :0047D396  , :0047D86E  , :0047DB97  , :0047DCE8  , :0047E0D6
| :0047E0E3  , :0047EBF9  , :0047ED98  , :0047FC90  , :004809F1
| :00480DA9  , :004821BA  , :00482E0A  , :00483939  , :00483B91
| :00484442  , :00485731  , :00486F5E  , :00489B30  , :0048A790
| :0048F409  , :0048FFCF  , :004900E7  , :00490291  , :0049029E
| :00490F0C  , :004912B7  , :00491CCC  , :0049330D  , :00493649
| :004939F7  , :00494E35  , :00495B72  , :004961F7  , :00496371
| :00496D09  , :0049769B  , :004995B0  , :004995C5  , :004998D7
| :004998EC  , :00499D04  , :0049A5AF  , :0049AB70  , :0049AD01
| :0049B176  , :0049BA9E  , :0049E7B1  , :004A00D9  , :004A1019
| :004A10C1  , :004A121C  , :004A1468  , :004A15B8  , :004A19A6
| :004A1CD7  , :004A1E70  , :004A1FC3  , :004A26DB  , :004A2BCF
| :004A2BDC  , :004A2C82  , :004A2E2B  , :004A2F8E  , :004A46A9
| :004A6C68  , :004A7F61  , :004AD539  , :004AE410  , :004AF0C3
| :004AF6F8  , :004AFF8A  , :004B1F87  , :004B20CF  , :004B21E0
| :004B2316  , :004B276E  , :004B277E  , :004B2799  , :004B27A6
| :004B27B3  , :004B2CE2  , :004B2CF2  , :004B2D0D  , :004B2D1A
| :004B2D27  , :004B2D34  , :004C6B0E  , :004C708F  , :004C7836
| :004C793B  , :004C7A98  , :004C7C42  , :004C7D31  , :004C845D
| :004C864B  , :004C8833  , :004C8B58  , :004C8B75  , :004C8E2A
| :004C912A  , :004C94D9  , :004C94E6  , :004C9503  , :004C9879
| :004C9AF7  , :004C9C9F  , :004C9D8B  , :004CA7B0  , :004CA8FC
```
