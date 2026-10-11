
:004E4047 8A45FF mov al, byte ptr [ebp-01] <<--al <= [ebp-01]

:004E404A 33D2 xor edx, edx

:004E404C 8AD3 mov dl, bl <<--dl==bl==2

:004E404E 8BF2 mov esi, edx <<--esi==edx==2

:004E4050 33D2 xor edx, edx

:004E4052 F7F6 div esi <<--eax==eax/esi

:004E4054 85D2 test edx, edx <<--测试余数 edx 是否为零

:004E4056 7503 jne 004E405B <<--不为 0 跳

:004E4058 FE45FD inc [ebp-03] <<--否则[ebp-03]+1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004E4056(C)

|

:004E405B 807DFD02 cmp byte ptr [ebp-03], 02 <<--比较

:004E405F 7606 jbe 004E4067 <<--小于等于跳

:004E4061 C645FE00 mov [ebp-02], 00 <<--[ebp-02]==0

:004E4065 EB05 jmp 004E406C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004E405F(C)

|

:004E4067 43 inc ebx <<--除数 ebx 加 1

:004E4068 FEC9 dec cl

:004E406A 75D9 jne 004E4045 <<--继续

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004E4009(C), :004E4040(C), :004E4065(U)

|

:004E406C 8A45FE mov al, byte ptr [ebp-02] <<--al <= [ebp-02]

:004E406F 5E pop esi

:004E4070 5B pop ebx

:004E4071 59 pop ecx

:004E4072 5D pop ebp

:004E4073 C3 ret

==================end call 004E4028=======================

`call 004028`的作用是取得字符的 16 进制数，作为被除数，除数从 2 开始递增与被除数进行除法运算，直至除数与被除数相等就结束运算。运算中间若该字符被除数整除，则置`al`为 0，否则置`al`为 1。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004E40F7(C)

|

:004E411E 83FB01 cmp ebx, 00000001 <<--是 1 吗？

:004E4121 740A je 004E412D <<--是跳，否则不跳

:004E4123 8B45F4 mov eax, dword ptr [ebp-0C] <<--eax=="NNEWELL/IPB001C2C64"

:004E4126 0FB64418FE movzx eax, byte ptr [eax+ebx-02] <<--取字符（从第一个开始）

:004E412B EB06 jmp 004E4133 <<--跳

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004E4121(C)

|

:004E412D 8B45F4 mov eax, dword ptr [ebp-0C] <<--eax=="NNEWELL/IPB001C2C64"

:004E4130 0FB600 movzx eax, byte ptr [eax] <<--取每个字符

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004E412B(U)

|

:004E4133 C1E003 shl eax, 03 <<--eax 左移 3 位

:004E4136 0145EC add dword ptr [ebp-14], eax <<--累加

:004E4139 43 inc ebx

:004E413A 4E dec esi

:004E413B 75AC jne 004E40E9 <<--累加结果为 0x2760

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004E40E2(C)

|

:004E413D 8145EC0A1A0000 add dword ptr [ebp-14], 00001A0A <<--[ebp-14]==0x2760+0x1A0A=0X416A (16746)

:004E4144 8D55D8 lea edx, dword ptr [ebp-28]

:004E4147 8B45EC mov eax, dword ptr [ebp-14] <<--eax==0x416A

:004E414A E82D50F2FF call 0040917C <<--转换为 10 进制

:004E414F 8B4DD8 mov ecx, dword ptr [ebp-28] <<--ecx=="16746"

:004E4152 8D45F4 lea eax, dword ptr [ebp-0C]

:004E4155 8B55F0 mov edx, dword ptr [ebp-10] <<--edx=="/ICC"

:004E4158 E8DF0AF2FF call 00404C3C <<--连接字符串

:004E415D 8B45F8 mov eax, dword ptr [ebp-08]

:004E4160 8B55F4 mov edx, dword ptr [ebp-0C] <<--edx=="/ICC16746"

:004E4163 E82408F2FF call 0040498C

:004E4168 33C0 xor eax, eax

:004E416A 5A pop edx

:004E416B 59 pop ecx

:004E416C 59 pop ecx

:004E416D 648910 mov dword ptr fs:[eax], edx

:004E4170 689F414E00 push 004E419F

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004E419D(U)

|

:004E4175 8D45D8 lea eax, dword ptr [ebp-28]

:004E4178 BA05000000 mov edx, 00000005

:004E417D E8DA07F2FF call 0040495C

:004E4182 8D45F0 lea eax, dword ptr [ebp-10]

:004E4185 BA02000000 mov edx, 00000002

:004E418A E8CD07F2FF call 0040495C

:004E418F 8D45FC lea eax, dword ptr [ebp-04]

:004E4192 E8A107F2FF call 00404938

:004E4197 C3 ret

==================end call 004E4074==================

:004E77F0 E897D1F1FF call 0040498C

:004E77F5 8D55EC lea edx, dword ptr [ebp-14]

:004E77F8 8B83F4020000 mov eax, dword ptr [ebx+000002F4]

:004E77FE E88513F6FF call 00448B88

:004E7803 8B55EC mov edx, dword ptr [ebp-14]

:004E7806 8D863C270000 lea eax, dword ptr [esi+0000273C]

:004E780C E87BD1F1FF call 0040498C

:004E7811 6A00 push 00000000

:004E7813 668B0D7C784E00 mov cx, word ptr [004E787C]

:004E781A B202 mov dl, 02

* Possible StringData Ref from Code Obj ->"Registration was successful, thank "

->"you for registering!"

|

:004E781C B888784E00 mov eax, 004E7888

:004E7821 E8E2A0F5FF call 00441908

:004E7826 EB15 jmp 004E783D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004E77D7(C)

|

:004E7828 6A00 push 00000000

:004E782A 668B0D7C784E00 mov cx, word ptr [004E787C]

:004E7831 B201 mov dl, 01

* Possible StringData Ref from Code Obj ->"Registration was unsuccessful, "

->"please check name and license "

->"code and try again."

|

:004E7833 B8C8784E00 mov eax, 004E78C8

:004E7838 E8CBA0F5FF call 00441908

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004E779B(C), :004E7826(U)

|

:004E783D 8BC6 mov eax, esi

:004E783F E8E0000000 call 004E7924

:004E7844 8BC3 mov eax, ebx

:004E7846 E83DDFF7FF call 00465788

:004E784B 33C0 xor eax, eax

:004E784D 5A pop edx

:004E784E 59 pop ecx

:004E784F 59 pop ecx

:004E7850 648910 mov dword ptr fs:[eax], edx

:004E7853 6875784E00 push 004E7875

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004E7873(U)

|

:004E7858 8D45EC lea eax, dword ptr [ebp-14]

:004E785B BA04000000 mov edx, 00000004

:004E7860 E8F7D0F1FF call 0040495C

:004E7865 8D45FC lea eax, dword ptr [ebp-04]

:004E7868 E8CBD0F1FF call 00404938

:004E786D C3 ret

-------------------------------------------------------------

`Nnewell[iPB][FCG]`

2002.07.17 `4dd466d0e1e0cc123afbe504da2462ca`
