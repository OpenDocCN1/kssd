
# 超级图标 V3.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6050.htm`](https://www.pediy.com/kssd/pediy06/pediy6050.htm)

软件大小： 2776 KB

软件语言： 简体中文

软件类别： 国产软件 / 共享版 / 图标工具

应用平台： `Win9x/NT/2000/XP`

软件介绍：

一个功能强大的集图标管理、搜索、处理、共享等于一体的强大的图标工具软件。
1. 图标库管理：采用特有的图标库文件格式，能够更快捷的管理、分类、检索、整理图标，多级的目录管理，使您的图标资源再多也能够一目了然。
2. 图标导入：您不仅可以从 `EXE`, `DLL`, `OCX`, `CPL`, `ICL`, `ICO`, `ANI`, `CUR` 等文件中导出图标，同时您也可以将 `JPG`, `BMP`, `PCX`, `PNG`, `TGA`, `EPS`, `TIF`, `PCD`, `DXF` 等图片转成图标，甚至您也可以通过截取屏幕图像来获得图标。
3. 图标导出：库中的图标可以另存为 `ICO`, `ICL`, `BMP`, `JPG`, `PCX`, `PNG`, `TIF`, `EPS` 等格式的文件。
4. 网络图标资源：通过不断更新的网络图标库列表、图标网站列表，您可以轻松获得海量图标资源。

作者申明：只是学习，无其他目的。

本人刚刚学破解，错误在所难免，写的也很乱，请各位包涵，也请各位高手指教。

这个软件是 `delphi` 编的，无壳，算法不难，但很繁琐，不料最后竟搞了个明码比较。

`name`：`wzh123`

`company`：`123`

`sn`：`1234－5678－9012－3456`

反编译后，很容易找到核心地方(以下的分析以我的注册信息为例)

```assembly
:00542624 8B8310030000           mov eax, dword ptr [ebx+00000310]
:0054262A E8F5D4F0FF            call 0044FB24
:0054262F 8B45E4                mov eax, dword ptr [ebp-1C]
:00542632 8D55E8                lea edx, dword ptr [ebp-18]
:00542635 E8026DECFF            call 0040933C
:0054263A FF75E8                push [ebp-18]
:0054263D 8D55DC                lea edx, dword ptr [ebp-24]
:00542640 8B8314030000          mov eax, dword ptr [ebx+00000314]
:00542646 E8D9D4F0FF            call 0044FB24
:0054264B 8B45DC                mov eax, dword ptr [ebp-24]
:0054264E 8D55E0                lea edx, dword ptr [ebp-20]
:00542651 E8E66CECFF            call 0040933C
:00542656 FF75E0                push [ebp-20]
:00542659 8D45FC                lea eax, dword ptr [ebp-04]
:0054265C BA04000000            mov edx, 00000004
:00542661 E8C629ECFF            call 0040502C
:00542666 8B832C030000          mov eax, dword ptr [ebx+0000032C]
:0054266C 8B4020                mov eax, dword ptr [eax+20]
:0054266F 8B55FC                mov edx, dword ptr [ebp-04]
:00542672 E8E5B1FEFF            call 0052D85C  ------->关键 call(1)，追入
:00542677 8B832C030000          mov eax, dword ptr [ebx+0000032C]
:0054267D 8B4020                mov eax, dword ptr [eax+20]
:00542680 80780900              cmp byte ptr [eax+09], 00
:00542684 7517                  jne 0054269D----------->一定要跳，不跳就死
:00542686 8D55D8                lea edx, dword ptr [ebp-28]

* Possible StringData Ref from Code Obj ->"Incomplete or incorrect registrationcode. "
                                   ->"Please input again."
                      |
:00542689 B89C275400            mov eax, 0054279C
:0054268E E8119AF6FF            call 004AC0A4
:00542693 8B45D8                mov eax, dword ptr [ebp-28]
:00542696 E83D62F0FF            call 004488D8
:0054269B EB79                  jmp 00542716
```

\--------------------关键 call(1) call `0052D85C`-------------------------------

```assembly
..........省略
:0052D8B6 8B45FC                mov eax, dword ptr [ebp-04]
:0052D8B9 E8A678EDFF            call 00405164
:0052D8BE 8B94BEA4020000        mov edx, dword ptr [esi+4*edi+000002A4]
:0052D8C5 8A1410                mov dl, byte ptr [eax+edx]
:0052D8C8 8D45F4                lea eax, dword ptr [ebp-0C]
:0052D8CB E8C475EDFF            call 00404E94
:0052D8D0 8B55F4                mov edx, dword ptr [ebp-0C]
:0052D8D3 8D4604                lea eax, dword ptr [esi+04]
:0052D8D6 E89976EDFF            call 00404F74
:0052D8DB 47                    inc edi
:0052D8DC 83FF04                cmp edi, 00000004
:0052D8DF 75D5                  jne 0052D8B6
:0052D8E1 33FF                  xor edi, edi
```

-------------------------取各组注册码的第一位，如我的为 `1`、`5`、`9`、`3`，连接成 `1593`

```assembly
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0052D90C(C)
|
:0052D8E3 8B45FC                mov eax, dword ptr [ebp-04]
:0052D8E6 E87978EDFF            call 00405164
:0052D8EB 8B94BEB4020000        mov edx, dword ptr [esi+4*edi+000002B4]
:0052D8F2 8A1410                mov dl, byte ptr [eax+edx]
:0052D8F5 8D45F0                lea eax, dword ptr [ebp-10]
:0052D8F8 E89775EDFF            call 00404E94
:0052D8FD 8B55F0                mov edx, dword ptr [ebp-10]
:0052D900 8D45F8                lea eax, dword ptr [ebp-08]
:0052D903 E86C76EDFF            call 00404F74
:0052D908 47                    inc edi
:0052D909 83FF0C                cmp edi, 0000000C
:0052D90C 75D5                  jne 0052D8E3
:0052D90E BF01000000            mov edi, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0052D957(C)
|
:0052D913 8D55EC                lea edx, dword ptr [ebp-14]
:0052D916 8B45FC                mov eax, dword ptr [ebp-04]
:0052D919 E8AEB7EDFF            call 004090CC
:0052D91E 8B45EC                mov eax, dword ptr [ebp-14]
:0052D921 50                    push eax
:0052D922 8D45E4                lea eax, dword ptr [ebp-1C]
:0052D925 50                    push eax
:0052D926 8BDF                  mov ebx, edi
:0052D928 8BCB                  mov ecx, ebx
:0052D92A 8B5604                mov edx, dword ptr [esi+04]
:0052D92D 8BC6                  mov eax, esi
:0052D92F E850000000            call 0052D984---------------->关键 call(2),算法，追入
:0052D934 8B45E4                mov eax, dword ptr [ebp-1C]
:0052D937 8D55E8                lea edx, dword ptr [ebp-18]
:0052D93A E88DB7EDFF            call 004090CC
:0052D93F 8B55E8                mov edx, dword ptr [ebp-18]
:0052D942 58                    pop eax
:0052D943 E86877EDFF            call 004050B0---------------->真假比较
:0052D948 7509                  jne 0052D953----------------->不等就跳
:0052D94A 885E08                mov byte ptr [esi+08], bl
:0052D94D C6460901              mov [esi+09], 01
:0052D951 EB06                  jmp 0052D959

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0052D948(C)
|
:0052D953 47                    inc edi\----------------------->跳到这里
:0052D954 83FF06                cmp edi, 00000006------------->表示有 6 个真码，这里我只算出一个
，其他的可以依次类推
:0052D957 75BA                  jne 0052D91------------------->跳上去循环
```

\-----------------------------关键 call(2),算法---------------------------------

```assembly
.......
:0052D9D0 8B45FC                mov eax, dword ptr [ebp-04]     "1593"-->eax
:0052D9D3 E88C77EDFF            call 00405164
:0052D9D8 8A1418                mov dl, byte ptr [eax+ebx]      依次取"1593"-->dl
:0052D9DB 8BC6                  mov eax, esi
:0052D9DD E85EFEFFFF            call 0052D840                    判断 1(0x31)在
"123...U"(0x31-0x55)中的位置，将位置-->eax,如果不在其中，则置 eax=0
:0052D9E2 0145F4                add dword ptr [ebp-0C], eax      将位置累加即
                                -------------->1+5+9+3=0x12-->ss:[0012EC8C]
:0052D9E5 43                    inc ebx
:0052D9E6 83FB04                cmp ebx, 00000004               "1593"取完否?
:0052D9E9 75E5                  jne 0052D9D0                    循环
:0052D9EB 33DB                  xor ebx, ebx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0052DA00(C)
|
:0052D9ED 8B45FC                mov eax, dword ptr [ebp-04]     "1593"-->eax
:0052D9F0 E86F77EDFF            call 00405164
:0052D9F5 0FB60418              movzx eax, byte ptr [eax+ebx]   依次"1593"-->dl
:0052D9F9 0145F0                add dword ptr [ebp-10], eax     累加->ss:[0012EC88]
                                ----0x31+0x35+0x39+0x33=0xD2
:0052D9FC 43                    inc ebx
:0052D9FD 83FB04                cmp ebx, 00000004              "1593"取完否?
```



:0052DA00 `75EB` `jne 0052D9ED` 循环

:0052DA02 `33DB` `xor ebx, ebx`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0052DAFE(C)

|

:0052DA04 `8B45FC` `mov eax, dword ptr [ebp-04]` "1593"-->`eax`

:0052DA07 `E85877EDFF` `call 00405164`

:0052DA0C `8A1418` `mov dl, byte ptr [eax+ebx]` "1"(0x361)-->`dl`

:0052DA0F `8BC6` `mov eax, esi`

:0052DA11 `E82AFEFFFF` `call 0052D840` 判断"1"(0x31)在 "123...U"(0x31-0x55)中的位置，将位置-->`eax`,如果不在其中，则置 `eax=0`

:0052DA16 `8945EC` `mov dword ptr [ebp-14], eax` 位置 `eax`->`ss:[0012EC84]`

------1、`1`->`ss:[0012EC84]`

------2、`5`->`ss:[0012EC84]`

------3、`9`->`ss:[0012EC84]`

------4、`3`->`ss:[0012EC84]`

:0052DA19 `8B45EC` `mov eax, dword ptr [ebp-14]` `ss:[0012EC84]`->`eax`

:0052DA1C `0345F4` `add eax, dword ptr [ebp-0C]`

---------------------`eax`+`ss:[0012EC8C]`->`eax`(见上)

:0052DA1F `B924000000` `mov ecx, 00000024` 24-->`ecx`

:0052DA24 `99` `cdq`

:0052DA25 `F7F9` `idiv ecx`

:0052DA27 `42` `inc edx` 余数+1

------1、`edx=0x14`

------2、`edx=0x18`

------3、`edx=0x1C`

------4、`edx=0x16`

:0052DA28 `8955E4` `mov dword ptr [ebp-1C], edx` `edx`-->`ss:[0012EC7C]`

:0052DA2B `8B45EC` `mov eax, dword ptr [ebp-14]` `ss:[0012EC84]`->`eax`

:0052DA2E `03C3` `add eax, ebx`

------1、`1+0`

------2、`5+1`

------3、`9+2`

------4、`3+3`

:0052DA30 `B90A000000` `mov ecx, 0000000A` 0xA-->`ecx`

:0052DA35 `99` `cdq`

:0052DA36 `F7F9` `idiv ecx`

:0052DA38 `8BFA` `mov edi, edx` 余数-->`edi`

------1、`edi=0x1`

------2、`edi=0x6`

------3、`edi=0x1`

------4、`edi=0x6`

:0052DA3A `8D45CC` `lea eax, dword ptr [ebp-34]` `ss:[0012EC64]`-->`eax`

:0052DA3D `6BD725` `imul edx, edi, 00000025` `edi*25`-->`edx`

------1、`edx=0x25`

------2、`edx=0xDE`

------3、`edx=0x25`

------4、`edx=0xDE`

:0052DA40 `8D1416` `lea edx, dword ptr [esi+edx]` 这里 `esi＝fff54c`, `[fff54c+edx]`->`edx`

\----------------------------------`esi` 指向内存为----------------------------------------

`00FFF54B` `EC D3 52 00 BC 55 00 01 01 00 73 64 66 6B 61` 煊 R.糢..sdfka B

`00FFF55B` `73 6D 76 63 78 32 31 32 33 6B 64 6B 73 32 6B 73` smvcx2123kdks2ks

`00FFF56B` `6B 61 39 32 38 34 33 37 73 6B 64 38 78 38 37 37` ka928437skd8x877

`00FFF57B` `32 38 39 33 32 6B 73 6B 64 6A 61 6B 73 6D 33 6B` 28932kskdjaksm3k

`00FFF58B` `32 39 38 64 38 73 6B 64 6A 32 39 38 33 39 32 39` 298d8skdj2983929

`00FFF59B` `30 31 6B 64 6B 73 6B 33 6B 32 38 64 39 73 30 39` 01kdksk3k28d9s09

`00FFF5AB` `63 6B 66 6C 6B 64 6B 73 39 32 30 61 6D 7A 6A 73` ckflkdks920amzjs

`00FFF5BB` `6A 6B 64 69 64 39 73 30 32 6B 32 33 78 63 73 6C` jkdid9s02k23xcsl

`00FFF5CB` `61 39 34 33 6B 64 6A 73 6B 78 61 7A 30 71 70 77` a943kdjskxaz0qpw

`00FFF5DB` `6F 69 79 30 77 69 73 61 73 70 71 30 73 39 78 31` oiy0wisaspq0s9x1

`00FFF5EB` `6B 64 6B 73 39 33 34 32 33 6A 61 6B 73 78 7A 78` kdks93423jaksxzx

`00FFF5FB` `6B 64 6C 73 39 30 33 32 39 33 6B 64 6B 73 6C 61` kdls903293kdksla

`00FFF60B` `30 73 64 64 38 32 37 33 6B 64 6A 73 6B 6D 7A 6E` 0sdd8273kdjskmzn

`00FFF61B` `78 6A 64 75 67 61 6C 73 6B 64 69 77 6B 64 6A 61` xjdugalskdiwkdja

`00FFF62B` `68 32 38 33 38 35 39 34 30 6B 6A 34 32 39 64 6A` h28385940kj429dj

`00FFF63B` `7A 6D 63 6E 33 69 64 39 73 6A 6B 61 6A 77 75 79` zmcn3id9sjkajwuy

`00FFF64B` `64 37 7A 68 78 31 39 73 38 64 30 32 6B 64 39 32` d7zhx19s8d02kd92

`00FFF65B` `38 64 6A 61 6B 78 6A 64 66 68 67 75 33 39 38 32` 8djakxjdfhgu3982

`00FFF66B` `79 61 6B 61 68 73 6B 61 68 7A 6E 78 68 73 75 38` yakahskahznxhsu8

`00FFF67B` `33 38 32 6A 78 6B 7A 78 75 64 39 73 30 31 6B 33` 382jxkzxud9s01k3

`00FFF68B` `6A 32 6B 64 6C 61 73 6A 64 6B 73 6C 62 6E 65 39` j2kdlasjdkslbne9

`00FFF69B` `32 30 39 64 73 6A 61 6C 39 32 37 38 64 75 61 6A` 209dsjal9278duaj

`00FFF6AB` `73 6B 64 6A 7A 78 6E 63 69 73 69 33 39 32 39 64` skdjzxncisi3929d

`00FFF6BB` `6A 73 6B 33 34 6A 6B 73 6B 73 6A 61 73 73 6B 64` jsk34jksksjasskd

`00FFF6CB` `6A 61 32 39 33 37 34 39 33 6A 64 6B 61 6B 64 6A` ja2937493jdkakdj

`00FFF6DB` `63 6D 63 7A 6E 78 68 63 6B 73 6B 65 33 39 38 31` cmcznxhckske3981

`00FFF6EB` `39 32 39 64 38 37 34 75 6A 61 6D 78 6E 63 62 61` 929d874ujamxncba

`00FFF6FB` `77 6A 31 64 64 6B 32 38 33 39 35 33 32 69 64 6B` wj1ddk2839532idk

`00FFF70B` `61 6B 73 64 6A 61 31 75 69 71 70 6C 78 6D 63 6E` aksdja1uiqplxmcn

`00FFF71B` `7A 39 33 38 32 6A 64 6E 6D 73 68 7A 6D 61 6C 71` z9382jdnmshzmalq

`00FFF72B` `6F 32 30 31 39 64 6A 78 6B 63 62 73 33 39 64 6A` o2019djxkcbs39dj

`00FFF73B` `61 6B 32 6A 66 75 61 73 6B 32 6D 7A 6E 63 62 68` ak2jfuask2mzncbh

`00FFF74B` `61 6B 78 70 7A 6C 73 6B 65 31 32 38 33 37 64 6A` akxpzlske12837dj

`00FFF75B` `78 38 64 39 73 6B 6A 79 75 72 65 6F 32 64 6B 63` x8d9skjyureo2dkc

`00FFF76B` `6A 32 39 34 30 73 6B 7A 6A 78 64 6B 32 33 39 61` j2940skzjxdk239a

`00FFF77B` `6A 78 7A 73 64 67 39 73 6A 65 75 69 71 6F 7A 61` jxzsdg9sjeuiqoza

`00FFF78B` `6C 78 6D 6E 76 62 61 71 31 33 6F 73 70 64 67 6B` lxmnvbaq13ospdgk

`00FFF79B` `64 6A 78 68 63 6B 61 77 33 64 6A 30 31 32 33 34` djxhckaw3dj01234

`00FFF7AB` `35 36 37 38 39 41 42 43 44 45 46 47 48 49 4A 4B` 56789ABCDEFGHIJK

`00FFF7BB` `4C 4D 4E 4F 50 51 52 53 54 55 56 57 58 59 5A 2D` LMNOPQRSTUVWXYZ-

`00FFF7CB` `44 55 42 30 59 56 52 48 34 54 2D 37 57 5A 4D 51` DUB0YVRH4T-7WZMQ

`00FFF7DB` `43 32 45 35 47 33 49 4A 41 4C 31 4E 50 46 38 53` C2E5G3IJAL1NPF8S

`00FFF7EB` `4B 4F 39 58 36 00 00 00 00 04 00 00 00 08 00 00` KO9X6.........

\------------------------------------------------------------------------------------------

:0052DA43 `8B4DE4` `mov ecx, dword ptr [ebp-1C]` `ss:[0012EC7C]`-->`ecx`

:0052DA46 `8A540A0A` `mov dl, byte ptr [edx+ecx+0A]`

------1、`0x14+0A+0x25=0x43[67(D)]`,取字符串的第 68 位， 即"8"->`dl`

------2、`0x18+0A+0xDE=0x100[256(D)]`,取字符串的第 257 位， 即"7"->`dl`

------3、`0x1C+0A+0x25=0x4B[75(D)]`,取字符串的第 76 位， 即"3"->`dl`

------4、`0x16+0A+0xDE=0xFE[254(D)]`,取字符串的第 255 位， 即"y"->`dl`

\-------------------------------------－－－－－－－－－－－－得到 8、7、3、y

:0052DA4A `E84574EDFF` `call 00404E94`

:0052DA4F `8B55CC` `mov edx, dword ptr [ebp-34]`

:0052DA52 `8D45E0` `lea eax, dword ptr [ebp-20]`

:0052DA55 `E81A75EDFF` `call 00404F74`

:0052DA5A `8B45FC` `mov eax, dword ptr [ebp-04]` "1593"-->`eax`

:0052DA5D `E80277EDFF` `call 00405164`

:0052DA62 `0FB60418` `movzx eax, byte ptr [eax+ebx]` 依次取"1593"-->`eax`



```
:0052DA66 8945E8                     mov dword ptr [ebp-18], eax      eax-->ss:[0012EC80]

:0052DA69 8B45E8                     mov eax, dword ptr [ebp-18]

:0052DA6C 0345F0                     add eax, dword ptr [ebp-10]      eax+ss\0012EC88\见上

                                       ------1、eax=0x31+0xD2=0x103

                                       ------2、eax=0x35+0xD2=0x107

                                       ------3、eax=0x39+0xD2=0x10B

                                       ------4、eax=0x33+0xD2=0x105

:0052DA6F B924000000                 mov ecx, 00000024                0x24-->ecx

:0052DA74 99                         cdq

:0052DA75 F7F9                       idiv ecx

:0052DA77 42                         inc edx                           余数+1

                                       ------1、edx=7+1=8

                                       ------2、edx=B+1=C

                                       ------3、edx=F+1=10

                                       ------4、edx=9+1=A

:0052DA78 8955E4                     mov dword ptr [ebp-1C], edx      edx-->ss:[0012EC7C]

:0052DA7B 8B45E8                     mov eax, dword ptr [ebp-18]      ss:\0012EC80\-->eax

:0052DA7E 03C3                       add eax, ebx

                                       ------1、0x31+0=0x31

                                       ------2、0x35+1=0x36

                                       ------3、0x39+2=0x3B

                                       ------4、0x33+3=0x36

:0052DA80 B90A000000                 mov ecx, 0000000A                0xA-->ecx

:0052DA85 99                         cdq

:0052DA86 F7F9                       idiv ecx

:0052DA88 8BFA                       mov edi, edx                      余数-->edi

                                       ------1、edi=9

                                       ------2、edi=4

                                       ------3、edi=9

                                       ------4、edi=4

:0052DA8A 8D45C8                     lea eax, dword ptr [ebp-38]

:0052DA8D 6BD725                     imul edx, edi, 00000025          edi*25-->edx

                                       ------1、edx=9*25=0x14D

                                       ------2、edx=4*25=0x94

                                       ------3、edx=9*25=0x14D

                                       ------4、edx=4*25=0x94

:0052DA90 8D1416                     lea edx, dword ptr [esi+edx]

:0052DA93 8B4DE4                     mov ecx, dword ptr [ebp-1C]      ss:\0012EC7C\-->ecx

:0052DA96 8A540A0A                   mov dl, byte ptr [edx+ecx+0A]

                                       ------1、0x8+0A+0x14D=0x15F[351(D)],取字符串的第 352 位，

                                                 即"s"->dl

                                       ------2、0xC+0A+0x94=0xAA[170(D)],取字符串的第 171 位，

                                                 即"k"->dl

                                       ------3、0x10+0A+0x14D=0x167[359(D)],取字符串的第 360 位，

                                                 即"i"->dl

                                       ------4、0xA+0A+0x94=0xA8[168(D)],取字符串的第 169 位，

                                                 即"j"->dl

-------------------------------------－－－－－－－－－－－－得到 s、k、i、j

:0052DA9A E8F573EDFF                 call 00404E94

:0052DA9F 8B55C8                     mov edx, dword ptr [ebp-38]

:0052DAA2 8D45E0                     lea eax, dword ptr [ebp-20]

:0052DAA5 E8CA74EDFF                 call 00404F74

:0052DAAA 8B45EC                     mov eax, dword ptr [ebp-14]      ss:\0012EC84\-->eax

:0052DAAD 0345F4                     add eax, dword ptr [ebp-0C]      eax+ss\0012C8C\

                                       ------1、eax=1+0x12=0x13

                                       ------2、eax=5+0x12=0x17

                                       ------3、eax=9+0x12=0x1B

                                       ------4、eax=3+0x12=0x15

:0052DAB0 33D2                       xor edx, edx

:0052DAB2 8A55FB                     mov dl, byte ptr [ebp-05]        ss:[0012EC93]-->dl

:0052DAB5 03C2                       add eax, edx

                                       ------1、eax=13+1=0x14

                                       ------2、eax=17+1=0x18

                                       ------3、eax=1B+1=0x1C

                                       ------4、eax=15+1=0x16

:0052DAB7 B924000000                 mov ecx, 00000024                24-->ecx

:0052DABC 99                         cdq

:0052DABD F7F9                       idiv ecx

:0052DABF 42                         inc edx                           余数+1

                                       ------1、edx=14+1=15

                                       ------2、edx=18+1=19

                                       ------3、edx=1C+1=1D

                                       ------4、edx=16+1=17

:0052DAC0 8955E4                     mov dword ptr [ebp-1C], edx      edx-->ss:[0012EC7C]

:0052DAC3 8B45E8                     mov eax, dword ptr [ebp-18]      ss:\0012EC80\-->eax

:0052DAC6 0345F0                     add eax, dword ptr [ebp-10]      eax+ss:\0012EC88\(见上)

                                       ------1、0x31+0xD2=0x103

                                       ------2、0x35+0xD2=0x107

                                       ------3、0x39+0xD2=0x10B

                                       ------4、0x33+0xD2=0x105

:0052DAC9 33D2                       xor edx, edx

:0052DACB 8A55FB                     mov dl, byte ptr [ebp-05]

:0052DACE 03C2                       add eax, edx

                                       ------1、eax=0x103+1=0x104

                                       ------2、eax=0x107+1=0x108

                                       ------3、eax=0x10B+1=0x10C

                                       ------4、eax=0x105+1=0x106

:0052DAD0 B90A000000                 mov ecx, 0000000A                0xA-->ecx

:0052DAD5 99                         cdq

:0052DAD6 F7F9                       idiv ecx

:0052DAD8 8BFA                       mov edi, edx                      余数-->eax

                                       ------1、edi=0

                                       ------2、edi=4

                                       ------3、edi=8

                                       ------4、edi=2

:0052DADA 8D45C4                     lea eax, dword ptr [ebp-3C]

:0052DADD 6BD725                     imul edx, edi, 00000025          edi*25-->edx

                                       ------1、edx=0*25=0

                                       ------2、edx=4*25=0x94

                                       ------3、edx=8*25=0x128

                                       ------4、edx=2*25=0x4A

:0052DAE0 8D1416                     lea edx, dword ptr [esi+edx]

:0052DAE3 8B4DE4                     mov ecx, dword ptr [ebp-1C]      ss:\0012EC7C\-->ecx

:0052DAE6 8A540A0A                   mov dl, byte ptr [edx+ecx+0A]

                                       ------1、0x15+0A+0=0x1F[31(D)],取字符串的第 32 位，

                                                 即"k"->dl

                                       ------2、0x19+0A+0x94=0xB7[183(D)],取字符串的第 184 位，

                                                 即"9"->dl

                                       ------3、0x1D+0A+0x128=0x14F[335(D)],取字符串的第 336 位，

                                                 即"2"->dl

                                       ------4、0x17+0A+0x4A=0x6B[107(D)],取字符串的第 108 位，

                                                 即"m"->dl

-------------------------------------－－－－－－－－－－－－得到 k、9、2、m

:0052DAEA E8A573EDFF                 call 00404E94

:0052DAEF 8B55C4                     mov edx, dword ptr [ebp-3C]

:0052DAF2 8D45E0                     lea eax, dword ptr [ebp-20]

:0052DAF5 E87A74EDFF                 call 00404F74

:0052DAFA 43                         inc ebx

:0052DAFB 83FB04                     cmp ebx, 00000004

:0052DAFE 0F8500FFFFFF               jne 0052DA04                    循环

:0052DB04 33DB                       xor ebx, ebx
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0052DB20(C)

|

```
:0052DB06 8B45FC                     mov eax, dword ptr [ebp-04]      "1593"-->eax

:0052DB09 E85676EDFF                 call 00405164
```



# 汇编代码分析

:0052DB0E `8A0418` `mov al, byte ptr [eax+ebx]`  
依次取 `"1593"` --> `eax`  

:0052DB11 `8B949EA4020000` `mov edx, dword ptr [esi+4*ebx+000002A4]`  

:0052DB18 `884415D0` `mov byte ptr [ebp+edx-30], al`  

:0052DB1C `43` `inc ebx`  

:0052DB1D `83FB04` `cmp ebx, 00000004`  

:0052DB20 `75E4` `jne 0052DB06`  

:0052DB22 `33DB` `xor ebx, ebx` 

* Referenced by a `(U)nconditional` or `(C)onditional` Jump at Address:  
`:0052DB3E`(`C`)

:0052DB24 `8B45E0` `mov eax, dword ptr [ebp-20]` `"8sk7k93i2yjm"`(上面所求得)-->`eax`  

排列规律：
```
873y
skij
k92m
```
按列顺序取数排列，也可以说按字符产生先后顺序排列  

:0052DB27 `E83876EDFF` `call 00405164`  

:0052DB2C `8A0418` `mov al, byte ptr [eax+ebx]`  

:0052DB2F `8B949EB4020000` `mov edx, dword ptr [esi+4*ebx+000002B4]`  

:0052DB36 `884415D0` `mov byte ptr [ebp+edx-30], al`  

:0052DB3A `43` `inc ebx`  

:0052DB3B `83FB0C` `cmp ebx, 0000000C`  

:0052DB3E `75E4` `jne 0052DB24` ------------->将`"8sk7k93i2yjm"`按规律排列为  
`"9ijm87s32ykk"`，规律为：  
第 1 位->第 5 位, 第 2 位->第 7 位, 第 3 位->第 12 位,  
第 4 位->第 6 位, 第 5 位->第 11 位, 第 6 位->第 1 位,  
第 7 位->第 8 位, 第 8 位->第 2 位, 第 9 位->第 9 位,  
第 10 位->第 10 位, 第 11 位->第 3 位, 第 12 位->第 4 位,  

:0052DB40 `8D45C0` `lea eax, dword ptr [ebp-40]`  

:0052DB43 `8D55D0` `lea edx, dword ptr [ebp-30]`  

:0052DB46 `B910000000` `mov ecx, 00000010`  

:0052DB4B `E8CC73EDFF` `call 00404F1C`------------->将 `1593` 分别插入上述字符串中  

:0052DB50 `8B45C0` `mov eax, dword ptr [ebp-40]` `"19ij5m879s323ykk"`-->`eax`,即将 `1593`  
与上面求得的字符串连接，可以看成 `19ij－5m87－9s32－3ykk`,即为真码  

:0052DB53 `8B5508` `mov edx, dword ptr [ebp+08]`  

:0052DB56 `E871B5EDFF` `call 004090CC`  

:0052DB5B `33C0` `xor eax, eax`  

:0052DB5D `5A` `pop edx`  

:0052DB5E `59` `pop ecx`  

:0052DB5F `59` `pop ecx`  

:0052DB60 `648910` `mov dword ptr fs:[eax], edx`  

:0052DB63 `688DDB5200` `push 0052DB8D` 

* Referenced by a `(U)nconditional` or `(C)onditional` Jump at Address:  
`:0052DB8B`(`U`)

:0052DB68 `8D45C0` `lea eax, dword ptr [ebp-40]`  

:0052DB6B `BA04000000` `mov edx, 00000004`  

:0052DB70 `E86371EDFF` `call 00404CD8`  

:0052DB75 `8D45E0` `lea eax, dword ptr [ebp-20]`  

:0052DB78 `E83771EDFF` `call 00404CB4`  

:0052DB7D `8D45FC` `lea eax, dword ptr [ebp-04]`  

:0052DB80 `E82F71EDFF` `call 00404CB4`  

:0052DB85 `C3` `ret` 

:0052DB86 `E9A96AEDFF` `jmp 00404634`  

:0052DB8B `EBDB` `jmp 0052DB68`  

:0052DB8D `5F` `pop edi`  

:0052DB8E `5E` `pop esi`  

:0052DB8F `5B` `pop ebx`  

:0052DB90 `8BE5` `mov esp, ebp`  

:0052DB92 `5D` `pop ebp`  

:0052DB93 `C20400` `ret 0004` 

# 注册码逻辑总结

所以注册码与姓名、公司无关，注册码格式 `s1s2s3s4-s5s6s7s8-s9s10s11s12-s13s14s15s16`  

`s1`,`s5`,`s9`,`s13` 任意，并推出其他各位，且对于任一个 `s1s5s9s13`,有 6 个正确的注册码  

给一个可用的注册码：`19IJ－5M87－9S32－3YKK`  

`4dd466d0e1e0cc123afbe504da2462ca`
