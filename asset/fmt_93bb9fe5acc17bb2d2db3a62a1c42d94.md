
```
:00466F21 8D442410             lea eax, dword ptr [esp+10]

* Possible Ref to Menu: NVIEWMENU, Item: "Crop   Ctrl+Y"

                                 |

:00466F25 6A20                 push 00000020

:00466F27 50                   push eax

* Possible Reference to Dialog: NVIEWAUTOCROP, CONTROL\_ID:07D1, ""

                                 |

:00466F28 68D1070000           push 000007D1

:00466F2D 56                   push esi

:00466F2E FFD7                 call edi

:00466F30 8D4C2470             lea ecx, dword ptr [esp+70]

:00466F34 51                   push ecx

* Possible StringData Ref from Data Obj ->"LicenseName"

                                 |

:00466F35 6828365600           push 00563628

:00466F3A 6A00                 push 00000000

:00466F3C E8FF5DFDFF           call 0043CD40           <-用户名写入注册表

:00466F41 8D54241C             lea edx, dword ptr [esp+1C]

:00466F45 52                   push edx

* Possible StringData Ref from Data Obj ->"LicenseNumber"

                                 |

:00466F46 6818365600           push 00563618

:00466F4B 6A00                 push 00000000

:00466F4D E8EE5DFDFF           call 0043CD40           <-注册码写入注册表

:00466F52 A12C8C5800           mov eax, dword ptr [00588C2C]

:00466F57 83C418               add esp, 00000018      问题：你怎么知道上面是写注册表？答：因为我用 `Regmon` 这个软件

                                                       对软件启动时的动作进行了监测的。:)

* Possible Ref to Menu: BROWSERMENU, Item: "Open..."

                                 |

:00466F5A C705488C580001000000    mov dword ptr [00588C48], 00000001

* Possible Ref to Menu: BROWSERMENU, Item: "Open..."

                                 |

:00466F64 6A01                 push 00000001

* Possible Ref to Menu: BROWSERMENU, Item: "Registration"

                                 |

:00466F66 68DE000000           push 000000DE

:00466F6B 50                   push eax

* Reference To: USER32.GetMenu, Ord:011Ch

                                 |

:00466F6C FF154CA55300         Call dword ptr [0053A54C]

:00466F72 50                   push eax

* Reference To: USER32.EnableMenuItem, Ord:00B5h

                                 |

:00466F73 FF1530A55300         Call dword ptr [0053A530]    <-将帮助菜单中注册选项禁用(显示为灰色)

:00466F79 8B15288C5800         mov edx, dword ptr [00588C28]    问题：你怎么知道这里是对菜单的禁用？答：因为 `EnableMenuItem`，

:00466F7F 8D4C2430             lea ecx, dword ptr [esp+30]        这在编程中是对一个控件的可用或不可能进行控制的，当它的

                                                       为 TRUE 时，可用；为 FALSE 时，则不可用。:)

* Possible Ref to Menu: NVIEWMENU, Item: "16 Grey scale (Dither)"

                                 |

:00466F83 6A40                 push 00000040

:00466F85 51                   push ecx

* Possible Reference to String Resource ID=05012: "Registration succesfull"

                                 |

:00466F86 6894130000           push 00001394

:00466F8B 52                   push edx

* Reference To: USER32.LoadStringA, Ord:01ABh

                                 |

:00466F8C FF15F8A55300         Call dword ptr [0053A5F8]

* Possible Ref to Menu: NVIEWMENU, Item: "16 Grey scale (Dither)"

                                 |

:00466F92 6A40                 push 00000040

:00466F94 8D442434             lea eax, dword ptr [esp+34]

:00466F98 689C325800           push 0058329C

:00466F9D 50                   push eax

:00466F9E 56                   push esi

* Reference To: USER32.MessageBoxA, Ord:01BEh

                                 |

:00466F9F FF157CA55300         Call dword ptr [0053A57C]    <-注册成功提示

:00466FA5 6A00                 push 00000000

:00466FA7 56                   push esi

* Reference To: USER32.EndDialog, Ord:00B9h

                                 |

:00466FA8 FF15F4A55300         Call dword ptr [0053A5F4]

:00466FAE 5F                   pop edi

* Possible Ref to Menu: BROWSERMENU, Item: "Open..."

                                 |

:00466FAF B801000000           mov eax, 00000001

:00466FB4 5E                   pop esi

:00466FB5 81C468010000         add esp, 00000168

:00466FBB C3                   ret

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00466E7C(C), :00466E88(C)                                      <-用户名和注册码为空跳到这里

|

:00466FBC 8B15288C5800         mov edx, dword ptr [00588C28]

:00466FC2 8D4C2430             lea ecx, dword ptr [esp+30]

* Possible Ref to Menu: NVIEWMENU, Item: "16 Grey scale (Dither)"

                                 |

:00466FC6 6A40                 push 00000040

:00466FC8 51                   push ecx

* Possible Reference to String Resource ID=05011: "Invalid registration"

                                 |

:00466FC9 6893130000           push 00001393

:00466FCE 52                   push edx

* Reference To: USER32.LoadStringA, Ord:01ABh

                                 |

:00466FCF FF15F8A55300         Call dword ptr [0053A5F8]

* Possible Ref to Menu: NVIEWDEFAULTME, Item: "Slide Show...   Ctrl+L"

                                 |

:00466FD5 6A10                 push 00000010

:00466FD7 8D442434             lea eax, dword ptr [esp+34]

:00466FDB 689C325800           push 0058329C

:00466FE0 50                   push eax

:00466FE1 56                   push esi

* Reference To: USER32.MessageBoxA, Ord:01BEh

                                 |

:00466FE2 FF157CA55300         Call dword ptr [0053A57C]      <-非法注册提示

:00466FE8 5F                   pop edi

* Possible Ref to Menu: BROWSERMENU, Item: "Open..."

                                 |

:00466FE9 B801000000           mov eax, 00000001

:00466FEE 5E                   pop esi

:00466FEF 81C468010000         add esp, 00000168

:00466FF5 C3                   ret

******************************************************************************

    跟入注册码算法 `call 0040AF40`...

.............

* Referenced by a CALL at Addresses:

|:0040B0FC   , :00466E98       <-程序有两处调用此 CALL 检查注册码的正确性，40B0FC 显然为程序启动时的调用。

|

:0040AF40 8B542404             mov edx, dword ptr [esp+04]        <-EDX 为用户名

:0040AF44 53                   push ebx

:0040AF45 55                   push ebp

:0040AF46 56                   push esi

:0040AF47 57                   push edi

:0040AF48 8BFA                 mov edi, edx                        <-EDI=用户名

:0040AF4A 83C9FF               or ecx, FFFFFFFF

:0040AF4D 33C0                 xor eax, eax                        <-EAX=0

:0040AF4F F2                   repnz                                 <\_ 取用户名

:0040AF50 AE                   scasb                                 的位数

:0040AF51 F7D1                 not ecx

:0040AF53 49                   dec ecx

* Possible StringData Ref from Data Obj ->"獕宁 Fx 鹦琪绻?蒔 s"

                                 |

:0040AF54 BE80355600           mov esi, 00563580                   <-字符表道地址入 ESI

:0040AF59 8BE9                 mov ebp, ecx                        <-用户名位数入 EBP

* Possible Ref to Menu: NVIEWMENU, Item: "Reopen   Ctrl+R"

                                 |

:0040AF5B B905000000           mov ecx, 00000005                   <-ECX=5
```
