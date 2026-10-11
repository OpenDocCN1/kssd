
### 注册码第一部分提取

```
005B3433  . 50             PUSH EAX
005B3434  . E8 273D0800    CALL starryni.00637160  *****该函数从一个字符串的左端取子串
005B3439  . 83C4 0C        ADD ESP,0C               *****所以这里取注册码第一部分 3 个字符的子串
005B343C  . 6A 04          PUSH 4
005B343E  . 8B45 0C        MOV EAX,DWORD PTR SS:[EBP+C]
005B3441  . 83C0 04        ADD EAX,4
005B3444  . 50             PUSH EAX
005B3445  . 8D45 CC        LEA EAX,DWORD PTR SS:[EBP-34]
005B3448  . 50             PUSH EAX
005B3449  . E8 123D0800    CALL starryni.00637160  *****第二部分 4 个字符
005B344E  . 83C4 0C        ADD ESP,0C
005B3451  . 6A 04          PUSH 4
005B3453  . 8B45 0C        MOV EAX,DWORD PTR SS:[EBP+C]
005B3456  . 83C0 09        ADD EAX,9
005B3459  . 50             PUSH EAX
005B345A  . 8D45 CC        LEA EAX,DWORD PTR SS:[EBP-34]
005B345D  . 50             PUSH EAX
005B345E  . E8 FD3C0800    CALL starryni.00637160  *****第三部分 4 个字符
005B3463  . 83C4 0C        ADD ESP,0C
005B3466  . 6A 04          PUSH 4
005B3468  . 8B45 0C        MOV EAX,DWORD PTR SS:[EBP+C]
005B346B  . 83C0 0E        ADD EAX,0E
005B346E  . 50             PUSH EAX
005B346F  . 8D45 CC        LEA EAX,DWORD PTR SS:[EBP-34]
005B3472  . 50             PUSH EAX
005B3473  . E8 E83C0800    CALL starryni.00637160  *****第四部分 4 个字符
005B3478  . 83C4 0C        ADD ESP,0C
```

以上代码把注册码的第 1,2,3,4 部分连接起来，去掉连字号，形成 `yr123456789abcd`，在 `[ebp-34]` 处。

### 字符串重排逻辑

```
005B347B  . 8D45 CF        LEA EAX,DWORD PTR SS:[EBP-31]  *****这里 eax 指向"23456789abcd"
005B347E  . 50             PUSH EAX
005B347F  . E8 83010000    CALL starryni.005B3607        *****这个函数根据一个表把上述字符串重新排序
```

内部函数 `starryni.005B3607` 详细逻辑：

```
            005B3616  MOV EDI,DWORD PTR SS:[EBP+8]
            005B3619  XOR EAX,EAX
            005B361B  OR ECX,FFFFFFFF
            005B361E  REPNE SCAS BYTE PTR ES:[EDI]
            005B3620  MOV EAX,-2
            005B3625  SUB EAX,ECX
            005B3627  CMP EAX,0C                *****长度检查：应该有 0xC 个字符
            005B362A  JE SHORT starryni.005B362E
            005B362C  JMP SHORT starryni.005B3663
            005B362E  XOR ESI,ESI               *****esi 作为索引
            005B3630  JMP SHORT starryni.005B3646
            005B3632  /MOVSX EBX,WORD PTR DS:[ESI*2+729CA8]  *****从位于[729CA8]的表中得到新索引
            005B363A  |MOV EAX,DWORD PTR SS:[EBP+8]
            005B363D  |MOVZX EDX,BYTE PTR DS:[EAX+EBX]    *****从原字符串中取出新索引对应的字符
            005B3641  |MOV BYTE PTR SS:[EBP+ESI-1C],DL    *****放在临时缓冲区的原索引对应位置
            005B3645  |INC ESI
            005B3646   CMP ESI,0C
            005B3649  \JL SHORT starryni.005B3632
            005B364B  MOV BYTE PTR SS:[EBP-10],0
            005B364F  MOV EAX,DWORD PTR SS:[EBP+8]
            005B3652  MOV BYTE PTR DS:[EAX],0
            005B3655  LEA EAX,DWORD PTR SS:[EBP-1C]
            005B3658  PUSH EAX
            005B3659  PUSH DWORD PTR SS:[EBP+8]
            005B365C  CALL starryni.00637130          *****用重排过的字符串代替原字符串
```

`[729CA8]` 表的内容如下：

```
            0B 00 02 00 06 00 08 00 04 00 07 00 0A 00 01 00 03 00 09 00 00 00 05 00
```

所以 `"23456789abcd"` 重排后变成 `"d48a69c35b27"`。

### 计算校验和

```
005B3484  . 59             POP ECX
005B3485  . 8D45 CC        LEA EAX,DWORD PTR SS:[EBP-34]   *****字符串现已变成"yr1d48a69c35b27"
005B3488  . 50             PUSH EAX                        *****作为参数压入堆栈
005B3489  . E8 CE020000    CALL starryni.005B375C          *****关键 call,计算检查和
```

内部函数 `starryni.005B375C` 详细逻辑：

```
            005B375C   PUSH EBP
            005B375D   MOV EBP,ESP
            005B375F   PUSH EBX
            005B3760   PUSH ESI
            005B3761   PUSH EDI
            005B3762   SUB ESP,24
            005B3765  >MOV DWORD PTR SS:[EBP-30],-32EB     *****这里有一个常数
            005B376C   PUSH 1E                               *****立即数指明所需字符串的长度
            005B376E   LEA EAX,DWORD PTR SS:[EBP-2C]         *****存放返回结果的缓冲区
            005B3771   PUSH EAX
            005B3772   PUSH DWORD PTR SS:[EBP+8]             *****"yr1d48a69c35b27"
            005B3775   CALL starryni.005B37BD                *****这个函数复制并连接字符串直到达到指定长度
            005B377A   ADD ESP,0C
            005B377D   TEST AL,AL                            *****返回非零表示成功
            005B377F   JNZ SHORT starryni.005B3788
            005B3781   MOV EAX,-11CD
            005B3786   JMP SHORT starryni.005B37B5
            005B3788   LEA EDI,DWORD PTR SS:[EBP-2C]         *****已经连接的字符串：
            005B378B   XOR EAX,EAX                           *****"yr1d48a69c35b27yr1d48a69c35b27"
            005B378D   OR ECX,FFFFFFFF
            005B3790   REPNE SCAS BYTE PTR ES:[EDI]
            005B3792   MOV ESI,-2
            005B3797   SUB ESI,ECX                           *****长度 0x1E
            005B3799   XOR EBX,EBX                           *****ebx 为索引 index
            005B379B   JMP SHORT starryni.005B37AE
            005B379D   /MOVZX ECX,BYTE PTR SS:[EBP+EBX-2C]   *****ecx = ch
            005B37A2   |MOV EDX,EBX                         *****edx = index
            005B37A4   |IMUL EDX,ECX                        *****edx = index*ch
            005B37A7   |IMUL EDX,ECX                        *****edx = index*ch*ch = index*ch**2
            005B37AA   |ADD DWORD PTR SS:[EBP-30],EDX        *****结果加到常数-32EB 上
            005B37AD   |INC EBX
            005B37AE    CMP EBX,ESI
            005B37B0   \JL SHORT starryni.005B379D
            005B37B2   MOV EAX,DWORD PTR SS:[EBP-30]         *****返回最后结果
            005B37B5   LEA ESP,DWORD PTR SS:[EBP-C]
            005B37B8   POP EDI
            005B37B9   POP ESI
            005B37BA   POP EBX
            005B37BB   POP EBP
            005B37BC   RETN
```

这个过程的工作是把字符串首尾相连达到 30 个字符 (`0x1E`)，然后计算检查和：

```
checksum = sum(index*ch**2)-0x32EB
```

### 提取最后部分并转换

```
005B348E  . 59             POP ECX
005B348F  . 0FB7C0        MOVZX EAX,AX                    *****抛弃检查和的高位字
005B3492  . 8D55 C0        LEA EDX,DWORD PTR SS:[EBP-40]
005B3495  . 52             PUSH EDX
005B3496  . 0FB7C0        MOVZX EAX,AX
005B3499  . 50             PUSH EAX
005B349A  . E8 40040000    CALL starryni.005B38DF          *****这个函数将检查和的低位字转成字符串行式
005B349F  . 59             POP ECX
005B34A0  . 59             POP ECX
005B34A1  . C645 B4 00     MOV BYTE PTR SS:[EBP-4C],0
005B34A5  . 6A 04          PUSH 4
005B34A7  . 8B7D 0C        MOV EDI,DWORD PTR SS:[EBP+C]    *****这是原来输入的原始形式的注册码
005B34AA  . 31C0          XOR EAX,EAX
005B34AC  . 83C9 FF       OR ECX,FFFFFFFF
005B34AF  . F2:AE         REPNE SCAS BYTE PTR ES:[EDI]
005B34B1  . BA FEFFFFFF   MOV EDX,-2
005B34B6  . 29CA          SUB EDX,ECX                     *****取长度
005B34B8  . 83C2 FC       ADD EDX,-4                      ***** -4
005B34BB  . 0355 0C       ADD EDX,DWORD PTR SS:[EBP+C]     *****edx 指向最后 4 个字符，即第五部分
005B34BE  . 52             PUSH EDX
005B34BF  . 8D45 B4        LEA EAX,DWORD PTR SS:[EBP-4C]
005B34C2  . 50             PUSH EAX
005B34C3  . E8 983C0800    CALL starryni.00637160         *****取出最后 4 个字符
005B34C8  . 83C4 0C        ADD ESP,0C
005B34CB  . 8D45 C0        LEA EAX,DWORD PTR SS:[EBP-40]   *****检查和字符串
005B34CE  . 50             PUSH EAX
005B34CF  . 8D45 B4        LEA EAX,DWORD PTR SS:[EBP-4C]   *****注册码最后部分字符串
```
