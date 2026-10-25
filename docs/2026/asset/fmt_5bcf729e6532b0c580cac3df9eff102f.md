
# 拦截Windows消息

![](https://bbs.kanxue.com/view/img/face/50.gif)

拦截Windows消息

拦截应用程序的菜单项时SoftIce提供了如下方式:

`:bmsg hMenu wm_command`

`:g`

`:bc*`

`:bpx k23thk1632prolog`

`:g`

`:bc*`

`:g ret`

一般都会来到如下程序处:

`XXXX:CALL [KERNEL32!K32Thk1632Prolog]`

`XXXX:CALL [...]`<-----菜单入口点

`XXXX:CALL [KERNEL32!K32Thk1632Epilog]`

对MASM、VC编写的程序跟踪都可找到其菜单处理的入口点，在98下跟踪路径为:

`0167:5f401BD1-->0167:5F401BFF-->0167:5F401C6D CALL [EAX+40]`

在SoftIce中用`":d eax+40"`就可看到我们关心的菜单处理程序的入口地址。

但是问题也就由此而来:

1) 在2K、XP中SoftIce的`":HWND 应用程序句柄"`不好使了!
2) 在IDA、OllyDbg中使用断点设置时我们要事先知道并通过Resscope、UltraEdit在可执行程序中寻找相关信息才可找到程序入口点。过程烦琐!!!有时根本找不到我们要的东西，程序也没加壳!有点丢人~!~
3) IDA、OllyDbg能否像98下SoftIce那样进行动态跟踪呢，何况他们提供了大量的可参考信息。在静态基础上有动态调试可更好的理解所分析的程序的思路。
4) 在没有经验时用IDA、OllyDbg跟踪消息处理会陷入"无限消息循环"之中而无法到我们关心的位置处!!
5) 在我们找寻拦截断点时是否有规律可循呢?

现在我们一步一步来看如何解决上述5难题，给后来者提供一个可参考的路标，让他们顺利进入解密行列或逆向分析行列中来!至于为什么~!~这就不说了

进入正题!

例:从上述5点疑问我们假设一命题叙述如下

假设:在一菜单中接收鼠标按键响应之后显示一模态模板,此模板上有按钮及其它一些可供选择的操作。

实现方式:在一主进程中只接收鼠标按键响应，将模板显示、可供选择的其它操作的具体代码放到以动态链接库中。如下图所示:

|        菜单按钮响应     |               |     {...~!~CreakMe} |

`XXXX.EXE`                                  `YYYY.DLL`

1. 选择一个菜单响应、具体实现代码都在主进程中的菜单项。如文件的打开或保存文件之类的，利用`AFX_MSGMAP_ENTRY`结构数组在可执行程序中找到此消息相对应的执行函数的指针。

具体做法为:

菜单响应时消息类型为`WM_COMMAND`，16进制为`0111H`。其值的获得参考《MADN Library Visual Studio 6.0》中`WM_COMMAND`的QuickInfo所指的`Winuser.h`。可看到`WM_COMMAND`的常量定义或使用SoftIce的`"WMSG WM_COMMAND"`查看其定义的值(这管保好使~!~)。

2. 利用资源探测器`Resscope`查到这个菜单项ID，将其转换为16进制(可利用`Resscope`将所有菜单项ID都列出并制作成Excel表格,为什么? ~!~好好想想)

3. 利用`AFXWIN.H`中的`AFX_MSGMAP_ENTRY`结构数组制作菜单、按钮的`WM_COMMAND`消息响应函数地址入口表。为了方便查询:)

叙述如下:

```c
struct AFX_MSGMAP_ENTRY
{
  UINT nMessage;   // 存储类型为"DWORD型";含义: 消息类型
  UINT nCode;      // 存储类型为"DWORD型";含义: 控制代码或WM_NOTIFY代码
  UINT nID;        // 存储类型为"DWORD型";含义: 控制ID
  UINT nLastID;    // 存储类型为"DWORD型";含义: 消息入口所用控制ID值域
                   // 中的某个值，即感兴趣的菜单项ID值
  UINT nSig;       // 存储类型为"DWORD型";含义: 消息动作标识
  AFX_PMSG pfn;    // 存储类型为"DWORD型";含义: 消息响应函数的入口地址
                   // (实际是一个和该消息对应的响应函数的指针)
};
```

设:资源为"打开工程"菜单项ID为`32211`(`7DD3H`),响应`WM_COMMAND`消息，消息响应入口在`004508A0`处。则在`.rdata`分段中存储格式为:

`XXXXH:  11010000     00000000     D37D0000    D37D0000  0C000000    A0084500`

`--------     --------     --------    --------  --------    --------`

`nMessage       nCode        nID        nLastID    nSig     AFX_PMSG pfn`

`0111H WM_COMMAND     00H        7DD3H        7DD3H     0CH       004508A0H`

`|             |`

`---------------`

`#define ID_PROJECT_OPEN 32211(7DD3H)`

利用2中的Excel表格制作消息响应函数地址入口表。例如:

`ResourceConst        Sid       NumID    .rDataAddr     nMessage   nCode    nID     nSig   AFX_PMSG pfn`

`3221,"打开工程&O..  ID_PROJE   7DD3H     XXXXXXXXH     0111H       00H    7DD3H    7DD3H   004508A0H`

`(Ctrl+O)"           CT_OPEN`

`.`

`.`

`.`

4. 在`OllyDbg`或`IDA`中载入要分析的程序并在已知响应函数入口地址点设置断点，运行载入程序。在载入程序中进行相应操作后来到前面设置的断点上，查看`[ESP]`中的值或"运行直到返回"都可看到调用此响应函数后的上一级函数地址。其调用语句为`XXXX: JMP SHORT ZZZZ`,在往上三个字节处的一条指令为`XXXX-3:  CALL DWORD ptr ss:[ebp+14]`;或者为`CALL [ebp+14]`。则`XXXX-3`为我们要设置的跟入`YYYY.DLL`的断点。

5. 重新装入要分析的程序，若此程序需打开相应工程文件才能提供执行3中有空缺项的菜单项(实现函数在`YYYY.DLL`)时，可重复执行`G:XXXX-3`。具体如下:

1o 用`OllyDbg`、`IDA`重新载入要分析的程序
2o `G:XXXX-3`
3o 在要分析的程序中打开工程文件
4o 在`OllyDbg`、`IDA`中用`G:XXXX-3`
5o 完成3中的打开工程文件动作
6o 在`OllyDbg`、`IDA`中用`G:XXXX-3`
7o 在要分析的程序中点选3.列表中有空缺项的菜单项。来到我们要跟踪进入`YYYY.DLL`的断点处

6. 在`OllyDbg`、`IDA`中用`"D ss:[ebp+4]"`查看时若显示为:`100AA260`,则调用`YYYY.DLL`中的函数入口地址及相应DLL名完全查出填入3.列表中,以供进一步分析使用

小结:总体思想为以已知某一路线后,以它为基准点向未知点进行拓展。程序是固定的一个，不管是什么语言编写其消息响应路线相对于这个程序是不变的。所谓水涨船高既是此道理~!~怎么有点像AutoCAD画图使用基准点后，以基准点为中心狂画参考线呀!!!呵呵，又想到一点有时我们也要机动灵活别闹出刻舟求剑的笑话就行!嘿嘿

郁闷ing~!~呵呵,有些时候一些知识点是相通的，可能有交叉点的存在。只是我们没有看到罢了~~~

参考文件: `WINUSER.H`

`AFX_WIN.H`

`WINNT.H`(中有PE文件格式声明)

附:SoftIce拦截过程

在98下对菜单用Mouse Hook查看其句柄，若为`0578`则在SoftIce中用`:hwnd 0578`以再次确认其正确性
`:hwnd 0578`
```
Window Handle    hQueue  SZ   QOwner       Class Name         Window Procedure
0578(1)         0F37   32    XXXX     \_Microsoft Word f      136F:00000878
```
`:bmsg 0578 wm_command`
`:g`              ;点选已知菜单项，发生中断
```
Break due to BMSG 0578 WM_COMMAND (ET=8.18 seconds)
hWnd=0578 wParam=7DB7 lParam=00000000 msg=0111 WM_COMMAND
****** 
```
`:bc*`
`:g 0167:4508A0`  ;到已知消息入口点
`:g ret`          ;或按F12键到上一级调用函数的返回点
;此处为      `0167:5F4023D5 CALL [EBP+14]`
;            `0167:5F4023D8 JMP XXXXXXXX` <--调用返回点
;则断点可使用`0167:[5F4023D8-3]`即:  `0167:5F4023D5`
`:g`
`:cls`
`:hwnd 0578`
```
Window Handle    hQueue  SZ   QOwner       Class Name         Window Procedure
0578(1)         0F37   32    XXXX     \_Microsoft Word f      136F:00000878
```
`:bmsg 0578 wm_command`
`:g`              ;点选未知菜单项，发生中断
```
Break due to BMSG 0578 WM_COMMAND (ET=8.18 seconds)
hWnd=0578 wParam=7D79 lParam=00000000 msg=0111 WM_COMMAND
****** 
```
`:bc*`
`:g 0167:5F4023D5`
```
Break due to G (ET=260.97 microseconds)
```
`:d ebp+14`
```
016F:01B0F814   60 A2 0A 10 ...;可看到入口地址为0167:100AA260
`** ** ** ** 
```
:按F8进入则可看到以7D79为ID的`WM_COMMAND`消息响应函数入口地址为:
`0167:100AA260`,调用的`.DLL`文件名为`QQQQ!.text+000A925F`中`QQQQ.DLL`,
此函数在`QQQQ`的相对位置为`000A925F`H处。

```
****** ***************************** ************************************************************************************************ *
```

连贯操作时为:
`:hwnd 0578`
`:bmsg 0578 wm_command`
`:g`     ;点选已知菜单项
`:bc*`
`:g 0167:4508A0`
`:g ret` ;F12
`0167:5F4023D5  CALL [EBP+14]`
`0167:5F4023D8  JMP XXXXXXXX`;调用后返回点，则断点为  `0167:5F4023D5`。记录之~!~
`:g`
`:cls`
`:hwnd 0578`
`:bmsg 0578 wm_command`
`:g`     ;点选未知菜单项
`:bc*`
`:g 0167:574023D5`
`:d ebp+14`
```
0167:01B0F814  60 A2 0A 10 ...;入口地址 0167:100AA260
** ** ** **
```
:F8调用`.DLL`文件名为`QQQQ!.text+000A925F`中`QQQQ.DLL`,此函数在`QQQQ`中的相对位置为
`!.text+000A925F`H处。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
