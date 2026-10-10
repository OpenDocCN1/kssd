

# 给记事本添加"总在前面"功能【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-100.htm`](https://www.pediy.com/kssd/pediy08/pediy8-100.htm)

标题:给记事本添加"总在前面"功能  
使用工具:OD, LoadPE,WindowsXP 英文版里面的记事本程序  
其实在本论坛里面,已经有很多高手做过这个工作了 呵呵 我今天也来做一个 方法还是大同小异的 下面我们开始工作  
给 Windows 应用程序添加功能主要是通过修改消息处理来实现.所以我们首先要找到的就是 windows 消息处理过程 然后在这里修改使其能够处理我们添加的消息,然后实现我们自己的功能  
要让窗口总在最上面需要使用一个函数 函数名字是:SetWindowPos  
函数原型:  
BOOL SetWindowPos(          HWND hWnd,  
    HWND hWndInsertAfter,  
    int X,  
    int Y,  
    int cx,  
    int cy,  
    UINT uFlags  
);  
首先用资源修改工具给记事本添加一个菜单项 ID 可以随便 我用的是 1C 十进制就是 28  
知道了函数原型和添加了菜单项目后我们可以开始工作了.  
用 OD 打开记事本 下断 bp RegisterClassExW  F9 运行 然后断了下来  
观察当前堆栈  
0007FDE4   01004556  /CALL 到 RegisterClassExW 来自 NOTEPAD.01004550  
0007FDE8   0007FDF0  \\pWndClassEx = 0007FDF0  
0007FDEC   77D49B69  USER32.LoadCursorW  
0007FDF0   00000030  
0007FDF4   00000000  
0007FDF8   01003429  NOTEPAD.01003429  
0007FDFC   00000000  
0007FE00   00000000  
0007FE04   01000000  NOTEPAD.01000000  
0007FE08   00980761  
0007FE0C   00010013  
0007FE10   00000006  
0007FE14   00000001  
0007FE18   01009020  UNICODE   
  
 呵呵 01003429 就是我们要的地址,为什么是这个地址呢 用 MSDN 看一个 RegisterClassExW 的原型就有答案了  
Enter 来到 01003429 可以看到这里就是消息处理函数的入口,OD 已经在注释里面给我们指出了每一个 case 了,我们来找个合适的位置处理我们的消息吧  ,拉着滚动条向下可以看到一个一个的 case 和消息处理  
哈哈 我找了这个位置  
01002EA5   . /E9 BD040000   JMP NOTEPAD.01003367  
01002EAA   > |83FF 1A       CMP EDI,1A  
01002EAD   . |0F8F 9D580000 JG NOTEPAD.01008750  
01002EB3   . |0F84 55010000 JE NOTEPAD.0100300E  
01002EB9   . |83EF 16       SUB EDI,16  
01002EBC   . |0F84 F4000000 JE NOTEPAD.01002FB6  
  
这里有个和 1A 判断的 而我们的 ID 是 1C 所以很接近读了一下代码发现可以开始了 我们的 1C 是大于 1A 的 所以只要修改  
01002EAD   . /0F8F 9D580000 JG NOTEPAD.01008750  
  
这个跳转 让它跳到我们的代码就好了 OK  注意:这里消息放在 EDI 里面  
我们就在记事本的空白地方找个位置写我们的代码吧我是从 1008750 开始的   
下面是我的代码  
01008747      00            DB 00  
01008748      00            DB 00  
01008749      00            DB 00  
0100874A   .  00000000      DD 00000000     ;这个位置是用来记录当前的状态的  
0100874E      00            DB 00  
0100874F      00            DB 00  
01008750   >  83FF 1C       CMP EDI,1C     ;比较是不是我们要的消息  
01008753   .  74 05         JE SHORT NOTEPAD.0100875A  ;如果是就用我们的代码来处理消息  
01008755   .^ E9 BFA8FFFF   JMP NOTEPAD.01003019       ;不是就回到原来的跳转位置  
0100875A   >  52            PUSH EDX        ;要用它来做临时寄存器用所以先入栈      
0100875B   .  8B15 4A870001 MOV EDX,DWORD PTR DS:\[100874A\]   ;把当前的状态放进去  
01008761   .  6A 13         PUSH 13                          ;参数压栈  
01008763   .  6A 00         PUSH 0  
01008765   .  6A 00         PUSH 0  
01008767   .  6A 00         PUSH 0  
01008769   .  6A 00         PUSH 0  
0100876B   .  83FA 00       CMP EDX,0                        ;比较当前的状态如果是在最上面状态就取消不是就使他在最上面  
0100876E   .  75 0A         JNZ SHORT NOTEPAD.0100877A       ;0 不是最上面  1 最上面  
01008770   .  6A FF         PUSH -1  
01008772   .  6A 01         PUSH 1  
01008774   .  8F05 4A870001 POP DWORD PTR DS:\[100874A\]       ;修改状态标记  
0100877A   >  74 0A         JE SHORT NOTEPAD.01008786  
0100877C   .  6A FE         PUSH -2  
0100877E   .  6A 00         PUSH 0  
01008780   .  8F05 4A870001 POP DWORD PTR DS:\[100874A\]       ;修改状态标记  
01008786   >  FF35 30980001 PUSH DWORD PTR DS:\[1009830\]              ; 记事本的 hWnd  
0100878C   .  FF15 1A400101 CALL DWORD PTR DS:\<&USER32.SetWindowPos>; call    
01008792   .  5A            POP EDX                          ;和前面入栈对照 自己压的自己出  
01008793   .^ E9 6DA4FFFF   JMP NOTEPAD.01002C05             ;执行完毕后返回  
  
然后在 OD 的反汇编窗口点右键-->复制到可执行文件-->右键保存文件(全部)-->选择你要保存的文件名-->保存  
工作结束  
注意事项:  
其实 SetWindowsPos 这个函数记事本本身是没有的 所以有两种方法来使用它   
1:用 LoadPE 等工具给它添加这个函数 该函数在 USER32.DLL 里面  
2:使用下面的方法  
  push USER32.DLL  
  call GetModuleHandleA  
  push SetWindowsPos  
  push eax  
  call GetProcAddress  
  call eax  ;eax 就是我们要的函数 SetWindowPos 了 (不要像我这么写无法运行的 你要自己写)  
感谢本坛的 ohuangkeo 大虾我有参考他的方法,还有一个大虾记不起名字了哈 谢谢了  
后:^\_^我的比他们两个的代码更少一点."总在最上"会遮挡下面内容的 怎么办那就再添加一个透明功能吧  
要让一个窗口透,首先要更改窗口样式,要用的函数是 GetWindowLong() SetWindowLong() SetLayeredWindowAttributes()  
思路和上面一样判断消息处理之下面的是我随便写的代码 我还没有实际测试  
使记事本透明  
cmp edi,1D  
je \*\*\*\*\*\*\*\*   
jnz \*\*\*\*\*\*\*\*\*\*  
push -14  
push hWnd  
call GetWindowLong  
OR eax,80000  
push eax  
push -14  
push hWnd  
call SetWindowLong  
push 2  
push 96  
push 0  
push hWnd  
call SetLayeredWindowAttributes  
  
取消透明  
push -14  
push hWnd  
call GetWindowLong  
and eax,fff7ffff  
push eax  
push -14  
push hWnd  
call SetWindowLong  
有时间弄下试试![

 4dd466d0e1e0cc123afbe504da2462ca