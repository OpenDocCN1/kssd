
# 汇编代码片段

```
:004D5333 8D45DC                 lea eax, dword ptr [ebp-24]
:004D5336 E8A5F6F2FF             call 004049E0
:004D533B 8D45F4                 lea eax, dword ptr [ebp-0C]
:004D533E BA03000000             mov edx, 00000003
:004D5343 E8BCF6F2FF             call 00404A04
:004D5348 8D450C                 lea eax, dword ptr [ebp+0C]
:004D534B E890F6F2FF             call 004049E0
:004D5350 C3                      ret
```

---

## 进入关键 CALL

`call 004B0604`

因为各个字符的取位置流程是相同的，因此只是记录了用户名的取值。

* Referenced by a CALL at Addresses:
```
:004D50FA , :004D5108 , :004D5132 , :004D5171 , :004D51A4
:004D51BC , :004D5203 , :004D5211 , :004D5240 , :004D529E
:004D5A57 , :004D6043 , :004D6057 , :004D6143 , :004D6157
:004EBB16 , :00510061 , :00510D54 , :00510D7E , :00513A3E
:00514258 , :00514288 , :00514C09 , :00514C34 , :00519EA5
:00519ECF , :00519F09 , :00519F1A
```

```
:004B0604 33D2                   xor edx, edx
:004B0606 EB01                   jmp 004B0609
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
```
:004B0618(C)
```

```
:004B0608 42                      inc edx
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
```
:004B0606(U)
```

```
:004B0609 33C9                   xor ecx, ecx
:004B060B 8ACA                   mov cl, dl
:004B060D 3A81FC225200           cmp al, byte ptr [ecx+005222FC]
```
> **注释**: 把用户名字符的 HEX 值与下张表中的值比较，取其所在表中的位置

## 内存数据表

`[005222FC]` 内存中有一张表：

```
005222FC  30 31 32 33 34 35 36 37 38 39 41 42 43 44 45 46  0123456789ABCDEF
0052230C  47 48 49 4A 4B 4C 4D 4E 4F 50 51 52 53 54 55 56  GHIJKLMNOPQRSTUV
0052231C  57 58 59 5A 61 62 63 64 65 66 67 68 69 6A 6B 6C  WXYZabcdefghijkl
0052232C  6D 6E 6F 70 71 72 73 74 75 76 77 78 79 7A 40 2E  mnopqrstuvwxyz@.
0052233C  88 22 4B 00                                                  ?K..
```

```
:004B0613 7405                   je 004B061A
:004B0615 80FA40                 cmp dl, 40
:004B0618 72EE                   jb 004B0608
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
```
:004B0613(C)
```

```
:004B061A 33C0                   xor eax, eax
:004B061C 8AC2                   mov al, dl
```

### 映射示例

**fly01**
> 66、6C、79、30、31 分别对应于：29、2F、3C、00、01

**fly01rusreg**
> 66、6C、79、30、31、72、75、73、72、65、67 对应于：29、2F、3C、00、01、35、38、36、35、28、2A

```
:004B061E 83E03F                 and eax, 0000003F
```
> **注释**: EAX AND 0000003F

```
:004B0621 C3                      ret
```

---

## 算法总结

1. 程序通过取用户名、用户名+rusreg、Internet Download Accelerator、E-mail 的字符在表中的位置值参与运算，得出：36
2. 对试炼码进行同样的取值运算得出 00，如果这 2 者相同则 0K！
3. 所以我们可以对注册码以此为目的进行求逆。有很多可能的组合。

---

## 注册信息保存

```
REGEDIT4

[HKEY_CURRENT_USER\Software\2V G\Internet Download Accelerator]
```

- `Name`="fly01"
- `Email`="fly@263.net"
- `REGKEY`="Bt6XnYgqvu1b63uX&quot;"

---

## 整理

- `Name` : fly01
- `E-mail`：fly@263.net
- 注册码：`fly[OCN][FCG]xyz`

简单求逆出一组注册码。

---

### 签名

> 青春都一饷
> 忍把浮名
> 换了 破解轻狂

```
                 Cracked By 巢水工作坊——fly [OCN][FCG]
                            2003-05-09  02:26  4dd466d0e1e0cc123afbe504da2462ca
```
