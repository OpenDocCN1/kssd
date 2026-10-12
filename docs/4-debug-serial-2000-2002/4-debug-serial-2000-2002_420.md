
# OICQ HACK 1.0

> 原文：[`www.pediy.com/kssd/pediy03/forum881.htm`](https://www.pediy.com/kssd/pediy03/forum881.htm)

OICQ HACK 1.0 B 可执行文件：`OicqHack.exe`

此程序未注册将无法开启多号码探测和号码文件列表模式。

演算方式：将用户输入的 `Register` 进行演算（输入的前 6 位不参于演算），将程序的 `Serial` 进行演算（同 `Award BIOS` 密码演算方法一样），比较两次演算的结果，相同则注册成功。

由于倒算注册码嫌麻烦，改用修改法：

## 破解过程如下

```
:00405C65 8935EC5E4400                 mov dword ptr [00445EEC], esi
:00405C6B 8BD6                         mov edx, esi
:00405C6D 83C224                       add edx, 00000024
:00405C70 B911000000                   mov ecx, 00000011
:00405C75 8B83D4010000                 mov eax, dword ptr [ebx+000001D4]
:00405C7B E8D48A0000                   call 0040E754
:00405C80 A1EC5E4400                   mov eax, dword ptr [00445EEC]
:00405C85 50                           push eax
:00405C86 E845F9FFFF                   call 004055D0————————————————进入总比较 call
:00405C8B 59                           pop ecx
:00405C8C 85C0                         test eax, eax————————————————为 0 就为未注册
:00405C8E 7415                         je 00405CA5
:00405C90 6A00                         push 00000000

* Possible StringData Ref from Data Obj ->"Registration Successed"
                                        |
:00405C92 68DF144400                   push 004414DF

* Possible StringData Ref from Data Obj ->"Thanks for your registration, "————————通过
                                        ->"all limits are removed now."
                                        |
:00405C97 68A5144400                   push 004414A5
:00405C9C 6A00                         push 00000000

* Reference To: USER32.MessageBoxA, Ord:0000h
                                        |
:00405C9E E8A3700300                   Call 0043CD46
:00405CA3 EB13                         jmp 00405CB8

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405C8E(C)
|
:00405CA5 6A00                         push 00000000

* Possible StringData Ref from Data Obj ->"Registration failed"
                                        |
:00405CA7 680F154400                   push 0044150F

* Possible StringData Ref from Data Obj ->"Incorrect register code."————————未通过
                                        |
:00405CAC 68F6144400                   push 004414F6
:00405CB1 6A00                         push 00000000

* Reference To: USER32.MessageBoxA, Ord:0000h
                                        |
:00405CB3 E88E700300                   Call 0043CD46

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405CA3(U)
|
:00405CB8 6A03                         push 00000003
:00405CBA 8B15EC5E4400                 mov edx, dword ptr [00445EEC]
:00405CC0 52                           push edx
:00405CC1 E8EEF8FFFF                   call 004055B4
:00405CC6 83C408                       add esp, 00000008
:00405CC9 8BC3                         mov eax, ebx
:00405CCB E8203E0100                   call 00419AF0
:00405CD0 8B55D8                       mov edx, dword ptr [ebp\-28]
:00405CD3 64891500000000               mov dword ptr fs:[00000000], edx
:00405CDA 5E                           pop esi
:00405CDB 5B                           pop ebx
:00405CDC 8BE5                         mov esp, ebp
:00405CDE 5D                           pop ebp
:00405CDF C3                           ret
```

### 总比较 call

```
:004055D0 55                           push ebp
:004055D1 8BEC                         mov ebp, esp
:004055D3 53                           push ebx
:004055D4 56                           push esi
:004055D5 57                           push edi
:004055D6 8B5D08                       mov ebx, dword ptr [ebp+08]
:004055D9 8D4304                       lea eax, dword ptr [ebx+04]
:004055DC 50                           push eax
:004055DD 53                           push ebx
:004055DE E8E5000000                   call 004056C8
:004055E3 83C408                       add esp, 00000008
:004055E6 8D7324                       lea esi, dword ptr [ebx+24]
:004055E9 83C606                       add esi, 00000006
:004055EC 56                           push esi
:004055ED E8DA340300                   call 00438ACC————————————此 call 将用户输入的 Register 进行演算
:004055F2 59                           pop ecx
:004055F3 8BF8                         mov edi, eax                     由于程序此后还将多次调用此 call 对用户输入的 Register 进行计算并比较
:004055F5 897B44                       mov dword ptr [ebx+44], edi      所以改下面的那个 jne 是没有用的，最好的方法是改这个 call
:004055F8 8D4304                       lea eax, dword ptr [ebx+04]
:004055FB 50                           push eax
:004055FC 53                           push ebx
:004055FD E89A000000                   call 0040569C————————————此 call 将程序的 Serial 进行演算，我的 Serial 是：OH100B4003312064，经此 call 演算后为 45303DA5
:00405602 83C408                       add esp, 00000008
:00405605 3BF8                         cmp edi, eax
:00405607 751A                         jne 00405623
:00405609 8D5324                       lea edx, dword ptr [ebx+24]
:0040560C 52                           push edx
:0040560D 53                           push ebx
:0040560E E82D040000                   call 00405A40
:00405613 83C408                       add esp, 00000008
:00405616 C70301000000                 mov dword ptr [ebx], 00000001
:0040561C B801000000                   mov eax, 00000001
:00405621 EB06                         jmp 00405629

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405607(C)
|
:00405623 33D2                         xor edx, edx
:00405625 33C0                         xor eax, eax
:00405627 8913                         mov dword ptr [ebx], edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405621(U)
|
:00405629 5F                           pop edi
:0040562A 5E                           pop esi
:0040562B 5B                           pop ebx
:0040562C 5D                           pop ebp
:0040562D C3                           ret
```

### 此 call 将用户输入的 Register 进行演算（有 N 多个地方将调用它，也就可知演算不只一次）

```
* Referenced by a CALL at Addresses:
|:004016B8  , :004017E3  , :00401F0F  , :00401F19  , :004023A5
|:004023AF  , :004023D1  , :004037E2  , :004037F3  , :0040389E
|:004038AA  , :00403BC4  , :00403C51  , :0040439E  , :004043A7
|:004047EC  , :00404815  , :0040483E  , :00404977  , :00404986
|:004049C5  , :004049D1  , :00405560  , :004055ED  , :00438B3B
|

:00438ACC 55                           push ebp
:00438ACD 8BEC                         mov ebp, esp
:00438ACF 53                           push ebx
:00438AD0 56                           push esi
:00438AD1 57                           push edi
:00438AD2 33FF                         xor edi, edi
:00438AD4 8B7508                       mov esi, dword ptr [ebp+08]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00438AE6(C)
|
:00438AD7 8A1E                         mov bl, byte ptr [esi]
:00438AD9 46                           inc esi
:00438ADA 0FBEC3                       movsx eax, bl
:00438ADD 50                           push eax
:00438ADE E8DD1B0000                   call 0043A6C0
:00438AE3 59                           pop ecx
:00438AE4 85C0                         test eax, eax
:00438AE6 75EF                         jne 00438AD7
:00438AE8 80FB2B                       cmp bl, 2B
:00438AEB 7405                         je 00438AF2
:00438AED 80FB2D                       cmp bl, 2D
:00438AF0 750E                         jne 00438B00

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00438AEB(C)
|
:00438AF2 80FB2D                       cmp bl, 2D
:00438AF5 0F94C0                       sete al
:00438AF8 83E001                       and eax, 00000001
:00438AFB 8A1E                         mov bl, byte ptr [esi]
:00438AFD 46                           inc esi
:00438AFE EB18                         jmp 00438B18

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00438AF0(C)
|
```



# 汇编代码与修改说明

## 原始汇编代码片段

```assembly
:00438B00 33C0                 xor eax, eax

:00438B02 EB14                 jmp 00438B18

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00438B20(C)

:00438B04 0FBECB               movsx ecx, bl

:00438B07 8BD7                 mov edx, edi

:00438B09 8A1E                 mov bl, byte ptr [esi]

:00438B0B 03D2                 add edx, edx

:00438B0D 8D1492               lea edx, dword ptr [edx+4*edx]

:00438B10 03D1                 add edx, ecx

:00438B12 83C2D0               add edx, FFFFFFD0

:00438B15 46                   inc esi

:00438B16 8BFA                 mov edi, edx

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:00438AFE(U), :00438B02(U)

:00438B18 80FB30               cmp bl, 30

:00438B1B 7C05                 jl 00438B22

:00438B1D 80FB39               cmp bl, 39

:00438B20 7EE2                 jle 00438B04

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00438B1B(C)

:00438B22 85C0                 test eax, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00438B24(C)
```

## 代码修改与注释

将上述代码中的相关部分修改如下，并将原本的逻辑替换为赋值指令和填充操作，以便最终返回指定值：

```assembly
:00438B22 85C0                 test eax, eax
; 注意：此处原意可能被改为直接赋值，见下方最终整理

:00438B24 7406                 je 00438B2C
:00438B26 8BC7                 mov eax, edi
:00438B28 F7D8                 neg eax
:00438B2A EB02                 jmp 00438B2E
```

**修改说明**：
*   将 `:00438B22` 处的逻辑忽略，改为目标值 `mov eax, 45303DA5` 的等效替换或后续流程调整。
*   在 `:00438B24` 至 `:00438B2A` 之间插入足够的 `nop` 指令以对齐字节数。
*   最终目标：退出此 `call` 后，`eax` 寄存器值为 `45303DA5`。

## 后续汇编代码

```assembly
:00438B2C 8BC7                 mov eax, edi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00438B2A(U)

:00438B2E 5F                   pop edi

:00438B2F 5E                   pop esi

:00438B30 5B                   pop ebx

:00438B31 5D                   pop ebp

:00438B32 C3                   ret
```

## 最终整理与操作指导

1.  **工具**：使用 `UltraEdit` 载入 `OicqHack.exe`。
2.  **查找字符串**（十六进制）：
    ```text
    7c 05 80 fb 39 7e e2 85 c0 74 06 8b c7 f7 d8 eb 02 8b c7 5f 5e 5b 5d
    ```
3.  **替换为**：
    ```text
    B8 a5 3d 30 45 90 90 90 90 90 90 90 5f 5e 5b 5d
    ```
    *注：`B8 a5 3d 30 45` 对应指令 `mov eax, 0x45303DA5`，随后的 `90` 是 `nop` 指令用于填充空间，末尾保留原有的 `pop` 序列和 `ret`。根据原意，前缀保持不变的部分用 `-` 表示，这里为了清晰直接给出修改后的关键字节序列。原注释中提到 “- 为不改变”，实际修改是将中间计算部分替换为直接赋值 `eax` 并填充 `nop`。*

    **精确替换逻辑**：
    原查找串包含 `85 c0` (test eax, eax) 直到 `5d` (pop ebp)。
    修改意图是让 `eax` 直接变为 `45303DA5`。
    因此替换后的字节流应为：
    `B8 A5 3D 30 45` (mov eax, 45303DA5) + 若干 `90` (nop) 以填补剩余空间，直到原本的 `pop` 序列开始或结束。

4.  **完成修改**。

## 附加说明

*   将程序的 `Serial` 改成 `0`，注册码填 `0` 即为注册版。
*   发现存在在先计算 `oicqpass` 的逻辑，但未完全破解，避免泄露核心算法。
*   MD5 校验值：`4dd466d0e1e0cc123afbe504da2462ca`
*   提示：以上步骤后可能仍有问题，需自行调试。
