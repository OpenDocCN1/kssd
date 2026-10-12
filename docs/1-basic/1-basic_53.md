
# movsx,cdq

> 原文：[`www.pediy.com/kssd/pediy03/forum072.htm`](https://www.pediy.com/kssd/pediy03/forum072.htm)

`:bpx hmemcpy`

Break due to BPX KERNEL!HMEMCPY  (ET=517.75 milliseconds)

`:bc *`

`:u cs:eip l 200`

```
0167:0043B5C0  55                  PUSH      EBP
0167:0043B5C1  8BEC                MOV       EBP,ESP
0167:0043B5C3  56                  PUSH      ESI
0167:0043B5C4  8B7508              MOV       ESI,[EBP+08]----输入密码
0167:0043B5C7  57                  PUSH      EDI
0167:0043B5C8  BF3412C72F          MOV       EDI,2FC71234
0167:0043B5CD  85F6                TEST      ESI,ESI
0167:0043B5CF  743A                JZ        0043B60B
0167:0043B5D1  8A06                MOV       AL,[ESI]
0167:0043B5D3  84C0                TEST      AL,AL
0167:0043B5D5  7434                JZ        0043B60B
0167:0043B5D7  53                  PUSH      EBX
0167:0043B5D8  BBF8110000          MOV       EBX,000011F8
0167:0043B5DD  0FBEC0              MOVSX     EAX,AL
0167:0043B5E0  50                  PUSH      EAX
0167:0043B5E1  E83F550600          CALL      004A0B25
0167:0043B5E6  8BC8                MOV       ECX,EAX
0167:0043B5E8  8BC3                MOV       EAX,EBX
0167:0043B5EA  99                  CDQ
0167:0043B5EB  F7F9                IDIV      ECX
0167:0043B5ED  83C404              ADD       ESP,04
0167:0043B5F0  81C390010000        ADD       EBX,00000190
0167:0043B5F6  40                  INC       EAX
0167:0043B5F7  0FAFF8              IMUL      EDI,EAX
0167:0043B5FA  8A4601              MOV       AL,[ESI+01]
0167:0043B5FD  46                  INC       ESI
0167:0043B5FE  84C0                TEST      AL,AL
0167:0043B600  75DB                JNZ      0043B5DD
0167:0043B602  8BC7                MOV       EAX,EDI
0167:0043B604  5B                  POP       EBX
0167:0043B605  5F                  POP       EDI
0167:0043B606  5E                  POP       ESI
0167:0043B607  5D                  POP       EBP
0167:0043B608  C20400              RET       0004
0167:0043B60B  8BC7                MOV       EAX,EDI
0167:0043B60D  5F                  POP       EDI
0167:0043B60E  5E                  POP       ESI
0167:0043B60F  5D                  POP       EBP
0167:0043B610  C20400              RET       0004
```

你好,希望这些注释对你有帮助.

```
0167:0043B5C1  8BEC                MOV       EBP,ESP
0167:0043B5C3  56                  PUSH      ESI
0167:0043B5C4  8B7508              MOV       ESI,[EBP+08]
0167:0043B5C7  57                  PUSH      EDI
0167:0043B5C8  BF3412C72F          MOV       EDI,2FC71234 <-EDI 初始化
0167:0043B5CD  85F6                TEST      ESI,ESI
0167:0043B5CF  743A                JZ        0043B60B
0167:0043B5D1  8A06                MOV       AL,[ESI] <-取密码的第一个字
0167:0043B5D3  84C0                TEST      AL,AL
0167:0043B5D5  7434                JZ        0043B60B <-如果空白,则跳出
0167:0043B5D7  53                  PUSH      EBX
0167:0043B5D8  BBF8110000          MOV       EBX,000011F8 <-EBX 初始化
0167:0043B5DD  0FBEC0              MOVSX     EAX,AL <- 将所取字移入 EAX
0167:0043B5E0  50                  PUSH      EAX <-保存
0167:0043B5E1  E83F550600          CALL      004A0B25 <-估计是对该字进行一些操作
0167:0043B5E6  8BC8                MOV       ECX,EAX
0167:0043B5E8  8BC3                MOV       EAX,EBX
0167:0043B5EA  99                  CDQ     <-EDX 清零
0167:0043B5EB  F7F9                IDIV      ECX <-EAX 除以 ECX,商放在 EAX,余数在 EDX
0167:0043B5ED  83C404              ADD       ESP,04
0167:0043B5F0  81C390010000        ADD       EBX,00000190 <-EBX+190h
0167:0043B5F6  40                  INC       EAX          <-EAX+1
0167:0043B5F7  0FAFF8              IMUL      EDI,EAX <- EDI*EAX 积在 EDI
0167:0043B5FA  8A4601              MOV       AL,[ESI+01] <-取下一个字
0167:0043B5FD  46                  INC       ESI  <-ESI+1 指向下一字
0167:0043B5FE  84C0                TEST      AL,AL <- 是否已取完所有字
0167:0043B600  75DB                JNZ      0043B5DD <-否,则继续对下一个进行运算
0167:0043B602  8BC7                MOV       EAX,EDI <-运算结果放入 EAX
0167:0043B604  5B                  POP       EBX
0167:0043B605  5F                  POP       EDI
0167:0043B606  5E                  POP       ESI
0167:0043B607  5D                  POP       EBP
0167:0043B608  C20400              RET       0004 <- 返回程序调用处
0167:0043B60B  8BC7                MOV       EAX,EDI
0167:0043B60D  5F                  POP       EDI
0167:0043B60E  5E                  POP       ESI
0167:0043B60F  5D                  POP       EBP
0167:0043B610  C20400              RET       0004
```

`MOVSX` 将字节(Byte)或字(Word)操作数分别扩展成 Word 或 Dword,并保留该值的符号(正负).

`CDQ` 是一个让很多初学者感到困惑的指令. 它大多出现在除法运算之前. 它实际的作用只是把 `EDX` 的所有位都设成 `EAX` 最高位的值. 也就是说,当 `EAX <80000000`, `EDX` 00000000; 当 `EAX >= 80000000`, `EDX` 则为 FFFFFFFF).

冰毒于 2000 年 5 月 28 日   4dd466d0e1e0cc123afbe504da2462ca
