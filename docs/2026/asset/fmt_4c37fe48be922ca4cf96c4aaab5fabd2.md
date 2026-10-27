
语句`If Mid(Text1.Text, i, 1) > "9" Or Mid(Text1.Text, i, 1) < "0"`中的两个调用了`MID`的地方（p-code代码的相对位置）：`00401BFE` 和 `00401C23`（VBExplorer 中看到的）

当程序中断在您设置的断点后，你可以逐步跟踪程序了，为了把握住程序的流向，您要时刻关注`ESI`值的变化，它通常在`40xxxx`范围内，它明确地指向VB P-CODE伪代码的流向，你不用担心不知道自己所在的位置。你可以随时切换到VBExplorer中，看看程序到底跑到哪里去了。

在您调试的过程中，您会发`CALL AX`，应该单步跟进去，它通常会带你返回那个正被调试的VB程序主域（或叫“领空”，学台湾人的说法），一般附近是一些`JMP`长跳转，这些是程序中需要调用函数，应该给予重视，可以尝试在相关函数上下断点。

当您看到`call bx`时，通常程序要调用真正的函数执行功能，你应该跟进去，一般在执行P-CODE比较伪指令的过程中，你会发现一个`call bx`，一定要跟进去，通常会它引导你看到真正的注册码以及你输入的假注册码进行比较。通过观察堆栈的显示，你就可以看到他们，两个可爱的字符串（通常出现在OLLYDBG的调试窗口的右下方）。

跟踪VB程序时，通常会看到如下的代码：

```
7353F7B7   3BFC             CMP EDI,ESP
7353F7B9   0F85 D67D0000    JNZ MSVBVM60.73547595
7353F7BF   33C0             XOR EAX,EAX
7353F7C1   8A06             MOV AL,BYTE PTR DS:[ESI]
7353F7C3   46               INC ESI
7353F7C4   FF2485 58FA5373  JMP DWORD PTR DS:[EAX\*4+7353FA58]
7353F7CB   0FB70E           MOVZX ECX,WORD PTR DS:[ESI]
7353F7CE   0FB77E 02        MOVZX EDI,WORD PTR DS:[ESI+2]
7353F7D2   83C6 04          ADD ESI,4
7353F7D5   03FC             ADD EDI,ESP
7353F7D7   8B55 AC          MOV EDX,DWORD PTR SS:[EBP-54]
7353F7DA   8B048A           MOV EAX,DWORD PTR DS:[EDX+ECX\*4]
7353F7DD   0BC0             OR EAX,EAX
7353F7DF   90               NOP
7353F7E0   0F84 027F0000    JE MSVBVM60.735476E8
7353F7E6   803D B4115573 00 CMP BYTE PTR DS:[735511B4],0
7353F7ED   0F85 037F0000    JNZ MSVBVM60.735476F6
7353F7F3   FFD0             CALL EAX
7353F7F5   3BFC             CMP EDI,ESP
7353F7F7   0F85 987D0000    JNZ MSVBVM60.73547595
7353F7FD   50               PUSH EAX
7353F7FE   33C0             XOR EAX,EAX
7353F800   8A06             MOV AL,BYTE PTR DS:[ESI]
7353F802   46               INC ESI
7353F803   FF2485 58FA5373  JMP DWORD PTR DS:[EAX\*4+7353FA58]
7353F80A   0FB706           MOVZX EAX,WORD PTR DS:[ESI]
```

其中：

```
7353F7BF   33C0             XOR EAX,EAX
7353F7C1   8A06             MOV AL,BYTE PTR DS:[ESI]
7353F7C3   46               INC ESI
7353F7C4   FF2485 58FA5373  JMP DWORD PTR DS:[EAX\*4+7353FA58]
```

既是伪代码的执行模板。`JMP`指令转入伪指令的执行，不断按下“F8”，盯着`esi`寄存器的变化，参照vbexplorer中提示的p-code，你就不会迷失方向（通常是这样。当然，如果你按错了键，那就是你自己的事了，别怪我没提醒你）

## 大结局

关于最后的修改VB P-CODE代码，暴力破解法，据我所知一般只要能将重要的转移指令改变就可以了。无非是在您选好的比较指令的位置将`BranchF`（伪代码`1c`）、`BranchT`（`1d`） 或者`Branch`（`1e`）改来改去。别的好招我还不会呢，等着咱们继续切磋。

有空的朋友，还可以分析程序中的算法，只要花功夫，没什么看不懂的。除非编程序的那位憋着劲和咱们作对，不怕，咱不用他的软件了还不行？我是菜鸟，我怕谁？

够了，说了一堆有用没用的废话，您自己看着办8，您也听累了，我手也酸了。总之，vb p-code毕竟是由高级语言转换来的，应该不难分析和跟踪，只是特点不同，掌握了基本的要点，用OLLDBG跟踪没什么难的。现在又有WKTVBDE 和vbexplorer助阵，一边捏着P-CODE，一边瞄着`ESI`，祭起OLLYDBG，一手按着F8，您说，咱还怕谁呀？

什么逆向工程，什么VB,  V JQK，谁是破解高手？不就是花时间嘛。

唉，所以高手都不来发言了，所以，您和我，还是菜鸟。苦恼中。。。。

要是那位高手作个OLLDBG的VB P-CODE插件，一遇到VB p-code中`JMP DWORD PTR DS:[EAX\*4+&\*%$@$^]` 就显示相关的P-CODE指令就更方便了，尤其期待中。。。。
