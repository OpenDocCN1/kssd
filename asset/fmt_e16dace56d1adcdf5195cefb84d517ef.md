
### 004F53C1 - 004F53C6

`:004F53C1` `7E03` `jle 004F53C6`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  * `:004F53BC(C)`

### 004F53C3 - 004F53C5

`:004F53C3` `32C0` `xor al, al`
`:004F53C5` `C3` `ret`

### 004F53C6 - 004F53C9

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  * `:004F53B7(C)`, `:004F53C1(C)`

`:004F53C6` `83F841` `cmp eax, 00000041`
`:004F53C9` `7C0F` `jl 004F53DA`

### 004F53CB - 004F53CE

`:004F53CB` `83F85A` `cmp eax, 0000005A`
`:004F53CE` `7F0A` `jg 004F53DA`

### 004F53D0 - 004F53D3

```
:004F53D0 83C0BF                 add eax, FFFFFFBF
                                 ====> EAX=4E - 41=D
                                 ====> EAX=53 - 41=12

:004F53D3 8A8028905000           mov al, byte ptr [eax+00509028]
                                 ====> 根据 EAX 值从 [00509028] 的表中取值！
                                 ====> AL=01
                                 ====> AL=01
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

`[00509028]`内存中是一张表：

`00509028` `01 01 00 01 01 01 01 01 01 01 01 00 01 01 01 01`

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

### 004F53D9

`:004F53D9` `C3` `ret`

这 2 次取值运算的作用不太清楚，可能是检测第 13 位、19 位的字符有没有踩上程序预埋的“地雷”吧？
因为此处内存中有的值是 `00`，如果你很走运就碰上此处的话，恭喜你：“`Game Over`”了！~@~~@~

### 004F53DA - 004F53E3

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  * `:004F53C9(C)`, `:004F53CE(C)`

```
:004F53DA 83C09F                 add eax, FFFFFF9F
:004F53DD 8A8028905000           mov al, byte ptr [eax+00509028]
:004F53E3 C3                      ret
```

关键 CALL 二 结 束

□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□
◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆
关键 CALL 三 开 始

### 进入关键 CALL：`004AD889`

`call 004F57F9`

* Referenced by a CALL at Addresses:
  * `:004AD889`, `:004ADE7F`, `:004C5E33`, `:004C77D2`, `:004C781F`
  * `:004D2871`, `:004D3199`, `:004D3546`, `:004D4A1C`, `:004D4CB7`
  * `:004D4F5A`

### 004F57F9 - 004F57FC

```
:004F57F9 53                      push ebx
:004F57FA 56                      push esi
:004F57FB 57                      push edi
:004F57FC 8B7C2410                mov edi, dword ptr [esp+10]
                                  ====> EDI=AE315678-ABCD-EFGHIJD4
```

因为在下面的运算中第 7 位和第 12 位必须满足几个条件，所以分析了几遍后把第 7 位改为 `4`，第 12 位改为 `3`。

`AE315678-ABCD-EFGHIJD4` 改为 `AE315648-AB3D-EFGHIJD4`

### 004F5800 - 004F5811

```
:004F5800 0FBE4706                movsx eax, byte ptr [edi+06]
                                  ====> EAX=34  取试炼码第 7 位字符的 HEX 值

:004F5804 83F830                 cmp eax, 00000030
:004F5807 7C0A                    jl 004F5813
:004F5809 83F839                 cmp eax, 00000039
:004F580C 7F05                    jg 004F5813
:004F580E 8D58D0                  lea ebx, dword ptr [eax-30]
                                  ====> 如果第 7 位字符是数字则此处-30
                                  ====> EBX=34 - 30=4

:004F5811 EB15                    jmp 004F5828
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  * `:004F5807(C)`, `:004F580C(C)`

### 004F5813 - 004F5825

```
:004F5813 83F841                  cmp eax, 00000041
:004F5816 0F8C92000000            jl 004F58AE
:004F581C 83F846                  cmp eax, 00000046
:004F581F 0F8F89000000            jg 004F58AE
:004F5825 8D58C9                  lea ebx, dword ptr [eax-37]
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  * `:004F5811(U)`

### 004F5828 - 004F5839

```
:004F5828 0FBE470B                movsx eax, byte ptr [edi+0B]
                                  ====> EAX=33  取试炼码第 12 位字符的 HEX 值

:004F582C 83F830                 cmp eax, 00000030
:004F582F 7C0A                    jl 004F583B
:004F5831 83F839                 cmp eax, 00000039
:004F5834 7F05                    jg 004F583B
:004F5836 8D70D0                  lea esi, dword ptr [eax-30]
                                  ====> 如果第 12 位字符是数字则此处-30
                                  ====> ESI=33 - 30=3

:004F5839 EB0D                    jmp 004F5848
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  * `:004F582F(C)`, `:004F5834(C)`

### 004F583B - 004F5845

```
:004F583B 83F841                  cmp eax, 00000041
:004F583E 7C6E                    jl 004F58AE
:004F5840 83F846                  cmp eax, 00000046
:004F5843 7F69                    jg 004F58AE
:004F5845 8D70C9                  lea esi, dword ptr [eax-37]
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  * `:004F5839(U)`

### 004F5848 - 004F585A

```
:004F5848 57                      push edi
:004F5849 E822FCFFFF              call 004F5470
                                  ====> 关键 CALL！⑦进入！

:004F584E 0FBE4F13                movsx ecx, byte ptr [edi+13]
                                  ====> ECX=4A  取试炼码第 20 位字符的 HEX 值

:004F5852 25FF000000              and eax, 000000FF
:004F5857 83C404                  add esp, 00000004
:004F585A 3BC8                    cmp ecx, eax
```

* `ECX=4A` 试炼码第 22 位字符的 `HEX` 值
* `EAX=37` 上面 `4F5849` 处运算得出的值
* 所以注册码的第 20 位应是 `7`

因此把试炼码 `AE315648-AB3D-EFGHIJD4` 再次修改为 `AE315648-AB3D-EFGHI7D4`

好了，`GAME OVER`！不，是终于成功了！！^O^^O^^O^^O^

### 004F585C - 004F589D

```
:004F585C 7550                    jne 004F58AE
:004F585E C1E604                  shl esi, 04
                                  ====> ESI=3 SHL 4=30

:004F5861 03F3                    add esi, ebx
                                  ====> ESI=30 + 4=34

:004F5863 B90D000000             mov ecx, 0000000D
                                  ====> ECX=D

:004F5868 8BC6                    mov eax, esi
:004F586A 99                      cdq
:004F586B F7F9                    idiv ecx
                                  ====> EDX=34 % D=0

:004F586D 85D2                    test edx, edx
                                  ====> 余数是否为 0？此处余数必须为 0！

:004F586F 753D                    jne 004F58AE
                                  ====> 跳则 OVER！

:004F5871 B84FECC44E             mov eax, 4EC4EC4F
:004F5876 F7EE                    imul esi
:004F5878 C1FA02                  sar edx, 02
:004F587B 8BC2                    mov eax, edx
:004F587D C1E81F                  shr eax, 1F
:004F5880 03D0                    add edx, eax
:004F5882 83FA03                  cmp edx, 00000003
:004F5885 7C17                    jl 004F589E
:004F5887 83FA09                  cmp edx, 00000009
:004F588A 7F12                    jg 004F589E

:004F588C B801000000             mov eax, 00000001
:004F5891 33C9                    xor ecx, ecx
:004F5893 3BC0                    cmp eax, eax
:004F5895 5F                      pop edi
:004F5896 0F94C1                  sete cl
:004F5899 5E                      pop esi
:004F589A 8AC1                    mov al, cl
:004F589C 5B                      pop ebx
:004F589D C3                      ret
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  * `:004F5885(C)`, `:004F588A(C)`

### 004F589E - 004F58AD

```
:004F589E 33C0                    xor eax, eax
:004F58A0 33C9                    xor ecx, ecx
:004F58A2 83F801                  cmp eax, 00000001
:004F58A5 5F                      pop edi
:004F58A6 0F94C1                  sete cl
:004F58A9 5E                      pop esi
:004F58AA 8AC1                    mov al, cl
:004F58AC 5B                      pop ebx
:004F58AD C3                      ret
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  * `:004F5816(C)`, `:004F581F(C)`, `:004F583E(C)`, `:004F5843(C)`, `:004F585C(C)`
  * `:004F586F(C)`

### 004F58AE - 004F58B3

```
:004F58AE 5F                      pop edi
:004F58AF 5E                      pop esi
:004F58B0 32C0                    xor al, al
:004F58B2 5B                      pop ebx
:004F58B3 C3                      ret
```

—————————————————————————————————

### 进入关键 CALL⑦：`004F5849`

`call 004F5470`

* Referenced by a CALL at Address:
  * `:004F5849`

### 004F5470

`:004F5470` `8B442404` `mov eax, dword ptr [esp+04]`
