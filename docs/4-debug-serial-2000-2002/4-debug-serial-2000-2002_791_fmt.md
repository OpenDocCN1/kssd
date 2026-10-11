
# EVC.SerialMe Beta

> 原文：[`www.pediy.com/kssd/pediy04/kanxue477.htm`](https://www.pediy.com/kssd/pediy04/kanxue477.htm)

```text
/////////////////////////////////////////////////////////////////////    
//   
//  目标软件：EVC.SerialMe   
//   
//  软件版本：Beta   
//   
//  官方网站：http://www.ebolaviruscrew.net/   
//   
//  软件授权：共享软件   
//   
//  操作系统：Win95/98/ME、WinNT/2000   
//   
//  软件简介：EVC 的考试程序(第 2 级)   
//   
//  软件保护：序列号保护、yoda's cryptor 1.2 加壳   
//   
/////////////////////////////////////////////////////////////////////    
//   
//  使用工具：TRW2000 v1.22 娃娃修改版(Include FPU Plugin)   
//            (主要用于调试分析)   
//   
//            Our Brain...:-)   
//   
/////////////////////////////////////////////////////////////////////    
//   
//  关于本文：本文主要目的在于教学，让初学者掌握一些基本的脱壳方法及   
//   
//            软件分析手段...请勿将此教程用于商业目的。   
//   
//                    Always Your Best Friend: FiNALSErAPH   
//   
//  水平有限，难免疏漏...   
//   
//  Any Question?   
//  Mail To: FiNALSErAPH@yahoo.com.cn   
//   
//                                            2001-12-21   
//   
/////////////////////////////////////////////////////////////////////    
//   
//  这个程序还是加了壳的，不过很简单。这里就不说了。   
//   
//  好像是 yoda's cryptor 1.2(不知道，脱壳后你可以看见图标了)   
//   
//  注册码验证主程序入口   
//   
/////////////////////////////////////////////////////////////////////    
```

* Referenced by a CALL at Address:

`:00401931`

|

```asm
:00401430 81ECF4010000               sub esp, 000001F4
:00401436 33C0                       xor eax, eax
:00401438 53                        push ebx
:00401439 89442408                  mov dword ptr [esp+08], eax
:0040143D 56                        push esi
:0040143E 89442410                  mov dword ptr [esp+10], eax
:00401442 57                        push edi
:00401443 89442418                  mov dword ptr [esp+18], eax
:00401447 33DB                      xor ebx, ebx
:00401449 8944241C                  mov dword ptr [esp+1C], eax
:0040144D B918000000                mov ecx, 00000018
:00401452 8DBC2439010000             lea edi, dword ptr [esp+00000139]
:00401459 89442420                  mov dword ptr [esp+20], eax
:0040145D 889C2438010000            mov byte ptr [esp+00000138], bl
:00401464 889C24D4000000            mov byte ptr [esp+000000D4], bl
:0040146B F3                        repz
:0040146C AB                        stosd
:0040146D 66AB                      stosw
:0040146F AA                        stosb
:00401470 B918000000                mov ecx, 00000018
:00401475 33C0                      xor eax, eax
:00401477 8DBC24D5000000            lea edi, dword ptr [esp+000000D5]
:0040147E 889C249C010000            mov byte ptr [esp+0000019C], bl
:00401485 F3                        repz
:00401486 AB                        stosd
:00401487 66AB                      stosw
:00401489 AA                        stosb
:0040148A B918000000                mov ecx, 00000018
:0040148F 33C0                      xor eax, eax
:00401491 8DBC249D010000            lea edi, dword ptr [esp+0000019D]
:00401498 8B15C49D4000              mov edx, dword ptr [00409DC4]
:0040149E F3                        repz
:0040149F AB                        stosd
:004014A0 33C9                      xor ecx, ecx
:004014A2 53                        push ebx
:004014A3 894C242C                  mov dword ptr [esp+2C], ecx
:004014A7 6A63                      push 00000063
:004014A9 894C2434                  mov dword ptr [esp+34], ecx
:004014AD 68C5000000                push 000000C5
:004014B2 66AB                      stosw
:004014B4 894C243C                  mov dword ptr [esp+3C], ecx
:004014B8 52                        push edx
:004014B9 894C2444                  mov dword ptr [esp+44], ecx
:004014BD 895C241C                  mov dword ptr [esp+1C], ebx
:004014C1 AA                        stosb
```

```text
/////////////////////////////////////////////////////////////////////    
//   
//  这里就是方程组的参数了，36 个。   
//   
//  这里遗留了我的一个问题：如何推算出 BE4CCCCD -> -0.2   
//   
/////////////////////////////////////////////////////////////////////    
```

```asm
:004014C2 C7442454CDCC4CBE         mov dword ptr [esp+00000054], BE4CCCCD
:004014CA C7442458CDCCCCBE         mov dword ptr [esp+00000058], BECCCCCD
:004014D2 C744245C9A99193F         mov dword ptr [esp+0000005C], 3F19999A
:004014DA C7442460CDCCCC3E         mov dword ptr [esp+00000060], 3ECCCCCD
:004014E2 C7442464CDCC4C3E         mov dword ptr [esp+00000064], 3E4CCCCD
:004014EA C7442468CDCCCCBE         mov dword ptr [esp+00000068], BECCCCCD
:004014F2 C744246CCDCCCCBE         mov dword ptr [esp+0000006C], BECCCCCD
:004014FA C7442470CDCC4C3E         mov dword ptr [esp+00000070], 3E4CCCCD
:00401502 C7442474CDCC4C3E         mov dword ptr [esp+00000074], 3E4CCCCD
:0040150A C7442478CDCC4C3F         mov dword ptr [esp+00000078], 3F4CCCCD
:00401512 C744247C9A9919BF         mov dword ptr [esp+0000007C], BF19999A
:0040151A C7842480000000CDCC4C3E   mov dword ptr [esp+00000080], 3E4CCCCD
:00401525 C7842484000000CDCC4C3F   mov dword ptr [esp+00000084], 3F4CCCCD
:00401530 C78424880000009A99193F   mov dword ptr [esp+00000088], 3F19999A
:0040153B C784248C000000CDCCCCBE   mov dword ptr [esp+0000008C], BECCCCCD
:00401546 C78424900000009A9919BF   mov dword ptr [esp+00000090], BF19999A
:00401551 C7842494000000CDCC4C3E   mov dword ptr [esp+00000094], 3E4CCCCD
:0040155C C7842498000000CDCCCCBE   mov dword ptr [esp+00000098], BECCCCCD
:00401567 C784249C000000CDCC4CBE   mov dword ptr [esp+0000009C], BE4CCCCD
:00401572 C78424A0000000CDCCCCBE   mov dword ptr [esp+000000A0], BECCCCCD
:0040157D C78424A40000009A99193F   mov dword ptr [esp+000000A4], 3F19999A
:00401588 C78424A8000000CDCCCC3E   mov dword ptr [esp+000000A8], 3ECCCCCD
:00401593 C78424AC000000CDCC4CBF   mov dword ptr [esp+000000AC], BF4CCCCD
:0040159E C78424B00000009A99193F   mov dword ptr [esp+000000B0], 3F19999A
:004015A9 C78424B4000000CDCCCCBE   mov dword ptr [esp+000000B4], BECCCCCD
:004015B4 C78424B8000000CDCC4C3E   mov dword ptr [esp+000000B8], 3E4CCCCD
:004015BF C78424BC000000CDCC4C3E   mov dword ptr [esp+000000BC], 3E4CCCCD
:004015CA C78424C0000000CDCC4CBE   mov dword ptr [esp+000000C0], BE4CCCCD
:004015D5 C78424C4000000CDCCCC3E   mov dword ptr [esp+000000C4], 3ECCCCCD
:004015E0 C78424C8000000CDCC4C3E   mov dword ptr [esp+000000C8], 3E4CCCCD
:004015EB C78424CC0000009A99193F   mov dword ptr [esp+000000CC], 3F19999A
:004015F6 C78424D0000000CDCC4C3E   mov dword ptr [esp+000000D0], 3E4CCCCD
:00401601 C78424D4000000CDCC4CBF   mov dword ptr [esp+000000D4], BF4CCCCD
:0040160C C78424D8000000CDCC4CBE   mov dword ptr [esp+000000D8], BE4CCCCD
:00401617 C78424DC000000CDCCCC3E   mov dword ptr [esp+000000DC], 3ECCCCCD
:00401622 C78424E0000000CDCC4C3E   mov dword ptr [esp+000000E0], 3E4CCCCD
:0040162D 895C2434                  mov dword ptr [esp+34], ebx
:00401631 894C2448                  mov dword ptr [esp+48], ecx
```

* Reference To: `USER32.SendMessageA`, Ord:`0000h`

```asm
:00401635 FF15F0804000              Call dword ptr [004080F0]
:0040163B 6A64                      push 00000064
:0040163D 8B0DC49D4000              mov ecx, dword ptr [00409DC4]
```

* Reference To: `USER32.GetWindowTextA`, Ord:`0000h`

```asm
:00401643 8B35D0804000              mov esi, dword ptr [004080D0]
:00401649 8D84243C010000            lea eax, dword ptr [esp+0000013C]
:00401650 50                        push eax
:00401651 51                        push ecx
:00401652 FFD6                      call esi
:00401654 A1C09D4000                mov eax, dword ptr [00409DC0]
```



```
:00401659 8D9424D4000000   lea edx, dword ptr [esp+000000D4]
:00401660 6A64             push 00000064
:00401662 52               push edx
:00401663 50               push eax
:00401664 FFD6             call esi
:00401666 8DBC2438010000   lea edi, dword ptr [esp+00000138]
:0040166D 83C9FF           or ecx, FFFFFFFF
:00401670 33C0             xor eax, eax
:00401672 F2               repnz
:00401673 AE               scasb
:00401674 F7D1             not ecx
:00401676 49               dec ecx
:00401677 83F904           cmp ecx, 00000004
:0040167A 7311             jnb 0040168D

                         //输入名字不少于 4 个字符(在此处无用)

:0040167C 6A30             push 00000030
:0040167E 68CC9D4000       push 00409DCC

* Possible StringData Ref from Data Obj ->"The name must contain at least "
                                 ->"4 chars!"
   |

:00401683 68D4914000       push 004091D4
:00401688 E97D010000       jmp 0040180A

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040167A(C)
|

:0040168D 8DBC24D4000000   lea edi, dword ptr [esp+000000D4]
:00401694 83C9FF           or ecx, FFFFFFFF
:00401697 33C0             xor eax, eax
:00401699 F2               repnz
:0040169A AE               scasb
:0040169B F7D1             not ecx
:0040169D 49               dec ecx
:0040169E 83F901           cmp ecx, 00000001
:004016A1 7324             jnb 004016C7

                         //总得输入点什么吧

:004016A3 8B942404020000   mov edx, dword ptr [esp+00000204]
:004016AA 6A30             push 00000030
:004016AC 68CC9D4000       push 00409DCC

* Possible StringData Ref from Data Obj ->"Enter a serial, bunghole!!!"
   |

:004016B1 68B8914000       push 004091B8
:004016B6 52               push edx

* Reference To: USER32.MessageBoxA, Ord:0000h
   |

:004016B7 FF15CC804000     Call dword ptr [004080CC]
:004016BD 5F               pop edi
:004016BE 5E               pop esi
:004016BF 5B               pop ebx
:004016C0 81C4F4010000     add esp, 000001F4
:004016C6 C3               ret

/////////////////////////////////////////////////////////////////////
//
//  注册码验程序的关键部分
//
/////////////////////////////////////////////////////////////////////

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004016A1(C)
|

:004016C7 8D442420         lea eax, dword ptr [esp+20]
:004016CB 8D4C241C         lea ecx, dword ptr [esp+1C]
:004016CF 50               push eax
:004016D0 8D54241C         lea edx, dword ptr [esp+1C]
:004016D4 51               push ecx
:004016D5 8D44241C         lea eax, dword ptr [esp+1C]
:004016D9 52               push edx
:004016DA 8D4C241C         lea ecx, dword ptr [esp+1C]
:004016DE 50               push eax
:004016DF 8D54241C         lea edx, dword ptr [esp+1C]
:004016E3 51               push ecx
:004016E4 52               push edx
:004016E5 8D8424EC000000   lea eax, dword ptr [esp+000000EC]

* Possible StringData Ref from Data Obj ->"%lu-%lu-%lu-%lu-%lu-%lu"
   |

:004016EC 68A0914000       push 004091A0
:004016F1 50               push eax
:004016F2 E82B030000       call 00401A22
:004016F7 83C420           add esp, 00000020
:004016FA 83F806           cmp eax, 00000006
:004016FD 7411             je 00401710

                         //第一处判断，根据上下文可以知道在判
                         //断输入注册码的格式

:004016FF 6A30             push 00000030
:00401701 68CC9D4000       push 00409DCC

* Possible StringData Ref from Data Obj ->"pfff...:(((("
   |

:00401706 6890914000       push 00409190
:0040170B E9FA000000       jmp 0040180A

/////////////////////////////////////////////////////////////////////
//
//  真正的校验运算开始
//
//  实际上，这是一个 6 元一次方程问题
//
//  我这里的 RegNum：223201-420003-2529-33-39178-3074
//  (这个号码与机器有关)
//
//  X1*-0.2 + X2*-0.4 + X3* 0.6 +X4* 0.4 + X5* 0.2 + X6*-0.4 = 223201    - ①
//  X1*-0.4 + X2* 0.2 + X3* 0.2 +X4* 0.8 + X5*-0.6 + X6* 0.2 = 420003    - ②
//  X1* 0.8 + X2* 0.6 + X3*-0.4 +X4*-0.6 + X5* 0.2 + X6*-0.4 = 2529      - ③
//  X1*-0.2 + X2*-0.4 + X3* 0.6 +X4* 0.4 + X5*-0.8 + X6* 0.6 = 33        - ④
//  X1*-0.4 + X2* 0.2 + X3* 0.2 +X4*-0.2 + X5* 0.4 + X6* 0.2 = 39178     - ⑤
//  X1* 0.6 + X2* 0.2 + X3*-0.8 +X4*-0.2 + X5* 0.4 + X6* 0.2 = 3074      - ⑥
//
/////////////////////////////////////////////////////////////////////

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004016FD(C)
|

:00401710 55               push ebp
:00401711 8D442448         lea eax, dword ptr [esp+48]
:00401715 8D742428         lea esi, dword ptr [esp+28]
:00401719 BF06000000       mov edi, 00000006

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040174B(C)
|

:0040171E D906             fld dword ptr [esi]
:00401720 8D4C2410         lea ecx, dword ptr [esp+10]
:00401724 BA06000000       mov edx, 00000006

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401743(C)
|

:00401729 8B29             mov ebp, dword ptr [ecx]
:0040172B 895C2444         mov dword ptr [esp+44], ebx
:0040172F 896C2440         mov dword ptr [esp+40], ebp
:00401733 83C004           add eax, 00000004
:00401736 DF6C2440         fild qword ptr [esp+40]
:0040173A 83C104           add ecx, 00000004
:0040173D 4A               dec edx
:0040173E D848FC           fmul dword ptr [eax-04]
:00401741 DEC1             faddp st(1), st(0)
:00401743 75E4             jne 00401729
:00401745 D91E             fstp dword ptr [esi]
:00401747 83C604           add esi, 00000004
:0040174A 4F               dec edi
:0040174B 75D1             jne 0040171E

/////////////////////////////////////////////////////////////////////

:0040174D D944243C         fld dword ptr [esp+3C]
:00401751 83EC08           sub esp, 00000008
:00401754 8D9424A8010000   lea edx, dword ptr [esp+000001A8]
:0040175B DD1C24           fstp qword ptr [esp]
:0040175E D9442440         fld dword ptr [esp+40]
:00401762 83EC08           sub esp, 00000008
:00401765 DD1C24           fstp qword ptr [esp]
:00401768 D9442444         fld dword ptr [esp+44]
:0040176C 83EC08           sub esp, 00000008
:0040176F DD1C24           fstp qword ptr [esp]
:00401772 D9442448         fld dword ptr [esp+48]
:00401776 83EC08           sub esp, 00000008
:00401779 DD1C24           fstp qword ptr [esp]
:0040177C D944244C         fld dword ptr [esp+4C]
:00401780 83EC08           sub esp, 00000008
:00401783 DD1C24           fstp qword ptr [esp]
:00401786 D9442450         fld dword ptr [esp+50]
:0040178A 83EC08           sub esp, 00000008
```



```
:0040178D DD1C24                   fstp qword ptr [esp]

* Possible StringData Ref from Data Obj ->"%.0f-%.0f-%.0f-%.0f-%.0f-%.0f"

:00401790 6870914000               push 00409170
:00401795 52                       push edx
:00401796 E835020000               call 004019D0

//由输入的注册码进行运算后的结果应该
//等于给出的字符串

:0040179B 83C438                   add esp, 00000038
:0040179E 8DB4243C010000           lea esi, dword ptr [esp+0000013C]
:004017A5 8D8424A0010000           lea eax, dword ptr [esp+000001A0]
:004017AC 5D                       pop ebp

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004017CB(C)
|
:004017AD 8A10                     mov dl, byte ptr [eax]
:004017AF 8ACA                     mov cl, dl
:004017B1 3A16                     cmp dl, byte ptr [esi]
:004017B3 751C                     jne 004017D1
:004017B5 3ACB                     cmp cl, bl
:004017B7 7414                     je 004017CD
:004017B9 8A5001                   mov dl, byte ptr [eax+01]
:004017BC 8ACA                     mov cl, dl
:004017BE 3A5601                   cmp dl, byte ptr [esi+01]
:004017C1 750E                     jne 004017D1
:004017C3 83C002                   add eax, 00000002
:004017C6 83C602                   add esi, 00000002
:004017C9 3ACB                     cmp cl, bl
:004017CB 75E0                     jne 004017AD

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004017B7(C)
|
:004017CD 33C0                     xor eax, eax
:004017CF EB05                     jmp 004017D6

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004017B3(C), :004017C1(C)
|
:004017D1 1BC0                     sbb eax, eax
:004017D3 83D8FF                   sbb eax, FFFFFFFF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004017CF(U)
|
:004017D6 3BC3                     cmp eax, ebx
:004017D8 7524                     jne 004017FE
:004017DA 8B842404020000           mov eax, dword ptr [esp+00000204]
:004017E1 6A40                     push 00000040

* Possible StringData Ref from Data Obj ->"Good Boy!"

:004017E3 6864914000               push 00409164

* Possible StringData Ref from Data Obj ->"You made it!"

:004017E8 6854914000               push 00409154
:004017ED 50                       push eax

* Reference To: USER32.MessageBoxA, Ord:0000h

:004017EE FF15CC804000             Call dword ptr [004080CC]
:004017F4 5F                       pop edi
:004017F5 5E                       pop esi
:004017F6 5B                       pop ebx
:004017F7 81C4F4010000            add esp, 000001F4
:004017FD C3                       ret
```

```
/////////////////////////////////////////////////////////////////////
//
// 我的结果：228838-461710-264942-646278-265453-42285
//
/////////////////////////////////////////////////////////////////////
//
// 呵呵，其实算法不难，就是方程组难解(数学忘光了)
//
// 至于注册机嘛，就是解方程了。不过这个方程解起来可以偷懒。
//
/////////////////////////////////////////////////////////////////////
```

`4dd466d0e1e0cc123afbe504da2462ca`
