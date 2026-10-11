
## 反汇编代码分析

004058CD  57              PUSH EDI                                    ;三次 PUSH + c H

004058CE  8DB7 71010000   LEA ESI,DWORD PTR DS:[EDI+171]             ;ESI = 40D1B8 正好到了 PETITE 结构

004058D4  8BCE            MOV ECX,ESI

004058D6  2BCF            SUB ECX,EDI

004058D8  F3:AA           REP STOS BYTE PTR ES:[EDI]                  ;内存以 0 填充 执行完毕后 EDI = ESI

004058DA  60              PUSHAD                                      ;+ 20 H

004058DB  66:9C           PUSHFW

004058DD  0FBA3C24 08     BTC DWORD PTR SS:[ESP],8                    ;位测试并取反 传说中的单步异常 设置 TF 位

004058E2  66:9D           POPFW

004058E4  5B              POP EBX                                     ;

004058E5  5A              POP EDX                                     ;单步异常

004058E6  64:8F05 0000000>POP DWORD PTR FS:[0]

### 第二次异常处理

00405B0E  33C0            XOR EAX,EAX

00405B10  64:8B18         MOV EBX,DWORD PTR FS:[EAX]                 ;FS[0]   Ptr32 _EXCEPTION_REGISTRATION_RECORD

00405B13  8B1B            MOV EBX,DWORD PTR DS:[EBX]                 ;_EXCEPTION_REGISTRATION_RECORD.Next
                                                                      ;这里找到建立第二次异常链后的堆栈

00405B15  8D63 AE         LEA ESP,DWORD PTR DS:[EBX-52]              ;  52H = 2AH -4H + 0CH +20H 定位到了第二次异常链建立好后
                                                                      ;pushad 后的 堆栈

00405B18  61              POPAD

00405B19  833E 00         CMP DWORD PTR DS:[ESI],0

00405B1C  ^ 0F84 C2FDFFFF  JE Notepad.004058E4

00405B22  0FBA26 1F       BT DWORD PTR DS:[ESI],1F                   ;测试最高位 PETITE.number

00405B26  73 05           JNB SHORT Notepad.00405B2D                ;CF 为 0 则跳

00405B28  83C6 0C         ADD ESI,0C

00405B2B  ^ EB EC         JMP SHORT Notepad.00405B19

00405B2D  8B7E 08         MOV EDI,DWORD PTR DS:[ESI+8]              ;PETITE.target2

00405B30  03FD            ADD EDI,EBP                               ;定位到解压后的节中

00405B32  8B4E 0C         MOV ECX,DWORD PTR DS:[ESI+C]              ;PETITE.xx

00405B35  D1F9            SAR ECX,1

00405B37  51              PUSH ECX

00405B38  72 15           JB SHORT Notepad.00405B4F

00405B3A  037E 04         ADD EDI,DWORD PTR DS:[ESI+4]              ;PETITE.??(从这段代码看 ??可能还是解压后有效数据的大小        ?)

00405B3D  C1F9 02         SAR ECX,2

00405B40  33C0            XOR EAX,EAX

00405B42  F3:AB           REP STOS DWORD PTR ES:[EDI]                ;清 0  字节数为 PETITE.xx/2(XX 看来跟解压后的无效数据大小有关  ?)

00405B44  59              POP ECX

00405B45  83E1 03         AND ECX,3

00405B48  F3:AA           REP STOS BYTE PTR ES:[EDI]

00405B4A  83C6 10         ADD ESI,10

00405B4D  ^ EB CA         JMP SHORT Notepad.00405B19
                                                                      ;下面就是最后的修正算法了

00405B4F  8B5E 04         MOV EBX,DWORD PTR DS:[ESI+4]              ;PETITE.??

00405B52  83EB 06         SUB EBX,6                                  ;PETITE.??-6

00405B55  33D2            XOR EDX,EDX

00405B57  ^ 72 E1         JB SHORT Notepad.00405B3A

00405B59  8A043A          MOV AL,BYTE PTR DS:[EDX+EDI]

00405B5C  3C E8           CMP AL,0E8                                 ;E8 和 E9 是 CALL 和 JMP 的机器码 开始修正 mov 的操作数

00405B5E  74 0E           JE SHORT Notepad.00405B6E

00405B60  3C E9           CMP AL,0E9

00405B62  74 0A           JE SHORT Notepad.00405B6E

00405B64  3C 0F           CMP AL,0F                                    ;0F80 - 0F8F 查的书上 似乎都是段内跳转指令的机器码  修改跳转指令的操作数

00405B66  74 12           JE SHORT Notepad.00405B7A

00405B68  42              INC EDX

00405B69  83EB 01         SUB EBX,1

00405B6C  ^ EB E9         JMP SHORT Notepad.00405B57

00405B6E  29543A 01       SUB DWORD PTR DS:[EDX+EDI+1],EDX           ;如果是 MOV 指令 就把后面的操作数机器码 - MOV 指令机器码相对于解密起始处的偏
                                                                      移

00405B72  83C2 05         ADD EDX,5

00405B75  83EB 05         SUB EBX,5

00405B78  ^ EB DD         JMP SHORT Notepad.00405B57

00405B7A  8A443A 01       MOV AL,BYTE PTR DS:[EDX+EDI+1]

00405B7E  3C 80           CMP AL,80

00405B80  ^ 72 E6         JB SHORT Notepad.00405B68

00405B82  3C 8F           CMP AL,8F

00405B84  ^ 77 E2         JA SHORT Notepad.00405B68

00405B86  29543A 02       SUB DWORD PTR DS:[EDX+EDI+2],EDX           ;如果是段内跳转指令就把

00405B8A  83C2 06         ADD EDX,6

00405B8D  83EB 06         SUB EBX,6

00405B90  ^ EB C5         JMP SHORT Notepad.00405B57

00405B92  59              POP ECX

00405B93  5E              POP ESI

00405B94  FD              STD

00405B95  33C0            XOR EAX,EAX

00405B97  B9 56030000     MOV ECX,356

00405B9C  E8 43B98624     CALL 24C714E4                               ;解密后 从这里 F7(解密后 这个 CALL 的转移地址要变的)

整个算法描述 有点嗦：如果碰到 CALL 和 JMP 指令或者段内转移指令 就把它们的操作数机器码 减去 本身相对于解密起始处的偏移

0040D039  5F              POP EDI                                     ; Notepad.00405BA1

0040D03A  F3:AA           REP STOS BYTE PTR ES:[EDI]

0040D03C  61              POPAD

0040D03D  66:9D           POPFW

0040D03F  83C4 08         ADD ESP,8

0040D042  >- E9 8540FFFF  JMP Notepad.004010CC                       ; 哈哈 回到加壳后的入口点地址 跳到 OEP

### 总结

整个脱壳过程中 数据复制 2 次 解压 2 次 解密一次 2 次异常  而且 关键跳 就在处 EP

快速脱壳 除了最后一次异常法还可以 用多次内存断点 或者 直接在 载入后的入口点下个硬件执行断点 F9 就可以了

### 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | Notepad.rar |
| :--- | :--- |
