
# DBPE 注册码判断核心

> 原文：[`www.pediy.com/kssd/pediy02/forum122.htm`](https://www.pediy.com/kssd/pediy02/forum122.htm)

DBPE 注册码判断核心

作者：IceWorld

网址：`http://iceworld.126.com/`

注意：装载请注明出处

```
MOV EAX,1
XOR EBX,EBX
MOV ESI,4086EE
ADD ESI,EBP

A_1:     MOV BL,[ESI]             取经过换算的注册码
         CMP BL,00                到末尾了吗？
         JZ A_2
         MUL EBX
         INC ESI
         JMP A_1

A_2:     XOR EAX,[EBP+40A01F]
         XOR EAX,[EBP+4084CF]
         XCHG AH,AL
         MOV EBX,EAX              这时 EBX 为正确的注册码
         MOV ESI,408702
         ADD ESI,EBP
         CALL Check_Serial        判断注册码正确与否
         CMP EBX,EAX
         JZ Serial_OK             如果正确则 AX=1
         MOV AX,0                 如果错误则 AX=0
         RET

Serial_OK: MOV AX,1
         RET
```

不知道 `IceWorld` 对付的是哪一版本呢？`1.2` 版本的得到算法代码不难，如果是 `1.5` 的话实在佩服！ `:)` `1.5` 在前面部分的算法和 `1.2` 有些不同！

文章中有点地方有误:

```
A_1:     MOV BL,[ESI]             取经过换算的注册码
         CMP BL,00                到末尾了吗？
         JZ A_2
         MUL EBX
         INC ESI
         JMP A_1
```

这是对输入的名字进行换算，得 `CRC` 到 `EAX`

```
A_2:     XOR EAX,[EBP+40A01F]    
         XOR EAX,[EBP+4084CF]
```

`xor register key` 和硬件代码。

```
         XCHG AH,AL
         MOV EBX,EAX              这时 EBX 为正确的注册码
         MOV ESI,408702
         ADD ESI,EBP
         CALL Check_Serial        判断注册码正确与否
```

这个 `CALL` 是 `Str to int`，把输入号放到 `EAX`.

```
         CMP EBX,EAX
         JZ Serial_OK             如果正确则 AX=1
         MOV AX,0                 如果错误则 AX=0
         RET
```

欢迎大家去 `K 幻影`，感谢大家对幻影有兴趣。 `^\_^` `4dd466d0e1e0cc123afbe504da2462ca`
