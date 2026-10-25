
# 破解Don't Panic! 4.0(我又来了) (5千字)

软件名称：Don't Panic! 4.0

保护方式：注册码

破解人：TAE[FCG] TAE[BCG]

软件简介：一个过滤网上黄色信息的软件！

下载地址:去华军软件园搜索一下吧!呵呵!

破解方法：计算出注册码

*****************************

*首先在这里感谢 小球[CCG]的鼎立相助，呵呵！*

*****************************

运行程序，大家都会吧？呵呵。

找到输入注册码的窗口，什么？找不到？我倒！

输入假注册码111－111－1111

进入trw的调试窗口，下断点`hmemcpy`

回到程序选“OK”立马被拦。

F10，然后到了这里：

* Reference To: USER32.GetDlgItemTextA, Ord:0107h

            |

:0041241D 8B2D48E34100            mov ebp, dword

ptr [0041E348]

:00412423 6800010000              push 00000100

:00412428 68043B6500              push 00653B04

* Possible Reference to Dialog: DLGFILEWAIT, CONTROL_ID:0448, ""

            |

:00412479 6848040000              push 00000448

:0041247E 83E103

and ecx, 00000003

:00412481 53

  push ebx

:00412482 F3

  repz

:00412483 A4

  movsb

:00412484 FFD5

  call ebp

:00412486 BF043C6500              mov edi,

00653C04

:0041248B 83C9FF

or ecx, FFFFFFFF

:0041248E 33C0

  xor eax, eax

:00412490 68043B6500              push 00653B04

:00412495 F2

  repnz

:00412496 AE

  scasb

:00412497 F7D1

  not ecx

:00412499 2BF9

  sub edi, ecx

:0041249B 8BF7

  mov esi, edi

:0041249D 8BD1

  mov edx, ecx

:0041249F BF043B6500              mov edi,

00653B04

:004124A4 83C9FF

or ecx, FFFFFFFF

:004124A7 F2

  repnz

:004124A8 AE

  scasb

:004124A9 8BCA

  mov ecx, edx

:004124AB 4F

  dec edi

:004124AC C1E902

shr ecx, 02

:004124AF F3

  repz

:004124B0 A5

  movsd

:004124B1 8BCA

  mov ecx, edx

:004124B3 83E103

and ecx, 00000003

:004124B6 F3

  repz

:004124B7 A4

  movsb

:004124B8 E8F3FEFFFF              call 004123B0

    \\运行此CAll后假注册码就变成了数值型

:004124BD 8BF0

  mov esi, eax

:004124BF 56

  push esi

:004124C0 E8DBFDFFFF              call 004122A0

      \\这里是关键

:004124C5 83C408

add esp, 00000008

:004124C8 85C0

  test eax, eax

:004124CA 7558

  jne 00412524        \\不跳肯定完蛋！

:004124CC 6A10

  push 00000010

* Possible StringData Ref from Data Obj ->"Don't Panic!"

            |

:004124CE 6810174200              push 00421710

* Possible StringData Ref from Data Obj ->"The registration number you have

"

                  ->"entered is

not valid.  Please "

                  ->"enter a valid

registration number."

            |

:004124D3 681C334200              push 0042331C

:004124D8 53

  push ebx

进入这个call `004123B0`，到了这里：

|:00405419 , :0040CD7A , :0040CDE9 , :0040DFEE , :004124C0

\\很多调用哟！

|

```
:004122A0 8B442404                mov
eax, dword ptr [esp+04]
:004122A4 68D135E2E1              push E1E235D1
  \\注意这个数 （1）
:004122A9 681953C633              push 33C65319
  \\注意这个数 （2）
:004122AE 50
  push eax
:004122AF E8DCFFFFFF              call 00412290
  \\进去瞧瞧
```
进入call `00412290`，这里：

```
:00412290 8B442404                mov
eax, dword ptr [esp+04]  \\这个是假注册码
:00412294 8B4C240C                mov
ecx, dword ptr [esp+0C]  \\这个是数（1）
:00412298 33C1
  xor eax, ecx
\\异或两个数
:0041229A 0FAF442408              imul eax,
dword ptr [esp+08]  \\然后乘数（2）
:0041229F C3
  ret
      \\返回到。。。下面
:004122B4 33D2
  xor edx, edx
\\清空寄存器edx
:004122B6 B9BB0B0000              mov ecx,
00000BBB            \\BBB放到ecx
:004122BB F7F1
  div ecx
    \\eax除ecx也就是除BBB，余数放进了edx
:004122BD 83C40C
add esp, 0000000C
:004122C0 8BC2
  mov eax, edx
\\余数给eax
:004122C2 F7D8
  neg eax
    \\求补
:004122C4 1BC0
  sbb eax, eax
\\eax-eax-cf=>eax *eax最后要等于0*
:004122C6 40
  inc eax
    \\加1            *这里eax才会变成1.才满足后面
                的条件,这也就是间接要求余数
                为0*
                    .
                    .
:004124C8 85C0
  test eax, eax
:004124CA 7558
  jne 00412524
\\跳过去就爽了!!
```
* Possible StringData Ref from Data Obj ->"Don't Panic!"

            |

:004124CE 6810174200              push 00421710

* Possible StringData Ref from Data Obj ->"The registration number you have

"

                  ->"entered is

not valid.  Please "

                  ->"enter a valid

registration number."

算法:

输入的注册码转换为数值型与`E1E235D1`异或再乘`33C65319`/`BBB`最后余数为0即可

一个有效的注册码:`3789698513`

再次感谢 小球[CCG]的鼎立相助，呵呵！
