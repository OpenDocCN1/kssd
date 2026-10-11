
# Mp3 Tag Clinic 1.50

> 原文：[`www.pediy.com/kssd/pediy03/forum1267.htm`](https://www.pediy.com/kssd/pediy03/forum1267.htm)

注册 `Mp3 Tag Clinic 1.50`

by `Fpc[CCG]/6767[BCG]` @2001/08

tools: `si`, `frogice`

level: `1.5`

`[Begin]`

这是在 BCG 论坛上请求破解的，难度不大，但我是在放弃又拣起才给破掉的。破解真是费力不讨好 :(

基本上这个软件是不利于 cracker 的：频繁 S 机。S 了十几次之后终于把它的注册机制找到，下面是过程。

以下简称 `MTC`，由 `aspack` 压缩，用 `craspr` 脱壳会有一个函数找不到。算了，追出注册码是目标。

如果运气好，`MTC` 运行后没 S 掉，会出现一个带注册按纽的‘关于’窗口。单击‘注册’到注册窗口，有两个输入框：上面的一个是根据硬件算出来的(实际上是计算机名和 Windows 的序列号)，下面的是要求你输入注册码。好了，输入：`1236549870`，`^D` 到 `SI`，设断点：`bpx hmemcpy`。回来，点确定，被拦到，Good!

按几次 `F12` 直到你见到下面的代码：

```
:004B24B6 8D55F4                  lea edx, dword ptr [ebp-0C]
:004B24B9 8B83D4020000            mov eax, dword ptr [ebx+02D4]
:004B24BF E81833F8FF              call 004357DC             <- 这个是读取输入的注册码(Irc)
:004B24C4 8B55F4                  mov edx, dword ptr [ebp-0C]      <- 返回处在此
:004B24C7 B83C565700              mov eax, 0057563C
:004B24CC E8FB18F5FF              call 00403DCC             <- 复制到 0x57563C 处
:004B24D1 66837DFA01              cmp word ptr [ebp-06], 0001
:004B24D6 0F85D1020000            jne 004B27AD              <- 不会跳
:004B24DC A1A4625700              mov eax, dword ptr [005762A4]
:004B24E1 C70005000000            mov dword ptr [eax], 00000005
:004B24E7 A13C565700              mov eax, dword ptr [0057563C]
:004B24EC E8071BF5FF              call 00403FF8             <- 取 Irc 长度
:004B24F1 83F810                  cmp eax, 00000010        <- 不能大于 16
:004B24F4 7D11                    jge 004B2507
:004B24F6 B83C565700              mov eax, 0057563C        <- eax 指向 Irc
:004B24FB BA7CDE0000              mov edx, 0000DE7C        <- 这个是常量
:004B2500 E813AFFAFF              call 0045D418             <- **!!** 这个是核心了；若返回的 eax 值小于等于 0 或大于 0x1fff 则注册失败
:004B2505 8BF8                    mov edi, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B24F4(C)
|

:004B2507 85FF                    test edi, edi             <- 检查返回值
:004B2509 0F8E55020000            jle 004B2764              <- 这里不能去
:004B250F 81FF00200000            cmp edi, 00002000
:004B2515 0F8D49020000            jnl 004B2764
... ...(不重要了，省略)
```

在 `:004B2500` 换 `F8`，追进 `call 0045D418`，看 `MTC` 是如何计算注册码。

* Referenced by a `CALL` at `Addresses`:
`|... ...`

```
:0045D418 55                      push ebp
:0045D419 8BEC                    mov ebp, esp
:0045D41B 83C4BC                  add esp, FFFFFFBC
... ...(初始化)
:0045D453 8D0568D44500            lea eax, dword ptr [0045D468]
:0045D459 83C00A                  add eax, 0000000A
:0045D45C 8905A8785700            mov dword ptr [005778A8], eax
:0045D462 EB04                    jmp 0045D468              <- **向下跳走
:0045D464 5F                      pop edi
:0045D465 5E                      pop esi
:0045D466 5B                      pop ebx
:0045D467 C3                      ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0045D462(U)
|

:0045D468 837DFCFF                cmp dword ptr [ebp-04], FFFFFFFF
:0045D46C 0F84D6010000            je 0045D648
:0045D472 E8159EFAFF              call 0040728C
:0045D477 83C02D                  add eax, 0000002D
:0045D47A A384785700              mov dword ptr [00577884], eax
:0045D47F E8089EFAFF              call 0040728C
:0045D484 3B0584785700            cmp eax, dword ptr [00577884]
:0045D48A 0F87B8010000            ja 0045D648               <- 这里不跳，不重要
:0045D490 33F6                    xor esi, esi
:0045D492 E891FCFFFF              call 0045D128             <- 这里会生成机器号(Hwn)，并以立即数的形式保存在 eax 中
                             <- 规则是：将在注册窗口看到的机器号的前 8 个字符(当然不包括‘-’)变为立即数
                             <- 比如：机器号=AC7D-680F-53，那么 eax=0xAC7D680F
:0045D497 8945F8                  mov dword ptr [ebp-08], eax      <- 将 Hwn 保存下来，以后用到
:0045D49A A114625700              mov eax, dword ptr [00576214]
:0045D49F 8B55FC                  mov edx, dword ptr [ebp-04]
:0045D4A2 8910                    mov dword ptr [eax], edx       <- 保存常数 0xDE7C
:0045D4A4 8D45E0                  lea eax, dword ptr [ebp-20]
:0045D4A7 8B17                    mov edx, dword ptr [edi]
:0045D4A9 E86269FAFF              call 00403E10             <- 不重要
:0045D4AE 8D45E0                  lea eax, dword ptr [ebp-20]
:0045D4B1 E85AEAFFFF              call 0045BF10             <- 复制一份 Irc，有一段代码是检查 Irc 中是否都为 HEX 字符
```

```
>>>>>>>>

  :0045BF31 8B30                  mov esi, dword ptr [eax]
  :0045BF33 09F6                  or esi, esi
  :0045BF35 7436                  je 0045BF6D
  :0045BF37 8B4EFC                mov ecx, dword ptr [esi-04]
  :0045BF3A 89F7                  mov edi, esi
  :0045BF3C E32F                  jcxz 0045BF6D
  :0045BF3E 89CB                  mov ebx, ecx
  :0045BF40 FC                    cld
  * Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0045BF61(C)
  |
  :0045BF41 AC                    lodsb
  :0045BF42 80F830                cmp al, 30
  :0045BF45 7219                  jb 0045BF60
  :0045BF47 80F839                cmp al, 39
  :0045BF4A 7613                  jbe 0045BF5F
  :0045BF4C 80F841                cmp al, 41
  :0045BF4F 720F                  jb 0045BF60
  :0045BF51 80F846                cmp al, 46
  :0045BF54 7609                  jbe 0045BF5F
  :0045BF56 660FBAF0              btr ax, F0          <- 此处在内存里不是这样的
  :0045BF5A 057303EBED            add eax, EDEB0373
  * Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  |:0045BF4A(C), :0045BF54(C)
  |
  :0045BF5F AA                    stosb
  * Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  |:0045BF45(C), :0045BF4F(C)
  |
  :0045BF60 49                    dec ecx
  :0045BF61 75DE                  jne 0045BF41
  :0045BF63 B020                  mov al, 20
  * Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0045BF6B(U)
  |
  :0045BF65 39FE                  cmp esi, edi
  :0045BF67 7404                  je 0045BF6D
  :0045BF69 AA                    stosb
  :0045BF6A 4B                    dec ebx
  :0045BF6B EBF8                  jmp 0045BF65

<<<<<<<<
```

```
:0045D4B6 8BD8                    mov ebx, eax
:0045D4B8 8D45E0                  lea eax, dword ptr [ebp-20]
:0045D4BB 8BD3                    mov edx, ebx
:0045D4BD E86A6EFAFF              call 0040432C
:0045D4C2 83C8FF                  or eax, FFFFFFFF
:0045D4C5 E88E58FAFF              call 00402D58
:0045D4CA 8945E4                  mov dword ptr [ebp-1C], eax
:0045D4CD E85AF6FFFF              call 0045CB2C            <- 取 Irc 长度
:0045D4D2 DD5DE8                  fstp qword ptr [ebp-18]
:0045D4D5 9B                      wait
:0045D4D6 83FB10                  cmp ebx, 00000010        <- 是否小于 16
:0045D4D9 0F8EB4000000            jle 0045D593              <- 小于是正常的，跳走
:0045D4DF 8D4DDC                  lea ecx, dword ptr [ebp-24]
:0045D4E2 8B07                    mov eax, dword ptr [edi]
... ...

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0045D4D9(C)
|

:0045D593 83FB0A                  cmp ebx, 0000000A        <- Irc 的长度也不能小于 10，正确值其实就是 10
:0045D596 7C76                    jl 0045D60E
:0045D598 8B0F                    mov ecx, dword ptr [edi]    <- Irc
```
