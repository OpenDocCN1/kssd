
# 矢量图形编辑控件

> 原文：[`www.pediy.com/kssd/pediy06/pediy6309.htm`](https://www.pediy.com/kssd/pediy06/pediy6309.htm)

非常好的矢量图形编辑控件。

以前是哪个兄弟提出过这个软件的。

在 `http://business.hol.gr/gardos/` 下载 `avaxsetup.exe`。

这个东西是一个插件，可是英文太差，看得比较胡涂，好在程序自己带了几个例子，这些例子也可以验证我们注册成功与否。

这个控件自己带一个注册用的程序，我们拿他开刀，`VB6`，无壳。

这个程序耗用我的时间最多，和我一样的新手可以拿他来锻炼耐心了。

### 1、序列号长度为 `1D` (29 位)

### 2、29 位注册码的形式如下：`xxxx-xxxx-xxxx-xxxx-xxxx-xxxxx`

### 3、开始用 `OD` 跟踪程序，有好多地方很关键，但是限于水平、精力、时间……（当然主要还是水平）只能一略而过，有兴趣的可以自己跟踪。经过反复跟踪终于得到注册码，以下是我的试验过程：

---

`1111-3333-1234-6780-8211-2222`    开始的试验码(各个部分力求不同)

`1111-3333-1234-6780-8213-2222`    第 24 位和第 7 位必须相等

```text
    |____________________|
```

`1111-3333-1234-6780-8013-2222`    第 19 位和第 22 位必须相等

```text
                    |__|
```

`1111-5318-1234-6780-8013-2222`    第 67 两位的和(或第 9 位)和第 21 位的值有一个关系(关键运算二没有仔细看,请自己分析)

```text
    ||                |
    和
```

`1221-5318-1234-2780-8023-2112`

```text
                  |_____|          对应位相等
|||| |||| ||||            ||||
```

`A992-748D-BF21-2780-6024-20D7` --->跟踪到 `748D`、`BF21`、`A992`、`20D7` 四部分，对其他部分相应更改，剩余两部分必须是数字

```text
                                ||||
```

`A992-748D-BF21-2780-6024-1253`---->更改后再跟踪找到最末位的正确值 `1253`

```text
||||
```

`14A9-748D-BF21-2780-6024-1253`---->再更改后再跟踪找到开头 4 位的正确值 `14A9`

最终成功(已经过了无数天……)

注意第五部分的 `6024`，`6` 这个数字也有运算

---

`/公司: dnpf (经过我的试验,这个东西好象不参与运算)`

`注册码: 14A9-748D-BF21-2780-6024-1253`

---

大家可以试验用其他数字组合，多找几组注册码

---

`* Referenced by a CALL at Address:`

```asm
:0040874B
|
:0040A830 55            push  ebp
:0040A831 8BEC          mov  ebp, esp
:0040A833 83EC14        sub  esp, 00000014
:0040A836 6806154000    push 00401506
:0040A83B 64A100000000  mov  eax, dword ptr fs:[00000000]
:0040A841 50            push  eax
:0040A842 64892500000000  mov  dword ptr fs:[00000000], esp
:0040A849 81ECC8000000  sub  esp, 000000C8
:0040A84F 53            push  ebx
:0040A850 56            push  esi
:0040A851 57            push  edi
:0040A852 8965EC        mov  dword ptr [ebp-14], esp
:0040A855 C745F0B8134000  mov  [ebp-10], 004013B8
:0040A85C 33C0          xor  eax, eax
:0040A85E 8945F4        mov  dword ptr [ebp-0C], eax
:0040A861 8945F8        mov  dword ptr [ebp-08], eax
:0040A864 8945E0        mov  dword ptr [ebp-20], eax
:0040A867 8945DC        mov  dword ptr [ebp-24], eax
:0040A86A 8945D8        mov  dword ptr [ebp-28], eax
:0040A86D 8945CC        mov  dword ptr [ebp-34], eax
:0040A870 8945C8        mov  dword ptr [ebp-38], eax
:0040A873 8945C4        mov  dword ptr [ebp-3C], eax
:0040A876 8945C0        mov  dword ptr [ebp-40], eax
:0040A879 8945BC        mov  dword ptr [ebp-44], eax
:0040A87C 8945B8        mov  dword ptr [ebp-48], eax
:0040A87F 8945A8        mov  dword ptr [ebp-58], eax
:0040A882 894598        mov  dword ptr [ebp-68], eax
:0040A885 894588        mov  dword ptr [ebp-78], eax
:0040A888 898578FFFFFF  mov  dword ptr [ebp+FFFFFF78], eax
:0040A88E 898568FFFFFF  mov  dword ptr [ebp+FFFFFF68], eax
:0040A894 898544FFFFFF  mov  dword ptr [ebp+FFFFFF44], eax
:0040A89A 898540FFFFFF  mov  dword ptr [ebp+FFFFFF40], eax
:0040A8A0 89853CFFFFFF  mov  dword ptr [ebp+FFFFFF3C], eax
:0040A8A6 898534FFFFFF  mov  dword ptr [ebp+FFFFFF34], eax
:0040A8AC 898530FFFFFF  mov  dword ptr [ebp+FFFFFF30], eax
:0040A8B2 6A01          push  00000001
* Reference To: MSVBVM60.__vbaOnError, Ord:0000h
                  |
:0040A8B4 FF15B0104000  Call dword ptr [004010B0]
* Possible StringData Ref from Code Obj ->"Gardos Software"
                  |
:0040A8BA BA24344000    mov  edx, 00403424
:0040A8BF 8D4DC8        lea  ecx, dword ptr [ebp-38]
* Reference To: MSVBVM60.__vbaStrCopy, Ord:0000h
                  |
:0040A8C2 8B3504124000  mov  esi, dword ptr [00401204]
:0040A8C8 FFD6          call  esi
* Possible StringData Ref from Code Obj ->"RegAVAX"
                  |
:0040A8CA BA90364000    mov  edx, 00403690
:0040A8CF 8D4DC4        lea  ecx, dword ptr [ebp-3C]
:0040A8D2 FFD6          call  esi
:0040A8D4 68F41E4000    push  00401EF4
* Reference To: MSVBVM60.__vbaNew, Ord:0000h
                  |
:0040A8D9 FF1550114000  Call dword ptr [00401150]
:0040A8DF 50            push  eax
:0040A8E0 8D45E0        lea  eax, dword ptr [ebp-20]
:0040A8E3 50            push  eax
* Reference To: MSVBVM60.__vbaObjSet, Ord:0000h
                  |
:0040A8E4 FF15B4104000  Call dword ptr [004010B4]
:0040A8EA 8B4DC8        mov  ecx, dword ptr [ebp-38]
:0040A8ED 51            push  ecx
* Reference To: MSVBVM60.rtcTrimBstr, Ord:0207h
                  |
:0040A8EE 8B1D6C104000  mov  ebx, dword ptr [0040106C]
:0040A8F4 FFD3          call  ebx
:0040A8F6 8BD0          mov  edx, eax
:0040A8F8 8D4DC0        lea  ecx, dword ptr [ebp-40]
* Reference To: MSVBVM60.__vbaStrMove, Ord:0000h
                  |
:0040A8FB 8B3D78124000  mov  edi, dword ptr [00401278]
:0040A901 FFD7          call  edi
:0040A903 50            push  eax
:0040A904 68D8374000    push  004037D8
* Reference To: MSVBVM60.__vbaStrCmp, Ord:0000h
                  |
:0040A909 FF1510114000  Call dword ptr [00401110]
:0040A90F 8BF0          mov  esi, eax
:0040A911 F7DE          neg  esi
:0040A913 1BF6          sbb  esi, esi
:0040A915 46            inc  esi
:0040A916 F7DE          neg  esi
:0040A918 8B550C        mov  edx, dword ptr [ebp+0C]
:0040A91B 8B02          mov  eax, dword ptr [edx]----------->输入的公司名
:0040A91D 50            push  eax
:0040A91E FFD3          call  ebx
:0040A920 8BD0          mov  edx, eax
:0040A922 8D4DB8        lea  ecx, dword ptr [ebp-48]
:0040A925 FFD7          call  edi
:0040A927 50            push  eax
:0040A928 68D8374000    push  004037D8
* Reference To: MSVBVM60.__vbaStrCmp, Ord:0000h
                  |
:0040A92D FF1510114000  Call dword ptr [00401110]
:0040A933 F7D8          neg  eax
:0040A935 1BC0          sbb  eax, eax
:0040A937 40            inc  eax
:0040A938 F7D8          neg  eax
:0040A93A 0BF0          or   esi, eax
:0040A93C 8B4D10        mov  ecx, dword ptr [ebp+10]
:0040A93F 8B11          mov  edx, dword ptr [ecx]---------------->输入的序列号
:0040A941 52            push  edx
:0040A942 FFD3          call  ebx
:0040A944 8BD0          mov  edx, eax
:0040A946 8D4DBC        lea  ecx, dword ptr [ebp-44]
:0040A949 FFD7          call  edi
:0040A94B 50            push  eax
```



```
:0040A94C 68D8374000                     push 004037D8   

\* Reference To: MSVBVM60.__vbaStrCmp, Ord:0000h   

                            |   

:0040A951 FF1510114000             Call dword ptr [00401110]   

:0040A957 F7D8                     neg eax   

:0040A959 1BC0                     sbb eax, eax   

:0040A95B 40                       inc eax   

:0040A95C F7D8                     neg eax   

:0040A95E 0BF0                     or esi, eax   

:0040A960 8D45B8                   lea eax, dword ptr [ebp-48]   

:0040A963 50                       push eax   

:0040A964 8D4DBC                   lea ecx, dword ptr [ebp-44]   

:0040A967 51                       push ecx   

:0040A968 8D55C0                   lea edx, dword ptr [ebp-40]   

:0040A96B 52                       push edx   

:0040A96C 6A03                     push 00000003   

\* Reference To: MSVBVM60.__vbaFreeStrList, Ord:0000h   

                            |   

:0040A96E FF150C124000             Call dword ptr [0040120C]   

:0040A974 83C410                   add esp, 00000010   

:0040A977 6685F6                   test si, si\--------------------   

:0040A97A 7435                     je 0040A9B1--------------------/SI 为零跳到下面   

:0040A97C B904000280               mov ecx, 80020004   

:0040A981 894D90                   mov dword ptr [ebp-70], ecx   

:0040A984 B80A000000               mov eax, 0000000A   

:0040A989 894588                   mov dword ptr [ebp-78], eax   

:0040A98C 894DA0                   mov dword ptr [ebp-60], ecx   

:0040A98F 894598                   mov dword ptr [ebp-68], eax   

:0040A992 8D45C4                   lea eax, dword ptr [ebp-3C]   

:0040A995 898570FFFFFF             mov dword ptr [ebp+FFFFFF70], eax   

:0040A99B C78568FFFFFF08400000     mov dword ptr [ebp+FFFFFF68], 00004008   

\* Possible StringData Ref from Code Obj ->"Invalid Data - Error 1"   

                            |   

:0040A9A5 C7458090394000           mov [ebp-80], 00403990   

:0040A9AC E91C030000               jmp 0040ACCD   

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0040A97A(C)   

|   

:0040A9B1 8B5D10                   mov ebx, dword ptr [ebp+10]------------->输入的序列号   

:0040A9B4 8B13                     mov edx, dword ptr [ebx]---------------->输入的序列号   

:0040A9B6 52                       push edx   

:0040A9B7 E8F4EBFFFF               call 004095B0--------------------------->关键一   

:0040A9BC 6685C0                   test ax, ax\----------------------------->AX 作为标志   

:0040A9BF 7535                     jne 0040A9F6---------------------------->不为零则跳走   

:0040A9C1 B904000280               mov ecx, 80020004   

:0040A9C6 894D90                   mov dword ptr [ebp-70], ecx   

:0040A9C9 B80A000000               mov eax, 0000000A   

:0040A9CE 894588                   mov dword ptr [ebp-78], eax   

:0040A9D1 894DA0                   mov dword ptr [ebp-60], ecx   

:0040A9D4 894598                   mov dword ptr [ebp-68], eax   

:0040A9D7 8D45C4                   lea eax, dword ptr [ebp-3C]   

:0040A9DA 898570FFFFFF             mov dword ptr [ebp+FFFFFF70], eax   

:0040A9E0 C78568FFFFFF08400000     mov dword ptr [ebp+FFFFFF68], 00004008   

\* Possible StringData Ref from Code Obj ->"Invalid Serial Number - Error "   

                                 ->"2"   

                            |   

:0040A9EA C74580C4394000           mov [ebp-80], 004039C4   

:0040A9F1 E9D7020000               jmp 0040ACCD   

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0040A9BF(C)   

|   

:0040A9F6 8B7508                   mov esi, dword ptr [ebp+08]   

:0040A9F9 56                       push esi\----------------------------->入栈保存   

:0040A9FA E871050000               call 0040AF70------------------------>   

:0040A9FF 6685C0                   test ax, ax\-------------------------->AX 作为标志   

:0040AA02 7578                     jne 0040AA7C------------------------->不为零则跳   

:0040AA04 B904000280               mov ecx, 80020004   

:0040AA09 894D90                   mov dword ptr [ebp-70], ecx   

:0040AA0C B80A000000               mov eax, 0000000A   

:0040AA11 894588                   mov dword ptr [ebp-78], eax   

:0040AA14 894DA0                   mov dword ptr [ebp-60], ecx   

:0040AA17 894598                   mov dword ptr [ebp-68], eax   

:0040AA1A 8D55C4                   lea edx, dword ptr [ebp-3C]   

:0040AA1D 895580                   mov dword ptr [ebp-80], edx   

:0040AA20 C78578FFFFFF08400000     mov dword ptr [ebp+FFFFFF78], 00004008   

:0040AA2A 8B06                     mov eax, dword ptr [esi]   

:0040AA2C 50                       push eax   

:0040AA2D 68083A4000               push 00403A08   

\* Reference To: MSVBVM60.__vbaStrCat, Ord:0000h   

                            |   

:0040AA32 8B3574104000             mov esi, dword ptr [00401074]   

:0040AA38 FFD6                     call esi   

:0040AA3A 8BD0                     mov edx, eax   

:0040AA3C 8D4DC0                   lea ecx, dword ptr [ebp-40]   

:0040AA3F FFD7                     call edi   

:0040AA41 50                       push eax   

\* Possible StringData Ref from Code Obj ->" File not found - Error 3"   

                            |   

:0040AA42 68143A4000               push 00403A14   

:0040AA47 FFD6                     call esi   

:0040AA49 8945B0                   mov dword ptr [ebp-50], eax   

:0040AA4C C745A808000000           mov [ebp-58], 00000008   

:0040AA53 8D4D88                   lea ecx, dword ptr [ebp-78]   

:0040AA56 51                       push ecx   

:0040AA57 8D5598                   lea edx, dword ptr [ebp-68]   

:0040AA5A 52                       push edx   

:0040AA5B 8D8578FFFFFF             lea eax, dword ptr [ebp+FFFFFF78]   

:0040AA61 50                       push eax   

:0040AA62 6A10                     push 00000010   

:0040AA64 8D4DA8                   lea ecx, dword ptr [ebp-58]   

:0040AA67 51                       push ecx   

\* Reference To: MSVBVM60.rtcMsgBox, Ord:0253h   

                            |   

:0040AA68 FF15B8104000             Call dword ptr [004010B8]   

:0040AA6E 8D4DC0                   lea ecx, dword ptr [ebp-40]   

\* Reference To: MSVBVM60.__vbaFreeStr, Ord:0000h   

                            |   

:0040AA71 FF15AC124000             Call dword ptr [004012AC]   

:0040AA77 E985020000               jmp 0040AD01   

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0040AA02(C)   

|   

:0040AA7C 8D55D8                   lea edx, dword ptr [ebp-28]   

:0040AA7F 52                       push edx\---------------------------------   

:0040AA80 8D45DC                   lea eax, dword ptr [ebp-24]   

:0040AA83 50                       push eax\----------------------------------|保存三个参数   

:0040AA84 8B0B                     mov ecx, dword ptr [ebx]   

:0040AA86 51                       push ecx\----------------------------------/   

:0040AA87 E834F5FFFF               call 00409FC0----------------------------->   

:0040AA8C 6685C0                   test ax, ax\------------------------------->AX 作为标志   

:0040AA8F 7575                     jne 0040AB06------------------------------>不为零则跳(否则出错)   

:0040AA91 B904000280               mov ecx, 80020004   

:0040AA96 894D90                   mov dword ptr [ebp-70], ecx   

:0040AA99 B80A000000               mov eax, 0000000A   

:0040AA9E 894588                   mov dword ptr [ebp-78], eax   

:0040AAA1 894DA0                   mov dword ptr [ebp-60], ecx   

:0040AAA4 894598                   mov dword ptr [ebp-68], eax   

:0040AAA7 8D55C4                   lea edx, dword ptr [ebp-3C]   

:0040AAAA 899570FFFFFF             mov dword ptr [ebp+FFFFFF70], edx   

:0040AAB0 C78568FFFFFF08400000     mov dword ptr [ebp+FFFFFF68], 00004008   
```



* Possible StringData Ref from Code Obj ->"Invalid Serial Number - Error "

->"4"

|

```
:0040AABA C745805C3A4000          mov [ebp-80], 00403A5C
:0040AAC1 C78578FFFFFF08000000    mov dword ptr [ebp+FFFFFF78], 00000008
:0040AACB 8D9578FFFFFF            lea edx, dword ptr [ebp+FFFFFF78]
:0040AAD1 8D4DA8                  lea ecx, dword ptr [ebp-58]
```

* Reference To: `MSVBVM60.__vbaVarDup`, Ord:0000h

|

```
:0040AAD4 FF1544124000            Call dword ptr [00401244]
:0040AADA 8D4588                  lea eax, dword ptr [ebp-78]
:0040AADD 50                      push eax
:0040AADE 8D4D98                  lea ecx, dword ptr [ebp-68]
:0040AAE1 51                      push ecx
:0040AAE2 8D9568FFFFFF            lea edx, dword ptr [ebp+FFFFFF68]
:0040AAE8 52                      push edx
:0040AAE9 6A10                    push 00000010
:0040AAEB 8D45A8                  lea eax, dword ptr [ebp-58]
:0040AAEE 50                      push eax
```

* Reference To: `MSVBVM60.rtcMsgBox`, Ord:0253h

|

```
:0040AAEF FF15B8104000            Call dword ptr [004010B8]
:0040AAF5 8D4D88                  lea ecx, dword ptr [ebp-78]
:0040AAF8 51                      push ecx
:0040AAF9 8D5598                  lea edx, dword ptr [ebp-68]
:0040AAFC 52                      push edx
:0040AAFD 8D45A8                  lea eax, dword ptr [ebp-58]
:0040AB00 50                      push eax
:0040AB01 E907020000              jmp 0040AD0D
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|
`:0040AA8F(C)`

|

```
:0040AB06 83CFFF                  or edi, FFFFFFFF-------------------------->上面几处跳转到这里才是正确的
:0040AB09 89BD44FFFFFF            mov dword ptr [ebp+FFFFFF44], edi
:0040AB0F 8D8D44FFFFFF            lea ecx, dword ptr [ebp+FFFFFF44]
:0040AB15 51                      push ecx
:0040AB16 E815080000              call 0040B330
:0040AB1B 898540FFFFFF            mov dword ptr [ebp+FFFFFF40], eax
:0040AB21 8B55D8                  mov edx, dword ptr [ebp-28]
:0040AB24 81EAC8010000            sub edx, 000001C8
:0040AB2A 0F806A020000            jo 0040AD9A
:0040AB30 899530FFFFFF            mov dword ptr [ebp+FFFFFF30], edx
:0040AB36 8B45DC                  mov eax, dword ptr [ebp-24]
:0040AB39 05C8010000              add eax, 000001C8
:0040AB3E 0F8056020000            jo 0040AD9A
:0040AB44 898534FFFFFF            mov dword ptr [ebp+FFFFFF34], eax
:0040AB4A 8B45E0                  mov eax, dword ptr [ebp-20]
:0040AB4D 8B08                    mov ecx, dword ptr [eax]
:0040AB4F 8D953CFFFFFF            lea edx, dword ptr [ebp+FFFFFF3C]
:0040AB55 52                      push edx
:0040AB56 8D9540FFFFFF            lea edx, dword ptr [ebp+FFFFFF40]
:0040AB5C 52                      push edx
:0040AB5D 8D9530FFFFFF            lea edx, dword ptr [ebp+FFFFFF30]
:0040AB63 52                      push edx
:0040AB64 8D9534FFFFFF            lea edx, dword ptr [ebp+FFFFFF34]
:0040AB6A 52                      push edx
:0040AB6B 8B550C                  mov edx, dword ptr [ebp+0C]
:0040AB6E 52                      push edx
:0040AB6F 53                      push ebx
:0040AB70 56                      push esi
:0040AB71 50                      push eax
:0040AB72 FF5120                  call [ecx+20]
:0040AB75 DBE2                    fclex
:0040AB77 85C0                    test eax, eax
:0040AB79 7D12                    jge 0040AB8D
:0040AB7B 6A20                    push 00000020
:0040AB7D 6840384000              push 00403840
:0040AB82 8B4DE0                  mov ecx, dword ptr [ebp-20]
:0040AB85 51                      push ecx
:0040AB86 50                      push eax
```

* Reference To: `MSVBVM60.__vbaHresultCheckObj`, Ord:0000h

|

```
:0040AB87 FF1584104000            Call dword ptr [00401084]
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|
`:0040AB79(C)`

|

```
:0040AB8D 33D2                    xor edx, edx
:0040AB8F 6639BD3CFFFFFF          cmp word ptr [ebp+FFFFFF3C], di
:0040AB96 0F94C2                  sete dl
:0040AB99 F7DA                    neg edx
:0040AB9B 6685D2                  test dx, dx
:0040AB9E 0F84F9000000            je 0040AC9D------------------------------>跳则错误
:0040ABA4 89BD44FFFFFF            mov dword ptr [ebp+FFFFFF44], edi
:0040ABAA 8D8544FFFFFF            lea eax, dword ptr [ebp+FFFFFF44]
:0040ABB0 50                      push eax
:0040ABB1 E87A070000              call 0040B330
:0040ABB6 898540FFFFFF            mov dword ptr [ebp+FFFFFF40], eax
:0040ABBC 8B45E0                  mov eax, dword ptr [ebp-20]
:0040ABBF 8B08                    mov ecx, dword ptr [eax]
:0040ABC1 8D953CFFFFFF            lea edx, dword ptr [ebp+FFFFFF3C]
:0040ABC7 52                      push edx
:0040ABC8 8D9540FFFFFF            lea edx, dword ptr [ebp+FFFFFF40]
:0040ABCE 52                      push edx
:0040ABCF 56                      push esi
:0040ABD0 50                      push eax
:0040ABD1 FF512C                  call [ecx+2C]
:0040ABD4 DBE2                    fclex
:0040ABD6 85C0                    test eax, eax
:0040ABD8 7D12                    jge 0040ABEC
:0040ABDA 6A2C                    push 0000002C
:0040ABDC 6840384000              push 00403840
:0040ABE1 8B4DE0                  mov ecx, dword ptr [ebp-20]
:0040ABE4 51                      push ecx
:0040ABE5 50                      push eax
```

* Reference To: `MSVBVM60.__vbaHresultCheckObj`, Ord:0000h

|

```
:0040ABE6 FF1584104000            Call dword ptr [00401084]
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|
`:0040ABD8(C)`

|

```
:0040ABEC 33D2                    xor edx, edx
:0040ABEE 6639BD3CFFFFFF          cmp word ptr [ebp+FFFFFF3C], di
:0040ABF5 0F94C2                  sete dl
:0040ABF8 F7DA                    neg edx
:0040ABFA 6685D2                  test dx, dx
:0040ABFD 742C                    je 0040AC2B------------------------->跳则错误
:0040ABFF 897DCC                  mov dword ptr [ebp-34], edi
:0040AC02 8B13                    mov edx, dword ptr [ebx]
:0040AC04 B938D04000              mov ecx, 0040D038
```

* Reference To: `MSVBVM60.__vbaStrCopy`, Ord:0000h

|

```
:0040AC09 8B3504124000            mov esi, dword ptr [00401204]
:0040AC0F FFD6                    call esi
:0040AC11 8B450C                  mov eax, dword ptr [ebp+0C]
:0040AC14 8B10                    mov edx, dword ptr [eax]
:0040AC16 B93CD04000              mov ecx, 0040D03C
:0040AC1B FFD6                    call esi
:0040AC1D 66C70540D040000000      mov word ptr [0040D040], 0000
:0040AC26 E9ED000000              jmp 0040AD18
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|
`:0040ABFD(C)`

|

```
:0040AC2B B904000280              mov ecx, 80020004
:0040AC30 894D90                  mov dword ptr [ebp-70], ecx
:0040AC33 B80A000000              mov eax, 0000000A
:0040AC38 894588                  mov dword ptr [ebp-78], eax
:0040AC3B 894DA0                  mov dword ptr [ebp-60], ecx
:0040AC3E 894598                  mov dword ptr [ebp-68], eax
:0040AC41 8D4DC4                  lea ecx, dword ptr [ebp-3C]
:0040AC44 898D70FFFFFF            mov dword ptr [ebp+FFFFFF70], ecx
```



```asm
:0040AC4A C78568FFFFFF08400000    mov dword ptr [ebp+FFFFFF68], 00004008   
* Possible StringData Ref from Code Obj ->"Error 5"   
                              |   
:0040AC54 C74580B83A4000         mov [ebp-80], 00403AB8   
:0040AC5B C78578FFFFFF08000000   mov dword ptr [ebp+FFFFFF78], 00000008   
:0040AC65 8D9578FFFFFF         lea edx, dword ptr [ebp+FFFFFF78]   
:0040AC6B 8D4DA8               lea ecx, dword ptr [ebp-58]   
* Reference To: MSVBVM60.__vbaVarDup, Ord:0000h   
                              |   
:0040AC6E FF1544124000         Call dword ptr [00401244]   
:0040AC74 8D5588               lea edx, dword ptr [ebp-78]   
:0040AC77 52                   push edx   
:0040AC78 8D4598               lea eax, dword ptr [ebp-68]   
:0040AC7B 50                   push eax   
:0040AC7C 8D8D68FFFFFF         lea ecx, dword ptr [ebp+FFFFFF68]   
:0040AC82 51                   push ecx   
:0040AC83 6A10                 push 00000010   
:0040AC85 8D55A8               lea edx, dword ptr [ebp-58]   
:0040AC88 52                   push edx   
* Reference To: MSVBVM60.rtcMsgBox, Ord:0253h   
                              |   
:0040AC89 FF15B8104000         Call dword ptr [004010B8]   
:0040AC8F 8D4588               lea eax, dword ptr [ebp-78]   
:0040AC92 50                   push eax   
:0040AC93 8D4D98               lea ecx, dword ptr [ebp-68]   
:0040AC96 51                   push ecx   
:0040AC97 8D55A8               lea edx, dword ptr [ebp-58]   
:0040AC9A 52                   push edx   
:0040AC9B EB70                 jmp 0040AD0D   
* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040AB9E(C)   
|   
:0040AC9D B904000280           mov ecx, 80020004   
:0040ACA2 894D90               mov dword ptr [ebp-70], ecx   
:0040ACA5 B80A000000           mov eax, 0000000A   
:0040ACAA 894588               mov dword ptr [ebp-78], eax   
:0040ACAD 894DA0               mov dword ptr [ebp-60], ecx   
:0040ACB0 894598               mov dword ptr [ebp-68], eax   
:0040ACB3 8D45C4               lea eax, dword ptr [ebp-3C]   
:0040ACB6 898570FFFFFF         mov dword ptr [ebp+FFFFFF70], eax   
:0040ACBC C78568FFFFFF08400000 mov dword ptr [ebp+FFFFFF68], 00004008   
* Possible StringData Ref from Code Obj ->"Error 6"   
                              |   
:0040ACC6 C74580CC3A4000       mov [ebp-80], 00403ACC   
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:0040A9AC(U), :0040A9F1(U)   
|   
:0040ACCD C78578FFFFFF08000000 mov dword ptr [ebp+FFFFFF78], 00000008   
:0040ACD7 8D9578FFFFFF         lea edx, dword ptr [ebp+FFFFFF78]   
:0040ACDD 8D4DA8               lea ecx, dword ptr [ebp-58]   
* Reference To: MSVBVM60.__vbaVarDup, Ord:0000h   
                              |   
:0040ACE0 FF1544124000         Call dword ptr [00401244]   
:0040ACE6 8D4D88               lea ecx, dword ptr [ebp-78]   
:0040ACE9 51                   push ecx   
:0040ACEA 8D5598               lea edx, dword ptr [ebp-68]   
:0040ACED 52                   push edx   
:0040ACEE 8D8568FFFFFF         lea eax, dword ptr [ebp+FFFFFF68]   
:0040ACF4 50                   push eax   
:0040ACF5 6A10                 push 00000010   
:0040ACF7 8D4DA8               lea ecx, dword ptr [ebp-58]   
:0040ACFA 51                   push ecx   
* Reference To: MSVBVM60.rtcMsgBox, Ord:0253h   
                              |   
:0040ACFB FF15B8104000         Call dword ptr [004010B8]   
* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040AA77(U)   
|   
:0040AD01 8D5588               lea edx, dword ptr [ebp-78]   
:0040AD04 52                   push edx   
:0040AD05 8D4598               lea eax, dword ptr [ebp-68]   
:0040AD08 50                   push eax   
:0040AD09 8D4DA8               lea ecx, dword ptr [ebp-58]   
:0040AD0C 51                   push ecx   
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:0040AB01(U), :0040AC9B(U)   
|   
:0040AD0D 6A03                 push 00000003   
* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h   
                              |   
:0040AD0F FF154C104000         Call dword ptr [0040104C]   
:0040AD15 83C410               add esp, 00000010   
* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040AC26(U)   
|   
:0040AD18 6840384000           push 00403840   
:0040AD1D 6A00                 push 00000000   
* Reference To: MSVBVM60.__vbaCastObj, Ord:0000h   
                              |   
:0040AD1F FF1574124000         Call dword ptr [00401274]   
:0040AD25 50                   push eax   
:0040AD26 8D55E0               lea edx, dword ptr [ebp-20]   
:0040AD29 52                   push edx   
* Reference To: MSVBVM60.__vbaObjSet, Ord:0000h   
                              |   
:0040AD2A FF15B4104000         Call dword ptr [004010B4]   
* Reference To: MSVBVM60.__vbaExitProc, Ord:0000h   
                              |   
:0040AD30 FF15A0104000         Call dword ptr [004010A0]   
:0040AD36 6883AD4000           push 0040AD83   
:0040AD3B EB2C                 jmp 0040AD69   
:0040AD3D 8D45B8               lea eax, dword ptr [ebp-48]   
:0040AD40 50                   push eax   
:0040AD41 8D4DBC               lea ecx, dword ptr [ebp-44]   
:0040AD44 51                   push ecx   
:0040AD45 8D55C0               lea edx, dword ptr [ebp-40]   
:0040AD48 52                   push edx   
:0040AD49 6A03                 push 00000003   
* Reference To: MSVBVM60.__vbaFreeStrList, Ord:0000h   
                              |   
:0040AD4B FF150C124000         Call dword ptr [0040120C]   
:0040AD51 8D4588               lea eax, dword ptr [ebp-78]   
:0040AD54 50                   push eax   
:0040AD55 8D4D98               lea ecx, dword ptr [ebp-68]   
:0040AD58 51                   push ecx   
:0040AD59 8D55A8               lea edx, dword ptr [ebp-58]   
:0040AD5C 52                   push edx   
:0040AD5D 6A03                 push 00000003   
* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h   
                              |   
:0040AD5F FF154C104000         Call dword ptr [0040104C]   
:0040AD65 83C420               add esp, 00000020   
:0040AD68 C3                   ret   
**********************************************************************   
**********************************************************************   
```

### 关键一

```asm
004095B0   $  55             PUSH EBP   
004095B1   .  8BEC          MOV EBP,ESP   
004095B3   .  83EC 08       SUB ESP,8   
004095B6   .  68 06154000   PUSH <JMP.&MSVBVM60.__vbaExceptHandler>  ;  SE handler installation   
004095BB   .  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]   
004095C1   .  50             PUSH EAX   
004095C2   .  64:8925 00000>MOV DWORD PTR FS:[0],ESP   
004095C9   .  81EC D8000000 SUB ESP,0D8   
004095CF   .  53             PUSH EBX   
004095D0   .  56             PUSH ESI   
004095D1   .  57             PUSH EDI   
004095D2   .  8965 F8       MOV DWORD PTR SS:[EBP-8],ESP   
004095D5   .  C745 FC 50134>MOV DWORD PTR SS:[EBP-4],REGAVAX.0040135>   
004095DC   .  8B55 08       MOV EDX,DWORD PTR SS:[EBP+8]   
004095DF   .  33C0          XOR EAX,EAX   
004095E1   .  8D4D D0       LEA ECX,DWORD PTR SS:[EBP-30]   
004095E4   .  8945 E4       MOV DWORD PTR SS:[EBP-1C],EAX   
```



```
004095E7   .  8945 E0       MOV DWORD PTR SS:[EBP-20],EAX   

004095EA   .  8945 DC       MOV DWORD PTR SS:[EBP-24],EAX   

004095ED   .  8945 D8       MOV DWORD PTR SS:[EBP-28],EAX   

004095F0   .  8945 D4       MOV DWORD PTR SS:[EBP-2C],EAX   

004095F3   .  8945 D0       MOV DWORD PTR SS:[EBP-30],EAX   

004095F6   .  8945 CC       MOV DWORD PTR SS:[EBP-34],EAX   

004095F9   .  8945 C8       MOV DWORD PTR SS:[EBP-38],EAX   

004095FC   .  8945 C4       MOV DWORD PTR SS:[EBP-3C],EAX   

004095FF   .  8945 C0       MOV DWORD PTR SS:[EBP-40],EAX   

00409602   .  8945 B0       MOV DWORD PTR SS:[EBP-50],EAX   

00409605   .  8945 A0       MOV DWORD PTR SS:[EBP-60],EAX   

00409608   .  8985 7CFFFFFF MOV DWORD PTR SS:[EBP-84],EAX   

0040960E   .  8985 78FFFFFF MOV DWORD PTR SS:[EBP-88],EAX   

00409614   .  8985 74FFFFFF MOV DWORD PTR SS:[EBP-8C],EAX   

0040961A   .  FF15 04124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCo];  MSVBVM60.__vbaStrCopy   

00409620   .  8B45 D0       MOV EAX,DWORD PTR SS:[EBP-30]   

00409623   .  50           PUSH EAX\------------------------------------>输入的注册码   

00409624   .  FF15 40104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaLenBs>;  MSVBVM60.__vbaLenBstr   

0040962A   .  83F8 1D       CMP EAX,1D---------------------------------->验证序列号的长度   

0040962D   .  0F85 21090000 JNZ REGAVAX.00409F54------------------------>长度不符合要求则跳   

00409633   .  8B55 D0       MOV EDX,DWORD PTR SS:[EBP-30]--------------->输入的注册码   

00409636   .  8B3D E8104000 MOV EDI,DWORD PTR DS:[<&MSVBVM60.#631>\]  ;  MSVBVM60.rtcMidCharBstr   

0040963C   .  8D4D B0       LEA ECX,DWORD PTR SS:[EBP-50]   

0040963F   .  C745 B8 01000>MOV DWORD PTR SS:[EBP-48],1   

00409646   .  51           PUSH ECX   

00409647   .  6A 05        PUSH 5   

00409649   .  52           PUSH EDX   

0040964A   .  C745 B0 02000>MOV DWORD PTR SS:[EBP-50],2   

00409651   .  FFD7         CALL EDI                             ;  <&MSVBVM60.#631>   

00409653   .  8B35 78124000 MOV ESI,DWORD PTR DS:[<&MSVBVM60.__vbaSt>;  MSVBVM60.__vbaStrMove   

00409659   .  8BD0         MOV EDX,EAX   

0040965B   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]   

0040965E   .  FFD6         CALL ESI                             ;  <&MSVBVM60.__vbaStrMove>   

00409660   .  50           PUSH EAX\--------------------------------------(第五位字符)   

00409661   .  68 D0374000   PUSH REGAVAX.004037D0----------------------(0x2D 即字符"-")/比较的内容   

00409666   .  FF15 10114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCm>;  MSVBVM60.__vbaStrCmp   

0040966C   .  8BD8         MOV EBX,EAX\-------------------------------->比较的结果(为零表示相等)   

0040966E   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]   

00409671   .  F7DB         NEG EBX   

00409673   .  1BDB         SBB EBX,EBX   

00409675   .  F7DB         NEG EBX   

00409677   .  F7DB         NEG EBX   

00409679   .  FF15 AC124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStr   

0040967F   .  8D4D B0       LEA ECX,DWORD PTR SS:[EBP-50]   

00409682   .  FF15 38104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVar   

00409688   .  66:85DB      TEST BX,BX\--------------------------------   

0040968B   .  0F85 C3080000 JNZ REGAVAX.00409F54----------------------->不相等则跳走返回   

00409691   .  8B4D D0       MOV ECX,DWORD PTR SS:[EBP-30]-------------->输入的注册码   

00409694   .  8D45 B0       LEA EAX,DWORD PTR SS:[EBP-50]   

00409697   .  50           PUSH EAX   

00409698   .  51           PUSH ECX   

00409699   .  C745 B8 01000>MOV DWORD PTR SS:[EBP-48],1   

004096A0   .  C745 B0 02000>MOV DWORD PTR SS:[EBP-50],2   

004096A7   .  FF15 40104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaLenBs>;  MSVBVM60.__vbaLenBstr   

004096AD   .  8B55 D0       MOV EDX,DWORD PTR SS:[EBP-30]   

004096B0   .  83E8 04       SUB EAX,4----------------------------------   

004096B3   .  0F80 01090000 JO REGAVAX.00409FBA------------------------/注册码的长度-4   

004096B9   .  50           PUSH EAX   

004096BA   .  52           PUSH EDX   

004096BB   .  FFD7         CALL EDI   

004096BD   .  8BD0         MOV EDX,EAX   

004096BF   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]   

004096C2   .  FFD6         CALL ESI   

004096C4   .  50           PUSH EAX\----------------------------(倒数第五位字符)   

004096C5   .  68 D0374000   PUSH REGAVAX.004037D0----------------(0x2D 即字符"-")/比较的内容   

004096CA   .  FF15 10114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCm>;  MSVBVM60.__vbaStrCmp   

004096D0   .  8BD8         MOV EBX,EAX\-------------------------------->比较的结果(为零表示相等)   

004096D2   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]   

004096D5   .  F7DB         NEG EBX   

004096D7   .  1BDB         SBB EBX,EBX   

004096D9   .  F7DB         NEG EBX   

004096DB   .  F7DB         NEG EBX   

004096DD   .  FF15 AC124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStr   

004096E3   .  8D4D B0       LEA ECX,DWORD PTR SS:[EBP-50]   

004096E6   .  FF15 38104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVar   

004096EC   .  66:85DB      TEST BX,BX\---------------------------------   

004096EF   .  0F85 5F080000 JNZ REGAVAX.00409F54------------------------>不相等则跳走返回   

004096F5   .  8B45 D0       MOV EAX,DWORD PTR SS:[EBP-30]--------------->输入的注册码   

004096F8   .  6A 04        PUSH 4-------------------------------------->位数   

004096FA   .  50           PUSH EAX\------------------------------------>输入的注册码   

004096FB   .  FF15 5C124000 CALL DWORD PTR DS:[<&MSVBVM60.#616>\]     ;  MSVBVM60.rtcLeftCharBstr(取左边四位)   

00409701   .  8BD0         MOV EDX,EAX\--------------------------------->最左边的 4 位字符   

00409703   .  8D4D E4       LEA ECX,DWORD PTR SS:[EBP-1C]--------------->新地址   

00409706   .  FFD6         CALL ESI\------------------------------------>__VBAStrMove(上面的地址)   

00409708   .  8B4D D0       MOV ECX,DWORD PTR SS:[EBP-30]--------------->输入的注册码   

0040970B   .  6A 04        PUSH 4-------------------------------------->位数   

0040970D   .  51           PUSH ECX\------------------------------------>输入的注册码   

0040970E   .  FF15 70124000 CALL DWORD PTR DS:[<&MSVBVM60.#618>\]     ;  MSVBVM60.rtcRightCharBstr(取右边四位)   

00409714   .  8BD0         MOV EDX,EAX\--------------------------------->最右边的四位   

00409716   .  8D4D DC       LEA ECX,DWORD PTR SS:[EBP-24]--------------->新地址   

00409719   .  FFD6         CALL ESI\------------------------------------>__VBAStrMove(上面的地址)   

0040971B   .  8B45 D0       MOV EAX,DWORD PTR SS:[EBP-30]--------------->输入的注册码   

0040971E   .  8D55 B0       LEA EDX,DWORD PTR SS:[EBP-50]   

00409721   .  52           PUSH EDX   

00409722   .  BB 02000000   MOV EBX,2   

00409727   .  6A 06        PUSH 6   

00409729   .  50           PUSH EAX   

0040972A   .  C745 B8 13000>MOV DWORD PTR SS:[EBP-48],13   

00409731   .  895D B0       MOV DWORD PTR SS:[EBP-50],EBX   

00409734   .  FFD7         CALL EDI\---------------------------------->掐头去尾   

00409736   .  8BD0         MOV EDX,EAX\------------------------------->掐头去尾的注册码   

00409738   .  8D4D D0       LEA ECX,DWORD PTR SS:[EBP-30]------------->新地址   

0040973B   .  FFD6         CALL ESI\---------------------------------->掐头去尾的注册码保存在上面的地址   

0040973D   .  8D4D B0       LEA ECX,DWORD PTR SS:[EBP-50]   

00409740   .  FF15 38104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVar   

00409746   .  8B55 D0       MOV EDX,DWORD PTR SS:[EBP-30]------------->掐头去尾的注册码   

00409749   .  8D4D B0       LEA ECX,DWORD PTR SS:[EBP-50]   

0040974C   .  51           PUSH ECX   

0040974D   .  6A 0B        PUSH 0B----------------------------------->从剩余注册码的第十一位开始   

0040974F   .  52           PUSH EDX   

00409750   .  C745 B8 04000>MOV DWORD PTR SS:[EBP-48],4--------------->取 4 位   
```



```
00409757   .  895D B0       MOV DWORD PTR SS:[EBP-50],EBX   

0040975A   .  FFD7         CALL EDI   

0040975C   .  8BD0         MOV EDX,EAX------------------------------->取得的 4 位字符   

0040975E   .  8D4D CC      LEA ECX,DWORD PTR SS:[EBP-34]------------->新地址   

00409761   .  FFD6         CALL ESI---------------------------------->保存于上面的地址   

00409763   .  8D45 CC      LEA EAX,DWORD PTR SS:[EBP-34]------------->保存的 4 位字符   

00409766   .  68 00384000  PUSH REGAVAX.00403800                    ;  UNICODE "&H"   

0040976B   .  50           PUSH EAX---------------------------------->4 位字符   

0040976C   .  E8 5F090000  CALL REGAVAX.0040A0D0--------------------->将上面的 4 位字符的顺序反转   

00409771   .  8BD0         MOV EDX,EAX------------------------------->反转后的 4 位字符   

00409773   .  8D4D C8      LEA ECX,DWORD PTR SS:[EBP-38]------------->新地址   

00409776   .  FFD6         CALL ESI---------------------------------->保存反转后的 4 位字符   

00409778   .  50           PUSH EAX---------------------------------->反转后的 4 位字符   

00409779   .  FF15 74104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCat]-->将"&H"加到反转后的 4 位字符的前面   

0040977F   .  8BD0         MOV EDX,EAX-------------------------------->反转后前面加上"&H"后的 4 位字符   

00409781   .  8D4D C4      LEA ECX,DWORD PTR SS:[EBP-3C]-------------->新地址   

00409784   .  FFD6         CALL ESI----------------------------------->保存于新地址   

00409786   .  50           PUSH EAX----------------------------------->入栈   

00409787   .  FF15 B0124000 CALL DWORD PTR DS:[<&MSVBVM60.#581>]     ;  MSVBVM60.rtcR8ValFromBstr   

0040978D   .  8B1D 58124000 MOV EBX,DWORD PTR DS:[<&MSVBVM60.__vbaFp>;  MSVBVM60.__vbaFpI4   

00409793   .  FFD3         CALL EBX                                   ;  <&MSVBVM60.__vbaFpI4>   

00409795   .  8D8D 74FFFFFF LEA ECX,DWORD PTR SS:[EBP-8C]   

0040979B   .  8985 74FFFFFF MOV DWORD PTR SS:[EBP-8C],EAX   

004097A1   .  51           PUSH ECX   

004097A2   .  E8 E9080000  CALL REGAVAX.0040A090--------------------->   

004097A7   .  D99D 70FFFFFF FSTP DWORD PTR SS:[EBP-90]   

004097AD   .  D985 70FFFFFF FLD DWORD PTR SS:[EBP-90]   

004097B3   .  DC2D 48134000 FSUBR QWORD PTR DS:[401348]   

004097B9   .  DFE0         FSTSW AX   

004097BB   .  A8 0D        TEST AL,0D-------------------------------->   

004097BD   .  0F85 F2070000 JNZ REGAVAX.00409FB5   

004097C3   .  FFD3         CALL EBX   

004097C5   .  8945 EC      MOV DWORD PTR SS:[EBP-14],EAX   

004097C8   .  8D55 C4      LEA EDX,DWORD PTR SS:[EBP-3C]   

004097CB   .  8D45 C8      LEA EAX,DWORD PTR SS:[EBP-38]   

004097CE   .  52           PUSH EDX   

004097CF   .  8D4D CC      LEA ECX,DWORD PTR SS:[EBP-34]   

004097D2   .  50           PUSH EAX   

004097D3   .  51           PUSH ECX   

004097D4   .  6A 03        PUSH 3   

004097D6   .  FF15 0C124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS];  MSVBVM60.__vbaFreeStrList   

004097DC   .  83C4 10      ADD ESP,10   

004097DF   .  8D4D B0      LEA ECX,DWORD PTR SS:[EBP-50]   

004097E2   .  FF15 38104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVar   

004097E8   .  8D55 B0      LEA EDX,DWORD PTR SS:[EBP-50]   

004097EB   .  C745 B8 01000>MOV DWORD PTR SS:[EBP-48],1   

004097F2   .  C745 B0 02000>MOV DWORD PTR SS:[EBP-50],2   

004097F9   .  52           PUSH EDX   

004097FA   .  8B45 D0      MOV EAX,DWORD PTR SS:[EBP-30]------------->掐头去尾的注册码   

004097FD   .  6A 05        PUSH 5------------------------------------>第 5 位   

004097FF   .  50           PUSH EAX---------------------------------->掐头去尾的注册码   

00409800   .  FFD7         CALL EDI---------------------------------->取相应位   

00409802   .  8BD0         MOV EDX,EAX------------------------------->取得的字符   

00409804   .  8D4D CC      LEA ECX,DWORD PTR SS:[EBP-34]   

00409807   .  FFD6         CALL ESI   

00409809   .  50           PUSH EAX---------------------------(取得的字符)   

0040980A   .  68 D0374000  PUSH REGAVAX.004037D0---------------------("-")/比较的内容   

0040980F   .  FF15 10114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCm>;  MSVBVM60.__vbaStrCmp   

00409815   .  8BD8         MOV EBX,EAX------------------------------------>为零表示相等   

00409817   .  8D4D CC      LEA ECX,DWORD PTR SS:[EBP-34]   

0040981A   .  F7DB         NEG EBX   

0040981C   .  1BDB         SBB EBX,EBX   

0040981E   .  F7DB         NEG EBX   

00409820   .  F7DB         NEG EBX   

00409822   .  FF15 AC124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStr   

00409828   .  8D4D B0      LEA ECX,DWORD PTR SS:[EBP-50]   

0040982B   .  FF15 38104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVar   

00409831   .  66:85DB      TEST BX,BX   

00409834   .  0F85 1A070000 JNZ REGAVAX.00409F54------------------------>不为零则跳走返回   

0040983A   .  8B55 D0      MOV EDX,DWORD PTR SS:[EBP-30]   

0040983D   .  8D4D B0      LEA ECX,DWORD PTR SS:[EBP-50]--------------->掐头去尾的注册码   

00409840   .  51           PUSH ECX------------------------------------>入栈   

00409841   .  6A 0A        PUSH 0A------------------------------------->从第十位开始   

00409843   .  52           PUSH EDX   

00409844   .  C745 B8 01000>MOV DWORD PTR SS:[EBP-48],1   

0040984B   .  C745 B0 02000>MOV DWORD PTR SS:[EBP-50],2   

00409852   .  FFD7         CALL EDI------------------------------------>取相应位   

00409854   .  8BD0         MOV EDX,EAX--------------------------------->保存   

00409856   .  8D4D CC      LEA ECX,DWORD PTR SS:[EBP-34]   

00409859   .  FFD6         CALL ESI   

0040985B   .  50           PUSH EAX-------------------------------(取得的相应位)   

0040985C   .  68 D0374000  PUSH REGAVAX.004037D0---------------------------("-")/比较的内容   

00409861   .  FF15 10114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCm>;  MSVBVM60.__vbaStrCmp   

00409867   .  8BD8         MOV EBX,EAX-------------------------------->为零表示相等   

00409869   .  8D4D CC      LEA ECX,DWORD PTR SS:[EBP-34]   

0040986C   .  F7DB         NEG EBX   

0040986E   .  1BDB         SBB EBX,EBX   

00409870   .  F7DB         NEG EBX   

00409872   .  F7DB         NEG EBX   

00409874   .  FF15 AC124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStr   

0040987A   .  8D4D B0      LEA ECX,DWORD PTR SS:[EBP-50]   

0040987D   .  FF15 38104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVar   

00409883   .  66:85DB      TEST BX,BX   

00409886   .  0F85 C8060000 JNZ REGAVAX.00409F54------------------------>不为零则跳走返回   

0040988C   .  8B4D D0      MOV ECX,DWORD PTR SS:[EBP-30]--------------->掐头去尾的注册码   

0040988F   .  8D45 B0      LEA EAX,DWORD PTR SS:[EBP-50]   

00409892   .  50           PUSH EAX   

00409893   .  6A 0F        PUSH 0F------------------------------------->第十五位   

00409895   .  51           PUSH ECX------------------------------------>掐头去尾的注册码   

00409896   .  C745 B8 01000>MOV DWORD PTR SS:[EBP-48],1   

0040989D   .  C745 B0 02000>MOV DWORD PTR SS:[EBP-50],2   

004098A4   .  FFD7         CALL EDI------------------------------------>取相应位   

004098A6   .  8BD0         MOV EDX,EAX   

004098A8   .  8D4D CC      LEA ECX,DWORD PTR SS:[EBP-34]   

004098AB   .  FFD6         CALL ESI   

004098AD   .  50           PUSH EAX-------------------------(取得的相应位字符)   

004098AE   .  68 D0374000  PUSH REGAVAX.004037D0-------------------------("-")/比较的内容   

004098B3   .  FF15 10114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCm>;  MSVBVM60.__vbaStrCmp   

004098B9   .  8BD8         MOV EBX,EAX--------------------------------->为零表示相等   

004098BB   .  8D4D CC      LEA ECX,DWORD PTR SS:[EBP-34]   

004098BE   .  F7DB         NEG EBX   

004098C0   .  1BDB         SBB EBX,EBX   

004098C2   .  F7DB         NEG EBX   

004098C4   .  F7DB         NEG EBX   
```



004098C6   .  FF15 AC124000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>; MSVBVM60.__vbaFreeStr   

004098CC   .  8D4D B0        LEA ECX,DWORD PTR SS:[EBP-50]   

004098CF   .  FF15 38104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>; MSVBVM60.__vbaFreeVar   

004098D5   .  66:85DB        TEST BX,BX   

004098D8   .  0F85 76060000 JNZ REGAVAX.00409F54------------------------>不相等则跳走返回   

004098DE   .  B8 01000000    MOV EAX,1   

004098E3   .  8D55 B0        LEA EDX,DWORD PTR SS:[EBP-50]   

004098E6   .  8945 B8        MOV DWORD PTR SS:[EBP-48],EAX   

004098E9   .  B9 02000000    MOV ECX,2   

004098EE   .  8945 A8        MOV DWORD PTR SS:[EBP-58],EAX   

004098F1   .  8B45 D0        MOV EAX,DWORD PTR SS:[EBP-30]--------------->掐头去尾的注册码   

004098F4   .  52             PUSH EDX   

004098F5   .  51             PUSH ECX   

004098F6   .  50             PUSH EAX   

004098F7   .  894D B0        MOV DWORD PTR SS:[EBP-50],ECX   

004098FA   .  894D A0        MOV DWORD PTR SS:[EBP-60],ECX   

004098FD   .  FFD7           CALL EDI   

004098FF   .  8BD0           MOV EDX,EAX   

00409901   .  8D4D CC        LEA ECX,DWORD PTR SS:[EBP-34]   

00409904   .  FFD6           CALL ESI   

00409906   .  8B55 D0        MOV EDX,DWORD PTR SS:[EBP-30]------------->掐头去尾的注册码   

00409909   .  8D4D A0        LEA ECX,DWORD PTR SS:[EBP-60]   

0040990C   .  50             PUSH EAX   

0040990D   .  51             PUSH ECX   

0040990E   .  6A 13          PUSH 13----------------------------------->取其第 19 位   

00409910   .  52             PUSH EDX   

00409911   .  FFD7           CALL EDI\---------------------------------->取相应位   

00409913   .  8BD0           MOV EDX,EAX   

00409915   .  8D4D C8        LEA ECX,DWORD PTR SS:[EBP-38]   

00409918   .  FFD6           CALL ESI   

0040991A   .  50             PUSH EAX\---------------------------------->准备比较(注册码的第 24 位和第 7 位)   

0040991B   .  FF15 10114000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCm>; MSVBVM60.__vbaStrCmp   

00409921   .  8BD8           MOV EBX,EAX\-------------------------------->比较的结果(为零表示相等)   

00409923   .  8D45 C8        LEA EAX,DWORD PTR SS:[EBP-38]   

00409926   .  F7DB           NEG EBX   

00409928   .  1BDB           SBB EBX,EBX   

0040992A   .  8D4D CC        LEA ECX,DWORD PTR SS:[EBP-34]   

0040992D   .  50             PUSH EAX   

0040992E   .  51             PUSH ECX   

0040992F   .  F7DB           NEG EBX   

00409931   .  6A 02          PUSH 2   

00409933   .  F7DB           NEG EBX   

00409935   .  FF15 0C124000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>; MSVBVM60.__vbaFreeStrList   

0040993B   .  8D55 A0        LEA EDX,DWORD PTR SS:[EBP-60]   

0040993E   .  8D45 B0        LEA EAX,DWORD PTR SS:[EBP-50]   

00409941   .  52             PUSH EDX   

00409942   .  50             PUSH EAX   

00409943   .  6A 02          PUSH 2   

00409945   .  FF15 4C104000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>; MSVBVM60.__vbaFreeVarList   

0040994B   .  83C4 18        ADD ESP,18   

0040994E   .  66:85DB        TEST BX,BX   

00409951   .  0F85 FD050000 JNZ REGAVAX.00409F54------------------------>不相等则跳走返回   

00409957   .  8B55 D0        MOV EDX,DWORD PTR SS:[EBP-30]--------------->掐头去尾的注册码   

0040995A   .  B9 02000000    MOV ECX,2   

0040995F   .  894D B0        MOV DWORD PTR SS:[EBP-50],ECX   

00409962   .  894D A0        MOV DWORD PTR SS:[EBP-60],ECX   

00409965   .  8D4D B0        LEA ECX,DWORD PTR SS:[EBP-50]   

00409968   .  B8 01000000    MOV EAX,1   

0040996D   .  51             PUSH ECX   

0040996E   .  6A 0E          PUSH 0E------------------------------------->上面注册码的第 14 位   

00409970   .  52             PUSH EDX\------------------------------------>掐头去尾的注册码   

00409971   .  8945 B8        MOV DWORD PTR SS:[EBP-48],EAX   

00409974   .  8945 A8        MOV DWORD PTR SS:[EBP-58],EAX   

00409977   .  FFD7           CALL EDI\------------------------------------>取相应位   

00409979   .  8BD0           MOV EDX,EAX\--------------------------------->取得的字符   

0040997B   .  8D4D CC        LEA ECX,DWORD PTR SS:[EBP-34]   

0040997E   .  FFD6           CALL ESI\------------------------------------>保存   

00409980   .  8B4D D0        MOV ECX,DWORD PTR SS:[EBP-30]--------------->掐头去尾的注册码   

00409983   .  50             PUSH EAX   

00409984   .  8D45 A0        LEA EAX,DWORD PTR SS:[EBP-60]   

00409987   .  50             PUSH EAX   

00409988   .  6A 11          PUSH 11------------------------------------->第 17 位   

0040998A   .  51             PUSH ECX\------------------------------------>掐头去尾的注册码   

0040998B   .  FFD7           CALL EDI\------------------------------------>取其相应位   

0040998D   .  8BD0           MOV EDX,EAX\--------------------------------->保存   

0040998F   .  8D4D C8        LEA ECX,DWORD PTR SS:[EBP-38]   

00409992   .  FFD6           CALL ESI\------------------------------------>保存   

00409994   .  50             PUSH EAX\------------------------------------>准备比较(这里比较的是 19 位和 22 位)   

00409995   .  FF15 10114000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCm>; MSVBVM60.__vbaStrCmp   

0040999B   .  8BD8           MOV EBX,EAX   

0040999D   .  8D55 C8        LEA EDX,DWORD PTR SS:[EBP-38]   

004099A0   .  F7DB           NEG EBX   

004099A2   .  1BDB           SBB EBX,EBX   

004099A4   .  8D45 CC        LEA EAX,DWORD PTR SS:[EBP-34]   

004099A7   .  52             PUSH EDX   

004099A8   .  50             PUSH EAX   

004099A9   .  F7DB           NEG EBX   

004099AB   .  6A 02          PUSH 2   

004099AD   .  F7DB           NEG EBX   

004099AF   .  FF15 0C124000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>; MSVBVM60.__vbaFreeStrList   

004099B5   .  8D4D A0        LEA ECX,DWORD PTR SS:[EBP-60]   

004099B8   .  8D55 B0        LEA EDX,DWORD PTR SS:[EBP-50]   

004099BB   .  51             PUSH ECX   

004099BC   .  52             PUSH EDX   

004099BD   .  6A 02          PUSH 2   

004099BF   .  FF15 4C104000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>; MSVBVM60.__vbaFreeVarList   

004099C5   .  83C4 18        ADD ESP,18   

004099C8   .  66:85DB        TEST BX,BX   

004099CB   .  0F85 83050000 JNZ REGAVAX.00409F54------------------------>不相等则跳走返回   

004099D1   .  8B4D D0        MOV ECX,DWORD PTR SS:[EBP-30]--------------->掐头去尾的注册码   

004099D4   .  8D45 A0        LEA EAX,DWORD PTR SS:[EBP-60]   

004099D7   .  50             PUSH EAX   

004099D8   .  BB 02000000    MOV EBX,2   

004099DD   .  6A 01          PUSH 1   

004099DF   .  51             PUSH ECX   

004099E0   .  C745 A8 04000>MOV DWORD PTR SS:[EBP-58],4   

004099E7   .  895D A0        MOV DWORD PTR SS:[EBP-60],EBX   

004099EA   .  FFD7           CALL EDI\------------------------------------>取掐头去尾后的前 4 位   

004099EC   .  8BD0           MOV EDX,EAX\--------------------------------->掐头去尾后的前 4 位   

004099EE   .  8D4D CC        LEA ECX,DWORD PTR SS:[EBP-34]--------------->新地址   

004099F1   .  FFD6           CALL ESI\------------------------------------>保存   

004099F3   .  8D55 CC        LEA EDX,DWORD PTR SS:[EBP-34]--------------->取得保存的地址   

004099F6   .  52             PUSH EDX\------------------------------------>入栈   

004099F7   .  E8 54090000    CALL REGAVAX.0040A350----------------------->关键运算 2(请自己跟踪)--往下用浮点指令比较   

004099FC   .  8B4D D0        MOV ECX,DWORD PTR SS:[EBP-30]--------------->掐头去尾后的注册码   

004099FF   .  8985 74FFFFFF  MOV DWORD PTR SS:[EBP-8C],EAX\--------------->关键运算二的结果   

00409A05   .  8D45 B0        LEA EAX,DWORD PTR SS:[EBP-50]   

00409A08   .  C745 B8 01000>MOV DWORD PTR SS:[EBP-48],1   

00409A0F   .  50             PUSH EAX   

00409A10   .  6A 10          PUSH 10---------------------------------->取掐头去尾注册码的第 16 位   

00409A12   .  51             PUSH ECX\--------------------------------->掐头去尾后的注册码   

00409A13   .  895D B0        MOV DWORD PTR SS:[EBP-50],EBX   

00409A16   .  FFD7           CALL EDI\--------------------------------->取得相应位字符
```



```
00409A18   .  8BD0             MOV EDX,EAX\------------------------------>保存取得的字符

00409A1A   .  8D4D C8         LEA ECX,DWORD PTR SS:[EBP-38]

00409A1D   .  FFD6            CALL ESI\--------------------------------->保存

00409A1F   .  50              PUSH EAX\--------------------------------->取得的字符的地址

00409A20   .  FF15 EC114000   CALL DWORD PTR DS:[<&MSVBVM60.__vbaR8Str>];字符串（ String ）转换双精度型（ Double ）

00409A26   .  DB85 74FFFFFF   FILD DWORD PTR SS:[EBP-8C]--------------->整数(前四位中的相应位)到 st(0)

00409A2C   .  DD9D 60FFFFFF   FSTP QWORD PTR SS:[EBP-A0]--------------->st(0)的值到[ebp-a0],然后执行一次出栈

00409A32   .  DC9D 60FFFFFF   FCOMP QWORD PTR SS:[EBP-A0]-------------->比较

00409A38   .  DFE0            FSTSW AX\--------------------------------->AX 保存状态字的值

00409A3A   .  F6C4 40         TEST AH,40------------------------------->测试

00409A3D   .  75 07           JNZ SHORT REGAVAX.00409A46--------------->如果此处不跳,则(此处不跳则错误)

00409A3F   .  B8 01000000     MOV EAX,1--------------------------------/EAX 置 1

00409A44   .  EB 02           JMP SHORT REGAVAX.00409A48---------------|跳到下面

00409A46   >  33C0            XOR EAX,EAX

00409A48   >  F7D8            NEG EAX\----------------------------------|NEG EAX

00409A4A   .  8BD8            MOV EBX,EAX\------------------------------|保存结果,此处直接影响 00409A72 处

00409A4C   .  8D55 C8         LEA EDX,DWORD PTR SS:[EBP-38]

00409A4F   .  8D45 CC         LEA EAX,DWORD PTR SS:[EBP-34]

00409A52   .  52              PUSH EDX

00409A53   .  50              PUSH EAX

00409A54   .  6A 02           PUSH 2

00409A56   .  FF15 0C124000   CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>];  MSVBVM60.__vbaFreeStrList

00409A5C   .  8D4D A0         LEA ECX,DWORD PTR SS:[EBP-60]

00409A5F   .  8D55 B0         LEA EDX,DWORD PTR SS:[EBP-50]

00409A62   .  51              PUSH ECX

00409A63   .  52              PUSH EDX

00409A64   .  6A 02           PUSH 2

00409A66   .  FF15 4C104000   CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>];  MSVBVM60.__vbaFreeVarList

00409A6C   .  83C4 18         ADD ESP,18

00409A6F   .  66:85DB        TEST BX,BX\-------------------------------

00409A72   .  0F85 DC040000   JNZ REGAVAX.00409F54---------------------/不相等则跳走返回

00409A78   .  8B4D D0         MOV ECX,DWORD PTR SS:[EBP-30]------------>掐头去尾的注册码

00409A7B   .  8D45 A0         LEA EAX,DWORD PTR SS:[EBP-60]

00409A7E   .  50              PUSH EAX

00409A7F   .  BB 02000000     MOV EBX,2

00409A84   .  6A 0B           PUSH 0B---------------------------------->从第 11 位开始

00409A86   .  51              PUSH ECX\--------------------------------->掐头去尾的注册码

00409A87   .  C745 A8 04000>MOV DWORD PTR SS:[EBP-58],4-------------->取四位

00409A8E   .  895D A0         MOV DWORD PTR SS:[EBP-60],EBX

00409A91   .  FFD7            CALL EDI\--------------------------------->取掐头去尾的注册码的相应 4 位

00409A93   .  8BD0            MOV EDX,EAX\------------------------------>保存取得的四位字符

00409A95   .  8D4D CC         LEA ECX,DWORD PTR SS:[EBP-34]

00409A98   .  FFD6            CALL ESI\--------------------------------->再保存

00409A9A   .  8D55 CC         LEA EDX,DWORD PTR SS:[EBP-34]

00409A9D   .  52              PUSH EDX

00409A9E   .  E8 AD080000     CALL REGAVAX.0040A350----------------------->关键运算 2--------往下用浮点指令比较

00409AA3   .  8B4D D0         MOV ECX,DWORD PTR SS:[EBP-30]--------------->掐头去尾的注册码

00409AA6   .  8985 74FFFFFF   MOV DWORD PTR SS:[EBP-8C],EAX

00409AAC   .  8D45 B0         LEA EAX,DWORD PTR SS:[EBP-50]

00409AAF   .  C745 B8 01000>MOV DWORD PTR SS:[EBP-48],1

00409AB6   .  50              PUSH EAX

00409AB7   .  6A 12           PUSH 12------------------------------------->18

00409AB9   .  51              PUSH ECX

00409ABA   .  895D B0         MOV DWORD PTR SS:[EBP-50],EBX

00409ABD   .  FFD7            CALL EDI

00409ABF   .  8BD0            MOV EDX,EAX\--------------------------------->取其第 18 位

00409AC1   .  8D4D C8         LEA ECX,DWORD PTR SS:[EBP-38]

00409AC4   .  FFD6            CALL ESI

00409AC6   .  50              PUSH EAX

00409AC7   .  FF15 EC114000   CALL DWORD PTR DS:[<&MSVBVM60.__vbaR8Str>];  MSVBVM60.__vbaR8Str

00409ACD   .  DB85 74FFFFFF   FILD DWORD PTR SS:[EBP-8C]

00409AD3   .  DD9D 58FFFFFF   FSTP QWORD PTR SS:[EBP-A8]

00409AD9   .  DC9D 58FFFFFF   FCOMP QWORD PTR SS:[EBP-A8]

00409ADF   .  DFE0            FSTSW AX

00409AE1   .  F6C4 40         TEST AH,40--------------------------------->以下这段和上面类似,具体分析略

00409AE4   .  75 07           JNZ SHORT REGAVAX.00409AED----------------->这里必须跳

00409AE6   .  B8 01000000     MOV EAX,1

00409AEB   .  EB 02           JMP SHORT REGAVAX.00409AEF

00409AED   >  33C0            XOR EAX,EAX

00409AEF   >  F7D8            NEG EAX

00409AF1   .  8BD8            MOV EBX,EAX

00409AF3   .  8D55 C8         LEA EDX,DWORD PTR SS:[EBP-38]

00409AF6   .  8D45 CC         LEA EAX,DWORD PTR SS:[EBP-34]

00409AF9   .  52              PUSH EDX

00409AFA   .  50              PUSH EAX

00409AFB   .  6A 02           PUSH 2

00409AFD   .  FF15 0C124000   CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>];  MSVBVM60.__vbaFreeStrList

00409B03   .  8D4D A0         LEA ECX,DWORD PTR SS:[EBP-60]

00409B06   .  8D55 B0         LEA EDX,DWORD PTR SS:[EBP-50]

00409B09   .  51              PUSH ECX

00409B0A   .  52              PUSH EDX

00409B0B   .  6A 02           PUSH 2

00409B0D   .  FF15 4C104000   CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>];  MSVBVM60.__vbaFreeVarList

00409B13   .  83C4 18         ADD ESP,18

00409B16   .  66:85DB        TEST BX,BX

00409B19   .  0F85 35040000   JNZ REGAVAX.00409F54------------------------>不能跳

00409B1F   .  8B4D D0         MOV ECX,DWORD PTR SS:[EBP-30]--------------->掐头去尾的注册码

00409B22   .  8D45 B0         LEA EAX,DWORD PTR SS:[EBP-50]

00409B25   .  50              PUSH EAX

00409B26   .  6A 0B           PUSH 0B------------------------------------->从第 11 位开始

00409B28   .  51              PUSH ECX

00409B29   .  C745 B8 04000>MOV DWORD PTR SS:[EBP-48],4----------------->取四位

00409B30   .  C745 B0 02000>MOV DWORD PTR SS:[EBP-50],2

00409B37   .  FFD7            CALL EDI\------------------------------------>去相应的 4 位字符

00409B39   .  8BD0            MOV EDX,EAX\--------------------------------->取得的结果

00409B3B   .  8D4D CC         LEA ECX,DWORD PTR SS:[EBP-34]

00409B3E   .  FFD6            CALL ESI\------------------------------------>保存结果

00409B40   .  8D55 CC         LEA EDX,DWORD PTR SS:[EBP-34]

00409B43   .  68 00384000     PUSH REGAVAX.00403800                     ;  UNICODE "&H"

00409B48   .  52              PUSH EDX

00409B49   .  E8 82050000     CALL REGAVAX.0040A0D0----------------------->4 位字符反转

00409B4E   .  8BD0            MOV EDX,EAX

00409B50   .  8D4D C8         LEA ECX,DWORD PTR SS:[EBP-38]

00409B53   .  FFD6            CALL ESI\------------------------------------>保存

00409B55   .  50              PUSH EAX

00409B56   .  FF15 74104000   CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCa>];  MSVBVM60.__vbaStrCat

00409B5C   .  8BD0            MOV EDX,EAX\--------------------------------->"&H"加到反转后的 4 位字符前面

00409B5E   .  8D4D C4         LEA ECX,DWORD PTR SS:[EBP-3C]

00409B61   .  FFD6            CALL ESI\------------------------------------>保存

00409B63   .  50              PUSH EAX\------------------------------------>结果

00409B64   .  FF15 B0124000   CALL DWORD PTR DS:[<&MSVBVM60.#581>]     ;  MSVBVM60.rtcR8ValFromBstr

00409B6A   .  8B1D 58124000   MOV EBX,DWORD PTR DS:[<&MSVBVM60.__vbaFp>];  MSVBVM60.__vbaFpI4

00409B70   .  FFD3            CALL EBX                                  ;  <&MSVBVM60.__vbaFpI4>

00409B72   .  8985 74FFFFFF   MOV DWORD PTR SS:[EBP-8C],EAX

00409B78   .  8D85 74FFFFFF   LEA EAX,DWORD PTR SS:[EBP-8C]

00409B7E   .  50              PUSH EAX
```



```
00409B7F   .  E8 0C050000    CALL REGAVAX.0040A090----------------------->   

00409B84   .  D99D 70FFFFFF  FSTP DWORD PTR SS:\[EBP\-90\]   

00409B8A   .  D985 70FFFFFF  FLD DWORD PTR SS:\[EBP\-90\]   

00409B90   .  DC2D 48134000  FSUBR QWORD PTR DS:\[401348\]   

00409B96   .  DFE0           FSTSW AX   

00409B98   .  A8 0D          TEST AL,0D--------------------------------->   

00409B9A   .  0F85 15040000  JNZ REGAVAX.00409FB5   

00409BA0   .  FFD3           CALL EBX   

00409BA2   .  8D4D C4        LEA ECX,DWORD PTR SS:\[EBP\-3C\]-------------->前面加"&H"的反转后的 4 位字符   

00409BA5   .  8BD8           MOV EBX,EAX   

00409BA7   .  8D55 C8        LEA EDX,DWORD PTR SS:\[EBP\-38\]   

00409BAA   .  51             PUSH ECX   

00409BAB   .  8D45 CC        LEA EAX,DWORD PTR SS:\[EBP\-34\]   

00409BAE   .  52             PUSH EDX   

00409BAF   .  50             PUSH EAX   

00409BB0   .  6A 03          PUSH 3   

00409BB2   .  895D E8        MOV DWORD PTR SS:\[EBP\-18\],EBX   

00409BB5   .  FF15 0C124000  CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeS>;  MSVBVM60.\_\_vbaFreeStrList   

00409BBB   .  83C4 10        ADD ESP,10   

00409BBE   .  8D4D B0        LEA ECX,DWORD PTR SS:\[EBP\-50\]   

00409BC1   .  FF15 38104000  CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeV>;  MSVBVM60.\_\_vbaFreeVar   

00409BC7   .  81E3 01000080  AND EBX,80000001   

00409BCD   .  C745 B8 04000>MOV DWORD PTR SS:\[EBP\-48\],4   

00409BD4   .  C745 B0 02000>MOV DWORD PTR SS:\[EBP\-50\],2   

00409BDB   .  79 05          JNS SHORT REGAVAX.00409BE2   

00409BDD   .  4B             DEC EBX   

00409BDE   .  83CB FE        OR EBX,FFFFFFFE   

00409BE1   .  43             INC EBX   

00409BE2   >  8B45 E8        MOV EAX,DWORD PTR SS:\[EBP\-18\]   

00409BE5   .  B9 09000000    MOV ECX,9   

00409BEA   .  99             CDQ   

00409BEB   .  F7F9           IDIV ECX   

00409BED   .  899D 54FFFFFF  MOV DWORD PTR SS:\[EBP\-AC\],EBX   

00409BF3   .  DB85 54FFFFFF  FILD DWORD PTR SS:\[EBP\-AC\]   

00409BF9   .  DD9D 48FFFFFF  FSTP QWORD PTR SS:\[EBP\-B8\]   

00409BFF   .  DB45 E8        FILD DWORD PTR SS:\[EBP\-18\]   

00409C02   .  8B85 48FFFFFF  MOV EAX,DWORD PTR SS:\[EBP\-B8\]   

00409C08   .  DD9D 40FFFFFF  FSTP QWORD PTR SS:\[EBP\-C0\]   

00409C0E   .  8B8D 44FFFFFF  MOV ECX,DWORD PTR SS:\[EBP\-BC\]   

00409C14   .  8995 50FFFFFF  MOV DWORD PTR SS:\[EBP\-B0\],EDX   

00409C1A   .  8B95 4CFFFFFF  MOV EDX,DWORD PTR SS:\[EBP\-B4\]   

00409C20   .  52             PUSH EDX   

00409C21   .  8B95 40FFFFFF  MOV EDX,DWORD PTR SS:\[EBP\-C0\]   

00409C27   .  50             PUSH EAX   

00409C28   .  51             PUSH ECX   

00409C29   .  52             PUSH EDX   

00409C2A   .  FF15 1C124000  CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaPower>;  MSVBVM60.\_\_vbaPowerR8   

00409C30   .  8B85 50FFFFFF  MOV EAX,DWORD PTR SS:\[EBP\-B0\]   

00409C36   .  B9 007D0000    MOV ECX,7D00   

00409C3B   .  0FAFC3         IMUL EAX,EBX   

00409C3E   .  8B5D E8        MOV EBX,DWORD PTR SS:\[EBP\-18\]   

00409C41   .  8D55 A0        LEA EDX,DWORD PTR SS:\[EBP\-60\]   

00409C44   .  0F80 70030000  JO REGAVAX.00409FBA   

00409C4A   .  2BCB           SUB ECX,EBX   

00409C4C   .  52             PUSH EDX   

00409C4D   .  0F80 67030000  JO REGAVAX.00409FBA   

00409C53   .  03C1           ADD EAX,ECX   

00409C55   .  C745 A0 05000>MOV DWORD PTR SS:\[EBP\-60\],5   

00409C5C   .  0F80 58030000  JO REGAVAX.00409FBA   

00409C62   .  8985 3CFFFFFF  MOV DWORD PTR SS:\[EBP\-C4\],EAX   

00409C68   .  DB85 3CFFFFFF  FILD DWORD PTR SS:\[EBP\-C4\]   

00409C6E   .  DD9D 34FFFFFF  FSTP QWORD PTR SS:\[EBP\-CC\]   

00409C74   .  DC85 34FFFFFF  FADD QWORD PTR SS:\[EBP\-CC\]   

00409C7A   .  D9E1           FABS   

00409C7C   .  DD5D A8        FSTP QWORD PTR SS:\[EBP\-58\]   

00409C7F   .  DFE0           FSTSW AX   

00409C81   .  A8 0D          TEST AL,0D   

00409C83   .  0F85 2C030000  JNZ REGAVAX.00409FB5   

00409C89   .  FF15 FC114000  CALL DWORD PTR DS:\[<&MSVBVM60.#572>\]     ;  MSVBVM60.rtcHexBstrFromVar   

00409C8F   .  8BD0           MOV EDX,EAX\-------------------------------->   

00409C91   .  8D4D CC        LEA ECX,DWORD PTR SS:\[EBP\-34\]   

00409C94   .  FFD6           CALL ESI   

00409C96   .  8B4D D0        MOV ECX,DWORD PTR SS:\[EBP\-30\]   

00409C99   .  8D45 B0        LEA EAX,DWORD PTR SS:\[EBP\-50\]   

00409C9C   .  50             PUSH EAX   

00409C9D   .  6A 01          PUSH 1   

00409C9F   .  51             PUSH ECX   

00409CA0   .  C785 7CFFFFFF&   

00409CAA   .  FFD7           CALL EDI   

00409CAC   .  8BD0           MOV EDX,EAX\------------------------------>掐头去尾取前 4 位   

00409CAE   .  8D4D C8        LEA ECX,DWORD PTR SS:\[EBP\-38\]   

00409CB1   .  FFD6           CALL ESI   

00409CB3   .  50             PUSH EAX   

00409CB4   .  8D55 CC        LEA EDX,DWORD PTR SS:\[EBP\-34\]   

00409CB7   .  8D85 7CFFFFFF  LEA EAX,DWORD PTR SS:\[EBP\-84\]   

00409CBD   .  52             PUSH EDX   

00409CBE   .  50             PUSH EAX   

00409CBF   .  E8 2C050000    CALL REGAVAX.0040A1F0   

00409CC4   .  8BD0           MOV EDX,EAX   

00409CC6   .  8D4D C4        LEA ECX,DWORD PTR SS:\[EBP\-3C\]   

00409CC9   .  FFD6           CALL ESI   

00409CCB   .  50             PUSH EAX\---------------------------------->准备比较(748D)   

00409CCC   .  FF15 10114000  CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaStrCm>;  MSVBVM60.\_\_vbaStrCmp(和掐头去尾后的前 4 位比较)   

00409CD2   .  8BD8           MOV EBX,EAX\------------------------------->保存比较的结果   

00409CD4   .  8D4D C4        LEA ECX,DWORD PTR SS:\[EBP\-3C\]   

00409CD7   .  F7DB           NEG EBX   

00409CD9   .  8D55 C8        LEA EDX,DWORD PTR SS:\[EBP\-38\]   

00409CDC   .  51             PUSH ECX   

00409CDD   .  1BDB           SBB EBX,EBX   

00409CDF   .  8D45 CC        LEA EAX,DWORD PTR SS:\[EBP\-34\]   

00409CE2   .  52             PUSH EDX   

00409CE3   .  50             PUSH EAX   

00409CE4   .  F7DB           NEG EBX   

00409CE6   .  6A 03          PUSH 3   

00409CE8   .  F7DB           NEG EBX   

00409CEA   .  FF15 0C124000  CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeS>;  MSVBVM60.\_\_vbaFreeStrList   

00409CF0   .  8D4D A0        LEA ECX,DWORD PTR SS:\[EBP\-60\]   

00409CF3   .  8D55 B0        LEA EDX,DWORD PTR SS:\[EBP\-50\]   

00409CF6   .  51             PUSH ECX   

00409CF7   .  52             PUSH EDX   

00409CF8   .  6A 02          PUSH 2   

00409CFA   .  FF15 4C104000  CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeV>;  MSVBVM60.\_\_vbaFreeVarList   

00409D00   .  83C4 1C        ADD ESP,1C   

00409D03   .  66:85DB        TEST BX,BX\--------------------------------   

00409D06   .  0F85 48020000  JNZ REGAVAX.00409F54----------------------->不相等则跳走   

00409D0C   .  8B85 50FFFFFF  MOV EAX,DWORD PTR SS:\[EBP\-B0\]   

00409D12   .  8B4D E8        MOV ECX,DWORD PTR SS:\[EBP\-18\]   

00409D15   .  6BC0 0A        IMUL EAX,EAX,0A   

00409D18   .  8B1D A8104000  MOV EBX,DWORD PTR DS:\[<&MSVBVM60.\_\_vbaI4>;  MSVBVM60.\_\_vbaI4Abs   

00409D1E   .  C745 B8 04000>MOV DWORD PTR SS:\[EBP\-48\],4   

00409D25   .  0F80 8F020000  JO REGAVAX.00409FBA   

00409D2B   .  81C1 803E0000  ADD ECX,3E80   

00409D31   .  C745 B0 02000>MOV DWORD PTR SS:\[EBP\-50\],2   

00409D38   .  0F80 7C020000  JO REGAVAX.00409FBA   

00409D3E   .  2BC8           SUB ECX,EAX   

00409D40   .  0F80 74020000  JO REGAVAX.00409FBA   

00409D46   .  FFD3           CALL EBX                                 ;  <&MSVBVM60.\_\_vbaI4Abs>   

00409D48   .  8985 30FFFFFF  MOV DWORD PTR SS:\[EBP\-D0\],EAX   

00409D4E   .  DB85 30FFFFFF  FILD DWORD PTR SS:\[EBP\-D0\]   

00409D54   .  DD9D 28FFFFFF  FSTP QWORD PTR SS:\[EBP\-D8\]   

00409D5A   .  DB85 54FFFFFF  FILD DWORD PTR SS:\[EBP\-AC\]   

00409D60   .  DD9D 20FFFFFF  FSTP QWORD PTR SS:\[EBP\-E0\]   

00409D66   .  DB45 E8        FILD DWORD PTR SS:\[EBP\-18\]   

00409D69   .  8B85 24FFFFFF  MOV EAX,DWORD PTR SS:\[EBP\-DC\]   

00409D6F   .  8B8D 20FFFFFF  MOV ECX,DWORD PTR SS:\[EBP\-E0\]   

00409D75   .  50             PUSH EAX   

00409D76   .  51             PUSH ECX   
```



```
00409D77   .  DD9D 18FFFFFF FSTP QWORD PTR SS:[EBP-E8]   

00409D7D   .  8B95 1CFFFFFF MOV EDX,DWORD PTR SS:[EBP-E4]   

00409D83   .  8B85 18FFFFFF MOV EAX,DWORD PTR SS:[EBP-E8]   

00409D89   .  52            PUSH EDX   

00409D8A   .  50            PUSH EAX   

00409D8B   .  FF15 1C124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaPower>;  MSVBVM60.__vbaPowerR8   

00409D91   .  DCAD 28FFFFFF FSUBR QWORD PTR SS:[EBP-D8]   

00409D97   .  8D4D A0       LEA ECX,DWORD PTR SS:[EBP-60]   

00409D9A   .  C745 A0 05000>MOV DWORD PTR SS:[EBP-60],5   

00409DA1   .  51            PUSH ECX   

00409DA2   .  DD5D A8       FSTP QWORD PTR SS:[EBP-58]   

00409DA5   .  DFE0          FSTSW AX   

00409DA7   .  A8 0D         TEST AL,0D   

00409DA9   .  0F85 06020000 JNZ REGAVAX.00409FB5   

00409DAF   .  FF15 FC114000 CALL DWORD PTR DS:[<&MSVBVM60.#572>]     ;  MSVBVM60.rtcHexBstrFromVar   

00409DB5   .  8BD0          MOV EDX,EAX   

00409DB7   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]   

00409DBA   .  FFD6          CALL ESI   

00409DBC   .  8D55 CC       LEA EDX,DWORD PTR SS:[EBP-34]   

00409DBF   .  8D85 7CFFFFFF LEA EAX,DWORD PTR SS:[EBP-84]   

00409DC5   .  52            PUSH EDX   

00409DC6   .  50            PUSH EAX   

00409DC7   .  C785 7CFFFFFF>MOV DWORD PTR SS:[EBP-84],4   

00409DD1   .  E8 1A040000   CALL REGAVAX.0040A1F0   

00409DD6   .  8BD0          MOV EDX,EAX   

00409DD8   .  8D4D C8       LEA ECX,DWORD PTR SS:[EBP-38]   

00409DDB   .  FFD6          CALL ESI   

00409DDD   .  8B55 D0       MOV EDX,DWORD PTR SS:[EBP-30]   

00409DE0   .  8D4D B0       LEA ECX,DWORD PTR SS:[EBP-50]   

00409DE3   .  51            PUSH ECX   

00409DE4   .  6A 06         PUSH 6   

00409DE6   .  52            PUSH EDX   

00409DE7   .  FFD7          CALL EDI   

00409DE9   .  8BD0          MOV EDX,EAX   

00409DEB   .  8D4D C4       LEA ECX,DWORD PTR SS:[EBP-3C]   

00409DEE   .  FFD6          CALL ESI   

00409DF0   .  50            PUSH EAX   

00409DF1   .  8D45 C8       LEA EAX,DWORD PTR SS:[EBP-38]   

00409DF4   .  50            PUSH EAX   

00409DF5   .  E8 D6020000   CALL REGAVAX.0040A0D0   

00409DFA   .  8BD0          MOV EDX,EAX   

00409DFC   .  8D4D C0       LEA ECX,DWORD PTR SS:[EBP-40]   

00409DFF   .  FFD6          CALL ESI   

00409E01   .  50            PUSH EAX---------------------------------->BF21(与掐头去尾后的第二个 4 位比较)   

00409E02   .  FF15 10114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCm>;  MSVBVM60.__vbaStrCmp   

00409E08   .  8BF8          MOV EDI,EAX------------------------------->保存比较结果   

00409E0A   .  8D4D C0       LEA ECX,DWORD PTR SS:[EBP-40]   

00409E0D   .  F7DF          NEG EDI   

00409E0F   .  8D55 C4       LEA EDX,DWORD PTR SS:[EBP-3C]   

00409E12   .  51            PUSH ECX   

00409E13   .  1BFF          SBB EDI,EDI   

00409E15   .  8D45 C8       LEA EAX,DWORD PTR SS:[EBP-38]   

00409E18   .  52            PUSH EDX   

00409E19   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]   

00409E1C   .  F7DF          NEG EDI   

00409E1E   .  50            PUSH EAX   

00409E1F   .  51            PUSH ECX   

00409E20   .  F7DF          NEG EDI   

00409E22   .  6A 04         PUSH 4   

00409E24   .  FF15 0C124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStrList   

00409E2A   .  8D55 A0       LEA EDX,DWORD PTR SS:[EBP-60]   

00409E2D   .  8D45 B0       LEA EAX,DWORD PTR SS:[EBP-50]   

00409E30   .  52            PUSH EDX   

00409E31   .  50            PUSH EAX   

00409E32   .  6A 02         PUSH 2   

00409E34   .  FF15 4C104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVarList   

00409E3A   .  83C4 20       ADD ESP,20   

00409E3D   .  66:85FF       TEST DI,DI---------------------------------   

00409E40   .  0F85 0E010000 JNZ REGAVAX.00409F54------------------------>测试比较结果,不相等则跳走   

00409E46   .  8D4D D4       LEA ECX,DWORD PTR SS:[EBP-2C]   

00409E49   .  8D55 D8       LEA EDX,DWORD PTR SS:[EBP-28]   

00409E4C   .  51            PUSH ECX   

00409E4D   .  8D45 D0       LEA EAX,DWORD PTR SS:[EBP-30]   

00409E50   .  52            PUSH EDX   

00409E51   .  50            PUSH EAX   

00409E52   .  E8 F9EDFFFF   CALL REGAVAX.00408C50   

00409E57   .  66:85C0       TEST AX,AX   

00409E5A   .  0F84 F4000000 JE REGAVAX.00409F54   

00409E60   .  8B4D D8       MOV ECX,DWORD PTR SS:[EBP-28]   

00409E63   .  8B7D EC       MOV EDI,DWORD PTR SS:[EBP-14]   

00409E66   .  03CF          ADD ECX,EDI   

00409E68   .  0F80 4C010000 JO REGAVAX.00409FBA   

00409E6E   .  FFD3          CALL EBX   

00409E70   .  8D4D B0       LEA ECX,DWORD PTR SS:[EBP-50]   

00409E73   .  8945 B8       MOV DWORD PTR SS:[EBP-48],EAX   

00409E76   .  51            PUSH ECX   

00409E77   .  C745 B0 03000>MOV DWORD PTR SS:[EBP-50],3   

00409E7E   .  FF15 FC114000 CALL DWORD PTR DS:[<&MSVBVM60.#572>]     ;  MSVBVM60.rtcHexBstrFromVar   

00409E84   .  8BD0          MOV EDX,EAX   

00409E86   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]   

00409E89   .  FFD6          CALL ESI   

00409E8B   .  8B4D D4       MOV ECX,DWORD PTR SS:[EBP-2C]   

00409E8E   .  C785 7CFFFFFF>MOV DWORD PTR SS:[EBP-84],4   

00409E98   .  03CF          ADD ECX,EDI   

00409E9A   .  0F80 1A010000 JO REGAVAX.00409FBA   

00409EA0   .  FFD3          CALL EBX   

00409EA2   .  8D55 A0       LEA EDX,DWORD PTR SS:[EBP-60]   

00409EA5   .  8945 A8       MOV DWORD PTR SS:[EBP-58],EAX   

00409EA8   .  52            PUSH EDX   

00409EA9   .  C745 A0 03000>MOV DWORD PTR SS:[EBP-60],3   

00409EB0   .  FF15 FC114000 CALL DWORD PTR DS:[<&MSVBVM60.#572>]     ;  MSVBVM60.rtcHexBstrFromVar   

00409EB6   .  8BD0          MOV EDX,EAX   

00409EB8   .  8D4D C4       LEA ECX,DWORD PTR SS:[EBP-3C]   

00409EBB   .  FFD6          CALL ESI   

00409EBD   .  8B45 DC       MOV EAX,DWORD PTR SS:[EBP-24]   

00409EC0   .  8D4D C4       LEA ECX,DWORD PTR SS:[EBP-3C]   

00409EC3   .  50            PUSH EAX   

00409EC4   .  8D95 78FFFFFF LEA EDX,DWORD PTR SS:[EBP-88]   

00409ECA   .  51            PUSH ECX   

00409ECB   .  52            PUSH EDX   

00409ECC   .  C785 78FFFFFF>MOV DWORD PTR SS:[EBP-88],4   

00409ED6   .  E8 15030000   CALL REGAVAX.0040A1F0   

00409EDB   .  8BD0          MOV EDX,EAX   

00409EDD   .  8D4D C0       LEA ECX,DWORD PTR SS:[EBP-40]   

00409EE0   .  FFD6          CALL ESI   

00409EE2   .  8B1D 10114000 MOV EBX,DWORD PTR DS:[<&MSVBVM60.__vbaSt>;  MSVBVM60.__vbaStrCmp   

00409EE8   .  50            PUSH EAX----------------------------------->20D7(和注册码最末四位比较)   

00409EE9   .  FFD3          CALL EBX                              ;  <&MSVBVM60.__vbaStrCmp>   

00409EEB   .  8BF8          MOV EDI,EAX-------------------------------->比较结果   

00409EED   .  8B45 E4       MOV EAX,DWORD PTR SS:[EBP-1C]-------------->注册码的最前 4 位   

00409EF0   .  F7DF          NEG EDI------------------------------------   

00409EF2   .  1BFF          SBB EDI,EDI                               |   

00409EF4   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]--------------->[ECX]=14A9C   

00409EF7   .  50            PUSH EAX------------------------------------>注册码最前 4 位   

00409EF8   .  8D95 7CFFFFFF LEA EDX,DWORD PTR SS:[EBP-84]--------------->4   

00409EFE   .  F7DF          NEG EDI                                 |   

00409F00   .  51            PUSH ECX------------------------------------>保存   

00409F01   .  52            PUSH EDX------------------------------------>保存   

00409F02   .  F7DF          NEG EDI                                 |   

00409F04   .  E8 E7020000   CALL REGAVAX.0040A1F0---------------------->取 4 位,得到 14A9(最前 4 位的正确值)   

00409F09   .  8BD0          MOV EDX,EAX-------------------------------->保存取得的结果 14A9   

00409F0B   .  8D4D C8       LEA ECX,DWORD PTR SS:[EBP-38]-------------->新地址   
```



### 反汇编代码段

```assembly
00409F0E   .  FFD6          CALL ESI\----------------------------------->保存在上面的地址(__VbastrMove)   

00409F10   .  50            PUSH EAX\----------------------------------->14A9(和注册码最前 4 位比较)   

00409F11   .  FFD3          CALL EBX\----------------------------------->比较!!(__VbaStrCmp)   

00409F13   .  F7D8          NEG EAX\------------------------------------>比较结果   

00409F15   .  1BC0          SBB EAX,EAX   

00409F17   .  8D4D C4       LEA ECX,DWORD PTR SS:[EBP\-3C\]-------------->最末 4 位的计算值   

00409F1A   .  F7D8          NEG EAX   

00409F1C   .  F7D8          NEG EAX   

00409F1E   .  0BF8          OR EDI,EAX\-------------------------------->两次比较的结果作 OR 运算***   

00409F20   .  8D45 C0       LEA EAX,DWORD PTR SS:[EBP\-40\]------------->注册码最末 4 位   

00409F23   .  50            PUSH EAX\---------------------------------->入栈   

00409F24   .  8D55 C8       LEA EDX,DWORD PTR SS:[EBP\-38\]------------->14A9(最前 4 位的正确值)   

00409F27   .  51            PUSH ECX\---------------------------------->最末 4 位的计算值   

00409F28   .  8D45 CC       LEA EAX,DWORD PTR SS:[EBP\-34\]------------->14A9C(最前 4 位的计算值)   

00409F2B   .  52            PUSH EDX\---------------------------------->入栈   

00409F2C   .  50            PUSH EAX\---------------------------------->入栈   

00409F2D   .  6A 04         PUSH 4   

00409F2F   .  FF15 0C124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStrList   

00409F35   .  8D4D A0       LEA ECX,DWORD PTR SS:[EBP\-60\]   

00409F38   .  8D55 B0       LEA EDX,DWORD PTR SS:[EBP\-50\]   

00409F3B   .  51            PUSH ECX   

00409F3C   .  52            PUSH EDX   

00409F3D   .  6A 02         PUSH 2   

00409F3F   .  FF15 4C104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVarList   

00409F45   .  83C4 20       ADD ESP,20   

00409F48   .  66:85FF       TEST DI,DI   

00409F4B   .  75 07         JNZ SHORT REGAVAX.00409F54   

00409F4D   .  C745 E0 FFFFF>MOV DWORD PTR SS:[EBP\-20\],-1   

00409F54   >  9B            WAIT\--------------------------------------->上面各处跳到这里之后返回   

00409F55   .  68 9E9F4000   PUSH REGAVAX.00409F9E   

00409F5A   .  EB 2C         JMP SHORT REGAVAX.00409F88   

00409F5C   .  8D45 C0       LEA EAX,DWORD PTR SS:[EBP\-40\]   

00409F5F   .  8D4D C4       LEA ECX,DWORD PTR SS:[EBP\-3C\]   

00409F62   .  50            PUSH EAX   

00409F63   .  8D55 C8       LEA EDX,DWORD PTR SS:[EBP\-38\]   

00409F66   .  51            PUSH ECX   

00409F67   .  8D45 CC       LEA EAX,DWORD PTR SS:[EBP\-34\]   

00409F6A   .  52            PUSH EDX   

00409F6B   .  50            PUSH EAX   

00409F6C   .  6A 04         PUSH 4   

00409F6E   .  FF15 0C124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStrList   

00409F74   .  8D4D A0       LEA ECX,DWORD PTR SS:[EBP\-60\]   

00409F77   .  8D55 B0       LEA EDX,DWORD PTR SS:[EBP\-50\]   

00409F7A   .  51            PUSH ECX   

00409F7B   .  52            PUSH EDX   

00409F7C   .  6A 02         PUSH 2   

00409F7E   .  FF15 4C104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVarList   

00409F84   .  83C4 20       ADD ESP,20   

00409F87   .  C3            RETN   

00409F88   >  8B35 AC124000 MOV ESI,DWORD PTR DS:[<&MSVBVM60.__vbaFr>;  MSVBVM60.__vbaFreeStr   

00409F8E   .  8D4D E4       LEA ECX,DWORD PTR SS:[EBP\-1C\]   

00409F91   .  FFD6          CALL ESI                              ;  <&MSVBVM60.__vbaFreeStr>   

00409F93   .  8D4D DC       LEA ECX,DWORD PTR SS:[EBP\-24\]   

00409F96   .  FFD6          CALL ESI   

00409F98   .  8D4D D0       LEA ECX,DWORD PTR SS:[EBP\-30\]   

00409F9B   .  FFD6          CALL ESI   

00409F9D   .  C3            RETN   

00409F9E   .  8B4D F0       MOV ECX,DWORD PTR SS:[EBP\-10\]   

00409FA1   .  66:8B45 E0    MOV AX,WORD PTR SS:[EBP\-20\]   

00409FA5   .  5F            POP EDI   

00409FA6   .  5E            POP ESI   

00409FA7   .  64:890D 00000>MOV DWORD PTR FS:[0\],ECX   

00409FAE   .  5B            POP EBX   

00409FAF   .  8BE5          MOV ESP,EBP   

00409FB1   .  5D            POP EBP   

00409FB2   .  C2 0400       RETN 4   

00409FB5   >^ E9 5275FFFF    JMP <JMP.&MSVBVM60.__vbaFPException>   

00409FBA   >  FF15 D4114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaError>;  MSVBVM60.__vbaErrorOverflow   

00409FC0   $  55            PUSH EBP   

00409FC1   .  8BEC          MOV EBP,ESP   

00409FC3   .  83EC 08       SUB ESP,8   

00409FC6   .  68 06154000   PUSH <JMP.&MSVBVM60.__vbaExceptHandler>  ;  SE handler installation   

00409FCB   .  64:A1 0000000>MOV EAX,DWORD PTR FS:[0\]   

00409FD1   .  50            PUSH EAX   

00409FD2   .  64:8925 00000>MOV DWORD PTR FS:[0\],ESP   

00409FD9   .  83EC 2C       SUB ESP,2C   

00409FDC   .  53            PUSH EBX   

00409FDD   .  56            PUSH ESI   

00409FDE   .  57            PUSH EDI   

00409FDF   .  8965 F8       MOV DWORD PTR SS:[EBP\-8\],ESP   

00409FE2   .  C745 FC 60134>MOV DWORD PTR SS:[EBP\-4\],REGAVAX.0040136>   

00409FE9   .  8B55 08       MOV EDX,DWORD PTR SS:[EBP+8\]   

00409FEC   .  33C0          XOR EAX,EAX   

00409FEE   .  8D4D E8       LEA ECX,DWORD PTR SS:[EBP\-18\]   

00409FF1   .  8945 EC       MOV DWORD PTR SS:[EBP\-14\],EAX   

00409FF4   .  8945 E8       MOV DWORD PTR SS:[EBP\-18\],EAX   

00409FF7   .  8945 D8       MOV DWORD PTR SS:[EBP\-28\],EAX   

00409FFA   .  FF15 04124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCo>;  MSVBVM60.__vbaStrCopy   

0040A000   .  8B4D E8       MOV ECX,DWORD PTR SS:[EBP\-18\]   

0040A003   .  8D45 D8       LEA EAX,DWORD PTR SS:[EBP\-28\]   

0040A006   .  50            PUSH EAX   

0040A007   .  6A 06         PUSH 6   

0040A009   .  51            PUSH ECX   

0040A00A   .  C745 E0 13000>MOV DWORD PTR SS:[EBP\-20\],13   

0040A011   .  C745 D8 02000>MOV DWORD PTR SS:[EBP\-28\],2   

0040A018   .  FF15 E8104000 CALL DWORD PTR DS:[<&MSVBVM60.#631>\]     ;  MSVBVM60.rtcMidCharBstr   

0040A01E   .  8BD0          MOV EDX,EAX   

0040A020   .  8D4D E8       LEA ECX,DWORD PTR SS:[EBP\-18\]   

0040A023   .  FF15 78124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrMo>;  MSVBVM60.__vbaStrMove   

0040A029   .  8D4D D8       LEA ECX,DWORD PTR SS:[EBP\-28\]   

0040A02C   .  FF15 38104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVar   

0040A032   .  8B55 10       MOV EDX,DWORD PTR SS:[EBP+10\]   

0040A035   .  8B45 0C       MOV EAX,DWORD PTR SS:[EBP+C\]   

0040A038   .  52            PUSH EDX   

0040A039   .  8D4D E8       LEA ECX,DWORD PTR SS:[EBP\-18\]   

0040A03C   .  50            PUSH EAX   

0040A03D   .  51            PUSH ECX   

0040A03E   .  E8 0DECFFFF   CALL REGAVAX.00408C50   

0040A043   .  66:85C0       TEST AX,AX   

0040A046   .  74 07         JE SHORT REGAVAX.0040A04F   

0040A048   .  C745 EC FFFFF>MOV DWORD PTR SS:[EBP\-14\],-1   

0040A04F   >  68 6AA04000   PUSH REGAVAX.0040A06A   

0040A054   .  EB 0A         JMP SHORT REGAVAX.0040A060   

0040A056   .  8D4D D8       LEA ECX,DWORD PTR SS:[EBP\-28\]   

0040A059   .  FF15 38104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVar   

0040A05F   .  C3            RETN   

0040A060   >  8D4D E8       LEA ECX,DWORD PTR SS:[EBP\-18\]   

0040A063   .  FF15 AC124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStr   

0040A069   .  C3            RETN   

0040A06A   .  8B4D F0       MOV ECX,DWORD PTR SS:[EBP\-10\]   

0040A06D   .  66:8B45 EC    MOV AX,WORD PTR SS:[EBP\-14\]   

0040A071   .  5F            POP EDI   

0040A072   .  5E            POP ESI   

0040A073   .  64:890D 00000>MOV DWORD PTR FS:[0\],ECX   

0040A07A   .  5B            POP EBX   

0040A07B   .  8BE5          MOV ESP,EBP   

0040A07D   .  5D            POP EBP   

0040A07E   .  C2 0C00       RETN 0C   

```

`4dd466d0e1e0cc123afbe504da2462ca`
