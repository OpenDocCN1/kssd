
# Access Animation

> 原文：[`www.pediy.com/kssd/pediy06/pediy6019.htm`](https://www.pediy.com/kssd/pediy06/pediy6019.htm)

软件大小：  1289 KB

软件语言：  英文

软件类别：  国外软件 / 共享版 / 动画制作

应用平台：  `Win9x/NT/2000/XP`

界面预览：  无

加入时间：  2003-05-02 17:23:25

下载次数：  872

破解工具：`TRW1.22` `PWdasmV10.0`

软件下载：`http://count.skycn.com/softdown.php?id=11878&url=http://on165-http.skycn.net/down/aanim.zip`

软件简介：`Access Animation` 是一套专门为了制作 `GIF` 动画而产生的软件，里面内建了完整的绘图机能。甚至一般绘图软件所拥有的喷枪、绘图刷等绘图工具，也一应俱全。在软件中你可以指定动画的张数以及动画之间的延迟时间，以及图形透明化程度等参数，让你的动画 `GIF` 档案更能和你的网页结合。

作者声明：初学 Crack，仅作学习交流，无其它目的。

`BC` 编写，无壳，下中断 `BPX GETDLGITEMTEXTA`，点 `OK` 键，程序被中断，按 `F12`，再按 `F10`，来到如下代码：

```
0167:00415D2D  PUSH     DWORD 0104                       <---程序中断于此

0167:00415D32  LEA      ECX,[EBP+FFFFFEE0]

0167:00415D38  PUSH     ECX

0167:00415D39  PUSH     BYTE +64

0167:00415D3B  PUSH     DWORD [EBP+08]

0167:00415D3E  CALL     USER32!GetDlgItemTextA           <---取输入的注册码

0167:00415D43  LEA      EAX,[EBP+FFFFFEE0]               <---EAX=输入的注册码

0167:00415D49  PUSH     EAX

0167:00415D4A  CALL     00436F68                          <---取注册码的位数

0167:00415D4F  POP      ECX

0167:00415D50  MOV      EDI,EAX

0167:00415D52  CMP      EDI,EBX                           <---判断有无输入注册码

0167:00415D54  JNG      00415DC2

0167:00415D56  MOV      AL,[EBP+EBX+FFFFFEE0]

0167:00415D5D  CMP      AL,2D                             <---判断是否为“-”

0167:00415D5F  JZ       00415D68

0167:00415D61  LEA      EDX,[EDI-01]

0167:00415D64  CMP      EBX,EDX

0167:00415D66  JNZ      00415DAE

0167:00415D68  LEA      ECX,[EDI-01]

0167:00415D6B  CMP      EBX,ECX

0167:00415D6D  JNZ      00415D7D

0167:00415D6F  MOV      AL,[EBP+EBX+FFFFFEE0]

0167:00415D76  MOV      [EBP+ESI+FFFFFDDC],AL             <---将每部分注册码移到 EBP+ESI+FFFFFDDC 的地址中

0167:00415D7D  LEA      EDX,[EBP+FFFFFDDC]

0167:00415D83  PUSH     EDX

0167:00415D84  CALL     0043BD5C                          <---将注册码的 10 进制换为 16 进制值

0167:00415D89  POP      ECX

0167:00415D8A  MOV      ECX,[EBP-04]

0167:00415D8D  MOV      [EBP+ECX*4-10],EAX                <----转换的值存入[EBP+ECX*4-10]中，后面会用到

0167:00415D91  INC      DWORD [EBP-04]                     即后面的[EBP-10] [EBP-0C] [EBP-08]

0167:00415D94  PUSH     DWORD 0104

0167:00415D99  PUSH     BYTE +00

0167:00415D9B  LEA      EAX,[EBP+FFFFFDDC]

0167:00415DA1  PUSH     EAX

0167:00415DA2  CALL     00436EA0

0167:00415DA7  ADD      ESP,BYTE +0C

0167:00415DAA  XOR      ESI,ESI

0167:00415DAC  JMP      SHORT 00415DBD

0167:00415DAE  MOV      AL,[EBP+EBX+FFFFFEE0]

0167:00415DB5  MOV      [EBP+ESI+FFFFFDDC],AL

0167:00415DBC  INC      ESI

0167:00415DBD  INC      EBX

0167:00415DBE  CMP      EDI,EBX

0167:00415DC0  JG       00415D56

0167:00415DC2  PUSH     DWORD 00475E50    <---475E50 为我们输入的用户名

0167:00415DC7  CALL     00436F68

0167:00415DCC  POP      ECX

0167:00415DCD  MOV      EDI,EAX

0167:00415DCF  TEST     EAX,EAX

0167:00415DD1  JNG      00415DED

0167:00415DD3  XOR      EBX,EBX

0167:00415DD5  PUSH     EBX

0167:00415DD6  PUSH     DWORD 00475E50

0167:00415DDB  CALL     00416034        <---计算注册码的关键 Call，调用 3 次，跟进

0167:00415DE0  ADD      ESP,BYTE +08

0167:00415DE3  MOV      [EBP+EBX*4-1C],EAX

0167:00415DE7  INC      EBX

0167:00415DE8  CMP      EBX,BYTE +03

0167:00415DEB  JL       00415DD5

0167:00415DED  MOV      EAX,[0046B12C]  <---[0046B12C]为程序运行的次数

0167:00415DF2  CMP      EAX,C8           <---此处为第一个关键判断,判断是否运行到 200 次

0167:00415DF7  JNZ      00415E78        <---跳到出错对话框，此处改为两个 nop nop

0167:00415DF9  PUSH     DWORD 00475F54

0167:00415DFE  LEA      EDX,[EBP+FFFFFEE0]

0167:00415E04  PUSH     EDX

0167:00415E05  CALL     KERNEL32!lstrcmpA

0167:00415E0A  TEST     EAX,EAX

0167:00415E0C  JZ       00415E2C

0167:00415E0E  MOV      ECX,[EBP-10]    <---取假码的第一部分

0167:00415E11  MOV      EAX,[EBP-1C]    <---取正确注册码的第一部分

0167:00415E14  CMP      ECX,EAX         <---关键比较 2,比较注册码第一部分

0167:00415E16  JNZ      00415E78

0167:00415E18  MOV      EDX,[EBP-0C]    <---取假码的第若部分

0167:00415E1B  MOV      ECX,[EBP-18]

0167:00415E1E  CMP      EDX,ECX         <---关键比较 3,比较注册码第二部分

0167:00415E20  JNZ      00415E78

0167:00415E22  MOV      EAX,[EBP-08]    <---取假码的第三部分

0167:00415E25  MOV      EDX,[EBP-14]

0167:00415E28  CMP      EAX,EDX         <---关键比较 4,比较注册码第三部分

0167:00415E2A  JNZ      00415E78

0167:00415E2C  MOV      DWORD [0043F09C],01

0167:00415E36  PUSH     BYTE +01

0167:00415E38  CALL     004030E3

0167:00415E3D  POP      ECX

0167:00415E3E  PUSH     DWORD 0046B0FE

0167:00415E43  PUSH     DWORD 004405D6

0167:00415E48  PUSH     DWORD 0046A922

0167:00415E4D  CALL     USER32!wsprintfA

0167:00415E52  ADD      ESP,BYTE +0C

0167:00415E55  PUSH     BYTE +40

0167:00415E57  PUSH     DWORD 0046B0FE

0167:00415E5C  PUSH     DWORD 0046A922

0167:00415E61  PUSH     DWORD [EBP+08]

0167:00415E64  CALL     USER32!MessageBoxA      <---注册成功！

0167:00415E69  PUSH     BYTE +00

0167:00415E6B  PUSH     DWORD [EBP+08]

0167:00415E6E  CALL     USER32!EndDialog

0167:00415E73  JMP      00415F0C

0167:00415E78  PUSH     BYTE +40

0167:00415E7A  PUSH     DWORD 00440619

0167:00415E7F  PUSH     DWORD 004405F3

0167:00415E84  PUSH     DWORD [EBP+08]

0167:00415E87  CALL     USER32!MessageBoxA      <---出错对话框！

0167:00415E8C  PUSH     BYTE +64

0167:00415E8E  PUSH     DWORD [EBP+08]

0167:00415E91  CALL     USER32!GetDlgItem

0167:00415E96  PUSH     EAX

0167:00415E97  CALL     USER32!SetFocus

0167:00415E9C  JMP      SHORT 00415F0C

0167:00415E9E  PUSH     DWORD 0104

0167:00415EA3  PUSH     DWORD 00475E50

0167:00415EA8  PUSH     BYTE +65

0167:00415EAA  PUSH     DWORD [EBP+08]
```

跟进上面的那个关键 `Call`，来到如下代码：

```
0167:00416034  PUSH     EBP

0167:00416035  MOV      EBP,ESP

0167:00416037  PUSH     ECX

0167:00416038  PUSH     EBX

0167:00416039  PUSH     ESI

0167:0041603A  PUSH     EDI

0167:0041603B  MOV      ESI,[EBP+0C]

0167:0041603E  MOV      EDI,[EBP+08]             <---EDI 中为用户名

0167:00416041  XOR      EBX,EBX

0167:00416043  PUSH     EDI

0167:00416044  CALL     00436F68                <---取用户名长度，返回值在 EAX 中

0167:00416049  POP      ECX

0167:0041604A  MOV      [EBP-04],EAX

0167:0041604D  MOV      EAX,[EBP-04]

0167:00416050  CMP      EAX,FF                  <---比较用户名长度是否大于 255，

0167:00416055  JNG      0041605E                谁会用这么长的名字?!

0167:00416057  MOV      DWORD [EBP-04],FF

0167:0041605E  XOR      ECX,ECX                 <---ECX=0,作用同指针

0167:00416060  MOV      EDX,[EBP-04]

0167:00416063  CMP      ECX,EDX

0167:00416065  JNL      NEAR 00416527

0167:0041606B  MOVSX    EAX,BYTE [EDI+ECX]      <---按位取用户名的每个字符值到 EAX 中

0167:0041606F  ADD      EAX,BYTE -20            <---EAX=EAX-20,小写字母转换成大写

0167:00416072  CMP      EAX,BYTE +5A            <---比较是否大于‘Z’

0167:00416075  JA       NEAR 004164CF

0167:0041607B  MOV      AL,[EAX+00416088]       <---用[EAX+00416088]的值查表给 AL 赋值

自 016F:0416088 处的表值如下：

016F:416088:1C 00 00 00 00 00 00 00-00 00 00 00 00 00 1B 00

016F:416098:00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00

016F:4160A8:00 1A 19 18 17 16 15 14-13 12 11 10 0F 0E 0D 0C

016F:4160B8:0B 0A 09 08 07 06 05 04 03 02 01 00 00 00 00 00

016F:4160C8:0A 1A 19 18 17 16 15 14-13 12 11 10 0F 0E 0D 0C

016F:4160D8:0B 0A 09 08 07 06 05 04 03 02 01 CF 04 41 00 09

0167:00416081  JMP      NEAR [EAX*4+004160E3]   <---根据[EAX*4+004160E3]的值查表决定程序跳转位置

自 004160E3 处的表值如下：

016F:004160E3:CF 64 41 00 09 64 41 00-44 64 41 00 25 64 41 00

016F:004160E3:7E 64 41 00 E7 63 41 00-C8 63 41 00 A6 63 41 00
```



016F:004160E3:87 63 41 00 65 63 41 00-63 64 41 00 46 63 41 00

016F:004160E3:27 63 41 00 05 63 41 00-E3 62 41 00 C1 62 41 00

016F:004160E3:F9 62 41 00 80 62 41 00-5E 62 41 00 3C 62 41 00

016F:004160E3:1A 62 41 00 F8 61 41 00-D9 61 41 00 BA 61 41 00

016F:004160E3:98 61 41 00 76 61 41 00-57 61 41 00 99 64 41 00

```
0167:00416088  SBB      AL,00

0167:0041608A  ADD      [EAX],AL

0167:0041608C  ADD      [EAX],AL

0167:0041608E  ADD      [EAX],AL

0167:00416090  ADD      [EAX],AL

0167:00416092  ADD      [EAX],AL

0167:00416094  ADD      [EAX],AL

0167:00416096  SBB      EAX,[EAX]

0167:00416098  ADD      [EAX],AL

0167:0041609A  ADD      [EAX],AL

0167:0041609C  ADD      [EAX],AL

0167:0041609E  ADD      [EAX],AL

0167:004160A0  ADD      [EAX],AL

0167:004160A2  ADD      [EAX],AL

0167:004160A4  ADD      [EAX],AL

0167:004160A6  ADD      [EAX],AL

0167:004160A8  ADD      [EDX],BL

0167:004160AA  SBB      [EAX],EBX

0167:004160AC  POP      SS

0167:004160AD  PUSH     SS

0167:004160AE  ADC      EAX,11121314

0167:004160B3  ADC      [EDI],CL

0167:004160B5  PUSH     CS

0167:004160B6  OR       EAX,090A0B0C

0167:004160BB  OR       [EDI],AL

0167:004160BD  PUSH     ES

0167:004160BE  ADD      EAX,01020304

0167:004160C3  ADD      [EAX],AL

0167:004160C5  ADD      [EAX],AL

0167:004160C7  ADD      [EAX],AL

0167:004160C9  SBB      BL,[ECX]

0167:004160CB  SBB      [EDI],DL

0167:004160CD  PUSH     SS

0167:004160CE  ADC      EAX,11121314

0167:004160D3  ADC      [EDI],CL

0167:004160D5  PUSH     CS

0167:004160D6  OR       EAX,090A0B0C

0167:004160DB  OR       [EDI],AL

0167:004160DD  PUSH     ES

0167:004160DE  ADD      EAX,01020304

0167:004160E3  IRET

0167:004160E4  INC      ECX

0167:004160E6  ADD      [ECX],CL

0167:004160E8  INC      ECX

0167:004160EA  ADD      [ESP+41],AL

0167:004160EE  ADD      [7E004164],AH

0167:004160F4  INC      ECX

0167:004160F6  ADD      BH,AH

0167:004160F8  ARPL     [ECX+00],AX

0167:004160FB  ENTER    4163,00

0167:004160FF  CMPSB

0167:00416100  ARPL     [ECX+00],AX

0167:00416103  XCHG     ESP,[EBX+41]

0167:00416106  ADD      [EBP+63],AH

0167:00416109  INC      ECX

0167:0041610A  ADD      [EBX+64],AH

0167:0041610D  INC      ECX

0167:0041610E  ADD      [ESI+63],AL

0167:00416111  INC      ECX

0167:00416112  ADD      [EDI],AH

0167:00416114  ARPL     [ECX+00],AX

0167:00416117  ADD      EAX,E3004163

0167:0041611C  BOUND    EAX,[ECX+00]

0167:0041611F  SHL      DWORD [EDX+41],00

0167:00416123  LAHF

0167:00416124  BOUND    EAX,[ECX+00]

0167:00416127  AND      BYTE [EDX+41],00

0167:0041612B  POP      ESI

0167:0041612C  BOUND    EAX,[ECX+00]

0167:0041612F  CMP      AL,62

0167:00416131  INC      ECX

0167:00416132  ADD      [EDX],BL

0167:00416134  BOUND    EAX,[ECX+00]

0167:00416137  CLC

0167:00416138  POPA

0167:00416139  INC      ECX

0167:0041613A  ADD      CL,BL

0167:0041613C  POPA

0167:0041613D  INC      ECX

0167:0041613E  ADD      [EDX+98004161],BH

0167:00416144  POPA

0167:00416145  INC      ECX

0167:00416146  ADD      [ESI+61],DH

0167:00416149  INC      ECX

0167:0041614A  ADD      [EDI+61],DL

0167:0041614D  INC      ECX

0167:0041614E  ADD      [ECX+B4004164],BL

0167:00416154  INC      ECX

0167:00416156  ADD      [EBX+BE0F7FC3],AL

0167:0041615C  ADC      AL,0F

0167:0041615E  TEST     BYTE [EDX+00444089],08

0167:00416165  JZ       NEAR 004164E0

0167:0041616B  ADD      EBX,FD

0167:00416171  JMP      004164E0

0167:00416176  ADD      EBX,02D3               <---EBX=EBX+02D3   ------------|

0167:0041617C  MOVSX    EAX,BYTE [EDI+ECX]     <---按位取用户名的每个字符     |

0167:00416180  TEST     BYTE [EAX+00444089],08<---根据[EAX+00444089]的值查表 |

0167:00416187  JZ       NEAR 004164E0          得到相应值与 08H 进行判断      |

0167:0041618D  ADD      EBX,86                 <---EBX=EBX+86                 |

0167:00416193  JMP      004164E0               <------------------------------|

0167:00416198  ADD      EBX,0929               <---以下同上，懒得写了|

0167:0041619E  MOVSX    EDX,BYTE [EDI+ECX]                             |

0167:004161A2  TEST     BYTE [EDX+00444089],08                           |

0167:004161A9  JZ       NEAR 004164E0                                      |

0167:004161AF  ADD      EBX,01D9                                           |

0167:004161B5  JMP      004164E0                                           |

0167:004161BA  ADD      EBX,030D                                           |

0167:004161C0  MOVSX    EAX,BYTE [EDI+ECX]                                 |

0167:004161C4  TEST     BYTE [EAX+00444089],08                             |

0167:004161CB  JZ       NEAR 004164E0                                      |

0167:004161D1  ADD      EBX,BYTE +47                                       |

0167:004161D4  JMP      004164E0                                           |

0167:004161D9  ADD      EBX,BYTE +02                                       |

0167:004161DC  MOVSX    EDX,BYTE [EDI+ECX]                                 |

0167:004161E0  TEST     BYTE [EDX+00444089],08                             |

0167:004161E7  JZ       NEAR 004164E0                                      |

0167:004161ED  ADD      EBX,1E61                                           |

0167:004161F3  JMP      004164E0                                           |

0167:004161F8  ADD      EBX,01C8                                           |

0167:004161FE  MOVSX    EAX,BYTE [EDI+ECX]                                 |

0167:00416202  TEST     BYTE [EAX+00444089],08                             |

0167:00416209  JZ       NEAR 004164E0                                      |

0167:0041620F  ADD      EBX,022B                                           |

0167:00416215  JMP      004164E0                                           |

0167:0041621A  ADD      EBX,0114                                           |

0167:00416220  MOVSX    EDX,BYTE [EDI+ECX]                                 |

0167:00416224  TEST     BYTE [EDX+00444089],08                             |

0167:0041622B  JZ       NEAR 004164E0                                      |

0167:00416231  ADD      EBX,0378                                           |

0167:00416237  JMP      004164E0                                           |

0167:0041623C  ADD      EBX,015A                                           |

0167:00416242  MOVSX    EAX,BYTE [EDI+ECX]                                 |

0167:00416246  TEST     BYTE [EAX+00444089],08                             |

0167:0041624D  JZ       NEAR 004164E0                                      |

0167:00416253  ADD      EBX,0235                                           |

0167:00416259  JMP      004164E0                                           |

0167:0041625E  ADD      EBX,F6                                             |

0167:00416264  MOVSX    EDX,BYTE [EDI+ECX]                                 |

0167:00416268  TEST     BYTE [EDX+00444089],08                             |

0167:0041626F  JZ       NEAR 004164E0                                      |

0167:00416275  ADD      EBX,03F2                                           |

0167:0041627B  JMP      004164E0                                           |

0167:00416280  ADD      EBX,1EB1                                           |

0167:00416286  MOVSX    EAX,BYTE [EDI+ECX]                                 |

0167:0041628A  TEST     BYTE [EAX+00444089],08                             |

0167:00416291  JZ       NEAR 004164E0                                      |

0167:00416297  ADD      EBX,BYTE +6E                                       |

0167:0041629A  JMP      004164E0                                           |
```



```
0167:0041629F  ADD      EBX,0278                                                               |

0167:004162A5  MOVSX    EDX,BYTE [EDI+ECX]                                                     |

0167:004162A9  TEST     BYTE [EDX+00444089],08                                                 |

0167:004162B0  JZ       NEAR 004164E0                                                          |

0167:004162B6  ADD      EBX,173A                                                               |

0167:004162BC  JMP      004164E0                                                               |

0167:004162C1  ADD      EBX,017F                                                               |

0167:004162C7  MOVSX    EAX,BYTE [EDI+ECX]                                                     |

0167:004162CB  TEST     BYTE [EAX+00444089],08                                                 |

0167:004162D2  JZ       NEAR 004164E0                                                          |

0167:004162D8  ADD      EBX,04FE                                                               |

0167:004162DE  JMP      004164E0                                                               |

0167:004162E3  ADD      EBX,115D                                                               |

0167:004162E9  MOVSX    EDX,BYTE [EDI+ECX]                                                     |

0167:004162ED  TEST     BYTE [EDX+00444089],08                                                 |

0167:004162F4  JZ       NEAR 004164E0                                                          |

0167:004162FA  ADD      EBX,0338                                                               |

0167:00416300  JMP      004164E0                                                               |

0167:00416305  ADD      EBX,2382                                                               |

0167:0041630B  MOVSX    EAX,BYTE [EDI+ECX]                                                     |

0167:0041630F  TEST     BYTE [EAX+00444089],08                                                 |

0167:00416316  JZ       NEAR 004164E0                                                          |

0167:0041631C  ADD      EBX,0DED                                                               |

0167:00416322  JMP      004164E0                                                               |

0167:00416327  ADD      EBX,0315                                                               |

0167:0041632D  MOVSX    EDX,BYTE [EDI+ECX]                                                     |

0167:00416331  TEST     BYTE [EDX+00444089],08                                                 |

0167:00416338  JZ       NEAR 004164E0                                                          |

0167:0041633E  ADD      EBX,BYTE +1B                                                           |

0167:00416341  JMP      004164E0                                                               |

0167:00416346  ADD      EBX,BYTE +17                                                           |

0167:00416349  MOVSX    EAX,BYTE [EDI+ECX]                                                     |

0167:0041634D  TEST     BYTE [EAX+00444089],08                                                 |

0167:00416354  JZ       NEAR 004164E0                                                          |

0167:0041635A  ADD      EBX,129C                                                               |

0167:00416360  JMP      004164E0                                                               |

0167:00416365  ADD      EBX,04DD                                                               |

0167:0041636B  MOVSX    EDX,BYTE [EDI+ECX]                                                     |

0167:0041636F  TEST     BYTE [EDX+00444089],08                                                 |

0167:00416376  JZ       NEAR 004164E0                                                          |

0167:0041637C  ADD      EBX,0137                                                               |

0167:00416382  JMP      004164E0                                                               |

0167:00416387  ADD      EBX,02D4                                                               |

0167:0041638D  MOVSX    EAX,BYTE [EDI+ECX]                                                     |

0167:00416391  TEST     BYTE [EAX+00444089],08                                                 |

0167:00416398  JZ       NEAR 004164E0                                                          |

0167:0041639E  ADD      EBX,BYTE +04                                                           |

0167:004163A1  JMP      004164E0                                                               |

0167:004163A6  ADD      EBX,0340                                                               |

0167:004163AC  MOVSX    EDX,BYTE [EDI+ECX]                                                     |

0167:004163B0  TEST     BYTE [EDX+00444089],08                                                 |

0167:004163B7  JZ       NEAR 004164E0                                                          |

0167:004163BD  ADD      EBX,06DF                                                               |

0167:004163C3  JMP      004164E0                                                               |

0167:004163C8  ADD      EBX,BYTE +12                                                           |

0167:004163CB  MOVSX    EAX,BYTE [EDI+ECX]                                                     |

0167:004163CF  TEST     BYTE [EAX+00444089],08                                                 |

0167:004163D6  JZ       NEAR 004164E0                                                          |

0167:004163DC  ADD      EBX,05BE                                                               |

0167:004163E2  JMP      004164E0                                                               |

0167:004163E7  ADD      EBX,16DA                                                               |

0167:004163ED  MOVSX    EDX,BYTE [EDI+ECX]                                                     |

0167:004163F1  TEST     BYTE [EDX+00444089],08                                                 |

0167:004163F8  JZ       NEAR 004164E0                                                          |

0167:004163FE  ADD      EBX,01BE                                                               |

0167:00416404  JMP      004164E0                                                               |

0167:00416409  ADD      EBX,BYTE +22                                                           |

0167:0041640C  MOVSX    EAX,BYTE [EDI+ECX]                                                     |

0167:00416410  TEST     BYTE [EAX+00444089],08                                                 |

0167:00416417  JZ       NEAR 004164E0                                                          |

0167:0041641D  ADD      EBX,BYTE +16                                                           |

0167:00416420  JMP      004164E0                                                               |

0167:00416425  ADD      EBX,BYTE +44                                                           |

0167:00416428  MOVSX    EDX,BYTE [EDI+ECX]                                                     |

0167:0041642C  TEST     BYTE [EDX+00444089],08                                                 |

0167:00416433  JZ       NEAR 004164E0                                                          |

0167:00416439  ADD      EBX,026A                                                               |

0167:0041643F  JMP      004164E0                                                               |

0167:00416444  ADD      EBX,029A                                                               |

0167:0041644A  MOVSX    EAX,BYTE [EDI+ECX]                                                     |

0167:0041644E  TEST     BYTE [EAX+00444089],08                                                 |

0167:00416455  JZ       NEAR 004164E0                                                          |

0167:0041645B  ADD      EBX,3039                                                               |

0167:00416461  JMP      SHORT 004164E0                                                         |

0167:00416463  ADD      EBX,0185                                                               |

0167:00416469  MOVSX    EDX,BYTE [EDI+ECX]                                                     |

0167:0041646D  TEST     BYTE [EDX+00444089],08                                                 |

0167:00416474  JZ       004164E0                                                               |

0167:00416476  ADD      EBX,1272                                                               |

0167:0041647C  JMP      SHORT 004164E0                                                         |

0167:0041647E  ADD      EBX,0271                                                               |

0167:00416484  MOVSX    EAX,BYTE [EDI+ECX]                                                     |

0167:00416488  TEST     BYTE [EAX+00444089],08                                                 |

0167:0041648F  JZ       004164E0                                                               |

0167:00416491  ADD      EBX,BA                                                                 |

0167:00416497  JMP      SHORT 004164E0                                                         |

0167:00416499  ADD      EBX,014D                                                               |

0167:0041649F  MOVSX    EDX,BYTE [EDI+ECX]                                                     |

0167:004164A3  TEST     BYTE [EDX+00444089],08                                                 |

0167:004164AA  JZ       004164E0                                                               |

0167:004164AC  ADD      EBX,0285                                                               |

0167:004164B2  JMP      SHORT 004164E0                                                         |

0167:004164B4  ADD      EBX,2342                                                               |
```



```
0167:004164BA  MOVSX   EAX,BYTE [EDI+ECX]                |

0167:004164BE  TEST    BYTE [EAX+00444089],08            |

0167:004164C5  JZ      004164E0                           |

0167:004164C7  ADD     EBX,B0                             |

0167:004164CD  JMP     SHORT 004164E0                      |

0167:004164CF  INC     EBX                                  |

0167:004164D0  MOVSX   EDX,BYTE [EDI+ECX]                |

0167:004164D4  TEST    BYTE [EDX+00444089],08            |

0167:004164DB  JZ      004164E0                           |

0167:004164DD  ADD     EBX,BYTE +03                    ----------------------|

0167:004164E0  TEST    ESI,ESI             <---判断计算注册码的第几部分,

0167:004164E2  JNZ     004164F5                  从而转向不同的运算,见后

0167:004164E4  MOV     EAX,EBX

0167:004164E6  PUSH    ECX

0167:004164E7  MOV     ECX,03                  <---ECX=3

0167:004164EC  XOR     EDX,EDX

0167:004164EE  DIV     ECX                   <---EAX=EAX/ECX

0167:004164F0  POP     ECX                   <---恢复 ECX 值

0167:004164F1  ADD     EBX,EAX               <---EBX=EBX+EAX

0167:004164F3  JMP     SHORT 0041651B

0167:004164F5  CMP     ESI,BYTE +01          <---计算注册码第二部分会走到这里|

0167:004164F8  JNZ     00416502                           |

0167:004164FA  MOV     EAX,EBX                          |

0167:004164FC  SHR     EAX,1                            |

0167:004164FE  ADD     EBX,EAX                          |

0167:00416500  JMP     SHORT 0041651B          <------------------------------|

0167:00416502  CMP     ESI,BYTE +02          <---计算注册码第三部分会走到这里|

0167:00416505  JNZ     0041651B                           |

0167:00416507  MOVSX   EDX,BYTE [EDI+01]                |

0167:0041650B  TEST    BYTE [EDX+00444089],08           |

0167:00416512  JZ      0041651B                           |

0167:00416514  MOV     EAX,EBX                          |

0167:00416516  SHR     EAX,02                           |

0167:00416519  ADD     EBX,EAX               <------------------------------|

0167:0041651B  INC     ECX                   <---ECX 加 1

0167:0041651C  MOV     EDX,[EBP-04]

0167:0041651F  CMP     ECX,EDX                 <---比较是否取完用户名

0167:00416521  JL      NEAR 0041606B

0167:00416527  MOV     EAX,EBX                 <---取运算后的最终值存入 EAX 中,

0167:00416529  POP     EDI                           即正确注册码的 16 进制值

0167:0041652A  POP     ESI

0167:0041652B  POP     EBX

0167:0041652C  POP     ECX

0167:0041652D  POP     EBP

0167:0041652E  RET
```

唉！写到这头都晕了，还是没能把这个软件详细分析出来，正如前面所看到的 `0167:00415DF2` 处的判断是判断程序运行的次数，很显然是小于 `200` 次，所以在这个地方肯定会跳，也即无法注册。猜想可能在此之前已进行了注册验证，但我功力有限，跟了几次还是一样，实在不想跟了，还请高手跟一下看看！不过根据作者的说明好像这是个演示版，因为还有功能限制，我不确定作者是否将这部分功能做在程序中。

程序中运用查表法对用户名进行有关运算后得到注册码的值，我实在不知该如何描述，各位自己看看吧！呵呵头都晕了，如果看不明白是我写得不好，请见谅！

我把 `0167:00415DF7` 处改为两个 `nop`，然后填入用户名:`ShenGe` 和注册码:`73348-109543-59405`，注册成功！

嘿嘿！别高兴得太早，等到你运行到 `151` 次时你就会看到那个讨厌的对话框出来了，不能运行程序了。前面已经讲过，`[0046B12C]` 中存放的为程序运行的次数，下 `BPM 46B12C`，重新运行程序，你会很快发现如下代码：

```
* Possible Reference to Dialog: IDD_MATRIX_EDIT, CONTROL_ID:0096, ""

:00401121 3D96000000           cmp eax, 00000096      <---判断是否运行超过了 150 次
:00401126 7628                 jbe 00401150           <---改为 jmp 00401150
:00401128 8B152CB14600         mov edx, dword ptr [0046B12C]
:0040112E 81FAC8000000         cmp edx, 000000C8      <---判断是否运行超过了 200 次
:00401134 731A                 jnb 00401150           <---上面改过了此处就不用改了
:00401136 6A40                 push 00000040
:00401138 68FEB04600           push 0046B0FE

* Possible StringData Ref from Data Obj ->"Thank you for using Access Image "

                                     ->"software.
You were already accessed "

                                     ->"this software more then 150 times.
If "

                                     ->"you want to continue using this "

                                     ->"software you must
buy it for $18 "

                                     ->"US only."

:0040113D 68B4F54300           push 0043F5B4
:00401142 6A00                 push 00000000

* Reference To: USER32.MessageBoxA, Ord:0000h

:00401144 E82BD50300           Call 0043E674
:00401149 33C0                 xor eax, eax
:0040114B E967020000           jmp 004013B7
```

将上面的`:00401126 7628 jbe 00401150` 改为 `EB28`，再运行一下，OK！又可以用了！但是还有那该死的对话框，我实在不想再跟了，各位有兴趣的话试试看吧！等等，这不是爆破吗？前面那么多岂不是白写了吗？到头来还是爆破，唉，不干了！睡觉去了！

最后还是爆破了，但还有功能限制，高手不妨试试看！本不想贴出来，但老外的算法还是比较有意思，各位如果有兴趣不妨研究一下。水平太低，失误之处还请各位老大指点！

----------------------------------------ShenGe---------------------------------- 爆破点：

1. 改 `0167:00401126 JBE 00401150` 为 `JMP 00401150`

2. 改 `0167:00415DF7 JNZ 00415E78` 为 `NOP NOP`

3. 改 `0167:00415E16 JNZ 00415E78` 为 `JZ 00415E78`

4. 改 `0167:00415E20 JNZ 00415E78` 为 `JZ 00415E78`

5. 改 `0167:00415E2A JNZ 00415E78` 为 `JZ 00415E78` `4dd466d0e1e0cc123afbe504da2462ca`
