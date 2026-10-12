
```
:004A9049 33D2                   xor edx, edx

:004A904B 8BC6                   mov eax, esi

:004A904D E87647FFFF             call 0049D7C8

:004A9052 8BC6                   mov eax, esi

:004A9054 E80F9EF5FF             call 00402E68

:004A9059 6A40                   push 00000040

* Possible StringData Ref from Code Obj -> "Information"

                                |

:004A905B B9C8904A00             mov ecx, 004A90C8

* Possible StringData Ref from Code Obj -> "Thank you for support !"    接着向上看

                                |

:004A9060 BAD4904A00             mov edx, 004A90D4

:004A9065 A134DC4A00             mov eax, dword ptr [004ADC34]

:004A906A 8B00                   mov eax, dword ptr [eax]

:004A906C E8B347FAFF             call 0044D824

:004A9071 EB18                   jmp 004A908B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :004A8FFA(C)

|

:004A9073 6A10                   push 00000010

* Possible StringData Ref from Code Obj -> "Error"

                                |

:004A9075 B9EC904A00             mov ecx, 004A90EC

* Possible StringData Ref from Code Obj -> "Sorry, this registration code "    向上看

                                    -> "is invalid."

                                |

:004A907A BAF4904A00             mov edx, 004A90F4

:004A907F A134DC4A00             mov eax, dword ptr [004ADC34]

:004A9084 8B00                   mov eax, dword ptr [eax]

:004A9086 E89947FAFF             call 0044D824
```

降龙十八掌最后一着:

总结：

序列号：`70721*****CWL`

`*****`这四位随便填

各位老大，我这样应该不算是爆破吧，呵呵！

请各位老大，前辈们指点一下！ `4dd466d0e1e0cc123afbe504da2462ca`
