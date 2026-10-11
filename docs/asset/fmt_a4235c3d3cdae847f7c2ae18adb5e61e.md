
## 进位循环的运算

### 计算前

`LF($44D1)=4F905B39971: EAX=05B39971; EDX=000004F9`

### 计算后

`ESI=000053F7, EAX=0B7D772E, ECX=0, EDI=0, EDX=0, EBX=00006ECB`

```
00405950  /$ 55                 PUSH EBP
00405951  |. 53                 PUSH EBX
00405952  |. 56                 PUSH ESI
00405953  |. 57                 PUSH EDI
00405954  |. 31FF               XOR EDI,EDI
00405956  |. 8B5C24 14          MOV EBX,DWORD PTR SS:[ESP+14]
0040595A  |. 8B4C24 18          MOV ECX,DWORD PTR SS:[ESP+18]
0040595E  |. 09C9               OR ECX,ECX
00405960  |. 75 08              JNZ SHORT 1.0040596A
00405962  |. 09D2               OR EDX,EDX
00405964  |. 74 5D              JE SHORT 1.004059C3
00405966  |. 09DB               OR EBX,EBX
00405968  |. 74 59              JE SHORT 1.004059C3
0040596A  |> 09D2               OR EDX,EDX
0040596C  |. 79 0A              JNS SHORT 1.00405978
0040596E  |. F7DA               NEG EDX
00405970  |. F7D8               NEG EAX
00405972  |. 83DA 00            SBB EDX,0
00405975  |. 83CF 01            OR EDI,1
00405978  |> 09C9               OR ECX,ECX
0040597A  |. 79 07              JNS SHORT 1.00405983
0040597C  |. F7D9               NEG ECX
0040597E  |. F7DB               NEG EBX
00405980  |. 83D9 00            SBB ECX,0
00405983  |> 89CD               MOV EBP,ECX
00405985  |. B9 40000000        MOV ECX,40
0040598A  |. 57                 PUSH EDI
0040598B  |. 31FF               XOR EDI,EDI
0040598D  |. 31F6               XOR ESI,ESI               //循环计算 64 次
0040598F  |> D1E0               /SHL EAX,1                //EAX=EAX SHL 1
00405991  |. D1D2               |RCL EDX,1                //EDX=EDX RCL 1
00405993  |. D1D6               |RCL ESI,1                //ESI=ESI RCL 1
00405995  |. D1D7               |RCL EDI,1                //EDI=EDI RCL 1
00405997  |. 39EF               |CMP EDI,EBP               //EBP=28363
00405999  |. 72 0B              |JB SHORT 1.004059A6
0040599B  |. 77 04              |JA SHORT 1.004059A1
0040599D  |. 39DE               |CMP ESI,EBX
0040599F  |. 72 05              |JB SHORT 1.004059A6
004059A1  |> 29DE               |SUB ESI,EBX               //ESI=ESI-EBX
004059A3  |. 19EF               |SBB EDI,EBP
004059A5  |. 40                 |INC EAX
004059A6  |>^E2 E7              \LOOPD SHORT 1.0040598F//EBX 作为循环的标志
004059A8  |. 89F0               MOV EAX,ESI                 //EAX=ESI
004059AA  |. 89FA               MOV EDX,EDI
004059AC  |. 5B                 POP EBX
004059AD  |. F7C3 01000000      TEST EBX,1
004059B3  |. 74 07              JE SHORT 1.004059
004059B5  |. F7DA               NEG EDX
004059B7  |. F7D8               NEG EAX
004059B9  |. 83DA 00            SBB EDX,0
004059BC  |> 5F                 POP EDI
004059BD  |. 5E                 POP ESI
004059BE  |. 5B                 POP EBX
004059BF  |. 5D                 POP EBP
004059C0  |. C2 0800            RETN 8
004059C3  |> F7F3               DIV EBX
004059C5  |. 92                 XCHG EAX,EDX
004059C6  |. 31D2               XOR EDX,EDX
004059C8  \.^EB F2              JMP SHORT 1.004059BC
004059CA  . C3                  RETN
```

很可惜进位循环的部分我不能用编程模拟(水平太差,也没有时间思考...)所以注册机不能完成.

不知道各位谁有时间可以跟踪一下然后给出进位循环的源码^\_^谢了!

这几日单位的事情非常之多,而且工作压力还特别的大~!所以想找一个软件分析一下,作为松弛神经的方法.

不知道各位有什么好的办法给自己减压啊!告诉小弟几招阿~小弟不胜感激~!

`4dd466d0e1e0cc123afbe504da2462ca`
