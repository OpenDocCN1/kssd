

# \[翻译\]魔兽争霸 1.24 Maphack 教程

> 原文：[`www.pediy.com/kssd/pediy12/130240.html`](https://www.pediy.com/kssd/pediy12/130240.html)

1.  写一个魔兽争霸 3 1.24d 的 maphack \[mAsm\]  
一场正在进行的暴风雪，使得所有快餐店都关了，看来无法摆脱我的宿舍了，正是如此，我再没有藉口去逃避写这篇教程了，该开始了：  
在我谈论之前，我想有必要提出以下要点：这也是我首次 hacking 魔兽 3,同时也是首次写 maphack，因此所有出现在这篇文章的要点完全规定-我鼓励你去指出任何我可以制造的错误，因为我有时不像 1337 那样。带着这些不碍事的，让我们在这里开始。  
  有需要提出的是你的地址可能会不同，因为 Game.dll 是一个可装载的 dll,它伸请了 6F000000H 的空间，然而，它会意想不到地重新分配它自己，所以要记住。如果它真的做了（重新分配它自己），你有两个选择：重新开始游戏，或者简单地离开你的地址，把所有的在这个教程中的地址作为 Game.dll+偏移的组合。哈哈，从每个地址中减去 6F000000H，并把结果加入你的基址里。  
   这篇教程是基于魔兽争霸 3 版本 1.24.3.6384（即 1.24d）。如果你使用一个不同的版本，偏移很可能会改变，但是这个方法仍然可行。  
   去尝试阻止人们 hacking 他们的游戏，暴雪公司放入了一些反调试的代码，主要以调用 SetSecurityInfo()形式，这个函数修改 Game.dll 的 DACL（任意访问控制表），并且不允许从外部修改。然而，知道这个我们通过修改 SetSecurityInfo 的能数可以很容易地修正这个行为，而这个参数不过是一些代表安全描述的位的组合罢了。  
   现在我们怎么着手去找这个？好吧，我们知道 Game.dll 必须导出一个函数到主执行文件，所以最好开始在那个地方。  
   最好的方法去着手调试魔兽争霸 3 是将它运行于窗口模式。你可以通过在它的快捷方式上点击右键，选属性，并在目标这个方框后面加上“ -window”（没有引号，但有个空格），  
   打开魔兽争霸 3 使它在主屏幕里空闲，并用 OLLY 附加到它上面（文件->附加）。一旦它被附加了，查看 Game.dll（右键->查看->Game）并按 Ctrl+N 弹出一些导入导出函数的列表。马上你会注意到 Game.dll 仅导出两个函数，只有一个我们感肖趣：  
代码：  
Names in Game, item 0  
Address=6F009870  
Section=.text  
Type=Export  
Name=GameMain  
Names in Game, item 1  
Address=6F7E1CE8  
Section=.text  
Type=Export  
Name=<ModuleEntryPoint>  
  
  
在 GameMain 上面按 Enter，你应该到这里：  
代码：  
  
  
  
  
6F009870 > 81EC 08010000    SUB ESP,108  
6F009876   A1 6041AC6F      MOV EAX,DWORD PTR DS:\[6FAC4160\]  
6F00987B   33C4             XOR EAX,ESP  
6F00987D   898424 04010000  MOV DWORD PTR SS:\[ESP+104\],EAX  
6F009884   56               PUSH ESI  
6F009885   8BB424 10010000  MOV ESI,DWORD PTR SS:\[ESP+110\]  
6F00988C   E8 1F2F0000      CALL Game.6F00C7B0  
6F009891   E8 7ABC6B00      CALL <JMP.&KERNEL32.GetTickCount>  
  
这个函数调用一些函数，在 Game.dll 及在 Kernel.dll 里。但它看起来是最好的开始这些函数的地方，因为它有意义在于它们会保护这些模块，并在继续执行前使这些反调试弄到一边去。所以知道这些，进入到 6F00C7B0H 的调用：  
代码：  
6F00C7B0   81EC 20020000    SUB ESP,220  
6F00C7B6   A1 6041AC6F      MOV EAX,DWORD PTR DS:\[6FAC4160\]  
6F00C7BB   33C4             XOR EAX,ESP  
6F00C7BD   898424 1C020000  MOV DWORD PTR SS:\[ESP+21C\],EAX  
  
没有感兴趣的东西在这里，只是个开始的函数；然而，如果你向下滚动一些，你会这个宝贵的函数调用：  
6F00C852   B8 A484886F      MOV EAX,Game.6F8884A4                    ; ASCII "SetSecurityInfo"  
6F00C857   8BCE             MOV ECX,ESI  
6F00C859   E8 32FFFFFF      CALL Game.6F00C790  
  
这个有趣-想知道为什么它们把字符串传到 EAX 中，跟随到 6F00C790H 的调用，马上你可以见到为什么了：  
代码：  
6F00C790   50               PUSH EAX  
6F00C791   51               PUSH ECX  
6F00C792   FF15 BCF2876F    CALL DWORD PTR DS:\[<&KERNEL32.GetProcAdd>; kernel32.GetProcAddress  
6F00C798   85C0             TEST EAX,EAX  
6F00C79A   75 03            JNZ SHORT Game.6F00C79F  
6F00C79C   C2 0400          RETN 4  
6F00C79F   8B5424 04        MOV EDX,DWORD PTR SS:\[ESP+4\]  
6F00C7A3   8902             MOV DWORD PTR DS:\[EDX\],EAX  
6F00C7A5   B8 01000000      MOV EAX,1  
6F00C7AA   C2 0400          RETN 4  
  
看来这个函数没有做什么只是取会这个传给 EAX 的地址。虽然我们知道这个，我们但我们也可以假定 SetSecurityInfo 很可能会在我们现在的函数中调用，所以回到 6F00C7B0H,向下滚动直到你到了这里：  
  
6F00C8AB   53               PUSH EBX  
6F00C8AC   8D4424 30        LEA EAX,DWORD PTR SS:\[ESP+30\]  
6F00C8B0   50               PUSH EAX  
6F00C8B1   53               PUSH EBX  
6F00C8B2   53               PUSH EBX  
6F00C8B3   68 04000080      PUSH 80000004  
6F00C8B8   6A 06            PUSH 6  
6F00C8BA   55               PUSH EBP  
6F00C8BB   FF5424 34        CALL DWORD PTR SS:\[ESP+34\]  
6F00C8BF   85C0             TEST EAX,EAX  
6F00C8C1   75 03            JNZ SHORT Game.6F00C8C6  
  
好了马上烦挠我的是在 6F00C8B3H 行的内容-我们在处理 WIN32 API 时知道大部份位标志通常以一些 8XXXXXXX 的形式派生。另一样事情是跳过这个调用。因为 SetSecurityInfo 是现在这个函数最后的调用 API，它看起来应该属于栈帧的顶部。  
现在我们怎么处理这个？转换到 C,这个函数看起来像：  
SetSecurityInfo(ebp,6,0x80000004,ebx,ebx,eax,ebx);  
在 MSDN 查这个函数，你可以看到第三个参数代表安全信息，如果我们简单地也一个 NULL 位标志入栈，函数会什么也不做。所以改变 PUSH 80000004H 成为 PUSH 0，你很轻松到这里：  
6F00C8B3   6A 00            PUSH 0  
6F00C8B5   90               NOP  
6F00C8B6   90               NOP  
6F00C8B7   90               NOP  
  
所有都完成的时候，最后我们可以设置断点了，开始 hacking 这个游戏了。  
在我们开始做之前，我们应该给这一个想法，因为写一个 maphack 不是像搜索一个显示了的数值那么容易。所有我们真正可以继续下去的事实就是作弊码“iseedeadpeople”让我们查看全地图，并且自从我过去研究过星际争霸的引擎，我们可以假定设定地图可见的状态为 0,然而搜索 0 及未知值将会是一个错误的方向去处理很多地址，所以我们无论如何也要削减它们。  
   然而我们也知道另一个事情，地图数组在地图开始时必须被初始化，所以我们可以用 Olly 找出那些内存段是从那里改变的。  
   在魔兽争霸 3 里，转到单人游戏，开始自定义游戏，从那里，向动到列表尽头并选择 Ice Crown 作为你的地图（这个会在后面解释），现在，在你开始前，切换到 Olly，按 Alt+M 弹出内存映射，并刷新它（右键->刷新），这所做的是基本上保存一个现时内存状态的参考指针，并且一旦我们再选择刷新，会比较这些内存段，并标出哪些被改变的。  
   我们的参考指针设置了，切换到魔兽争霸 3,开始你的游戏，并等待它装载。当最后你的单位出现在屏幕前，切换到 Olly，并再次刷新内存并放大内存的方框，看看成堆的红和黄的地址。那些红色高亮的是我们感兴趣的。因为它们代表着改变的内存段。然而你会注意到它们混合着，所以现在开始分开它们两部分这个有趣的任务。  
   不要在你排序时，在任何环境下给予魔兽争霸 3 焦点。所有的内存会被重刷新，而你会失去所有。  
我建议你有时间才去做这件事，因为这会耗费一些时间并确保你做一些吸引你的事去使你的思想得到休息。例如：我在洗衣服。  
我计厌洗衣服。  
我建议的最简单的方法去做这件事就是按大小来排序这些地址（右键->排序->按大小），并复制整个表到记事本，并从底部（通常大小是 1000H）到大小是 9000H 的我们可以删除。为什么？记得我是怎样选“Ice Crown”作为我们的地图？好，一个在 GOOGLE 上快速搜索会告诉你地图的大小是 192\* 192.假定地图是一系列的一字节元素组合到一个数组（不是一次坏的假定），我们知道魔兽争霸 3 需要申请最小 36864 字节内存，在十六进制中就是 9000H。  
余下黄色的，你不得不手动把那些去除，但超过列表的 3/4 是在 9000H 大小以下的，这个不会用太多时间。  
一旦你拥有所有已排序的地址，是时候拿出便利的十六进制计算器，ArtMoney（或者任可内存搜索工具），并且有一些空余时间，开始搜索。当在搜索时，确保你的搜索在一个范围内，通过 Option->Searching，并选择“Search in address range”方框里。我们也想确保我们减小我们的搜索甚至通过只搜索一字节的值。要做这个，当你弹出搜索框时，确保你选择的椭圆框下一个类型，并选择“Integer 1 byte”。不同于那个，所有我们知道的是当作弊激活，值是 0,当作弊取消，它们会是未知的值。  
在一些搜索后你会遇到一个范围的值会在作弊激活时变换为 0 ，在作弊取消时是 1，128.  
典型地我这里弹出接近大小 9000 块内存，所以我会建议从底部开始。  
带着你找到的地址，选择一个，然后在 Olly 转到它那里（Ctrl+G，在框里输入地址），然后设置一个内存断点（右键->断点->内存写入），并返回游戏，它应该立刻弹出，但如果它没有弹出，切换地图作弊码（iseedeadpeople）去强迫它。如果你选择了个一正确的地址，它应该在停在这里。  
6F2AC1F7   66:894E 2E       MOV WORD PTR DS:\[ESI+2E\],CX  
  
这个有趣-如果你向上滚动到函数的顶部并查询它的参考（在 0F2AC1D0H 上 Ctrl+R），你会看到它被几个地方调用。给出这个，它很可能时一个共享的函数负责设定特定的几个值，所以我们需要去到上一个级别的函数（调用这个函数的函数）。按 Ctrl+F9 执行到返回，并按 F7 步入下一条指令，Olly 会引你到这里：  
代码：  
6F39A882  |. 52             PUSH EDX  
6F39A883  |. 50             PUSH EAX  
6F39A884  |. 8BCE           MOV ECX,ESI  
6F39A886  |. E8 4519F1FF    CALL Game.6F2AC1D0   ;function we were just in  
6F39A88B  |. 83BF 28030000 >CMP DWORD PTR DS:\[EDI+328\],0  
      
嗯，这次我们离目标更近一些了。我们在一个更大的函数里，但我们没有它的任何线索。是时候拿起棒子反任意抗了，用 NOP 填充它，并观察。向上滚到顶部（6F39A7E0H）并选择全部代三直至 6F39AB64H（留下 retn 4），并用 NOP 填充它（二制制->用 NOP 填充），回到你的游戏，你应该注意到没有树。好吧这更有趣了-看来共享函数被绘图函数所参考（即调用），并且我们也希望阴影函数也参考它，因为它们通常被连接到上面提及的函数。  
回到 6F2AC1D0H，并再次弹出参考的列表，这次全部完成它们。在用 NOP 操作符填充完这些在参考周围的代码后，你应该注意到两个没有做任何东西，然而有三个我们非常感兴趣：  
6F39A475 - function controls drawing of sprites  
6f39a886 - function controls drawing of trees  
6f39bccd - function controls shade   
  
不打算深入解释它，但是如果你看这个负责绘图的函数你会找到这个宝贵的函数  
代码：  
6F39A43F  |. 51             PUSH ECX                              ; /Arg4  
6F39A440  |. 8D5424 14      LEA EDX,DWORD PTR SS:\[ESP+14\]            ; |  
6F39A444  |. 894424 48      MOV DWORD PTR SS:\[ESP+48\],EAX            ; |  
6F39A448  |. 52             PUSH EDX                               ; |Arg3  
6F39A449  |. 8D4424 20      LEA EAX,DWORD PTR SS:\[ESP+20\]            ; |  
6F39A44D  |. 50             PUSH EAX                                ; |Arg2  
6F39A44E  |. 8D4C24 38      LEA ECX,DWORD PTR SS:\[ESP+38\]         ;   
|6F39A452  |. 51             PUSH ECX                                ; |Arg1  
6F39A453  |. 8B8F 78010000  MOV ECX,DWORD PTR DS:\[EDI+178\]           ; |  
6F39A459  |. 896C24 20      MOV DWORD PTR SS:\[ESP+20\],EBP            ; |  
6F39A45D  |. 896C24 28      MOV DWORD PTR SS:\[ESP+28\],EBP            ; |  
6F39A461  |. E8 9A7B0000    CALL Game.6F3A2000            ; \\Game.6F3A2000  
  
当用 NOP 填充，你会注意到单位有时从不被显示，但有时在迷雾中显示，在它上面按 Enter 找出  
6F3A2043     66:8B0C41      MOV CX,WORD PTR DS:\[ECX+EAX\*2\]  
  
可以注意到这个和 Chotic 的教程这间的相似点吗？  
这个仅为了 LOLS  
这三个全部都有趣，然而最后的那个看起来最值得怀疑，因为当它用 NOP 填充时它在我们可见的区域都覆盖了阴影。  
是时候看看这个什么时候被调用-向上滚动到 6F89BC40H 并通过按 Ctrl+R 弹出一个参考列表：  
代码：  
References in Game:.text to 6F39BC40  
Address    Disassembly                               Comment  
6F39BE22  MOV  EDX,Game.6F39BC40          6F39BC40=Game.6F39BC40  
6F39C1DF  MOV EDX,Game.6F39BC40           6F39BC40=Game.6F39BC40  
  
两个连续的 MOV 指令？这个相当奇怪。。。让我们看第一个尝试找出这里发生了什么。  
代码：  
6F39BE22  |. BA 40BC396F    MOV EDX,Game.6F39BC40  
6F39BE27  |. 8BC8           MOV ECX,EAX  
6F39BE29     E8 321E0D00    CALL Game.6F46DC60  
  
   好那个比较有趣，从我制造它的地方，魔兽争霸 3 传送到指向先前的我们统治的函数指针送到寄存器，并且使用另一个普通函数去处理它。但是为什么有两个参考？我首个想法是可能魔兽争霸 3 用一个来为单位绘制合适的阴影，如果我们向上滚动到现在我们在里面的这个函数的顶部（6F39BDB0H）,并用 NOP 填充这个从 6F39D05CH 的调用，我们可以看到我们所有的单位毁坏或者消失。如果你花费更多时间在这个函数你会看到它主要负责设定视半径，其它视野函数涉及单位。  
所以留下其它参考：转到 6F39C1DFH 并且你应该到一个小函数里：  
代码：  
6F39C1C0     837C24 04 00   CMP DWORD PTR SS:\[ESP+4\],0  
6F39C1C5     56             PUSH ESI  
6F39C1C6     8BF1           MOV ESI,ECX  
6F39C1C8     74 21          JE SHORT Game.6F39C1EB  
6F39C1CA     6A 00          PUSH 0                      ; /Arg1 = 00000000  
6F39C1CC     8D8E 00060000  LEA ECX,DWORD PTR DS:\[ESI+600\]           ; |  
6F39C1D2     E8 198DF9FF    CALL Game.6F334EF0        ; \\Game.6F334EF0  
6F39C1D7     6A 00          PUSH 0  
6F39C1D9     56             PUSH ESI  
6F39C1DA     E8 910AEDFF    CALL Game.6F26CC70  
6F39C1DF     BA 40BC396F    MOV EDX,Game.6F39BC40  
6F39C1E4     8BC8           MOV ECX,EAX  
6F39C1E6     E8 751A0D00    CALL Game.6F46DC60  
6F39C1EB     8D86 00060000  LEA EAX,DWORD PTR DS:\[ESI+600\]  
6F39C1F1     5E             POP ESI  
6F39C1F2  \\. C2 0400        RETN 4  
  
你应该注意到同一个普通函数被调用到寄存器指针，所以领导我相信我们太低了。如果你向上滚动到 6F39C1C0H，你会看到这个从四个地方被调用。  
代码：  
References in Game:.text to 6F39C1C0  
Address    Disassembly                               Comment  
6F3573A5   CALL Game.6F39C1C0  
6F362147   CALL Game.6F39C1C0  
6F3A34B2   CALL Game.6F39C1C0  
6F40B5AE   CALL Game.6F39C1C0  
  
如果你跟随前两个调用到不同的级别，你会注意到它们会到达同一个地方，并且看起来盘旋在单位阴影函数周围-所以让我们把焦点放在第三个上。跟随代码最终到达：  
代码：  
6F3A34AF     53             PUSH EBX  
6F3A34B0     8BCF           MOV ECX,EDI  
6F3A34B2     E8 098DFFFF    CALL Game.6F39C1C0  
  
如果你向上滚动到函数的顶部你可以看到它被看起来涉及地图的函数所参考，自从我们知道直到调用 6F39C1C0H 地图指针才初始化，我们可以安全地忽略在它之前的任何东西。在 6F3A34BDH 的调用起来像普通的函数调用，所以可被忽略。但是在向下滚动之前看到一个字符串使我每次见到它时都大笑。  
代码：  
6F58AE6C  |. B9 446C976F  MOV ECX,Game.6F976C44 ;ASCII "LOADING\_LOADING"  
  
Derp derp, LOADING\_LOADING! \*  
任何谁，去到下一个在 6F3A34C5H 的调用，第一件事突然出现在你前面的是 memset:  
代码：  
6F40B446   . 52             PUSH EDX                                 ; /n  
6F40B447     6A 00          PUSH 0  
6F40B449   . 50             PUSH EAX                                 ; |s  
6F40B44A   . E8 73643D00    CALL <JMP.&MSVCR80.memset>          ; \\memset  
  
好，好，设置了一大块内存为 0.，退一步说相当可疑。用 NOP 填充整个调用（包括在 6F40B44FH 的栈平衡）并回到游戏。什么都没有。。。除非我们进入一个区域并离开，哇！看起来我们拥有负责重设地图数组每帧的这个部份代码！  
是的，它很笨拙及浪费，看来地图数组每帧都重新设置，然后适当地反阴影。  
但是怎么处理这个？好吧，让我们尝试改变 PUSH 0 到一些更加用户友好的-像 PUSH FH，在魔兽争霸 3 里显示每样东西到每个玩家。回到游戏，你应该注意到你现在拥有一个史诗般的 maphack，开始编码！  
自从有个相同的想法产生，一些基本的东西我逐字地从我先前的教程复制及粘贴过来。  
我为我的懒惰道歉。  
  
所以这里我很可能失去你们中的很多人，因为我们打算用汇编写。我会给你一个机会去点击在浏览器上的后退按钮。。。  
那些仍在周围的，让我们开始吧。。。  
我个人会用 RadASM，虽然我强烈地建议，你拥有自由去用任何你想用的去写代码。我在用 MASM 汇编并连接，所以可能它里面的一些代码不能在其它 ASM 包中工作，但是它不应该太难编辑。  
首先，让我们创建 Shell（这是一个 DLL 不用猜测）  
代码：  
.386  
.model flat,stdcall  
option casemap:none  
  
.code  
    main:  
  
    end main  
  
现在，在代码段之前，但在选项之下，我们需要写一些包含文件：  
include \\masm32\\include\\kernel32.inc  
include \\masm32\\include\\user32.inc  
includelib \\masm32\\lib\\kernel32.lib  
includelib \\masm32\\lib\\user32.lib  
  
在我们继续之前，我们需要一些变量，所以让我们停下来想一想关于我们实际需要的。显然，我们需要声明一个字符串包含“Game.dll”所以我们可以得到我们的地址（因为 Game.dll 是动态分配，即使它申请 6FXXXXXX 作为基址），并且我们需要一些东西保留这个地址。所以在代码上，但在包含文件下面：  
代码：  
.data  
    address dd 0  
    dllname db "Game.dll"  
  
好了，现在去到实际的 DLL，在 Main，在@@后面：  
代码：  
push ebp  
        mov ebp,esp  
        mov eax,dword ptr ss:\[ebp+0ch\]  
        cmp eax,1  
        jnz @returnf  
        push eax  
    
        pop eax  
        @returnf:  
        leave  
        retn 0ch  
  
我确实给了你离开的机会。如果你打算用汇编写代码，你最好准备用汇编方式写，而不是这些半 HLA\*\*\*\*，MASM 现在支持  
好了，基本的代码解释，因为这是一个 DLL，一个我们主要关心的是保持所有东西完好及平衡。首先我们基址指针入栈，然后把栈顶指针传送到基址指针，这会基本告诉代码目前的程序的基址就是我们 DLL 的基址，这也是计算机先生需要知道的（在互联网上没有女孩，因此，电脑一直是伙计们的）以致可以找出返回以上调用的地方，跳转，计算，等等。PUSH 保存了基址指针在栈里面，所以当我们离开我们的代码并返回，基址又再设为魔兽 3 的基址指针了。在那之后我们转送数据到基址指针+CH(12D)，这个是持有调用我们的 DLL 的原因。  
ebp = base  
ebp + 4 = \_\_stdcall DllMain  
ebp + 8 = hModule  
ebp + 12 = ul\_reason\_for\_call  
ebp + 16 = lpReserved \*  
    
如果原因是 DLL\_PROCESS\_ATTACH(或者是 1H)，那么我们想执行我们的代码，否则我们想离开我们的 DLL，并且返回执行流向魔兽 3（确保栈平衡-3 个参数＝retn CH）  
  EAX 在我们的 DLL 返回时需要保持调用 DLL 的原因，所以我们在我们的函数调用开始前通过把它入栈来保存它，并在我们返回前把它从栈中弹出。  
  马上，我们需要取回我们的地址-在我们的相反的聚会基址总是属于 6F000000H，我们的代码在 6F40B447H，所以从这里我们可以推断我们的的地址放在基址+40B448H。从那里，所有我们需要做的是取回目前的基址，我们很好完成：  
    lea eax,dllname  
    push eax  
    call GetModuleHandle  
    add eax,40b448h  
mov address,eax  
  
LEA 代表载入有效地址，这是一个汇编的方式去把一个指向一段内存的指针送入寄存器。  
现在，在我们可以开始写代码到我们的地址前，我们需要首先去反保护它，并且在我们可以做之前，我们段要释放一些空间去保存旧的保护类型。  
代码：  
push 40h  
  push 1000h  
    push 4h  
  push 0  
  call VirtualAlloc     
  mov ebx,eax  
基础地，这在当前我们玩的进程里分配分配 4 字节的内存空间（MEM\_COMMIT,PAGE\_EXECUTE\_READWRITE）我们把这个空间的地址传送到 EBX，因为 EAX 会很快被下列的代码调用破坏：  
接下来，在我们可以写之前，我们需要在我们的地址的代码去掉保护：  
push ebx  
push 40h  
push 1h  
push address  
call VirtualProtect  
  
  现在我们的数据没有保护，我们可以最终写到它里面：  
mov eax,address  
mov byte ptr ds:\[eax\],0fh  
  
两个 MOV 指令看起来有点多，所以让我自己解释：因为地址保持这个编移，我们不能直接传送一个值给它-相反，我们需要把它传送它指向的值所以要实现这个，我们把它传送 EAX，然后当我们的指针使用。  
  
用我们的新地址现在写，是时候整理所有东西了：  
push ebx  
        push dword ptr ds:\[ebx\]  
        push 1h  
        push address  
        call VirtualProtect     
        push 4000h  
        push 4h  
        push ebx  
        call VirtualFree  
  
  
最终代码：  
.386                    ;attilathedud  
.model flat,stdcall  
option casemap:none  
  
include \\masm32\\include\\kernel32.inc  
include \\masm32\\include\\user32.inc  
includelib \\masm32\\lib\\kernel32.lib  
includelib \\masm32\\lib\\user32.lib  
  
.data  
    address dd 0  
    dllname db "Game.dll"  
  
.code  
    main:  
        push ebp  
            mov ebp,esp  
            mov eax,dword ptr ss:\[ebp+0ch\]  
            cmp eax,1  
            jnz @returnf  
            push eax  
            lea eax,dllname  
        push eax  
        call GetModuleHandle  
        add eax,40b448h  
        mov address,eax  
        push 40h  
            push 1000h  
            push 4h  
            push 0  
            call VirtualAlloc     
        mov ebx,eax  
        push ebx  
            push 40h  
            push 1h  
            push address  
            call VirtualProtect    
            mov eax,address  
        mov byte ptr ds:\[eax\],0fh    
        push ebx  
            push dword ptr ds:\[ebx\]  
            push 1h  
            push address  
            call VirtualProtect     
            push 4000h  
            push 4h  
            push ebx  
            call VirtualFree        
            pop eax  
            @returnf:  
            leave  
            retn 0ch  
  
end main  
  
  
这是它所有的东西。现在，如果这个在 Battle.net 上被检测，还是没有，我不能说，因为我没有一份合法的魔兽 3 副本，但是一些可以告诉我的事情是：任何人，希望你喜欢这个教程，直到下次。  
  
  
原文在 3 楼。 

原网站是：http://www.gamethreat.net/forums/open-source-tutorials/42277-writing-maphack-warcraft-3-1-24d-masm.html，刚看到今天就把它翻译了，累。本来想翻译来自己用来看的，但是想想分享给大家也是一种好的生活态度。个人英语水平不高，还望大家多多指正。