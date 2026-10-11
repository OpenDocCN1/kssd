
# cuteftp4.0.19

> 原文：[`www.pediy.com/kssd/pediy03/forum441.htm`](https://www.pediy.com/kssd/pediy03/forum441.htm)

**标题**: 忙了一个月后，终于有空了。闲来把`cuteftp4.0.19`的注册码追了一下，却发现其注册码与用户名好像无关。码：`a2222222222222`(`a`后共有13个`2`)，名：任意。

**内容**:

大致过程如下：

经过痛苦的追踪，来到下处：

```
:00489A76 6AFF                     push FFFFFFFF
:00489A78 6820F94F00              push 004FF920
:00489A7D 50                       push eax
:00489A7E 64892500000000          mov dword ptr fs:[00000000], esp
:00489A85 83EC44                  sub esp, 00000044
:00489A88 55                       push ebp
:00489A89 56                       push esi
:00489A8A 57                       push edi
:00489A8B 8BF1                     mov esi, ecx
:00489A8D 6A01                     push 00000001
:00489A8F E87ED00300               call 004C6B12
:00489A94 85C0                     test eax, eax
:00489A96 0F84B0050000            je 0048A04C
:00489A9C 8DBE94000000            lea edi, dword ptr [esi+00000094]
:00489AA2 6A00                     push 00000000
:00489AA4 8BCF                     mov ecx, edi
:00489AA6 C7442410FFFFFF7F        mov [esp+10], 7FFFFFFF
:00489AAE E874EA0300               call 004C8527
:00489AB3 50                       push eax    ***d eax 将看到你输入的假注册码***
:00489AB4 8D442410                 lea eax, dword ptr [esp+10]
:00489AB8 50                       push eax
:00489AB9 E822160200               call 004AB0E0      ***F8 进去***
:00489ABE 83C408                   add esp, 00000008
:00489AC1 6685C0                   test ax, ax
:00489AC4 756E                     jne 00489B34
```

下面是`489ab9`按`F8`进去后的程序段：

```
:004AB0E0 83EC20                   sub esp, 00000020
:004AB0E3 83C9FF                   or ecx, FFFFFFFF
:004AB0E6 33C0                     xor eax, eax
:004AB0E8 56                       push esi
:004AB0E9 8B74242C                 mov esi, dword ptr [esp+2C]
:004AB0ED 57                       push edi
:004AB0EE 8BFE                     mov edi, esi
:004AB0F0 F2                       repnz
:004AB0F1 AE                       scasb
:004AB0F2 F7D1                     not ecx
:004AB0F4 49                       dec ecx     ***ecx 中存放假注册码的长度***
:004AB0F5 83F90E                   cmp ecx, 0000000E    ***比较假注册码的长度是否为 14***
:004AB0F8 7573                     jne 004AB16D
:004AB0FA 56                       push esi
:004AB0FB E88A6A0000               call 004B1B8A       ***假注册码大小写转换段***

* Possible Reference to String Resource ID=00014: "Paste Url"

                                    |
:004AB100 6A0E                     push 0000000E
:004AB102 8D442420                 lea eax, dword ptr [esp+20]
:004AB106 56                       push esi
:004AB107 50                       push eax
:004AB108 E823480000               call 004AF930
:004AB10D 8D4C2428                 lea ecx, dword ptr [esp+28]
:004AB111 C644243600               mov [esp+36], 00
:004AB116 51                       push ecx
:004AB117 E80467FEFF               call 00491820
:004AB11C 8BF0                     mov esi, eax
:004AB11E 56                       push esi
:004AB11F E8EC63FEFF               call 00491510
:004AB124 8D542420                 lea edx, dword ptr [esp+20]    ***d edx 将看到
                                                                      第二行就是你输入
                                                                      的假注册码***
:004AB128 8BF8                     mov edi, eax
:004AB12A 52                       push edx
:004AB12B 56                       push esi
:004AB12C C644242800               mov [esp+28], 00
:004AB131 E86A65FEFF               call 004916A0      ***按过此 call 后，将在假注册码
                                                                    上一行，出现真正的注册码***
:004AB136 8D442438                 lea eax, dword ptr [esp+38]

* Possible Reference to String Resource ID=00014: "Paste Url"

                                    |
:004AB13A 6A0E                     push 0000000E
:004AB13C 8D4C242C                 lea ecx, dword ptr [esp+2C]
:004AB140 50                       push eax
:004AB141 51                       push ecx
:004AB142 E889530000               call 004B04D0       ***比较真假注册码***
:004AB147 83C42C                   add esp, 0000002C
:004AB14A 85C0                     test eax, eax
:004AB14C 7510                     jne 004AB15E
```

虽然在此前也对假注册码做过一些变换，但对真正的比较并不起作用。

<Cracked by KanKer> `4dd466d0e1e0cc123afbe504da2462ca`
