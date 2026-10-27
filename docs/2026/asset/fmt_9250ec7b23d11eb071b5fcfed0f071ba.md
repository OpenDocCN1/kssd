
```text
:004A6A76 83C408                  add esp, 00000008
:004A6A79 C3                      ret
```

我们分析了以上，就明白，要获得企业版，那末就要检测狗成功并{让`dword ptr [034CEAB4]`等于2，让`dword ptr [esi+0192D5C0]`等于1},系统时间在2003-2004之间，当然现在小于2004就行。

这时我们在回头看看，发现这和我的上篇破文【看雪论坛精华5】一样啊，对，上次我是初学破解，很多不明白，现在我好像又更深一步了。

我们再来看看最前面的

```text
\* Possible StringData Ref from Data Obj ->"%s"

                                 |
:004A5F2D 6824C45600              push 0056C424
:004A5F32 68E8E94C03              push 034CE9E8
:004A5F37 E8C22B0600              call 00508AFE
:004A5F3C 83C40C                  add esp, 0000000C
:004A5F3F 8BCE                    mov ecx, esi
:004A5F41 E8CA080000              call 004A6810  ====>这次我们看看这个call的作用
:004A5F46 83F801                  cmp eax, 00000001
:004A5F49 0F85CA000000            jne 004A6019  ====>关键的跳
:004A5F4F 8986C0D59201            mov dword ptr [esi+0192D5C0], eax
:004A5F55 C705B4EA4C0302000000    mov dword ptr [034CEAB4], 00000002
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A6062(C)  ====>成功，来到这里，前提是上面的，从004A6062跳来的，
|
:004A5F5F 8BCE                    mov ecx, esi
:004A5F61 E86A040000              call 004A63D0
:004A5F66 B9C0C0C000              mov ecx, 00C0C0C0
:004A5F6B B880808000              mov eax, 00808080
:004A5F70 890DDC321102            mov dword ptr [021132DC], ecx
:004A5F76 890DE0321102            mov dword ptr [021132E0], ecx
```

`..............省`

`====>` `call 004A6810` 这次我们看看这个call的作用

```text
\* Referenced by a CALL at Addresses:
|:004A5F41   , :004B5AA3   我们看到的call，有2个，就是第2个在作怪，我开始没注意到，
|
:004A6810 6AFF                    push FFFFFFFF
:004A6812 68464A5300              push 00534A46
:004A6817 64A100000000            mov eax, dword ptr fs:[00000000]
:004A681D 50                      push eax
:004A681E 64892500000000          mov dword ptr fs:[00000000], esp
:004A6825 81EC80000000            sub esp, 00000080
:004A682B 8D442404                lea eax, dword ptr [esp+04]
:004A682F 8D4C241C                lea ecx, dword ptr [esp+1C]
:004A6833 50                      push eax
:004A6834 51                      push ecx
:004A6835 C744240C90010000        mov [esp+0C], 00000190
\* Reference To: KERNEL32.GetComputerNameA, Ord:00CEh  ====>获得你的
                                 |
:004A683D FF1594A35300            Call dword ptr [0053A394]
:004A6843 8B15545E5700            mov edx, dword ptr [00575E54]
:004A6849 89542400                mov dword ptr [esp], edx
:004A684D 8D44241C                lea eax, dword ptr [esp+1C]
:004A6851 8D4C2400                lea ecx, dword ptr [esp] ====>ecx就是你的计算机名
:004A6855 50                      push eax
\* Possible StringData Ref from Data Obj ->"%s"
                                 |
:004A6856 6824C45600              push 0056C424
:004A685B 51                      push ecx
:004A685C C784249400000000000000  mov dword ptr [esp+00000094], 00000000
:004A6867 E892220600              call 00508AFE
:004A686C 83C40C                  add esp, 0000000C
:004A686F 8D4C2400                lea ecx, dword ptr [esp]
:004A6873 6A00                    push 00000000
\* Possible StringData Ref from Data Obj ->"OEMCOMPUTER"  ====>计算机名的比较，以下都是
                                 |
:004A6875 68F8215700              push 005721F8
:004A687A E84C1F0600              call 005087CB
:004A687F 85C0                    test eax, eax
:004A6881 0F84EB000000            je 004A6972   ====>跳！？
:004A6887 6A00                    push 00000000
\* Possible StringData Ref from Data Obj ->"147"
                                 |
:004A6889 68F4215700              push 005721F4
:004A688E 8D4C2408                lea ecx, dword ptr [esp+08]
:004A6892 E8341F0600              call 005087CB
:004A6897 85C0                    test eax, eax
:004A6899 0F84D3000000            je 004A6972
:004A689F 6A00                    push 00000000
\* Possible StringData Ref from Data Obj ->"ANSYS"
                                 |
:004A68A1 68EC215700              push 005721EC
:004A68A6 8D4C2408                lea ecx, dword ptr [esp+08]
:004A68AA E81C1F0600              call 005087CB
:004A68AF 85C0                    test eax, eax
:004A68B1 0F84BB000000            je 004A6972
:004A68B7 6A00                    push 00000000
\* Possible StringData Ref from Data Obj ->"MSTCAD"  ====>计算机名的比较，以下都是
                                 |
:004A68B9 68E4215700              push 005721E4
:004A68BE 8D4C2408                lea ecx, dword ptr [esp+08]
:004A68C2 E8041F0600              call 005087CB
:004A68C7 85C0                    test eax, eax
:004A68C9 0F84A3000000            je 004A6972
:004A68CF 6A00                    push 00000000
\* Possible StringData Ref from Data Obj ->"MST"
                                 |
:004A68D1 68E0215700              push 005721E0
:004A68D6 8D4C2408                lea ecx, dword ptr [esp+08]
:004A68DA E8EC1E0600              call 005087CB
:004A68DF 85C0                    test eax, eax
:004A68E1 0F848B000000            je 004A6972
:004A68E7 6A00                    push 00000000
\* Possible StringData Ref from Data Obj ->"FAZURE"
                                 |
:004A68E9 68D8215700              push 005721D8
:004A68EE 8D4C2408                lea ecx, dword ptr [esp+08]
:004A68F2 E8D41E0600              call 005087CB
:004A68F7 85C0                    test eax, eax
:004A68F9 7477                    je 004A6972
:004A68FB 6A00                    push 00000000
\* Possible StringData Ref from Data Obj ->"WCZGN"
                                 |
:004A68FD 68D0215700              push 005721D0
:004A6902 8D4C2408                lea ecx, dword ptr [esp+08]
:004A6906 E8C01E0600              call 005087CB
:004A690B 85C0                    test eax, eax
:004A690D 7463                    je 004A6972
:004A690F 6A00                    push 00000000
\* Possible StringData Ref from Data Obj ->"JAVAS"
                                 |
:004A6911 68C8215700              push 005721C8
:004A6916 8D4C2408                lea ecx, dword ptr [esp+08]
:004A691A E8AC1E0600              call 005087CB
:004A691F 85C0                    test eax, eax
:004A6921 744F                    je 004A6972
\* Possible StringData Ref from Data Obj ->"LDD"
                                 |
:004A6923 68C4215700              push 005721C4
:004A6928 8D4C2404                lea ecx, dword ptr [esp+04]
:004A692C E88C1E0600              call 005087BD
:004A6931 85C0                    test eax, eax
:004A6933 743D                    je 004A6972
\* Possible StringData Ref from Data Obj ->"7-208"
                                 |
:004A6935 68BC215700              push 005721BC
:004A693A 8D4C2404                lea ecx, dword ptr [esp+04]
:004A693E E87A1E0600              call 005087BD
:004A6943 85C0                    test eax, eax
:004A6945 742B                    je 004A6972  ====>如果到这里还不同，那
:004A6947 8D4C2400                lea ecx, dword ptr [esp]
:004A694B C7842488000000FFFFFFFF  mov dword ptr [esp+00000088], FFFFFFFF
:004A6956 E8A5840600              call 0050EE00
:004A695B 33C0                    xor eax, eax  ====>如果到这里还不同，那eax请零，返回失败
:004A695D 8B8C2480000000          mov ecx, dword ptr [esp+00000080]
:004A6964 64890D00000000          mov dword ptr fs:[00000000], ecx
:004A696B 81C48C000000            add esp, 0000008C
:004A6971 C3                      ret   ====>，返回
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004A6881(C), :004A6899(C), :004A68B1(C), :004A68C9(C), :004A68E1(C)
|:004A68F9(C), :004A690D(C), :004A6921(C), :004A6933(C), :004A6945(C)
|
:004A6972 56                      push esi
:004A6973 8D4C240C                lea ecx, dword ptr [esp+0C]
:004A6977 E89D890600              call 0050F319
:004A697C 6A00                    push 00000000
:004A697E 6A00                    push 00000000
\* Possible StringData Ref from Data Obj ->"c:\windows\help\m$.TMP.txt"
                                 |
```
