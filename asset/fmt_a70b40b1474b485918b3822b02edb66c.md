
# Numerology Star Reader

> 原文：[`www.pediy.com/kssd/pediy03/forum352.htm`](https://www.pediy.com/kssd/pediy03/forum352.htm)

计算 `Numerology Star Reader` (version 15.0) 注册码

这是一个有趣的算命软件，使用后感觉比平时看到的西方星命书都好，只可惜是英文的。具体下载地址忘了，好象来自于 `http://www.esoftware.com.cn`，当然你也可以从它老家 `http://www2.pitnet.net/numer/` 下载，约 900K。

安装完成后发现不注册根本不能使用，所以只能跟作者玩玩智力游戏了。

用 `WDASM` 反汇编，看到

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00401D52(C)

|

:00401EDB 6A09             push 00000009
:00401EDD 6830200000      push 00002030

* Possible StringData Ref from Data Obj ->"NUMEROLOGY STAR READER"

   |

:00401EE2 68E4414000      push 004041E4

* Possible StringData Ref from Data Obj ->"Your Registration ID is not valid! "

->"  "

   |

:00401EE7 6854414000      push 00404154
:00401EEC 53              push ebx

* Reference To: USER32.MessageBoxExA, Ord:0196h

   |

:00401EED FF15F8544000    Call dword ptr [004054F8]
:00401EF3 681C444000      push 0040441C
```

向上看 `:00401D52`，

```
:00401D4B E810FFFFFF      call 00401C60
:00401D50 85C0           test eax, eax   <--eax=1, sucess!
:00401D52 0F8483010000   je 00401EDB

......

:00401DF2 6840200000      push 00002040

* Possible StringData Ref from Data Obj ->"NUMEROLOGY STAR READER"

   |

:00401DF7 68E4414000      push 004041E4

* Possible StringData Ref from Data Obj ->"Your registration was completed "

->"successfully!    "

   |

:00401DFC 68B0414000      push 004041B0
:00401E01 6A00            push 00000000

* Reference To: USER32.MessageBoxExA, Ord:0196h

   |

:00401E03 FF15F8544000    Call dword ptr [004054F8]
```

所以 `:00401D4B E810FFFFFF call 00401C60` 是关键。

输入注册码后，用 `TRW2000` 下 `BPX 401D4B`，按下 `REGISTER` 键后被截。用 `F8` 进入看看：

```
:00401C69 8378F816       cmp dword ptr [eax-08], 00000016
                         <--22 位注册码，别多一位！
:00401C6D 7533           jne 00401CA2
:00401C6F 8BCE           mov ecx, esi
:00401C71 E86AFCFFFF     call 004018E0  <--(1)
:00401C76 85C0           test eax, eax  <--eax=1,sucess!
:00401C78 7428           je 00401CA2
:00401C7A 8BCE           mov ecx, esi
:00401C7C E83FFDFFFF     call 004019C0  <--2)
:00401C81 85C0           test eax, eax  <--eax=1,sucess!
:00401C83 741D           je 00401CA2
:00401C85 8BCE           mov ecx, esi
:00401C87 E814FEFFFF     call 00401AA0  <--(3)
:00401C8C 85C0           test eax, eax  <--eax=1,sucess!
:00401C8E 7412           je 00401CA2
:00401C90 8BCE           mov ecx, esi
:00401C92 E8E9FEFFFF     call 00401B80  <--(4)
:00401C97 85C0           test eax, eax  <--eax=1,sucess!
:00401C99 7407           je 00401CA2
:00401C9B B801000000     mov eax, 00000001
:00401CA0 5E             pop esi
:00401CA1 C3             ret

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00401C6D(C), :00401C78(C), :00401C83(C), :00401C8E(C), :00401C99(C)

|

:00401CA2 33C0           xor eax, eax <--die!!!
:00401CA4 5E             pop esi
:00401CA5 C3             ret
```

上面 4 个 `CALL` 的比较相似，就讲第一个吧！看 `:00401C71 call 004018E0`，用 `F8` 跟进，

```
......

......

:00401930 8B542414          mov edx, dword ptr [esp+14]
:00401934 8D0C8500000000    lea ecx, dword ptr [4*eax+00000000]
:0040193B 83C404            add esp, 00000004
:0040193E 8D4203            lea eax, dword ptr [edx+03]
:00401941 8D0440            lea eax, dword ptr [eax+2*eax]
:00401944 2BC1              sub eax, ecx
:00401946 83F815            cmp eax, 00000015
:00401949 7550              jne 0040199B
:0040194B 8BAD10010000      mov ebp, dword ptr [ebp+00000110]
:00401951 A08B344000        mov al, byte ptr [0040348B]
:00401956 8A4D00            mov cl, byte ptr [ebp+00]
:00401959 3AC8              cmp cl, al
:0040195B 753E              jne 0040199B
:0040195D 8A5502            mov dl, byte ptr [ebp+02]
:00401960 A01D374000        mov al, byte ptr [0040371D]
:00401965 3AD0              cmp dl, al
:00401967 7532              jne 0040199B
:00401969 8A4504            mov al, byte ptr [ebp+04]
:0040196C 8A0D67364000      mov cl, byte ptr [00403667]
:00401972 3AC1              cmp al, cl
:00401974 7525              jne 0040199B

......
```

翻译一下就是：(第二位数字/4)-(第四位数字/3)=1（十进制〕

第一位=`q`, 第三位=`h`, 第五位=`S`

如此，得到注册码：`q8h3S-Ey4%2-7G6zj-5U*1e__(最后两位随便〕`

from: china crack group

2000.10.02

end. `4dd466d0e1e0cc123afbe504da2462ca`
