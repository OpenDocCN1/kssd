
# AB Email Searcher 1.6

> 原文：[`www.pediy.com/kssd/pediy03/forum1254.htm`](https://www.pediy.com/kssd/pediy03/forum1254.htm)

用 `DEDE` 破 `nag` 窗口

**破解人**：风飘雪

`http://fpx.yeah.net` `gd1@yeah.net`

**目的**：加入 `BCG`，学习破 `nag`

**试验品**：`AB Email Searcher 1.6`（国产）

**住址**：华军

**工具**：`dede`，`w32dasm`，`trw`，`ultraedit`，`caspr`，`language`

## 1. `caspr` 脱 `aspack` 壳

## 2. `dede` 反汇编

窗体菜单发现 `TForm1` 为主窗口，`TForm2` 为 `nag` 窗口；

程序菜单 `Unit2` `TForm2` 的 `FormCreate` 为 `11`

在 `Unit1` `TForm1` 的 `FormCreate` 发现 `11`，进入 `TForm1.FormCreate`，查找第一次出现 `TForm2` 的地方

*   Reference to field `TForm1.OFFS_03E9`

    |

    ```
    00477025  8A83E9030000   mov     al, byte ptr [ebx+$03E9]
    0047702B  84C0           test    al, al
    0047702D  84C0           test    al, al
    0047702F  7540           jnz     00477071======上面的第一个跳转 方法 1： jnz=>jmp 一条（跳）就胡，一饼（EB）伺候
    00477031  8BCB           mov     ecx, ebx
    00477033  B201           mov     dl, $01
    ```

*   Reference to class `TForm2` =========================第一次出现 `TForm2`

    |

    ```
    00477035  A1B4454700     mov     eax, dword ptr [$4745B4]
    0047703A  E875D8FCFF     call    004448B4
    0047703F  8BF0           mov     esi, eax
    00477041  8D55EC         lea     edx, [ebp-$14]
    00477044  8BC3           mov     eax, ebx
    00477046  E8611E0000     call    00478EAC
    0047704B  8B55EC         mov     edx, [ebp-$14]
    ```

*   Reference to control `TForm2.Edit1 : TEdit`

    |

    ```
    0047704E  8B86E8020000  mov     eax, [esi+$02E8]
    00477054  E86B62FBFF     call    0042D2C4
    00477059  8BC6           mov     eax, esi
    0047705B  8B10           mov     edx, [eax]
    ```

*   Possible reference to virtual method `TForm2.OFFS_00D8`

    |

    ```
    0047705D  FF92D8000000  call    dword ptr [edx+$00D8]====此 call 呼叫 nag 窗口 方法 2 ：nop 掉此 call 九筒（90）就胡
    ```

*   Reference to field `TForm1.OFFS_03E8`

    |

    ```
    00477063  C683E803000001 mov     byte ptr [ebx+$03E8], $01
    0047706A  8BC6           mov     eax, esi
    0047706C  E8FFBCF8FF     call    00402D70
    ```

## 2. 去掉功能限制

`w32dasm` 反汇编，发现多处 `blowfish` 字样，好像是一种算法，还是这里一个人的名字，好怕怕呀！

串式参考 `"保存结果，仅限于注册用户使用"` 有两处

### （1）

```
:00477107 FF524C           call [edx+4C]
:0047710A 84C0             test al, al
:0047710C 7415             je 00477123==========一条（跳）就死，九筒（90）就胡
:0047710E 8B8398030000     mov eax, dword ptr [ebx+00000398]
:00477114 83C070           add eax, 00000070
```

*   Possible StringData Ref from Code Obj -> `"保存结果"`

    |

    ```
    :00477117 BA24724700    mov edx, 00477224
    :0047711C E8EFC9F8FF    call 00403B10
    :00477121 EB13          jmp 00477136
    ```

*   Referenced by a (U)nconditional or (C)onditional Jump at Address:

    |:0047710C(C)===================================================双击鼠标右键

    |

    ```
    :00477123 8B8398030000  mov eax, dword ptr [ebx+00000398]
    :00477129 83C070        add eax, 00000070
    ```

*   Possible StringData Ref from Code Obj -> `"保存结果，仅限于注册用户使用"`

    |

    ```
    :0047712C BA38724700    mov edx, 00477238
    :00477131 E8DAC9F8FF    call 00403B10
    ```

### （2）

```
:00478130 FF524C           call [edx+4C]
:00478133 84C0             test al, al
:00478135 7415             je 0047814C======================一条（跳）就死，九筒（90）就胡
:00478137 8B8698030000     mov eax, dword ptr [esi+00000398]
:0047813D 83C070           add eax, 00000070
```

*   Possible StringData Ref from Code Obj -> `"保存结果"`

    |

    ```
    :00478140 BAF8814700    mov edx, 004781F8
    :00478145 E8C6B9F8FF    call 00403B10
    :0047814A EB13          jmp 0047815F
    ```

*   Referenced by a (U)nconditional or (C)onditional Jump at Address:

    |:00478135(C)=======================================================双击鼠标右键

    |

    ```
    :0047814C 8B8698030000  mov eax, dword ptr [esi+00000398]
    :00478152 83C070        add eax, 00000070
    ```

*   Possible StringData Ref from Code Obj -> `"保存结果，仅限于注册用户使用"`

    |

    ```
    :00478155 BA0C824700    mov edx, 0047820C
    :0047815A E8B1B9F8FF    call 00403B10
    ```

`2001.8.22` `4dd466d0e1e0cc123afbe504da2462ca`
