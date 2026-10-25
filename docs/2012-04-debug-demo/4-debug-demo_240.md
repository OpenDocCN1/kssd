
# Dynamic Desktop 1.4.2

> 原文：[`www.pediy.com/kssd/pediy02/forum220.htm`](https://www.pediy.com/kssd/pediy02/forum220.htm)

## 菜鸟破解实录之 Dynamic Desktop 1.4.2

- **软件名称**：`Dynamic Desktop`
- **软件版本**：`1.4.2`
- **软件大小**：`1352KB`
- **发布公司**：`http://www.dancingbits.com/`
- **软件简介**：可以动态更换墙纸，支持将桌面图标文本背景透明化。

- **作者**：`xiA Qin`
- **级别**：刚学不久
- **解密日期**：2000 年 8 月 5 日
- **解密工具**：`S-ICE 4.00`
- **破解目的**：学习注册码的破解。(中)
- **说明**：

> 本文是在我的软件破解记录上整理出来的。如若有纰漏，请各位大侠多指教！

## 破解步骤

首先运行 `Dynamic Desktop`，输入注册信息：

- `Code: 12345678901` &任意输入

下指令 `bpx hmemcpy` //下中断点

按 `F5` 回到程序，按确定，这时会被 `Trw2000` 拦截到。

下指令 `bd \*` //屏障中断点

下指令 `pmodule` //直接跳到程序的领空(需要两次)

按 `F10` 来到下面指令：

```
015F:00403476  85C0                TEST      EAX,EAX
015F:00403478  0F8462DDFFFF        JZ        004011E0
015F:0040347E  681CEA4600          PUSH      0046EA1C
015F:00403483  8D85F0FDFFFF        LEA       EAX,[EBP-0210]
015F:00403489  6840EA4600          PUSH      0046EA40
015F:0040348E  50                  PUSH      EAX
015F:0040348F  8D85F8E5FFFF        LEA       EAX,[EBP+FFFFE5F8]
015F:00403495  50                  PUSH      EAX
015F:00403496  8D8584F5FFFF        LEA       EAX,[EBP+FFFFF584]
015F:0040349C  50                  PUSH      EAX
015F:0040349D  E8A73F0000          CALL      00407449               //注册码运算，按 F8 进入。
015F:004034A2  83C414              ADD       ESP,14
015F:004034A5  83F802              CMP       EAX,02
```

此处是使用 `EAX` 当作一个标值。

也就是以 `EAX` 的值来决定注册成功或失败。

当 `EAX=2` 时，注册成功。`EAX=1`,注册失败。(关键，破解的关键)

```
015F:004034A8  7441                JZ        004034EB               //注册码比较
015F:004034AA  0FB605A0FA4600       MOVZX     EAX,BYTE PTR [0046FAA0]
015F:004034B1  6A30                PUSH      30
015F:004034B3  6878854600          PUSH      00468578
015F:004034B8  FF3485AC174600      PUSH      DWORD PTR [EAX*4+004617AC]
015F:004034BF  FF7508              PUSH      DWORD PTR [EBP+08]
015F:004034C2  FF15ACB24400        CALL      [0044B2AC]             //注册失败对话框
015F:004034C8  A0A0FA4600          MOV       AL,[0046FAA0]
015F:004034CD  888584F5FFFF        MOV       [EBP+FFFFF584],AL
015F:004034D3  8D8584F5FFFF        LEA       EAX,[EBP+FFFFF584]
015F:004034D9  50                  PUSH      EAX
```

按 `F8` 进入 `0040349D  CALL 00407449` 来到下面的指令。要按很久哦!

```
015F:00407502  7418                JZ        0040751C
015F:00407504  56                  PUSH      ESI
015F:00407505  E816010000          CALL      00407620
015F:0040750A  59                  POP       ECX
015F:0040750B  B94B144600          MOV       ECX,0046144B
015F:00407510  2BC8                SUB       ECX,EAX
015F:00407512  8A01                MOV       AL,[ECX]
015F:00407514  8806                MOV       [ESI],AL
015F:00407516  46                  INC       ESI
015F:00407517  803E00              CMP       BYTE PTR [ESI],00
015F:0040751A  75E8                JNZ       00407504               (JUMP )
015F:0040751C  80BD00FCFFFF00      CMP       BYTE PTR [EBP-0400],00
015F:00407523  8DB500FCFFFF        LEA       ESI,[EBP-0400]
015F:00407529  8DBD00FCFFFF        LEA       EDI,[EBP-0400]
015F:0040752F  742E                JZ        0040755F
015F:00407531  807E0100            CMP       BYTE PTR [ESI+01],00
015F:00407535  8D5E01              LEA       EBX,[ESI+01]
015F:00407538  7425                JZ        0040755F
015F:0040753A  56                  PUSH      ESI
015F:0040753B  E8E0000000          CALL      00407620
015F:00407540  8BD0                MOV       EDX,EAX
015F:00407542  53                  PUSH      EBX
015F:00407543  C0E204              SHL       DL,04
015F:00407546  895508              MOV       [EBP+08],EDX
015F:00407549  E8D2000000          CALL      00407620
015F:0040754E  59                  POP       ECX
015F:0040754F  59                  POP       ECX
015F:00407550  8B4D08              MOV       ECX,[EBP+08]
015F:00407553  02C8                ADD       CL,AL
015F:00407555  46                  INC       ESI
015F:00407556  46                  INC       ESI
015F:00407557  880F                MOV       [EDI],CL
015F:00407559  47                  INC       EDI
015F:0040755A  803E00              CMP       BYTE PTR [ESI],00
015F:0040755D  75D2                JNZ       00407531
015F:0040755F  802700              AND       BYTE PTR [EDI],00
015F:00407562  6A01                PUSH      01                  <-修改
015F:00407564  58                  POP       EAX
015F:00407565  5F                  POP       EDI
015F:00407566  388500FCFFFF        CMP       [EBP-0400],AL
015F:0040756C  5B                  POP       EBX
015F:0040756D  753B                JNZ       004075AA
015F:0040756F  8B9501FCFFFF        MOV       EDX,[EBP-03FF]
015F:00407575  6A05                PUSH      05
015F:00407577  59                  POP       ECX
015F:00407578  0FB6B40D00FCFFFF    MOVZX     ESI,BYTE PTR [ECX+EBP-0400]
015F:00407580  2BD6                SUB       EDX,ESI
015F:00407582  41                  INC       ECX
015F:00407583  83F90D              CMP       ECX,0D
015F:00407586  7CF0                JL        00407578
015F:00407588  85D2                TEST      EDX,EDX
015F:0040758A  0F858D000000        JNZ       0040761D
015F:00407590  8B4514              MOV       EAX,[EBP+14]
015F:00407593  8B8D05FCFFFF        MOV       ECX,[EBP-03FB]
015F:00407599  8908                MOV       [EAX],ECX
015F:0040759B  8B4518              MOV       EAX,[EBP+18]
015F:0040759E  8B8D09FCFFFF        MOV       ECX,[EBP-03F7]
015F:004075A4  8908                MOV       [EAX],ECX
015F:004075A6  33C0                XOR       EAX,EAX
015F:004075A8  EB73                JMP       0040761D
015F:004075AA  80BD00FCFFFF02      CMP       BYTE PTR [EBP-0400],02
015F:004075B1  756A                JNZ       0040761D
015F:004075B3  80BD05FCFFFF00      CMP       BYTE PTR [EBP-03FB],00
015F:004075BA  8BB501FCFFFF        MOV       ESI,[EBP-03FF]
015F:004075C0  8D8D05FCFFFF        LEA       ECX,[EBP-03FB]
015F:004075C6  7413                JZ        004075DB
015F:004075C8  8A9505FCFFFF        MOV       DL,[EBP-03FB]
015F:004075CE  0FB6D2              MOVZX     EDX,DL
015F:004075D1  2BF2                SUB       ESI,EDX
015F:004075D3  8A5101              MOV       DL,[ECX+01]
015F:004075D6  41                  INC       ECX
015F:004075D7  84D2                TEST      DL,DL
015F:004075D9  75F3                JNZ       004075CE
015F:004075DB  85F6                TEST      ESI,ESI
015F:004075DD  753E                JNZ       0040761D
015F:004075DF  8B7101              MOV       ESI,[ECX+01]
015F:004075E2  41                  INC       ECX
015F:004075E3  83C104              ADD       ECX,04
015F:004075E6  8A11                MOV       DL,[ECX]
015F:004075E8  84D2                TEST      DL,DL
015F:004075EA  7408                JZ        004075F4
015F:004075EC  0FB6D2              MOVZX     EDX,DL
015F:004075EF  2BF2                SUB       ESI,EDX
015F:004075F1  41                  INC       ECX
015F:004075F2  EBF2                JMP       004075E6
015F:004075F4  85F6                TEST      ESI,ESI
015F:004075F6  7525                JNZ       0040761D
015F:004075F8  8D8505FCFFFF        LEA       EAX,[EBP-03FB]
015F:004075FE  50                  PUSH      EAX
015F:004075FF  FF750C              PUSH      DWORD PTR [EBP+0C]
015F:00407602  E8D0FBFFFF          CALL      004071D7
015F:00407607  8D84050AFCFFFF      LEA       EAX,[EAX+EBP-03F6]
```



```
015F:0040760E  50                       PUSH      EAX   

015F:0040760F  FF7510                  PUSH      DWORD PTR [EBP+10]   

015F:00407612  E8C0FBFFFF              CALL      004071D7   

015F:00407617  83C410                  ADD       ESP,10   

015F:0040761A  6A02                    PUSH      02   

015F:0040761C  58                      POP       EAX   

015F:0040761D  5E                      POP       ESI   

015F:0040761E  C9                      LEAVE   

015F:0040761F  C3                      RET   
```

............................

如果要使`EAX=2`，从上面可以看出，

将`00407562 6A01                 PUSH      01`
改`00407562 6A02                 PUSH      02`

就可以注册。

整里一下，用`Ultraedt`打开`Dynamic Desktop.exe`

找到`6A 01 58 5F 38 85`
改成`6A 02 58 5F 38 85`

保存修改文件，重新运行。

噢！已经注册了。

嗨！只是看不明白注册码、每次注册码都不相同，看来我太暴力了。

### 第二种破解方法：

1、
找到`004075B1 756A               JNZ       0040761D`
改成`004075B1 746A               JZ        0040761D`

2、
找到`004075DD 753E               JNZ       0040761D`
改成`004075DD 743E               JZ        0040761D`

3、
找到`004075F6 7525               JNZ       0040761D`
改成`004075F6 7425               JZ        0040761D`

同样用`Ultraedt`打开`Dynamic Desktop.exe`

1、
找到`75 6A 80 BD 05 FC FF`
改成`74 6A 80 BD 05 FC FF`

2、
找到`75 3E 8B 71 01`
改成`74 3E 8B 71 01`

3、
找到`75 25 8D 85 05 FC FF FF`
改成`74 25 8D 85 05 FC FF FF`

注册信息在注册表中

`[HKEY_CURRENT_USER\Software\Dancing Bits\Dynamic Desktop]`

`"szRegistered"="1234567890"   4dd466d0e1e0cc123afbe504da2462ca`
