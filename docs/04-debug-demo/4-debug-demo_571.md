
# Ad Muncher V4.3d

> 原文：[`www.pediy.com/kssd/pediy04/kanxue253.htm`](https://www.pediy.com/kssd/pediy04/kanxue253.htm)

`Ad Muncher V4.3d`

`http://www.admuncher.com/download.shtml`

`regged by Fpc/CCG&BCG`

`tools: TRW NYDoll/CCG 汉化版`

```
0167:0057C4D9  MOV     EDI,005D487B     <- 这个地址是输入的注册码
0167:0057C4DE  MOV     AL,2D
0167:0057C4E0  MOV     ECX,14
0167:0057C4E5  REPNE SCASB     <- 检查有没有'-'
0167:0057C4E7  JNZ     NEAR 0057C5D3   <- 没有则失败，跳下去。有则继续
0167:0057C4ED  MOV     BYTE [EDI-01],00
0167:0057C4F1  PUSH    EDI     <- ***
0167:0057C4F2  MOV     EDI,10
0167:0057C4F7  MOV     ESI,005D487B
0167:0057C4FC  CALL    0057CCB2    <- 将'-'前面的部分 string->Hex，到 eax
0167:0057C501  MOV     EBX,EAX   <- 保存到 ebx
0167:0057C503  MOV     ESI,005D477B    <- 这里是指向名字的
0167:0057C508  XOR     ECX,ECX
0167:0057C50A  XOR     EAX,EAX
0167:0057C50C  LODSB
0167:0057C50D  ADD     ECX,EAX
0167:0057C50F  ROR     ECX,08
0167:0057C512  ADD     ECX,EBX
0167:0057C514  CMP     AL,00
0167:0057C516  JNZ     0057C50C    <- 上面这个小循环将名字与注册码前一部分混合运算
0167:0057C518  NOT     ECX     <- 取反
0167:0057C51A  POP     ESI     <- ***
0167:0057C51B  PUSH    ECX
0167:0057C51C  MOV     EDI,10
0167:0057C521  CALL    0057CCB2    <- 将'-'后面的部分 string->Hex，到 eax
0167:0057C526  POP     ECX
0167:0057C527  CMP     ECX,EAX   <- 这里是关键的比较啦
0167:0057C529  JNZ     NEAR 0057C5D3
```

(网上校验的解除)

by `Fpc/CCG`

在联网状态下，如果你注册后反复关闭和运行几次，它会出现一个消息框，说你的帐户不正确，请与作者联系，如果不联系呢，只有自己解决了~~

首先这部分代码是在 `admunch.dll` 中，`upx` 1.20 加的壳，用 `upx` 可脱掉，并且需要把这个文件另存一份为 `admunch.old`。

设断点：`messageboxa`。拦下后 `F12` 返回在这里:

```
:0040774E FF0579C54100          inc dword ptr [0041C579]
:00407754 E844FBFFFF            call 0040729D
:00407759 E960010000            jmp 004078BE

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004075C6(C)
|

:0040775E 833D79C5410002        cmp dword ptr [0041C579], 00000002
:00407765 0F8553010000          jne 004078BE
:0040776B 803DA6544A0000        cmp byte ptr [004A54A6], 00
:00407772 0F8621010000          jbe 00407899
:00407778 8B3595544A00          mov esi, dword ptr [004A5495]
:0040777E 813E45525220          cmp dword ptr [esi], 20525245
:00407784 755F                  jne 004077E5
:00407786 833D89544A0000        cmp dword ptr [004A5489], 00000000
:0040778D 7656                  jbe 004077E5
:0040778F 83C604               add esi, 00000004
:00407792 8B3D9D544A00         mov edi, dword ptr [004A549D]
:00407798 C60700               mov byte ptr [edi], 00
:0040779B B86E153250           mov eax, 5032156E
:004077A0 050FB00FB0           add eax, B00FB00F
:004077A5 6A00                 push 00000000
:004077A7 50                   push eax
:004077A8 56                   push esi
:004077A9 FF7508               push [ebp+08]
:004077AC 6A5C                 push 0000005C
:004077AE E85DF8FFFF           call 00407010     <-
:004077B3 C605A83E440000       mov byte ptr [00443EA8], 00   <- 返回处
:004077BA B890413250           mov eax, 50324190
:004077BF 050FB00FB0           add eax, B00FB00F

* Possible Reference to Dialog: DialogID_0001
                                |
:004077C4 6A01                 push 0000001
:004077C6 68A83E4400           push 00443EA8

* Possible Reference to Dialog: DialogID_0001
                                |
:004077CB 6A01                 push 0000001
:004077CD 6A00                 push 00000000
:004077CF 50                   push eax
:004077D0 FF357B494600         push dword ptr [0046497B]
:004077D6 68D4000000           push 000000D4
:004077DB E830F8FFFF           call 00407010     <- 还是调用这个 call，不过不显示消息框了
:004077E0 E9B4000000           jmp 00407899

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00407784(C), :0040778D(C)
|
:004077E5 803DCFFB410001        cmp byte ptr [0041FBCF], 01
:004077EC 7505                 jne 004077F3
:004077EE E9A6000000           jmp 00407899

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004077EC(C)
|
:004077F3 8B06                 mov eax, dword ptr [esi]
:004077F5 034604               add eax, dword ptr [esi+04]
:004077F8 034608               add eax, dword ptr [esi+08]
:004077FB 03460C               add eax, dword ptr [esi+0C]
```

好的，在显示消息框之前的代码处设下断点，重新进行注册。再次拦下后，跟进那个 `call`，可见到作者是聪明的：

```
* Referenced by a CALL at Addresses:
|:00401182  , :00401191  , :004011D6  , :004011E2  , :004011F1
... ... 这样的调用有几百处之多，为什么？
|:0040CD90
|

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040C730(U)
|

:00407010 59                   pop ecx
:00407011 58                   pop eax
:00407012 51                   push ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407025(U)
|

:00407013 833D98364A0000        cmp dword ptr [004A3698], 00000000
:0040701A 750B                 jne 00407027
:0040701C 60                   pushad
:0040701D 6A64                 push 00000064

* Reference To: KERNEL32.Sleep, Ord:0000h
                                |
:0040701F E8245E0000           Call 0040CE48
:00407024 61                   popad
:00407025 EBEC                 jmp 00407013

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040701A(C)
|

:00407027 56                   push esi
:00407028 57                   push edi
:00407029 53                   push ebx
:0040702A 8BD8                 mov ebx, eax

* Possible StringData Ref from Data Obj ->"y+m+d:氩擡E 鷘>鮾 z 貔硭楢傰€?g?ydN"
                                ->"?4 儉|w 桏踑翛 X??恂 v?g<鐷絏 i 寔獝\\熙 R蘒?"
                                ->"Q 絺    鐇篂蛐睷 v 薚-澐８?窖滯 jY 鐇鶡 8 氾 N?崭楳 0%?
                                ->"1 雉鄇 y 鸡?鯀 L5 桦€羂翎琑 n.鏛 L.ez>,)yq1 結 wu%?
                                ->" <錇 YL 盻眢胁"
                                |
:0040702C BECBC24100           mov esi, 0041C2CB
:00407031 8DBBCBC24100         lea edi, dword ptr [ebx+0041C2CB]
:00407037 33D2                 xor edx, edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407044(U)
|

:00407039 3BF7                 cmp esi, edi
:0040703B 7309                 jnb 00407046
:0040703D AD                   lodsd
:0040703E 83F800               cmp eax, 00000000
:00407041 7501                 jne 00407044
:00407043 42                   inc edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407041(C)
|

:00407044 EBF3                 jmp 00407039

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040703B(C)
|

:00407046 C1E202               shl edx, 02
:00407049 8B82E0364A00         mov eax, dword ptr [edx+004A36E0]
:0040704F A39C364A00           mov dword ptr [004A369C], eax
:00407054 8B83CBC24100         mov eax, dword ptr [ebx+0041C2CB]
:0040705A 3504839278           xor eax, 78928304
:0040705F 6A00                 push 00000000
:00407061 50                   push eax
:00407062 FF359C364A00         push dword ptr [004A369C]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004071A7(U)
|

:00407068 8B3424               mov esi, dword ptr [esp]
```



:0040706B `66813E4D5A` `cmp word ptr [esi], 5A4D`

:00407070 `7571` `jne 004070E3`

:00407072 `90` `nop`

:00407073 `90` `nop`

:00407074 `90` `nop`

:00407075 `90` `nop`

:00407076 `8BD6` `mov edx, esi`

:00407078 `03563C` `add edx, dword ptr [esi+3C]`

:0040707B `813A50450000` `cmp dword ptr [edx], 00004550`

:00407081 `7560` `jne 004070E3`

:00407083 `8B5278` `mov edx, dword ptr [edx+78]`

:00407086 `03D6` `add edx, esi`

:00407088 `8B4A18` `mov ecx, dword ptr [edx+18]`

:0040708B `8B5A20` `mov ebx, dword ptr [edx+20]`

:0040708E `03DE` `add ebx, esi`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:004070AC(C)` |

:00407090 `8B7C8BFC` `mov edi, dword ptr [ebx+4*ecx-04]`

:00407094 `03FE` `add edi, esi`

:00407096 `33C0` `xor eax, eax`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:004070A3(U)` |

:00407098 `803F00` `cmp byte ptr [edi], 00`

:0040709B `7408` `je 004070A5`

:0040709D `3207` `xor al, byte ptr [edi]`

:0040709F `C1C004` `rol eax, 04`

:004070A2 `47` `inc edi`

:004070A3 `EBF3` `jmp 00407098`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:0040709B(C)` |

:004070A5 `3B442404` `cmp eax, dword ptr [esp+04]`

:004070A9 `7405` `je 004070B0`

:004070AB `49` `dec ecx`

:004070AC `75E2` `jne 00407090`

:004070AE `EB33` `jmp 004070E3`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:004070A9(C)` |

:004070B0 `8B4224` `mov eax, dword ptr [edx+24]`

:004070B3 `03C6` `add eax, esi`

:004070B5 `0FB74448FE` `movzx eax, word ptr [eax+2*ecx-02]`

:004070BA `8B521C` `mov edx, dword ptr [edx+1C]`

:004070BD `03D6` `add edx, esi`

:004070BF `8B0482` `mov eax, dword ptr [edx+4*eax]`

:004070C2 `8BD6` `mov edx, esi`

:004070C4 `03563C` `add edx, dword ptr [esi+3C]`

:004070C7 `3B4278` `cmp eax, dword ptr [edx+78]`

:004070CA `7211` `jb 004070DD`

:004070CC `8B5A78` `mov ebx, dword ptr [edx+78]`

:004070CF `035A7C` `add ebx, dword ptr [edx+7C]`

:004070D2 `3BC3` `cmp eax, ebx`

:004070D4 `7707` `ja 004070DD`

:004070D6 `03C6` `add eax, esi`

:004070D8 `33C9` `xor ecx, ecx`

:004070DA `41` `inc ecx`

:004070DB `EB08` `jmp 004070E5`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

| `:004070CA(C), :004070D4(C)` |

:004070DD `03C6` `add eax, esi`

:004070DF `33C9` `xor ecx, ecx`

:004070E1 `EB02` `jmp 004070E5`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

| `:00407070(C), :00407081(C), :004070AE(U)` |

:004070E3 `33C0` `xor eax, eax`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

| `:004070DB(U), :004070E1(U)` |

:004070E5 `83C40C` `add esp, 0000000C`

:004070E8 `837C24FC00` `cmp dword ptr [esp-04], 00000000`

:004070ED `7404` `je 004070F3`

:004070EF `FF6424FC` `jmp [esp-04]`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:004070ED(C)` |

:004070F3 `83F800` `cmp eax, 00000000`

:004070F6 `7517` `jne 0040710F`

:004070F8 `6A00` `push 00000000`

:004070FA `6A00` `push 00000000`

* Possible StringData Ref from Data Obj -> `"GetProc error"`

:004070FC `68DDC44100` `push 0041C4DD`

:00407101 `6A00` `push 00000000`

* Reference To: `USER32.MessageBoxA`, Ord:`0000h`

:00407103 `E89E5C0000` `Call 0040CDA6`

:00407108 `6A00` `push 00000000`

* Reference To: `KERNEL32.ExitProcess`, Ord:`0000h`

:0040710A `E84B5D0000` `Call 0040CE5A`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:004070F6(C)` |

:0040710F `83F901` `cmp ecx, 00000001`

:00407112 `0F85B9000000` `jne 004071D1`

:00407118 `8B1D04C54100` `mov ebx, dword ptr [0041C504]`

:0040711E `C1E308` `shl ebx, 08`

:00407121 `81C36A184E00` `add ebx, 004E186A`

:00407127 `FF0D04C54100` `dec dword ptr [0041C504]`

:0040712D `750A` `jne 00407139`

* Possible Reference to Dialog: `DialogID_000F`

:0040712F `C70504C541000F000000` `mov dword ptr [0041C504], 0000000F`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

| `:0040712D(C), :004071CB(C)` |

:00407139 `8D7B08` `lea edi, dword ptr [ebx+08]`

:0040713C `8BF0` `mov esi, eax`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:00407144(U)` |

:0040713E `AC` `lodsb`

:0040713F `3C2E` `cmp al, 2E`

:00407141 `7403` `je 00407146`

:00407143 `AA` `stosb`

:00407144 `EBF8` `jmp 0040713E`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:00407141(C)` |

:00407146 `C60700` `mov byte ptr [edi], 00`

:00407149 `33C9` `xor ecx, ecx`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:00407155(U)` |

:0040714B `AC` `lodsb`

:0040714C `3C00` `cmp al, 00`

:0040714E `7407` `je 00407157`

:00407150 `32C8` `xor cl, al`

:00407152 `C1C104` `rol ecx, 04`

:00407155 `EBF4` `jmp 0040714B`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:0040714E(C)` |

:00407157 `890B` `mov dword ptr [ebx], ecx`

:00407159 `8D4308` `lea eax, dword ptr [ebx+08]`

:0040715C `50` `push eax`

* Reference To: `KERNEL32.LoadLibraryA`, Ord:`0000h`

:0040715D `E8C25C0000` `Call 0040CE24`

:00407162 `83F800` `cmp eax, 00000000`

:00407165 `7517` `jne 0040717E`

:00407167 `6A00` `push 00000000`

:00407169 `6A00` `push 00000000`

* Possible StringData Ref from Data Obj -> `"LoadLibrary error"`

:0040716B `68CBC44100` `push 0041C4CB`

:00407170 `6A00` `push 00000000`

* Reference To: `USER32.MessageBoxA`, Ord:`0000h`

:00407172 `E82F5C0000` `Call 0040CDA6`

:00407177 `6A00` `push 00000000`

* Reference To: `KERNEL32.ExitProcess`, Ord:`0000h`

:00407179 `E8DC5C0000` `Call 0040CE5A`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:00407165(C)` |

:0040717E `894304` `mov dword ptr [ebx+04], eax`

:00407181 `BEE0364A00` `mov esi, 004A36E0`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:



```
:0040718F(C)   

:00407186 AD                          lodsd   

:00407187 83F800                       cmp eax, 00000000   

:0040718A 740D                         je 00407199   

:0040718C 3B4304                       cmp eax, dword ptr [ebx+04]   

:0040718F 75F5                         jne 00407186   

:00407191 FF7304                       push [ebx+04]   

* Reference To: KERNEL32.FreeLibrary, Ord:0000h   

:00407194 E8D35C0000                   Call 0040CE6C   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0040718A(C)   

:00407199 8B4304                       mov eax, dword ptr [ebx+04]   

:0040719C 8946FC                       mov dword ptr [esi-04], eax   

:0040719F 68AC714000                   push 004071AC   

:004071A4 FF33                         push dword ptr [ebx]   

:004071A6 50                           push eax   

:004071A7 E9BCFeFFFF                   jmp 00407068   

:004071AC 83F800                       cmp eax, 00000000   

:004071AF 7517                         jne 004071C8   

:004071B1 6A00                         push 00000000   

:004071B3 6A00                         push 00000000   

* Possible StringData Ref from Data Obj -> "Redirected GetProc error"   

:004071B5 68EBC44100                   push 0041C4EB   

:004071BA 6A00                         push 00000000   

* Reference To: USER32.MessageBoxA, Ord:0000h   

:004071BC E8E55B0000                   Call 0040CDA6   

:004071C1 6A00                         push 00000000   

* Reference To: KERNEL32.ExitProcess, Ord:0000h   

:004071C3 E8925C0000                   Call 0040CE5A   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:004071AF(C)   

:004071C8 83F901                       cmp ecx, 00000001   

:004071CB 0F8468FFFFFF                 je 00407139   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00407112(C)   

:004071D1 5B                           pop ebx   

:004071D2 5F                           pop edi   

:004071D3 5E                           pop esi   

:004071D4 FFE0                         jmp eax
```

动态跟踪几次得知，程序动态取得 `window api` 的地址，放到 `eax` 中，最后是 `jmp eax` 跳向那里，怎么样，是不是有点爽？重新看过上面主线中的代码：

```
:004077A5 6A00                         push 00000000   
    push 0   

:004077A7 50                           push eax   
    [eax]="registration error", messageboxa 的参数，标题   

:004077A8 56                           push esi   
    [esi]="Your account is not valid..", 内容   

:004077A9 FF7508                       push [ebp+08]   
    句柄   

:004077AC 6A5C                         push 0000005C   
    这个参数其实是决定了该调用什么 api，因为在 call 之中用它来取得 api 的地址，在这里 5c=messageboxa   

:004077AE E85DF8FFFF                   call 00407010   

* Possib
```
