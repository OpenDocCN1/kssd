
## 反汇编分析

> 字符串参考："认证码及序列号错误！"，得如下结果：
>
> **机器码**：`84345`
>
> **注册码**：`w3336m5v593}f08283~`

```
:00507038 55                    push ebp
:00507039 8BEC                  mov ebp, esp
:0050703B 81C458FFFFFF          add esp, FFFFFF58
:00507041 53                    push ebx
:00507042 33C9                  xor ecx, ecx
:00507044 898D58FFFFFF          mov dword ptr [ebp+FFFFFF58], ecx
:0050704A 898D5CFFFFFF          mov dword ptr [ebp+FFFFFF5C], ecx
:00507050 898D74FFFFFF          mov dword ptr [ebp+FFFFFF74], ecx
:00507056 898D70FFFFFF          mov dword ptr [ebp+FFFFFF70], ecx
:0050705C 898D6CFFFFFF          mov dword ptr [ebp+FFFFFF6C], ecx
:00507062 898D68FFFFFF          mov dword ptr [ebp+FFFFFF68], ecx
:00507068 898D64FFFFFF          mov dword ptr [ebp+FFFFFF64], ecx
:0050706E 898D60FFFFFF          mov dword ptr [ebp+FFFFFF60], ecx
:00507074 894DFC                mov dword ptr [ebp-04], ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00507010(C)
|

:00507077 8BD8                  mov ebx, eax
:00507079 33C0                  xor eax, eax
:0050707B 55                    push ebp
:0050707C 686E725000           push 0050726E
:00507081 64FF30               push dword ptr fs:[eax]
:00507084 648920               mov dword ptr fs:[eax], esp
:00507087 6880000000           push 00000080

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00507017(C)
|

:0050708C 8D857BFFFFFF          lea eax, dword ptr [ebp+FFFFFF7B]
:00507092 50                    push eax

* Reference To: kernel32.GetSystemDirectoryA, Ord:0000h
                          |

:00507093 E8FCFFEFFF           Call 00407094
:00507098 8D45FC               lea eax, dword ptr [ebp-04]
:0050709B 8D957BFFFFFF         lea edx, dword ptr [ebp+FFFFFF7B]
:005070A1 B981000000           mov ecx, 00000081
:005070A6 E881D9EFFF           call 00404A2C
:005070AB 8B8318030000         mov eax, dword ptr [ebx+00000318]
:005070B1 8B8008020000         mov eax, dword ptr [eax+00000208]
:005070B7 33D2                 xor edx, edx
:005070B9 E84677F6FF           call 0046E804
:005070BE 8B400C               mov eax, dword ptr [eax+0C]

* Possible StringData Ref from Code Obj -> "中华灯谜-软件已注册"
                          |

:005070C1 BA84725000           mov edx, 00507284
:005070C6 E8F5DAEFFF           call 00404BC0
:005070CB 750F                 jne 005070DC

* Possible StringData Ref from Code Obj -> "软件已注册"
                          |

:005070CD B8A0725000           mov eax, 005072A0
:005070D2 E82533F3FF           call 0043A3FC
:005070D7 E941010000           jmp 0050721D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:005070CB(C)
|

:005070DC 8D9574FFFFFF          lea edx, dword ptr [ebp+FFFFFF74]
:005070E2 8B8350040000         mov eax, dword ptr [ebx+00000450]
:005070E8 E81BA1F3FF           call 00441208
:005070ED 83BD74FFFFFF00       cmp dword ptr [ebp+FFFFFF74], 00000000
:005070F4 0F840C010000         je 00507206
:005070FA 8D9570FFFFFF         lea edx, dword ptr [ebp+FFFFFF70]
:00507100 8B8350040000         mov eax, dword ptr [ebx+00000450]
:00507106 E8FDA0F3FF           call 00441208
:0050710B 8B8570FFFFFF         mov eax, dword ptr [ebp+FFFFFF70]
:00507111 50                    push eax

* Possible StringData Ref from Code Obj -> "w33"
                          |

:00507112 68B4725000           push 005072B4---------------------------------由此往下的分析见上面的文章
:00507117 8D9560FFFFFF         lea edx, dword ptr [ebp+FFFFFF60]
:0050711D 8B8348040000         mov eax, dword ptr [ebx+00000448]
:00507123 E8E0A0F3FF           call 00441208
:00507128 8B8560FFFFFF         mov eax, dword ptr [ebp+FFFFFF60]
:0050712E E89521F0FF           call 004092C8
:00507133 8D9564FFFFFF         lea edx, dword ptr [ebp+FFFFFF64]
:00507139 E80EDBFFFF           call 00504C4C
:0050713E 8B8564FFFFFF         mov eax, dword ptr [ebp+FFFFFF64]
:00507144 E87F21F0FF           call 004092C8
:00507149 8D9568FFFFFF         lea edx, dword ptr [ebp+FFFFFF68]
:0050714F E8D8DBFFFF           call 00504D2C
:00507154 FFB568FFFFFF         push dword ptr [ebp+FFFFFF68]
:0050715A 68C0725000           push 005072C0
:0050715F 8D856CFFFFFF         lea eax, dword ptr [ebp+FFFFFF6C]
:00507165 BA03000000           mov edx, 00000003
:0050716A E8CDD9EFFF           call 00404B3C
:0050716F 8B956CFFFFFF         mov edx, dword ptr [ebp+FFFFFF6C]--------------->真注册码
:00507175 58                   pop eax\----------------------------------------->输入的注册码
:00507176 E845DAEFFF           call 00404BC0----------------------------------->验证
:0050717B 0F8585000000         jne 00507206------------------------------------>跳到错误

* Possible StringData Ref from Code Obj -> "注册成功，谢谢您的注册！"
                          |

:00507181 B8CC725000           mov eax, 005072CC
:00507186 E87132F3FF           call 0043A3FC
:0050718B 8B8318030000         mov eax, dword ptr [ebx+00000318]
:00507191 8B8008020000         mov eax, dword ptr [eax+00000208]
:00507197 33D2                 xor edx, edx
:00507199 E86676F6FF           call 0046E804

* Possible StringData Ref from Code Obj -> "中华灯谜-软件已注册"
                          |

:0050719E BA84725000           mov edx, 00507284
:005071A3 E8B875F6FF           call 0046E760
:005071A8 8D855CFFFFFF         lea eax, dword ptr [ebp+FFFFFF5C]

* Possible StringData Ref from Code Obj -> "ilmCoIn_0_38.dll"
                          |

:005071AE B9F0725000           mov ecx, 005072F0
:005071B3 8B55FC               mov edx, dword ptr [ebp-04]
:005071B6 E80DD9EFFF           call 00404AC8
:005071BB 8B8D5CFFFFFF         mov ecx, dword ptr [ebp+FFFFFF5C]
:005071C1 B201                 mov dl, 01

* Possible StringData Ref from Code Obj -> "$XF"
                          |

:005071C3 A1444E4600           mov eax, dword ptr [00464E44]
:005071C8 E827DDF5FF           call 00464EF4

* Possible StringData Ref from Code Obj -> "326t5wefgs"
                          |

:005071CD 680C735000           push 0050730C

* Possible StringData Ref from Code Obj -> "setet567"
                          |

:005071D2 B920735000           mov ecx, 00507320

* Possible StringData Ref from Code Obj -> "sym"
                          |

:005071D7 BA34735000           mov edx, 00507334
:005071DC 8B18                 mov ebx, dword ptr [eax]
:005071DE FF5304               call [ebx+04]
:005071E1 8D8558FFFFFF         lea eax, dword ptr [ebp+FFFFFF58]

* Possible StringData Ref from Code Obj -> "ilmCoIn_0_38.dll"
                          |

:005071E7 B9F0725000           mov ecx, 005072F0
:005071EC 8B55FC               mov edx, dword ptr [ebp-04]
:005071EF E8D4D8EFFF           call 00404AC8
:005071F4 8B8558FFFFFF         mov eax, dword ptr [ebp+FFFFFF58]
```
