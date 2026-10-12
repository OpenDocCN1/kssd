
# 万年历 V2.4.053

> 原文：`www.pediy.com/kssd/pediy06/pediy6034.htm`

## 算法浅探——万年历 V2.4.053

下载页面：`www.skycn.com/soft/11961.html`

- 软件大小：241 KB
- 软件语言：简体中文
- 软件类别：国产软件 / 共享版 / 时钟日历
- 应用平台：Win9x/NT/2000/XP
- 加入时间：2003-05-05 11:11:05
- 下载次数：29
- 推荐等级：***

### 软件简介
本万年历工作范围 1901-2100 年，共 200 年。能显示公历、农历日期，交节气时间，年月日时干支，及简单的占卜等；想知道今天的运气如何？可以试试………

### 软件限制
20 次试用。免费软件。

### 作者声明
初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

### 破解工具
`TRW2000` 娃娃修改版、`Ollydbg1.09`、`PEiD`、`AspackDie`、`W32Dasm 9.0` 白金版

---

### 过 程

这个小东东的算法实在是有点麻烦，呵呵，跟到中间有点头大眼花了。^O^ ^O^

`万年历 V2.4.053.exe` 是 `ASPack 2.12` 壳，用 `AspackDie` 脱之。2410K->708K。`Delphi` 编写。

- 用户名：`fly`
- 机器码：`c7a5f8d8`
- 试炼码：`13572468`

---

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:00475C1C(C)
|
:00475C31 8D55F8              lea edx, dword ptr [ebp-08]
:00475C34 8B831C030000        mov eax, dword ptr [ebx+0000031C]
:00475C3A E83924FCFF          call 00438078
:00475C3F 837DF800            cmp dword ptr [ebp-08], 00000000
                                ====>没有注册码？
:00475C43 7523                jne 00475C68
:00475C45 8B831C030000        mov eax, dword ptr [ebx+0000031C]
:00475C4B 8B10                mov edx, dword ptr [eax]
:00475C4D FF92C4000000        call dword ptr [edx+000000C4]
```

* Possible StringData Ref from Code Obj -> "未输入注册码"

```
                                |
:00475C53 BA885E4700            mov edx, 00475E88
:00475C58 8B8308030000          mov eax, dword ptr [ebx+00000308]
:00475C5E E84524FCFF            call 004380A8
:00475C63 E987010000            jmp 00475DEF
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:00475C43(C)
|
:00475C68 8D55F4              lea edx, dword ptr [ebp-0C]
:00475C6B 8B831C030000        mov eax, dword ptr [ebx+0000031C]
:00475C71 E80224FCFF          call 00438078
:00475C76 8B45F4              mov eax, dword ptr [ebp-0C]
                                ====>EAX=13572468           试炼码
:00475C79 50                  push eax
:00475C7A 8D55EC              lea edx, dword ptr [ebp-14]
:00475C7D 8B8314030000        mov eax, dword ptr [ebx+00000314]
:00475C83 E8F023FCFF          call 00438078
:00475C88 8B45EC              mov eax, dword ptr [ebp-14]
                                ====>EAX=c7a5f8d8           机器码
:00475C8B 50                  push eax
:00475C8C 8D45F0              lea eax, dword ptr [ebp-10]
:00475C8F 50                  push eax
:00475C90 8D55E8              lea edx, dword ptr [ebp-18]
:00475C93 A198C54700          mov eax, dword ptr [0047C598]
:00475C98 8B00                mov eax, dword ptr [eax]
:00475C9A E87D1CFEFF          call 0045791C
:00475C9F 8B45E8              mov eax, dword ptr [ebp-18]
:00475CA2 50                  push eax
:00475CA3 8D55E4              lea edx, dword ptr [ebp-1C]
:00475CA6 8B8310030000        mov eax, dword ptr [ebx+00000310]
:00475CAC E8C723FCFF          call 00438078
:00475CB1 8B45E4              mov eax, dword ptr [ebp-1C]
                                ====>EAX=fly                 用户名
```

* Possible StringData Ref from Code Obj -> "China"

```
                                |
:00475CB4 BAA05E4700            mov edx, 00475EA0
                                ====>EDX=China               程序自给的参数
:00475CB9 59                    pop ecx
:00475CBA E8E9F2FFFF            call 00474FA8
                                ====>关键 CALL！进入！
:00475CBF 8B55F0                mov edx, dword ptr [ebp-10]
                                ====>EDX=901e494b             注册码
:00475CC2 58                    pop eax
                                ====>EAX=13572468             试炼码
:00475CC3 E89CE7F8FF            call 00404464
                                ====>比较 CALL！
:00475CC8 742B                  je 00475CF5
                                ====>不跳则 OVER！
:00475CCA 6A30                  push 00000030
```

* Possible StringData Ref from Code Obj -> "警告"

```
                                |
:00475CCC 68A85E4700            push 00475EA8
```

* Possible StringData Ref from Code Obj -> "注册码不符!"

```
                                ====>BAD BOY！
:00475CD1 68B05E4700            push 00475EB0
:00475CD6 8BC3                  mov eax, ebx
:00475CD8 E88F8BFCFF            call 0043E86C
:00475CDD 50                    push eax
```

* Reference To: `user32.MessageBoxA`, Ord:0000h

```
                                |
:00475CDE E88D0BF9FF            Call 00406870
:00475CE3 B201                  mov dl, 01
:00475CE5 8B8330030000          mov eax, dword ptr [ebx+00000330]
:00475CEB E8A822FCFF            call 00437F98
:00475CF0 E9FA000000            jmp 00475DEF
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:00475CC8(C)
|
:00475CF5 33C0                  xor eax, eax
:00475CF7 E80CF4FFFF            call 00475108
:00475CFC 3C02                  cmp al, 02
:00475CFE 0F85EB000000          jne 00475DEF
```

* Possible StringData Ref from Code Obj -> "注册成功"

```
                                ====>呵呵，胜利女神！
:00475D04 BAC45E4700            mov edx, 00475EC4
:00475D09 8B8308030000          mov eax, dword ptr [ebx+00000308]
:00475D0F E89423FCFF            call 004380A8
```

---

### 进入关键 CALL：`00475CBA call 00474FA8`

* Referenced by a CALL at Addresses:

```
|:00475606   , :00475CBA
|
:00474FA8 55                      push ebp
:00474FA9 8BEC                    mov ebp, esp
:00474FAB 83C4EC                  add esp, FFFFFFEC
:00474FAE 53                      push ebx
:00474FAF 33DB                    xor ebx, ebx
:00474FB1 895DEC                  mov dword ptr [ebp-14], ebx
:00474FB4 895DF0                  mov dword ptr [ebp-10], ebx
:00474FB7 894DF4                  mov dword ptr [ebp-0C], ecx
:00474FBA 8955F8                  mov dword ptr [ebp-08], edx
:00474FBD 8945FC                  mov dword ptr [ebp-04], eax
                                ====>EAX=fly                 用户名
:00474FC0 8B45FC                  mov eax, dword ptr [ebp-04]
:00474FC3 E840F5F8FF              call 00404508
:00474FC8 8B45F8                  mov eax, dword ptr [ebp-08]
                                ====>EAX=China               程序自给的参数
:00474FCB E838F5F8FF              call 00404508
:00474FD0 8B45F4                  mov eax, dword ptr [ebp-0C]
                                ====>EAX=万年历 (1901-2100) 程序自给的参数
:00474FD3 E830F5F8FF              call 00404508
:00474FD8 8B450C                  mov eax, dword ptr [ebp+0C]
                                ====>EAX=c7a5f8d8             机器码
:00474FDB E828F5F8FF              call 00404508
:00474FE0 33C0                    xor eax, eax
:00474FE2 55                      push ebp
:00474FE3 684E504700              push 0047504E
:00474FE8 64FF30                  push dword ptr fs:[eax]
:00474FEB 648920                  mov dword ptr fs:[eax], esp
:00474FEE FF75FC                  push [ebp-04]
:00474FF1 FF75F8                  push [ebp-08]
:00474FF4 FF75F4                  push [ebp-0C]
:00474FF7 FF750C                  push [ebp+0C]
:00474FFA 8B450C                  mov eax, dword ptr [ebp+0C]
:00474FFD 50                      push eax
:00474FFE 8D45EC                  lea eax, dword ptr [ebp-14]
:00475001 50                      push eax
:00475002 8B4DF4                  mov ecx, dword ptr [ebp-0C]
```
