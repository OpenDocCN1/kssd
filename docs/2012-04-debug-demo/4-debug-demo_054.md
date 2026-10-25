
# Keymake 注册机编写器使用指南

> 原文：[www.pediy.com/kssd/tutorial/chap9-3-2.htm](https://www.pediy.com/kssd/tutorial/chap9-3-2.htm)
>
> **Keymake 使用（作者：刘健英 知软网）**

这个注册机编写器以前一直是我自己为写注册机而编写的，通过它只要略有汇编基础很快就能写出一个注册机。而不需要再过多的了解程序的指令算法。 整个程序体实际上只是我用汇编写的一个模板。所以大家也可以在其中自定义自己的界面和提示信息。可以用 `VC++` 或 `BC++` 等资源编辑工具自行修改 `key1.res` 资源文件，但请不要修改它们对应的 `ID` 号。

点击此下载例子程序，主要用来举例说明这个程序的使用（我先假设自己并不太懂 `Win32` 汇编）。

通过动态调试或反汇编例子程序可以得到以下注册码的计算过程：

```
xxxx:00401077 CALL GetCommandLineA
xxxx:0040107C CMP BYTE PTR [EAX],22
xxxx:0040107F JNZ 401082
xxxx:00401081 INC EAX
xxxx:00401082 MOV CX,WORD PTR [EAX]
xxxx:00401085 MOV WORD PTR [0040306C],CX
xxxx:0040108C MOV WORD PTR [0040306E],5C
xxxx:00401095 PUSH 0
xxxx:00401097 PUSH 0
xxxx:00401099 PUSH 0
xxxx:0040109B PUSH 0
xxxx:0040109D PUSH DWORD 00403058
xxxx:004010A2 PUSH 0
xxxx:004010A4 PUSH 0
xxxx:004010A6 PUSH DWORD 0040306C
xxxx:004010AB CALL GetVolumeInformationA
...
xxxx:0040111E MOV EAX,1
xxxx:00401123 CPUID
xxxx:00401125 MOV ECX,DWORD PTR [00403058]
xxxx:0040112B XOR EDX,EDX
xxxx:0040112D MUL ECX
xxxx:0040112F ADD EAX,EDX
xxxx:00401131 PUSH EAX
xxxx:00401132 PUSH DWORD 0040303E    ; 在这里下 D 40303E 可以看到数据窗口中显示为“%1X”
xxxx:00401137 PUSH DWORD 0040305C
xxxx:0040113C CALL wsprintfA
```

对于以上的指令并不需要过多的了解它在干什么，只要将其中的每个地址改成一个变量地址的声明，然后再原封不动的抄到注册机编写器的代码窗口中即可。

这是写好的声明：

```
a1 dd 0 ; 这是一个双字的内存空间，对应于上面的 403058。
;（因为 40109D 处的指令是 DWORD 403058，所以用 dd，如果是 WORD 就用 dw，如果是 BYTE 就用 db）

a2 dd 0 ; 对应于上面的 40306C

a3 db "%1X",0 ; 对应于上面的 40303E 指向的字符串

a4 db 20 dup (0) ; 这是 20 个字节的内存空间，用来存放输出的注册码，对应于上面的 40305C
```

输入如图所示：

![](img/dbb4999574c72f812929b4b3e066e11d.jpg)

这是写好的程序代码：

```
CALL GetCommandLineA
CMP BYTE PTR [EAX],22h ; 后面加 h 表示是十六进制

JNZ n1
INC EAX

n1:
MOV CX,WORD PTR [EAX]
MOV WORD PTR a2,CX
MOV WORD PTR a2+2,5Ch

PUSH 0
PUSH 0
PUSH 0
PUSH 0
LEA EAX,a1 ; 令 EAX 指向 a1
PUSH EAX
PUSH 0
PUSH 0
LEA EAX,a2
PUSH EAX ; 令 EAX 指向 a2

CALL GetVolumeInformationA
; 当然这几条语句也可以直接写成 invoke GetVolumeInformationA,addr a2,0,0,addr a1,0,0,0,0 的形式

MOV EAX,1
CPUID
MOV ECX,a1
XOR EDX,EDX
MUL ECX
ADD EAX,EDX
PUSH EAX

LEA EAX,a3 ; 令 EAX 指向 a3，也就是指向字符“%1X”。
PUSH EAX
LEA EAX,a4
PUSH EAX

CALL wsprintfA
LEA EAX,a4 ; 令 EAX 指向 a4。因为程序最后显示的就是 EAX 寄存器所指向的内存地址的数据。
```

输入如下图所示：

![](img/00787e7c9abaaf48b3badd97c01b93aa.jpg)

可以看到这与上面的反汇编代码基本相同。点击编译就可以很容易的得到一个注册机了。

生成的注册机如下图所示（这只是我写的外观，你也可以自行修改）：

![](img/add0bd48b42cbe0eba67cc25e4211487.jpg)

需要说明的是因为这只是我写的一个模板，所以我也在程序中使用了几个变量。 请不要再重复声明：`hCursorHandle`、`hInstance`、`hIcon`、`hTempEbp`、`hInput`、`hMode` 以免出错。 还有就是你一定要在指令结束时令 `EAX` 指向正确的注册码地址。

这种算注册码的方法不是直接从用户所输入的序列号来计算注册码的，所以并不适用于所有情况。有时我们可能需要根据用户所填入的序列号来计算注册码。考虑到这种情况所以我在程序初使时令 `EAX` 指向第一个编辑框（也就是输入序列号的窗口）收到用户输入的数据，令 `EBX` 指向第二个编辑框收到用户输入的数据，令 `ECX` 指向第三个编辑框收到用户输入的数据。你如果要对用户输入的序列号或用户名等信息进行操作，那么可以就对 `EAX`、`EBX`、`ECX` 进行操作。

下面的代码是本例根据用户所填入的序列号来计算注册码的方法：

```
MOV ECX,EAX ; EAX 指向用户输入的八位序列号，现在暂将它移动到 ECX 寄存器
XOR EBX,EBX ; 以下是一段典型的将内存中的 ASCII 码转换为十六进制代码。

n1:
MOVZX EAX,BYTE PTR [ECX]
OR AL,AL
JZ n3
CMP AL,3Ah
JC n2
SUB AL,7

n2:
SUB AL,30h
SHL EBX,4
ADD EBX,EAX
INC ECX
JMP n1

n3: ; 我想以上的一段代码应该懂汇编就能写的出（如果不是很明白，那么你对我编写好的注册机用 TRW 调试一次就知道了）。
PUSH EBX ; 最后的十六进制结果都保存在寄存器 EBX 中
; 将 EBX 入栈，这是为了将 EBX 寄存器的数据保存起来。因为经过 CPUID 这个指令后 EBX 的值将被修改。
; 然后后面就和上面一样照抄程序中的指令。以下计算注册码的方法很简短。但在实际的破解过程中，程序的算法可能会相当复杂。
; 这就需要你通过调试或反汇编将关键的运算指令都找出来，再写进来。

MOV EAX,1
CPUID
POP ECX ; 恢复 EBX 的值到 ECX 寄存器
XOR EDX,EDX
MUL ECX
ADD EAX,EDX
PUSH EAX

LEA EAX,a3 ; 令 EAX 指向 a3，也就是指向字符“%1X”。
PUSH EAX
LEA EAX,a4
PUSH EAX

CALL wsprintfA
LEA EAX,a4 ; 令 EAX 指向 a4。因为程序最后显示的就是 EAX 寄存器所指向的内存地址的数据。
```

下图分别是采用方案二和方案三编译后的窗口界面：

![](img/89e0ae4fad5281a5724292f798813ed1.jpg)
![](img/e2322f810ff5d7c50e500b013d30bfff.jpg)

如果你不采用这种方法编写那么也可以使用程序中的另一个功能“另类注册机”。它和 `CrackCode2000` 一样，也是通过拦截程序指令并显示出注册码。 我之所以再写一个是因为以前的 `CrackCode2000` 提供的拦截选项非常有限，而且也不对 `Wide` 格式（出现在 `VB` 程序中用 `00` 将 `ASCII` 码分隔开）提供支持，还有一个毛病是在有些程序中会出现莫名的非法操作或无法获得注册码的情况（就好比这个例子程序）。 另外它的界面也太简陋了，只是一个消息框。所以我用汇编写了这个另类注册机非常小巧，它最多可以设置 `100` 个断点（其实可以更多，但好象没这个必要），以及在每个断点处中断多少次，如下图所示：

![](img/f0b3ef447c804689fe9b2fceba965f67.jpg)

对于以上界面的说明：程序名称和中断地址的设置，这个我想用不着多说，会破解的人都懂。 而获取注册码的方式上需要说明一下，我是这样考虑的：寄存器方式－因为注册码可能是会放在寄存器中比较，并且可能经过十进制或十六进制的转换， 所以就提供了一个寄存器方式及十进制或十六进制的选项。 内存方式－是指：如设置为 `EDX`，即指注册码保存在 `EDX` 所指向的内存地址中（而不是寄存器里）这样就为获取注册码提供了更多的选择。

可以自定义在弹出的对话框中将显示的主页、邮件和窗口标题，如下图所示：

![](img/4a135ba695b7e677e8e45a68402bdd5d.jpg)

生成的注册机有二个可选界面如下图所示：

![](img/d4fe4f907cfec2bee1d456825bad4bf0.jpg)
![](img/1433abd15fa8543e6593f32de917a5af.jpg)

需要说明的一点是这个对话框只在拦截成功的时候出现。

除此之外还有一个制作破解补丁的附加工具，制作界面如图所示。

![](img/9e6fa6efc0958bcd0aa6319930015362.jpg)

这是完成界面。

![](img/8a266c1b4152032427f8a55511a86663.jpg)
