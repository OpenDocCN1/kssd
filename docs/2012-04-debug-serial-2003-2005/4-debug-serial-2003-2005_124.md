
# E 族百变桌面 6.0

> 原文：`www.pediy.com/kssd/pediy06/pediy6161.htm` (https://www.pediy.com/kssd/pediy06/pediy6161.htm)

## 软件信息

- **软件名称**：E 族百变桌面 6.0
- **软件简介**：e 族百变桌面 6.0
- **软件大小**：1386KB
- **软件语言**：简体中文
- **软件类别**：国产软件/共享版/布景保护
- **运行环境**：Win9x/Me/NT/2000/XP
- **加入时间**：2003-6-30 16:56:23
- **下载次数**：642
- **软件评级**：
- **联系人**：未知
- **开发商**：Home Page
- **软件介绍**：提供 25 种变换桌面的方式，让您的桌面焕然一新。操作简单，无需费力学习。支持多种 Internet 流行图片格式。将壁纸文件打包，方便存储、转发。将壁纸包展开，还原图片文件。e 族网真情奉献，免费下载，免费使用。e 族百变桌面，桌面从此精彩！
- **下载地址**：`www.onlinedown.net/index.htm` (http://www.onlinedown.net/index.htm)

## 破解说明

- **破解作者**：`yzez[DFCG]`
- **破解工具**：`DeDE3.5`、`Ollydbg1.09`
- **破解目的**：初次学破解，交流经验。研究注册算法，提高水平。

## 详细过程

1. 用 `FI` 检查是 `Delphi` 程序，所以用 `DeDE3.5` 反汇编，选过程，在出现的窗口中选择 `frmbuy--bnNextClick`, 找到关键点：

```
  004CCA4A    E8CDC2F3FF              call     00408D1C
004CCA4F    8945F0                mov      [ebp-$10], eax
004CCA52    8955F4                mov      [ebp-$0C], edx
004CCA55    8B45FC                mov      eax, [ebp-$04]
004CCA58    FFB0C4030000          push     dword ptr [eax+$03C4]
004CCA5E    FFB0C0030000          push     dword ptr [eax+$03C0]
004CCA64    FF75F4                push     dword ptr [ebp-$0C]
004CCA67    FF75F0                push     dword ptr [ebp-$10]
004CCA6A    B001                  mov      al, $01 

|
004CCA6C    E85FF3FFFF            call     004CBDD0---------这是我们要找的关键 CALL，这一步我不想详细介绍，这篇破文主要研究算法。
```

2. 退出 `DeDE3.5`，用 `Ollydbg` 调试：

```
004CCA6A    .  B0 01           MOV       AL, 1
004CCA6C    .  E8 5FF3FFFF    CALL      EPAPER.004CBDD0---------------------------在此按 F2 键设断，按 F9 运行程序，填入注册信息后，
                                                                                     点确定，被断，按 F7 跟进这个 CALL
004CCA71    .  8B15 90AF4D00 MOV       EDX, DWORD PTR DS:[4DAF90]       ;  EPAPER.004DCD94
004CCA77    .  8802           MOV       BYTE PTR DS:[EDX], AL
004CCA79    .  A1 90AF4D00   MOV       EAX, DWORD PTR DS:[4DAF90]
004CCA7E    .  8038 00       CMP       BYTE PTR DS:[EAX], 0
004CCA81    .  74 4C         JE        SHORT EPAPER.004CCACF
004CCA83    .  8B45 FC       MOV       EAX, DWORD PTR SS:[EBP-4]
004CCA86    .  C780 CC030000>MOV       DWORD PTR DS:[EAX+3CC], 4
004CCA90    .  FF75 F4       PUSH      DWORD PTR SS:[EBP-C]            ; /Arg2 = 00000000
004CCA93    .  FF75 F0       PUSH      DWORD PTR SS:[EBP-10]           ; |Arg1 = 499602D2
004CCA96    .  8D45 C0       LEA       EAX, DWORD PTR SS:[EBP-40]      ; |
004CCA99    .  E8 BAC1F3FF   CALL      EPAPER.00408C58                 ; \EPAPER.00408C58
004CCA9E    .  8B55 C0       MOV       EDX, DWORD PTR SS:[EBP-40]
004CCAA1    .  8B45 FC       MOV       EAX, DWORD PTR SS:[EBP-4]
004CCAA4    .  8B80 44030000 MOV       EAX, DWORD PTR DS:[EAX+344]
004CCAAA    .  E8 25F3F8FF   CALL      EPAPER.0045BDD4
```

--------------------追进 `004CCA6C . E8 5FF3FFFF CALL EPAPER.004CBDD0` 我们来到这里：

```
004CBDD0  /$  55             PUSH      EBP\----------------------停在这一行
004CBDD1  |.  8BEC          MOV       EBP, ESP
004CBDD3  |.  83C4 F8       ADD       ESP, -8
004CBDD6  |.  53            PUSH      EBX
004CBDD7  |.  56            PUSH      ESI                       ;  ePaper.0049F0E8
004CBDD8  |.  33D2          XOR       EDX, EDX
004CBDDA  |.  8955 F8       MOV       [LOCAL.2], EDX
004CBDDD  |.  8845 FF       MOV       BYTE PTR SS:[EBP-1], AL
004CBDE0  |.  33C0          XOR       EAX, EAX
004CBDE2  |.  55            PUSH      EBP
004CBDE3  |.  68 67BE4C00   PUSH      ePaper.004CBE67
004CBDE8  |.  64:FF30       PUSH      DWORD PTR FS:[EAX]
004CBDEB  |.  64:8920       MOV       DWORD PTR FS:[EAX], ESP
004CBDEE  |.  33DB          XOR       EBX, EBX
004CBDF0  |.  FF75 14       PUSH      [ARG.4]                    ;  ePaper.0049F0CF
004CBDF3  |.  FF75 10       PUSH      [ARG.3]
004CBDF6  |.  E8 B1000000   CALL      ePaper.004CBEAC------按 F7 再追进这个 CALL.为什么？看看下面的代码，这是很经典的比较语句
004CBDFB  |.  3B55 0C       CMP       EDX, [ARG.2]                ;  ePaper.0049F10B
004CBDFE  |.  75 07         JNZ       SHORT ePaper.004CBE07
004CBE00  |.  3B45 08       CMP       EAX, [ARG.1]
004CBE03  |.  75 02         JNZ       SHORT ePaper.004CBE07
004CBE05  |.  B3 01         MOV       BL, 1
004CBE07  |>  807D FF 00    CMP       BYTE PTR SS:[EBP-1], 0
004CBE0B  |.  74 44         JE        SHORT ePaper.004CBE51
```

-----------------追进 `004CBDF6 |. E8 B1000000 CALL ePaper.004CBEAC`，我们又会来到这里：

```
004CBEA4    .  52 65 67 43 6>ASCII     "RegCode",0――注册码，可见我们找到了地方
004CBEAC  /$  55             PUSH      EBP\---------停在这一行
004CBEAD  |.  8BEC          MOV       EBP, ESP
004CBEAF  |.  83C4 E0       ADD       ESP, -20
004CBEB2  |.  53            PUSH      EBX
004CBEB3  |.  56            PUSH      ESI                       ;  ePaper.0049F0E8
004CBEB4  |.  33C0          XOR       EAX, EAX
004CBEB6  |.  8945 E0       MOV       [LOCAL.8], EAX
004CBEB9  |.  8945 EC       MOV       [LOCAL.5], EAX
004CBEBC  |.  8945 E8       MOV       [LOCAL.6], EAX
004CBEBF  |.  8945 E4       MOV       [LOCAL.7], EAX
004CBEC2  |.  33C0          XOR       EAX, EAX
004CBEC4  |.  55            PUSH      EBP
004CBEC5  |.  68 D9BF4C00   PUSH      ePaper.004CBFD9
004CBECA  |.  64:FF30       PUSH      DWORD PTR FS:[EAX]
004CBECD  |.  64:8920       MOV       DWORD PTR FS:[EAX], ESP
004CBED0  |.  FF75 0C       PUSH      [ARG.2]                   ; /Arg2 = 00000000
004CBED3  |.  FF75 08       PUSH      [ARG.1]                   ; |Arg1 = 499602D2
004CBED6  |.  8D45 EC       LEA       EAX, [LOCAL.5]            ; |
004CBED9  |.  E8 7ACDF3FF   CALL      ePaper.00408C58           ; \ePaper.00408C58
004CBEDE  |.  8B45 08       MOV       EAX, [ARG.1]-―――――把机器码的十六进制值赋给 EAX
004CBEE1  |.  8945 F0       MOV       [LOCAL.4], EAX
004CBEE4  |.  8B45 0C       MOV       EAX, [ARG.2]
004CBEE7  |.  8945 F4       MOV       [LOCAL.3], EAX
004CBEEA  |.  8B45 EC       MOV       EAX, [LOCAL.5]
004CBEED  |.  E8 F288F3FF   CALL      ePaper.004047E4
004CBEF2  |.  8BD8          MOV       EBX, EAX
004CBEF4  |.  85DB          TEST      EBX, EBX
004CBEF6  |.  7E 56         JLE       SHORT ePaper.004CBF4E
004CBEF8  |.  BE 01000000   MOV       ESI, 1
004CBEFD  |>  8D45 E0       /LEA      EAX, [LOCAL.8]--从这里开始是一个循环,一共循环 10 次,因为机器码是:10 位
004CBF00  |.  8B55 EC       |MOV      EDX, [LOCAL.5]-----赋 EDX 为 0
004CBF03  |.  8A5432 FF    |MOV      DL, BYTE PTR DS:[EDX+ESI-1]
004CBF07  |.  E8 0088F3FF   |CALL     ePaper.0040470C
004CBF0C  |.  8B45 E0       |MOV      EAX, [LOCAL.8]
004CBF0F  |.  E8 08E5F3FF   |CALL     ePaper.0040A41C
004CBF14  |.  DB2D F0BF4C00 |FLD      TBYTE PTR DS:[4CBFF0]----------------装入实数:4CBFF0 的值是:3.1415926535897932380
004CBF1A  |.  DEC9          |FMULP     ST(1), ST\--------------ST(1)*ST 送入 ST,然后出栈,ST 是 3.1415926535897932380,ST(1)是机器码按
位取数,如第一次循环,取第一位是:3,所以执行运算后其结果是:3.1415926535897932380*3.0000000000000000000=9.4247779607693793480
004CBF1C  |.  E8 4F6CF3FF   |CALL     ePaper.00402B70----------这个 CALL 干什么？
```

\--------------下面是 `004CBF1C |. E8 4F6CF3FF |CALL ePaper.00402B70` 这个 `CALL` 的代码

```
00402B70  /$  83EC 08       SUB       ESP, 8
00402B73  |.  DF3C24        FISTP     QWORD PTR SS:[ESP]--------将 ST 以整数形式保存,注意四舍五入，如：9.4247779607693793480 保留
                                                                   整数是 9。
00402B76  |.  9B            WAIT
00402B77  |.  58            POP       EAX                         ;  ePaper.00404719
00402B78  |.  5A            POP       EDX                         ;  ePaper.00404719
```

\------------------------------------------------------------------――――――――――

```
004CBF21  |.  3345 F0       |XOR       EAX, [LOCAL.4]  ;  ePaper.0049F0E8----------------第一次循环是把机器码的十六进制值与 EAX
```



异或送入 `EAX`，`EAX` 是多少? 就是上面浮点运算取得的整数值(转为十六进制)。如：第一次循环就是：`E62C6708` XOR `9` = `E62C6701`

(`E62C6708` 是我的机器码 `3861669640` 的十六进制值)

```
004CBF24  |.  3355 F4       |XOR     EDX, [LOCAL.3]----------------------EDX 清零  
004CBF27  |.  81F0 A09EF21B |XOR     EAX, 1BF29EA0---------------------E62C6701 XOR 1BF29EA0＝FDDEF9A1  
004CBF2D  |.  81F2 00000000 |XOR     EDX, 0  
004CBF33  |.  85D2          |TEST    EDX, EDX  
004CBF35  |.  7D 07         |JGE     SHORT ePaper.004CBF3E－－大于等于转移  
004CBF37  |.  F7D8          |NEG     EAX  
004CBF39  |.  83D2 00       |ADC     EDX, 0-------------带进位+0  
004CBF3C  |.  F7DA          |NEG     EDX  
004CBF3E  |>  0345 F0       |ADD     EAX, [LOCAL.4]  ;  ePaper.0049F0E8－－－－------FDDEF9A1+E62C6708=E40B60A9  
004CBF41  |.  1355 F4       |ADC     EDX, [LOCAL.3]－－带进位加：EDX+EDX=1+0=1 赋给 Arg2  
004CBF44  |.  8945 F0       |MOV     [LOCAL.4], EAX－－E40B60A9 替换 E62C6708 赋值给 Arg1  
004CBF47  |.  8955 F4       |MOV     [LOCAL.3], EDX  
004CBF4A  |.  46           |INC     ESI     ;  ePaper.0049F0E8------调整  
004CBF4B  |.  4B           |DEC     EBX－－-------------------------EBX 减一，  
004CBF4C  |.^ 75 AF        \\JNZ    SHORT ePaper.004CBEFD－－完了吗？没完继续．跳回前面重新来过!  
004CBF4E  |>  FF75 F4      PUSH    [LOCAL.3]                  ; /Arg2 = 000003F7  
004CBF51  |.  FF75 F0      PUSH    [LOCAL.4]                  ; |Arg1 = 1C02DF42 这是十次循环计算后的 Arg1 和 Arg2 的值  
004CBF54  |.  8D45 E8      LEA     EAX, [LOCAL.6]             ; |  
004CBF57  |.  E8 FCCCF3FF  CALL    ePaper.00408C58  ; \\ePaper.00408C58----------------此 CALL 把上述运算得到的 Arg1、Arg2 连接  
                                起来,就是;3F71C02DF42,再转化为十进制值是:4359861755714  
004CBF5C  |.  8B45 E8      MOV     EAX, [LOCAL.6]             ;  ePaper.004CBE67  
004CBF5F  |.  E8 8088F3FF  CALL    ePaper.004047E4  
004CBF64  |.  8BD8         MOV     EBX, EAX----------------------取位数,EAX 是 0D(即 13)，上面转化的数值就是十三位数  
004CBF66  |.  8B45 EC      MOV     EAX, [LOCAL.5]  
004CBF69  |.  E8 7688F3FF  CALL    ePaper.004047E4  
004CBF6E  |.  3BD8         CMP     EBX, EAX----------------------比较位数:EAX 为 0A(10),EBX 为 0D(13)  
004CBF70  |.  7D 08        JGE     SHORT ePaper.004CBF7A---------大于等于转移  
004CBF72  |.  8B45 E8      MOV     EAX, [LOCAL.6]             ;  ePaper.004CBE67  
004CBF75  |.  E8 DA91F6FF  CALL    ePaper.00435154  
004CBF7A  |>  8B45 E8      MOV     EAX, [LOCAL.6] ----------------转移到此 ,赋注册码位数为 0A 即 10 位                  
004CBF7D  |.  E8 6288F3FF  CALL    ePaper.004047E4  
004CBF82  |.  8BD8         MOV     EBX, EAX  
004CBF84  |.  8B45 EC      MOV     EAX, [LOCAL.5]  
004CBF87  |.  E8 5888F3FF  CALL    ePaper.004047E4  
004CBF8C  |.  2BD8         SUB     EBX, EAX----------------------这里调整位数:EBX=0D-0A  
004CBF8E  |.  43           INC     EBX---------------------------EBX+1  
004CBF8F  |>  8D45 E4      /LEA    EAX, [LOCAL.7]  
004CBF92  |.  50           |PUSH    EAX  
004CBF93  |.  8B45 EC      |MOV     EAX, [LOCAL.5]  
004CBF96  |.  E8 4988F3FF  |CALL    ePaper.004047E4  
004CBF9B  |.  8BC8         |MOV     ECX, EAX  
004CBF9D  |.  8BD3         |MOV     EDX, EBX  
004CBF9F  |.  8B45 E8      |MOV     EAX, [LOCAL.6]  ;  ePaper.004CBE67-------------注册码只取十位  
004CBFA2  |.  E8 9D8AF3FF  |CALL    ePaper.00404A44  
004CBFA7  |.  4B           |DEC     EBX---------------------------------------------EBX 减一就是 3  
004CBFA8  |.  8B45 E4      |MOV     EAX, [LOCAL.7]------把 4359861755714 去掉前三位:9861755714 赋给 EAX 这就是我们所需要的注册码！  
004CBFAB  |.  8038 30      |CMP     BYTE PTR DS:[EAX], 30  
004CBFAE  |.^ 74 DF        \\JE     SHORT ePaper.004CBF8F  
004CBFB0  |.  8B45 E4      MOV     EAX, [LOCAL.7]  
004CBFB3  |.  E8 64CDF3FF  CALL    ePaper.00408D1C  
004CBFB8  |.  8945 F8      MOV     [LOCAL.2], EAX  
004CBFBB  |.  8955 FC      MOV     [LOCAL.1], EDX  
004CBFBE  |.  33C0         XOR     EAX, EAX  
004CBFC0  |.  5A           POP     EDX                   ;  0012F7B0  
004CBFC1  |.  59           POP     ECX                   ;  0012F7B0  
004CBFC2  |.  59           POP     ECX                   ;  0012F7B0  
004CBFC3  |.  64:8910      MOV     DWORD PTR FS:[EAX], EDX  
004CBFC6  |.  68 E0BF4C00  PUSH    ePaper.004CBFE0  
004CBFCB  |>  8D45 E0      LEA     EAX, [LOCAL.8]  
004CBFCE  |.  BA 04000000  MOV     EDX, 4  
004CBFD3  |.  E8 7085F3FF  CALL    ePaper.00404548  
004CBFD8  \\   C3          RETN  
```

## 算法小结

这是根据软件随机产生的十位机器码进行计算，得出正确的注册码。计算分两步：

1.  机器码经过按位运算产生 `Arg1` 和 `Arg2` 两组值。`Arg1` 的运算是这样：首先按位取值，取出注册码的一位值乘 `3.1415926535897932380`，把结果取整数(注意四舍五入)赋给 `EAX`，然后机器码的十六进制值异或 `EAX`，结果存 `EAX`(注意这是第一次循环)，然后再与常数：`1BF29EA0` 相异或，结果加上机器码的十六进制值，完成第一次循环，把这一运算结果保存，进行第二次循环。语言太差，总觉得说不清楚，下面举例说明：
    *   第一次循环：`Arg1(1)` 我假定为 `Arg1(1)`，`Arg1(1)` = 机器码 XOR `3.1415926535897932380 * 3.0000000000000000000` 的整数值的十六进制(9) XOR `1BF29EA0` + 机器码 = `FDDEF9A1 + E62C6708 = E40B60A9`
    *   第二次循环：`Arg1(2)` = `Arg1(1)` XOR `3.1415926535897932380 * 8.0000000000000000000` 的整数值的十六进制(19) XOR `1BF29EA0` + `Arg1(1)` = `E4055EB9`
    *   第十次循环：`Arg1(10)` = `Arg1(9)` XOR `3.1415926535897932380 * 0.0000000000000000000` 的整数值的十六进制(0) XOR `1BF29EA0` + `Arg1(9)` = `1C02DF42`，这就是：`Arg1` 的最后结果。
    *   `Arg2` 的运算：`Arg2` =（位数－1）+1+（位数－1）(这也是第一次的循环), 后面的循环: `Arg2(n)` = `Arg2(n-1)` + 1 + `Arg2(n-1)`, 如第一次循环：
        *   `Arg2(1)` = 0（1－1）＋1＋0（1－1）＝1；
        *   `Arg2(2)` = 1＋1＋1＝3
        *   `Arg2(10)` = `1FB` ＋1＋`1FB` ＝`3F7`，这是 `Arg2` 的值。

2.  这是第二步运算，把 `Arg1` 和 `Arg2` 连接起来，构成一个十六进制值，即 `Arg2Arg1 = 3F71C02DF42`, 转化为十进制值是：`4359861755714`，注册码是 10 位，把前面 3 位去掉，就是真正的注册码，如我的注册码是：`9861755714`。

`4dd466d0e1e0cc123afbe504da2462ca`
