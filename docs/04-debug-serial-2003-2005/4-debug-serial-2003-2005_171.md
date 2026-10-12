
# IE 浏览器修复工具 V1.1

> 原文：[`www.pediy.com/kssd/pediy06/pediy6152.htm`](https://www.pediy.com/kssd/pediy06/pediy6152.htm)

IE 浏览器修复工具 V1.1 分析笔记

机器码：`B6D6DE88A6`

假码：`87654321`

注册码：`93FDAF96`

下载地址：[`www.downloadsky.com/soft/12744.html`](http://www.downloadsky.com/soft/12744.html)

重启验证型，侦壳，`ASPack 2.12` -> `Alexey Solodovnikov` 的壳，`Delphi 6.0` 编写，`PE-id` 脱壳。经查，注册信息写在“`IE 浏览器修复工具`”目录下的 `Dhsys.ini` 文件中。

```
* Possible StringData Ref from Data Obj -> "Dhsys.ini"

:004860B7 B988614800             mov ecx, 00486188
:004860BC E833E3F7FF             call 004043F4
:004860C1 8B4DF4                 mov ecx, dword ptr [ebp-0C]
:004860C4 B201                   mov dl, 01

* Possible StringData Ref from Data Obj -> ",}C"

:004860C6 A1E0724300             mov eax, dword ptr [004372E0]
:004860CB E8C012FBFF             call 00437390
:004860D0 8945F0                 mov dword ptr [ebp-10], eax
:004860D3 33C0                   xor eax, eax
:004860D5 55                     push ebp
:004860D6 6842614800             push 00486142
:004860DB 64FF30                 push dword ptr fs:[eax]
:004860DE 648920                 mov dword ptr fs:[eax], esp
:004860E1 6A00                   push 00000000
:004860E3 8D45E4                 lea eax, dword ptr [ebp-1C]
:004860E6 50                     push eax

* Possible StringData Ref from Data Obj -> "id"     //注册成功后会生成

:004860E7 B99C614800             mov ecx, 0048619C

* Possible StringData Ref from Data Obj -> "reg"

:004860EC BAA8614800             mov edx, 004861A8
:004860F1 8B45F0                 mov eax, dword ptr [ebp-10]
:004860F4 8B18                   mov ebx, dword ptr [eax]
:004860F6 FF13                   call dword ptr [ebx]
:004860F8 8B45E4                 mov eax, dword ptr [ebp-1C]
:004860FB 8D4DF8                 lea ecx, dword ptr [ebp-08]

* Possible StringData Ref from Data Obj -> "IETOOLS"

:004860FE BAB4614800             mov edx, 004861B4
:00486103 E8D8C2FFFF             call 004823E0
:00486108 8D55E0                 lea edx, dword ptr [ebp-20]

* Possible StringData Ref from Data Obj -> "IETOOLS"

:0048610B B8B4614800             mov eax, 004861B4
:00486110 E88FB4FFFF             call 004815A4     //算法 call，跟进
:00486115 8B45E0                 mov eax, dword ptr [ebp-20]
:00486118 8B55F8                 mov edx, dword ptr [ebp-08]
:0048611B E8D4E3F7FF             call 004044F4     //关键比较，eax 真码
:00486120 7506                   jne 00486128      //不等则跳，跳则 over
:00486122 C645FF01               mov [ebp-01], 01
:00486126 EB04                   jmp 0048612C
```

★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★

跟进 `486110 E88FB4FFFF` `call 004815A4` 此 `call`：

```
:004815DF 8B45E8                 mov eax, dword ptr [ebp-18]   //eax=机器码：B6D6DE88A6
:004815E2 8D55EC                 lea edx, dword ptr [ebp-14]
:004815E5 E8BA0E0000             call 004824A4      //跟进看看
:004815EA 8B45EC                 mov eax, dword ptr [ebp-14]   //eax=F03FEE63D728BC7FF33F
:004815ED B90A000000             mov ecx, 0000000A  //ecx=A
:004815F2 33D2                   xor edx, edx
:004815F4 E80F30F8FF             call 00404608      //取上面 eax 值的前 10 位
:004815F9 8B45F0                 mov eax, dword ptr [ebp-10]   //eax=F03FEE63D7
:004815FC 8D55F4                 lea edx, dword ptr [ebp-0C]
:004815FF E8246CF8FF             call 00408228
:00481604 8B45F4                 mov eax, dword ptr [ebp-0C]   //eax=F03FEE63D7
:00481607 8D55F8                 lea edx, dword ptr [ebp-08]
:0048160A E8C969F8FF             call 00407FD8
:0048160F 8D45E0                 lea eax, dword ptr [ebp-20]
:00481612 50                     push eax
:00481613 8D4DDC                 lea ecx, dword ptr [ebp-24]
:00481616 8B55FC                 mov edx, dword ptr [ebp-04]
:00481619 8B45F8                 mov eax, dword ptr [ebp-08]   //eax=F03FEE63D7
:0048161C E8D70B0000             call 004821F8      //跟进看看
:00481621 8B45DC                 mov eax, dword ptr [ebp-24]   //eax=93FDAF961ECEA4865ADACBA5EE73066E
:00481624 B908000000             mov ecx, 00000008  //ecx=8
:00481629 33D2                   xor edx, edx
:0048162B E8D82FF8FF             call 00404608      //取上面 eax 值的前 8 位即为真码
:00481630 8B45E0                 mov eax, dword ptr [ebp-20]   //eax=注册码：93FDAF96
:00481633 8D55E4                 lea edx, dword ptr [ebp-1C]
:00481636 E8ED6BF8FF             call 00408228
:0048163B 8B45E4                 mov eax, dword ptr [ebp-1C]
:0048163E 8BD3                   mov edx, ebx
:00481640 E89369F8FF             call 00407FD8
:00481645 33C0                   xor eax, eax
```

★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★

跟进 `4815E5 E8BA0E0000` `call 004824A4` 这个 `call`：

```
:004824D6 8B45FC                 mov eax, dword ptr [ebp-04]   //eax=机器码：B6D6DE88A6
:004824D9 E8CA1EF8FF             call 004043A8    //取得机器码的位数
:004824DE 8BF0                   mov esi, eax     //esi=eax=A
:004824E0 85F6                   test esi, esi
:004824E2 7E47                   jle 0048252B
:004824E4 C745F801000000         mov [ebp-08], 00000001   //[ebp-08]置 1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00482529(C)

:004824EB 8B45FC                 mov eax, dword ptr [ebp-04]   //eax=机器码：B6D6DE88A6
:004824EE 8B55F8                 mov edx, dword ptr [ebp-08]   //edx=1
:004824F1 8A4410FF               mov al, byte ptr [eax+edx-01]   //依次取机器码字符的 hex 值送 al
:004824F5 328338814800           xor al, byte ptr [ebx+00488138]   //al=al xor 内存地址 488138+ebx 中的值
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

内存中的值：

```
018F:00488138 B2 09 AA 55 93 6D 84 47-39 31 29 21 19 11 09 01 ?猆搈凣 91)!....
018F:00488148 3B 33 2B 23 1B 13 0B 03-3D 35 2D 25 1D 15 0D 05 ;3+#....=5-%....
018F:00488158 3F 37 2F 27 1F 17 0F 07-38 30 28 20 18 10 08 00 ?7/'....80( ....
018F:00488168 3A 32 2A 22 1A 12 0A 02-3C 34 2C 24 1C 14 0C 04 :2*"....<4,$....
018F:00488178 3E 36 2E 26 1E 16 0E 06-27 07 2F 0F 37 17 3F 1F >6.&....'./.7.?.
018F:00488188 26 06 2E 0E 36 16 3E 1E-25 05 2D 0D 35 15 3D 1D &...6.>.%.-.5.=.
018F:00488198 24 04 2C 0C 34 14 3C 1C-23 03 2B 0B 33 13 3B 1B $.,.4.<.#.+.3.;.
018F:004881A8 22 02 2A 0A 32 12 3A 1A-21 01 29 09 31 11 39 19 ".*.2.:.!.).1.9.
018F:004881B8 20 00 28 08 30 10 38 18-1F 00 00 00 00 00 00 00 .(.0.8.........
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

```
:004824FB 25FF000000             and eax, 00000FF
:00482500 8D4DF4                 lea ecx, dword ptr [ebp-0C]
:00482503 BA02000000             mov edx, 00000002
:00482508 E82F5FF8FF             call 0040843C
:0048250D 8B55F4                 mov edx, dword ptr [ebp-0C]     //把上面异或后的值依次放到 edx 中，循环结束 edx 的值为：F03FEE63D728BC7FF33F
:00482510 8BC7                   mov eax, edi
:00482512 E8991EF8FF             call 004043B0
:00482517 43                     inc ebx      //ebx 加 1
:00482518 81E307000080           and ebx, 80000007
:0048251E 7905                   jns 00482525
:00482520 4B                     dec ebx
:00482521 83CBF8                 or ebx, FFFFFFF8
:00482524 43                     inc ebx
```



* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0048251E(C)

|

:00482525 FF45F8               `inc [ebp-08]`

:00482528 4E                   `dec esi` //机器码的位数减 1

:00482529 75C0                 `jne 004824EB` //机器码没取完返回继续循环。

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

跟进 `48161C E8D70B0000`  `call 004821F8` 这个 `call`：

:00482235 8B55F8               `mov edx, dword ptr [ebp-08]` //`edx`=`IETOOLS`

:00482238 8B45FC               `mov eax, dword ptr [ebp-04]` //`eax`=`F03FEE63D7`

:0048223B E840FCFFFF          `call 00481E80` //跟进这里看看

:00482240 8D45F4               `lea eax, dword ptr [ebp-0C]`

:00482243 E8A01EF8FF          `call 004040E8`

:00482248 8B45F0               `mov eax, dword ptr [ebp-10]` //这里`[ebp-10]`内存地址里存放着以后的注册码，

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

```
018F:01115BF8 93 FD AF 96 1E CE A4 86-5A DA CB A5 EE 73 06 6E 擙瘱.韦哯谒ヮs.n
018F:01115C08 00 5C 11 01 0C 5C 11 01-0C 5C 11 01 34 00 00 00 .\\...\\...\\..4...
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

:0048224B E85821F8FF          `call 004043A8`

:00482250 8BD8                 `mov ebx, eax`

:00482252 4B                   `dec ebx`

:00482253 85DB                 `test ebx, ebx`

:00482255 7C4E                 `jl 004822A5`

:00482257 43                   `inc ebx`

:00482258 33F6                 `xor esi, esi`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004822A3(C)

|

:0048225A 8D45EC               `lea eax, dword ptr [ebp-14]`

:0048225D 50                   `push eax`

:0048225E 8B45F0               `mov eax, dword ptr [ebp-10]`

:00482261 0FB60430             `movzx eax, byte ptr [eax+esi]` //`eax`=`93`、`FD`、`AF`、`96`、`1E`、`CE`、`A4`、`86`、`5A`、`DA`、`CB`、`A5`、`EE`、`73`、`06`、`6E`

:00482265 8945E4               `mov dword ptr [ebp-1C], eax`

:00482268 C645E800             `mov [ebp-18], 00`

:0048226C 8D55E4               `lea edx, dword ptr [ebp-1C]`

:0048226F 33C9                 `xor ecx, ecx`

* Possible StringData Ref from Data Obj ->`%x`

|

:00482271 B8E0224800           `mov eax, 004822E0`

:00482276 E80D6EF8FF          `call 00409088`

:0048227B 8B45EC               `mov eax, dword ptr [ebp-14]` //`eax`=`93`、`FD`、`AF`、`96`、`1E`、`CE`、`A4`、`86`、`5A`、`DA`、`CB`、`A5`、`EE`、`73`、`06`、`6E`

:0048227E E82521F8FF          `call 004043A8`

:00482283 48                   `dec eax`

:00482284 7510                 `jne 00482296`

:00482286 8D45EC               `lea eax, dword ptr [ebp-14]`

:00482289 8B4DEC               `mov ecx, dword ptr [ebp-14]`

* Possible StringData Ref from Data Obj ->`00`

|

:0048228C BAEC224800           `mov edx, 004822EC`

:00482291 E85E21F8FF          `call 004043F4`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00482284(C)

|

:00482296 8D45F4               `lea eax, dword ptr [ebp-0C]`

:00482299 8B55EC               `mov edx, dword ptr [ebp-14]` //`edx`=`93`、`FD`、`AF`、`96`、`1E`、`CE`、`A4`、`86`、`5A`、`DA`、`CB`、`A5`、`EE`、`73`、`06`、`6E`

:0048229C E80F21F8FF          `call 004043B0`

:004822A1 46                   `inc esi` //计算器 `esi` 加 1

:004822A2 4B                   `dec ebx` //减 1

:004822A3 75B5                 `jne 0048225A`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00482255(C)

|

:004822A5 8BC7                 `mov eax, edi`

:004822A7 8B55F4               `mov edx, dword ptr [ebp-0C]` //把上面计算得到的数连接起来，`edx`=`93FDAF961ECEA4865ADACBA5EE73066E`

:004822AA E88D1EF8FF          `call 0040413C`

:004822AF 33C0                 `xor eax, eax`

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

跟进 `48223B E840FCFFFF`  `call 00481E80` 这个 `call`： 这个 `call` 最终没有看明白，请高手指点！

:00481E80 55                   `push ebp`

:00481E81 8BEC                 `mov ebp, esp`

:00481E83 83C4CC               `add esp, FFFFFFCC`

:00481E86 53                   `push ebx`

:00481E87 56                   `push esi`

:00481E88 33DB                 `xor ebx, ebx`

:00481E8A 895DCC               `mov dword ptr [ebp-34], ebx`

:00481E8D 895DD8               `mov dword ptr [ebp-28], ebx`

:00481E90 894DF4               `mov dword ptr [ebp-0C], ecx`

:00481E93 8955F8               `mov dword ptr [ebp-08], edx`

:00481E96 8945FC               `mov dword ptr [ebp-04], eax`

:00481E99 8B45FC               `mov eax, dword ptr [ebp-04]`

:00481E9C E8F726F8FF          `call 00404598`

:00481EA1 8B45F8               `mov eax, dword ptr [ebp-08]`

:00481EA4 E8EF26F8FF          `call 00404598`

:00481EA9 33C0                 `xor eax, eax`

:00481EAB 55                   `push ebp`

:00481EAC 681D204800           `push 0048201D`

:00481EB1 64FF30               `push dword ptr fs:[eax]`

:00481EB4 648920               `mov dword ptr fs:[eax], esp`

:00481EB7 8B45FC               `mov eax, dword ptr [ebp-04]`

:00481EBA E8E924F8FF          `call 004043A8`

:00481EBF 85C0                 `test eax, eax`

:00481EC1 7E28                 `jle 00481EEB`

:00481EC3 8B45FC               `mov eax, dword ptr [ebp-04]` //`eax`=`F03FEE63D7`

:00481EC6 E8DD24F8FF          `call 004043A8`

:00481ECB 8B55FC               `mov edx, dword ptr [ebp-04]` //`edx`=`F03FEE63D7`

:00481ECE 807C02FF00           `cmp byte ptr [edx+eax-01], 00`

:00481ED3 7516                 `jne 00481EEB`

* Possible StringData Ref from Data Obj ->`Error: the last char is NULL char.`

|

:00481ED5 B934204800           `mov ecx, 00482034`

:00481EDA B201                 `mov dl, 01`

* Possible StringData Ref from Data Obj ->``

|

:00481EDC A13C734000           `mov eax, dword ptr [0040733C]`

:00481EE1 E8BA9BF8FF          `call 0040BAA0`

:00481EE6 E8391CF8FF          `call 00403B24`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00481EC1(C), :00481ED3(C)

|

:00481EEB 8B45F8               `mov eax, dword ptr [ebp-08]` //`eax`=`IETOOLS`

:00481EEE E8B524F8FF          `call 004043A8`

:00481EF3 83F808               `cmp eax, 00000008`

:00481EF6 7D2B                 `jge 00481F23`

:00481EF8 EB0D                 `jmp 00481F07`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00481F12(C)

|

:00481EFA 8D45F8               `lea eax, dword ptr [ebp-08]`

:00481EFD BA60204800           `mov edx, 00482060`

:00481F02 E8A924F8FF          `call 004043B0`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00481EF8(U)

|

:00481F07 8B45F8               `mov eax, dword ptr [ebp-08]` //`eax`=`IETOOLS`

:00481F0A E89924F8FF          `call 004043A8`

:00481F0F 83F808               `cmp eax, 00000008`

:00481F12 7CE6                 `jl 00481EFA`

:00481F14 EB0D                 `jmp 00481F23`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00481F39(C)

|

:00481F16 8D45FC               `lea eax, dword ptr [ebp-04]`

:00481F19 BA60204800           `mov edx, 00482060`

:00481F1E E88D24F8FF          `call 004043B0`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00481EF6(C), :00481F14(U)

|

:00481F23 8B45FC               `mov eax, dword ptr [ebp-04]` //`eax`=`F03FEE63D7`

:00481F26 E87D24F8FF          `call 004043A8` //取得位数



```asm
:00481F2B 2507000080           and eax, 80000007

:00481F30 7905               jns 00481F37

:00481F32 48                 dec eax

:00481F33 83C8F8             or eax, FFFFFFF8

:00481F36 40                 inc eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00481F30(C)

|

:00481F37 85C0               test eax, eax

:00481F39 75DB               jne 00481F16

:00481F3B 33DB               xor ebx, ebx

:00481F3D 8D45DC             lea eax, dword ptr [ebp-24]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00481F4D(C)

|

:00481F40 8B55F8             mov edx, dword ptr [ebp-08]    //edx=IETOOLS

:00481F43 8A141A             mov dl, byte ptr [edx+ebx]    //依次取 IETOOLS 各字符的 hex 送 dl

:00481F46 8810               mov byte ptr [eax], dl

:00481F48 43                 inc ebx

:00481F49 40                 inc eax

:00481F4A 83FB08             cmp ebx, 00000008    //如果超过 8 位，则只取前 8 位

:00481F4D 75F1               jne 00481F40         //没取完继续循环

:00481F4F 6A0F               push 0000000F

:00481F51 B96C9C4800         mov ecx, 00489C6C

:00481F56 8D45DC             lea eax, dword ptr [ebp-24]

:00481F59 BA07000000         mov edx, 00000007

:00481F5E E8EDFAFFFF         call 00481A50

:00481F63 8D45D8             lea eax, dword ptr [ebp-28]

:00481F66 E87D21F8FF         call 004040E8

:00481F6B 8B45FC             mov eax, dword ptr [ebp-04]      //eax=F03FEE63D7

:00481F6E E83524F8FF         call 004043A8

:00481F73 85C0               test eax, eax

:00481F75 7903               jns 00481F7A

:00481F77 83C007             add eax, 00000007

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00481F75(C)

|

:00481F7A C1F803             sar eax, 03

:00481F7D 48                 dec eax

:00481F7E 85C0               test eax, eax

:00481F80 7C65               jl 00481FE7

:00481F82 40                 inc eax

:00481F83 8945D0             mov dword ptr [ebp-30], eax

:00481F86 C745D400000000     mov [ebp-2C], 00000000

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00481FE5(C)

|

:00481F8D 33DB               xor ebx, ebx

:00481F8F 8D45EC             lea eax, dword ptr [ebp-14]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00481FA7(C)

|

:00481F92 8B55D4             mov edx, dword ptr [ebp-2C]

:00481F95 C1E203             shl edx, 03

:00481F98 03D3               add edx, ebx

:00481F9A 8B4DFC             mov ecx, dword ptr [ebp-04]      //ecx=F03FEE63D7

:00481F9D 8A1411             mov dl, byte ptr [ecx+edx]    //依次取 F03FEE63D7 各字符的 hex 送 dl

:00481FA0 8810               mov byte ptr [eax], dl

:00481FA2 43                 inc ebx

:00481FA3 40                 inc eax

:00481FA4 83FB08             cmp ebx, 00000008    //如果超过 8 位，则只取前 8 位

:00481FA7 75E9               jne 00481F92         //没取完继续循环

:00481FA9 8D45E4             lea eax, dword ptr [ebp-1C]

:00481FAC 50                 push eax

:00481FAD 6A07               push 00000007

:00481FAF 8D55EC             lea edx, dword ptr [ebp-14]

:00481FB2 B907000000         mov ecx, 00000007

:00481FB7 33C0               xor eax, eax

:00481FB9 E8EAFCFFFF         call 00481CA8

:00481FBE BB08000000         mov ebx, 00000008

:00481FC3 8D75E4             lea esi, dword ptr [ebp-1C]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00481FDD(C)

|

:00481FC6 8D45CC             lea eax, dword ptr [ebp-34]

:00481FC9 8A16               mov dl, byte ptr [esi]      //依次取[esi]中的值送 dl，93、FD、AF、96、1E、CE、A4、86、5A、DA、CB、A5、EE、73、06、6E

:00481FCB E80023F8FF         call 004042D0

:00481FD0 8B55CC             mov edx, dword ptr [ebp-34]

:00481FD3 8D45D8             lea eax, dword ptr [ebp-28]

:00481FD6 E8D523F8FF         call 004043B0

:00481FDB 46                 inc esi

:00481FDC 4B                 dec ebx

:00481FDD 75E7               jne 00481FC6

:00481FDF FF45D4             inc [ebp-2C]

:00481FE2 FF4DD0             dec [ebp-30]

:00481FE5 75A6               jne 00481F8D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00481F80(C)

|

:00481FE7 8B45F4             mov eax, dword ptr [ebp-0C]

:00481FEA 8B55D8             mov edx, dword ptr [ebp-28]

:00481FED E84A21F8FF         call 0040413C

:00481FF2 33C0               xor eax, eax

:00481FF4 5A                 pop edx

:00481FF5 59                 pop ecx

:00481FF6 59                 pop ecx

:00481FF7 648910             mov dword ptr fs:[eax], edx

:00481FFA 6824204800         push 00482024

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00482022(U)

|

:00481FFF 8D45CC             lea eax, dword ptr [ebp-34]

:00482002 E8E120F8FF         call 004040E8

:00482007 8D45D8             lea eax, dword ptr [ebp-28]

:0048200A E8D920F8FF         call 004040E8

:0048200F 8D45F8             lea eax, dword ptr [ebp-08]

:00482012 BA02000000         mov edx, 00000002

:00482017 E8F020F8FF         call 0040410C

:0048201C C3                 ret
```

李逍遥[`cschina`]

2003.07.06

`4dd466d0e1e0cc123afbe504da2462ca`
