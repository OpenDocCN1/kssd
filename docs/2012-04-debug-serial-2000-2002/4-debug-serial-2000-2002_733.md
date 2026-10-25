
# 易经八卦占卜程序 7.0

> 原文：[`www.pediy.com/kssd/pediy03/forum1280.htm`](https://www.pediy.com/kssd/pediy03/forum1280.htm)

★易经八卦占卜程序 7.0 的破解★

共享软件，注册费用 RMB18（什么，就这破软件也要注册费 18 元，我倒！），未注册版无法打印占卜的内容，也无法根据公历日期换算到农历日期。

废话少说，先随便乱填一气，蹦出对话框“对不起，用户名和注册码不匹配。注册失败！”。

用 `W32Dasm` 反汇编后，查找该字符串，向上看去，来到：

```
:0040905E E8E589FFFF                call 00401A48       <--关键 Call，要跟进

:00409063 84C0                      test al, al         <--al 为标志寄存器

:00409065 742A                      je 00409091         <--跳，则去死

:00409067 6A40                      push 00000040

* Possible StringData Ref from Data Obj ->"提示"

                                                          |

:00409069 B9B0585200                mov ecx, 005258B0

* Possible StringData Ref from Data Obj ->"恭喜！
注册成功！"

                                                          |

:0040906E BA9E585200                mov edx, 0052589E

:00409073 A104075300                mov eax, dword ptr [00530704]

:00409078 8B00                      mov eax, dword ptr [eax]

:0040907A E8196C1100                call 0051FC98

:0040907F 8B45D0                    mov eax, dword ptr [ebp-30]

:00409082 E831000000                call 004090B8

:00409087 8B45D0                    mov eax, dword ptr [ebp-30]

:0040908A E8CD010000                call 0040925C

:0040908F EB18                      jmp 004090A9

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00409065(C)

|

:00409091 6A10                      push 00000010

* Possible StringData Ref from Data Obj ->"错误"

                                                          |

:00409093 B9DF585200                mov ecx, 005258DF

* Possible StringData Ref from Data Obj ->"对不起，用户名和注册码不匹配。
注册失败！"

                                                          |

:00409098 BAB5585200                mov edx, 005258B5

:0040909D A104075300                mov eax, dword ptr [00530704]

:004090A2 8B00                      mov eax, dword ptr [eax]

:004090A4 E8EF6B1100                call 0051FC98

-------------------------------------------------------------------------------------

* Referenced by a CALL at Addresses:

|:004040C2  , :0040905E 

|

:00401A48 55                        push ebp

:00401A49 8BEC                      mov ebp, esp

:00401A4B 83C4B8                    add esp, FFFFFFB8

:00401A4E 53                        push ebx

:00401A4F 8955F8                    mov dword ptr [ebp-08], edx

:00401A52 8945FC                    mov dword ptr [ebp-04], eax

:00401A55 B8682E5200                mov eax, 00522E68

:00401A5A E8B5431100                call 00515E14

:00401A5F C745E802000000            mov [ebp-18], 00000002

:00401A66 8D55FC                    lea edx, dword ptr [ebp-04]

:00401A69 8D45FC                    lea eax, dword ptr [ebp-04]

:00401A6C E813E31100                call 0051FD84

:00401A71 FF45E8                    inc [ebp-18]

:00401A74 66C745DC0800              mov [ebp-24], 0008

:00401A7A 8D55F8                    lea edx, dword ptr [ebp-08]

:00401A7D 8D45F8                    lea eax, dword ptr [ebp-08]

:00401A80 E8FFE21100                call 0051FD84

.......................................省略一大段......................................

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00401B03(C)

|

:00401B38 8D45FC                    lea eax, dword ptr [ebp-04]

:00401B3B E868010000                call 00401CA8                  <--eax 返回用户名的长度

:00401B40 8BD8                      mov ebx, eax                  <--ebx=eax

:00401B42 8D45F8                    lea eax, dword ptr [ebp-08]

:00401B45 E85E010000                call 00401CA8                  <--eax 返回序列号的长度

:00401B4A 3BD8                      cmp ebx, eax                  <--比较二者是否相等

:00401B4C 7433                      je 00401B81                   <--不相等，则去死

:00401B4E 33C0                      xor eax, eax      

:00401B50 50                        push eax

:00401B51 FF4DE8                    dec [ebp-18]

:00401B54 8D45F8                    lea eax, dword ptr [ebp-08]

:00401B57 BA02000000                mov edx, 00000002

:00401B5C E84BE31100                call 0051FEAC

:00401B61 FF4DE8                    dec [ebp-18]

:00401B64 8D45FC                    lea eax, dword ptr [ebp-04]

:00401B67 BA02000000                mov edx, 00000002

:00401B6C E83BE31100                call 0051FEAC

:00401B71 58                        pop eax

:00401B72 8B55CC                    mov edx, dword ptr [ebp-34]

:00401B75 64891500000000            mov dword ptr fs:[00000000], edx

:00401B7C E920010000                jmp 00401CA1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00401B4C(C)

|

:00401B81 8D45F8                    lea eax, dword ptr [ebp-08]

:00401B84 E81F010000                call 00401CA8

:00401B89 8945BC                    mov dword ptr [ebp-44], eax

:00401B8C 8B55BC                    mov edx, dword ptr [ebp-44]

:00401B8F 42                        inc edx

:00401B90 52                        push edx

:00401B91 E816341100                call 00514FAC

:00401B96 59                        pop ecx

:00401B97 8945C8                    mov dword ptr [ebp-38], eax

:00401B9A 8B4DBC                    mov ecx, dword ptr [ebp-44]

:00401B9D 41                        inc ecx

:00401B9E 51                        push ecx

:00401B9F E808341100                call 00514FAC

:00401BA4 59                        pop ecx

:00401BA5 8945C4                    mov dword ptr [ebp-3C], eax

:00401BA8 8B55FC                    mov edx, dword ptr [ebp-04]

:00401BAB 8B45C8                    mov eax, dword ptr [ebp-38]

:00401BAE E8A5771000                call 00509358

:00401BB3 8B55F8                    mov edx, dword ptr [ebp-08]

:00401BB6 8B45C4                    mov eax, dword ptr [ebp-3C]

:00401BB9 E89A771000                call 00509358

:00401BBE 33C9                      xor ecx, ecx

:00401BC0 894DC0                    mov dword ptr [ebp-40], ecx

:00401BC3 8B45C0                    mov eax, dword ptr [ebp-40]

:00401BC6 8B55BC                    mov edx, dword ptr [ebp-44]

:00401BC9 3BC2                      cmp eax, edx

:00401BCB 7D64                      jge 00401C31

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00401C2F(C)

|

:00401BCD 8B4DC8                    mov ecx, dword ptr [ebp-38]      <--开始循环

:00401BD0 8B45C0                    mov eax, dword ptr [ebp-40]

:00401BD3 8A1401                    mov dl, byte ptr [ecx+eax]

:00401BD6 8855BB                    mov byte ptr [ebp-45], dl

:00401BD9 0FBE4DBB                   movsx ecx, byte ptr [ebp-45]

:00401BDD 0FBE45BB                   movsx eax, byte ptr [ebp-45]

:00401BE1 0FAFC8                     imul ecx, eax

:00401BE4 0FBE55BB                   movsx edx, byte ptr [ebp-45]

:00401BE8 0FAFCA                     imul ecx, edx

:00401BEB 8B45C0                    mov eax, dword ptr [ebp-40]

:00401BEE 40                         inc eax

:00401BEF 8B55C0                    mov edx, dword ptr [ebp-40]

:00401BF2 42                         inc edx

:00401BF3 F7EA                        imul edx

:00401BF5 2BC8                        sub ecx, eax

:00401BF7 8B45C0                    mov eax, dword ptr [ebp-40]

:00401BFA 40                         inc eax

:00401BFB 0FBE55BB                   movsx edx, byte ptr [ebp-45]

:00401BFF F7EA                        imul edx

:00401C01 2BC8                        sub ecx, eax

:00401C03 51                         push ecx

:00401C04 E8C3000000                call 00401CCC

:00401C09 59                         pop ecx

:00401C0A B94B000000                mov ecx, 0000004B

:00401C0F 99                         cdq
```



```
:00401C10 F7F9                  idiv ecx

:00401C12 80C230                add dl, 30

:00401C15 8855BA                mov byte ptr [ebp-46], dl                <--dl 为正确的注册码

:00401C18 8B45C8                mov eax, dword ptr [ebp-38]

:00401C1B 8B55C0                mov edx, dword ptr [ebp-40]

:00401C1E 8A4DBA                mov cl, byte ptr [ebp-46]                <--cl=dl

:00401C21 880C10                mov byte ptr [eax+edx], cl    <--相应注册码存入用户名的地址中

:00401C24 FF45C0                inc [ebp-40]

:00401C27 8B45C0                mov eax, dword ptr [ebp-40]

:00401C2A 8B55BC                mov edx, dword ptr [ebp-44]

:00401C2D 3BC2                  cmp eax, edx

:00401C2F 7C9C                  jl 00401BCD                             <--返回循环开始处

                                                              AirHolder

                                                              2001.8.31.  4dd466d0e1e0cc123afbe504da2462ca
```
