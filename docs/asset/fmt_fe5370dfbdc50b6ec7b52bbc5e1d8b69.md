
```
00402B43       |.   83FE 10           |CMP     ESI,10

00402B46       |.^  7C E9            \JL      SHORT QQ 聊天~1.00402B31

00402B48       |.   33C0            XOR     EAX,EAX                       ;  EAX 清 0

00402B4A       |.   8D4C24 10       LEA     ECX,DWORD PTR SS:[ESP+10]

00402B4E       |.   5E              POP     ESI

00402B4F       |>   8A51 01         /MOV    DL,BYTE PTR DS:[ECX+1]         ;  依取出偶数的假码字符！送入到 DL

00402B52       |.   8A19            |MOV    BL,BYTE PTR DS:[ECX]           ;  依取出奇数的假码字符！送入到 BL

00402B54       |.   C0E2 04         |SHL     DL,4                          ;  左移，乘 2⁴(十六进制 10)(十进制为 16)

00402B57       |.   02D3            |ADD     DL,BL                        ;  DL=左移 4 位的 DL+BL

00402B59       |.   83C1 02         |ADD     ECX,2                        ;  ECX+2 做好准备取下一位偶数假码字符！

00402B5C       |.   885404 04       |MOV     BYTE PTR SS:[ESP+EAX+4],DL    ;  累加后保存到 0012EE40（21436587A9CBEDFF）

00402B60       |.   40              |INC     EAX                          ;  EAX 加 1 为计数器器

00402B61       |.   83F8 08         |CMP     EAX,8                        ;  比较 EAX 是否小于 8

00402B64       |.^  7C E9           \JL      SHORT QQ 聊天~1.00402B4F

00402B66       |.   8A4424 07       MOV     AL,BYTE PTR SS:[ESP+7]        ;  AL=等于计算后的 4 位

00402B6A       |.   8A5C24 04       MOV     BL,BYTE PTR SS:[ESP+4]        ;  BL=等于计算后的 1 位

00402B6E       |.   8A4C24 0B       MOV     CL,BYTE PTR SS:[ESP+B]        ;  CL=等于计算后的 8 位

00402B72       |.   8A5424 05       MOV     DL,BYTE PTR SS:[ESP+5]        ;  DL==等于计算后的 2 位

00402B76       |.   32C3            XOR     AL,BL

00402B78       |.   8A5C24 06       MOV     BL,BYTE PTR SS:[ESP+6]        ;  BL=等于计算后的 3 位

00402B7C       |.   32CA            XOR     CL,DL

00402B7E       |.   8A5424 09       MOV     DL,BYTE PTR SS:[ESP+9]        ;  DL=等于计算后的 6 位

00402B82       |.   32D3            XOR     DL,BL

00402B84       |.   8A5C24 08       MOV     BL,BYTE PTR SS:[ESP+8]        ;  BL==等于计算后的 5 位

00402B88       |.   325C24 0A       XOR     BL,BYTE PTR SS:[ESP+A]        ;  BL 和=等于计算后的 7 位

00402B8C       |.   3C 38           CMP     AL,38                         ;  比较 1 XOR 4 是否等于 38

00402B8E       |.   75 35           JNZ     SHORT QQ 聊天~1.00402BC5      ;  不满足则跳走失败！！

00402B90       |.   80F9 6E         CMP     CL,6E                         ;  比较 2 XOR 8 是否等于 6E

00402B93       |.   75 30           JNZ     SHORT QQ 聊天~1.00402BC5      ;  不满足则跳走失败！！

00402B95       |.   80FA 4E         CMP     DL,4E                         ;  比较 6 XOR 3 是否等于 4E

00402B98       |.   75 2B           JNZ     SHORT QQ 聊天~1.00402BC5      ;  不满足则跳走失败！！

00402B9A       |.   80FB 1A         CMP     BL,1A                         ;  比较 5 XOR 7 是否等于 1A

00402B9D       |.   75 26           JNZ     SHORT QQ 聊天~1.00402BC5      ;  不满足则跳走失败！！

00402B9F       |.   8D4C24 2C       LEA     ECX,DWORD PTR SS:[ESP+2C]

00402BA3       |.   C74424 24 FFFF  MOV     DWORD PTR SS:[ESP+24],-1

00402BAB       |.   E8 C2860100     CALL    QQ 聊天~1.0041B272

00402BB0       |.   B8 01000000     MOV     EAX,1                         ;  EAX 置 1（注册成功标志，这个标志也全局变量）爆破记得这里哦！

00402BB5       |.   5B              POP     EBX

00402BB6       |.   8B4C24 18       MOV     ECX,DWORD PTR SS:[ESP+18]

00402BBA       |.   64:890D 000000  MOV     DWORD PTR FS:[0],ECX

00402BC1       |.   83C4 24         ADD     ESP,24

00402BC4       |.   C3              RETN

00402BC5       |>   8D4C24 2C       LEA     ECX,DWORD PTR SS:[ESP+2C]

00402BC9       |.   C74424 24 FFFF  MOV     DWORD PTR SS:[ESP+24],-1

00402BD1       |.   E8 9C860100     CALL    QQ 聊天~1.0041B272

00402BD6       |.   8B4C24 1C       MOV     ECX,DWORD PTR SS:[ESP+1C]

00402BDA       |.   33C0            XOR     EAX,EAX

00402BDC       |.   5B              POP     EBX

00402BDD       |.   64:890D 000000  MOV     DWORD PTR FS:[0],ECX

00402BE4       |.   83C4 24         ADD     ESP,24

00402BE7       \\.  C3              RETN
```

### 分析总结

这个软件的算法很简单，简单点说，这个注册方式一定要`16`位字符（`0~~F`）。

打比方，我输入的假码：

**第一步**：(输入的假码）
`01 23 45 67 89 AB CD EF`

**第二步**：(假码字符排列）
`10 32 54 76 98 BA DC FE`

我们来分一下假码排列吧！把假码分为`8`组！
`10 32 54 76 98 BA DC FE`
`－－－－－－－－－－－－－－`
`1 2 3 4 5 6 7 8`

**第三步**：注册码算法就是做一个`XOR`运算了！

`1(10) XOR 4(32) ＝ 38`
`2(54) XOR 8(76) ＝ 6E`
`6(98) XOR 3(BA) ＝ 4E`
`5(DC) XOR 7(FE) ＝ 1A`

满足以上等于号的条件就可以注册成功了。接下来就是为真注册码排列注册方式！

**第四步**：这是一个真注册码`1274749121908092`
`21 47 47 19 12 09 08 29`
`----------------------------------------------`
`1 2 3 4 5 6 7 8`

然后排列为`1274749121908092`这个就是真注册码了！

### 后话

这个软件算法超简单了，适合菜鸟们学习。感觉谢谢你看完这篇破文，希望能给刚入门的菜鸟们带来一点点的启发！时间比较仓促（难免错漏），写得不好请`DX`位不要拍砖哦！（我不坐砖机哦！）

`22:22 2006-6-6`
`4dd466d0e1e0cc123afbe504da2462ca`
