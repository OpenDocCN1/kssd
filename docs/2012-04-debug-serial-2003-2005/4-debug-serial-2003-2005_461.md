
# 奇门遁甲演义 V6.3

> 原文：[`www.pediy.com/kssd/pediy05/pediy50505.htm`](https://www.pediy.com/kssd/pediy05/pediy50505.htm)

奇门遁甲演义 V6.3 破解手记--注册码算法分析

作者：`newlaos[DFCG]`

软件名称：奇门遁甲演义 V6.3(周易)

文件大小：2360KB(软件做好后，没有做任何处理，建议作者有类似 `UPX` 将软件压缩一下)

软件授权：共享软件

使用平台：`Win9x/Me/NT/2000/XP`

发布公司：[`www.380000.com/`](http://www.380000.com/)

软件简介：自动排出从 1900 年到 2050 年内的任意时间的时家、日家、月家、年家奇门遁甲局况；附有相应的古注语和现注解，用彩色标出其中常见的凶格和吉格；方便的时间查找功能，自动查找满足输入条件的时间；可以导出局图的纯文字版本，方便复制和传播；时间调整功能；更详细的帮助文档。

加密方式：注册码

功能限制：频繁弹出注册对话框

`PJ` 工具：`TRW20001.23` 注册版、`W32Dasm8.93` 黄金版，`FI2.5`

`PJ` 日期：2003-03-23

作者 `newlaos` 申明：只是学习，请不用于商业用途或是将本文方法制作的注册机任意传播，造成后果，本人一概不负。

1、先用 `FI2.5` 看一下主程序“奇门遁甲演义 VI.exe”，没有加壳。是用 `VB` 编写的。

2、用 `W32Dasm8.93` 黄金版对主文件进行静态反汇编，再用串式数据参考，找到"注册成功！"(很经典的句子)，双击来到下面代码段。这样就找到注册码的计算部分。

3、再用 `TRW20001.23` 注册版进行动态跟踪，下断 `BPX 0040B73E`(通常在注册成功与否的前面一些下断，这样，才能找到关键部分)，先输入假码 `78787878`

```
.......  
.......  
:0040B73E FF15C4D14100            Call dword ptr [0041D1C4]  
:0040B744 A1D0D64100              mov eax, dword ptr [0041D6D0]  
:0040B749 8D4C2448                lea ecx, dword ptr [esp+48]  
:0040B74D 50                      push eax

* Possible Reference to String Resource ID=00002: "212112121222#8"  
                   |  
:0040B74E 6A02                    push 00000002

* Possible StringData Ref from Data Obj ->"register.ini"  
                   |  
:0040B750 68782D4200              push 00422D78           <===文件里放了只有我们输入的假码(78787878)  
:0040B755 E816830000              call 00413A70           
:0040B75A 8D7C2408                lea edi, dword ptr [esp+08]  
:0040B75E 83C9FF                  or ecx, FFFFFFFF  
:0040B761 33C0                    xor eax, eax  
:0040B763 F2                      repnz  
:0040B764 AE                      scasb  
:0040B765 F7D1                    not ecx  
:0040B767 49                      dec ecx  
:0040B768 51                      push ecx  
:0040B769 8D4C240C                lea ecx, dword ptr [esp+0C]  
:0040B76D 51                      push ecx  
:0040B76E 8D4C2450                lea ecx, dword ptr [esp+50]  
:0040B772 E835830000              call 00413AAC  
:0040B777 8D4C2448                lea ecx, dword ptr [esp+48]  
:0040B77B E8827E0000              call 00413602  
:0040B780 8B0D643F4200            mov ecx, dword ptr [00423F64]  
:0040B786 E8F5020000              call 0040BA80  <===这个 CALL 会改变 88FCA4 处的值(这是一个标志位)，所以 F8 跟进  
:0040B78B 8B15643F4200            mov edx, dword ptr [00423F64]  
:0040B791 5F                      pop edi  
:0040B792 8A8228A40000            mov al, byte ptr [edx+0000A428]   
                     <===EDX=88587C(在 88FCA4 处的值很关键, 不能为 0)  
:0040B798 84C0                    test al, al           <===要不为 0，才能正确跳转  
:0040B79A 7515                    jne 0040B7B1          <===关键跳转，跳才正确

* Possible Reference to String Resource ID=00016: "221212212121#3"  
                   |  
:0040B79C 6A10                    push 00000010

* Possible StringData Ref from Data Obj ->"错误"  
                   |  
:0040B79E 6874004200              push 00420074

* Possible StringData Ref from Data Obj ->"注册失败！"  
                   |  
:0040B7A3 686C2D4200              push 00422D6C  
:0040B7A8 56                      push esi

* Reference To: USER32.MessageBoxA, Ord:01BEh  
                   |  
:0040B7A9 FF1580D14100            Call dword ptr [0041D180]  
:0040B7AF EB78                    jmp 0040B829

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0040B79A(C)  
|

* Possible Reference to String Resource ID=00064: "212112121222#13"  
                   |  
:0040B7B1 6A40                    push 00000040

* Possible StringData Ref from Data Obj ->"提示"  
                   |  
:0040B7B3 68FC2C4200              push 00422CFC

* Possible StringData Ref from Data Obj ->"注册成功！"  
                   |  
:0040B7B8 68602D4200              push 00422D60  
:0040B7BD 56                      push esi

* Reference To: USER32.MessageBoxA, Ord:01BEh  
                   |  
:0040B7BE FF1580D14100            Call dword ptr [0041D180]  
:0040B7C4 A1643F4200              mov eax, dword ptr [00423F64]  
:0040B7C9 6A00                    push 00000000  
:0040B7CB 56                      push esi  
:0040B7CC C78004A40000FA000000    mov dword ptr [ebx+0000A404], 000000FA

* Reference To: USER32.EndDialog, Ord:00B9h  
                   |  
:0040B7D6 FF15DCD14100            Call dword ptr [0041D1DC]  
:0040B7DC EB4B                    jmp 0040B829  
.......  
.......
```

\-----`0040B786 call 0040BA80` 我们按 `F8` 跟进来到下面代码段-------

这一段的功能：是把我的注册码以"-"为标志分成 5 串，而后与机器生成的 5 串依次比较，只一次错就跳出

```
:0040BA80 6AFF                    push FFFFFFFF  
:0040BA82 6858BB4100              push 0041BB58  
:0040BA87 64A100000000            mov eax, dword ptr fs:[00000000]  
:0040BA8D 50                      push eax  
:0040BA8E 64892500000000          mov dword ptr fs:[00000000], esp  
:0040BA95 81EC80000000            sub esp, 00000080  
:0040BA9B 55                      push ebp  
:0040BA9C 8BE9                    mov ebp, ecx  
:0040BA9E 57                      push edi

* Possible Reference to String Resource ID=00001: "121121212221#19"  
                   |  
:0040BA9F 6A01                    push 00000001  
:0040BAA1 8D4C244C                lea ecx, dword ptr [esp+4C]  
:0040BAA5 E8587A0000              call 00413502

* Possible Reference to String Resource ID=00007: "121212212121#13"  
                   |  
:0040BAAA B907000000              mov ecx, 00000007  
:0040BAAF 33C0                    xor eax, eax  
:0040BAB1 8D7C2428                lea edi, dword ptr [esp+28]  
:0040BAB5 6A00                    push 00000000  
:0040BAB7 F3                      repz  
:0040BAB8 AB                      stosd

* Possible StringData Ref from Data Obj ->"register.ini"  
                   |  
:0040BAB9 68782D4200              push 00422D78  
:0040BABE C784249800000000000000  mov dword ptr [esp+00000098], 00000000  
:0040BAC9 66AB                    stosw  
:0040BACB E879FC0000              call 0041B749  
:0040BAD0 83C408                  add esp, 00000008  
:0040BAD3 83F8FF                  cmp eax, FFFFFFFF       
:0040BAD6 0F84CC000000            je 0040BBA8           <===这是不能跳，跳过去就 OVER 了。  
:0040BADC A1D0D64100              mov eax, dword ptr [0041D6D0]  
:0040BAE1 8D4C2448                lea ecx, dword ptr [esp+48]  
:0040BAE5 50                      push eax

* Possible Reference to String Resource ID=00001: "121121212221#19"  
                   |  
:0040BAE6 6A01                    push 00000001

* Possible StringData Ref from Data Obj ->"register.ini"  
                   |  
:0040BAE8 68782D4200              push 00422D78  
:0040BAED E8D47A0000              call 004135C6  
:0040BAF2 8B54244C                mov edx, dword ptr [esp+4C]

* Possible Reference to String Resource ID=00010: "121121212221#10"  
                   |  
:0040BAF6 6A0A                    push 0000000A  
:0040BAF8 8D4C242C                lea ecx, dword ptr [esp+2C]

* Possible Reference to String Resource ID=00030: "1221123121221#30"  
                   |  
:0040BAFC 6A1E                    push 0000001E  
:0040BAFE 42                      inc edx  
:0040BAFF 51                      push ecx  
:0040BB00 8D4C2454                lea ecx, dword ptr [esp+54]  
:0040BB04 89542458                mov dword ptr [esp+58], edx  
:0040BB08 E8E27F0000              call 00413AEF  
:0040BB0D 8D4C2448                lea ecx, dword ptr [esp+48]  
:0040BB11 E8EC7A0000              call 00413602  
:0040BB16 8BCD                    mov ecx, ebp  
:0040BB18 E8A3FDFFFF              call 0040B8C0
```



` :0040BB1D 8D542414` `lea edx, dword ptr [esp+14]`

` :0040BB21 89442408` `mov dword ptr [esp+08], eax`

` :0040BB25 52` `push edx`

` :0040BB26 8D44242C` `lea eax, dword ptr [esp+2C]`

`* Possible Reference to String Resource ID=00045: "112112122212#13"`

` :0040BB2A 6A2D` `push 0000002D`

` :0040BB2C 50` `push eax`

` :0040BB2D 8BCD` `mov ecx, ebp`

` :0040BB2F E8FC4F0000` `call 00410B30` `<===这个 CALL 主要功能是看输入的注册码里有没有五个“-”字符，到这里我们将假码改为 78-78-78-78-78-78，重新来(为了说明方便，我们定义为注册码的 1、2、3、4、5 串)`

` :0040BB34 83F805` `cmp eax, 00000005` `<===EAX 必须等于 5，否则就跳走出错了。`

` :0040BB37 756F` `jne 0040BBA8` `<===因为跳到错的地方有两个，上面那不会跳，所以这里是一个很关键的跳转`

` :0040BB39 53` `push ebx`

` :0040BB3A 56` `push esi`

` :0040BB3B 33FF` `xor edi, edi` `<===EDI 是个计数器，共 5 次，对应注册码有 5 串`

` :0040BB3D 8D5C241C` `lea ebx, dword ptr [esp+1C]`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address: :0040BB8D(C)`

` :0040BB41 8B542410` `mov edx, dword ptr [esp+10]`

` :0040BB45 8D4C2414` `lea ecx, dword ptr [esp+14]`

` :0040BB49 51` `push ecx`

` :0040BB4A 57` `push edi`

` :0040BB4B 52` `push edx`

` :0040BB4C 8BCD` `mov ecx, ebp`

` :0040BB4E E89DFDFFFF` `call 0040B8F0` `<===呵呵，这个就是算出正确的串值的 CALL 了。F8 跟进`

` :0040BB53 8B33` `mov esi, dword ptr [ebx]`

` :0040BB55 8D442414` `lea eax, dword ptr [esp+14]` `<===我的机器出现 EAX=为正确的串值(这里用于比较的)，共 5 次，在这里设断，得到一个值就将它改为注册码的对应的串，再点注册，又失败，但得到下一串的值，如此循环，注册失败 5 次后，你就得到全部的注册码了，我的是 PLT6D-1DNLD-1KFDD-0XXXX-PLSCG`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address: :0040BB77(C)`

` :0040BB59 8A10` `mov dl, byte ptr [eax]`

` :0040BB5B 8ACA` `mov cl, dl`

` :0040BB5D 3A16` `cmp dl, byte ptr [esi]`

` :0040BB5F 751C` `jne 0040BB7D`

` :0040BB61 84C9` `test cl, cl`

` :0040BB63 7414` `je 0040BB79`

` :0040BB65 8A5001` `mov dl, byte ptr [eax+01]`

` :0040BB68 8ACA` `mov cl, dl`

` :0040BB6A 3A5601` `cmp dl, byte ptr [esi+01]`

` :0040BB6D 750E` `jne 0040BB7D`

` :0040BB6F 83C002` `add eax, 00000002`

` :0040BB72 83C602` `add esi, 00000002`

` :0040BB75 84C9` `test cl, cl`

` :0040BB77 75E0` `jne 0040BB59` `<===这里构成一个小循环功能是把我们输入的假码以“-”字符分开的字符与在 0040BB55 给出的 EAX 值作比较，只要一处错就跳到 40BB7D，就 OVER 了。`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address: :0040BB63(C)`

` :0040BB79 33C0` `xor eax, eax`

` :0040BB7B EB05` `jmp 0040BB82`

`* Referenced by a (U)nconditional or (C)onditional Jump at Addresses: :0040BB5F(C), :0040BB6D(C)`

` :0040BB7D 1BC0` `sbb eax, eax` `<===(带位减法)EAX 清 0`

` :0040BB7F 83D8FF` `sbb eax, FFFFFFFF` `<===EAX=1`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address: :0040BB7B(U)`

` :0040BB82 85C0` `test eax, eax` `<===EAX=1 说明上面循环比较出错了，EAX=0 就说明比较对了。`

` :0040BB84 750B` `jne 0040BB91` `<===如果 EAX=1，这里再跳走，EAX=0 不跳`

` :0040BB86 47` `inc edi`

` :0040BB87 83C304` `add ebx, 00000004`

` :0040BB8A 83FF05` `cmp edi, 00000005`

` :0040BB8D 7CB2` `jl 0040BB41` `<===这里是个大循环，共 5 次，分别比较机器算出的 5 个串和我们输入的 5 个串做比较。`

` :0040BB8F EB07` `jmp 0040BB98` `<===5 次循环都正确，就到这里再跳向胜利。`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address: :0040BB84(C)`

` :0040BB91 C68528A4000000` `mov byte ptr [ebp+0000A428], 00`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address: :0040BB8F(U)`

` :0040BB98 5E` `pop esi`

` :0040BB99 83FF05` `cmp edi, 00000005` `<===这里也是一个很关键的对比，上面 0040BB91 已经把标志位清 0`

` :0040BB9C 5B` `pop ebx`

` :0040BB9D 7510` `jne 0040BBAF` `<===如果这里跳走，就 OVER`

` :0040BB9F C68528A4000001` `mov byte ptr [ebp+0000A428], 01` `<===[ebp+0000A428]是个标志位，能到这里，说明成功`

` :0040BBA6 EB07` `jmp 0040BBAF`

`* Referenced by a (U)nconditional or (C)onditional Jump at Addresses: :0040BAD6(C), :0040BB37(C)`

` :0040BBA8 C68528A4000000` `mov byte ptr [ebp+0000A428], 00` `<===[ebp+0000A428]是个标志位，要是到这里，说明失败`

`* Referenced by a (U)nconditional or (C)onditional Jump at Addresses: :0040BB9D(C), :0040BBA6(U)`

` :0040BBAF 8D4C2454` `lea ecx, dword ptr [esp+54]`

` :0040BBB3 C7842490000000FFFFFFFF` `mov dword ptr [esp+00000090], FFFFFFFF`

` :0040BBBE E8F0790000` `call 004135B3`

` :0040BBC3 8D4C2454` `lea ecx, dword ptr [esp+54]`

` :0040BBC7 E8DE780000` `call 004134AA`

` :0040BBCC 8B8C2488000000` `mov ecx, dword ptr [esp+00000088]`

` :0040BBD3 5F` `pop edi`

` :0040BBD4 5D` `pop ebp`

` :0040BBD5 64890D00000000` `mov dword ptr fs:[00000000], ecx`

` :0040BBDC 81C48C000000` `add esp, 0000008C`

` :0040BBE2 C3` `ret`

`-----0040BB4E call 0040B8F0 这个就是算出正确的串值的 CALL 了---------------`

我的注册码机器 454376902，注册码就是 PLT6D-1DNLD-1KFDD-0XXXX-PLSCG

`* Referenced by a CALL at Address: :0040BB4E`

` :0040B8F0 83EC18` `sub esp, 00000018`

` :0040B8F3 33C0` `xor eax, eax` `<===每次进来 EAX 都要清 0`

` :0040B8F5 53` `push ebx`

` :0040B8F6 8B5C2428` `mov ebx, dword ptr [esp+28]`

` :0040B8FA 8BCB` `mov ecx, ebx`

` :0040B8FC 56` `push esi`

` :0040B8FD 57` `push edi`

` :0040B8FE 8901` `mov dword ptr [ecx], eax`

` :0040B900 66894104` `mov word ptr [ecx+04], ax`

` :0040B904 E8B68F0000` `call 004148BF`

` :0040B909 99` `cdq`

`* Possible Reference to String Resource ID=00100: "221121121221#5"`

` :0040B90A B964000000` `mov ecx, 00000064`

` :0040B90F F7F9` `idiv ecx` `<===整数除法(EAX/ECX)，商回 EAX，余数回 ECX`

` :0040B911 33C0` `xor eax, eax`

` :0040B913 85D2` `test edx, edx`

` :0040B915 7E14` `jle 0040B92B`

` :0040B917 8B4C2428` `mov ecx, dword ptr [esp+28]`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address: :0040B929(C)`

` :0040B91B 85C0` `test eax, eax`

` :0040B91D 7405` `je 0040B924`

` :0040B91F 0FAFC8` `imul ecx, eax`

` :0040B922 EB02` `jmp 0040B926`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address: :0040B91D(C)`

` :0040B924 8BCA` `mov ecx, edx`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address: :0040B922(U)`

` :0040B926 40` `inc eax`

` :0040B927 3BC2` `cmp eax, edx`

` :0040B929 7CF0` `jl 0040B91B` `<===这里是一个循环结构`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address: :0040B915(C)`

` :0040B92B 8B44242C` `mov eax, dword ptr [esp+2C]` `<===从 EAX=0 开始，每次循环到里，就加 1，最后`

` :0040B92F 83F804` `cmp eax, 00000004` `<===最后 EAX=4，退出，共循环 5 次，正好是注册码的 5 个串值`

` :0040B932 7761` `ja 0040B995`

` :0040B934 FF248560BA4000` `jmp dword ptr [4*eax+0040BA60]` `<===注意由于每次 EAX 都加 1，所以导致这个无条件跳转到的位置不同，每次不同跳转产生不同的值共同构成这个注册码。`



# 汇编代码分析与流程追踪

## 第一部分：入口与机器码处理

`0040B93B 8B442428` `mov eax, dword ptr [esp+28]`
第一次跳到这里，`EAX=1B153DC6`，它的十进制是 `454376732`，正好是我的机器码。

`0040B93F 3503008519` `xor eax, 19850003`
`1B153DC6` 与 `19850003` 做异或运算。

`0040B944 0D34120000` `or eax, 00001234`
`EAX` 再与 `1234` 做或运算，值放入 `EAX`（为说明问题，我们定义为 `EAX1`）。

`0040B949 EB4E` `jmp 0040B999`
跳走。

`0040B94B 8B442428` `mov eax, dword ptr [esp+28]`
第二次跳到这里，`EAX=1B153DC6`，它的十进制是 `454376732`，正好是我的机器码。

**Possible Reference to String Resource ID=`00001`: "121121212221#19"**

`0040B94F B901000000` `mov ecx, 00000001`
`ECX` 初始化为 `1`，计数器。

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**
`:0040B95D(C)`

`0040B954 8BD1` `mov edx, ecx`
`0040B956 0FAFC2` `imul eax, edx`
但实际动态跟踪时发现是 `EAX=EAX*1*2*3*4*5*6*7*8*9*A*B*C*D*E*F*10*11*12*13`。

`0040B959 41` `inc ecx`
`ECX=ECX+1`。

`0040B95A 83F913` `cmp ecx, 00000013`
`0040B95D 7EF5` `jle 0040B954`
共循环 `16`（注 `13` 是十六进制）次。

`0040B95F 0D34120000` `or eax, 00001234`
`EAX` 再与 `1234` 做或运算，值放入 `EAX`（为说明问题，我们定义为 `EAX2`）。

`0040B964 EB33` `jmp 0040B999`
跳走。

`0040B966 8B442428` `mov eax, dword ptr [esp+28]`
第三次跳到这里，`EAX=1B153DC6`，它的十进制是 `454376732`，正好是我的机器码。

`0040B96A 69C03C642B01` `imul eax, 012B643C`
`EAX=EAX*12B643C`。

`0040B970 0D34120000` `or eax, 00001234`
`EAX` 再与 `1234` 做或运算，值放入 `EAX`（为说明问题，我们定义为 `EAX3`）。

`0040B975 EB22` `jmp 0040B999`
跳走。

`0040B977 8B442428` `mov eax, dword ptr [esp+28]`
第四次跳到这里，`EAX=1B153DC6`，它的十进制是 `454376732`，正好是我的机器码。

**Possible Reference to String Resource ID=`00010`: "121121212221#10"**

`0040B97B B90A000000` `mov ecx, 0000000A`
将 `ECX` 初始化为 `10`。

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**
`:0040B986(C)`

`0040B980 8BD0` `mov edx, eax`
`0040B982 0FAFC2` `imul eax, edx`
最后（`EAX` 的值用 C 语言表示）：

```c
eax=1B153DC6;
for(i=0;i<>10;i++)
{
    eax=eax*eax;
}
```

`0040B985 49` `dec ecx`
`ECX=ECX-1`。

`0040B986 75F8` `jne 0040B980`
共一个小的循环结构，共循环 `10` 次，后退出。

`0040B988 EB0F` `jmp 0040B999`
跳走，这里的 `EAX`（为说明问题，我们定义为 `EAX4`）。

`0040B98A 8B442428` `mov eax, dword ptr [esp+28]`
`0040B98E 3506068519` `xor eax, 19850606`
`0040B993 EB04` `jmp 0040B999`

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**
`:0040B932(C)`

`0040B995 8B442428` `mov eax, dword ptr [esp+28]`
第五次跳到这里，`EAX=1B153DC6`，它的十进制是 `454376732`，正好是我的机器码。这里 `EAX` 不再做任何处理，直接拿来用了（为说明问题，我们定义为 `EAX5`）。

**Referenced by a (U)nconditional or (C)onditional Jump at Addresses:**
`:0040B949(U), :0040B964(U), :0040B975(U), :0040B988(U), :0040B993(U)`

`0040B999 8D4C240C` `lea ecx, dword ptr [esp+0C]`

**Possible Reference to String Resource ID=`00036`: "2114112212221#24"**

`0040B99D 6A24` `push 00000024`
`0040B99F 51` `push ecx`
`ECX=8850D8`。

`0040B9A0 50` `push eax`
这里的 `EAX1`，`EAX2`，`EAX3`，`EAX4`，`EAX5` 分别为 `2903FF5`，`B2F61234`，`CB5DD27`，`0000000`，`2903BC0`。

`0040B9A1 E879FD0000` `call 0041B71F`
这里面，`EAX1`，`EAX2`，`EAX3`，`EAX4`，`EAX5` 再进行处理，就生成最终对应的串值，并放在出来的 `EAX` 中，`F8` 跟进。

`0040B9A6 8D7C2418` `lea edi, dword ptr [esp+18]`
`0040B9AA 83C9FF` `or ecx, FFFFFFFF`
`0040B9AD 33C0` `xor eax, eax`
`0040B9AF 83C40C` `add esp, 0000000C`
`0040B9B2 F2` `repnz`
`0040B9B3 AE` `scasb`
`0040B9B4 F7D1` `not ecx`
`0040B9B6 49` `dec ecx`
`0040B9B7 8BF1` `mov esi, ecx`
`0040B9B9 33C9` `xor ecx, ecx`
`0040B9BB 85F6` `test esi, esi`
`0040B9BD 7E17` `jle 0040B9D6`

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**
`:0040B9D4(C)`

`0040B9BF 8A440C0C` `mov al, byte ptr [esp+ecx+0C]`
`0040B9C3 3C61` `cmp al, 61`
`0040B9C5 7C0A` `jl 0040B9D1`
`0040B9C7 3C7A` `cmp al, 7A`
`0040B9C9 7F06` `jg 0040B9D1`
`0040B9CB 2C20` `sub al, 20`
`0040B9CD 88440C0C` `mov byte ptr [esp+ecx+0C], al`

**Referenced by a (U)nconditional or (C)onditional Jump at Addresses:**
`:0040B9C5(C), :0040B9C9(C)`

`0040B9D1 41` `inc ecx`
`0040B9D2 3BCE` `cmp ecx, esi`
`0040B9D4 7CE9` `jl 0040B9BF`

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**
`:0040B9BD(C)`

`0040B9D6 83FE05` `cmp esi, 00000005`
`0040B9D9 7C1C` `jl 0040B9F7`
`0040B9DB 8D54240C` `lea edx, dword ptr [esp+0C]`
`0040B9DF 52` `push edx`

**Possible StringData Ref from Data Obj -> "%.5s"**

`0040B9E0 68F82D4200` `push 00422DF8`
`0040B9E5 53` `push ebx`
`0040B9E6 E8EE890000` `call 004143D9`
`0040B9EB 83C40C` `add esp, 0000000C`
`0040B9EE 5F` `pop edi`
`0040B9EF 5E` `pop esi`
`0040B9F0 5B` `pop ebx`
`0040B9F1 83C418` `add esp, 00000018`
`0040B9F4 C20C00` `ret 000C`

## 第二部分：Call 0041B71F 跟进

**分段标识**: `0040B9A1 call 0041B71F` 按 `F8` 跟进来到下面代码段

这段代码的作用：将 `EAX1`，`EAX2`，`EAX3`，`EAX4`，`EAX5` 再进行处理，就生成最终对应的串值，并放在出来的 `EAX` 中。

**Referenced by a CALL at Addresses:**
`:0040B80D`, `:0040B9A1`

`0041B71F 55` `push ebp`
`0041B720 8BEC` `mov ebp, esp`
`0041B722 33C0` `xor eax, eax`
`0041B724 837D100A` `cmp dword ptr [ebp+10], 0000000A`
`[EBP+10]` 每次都是 `24`。

`0041B728 7508` `jne 0041B732`
所以这里每次跳走。

`0041B72A 394508` `cmp dword ptr [ebp+08], eax`
`0041B72D 7D03` `jge 0041B732`

**Possible Reference to String Resource ID=`00001`: "121121212221#19"**

`0041B72F 6A01` `push 00000001`
`0041B731 58` `pop eax`

**Referenced by a (U)nconditional or (C)onditional Jump at Addresses:**
`:0041B728(C), :0041B72D(C)`

`0041B732 50` `push eax`
`0041B733 FF7510` `push [ebp+10]`
跳到这里，将 `24`（十六进制）压入栈。

`0041B736 FF750C` `push [ebp+0C]`
将 `8850D8` 压入栈，这个是最终串值返回时的地址。

`0041B739 FF7508` `push [ebp+08]`
依次将 `EAX1`，`EAX2`，`EAX3`，`EAX4`，`EAX5` 压入栈。

`0041B73C E882FFFFFF` `call 0041B6C3`
这个 `CALL` 出来，`EAX` 里就是正确的串值了，`F8` 再跟进。

`0041B741 8B450C` `mov eax, dword ptr [ebp+0C]`
最终将正确的串值放入 `EAX` 中。

`0041B744 83C410` `add esp, 00000010`
`0041B747 5D` `pop ebp`
`0041B748 C3` `ret`

## 第三部分：Call 0041B6C3 跟进

**分段标识**: `0041B73C call 0041B6C3` 按 `F8` 跟进来到下面代码段

这段代码的作用：将 `EAX1`，`EAX2`，`EAX3`，`EAX4`，`EAX5` 再进行处理，就生成最终对应的串值，并放在出来的 `EAX` 中。

**Referenced by a CALL at Addresses:**
`:0041B6B6`, `:0041B73C`

`0041B6C3 55` `push ebp`
`0041B6C4 8BEC` `mov ebp, esp`
`0041B6C6 837D1400` `cmp dword ptr [ebp+14], 00000000`
这里 `5` 次都相等。

`0041B6CA 8B4D0C` `mov ecx, dword ptr [ebp+0C]`



## 汇编代码片段

```
:0041B6CD 53           push ebx
:0041B6CE 56           push esi
:0041B6CF 57           push edi
:0041B6D0 74 0B        je 0041B6DD               ; <=== 这里 5 次都跳走

:0041B6D2 8B 75 08     mov esi, dword ptr [ebp+08]
:0041B6D5 C6 01 2D     mov byte ptr [ecx], 2D
:0041B6D8 41           inc ecx
:0041B6D9 F7 DE        neg esi
:0041B6DB EB 03        jmp 0041B6E0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041B6D0(C)
|
:0041B6DD 8B 75 08     mov esi, dword ptr [ebp+08]   ; <=== 跳到这里，ESI=EAX(1, 2, 3, 4, 5)

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041B6DB(U)
|
:0041B6E0 8B F9        mov edi, ecx                 ; <=== EDI=8850D8

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041B706(C)
|
:0041B6E2 8B C6        mov eax, esi                 ; <=== EAX=EAX(1, 2, 3, 4, 5)
:0041B6E4 33 D2        xor edx, edx                 ; <=== 将 EDX 清 0
:0041B6E6 F7 75 10     div [ebp+10]                 ; <=== 将 EAX(1, 2, 3, 4, 5) 除以 24，商回 EAX，余数回 EDX
:0041B6E9 8B C6        mov eax, esi                 ; <=== EAX=EAX(1, 2, 3, 4, 5)
:0041B6EB 8B DA        mov ebx, edx                 ; <=== EBX=余数
:0041B6ED 33 D2        xor edx, edx                 ; <=== 将 EDX 清 0
:0041B6EF F7 75 10     div [ebp+10]                 ; <=== 将 EAX(1, 2, 3, 4, 5) 除以 24(十六进制)，商回 EAX，余数回 EDX (不明白为什么又除一遍)
:0041B6F2 83 FB 09     cmp ebx, 00000009            ; <=== 这个判断很重要!
:0041B6F5 8B F0        mov esi, eax
:0041B6F7 76 05        jbe 0041B6FE                 ; <=== 如果余数是数字，就跳转。如果余数是字符，就不跳转
:0041B6F9 80 C3 57     add bl, 57
:0041B6FC EB 03        jmp 0041B701

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041B6F7(C)
|
:0041B6FE 80 C3 30     add bl, 30

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041B6FC(U)
|
:0041B701 88 19        mov byte ptr [ecx], bl       ; <=== 其实这个判断的处理作用就是将余数处理为其对应的 ASCII 值(字符是小写)
:0041B703 41           inc ecx
:0041B704 85 F6        test esi, esi
:0041B706 77 DA        ja 0041B6E2                  ; <=== 从这里向上，构成一个循环结构，共循环 5 次，第 n 次在 bl 处得到余数转为字符后，就构成串值的第 n 位
```

**这一小段的循环作用** (以 `EAX1` 为例)：

| 公式 | 商 | 余数 | 对应的字符 |
| :--- | :--- | :--- | :--- |
| `2903FF5/24` | `123AAA` | `D` | `d` |
| `123AAA/24` | `81A1` | `6` | `6` |
| `81A1/24` | `339` | `1D` | `t` (d 字符后面的 16 位的字符) 以 ASC 码表为顺序 |
| `339/24` | `19` | `15` | `l` (数字 5 后面的 16 位的字符) |
| `19/24` | `0` | `19` | `p` (数字 9 后面的 16 位的字符) |

最后在内存的 `8850D8` 位置上得到串值 `d6tlp`。

```
:0041B708 80 21 00     and byte ptr [ecx], 00
:0041B70B 49           dec ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041B718(C)
|
:0041B70C 8A 17        mov dl, byte ptr [edi]
:0041B70E 8A 01        mov al, byte ptr [ecx]
:0041B710 88 11        mov byte ptr [ecx], dl
:0041B712 88 07        mov byte ptr [edi], al
:0041B714 49           dec ecx
:0041B715 47           inc edi
:0041B716 3B F9        cmp edi, ecx
:0041B718 72 F2        jb 0041B70C
                     ; <=== 从这里向上，构成一个循环结构，作用是将上面得到串值反向再取值，所以最后 8850D8 位置上得到串值 plt6d(其它程序段会把它转为大写)

:0041B71A 5F           pop edi
:0041B71B 5E           pop esi
:0041B71C 5B           pop ebx
:0041B71D 5D           pop ebp
:0041B71E C3           ret
```

### 此段总结
依照上面的步骤将 `EAX1`、`EAX2`、`EAX3`、`EAX4`、`EAX5` 最后处理为 `plt6d`, `1dnld`, `1kfdd`, `0xxxx`, `plscg` ---- 注册码浮出水面。

## 4、算法总结
**类型**：`f(机器码)=注册码`

- **a、** 将机器码 `454376732` (每台机器码都不一样)，转换为十六进制 `1B153DC6`。
- **b、** 将 `1B153DC6` 分别做 5 种的处理，得到相应串值的第一次变形：
    - **第一种处理**：`1B153DC6` 与 `19850003` 做异或运算，再与 `1234` 做或运算，得到 `2903FF5`。
    - **第二种处理**：`1B153DC6`，`EAX=EAX*1*2*3*4*5*6*7*8*9*A*B*C*D*E*F*10*11*12*13`，再与 `1234` 做或运算，得到 `B2F61234`。
    - **第三种处理**：`1B153DC6*12B643C`，再与 `1234` 做或运算，得到 `CB5DD27C`。
    - **第四种处理**：`eax=1B153DC6; for(i=0;i<>10;i++) {eax=eax²;}` 一般情况 `EAX` 都会因为值太大，而爆为 `00000000`。
    - **第五种处理**：`1B153DC6` 不做处理。
- **c、** 分别将上一步得到的 5 串数字，做同样的处理，也就是(以 `EAX1` 为例)：

    | 公式 | 商 | 余数 | 对应的字符 |
    | :--- | :--- | :--- | :--- |
    | `2903FF5/24` | `123AAA` | `D` | `d` |
    | `123AAA/24` | `81A1` | `6` | `6` |
    | `81A1/24` | `339` | `1D` | `t` (d 字符后面的 16 位的字符) 以 ASC 码表为顺序 |
    | `339/24` | `19` | `15` | `l` (数字 5 后面的 16 位的字符) |
    | `19/24` | `0` | `19` | `p` (数字 9 后面的 16 位的字符) |

    最后改为字符串 `d6tlp`，其它的四个串也依次类推。

- **d、** 将上一步得到的 5 个字符分别反向取值，分别得到 `plt6d`, `1dnld`, `1kfdd`, `0xxxx`, `plscg`，最后的注册码就是将它转为大写后，中间分别加上“-”，`PLT6D-1DNLD-1KFDD-0XXXX-PLSCG`。

## 5、注册信息存放
注册信息存放在 `奇门遁甲演义 V6.3` 所在目录的 `register.ini` 文件中。以明文件形式保留。
`4dd466d0e1e0cc123afbe504da2462ca`
