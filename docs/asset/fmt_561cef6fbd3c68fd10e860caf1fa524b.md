
# 汇编代码分析

## 数据初始化

```
00415869  |.  C74424 1C 020>MOV DWORD PTR SS:[ESP+1C],2       把 2 存到内存单元中，[ESP+1C]=2
00415871  |.  894424 34     MOV DWORD PTR SS:[ESP+34],EAX
00415875  |.  C74424 20 060>MOV DWORD PTR SS:[ESP+20],6       把 6 存到内存单元中, [ESP+20]=6
0041587D  |.  894424 38     MOV DWORD PTR SS:[ESP+38],EAX
00415881  |.  C74424 24 F9F>MOV DWORD PTR SS:[ESP+24],-7      把-7 存到内存单元中,[ESP+24]=-7
00415889  |.  894424 3C     MOV DWORD PTR SS:[ESP+3C],EAX
0041588D  |.  C74424 28 040>MOV DWORD PTR SS:[ESP+28],4       把 4 存到内存单元中，[ESP+28]=4
00415895  |.  C74424 2C FFF>MOV DWORD PTR SS:[ESP+2C],-1      把-1 存到内存单元中，[ESP+2C]=-1
0041589D  |.  C74424 30 080>MOV DWORD PTR SS:[ESP+30],8       把 8 存到内存单元中，[ESP+30]=8
```

## 核心算法逻辑

```
004158A5  |.  894424 40     MOV DWORD PTR SS:[ESP+40],EAX     下面要开始算法了，我见过最简单的算法。
004158A9  |.  8D7C24 1C     LEA EDI,DWORD PTR SS:[ESP+1C]     这是传第一个初值数的地址。
004158AD  |>  8B4C24 0C     /MOV ECX,DWORD PTR SS:[ESP+C]     把 54321 移到 ECX，ECX=54321
004158B1  |.  8A17          |MOV DL,BYTE PTR DS:[EDI]         这是寄存器间接寻址，把 2 移到 DL，DL=2
004158B3  |.  8A040E        |MOV AL,BYTE PTR DS:[ESI+ECX]     把第一个数 5（35）移到 AL 里，AL=35
004158B6  |.  02C2          |ADD AL,DL                         AL=AL（35）+DL（2）=37
004158B8  |.  3C 30         |CMP AL,30                         判断是否大于 30（即 0）
004158BA  |.  884424 14     |MOV BYTE PTR SS:[ESP+14],AL       把 AL 的值存起来，[ESP+14]=AL=37
004158BE  |.  7D 06         |JGE SHORT Unpacked.004158C6       大于就跳，不大就往下走。
004158C0  |.  04 0A         |ADD AL,0A                         把不大于的值加 A
004158C2  |.  884424 14     |MOV BYTE PTR SS:[ESP+14],AL       把 AL 的值存起来，[ESP+14]=AL
004158C6  |>  3C 39         |CMP AL,39                         判断 AL 的值，是否小于 39（即，9），小于就跳
004158C8  |.  7E 06         |JLE SHORT Unpacked.004158D0
004158CA  |.  04 F6         |ADD AL,0F6                        不小于的话，AL=AL+0F6
004158CC  |.  884424 14     |MOV BYTE PTR SS:[ESP+14],AL       把 AL 存起来，[ESP+14]=AL
004158D0  |>  8B5424 14     |MOV EDX,DWORD PTR SS:[ESP+14]
004158D4  |.  8D4C24 0C     |LEA ECX,DWORD PTR SS:[ESP+C]
004158D8  |.  52           |PUSH EDX
004158D9  |.  56           |PUSH ESI
004158DA  |.  E8 27B00800   |CALL Unpacked.004A0906            这个 CALL 是把 AL 转化为相应 ASCii 字符，即 37=7
004158DF  |.  46           |INC ESI
004158E0  |.  83C7 04       |ADD EDI,4                         这是移到下个初值的内存单元。
004158E3  |.  83FE 05       |CMP ESI,5                         判断是否取完 5 个数了？没有继续循环。
004158E6  |.^ 7C C5         \JL SHORT Unpacked.004158AD        后面 4 个数不加了，code[1]+2,code[2]+6,code[3]+F9,code[4]+4,code[5]+FF
```

## 结果处理

```
004158E8  |.  8D4C24 0C     LEA ECX,DWORD PTR SS:[ESP+C]       算完后 EAX=70660
004158EC  |.  E8 03B00800   CALL Unpacked.004A08F4            这个 CALL 再把算好的注册码，再颠倒回去,EAX=06607,
004158F1  |.  8B7424 10     MOV ESI,DWORD PTR SS:[ESP+10]     取出计算后的值到 ESI，ESI=06607
004158F5  |.  8B4424 0C     MOV EAX,DWORD PTR SS:[ESP+C]       EAX=67890
004158F9  |>  8A10          /MOV DL,BYTE PTR DS:[EAX]
004158FB  |.  8A1E          |MOV BL,BYTE PTR DS:[ESI]          后面是真假注册码，逐位比较，不等就 OVER 了。
004158FD  |.  8ACA          |MOV CL,DL
004158FF  |.  3AD3          |CMP DL,BL
00415901  |.  75 1E         |JNZ SHORT Unpacked.00415921
00415903  |.  84C9          |TEST CL,CL
00415905  |.  74 16         |JE SHORT Unpacked.0041591D
00415907  |.  8A50 01       |MOV DL,BYTE PTR DS:[EAX+1]
0041590A  |.  8A5E 01       |MOV BL,BYTE PTR DS:[ESI+1]
0041590D  |.  8ACA          |MOV CL,DL
0041590F  |.  3AD3          |CMP DL,BL
00415911  |.  75 0E         |JNZ SHORT Unpacked.00415921
00415913  |.  83C0 02       |ADD EAX,2
00415916  |.  83C6 02       |ADD ESI,2
00415919  |.  84C9          |TEST CL,CL
0041591B  |.^ 75 DC         \JNZ SHORT Unpacked.004158F9
0041591D  |>  33C0          XOR EAX,EAX
0041591F  |.  EB 05         JMP SHORT Unpacked.00415926
```

--------------------------------------------------------------------------------

# 破解总结

## 用户名与捐助码关系
用户名和捐助码无关，捐助码必须等于 10 位，分别取 5 位 4 位 3 位 2 位 1 位进行运算。

## 运算规则
后 5 位是经过前 5 位所算出来的。运算如下（运算过程中取的都是低位，这里容易出错）：

`CODE = code[1]code[2]code[3]code[4]code[5]SUM5SUM4SUM3SUM2SUM1` (十位数，前 5 位取 0-9，最好算）

*   `SUM1 = toascii(code[5]) + 2`
    *   如果 `code[5]` 是“!、@、#、$、*等”，`SUM1 = SUM1 + A`
    *   如果 `SUM1` 大于 39 还要加 `F6`。并取 `SUM1` 低位，转化成相应 `ASCii` 字符
*   `SUM2 = toascii(code[4]) + 6`
    *   如果 `code[4]` 是“!、@、#、$、*等”，`SUM2 = SUM2 + A`
    *   如果 `SUM2` 大于 39 还要加 `F6`。并取 `SUM2` 低位，转化成相应 `ASCii` 字符
*   `SUM3 = toascii(code[3]) + F9`
    *   如果 `code[3]` 是“!、@、#、$、*等”，`SUM3 = SUM3 + A`
    *   如果 `SUM3` 大于 39 还要加 `F6`。并取 `SUM3` 低位，转化成相应 `ASCii` 字符
*   `SUM4 = toascii(code[2]) + 4`
    *   如果 `code[2]` 是“!、@、#、$、*等”，`SUM4 = SUM4 + A`
    *   如果 `SUM4` 大于 39 还要加 `F6`。并取 `SUM4` 低位，转化成相应 `ASCii` 字符
*   `SUM5 = toascii(code[1]) + FF`
    *   如果 `code[1]` 是“!、@、#、$、*等”，`SUM5 = SUM5 + A`
    *   如果 `SUM5` 大于 39 还要加 `F6`。并取 `SUM5` 低位，转化成相应 `ASCii` 字符

## 后话
这个算法比较简单，所以破文写的还是详细，难度还没有 `Crackme` 大，只要加上几个固定数转化下就出来了。

算法分析，汇编基本功很重要，七种寻址方式，N 种算术逻辑运算，都要很好掌握，以后研究起算法来才不觉得吃力。

写算法分析文章，就是我写的累，你看得也累，呵呵^_^,感谢你看完全文！

--------------------------------------------------------------------------------

# 用户名、密码

*   用户名：`zmw1224`
*   捐助码：`1234506607` (给个 5 发的码 `8888872140`)

--------------------------------------------------------------------------------

# 版权声明

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
