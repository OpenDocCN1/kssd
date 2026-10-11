
# RegEditor 1.2

> 原文：[`www.pediy.com/kssd/pediy06/pediy6220.htm`](https://www.pediy.com/kssd/pediy06/pediy6220.htm)

## RegEditor 注册表修改器 1.2 注册算法简析

- **软件名称**：`RegEditor` 注册表修改器 `1.2`
- **软件简介**：`RegEditor` 注册表修改器 `1.2`
- **软件大小**：`1881KB`
- **软件语言**：简体中文
- **软件类别**：国产软件/共享版/系统设置
- **运行环境**：`Win9x`/`Me`/`NT`/`2000`/`XP`
- **加入时间**：`2003-9-6 16:14:57`
- **下载次数**：`189`
- **联系人**：`kjsg1@hotmail.com`
- **开发商**：未知

### 软件介绍

`RegEditor` 是一款注册表修改工具。用户无须打开注册表编辑器，软件提供许多常用的键值，自动对注册表进行修改。安全，方便，快捷。

**下载地址**：`http://www.onlinedown.net/soft/18496.htm`

---

### 破解信息

- **破解作者**：`yzez[DFCG]`
- **破解工具**：`0llydbg1.09`
- **破解目的**：本不为破解而破解，只因为技术而破解！

### 详细过程

1. 用 `0llydbg1.09` 载入程序，设断：`vbastrcomp`, 按 `F9`，出现注册画面后，输入假注册码：`123456654321`
2. 断下来后，不用管它，猛按 `F8` 走，直到出现你的注册序列号为止，下面我列出关键代码：

```
0043F713   . JGE      SHORT regedito.0043F727
0043F715   . PUSH     0A0
0043F71A   . PUSH     regedito.0040F09C
0043F71F   . PUSH     ESI
0043F720   . PUSH     EAX
0043F721   . CALL     DWORD PTR DS:[<&MSVBVM60.__vbaHr>;  MSVBVM60.__vbaHresultCheckObj
0043F727   > MOV      EDX, DWORD PTR SS:[EBP-18]
0043F72A   . PUSH     EDX
0043F72B   . CALL     DWORD PTR DS:[<&MSVBVM60.__vbaR8>;  MSVBVM60.__vbaR8Str
0043F731   . MOV      EAX, DWORD PTR SS:[EBP-1C] 把注册序列号移入 EAX，我的序列号是：705547511577
0043F734   . FSTP     QWORD PTR SS:[EBP-C8] 把输入的假注册码保存，ST=1.2345665432100000000E+11
SS:[EBP-C8]存放转换后的假注册码：1.2345665432100000000E+11
0043F73A   . PUSH     EAX 序列号入栈。
0043F73B   . CALL     DWORD PTR DS:[<&MSVBVM60.__vbaR8> 此 CALL 把序列号装入 ST(0)
0043F741   . FADD     QWORD PTR DS:[401368] 浮点数加：ST=7.0554751157700000000E+11，DS:[401368]
存放常数：123456789054.0000，ST=7.0554751157700000000E+11+123456789054.0000=8.2900430063100000000E+11
0043F747   . FADD     ST, ST 浮点数加：ST=ST+ST=8.2900430063100000000E+11+8.2900430063100000000E+11
=1.6580086012620000000E+12
0043F749   . FSUB     QWORD PTR DS:[401360] 浮点数减：DS:[401360]存放常数：111111111111.0000
ST=1.6580086012620000000E+12-111111111111.0000=1.5468974901510000000E+12
0043F74F   . FADD     QWORD PTR DS:[401358] 浮点数加：DS:[401358]存入常数：5674.000000000000
ST=1.5468974901510000000E+12+5674.000000000000=1.5468974958250000000E+12
0043F755   . FSTSW    AX
0043F757   . TEST     AL, 0D 测试 AL 是不是 0D，即 13。
0043F759   . JNZ      regedito.0043FBE4 不相等就跳。
0043F75F   . CALL     DWORD PTR DS:[<&MSVBVM60.__vbaFp>] 此 CALL 把上述运算的值保存到 ST
0043F765   . FCOMP    QWORD PTR SS:[EBP-C8] 比较指令，ST=1.5468974958250000000E+12，
SS:[EBP-C8]=123456654321.0000 这是我们输入的假码。比较什么？是不是可疑？
0043F76B   . FSTSW    AX
0043F76D   . TEST     AH, 40
0043F770   . JE       SHORT regedito.0043F779 相等就跳，跳到 0043F779，不相等当然就不跳!
0043F772   . MOV      EAX, 1 不跳，这里就赋注册成功标志位：1，所以上面运算得到的值就是我们
所需要的注册码，注册码如何取？0043F757 这里的与 13 比较说明了什么？很可能是注册码的位数，所以注册码
应该是 13 位，所以我的注册码是：1546897495825，输入进去，我们看一看：注册成功!
0043F777   . JMP      SHORT regedito.0043F77B
0043F779   > XOR      EAX, EAX 跳到这里，我们往下看看，有什么东东？
0043F77B   > LEA      ECX, DWORD PTR SS:[EBP-18]
0043F77E   . LEA      EDX, DWORD PTR SS:[EBP-1C]
```

#### 中间省略了一些代码!

```
0043FAC4   .  LEA      EDX, DWORD PTR SS:[EBP-54]
0043FAC7   .  PUSH     ECX
0043FAC8   .  LEA      EAX, DWORD PTR SS:[EBP-44]
0043FACB   .  PUSH     EDX
0043FACC   .  PUSH     EAX
0043FACD   .  LEA      ECX, DWORD PTR SS:[EBP-34]
0043FAD0   .  PUSH     30
0043FAD2   .  PUSH     ECX
0043FAD3   .  CALL     DWORD PTR DS:[<&MSVBVM60.#595>] F8 往下，我们到了这里!这里就给脸
色你看了：“错误的软件注册码!”
0043FAD9   .  LEA      EDX, DWORD PTR SS:[EBP-64]
0043FADC   .  EAX, DWORD PTR SS:[EBP-54]
0043FADF   .  PUSH     EDX
```

2. 好吧! 还是简单小结一下吧，算法也算简单的了，只是涉及到浮点运算：

   注册码 `SN = (序列号 + 123456789054) + (序列号 + 123456789054) - 111111111111 + 5674`

3. **验证注册算法**：

   以我的序列号为例。

   `SN = (705547511577 + 123456789054) * 2 - 111111111111 + 5674`
   `= 1658008601262 - 111111111111 + 5674`
   `= 1546897490151 + 5674`
   `= 1546897495825` 这就是我的注册码。

`4dd466d0e1e0cc123afbe504da2462ca`
