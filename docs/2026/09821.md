
# VB输入限制的内存破解 (2千字)

破解的程序：`Windows超级设置723`。

前几天在论坛上看了一篇破解文本框输入限制的文章，得到些启示，于是有了这篇破文。

运行程序，提示注册，可输入框竟不能输入，尝试用粘贴法也行。注册码很容易找到，用`__vbastrcomp`比较，`ollydbg`跟踪即可看到。可无论如何无法输入。555!!!

跟踪`regform_load`，发现不像是`text1.enabled=false`，因为如果那样，输入框没有光标闪动，且不接受右键点击。反汇编找`__vbaobjectset`，看看前面是否有`push 00`(`enabled=false`)没找到。用`vbded`反编译也没看出什么东西。看来是另做了手脚。

没办法，只好想办法改内存来给它数据。以下用另一VB程序来模拟输入。

```asm
00413013
LEA ECX,DWORD PTR SS:[EBP-38]      --->文本框指针

00413016
PUSH ECX

00413017  PUSH EDI
00413018  MOV EAX,DWORD PTR DS:[EDI]
0041301A  CALL DWORD PTR DS:[EAX+A0]        --->读文本框数据

00413020  TEST EAX,EAX
00413022  FCLEX
00413024
JGE SHORT EX611.00413038

00413026  PUSH 0A0
0041302B  PUSH
EX611.00411A48

00413030  PUSH EDI
00413031  PUSH EAX
00413032  CALL DWORD PTR DS:[<&MSVBVM60.__vbaHresu>;
MSVBVM60.__vbaHresultCheckObj

00413038  MOV EDX,DWORD PTR SS:[EBP-38]      --->得到输入码

0041303B  LEA ECX,DWORD PTR SS:[EBP-34]
0041303E  MOV DWORD PTR SS:[EBP-38],0

00413045
CALL ESI

00413047  LEA ECX,DWORD PTR SS:[EBP-48]
0041304A
CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeO>;
MSVBVM60.__vbaFreeObj

00413050  MOV EDX,DWORD PTR SS:[EBP-30]
00413053
MOV EAX,DWORD PTR SS:[EBP-34]

00413056  PUSH EDX
                    --->输入码

00413057  PUSH EAX
          --->正确码

00413058  PUSH 0
0041305A
CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCo>;
MSVBVM60.__vbaStrComp              --->比较是否相等
```

在`0041301A`处跟入，里面还有个`call`，再跟入，直到：

```asm
***CED2  PUSH DWORD PTR SS:[EBP+10]
***CED5  PUSH DWORD PTR SS:[EBP+C]
--->输入码。在OllyDbg中转存跟随数值，如果输入为空，
可能显示ABEF0108等乱码。将其改为正确注册码即可。注意不要用宽字符，输完后以00结束(98中好像用0001结
束)。

***CED8  CALL MSVBVM60.733BD01B
```
以上地址前四位可能不同，但后四位应该是一样的。

再次运行`Windows超级设置723`，出现注册提示直接点击确定，在`***CED5`处中断。修改内存，收工。

本人水平不高，不能破解输入限制，只能改内存。麻烦哪位高手看看怎么破解，我等也好学习。
