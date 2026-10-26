
# 固定资产管理系统算法分析

- 破文作者：YuanQiao[BCG]
- 文章题目：固定资产管理系统
- 破解难度：+++初级+++ / 中级 / 高级 / 超难
- 破解平台：WinXP SP2

## 文章简介

资产管理系统建立在信息技术基础上，以系统化的管理思想，帮助各类组织，对其整个组织范围内的各类资产，提供入库、资产查询、分布查询、统计、领用出库、报废一系列资产全生命周期管理功能；并在资产信息集中的基础上，向资产管理人员提供强大的智能决策支持。从而为组织内决策层、职能层、执行层等提供集决策、管理、维护手段为一体的资产管理全面解决方案。

## 破解过程

CPU号：`39056758417437`

CPU编号为：`BFEBFBFF`转换为带符号的十进制数 `-1075053569`。

CPU号与CPU编号组合为：`39056758417437-1075053569`，取前14位数则为`39056758417437`。

```
3  9  0  5  6  7  5  8  4  1  7  4  3  7
33 39 30 35 36 37 35 38 34 31 37 34 33 37
1  2  3  4  5  6  7  8  9  a  b  c  d  e

(0x33 and 0xff)*0x11*0x1
(0x39 and 0xff)*0x11*0x2
(0x30 and 0xff)*0x11*0x3
(0x35 and 0xff)*0x11*0x4
(0x36 and 0xff)*0x11*0x5
(0x37 and 0xff)*0x11*0x6
(0x35 and 0xff)*0x11*0x7
(0x38 and 0xff)*0x11*0x8
(0x34 and 0xff)*0x11*0x9
(0x31 and 0xff)*0x11*0xa
(0x37 and 0xff)*0x11*0xb
(0x34 and 0xff)*0x11*0xc
(0x33 and 0xff)*0x11*0xd
(0x37 and 0xff)*0x11*0xe
```

反编译后的PB语言程序：

```
integer i
long ll_kl
string ls_tmp
for i = 1 to len(ls_kl)
        ls_tmp = mid(ls_kl,i,1)
        ll_kl = ll_kl + (asc(ls_tmp) * 17) * i
next
return ll_kl
```

上面计算之后累加为：`0x011A58` -> `94520`。

```
(0x33 and 0xff)*0xd*0x1
(0x39 and 0xff)*0xd*0x2
(0x30 and 0xff)*0xd*0x3
(0x35 and 0xff)*0xd*0x4
(0x36 and 0xff)*0xd*0x5
(0x37 and 0xff)*0xd*0x6
(0x35 and 0xff)*0xd*0x7
(0x38 and 0xff)*0xd*0x8
(0x34 and 0xff)*0xd*0x9
(0x31 and 0xff)*0xd*0xa
(0x37 and 0xff)*0xd*0xb
(0x34 and 0xff)*0xd*0xc
(0x33 and 0xff)*0xd*0xd
(0x37 and 0xff)*0xd*0xe
```

反编译后的PB语言程序：

```
integer i
long ll_kl
string ls_tmp
for i = 1 to len(ls_kl)
        ls_tmp = mid(ls_kl,i,1)
        ll_kl = ll_kl + (asc(ls_tmp) * 13) * i
next
return ll_kl
```

上面计算之后累加为：`0x017138` -> `72280`。

`94520-72280` 就是注册码。

```
0070A8C8  /.  55            PUSH EBP
0070A8C9  |.  8BEC          MOV EBP,ESP
0070A8CB  |.  6A 00         PUSH 0
0070A8CD  |.  6A 00         PUSH 0
0070A8CF  |.  6A 00         PUSH 0
0070A8D1  |.  53            PUSH EBX
0070A8D2  |.  8BD8          MOV EBX,EAX
0070A8D4  |.  33C0          XOR EAX,EAX
0070A8D6  |.  55            PUSH EBP
0070A8D7  |.  68 C3A97000   PUSH ASSET.0070A9C3
0070A8DC  |.  64:FF30       PUSH DWORD PTR FS:[EAX]
0070A8DF  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP
0070A8E2  |.  8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]
0070A8E5  |.  8B83 10030000 MOV EAX,DWORD PTR DS:[EBX+310]
0070A8EB  |.  E8 C852D8FF   CALL ASSET.0048FBB8
0070A8F0  |.  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]
0070A8F3  |.  E8 70E5F4FF   CALL ASSET.00658E68
0070A8F8  |.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]
0070A8FB  |.  8D55 F8       LEA EDX,DWORD PTR SS:[EBP-8]
0070A8FE  |.  E8 59E6F4FF   CALL ASSET.00658F5C
0070A903  |.  8B55 F8       MOV EDX,DWORD PTR SS:[EBP-8]
0070A906  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0070A909  |.  E8 12A1CFFF   CALL ASSET.00404A20                                明码比较
0070A90E  |.  75 46         JNZ SHORT ASSET.0070A956                        关键跳（爆破口）
0070A910  |.  A1 E4007200   MOV EAX,DWORD PTR DS:[7200E4]
0070A915  |.  C600 01       MOV BYTE PTR DS:[EAX],1
0070A918  |.  B9 D8A97000   MOV ECX,ASSET.0070A9D8
0070A91D  |.  BA E4A97000   MOV EDX,ASSET.0070A9E4                   ;  ASCII "Value"
0070A922  |.  B8 F4A97000   MOV EAX,ASSET.0070A9F4                   ;  ASCII "IsReg"
0070A927  |.  E8 60E7F4FF   CALL ASSET.0065908C
0070A92C  |.  8B4D FC       MOV ECX,DWORD PTR SS:[EBP-4]
0070A92F  |.  BA E4A97000   MOV EDX,ASSET.0070A9E4                   ;  ASCII "Value"
0070A934  |.  B8 04AA7000   MOV EAX,ASSET.0070AA04                   ;  ASCII "RegNum"
0070A939  |.  E8 4EE7F4FF   CALL ASSET.0065908C
0070A93E  |.  B2 01         MOV DL,1
0070A940  |.  B8 14AA7000   MOV EAX,ASSET.0070AA14                   ;  注册成功！欢迎您使用
0070A945  |.  E8 6AA8F4FF   CALL ASSET.006551B4
0070A94A  |.  C783 4C020000>MOV DWORD PTR DS:[EBX+24C],1
0070A954  |.  EB 52         JMP SHORT ASSET.0070A9A8
0070A956  |>  A1 E4007200   MOV EAX,DWORD PTR DS:[7200E4]
0070A95B  |.  C600 00       MOV BYTE PTR DS:[EAX],0
0070A95E  |.  33C9          XOR ECX,ECX
0070A960  |.  BA E4A97000   MOV EDX,ASSET.0070A9E4                   ;  ASCII "Value"
0070A965  |.  B8 F4A97000   MOV EAX,ASSET.0070A9F4                   ;  ASCII "IsReg"
0070A96A  |.  E8 1DE7F4FF   CALL ASSET.0065908C
0070A96F  |.  33C9          XOR ECX,ECX
0070A971  |.  BA E4A97000   MOV EDX,ASSET.0070A9E4                   ;  ASCII "Value"
0070A976  |.  B8 04AA7000   MOV EAX,ASSET.0070AA04                   ;  ASCII "RegNum"
0070A97B  |.  E8 0CE7F4FF   CALL ASSET.0065908C
0070A980  |.  B2 01         MOV DL,1
0070A982  |.  B8 40AA7000   MOV EAX,ASSET.0070AA40                   ;  您输入的是错误的注册码，请重新输入
0070A987  |.  E8 28A8F4FF   CALL ASSET.006551B4
0070A98C  |.  8B83 10030000 MOV EAX,DWORD PTR DS:[EBX+310]
0070A992  |.  8B10          MOV EDX,DWORD PTR DS:[EAX]
0070A994  |.  FF92 E0000000 CALL DWORD PTR DS:[EDX+E0]
0070A99A  |.  8B83 10030000 MOV EAX,DWORD PTR DS:[EBX+310]
0070A9A0  |.  8B10          MOV EDX,DWORD PTR DS:[EAX]
0070A9A2  |.  FF92 C4000000 CALL DWORD PTR DS:[EDX+C4]
0070A9A8  |>  33C0          XOR EAX,EAX
0070A9AA  |.  5A            POP EDX
0070A9AB  |.  59            POP ECX
0070A9AC  |.  59            POP ECX
0070A9AD  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
0070A9B0  |.  68 CAA97000   PUSH ASSET.0070A9CA
0070A9B5  |>  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]
0070A9B8  |.  BA 03000000   MOV EDX,3
0070A9BD  |.  E8 769CCFFF   CALL ASSET.00404638
0070A9C2  \.  C3            RETN
```

下面是把CPU号与CPU编号组合之后判断位数是否大于等于14位。如果大于、等于则取前14位，小于则取`19780302198302`。

`CALL ASSET.00658E68`：

```
00658E68  /$  55            PUSH EBP
00658E69  |.  8BEC          MOV EBP,ESP
00658E6B  |.  83C4 E4       ADD ESP,-1C
00658E6E  |.  53            PUSH EBX
00658E6F  |.  56            PUSH ESI
00658E70  |.  57            PUSH EDI
00658E71  |.  33D2          XOR EDX,EDX
00658E73  |.  8955 E4       MOV DWORD PTR SS:[EBP-1C],EDX
00658E76  |.  8955 E8       MOV DWORD PTR SS:[EBP-18],EDX
00658E79  |.  8955 FC       MOV DWORD PTR SS:[EBP-4],EDX
00658E7C  |.  8BF8          MOV EDI,EAX
00658E7E  |.  33C0          XOR EAX,EAX
00658E80  |.  55            PUSH EBP
00658E81  |.  68 368F6500   PUSH ASSET.00658F36
00658E86  |.  64:FF30       PUSH DWORD PTR FS:[EAX]
00658E89  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP
00658E8C  |.  8BC7          MOV EAX,EDI
00658E8E  |.  E8 81B7DAFF   CALL ASSET.00404614
00658E93  |.  8D45 EC       LEA EAX,DWORD PTR SS:[EBP-14]
00658E96  |.  E8 B5FFFFFF   CALL ASSET.00658E50
00658E9B  |.  BE 04000000   MOV ESI,4
00658EA0  |.  8D5D EC       LEA EBX,DWORD PTR SS:[EBP-14]
00658EA3  |>  8D55 E8       /LEA EDX,DWORD PTR SS:[EBP-18]
00658EA6  |.  8B03          |MOV EAX,DWORD PTR DS:[EBX]
00658EA8  |.  E8 A711DBFF   |CALL ASSET.0040A054
00658EAD  |.  8B55 E8       |MOV EDX,DWORD PTR SS:[EBP-18]
00658EB0  |.  8D45 FC       |LEA EAX,DWORD PTR SS:[EBP-4]
00658EB3  |.  E8 24BADAFF   |CALL ASSET.004048DC
00658EB8  |.  83C3 04       |ADD EBX,4
00658EBB  |.  4E            |DEC ESI
00658EBC  |.^ 75 E5         \JNZ SHORT ASSET.00658EA3
00658EBE  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00658EC1  |.  E8 0EBADAFF   CALL ASSET.004048D4              ;  计算位数为25位数
00658EC6  |.  83F8 0E       CMP EAX,0E                       ;  位数小于等于14位就转
```
