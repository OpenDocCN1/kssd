
# APIS32

> 原文：[`www.pediy.com/kssd/pediy03/forum1204.htm`](https://www.pediy.com/kssd/pediy03/forum1204.htm)

《`APIS32`》的注册码算法

目标：`APIS32`

不用多说，相信大家都用过！

作者：`RATARICE[BCG]`

工具：`FI`、`WB2000`、`TRW2000` `1.23`、`W32DSM89`

过程：

一、 用 `FI` 检查，发现是用 `PETITE` `1.2` 压过。用 `WB2000` 找到程序入口，在用 `TRW` 脱掉。

二、 运行软件，填注册信息后，下 `BPX` `HMEMCPY`。被拦，来到下面代码：

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

\* Referenced by a `CALL` at `Addresses`:

`:00401711` , `:004018AF` , `:0040248C` , `:004026B9` , `:00402E96` ----------->共有 5 处检查

|

```
:00405040 51                     push ecx
:00405041 53                     push ebx
:00405042 55                     push ebp
:00405043 56                     push esi
:00405044 57                     push edi
:00405045 6A50                   push 00000050
:00405047 6840B74000             push 0040B740

* Possible StringData Ref from Data Obj ->"UserKey"

:0040504C 6888A64000             push 0040A688
:00405051 E81A030000             call 00405370
:00405056 83C40C                 add esp, 0000000C
:00405059 83F810                 cmp eax, 00000010 ---------------->注册码必须大于等于 16
:0040505C 7D08                   jge 00405066
:0040505E 33C0                   xor eax, eax
:00405060 5F                     pop edi
:00405061 5E                     pop esi
:00405062 5D                     pop ebp
:00405063 5B                     pop ebx
:00405064 59                     pop ecx
:00405065 C3                     ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040505C(C)

|
:00405066 6A2F                   push 0000002F
:00405068 68C0C34000             push 0040C3C0

* Possible StringData Ref from Data Obj ->"UserName"

:0040506D 6878A64000             push 0040A678
:00405072 E8F9020000             call 00405370
:00405077 83C40C                 add esp, 0000000C
:0040507A 83F805                 cmp eax, 00000005 --------------------->名字必须大于等于 5
:0040507D 7D08                   jge 00405087
:0040507F 33C0                   xor eax, eax
:00405081 5F                     pop edi
:00405082 5E                     pop esi
:00405083 5D                     pop ebp
:00405084 5B                     pop ebx
:00405085 59                     pop ecx
:00405086 C3                     ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040507D(C)

|
:00405087 BF40B74000             mov edi, 0040B740
:0040508C 83C9FF                 or ecx, FFFFFFFF
:0040508F 33C0                   xor eax, eax
:00405091 C60551B7400000         mov byte ptr [0040B751], 00
:00405098 F2                     repnz
:00405099 AE                     scasb
:0040509A F7D1                   not ecx
:0040509C 2BF9                   sub edi, ecx
:0040509E 8BC1                   mov eax, ecx
:004050A0 8BF7                   mov esi, edi
:004050A2 BF54B74000             mov edi, 0040B754
:004050A7 C1E902                 shr ecx, 02
:004050AA F3                     repz
:004050AB A5                     movsd
:004050AC 8BC8                   mov ecx, eax
:004050AE 33C0                   xor eax, eax
:004050B0 83E103                 and ecx, 00000003
:004050B3 F3                     repz
:004050B4 A4                     movsb
:004050B5 BF49B74000             mov edi, 0040B749
:004050BA 83C9FF                 or ecx, FFFFFFFF
:004050BD F2                     repnz
:004050BE AE                     scasb
:004050BF F7D1                   not ecx
:004050C1 2BF9                   sub edi, ecx
:004050C3 8BD1                   mov edx, ecx
:004050C5 8BF7                   mov esi, edi
:004050C7 BF5CB74000             mov edi, 0040B75C
:004050CC C1E902                 shr ecx, 02
:004050CF F3                     repz
:004050D0 A5                     movsd
:004050D1 8BCA                   mov ecx, edx
:004050D3 83E103                 and ecx, 00000003
:004050D6 32DB                   xor bl, bl
:004050D8 F3                     repz
:004050D9 A4                     movsb
:004050DA BE41B74000             mov esi, 0040B741
:004050DF BF54B74000             mov edi, 0040B754

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00405103(C)

|                                                   注册码第一次变形
:004050E4 57                     push edi ------------------       
:004050E5 E8E6010000             call 004052D0               |---->要追进去，它算出的数给 al
:004050EA 8ACB                   mov cl, bl               |---->bl 初值等于 0
:004050EC 83C404                 add esp, 00000004             |
:004050EF 80C150                 add cl, 50                 |---->cl=bl+50
:004050F2 83C702                 add edi, 00000002             |
:004050F5 32C1                   xor al, cl                 |---->al 与 cl 异或运算
:004050F7 FEC3                   inc bl                     |---->bl 加一
:004050F9 8846FF                 mov byte ptr [esi-01], al   |---->将结果放入 esi-01
:004050FC C60600                 mov byte ptr [esi], 00      |---->将结果的下一位置 0
:004050FF 46                     inc esi                     |---->将指针进一
:00405100 80FB08                 cmp bl, 08                   |---->要循环 8 次
:00405103 72DF                   jb 004050E4 ---------------               
:00405105 6854B74000             push 0040B754   注意： 注册码计算后变形，记作：（1）
:0040510A 6840B74000             push 0040B740
:0040510F E8EC010000             call 00405300 ------------------>注册码的第二次变形，要追入！
:00405114 BFC0C34000             mov edi, 0040C3C0
:00405119 83C9FF                 or ecx, FFFFFFFF
:0040511C 33C0                   xor eax, eax
:0040511E 83C408                 add esp, 00000008
:00405121 F2                     repnz
:00405122 AE                     scasb
:00405123 F7D1                   not ecx
:00405125 2BF9                   sub edi, ecx
:00405127 33ED                   xor ebp, ebp
:00405129 8BD1                   mov edx, ecx
:0040512B 8BF7                   mov esi, edi
:0040512D BF5EB74000             mov edi, 0040B75E
:00405132 C1E902                 shr ecx, 02
:00405135 F3                     repz
:00405136 A5                     movsd
:00405137 8BCA                   mov ecx, edx
:00405139 83E103                 and ecx, 00000003
:0040513C F3                     repz
:0040513D A4                     movsb
:0040513E BFC0C34000             mov edi, 0040C3C0
:00405143 83C9FF                 or ecx, FFFFFFFF
:00405146 F2                     repnz
:00405147 AE                     scasb
:00405148 F7D1                   not ecx
:0040514A 49                     dec ecx
:0040514B 80F908                 cmp cl, 08
:0040514E 884C2410               mov byte ptr [esp+10], cl
:00405152 7330                   jnb 00405184
:00405154 8B542410               mov edx, dword ptr [esp+10]
:00405158 BFC0C34000             mov edi, 0040C3C0
:0040515D 81E2FF000000           and edx, 000000FF
:00405163 83C9FF                 or ecx, FFFFFFFF
:00405166 81C25EB74000           add edx, 0040B75E
:0040516C F2                     repnz
:0040516D AE                     scasb
:0040516E F7D1                   not ecx
:00405170 2BF9                   sub edi, ecx
:00405172 8BC1                   mov eax, ecx
:00405174 8BF7                   mov esi, edi
:00405176 8BFA                   mov edi, edx
```



## 汇编代码段 1

```
:00405178 C1E902                 shr ecx, 02
:0040517B F3                     repz
:0040517C A5                     movsd
:0040517D 8BC8                   mov ecx, eax
:0040517F 83E103                 and ecx, 00000003
:00405182 F3                     repz
:00405183 A4                     movsb

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405152(C)
|

:00405184 C60566B7400000         mov byte ptr [0040B766], 00 -------------->取名字的前 8 位！
:0040518B B954B74000             mov ecx, 0040B754
:00405190 BE08000000             mov esi, 00000008

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004051B9(C)
|

:00405195 8A01                   mov al, byte ptr [ecx] --------------
:00405197 3C20                   cmp al, 20    |
:00405199 730E                   jnb 004051A9      |
:0040519B 33D2                   xor edx, edx      |
:0040519D 25FF000000             and eax, 000000FF    |
:004051A2 8A510A                 mov dl, byte ptr [ecx+0A]           |
:004051A5 0BD0                   or edx, eax                     |
:004051A7 EB0C                   jmp 004051B5                    |
      |
* Referenced by a (U)nconditional or (C)onditional Jump at Address:    |
|:00405199(C)                    | 分别取（2）与名字
|                                | 进行异或运算
:004051A9 33D2                   xor edx, edx                     | 结果累加到 ebp
:004051AB 25FF000000             and eax, 000000FF                |
:004051B0 8A510A                 mov dl, byte ptr [ecx+0A]           |
:004051B3 33D0                   xor edx, eax                     |
      |
* Referenced by a (U)nconditional or (C)onditional Jump at Address:    |
|:004051A7(U)                    |
|                                |
:004051B5 03EA                   add ebp, edx                       |
:004051B7 41                     inc ecx                        |
:004051B8 4E                     dec esi                        |
:004051B9 75DA                   jne 00405195 ------------------------
:004051BB 33C0                   xor eax, eax
:004051BD 5F                     pop edi
:004051BE 85ED                   test ebp, ebp ----------------------->检查 ebp 是否为 0，若为 0
:004051C0 5E                     pop esi            则注册成功，反之失败！
:004051C1 5D                     pop ebp
:004051C2 0F94C0                 sete al ----------------------------->置标志位
:004051C5 5B                     pop ebx
:004051C6 59                     pop ecx
:004051C7 C3                     ret
```

***

***

## 汇编代码段 2

```
* Referenced by a CALL at Address:
|:004050E5
|

:004052D0 8B4C2404               mov ecx, dword ptr [esp+04] ---------->将注册码的地址赋给 ecx
:004052D4 8A01                   mov al, byte ptr [ecx] --------------->取第一个数给 al
:004052D6 3C39                   cmp al, 39 --------------------------->与 39 比较
:004052D8 7E04                   jle 004052DE ------------------------->小于等于就跳
:004052DA 04C9                   add al, C9 --------------------------->若大于 al=al+c9
:004052DC EB02                   jmp 004052E0 ------------------------->跳

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004052D8(C)
|

:004052DE 04D0                   add al, D0 --------------------------->al=al+d0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004052DC(U)
|

:004052E0 8A4901                 mov cl, byte ptr [ecx+01] ------------>取第二个数给 cl
:004052E3 80F939                 cmp cl, 39 --------------------------->与 39 比较
:004052E6 7E09                   jle 004052F1 ------------------------->小于等于就跳
:004052E8 C0E004                 shl al, 04 --------------------------->若大于 al 逻辑左移 4 位
:004052EB 80E937                 sub cl, 37 --------------------------->cl=cl-37
:004052EE 0AC1                   or al, cl ---------------------------->al 与 cl 进行或运算
:004052F0 C3                     ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004052E6(C)
|

:004052F1 C0E004                 shl al, 04 -------------------------->al 逻辑左移 4 位
:004052F4 80E930                 sub cl, 30 -------------------------->cl=cl-30
:004052F7 0AC1                   or al, cl --------------------------->al 与 cl 进行或运算
:004052F9 C3                     ret
```

***

***

## 汇编代码段 3

```
* Referenced by a CALL at Address:  共两重循环，见下：
|:0040510F
|

:00405300 53                     push ebx
:00405301 55                     push ebp
:00405302 8B6C2410               mov ebp, dword ptr [esp+10]
:00405306 56                     push esi
:00405307 57                     push edi
:00405308 8B7C2414               mov edi, dword ptr [esp+14]
:0040530C 33C9                   xor ecx, ecx
:0040530E 2BFD                   sub edi, ebp
:00405310 897C2418               mov dword ptr [esp+18], edi
:00405314 EB04                   jmp 0040531A

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405364(C)
|

:00405316 8B7C2418               mov edi, dword ptr [esp+18]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405314(U)
|

:0040531A 8D3429                 lea esi, dword ptr [ecx+ebp]
:0040531D 33D2                   xor edx, edx
:0040531F B801000000             mov eax, 00000001
:00405324 C744241407000000       mov [esp+14], 00000007
:0040532C 8A1437                 mov dl, byte ptr [edi+esi]
:0040532F 8BFA                   mov edi, edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405350(C)
|

:00405331 8BD7                   mov edx, edi -----------------------  取出（1）中的数，自己与
:00405333 0FAFC2                 imul eax, edx                          | 自己相乘得一个数，直到
:00405336 3D99880000             cmp eax, 00008899                       | 这个数大于 8899，再将这个
:0040533B 7E0A                   jle 00405347                            | 数与 8899 相除，取余。共乘
:0040533D 99                     cdq                                     | 8 会（1）中的数。最后将
:0040533E BB99880000             mov ebx, 00008899                       | 得数赋给 eax。
:00405343 F7FB                   idiv ebx                                |
:00405345 8BC2                   mov eax, edx                            | 退出第一重循环，进入第二
:00405345                         |                                      重循环！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  |
|:0040533B(C)                                           |
|                                               |

:00405347 8B542414               mov edx, dword ptr [esp+14]        |
:0040534B 4A                     dec edx                             |
:0040534C 89542414               mov dword ptr [esp+14], edx        |
:00405350 75DF                   jne 00405331 -----------------------                   |
:00405352 99                     cdq --------------------------------  再将 eax 与 bb 相除，把余数
:00405353 BFBB000000             mov edi, 000000BB                   | 作为结果保存。
:00405358 F7FF                   idiv edi                              |
:0040535A 41                     inc ecx                             | 这是第二重循环，共 8 回！
```



```
:0040535B 83F908                 cmp ecx, 00000008
:0040535E 8816                   mov byte ptr [esi], dl
:00405360 C6042900               mov byte ptr [ecx+ebp], 00
:00405364 7CB0                   jl 00405316 ------------------------
:00405366 5F                     pop edi
:00405367 5E                     pop esi
:00405368 5D                     pop ebp
:00405369 5B                     pop ebx
:0040536A C3                     ret
```

三、 虽然明白了它的注册算法，但终因本人功力不够，没能写出注册机，还请大侠们帮忙写出它的注册机，让我也可早日拿到注册码！！！也能从中多学点知识！！！

！！！！！！！！！！！！！！！！！！静待佳音！！！！！！！！！！！！！！！！！！ `4dd466d0e1e0cc123afbe504da2462ca`
