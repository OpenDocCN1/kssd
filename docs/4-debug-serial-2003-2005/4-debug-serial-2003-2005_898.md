
# 简单脱壳＋简单算法之《批量更名快车 2005》

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-98.htm`](https://www.pediy.com/kssd/pediy07/pediy7-98.htm)

简单脱壳＋简单算法之《批量更名快车 2005》

作者：jney2

日期：2005.02.22

环境：XP+ImportREC＋OllyDbg＋Peid＋W32dasm

说明：软件来自“天空软件站”，破解无它，爱好＋学习。

安装后，用 `peid` 查壳，“未发现 \*”，`EP` 区段：xiaohui。还是用 `OllyDbg` 试试手工脱壳吧，

用 `OllyDbg` 载入程序，停在入口处：

```
0051F000 >   9C              PUSHFD
0051F001    E8 00000000     CALL pi.0051F006
0051F006    5D              POP EBP
0051F007    8BD5            MOV EDX,EBP
0051F009    81ED C62B4000   SUB EBP,pi.00402BC6
0051F00F    2B95 61344000   SUB EDX,DWORD PTR SS:[EBP+403461]
```

`F8` 运行一次后，用“`ESP` 定律”对 `ESP` 下硬件访问断点，`F9` 运行，在下面断下：

```
0051F414    61              POPAD
0051F415    50              PUSH EAX          ; pi.004C3684 //断在这里，EAX 为程序的真正 OEP
0051F416    C3              RETN
004C3684    55              PUSH EBP          //在这里用插件脱壳。脱壳后的程序不能运行，打开 ImportREC,选择进程 PI.exe，OEP 中填入：000c3684，点“IAT 自动搜索”，再点“get imports”，最后点“fix dump”，选择你刚脱壳保存的文件，OK 搞定！，修复后的程序正常运行，再查壳，Delphi 的东东。
004C3685    8BEC            MOV EBP,ESP
004C3687    B9 04000000     MOV ECX,4
004C368C    6A 00          PUSH 0
004C368E    6A 00          PUSH 0
004C3690    49             DEC ECX
004C3691  ^ 75 F9          JNZ SHORT pi.004C368C
```

```
:004BDB62 E87570F8FF          call 00444BDC
:004BDB67 8B45FC              mov eax, dword ptr [ebp-04]
:004BDB6A E8516CF4FF          call 004047C0                         //取注册码长度
:004BDB6F 83F808              cmp eax, 00000008                     //注册码长度是否为 8
:004BDB72 757F                jne 004BDBF3                          //不等于则跳走
:004BDB74 8B45FC              mov eax, dword ptr [ebp-04]
:004BDB77 8A00                mov al, byte ptr [eax]                 //取注册码第 1 位
:004BDB79 8B55FC              mov edx, dword ptr [ebp-04]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004BDB0D(C)

|

:004BDB7C 3A4202              cmp al, byte ptr [edx+02]              //注册码第 1 位与注册码第 3 位比较
:004BDB7F 7550                jne 004BDBD1                            //不等于则跳走
:004BDB81 8B45FC              mov eax, dword ptr [ebp-04]
:004BDB84 80780138            cmp byte ptr [eax+01], 38               //注册码第 2 位是否为"8"
:004BDB88 7547                jne 004BDBD1
:004BDB8A 6A40                push 00000040
:004BDB8C 6848DC4B00          push 004BDC48

* Possible StringData Ref from Code Obj ->"您的卡号是正确的，感谢注册。 "

                     |

:004BDB91 684CDC4B00          push 004BDC4C
:004BDB96 8BC3                mov eax, ebx
:004BDB98 E8EBD7F8FF          call 0044B388
:004BDB9D 50                  push eax

* Reference To: user32.MessageBoxA, Ord:01DDh

                     |

:004BDB9E E84598F4FF          Call 004073E8
:004BDBA3 8D55F4              lea edx, dword ptr [ebp-0C]
:004BDBA6 8B45FC              mov eax, dword ptr [ebp-04]
:004BDBA9 E8C6AEF4FF          call 00408A74
:004BDBAE 8B4DF4              mov ecx, dword ptr [ebp-0C]
:004BDBB1 A150594C00          mov eax, dword ptr [004C5950]
:004BDBB6 8B00                mov eax, dword ptr [eax]
:004BDBB8 8B8010050000        mov eax, dword ptr [eax+00000510]

* Possible StringData Ref from Code Obj ->"edit88"

                     |

:004BDBBE BA74DC4B00          mov edx, 004BDC74
:004BDBC3 E810E5FAFF          call 0046C0D8
:004BDBC8 8BC3                mov eax, ebx
:004BDBCA E8F146FAFF          call 004622C0
:004BDBCF EB42                jmp 004BDC13

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004BDB7F(C), :004BDB88(C)

|

:004BDBD1 6A40                push 00000040
:004BDBD3 6848DC4B00          push 004BDC48

* Possible StringData Ref from Code Obj ->"您的卡号是非法的，请注册。 "

                     |

:004BDBD8 687CDC4B00          push 004BDC7C
:004BDBDD 8BC3                mov eax, ebx
:004BDBDF E8A4D7F8FF          call 0044B388
:004BDBE4 50                  push eax

* Reference To: user32.MessageBoxA, Ord:01DDh

                     |

:004BDBE5 E8FE97F4FF          Call 004073E8
:004BDBEA 8BC3                mov eax, ebx
:004BDBEC E8CF46FAFF          call 004622C0
:004BDBF1 EB20                jmp 004BDC13

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004BDB72(C)

|

:004BDBF3 6A40                push 00000040
:004BDBF5 6848DC4B00          push 004BDC48

* Possible StringData Ref from Code Obj ->"您的卡号是非法的，请注册。 "

                     |

:004BDBFA 687CDC4B00          push 004BDC7C
:004BDBFF 8BC3                mov eax, ebx
:004C0676 A1B86D4C00          mov eax, dword ptr [004C6DB8]
:004C067B 8B10                mov edx, dword ptr [eax]
:004C067D FF5244             call [edx+44]
:004C0680 83BB1405000007     cmp dword ptr [ebx+00000514], 00000007
:004C0687 7E38                jle 004C06C1
:004C0689 8BC3                mov eax, ebx
:004C068B E8C42B0000          call 004C3254                          //关键 CALL，跟进来！
:004C0690 84C0                test al, al
:004C0692 752D                jne 004C06C1
:004C0694 6A40                push 00000040
:004C0696 682C154C00          push 004C152C

* Possible StringData Ref from Code Obj ->"您已经试用了 7 天，如果您感觉满意，欢迎购买。"

                     |

:004C069B 6858154C00          push 004C1558
:004C06A0 8BC3                mov eax, ebx
:004C06A2 E8E1ACF8FF          call 0044B388
:004C06A7 50                  push eax

* Referenced by a CALL at Addresses:

|:004BE377   , :004C068B   , :004C3226   , :004C3238

|

:004C3254 55                  push ebp
:004C3255 8BEC                mov ebp, esp
:004C3257 6A00                push 00000000
:004C3259 6A00                push 00000000
:004C325B 6A00                push 00000000
:004C325D 53                  push ebx
:004C325E 56                  push esi
:004C325F 8BF0                mov esi, eax
:004C3261 33C0                xor eax, eax
:004C3263 55                  push ebp
:004C3264 6804334C00          push 004C3304
:004C3269 64FF30              push dword ptr fs:[eax]
:004C326C 648920              mov dword ptr fs:[eax], esp
:004C326F C645FF00            mov [ebp-01], 00
:004C3273 33DB                xor ebx, ebx
:004C3275 8D55F8              lea edx, dword ptr [ebp-08]
:004C3278 8B860C050000        mov eax, dword ptr [esi+0000050C]
:004C327E E85919F8FF          call 00444BDC
:004C3283 8B45F8              mov eax, dword ptr [ebp-08]            //"Label_reg"
:004C3286 0FB64001            movzx eax, byte ptr [eax+01]           //EAX="a"
:004C328A 83E80C              sub eax, 0000000C                      //EAX="U"
:004C328D 8B9618050000        mov edx, dword ptr [esi+00000518]      //读入注册码
:004C3293 3A4204              cmp al, byte ptr [edx+04]              //比较注册码第 5 位是为"U"
:004C3296 7504                jne 004C329C
:004C3298 C645FF01            mov [ebp-01], 01
```



* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:004C3296(C)  

|  

:004C329C 807DFF00                cmp byte ptr [ebp-01], 00  

:004C32A0 7444                    je 004C32E6  

:004C32A2 8D45F4                  lea eax, dword ptr [ebp-0C]  

:004C32A5 50                      push eax  

:004C32A6 8B8618050000            mov eax, dword ptr [esi+00000518]  

:004C32AC B902000000              mov ecx, 00000002  

:004C32B1 BA07000000              mov edx, 00000007  

:004C32B6 E85D17F4FF              call 00404A18  

:004C32BB 8B45F4                  mov eax, dword ptr [ebp-0C]                  //取注册码最后两位  

:004C32BE E8315AF4FF              call 00408CF4                                 //转换为十六进制  

:004C32C3 8BF0                    mov esi, eax  

:004C32C5 8BCE                    mov ecx, esi  

:004C32C7 49                      dec ecx  

:004C32C8 83E902                  sub ecx, 00000002  

:004C32CB 7C17                    jl 004C32E4                                    //小于 3 则跳走  

:004C32CD 41                      inc ecx                                         //循环次数为 N－2  

:004C32CE BB02000000              mov ebx, 00000002                               //除数从 2 开始  

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:004C32E2(C)  

|  

:004C32D3 8BC6                    mov eax, esi  

:004C32D5 99                      cdq  

:004C32D6 F7FB                    idiv ebx  

:004C32D8 85D2                    test edx, edx  

:004C32DA 7504                    jne 004C32E0                                    //未整除则继续循环  

:004C32DC 33DB                    xor ebx, ebx  

:004C32DE EB06                    jmp 004C32E6  

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:004C32DA(C)  

|  

:004C32E0 43                      inc ebx  

:004C32E1 49                      dec ecx  

:004C32E2 75EF                    jne 004C32D3                                    //该循环实际是判断最后两位是否为素数  

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:004C32CB(C)  

|  

:004C32E4 B301                    mov bl, 01  

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

|:004C32A0(C), :004C32DE(U)  

|  

:004C32E6 33C0                    xor eax, eax  

:004C32E8 5A                      pop edx  

:004C32E9 59                      pop ecx  

:004C32EA 59                      pop ecx  

:004C32EB 648910                  mov dword ptr fs:[eax], edx  

:004C32EE 680B334C00              push 004C330B  

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:004C3309(U)  

|  

:004C32F3 8D45F4                  lea eax, dword ptr [ebp-0C]  

:004C32F6 E80D12F4FF              call 00404508  

:004C32FB 8D45F8                  lea eax, dword ptr [ebp-08]  

:004C32FE E80512F4FF              call 00404508  

:004C3303 C3                      ret  

:004C3304 E9A30BF4FF              jmp 00403EAC  

:004C3309 EBE8                    jmp 004C32F3  

:004C330B 8BC3                    mov eax, ebx  

:004C330D 5E                      pop esi  

:004C330E 5B                      pop ebx  

:004C330F 8BE5                    mov esp, ebp  

:004C3311 5D                      pop ebp  

:004C3312 C3                      ret  

所以，正确的注册码是：

1. 注册码长度为 8 位字符；
2. 注册码的第 1 位和第 3 位相等；
3. 注册码的第 2 位为字符“8”；
4. 注册码的第 5 位为字符“U”；
5. 注册码的最后两位为素数。

注册码保存在：

```
[HKEY_LOCAL_MACHINE\SOFTWARE\ExeSoft\change]
```

`"edit88"="0808U023"`                                          //可用的注册码。

本文完。

`4dd466d0e1e0cc123afbe504da2462ca`
