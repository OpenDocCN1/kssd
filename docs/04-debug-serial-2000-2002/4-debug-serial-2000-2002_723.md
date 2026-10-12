
# 新纪元财务、进销存一体化软件

> 原文：[`www.pediy.com/kssd/pediy03/forum863.htm`](https://www.pediy.com/kssd/pediy03/forum863.htm)

新纪元财务、进销存一体化软件

下载地址：`http://www.shareware.net.cn/dl.asp?id={AA5CFB89-2BCB-4474-9C43-3FE0CB156308}`

使用工具：`HIEW`、`W32DASM`、`language`

操作步骤：

一、运行 `language` 打开 `xsj.exe` 显示格式为 DELPHI 语言

二、用 `W32DASM` 反汇编 `xsj.exe`

相关如下：

```
:512538F2 0F850B010000             jne 51253A03  ==>00252cf2h

909090909090                 ==>修改（注：不让此行跳到出错地址 ）

:512538F8 8B8680030000             mov eax, dword ptr [esi+00000380]

:512538FE 33D2                     xor edx, edx

:51253900 E8EB2DDFFF               call 510466F0

:51253905 BA02000080               mov edx, 80000002

:5125390A A1B4102651               mov eax, dword ptr [512610B4]

:5125390F E8BC9EE3FF               call 5108D7D0

:51253914 B101                     mov cl, 01

* Possible StringData Ref from Code Obj ->"\\SoftWare\\Microsoft_98"

                                                    |

:51253916 BACC3A2551               mov edx, 51253ACC

:5125391B A1B4102651               mov eax, dword ptr [512610B4]

:51253920 E8EF9FE3FF               call 5108D914

:51253925 8B45F4                   mov eax, dword ptr [ebp-0C]

:51253928 E8C361DBFF               call 51009AF0

:5125392D 8BC8                     mov ecx, eax

:5125392F 81C100040000             add ecx, 00000400

:51253935 BAC03A2551               mov edx, 51253AC0

:5125393A A1B4102651               mov eax, dword ptr [512610B4]

:5125393F E8C4A5E3FF               call 5108DF08

:51253944 A1B4102651               mov eax, dword ptr [512610B4]

:51253949 E8529EE3FF               call 5108D7A0

:5125394E 6A00                     push 00000000

* Possible StringData Ref from Code Obj ->"注册完成!

序列号："

                                                    |

:51253950 68EC3A2551               push 51253AEC

:51253955 8D95A4FEFFFF             lea edx, dword ptr [ebp+FFFFFEA4]

:5125395B A154EB2551               mov eax, dword ptr [5125EB54]

:51253960 8B00                     mov eax, dword ptr [eax]

:51253962 8B80E8020000             mov eax, dword ptr [eax+000002E8]

:51253968 E8B325DEFF               call 51035F20

:5125396D 8B85A4FEFFFF             mov eax, dword ptr [ebp+FFFFFEA4]

:51253973 8D95A8FEFFFF             lea edx, dword ptr [ebp+FFFFFEA8]

:51253979 E84E5EDBFF               call 510097CC

:5125397E FFB5A8FEFFFF             push dword ptr [ebp+FFFFFEA8]

* Possible StringData Ref from Code Obj ->"

"

                                                    |

:51253984 680C3B2551               push 51253B0C

* Possible StringData Ref from Code Obj ->"注册码："

                                                    |

:51253989 681C3B2551               push 51253B1C

:5125398E 8D959CFEFFFF             lea edx, dword ptr [ebp+FFFFFE9C]

:51253994 A154EB2551               mov eax, dword ptr [5125EB54]

:51253999 8B00                     mov eax, dword ptr [eax]

:5125399B 8B80E0020000             mov eax, dword ptr [eax+000002E0]

:512539A1 E87A25DEFF               call 51035F20

:512539A6 8B859CFEFFFF             mov eax, dword ptr [ebp+FFFFFE9C]

:512539AC 8D95A0FEFFFF             lea edx, dword ptr [ebp+FFFFFEA0]

:512539B2 E8155EDBFF               call 510097CC

:512539B7 FFB5A0FEFFFF             push dword ptr [ebp+FFFFFEA0]

* Possible StringData Ref from Code Obj ->"

"

                                                    |

:512539BD 680C3B2551               push 51253B0C

* Possible StringData Ref from Code Obj ->"退出本软件重新进入后,您将成为正式用户!"

                                                    |

:512539C2 68303B2551               push 51253B30

:512539C7 8D85ACFEFFFF             lea eax, dword ptr [ebp+FFFFFEAC]

:512539CD BA07000000               mov edx, 00000007

:512539D2 E85D06DBFF               call 51004034

:512539D7 8B85ACFEFFFF             mov eax, dword ptr [ebp+FFFFFEAC]

:512539DD E85607DBFF               call 51004138

:512539E2 8BD0                     mov edx, eax

* Possible StringData Ref from Code Obj ->"祝贺!"

                                                    |

:512539E4 B9583B2551               mov ecx, 51253B58

:512539E9 A138EE2551               mov eax, dword ptr [5125EE38]

:512539EE 8B00                     mov eax, dword ptr [eax]

:512539F0 E84712E0FF               call 51054C3C

:512539F5 A138EE2551               mov eax, dword ptr [5125EE38]

:512539FA 8B00                     mov eax, dword ptr [eax]

:512539FC E89711E0FF               call 51054B98

:51253A01 EB18                     jmp 51253A1B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:512538F2(C)

|

:51253A03 6A00                     push 00000000

* Possible StringData Ref from Code Obj ->"警告!"

                                                    |

:51253A05 B9603B2551               mov ecx, 51253B60

* Possible StringData Ref from Code Obj ->"注册号不正确!"

                                                    |

:51253A0A BA683B2551               mov edx, 51253B68
```

三、用 `HIEW` 查找 `0f850b010000` 修改为 `909090909090`

成功，输入任意号码注册完成。~~~~

# 新纪元出纳专用软件

下载地址：`http://www.shareware.net.cn/dl.asp?id={45B78F75-5253-470E-B67D-02D1924E2E6E}`

使用工具：`HIEW`、`language`、`W32DASM`

具体步骤和上面那个差不多就不多说了

```
:004F3899 E85607F1FF                 call 00403FF4

:004F389E 0F850B010000               jne 004F39AF  ==>修改为 909090909090

:004F38A4 8B86B0030000               mov eax, dword ptr [esi+000003B0]

:004F38AA 33D2                       xor edx, edx

:004F38AC E8DF12F5FF                 call 00444B90

:004F38B1 BA02000080                 mov edx, 80000002

:004F38B6 A1A0924F00                 mov eax, dword ptr [004F92A0]

:004F38BB E858BDF8FF                 call 0047F618

:004F38C0 B101                       mov cl, 01

* Possible StringData Ref from Code Obj ->"\\SoftWare\\Microsoft_98"

                                                    |

:004F38C2 BA783A4F00                 mov edx, 004F3A78

:004F38C7 A1A0924F00                 mov eax, dword ptr [004F92A0]

:004F38CC E8ABBDF8FF                 call 0047F67C

:004F38D1 8B45F4                     mov eax, dword ptr [ebp-0C]

:004F38D4 E8635DF1FF                 call 0040963C

:004F38D9 8BC8                       mov ecx, eax

:004F38DB 81C100040000               add ecx, 00000400

:004F38E1 BA6C3A4F00                 mov edx, 004F3A6C

:004F38E6 A1A0924F00                 mov eax, dword ptr [004F92A0]

:004F38EB E800BFF8FF                 call 0047F7F0

:004F38F0 A1A0924F00                 mov eax, dword ptr [004F92A0]

:004F38F5 E8EEBCF8FF                 call 0047F5E8

:004F38FA 6A00                       push 00000000

* Possible StringData Ref from Code Obj ->"注册完成!

序列号："

                                                    |

:004F38FC 68983A4F00                 push 004F3A98

:004F3901 8D95A4FEFFFF               lea edx, dword ptr [ebp+FFFFFEA4]

:004F3907 A1DC784F00                 mov eax, dword ptr [004F78DC]

:004F390C 8B00                       mov eax, dword ptr [eax]

:004F390E 8B80E8020000               mov eax, dword ptr [eax+000002E8]

:004F3914 E8970CF4FF                 call 004345B0

:004F3919 8B85A4FEFFFF               mov eax, dword ptr [ebp+FFFFFEA4]

:004F391F 8D95A8FEFFFF               lea edx, dword ptr [ebp+FFFFFEA8]

:004F3925 E8225AF1FF                 call 0040934C

:004F392A FFB5A8FEFFFF               push dword ptr [ebp+FFFFFEA8]

* Possible StringData Ref from Code Obj ->"

"

                                                    |

:004F3930 68B83A4F00                 push 004F3AB8

* Possible StringData Ref from Code Obj ->"注册码："

                                                    |

:004F3935 68C83A4F00                 push 004F3AC8

:004F393A 8D959CFEFFFF               lea edx, dword ptr [ebp+FFFFFE9C]

:004F3940 A1DC784F00                 mov eax, dword ptr [004F78DC]

:004F3945 8B00                       mov eax, dword ptr [eax]
```



```asm
:004F3947 8B80E0020000           mov eax, dword ptr [eax+000002E0]

:004F394D E85E0CF4FF            call 004345B0

:004F3952 8B859CFEFFFF         mov eax, dword ptr [ebp+FFFFFE9C]

:004F3958 8D95A0FEFFFF         lea edx, dword ptr [ebp+FFFFFEA0]

:004F395E E8E959F1FF           call 0040934C

:004F3963 FFB5A0FEFFFF         push dword ptr [ebp+FFFFFEA0]

* Possible StringData Ref from Code Obj ->"

"

                                                        |

:004F3969 68B83A4F00           push 004F3AB8

* Possible StringData Ref from Code Obj ->"退出本软件重新进入后,您将成为正式用户!"

                                                        |

:004F396E 68DC3A4F00           push 004F3ADC

:004F3973 8D85ACFEFFFF         lea eax, dword ptr [ebp+FFFFFEAC]

:004F3979 BA07000000           mov edx, 00000007

:004F397E E82106F1FF           call 00403FA4

:004F3983 8B85ACFEFFFF         mov eax, dword ptr [ebp+FFFFFEAC]

:004F3989 E81A07F1FF           call 004040A8

:004F398E 8BD0                 mov edx, eax

* Possible StringData Ref from Code Obj ->"祝贺!"

                                                        |

:004F3990 B9043B4F00           mov ecx, 004F3B04

:004F3995 A15C7B4F00           mov eax, dword ptr [004F7B5C]

:004F399A 8B00                 mov eax, dword ptr [eax]

:004F399C E83BF7F5FF           call 004530DC

:004F39A1 A15C7B4F00           mov eax, dword ptr [004F7B5C]

:004F39A6 8B00                 mov eax, dword ptr [eax]

:004F39A8 E88BF6F5FF           call 00453038

:004F39AD EB18                 jmp 004F39C7

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004F389E(C)

|

:004F39AF 6A00                 push 00000000

* Possible StringData Ref from Code Obj ->"警告!"

                                                        |

:004F39B1 B90C3B4F00           mov ecx, 004F3B0C

* Possible StringData Ref from Code Obj ->"注册号不正确!"

                                                        |

:004F39B6 BA143B4F00           mov edx, 004F3B14

:004F39BB A15C7B4F00           mov eax, dword ptr [004F7B5C]

:004F39C0 8B00                 mov eax, dword ptr [eax]

:004F39C2 E815F7F5FF           call 004530DC

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004F356D(C), :004F39AD(U)

|

:004F39C7 33C0                 xor eax, eax
```

## 再破新纪元系列软件——出纳专用软件（注册码破解）

由于本人在这方面是个菜鸟！刚得`幻影 2001`的帮助才得以破了这个软件！先献丑如下：

用`TRW2000`还是老办法：

一、先填注册码：`87654321`（软件的用户名它已经填好了，不能动）

二、`ctrl+N`下`bpx hmemcpy`

三、`ctrl+N`点“注册”

四、被拦。

```asm
    bd *
    pmodule
```

按`8`次`F12`后，弹出“注册失败”

五、重复`1--4`，只按`7`次`F12`，再按一次`F10`。看到如下代码：

```asm
015F:004F387F  MOV      EAX,[EBP+FFFFFEB0]

015F:004F3885  LEA      EDX,[EBP+FFFFFEB4]

015F:004F388B  CALL     0040934C

015F:004F3890  MOV      EDX,[EBP+FFFFFEB4] <-------------F10,d edx 显示 87654321

015F:004F3896  MOV      EAX,[EBP-0C]       <-------------F10,d eax 显示 8454953（真注册码）

015F:004F3899  CALL     00403FF4          <-------------比较注册码的真假！

015F:004F389E  JNZ      NEAR 004F39AF                       *****跳转*****

          .

          .

          .

015F:004F39AF  PUSH     BYTE +00

015F:004F39B1  MOV      ECX,004F3B0C

015F:004F39B6  MOV      EDX,004F3B14

015F:004F39BB  MOV      EAX,[004F7B5C]

015F:004F39C0  MOV      EAX,[EAX]

015F:004F39C2  CALL     004530DC                        *****出错*****
```

是不是很简单！本人自识学艺肤浅，望众高手不要取笑！

我会再接再厉的！

```c
#include "stdio.h"

main()

{

char string[8];

int i,c;

printf(" *****NEWXSJ******\nThis keygen is made by Pretender\nPlease input your ID :  ");

gets(string);

for(i=0;(c=string[i])!='\0';i++)

{if(c==70||c==102) string[i]=0;

 else if(c<70&&c>64) string[i]-=64;

 else if(c<102&&c>96) string[i]-=96;

 else string[i]-=48;

}

 printf("Your Register code is:  85");

for(i=0;i<4;i++)

printf("%d",(string[i]+string[7-i])%10);

printf("%d",(string[0]+string[1]+string[2]+string[3]+string[4]+string[5]+string[6]+string[7])%10);

printf("\n *********************\n *Welcome to WWW.CRACKNOW.COM*\n *********************\n");}

4dd466d0e1e0cc123afbe504da2462ca
```
