
# 数据转换器 V8.6

> 原文：[`www.pediy.com/kssd/pediy06/pediy6110.htm`](https://www.pediy.com/kssd/pediy06/pediy6110.htm)

## 软件信息

数据转换器 V8.6 通用版算法分析

分析者：`wzh123`
软件大小：1508 KB
软件语言：简体中文
软件类别：国产软件 / 共享版 / 数据库类
应用平台：Win9x/NT/2000/XP

### 软件介绍

数据转换器(通用版)是一个可以对各种数据库进行处理的软件，是为专业人员量身定制的专用工具。非专业人员使用数据转换器可以处理一般的数据问题，而专业人员采用数据转换器更是如虎添翼，可以更快更好的进行应用软件的开发。

### 作者申明

只是学习，无其他目的。

本人刚刚学破解，错误在所难免，写的也很乱，请各位包涵，也请各位高手指教。

这个软件是 `delphi` 编的，属于明码比较，算法不难，但是要弄清算法，就要跟进多个 `call` 中，要有耐心。

- 机器码：`6ED363VT`(好眼熟耶，其实就是硬盘 id)
- 注册版本：正式版
- 用户名：`wzh123`
- 注册码：`a234567890`

先脱壳，反编译后，很容易找到核心地方，以下的分析以我的注册信息为例，大家可以根据自己的情况算出注册码。

### 反汇编代码分析

```
.............省略

:0075070C FF92CC000000              call dword ptr [edx+000000CC]
:00750712 8BD0                      mov edx, eax
:00750714 8BC3                      mov eax, ebx
:00750716 59                        pop ecx
:00750717 E8E0000000                call 007507FC-------关键 call(1)，追入
:0075071C 84C0                      test al, al
:0075071E 741D                      je 0075073D---------不能跳
:00750720 33C9                      xor ecx, ecx

* Possible StringData Ref from Code Obj ->"单机版"

|
:00750722 BACC077500                mov edx, 007507CC
:00750727 8BC3                      mov eax, ebx
:00750729 E88E010000                call 007508BC
:0075072E 33D2                      xor edx, edx
:00750730 8B8354030000             mov eax, dword ptr [ebx+00000354]
:00750736 8B08                      mov ecx, dword ptr [eax]
:00750738 FF5164                   call [ecx+64]
:0075073B EB0A                      jmp 00750747

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0075071E(C)
|

* Possible StringData Ref from Code Obj ->"注册信息不正确，请仔细检查！"

|
:0075073D B8DC077500                mov eax, 007507DC
:00750742 E8B5D20000                call 0075D9FC
```

`----------------------------关键 call(1)-----------------------------`

```
.............省略

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00750839(C)
|

:0075085F 8D45F4                   lea eax, dword ptr [ebp-0C]
:00750862 50                        push eax
:00750863 B901000000               mov ecx, 00000001
:00750868 8B550C                   mov edx, dword ptr[ebp+0C]---edx 中为用户名，我的为"wzh123"
:0075086B 8B45FC                   mov eax, dword ptr[ebp-04]----eax 中为硬盘 id，我的为"6ED363VT"
:0075086E E8A9B10000               call 0075BA1C--------关键 call(2)，算法，追入
:00750873 8B45F4                   mov eax, dword ptr [ebp-0C]----------真码
:00750876 8B5508                   mov edx, dword ptr [ebp+08]----------假码
:00750879 E84A44CBFF               call 00404CC8------------------------比较
:0075087E 7502                     jne 00750882-------------------------不等就跳
:00750880 B301                     mov bl, 01---------------------------不跳成功

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0075083B(U), :00750859(C), :0075085D(U), :0075087E(C)
|

:00750882 33C0                     xor eax, eax
:00750884 5A                        pop edx
:00750885 59                        pop ecx
:00750886 59                        pop ecx
:00750887 648910                    mov dword ptr fs:[eax], edx
:0075088A 68B1087500               push 007508B1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:007508AF(U)
|

:0075088F 8D45F4                   lea eax, dword ptr [ebp-0C]
:00750892 BA03000000               mov edx, 00000003
:00750897 E84440CBFF               call 004048E0
:0075089C 8D4508                   lea eax, dword ptr [ebp+08]
:0075089F BA02000000               mov edx, 00000002
:007508A4 E83740CBFF               call 004048E0
:007508A9 C3                       ret
```

`---------------------------关键 call(2)，算法-----------------------------------------`

```
........................省略

:0075BA67 BB01000000               mov ebx, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0075BA89(C)
|

:0075BA6C 8D55E8                   lea edx, dword ptr [ebp-18]
:0075BA6F 8B45FC                   mov eax, dword ptr [ebp-04]

                                  硬盘 id,"6ED363VT"-->eax

:0075BA72 0FB64418FF               movzx eax, byte ptr [eax+ebx-01]

                                  依次取"6ED363VT"-->eax

:0075BA77 E8D0E1CAFF               call 00409C4C--------------------关键 call(3)
:0075BA7C 8B55E8                   mov edx, dword ptr [ebp-18]

                               1、edx=54
                               2、edx=69
                               3、edx=68
                               4、edx=51
                               5、edx=54
                               6、edx=51
                               7、edx=86
                               8、edx=84
                               得到一组数"5469685154518684"

:0075BA7F 8D45F0                   lea eax, dword ptr [ebp-10]
:0075BA82 E8FD90CAFF               call 00404B84
:0075BA87 43                       inc ebx
:0075BA88 4E                       dec esi
:0075BA89 75E1                     jne 0075BA6C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0075BA65(C)
|

:0075BA8B 8B45F8                   mov eax, dword ptr [ebp-08]
:0075BA8E E8E990CAFF               call 00404B7C
:0075BA93 8BF0                     mov esi, eax
:0075BA95 85F6                     test esi, esi
:0075BA97 7E3F                     jle 0075BAD8
:0075BA99 BB01000000               mov ebx, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0075BAD6(C)
|

:0075BA9E 8D55E4                   lea edx, dword ptr [ebp-1C]
:0075BAA1 8B45F8                   mov eax, dword ptr [ebp-08]

                                  "wzh123"-->eax

:0075BAA4 0FB64418FF               movzx eax, byte ptr [eax+ebx-01]

                                  依次取"wzh123"-->eax

:0075BAA9 E89EE1CAFF               call 00409C4C                关键 call(3),算法同上
:0075BAAE 8B55E4                   mov edx, dword ptr [ebp-1C]

                               1、edx=119
                               2、edx=122
                               3、edx=104
                               4、edx=49
                               5、edx=50
                               6、edx=51
                               得到一组数"119122104495051"

:0075BAB1 8D45F0                   lea eax, dword ptr [ebp-10]
:0075BAB4 E8CB90CAFF               call 00404B84
:0075BAB9 8D55E0                   lea edx, dword ptr [ebp-20]
:0075BABC 8B45F8                   mov eax, dword ptr [ebp-08]
:0075BABF 0FB64418FF               movzx eax, byte ptr [eax+ebx-01]
:0075BAC4 E883E1CAFF               call 00409C4C------------又来一遍，这里应该是对中文字串进行处理,我没有追了，有兴趣的兄弟可以跟一下
:0075BAC9 8B55E0                   mov edx, dword ptr [ebp-20]
:0075BACC 8D45EC                   lea eax, dword ptr [ebp-14]
:0075BACF E8B090CAFF               call 00404B84
:0075BAD4 43                       inc ebx
:0075BAD5 4E                       dec esi
:0075BAD6 75C6                     jne 0075BA9E-------------向上循环

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0075BA97(C)
|

:0075BAD8 8D45F4                   lea eax, dword ptr [ebp-0C]
:0075BADB 50                        push eax
:0075BADC 8B45F0                   mov eax, dword ptr [ebp-10]  这里，将对机器码与用户名进行运算得到的两组数连起来"5469685154518684119122104495051"---->eax
:0075BADF E89890CAFF               call 00404B7C

                                  取字串的长度,我这里为 0x1F-->eax
```



```
:0075BAE4 8BD0  mov edx, eax  0x1F-->eax

:0075BAE6 83EA0A  sub edx, 0000000A  edx\=0x1F-0xA=0x15

:0075BAE9 B90A000000  mov ecx, 0000000A  0xA-->ecx

:0075BAEE 8B45F0  mov eax, dword ptr \[ebp\-10\]  "5469685154518684119122104495051"---->eax

:0075BAF1 E8E692CAFF  call 00404DDC----------------关键 call(6)，追入

:0075BAF6 83EF01  sub edi, 00000001

:0075BAF9 720B  jb 0075BB06

:0075BAFB 0F84D0000000  je 0075BBD1  我的在这里会跳下去

:0075BB01 E933020000  jmp 0075BD39

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0075BAF9(C)

|  这里我没有跟，可能是因为我在注册时选择了完全正式版，如果选择的是学习版，也许会走到这里，这只是我的猜测，也许有误，请指正。

* Possible StringData Ref from Code Obj ->"JLH-"

:0075BB06 6870BD7500  push 0075BD70

:0075BB0B 8D45DC  lea eax, dword ptr \[ebp\-24\]

:0075BB0E 8B55F0  mov edx, dword ptr \[ebp\-10\]

:0075BB11 8A5201  mov dl, byte ptr \[edx+01\]

:0075BB14 E88B8FCAFF  call 00404AA4

:0075BB19 FF75DC  push \[ebp\-24\]

:0075BB1C 8D45D8  lea eax, dword ptr \[ebp\-28\]

:0075BB1F 8B55F0  mov edx, dword ptr \[ebp\-10\]

:0075BB22 8A5203  mov dl, byte ptr \[edx+03\]

:0075BB25 E87A8FCAFF  call 00404AA4

:0075BB2A FF75D8  push \[ebp\-28\]

:0075BB2D 8D45D4  lea eax, dword ptr \[ebp\-2C\]

:0075BB30 8B55F0  mov edx, dword ptr \[ebp\-10\]

:0075BB33 8A5205  mov dl, byte ptr \[edx+05\]

:0075BB36 E8698FCAFF  call 00404AA4

:0075BB3B FF75D4  push \[ebp\-2C\]

:0075BB3E 8D45D0  lea eax, dword ptr \[ebp\-30\]

:0075BB41 8B55F0  mov edx, dword ptr \[ebp\-10\]

:0075BB44 8A5207  mov dl, byte ptr \[edx+07\]

:0075BB47 E8588FCAFF  call 00404AA4

:0075BB4C FF75D0  push \[ebp\-30\]

:0075BB4F 8D45CC  lea eax, dword ptr \[ebp\-34\]

:0075BB52 8B55F0  mov edx, dword ptr \[ebp\-10\]

:0075BB55 8A5209  mov dl, byte ptr \[edx+09\]

:0075BB58 E8478FCAFF  call 00404AA4

:0075BB5D FF75CC  push \[ebp\-34\]

* Possible StringData Ref from Code Obj ->"-SOFT-"

:0075BB60 6880BD7500  push 0075BD80

:0075BB65 8D45C8  lea eax, dword ptr \[ebp\-38\]

:0075BB68 8B55F4  mov edx, dword ptr \[ebp\-0C\]

:0075BB6B 8A5201  mov dl, byte ptr \[edx+01\]

:0075BB6E E8318FCAFF  call 00404AA4

:0075BB73 FF75C8  push \[ebp\-38\]

:0075BB76 8D45C4  lea eax, dword ptr \[ebp\-3C\]

:0075BB79 8B55F4  mov edx, dword ptr \[ebp\-0C\]

:0075BB7C 8A5203  mov dl, byte ptr \[edx+03\]

:0075BB7F E8208FCAFF  call 00404AA4

:0075BB84 FF75C4  push \[ebp\-3C\]

:0075BB87 8D45C0  lea eax, dword ptr \[ebp\-40\]

:0075BB8A 8B55F4  mov edx, dword ptr \[ebp\-0C\]

:0075BB8D 8A5205  mov dl, byte ptr \[edx+05\]

:0075BB90 E80F8FCAFF  call 00404AA4

:0075BB95 FF75C0  push \[ebp\-40\]

:0075BB98 8D45BC  lea eax, dword ptr \[ebp\-44\]

:0075BB9B 8B55F4  mov edx, dword ptr \[ebp\-0C\]

:0075BB9E 8A5207  mov dl, byte ptr \[edx+07\]

:0075BBA1 E8FE8ECAFF  call 00404AA4

:0075BBA6 FF75BC  push \[ebp\-44\]

:0075BBA9 8D45B8  lea eax, dword ptr \[ebp\-48\]

:0075BBAC 8B55F4  mov edx, dword ptr \[ebp\-0C\]

:0075BBAF 8A5209  mov dl, byte ptr \[edx+09\]

:0075BBB2 E8ED8ECAFF  call 00404AA4

:0075BBB7 FF75B8  push \[ebp\-48\]

* Possible StringData Ref from Code Obj ->"-WARE0"

:0075BBBA 6890BD7500  push 0075BD90

:0075BBBF 8B4508  mov eax, dword ptr \[ebp+08\]

:0075BBC2 BA0D000000  mov edx, 0000000D

:0075BBC7 E87090CAFF  call 00404C3C

:0075BBCC E968010000  jmp 0075BD39

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0075BAFB(C)

* Possible StringData Ref from Code Obj ->"JLH-"

:0075BBD1 6870BD7500  push 0075BD70  跳到这里，JLH-入栈，即注册码的第一部分

:0075BBD6 8D45B4  lea eax, dword ptr \[ebp\-4C\]

:0075BBD9 8B55F0  mov edx, dword ptr\[ebp\-10\]  "5469685154518684119122104495051"---->edx

:0075BBDC 8A12  mov dl, byte ptr \[edx\]  取字串的第一位即"5"->dl

:0075BBDE E8C18ECAFF  call 00404AA4

:0075BBE3 FF75B4  push \[ebp\-4C\]

:0075BBE6 8D45B0  lea eax, dword ptr \[ebp\-50\]

:0075BBE9 8B55F0  mov edx, dword ptr \[ebp\-10\]

:0075BBEC 8A5202  mov dl, byte ptr \[edx+02\]  取字串的第三位即"6"->dl

:0075BBEF E8B08ECAFF  call 00404AA4

:0075BBF4 FF75B0  push \[ebp\-50\]

:0075BBF7 8D45AC  lea eax, dword ptr \[ebp\-54\]

:0075BBFA 8B55F0  mov edx, dword ptr \[ebp\-10\]

:0075BBFD 8A5204  mov dl, byte ptr \[edx+04\]  取字串的第五位即"6"->dl

:0075BC00 E89F8ECAFF  call 00404AA4

:0075BC05 FF75AC  push \[ebp\-54\]

:0075BC08 8D45A8  lea eax, dword ptr \[ebp\-58\]

:0075BC0B 8B55F0  mov edx, dword ptr \[ebp\-10\]

:0075BC0E 8A5206  mov dl, byte ptr \[edx+06\]  取字串的第七位即"5"->dl

:0075BC11 E88E8ECAFF  call 00404AA4

:0075BC16 FF75A8  push \[ebp\-58\]

:0075BC19 8D45A4  lea eax, dword ptr \[ebp\-5C\]

:0075BC1C 8B55F0  mov edx, dword ptr \[ebp\-10\]

:0075BC1F 8A5208  mov dl, byte ptr \[edx+08\]  取字串的第九位即"5"->dl

:0075BC22 E87D8ECAFF  call 00404AA4

:0075BC27 FF75A4  push \[ebp\-5C\]

\-----------------------这一段是将字串"5469685154518684119122104495051"的第 1、3、 5、7、9 位取出得到注册码的第二部分,最终得到 JLH-56655

* Possible StringData Ref from Code Obj ->"-SOFT-"

:0075BC2A 6880BD7500  push 0075BD80  -SOFT-入栈

:0075BC2F 8D45A0  lea eax, dword ptr \[ebp\-60\]

:0075BC32 8B55F4  mov edx, dword ptr \[ebp\-0C\]  "2210449505"(关键 call(6)中的分析）-->edx

:0075BC35 8A12  mov dl, byte ptr \[edx\]  取字串的第一位即"2"->dl

:0075BC37 E8688ECAFF  call 00404AA4

:0075BC3C FF75A0  push \[ebp\-60\]

:0075BC3F 8D459C  lea eax, dword ptr \[ebp\-64\]

:0075BC42 8B55F4  mov edx, dword ptr \[ebp\-0C\]

:0075BC45 8A5202  mov dl, byte ptr \[edx+02\]  取字串的第三位即"1"->dl
```



:0075BC48 E8578ECAFF  call 00404AA4
:0075BC4D FF759C  push [ebp-64]
:0075BC50 8D4598  lea eax, dword ptr [ebp-68]
:0075BC53 8B55F4  mov edx, dword ptr [ebp-0C]
:0075BC56 8A5204  mov dl, byte ptr [edx+04]
取字串的第五位即"4"->dl
:0075BC59 E8468ECAFF  call 00404AA4
:0075BC5E FF7598  push [ebp-68]
:0075BC61 8D4594  lea eax, dword ptr [ebp-6C]
:0075BC64 8B55F4  mov edx, dword ptr [ebp-0C]
:0075BC67 8A5206  mov dl, byte ptr [edx+06]
取字串的第七位即"9"->dl
:0075BC6A E8358ECAFF  call 00404AA4
:0075BC6F FF7594  push [ebp-6C]
:0075BC72 8D4590  lea eax, dword ptr [ebp-70]
:0075BC75 8B55F4  mov edx, dword ptr [ebp-0C]
:0075BC78 8A5208  mov dl, byte ptr [edx+08]
取字串的第九位即"0"->dl
:0075BC7B E8248ECAFF  call 00404AA4
:0075BC80 FF7590  push [ebp-70]

这一段是将字串"22104495050"的第 1、3、5、7、9 位取出得到注册码的第三、四部分,最终得到 `JLH-56655-SOFT-21490`

`*` Possible StringData Ref from Code Obj -> `-WARE1-`
|
:0075BC83 68A0BD7500  push 0075BDA0  `-WARE1-`入栈
:0075BC88 8D458C  lea eax, dword ptr [ebp-74]
:0075BC8B 8B55EC  mov edx, dword ptr [ebp-14]
将对用户名进行运算得到的数"119122104495051"-->edx
:0075BC8E 8A5207  mov dl, byte ptr [edx+07]
取字串的第八位即"0"->dl
:0075BC91 E80E8ECAFF  call 00404AA4
:0075BC96 FF758C  push [ebp-74]
:0075BC99 8D4588  lea eax, dword ptr [ebp-78]
:0075BC9C 8B55EC  mov edx, dword ptr [ebp-14]
:0075BC9F 8A5204  mov dl, byte ptr [edx+04]
取字串的第五位即"2"->dl
:0075BCA2 E8FD8DCAFF  call 00404AA4
:0075BCA7 FF7588  push [ebp-78]
:0075BCAA 8D4584  lea eax, dword ptr [ebp-7C]
:0075BCAD 8B55EC  mov edx, dword ptr [ebp-14]
:0075BCB0 8A5206  mov dl, byte ptr [edx+06]
取字串的第七位即"1"->dl
:0075BCB3 E8EC8DCAFF  call 00404AA4
:0075BCB8 FF7584  push [ebp-7C]
:0075BCBB 8D4580  lea eax, dword ptr [ebp-80]
:0075BCBE 8B55EC  mov edx, dword ptr [ebp-14]
:0075BCC1 8A5203  mov dl, byte ptr [edx+03]
取字串的第四位即"1"->dl
:0075BCC4 E8DB8DCAFF  call 00404AA4
:0075BCC9 FF7580  push [ebp-80]
:0075BCCC 8D857CFFFFFF  lea eax, dword ptr [ebp+FFFFFF7C]
:0075BCD2 8B55EC  mov edx, dword ptr [ebp-14]
:0075BCD5 8A5201  mov dl, byte ptr [edx+01]
取字串的第二位即"1"->dl
:0075BCD8 E8C78DCAFF  call 00404AA4
:0075BCDD FFB57CFFFFFF  push dword ptr [ebp+FFFFFF7C]
:0075BCE3 8D8578FFFFFF  lea eax, dword ptr [ebp+FFFFFF78]
:0075BCE9 8B55EC  mov edx, dword ptr [ebp-14]
:0075BCEC 8A5202  mov dl, byte ptr [edx+02]
取字串的第三位即"9"->dl
:0075BCEF E8B08DCAFF  call 00404AA4
:0075BCF4 FFB578FFFFFF  push dword ptr [ebp+FFFFFF78]
:0075BCFA 8D8574FFFFFF  lea eax, dword ptr [ebp+FFFFFF74]
:0075BD00 8B55EC  mov edx, dword ptr [ebp-14]
:0075BD03 8A12  mov dl, byte ptr [edx]
取字串的第一位即"1"->dl
:0075BD05 E89A8DCAFF  call 00404AA4
:0075BD0A FFB574FFFFFF  push dword ptr [ebp+FFFFFF74]
:0075BD10 8D8570FFFFFF  lea eax, dword ptr [ebp+FFFFFF70]
:0075BD16 8B55EC  mov edx, dword ptr [ebp-14]
:0075BD19 8A5201  mov dl, byte ptr [edx+01]
取字串的第二位即"1"->dl
:0075BD1C E8838DCAFF  call 00404AA4
:0075BD21 FFB570FFFFFF  push dword ptr [ebp+FFFFFF70]

这一段是将字串"119122104495051"的第 8、5、7、4、2、3、1、2 位取出得到注册码的第五、六部分,最终得到 `JLH-56655-SOFT-21490-WARE1-02111911`

`*` Possible StringData Ref from Code Obj -> `-SL`
|
:0075BD27 68B0BD7500  push 0075BDB0  `-SL` 入栈，即为注册码的第七部分
:0075BD2C 8B4508  mov eax, dword ptr [ebp+08]
:0075BD2F BA16000000  mov edx, 00000016
:0075BD34 E8038FCAFF  call 00404C3C

最终得到注册码"JLH-56655-SOFT-21490-WARE1-02111911-SL"

`*` Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0075BB01(U), :0075BBCC(U)
|
:0075BD39 33C0  xor eax, eax
:0075BD3B 5A  pop edx
:0075BD3C 59  pop ecx
:0075BD3D 59  pop ecx
:0075BD3E 648910  mov dword ptr fs:[eax], edx
:0075BD41 685EBD7500  push 0075BD5E

`*` Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0075BD5C(U)
|
:0075BD46 8D8570FFFFFF  lea eax, dword ptr [ebp+FFFFFF70]
:0075BD4C BA24000000  mov edx, 00000024
:0075BD51 E88A8BCAFF  call 004048E0
:0075BD56 C3  ret

关键 `call(3)`

```
:00409C4C 56  push esi
:00409C4D 89E6  mov esi, esp
:00409C4F 83EC10  sub esp, 00000010
:00409C52 31C9  xor ecx, ecx
:00409C54 52  push edx
:00409C55 31D2  xor edx, edx
:00409C57 E8A4FFFFFF  call 00409C00-------------------关键 `call(4)`，追入
:00409C5C 89F2  mov edx, esi
:00409C5E 58  pop eax
:00409C5F E848ADFFFF  call 004049AC
:00409C64 83C410  add esp, 00000010
:00409C67 5E  pop esi
:00409C68 C3  ret
```

关键 `call(4)`

```
:00409C00 08C9  or cl, cl
:00409C02 7517  jne 00409C1B
:00409C04 09C0  or eax, eax
:00409C06 790E  jns 00409C16
:00409C08 F7D8  neg eax
:00409C0A E807000000  call 00409C16--------------------关键 `call(5)`,追入
:00409C0F B02D  mov al, 2D
:00409C11 41  inc ecx
:00409C12 4E  dec esi
:00409C13 8806  mov byte ptr [esi], al
:00409C15 C3  ret
```

关键 `call(5)`

```
:00409C16 B90A000000  mov ecx, 0000000A  0xA-->ecx

`*` Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00409C02(C)
|
:00409C1B 52  push edx
:00409C1C 56  push esi

`*` Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00409C31(C)
|
:00409C1D 31D2  xor edx, edx  edx 清零
:00409C1F F7F1  div ecx\--------举例，如取我的硬盘 id"6ED363VT"第一位 6(0x36)
```



### 1. 除法运算逻辑

- **0x36(6)/0xA**：`eax = 0x5`, `edx = 0x4`
- **0x5/0xA**：`eax = 0x0`, `edx = 0x5`

```asm
:00409C21 4E          dec esi
:00409C22 80C230      add dl, 30
```

- `dl=0x34` (4)
- `dl=0x35` (5)

这两位连起来得到一个数`"54"`，余下的数依此类推；将硬盘`id "6ED363VT"`进行运算后，得到一组数`5469685154518684`。

### 2. 比较与跳转

```asm
:00409C25 80FA3A      cmp dl, 3A    ; 与 0x3A 比较
:00409C28 7203        jb 00409C2D   ; 小于就跳
:00409C2A 80C207      add dl, 07    ; 否则 dl+7
```

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**
`:00409C28(C)`

```asm
:00409C2D 8816        mov byte ptr [esi], dl
:00409C2F 09C0        or eax, eax   ; eax 为零吗？
:00409C31 75EA        jne 00409C1D  ; 没除尽就向上循环
:00409C33 59          pop ecx
:00409C34 5A          pop edx
:00409C35 29F1        sub ecx, esi
:00409C37 29CA        sub edx, ecx
:00409C39 7610        jbe 00409C4B
:00409C3B 01D1        add ecx, edx
:00409C3D B030        mov al, 30
:00409C3F 29D6        sub esi, edx
:00409C41 EB03        jmp 00409C46
```

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**
`:00409C47(C)`

```asm
:00409C43 880432      mov byte ptr [edx+esi], al
```

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**
`:00409C41(U)`

```asm
:00409C46 4A          dec edx
:00409C47 75FA        jne 00409C43
:00409C49 8806        mov byte ptr [esi], al
```

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**
`:00409C39(C)`

```asm
:00409C4B C3          ret
```

### 3. 关键 call 分析

**-------------------------------- 关键 call(6) --------------------------------**

```asm
:00404DDC 53          push ebx
:00404DDD 85C0        test eax, eax     ; eax 为零吗？（我这里）
                                     ; eax = "5469685154518684119122104495051"
:00404DDF 742D        je 00404E0E       ; 等于零跳（我这里不跳）
:00404DE1 8B58FC      mov ebx, dword ptr [eax-04] ; 字串的长度（我这里为 0x1F）--> ebx
:00404DE4 85DB        test ebx, ebx
:00404DE6 7426        je 00404E0E       ; 等于零跳（我这里不跳）
:00404DE8 4A         dec edx           ; edx=0x15-0x1=0x14（见上）
:00404DE9 7C1B        jl 00404E06       ; 小于零跳（我这里不跳）
:00404DEB 39DA        cmp edx, ebx      ; 0x14 : 0x1F
:00404DED 7D1F        jge 00404E0E      ; 大于等于跳（我这里不跳）
```

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**
`:00404E08(U)`

```asm
:00404DEF 29D3        sub ebx, edx      ; ebx=1F-14=0xB
:00404DF1 85C9        test ecx, ecx     ; ecx=0xA（见上）
:00404DF3 7C19        jl 00404E0E       ; 小于零跳（我这里不跳）
:00404DF5 39D9        cmp ecx, ebx      ; 0xA : 0xB
:00404DF7 7F11        jg 00404E0A       ; 大于等于跳（我这里不跳）
```

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**
`:00404E0C(U)`

```asm
:00404DF9 01C2        add edx, eax      ; 将字串"5469685154518684119122104495051"从第 edx 位开始赋给 edx，我这里为"22104495051"
:00404DFB 8B442408    mov eax, dword ptr [esp+08]
:00404DFF E8A8FBFFFF  call 004049AC
:00404E04 EB11        jmp 00404E17      ; 这里跳出去了
```

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**
`:00404DE9(C)`
*注：从这里开始，我都没有跟了，大家可以看看*

```asm
:00404E06 31D2        xor edx, edx      ; 从 00404DE9 跳来，edx 清零
:00404E08 EBE5        jmp 00404DEF      ; 跳上去循环
```

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**
`:00404DF7(C)`

```asm
:00404E0A 89D9        mov ecx, ebx      ; 从 00404DF7 跳来，ebx-->ecx
:00404E0C EBEB        jmp 00404DF9      ; 跳上去循环
```

**Referenced by a (U)nconditional or (C)onditional Jump at Addresses:**
`:00404DDF(C), :00404DE6(C), :00404DED(C), :00404DF3(C)`

```asm
:00404E0E 8B442408    mov eax, dword ptr [esp+08]
:00404E12 E8A5FAFFFF  call 004048BC
```

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**
`:00404E04(U)`

```asm
:00404E17 5B         pop ebx
:00404E18 C20400     ret 0004
```

### 总结

- **机器码**：`6ED363VT` (好眼熟耶，其实就是硬盘 id)
- **注册版本**：正式版
- **用户名**：`wzh123`
- **注册码**：`JLH-56655-SOFT-21490-WARE1-02111911-SL`
- **校验值**：`4dd466d0e1e0cc123afbe504da2462ca`
