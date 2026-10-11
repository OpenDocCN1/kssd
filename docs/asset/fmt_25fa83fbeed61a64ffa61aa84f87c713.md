
## 汇编代码分析

:004FDA17 50 `push eax`

:004FDA18 B90F000000 `mov ecx, 0000000F`

:004FDA1D BA01000000 `mov edx, 00000001`

:004FDA22 8B45E8 `mov eax, dword ptr [ebp-18]`

:004FDA25 E8EA70F0FF `call 00404B14`（保留前 15 位，注册名最好 5 位以上，否则可能有麻烦）

:004FDA2A 8D45E4 `lea eax, dword ptr [ebp-1C]`

:004FDA2D E8D26BF0FF `call 00404604`

:004FDA32 BE0F000000 `mov esi, 0000000F`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`|:004FDA78(C)`

`|`

:004FDA37 8D8534FDFFFF `lea eax, dword ptr [ebp+FFFFFD34]`

:004FDA3D 50 `push eax`

:004FDA3E B901000000 `mov ecx, 00000001`

:004FDA43 8BD6 `mov edx, esi`

:004FDA45 8B45E8 `mov eax, dword ptr [ebp-18]`

:004FDA48 E8C770F0FF `call 00404B14`

:004FDA4D 8B8534FDFFFF `mov eax, dword ptr [ebp+FFFFFD34]`

:004FDA53 8A00 `mov al, byte ptr [eax]`

:004FDA55 E82250F0FF `call 00402A7C`

:004FDA5A 8BD0 `mov edx, eax`

:004FDA5C 8D8538FDFFFF `lea eax, dword ptr [ebp+FFFFFD38]`

:004FDA62 E87D6DF0FF `call 004047E4`

:004FDA67 8B9538FDFFFF `mov edx, dword ptr [ebp+FFFFFD38]`

:004FDA6D 8D45E4 `lea eax, dword ptr [ebp-1C]`

:004FDA70 E84F6EF0FF `call 004048C4`

:004FDA75 4E `dec esi`

:004FDA76 85F6 `test esi, esi`

:004FDA78 75BD `jne 004FDA37`

以上这段程序将经复制保留的注册码第 3 位一组变换排列顺序，

如 123 456 789 012 345（下面称原始排列用“A”表示）

排列成 321 654 987 210 543（下面称变化排列用“B”表示）

:004FDA7A 8D45EC `lea eax, dword ptr [ebp-14]`

:004FDA7D E8826BF0FF `call 00404604`

:004FDA82 8D45F0 `lea eax, dword ptr [ebp-10]`

:004FDA85 E87A6BF0FF `call 00404604`

:004FDA8A 8D45E0 `lea eax, dword ptr [ebp-20]`

:004FDA8D 8B55F8 `mov edx, dword ptr [ebp-08]`

:004FDA90 E8076CF0FF `call 0040469C`

:004FDA95 BE01000000 `mov esi, 00000001`

:004FDA9A 8D7B1A `lea edi, dword ptr [ebx+1A]`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`|:004FDB2B(C)`

`|`

:004FDA9D 8B45E4 `mov eax, dword ptr [ebp-1C]`（“B”入 eax）

:004FDAA0 8A4430FF `mov al, byte ptr [eax+esi-01]`（取“B”第一位）

:004FDAA4 E8D34FF0FF `call 00402A7C`

:004FDAA9 25FF000000 `and eax, 000000FF`

:004FDAAE 50 `push eax`

:004FDAAF 8B45E8 `mov eax, dword ptr [ebp-18]`（“A”入 eax）

:004FDAB2 8A4430FF `mov al, byte ptr [eax+esi-01]`（取“A”第一位）

:004FDAB6 E8C14FF0FF `call 00402A7C`

:004FDABB 25FF000000 `and eax, 000000FF`

:004FDAC0 6BC00D `imul eax, 0000000D`（“A”第一位乘以“D”入 eax）

:004FDAC3 8D0443 `lea eax, dword ptr [ebx+2*eax]`（eax*2 后查表取值再入 eax）

:004FDAC6 5A `pop edx`

:004FDAC7 8A941025F9FFFF `mov dl, byte ptr [eax+edx-000006DB]`（eax 加“B”第一位再减 6DB 的值入 dl）

:004FDACE 8D8530FDFFFF `lea eax, dword ptr [ebp+FFFFFD30]`

:004FDAD4 E80B6DF0FF `call 004047E4`

:004FDAD9 8B9530FDFFFF `mov edx, dword ptr [ebp+FFFFFD30]`

:004FDADF 8D45EC `lea eax, dword ptr [ebp-14]`

:004FDAE2 E8DD6DF0FF `call 004048C4`（把运算得值存入一表）

:004FDAE7 8D8528FDFFFF `lea eax, dword ptr [ebp+FFFFFD28]`

:004FDAED 50 `push eax`

:004FDAEE B901000000 `mov ecx, 00000001`

:004FDAF3 8BD6 `mov edx, esi`

:004FDAF5 8B45E0 `mov eax, dword ptr [ebp-20]`

:004FDAF8 E81770F0FF `call 00404B14`（取机器码第一位）

:004FDAFD 8B8528FDFFFF `mov eax, dword ptr [ebp+FFFFFD28]`

:004FDB03 E850B6F0FF `call 00409158`

:004FDB08 8A1407 `mov dl, byte ptr [edi+eax]`（按机器码数查表求值入 dl）

:004FDB0B 8D852CFDFFFF `lea eax, dword ptr [ebp+FFFFFD2C]`

:004FDB11 E8CE6CF0FF `call 004047E4`

:004FDB16 8B952CFDFFFF `mov edx, dword ptr [ebp+FFFFFD2C]`

:004FDB1C 8D45F0 `lea eax, dword ptr [ebp-10]`

:004FDB1F E8A06DF0FF `call 004048C4`（将所求值存入二表）

:004FDB24 46 `inc esi`

:004FDB25 83C71A `add edi, 0000001A`

:004FDB28 83FE10 `cmp esi, 00000010`

:004FDB2B 0F856CFFFFFF `jne 004FDA9D`

以上程序循环算出两组 15 位的大写字母表（下面要别称其为“C”和“D”，对应关系为：由“A”、“B”运算得出 C，由机器码运算得出“D”），下面再进行运算：

:004FDB31 8B45F4 `mov eax, dword ptr [ebp-0C]`

:004FDB34 E8CB6AF0FF `call 00404604`

:004FDB39 BE01000000 `mov esi, 00000001`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`|:004FDB95(C)`

`|`

:004FDB3E 8D8524FDFFFF `lea eax, dword ptr [ebp+FFFFFD24]`

:004FDB44 8B55F0 `mov edx, dword ptr [ebp-10]`（“D”入 edx）

:004FDB47 0FB65432FF `movzx edx, byte ptr [edx+esi-01]`（取“D”的第一位入 edx）

:004FDB4C 8B4DEC `mov ecx, dword ptr [ebp-14]`（“C”入 ecx）

:004FDB4F 0FB64C31FF `movzx ecx, byte ptr [ecx+esi-01]`（取“C”的第一位入 ecx）

:004FDB54 6BC90D `imul ecx, 0000000D`（运算：ecx*d）

:004FDB57 8D0C4B `lea ecx, dword ptr [ebx+2*ecx]`（运算：ecx 再乘以 2 后再查表求值，入 ecx）

:004FDB5A 8A941125F9FFFF `mov dl, byte ptr [ecx+edx-000006DB]`（ecx 的值加“D”的第一位再减 6DB 的值入 dl，此值为注册码第一位）

:004FDB61 E87E6CF0FF `call 004047E4`

:004FDB66 8B9524FDFFFF `mov edx, dword ptr [ebp+FFFFFD24]`

:004FDB6C 8B45F4 `mov eax, dword ptr [ebp-0C]`

:004FDB6F E8506DF0FF `call 004048C4`

:004FDB74 8B45F4 `mov eax, dword ptr [ebp-0C]`

:004FDB77 83FE05 `cmp esi, 00000005`（比较计算出的注册码是否够 5 位）

:004FDB7A 7405 `je 004FDB81`（够则跳走加“-”）

:004FDB7C 83FE0A `cmp esi, 0000000A`（比较计算出的注册码是否够 10 位）

:004FDB7F 7510 `jne 004FDB91`（相等时则加“-”）

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`|:004FDB7A(C)`

`|`

以下几行代码负责置注册码的第 6 位、11 位为“-”，即注册码形式为：12345-67890-12345

:004FDB81 8B45F4 `mov eax, dword ptr [ebp-0C]`

:004FDB84 BAE8DB4F00 `mov edx, 004FDBE8`

:004FDB89 E8366DF0FF `call 004048C4`

:004FDB8E 8B45F4 `mov eax, dword ptr [ebp-0C]`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`|:004FDB7F(C)`

`|`

:004FDB91 46 `inc esi`

:004FDB92 83FE10 `cmp esi, 00000010`（比较计算出的注册码是否够 16 位）

:004FDB95 75A7 `jne 004FDB3E`（不够则跳走循环运算直至够 15 位）

:004FDB97 33C0 `xor eax, eax`

:004FDB99 5A `pop edx`

:004FDB9A 59 `pop ecx`

:004FDB9B 59 `pop ecx`

:004FDB9C 648910 `mov dword ptr fs:[eax], edx`

:004FDB9F 68D6DB4F00 `push 004FDBD6`

## 整理

分析过程中：

* “A”=`YQM JCH YQM JCH YQM`（无空格）
* “B”=`MQY HCJ MQY HCJ MQY`（无空格）
* “C”=`WIXEGUWIXEGUWIX`
* “D”=`NACEEOIEOXDAEXQ`

机器码：`349910439465063`

注册名：`YQMJCH`

注册码：`UNXIT-ITQXA-IAGIA`

注册信息存放位置：

`HKEY_LOCAL_MACHINE\software\白领软件工作室\日语天天背单词\v1.4`

另外，不知有没有暗桩，不妥之处请高手指点。

`4dd466d0e1e0cc123afbe504da2462ca`
