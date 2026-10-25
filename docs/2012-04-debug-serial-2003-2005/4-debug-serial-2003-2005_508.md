
# 网络电话 Talking anywhere 5.0

> 原文：[`www.pediy.com/kssd/pediy05/pediy50485.htm`](https://www.pediy.com/kssd/pediy05/pediy50485.htm)

简单算法——网络电话 `Talking anywhere` 5.0

下载页面：`http://www.softreg.com.cn/shareware\_view.asp?id=/D0DCF113-BAEA-41A8-B5F9-2A6B594C6D12/`

最新版本：`5.0v`

软件大小：`204K`

下载次数：共 `166` 次

适用平台：`WIN9x`, `Win2000`, `WinNT`

作者主页：[`www.zjjs.net/iptalk/`](http://www.zjjs.net/iptalk/)

【软件简介】：`Talking anywhere` 是绿色通讯软件，双方都运行后，一方输入对方 IP 地址即可进行通话。低带宽，高清晰语音！最大特点在于语音清晰，无同类网络电话的复杂性和通语延时。采用 `MPEG2` 高压缩比，及点对点通讯技术！无需服务器支持！如果你好用的话，请送给你的好朋友使用！如果不清楚你的 IP 地址或对方 IP 地址的话，可以登陆我们的帮助站点：

[`www.zjjs.net/iptalk/`](http://www.zjjs.net/iptalk/)

[`www.zjjs.net/iptalk/help.htm`](http://www.zjjs.net/iptalk/help.htm)

【软件限制】：`NAG`

【作者声明】：初学 `Crack`，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：`TRW2000` 娃娃修改版、`Ollydbg1.09`、`FI2.5`、`W32Dasm8.93` 黄金版

—————————————————————————————————

### 【过 程】

这个软件是 `ljp` 朋友提出来的。试了试，应该算是明码比较了。只是仿佛有点 `VB` 的味道。

`iptalk.exe` 用 `FI` 看不认识。`C++` 编写。好了，直接调试吧。可以拦截出错提示到达下面的地址。

系统 ID： `95065`

姓 名： `fly`

公 司：`【OCN】`

试炼码： `13572468`

—————————————————————————————————

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00492D20(C)

|

:00492D2C `8B3DF4EB4900` mov edi, dword ptr \[0049EBF4\]

:00492D32 `83C9FF` or ecx, FFFFFFFF

:00492D35 `33C0` xor eax, eax

:00492D37 `F2` repnz

:00492D38 `AE` scasb

:00492D39 `8B3DD4EB4900` mov edi, dword ptr \[0049EBD4\]

:00492D3F `F7D1` not ecx

:00492D41 `49` dec ecx

:00492D42 `8D740C20` lea esi, dword ptr \[esp+ecx+20\]

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;====>ESI\=13572468

:00492D46 `83C9FF` or ecx, FFFFFFFF

:00492D49 `F2` repnz

:00492D4A `AE` scasb

:00492D4B `F7D1` not ecx

:00492D4D `49` dec ecx

:00492D4E `8BD6` mov edx, esi

:00492D50 `2BD1` sub edx, ecx

:00492D52 `8BFE` mov edi, esi

:00492D54 `83C9FF` or ecx, FFFFFFFF

:00492D57 `F2` repnz

:00492D58 `AE` scasb

:00492D59 `F7D1` not ecx

:00492D5B `49` dec ecx

:00492D5C `88040A` mov byte ptr \[edx+ecx\], al

:00492D5F `8BCE` mov ecx, esi

:00492D61 `E84A5B0000` call 004988B0

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;====>检测输入的密码是否是数字？

:00492D66 `85C0` test eax, eax

:00492D68 `750A` jne 00492D74

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;====>不跳则 OVER！

:00492D6A `B8FDFFFFFF` mov eax, FFFFFFFD

:00492D6F `E9F9000000` jmp 00492E6D

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00492D68(C)

|

:00492D74 `BA64B14900` mov edx, 0049B164

:00492D79 `8BCE` mov ecx, esi

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;====>13572468 入 ECX

:00492D7B `E8705B0000` call 004988F0

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;====>此 CALL 将 13572468 转换为 16 进制值表示

:00492D80 `8BF8` mov edi, eax

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;====>EDI\=EAX\=CF1974（H）=13572468（D）

:00492D82 `66A154EC4900` mov ax, word ptr \[0049EC54\]

:00492D88 `663D0100` cmp ax, 0001

:00492D8C `7546` jne 00492DD4

:00492D8E `66A1FAEB4900` mov ax, word ptr \[0049EBFA\]

:00492D94 `8B1500EC4900` mov edx, dword ptr \[0049EC00\]

:00492D9A `33C9` xor ecx, ecx

:00492D9C `8ACC` mov cl, ah

:00492D9E `25FF000000` and eax, 000000FF

:00492DA3 `8BF1` mov esi, ecx

:00492DA5 `8BC8` mov ecx, eax

:00492DA7 `E854FBFFFF` call 00492900

:00492DAC `8B15FCEB4900` mov edx, dword ptr \[0049EBFC\]

:00492DB2 `03F8` add edi, eax

:00492DB4 `6685F6` test si, si

:00492DB7 `7504` jne 00492DBD

:00492DB9 `33C9` xor ecx, ecx

:00492DBB `EB03` jmp 00492DC0

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00492DB7(C)

|

:00492DBD `8D4E01` lea ecx, dword ptr \[esi+01\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00492DBB(U)

|

:00492DC0 `E83BFBFFFF` call 00492900

:00492DC5 `8BC8` mov ecx, eax

:00492DC7 `85C9` test ecx, ecx

:00492DC9 `7438` je 00492E03

:00492DCB `8BC7` mov eax, edi

:00492DCD `99` cdq

:00492DCE `F7F9` idiv ecx

:00492DD0 `8BC2` mov eax, edx

:00492DD2 `EB27` jmp 00492DFB

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00492D8C(C)

|

:00492DD4 `663D0200` cmp ax, 0002

:00492DD8 `7529` jne 00492E03

:00492DDA `8B15FCEB4900` mov edx, dword ptr \[0049EBFC\]

:00492DE0 `A100EC4900` mov eax, dword ptr \[0049EC00\]

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;====>EAX\=fly

:00492DE5 `8B0D38ED4900` mov ecx, dword ptr \[0049ED38\]

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;====>ECX\=17359（H）=95605（D） 即：系统 ID 号

:00492DEB `52` push edx

:00492DEC `668B15FAEB4900` mov dx, word ptr \[0049EBFA\]

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;====>DX\=6502 此处的 6502 是如何得出的？烦请指教！

:00492DF3 `50` push eax

:00492DF4 `E897FBFFFF` call 00492990

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;====>算法！进入！此 CALL 运算得出 EAX=212DB

:00492DF9 `2BC7` sub eax, edi

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;====>EAX\=212DB-CF1974=FF32F967（H）=-13436569（D）

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;呵呵，如果此处相等即 OK！所以真码=EAX\=212DB（H）=135899（D）

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00492DD2(U)

|

:00492DFB `85C0` test eax, eax

:00492DFD `0F848E000000` je 00492E91

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;====>应跳！

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00492DC9(C), :00492DD8(C)

|

:00492E03 `B8FBFFFFFF` mov eax, FFFFFFFB

:00492E08 `EB63` jmp 00492E6D

下面应该是保存注册信息了。~Q~~Q~~Q~~Q~

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00492C71(C), :00492C7A(C), :00492DFD(C), :00492E1C(C), :00492E20(C)

|:00492E68(C)

|

:00492E91 `668B0DCEEB4900` mov cx, word ptr \[0049EBCE\]

:00492E98 `66394C2410` cmp word ptr \[esp+10\], cx

:00492E9D `0F8D16010000` jnl 00492FB9

:00492EA3 `8B0D04EF4900` mov ecx, dword ptr \[0049EF04\]

:00492EA9 `B800002000` mov eax, 00200000

:00492EAE `85C8` test eax, ecx

:00492EB0 `7518` jne 00492ECA

:00492EB2 `8BD1` mov edx, ecx

:00492EB4 `8B0D70EE4900` mov ecx, dword ptr \[0049EE70\]

:00492EBA `0BD0` or edx, eax

:00492EBC `33C8` xor ecx, eax

:00492EBE `891504EF4900` mov dword ptr \[0049EF04\], edx

:00492EC4 `890D70EE4900` mov dword ptr \[0049EE70\], ecx

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00492EB0(C)

|

:00492ECA `8B742410` mov esi, dword ptr \[esp+10\]

:00492ECE `BA01000000` mov edx, 00000001

:00492ED3 `8BCE` mov ecx, esi

:00492ED5 `E816F7FFFF` call 004925F0

:00492EDA `85C0` test eax, eax

:00492EDC `0F8493000000` je 00492F75

:00492EE2 `8B0D04EF4900` mov ecx, dword ptr \[0049EF04\]

:00492EE8 `8B2D70EE4900` mov ebp, dword ptr \[0049EE70\]

:00492EEE `0FBFD6` movsx edx, si



:00492EF1 `8B742418`                `mov esi, dword ptr [esp+18]`

:00492EF5 `B8FFFFDFFF`             `mov eax, FFDFFFFF`

:00492EFA `23C8`                   `and ecx, eax`

:00492EFC `33E8`                   `xor ebp, eax`

:00492EFE `A1ECEB4900`            `mov eax, dword ptr [0049EBEC]`

:00492F03 `890D04EF4900`          `mov dword ptr [0049EF04], ecx`

:00492F09 `C1E206`                `shl edx, 06`

:00492F0C `892D70EE4900`          `mov dword ptr [0049EE70], ebp`

:00492F12 `6810100000`            `push 00001010`

:00492F17 `8B4C0238`              `mov ecx, dword ptr [edx+eax+38]`

:00492F1B `685CB14900`            `push 0049B15C`

:00492F20 `51`                    `push ecx`

:00492F21 `56`                    `push esi`

:00492F22 `FF1500F24900`          `call dword ptr [0049F200]`

:00492F28 `6801100000`            `push 00001001`

:00492F2D `56`                    `push esi`

:00492F2E `FF15B8F14900`          `call dword ptr [0049F1B8]`

:00492F34 `8BC8`                  `mov ecx, eax`

:00492F36 `E8B5EAFFFF`            `call 004919F0`

:00492F3B `8B1528ED4900`          `mov edx, dword ptr [0049ED28]`

:00492F41 `33F6`                  `xor esi, esi`

:00492F43 `56`                    `push esi`

:00492F44 `6802800000`            `push 00008002`

:00492F49 `6811010000`            `push 00000111`

:00492F4E `52`                    `push edx`

:00492F4F `6689358EEC4900`        `mov word ptr [0049EC8E], si`

:00492F56 `6689359AB24900`        `mov word ptr [0049B29A], si`

:00492F5D `FF1508F24900`          `call dword ptr [0049F208]`

:00492F63 `66893526ED4900`        `mov word ptr [0049ED26], si`

:00492F6A `5F`                    `pop edi`

:00492F6B `5E`                    `pop esi`

:00492F6C `5D`                    `pop ebp`

:00492F6D `5B`                    `pop ebx`

:00492F6E `81C4AC000000`          `add esp, 000000AC`

:00492F74 `C3`                    `ret`

---

### 进入算法 CALL

地址：`492DF4` 指令：`call 00492990`

* Referenced by a CALL at Addresses:
`00492DF4` , `004931DA`

:00492990 `53`                    `push ebx`

:00492991 `56`                    `push esi`

:00492992 `668BDA`               `mov bx, dx`
> `BX` = `DX` = `6502`

:00492995 `8BF1`                  `mov esi, ecx`
> `ESI` = `17359`

:00492997 `8B54240C`              `mov edx, dword ptr [esp+0C]`
> `EDX` = `fly`

:0049299B `8ACB`                  `mov cl, bl`
> `CL` = `BL` = `02`

:0049299D `57`                    `push edi`

:0049299E `81E1FF000000`          `and ecx, 000000FF`
> `ECX` = `17302` & `000000FF` = `02`
> 注：此处不明，`17302` 是如何得出的？请各位老师指教。

:004929A4 `E857FFFFFF`            `call 00492900`
> 进入子运算 CALL 1，得出下面的 `EAX` = `9159`

:004929A9 `8B542414`              `mov edx, dword ptr [esp+14]`

:004929AD `8BF8`                  `mov edi, eax`
> `EDI` = `EAX` = `9159`

:004929AF `33C0`                  `xor eax, eax`

:004929B1 `8AC7`                  `mov al, bh`
> `AL` = `BH` = `65`
> 注：此处不明。

:004929B3 `6685C0`                `test ax, ax`

:004929B6 `7512`                  `jne 004929CA`
> 跳下去继续运算！

:004929B8 `33C9`                  `xor ecx, ecx`

:004929BA `E841FFFFFF`            `call 00492900`

:004929BF `03FE`                  `add edi, esi`

:004929C1 `0FAFC7`                `imul eax, edi`

:004929C4 `5F`                    `pop edi`

:004929C5 `5E`                    `pop esi`

:004929C6 `5B`                    `pop ebx`

:004929C7 `C20800`                `ret 0008`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`004929B6` (C)

:004929CA `8D4801`                `lea ecx, dword ptr [eax+01]`
> `ECX` = `66`

:004929CD `E82EFFFFFF`            `call 00492900`
> 进入子运算 CALL 2，得出下面的 `EAX` 值！

:004929D2 `03C7`                  `add eax, edi`
> `EAX` = `E29` + `9159` = `9F82`

:004929D4 `03C6`                  `add eax, esi`
> `EAX` = `9F82` + `17359` = `212DB`

:004929D6 `5F`                    `pop edi`

:004929D7 `5E`                    `pop esi`

:004929D8 `5B`                    `pop ebx`

:004929D9 `C20800`                `ret 0008`

---

1. 进入子运算 CALL 1：`4929A4` `call 00492900`
2. 进入子运算 CALL 2：`4929CD` `call 00492900`

* Referenced by a CALL at Addresses:
`004929A4` , `004929BA` , `004929CD` , `00492DA7` , `00492DC0`

:00492900 `53`                    `push ebx`

:00492901 `8BDA`                  `mov ebx, edx`

:00492903 `56`                    `push esi`

:00492904 `8BF1`                  `mov esi, ecx`

:00492906 `85DB`                  `test ebx, ebx`

:00492908 `7472`                  `je 0049297C`

:0049290A `803B00`                `cmp byte ptr [ebx], 00`

:0049290D `746D`                  `je 0049297C`

:0049290F `57`                    `push edi`

:00492910 `8BFB`                  `mov edi, ebx`

:00492912 `83C9FF`                `or ecx, FFFFFFFF`

:00492915 `33C0`                  `xor eax, eax`

:00492917 `F2`                    `repnz`

:00492918 `AE`                    `scasb`

:00492919 `F7D1`                  `not ecx`

:0049291B `49`                    `dec ecx`
> `ECX` = `3`，上面代码是取用户名的位数

:0049291C `6685F6`                `test si, si`

:0049291F `7443`                  `je 00492964`

:00492921 `6683FE01`              `cmp si, 0001`

:00492925 `743D`                  `je 00492964`

:00492927 `81E6FFFF0000`          `and esi, 0000FFFF`

:0049292D `8BC6`                  `mov eax, esi`
> 1. `EAX` = `ESI` = `2`
> 2. `EAX` = `ESI` = `66`

:0049292F `99`                    `cdq`

:00492930 `F7F9`                  `idiv ecx`
> 1. `EDX` = `2` % `3` = `2`
> 2. `EDX` = `66` % `3` = `0`

:00492932 `0FBE041A`              `movsx eax, byte ptr [edx+ebx]`
> 依据 `EDX` 值从 `fly` 中取字符值
> 1. `EAX` = `79`
> 2. `EAX` = `66`

:00492936 `0FAFC6`                `imul eax, esi`
> 1. `EAX` = `79` * `02` = `F2`
> 2. `EAX` = `66` * `66` = `28A4`

:00492939 `0FAFC2`                `imul eax, edx`
> 1. `EAX` = `F2` * `02` = `1E4`
> 2. `EAX` = `28A4` * `0` = `0`

:0049293C `03C1`                  `add eax, ecx`
> 1. `EAX` = `1E4` + `3` = `1E7`
> 2. `EAX` = `0` + `3` = `3`

:0049293E `33D2`                  `xor edx, edx`

:00492940 `85C9`                  `test ecx, ecx`

:00492942 `7E16`                  `jle 0049295A`

:00492944 `8BF9`                  `mov edi, ecx`
> `EDI` = `ECX` = `3`

:00492946 `2BFE`                  `sub edi, esi`
> 1. `EDI` = `3` - `2` = `1`
> 2. `EDI` = `3` - `66` = `FFFFFF9D` (H) = `-99` (D)

:00492948 `83C76F`                `add edi, 0000006F`
> 1. `EDI` = `1` + `0000006F` = `70`
> 2. `EDI` = `FFFFFF9D` + `0000006F` = `C`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`00492958` (C)



# 汇编代码分析

## 指令执行过程

### 1. 字符加载
```assembly
:0049294B 0FBE341A movsx esi, byte ptr [edx+ebx]
```
- `esi` 依次从 `fly` 中取字符值
  ① `esi=66`
  ② `esi=6C`
  ③ `esi=79`

### 2. 乘法运算
```assembly
:0049294F 0FAFF7 imul esi, edi
```
- 第一组计算
  ① `esi=66 * 70=2CA0`
  ② `esi=6C * 6F=2ED4`
  ③ `esi=79 * 6E=33FE`

—————————————————————

- 第二组计算
  ① `esi=66 * C=4C8`
  ② `esi=6C * B=4A4`
  ③ `esi=79 * A=4BA`

### 3. 加法运算
```assembly
:00492952 03C6 add eax, esi
```
- 第一组计算
  ① `eax=1E7 + 2CA0=2E87`
  ② `eax=2E87 + 2ED4=5D5B`
  ③ `eax=5D5B + 33FE=9159`

—————————————————————

- 第二组计算
  ① `eax=66 + 4C8=4C8`
  ② `eax=4CB + 4A4=96F`
  ③ `eax=4BA + 96F=E29`

### 4. 指针更新
```assembly
:00492954 42 inc edx
```
- `edx` 依次增 1

### 5. 循环控制变量递减
```assembly
:00492955 4F dec edi
```
- `edi` 依次减 1

### 6. 循环条件判断
```assembly
:00492956 3BD1 cmp edx, ecx
:00492958 7CF1 jl 0049294B
```
- 循环用户名位数次数，共 3 次

**参考跳转：**
*:00492942(C)*

### 7. 结果测试
```assembly
:0049295A 85C0 test eax, eax
```
- 第一组结果：`eax=9159`
- 第二组结果：`eax=E29`

### 8. 条件跳转
```assembly
:0049295C 7D1A jge 00492978
```

### 9. 清理寄存器并返回
```assembly
:0049295E 5F pop edi
:0049295F 5E pop esi
:00492960 F7D8 neg eax
:00492962 5B pop ebx
:00492963 C3 ret
```

# KeyMake 内存注册机参数

**中断地址：** `492DF9`

**中断次数：** 1

**第一字节：** `2B`

**指令长度：** 2

**寄存器方式：** `EAX` (十进制)

> 注意：试炼码必须全部输入数字！

# 注册信息保存

### 1. 注册表信息
- 根键：`HKEY_CLASSES_ROOT`
- 子键：`{MKtTBF8o21}`
- 内容：`@="NUQ=$!!1!$1!8!$M!$A#I!D5Q.4)U!!!!!!\="=R1!!>\`\]R-T5X-D1W/!!!!!!!N!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!#!!!!!!!!N!!!!!!!1!.-(!Q!&!!Y!!!!Q!$M!0A-!!!)!!!!!!!!!!*W_8Q&G<(E!:GRZ7=WNZ!!!!!!!!!!!!!!!!!!!!!!!!!!!!""`

### 2. 系统应用程序 ID
- 根键：`HKEY_CLASSES_ROOT\SystemAppIDs`
- 内容：`@="6!1!!!!!!!!\=""\\\\45NU6%'/'\]S-8U!"`

### 3. 文件注册
- 路径：`C:\WINDOWS\SYSTEM\access.ctl`

> 如果想重新注册，必须删除以上 3 处信息。

# 整理信息

- **系统 ID：** `95065`
- **姓名：** `fly`
- **公司：** `OCN`
  （公司名不参与运算，可以随意输入）
- **密码：** `135899`

---

**Cracked By 巢水工作坊——fly【OCN】**

**时间：** `2003-03-14 16:26:02`

**哈希值：** `4dd466d0e1e0cc123afbe504da2462ca`
