
# 名片网印工艺 V2.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6107.htm`](https://www.pediy.com/kssd/pediy06/pediy6107.htm)

下载页面： [`www.onlinedown.net/mpwygy.htm`](http://www.onlinedown.net/mpwygy.htm)

整理日期： 2003.4.23

文件大小： 1855KB

使用平台： Win9x/Me/NT/2000/XP

**【软件简介】**：详细讲述手工网印名片的全过程。包括材料配置、网印技术、相关问题的解决等。是一部指导丝印从业者印好普通名片、承接金属、塑料等高档次名片的教程软件。

**【软件限制】**：功能限制

**【作者声明】**：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

**【破解工具】**：`TRW2000` 娃娃修改版、`Ollydbg1.09`、`PEiD`、`W32Dasm 9.0` 白金版

—————————————————————————————————

**【过程】**：

跟踪这个程序我有一种似曾相识的感觉，是的，和《电脑故障、软件一查通 V1.2》太相似了！算法过程都是在一个 `ocx` 文件里，流程几乎是一样的！所以我觉得如果这些程序不是一个作者的话，那么它们就采用了相同的某个加密算法工具包加密的！如果有哪位朋友知道的话请指点迷津，谢谢！猜测一下：`Nelco`。这次虽然我跟进了算法 `CALL`，但是因为 `nslock15vb5.ocx` 里几乎全是算法循环，最后还是放弃了。

需要说明的是：这个程序注册后才可以下载数据文件，如果服务器方面还有对注册用户机器码的验证，可能是无法下载数据文件的。

`名片网印工艺 V2.0.exe` 无壳。 `Visual Basic 6.0` 编写。

机器号：`566C71F6575BD4B3`

试炼码：`13572468`

—————————————————————————————————

可以用 `S` 大法+`BPM` 内存断点 找到下面的地方

```
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0042E13A(C)

|

:0042E154 83EC10                 sub esp, 00000010
:0042E157 8B45E8                 mov eax, dword ptr [ebp-18]
                                  ==== >EAX=13572468           试炼码
:0042E15A 8BD4                   mov edx, esp
:0042E15C B908000000             mov ecx, 00000008
:0042E161 894DD0                 mov dword ptr [ebp-30], ecx
:0042E164 8945D8                 mov dword ptr [ebp-28], eax
:0042E167 890A                   mov dword ptr [edx], ecx
:0042E169 8B4DD4                 mov ecx, dword ptr [ebp-2C]
:0042E16C 6807000368             push 68030007
:0042E171 56                     push esi
:0042E172 894A04                 mov dword ptr [edx+04], ecx
:0042E175 8B0E                   mov ecx, dword ptr [esi]
:0042E177 895DE8                 mov dword ptr [ebp-18], ebx
:0042E17A 894208                 mov dword ptr [edx+08], eax
:0042E17D 8B45DC                 mov eax, dword ptr [ebp-24]
:0042E180 89420C                 mov dword ptr [edx+0C], eax
:0042E183 FF912C030000           call dword ptr [ecx+0000032C]
:0042E189 8D55E0                 lea edx, dword ptr [ebp-20]
:0042E18C 50                     push eax
:0042E18D 52                     push edx
:0042E18E FFD7                   call edi
:0042E190 50                     push eax

\* Reference To: MSVBVM60.__vbaLateIdSt, Ord:0000h

                                  |

:0042E191 FF152C124000           Call dword ptr [0040122C]
                                  ==== >几组字符和“//\/e7C0$0fT\/\/ArE”运算生成注册表里的信息
:0042E197 8D45E0                 lea eax, dword ptr [ebp-20]
:0042E19A 8D4DE4                 lea ecx, dword ptr [ebp-1C]
:0042E19D 50                     push eax
:0042E19E 51                     push ecx
:0042E19F 6A02                   push 00000002

\* Reference To: MSVBVM60.__vbaFreeObjList, Ord:0000h

                                  |

:0042E1A1 FF1548104000           Call dword ptr [00401048]

\* Reference To: MSVBVM60.__vbaFreeVar, Ord:0000h

                                  |

:0042E1A7 8B1D28104000           mov ebx, dword ptr [00401028]
:0042E1AD 83C40C                 add esp, 0000000C
:0042E1B0 8D4DD0                 lea ecx, dword ptr [ebp-30]
:0042E1B3 FFD3                   call ebx
:0042E1B5 8B16                   mov edx, dword ptr [esi]
:0042E1B7 6A00                   push 00000000
:0042E1B9 6806000368             push 68030006
:0042E1BE 56                     push esi
:0042E1BF FF922C030000           call dword ptr [edx+0000032C]
:0042E1C5 50                     push eax
:0042E1C6 8D45E4                 lea eax, dword ptr [ebp-1C]
:0042E1C9 50                     push eax
:0042E1CA FFD7                   call edi
:0042E1CC 8D4DD0                 lea ecx, dword ptr [ebp-30]
:0042E1CF 50                     push eax
:0042E1D0 51                     push ecx

\* Reference To: MSVBVM60.__vbaLateIdCallLd, Ord:0000h

                                  |

:0042E1D1 FF1520114000           Call dword ptr [00401120]
                                  ==== >关键 CALL！这里会进入新的“天空”  ^O^ ^O^
:0042E1D7 83C410                 add esp, 00000010
:0042E1DA 50                     push eax

\* Reference To: MSVBVM60.__vbaBoolVar, Ord:0000h

                                  |

:0042E1DB FF15BC104000           Call dword ptr [004010BC]
:0042E1E1 8D4DE4                 lea ecx, dword ptr [ebp-1C]
:0042E1E4 6689855CFFFFFF         mov word ptr [ebp+FFFFFF5C], ax

\* Reference To: MSVBVM60.__vbaFreeObj, Ord:0000h

                                  |

:0042E1EB FF1544124000           Call dword ptr [00401244]
:0042E1F1 8D4DD0                 lea ecx, dword ptr [ebp-30]
:0042E1F4 FFD3                   call ebx
:0042E1F6 6683BD5CFFFFFF00       cmp word ptr [ebp+FFFFFF5C], 0000
:0042E1FE 0F84E8000000           je 0042E2EC
                                  ==== >跳则 OVER！
:0042E204 8B16                   mov edx, dword ptr [esi]
:0042E206 56                     push esi
:0042E207 FF92FC020000           call dword ptr [edx+000002FC]
:0042E20D 50                     push eax
:0042E20E 8D45E4                 lea eax, dword ptr [ebp-1C]
:0042E211 50                     push eax
:0042E212 FFD7                   call edi
:0042E214 8BD8                   mov ebx, eax
:0042E216 6A00                   push 00000000
:0042E218 53                     push ebx
:0042E219 8B0B                   mov ecx, dword ptr [ebx]
:0042E21B FF919C000000           call dword ptr [ecx+0000009C]
:0042E221 85C0                   test eax, eax
:0042E223 DBE2                   fclex
:0042E225 7D12                   jge 0042E239
:0042E227 689C000000             push 0000009C
:0042E22C 68B4004100             push 004100B4
:0042E231 53                     push ebx
:0042E232 50                     push eax

\* Reference To: MSVBVM60.__vbaHresultCheckObj, Ord:0000h

                                  |

:0042E233 FF1568104000           Call dword ptr [00401068]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0042E225(C)

|

\* Reference To: MSVBVM60.__vbaFreeObj, Ord:0000h

                                  |

:0042E239 8B1D44124000           mov ebx, dword ptr [00401244]
:0042E23F 8D4DE4                 lea ecx, dword ptr [ebp-1C]
:0042E242 FFD3                   call ebx
:0042E244 B904000280             mov ecx, 80020004
:0042E249 B80A000000             mov eax, 0000000A
:0042E24E 894DA8                 mov dword ptr [ebp-58], ecx
:0042E251 894DB8                 mov dword ptr [ebp-48], ecx
:0042E254 894DC8                 mov dword ptr [ebp-38], ecx
:0042E257 8D5590                 lea edx, dword ptr [ebp-70]
:0042E25A 8D4DD0                 lea ecx, dword ptr [ebp-30]
:0042E25D 8945A0                 mov dword ptr [ebp-60], eax
:0042E260 8945B0                 mov dword ptr [ebp-50], eax
:0042E263 8945C0                 mov dword ptr [ebp-40], eax

\* Possible StringData Ref from Code Obj ->"鑜孮 b 烺!"

                                  |

:0042E266 C74598C8004100         mov [ebp-68], 004100C8
:0042E26D C7459008000000         mov [ebp-70], 00000008

\* Reference To: MSVBVM60.__vbaVarDup, Ord:0000h

                                  |

:0042E274 FF15F0114000           Call dword ptr [004011F0]
:0042E27A 8D55A0                 lea edx, dword ptr [ebp-60]
:0042E27D 8D45B0                 lea eax, dword ptr [ebp-50]
:0042E280 52                     push edx
:0042E281 8D4DC0                 lea ecx, dword ptr [ebp-40]
:0042E284 50                     push eax
```



```
:0042E285 51                                                                 push ecx
:0042E286 8D55D0                                                            lea edx, dword ptr [ebp-30]
:0042E289 6A00                                                              push 00000000
:0042E28B 52                                                                push edx

* Reference To: MSVBVM60.rtcMsgBox, Ord:0253h
|
:0042E28C FF1594104000                                                    Call dword ptr [00401094]
                                                                         ==== > 呵呵，胜利女神！

:0042E292 8D45A0                                                            lea eax, dword ptr [ebp-60]
:0042E295 8D4DB0                                                            lea ecx, dword ptr [ebp-50]
:0042E298 50                                                                push eax
:0042E299 8D55C0                                                            lea edx, dword ptr [ebp-40]
:0042E29C 51                                                                push ecx
:0042E29D 8D45D0                                                            lea eax, dword ptr [ebp-30]
:0042E2A0 52                                                                push edx
:0042E2A1 50                                                                push eax
:0042E2A2 6A04                                                              push 00000004

* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h
|
:0042E2A4 FF1538104000                                                    Call dword ptr [00401038]

:0042E2AA 8B0E                                                              mov ecx, dword ptr [esi]
:0042E2AC 83C414                                                            add esp, 00000014
:0042E2AF 56                                                                push esi
:0042E2B0 FF911C030000                                                    call dword ptr [ecx+0000031C]
:0042E2B6 8D55E4                                                            lea edx, dword ptr [ebp-1C]
:0042E2B9 50                                                                push eax
:0042E2BA 52                                                                push edx
:0042E2BB FFD7                                                              call edi
:0042E2BD 8BF0                                                              mov esi, eax
:0042E2BF 6AFF                                                              push FFFFFFFF
:0042E2C1 56                                                                push esi
:0042E2C2 8B06                                                              mov eax, dword ptr [esi]
:0042E2C4 FF908C000000                                                    call dword ptr [eax+0000008C]
:0042E2CA 85C0                                                              test eax, eax
:0042E2CC DBE2                                                              fclex
:0042E2CE 7D12                                                              jge 0042E2E2
:0042E2D0 688C000000                                                        push 0000008C

* Possible StringData Ref from Code Obj ->"馧?檉??"
|
:0042E2D5 6884004100                                                        push 00410084
:0042E2DA 56                                                                push esi
:0042E2DB 50                                                                push eax

* Reference To: MSVBVM60.__vbaHresultCheckObj, Ord:0000h
|
:0042E2DC FF1568104000                                                    Call dword ptr [00401068]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0042E2CE(C)
|
:0042E2E2 8D4DE4                                                            lea ecx, dword ptr [ebp-1C]
:0042E2E5 FFD3                                                              call ebx
:0042E2E7 E986010000                                                        jmp 0042E472
```

…… ……省略…… ……

```
* Reference To: MSVBVM60.rtcMsgBox, Ord:0253h
|
:0042E334 FF1594104000                                                    Call dword ptr [00401094]
                                                                         ==== > BAD BOY！
```

—————————————————————————————————

用`S`大法加`BPM`会来到下面的地方。其实是同目录下的`nslock15vb5.ocx`领空。

…… ……省略…… ……

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1647225B(C)
|
:1647226E 8D4588                                                            lea eax, dword ptr [ebp-78]
:16472271 50                                                                push eax

* Reference To: MSVBVM50.__vbaStrVarMove, Ord:0000h
|
:16472272 FF152C624716                                                    Call dword ptr [1647622C]
:16472278 8BD0                                                              mov edx, eax
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

下面的值是根据机器号生成的：（具体算法不清楚 `^O^` `^O^`）

```
00454268  31 00 45 00 36 00 34 00 31 00 38 00 35 00 35 00  1.E.6.4.1.8.5.5.
00454278  30 00 32 00 37 00 36 00 30 00 33 00 31 00 30 00  0.2.7.6.0.3.1.0.
00454288  30 00 37 00 35 00 37                                        0.7.5.7
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

```
:1647227A 8D4DB0                                                            lea ecx, dword ptr [ebp-50]

* Reference To: MSVBVM50.__vbaStrMove, Ord:0000h
|
:1647227D FF15AC634716                                                    Call dword ptr [164763AC]
:16472283 8D4D88                                                            lea ecx, dword ptr [ebp-78]

* Reference To: MSVBVM50.__vbaFreeVar, Ord:0000h
|
:16472286 FF1524624716                                                    Call dword ptr [16476224]

* Reference To: MSVBVM50.__vbaStrCopy, Ord:0000h
|
:1647228C 8B3D44634716                                                    mov edi, dword ptr [16476344]

* Possible StringData Ref from Code Obj ->"/\\/e7C0$0fT\\/\\/ArE"
|
:16472292 BA1C3B4616                                                        mov edx, 16463B1C
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

这个值是固定的：

```
004544EC  2F 00 5C 00 2F 00 65 00 37 00 43 00 30 00 24 00  /.\\./.e.7.C.0.$.
004544FC  30 00 66 00 54 00 5C 00 2F 00 5C 00 2F 00 41 00  0.f.T.\\./.\\./.A.
0045450C  72 00 45                                                    r.E
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

```
:16472297 8D4D9C                                                            lea ecx, dword ptr [ebp-64]
:1647229A FFD7                                                              call edi
:1647229C 8D4D9C                                                            lea ecx, dword ptr [ebp-64]
:1647229F 8D55B0                                                            lea edx, dword ptr [ebp-50]
:164722A2 51                                                                push ecx
:164722A3 52                                                                push edx
:164722A4 E857060000                                                        call 16472900
                                                                         ==== > 上面 2 组值异或运算  得到下面的结果

:164722A9 8BD0                                                              mov edx, eax
                                                                         ==== > EDX=1870553471
```

—————————————————————————————————

继续跟踪：`GO GO GO` `^O^` `^O^`

```
:164689C8 FF92F4070000                                                    call dword ptr [edx+000007F4]
:164689CE 8B0E                                                              mov ecx, dword ptr [esi]
:164689D0 8D55DC                                                            lea edx, dword ptr [ebp-24]
:164689D3 52                                                                push edx
:164689D4 56                                                                push esi
:164689D5 FF91F8070000                                                    call dword ptr [ecx+000007F8]
:164689DB 8B45E0                                                            mov eax, dword ptr [ebp-20]
                                                                         ==== > EDX=1870553471

:164689DE 8B4DDC                                                            mov ecx, dword ptr [ebp-24]
                                                                         ==== > ECX=R64DG-RXGMD-G9BM6-V3H9G-2R348
                                                                         ==== > 这个是你的 Windows 的 ProductKey

:164689E1 50                                                                push eax
:164689E2 51                                                                push ecx

* Reference To: MSVBVM50.__vbaStrCat, Ord:0000h
|
:164689E3 FF1550624716                                                    Call dword ptr [16476250]
                                                                         ==== > 把上面 2 组字符连接起来！

* Reference To: MSVBVM50.__vbaStrMove, Ord:0000h
|
:164689E9 8B1DAC634716                                                    mov ebx, dword ptr [164763AC]
:164689EF 8BD0                                                              mov edx, eax
                                                                         ==== > EDX=1870553471R64DG-RXGMD-G9BM6-V3H9G-2R348

:164689F1 8D4DD8                                                            lea ecx, dword ptr [ebp-28]
:164689F4 FFD3                                                              call ebx
:164689F6 50                                                                push eax
:164689F7 E874710000                                                        call 1646FB70
                                                                         ==== > 加密算法 CALL ①！     得出下面的结果！

:164689FC 8BD0                                                              mov edx, eax
                                                                         ==== > EDX=566C71F6575BD4B395FE34B037F4E04B4DD6D78B

:164689FE 8D4DE8                                                            lea ecx, dword ptr [ebp-18]
:16468A01 FFD3                                                              call ebx
:16468A03 8D55D8                                                            lea edx, dword ptr [ebp-28]
:16468A06 8D45DC                                                            lea eax, dword ptr [ebp-24]
:16468A09 52                                                                push edx
:16468A0A 8D4DE0                                                            lea ecx, dword ptr [ebp-20]
:16468A0D 50                                                                push eax
:16468A0E 51                                                                push ecx
:16468A0F 6A03                                                              push 00000003

* Reference To: MSVBVM50.__vbaFreeStrList, Ord:0000h
|
:16468A11 FF154C634716                                                    Call dword ptr [1647634C]
:16468A17 0FBF4646                                                        movsx eax, word ptr [esi+46]
:16468A1B 83C410                                                            add esp, 00000010
```



:16468A1E 8D55E8  lea edx, dword ptr [ebp-18]  

:16468A21 8955C0  mov dword ptr [ebp-40], edx  

:16468A24 8D4DB8  lea ecx, dword ptr [ebp-48]  

:16468A27 50  push eax  

:16468A28 8D55C8  lea edx, dword ptr [ebp-38]  

:16468A2B 51  push ecx  

:16468A2C 52  push edx  

:16468A2D C745B808400000  mov [ebp-48], 00004008 

\* Reference To: `MSVBVM50.rtcLeftCharVar`, Ord:0269h  

&nbsp; |  

:16468A34 FF15A0634716  Call dword ptr [164763A0]  

&nbsp; ===&gt;取上面所得字符串的前 16 位！ 

:16468A3A 8D45C8  lea eax, dword ptr [ebp-38]  

:16468A3D 50  push eax 

\* Reference To: `MSVBVM50.__vbaStrVarMove`, Ord:0000h  

&nbsp; |  

:16468A3E FF152C624716  Call dword ptr [1647622C]  

:16468A44 8BD0  mov edx, eax  

&nbsp; ===&gt;EDX=566C71F6575BD4B3 

…… ……省略…… …… 

—————————————————————————————————  

继续跟踪：GO GO GO ^O^ ^O^ 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:16468768(C)  

|  

:1646877C 8B55DC  mov edx, dword ptr [ebp-24]  

&nbsp; ===&gt;EDX=566C71F6575BD4B3 

:1646877F 8B463C  mov eax, dword ptr [esi+3C]  

&nbsp; ===&gt;EAX=904 固定值 

\* Reference To: `MSVBVM50.__vbaStrCat`, Ord:0000h  

&nbsp; |  

:16468782 8B1D50624716  mov ebx, dword ptr [16476250]  

:16468788 52  push edx  

:16468789 50  push eax  

:1646878A FFD3  call ebx  

&nbsp; ===&gt;连接以上 2 个字符串 

\* Reference To: `MSVBVM50.__vbaStrMove`, Ord:0000h  

&nbsp; |  

:1646878C 8B3DAC634716  mov edi, dword ptr [164763AC]  

:16468792 8BD0  mov edx, eax  

&nbsp; ===&gt;EDX=566C71F6575BD4B3904 

:16468794 8D4DD8  lea ecx, dword ptr [ebp-28]  

:16468797 FFD7  call edi  

:16468799 50  push eax 

\* Possible StringData Ref from Code Obj -&gt;"123"  

&nbsp; |  

:1646879A 685C3E4616  push 16463E5C  

&nbsp; ===&gt;16463E5C=123 固定值 

:1646879F FFD3  call ebx  

&nbsp; ===&gt;连接以上 2 个字符串 

:164687A1 8BD0  mov edx, eax  

&nbsp; ===&gt;EDX=566C71F6575BD4B3904123 

:164687A3 8D4DD4  lea ecx, dword ptr [ebp-2C]  

:164687A6 FFD7  call edi  

:164687A8 50  push eax  

:164687A9 E8C2730000  call 1646FB70  

&nbsp; ===&gt;加密算法 CALL ②！和 164689F7 call 处的流程一致 

:164687AE 8BD0  mov edx, eax  

&nbsp; ===&gt;EDX=F968F0BBA09FA0E740C482592F641FBD33BAC0FF 

:164687B0 8D4DE8  lea ecx, dword ptr [ebp-18]  

:164687B3 FFD7  call edi  

:164687B5 8D4DD4  lea ecx, dword ptr [ebp-2C]  

:164687B8 8D55D8  lea edx, dword ptr [ebp-28]  

:164687BB 51  push ecx  

:164687BC 8D45DC  lea eax, dword ptr [ebp-24]  

:164687BF 52  push edx  

:164687C0 50  push eax  

:164687C1 6A03  push 00000003 

\* Reference To: `MSVBVM50.__vbaFreeStrList`, Ord:0000h  

&nbsp; |  

:164687C3 FF154C634716  Call dword ptr [1647634C]  

:164687C9 8B5E40  mov ebx, dword ptr [esi+40]  

:164687CC 83C410  add esp, 00000010  

:164687CF 83EC10  sub esp, 00000010  

:164687D2 895DAC  mov dword ptr [ebp-54], ebx  

:164687D5 8BDC  mov ebx, esp  

:164687D7 B90A000000  mov ecx, 0000000A  

:164687DC B804000280  mov eax, 80020004  

:164687E1 83EC10  sub esp, 00000010  

:164687E4 890B  mov dword ptr [ebx], ecx  

:164687E6 8B8D78FFFFFF  mov ecx, dword ptr [ebp+FFFFFF78] 

\* Possible StringData Ref from Code Obj -&gt;"LiberationKey"  

&nbsp; |  

:164687EC BA3C3E4616  mov edx, 16463E3C  

:164687F1 C7459408000000  mov [ebp-6C], 00000008  

:164687F8 894B04  mov dword ptr [ebx+04], ecx  

:164687FB 8BCC  mov ecx, esp  

:164687FD 83EC10  sub esp, 00000010 

\* Possible StringData Ref from Code Obj -&gt;"ActiveLock"  

&nbsp; |  

:16468800 C7459CD03D4616  mov [ebp-64], 16463DD0  

:16468807 894308  mov dword ptr [ebx+08], eax  

:1646880A 8B4580  mov eax, dword ptr [ebp-80]  

:1646880D C745A408000000  mov [ebp-5C], 00000008  

:16468814 89430C  mov dword ptr [ebx+0C], eax  

:16468817 B808000000  mov eax, 00000008  

:1646881C 8901  mov dword ptr [ecx], eax  

:1646881E 8B4588  mov eax, dword ptr [ebp-78]  

:16468821 894104  mov dword ptr [ecx+04], eax  

:16468824 8BC4  mov eax, esp  

:16468826 83EC10  sub esp, 00000010  

:16468829 895108  mov dword ptr [ecx+08], edx  

:1646882C 8B5590  mov edx, dword ptr [ebp-70]  

:1646882F 89510C  mov dword ptr [ecx+0C], edx  

:16468832 8B4D94  mov ecx, dword ptr [ebp-6C]  

:16468835 8B5598  mov edx, dword ptr [ebp-68]  

:16468838 8908  mov dword ptr [eax], ecx  

:1646883A 8B4D9C  mov ecx, dword ptr [ebp-64]  

:1646883D 895004  mov dword ptr [eax+04], edx  

:16468840 8B55A0  mov edx, dword ptr [ebp-60]  

:16468843 894808  mov dword ptr [eax+08], ecx  

:16468846 8B4DA4  mov ecx, dword ptr [ebp-5C]  

:16468849 89500C  mov dword ptr [eax+0C], edx  

:1646884C 8B55A8  mov edx, dword ptr [ebp-58]  

:1646884F 8BC4  mov eax, esp  

:16468851 8908  mov dword ptr [eax], ecx  

:16468853 8B4DAC  mov ecx, dword ptr [ebp-54]  

:16468856 895004  mov dword ptr [eax+04], edx  

:16468859 8B55B0  mov edx, dword ptr [ebp-50]  

:1646885C 894808  mov dword ptr [eax+08], ecx  

:1646885F 89500C  mov dword ptr [eax+0C], edx  

:16468862 E879960000  call 16471EE0  

:16468867 8BD0  mov edx, eax  

:16468869 8D4DE0  lea ecx, dword ptr [ebp-20]  

:1646886C FFD7  call edi  

:1646886E 8D45E8  lea eax, dword ptr [ebp-18]  

:16468871 8945AC  mov dword ptr [ebp-54], eax  

:16468874 0FBF4E44  movsx ecx, word ptr [esi+44]  

:16468878 8D55A4  lea edx, dword ptr [ebp-5C]  

:1646887B 51  push ecx  

:1646887C 8D45C4  lea eax, dword ptr [ebp-3C]  

:1646887F 52  push edx  

:16468880 50  push eax  

:16468881 C745A408400000  mov [ebp-5C], 00004008 

\* Reference To: `MSVBVM50.rtcLeftCharVar`, Ord:0269h  

&nbsp; |  

:16468888 FF15A0634716  Call dword ptr [164763A0]  

\====&gt;用 `MSVBVM50.rtcLeftCharVar` 取 F968F0BBA09FA0E740C482592F641FBD33BAC0FF 的左边 16 位  

&nbsp; ===&gt;F968F0BBA09FA0E7 这个就是注册码！ ^O^ ^O^ 



# 反汇编代码分析

## 数据交换与比较

:1646888E 8B4DE0 mov ecx, dword ptr [ebp-20]

====>ECX=13572468 试炼码

:16468891 8D55C4 lea edx, dword ptr [ebp-3C]

:16468894 8D4594 lea eax, dword ptr [ebp-6C]

:16468897 52 push edx

:16468898 50 push eax

:16468899 894D9C mov dword ptr [ebp-64], ecx

:1646889C C7459408800000 mov [ebp-6C], 00008008

* Reference To: `MSVBVM50.__vbaVarTstEq`, Ord:0000h

:164688A3 FF15C0624716 Call dword ptr [164762C0]

====>呵呵，`MSVBVM50.__vbaVarTstEq` 比较 CALL！

:164688A9 8D4DC4 lea ecx, dword ptr [ebp-3C]

:164688AC 668BF8 mov di, ax

## 进入加密算法 CALL

进入加密算法 CALL：`164689F7 call 1646FB70`

* Referenced by a CALL at Addresses:

:164687A9 , :164689F7

…… 省略 ……

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:164703B1(U)

:1647014D B901000000 mov ecx, 00000001

:16470152 663BC1 cmp ax, cx

:16470155 0F8C5B020000 jl 164703B6

:1647015B 894D8C mov dword ptr [ebp-74], ecx

:1647015E 668B4DCC mov cx, word ptr [ebp-34]

:16470162 666BC940 imul cx, 0040

:16470166 0F80F10F0000 jo 1647115D

:1647016C 666BDB04 imul bx, 0004

:16470170 0F80E70F0000 jo 1647115D

:16470176 6603CB add cx, bx

:16470179 8D5584 lea edx, dword ptr [ebp-7C]

:1647017C 0F80DB0F0000 jo 1647115D

:16470182 6603C8 add cx, ax

:16470185 8B45DC mov eax, dword ptr [ebp-24]

:16470188 52 push edx

:16470189 C7458402000000 mov [ebp-7C], 00000002

:16470190 0F80C70F0000 jo 1647115D

:16470196 0FBFD1 movsx edx, cx

:16470199 52 push edx

====>取字符的某几位

:1647019A 50 push eax

## 内存数据

内存中的值：

```
004547F4  31 00 38 00 37 00 30 00 35 00 35 00 33 00 34 00  1.8.7.0.5.5.3.4.
00454804  37 00 31 00 52 00 36 00 34 00 44 00 47 00 2D 00  7.1.R.6.4.D.G.-.
00454814  52 00 58 00 47 00 4D 00 44 00 2D 00 47 00 39 00  R.X.G.M.D.-.G.9.
00454824  42 00 4D 00 36 00 2D 00 56 00 33 00 48 00 39 00  B.M.6.-.V.3.H.9.
00454834  47 00 2D 00 32 00 52 00 33 00 34 00 38 00 80 00  G.-.2.R.3.4.8.€.
00454844  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
00454854  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
00454864  00 00 00 00 00 00 00 00 00 00 00 00 01 00 50 00  ............P.
```

## 字符串处理与字符反转

* Reference To: `MSVBVM50.rtcMidCharBstr`, Ord:0277h

:1647019B FF15A0624716 Call dword ptr [164762A0]

====>每隔 4 位依次倒序取上面字符串

:164701A1 8BD0 mov edx, eax

:164701A3 8D4DAC lea ecx, dword ptr [ebp-54]

:164701A6 FFD6 call esi

:164701A8 50 push eax

* Reference To: `MSVBVM50.rtcAnsivalueBstr`, Ord:0204h

:164701A9 FF153C624716 Call dword ptr [1647623C]

====>取字符所对应的 HEX 值

:164701AF 6689857CFFFFFF mov word ptr [ebp+FFFFFF7C], ax

…… 省略 ……

## HEX 值转换

* Reference To: `MSVBVM50.rtcHexBstrFromVar`, Ord:023Ch

:16470238 FF153C634716 Call dword ptr [1647633C]

====>取其 HEX 值

:1647023E 8BD0 mov edx, eax

:16470240 8D4DA8 lea ecx, dword ptr [ebp-58]

:16470243 FFD6 call esi

:16470245 8B9590FEFFFF mov edx, dword ptr [ebp+FFFFFE90]

:1647024B 50 push eax

:1647024C 8B45B4 mov eax, dword ptr [ebp-4C]

:1647024F 8B480C mov ecx, dword ptr [eax+0C]

:16470252 8B0411 mov eax, dword ptr [ecx+edx]

:16470255 50 push eax

## 字符串连接

* Reference To: `MSVBVM50.__vbaStrCat`, Ord:0000h

:16470256 FF1550624716 Call dword ptr [16476250]

====>依次连接所得字符

:1647025C 8BD0 mov edx, eax

1. ====>`EDX=31383730` 即：`1870`
2. ====>`EDX=35353334` 即：`5534`
3. ====>`EDX=37315236` 即：`71R6`
4. ====>`EDX=3444472D` 即：`4DG-`
5. ====>`EDX=5258474D` 即：`RXGM`
6. ====>`EDX=442D4739` 即：`D-G9`
7. ====>`EDX=424D362D` 即：`BM6-`
8. ====>`EDX=56334839` 即：`V3H9`
9. ====>`EDX=472D3252` 即：`G-2R`
10. ====>`EDX=33343880` 即：`348`

…… 省略 ……

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:1647035D(U)

:1647036A 8B500C mov edx, dword ptr [eax+0C]

:1647036D 8B858CFEFFFF mov eax, dword ptr [ebp+FFFFFE8C]

:16470373 6838514616 push 16465138

:16470378 8B0C02 mov ecx, dword ptr [edx+eax]

:1647037B 51 push ecx

* Reference To: `MSVBVM50.__vbaStrCat`, Ord:0000h

:1647037C FF1550624716 Call dword ptr [16476250]

====>依次连接所得字符

:16470382 8BD0 mov edx, eax

11. ====>`EDX=00000000`
12. ====>`EDX=00000000`
13. ====>`EDX=00000000`
14. ====>`EDX=00000000`
15. ====>`EDX=00000000`
16. ====>`EDX=00000150`

:16470384 8D4DAC lea ecx, dword ptr [ebp-54]

:16470387 FFD6 call esi

:16470389 8BD0 mov edx, eax

:1647038B 8B45B4 mov eax, dword ptr [ebp-4C]

:1647038E 8B480C mov ecx, dword ptr [eax+0C]

:16470391 03CB add ecx, ebx

:16470393 FFD7 call edi

:16470395 8D4DAC lea ecx, dword ptr [ebp-54]

## 资源释放与循环逻辑

* Reference To: `MSVBVM50.__vbaFreeStr`, Ord:0000h

:16470398 FF15D4634716 Call dword ptr [164763D4]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:164702ED(C)

:1647039E 8B5DD8 mov ebx, dword ptr [ebp-28]

:164703A1 83C8FF or eax, FFFFFFFF

:164703A4 660345E8 add ax, word ptr [ebp-18]

:164703A8 0F80AF0D0000 jo 1647115D

:164703AE 8945E8 mov dword ptr [ebp-18], eax

:164703B1 E997FDFFFF jmp 1647014D

====>循环！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:16470155(C)

:164703B6 B801000000 mov eax, 00000001

:164703BB 6603C3 add ax, bx

:164703BE 0F80990D0000 jo 1647115D

:164703C4 8BD8 mov ebx, eax

:164703C6 E91EFDFFFF jmp 164700E9

====>大循环！

…… 省略 ……

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:16470465(U)

:16470470 8B490C mov ecx, dword ptr [ecx+0C]



`mov edx, dword ptr [ebp+FFFFFE88]` `:16470473 8B9588FEFFFF`  
`mov edx, dword ptr [ecx+edx]` `:16470479 8B1411`  
`mov eax, dword ptr [ecx+eax]` `:1647047C 8B0401`  
`push edx` `:1647047F 52`  
`push eax` `:16470480 50`  
`call 1646F840` `:16470481 E8BAF3FFFF`  

1、 ====> 取 `37315236` 和 `00000000` 异或  
2、 ====> 取 `33343880` 和 `00000000` 异或  
3、 ====> 取 `00000150` 和 `00000000` 异或  
4、 ====> 取 `8248AEA8` 和 `00000000` 异或  
5、 ====> 取 `648A9932` 和 `00000000` 异或  
…… 省略 ……  

`mov edx, eax` `:16470486 8BD0`  

1、 ====> `EDX=37315236`  
2、 ====> `EDX=33343880`  
3、 ====> `EDX=00000150`  
…… 省略 ……  

…… 省略 ……  

`call 1646F840` `:164704E7 E854F3FFFF`  

1、 ====> 取 `37315236` 和第 9 组 `472D3252` 异或  
2、 ====> 取 `33343880` 和第 4 组 `3444472D` 异或  
3、 ====> 取 `00000150` 和第 5 组 `5258474D` 异或  
4、 ====> 取 `8248AEA8` 和第 6 组 `442D4739` 异或  
5、 ====> 取 `648A9932` 和第 7 组 `424D362D` 异或  
…… 省略 ……  

`mov edx, eax` `:164704EC 8BD0`  

1、 ====> `EDX=701C6064`  
2、 ====> `EDX=07707FAD`  
3、 ====> `EDX=5258461D`  
4、 ====> `EDX=C665E991`  
5、 ====> `EDX=26C7AF1F`  
…… 省略 ……  

…… 省略 ……  

`call 1646F840` `:1647054B E8F0F2FFFF`  

1、 ====> 取上面的 `701C6064` 和第 1 组 `31383730` 异或  
2、 ====> 取上面的 `07707FAD` 和第 2 组 `35353334` 异或  
3、 ====> 取上面的 `5258461D` 和第 3 组 `37315236` 异或  
4、 ====> 取上面的 `C665E991` 和第 4 组 `3444472D` 异或  
5、 ====> 取上面的 `26C7AF1F` 和第 5 组 `5258474D` 异或  
…… 省略 ……  

`mov edx, eax` `:16470550 8BD0`  

1、 ====> `EDX=41245754`  
2、 ====> `EDX=32454C99`  
3、 ====> `EDX=6569142B`  
4、 ====> `EDX=F221AEBC`  
5、 ====> `EDX=749FE852`  
…… 省略 ……  

…… 省略 ……  

`call 1646F3B0` `:164705A1 E80AEEFFFF`  

====> 依次将上面所得的结果 乘以 2  

注意：这里面有个进位的处理！进位没有舍去，而是加到积的末位！  

如：`D8CC2756 * 2=1B1984EAC` 但是处理后的结果=`B1984EAD`  

`mov edx, eax` `:164705A6 8BD0`  

1、 ====> `EDX=8248AEA8`  
2、 ====> `EDX=648A9932`  
3、 ====> `EDX=CAD22856`  
4、 ====> `EDX=E4435D79`  
5、 ====> `EDX=E93FD0A4`  
…… 省略 ……  

…… 省略 ……  

`jmp 164703D2` `:164705F2 E9DBFDFFFF`  

====> 循环 80 次！  

…… 省略 ……  

`mov edx, dword ptr [ebp-40]` `:1647060D 8B55C0`  

====> `EDX=98badcfe`  

`mov eax, dword ptr [ebp-38]` `:16470610 8B45C8`  

====> `EAX=efcdab89`  

`push edx` `:16470613 52`  
`push eax` `:16470614 50`  
`call 1646EAA0` `:16470615 E886E4FFFF`  

====> 对上面 2 个参数运算  

`mov edx, eax` `:1647061A 8BD0`  

====> `EDX=88888888`  

`lea ecx, dword ptr [ebp-60]` `:1647061C 8D4DA0`  
`call esi` `:1647061F FFD6`  
`mov ecx, dword ptr [ebp-38]` `:16470621 8B4DC8`  
`push ecx` `:16470624 51`  
`call 1646EE70` `:16470625 E846E8FFFF`  

====> 对 `efcdab89` 运算  

`mov edx, eax` `:1647062A 8BD0`  

====> `EDX=10325476` 这个得数可以看作是固定值！  

`lea ecx, dword ptr [ebp-64]` `:1647062C 8D4D9C`  
`call esi` `:1647062F FFD6`  
`mov edx, dword ptr [ebp-64]` `:16470631 8B559C`  
`mov ebx, dword ptr [ebp-60]` `:16470634 8B5DA0`  
`xor eax, eax` `:16470637 33C0`  
`lea ecx, dword ptr [ebp-54]` `:16470639 8D4DAC`  
`mov dword ptr [ebp-64], eax` `:1647063C 89459C`  
`mov dword ptr [ebp-60], eax` `:1647063F 8945A0`  
`mov eax, dword ptr [ebp-44]` `:16470642 8B45BC`  
`push eax` `:16470645 50`  
`call esi` `:16470646 FFD6`  
`push eax` `:16470648 50`  
`call 1646EAA0` `:16470649 E852E4FFFF`  
`mov edx, eax` `:1647064E 8BD0`  

====> `EDX=10325476`  

`lea ecx, dword ptr [ebp-5C]` `:16470650 8D4DA4`  
`call esi` `:16470653 FFD6`  
`push eax` `:16470655 50`  
`mov edx, ebx` `:16470656 8BD3`  
`lea ecx, dword ptr [ebp-58]` `:16470658 8D4DA8`  
`call esi` `:1647065B FFD6`  
`push eax` `:1647065D 50`  

====> `EDX=88888888`  

`call 1646F080` `:1647065E E81DEAFFFF`  

====> `10325476 + 88888888=98BADCFE`  

`mov edx, eax` `:16470663 8BD0`  

====> `EDX=98BADCFE`  

`lea ecx, dword ptr [ebp-3C]` `:16470665 8D4DC4`  
`call esi` `:16470668 FFD6`  
`lea ecx, dword ptr [ebp-64]` `:1647066A 8D4D9C`  
`lea edx, dword ptr [ebp-60]` `:1647066D 8D55A0`  
`push ecx` `:16470670 51`  
`lea eax, dword ptr [ebp-5C]` `:16470671 8D45A4`  
`push edx` `:16470674 52`  
`lea ecx, dword ptr [ebp-58]` `:16470675 8D4DA8`  
`push eax` `:16470678 50`  
`lea edx, dword ptr [ebp-54]` `:16470679 8D55AC`  
`push ecx` `:1647067C 51`  
`push edx` `:1647067D 52`  
`push 00000005` `:1647067E 6A05`  

* Reference To: `MSVBVM50.__vbaFreeStrList`, Ord:0000h  

`Call dword ptr [1647634C]` `:16470680 FF154C634716`  
`mov eax, dword ptr [ebp-30]` `:16470686 8B45D0`  

====> `EAX=67452301`  

`add esp, 00000018` `:16470689 83C418`  
`push 00000005` `:1647068C 6A05`  
`push eax` `:1647068E 50`  
`call 1646F3B0` `:1647068F E81CEDFFFF`  
`mov edx, eax` `:16470694 8BD0`  

====> `EAX=E8A4602C`  

`lea ecx, dword ptr [ebp-64]` `:16470696 8D4D9C`  
`call esi` `:16470699 FFD6`  
`mov ecx, dword ptr [ebp-4C]` `:1647069B 8B4DB4`  
`test ecx, ecx` `:1647069E 85C9`  
`je 164706CA` `:164706A0 7428`  
`cmp word ptr [ecx], 0001` `:164706A2 66833901`  
`jne 164706CA` `:164706A6 7522`  
`movsx ebx, word ptr [ebp-28]` `:164706A8 0FBF5DD8`  
`mov edx, dword ptr [ecx+14]` `:164706AC 8B5114`  
`mov eax, dword ptr [ecx+10]` `:164706AF 8B4110`  
`sub ebx, edx` `:164706B2 2BDA`  
`cmp ebx, eax` `:164706B4 3BD8`  
`jb 164706C1` `:164706B6 7209`  



* Reference To: `MSVBVM50.__vbaGenerateBoundsError`, Ord: `0000h`

              |

:164706B8 `FF15B8624716`         `Call dword ptr [164762B8]`

:164706BE `8B4DB4`               `mov ecx, dword ptr [ebp-4C]`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:164706B6(C)
|
:164706C1 `8D049D00000000`       `lea eax, dword ptr [4*ebx+00000000]`

:164706C8 `EB09`                 `jmp 164706D3`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:164706A0(C), :164706A6(C)
|
* Reference To: `MSVBVM50.__vbaGenerateBoundsError`, Ord: `0000h`

              |
:164706CA `FF15B8624716`         `Call dword ptr [164762B8]`

:164706D0 `8B4DB4`               `mov ecx, dword ptr [ebp-4C]`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:164706C8(U)
|
:164706D3 `8B5D9C`               `mov ebx, dword ptr [ebp-64]`

:164706D6 `C7459C00000000`       `mov [ebp-64], 00000000`

:164706DD `8B490C`               `mov ecx, dword ptr [ecx+0C]`

* Possible StringData Ref from Code Obj -> `"5A827999"`

              |
:164706E0 `68C8514616`           `push 164651C8`

:164706E5 `8B1401`               `mov edx, dword ptr [ecx+eax]`

              ==== > `EDX=31383730`

:164706E8 `52`                   `push edx`

:164706E9 `E892E6FFFF`           `call 1646ED80`

              ==== > `31383730 + 5A827999=8BBAB0C9`

:164706EE `8BD0`                 `mov edx, eax`

              ==== > `EDX=8BBAB0C9`

:164706F0 `8D4DAC`               `lea ecx, dword ptr [ebp-54]`

:164706F3 `FFD6`                 `call esi`

:164706F5 `50`                   `push eax`

:164706F6 `8B45B8`               `mov eax, dword ptr [ebp-48]`

              ==== > `EDX=c3d2e1f0`

:164706F9 `50`                   `push eax`

:164706FA `E881E6FFFF`           `call 1646ED80`

              ==== > `8BBAB0C9 + c3d2e1f0=4F8D92B9`

:164706FF `8BD0`                 `mov edx, eax`

              ==== > `EDX=4F8D92B9`

:16470701 `8D4DA8`               `lea ecx, dword ptr [ebp-58]`

:16470704 `FFD6`                 `call esi`

:16470706 `8B4DC4`               `mov ecx, dword ptr [ebp-3C]`

              ==== > `ECX=98BADCFE`

:16470709 `50`                   `push eax`

:1647070A `51`                   `push ecx`

:1647070B `E870E6FFFF`           `call 1646ED80`

              ==== > `4F8D92B9 + 98BADCFE=E8486FB7`

:16470710 `8BD0`                 `mov edx, eax`

              ==== > `EDX=E8486FB7`

:16470712 `8D4DA0`               `lea ecx, dword ptr [ebp-60]`

:16470715 `FFD6`                 `call esi`

:16470717 `50`                   `push eax`

:16470718 `8BD3`                 `mov edx, ebx`

              ==== > `EDX=E8A4602C`

:1647071A `8D4DA4`               `lea ecx, dword ptr [ebp-5C]`

:1647071D `FFD6`                 `call esi`

:1647071F `50`                   `push eax`

:16470720 `E85BE6FFFF`           `call 1646ED80`

              ==== > `E8486FB7 + E8A4602C=D0ECCFE3`

:16470725 `8BD0`                 `mov edx, eax`

              ==== > `EBX=D0ECCFE3`

:16470727 `8D4DC4`               `lea ecx, dword ptr [ebp-3C]`

:1647072A `FFD6`                 `call esi`

:1647072C `8D559C`               `lea edx, dword ptr [ebp-64]`

:1647072F `8D45A0`               `lea eax, dword ptr [ebp-60]`

:16470732 `52`                   `push edx`

:16470733 `8D4DA4`               `lea ecx, dword ptr [ebp-5C]`

:16470736 `50`                   `push eax`

:16470737 `8D55A8`               `lea edx, dword ptr [ebp-58]`

:1647073A `51`                   `push ecx`

:1647073B `8D45AC`               `lea eax, dword ptr [ebp-54]`

:1647073E `52`                   `push edx`

:1647073F `50`                   `push eax`

:16470740 `6A05`                 `push 00000005`

* Reference To: `MSVBVM50.__vbaFreeStrList`, Ord: `0000h`

              |
:16470742 `FF154C634716`         `Call dword ptr [1647634C]`

:16470748 `8B55BC`               `mov edx, dword ptr [ebp-44]`

:1647074B `83C418`               `add esp, 00000018`

:1647074E `8D4DB8`               `lea ecx, dword ptr [ebp-48]`

:16470751 `FFD7`                 `call edi`

:16470753 `8B55C0`               `mov edx, dword ptr [ebp-40]`

:16470756 `8D4DBC`               `lea ecx, dword ptr [ebp-44]`

:16470759 `FFD7`                 `call edi`

:1647075B `8B4DC8`               `mov ecx, dword ptr [ebp-38]`

:1647075E `6A1E`                 `push 0000001E`

:16470760 `51`                   `push ecx`

:16470761 `E84AECFFFF`           `call 1646F3B0`

:16470766 `8BD0`                 `mov edx, eax`

:16470768 `8D4DC0`               `lea ecx, dword ptr [ebp-40]`

:1647076B `FFD6`                 `call esi`

:1647076D `8B55D0`               `mov edx, dword ptr [ebp-30]`

:16470770 `8D4DC8`               `lea ecx, dword ptr [ebp-38]`

:16470773 `FFD7`                 `call edi`

:16470775 `8B55C4`               `mov edx, dword ptr [ebp-3C]`

:16470778 `8D4DD0`               `lea ecx, dword ptr [ebp-30]`

:1647077B `FFD7`                 `call edi`

:1647077D `B801000000`           `mov eax, 00000001`

:16470782 `660345D8`             `add ax, word ptr [ebp-28]`

:16470786 `0F80D1090000`         `jo 1647115D`

:1647078C `8945D8`               `mov dword ptr [ebp-28], eax`

:1647078F `E96AFEFFFF`           `jmp 164705FE`

              ==== > 晕倒 再循环 20 次

…… ……省略…… …… 实在是太多的循环了！…… 晕 ……

第一次加密算法 `CALL` 最后得出：`566C71F6575BD4B395FE34B037F4E04B4DD6D78B`

第二次加密算法 `CALL` 最后得出：`F968F0BBA09FA0E740C482592F641FBD33BAC0FF`

—————————————————————————————————

程序启动注册窗口时比较是否已经注册

* Reference To: `MSVBVM60.__vbaLateIdCallLd`, Ord: `0000h`

              |
:0042EC8A `FF1520114000`         `Call dword ptr [00401120]`

:0042EC90 `83C410`               `add esp, 00000010`

:0042EC93 `50`                   `push eax`

* Reference To: `MSVBVM60.__vbaBoolVar`, Ord: `0000h`

              |
:0042EC94 `FF15BC104000`         `Call dword ptr [004010BC]`

:0042EC9A `8D4DCC`               `lea ecx, dword ptr [ebp-34]`

:0042EC9D `668945A0`             `mov word ptr [ebp-60], ax`

* Reference To: `MSVBVM60.__vbaFreeObj`, Ord: `0000h`

              |
:0042ECA1 `FF1544124000`         `Call dword ptr [00401244]`

:0042ECA7 `8D4DB0`               `lea ecx, dword ptr [ebp-50]`

* Reference To: `MSVBVM60.__vbaFreeVar`, Ord: `0000h`

              |
:0042ECAA `FF1528104000`         `Call dword ptr [00401028]`

:0042ECB0 `66837DA000`           `cmp word ptr [ebp-60], 0000`

:0042ECB5 `8B0E`                 `mov ecx, dword ptr [esi]`

:0042ECB7 `56`                   `push esi`

:0042ECB8 `7433`                 `je 0042ECED`

              ==== > 爆破点！

:0042ECBA `FF91FC020000`         `call dword ptr [ecx+000002FC]`

:0042ECC0 `8D55CC`               `lea edx, dword ptr [ebp-34]`

:0042ECC3 `50`                   `push eax`

:0042ECC4 `52`                   `push edx`

:0042ECC5 `FFD3`                 `call ebx`

:0042ECC7 `8BF0`                 `mov esi, eax`

:0042ECC9 `6A00`                 `push 00000000`

:0042ECCB `56`                   `push esi`

:0042ECCC `8B06`                 `mov eax, dword ptr [esi]`

:0042ECCE `FF909C000000`         `call dword ptr [eax+0000009C]`

:0042ECD4 `85C0`                 `test eax, eax`

:0042ECD6 `DBE2`                 `fclex`

:0042ECD8 `0F8DF4010000`         `jnl 0042EED2`

:0042ECDE `689C000000`           `push 0000009C`



```
:0042ECE3 68B4004100                 push 004100B4  

:0042ECE8 E9DD010000                 jmp 0042EECA 
```

【完美爆破】：

```
0042ECB8 7433                        je 0042ECED  
```

改为：

```
9090                                 NOP 掉   
```

【KeyMake 之`{92th}`内存注册机】：

呵呵，这个东东的内存注册机比较有趣，有兴趣的朋友自己想想吧。`^O^` `^O^`

【注册信息保存】：

```
REGEDIT4 

[HKEY_LOCAL_MACHINE\Software\Nelco\166C1B\6E3F5B0C41267C4B530D]  

"7D3D4101582E7B4149"="1E641855027603100757"  

"663246115E225C60511231"="1F6F02531A7B1015065C6568156D1E"  

"63354D004522444D5F081F3956"="6965195D7173726671566D1A6E6C6A76" 
```

【整理】：

机器号：`566C71F6575BD4B3`  

注册码：`F968F0BBA09FA0E7`

```
               ,     \_/   
            /| \_.-~/           \\\     ,       青春都一饷  
            ( /~   /             \\~-.\_ |\   
            `\\\  \_/             \\    ~\ )    忍把浮名   
             _-~~~-.)  )\__/;;,.        \\\  //'  
             /'\_,\   --~   \ ~~~-  ,;;\_\_\_(  (.-~~~-.    换了破解轻狂  
             \`~ \_( ,\_..--\ (     ,;'' /    ~--   /.\_\`\\ 
             /~~//'    /' \`~\     ) /--..\_, )\_  \`~  
             "  \`~"  "     \`"     /~'\`\\    \`\\\\~~\   
                            
                             "     "   "~'  ""  
                             Cracked By 巢水工作坊——fly [OCN][FCG]  
                                      2003-06-09  17:22

4dd466d0e1e0cc123afbe504da2462ca
```
