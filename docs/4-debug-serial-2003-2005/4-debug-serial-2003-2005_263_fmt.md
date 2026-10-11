
# Super DVD Creator V5.02

> 原文：[`www.pediy.com/kssd/pediy06/pediy6289.htm`](https://www.pediy.com/kssd/pediy06/pediy6289.htm)

## Super DVD Creator V5.02 超简单算法

### 软件简介

Super DVD Creator V5.02

软件大小：2776 KB  
软件语言：英文  
软件类别：国外软件 / 共享版 / 视频转换  
应用平台：`Win9x/NT/2000/XP`  
界面预览：无  
加入时间：2003-10-30 15:56:07  
下载次数：2704  
推荐等级：  
开发商：`http://www.alldj.com/sdvdc/index.htm`

### 软件介绍

DVD/VCD/SVCD 制作软件，可以将 `Avi/Divx` 影片转换刻录为 `DVD/VCD/SVCD`，还可以将 `Avi/Divx` 转换为 `Mpeg` 格式。

### 下载地址

`http://www.skycn.com/soft/13309.html`

### 破解作者

`yzez[DFCG][BCG]`

### 破解工具

`OLLYDBG1.09`

### 破解目的

只为交流技术，请不要用于商业目的。

### 破解过程

1. 查壳，是 `ASPACK2.001` 的壳，想脱就脱吧! 我只写过程，看下面：

```
0040D8D5    CALL    MCreator.00516504
0040D8DA    LEA     EAX, DWORD PTR SS:[EBP-4]
0040D8DD    CALL    MCreator.0040C184*****我们在此 CALL 设断，为什么在此 CALL 设断，你看下面!一个跳转，一个出错!
0040D8E2    CMP     EAX, 15*******************EAX 存放输入的试验码的位数，输入的试验码要 15(转化成十六进制值是 21)位
0040D8E5    JE      MCreator.0040D971********相等就跳，一跳就有戏，不跳，跟你说 BYE-BYE
0040D8EB    MOV     WORD PTR SS:[EBP-48], 5C
0040D8F1    MOV     EDX, MCreator.00530252*****"Registration Code ErrorPlease Check it and try again0!"出错信息!
0040D8F6    LEA     EAX, DWORD PTR SS:[EBP-20]
0040D8F9    CALL    MCreator.00516338

************************省略部分代码!************************************************************

0040D958    MOV     EDX, 2
0040D95D    CALL    MCreator.00516504
0040D962    MOV     ECX, DWORD PTR SS:[EBP-58]
0040D965    MOV     DWORD PTR FS:[0], ECX
0040D96C    JMP     MCreator.0040DBF7
0040D971    MOV     WORD PTR SS:[EBP-48], 68********跳到这里，我们按 F8 往下走!
0040D977    LEA     EAX, DWORD PTR SS:[EBP-24]
0040D97A    CALL    MCreator.00401F68
0040D97F    PUSH    EAX
0040D980    INC     DWORD PTR SS:[EBP-3C]
0040D983    LEA     EAX, DWORD PTR SS:[EBP-4]
0040D986    MOV     ECX, 8
0040D98B    MOV     EDX, 5
0040D990    CALL    MCreator.0051673C
0040D995    LEA     EDX, DWORD PTR SS:[EBP-24]
0040D998    LEA     EAX, DWORD PTR SS:[EBP-8]
0040D99B    CALL    MCreator.00516534
0040D9A0    DEC     DWORD PTR SS:[EBP-3C]
0040D9A3    LEA     EAX, DWORD PTR SS:[EBP-24]
0040D9A6    MOV     EDX, 2

************************省略部分代码!************************************************************

0040D9EA    CALL    MCreator.00516504
0040D9EF    PUSH    DWORD PTR SS:[EBP-8]
0040D9F2    PUSH    DWORD PTR SS:[EBP-5C]
0040D9F5    CALL    MCreator.0040DBFC*****此 CALL 检测输入的试验码从第 5 位开始的 8 位试验码是否是数字!
0040D9FA    ADD     ESP, 8
0040D9FD    TEST    AL, AL************************如果输入的是 0--9 之间的数字，则 AL 的值为 1，否则这里就是 0
0040D9FF    JE      SHORT MCreator.0040DA17*相等就跳，一跳就请你回头!你看跳到下面出错了，这里不能跳!
0040DA01    PUSH    DWORD PTR SS:[EBP-C]****不跳往下，把试验码的最后 8 位入栈!
0040DA04    PUSH    DWORD PTR SS:[EBP-5C]
0040DA07    CALL    MCreator.0040DBFC
0040DA0C    ADD     ESP, 8
0040DA0F    TEST    AL, AL************************最后 8 位输入的也是数字 0--9 之间吗？是则置 AL 的值为 1，否则就为 0
0040DA11    JNZ     MCreator.0040DA9D*****不为 0 就跳，一跳你又可以笑两声，不能多笑!一定要跳!
0040DA17    MOV     WORD PTR SS:[EBP-48], 80
0040DA1D    MOV     EDX, MCreator.00530289*****"Registration Code ErrorPlease Check it and try again!3"又是出错信息!

************省略部分代码!************************************************************

0040DA98    JMP     MCreator.0040DBF7
0040DA9D    LEA     EAX, DWORD PTR SS:[EBP-8]****上面跳到这里
0040DAA0    CALL    MCreator.00516820
0040DAA5    MOV     DWORD PTR SS:[EBP-64], EAX**EAX 存放的是从第 5 位试验码开始取的 8 位(我的是 56789009)的十六进值
*******************************************************************03628811。把这个数值保存到：EBP-64 的内存地址处。
0040DAA8    LEA     EAX, DWORD PTR SS:[EBP-C]****赋 EAX 地址值
0040DAAB    CALL    MCreator.00516820*********此 CALL 把试验码的最后 8 位转化成十六进制值，我的是：12345678(BC614E)
0040DAB0    MOV     DWORD PTR SS:[EBP-68], EAX**把这个数值保存到：EBP-68 的内存地址处
0040DAB3    MOV     EDX, DWORD PTR SS:[EBP-64]***把 03628811 移入 EDX 中
0040DAB6    ADD     EDX, DWORD PTR SS:[EBP-68]***EDX=EDX+[EBP-68]存放的值=03628811+BC614E=41EE95F
0040DAB9    CMP     EDX, 4C10AE5*******************比较 41EE95F 与 4C10AE5 相等吗？
0040DABF    JE      MCreator.0040DB4B***********相等就跳，一跳你可以笑三声，也不能多笑!
0040DAC5    MOV     WORD PTR SS:[EBP-48], 8C*****不相等往下，你看，你又得回头，重新来过!
0040DACB    MOV     EDX, MCreator.005302C0*****"Registration Code ErrorPlease Check it and try again!2"出错啦!
0040DAD0    LEA     EAX, DWORD PTR SS:[EBP-30]
0040DAD3    CALL    MCreator.00516338
0040DAD8    INC     DWORD PTR SS:[EBP-3C]

************省略部分代码!************************************************************

0040DB46    JMP     MCreator.0040DBF7
0040DB4B    MOV     EAX, DWORD PTR SS:[EBP-64]****跳到这里同上处理，我看不懂，同样的比较为什么还要来一次？
0040DB4E    ADD     EAX, DWORD PTR SS:[EBP-68]
0040DB51    CMP     EAX, 4C10AE5
0040DB56    JNZ     SHORT MCreator.0040DBA3*****不相等就跳，一跳就失败，一定不能跳!
0040DB58    PUSH    DWORD PTR SS:[EBP-4]
0040DB5B    PUSH    DWORD PTR SS:[EBP-10]
0040DB5E    PUSH    DWORD PTR SS:[EBP-5C]
0040DB61    CALL    MCreator.0040DD10
0040DB66    ADD     ESP, 0C
0040DB69    MOV     WORD PTR SS:[EBP-48], 98
0040DB6F    MOV     EDX, MCreator.005302F7*****"Register Successfully ,Thanks for your using Super

********** DVD Creator!

The Software will Close the check to Registration Code,Please run it again."

**********不跳到这里你是不是成功了?!

0040DB74    LEA     EAX, DWORD PTR SS:[EBP-34]
0040DB77    CALL    MCreator.00516338
0040DB7C    INC     DWORD PTR SS:[EBP-3C]
0040DB7F    MOV     EAX, DWORD PTR DS:[EAX]
0040DB81    CALL    MCreator.004DE8DC
0040DB86    DEC     DWORD PTR SS:[EBP-3C]
0040DB89    LEA     EAX, DWORD PTR SS:[EBP-34]
0040DB8C    MOV     EDX, 2
0040DB91    CALL    MCreator.00516504
0040DB96    MOV     ECX, DWORD PTR DS:[53966C]        
0040DB9C    MOV     EAX, DWORD PTR DS:[ECX]
```

2. 也算是简单小结吧!

注册码的计算是这样的，从第 5 位开始取 8 位注册码(注意必须是 0--9 之间的数字)转化成十六进制值我们设为 `X`，再取最后 8 位注册码转化成十六进制值，我们设为 `Y`，则满足条件：`X+Y=4C10AE5` 注册就成功。

一个可用的注册码：`123456789009822971092`

`4dd466d0e1e0cc123afbe504da2462ca`
