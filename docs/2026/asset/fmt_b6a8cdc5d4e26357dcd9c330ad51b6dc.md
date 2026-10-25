
# [原创]010editor 8.0算法分析[附注册机]

## 1. 定位到注册算法函数处

此处步骤略，注册算法函数在地址`013BD900`，参数情况为：`arg1 = 0x9 ;arg2 = 0x4389;`如果要注册成功，需要其返回`0xDB`，所以需要进入`case:0x2D`，需要函数`0040A82B`返回`0x2D`。

```asm
013BE220      PUSH EBP
013BE221      MOV EBP,ESP
013BE223      PUSH ESI
013BE224      MOV ESI,ECX
013BE226      CMP DWORD PTR DS:[ESI+0x2C],0x0
013BE22A  |.  JE SHORT 010Edito.013BE236
013BE22C  |.  MOV EAX,0x113
013BE231  |.  POP ESI
013BE232  |.  POP EBP
013BE233  |.  RETN 0x8
013BE236  |>  PUSH EDI
013BE237  |.  PUSH [ARG.2]
013BE23A  |.  MOV EDI,[ARG.1]
013BE23D  |.  PUSH EDI
013BE23E  |.  CALL 010Edito.0040A82B
013BE243  |.  CMP EAX,0x2D                       ;  Switch (cases 2D..E7)
.................................
013BE2EF  |>  POP EDI                            ;  Case 2D of switch 013BE243
013BE2F0  |.  MOV EAX,0xDB
013BE2F5  |.  POP ESI
013BE2F6  |.  POP EBP
013BE2F7  \.  RETN 0x8
```

这里需要进入函数`0040A82B`。

## 2. 整体轮廓

### 2.1 注册码的保存形式

进入函数`0040A82B`，函数首先将注册码按每两位16进制数的形式存入局部变量数组中：

```asm
013BD96D MOV ECX,EDI
013BD96F PUSH EAX
013BD970 CALL 010Edito.00409B74
```

比如输入注册码`12345678876543210011`，对应：

```asm
[ebp-0x24]  KEY[0] = 0x12
[ebp-0x23]  KEY[1] = 0x34
......
[ebp-0x1B]  KEY[9] = 0x11
```

### 2.2 对注册码的处理和函数00402E50(处理用户名)的调用

接下来，寄存器`BL = KEY[3]`，寄存器`BH = KEY[5]`；根据`KEY[3]`的值进行注册类型的判断，有`0X9C`、`0XFC`、`0XAC`三种情况。每种类型会对`esi`、`DWORD PTR DS:[EDI+0x20]`等进行赋值，影响后面的操作。由于篇幅有限，所以这里不逐句写汇编，更加详细的写在附件文档中。为了有整体的了解，这里先跳过每种类型的处理，来观察在处理后（也就是上面的值都有了着落）调用用户名处理函数的汇编代码：

调用处：

```asm
013BDAD8   .  PUSH DWORD PTR DS:[EDI+0x20]   ;参数4:DWORD PTR DS:[EDI+0x20]
013BDADB   .  XOR EAX,EAX ;
013BDADD   .  MOV DWORD PTR SS:[EBP-0x4],0x0
013BDAE4   .  CMP BL,0xFC    ;由前面可知BL为KEY[3]
013BDAE7   .  LEA ECX,DWORD PTR SS:[EBP-0x14]
013BDAEA   .  PUSH ESI ;ESI ;参数3,ESI
013BDAEB   .  SETNE AL ;如果BL != 0xFC, AL = 1
013BDAEE   .  PUSH EAX ; ;参数2:这里根据KEY[3]的值和0xFC比较,如果不是0xFC类型就设置为1,否则为0
013BDAEF   .  CALL DWORD PTR DS:[<&Qt5Core.?data>;  Qt5Core.?data@QByteArray@@QAEPADXZ// 这个函数没有参数,传出用户名
013BDAF5   .  PUSH EAX     ;参数1:用户名
013BDAF6   .  CALL 010Edito.00402E50    ;处理用户名
```

之后会根据`CALL 010Edito.00402E50`的返回值和其他一些值进行判断给`eax`赋值，最后程序返回。

### 2.3 最后，更多有用的信息：

为了有整体的了解，先跳过处理用户名函数中的具体内容，我们来观察最后程序是怎样进行处理的。这里由于确定了返回值需要为`0x2D`，所以从下往上看，可以跟着序号看，倒着看可以看到`013BDB24`地址处，可以得出结论：

1. `0xFC`情况无法返回正确值。
2. `0x9C`情况：需要满足`[ EBP + 0x8 ] <= [ EDI + 0x1C ]`，等价于参数1也就是`9`应该小于等于`[ EDI + 0x1C ]`。
3. `0xAC`情况：需要满足`[ EBP - 0x10 ] >= [ EBP + 0xC ]`，等价于`[ EBP - 0x10 ]`大于等于参数2也就是`0x4389`。

具体的`0x9C`情况中的`[ EDI + 0x1C ]`和`0xAC`情况的`[ EBP - 0x10 ]`的值各是多少，需要结合这两种情况的汇编代码来看，这里先放放。

接着我们从前往看，若定义`char result[4]`，`*(DWORD*)result = 处理用户名函数()`；则：

- `KEY[ 4 ] = result[ 0 ]`
- `KEY[ 5 ] = result[ 1 ]`
- `KEY[ 6 ] = result[ 2 ]`
- `KEY[ 7 ] = result[ 3 ]`

这些信息对最后写注册机很有帮助。

```asm
013BDAF6   .  CALL 010Edito.00402E50   ;处理用户名
013BDAFB   .  MOV EDX,EAX
013BDAFD   .  ADD ESP,0x10
013BDB00   .  CMP BYTE PTR SS:[EBP-0x20],DL ;KEY[4]和返回值比较
013BDB03   .  JNZ 010Edito.013BDB8A ;应该相等
013BDB09   .  MOV ECX,EDX
013BDB0B   .  SHR ECX,0x8
013BDB0E   .  CMP BH,CL  ;返回值>>0x8 和 KEY[5]比较
013BDB10   .  JNZ SHORT 010Edito.013BDB8A;应该相等
013BDB12   .  MOV ECX,EDX
013BDB14   .  SHR ECX,0x10
013BDB17   .  CMP BYTE PTR SS:[EBP-0x1E],CL;返回值>>0x10 和 KEY[6]比较
013BDB1A   .  JNZ SHORT 010Edito.013BDB8A;应该相等
013BDB1C   .  SHR EAX,0x18
013BDB1F   .  CMP BYTE PTR SS:[EBP-0x1D],AL;返回值>>0x18 和 KEY[7]比较
013BDB22   .  JNZ SHORT 010Edito.013BDB8A;应该相等
013BDB24   .  CMP BL,0x9C                        ;  Switch (cases 9C..FC)
013BDB27   .  JNZ SHORT 010Edito.013BDB38
013BDB29   .  MOV EAX,DWORD PTR SS:[EBP+0x8]     ;  Case 9C of switch 013BDB24
013BDB2C   .  CMP EAX,DWORD PTR DS:[EDI+0x1C]
013BDB2F   .  JBE SHORT 010Edito.013BDB83   // <= 跳到013BDB83,即正确处③
013BDB31   .  MOV ESI,0x4E
013BDB36   .  JMP SHORT 010Edito.013BDB8F
013BDB38   >  CMP BL,0xFC   ;
013BDB3B   .  JNZ SHORT 010Edito.013BDB6B
013BDB3D   .  MOVZX ECX,BYTE PTR SS:[EBP-0x22]   ;  Case FC of switch 013BDB24
...........;期间无跳到正确答案处
013BDB70   .  MOV EAX,DWORD PTR SS:[EBP-0x10]    ;  Case AC of switch 013BDB24
013BDB73   .  TEST EAX,EAX
013BDB75   .  JE SHORT 010Edito.013BDB8A
013BDB77   .  CMP DWORD PTR SS:[EBP+0xC],EAX
013BDB7A   .  JBE SHORT 010Edito.013BDB83   //<=跳到013BDB83,即正确处③
013BDB7C   .  MOV ESI,0x4E
013BDB81   .  JMP SHORT 010Edito.013BDB8F
013BDB83   >  MOV ESI,0x2D  // 对ESI赋值0x2D ②
013BDB88   .  JMP SHORT 010Edito.013BDB8F
013BDB8A   >  MOV ESI,0xE7                       ;  Default case of switch 013BDB24
013BDB8F   >  LEA ECX,DWORD PTR SS:[EBP-0x14]
013BDB92   .  MOV DWORD PTR SS:[EBP-0x4],-0x1
013BDB99   .  CALL DWORD PTR DS:[<&Qt5Core.??1QB>;  Qt5Core.??1QByteArray@@QAE@XZ
013BDB9F   .  MOV EAX,ESI  // 设置返回值 = ESI ①
013BDBA1   .  MOV ECX,DWORD PTR SS:[EBP-0xC]
013BDBA4   .  MOV DWORD PTR FS:[0],ECX
013BDBAB   .  POP ECX
013BDBAC   .  POP EDI
013BDBAD   .  POP ESI
013BDBAE   .  POP EBX
013BDBAF   .  MOV ESP,EBP
013BDBB1   .  POP EBP
013BDBB2   .  RETN 0x8
```

## 3. 疑惑的解开

2.3中提到的`0x9C`情况中的`[ EDI + 0x1C ]`和`0xAC`情况的`[ EBP - 0x10 ]`的值各是多少？？？各自的限制条件又有什么用？？？

这里就需要看在处理用户名函数之前对注册码的处理了，这里由于篇幅只列出伪代码，更详细的见附件。另外，在处理完这些代码后，我们就能得到函数`00402E50`（也就是用户名处理函数）的参数`ESI`和`DWORD PTR DS:[EDI+0x20]`。

### 3.1 对于0x9c情况

```cpp
case 0x9C:
{
DWORD EAX = (((((KEY[ 1 ] ^ KEY[ 7 ]) * 0x100) + (KEY[ 2 ] ^ KEY[ 5 ])) ^ 0x7892) + 0x4d30) ^ 0x3421;
DWORD ECX = (((KEY[ 0 ] ^ KEY[ 6 ]) ^ 0x18) + 0x3D) ^ 0xA7;
if (EAX % 0xB != 0)
{
EAX = 0;//error
}
if (EAX == 0 || EAX > 0x3E8)
{
return;//error
}
if(ECX == 0)
{
return;//error
}
if (ECX < 0x2)// 这里ECX如果大于0x2 ESI=0否则=ECX=1,至于ECX的值是多少,在后面可以看到
{
// ECX == 1,ESI=1
ESI = ECX;
}
else
{
ESI = 0;
}
DWORD PTR DS:[EDI+0x1C] = ECX = ((((KEY[0] xor KEY[6]) xor 0x18) + 0x3D)xor 0xA7);
DWORD PTR DS:[EDI+0x20] =
((((((KEY[ 1 ] ^ KEY[ 7 ]) * 0x100) + (KEY[ 2 ] ^ KEY[ 5 ])) ^ 0x7892) + 0x4d30) ^ 0x3421)/0xB;
break;
}
```

可以看出`ESI`可能等于`ECX`或者`0`，具体等于多少呢，我们看本文2.3得出的一个结论：

`0x9C`情况：需要满足`[ EBP + 0x8 ] <= [ EDI + 0x1C ]`，等价于参数1也就是`9`应该小于等于`[ EDI + 0x1C ]`。这个至关重要，由上面的条件，由于`ECX = DWORD PTR DS:[EDI+0x1C]`，所以`ECX >= 9`，所以`ESI`只能等于`0`。

所以得出结论：

对于`0x9C`情况，函数`00402E50`（也就是用户名处理函数）的参数情况为：

- 参数3 `ESI`：`0`
- 参数4 `[EDI+0x20]`：`((((((KEY[ 1 ] ^ KEY[ 7 ]) * 0x100) + (KEY[ 2 ] ^ KEY[ 5 ])) ^ 0x7892) + 0x4d30) ^ 0x3421)/0xB`
- 参数2（只要非`0xFC`就是`1`）：`1`
- 参数1：用户名

以及：

- `DWORD PTR DS:[EDI+0x1C] = ECX = ((((KEY[0] xor KEY[6]) xor 0x18) + 0x3D)xor 0xA7) >= 9`
- `((((((KEY[ 1 ] ^ KEY[ 7 ]) * 0x100) + (KEY[ 2 ] ^ KEY[ 5 ])) ^ 0x7892) + 0x4d30) ^ 0x3421)` 能够被`0xB`整除

### 3.2 对于0xac情况

```cpp
case 0xAC:
{
DWORD EAX = (((((KEY[ 1 ] ^ KEY[ 7 ]) * 0x100) + (KEY[ 2 ] ^ KEY[ 5 ])) ^ 0x7892) + 0x4d30) ^ 0x3421;
if(EAX % 0xB != 0)
{
EAX = 0;//error
}
if(EAX == 0 || EAX > 0x3E8)
{
return;//error
}
DWORD ECX = (((((((((KEY[ 9 ] ^ KEY[ 5 ]) << 8) + (KEY[ 4 ] ^ KEY[ 5 ])) << 8) + (KEY[ 6 ] ^ KEY[ 0 ])) ^ 0x05B8C27) ^ 0x22c078) - 0x2C175) ^ 0xFFE53167) & 0xFFFFFF;
// 这里EAX被重新复制
EAX = 0xF0F0F0F1;
DWORD EDX = 0;
伪代码:EDX:EAX = EAX*ECX;
EDX = EDX >> 0x4;
EAX = (EDX << 0x4) + EDX;
ECX = ECX - EAX;
if (ECX == 0)// ECX = EAX
{
EAX = EDX;//right
}
else
{
EAX = 0;//error
}
// 结果:
EDI0x20 = ((((((KEY[ 1 ] ^ KEY[ 7 ]) << 2) + (KEY[ 2 ] ^ KEY[ 5 ])) ^ 0x7892) + 0x4d30) ^ 0x3421)/0xB;
// 这里的EAX是最后结尾的EAX,也就是上面的判断中的EAX
伪代码:DWORD PTR SS:[EBP-0x10] = ESI = EAX;
break;
}
```

程序很复杂，中间的一个乘法（伪代码：`EDX:EAX = EAX*ECX;`）令人崩溃，但是后面发现还是有解决方案的。简单的说，前面和`0x9c`的差不多，同样`(((((KEY[ 1 ] ^ KEY[ 7 ]) * 0x100) + (KEY[ 2 ] ^ KEY[ 5 ])) ^ 0x7892) + 0x4d30) ^ 0x3421`要被`0xB`整除，且`0 < 值 < 0x3E8`，但是后面的就大不相同了，`ESI`的值很难确定.........

这时候我们来看看`DWORD PTR SS:[EBP-0x10] = ESI = EAX`，其中`EAX`有上面的`if else`来决定。回头看本文2.3的结论3：`0xAC`情况：需要满足`[ EBP - 0x10 ] >= [ EBP + 0xC ]`，等价于`[ EBP - 0x10 ]`大于等于参数2也就是`0x4389`。也就是说`WORD PTR SS:[EBP-0x10] = ESI = EAX`，这三个都要大于`0x4389`，就不可能等于`0`。那么回到上面的代码中，`EAX != 0 >> EAX = EDX >> ECX == EAX`，那么这里就可以暴力获取了，思路为：设置`ECX = 0xFFFFFF`，接下来按上面代码走，打印的条件为`EAX = ECX`，这样就可以获得最后的`DWORD PTR SS:[EBP-0x10]`和`ESI`，以及对应的`ECX`（用于后面写注册机）。

具体代码为：

```cpp
VOID main()
{
    unsigned int num_ecx = 0xFFFFFF;
    unsigned int num_eax = 0;
    unsigned int num_edx = 0;
    while(1)
    {
        __asm {
            MOV ECX , num_ecx;
            MOV EAX , 0XF0F0F0F1;
            MUL ECX;
            SHR EDX , 0x4 ;
            MOV EAX , EDX;
            SHL EAX , 0x4;
```
