
```
001B:00437C8E  43                  INC       EBX

001B:00437C8F  0023                ADD       [EBX],AH

001B:00437C91  D18A0688078A        ROR       DWORD PTR [EDX+8A078806],1

001B:00437C97  46                  INC       ESI

001B:00437C98  018847018A46        ADD       [EAX+468A0147],ECX

001B:00437C9E  02C1                ADD       AL,CL

001B:00437CA0  E902884702          JMP       028B04A7

001B:00437CA5  83C603              ADD       ESI,03

001B:00437CA8  83C703              ADD       EDI,03

001B:00437CAB  83F908              CMP       ECX,08

001B:00437CAE  72CC                JB        00437C7C

001B:00437CB0  F3A5                REPZ MOVSD

001B:00437CB2  FF2495687D4300      JMP       [EDX*4+00437D68]

001B:00437CB9  8D4900              LEA       ECX,[ECX+00]

001B:00437CBC  23D1                AND       EDX,ECX

001B:00437CBE  8A06                MOV       AL,[ESI]

001B:00437CC0  8807                MOV       [EDI],AL

001B:00437CC2  8A4601              MOV       AL,[ESI+01]

001B:00437CC5  C1E902              SHR       ECX,02

001B:00437CC8  884701              MOV       [EDI+01],AL

001B:00437CCB  83C602              ADD       ESI,02

001B:00437CCE  83C702              ADD       EDI,02

001B:00437CD1  83F908              CMP       ECX,08

001B:00437CD4  72A6                JB        00437C7C

001B:00437CD6  F3A5                REPZ MOVSD

001B:00437CD8  FF2495687D4300      JMP       [EDX*4+00437D68]

001B:00437CDF  90                  NOP

001B:00437CE0  23D1                AND       EDX,ECX

001B:00437CE2  8A06                MOV       AL,[ESI]

001B:00437CE4  8807                MOV       [EDI],AL

001B:00437CE6  46                  INC       ESI

001B:00437CE7  C1E902              SHR       ECX,02

001B:00437CEA  47                  INC       EDI

001B:00437CEB  83F908              CMP       ECX,08

001B:00437CEE  728C                JB        00437C7C

001B:00437CF0  F3A5                REPZ MOVSD

001B:00437CF2  FF2495687D4300      JMP       [EDX*4+00437D68]

001B:00437CF9  8D4900              LEA       ECX,[ECX+00]

001B:00437CFC  5F                  POP       EDI

001B:00437CFD  7D43                JGE       00437D42

001B:00437CFF  004C7D43            ADD       [EDI*2+EBP+43],CL

001B:00437D03  00447D43            ADD       [EDI*2+EBP+43],AL

001B:00437D07  003C7D4300347D      ADD       [EDI*2+7D340043],BH

001B:00437D0E  43                  INC       EBX

001B:00437D0F  002C7D4300247D      ADD       [EDI*2+7D240043],CH

001B:00437D16  43                  INC       EBX

001B:00437D17  001C7D43008B44      ADD       [EDI*2+448B0043],BL

001B:00437D1E  8EE4                MOV       FS,SP

001B:00437D20  89448FE4            MOV       [ECX*4+EDI-1C],EAX

001B:00437D24  8B448EE8            MOV       EAX,[ECX*4+ESI-18]

001B:00437D28  89448FE8            MOV       [ECX*4+EDI-18],EAX

001B:00437D2C  8B448EEC            MOV       EAX,[ECX*4+ESI-14]

001B:00437D30  89448FEC            MOV       [ECX*4+EDI-14],EAX

001B:00437D34  8B448EF0            MOV       EAX,[ECX*4+ESI-10]

001B:00437D38  89448FF0            MOV       [ECX*4+EDI-10],EAX

001B:00437D3C  8B448EF4            MOV       EAX,[ECX*4+ESI-0C]

001B:00437D40  89448FF4            MOV       [ECX*4+EDI-0C],EAX

001B:00437D44  8B448EF8            MOV       EAX,[ECX*4+ESI-08]

001B:00437D48  89448FF8            MOV       [ECX*4+EDI-08],EAX

001B:00437D4C  8B448EFC            MOV       EAX,[ECX*4+ESI-04]

001B:00437D50  89448FFC            MOV       [ECX*4+EDI-04],EAX

001B:00437D54  8D048D00000000      LEA       EAX,[ECX*4+00000000]

001B:00437D5B  03F0                ADD       ESI,EAX

001B:00437D5D  03F8                ADD       EDI,EAX

001B:00437D5F  FF2495687D4300      JMP       [EDX*4+00437D68]

001B:00437D66  8BFF                MOV       EDI,EDI

001B:00437D68  787D                JS        00437DE7

001B:00437D6A  43                  INC       EBX

001B:00437D6B  00807D43008C        ADD       [EAX+8C00437D],AL

001B:00437D71  7D43                JGE       00437DB6

001B:00437D73  00A07D43008B        ADD       [EAX+8B00437D],AH

001B:00437D79  45                  INC       EBP

001B:00437D7A  085E5F              OR        [ESI+5F],BL

001B:00437D7D  C9                  LEAVE

001B:00437D7E  C3                  RET
```

---

# 三、破解

也可以用 `W32DASM` 反汇编，寻找正版标志 `[0051041C]`、`限制条数 [0050E478]`，注意所有被写入数据的地方

## 改★1、网上验证破解；

```
001B:00427130  .........

001B:004271F6  E8EAEB0000          CALL      00435DE5        //★★关键★★

001B:004271FB  83C404              ADD       ESP,04

001B:004271FE  83F8FF              CMP       EAX,-01

001B:00427201  0F84AC090000        JZ        00427BB3        //EAX=FFFFFFFF跳到：置标志 3 表示“服务器忙”改为：

001B:00427207  85C0                TEST      EAX,EAX

001B:00427209  7E14                JLE       0042721F        //EAX=0 正常执行

001B:0042720B  83F803              CMP       EAX,03

001B:0042720E  7F0F                JG        0042721F        //EAX>3 正常执行

001B:00427210  C7869400000002000000MOV       DWORD PTR [ESI+00000094],00000002        //标志 2 表示“非法口令”

001B:0042721A  E99E090000          JMP       00427BBD

.........

001B:004272DE  F7D8                NEG       EAX

001B:004272E0  A31C045100          MOV       [0051041C],EAX //★★正版标志位被重置
```

-----------------------------------------------------

### 改★1⑴为：跳过非法口令信息

```
001B:004271FE        33C0         xor eax,eax

001B:00427200        90909090909090        7个 NOP

001B:00427207  85C0                TEST      EAX,EAX
......
```

### 改★1⑵为：标志位被重置为正版！

```
001B:004272DE  33C0                XOR      EAX,EAX

001B:004272E0  A31C045100          MOV       [0051041C],EAX //★★标志位被重置为正版！
```

### 改★1⑶为：网上验证后修改限制数值位

```
001B:0040C2E1  A378E45000          MOV       [0050E478],EAX        //网上验证后修改限制标志位， EAX=0000000A（10），改为：

001B:0040C2E1        9090909090                5个NOP
```

## 改★2、文件VA `[0051041C]` 处存放正版标志数值，改为：`00000000`

## 改★3：`0040849B`  `C7051C04510001000000`    `MOV   DWORD PTR [0051041C],00000001`//正版标志位，此处被重置为 1 ，表示试用版，此处改后，全部条目自动可以选择而且可以编辑。改为：

```
0040849B            C7051C04510000000000        MOV   DWORD PTR [0051041C],00000000
```

----------------------------------------------------------------

## 改★4：破解只有 5 条信息可以发送的限制及试用客户每次只能发送 10 条信息的限制

程序VA `[0050E478]` 存放限制条目数，改为：`0FFFFFFF`

## 改★5：破解最多只有 1000 条信息可以选择

在上述破解后，最多只有 1000 条信息可以选择，因为限制数目被重置；

```
004084B6   B8E8030000                MOV  EAX,000003E8        //再次限制条目为 1000,改为：

004084B6   B8FFFFFF0F                MOV  EAX,0FFFFFFF
```

## 改★6、其他修改：在选择窗口单击鼠标右键，选择查看源文件，发现竟然只是一个普通的网页文件，加了些 javascript 函数程序，研究这个程序，修改  javascript 函数，再复制回原程序（注意字节长度、位置应完全一样），即可破解选择限制

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
