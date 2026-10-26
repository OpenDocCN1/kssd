
```
0046615E    83C4 10         ADD ESP,10
00466161    83C8 FF         OR EAX,FFFFFFFF
00466164    03D8            ADD EBX,EAX
00466166  ^ E9 EDFEFEFF     JMP FXRL.00466058
0046616B    50              PUSH EAX                                 ; 循环的最后一轮处理过注册名第2字符'b'后，跳出到这里，准备返回上级
0046616C    FF15 40124000   CALL [DWORD DS:401240]                   ; MSVBVM60.__vbaR8Str
00466172    DC25 20144000   FSUB [QWORD DS:401420]
00466178    DD5D D8         FSTP [QWORD SS:EBP-28]
0046617B    FF15 D8104000   CALL [DWORD DS:4010D8]                   ; MSVBVM60.__vbaExitProc
00466181    9B              WAIT
00466182    68 13624600     PUSH FXRL.00466213
00466187    EB 69           JMP SHORT FXRL.004661F2
...
004661F2    8D4D E0         LEA ECX,[DWORD SS:EBP-20]
004661F5    8B35 4C134000   MOV ESI,[DWORD DS:40134C]                ; MSVBVM60.__vbaFreeStr
004661FB    FFD6            CALL ESI
004661FD    8D4D CC         LEA ECX,[DWORD SS:EBP-34]
00466200    FFD6            CALL ESI
00466202    8D4D BC         LEA ECX,[DWORD SS:EBP-44]
00466205    8B35 20104000   MOV ESI,[DWORD DS:401020]                ; MSVBVM60.__vbaFreeVar
0046620B    FFD6            CALL ESI
0046620D    8D4D A4         LEA ECX,[DWORD SS:EBP-5C]
00466210    FFD6            CALL ESI
00466212    C3              RETN
```

上面返回到下面这部分

```
004D2BE5    FF15 4C104000   CALL [DWORD DS:40104C]                   ; MSVBVM60.__vbaFreeObjList
004D2BEB    83C4 0C         ADD ESP,0C
004D2BEE    C745 FC 1900000>MOV [DWORD SS:EBP-4],19
004D2BF5    E8 862CF9FF     CALL FXRL.00465880                       ; 调用注册验证部分
004D2BFA    DD9D 78FFFFFF   FSTP [QWORD SS:EBP-88]                   ; 验证结果的字串转换成浮点数
004D2C00    DD05 D04B4000   FLD [QWORD DS:404BD0]
004D2C06    DC85 78FFFFFF   FADD [QWORD SS:EBP-88]                   ; 加常数98.0
004D2C0C    DC0D C84B4000   FMUL [QWORD DS:404BC8]                   ; 乘常数3.0
004D2C12    FF15 14114000   CALL [DWORD DS:401114]                   ; MSVBVM60.__vbaFpR8
004D2C18    DC1D C04B4000   FCOMP [QWORD DS:404BC0]                  ; 与常数291.0相比，必须相等
004D2C1E    DFE0            FSTSW AX
004D2C20    F6C4 40         TEST AH,40
004D2C23    74 0C           JE SHORT FXRL.004D2C31                   ; 不跳就是注册版
```

由`(0-1+98)*3=291`可知，调用返回的字串必须是`"0"`

### 总结

1. 程序自己计算出机器码；用户注册名最少2字符

    注册码是数字组成的字串，开头两个字符分别是注册名前两字符ASCII值的十进制个位数字

2. 由机器码前3字符(加法)和后半部分(乘法)计算出`RES`

    `TAIL` 是 `sqrt(RES)` 浮点数转换成字符串(长度16，小数部分11位)后的最后5个数字，变成浮点数(整)

    `W=Int(sqrt(RES)*10000.0)+TAIL`，此`W`后面多次要用到

    注册名第1字符ASCII值减7，变成浮点数，设为`CA`

    注册码第3字符开始的余部设为`HM`

    `SS=HM-Int(ln(sqrt(W))*100.0)*CA`，此`SS`是最后循环体计算用的初始值

3. 最后，循环倒取注册名各字符，直到第2字符计算过

    `Cn`=当前字符ASCII值减32（转换成浮点数）

    `D=Int(ln(W)*100.0)`

    `SS=SS-W+Cn+D`

    循环最后`SS`要等于0

4. 根据第2步不涉及注册码余部的一些计算结果，对第3步反向循环不难得出初始的`SS`，由它可得注册码剩余部分

    `HM=Int(ln(sqrt(W))*100.0)*CA+W*n-(D*n+CN)`

其中`n`是注册名长度减1、`CN`是注册名除第1字符外各字符ASCII值减`32h`后的总和（转换成浮点数），`Int`表示取整，`ln`表示求自然对数，`sqrt`表示求平方根。

第一次写这么多，感谢你读完全文！

月中人  
于2006年端午节

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
