
# getpassword

> 原文：[`www.pediy.com/kssd/pediy06/pediy6184.htm`](https://www.pediy.com/kssd/pediy06/pediy6184.htm)

软件大小： 111 KB

软件语言： 简体中文

软件类别： 国产软件 / 共享版 /

应用平台： `Win9x/NT/2000/XP`

界面预览： 无

在线注册： 点击这里成为正版用户==>

【作者声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：`TRW2000`

```
* Possible Reference to Dialog: DialogID_0065, CONTROL_ID:03E9, ""
                                                                            
:004061C1 68E9030000                  push 000003E9 

* Reference To: MFC42.Ordinal:0C19, Ord:0C19h
                                                                            
:004061C6 E8BB230000                  Call 00408586  
:004061CB 8D4DEC                      lea ecx, dword ptr [ebp-14]        //ECX=78787878

* Reference To: MFC42.Ordinal:188A, Ord:188Ah
                                                                            
:004061CE E855210000                  Call 00408328  
:004061D3 8D4DEC                      lea ecx, dword ptr [ebp-14]        //ECX=WEIFENG

* Reference To: MFC42.Ordinal:188B, Ord:188Bh
                                                                            
:004061D6 E847210000                  Call 00408322  
:004061DB 8D4DF0                      lea ecx, dword ptr [ebp-10]

* Reference To: MFC42.Ordinal:188A, Ord:188Ah
                                                                            
:004061DE E845210000                  Call 00408328  
:004061E3 8D4DF0                      lea ecx, dword ptr [ebp-10]

* Reference To: MFC42.Ordinal:188B, Ord:188Bh
                                                                            
:004061E6 E837210000                  Call 00408322  
:004061EB 8B45EC                      mov eax, dword ptr [ebp-14]  
:004061EE 3978F8                      cmp dword ptr [eax-08], edi  
:004061F1 0F846A030000                je 00406561  
:004061F7 8B45F0                      mov eax, dword ptr [ebp-10]  
:004061FA 3978F8                      cmp dword ptr [eax-08], edi  
:004061FD 0F845E030000                je 00406561  
:00406203 8D4DEC                      lea ecx, dword ptr [ebp-14] 

* Reference To: MFC42.Ordinal:106A, Ord:106Ah
                                                                            
:00406206 E825220000                  Call 00408430  
:0040620B 8D4DB4                      lea ecx, dword ptr [ebp-4C]

* Reference To: MFC42.Ordinal:021D, Ord:021Dh
                                                                            
:0040620E E86D230000                  Call 00408580  

\====>下面是黑名单比较.要是用黑名单上的就用户就 OVER 了

* Possible StringData Ref from Data Obj ->"guodong"
                                                                            
:00406213 68D0E64000                  push 0040E6D0  
:00406218 8D4DB4                      lea ecx, dword ptr [ebp-4C]  
:0040621B FF75BC                      push [ebp-44]  
:0040621E C645FC02                    mov [ebp-04], 02

* Reference To: MFC42.Ordinal:16E5, Ord:16E5h
                                                                            
:00406222 E853230000                  Call 0040857A 

* Possible StringData Ref from Data Obj ->"ttian"
                                                                            
:00406227 68C8E64000                  push 0040E6C8  
:0040622C 8D4DB4                      lea ecx, dword ptr [ebp-4C]  
:0040622F FF75BC                      push [ebp-44] 

* Reference To: MFC42.Ordinal:16E5, Ord:16E5h
                                                                            
:00406232 E843230000                  Call 0040857A 

* Possible StringData Ref from Data Obj ->"fpx"
                                                                            
:00406237 68C4E64000                  push 0040E6C4  
:0040623C 8D4DB4                      lea ecx, dword ptr [ebp-4C]  
:0040623F FF75BC                      push [ebp-44] 

* Reference To: MFC42.Ordinal:16E5, Ord:16E5h
                                                                            
:00406242 E833230000                  Call 0040857A 

* Possible StringData Ref from Data Obj ->"fpxfpx"
                                                                            
:00406247 68BCE64000                  push 0040E6BC  
:0040624C 8D4DB4                      lea ecx, dword ptr [ebp-4C]  
:0040624F FF75BC                      push [ebp-44] 

* Reference To: MFC42.Ordinal:16E5, Ord:16E5h
                                                                            
:00406252 E823230000                  Call 0040857A  
:00406257 33F6                        xor esi, esi  
:00406259 397DBC                      cmp dword ptr [ebp-44], edi  
:0040625C 7E3A                        jle 00406298 

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00406296(C)
|
:0040625E 8D45DC                      lea eax, dword ptr [ebp-24]  
:00406261 56                          push esi  
:00406262 50                          push eax  
:00406263 8D4DB4                      lea ecx, dword ptr [ebp-4C]  
:00406266 E89C090000                 call 00406C07  
:0040626B 8D4DDC                      lea ecx, dword ptr [ebp-24]  
:0040626E C645FC03                    mov [ebp-04], 03

* Reference To: MFC42.Ordinal:106A, Ord:106Ah
                                                                            
:00406272 E8B9210000                  Call 00408430  
:00406277 FF75EC                      push [ebp-14]  
:0040627A 8D4DDC                      lea ecx, dword ptr [ebp-24] 

* Reference To: MFC42.Ordinal:0ACC, Ord:0ACCh
                                                                            
:0040627D E89A200000                  Call 0040831C  
:00406282 85C0                        test eax, eax  
:00406284 7D67                        jge 004062ED  
:00406286 8D4DDC                      lea ecx, dword ptr [ebp-24]  
:00406289 C645FC02                    mov [ebp-04], 02

* Reference To: MFC42.Ordinal:0320, Ord:0320h
                                                                            
:0040628D E824200000                  Call 004082B6  
:00406292 46                          inc esi  
:00406293 3B75BC                      cmp esi, dword ptr [ebp-44]  
:00406296 7CC6                        jl 0040625E 

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040625C(C)
|
:00406298 8D45D4                      lea eax, dword ptr [ebp-2C] 

* Possible Reference to String Resource ID=00001: "Option.ini"
                                                                            
:0040629B 6A01                        push 00000001  
:0040629D 50                          push eax  
:0040629E 8D4DF0                      lea ecx, dword ptr [ebp-10] 

* Reference To: MFC42.Ordinal:1021, Ord:1021h
                                                                            
:004062A1 E854210000                  Call 004083FA  
:004062A6 8B00                        mov eax, dword ptr [eax]   

* Reference To: MSVCRT._mbscmp, Ord:0159h
                                                                            
:004062A8 8B35D8A34000                mov esi, dword ptr [0040A3D8] 

* Possible StringData Ref from Data Obj ->"0"       
    
                                                                            
:004062AE BBE0E24000                  mov ebx, 0040E2E0       
    
:004062B3 C645FC04                    mov [ebp-04], 04  
:004062B7 53                          push ebx  
:004062B8 50                          push eax  
:004062B9 FFD6                        call esi       //注册码第一位是否为 0，   
    
:004062BB 59                          pop ecx  
:004062BC 85C0                        test eax, eax  
:004062BE 59                          pop ecx  
:004062BF 744E                        je 0040630F       //为 0 就 OVER  
:004062C1 8D45D0                      lea eax, dword ptr [ebp-30] 

* Possible Reference to String Resource ID=00001: "Option.ini"
                                                                            
:004062C4 6A01                        push 00000001  
:004062C6 50                          push eax  
:004062C7 8D4DF0                      lea ecx, dword ptr [ebp-10] 

* Reference To: MFC42.Ordinal:164E, Ord:164Eh
                                                                            
:004062CA E837210000                  Call 00408406  
:004062CF 8B00                        mov eax, dword ptr [eax]  
:004062D1 53                          push ebx  
:004062D2 50                          push eax  
:004062D3 FFD6                        call esi  
:004062D5 8BD8                        mov ebx, eax  
:004062D7 59                          pop ecx  
:004062D8 F7DB                        neg ebx  
:004062DA 59                          pop ecx  
:004062DB 1ADB                        sbb bl, bl  
:004062DD 8D4DD0                      lea ecx, dword ptr [ebp-30]  
:004062E0 FEC3                        inc bl 

* Reference To: MFC42.Ordinal:0320, Ord:0320h
```



```
:004062E2 E8CF1F0000          Call 004082B6

:004062E7 84DB                test bl, bl

:004062E9 7524                jne 0040630F

:004062EB EB24                jmp 00406311 

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00406284(C)

|

:004062ED 51                  push ecx

:004062EE 8BCC                mov ecx, esp

:004062F0 8965E0              mov dword ptr [ebp-20], esp 

* Possible StringData Ref from Data Obj -> "注册失败！"

                      |                      //黑名单的都到这儿了

:004062F3 68B0E64000          push 0040E6B0 

* Reference To: MFC42.Ordinal:0219, Ord:0219h

                      |

:004062F8 E843200000         Call 00408340

:004062FD E884070000         call 00406A86

:00406302 59                  pop ecx

:00406303 C645FC02            mov [ebp-04], 02

:00406307 8D4DDC              lea ecx, dword ptr [ebp-24]

:0040630A E93F020000          jmp 0040654E 

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004062BF(C), :004062E9(C)

|

:0040630F B301                mov bl, 01 

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004062EB(U)

|

:00406311 8D4DD4              lea ecx, dword ptr [ebp-2C]

:00406314 C645FC02            mov [ebp-04], 02 

* Reference To: MFC42.Ordinal:0320, Ord:0320h

                      |

:00406318 E8991F0000          Call 004082B6

:0040631D 84DB                test bl, bl

:0040631F 741B                je 0040633C

:00406321 51                  push ecx

:00406322 8BCC                mov ecx, esp

:00406324 8965DC              mov dword ptr [ebp-24], esp 

* Possible StringData Ref from Data Obj -> "注册失败！"

                      |

:00406327 68B0E64000          push 0040E6B0 

* Reference To: MFC42.Ordinal:0219, Ord:0219h

                      |

:0040632C E80F200000          Call 00408340

:00406331 E850070000          call 00406A86

:00406336 59                  pop ecx

:00406337 E917020000          jmp 00406553 

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040631F(C)

|

:0040633C 8D4DE4              lea ecx, dword ptr [ebp-1C]

:0040633F 897DD8              mov dword ptr [ebp-28], edi 

* Reference To: MFC42.Ordinal:021C, Ord:021Ch

                      |

:00406342 E88D1F0000          Call 004082D4

:00406347 8D45E4              lea eax, dword ptr [ebp-1C]

:0040634A C645FC05            mov [ebp-04], 05

:0040634E 50                  push eax

:0040634F 51                  push ecx

:00406350 8D45EC              lea eax, dword ptr [ebp-14]

:00406353 8BCC                mov ecx, esp

:00406355 8965C8              mov dword ptr [ebp-38], esp

:00406358 50                  push eax 

* Reference To: MFC42.Ordinal:0217, Ord:0217h

                      |

:00406359 E8B21F0000          Call 00408310

:0040635E E8B9180000          call 00407C1C //有兴趣跟入此 CALL 看看（与 d2VpZmVuZw==有关），我搞不定

:00406363 59                  pop ecx

:00406364 59                  pop ecx

:00406365 8D4DE4              lea ecx, dword ptr [ebp-1C] 

* Reference To: MFC42.Ordinal:188A, Ord:188Ah

                      |

:00406368 E8BB1F0000          Call 00408328                

:0040636D 8D4DE4              lea ecx, dword ptr [ebp-1C]//到这里发现寄存器的值是 d2VpZmVuZw==

* Reference To: MFC42.Ordinal:188B, Ord:188Bh

                      |

:00406370 E8AD1F0000          Call 00408322

:00406375 8B45E4              mov eax, dword ptr [ebp-1C]

:00406378 33C9                xor ecx, ecx

:0040637A 8B50F8              mov edx, dword ptr [eax-08]

:0040637D 3BD7                cmp edx, edi

:0040637F 7E0C                jle 0040638D 

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040638B(C)

|

:00406381 0FBE3401            movsx esi, byte ptr [ecx+eax]//分别取 d2VpZmVuZw==每一位

:00406385 0175D8              add dword ptr [ebp-28], esi  //其 ASCII 累加（得 439h）

:00406388 41                  inc ecx

:00406389 3BCA                cmp ecx, edx

:0040638B 7CF4                jl 00406381 

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040637F(C)

|

:0040638D 8B45F0              mov eax, dword ptr [ebp-10]

:00406390 8D4DF0              lea ecx, dword ptr [ebp-10]

:00406393 8B40F8              mov eax, dword ptr [eax-08]

:00406396 83C0FE              add eax, FFFFFFFE          //EAX=8 + (-2)=6   

:00406399 50                  push eax

:0040639A 8D45D0              lea eax, dword ptr [ebp-30]

:0040639D 57                  push edi

:0040639E 50                  push eax 

* Reference To: MFC42.Ordinal:10B6, Ord:10B6h

                      |

:0040639F E86E200000          Call 00408412

:004063A4 FF30                push dword ptr [eax] 

* Reference To: MSVCRT.atol, Ord:023Eh

                      |

:004063A6 8B35F8A34000        mov esi, dword ptr [0040A3F8]

:004063AC FFD6                call esi            //此 CALL 作用是取前 6 位入 ECX，其 16 进制入 EAX

:004063AE 59                  pop ecx             //ECX=787878

:004063AF 8BD8                mov ebx, eax        //EBX=EAX=C05A6H=787878D（可用？EAX 查看）

:004063B1 8D4DD0              lea ecx, dword ptr [ebp-30] 

* Reference To: MFC42.Ordinal:0320, Ord:0320h

                      |

:004063B4 E8FD1E0000          Call 004082B6

:004063B9 8D45D0              lea eax, dword ptr [ebp-30] 

* Possible Reference to String Resource ID=00002: "wintask.dll"

                      |

:004063BC 6A02                push 00000002

:004063BE 50                  push eax

:004063BF 8D4DF0              lea ecx, dword ptr [ebp-10] 

* Reference To: MFC42.Ordinal:164E, Ord:164Eh

                      |

:004063C2 E83F200000          Call 00408406

:004063C7 FF30                push dword ptr [eax] 

* Reference To: MSVCRT.atoi, Ord:023Dh

                      |

:004063C9 FF15FCA34000        Call dword ptr [0040A3FC]//此 CALL 作用是取后 2 位入 ECX，其 16 进制入 EAX

:004063CF 59                  pop ecx                 //ECX=78       

:004063D0 8BF8                mov edi, eax            //EDI=EAX=4EH=78D

:004063D2 8D4DD0              lea ecx, dword ptr [ebp-30] 

* Reference To: MFC42.Ordinal:0320, Ord:0320h

                      |

:004063D5 E8DC1E0000          Call 004082B6

:004063DA 33DF                xor ebx, edi            //EDX=EDX XOR  EDI=C05A6H XOR 4EH=C05E8H

:004063DC 395DD8              cmp dword ptr [ebp-28], ebx    //439h 与 C05E8H 比较，相等就注册成功

:004063DF 0F854B010000        jne 00406530                  //发现用 TRW 调试时是 JNZ？

:004063E5 8D4DE8              lea ecx, dword ptr [ebp-18] 

* Reference To: MFC42.Ordinal:021C, Ord:021Ch

                      |

:004063E8 E8E71E0000          Call 004082D4 

* Possible Reference to String Resource ID=00001: "Option.ini"

                      |

:004063ED 6A01                push 00000001

:004063EF 8D4DE8              lea ecx, dword ptr [ebp-18]

:004063F2 C645FC06            mov [ebp-04], 06 

* Reference To: MFC42.Ordinal:1040, Ord:1040h

                      |

:004063F6 E8ED1F0000          Call 004083E8
```



:004063FB 8D45D0 `lea eax, dword ptr [ebp-30]`

:004063FE 50 `push eax`

:004063FF E89BBFFFFF `call 0040239F`

:00406404 8D45D0 `lea eax, dword ptr [ebp-30]`

\* Possible StringData Ref from Data Obj -> "\\"

:00406407 C70424A4E24000 `mov dword ptr [esp], 0040E2A4`

:0040640E 50 `push eax`

:0040640F 8D45C8 `lea eax, dword ptr [ebp-38]`

:00406412 50 `push eax`

:00406413 C645FC07 `mov [ebp-04], 07`

\* Reference To: MFC42.Ordinal:039C, Ord:039Ch

:00406417 E8BA1F0000 `Call 004083D6`

:0040641C 8D4DE8 `lea ecx, dword ptr [ebp-18]`

:0040641F C645FC08 `mov [ebp-04], 08`

:00406423 51 `push ecx`

:00406424 50 `push eax`

:00406425 8D45CC `lea eax, dword ptr [ebp-34]`

:00406428 50 `push eax`

\* Reference To: MFC42.Ordinal:039A, Ord:039Ah

:00406429 E8EA1F0000 `Call 00408418`

:0040642E 50 `push eax`

:0040642F 8D4DE8 `lea ecx, dword ptr [ebp-18]`

:00406432 C645FC09 `mov [ebp-04], 09`

\* Reference To: MFC42.Ordinal:035A, Ord:035Ah

:00406436 E80B1F0000 `Call 00408346`

:0040643B 8D4DCC `lea ecx, dword ptr [ebp-34]`

:0040643E C645FC08 `mov [ebp-04], 08`

\* Reference To: MFC42.Ordinal:0320, Ord:0320h

:00406442 E86F1E0000 `Call 004082B6`

:00406447 8D4DC8 `lea ecx, dword ptr [ebp-38]`

:0040644A C645FC07 `mov [ebp-04], 07`

\* Reference To: MFC42.Ordinal:0320, Ord:0320h

:0040644E E8631E0000 `Call 004082B6`

:00406453 8D45F0 `lea eax, dword ptr [ebp-10]`

:00406456 8D4DD8 `lea ecx, dword ptr [ebp-28]`

:00406459 50 `push eax`

\* Reference To: MFC42.Ordinal:0217, Ord:0217h

:0040645A E8B11E0000 `Call 00408310`

:0040645F 6A00 `push 00000000`

:00406461 C645FC0A `mov [ebp-04], 0A`

\* Reference To: MSVCRT.time, Ord:02D0h

:00406465 FF15CCA34000 `Call dword ptr [0040A3CC]`

:0040646B 50 `push eax`

\* Reference To: MSVCRT.srand, Ord:02B4h

:0040646C FF15D0A34000 `Call dword ptr [0040A3D0]`

:00406472 59 `pop ecx`

:00406473 59 `pop ecx`

\* Reference To: MSVCRT.rand, Ord:02A6h

:00406474 FF15D4A34000 `Call dword ptr [0040A3D4]`

:0040647A 8D4DD4 `lea ecx, dword ptr [ebp-2C]`

:0040647D 8BF8 `mov edi, eax`

\* Reference To: MFC42.Ordinal:021C, Ord:021Ch

:0040647F E8501E0000 `Call 004082D4`

:00406484 57 `push edi`

:00406485 8D45D4 `lea eax, dword ptr [ebp-2C]`

\* Possible StringData Ref from Data Obj -> "%d"

:00406488 68ACE64000 `push 0040E6AC`

:0040648D 50 `push eax`

:0040648E C645FC0B `mov [ebp-04], 0B`

\* Reference To: MFC42.Ordinal:0B02, Ord:0B02h

:00406492 E89D1E0000 `Call 00408334`

:00406497 FF75D8 `push [ebp-28]`

:0040649A FFD6 `call esi`

:0040649C 8B4DD4 `mov ecx, dword ptr [ebp-2C]`

:0040649F 33C7 `xor eax, edi`

:004064A1 50 `push eax`

:004064A2 57 `push edi`

:004064A3 8B49F8 `mov ecx, dword ptr [ecx-08]`

:004064A6 8D45D8 `lea eax, dword ptr [ebp-28]`

:004064A9 51 `push ecx`

\* Possible StringData Ref from Data Obj -> "%d%d%d"

:004064AA 68A4E64000 `push 0040E6A4`

:004064AF 50 `push eax`

\* Reference To: MFC42.Ordinal:0B02, Ord:0B02h

:004064B0 E87F1E0000 `Call 00408334`

:004064B5 83C424 `add esp, 00000024`

\* Reference To: KERNEL32.WritePrivateProfileStringA, Ord:02E5h

:004064B8 8B3554A04000 `mov esi, dword ptr [0040A054]`

\* Possible StringData Ref from Data Obj -> "REGINFO"

:004064BE BFC0E24000 `mov edi, 0040E2C0`

:004064C3 FF75E8 `push [ebp-18]`

:004064C6 FF75EC `push [ebp-14]`

\* Possible StringData Ref from Data Obj -> "USERNAME"

:004064C9 68B4E24000 `push 0040E2B4`

:004064CE 57 `push edi`

:004064CF FFD6 `call esi`

:004064D1 FF75E8 `push [ebp-18]`

:004064D4 FF75D8 `push [ebp-28]`

\* Possible StringData Ref from Data Obj -> "PASSWORD"

:004064D7 68A8E24000 `push 0040E2A8`

:004064DC 57 `push edi`

:004064DD FFD6 `call esi`

:004064DF 8B4DE0 `mov ecx, dword ptr [ebp-20]`

:004064E2 6830100000 `push 00001030`

\* Possible StringData Ref from Data Obj -> "注册信息"

:004064E7 6898E64000 `push 0040E698`

\* Possible StringData Ref from Data Obj -> "您成功注册！"

:004064EC 6888E64000 `push 0040E688`

\* Reference To: MFC42.Ordinal:1080, Ord:1080h

:004064F1 E8201E0000 `Call 00408316`

:004064F6 8B4DE0 `mov ecx, dword ptr [ebp-20]`

### 注册码整理

1. 将由用户名 `weifeng` 生成的字符串 `d2VpZmVuZw==` 的每位 ASCII 相加得 `439H`
2. 将假码的后两位 `78` 化成 16 进制 `4E`
3. `439H` XOR `4EH` = `477H` = `1143D` 为前几位数
4. 所以注册码为 `114378`，可以是其它

### 后记

这软件是我刚学破解时在看雪论坛上发贴求助过的，想不到今天找到了它的注册码，其实真的要谢 FLY，他前破过该软件作者的一个软件叫“邮件精灵 V2.0”，可我用他在邮件精灵 V2.0 里找到的注册码注册不成功，所以就有了这篇文章了，呵呵~~原来这软件还在用户名上做手脚。

而算法 `90%` 以上是相同的

`4dd466d0e1e0cc123afbe504da2462ca`
