
# Clone ExeShield Protector License Generator

> 原文：[`www.pediy.com/kssd/pediy10/57801.html`](https://www.pediy.com/kssd/pediy10/57801.html)

Clone ExeShield Protector License Generator

Writer by KuNgBiM/[CCG]

Email: `kungbim@163.com`

Date: January 6, 2008

Debuger Tools: `OllyICE`

Clone License Generator Tools: `ExeShield Protector` (Arbitrary Version > `v3.8.1.7`)

## 【前言】

大家研究 `ExeShield Protector` 最主要的目的，无非也就是研究怎样解除它所保护的程序中，使用的无源码授权保护功能。前段时间我从官网下载了一个 Trial 版本的 `ExeShield Protector`，经过一番乱弹，本来打算写篇 `ExeShield Protector` 脱壳分析，不过，深入研究发现，该款加密保护壳，居然存在与早期的 `EncryptPE` 相同的授权功能 BUG，从而使我们可以利用该漏洞制作出对应程序的 Clone License Generator。

## 【为了公正起见，程序由我朋友加密，以下是我朋友提供给我的一部分加密设置】

```
--------------[Protection Options]--------------

[Protector Name]

ExeShield Protector

[Protector Version]

3.8.1.7 Pro

[Protection Options]

Use Enhanced Protection

[Hardware Usage]

N/A

[Compression Options]

Max

[Other Options]

Expire if clock is set back or forward
TimeStamp Date Check
Reset Trial Period on Newer Version
Force Client to Re-Register
Anti-KeyGen

---------------------[Keys]（未提供）---------------------

[Keys Type]

Unkown

--------------------[License]（未提供）-------------------

[License(s) Type]

Unkown

[Encryption Key for this License]

Unkown
```

## 【分析正文】

`OllyICE` 载入目标程序（入口貌似：`PECompact`）

```
00401000 >   B8 D0BB9B00              mov eax,NOTEPAD.009BBBD0                 ; EP
00401005    50                       push eax
00401006    64:FF35 00000000         push dword ptr fs:[0]                     ; hr esp
0040100D    64:8925 00000000         mov dword ptr fs:[0],esp
00401014    33C0                     xor eax,eax
00401016    8908                     mov dword ptr ds:[eax],ecx
00401018    50                       push eax
00401019    45                       inc ebp
0040101A    43                       inc ebx
0040101B    6F                       outsd
0040101C    6D                       insd
0040101D    70 61                    jo short NOTEPAD.00401080
0040101F    637432 00                arpl word ptr ds:[edx+esi],si
00401023    CE                       into
```

（2 次 `F8` 后，利用 `ESP` 定律可到达加密程序的授权模块内）

连续 7 次 `Shift+F9` 后到达：

```
0046DE50    55                       push ebp                                   ; 授权模块界面 EP
0046DE51    8BEC                     mov ebp,esp
0046DE53    83C4 F4                  add esp,-0C
0046DE56    53                       push ebx
0046DE57    B8 40DC4600              mov eax,NOTEPAD.0046DC40
0046DE5C    E8 6384F9FF              call NOTEPAD.004062C4
0046DE61    8B1D 442D4700            mov ebx,dword ptr ds:[472D44]              ; NOTEPAD.004737B0
0046DE67    8B03                     mov eax,dword ptr ds:[ebx]
0046DE69    E8 3A6FFDFF              call NOTEPAD.00444DA8
0046DE6E    8B03                     mov eax,dword ptr ds:[ebx]
0046DE70    BA C8DE4600              mov edx,NOTEPAD.0046DEC8
0046DE75    E8 526BFDFF              call NOTEPAD.004449CC
0046DE7A    8B0D 742B4700            mov ecx,dword ptr ds:[472B74]              ; NOTEPAD.00473AD0
0046DE80    8B03                     mov eax,dword ptr ds:[ebx]
0046DE82    8B15 603E4600            mov edx,dword ptr ds:[463E60]              ; NOTEPAD.00463EAC
0046DE88    E8 336FFDFF              call NOTEPAD.00444DC0
0046DE8D    8B0D B82B4700            mov ecx,dword ptr ds:[472BB8]              ; NOTEPAD.00473AB4
0046DE93    8B03                     mov eax,dword ptr ds:[ebx]
0046DE95    8B15 1C2F4600            mov edx,dword ptr ds:[462F1C]              ; NOTEPAD.00462F68
0046DE9B    E8 206FFDFF              call NOTEPAD.00444DC0
0046DEA0    8B0D 682E4700            mov ecx,dword ptr ds:[472E68]              ; NOTEPAD.00473AC0
0046DEA6    8B03                     mov eax,dword ptr ds:[ebx]
0046DEA8    8B15 3C394600            mov edx,dword ptr ds:[46393C]              ; NOTEPAD.00463988
```

到达授权模块界面 `EP` 后，删除硬件断点。

## 【License(s) Type 分析】

命令函数断点：`bp SetEnvironmentVariableA`

`Shift+F9` 运行后中断（注意观察调用窗）：

```
7C833400 >   8BFF                     mov edi,edi                                ; 中断在此，Alt+F9 返回
7C833402    55                       push ebp
7C833403    8BEC                     mov ebp,esp
7C833405    83EC 20                  sub esp,20
7C833408    56                       push esi
7C833409    8B35 8812807C            mov esi,dword ptr ds:[<&ntdll.RtlInitStr>; ntdll.RtlInitString
7C83340F    57                       push edi
7C833410    FF75 08                  push dword ptr ss:[ebp+8]
7C833413    8D45 E8                  lea eax,dword ptr ss:[ebp-18]
7C833416    50                       push eax
7C833417    FFD6                     call esi
7C833419    8B3D 8810807C            mov edi,dword ptr ds:[<&ntdll.RtlAnsiStr>; ntdll.RtlAnsiStringToUnicodeString
7C83341F    6A 01                    push 1
7C833421    8D45 E8                  lea eax,dword ptr ss:[ebp-18]
7C833424    50                       push eax
7C833425    8D45 F8                  lea eax,dword ptr ss:[ebp-8]
7C833428    50                       push eax
7C833429    FFD7                     call edi
```

```
---------------------------------【调用窗】----------------------------------

000CD7E0   004026FE   NOTEPAD.004026FE
000CD7E4   0103E794   ASCII "Trial License"
000CD7E8   0046B56A   /CALL 到 SetEnvironmentVariableA 来自 NOTEPAD.0046B565
000CD7EC   0046D9C4   |VarName = "LICENSE"
000CD7F0   0103E794   \Value = "Trial License"
000CD7F4   000CD800   指向下一个 SEH 记录的指针
000CD7F8   0046B596   SE 处理程序

-----------------------------------------------------------------------------
```

★从上数据可知，在目标程序所设置的 License(s) Type 中，至少包含一种"Trial License"类型的授权方式。

另外，在数据窗中，可以找到其它类型的授权方式，所以我们这时可以偷个小懒，省略分析其他 License(s) Type。

```
-----------------------------【数据窗中数据】------------------------------

0099DE4B  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0099DE5B  0D 54 72 69 61 6C 20 4C 69 63 65 6E 73 65 00 00  .Trial License..
0099DE6B  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0099DE7B  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0099DE8B  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0099DE9B  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0099DEAB  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0099DEBB  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0099DECB  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0099DEDB  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0099DEEB  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0099DEFB  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0099DF0B  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0099DF1B  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0099DF2B  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0099DF3B  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0099DF4B  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0099DF5B  12 52 65 67 69 73 74 65 72 65 64 20 4C 69 63 65  Registered Lice
0099DF6B  6E 73 65 00 00 00 00 00 00 00 00 00 00 00 00 00  nse.............
0099DF7B  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0099DF8B  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0099DF9B  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................

-----------------------------------------------------------------------------
```

★在数据窗上下查找了一番，发现程序作者只设立 2 种类型的授权方式：

1. Trial License
2. Registered License

返回在此：（下面均为 `F8` 跟踪调试）

```
0046B55F    50                       push eax                                   ; 【分析调试开始】
```



```
0046B560    68 C4D94600               push NOTEPAD.0046D9C4                 ; ASCII "LICENSE"

0046B565    E8 F6B0F9FF               call NOTEPAD.00406660                ; jmp 到 kernel32.SetEnvironmentVariableA

0046B56A    68 CCD94600               push NOTEPAD.0046D9CC                ; ASCII "TRUE"

0046B56F    68 D4D94600               push NOTEPAD.0046D9D4                ; ASCII "PROTECTED"

0046B574    E8 E7B0F9FF               call NOTEPAD.00406660                ; jmp 到 kernel32.SetEnvironmentVariableA

0046B579    8B45 94                   mov eax,dword ptr ss:[ebp-6C]        ; ASCII "A512-A7C1"

0046B57C    E8 DF8AF9FF               call NOTEPAD.00404060

0046B581    50                        push eax

0046B582    68 E0D94600               push NOTEPAD.0046D9E0                ; ASCII "REGNUMBER"

0046B587    E8 D4B0F9FF               call NOTEPAD.00406660                ; jmp 到 kernel32.SetEnvironmentVariableA

0046B58C    33C0                      xor eax,eax

0046B58E    5A                        pop edx

0046B58F    59                        pop ecx

0046B590    59                        pop ecx

0046B591    64:8910                   mov dword ptr fs:[eax],edx

0046B594    EB 0A                     jmp short NOTEPAD.0046B5A0

0046B596  ^ E9 B97EF9FF               jmp NOTEPAD.00403454

0046B59B    E8 6481F9FF               call NOTEPAD.00403704

0046B5A0    8D85 70FFFFFF             lea eax,dword ptr ss:[ebp-90]

0046B5A6    BA 90EC9900               mov edx,NOTEPAD.0099EC90

0046B5AB    E8 9088F9FF               call NOTEPAD.00403E40

0046B5B0    33C0                      xor eax,eax

0046B5B2    55                        push ebp

0046B5B3    68 FFB54600               push NOTEPAD.0046B5FF

0046B5B8    64:FF30                   push dword ptr fs:[eax]

0046B5BB    64:8920                   mov dword ptr fs:[eax],esp

0046B5BE    83BD 70FFFFFF 00          cmp dword ptr ss:[ebp-90],0          ; 用户名长度是否等于 0

0046B5C5    74 16                     je short NOTEPAD.0046B5DD

0046B5C7    8B85 70FFFFFF             mov eax,dword ptr ss:[ebp-90]

0046B5CD    E8 8E8AF9FF               call NOTEPAD.00404060

0046B5D2    50                        push eax

0046B5D3    68 ECD94600               push NOTEPAD.0046D9EC                ; ASCII "USERNAME"

0046B5D8    E8 83B0F9FF               call NOTEPAD.00406660                ; jmp 到 kernel32.SetEnvironmentVariableA

0046B5DD    83BD 70FFFFFF 00          cmp dword ptr ss:[ebp-90],0          ; 用户名长度等于 0，那么判断为未授权版本

0046B5E4    75 0F                     jnz short NOTEPAD.0046B5F5

0046B5E6    68 F8D94600               push NOTEPAD.0046D9F8                ; ASCII "NOT REGISTERED"

0046B5EB    68 ECD94600               push NOTEPAD.0046D9EC                ; ASCII "USERNAME"

0046B5F0    E8 6BB0F9FF               call NOTEPAD.00406660                ; jmp 到 kernel32.SetEnvironmentVariableA

0046B5F5    33C0                      xor eax,eax

0046B5F7    5A                        pop edx

0046B5F8    59                        pop ecx

0046B5F9    59                        pop ecx

0046B5FA    64:8910                   mov dword ptr fs:[eax],edx

0046B5FD    EB 0A                     jmp short NOTEPAD.0046B609

0046B5FF  ^ E9 507EF9FF               jmp NOTEPAD.00403454

0046B604    E8 FB80F9FF               call NOTEPAD.00403704

0046B609    83BD 70FFFFFF 00          cmp dword ptr ss:[ebp-90],0          ; 此处同样是作注册标志判断

0046B610    74 0F                     je short NOTEPAD.0046B621

0046B612    A1 A02C4700               mov eax,dword ptr ds:[472CA0]

0046B617    BA 4CD94600               mov edx,NOTEPAD.0046D94C            ; ASCII "REGISTERED"

0046B61C    E8 5386F9FF               call NOTEPAD.00403C74

0046B621    83BD 70FFFFFF 00          cmp dword ptr ss:[ebp-90],0          ; 同上

0046B628    75 0F                     jnz short NOTEPAD.0046B639

0046B62A    A1 A02C4700               mov eax,dword ptr ds:[472CA0]

0046B62F    BA 10DA4600               mov edx,NOTEPAD.0046DA10            ; ASCII "NOTREGISTERED"

0046B634    E8 3B86F9FF               call NOTEPAD.00403C74

0046B639    B9 F4D54600               mov ecx,NOTEPAD.0046D5F4            ; ASCII "netctrl.ini"

0046B63E    B2 01                     mov dl,1

0046B640    A1 60134500               mov eax,dword ptr ds:[451360]

0046B645    E8 BE5DFEFF               call NOTEPAD.00451408               ; 授权文件信息检测

0046B64A    A3 20F09900               mov dword ptr ds:[99F020],eax

0046B64F    6A 00                     push 0

0046B651    8D85 34FFFFFF             lea eax,dword ptr ss:[ebp-CC]

0046B657    50                        push eax

0046B658    8B15 7C2E4700             mov edx,dword ptr ds:[472E7C]        ; ASCII "NOTEPAD.LIC"

0046B65E    8B12                      mov edx,dword ptr ds:[edx]

0046B660    B9 28DA4600               mov ecx,NOTEPAD.0046DA28            ; ASCII "Serial"

0046B665    A1 20F09900               mov eax,dword ptr ds:[99F020]

0046B66A    8B18                      mov ebx,dword ptr ds:[eax]

0046B66C    FF13                      call dword ptr ds:[ebx]

0046B66E    B2 01                     mov dl,1

0046B670    A1 20F09900               mov eax,dword ptr ds:[99F020]

0046B675    8B08                      mov ecx,dword ptr ds:[eax]

0046B677    FF51 FC                   call dword ptr ds:[ecx-4]

0046B67A    80BD 1FFFFFFF 01          cmp byte ptr ss:[ebp-E1],1           ; 小于 1 则表明没有授权文件

0046B681    75 0C                     jnz short NOTEPAD.0046B68F

0046B683    C685 3FFFFFFF 01          mov byte ptr ss:[ebp-C1],1

0046B68A    E9 80150000               jmp NOTEPAD.0046CC0F

0046B68F    8D85 84DEF9FF             lea eax,dword ptr ss:[ebp+FFF9DE84]

0046B695    BA 15CD9900               mov edx,NOTEPAD.0099CD15

0046B69A    E8 A187F9FF               call NOTEPAD.00403E40               ; 难道是授权码？不会吧

0046B69F    8B95 84DEF9FF             mov edx,dword ptr ss:[ebp+FFF9DE84]  ; ASCII "FD41-3CFC-81AE-BBA3-D3B2"

0046B6A5    8B45 DC                   mov eax,dword ptr ss:[ebp-24]

0046B6A8    E8 FF88F9FF               call NOTEPAD.00403FAC

0046B6AD    75 20                     jnz short NOTEPAD.0046B6CF

0046B6AF    C605 58F09900 01          mov byte ptr ds:[99F058],1

0046B6B6    803D 48F09900 00          cmp byte ptr ds:[99F048],0

0046B6BD    0F85 4C150000             jnz NOTEPAD.0046CC0F

0046B6C3    C685 3FFFFFFF 01          mov byte ptr ss:[ebp-C1],1

0046B6CA    E9 40150000               jmp NOTEPAD.0046CC0F

0046B6CF    803D 99ED9900 01          cmp byte ptr ds:[99ED99],1           ; 小于 1 则表明没有授权码

0046B6D6    0F85 B4000000             jnz NOTEPAD.0046B790

0046B6DC    E8 5BE0F9FF               call NOTEPAD.0040973C

0046B6E1    DC1D 91ED9900             fcomp qword ptr ds:[99ED91]

0046B6E7    DFE0                      fstsw ax

0046B6E9    9E                        sahf

0046B6EA    0F82 A0000000             jb NOTEPAD.0046B790

0046B6F0    C605 07CD9900 01          mov byte ptr ds:[99CD07],1

0046B6F7    8D85 6CDEF9FF             lea eax,dword ptr ss:[ebp+FFF9DE6C]

0046B6FD    E8 A688FEFF               call NOTEPAD.00453FA8

0046B702    FFB5 6CDEF9FF             push dword ptr ss:[ebp+FFF9DE6C]

0046B708    68 9CD84600               push NOTEPAD.0046D89C

0046B70D    FF75 E0                   push dword ptr ss:[ebp-20]

0046B710    8D85 84DEF9FF             lea eax,dword ptr ss:[ebp+FFF9DE84]

0046B716    BA 03000000               mov edx,3

0046B71B    E8 3C88F9FF               call NOTEPAD.00403F5C

0046B720    8B95 84DEF9FF             mov edx,dword ptr ss:[ebp+FFF9DE84]

0046B726    8B45 FC                   mov eax,dword ptr ss:[ebp-4]

0046B729    E8 DEBBFFFF               call NOTEPAD.0046730C
```



0046B72E 6A 00 `push 0`

0046B730 6A 01 `push 1`

0046B732 A1 0CF09900 `mov eax,dword ptr ds:\[99F00C\]`

0046B737 50 `push eax`

0046B738 E8 03AFF9FF `call NOTEPAD.00406640` ; jmp 到 kernel32.ReleaseSemaphore

0046B73D 8B45 FC `mov eax,dword ptr ss:\[ebp-4\]`

0046B740 E8 E3B6FFFF `call NOTEPAD.00466E28`

0046B745 A1 A02C4700 `mov eax,dword ptr ds:\[472CA0\]`

0046B74A 8B00 `mov eax,dword ptr ds:\[eax\]`

0046B74C BA 4CD94600 `mov edx,NOTEPAD.0046D94C` ; ASCII "REGISTERED"

0046B751 E8 5688F9FF `call NOTEPAD.00403FAC`

0046B756 75 0C `jnz short NOTEPAD.0046B764`

0046B758 C685 3FFFFFFF 01 `mov byte ptr ss:\[ebp-C1\],1`

0046B75F E9 AB140000 `jmp NOTEPAD.0046CC0F`

0046B764 A1 A02C4700 `mov eax,dword ptr ds:\[472CA0\]`

0046B769 8B00 `mov eax,dword ptr ds:\[eax\]`

0046B76B BA 4CD94600 `mov edx,NOTEPAD.0046D94C` ; ASCII "REGISTERED"

0046B770 E8 3788F9FF `call NOTEPAD.00403FAC`

0046B775 74 19 `je short NOTEPAD.0046B790`

0046B777 A1 442D4700 `mov eax,dword ptr ds:\[472D44\]`

0046B77C 8B00 `mov eax,dword ptr ds:\[eax\]`

0046B77E E8 7197FDFF `call NOTEPAD.00444EF4`

0046B783 8B45 FC `mov eax,dword ptr ss:\[ebp-4\]`

0046B786 E8 C164FDFF `call NOTEPAD.00441C4C`

0046B78B E9 5C1D0000 `jmp NOTEPAD.0046D4EC`

0046B790 803D 85EC9900 00 `cmp byte ptr ds:\[99EC85\],0`

0046B797 0F84 C4000000 `je NOTEPAD.0046B861`

0046B79D 8D85 84DEF9FF `lea eax,dword ptr ss:\[ebp+FFF9DE84\]` ; 授权信息标示，类似于 SentinelLM 流水号

0046B7A3 BA 85EC9900 `mov edx,NOTEPAD.0099EC85` ; ASCII 05,"43363"

0046B7A8 E8 9386F9FF `call NOTEPAD.00403E40`

0046B7AD 8B85 84DEF9FF `mov eax,dword ptr ss:\[ebp+FFF9DE84\]`

0046B7B3 8B55 D4 `mov edx,dword ptr ss:\[ebp-2C\]` ; ASCII "43363"

0046B7B6 E8 F187F9FF `call NOTEPAD.00403FAC` ; 是否存在授权标示

0046B7BB 0F84 A0000000 `je NOTEPAD.0046B861` ; 不存在就跳过

0046B7C1 C605 07CD9900 01 `mov byte ptr ds:\[99CD07\],1`

0046B7C8 8D85 6CDEF9FF `lea eax,dword ptr ss:\[ebp+FFF9DE6C\]`

0046B7CE E8 D587FEFF `call NOTEPAD.00453FA8`

0046B7D3 FFB5 6CDEF9FF `push dword ptr ss:\[ebp+FFF9DE6C\]`

0046B7D9 68 9CD84600 `push NOTEPAD.0046D89C`

0046B7DE FF75 E0 `push dword ptr ss:\[ebp-20\]`

0046B7E1 8D85 84DEF9FF `lea eax,dword ptr ss:\[ebp+FFF9DE84\]`

0046B7E7 BA 03000000 `mov edx,3`

0046B7EC E8 6B87F9FF `call NOTEPAD.00403F5C`

0046B7F1 8B95 84DEF9FF `mov edx,dword ptr ss:\[ebp+FFF9DE84\]`

0046B7F7 8B45 FC `mov eax,dword ptr ss:\[ebp-4\]`

0046B7FA E8 0DBBFFFF `call NOTEPAD.0046730C`

0046B7FF 6A 00 `push 0`

0046B801 6A 01 `push 1`

0046B803 A1 0CF09900 `mov eax,dword ptr ds:\[99F00C\]`

0046B808 50 `push eax`

0046B809 E8 32AEF9FF `call NOTEPAD.00406640` ; jmp 到 kernel32.ReleaseSemaphore

0046B80E 8B45 FC `mov eax,dword ptr ss:\[ebp-4\]`

0046B811 E8 12B6FFFF `call NOTEPAD.00466E28`

0046B816 A1 A02C4700 `mov eax,dword ptr ds:\[472CA0\]`

0046B81B 8B00 `mov eax,dword ptr ds:\[eax\]`

0046B81D BA 4CD94600 `mov edx,NOTEPAD.0046D94C` ; ASCII "REGISTERED"

0046B822 E8 8587F9FF `call NOTEPAD.00403FAC`

0046B827 75 0C `jnz short NOTEPAD.0046B835`

0046B829 C685 3FFFFFFF 01 `mov byte ptr ss:\[ebp-C1\],1`

0046B830 E9 DA130000 `jmp NOTEPAD.0046CC0F`

0046B835 A1 A02C4700 `mov eax,dword ptr ds:\[472CA0\]`

0046B83A 8B00 `mov eax,dword ptr ds:\[eax\]`

0046B83C BA 4CD94600 `mov edx,NOTEPAD.0046D94C` ; ASCII "REGISTERED"

0046B841 E8 6687F9FF `call NOTEPAD.00403FAC`

0046B846 74 19 `je short NOTEPAD.0046B861`

0046B848 A1 442D4700 `mov eax,dword ptr ds:\[472D44\]`

0046B84D 8B00 `mov eax,dword ptr ds:\[eax\]`

0046B84F E8 A096FDFF `call NOTEPAD.00444EF4`

0046B854 8B45 FC `mov eax,dword ptr ss:\[ebp-4\]`

0046B857 E8 F063FDFF `call NOTEPAD.00441C4C`

0046B85C E9 8B1C0000 `jmp NOTEPAD.0046D4EC`

0046B861 837D D4 00 `cmp dword ptr ss:\[ebp-2C\],0`

0046B865 74 0A `je short NOTEPAD.0046B871`

0046B867 837D D0 00 `cmp dword ptr ss:\[ebp-30\],0`

0046B86B 0F85 7A010000 `jnz NOTEPAD.0046B9EB` ; 继续跳过

0046B871 C605 07CD9900 01 `mov byte ptr ds:\[99CD07\],1`

0046B878 8D85 6CDEF9FF `lea eax,dword ptr ss:\[ebp+FFF9DE6C\]`

0046B87E E8 2587FEFF `call NOTEPAD.00453FA8`

0046B883 FFB5 6CDEF9FF `push dword ptr ss:\[ebp+FFF9DE6C\]`

0046B889 68 9CD84600 `push NOTEPAD.0046D89C`

0046B88E FF75 E0 `push dword ptr ss:\[ebp-20\]`

0046B891 8D85 84DEF9FF `lea eax,dword ptr ss:\[ebp+FFF9DE84\]`

0046B897 BA 03000000 `mov edx,3`

0046B89C E8 BB86F9FF `call NOTEPAD.00403F5C`

0046B8A1 8B95 84DEF9FF `mov edx,dword ptr ss:\[ebp+FFF9DE84\]`

0046B8A7 8B45 FC `mov eax,dword ptr ss:\[ebp-4\]`

0046B8AA E8 5DBAFFFF `call NOTEPAD.0046730C`

0046B8AF 8D85 D4DCF9FF `lea eax,dword ptr ss:\[ebp+FFF9DCD4\]`

0046B8B5 8D95 FFFCFFFF `lea edx,dword ptr ss:\[ebp-301\]`

0046B8BB B9 91000000 `mov ecx,91`

0046B8C0 E8 3F75F9FF `call NOTEPAD.00402E04`

0046B8C5 8D95 D4DCF9FF `lea edx,dword ptr ss:\[ebp+FFF9DCD4\]`

0046B8CB 8D85 40DCF9FF `lea eax,dword ptr ss:\[ebp+FFF9DC40\]`

0046B8D1 E8 6672F9FF `call NOTEPAD.00402B3C`

0046B8D6 BA 30DA4600 `mov edx,NOTEPAD.0046DA30`

0046B8DB 8D85 40DCF9FF `lea eax,dword ptr ss:\[ebp+FFF9DC40\]`

0046B8E1 B1 92 `mov cl,92`

0046B8E3 E8 2472F9FF `call NOTEPAD.00402B0C`

0046B8E8 8D95 40DCF9FF `lea edx,dword ptr ss:\[ebp+FFF9DC40\]`

0046B8EE 8D85 84DEF9FF `lea eax,dword ptr ss:\[ebp+FFF9DE84\]`

0046B8F4 E8 4785F9FF `call NOTEPAD.00403E40`

0046B8F9 8D85 84DEF9FF `lea eax,dword ptr ss:\[ebp+FFF9DE84\]`

0046B8FF 8B55 E0 `mov edx,dword ptr ss:\[ebp-20\]`

0046B902 E8 9D85F9FF `call NOTEPAD.00403EA4`

0046B907 8B85 84DEF9FF `mov eax,dword ptr ss:\[ebp+FFF9DE84\]`

0046B90D E8 C6C9F9FF `call NOTEPAD.004082D8`

0046B912 8945 C8 `mov dword ptr ss:\[ebp-38\],eax`

0046B915 33DB `xor ebx,ebx`

0046B917 66:83CB 02 `or bx,2`

0046B91B 8D85 D4DCF9FF `lea eax,dword ptr ss:\[ebp+FFF9DCD4\]`

0046B921 8D95 FFFCFFFF `lea edx,dword ptr ss:\[ebp-301\]`

0046B927 B9 91000000 `mov ecx,91`

0046B92C E8 D374F9FF `call NOTEPAD.00402E04`

0046B931 8D95 D4DCF9FF `lea edx,dword ptr ss:\[ebp+FFF9DCD4\]`



```
0046B937    8D85 40DCF9FF             lea eax,dword ptr ss:\[ebp+FFF9DC40\]

0046B93D    E8 FA71F9FF              call NOTEPAD.00402B3C

0046B942    BA 30DA4600              mov edx,NOTEPAD.0046DA30

0046B947    8D85 40DCF9FF            lea eax,dword ptr ss:\[ebp+FFF9DC40\]

0046B94D    B1 92                    mov cl,92

0046B94F    E8 B871F9FF              call NOTEPAD.00402B0C

0046B954    8D95 40DCF9FF            lea edx,dword ptr ss:\[ebp+FFF9DC40\]

0046B95A    8D85 84DEF9FF            lea eax,dword ptr ss:\[ebp+FFF9DE84\]

0046B960    E8 DB84F9FF              call NOTEPAD.00403E40

0046B965    8D85 84DEF9FF            lea eax,dword ptr ss:\[ebp+FFF9DE84\]

0046B96B    8B55 DC                  mov edx,dword ptr ss:\[ebp-24\]

0046B96E    E8 3185F9FF              call NOTEPAD.00403EA4

0046B973    8B85 84DEF9FF            mov eax,dword ptr ss:\[ebp+FFF9DE84\]

0046B979    0FB7D3                   movzx edx,bx

0046B97C    E8 1BCBF9FF              call NOTEPAD.0040849C

0046B981    8B45 C8                  mov eax,dword ptr ss:\[ebp-38\]

0046B984    E8 D7C9F9FF              call NOTEPAD.00408360

0046B989    6A 00                    push 0

0046B98B    6A 01                    push 1

0046B98D    A1 0CF09900              mov eax,dword ptr ds:\[99F00C\]

0046B992    50                       push eax

0046B993    E8 A8ACF9FF              call NOTEPAD.00406640                    ; jmp 到 kernel32.ReleaseSemaphore

0046B998    8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]

0046B99B    E8 88B4FFFF              call NOTEPAD.00466E28

0046B9A0    A1 A02C4700              mov eax,dword ptr ds:\[472CA0\]

0046B9A5    8B00                     mov eax,dword ptr ds:\[eax\]

0046B9A7    BA 4CD94600              mov edx,NOTEPAD.0046D94C                ; ASCII "REGISTERED"

0046B9AC    E8 FB85F9FF              call NOTEPAD.00403FAC

0046B9B1    75 0C                    jnz short NOTEPAD.0046B9BF

0046B9B3    C685 3FFFFFFF 01         mov byte ptr ss:\[ebp-C1\],1

0046B9BA    E9 50120000              jmp NOTEPAD.0046CC0F

0046B9BF    A1 A02C4700              mov eax,dword ptr ds:\[472CA0\]

0046B9C4    8B00                     mov eax,dword ptr ds:\[eax\]

0046B9C6    BA 4CD94600              mov edx,NOTEPAD.0046D94C                ; ASCII "REGISTERED"

0046B9CB    E8 DC85F9FF              call NOTEPAD.00403FAC

0046B9D0    74 19                    je short NOTEPAD.0046B9EB

0046B9D2    A1 442D4700              mov eax,dword ptr ds:\[472D44\]

0046B9D7    8B00                     mov eax,dword ptr ds:\[eax\]

0046B9D9    E8 1695FDFF              call NOTEPAD.00444EF4

0046B9DE    8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]

0046B9E1    E8 6662FDFF              call NOTEPAD.00441C4C

0046B9E6    E9 011B0000              jmp NOTEPAD.0046D4EC

0046B9EB    837D D4 00               cmp dword ptr ss:\[ebp-2C\],0

0046B9EF    0F84 420D0000            je NOTEPAD.0046C737

0046B9F5    A1 1C2B4700              mov eax,dword ptr ds:\[472B1C\]

0046B9FA    8038 01                  cmp byte ptr ds:\[eax\],1

0046B9FD    75 2C                    jnz short NOTEPAD.0046BA2B              ; 跳走咯

0046B9FF    8B0D 642C4700            mov ecx,dword ptr ds:\[472C64\]          ; NOTEPAD.00473888

0046BA05    8B09                     mov ecx,dword ptr ds:\[ecx\]

0046BA07    8B15 482C4700            mov edx,dword ptr ds:\[472C48\]          ; NOTEPAD.00473884

0046BA0D    8B12                     mov edx,dword ptr ds:\[edx\]

0046BA0F    8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]

0046BA12    E8 3D8FFFFF              call NOTEPAD.00464954

0046BA17    A1 442D4700              mov eax,dword ptr ds:\[472D44\]

0046BA1C    8B00                     mov eax,dword ptr ds:\[eax\]

0046BA1E    E8 D194FDFF              call NOTEPAD.00444EF4

0046BA23    8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]

0046BA26    E8 2162FDFF              call NOTEPAD.00441C4C

0046BA2B    A1 2FD49900              mov eax,dword ptr ds:\[99D42F\]

0046BA30    8945 BC                  mov dword ptr ss:\[ebp-44\],eax

0046BA33    A1 1FD09900              mov eax,dword ptr ds:\[99D01F\]

0046BA38    83E0 40                  and eax,40

0046BA3B    83F8 40                  cmp eax,40

0046BA3E    75 0B                    jnz short NOTEPAD.0046BA4B

0046BA40    8D55 94                  lea edx,dword ptr ss:\[ebp-6C\]

0046BA43    8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]

0046BA46    E8 898AFFFF              call NOTEPAD.004644D4

0046BA4B    A1 1FD09900              mov eax,dword ptr ds:\[99D01F\]

0046BA50    83E0 10                  and eax,10

0046BA53    83F8 10                  cmp eax,10

0046BA56    75 2E                    jnz short NOTEPAD.0046BA86

0046BA58    8D45 94                  lea eax,dword ptr ss:\[ebp-6C\]

0046BA5B    E8 508DFFFF              call NOTEPAD.004647B0

0046BA60    837D 94 00               cmp dword ptr ss:\[ebp-6C\],0

0046BA64    75 20                    jnz short NOTEPAD.0046BA86

0046BA66    E8 0D85FEFF              call NOTEPAD.00453F78

0046BA6B    8985 68DDF9FF            mov dword ptr ss:\[ebp+FFF9DD68\],eax

0046BA71    DB85 68DDF9FF            fild dword ptr ss:\[ebp+FFF9DD68\]

0046BA77    83C4 F4                  add esp,-0C

0046BA7A    DB3C24                   fstp tbyte ptr ss:\[esp\]

0046BA7D    9B                       wait

0046BA7E    8D45 94                  lea eax,dword ptr ss:\[ebp-6C\]

0046BA81    E8 C2D7F9FF              call NOTEPAD.00409248

0046BA86    A1 1FD09900              mov eax,dword ptr ds:\[99D01F\]

0046BA8B    83E0 01                  and eax,1

0046BA8E    48                       dec eax

0046BA8F    75 19                    jnz short NOTEPAD.0046BAAA

0046BA91    8D85 84DEF9FF            lea eax,dword ptr ss:\[ebp+FFF9DE84\]

0046BA97    E8 1884FEFF              call NOTEPAD.00453EB4

0046BA9C    8B95 84DEF9FF            mov edx,dword ptr ss:\[ebp+FFF9DE84\]

0046BAA2    8D45 94                  lea eax,dword ptr ss:\[ebp-6C\]

0046BAA5    E8 FA83F9FF              call NOTEPAD.00403EA4

0046BAAA    A1 1FD09900              mov eax,dword ptr ds:\[99D01F\]

0046BAAF    83E0 02                  and eax,2

0046BAB2    83F8 02                  cmp eax,2

0046BAB5    75 31                    jnz short NOTEPAD.0046BAE8

0046BAB7    E8 BC84FEFF              call NOTEPAD.00453F78

0046BABC    8985 68DDF9FF            mov dword ptr ss:\[ebp+FFF9DD68\],eax

0046BAC2    DB85 68DDF9FF            fild dword ptr ss:\[ebp+FFF9DD68\]

0046BAC8    83C4 F4                  add esp,-0C

0046BACB    DB3C24                   fstp tbyte ptr ss:\[esp\]

0046BACE    9B                       wait

0046BACF    8D85 84DEF9FF            lea eax,dword ptr ss:\[ebp+FFF9DE84\]

0046BAD5    E8 6ED7F9FF              call NOTEPAD.00409248

0046BADA    8B95 84DEF9FF            mov edx,dword ptr ss:\[ebp+FFF9DE84\]

0046BAE0    8D45 94                  lea eax,dword ptr ss:\[ebp-6C\]

0046BAE3    E8 BC83F9FF              call NOTEPAD.00403EA4

0046BAE8    A1 1FD09900              mov eax,dword ptr ds:\[99D01F\]

0046BAED    83E0 04                  and eax,4

0046BAF0    83F8 04                  cmp eax,4

0046BAF3    75 19                    jnz short NOTEPAD.0046BB0E

0046BAF5    8D85 84DEF9FF            lea eax,dword ptr ss:\[ebp+FFF9DE84\]

0046BAFB    E8 7883FEFF              call NOTEPAD.00453E78

0046BB00    8B95 84DEF9FF            mov edx,dword ptr ss:\[ebp+FFF9DE84\]

0046BB06    8D45 94                  lea eax,dword ptr ss:\[ebp-6C\]

0046BB09    E8 9683F9FF              call NOTEPAD.00403EA4

0046BB0E    A1 1FD09900              mov eax,dword ptr ds:\[99D01F\]

0046BB13    83E0 08                  and eax,8

0046BB16    83F8 08                  cmp eax,8
```



0046BB19  75 7D                  jnz short NOTEPAD.0046BB98  

0046BB1B  E8 748BFFFF           call NOTEPAD.00464694  

0046BB20  83F8 01               cmp eax,1  

0046BB23  75 73                 jnz short NOTEPAD.0046BB98  

0046BB25  8B45 FC               mov eax,dword ptr ss:\[ebp-4\]  

0046BB28  8B80 D4020000        mov eax,dword ptr ds:\[eax+2D4\]  

0046BB2E  E8 A1DDFEFF          call NOTEPAD.004598D4  

0046BB33  E8 20B8FEFF          call NOTEPAD.00457358  

0046BB38  83E0 7F               and eax,7F  

0046BB3B  8D95 84DEF9FF         lea edx,dword ptr ss:\[ebp+FFF9DE84\]  

0046BB41  E8 22C6F9FF          call NOTEPAD.00408168  

0046BB46  FFB5 84DEF9FF        push dword ptr ss:\[ebp+FFF9DE84\]  

0046BB4C  8B45 FC               mov eax,dword ptr ss:\[ebp-4\]  

0046BB4F  8B80 D4020000        mov eax,dword ptr ds:\[eax+2D4\]  

0046BB55  FF70 30               push dword ptr ds:\[eax+30\]  

0046BB58  8B45 FC               mov eax,dword ptr ss:\[ebp-4\]  

0046BB5B  8B80 D4020000        mov eax,dword ptr ds:\[eax+2D4\]  

0046BB61  E8 6EDDFEFF          call NOTEPAD.004598D4  

0046BB66  FF70 0C               push dword ptr ds:\[eax+C\]  

0046BB69  8B45 FC               mov eax,dword ptr ss:\[ebp-4\]  

0046BB6C  8B80 D4020000        mov eax,dword ptr ds:\[eax+2D4\]  

0046BB72  E8 5DDDFEFF          call NOTEPAD.004598D4  

0046BB77  FF70 08               push dword ptr ds:\[eax+8\]  

0046BB7A  8D85 54FFFFFF         lea eax,dword ptr ss:\[ebp-AC\]  

0046BB80  BA 04000000           mov edx,4  

0046BB85  E8 D283F9FF          call NOTEPAD.00403F5C  

0046BB8A  8D45 94               lea eax,dword ptr ss:\[ebp-6C\]  

0046BB8D  8B95 54FFFFFF        mov edx,dword ptr ss:\[ebp-AC\]  

0046BB93  E8 0C83F9FF          call NOTEPAD.00403EA4  

0046BB98  A1 1FD09900           mov eax,dword ptr ds:\[99D01F\]  

0046BB9D  83E0 20               and eax,20  

0046BBA0  83F8 20               cmp eax,20  

0046BBA3  75 31                 jnz short NOTEPAD.0046BBD6  

0046BBA5  E8 CE8AFFFF          call NOTEPAD.00464678  

0046BBAA  8985 68DDF9FF        mov dword ptr ss:\[ebp+FFF9DD68\],eax  

0046BBB0  DB85 68DDF9FF        fild dword ptr ss:\[ebp+FFF9DD68\]  

0046BBB6  83C4 F4               add esp,-0C  

0046BBB9  DB3C24               fstp tbyte ptr ss:\[esp\]  

0046BBBC  9B                    wait  

0046BBBD  8D85 84DEF9FF         lea eax,dword ptr ss:\[ebp+FFF9DE84\]  

0046BBC3  E8 80D6F9FF          call NOTEPAD.00409248  

0046BBC8  8B95 84DEF9FF        mov edx,dword ptr ss:\[ebp+FFF9DE84\]  

0046BBCE  8D45 94               lea eax,dword ptr ss:\[ebp-6C\]  

0046BBD1  E8 CE82F9FF          call NOTEPAD.00403EA4  

0046BBD6  8D85 84DEF9FF         lea eax,dword ptr ss:\[ebp+FFF9DE84\]     ; 文件加密设置信息：Name of Product  

0046BBDC  BA 8DCA9900           mov edx,NOTEPAD.0099CA8D                ; ASCII "/Clone ExeShield Protector License Generator Example"  

0046BBE1  E8 5A82F9FF          call NOTEPAD.00403E40  

0046BBE6  8B95 84DEF9FF        mov edx,dword ptr ss:\[ebp+FFF9DE84\]  

0046BBEC  A1 442D4700           mov eax,dword ptr ds:\[472D44\]  

0046BBF1  8B00                  mov eax,dword ptr ds:\[eax\]  

0046BBF3  E8 D48DFDFF          call NOTEPAD.004449CC  

0046BBF8  8D45 DC               lea eax,dword ptr ss:\[ebp-24\]  

0046BBFB  50                    push eax  

0046BBFC  FF75 94               push dword ptr ss:\[ebp-6C\]  

0046BBFF  8D85 6CDEF9FF         lea eax,dword ptr ss:\[ebp+FFF9DE6C\]  

0046BC05  BA 6FEC9900           mov edx,NOTEPAD.0099EC6F  

0046BC0A  E8 3182F9FF          call NOTEPAD.00403E40  

0046BC0F  FFB5 6CDEF9FF        push dword ptr ss:\[ebp+FFF9DE6C\]  

0046BC15  A1 7C2E4700           mov eax,dword ptr ds:\[472E7C\]  

0046BC1A  FF30                  push dword ptr ds:\[eax\]  

0046BC1C  8D85 84DEF9FF         lea eax,dword ptr ss:\[ebp+FFF9DE84\]  

0046BC22  BA 03000000           mov edx,3  

0046BC27  E8 3083F9FF          call NOTEPAD.00403F5C                  ; 本机授权文件名称组合  

0046BC2C  8B85 84DEF9FF        mov eax,dword ptr ss:\[ebp+FFF9DE84\]   ; ASCII "A512-A7C1NOTEPAD.LIC"  

0046BC32  8D95 74DEF9FF         lea edx,dword ptr ss:\[ebp+FFF9DE74\]  

0046BC38  E8 BB75FEFF          call NOTEPAD.004531F8  

0046BC3D  8D8D 74DEF9FF         lea ecx,dword ptr ss:\[ebp+FFF9DE74\]  

0046BC43  BA 98D94600           mov edx,NOTEPAD.0046D998                ; ASCII "100293882"  

0046BC48  B8 D4D54600           mov eax,NOTEPAD.0046D5D4                ; ASCII "MD5String"  

0046BC4D  E8 3A8CFFFF          call NOTEPAD.0046488C                  ; 文件校验  

0046BC52  8B55 DC               mov edx,dword ptr ss:\[ebp-24\]        ; ASCII "MD5String('100293882') =66d5daab98ac0a88e9cbe680486d3e75"  

0046BC55  B8 E8D54600           mov eax,NOTEPAD.0046D5E8  

0046BC5A  E8 2585F9FF          call NOTEPAD.00404184  

0046BC5F  8BF0                  mov esi,eax  

0046BC61  8D45 DC               lea eax,dword ptr ss:\[ebp-24\]  

0046BC64  8BCE                  mov ecx,esi  

0046BC66  BA 01000000           mov edx,1  

0046BC6B  E8 7084F9FF          call NOTEPAD.004040E0  

0046BC70  8B45 DC               mov eax,dword ptr ss:\[ebp-24\]  

0046BC73  E8 2482F9FF          call NOTEPAD.00403E9C  

0046BC78  8BC8                  mov ecx,eax  

0046BC7A  83E9 08               sub ecx,8  

0046BC7D  8D45 DC               lea eax,dword ptr ss:\[ebp-24\]  

0046BC80  BA 08000000           mov edx,8  

0046BC85  E8 5684F9FF          call NOTEPAD.004040E0  

0046BC8A  8D55 DC               lea edx,dword ptr ss:\[ebp-24\]  

0046BC8D  B9 05000000           mov ecx,5  

0046BC92  B8 ACD94600           mov eax,NOTEPAD.0046D9AC  

0046BC97  E8 8C84F9FF          call NOTEPAD.00404128  

0046BC9C  8D95 84DEF9FF         lea edx,dword ptr ss:\[ebp+FFF9DE84\]  

0046BCA2  8B45 94               mov eax,dword ptr ss:\[ebp-6C\]  

0046BCA5  E8 0AC4F9FF          call NOTEPAD.004080B4  

0046BCAA  83BD 84DEF9FF 00     cmp dword ptr ss:\[ebp+FFF9DE84\],0     ; 机器码是否为 0  

0046BCB1  75 0D                 jnz short NOTEPAD.0046BCC0             ; 不为 0 就跳  

0046BCB3  8D45 DC               lea eax,dword ptr ss:\[ebp-24\]  

0046BCB6  BA B8D94600           mov edx,NOTEPAD.0046D9B8                ; ASCII "A512-A7C1"  

0046BCBB  E8 F87FF9FF          call NOTEPAD.00403CB8  

0046BCC0  8D55 94               lea edx,dword ptr ss:\[ebp-6C\]  

0046BCC3  8B45 DC               mov eax,dword ptr ss:\[ebp-24\]  

0046BCC6  E8 71C2F9FF          call NOTEPAD.00407F3C                  ; 是否为黑名单的机器码  

0046BCCB  803D 57CB9900 00     cmp byte ptr ds:\[99CB57\],0  

0046BCD2  74 2B                 je short NOTEPAD.0046BCFF             ; 不是就继续计算，跳！  

0046BCD4  FF75 94               push dword ptr ss:\[ebp-6C\]  

0046BCD7  68 ACD94600           push NOTEPAD.0046D9AC  

0046BCDC  8D85 84DEF9FF         lea eax,dword ptr ss:\[ebp+FFF9DE84\]  

0046BCE2  BA 57CB9900           mov edx,NOTEPAD.0099CB57  

0046BCE7  E8 5481F9FF          call NOTEPAD.00403E40  

0046BCEC  FFB5 84DEF9FF        push dword ptr ss:\[ebp+FFF9DE84\]  

0046BCF2  8D45 94               lea eax,dword ptr ss:\[ebp-6C\]  

0046BCF5  BA 03000000           mov edx,3  

0046BCFA  E8 5D82F9FF          call NOTEPAD.00403F5C  

0046BCFF  8D45 90               lea eax,dword ptr ss:\[ebp-70\]        ; ▲文章的关键【Encryption Key for this License】  

0046BD02  8B55 BC               mov edx,dword ptr ss:\[ebp-44\]  



`shl edx,5`

`lea edx,dword ptr ds:[edx*8+99D35B]` ; 出现并调用第一组【License 加密特征码】

`call NOTEPAD.00403E40` ; ▲出现 License 加密特征码后，注意查看数据窗中数据

........

---【数据窗中数据】---

`0099D44B` `00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00` `................`

`0099D45B` `FA 79 6B 33 4D 53 72 79 33 6D 73 63 48 45 6A 23` `k3MSry3mscHEj#`

`0099D46B` `57 6F 6B 36 51 63 39 31 44 36 26 61 77 76 21 35` `Wok6Qc91D6&awv!5`

`0099D47B` `78 38 58 53 2A 34 54 64 39 78 4E 44 42 34 51 6D` `x8XS*4Td9xNDB4Qm`

`0099D48B` `35 77 4D 71 63 53 70 4D 47 25 37 58 74 72 38 30` `5wMqcSpMG%7Xtr80`

`0099D49B` `6F 32 6A 72 77 44 64 31 49 38 52 31 48 63 38 77` `o2jrwDd1I8R1Hc8w`

`0099D4AB` `79 57 25 2A 33 21 4B 21 56 67 5A 48 56 6A 4A 7A` `yW%*3!K!VgZHVjJz`

`0099D4BB` `45 66 55 46 31 58 23 2A 58 79 32 4F 48 73 58 54` `EfUF1X#*Xy2OHsXT`

`0099D4CB` `45 53 5A 6A 54 35 6D 38 33 77 6C 66 71 43 75 31` `ESZjT5m83wlfqCu1`

`0099D4DB` `4F 46 2A 24 43 21 65 53 63 79 50 74 78 75 5E 6D` `OF*$C!eScyPtxu^m`

`0099D4EB` `67 70 5E 79 4F 69 6E 35 7A 74 35 73 64 61 6F 6F` `gp^yOin5zt5sdaoo`

`0099D4FB` `78 44 40 6F 37 25 61 67 61 26 71 25 65 49 55 46` `xD@o7%aga&q%eIUF`

`0099D50B` `4E 6A 56 56 55 5A 31 5A 6D 46 50 6D 68 49 56 50` `NjVVUZ1ZmFPmhIVP`

`0099D51B` `23 24 59 67 33 75 23 62 66 57 65 6A 32 46 4A 34` `#$Yg3u#bfWej2FJ4`

`0099D52B` `4C 67 5E 42 73 38 64 5E 7A 61 52 74 65 58 6D 26` `Lg^Bs8d^zaRteXm&`

`0099D53B` `37 6C 74 48 23 4B 41 73 25 55 79 55 41 47 6C 23` `7ltH#KAs%UyUAGl#`

`0099D54B` `24 48 6B 75 7A 52 73 38 25 78 5E 00 00 00 00 00` `$HkuzRs8%x^.....`

`0099D55B` `FA 57 74 77 23 49 41 79 23 40 77 51 68 35 4C 63` `tw#IAy#@wQh5Lc`

`0099D56B` `24 30 44 25 63 5E 7A 38 41 53 48 4F 31 53 72 49` `$0D%c^z8ASHO1SrI`

`0099D57B` `78 4C 74 37 68 4C 40 6B 75 2A 67 64 52 72 67 49` `xLt7hL@ku*gdRrgI`

`0099D58B` `44 4C 74 68 52 4C 56 45 53 43 59 66 42 4F 5A 75` `DLthRLVESCYfBOZu`

`0099D59B` `6A 4E 4B 67 61 34 30 65 6D 43 26 26 69 6E 5A 32` `jNKga40emC&&inZ2`

`0099D5AB` `68 75 6F 49 21 43 48 6A 6D 46 71 26 6E 51 53 62` `huoI!CHjmFq&nQSb`

`0099D5BB` `47 79 4D 71 4E 74 69 4C 69 57 43 30 62 63 4A 73` `GyMqNtiLiWC0bcJs`

`0099D5CB` `4D 26 58 4C 2A 30 41 6A 32 79 23 32 65 6E 76 25` `M&XL*0Aj2y#2env%`

`0099D5DB` `4C 4D 4B 52 50 4E 67 46 38 79 59 6C 5A 52 79 52` `LMKRPNgF8yYlZRyR`

`0099D5EB` `47 38 33 34 41 38 42 44 55 21 4D 4D 26 78 72 4F` `G834A8BDU!MM&xrO`

`0099D5FB` `66 43 21 58 6E 78 72 57 6E 76 6B 36 4C 6E 77 58` `fC!XnxrWnvk6LnwX`

`0099D60B` `4F 41 4B 4F 4C 30 58 38 46 44 4D 45 34 6B 78 52` `OAKOL0X8FDME4kxR`

`0099D61B` `77 30 71 32 44 59 25 77 45 4F 43 44 7A 47 52 36` `w0q2DY%wEOCDzGR6`

`0099D62B` `6C 58 49 6A 65 31 53 57 6D 53 70 7A 79 4E 50 5A` `lXIje1SWmSpzyNPZ`

`0099D63B` `72 58 30 5A 57 43 70 78 54 24 55 41 58 24 52 36` `rX0ZWCpxT$UAX$R6`

`0099D64B` `45 31 7A 33 6E 48 50 70 75 4E 42 00 00 00 00 00` `E1z3nHPpuNB.....`

`0099D65B` `00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00` `................`

-----------------------------------------------------------------------------

在 `0046BD08` 处，我们可以发现程序调用 `Encryption Key for this License` 加密模块数据时，是以二进制代码“`FA`”作为加密特征码调用开始的特征，然后再使用该代码后的密钥作为授权类型的 License 加密特征码，调用完毕后，再以“`00`”作为加密特征码调用的结束特征。

【整理后得到】

第一组 License 加密特征码：（`0099D45C` -- `0099D555`）

```
yk3MSry3mscHEj#Wok6Qc91D6&awv!5x8XS*4Td9xNDB4Qm5wMqcSpMG%7Xtr80o2jrwDd1I8R1Hc8wyW%*3!K!VgZHVjJzEfUF1X#*Xy2OHsXTESZjT5m83wlfqCu1OF*$C!eScyPtxu^mgp^yOin5zt5sdaooxD@o7%aga&q%eIUFNjVVUZ1ZmFPmhIVP#$Yg3u#bfWej2FJ4Lg^Bs8d^zaRteXm&7ltH#KAs%UyUAGl#$HkuzRs8%x^
```

第二组 License 加密特征码：（`0099D55C` -- `0099D655`）

```
Wtw#IAy#@wQh5Lc$0D%c^z8ASHO1SrIxLt7hL@ku*gdRrgIDLthRLVESCYfBOZujNKga40emC&&inZ2huoI!CHjmFq&nQSbGyMqNtiLiWC0bcJsM&XL*0Aj2y#2env%LMKRPNgF8yYlZRyRG834A8BDU!MM&xrOfC!XnxrWnvk6LnwXOAKOL0X8FDME4kxRw0q2DY%wEOCDzGR6lXIje1SWmSpzyNPZrX0ZWCpxT$UAX$R6E1z3nHPpuNB
```

又因为以上分析到的环节是以 `Trial License` 作为授权类型，那么我们可以大胆的得出结论：

1. `Trial License` = 第一组 License 加密特征码
2. `Registered License` = 第二组 License 加密特征码

到此，下面的代码我们就可以不必再分析下去了。

【Clone ExeShield Protector License Generator】

★注意：我们为了保证 License 合法性，所使用的 ExeShield Protector 版本最好是大于等于 `v3.8.1.7` 的。

（这里我所使用的是官方最新的 `3.9.6.7`）

1. 随便找一个程序，我这里所使用的为 Windows 自带的计算器；
2. 新建一个工程，加密的目标文件设置为该计算器，其他选项均为默认值亦可；
3. 在 License 选项一栏中，创建 2 个授权方式（`Create License`），一个为 "Trial License"，另一个则为 "Registered License"；
4. 分别依次在这 2 个授权方式的 `Encryption Key for this License` 栏中填上我们所获得的 License 加密特征码，然后点击 “`Update License`”
   ★注意！ "Registered License" 模式中，只选择 `Application never expires` ★
5. 保存工程文件（作为 `License Generator`），不需要真正对目标文件加密亦可；

`License Generator` 制作完毕！

★注意！制作出来的授权文件名要与目标文件名对应★

试试看用我们 Clone 出来的 `License Generator` 能否授权原目标文件 :)

--------------------------------------------------------------------------------

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `Clone ExeShield Protector License Generator.rar` [附件请到论坛下载：`bbs.pediy.com/showthread.php?t=57801`](http://bbs.pediy.com/showthread.php?t=57801) |
| :--- | :--- | :--- |
