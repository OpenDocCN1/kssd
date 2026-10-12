
#### :0040FC5E(C), :0040FC74(C), :0040FC8A(C), :0040FCA0(C), :0040FCB6(C)

:0040FCCE `C786C400000000000000` mov `dword ptr [esi+000000C4], 00000000`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0040FCCC(C)

:0040FCD8 `8B9424E4000000` mov `edx, dword ptr [esp+000000E4]`
====> 用户名 `fly` 移入 `EDX`

:0040FCDF `33C9` xor `ecx, ecx`

:0040FCE1 `53` push `ebx`

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
===> 下面这段代码是把 `[esp+10]` 依次处放入 “`huydong`” 字符串！

:0040FCE2 `C644240868` mov `[esp+08], 68`

:0040FCE7 `8B72F8` mov `esi, dword ptr [edx-08]`
====> 用户名长度送 `esi=3`

:0040FCEA `C644240975` mov `[esp+09], 75`

:0040FCEF `85F6` test `esi, esi`

:0040FCF1 `C644240A79` mov `[esp+0A], 79`

:0040FCF6 `C644240B64` mov `[esp+0B], 64`

:0040FCFB `C644240C6F` mov `[esp+0C], 6F`

:0040FD00 `C644240D6E` mov `[esp+0D], 6E`

:0040FD05 `C644240E67` mov `[esp+0E], 67`

:0040FD0A `C644240F00` mov `[esp+0F], 00`

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

:0040FD0F `7E3F` jle `0040FD50`

:0040FD11 `55` push `ebp`

:0040FD12 `57` push `edi`

:0040FD13 `8D7C3417` lea `edi, dword ptr [esp+esi+17]`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0040FD4C(C)

====> 以下就是运算核心了！

:0040FD17 `8B8424F0000000` mov `eax, dword ptr [esp+000000F0]`
====> `fly` 移入 `EAX`

:0040FD1E `BD07000000` mov `ebp, 00000007`

:0040FD23 `8A1C01` mov `bl, byte ptr [ecx+eax]`
====> 依次取用户名。
====> 1、？`BL=66` 即 `f` 的 `HEX` 值
====> 2、？`BL=6C` 即 `l` 的 `HEX` 值
====> 3、？`BL=79` 即 `y` 的 `HEX` 值

:0040FD26 `8BC1` mov `eax, ecx`

:0040FD28 `99` cdq

:0040FD29 `F7FD` idiv `ebp`

:0040FD2B `0FBEC3` movsx `eax, bl`
====> 1、？`EAX=66` 即 `f` 的 `HEX` 值
====> 2、？`EAX=6C` 即 `l` 的 `HEX` 值
====> 3、？`EAX=79` 即 `y` 的 `HEX` 值

:0040FD2E `8BD9` mov `ebx, ecx`
====> 1、？`EBX=0`
====> 2、？`EBX=1`
====> 3、？`EBX=2`

:0040FD30 `0FBE541410` movsx `edx, byte ptr [esp+edx+10]`
====> 依次从 “`huydong`” 字符串中取字符入 `EDX`
====> 1、？`EDX=68` 即 `h` 的 `HEX` 值
====> 2、？`EDX=75` 即 `u` 的 `HEX` 值
====> 3、？`EDX=79` 即 `y` 的 `HEX` 值

:0040FD35 `03DA` add `ebx, edx`
====> 1、`EBX=0+68=68`
====> 2、`EBX=1+75=76`
====> 3、`EBX=2+79=7B`

:0040FD37 `03C3` add `eax, ebx`
====> 1、`EAX=66+68=CE`
====> 2、`EAX=6C+76=E2`
====> 3、`EAX=79+7B=F4`

:0040FD39 `BB09000000` mov `ebx, 00000009`
====> `9` 送 `ebx`

:0040FD3E `03C6` add `eax, esi`
====> `esi` 是用户名长度
====> 1、`EAX=CE+3=D1`
====> 2、`EAX=E2+3=E5`
====> 3、`EAX=F4+3=F7`

:0040FD40 `99` cdq

:0040FD41 `F7FB` idiv `ebx`
====> `EAX` 依次除以 `9`
====> 1、`EAX=D1/9=17` 余 `2`
====> 2、`EAX=E5/9=19` 余 `4`
====> 3、`EAX=F7/9=1B` 余 `4`

:0040FD43 `80C230` add `dl, 30`
====> 余数入 `DL`，依次加 `30`
====> 1、`DL=2+30=32`
====> 2、`DL=4+30=34`
====> 3、`DL=4+30=34`

:0040FD46 `41` inc `ecx`
====> `ecx` 依次增 `1`

:0040FD47 `8817` mov `byte ptr [edi], dl`
====> `DL->[edi]`
====> 循环 `3` 次后，`D EDI=442`
****这是真码的前 `3` 个数！！！

:0040FD49 `4F` dec `edi`

:0040FD4A `3BCE` cmp `ecx, esi`
====> 比较用户名是否取完

:0040FD4C `7CC9` jl `0040FD17`
====> 没有取完，跳上去继续循环
====> 共循环 `3` 次。

:0040FD4E `5F` pop `edi`

:0040FD4F `5D` pop `ebp`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0040FD0F(C)

:0040FD50 `8D464D` lea `eax, dword ptr [esi+4D]`
====> ？`ESI=3`
实际上是用户名长度加上 `4D` 的结果送 `eax`
====> 过此 ？`EAX=50`

:0040FD53 `B909000000` mov `ecx, 00000009`
====> `9` 送 `ecx`

:0040FD58 `99` cdq

:0040FD59 `F7F9` idiv `ecx`
====> `EAX/9=8` 余 `8`
====> 余数 `8` 入 `DL`

:0040FD5B `8B8424EC000000` mov `eax, dword ptr [esp+000000EC]`
====> 试炼码送 `eax`

:0040FD62 `80C230` add `dl, 30`
====> `DL=8+30=38`
****这是真码的最后 `1` 个数！！！

:0040FD65 `88543410` mov `byte ptr [esp+esi+10], dl`
====> `DL` 移入 `[esp+17]` 处

:0040FD69 `C644341100` mov `[esp+esi+11], 00`

:0040FD6E `8D742410` lea `esi, dword ptr [esp+10]`
====> 真正的注册码送 `ESI`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0040FD94(C)

====> 这里向下是将真假注册码逐位的进行比较，一个经典的组合！

:0040FD72 `8A10` mov `dl, byte ptr [eax]`
====> `D EAX=试炼码`

:0040FD74 `8A1E` mov `bl, byte ptr [esi]`
====> `D ESI=真码！！！！`

:0040FD76 `8ACA` mov `cl, dl`

:0040FD78 `3AD3` cmp `dl, bl`

:0040FD7A `751E` jne `0040FD9A`

:0040FD7C `84C9` test `cl, cl`

:0040FD7E `7416` je `0040FD96`

:0040FD80 `8A5001` mov `dl, byte ptr [eax+01]`

:0040FD83 `8A5E01` mov `bl, byte ptr [esi+01]`

:0040FD86 `8ACA` mov `cl, dl`

:0040FD88 `3AD3` cmp `dl, bl`

:0040FD8A `750E` jne `0040FD9A`

:0040FD8C `83C002` add `eax, 00000002`

:0040FD8F `83C602` add `esi, 00000002`

:0040FD92 `84C9` test `cl, cl`

:0040FD94 `75DC` jne `0040FD72`

—————————————————————————————

### 【KeyMake 之内存注册机】：

中断地址：`40F287`

中断次数：`1`

第一字节：`E8`

指令长度：`5`

中断地址：`40FD74`

中断次数：`1`

第一字节：`A8`

指令长度：`2`

内存方式：`ESI`

—————————————————————————————

### 【注册信息保存】：

```
REGEDIT4

[HKEY_CURRENT_USER\Software\SeaMoonTech\EZ Extract Resource\Settings]  
"SearchID2"="4428"

[HKEY_CURRENT_USER\Software\SeaMoonTech\EZ Extract Resource\RegInfo]  
"RegUserName"="fly"
```

—————————————————————————————

### 【整理】：

`Registartion Name`：`fly`

`Registartion Code`：`4428`

—————————————————————————————

Cracked By 巢水工作坊——`fly`【`OCN`】
