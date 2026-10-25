
# 软件：leapftp2.7.4

闲着没事，从网上拉下来分析了一下，发现这个软件注册算法很有趣。他一共有两种注册码算法。

第一种注册码和用户名无关，可以随便输入。运算手法只是将你输入假码经过运算，得出真正的注册码，格式为：

`XXXX－XXXX－XXXX－XXXX`。

第二种注册码是将用户名运算出一个注册码，然后再和一个`214065`合在一起，也就是楼主所说的在内存中看到的不变的数字。格式为：

`214065-XXXXXXXXX` \\注：这里的`XXXXXXXXX`是将用户名运算出来的，不一定是九位。

以下是我粗粗的分析了一下，不对之处请大家指教。

```
:0048742A 8B45F8                  mov eax, dword ptr [ebp-08]            \\中断在这里，此时EAX值为输入假码。
:0048742D 8D55FC                  lea edx, dword ptr [ebp-04]
:00487430 E87B16F8FF              call 00408AB0　　　　　　　　　　　　　\\这个Call虽然可疑，但不是关键点。
:00487435 80BBF402000000          cmp byte ptr [ebx+000002F4], 00
:0048743C 740E                    je 0048744C
:0048743E 8B55FC                  mov edx, dword ptr [ebp-04]
:00487441 8BC3                    mov eax, ebx
:00487443 E888030000              call 004877D0　　　　　　　　　　　　　\\第一个关健Call，算出第一种注册码，格式为：XXXX－XXXX－XXXX－XXXX。
:00487448 84C0                    test al, al
:0048744A 7526                    jne 00487472　　　　　　　　　　　　　\\一开始已经说过，这个软件有两种注册码算法，所以如果第一个注册码是对的就直接跳到注册成功处，如果第一个注册码是错的话这里将不跳继续算出第二种注册码。
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0048743C(C)

|

```
:0048744C 8B83F0020000            mov eax, dword ptr [ebx+000002F0]　　　\\第一次注册码错的话在这里继续运算。
:00487452 50                      push eax
:00487453 8D55F4                  lea edx, dword ptr [ebp-0C]
:00487456 8B83D0020000            mov eax, dword ptr [ebx+000002D0]
:0048745C E867C5FAFF              call 004339C8
:00487461 8B55F4                  mov edx, dword ptr [ebp-0C]
:00487464 8B4DFC                  mov ecx, dword ptr [ebp-04]
:00487467 8BC3                    mov eax, ebx
:00487469 E8BA010000              call 00487628　　　　　　　　　　　　\\第二个关健Call，算出第二种注册码，格式为：214065-XXXXXXXXX。
:0048746E 84C0                    test al, al
:00487470 7462                    je 004874D4　　　　　　　　　　　　　\\若第二种注册码也是错的话，这里将跳到出错点。
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0048744A(C)

|

```
:00487472 8D55F0                  lea edx, dword ptr [ebp-10]
:00487475 8B83E4020000            mov eax, dword ptr [ebx+000002E4]
:0048747B E848C5FAFF              call 004339C8
:00487480 8B45F0                  mov eax, dword ptr [ebp-10]
:00487483 50                      push eax
:00487484 8D55EC                  lea edx, dword ptr [ebp-14]
:00487487 8B83D0020000            mov eax, dword ptr [ebx+000002D0]
:0048748D E836C5FAFF              call 004339C8
:00487492 8B4DEC                  mov ecx, dword ptr [ebp-14]
:00487495 8B93EC020000            mov edx, dword ptr [ebx+000002EC]
:0048749B 8BC3                    mov eax, ebx
:0048749D E8AE040000              call 00487950
```

\* Possible StringData Ref from Code Obj ->"感谢你的注册！"

                                  |

```
:004874A2 B820754800              mov eax, 00487520
:004874A7 E87834FDFF              call 0045A924
:004874AC C7833402000001000000    mov dword ptr [ebx+00000234], 00000001
:004874B6 8D55E8                  lea edx, dword ptr [ebp-18]
:004874B9 8B83D0020000            mov eax, dword ptr [ebx+000002D0]
:004874BF E804C5FAFF              call 004339C8
:004874C4 8B55E8                  mov edx, dword ptr [ebp-18]
:004874C7 8D83E8020000            lea eax, dword ptr [ebx+000002E8]
:004874CD E806C8F7FF              call 00403CD8
:004874D2 EB15                    jmp 004874E9
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00487470(C)

|

```
:004874D4 6A00                    push 00000000
:004874D6 668B0D3C754800          mov cx, word ptr [0048753C]
:004874DD B201                    mov dl, 01
```

\* Possible StringData Ref from Code Obj ->"你输入的许可密匙是不正确的. 要确保准确, "

                                        ->"你应该直接总你的购买确认 E-Mail "

                                        ->"中复制并粘贴序列号. 如果你继续操作后碰到麻烦, "

                                        ->"请联系support@leapware.com."

                                  |

```
:004874DF B848754800              mov eax, 00487548
:004874E4 E84333FDFF              call 0045A82C　　　　　　\\跳到这里已经完了。
```

这里是第一个关健Call

```
:004877D0 55                      push ebp
:004877D1 8BEC                    mov ebp, esp
:004877D3 83C4F4                  add esp, FFFFFFF4
:004877D6 53                      push ebx
:004877D7 56                      push esi
:004877D8 57                      push edi
:004877D9 8955FC                  mov dword ptr [ebp-04], edx
:004877DC 8B45FC                  mov eax, dword ptr [ebp-04]
:004877DF E8D4C8F7FF              call 004040B8
:004877E4 33C0                    xor eax, eax
:004877E6 55                      push ebp
:004877E7 683D794800              push 0048793D
:004877EC 64FF30                  push dword ptr fs:[eax]
:004877EF 648920                  mov dword ptr fs:[eax], esp
:004877F2 C645FB00                mov [ebp-05], 00
:004877F6 8B45FC                  mov eax, dword ptr [ebp-04]
:004877F9 E806C7F7FF              call 00403F04　　　　　　　　\\这个Call得到注册码位数
:004877FE 83F813                  cmp eax, 00000013
:00487801 0F8520010000            jne 00487927　　　　　　　　　\\若注册码位数不等于十六进制13位，就跳出这个Call进行第二种注册码运算。
:00487807 8B45FC                  mov eax, dword ptr [ebp-04]
:0048780A 8078042D                cmp byte ptr [eax+04], 2D　　\\注册码第5位必须是ASLL码“-”
:0048780E 0F8513010000            jne 00487927
:00487814 8B45FC                  mov eax, dword ptr [ebp-04]
:00487817 8078092D                cmp byte ptr [eax+09], 2D　　\\注册码第10位必须是ASLL码“-”
:0048781B 0F8506010000            jne 00487927
:00487821 8B45FC                  mov eax, dword ptr [ebp-04]
:00487824 80780E2D                cmp byte ptr [eax+0E], 2D　　\\注册码第15位必须是ASLL码“-”
```

从这里开始是将注册码的第一位至十四位运算出4个值分别保存。我这里就不注释了。有兴趣的朋友自己分析分析。

```
:00487828 0F85F9000000            jne 00487927
:0048782E 33F6                    xor esi, esi
:00487830 33FF                    xor edi, edi
:00487832 33C0                    xor eax, eax
:00487834 8945F4                  mov dword ptr [ebp-0C], eax
:00487837 BB01000000              mov ebx, 00000001
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004878C2(C)

|

```
:0048783C 8BC3                    mov eax, ebx
:0048783E 2503000080              and eax, 80000003
:00487843 7905                    jns 0048784A
:00487845 48                      dec eax
:00487846 83C8FC                  or eax, FFFFFFFC
:00487849 40                      inc eax
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00487843(C)

|

```
:0048784A 85C0                    test eax, eax
:0048784C 7516                    jne 00487864
:0048784E 8B45FC                  mov eax, dword ptr [ebp-04]
:00487851 8A4418FF                mov al, byte ptr [eax+ebx-01]
:00487855 E84EFFFFFF              call 004877A8
:0048785A 84C0                    test al, al
:0048785C 0F84C5000000            je 00487927
:00487862 EB22                    jmp 00487886
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0048784C(C)

|

```
:00487864 8BC3                    mov eax, ebx
:00487866 B905000000              mov ecx, 00000005
:0048786B 99                      cdq
:0048786C F7F9                    idiv ecx
:0048786E 85D2                    test edx, edx
:00487870 7414                    je 00487886
:00487872 8B45FC                  mov eax, dword ptr [ebp-04]
:00487875 8A4418FF                mov al, byte ptr [eax+ebx-01]
:00487879 E83EFFFFFF              call 004877BC
:0048787E 84C0                    test al, al
:00487880 0F84A1000000            je 00487927
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00487862(U), :00487870(C)

|

```
:00487886 8B45FC                  mov eax, dword ptr [ebp-04]
:00487889 8A4418FF                mov al, byte ptr [eax+ebx-01]
:0048788D 3C2D                    cmp al, 2D
:0048788F 742D                    je 004878BE
:00487891 83FB05                  cmp ebx, 00000005
:00487894 7D0C                    jge 004878A2
:00487896 8B55FC                  mov edx, dword ptr [ebp-04]
```
