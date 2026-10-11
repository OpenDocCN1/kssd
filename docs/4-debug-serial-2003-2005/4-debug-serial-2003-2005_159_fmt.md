
# HappyEO 电子琴 3.05

> 原文：[`www.pediy.com/kssd/pediy06/pediy6135.htm`](https://www.pediy.com/kssd/pediy06/pediy6135.htm)

HappyEO 电子琴 3.05 标准版破解手记---算法分析

作者：`yqmjch`

文件大小：`1901kb`

软件授权：共享软件

下载页面：`http://www.happyeo.com/Download.htm`

软件简介：是否早已对乐器店里的电子琴心仪已久却又囊中羞涩？是否早就想学习音乐却无老师指导？是否想给孩子进行音乐教育却力不从心？是否早就想学习电脑音乐？试试《HappyEO 电子琴》！它将把你的电脑变成一架高级电子琴！初学者可以用它来学习乐理和识谱，高手则可用它在电脑键盘上弹奏出美妙的乐曲。还可用它来在电脑上练习架子鼓。`128` 种音色。支持键盘分离、多种效果、自动伴奏、录音、放音、多音轨录音，支持 `MIDI` 文件的播放、输入和输出，支持音色和效果的编辑，支持外接 `MIDI` 键盘或带 `MIDI` 接口的电子琴！让你充分领略电脑音乐的美妙！

运行软件，在软件界面上点鼠标右键选注册，依次填入注册信息（姓名不少于 `4` 以上、地址 `12` 位以上、注册码 `8` 位，不然会出错），

注册名：`yqmjch`

地址：`abcdefg@163.com`

注册码：`qwertyui`

点确定后，发现是重启后比较注册码的。再次启动软件，用 `RegSnap` 监视注册，发现注册信息存放在注册表 `HKEY_USERS\.DEFAULT\Software\Happye03SC` 下。关闭软件，用 `PEiD` 查，有 `ASPack 2.12` 壳，用 `AspackDie` 可轻松脱掉。`w32Dasm` 反汇编，在字符串参考中查找 `\\Software\\HappyEO3SC`，共三处，经分析，可知 `510C227D` 处为启动时验证注册码处。用 `ICE` 装入软件，下 `bpx regqueryvalueexa`，中断后下 `bc *`，再下 `g 510c227d` 来到下面：

```
* Possible StringData Ref from Code Obj ->"\\Software\\HappyEO3SC"

:510C227D              mov edx, 510C2364
:510C2282              mov eax, dword ptr [ebp-10]

……
省略
……

* Possible StringData Ref from Code Obj ->"UserName"（读注册名）

:510C22B4              mov edx, 510C23A4
:510C22B9              mov eax, dword ptr [ebp-10]

……
省略
……

* Possible StringData Ref from Code Obj ->"Address"（读地址）

:510C22CE              mov edx, 510C23B8
:510C22D3              mov eax, dword ptr [ebp-10]
:510C22D6              call 5106F4A0

……
省略
……

* Possible StringData Ref from Code Obj ->"RegisterCode"（读假注册码）

:510C22E8              mov edx, 510C23C8
:510C22ED              mov eax, dword ptr [ebp-10]
:510C22F0              call 5106F4A0

……
省略
……

单步跟踪后跳到此处：

:510C247B              call 510C23D8
:510C2480              mov edx, dword ptr [51104238]
:510C2486              mov byte ptr [edx], al
:510C2488              mov eax, dword ptr [51104238]
:510C248D              cmp byte ptr [eax], 00
:510C2490              je 510C249B（不为零就跳）
:510C2492              call 5100A334
:510C2497              fstp qword ptr [ebp-10]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:510C2490(C)
|

:510C249B              mov eax, dword ptr [ebx]
:510C249D              call 51004898（读注册名位数入 EAX）
:510C24A2              cmp eax, 00000004
:510C24A5              jl 510C24E2（小于 4 位就跳，跳则 over）
:510C24A7              mov eax, dword ptr [esi]
:510C24A9              call 51004898（读地址位数入 EAX）
:510C24AE              cmp eax, 0000000C
:510C24B1              jl 510C24E2（小于 12 位就跳，跳则 over）
:510C24B3              mov eax, dword ptr [ebp+08]
:510C24B6              mov eax, dword ptr [eax]
:510C24B8              call 51004898（读注册码位数入 EAX）
:510C24BD              cmp eax, 00000008
:510C24C0              jne 510C24E2（不等于 8 位就跳，跳则 over）
:510C24C2              lea ecx, dword ptr [ebp-1C]
:510C24C5              mov edx, dword ptr [esi]
:510C24C7              mov eax, dword ptr [ebx]
:510C24C9              call 510C1F98（算法 call 跟进）
:510C24CE              mov eax, dword ptr [ebp-1C]
:510C24D1              mov edx, dword ptr [ebp+08]
:510C24D4              mov edx, dword ptr [edx]
:510C24D6              call 510049DC（关键 call，比较真假注册码）
:510C24DB              jne 510C24E2（关键跳，不相等就跳，跳则 over）
:510C24DD              mov byte ptr [edi], 04
:510C24E0              jmp 510C252D

跟进算法 call:

:510C1F98              push ebp
:510C1F99              mov ebp, esp
……
省略
……
:510C1FCA              mov eax, ebx
:510C1FCC              call 510045C4

:510C1FD1              mov eax, dword ptr [ebp-04]
:510C1FD4              call 51004898
:510C1FD9              cmp eax, 00000004
:510C1FDC              jl 510C2008
:510C1FDE              mov eax, dword ptr [ebp-08]
:510C1FE1              call 51004898
:510C1FE6              cmp eax, 0000000C
:510C1FE9              jl 510C2008
以上这段程序仍是对注册名、地址位数的比较。

:510C1FEB              lea eax, dword ptr [ebp-0C]
:510C1FEE              mov ecx, dword ptr [ebp-08]（地址 abcdefg@163.com 入 ECX）
:510C1FF1              mov edx, dword ptr [ebp-04]（注册名 yqmjch 入 EDX）
:510C1FF4              call 510048E4（将注册名与地址连接）
:510C1FF9              mov eax, dword ptr [ebp-0C]（连接后的 yqmjchabcdefg@163.com 入 EAX）
:510C1FFC              call 510C1EA0（跟进此 call①，作用：用 yqmjchabcdefg@163.com 算出一个参数）
:510C2001              mov edx, ebx
:510C2003              call 510C1F18（跟进此 call②，作用：将参数运算后查表得出注册码）

跟进 call①：

* Referenced by a CALL at Address:
|:510C1FFC   
|
:510C1EA0              push ebp
:510C1EA1              mov ebp, esp
……
省略
……
:510C1EC0              or ebx, FFFFFFFF
:510C1EC3              mov eax, dword ptr [ebp-04]（yqmjchabcdefg@163.com 入 EAX）
:510C1EC6              call 51004898（取 yqmjchabcdefg@163.com 的位数）
:510C1ECB              mov esi, eax
:510C1ECD              test esi, esi
:510C1ECF              jle 510C1EEA
:510C1ED1              mov edi, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:510C1EE8(C)
|
:510C1ED6              mov eax, dword ptr [ebp-04] （yqmjchabcdefg@163.com 入 EAX）
:510C1ED9              mov al, byte ptr [eax+edi-01]（取第一位“y”）
:510C1EDD              mov edx, ebx
:510C1EDF              call 510C1E80（运算 call 跟进）
                          :510C1E80       xor al, dl（与 dl 参数值进行异或运算）
                          :510C1E82       and eax, 000000FF（与运算）
                          :510C1E87       mov eax, dword ptr [4*eax+511012B0]（乘、加运算）
                          :510C1E8E       shr edx, 08（逻辑右移）
                          :510C1E91       and edx, 00FFFFFF（与运算）
                          :510C1E97       xor eax, edx（与 edx 异或运算）
                          :510C1E99       ret
:510C1EE4               mov ebx, eax
:510C1EE6               inc edi
:510C1EE7               dec esi
:510C1EE8               jne 510C1ED6（向上循环，直到取完 yqmjchabcdefg@163.com）

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:510C1ECF(C)
|
:510C1EEA               mov eax, ebx（得出的参数值入 eax）
:510C1EEC               call 510C1E9C（再与-1 作异或运算后作为返回值）
                           :510C1E9C       xor eax, FFFFFFFF
                           :510C1E9F       ret
:510C1EF1               mov ebx, eax
:510C1EF3               xor eax, eax
:510C1EF5               pop edx
:510C1EF6               pop ecx
:510C1EF7               pop ecx
:510C1EF8               mov dword ptr fs:[eax], edx
:510C1EFB               push 510C1F10

跟进 call②：

* Referenced by a CALL at Address:
|:510C2003   
|
:510C1F18               push ebp
:510C1F19               mov ebp, esp
……
省略
……
:510C1F2F               mov dword ptr fs:[eax], esp
:510C1F32               mov eax, edi
:510C1F34               call 510045C4
:510C1F39               mov esi, 00000008（注册码位数入 esi）

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:510C1F72(C)
|
:510C1F3E               mov eax, ebx（call①计算出的参数入 eax）
:510C1F40               mov ecx, 00000024（参数 24 入 ecx）
:510C1F45               xor edx, edx
:510C1F47               div ecx（call①计算出的参数除以 24，余数入 edx）
:510C1F49               mov eax, dword ptr [5110419C]（此处有一注册码表：$3456789BCDEFGHIJKLMNOPQRSTUVWXYZ012A）
:510C1F4E               mov dl, byte ptr [eax+edx+01]（查表取值入 dl）
:510C1F52               lea eax, dword ptr [ebp-04]
:510C1F55               call 510047A4
:510C1F5A               mov edx, dword ptr [ebp-04]（得出的注册码入 edx）
:510C1F5D               mov eax, edi
:510C1F5F               call 510048A0（将得出的注册码连接）
:510C1F64               mov eax, ebx（call①计算出的参数入 eax）
:510C1F66               mov ecx, 00000007（参数７入 ecx）
:510C1F6B               xor edx, edx
```



### 汇编指令详解

- `:510C1F6D` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `div ecx`（call①计算出的参数除以７，商入 `eax`）
- `:510C1F6F` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `mov ebx, eax`（将 `eax` 的值存入 `ebx`，作为下次计算求值的参数）
- `:510C1F71` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `dec esi`
- `:510C1F72` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `jne 510C1F3E`（循环计算取值，得出注册码）
- `:510C1F74` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `xor eax, eax`

### 注册信息整理

- **注册名**：`yqmjch`
- **地 址**：`abcdefg@163.com`
- **注册码**：`QWXS2O58`

注册信息存放在注册表 `HKEY_USERS\.DEFAULT\Software\Happye03SC` 下。

再有就是不知是否有暗桩，初学破解，不妥之处还望高手指点。

```
4dd466d0e1e0cc123afbe504da2462ca
```
