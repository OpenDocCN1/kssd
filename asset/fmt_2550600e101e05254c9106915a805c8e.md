
# 发个 ARM 指令集的 Crack 过程

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-195.htm`](https://www.pediy.com/kssd/pediy09/pediy09-195.htm)

> ARM 指令集大家应该都或多或少有点印象吧.
> 就是目前诺基亚等大部分手机厂商使用的指令集,构架在该结构上的称为 Symbian 操作系统.
> 比如 QD/7610/N70/N95/E50.....
> 

> 其中 Symbian 根据操作系统又细分成
> Symbian OS 6.x
> Symbian OS 7.x
> Symbian OS 8.x
> Symbian OS 9.x
> 

> 根据界面分为 UIQ2.0 3.0
> Seires 60 1st 2nd 3rd
> Seires 80
> 

> 考虑到如今大部分的手机是 Seires 60 2nd 的,我今天就献丑一下.叙述下如何 Crack 一个 Symbian OS 的程序.

> 首先扫个盲.
> ARM 汇编程序由机器指令\伪指令(符号,数据定义) 和宏指令组成.
> 基本的指令格式: `<opcode>{<cond>}{S} <Rd>,<Rn>('<opcode2>}`
> `opcode` 指令的助记符, `LDR` `.STR`
> `cond` 执行条件
> `S` 是否影响 `CPSR` 寄存器的值? 有则为 TRUN 没有则 FALSE
> `Rd` 目标寄存器
> `Rn` 第一个操作数的寄存器
> `opcode2` 第二个操作数

> 由于 CPU 的核心频率的限制,Symbian 的开发人员都尽量避免浮点运算.

> Symbian 开发语言有 `VS 2003(C++)` ,`CARBIDE C++`, `VB` , `CARBIDE J.` `JAVA`, `Pthoy`.
> 编译完成的主程序是`.app`,在 3RD 上是`.exe` ,其中安装文件是 `SIS`

> 好了,下面开始 Crack 一个 Symbian 程序的过程

> 程序名称:`colorpicker v1.00`
> 使用工具: `Ida Pro 5.0` `HEX` 计算器

> 新建一个工程. 选择 PDAs/Handhelds/Phones
> 然后选择`.sis` Symbian EPOC installation File.载入刚才那个 SIS 文件

> 分析完毕后.我们开始研究.

> 首先我们需要知道程序什么时候要求注册,Symbian 程序注册的原理很简单,就是通过安装手机的串号,就是 imei,也就是你在手机上拨\*#06#之后产生的代码.
> 而程序在做注册前必须读取这个数值,于是需要命令 `GetMachineIdL`

> 在汇编中搜索该单词.

```
.text:10000D98 BL PlpVariant::GetMachineIdL(TBuf<128> &)
```

> 此处获取了手机的串号

> 之后的代码:

```
.text:10000D98 BL PlpVariant::GetMachineIdL(TBuf<128> &)
.text:10000D98
.text:10000D9C ADD R0, R4, #0
.text:10000D9E MOV R1, SP
.text:10000DA0 BL TDes16::Copy(TDesC16 const &)
.text:10000DA0
.text:10000DA4 ADD SP, SP, #0x108
.text:10000DA6 POP {R4}
.text:10000DA8 POP {R0}
.text:10000DAA BX R0
.text:10000DAA
.text:10000DAA ; End of function sub_10000D88
```

> 注意,从`.text:10000DC2` 这开始
> ![](img/3b01371560193b12d71b80f328ffb2e4.jpg)

```
.text:10000DC2 MOV R5, #0 //R5 移动为 0
.text:10000DC4 MOV R6, #0 //R6 0
.text:10000DC6 MOV R7, #0 //R7 0
.text:10000DC6
.text:10000DC8
.text:10000DC8 loc_10000DC8 ; CODE XREF: sub_10000DAC+3Ej
.text:10000DC8 SUB R0, R7, #5 //R0=R7-5
.text:10000DCA CMP R0, #4
.text:10000DCC BLS loc_10000DE6 //如果 R0 比 4 小.JMP 到 10000DE6
.text:10000DCC
.text:10000DCE MOV R0, SP //R0 指针(IMEI 的位置)
.text:10000DD0 ADD R1, R7, #0
.text:10000DD2 BL TDesC16::AtC(int) // 获取 ASCII 代码
.text:10000DD2
.text:10000DD6 LDRH R0, [R0] //载入 R0 的值
.text:10000DD8 CMP R7, #0
.text:10000DDA BEQ loc_10000DE0
.text:10000DDA
.text:10000DDC LDR R4, =0x30B
.text:10000DDE ADD R5, R5, R4 //R5=R5+R4
.text:10000DDE
.text:10000DE0
.text:10000DE0 loc_10000DE0 ; CODE XREF: sub_10000DAC+2Ej
.text:10000DE0 LSL R0, R0, #1 //+2
.text:10000DE2 ADD R0, R6, R0 //R0 = R6 + R0 = 0 + R0
.text:10000DE4 ADD R5, R5, R0 //R5 = R5 + R0
.text:10000DE4
.text:10000DE6
.text:10000DE6 loc_10000DE6 ; CODE XREF: sub_10000DAC+20j
.text:10000DE6 ADD R7, #1 //R7 = R7 + 1
.text:10000DE8 CMP R7, #0xE
.text:10000DEA BLE loc_10000DC8 //如果 R7 小于 E ,JMP 到 10000DC8
.text:10000DEA
.text:10000DEC ADD R0, R5, #0 //R0 = R5 + 0
```

> 通过上面的代码,我们可以找到这么几个规律:
> 1.每次循环 R7 会+1
> 2.在 `10000DC8` 处 `R0` 移到 `R7 - 5`

> 这里 R0~R7 是 IMEI 中的数字

> 这时候用任意一个 HEX 计算器
> 在第一次循环时,(10000DD6)处,假设读取到的第一个 IMEI 是 3,3 的 ASCII 码为 33,之后 R7 根据条件跳转到 10000DE0,R0 此时值为 66,R5 为 66
> 第二次循环,5, `R0=35`,根据 R7 的判断,`R4=30B`.`R5=66+30B=371`,之后 R0 的值要加在 R5 上,`R5=3DB`
> ...................

> 之后你转换完成 R5 的值,既可以完成注册

> 此为算法

> 如果想制作一个随意数字完成注册的.就需要修改
> 注意这里:
> `.text:10000DEC ADD R0, R5, #0 ---> R0 = R5 + 0`
> 如果修改成:
> `.text:10000DEC MOV R0, #0`
> 那么此时 0 为正确的注册码

> 如果改成:
> `.text:10000DEC MOV R0, #1`
> 那么此时 1 为正确的注册码
> ............................

> 完毕.

> 自我介绍一下,黑手 K9998. QQ35333120 MAIL:K9998@foxmail.com
> 目前从事 Symbian 的开发,是搞汉化的![](img/c367a558167d303a470cbda3563c57f3.jpg "使眼色") ...
> 因为被 BT 壳所困![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!") ....所以过来参考.........
