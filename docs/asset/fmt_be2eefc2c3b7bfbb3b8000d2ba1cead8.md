
#### 注册码验证逻辑分析

:004D50BC 0F855C020000 jne 004D531E
> 注册码是否 16 位？若不等则跳转至 `004D531E`（结束/失败）。

:004D50C2 8B45F8 mov eax, dword ptr [ebp-08]
> `EAX` = `fly01`（用户名）

:004D50C5 E8CEFBF2FF call 00404C98
> 获取用户名长度，`EAX` = 5

:004D50CA 83F805 cmp eax, 00000005
:004D50CD 0F8C4B020000 jl 004D531E
> 用户名不能少于 5 位，若小于则跳转至 `004D531E`（结束/失败）。

:004D50D3 8B45FC mov eax, dword ptr [ebp-04]
> `EAX` = `fly@263.net`（E-Mail）

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :004D5067(C)
|

:004D50D6 E8BDFBF2FF call 00404C98
> 获取 E-Mail 长度，`EAX` = 5

:004D50DB 83F805 cmp eax, 00000005
:004D50DE 0F8C3A020000 jl 004D531E
> E-Mail 不能少于 5 位，若小于则跳转至 `004D531E`（结束/失败）。

:004D50E4 33FF xor edi, edi
:004D50E6 33C0 xor eax, eax
:004D50E8 8945E8 mov dword ptr [ebp-18], eax

:004D50EB 8B45F8 mov eax, dword ptr [ebp-08]
> `EAX` = `fly01`（用户名）

:004D50EE E8A5FBF2FF call 00404C98
> 获取用户名长度，`EAX` = 5

:004D50F3 8B55F8 mov edx, dword ptr [ebp-08]
:004D50F6 8A4402FF mov al, byte ptr [edx+eax-01]
> 获取用户名末位字符的 HEX 值

:004D50FA E805B5FDFF call 004B0604
> **关键 CALL！** 获取字符在表中的位置。

:004D50FF 33DB xor ebx, ebx
:004D5101 8AD8 mov bl, al
> `BL` = 01

:004D5103 8B45F8 mov eax, dword ptr [ebp-08]
> `EAX` = `fly01`（用户名）

:004D5106 8A00 mov al, byte ptr [eax]
> 获取用户名首位字符的 HEX 值

:004D5108 E8F7B4FDFF call 004B0604
> 获取字符在表中的位置。

:004D510D 25FF000000 and eax, 000000FF
> `EAX` = 29

:004D5112 0FAFD8 imul ebx, eax
> `EBX` = 01 * 29 = 29

:004D5115 895DF0 mov dword ptr [ebp-10], ebx
> `[ebp-10]` = `EBX` = 29

:004D5118 8B45F8 mov eax, dword ptr [ebp-08]
:004D511B E878FBF2FF call 00404C98
:004D5120 8BF0 mov esi, eax
:004D5122 85F6 test esi, esi
:004D5124 7E21 jle 004D5147
:004D5126 BB01000000 mov ebx, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :004D5145(C)
|

:004D512B 8B45F8 mov eax, dword ptr [ebp-08]
> `EAX` = `fly01`（用户名）

:004D512E 8A4418FF mov al, byte ptr [eax+ebx-01]
> 依次正序获取用户名字符的 HEX 值

:004D5132 E8CDB4FDFF call 004B0604
> **关键 CALL！** 获取字符在表中的位置。

:004D5137 25FF000000 and eax, 000000FF
1. `EAX` = 29
2. `EAX` = 2F
3. `EAX` = 3C
4. `EAX` = 00
5. `EAX` = 01

:004D513C F76DF0 imul [ebp-10]
1. `EAX` = 29 * 29 = 00000691
2. `EAX` = 2F * 29 = 00000787
3. `EAX` = 3C * 29 = 0000099C
4. `EAX` = 00 * 29 = 00000000
5. `EAX` = 01 * 29 = 00000029

:004D513F 03F8 add edi, eax
1. `EDI` = 00000000 + 00000691 = 00000691
2. `EDI` = 00000692 + 00000787 = 00000E19
3. `EDI` = 00000E1B + 0000099C = 000017B7
4. `EDI` = 000017BA + 00000000 = 000017BA
5. `EDI` = 000017BE + 00000029 = 000017E7

:004D5141 03FB add edi, ebx
1. `EDI` = 00000691 + 1 = 00000692
2. `EDI` = 00000E19 + 2 = 00000E1B
3. `EDI` = 000017B7 + 3 = 000017BA
4. `EDI` = 000017BA + 4 = 000017BE
5. `EDI` = 000017E7 + 5 = 000017EC

:004D5143 43 inc ebx
> `EBX` 增 1

:004D5144 4E dec esi
:004D5145 75E4 jne 004D512B
> 循环执行用户名位数次

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :004D5124(C)
|

:004D5147 8D45DC lea eax, dword ptr [ebp-24]

* Possible StringData Ref from Code Obj -> "rusreg"
|

:004D514A B96C534D00 mov ecx, 004D536C
> `ECX` = `rusreg`

:004D514F 8B55F8 mov edx, dword ptr [ebp-08]
> `EDX` = `fly01`（用户名）

:004D5152 E88DFBF2FF call 00404CE4
> 将用户名和 `rusreg` 连接起来

:004D5157 8B45DC mov eax, dword ptr [ebp-24]
> `EAX` = `fly01rusreg`

:004D515A E839FBF2FF call 00404C98
> 获取 `fly01rusreg` 长度

:004D515F 8BF0 mov esi, eax
> `ESI` = `EAX` = B (11)

:004D5161 85F6 test esi, esi
:004D5163 7E25 jle 004D518A
:004D5165 BB01000000 mov ebx, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :004D5188(C)
> 运算流程同上，此处仅记录结果。

:004D516A 8B45DC mov eax, dword ptr [ebp-24]
> `EAX` = `fly01rusreg`

:004D516D 8A4418FF mov al, byte ptr [eax+ebx-01]
> 依次正序获取 `fly01rusreg` 字符的 HEX 值

:004D5171 E88EB4FDFF call 004B0604
> **关键 CALL！** 获取字符在表中的位置。

:004D5176 25FF000000 and eax, 000000FF
:004D517B F76DF0 imul [ebp-10]
:004D517E 0345E8 add eax, dword ptr [ebp-18]
:004D5181 03C3 add eax, ebx
:004D5183 8945E8 mov dword ptr [ebp-18], eax
> 最后结果 = `[ebp-18]` = 000047D9

:004D5186 43 inc ebx
:004D5187 4E dec esi
:004D5188 75E0 jne 004D516A
> 循环 11 次

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :004D5163(C)
|

:004D518A 8B450C mov eax, dword ptr [ebp+0C]
> `EAX` = `Internet Download Accelerator`

```
☆☆ ☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆ ☆☆☆
这个参数是固定的：
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆ ☆☆☆☆☆☆☆☆☆☆☆☆☆
```

```
0174F9C8  49 6E 74 65 72 6E 65 74 20 44 6F 77 6E 6C 6F 61   Internet Downloa
0174F9D8  64 20 41 63 63 65 6C 65 72 61 74 6F 72             d Accelerator
```

:004D518D E806FBF2FF call 00404C98
> 获取 `Internet Download Accelerator` 长度
