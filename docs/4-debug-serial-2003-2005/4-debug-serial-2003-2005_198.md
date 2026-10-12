
# Modem Spy V3.2 + Build 2002.11.10

> 原文：[`www.pediy.com/kssd/pediy05/pediy50491.htm`](https://www.pediy.com/kssd/pediy05/pediy50491.htm)

## 简单算法

- 软件版本：`Modem Spy V3.2 + Build 2002.11.10`
- 下载页面：[`www.skycn.com/soft/5631.html`](http://www.skycn.com/soft/5631.html)
- 软件大小：`244 KB`
- 软件语言：英文
- 软件类别：国外软件 / 共享版 / 网络电话
- 应用平台：`Win9x/2000/XP`
- 加入时间：`2002-11-12 09:45:13`
- 下载次数：`4745`
- 推荐等级：\*\*\*\*
- 开发商：[`www.modemspy.com/`](http://www.modemspy.com/)

### 软件简介

`Modem Spy` 可以对网络电话进行谈话录音、纪录所有来电资料、软件内置自动应答功能、可以检测显示来电者的电脑 ID，录音的声音文件可存成 `MP3` 或 `WAV` 文件。

### 软件限制

`30 days`

### 作者声明

初学 `Crack`，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

### 破解工具

`TRW2000` 娃娃修改版、`Ollydbg1.09`、`FI2.5`、`W32Dasm8.93` 黄金版

---

### 过程

从 天空 看见一个小小的 E 文软件，拉下来看看，呵呵，很简单。

`modemspy.exe` 无壳，`VC++6.0` 编写。我等菜鸟喜欢的类型。^O^^O^

反汇编，查找关键提示，发现核心。

Your Name：`fly`

试炼码：`13572468`

---

```assembly
\* Referenced by a CALL at Address:

|:0041E40D

|

:0041E460 56                        push esi

\* Possible Ref to Menu: MenuID_0017, Item: "Delete"

                              |

:0041E461 6880000000              push 00000080
:0041E466 6804A44300              push 0043A404
:0041E46B 8BF1                    mov esi, ecx
:0041E46D 6A53                    push 00000053
:0041E46F E8BC72FEFF              call 00405730

\* Possible Ref to Menu: MenuID_0017, Item: "Delete"

                              |

:0041E474 6880000000              push 00000080
:0041E479 6884A44300              push 0043A484
:0041E47E 6A55                    push 00000055
:0041E480 8BCE                    mov ecx, esi
:0041E482 E8A972FEFF              call 00405730
:0041E487 E814040000              call 0041E8A0
                              ==== >关键 CALL！进入！
:0041E48C 85C0                    test eax, eax
:0041E48E 7408                    je 0041E498
                              ==== >跳则 OVER！
:0041E490 6A40                    push 00000040

\* Possible Reference to String Resource ID=00059: "Thanks"

                              |

:0041E492 6A3B                    push 0000003B

\* Possible Reference to String Resource ID=00057: "Thank you for your support!"

                              ==== >呵呵，胜利女神！
:0041E494 6A39                    push 00000039
:0041E496 EB14                    jmp 0041E4AC

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041E48E(C)

|

\* Possible Reference to String Resource ID=00016: "Dialtone detected"

                              |

:0041E498 6A10                    push 00000010

\* Possible Reference to String Resource ID=00045: "Error"

                              |

:0041E49A 6A2D                    push 0000002D
:0041E49C C60584A4430000          mov byte ptr [0043A484], 00
:0041E4A3 C60504A4430000          mov byte ptr [0043A404], 00

\* Possible Reference to String Resource ID=00062: "Wrong key or name!
Please contact the author"

                              ==== >BAD BOY！
:0041E4AA 6A3E                    push 0000003E
```

进入关键 CALL：`41E487 call 0041E8A0`

```assembly
\* Referenced by a CALL at Addresses:

|:004165EA    , :00416C78    , :0041E487    , :0041E9D0    , :0041FD83

|

:0041E8A0 A084A44300              mov al, byte ptr [0043A484]
:0041E8A5 53                      push ebx
:0041E8A6 56                      push esi
:0041E8A7 3C6D                    cmp al, 6D
:0041E8A9 57                      push edi
:0041E8AA BB01000000              mov ebx, 00000001
:0041E8AF 7479                    je 0041E92A
:0041E8B1 3C4D                    cmp al, 4D
:0041E8B3 7475                    je 0041E92A
:0041E8B5 6884A44300              push 0043A484
:0041E8BA E853310000              call 00421A12
                              ==== >把试炼码转换成 16 进制值
:0041E8BF 8BF8                    mov edi, eax
                              ==== >EDI=EAX=00CF1974（H）=13572468（D）
:0041E8C1 83C404                  add esp, 00000004
:0041E8C4 85FF                    test edi, edi
:0041E8C6 7429                    je 0041E8F1

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041E977(C)

|

\* Possible Reference to Dialog:

                              |

:0041E8C8 BE18134300              mov esi, 00431318

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041E8EF(C)

|

:0041E8CD 8B4E04                  mov ecx, dword ptr [esi+04]
:0041E8D0 6804A44300              push 0043A404
:0041E8D5 51                      push ecx
:0041E8D6 E825010000              call 0041EA00
:0041E8DB 83C408                  add esp, 00000008
:0041E8DE 85C0                    test eax, eax
:0041E8E0 7404                    je 0041E8E6
:0041E8E2 393E                    cmp dword ptr [esi], edi
:0041E8E4 7422                    je 0041E908

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041E8E0(C)

|

:0041E8E6 83C608                  add esi, 00000008
:0041E8E9 81FE60144300            cmp esi, 00431460
:0041E8EF 72DC                    jb 0041E8CD

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041E8C6(C)

|

:0041E8F1 6884A44300              push 0043A484
:0041E8F6 6804A44300              push 0043A404
:0041E8FB E810FCFFFF              call 0041E510
                              ==== >关键 CALL！进入！
:0041E900 83C408                  add esp, 00000008
:0041E903 83F801                  cmp eax, 00000001
:0041E906 7D77                    jge 0041E97F
                              ==== >不跳则 OVER！

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0041E8E4(C), :0041E93A(C), :0041E97D(U)

|

:0041E908 33D2                    xor edx, edx

\* Possible Reference to String Resource ID=00032: "Another program accepted the call"

                              |

:0041E90A B920000000              mov ecx, 00000020
:0041E90F 33C0                    xor eax, eax
:0041E911 BF84A44300              mov edi, 0043A484
:0041E916 F3                      repz
:0041E917 AB                      stosd

\* Possible Reference to String Resource ID=00032: "Another program accepted the call"

                              |

:0041E918 B920000000              mov ecx, 00000020
:0041E91D BF04A44300              mov edi, 0043A404
:0041E922 F3                      repz
:0041E923 AB                      stosd
:0041E924 5F                      pop edi
:0041E925 5E                      pop esi
:0041E926 8BC2                    mov eax, edx
:0041E928 5B                      pop ebx
:0041E929 C3                      ret
```

---

进入关键 CALL：`41E8FB call 0041E510`

```assembly
\* Referenced by a CALL at Address:

|:0041E8FB

|

:0041E510 53                      push ebx
:0041E511 8B5C240C                mov ebx, dword ptr [esp+0C]
:0041E515 56                      push esi
:0041E516 57                      push edi

\* Reference To: KERNEL32.lstrlenA, Ord:0308h

                              |

:0041E517 8B3DC8914200            mov edi, dword ptr [004291C8]
:0041E51D 53                      push ebx
:0041E51E FFD7                    call edi
:0041E520 83F803                  cmp eax, 00000003
:0041E523 0F8C07010000            jl 0041E630
:0041E529 8B742410                mov esi, dword ptr [esp+10]
:0041E52D 56                      push esi
:0041E52E FFD7                    call edi
:0041E530 83F803                  cmp eax, 00000003
:0041E533 0F8CF7000000            jl 0041E630
:0041E539 8A03                    mov al, byte ptr [ebx]
:0041E53B 3C4D                    cmp al, 4D
:0041E53D 0F848E000000            je 0041E5D1
```



```
:0041E543 3C6D                  cmp al, 6D  

:0041E545 0F8486000000          je 0041E5D1  

:0041E54B 53                    push ebx  

:0041E54C FFD7                  call edi  

:0041E54E 8D4418FF              lea eax, dword ptr [eax+ebx-01]  

:0041E552 3BC3                  cmp eax, ebx  

:0041E554 760A                  jbe 0041E560
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

`|:0041E55E(C)`  

`|`  

```
:0041E556 80382D                cmp byte ptr [eax], 2D  

:0041E559 7405                  je 0041E560  

:0041E55B 48                    dec eax  

:0041E55C 3BC3                  cmp eax, ebx  

:0041E55E 77F6                  ja 0041E556
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

`|:0041E554(C), :0041E559(C)`  

`|`  

```
:0041E560 8A06                  mov al, byte ptr [esi]  

:0041E562 33FF                  xor edi, edi  

:0041E564 84C0                  test al, al  

:0041E566 8BCE                  mov ecx, esi  

:0041E568 7439                  je 0041E5A3
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

`|:0041E59B(C)`  

`|`  

```
:0041E56A 3C20                  cmp al, 20  

:0041E56C 7427                  je 0041E595  

:0041E56E 3C0D                  cmp al, 0D  

:0041E570 7423                  je 0041E595  

:0041E572 3C0A                  cmp al, 0A  

:0041E574 741F                  je 0041E595  

:0041E576 3C61                  cmp al, 61  

:0041E578 7C0C                  jl 0041E586  

:0041E57A 3C7A                  cmp al, 7A  

:0041E57C 7F08                  jg 0041E586  

:0041E57E 0FBEC0                movsx eax, al  

                ==== > 依次取 `fly` 字符的 `HEX` 值  

                1、    ====> `EAX = 66`  

                2、    ====> `EAX = 6C`  

                3、    ====> `EAX = 79`
```

```
:0041E581 83E820                sub eax, 00000020  

                1、    ====> `EAX = 66 - 20 = 46`  

                2、    ====> `EAX = 6C - 20 = 4C`  

                3、    ====> `EAX = 79 - 20 = 59`
```

```
:0041E584 EB03                  jmp 0041E589
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

`|:0041E578(C), :0041E57C(C)`  

`|`  

```
:0041E586 0FBEC0                movsx eax, al
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

`|:0041E584(U)`  

`|`  

```
:0041E589 8D1440                lea edx, dword ptr [eax+2*eax]  

                1、    ====> `EDX = 46 * 3 = D2`  

                2、    ====> `ECX = 4C * 3 = E4`  

                3、    ====> `ECX = 59 * 3 = 10B`
```

```
:0041E58C C1E203                shl edx, 03  

                1、    ====> `EDX = 66 << 3 = 690`  

                2、    ====> `ECX = E4 << 3 = 720`  

                3、    ====> `ECX = 10B << 3 = 858`
```

```
:0041E58F 2BD0                  sub edx, eax  

                1、    ====> `EDX = 690 - 46 = 64A`  

                2、    ====> `EDX = 720 - 4C = 6D4`  

                3、    ====> `EDX = 858 - 59 = 7FF`
```

```
:0041E591 8D7C1713              lea edi, dword ptr [edi+edx+13]  

                1、    ====> `EDI = 0 + 64A + 13 = 65D`  

                2、    ====> `EDI = 65D + 6D4 + 13 = D44`  

                3、    ====> `EDI = D44 + 7FF + 13 = 1556`
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

`|:0041E56C(C), :0041E570(C), :0041E574(C)`  

`|`  

```
:0041E595 8A4101                mov al, byte ptr [ecx+01]  

                ==== > 依次取 `fly` 字符
```

```
:0041E598 41                    inc ecx  

:0041E599 84C0                  test al, al  

:0041E59B 75CD                  jne 0041E56A  

:0041E59D 85FF                  test edi, edi  

                ==== > `EDI = 1556`
```

```
:0041E59F 7D02                  jge 0041E5A3  

:0041E5A1 F7DF                  neg edi
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

`|:0041E568(C), :0041E59F(C)`  

`|`  

```
:0041E5A3 8A03                  mov al, byte ptr [ebx]  

:0041E5A5 8BCB                  mov ecx, ebx  

:0041E5A7 84C0                  test al, al  

:0041E5A9 7410                  je 0041E5BB
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

`|:0041E5B9(C)`  

`|`  

```
:0041E5AB 3C30                  cmp al, 30  

:0041E5AD 7C04                  jl 0041E5B3  

:0041E5AF 3C39                  cmp al, 39  

:0041E5B1 7E08                  jle 0041E5BB
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

`|:0041E5AD(C)`  

`|`  

```
:0041E5B3 8A4101                mov al, byte ptr [ecx+01]  

:0041E5B6 41                    inc ecx  

:0041E5B7 84C0                  test al, al  

:0041E5B9 75F0                  jne 0041E5AB
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

`|:0041E5A9(C), :0041E5B1(C)`  

`|`  

```
:0041E5BB 51                    push ecx
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

`|:0041E62E(U)`  

`|`  

```
:0041E5BC E851340000            call 00421A12  

:0041E5C1 83C404                add esp, 00000004  

:0041E5C4 33C9                  xor ecx, ecx  

:0041E5C6 3BC7                  cmp eax, edi  

                ==== > 比较注册码！  

                ==== > `EAX = 00CF1974`（H）` = 13572468`（D）  

                ==== > `EDI = 00001556`（H）` = 5462`（D）  

                呵呵，明码比较。把 `1556`（H）转换成 `10` 进制值 `5462` 就是注册码了！
```

```
:0041E5C8 0F94C1                sete cl  

                ==== > 设标志位
```

```
:0041E5CB 5F                    pop edi  

:0041E5CC 5E                    pop esi  

:0041E5CD 8BC1                  mov eax, ecx  

:0041E5CF 5B                    pop ebx  

:0041E5D0 C3                    ret
```

---

## KeyMake 之内存注册机

- 中断地址：`41E5C6`
- 中断次数：`1`
- 第一字节：`3B`
- 指令长度：`2`
- 寄存器方式：`EDI`
- 十进制

---

## 注册信息保存

同文件夹下的`modemspy.ini`中。

`rkey=5462`

`rname=fly`

---

## 整 理

`Your Name`：`fly`

`UnLock Code`：`5462`

---

Cracked By 巢水工作坊——`fly`【`OCN`】

`2003-03-15 14:13:21  4dd466d0e1e0cc123afbe504da2462ca`
