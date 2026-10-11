
# `freeResV0.94`

> 原文：[`www.pediy.com/kssd/pediy04/kanxue336.htm`](https://www.pediy.com/kssd/pediy04/kanxue336.htm)

`freeResV0.94` 疯狂暴力破解

===============================================================

多情俏狐

peiyou henan china

所用工具:

1. `w32dasm` 白金版或 9.00
2. `trw2000` 1.23
3. `dede30`
4. `procdump` 1.62
5. `hedit2`(我喜欢用,它最小最快)

先用 `Procdump` 脱 `Asppack` 壳的 `script.ini` 扩展 脱掉程序的壳。

再用 `w32dasm`(白金版)查找"300 KB"，找到两处如下:

第一处:

```
:004C96C6 3D00B00400             cmp eax, 0004B000
:004C96CB 7426                   jle 004C96F3
\=====>74 改 EB
:004C96CD E80E27FFFF             call 004BBDE0
:004C96D2 6A10                   push 00000010
:004C96D4 A1E8134D00             mov eax, dword ptr [004D13E8]
:004C96D9 8B00                   mov eax, dword ptr [eax]
:004C96DB E8F0A9F3FF             call 004040D0
:004C96E0 8BC8                   mov ecx, eax
\* Possible StringData Ref from Data Obj ->"该软件未注册，所以大于 300 KB "
                                  ->"的文件不能执行此项操作！"
                                  |
:004C96E2 BAB49B4C00             mov edx, 004C9BB4
:004C96E7 A100144D00             mov eax, dword ptr [004D1400]
:004C96EC 8B00                   mov eax, dword ptr [eax]
:004C96EE E83D70F8FF             call 00450730
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004C96B6(C), :004C96CB(U)
|
:004C96F3 E80825FFFF             call 004BBC00
```

第二处:

```
:004CD305 3D00B00400             cmp eax, 0004B000
:004CD30A 7E26                   jle 004CD332
\=====>74 改 EB 搞定就行了
:004CD30C E8CFEAFEFF             call 004BBDE0
:004CD311 6A10                   push 00000010
:004CD313 A1E8134D00             mov eax, dword ptr [004D13E8]
:004CD318 8B00                   mov eax, dword ptr [eax]
:004CD31A E8B16DF3FF             call 004040D0
:004CD31F 8BC8                   mov ecx, eax
\* Possible StringData Ref from Data Obj ->"该软件未注册，所以大于 300 KB "
                                  ->"的文件不能执行此项操作！"
                                  |
:004CD321 BAF4D44C00             mov edx, 004CD4F4
:004CD326 A100144D00             mov eax, dword ptr [004D1400]
:004CD32B 8B00                   mov eax, dword ptr [eax]
:004CD32D E8FE33F8FF             call 00450730
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004CD2F5(U), :004CD30A(C)
|
:004CD332 E8C9E8FEFF             call 004BBC00
```

最后用 `dede30` 找到"建立可编辑资源"的处理程序段 `build1Click`

进而查出处理程序的首址为 `004cd2d0`,然后用 `trw2000` 跟踪，

分别用小于 300K 和大于 300K 的压缩程序进行卑鄙大法跟踪

得到如下结果.

```
4cd339 都不跳
4cd34e  offset cc94e ===>一跳就死 nop 掉
4cd35e 都跳
4cd41b 不跳
0F8F78010000 nop 掉就行了
```

我不想找出注册码，但我要用，所以这样就行了.

注:

天山折梅手解释:

```
:004BBD09 E80E83F4FF             call 0040401C--------->暗桩！“假假”
注册码对比，如相等则作者发觉你在破解它然后退出！
:004BBD0E 750C                   jne 004BBD1C---------->一定要跳才行，
offset bb30e
不能让“假假”注册码相等！
:004BBD48 E8CF82F4FF             call 0040401C--------------->注册名与注册码变形后之值对比！
:004BBD4D 7504                   jne 004BBD53---------------->一定不能跳！即值要相等。
```

`oake` 的脱壳扩展:

在`[INDEX]`后添加

`pXX=Aspack` (`XX` 为十六进制)

```
[Aspack]
L1=OBJR
L2=LOOK EB,?
L3=JZ 5
L4=QUIT
L5=BP
L6=WALK
L7=OBJR
L8=LOOK 61,75
L9=BP
LA=STEP
OPTL1=00000000
OPTL2=01010001
OPTL3=01010001
OPTL4=00030000
OPTL5=00000000
4dd466d0e1e0cc123afbe504da2462ca
```
