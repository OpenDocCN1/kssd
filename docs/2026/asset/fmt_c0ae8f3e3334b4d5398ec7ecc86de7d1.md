
`LParam = 0x00C20017`是一个32位的值,其实这个值的高16位`0x00C2`就是十进制的194,那低16位`0x0017`对应的十进制就是23了;现在只要再知道棋子的高跟宽就能算出棋盘数组中每一个棋子的坐标了,求棋子的宽度与高度,我们用最直接的方法,就是截取一张整个棋盘的图片(大小跟棋盘偏差不要太大),然后用一个图片编辑工具打开(我用的是Windows自带的图片编辑器),就能看到该图片的宽度与高度,然后宽/11,高/19,用四舍五入取整数的商,就是一个棋子的宽与高了,看下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_t26mggkp8dmgw0r.png)

上图中,棋盘的宽为594,高为388.

那么594/19≈31     388/11≈35

参数`LParam`就等于`(y<<16) + x`.

那么,如果棋子p1的坐标是`x = 2`,`y = 3`的话,`LParam`就表示为`(35\*3<<16) + 31\*2`;还要注意的是这个坐标是相对于棋盘第一格的坐标的,要表示相对窗口的坐标的话,还要加上第一格的坐标,`LParam = ((35\*p1.y + 194)<<16)+(31\*p1.x + 23)`这才是表示棋子相对窗口的坐标,有了这些参数,我们就可以编写代码实现消去一对棋子了,

```
BOOL click2p(POINT A,POINT B)

{

  int x = 23;

  int y = 193;  //第一格棋子坐标

  int LPARAM = ((y+35\*A.y)<<16)+x+31\*A.x;

  // TODO: Add your control notification handler code here

  HWND hwnd = FindWindow(NULL,GEAM_CAPTION);

  if (hwnd == NULL)

  {

    return FALSE;

  }

  SendMessage(hwnd,WM_LBUTTONDOWN,0,LPARAM);

  SendMessage(hwnd,WM_LBUTTONUP,0,LPARAM);

  LPARAM = ((y+35\*B.y)<<16)+x+31\*B.x;

  SendMessage(hwnd,WM_LBUTTONDOWN,0,LPARAM);

  SendMessage(hwnd,WM_LBUTTONUP,0,LPARAM);

  return TRUE;

}
```

### 6. 现在双击”单消”按钮,去到实现单消的成员函数,在这函数里,只要调用`ClearPiar()`函数即可,下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_nmb8quwotw2kmck.png)

当然还要把头文件夹`GeamProcess.h`包含进去才能调用,下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_63besgen89oz3gu.png)

现在单消的功能已经能够实现了

## 六、 修补与完善外挂

### 1. 既然已经实现了单消，那么实现全消也不是什么问题了，只要循环调用单消就行了，当然这个循环肯定要有个退出的条件，可以用剩余的棋子数作为退出的循环的条件，如图：

![](https://bbs.kanxue.com/upload/attach/201301/480923_sm36zqamaxmumqc.png)

在棋盘下方的剩余方块的这个地方，就是表示棋盘上的棋子数，只要找出这个地址，然后读出这个地址的数据，我们还是用`CE`工具来查找，这个很容易找到，因为它是个可见的值，一直用精确值来查找就行了.找到之后就是编写代码读出数据,这跟之前读出棋盘数据差不多.只要把那段代码稍微改一下就好.

```
#define ChessNumAddre (LPVOID)0x001159FC  //棋子数基址

LPVOID chessnumber;

void UpChessNum()

{

  DWORD ProcID;

  HANDLE ProcHwnd;

  LPCVOID BackSize;

  HWND hwnd = FindWindow(NULL, GEAM_CAPTION);

  if (hwnd == NULL)

  {

    return;

  }

  GetWindowThreadProcessId(hwnd,&ProcID);

  ProcHwnd = OpenProcess(PROCESS_ALL_ACCESS,FALSE,ProcID);

  ReadProcessMemory(ProcHwnd,ChessNumAddre,&chessnumber,1,(LPDWORD)&BackSize);

  CloseHandle(ProcHwnd);
```

上面的`UpChessNum()`函数跟之前的`UpdateChess()`函数几乎是一样的,只改动了几个地方(读取进程的地址、用来存放进程内容的变量、还有读取的大小).

我们把这个函数插入到实现消去棋子的`click2p(POINT A,POINT B)`函数里面去,(注意: `UpChessNum()`函数要定义在`click2p(POINT A,POINT B)`函数的前面)

![](https://bbs.kanxue.com/upload/attach/201301/480923_sm36zqamaxmumqc.png)

这样只要每消去一对棋子,棋子数就会更新.现在可以编写”全消”按钮的成员函数代码了(双击”全消”按钮就可以去到该函数的定义).

```
void CQQllkWgDlg::OnExinction()

{

  // TODO: Add your control notification handler code here

  UpChessNum();//更新棋子数

  while(chessnumber)

  {

    ClearPiar();

  }

}
```

函数中,一开始先更新棋子数,然后再进入`while`循环.

### 2.
![](https://bbs.kanxue.com/upload/attach/201301/480923_k9c02bg76enpikx.png)

如上图,在游戏的过程中,还会碰到无解的情况,如果点击了”全消”按钮,在这种情况下,`while`循环就是一个死循环,程序会崩溃掉.解决这个问题,我们可以在游戏窗口中,点击重列的道具,我们也可以自动的模拟鼠标点击重列.首先用`Spy++`获取到重列图标相对窗口的坐标,(获取的方法,跟前面获取棋盘数组第一格的坐标相同,这里就不再叙述了)然而这个重列道具是有限的,还要用`CE`工具找到该地址(这个重列道具数也是可见的,它的地址也很容易找得到),然后读出重列道具数.

```
#define RESTATED_NUM (LPVOID)0x001179C6   //重列地址

LPVOID restatednumber;

VOID ReadRestated()

{

  DWORD ProcID;

  DWORD BackSize;

  HWND hwnd = FindWindow(NULL,GEAM_CAPTION);

  if (hwnd != NULL)

  {

    GetWindowThreadProcessId(hwnd,&ProcID);

    HANDLE ProcH = OpenProcess(PROCESS_ALL_ACCESS,FALSE,ProcID);

ReadProcessMemory(ProcH,RESTATED_NUM,&restatednumber,1,&BackSize);

    CloseHandle(ProcH);

  }

}
```

执行`ReadRestated()`函数就会在`restatednumber`返回重列道具数,

再编写代码模拟鼠标点击重列图标.

```
VOID ReChess()  //重列棋盘

{

  LPARAM LPARAM = RESTATED_COORDINATE;

  HWND hwnd = FindWindow(NULL,GEAM_CAPTION);

  if (hwnd != NULL)

  {

    SendMessage(hwnd,WM_LBUTTONDOWN,0,LPARAM);

    SendMessage(hwnd,WM_LBUTTONUP,0,LPARAM);

  }

}
```

现在代码编写好了,要怎么样才能自动调用呢? `ClearPiar()`函数是实现消去一个棋子,它还是一个布尔类型的函数,当它的返回值是`true`时,表示当返回值是`false`时就表示没有消去棋子,返回`false`的情况只有两种,一是整个棋盘的棋子都消去了,没有棋子了,二是无解.那么我们就在`ClearPiar()`函数返回`false`的语句的前面,插入`ReChess()`函数,当然,不能直接插入,还要加一些判断.

```
ReadRestated();  //读出重列道具数

if (chessnumber && restatednumber)//如果棋子数与重列道具都不为零

{

  ReChess();

}
```

在`ClearPiara()`函数中插入以上代码,就可以实现自动重列了,如下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_e3ouy199stigvv6.png)

### 3.  现在再加上挂机速度跟自动挂机的功能,先把滑块的属性设置好,右键滑块--->Properties(属性)---->Styles(样式)--->钩上`Tick marks`(刻度标记)跟`Enable selection`(启用选择)还有`Auto ticks`(自动刻度)那个`Point`(点)的选项,随个人爱好选择,如下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_qr57hfnsr793cpy.png)

然后再给滑块关联一个控件,使用控件调用类成员函数设置滑块的最小值跟最大值,以及初始值,还有刻度值.右键滑块--->ClassWizard(类向导)--->Member Variables(成员变量)--->双击滑块ID (`IDC_SLIDER1`),给滑块添加一个控件类型的变量.如下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_lca0lm7c38by81b.png)

在`Category`(类别)中选择`Control`(控件),写入变量名.点OK,如下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_mxkmt8poe8uvy7n.png)

在以同样的方法添加一个数值类型的变量,如下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_0k3brt7vapk41jr.png)

现在去初始化对话框的类中,添加设置控件信息的代码,在对话框空白处右键--->Events(事件)--->双击右边的初始化对话框消息,就能去到初始化对话框的成员函数,好下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_goslsvbdm6jl0fe.png)

在函数的尾部,返回值之前,添加上代码.

```
m_Speed_Slider.SetRange(30,2000);//设置滑块范围

m_Speed_Slider.SetPos(800); //设置初始位置

m_Speed_Slider.SetTicFreq(100); //设置移动频率
```

以上三个函数,看字面意思就能大概明白意思.添加位置如下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_wbesoceg8w0vf3s.png)

滑块的基本信息设置好了,再给它添加一个编辑框,用来显示出当前滑块的数值(我给该编辑框关联了一个`int`型的变量`m_SheepNum`).然后再给滑块关联一个`NM_RELEASEDCAPTURER` 的消息响应(为什么是`NM_RELEASEDCAPTURER`消息,我也不是很清楚,找了一些资料,只朦胧的知道它能捕获鼠标抬起的消息,)右键滑块--->Events(事件)--->双击`NM_RELEASEDCAPTURER`--->OK.如下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_1jex5flcg68v7e6.png)

再双击右边已经关联的`NM_RELEASEDCAPTURER`事件去到该成员函数的实现.如下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_omz62npzgmx27eh.png)

```
void CQQllkWgDlg::OnReleasedcaptureSlider1(NMHDR* pNMHDR, LRESULT* pResult)

{

  // TODO: Add your control notification handler code here

  UpdateData(TRUE);//更新控件的值到变量

  m_SheepNum = m_SliderVar;

//m_SheepNum = m_Speed_Slider.GetPos();//此语句跟上面语句等价

  UpdateData(FALSE);//更新变量到编辑框

  *pResult = 0;

}
```

以上代码:当在滑块触发了鼠标抬起消息,用`UpdateData(TRUE)`函数更新滑块的数值到变量`m_SliderVar`中,然后再赋值给`m_SheepNum`,最后再用`UpdateData(FALSE)`函数把`m_SheepNum`的值更新到编辑框.

这样我们就能在编辑框上看到滑块的数值了(当移滑块的时候,是鼠标抬起了,才更新数值).下面再来实现把挂机速度跟滑块关联想来,当钩上”挂机速度”的时候,才能够移动滑块,如果不钩上”挂机速度”就不能移动滑块.

### 4.  给”自动挂机”复选框关联一个`BOOL`的变量`m_GeamSpeed`,然后双击复选框,去到成员函数的实现.

```
void CQQllkWgDlg::OnCheck2()

{

  // TODO: Add your control notification handler code here

  UpdateData(TRUE);

  ::EnableWindow(m_Speed_Slider.m_hWnd,m_GeamSpeed); //该函数允许/禁止指定的窗口或控件接受鼠标和键盘的输入.

}
```

这现在再编译的话,就能用`m_GeamSpeed`变量的值控制滑块是否接收消息,还有一点没完整,就是一开始的时候, `m_GeamSpeed`默认是没钩上的,此时滑块还能接收鼠标的消息,这是因为这时没有触发调用`void CQQllkWgDlg::OnCheck2()`函数的消息,所以没起到理想的效果.解决问题很简单,只要把`m_GeamSpeed`变量初始化的值改为`true`就行了.右键点击`m_GeamSpeed`变量--->Go To Reference to m_GeamSpeed(转到参考),可以看到`m_GeamSpeed`变量初始化为`false`,把它改成`true`好了.如下图:

### 5.  实现自动挂机.同样的先给”自动挂机”复选框关联一个`BOOL`类型的变量(`m_AutoGame`),去到成员函数的实现.

```
void CALLBACK PlayProc(

            HWND hwnd,

            UINT uMsg,

            UINT idEvent,

            DWORD dwTime

            )//定义一个回调函数,

{

  ClearPiar();

}

CONST AUTOTIME = 111;//自动挂机定时器(全局变量)

void CQQllkWgDlg::OnCheck1() //自动挂机成员函数

{

  // TODO: Add your control notification handler code here

  UpdateData(TRUE);  //更新数据到变量

  if (m_AutoGame)

  {

    SetTimer(AUTOTIME,m_SliderVar,&PlayProc);//创建一个定时器

  }

  else

  {

    KillTimer(AUTOTIME);

  }

}
```

以上代码先定义一个回调函数(实现单消功能),然后在`void CQQllkWgDlg::OnCheck1()`成员函数中建一个定时器,每隔一定的时间(`m_SliderVar`)就调用回调函数,从而实现自动挂机的功能.随着滑块的变动, `m_SliderVar`变量也跟着变动,这样挂机的速度也改变了,其实结果并不像我想的一样,测试的时候发现,当使用自动挂机时,滑动滑块,挂机速度并没有跟着改变,而是要去掉自动挂机重新钩上的时候速度才跟着改变,还要把代码改进一下.在滑块的成员函数中,我们添加一些代码.如下图:

当滑动滑块时,先关掉挂机定时器,然后再判断自动挂机变量,如果钩上了才打开.这样可以说比较完美的实现了自动挂机功能.

### 6.  现在再添加上去除游戏倒计时时间,用`CE`工具找出游戏倒计时地址(开始用大于0扫描,然后它是一直在减小的,所以一直用减少了的值扫描,直到时间跑完了,就用精确值为0扫描),然后再建立一个定时器,不断地把新时间写入到该地址(之前在我看的资料中,是用`CE`工具找到写入该地址的代码,然后把该代码去掉,这样我测试的时候,的确能实现,但是新版的qq游戏中,只要把这代码改了,就马上弹出警告,然后退出游戏,我一直也没找到解决这个问题的方法,如果有高人知道,希望能指点一二).这样游戏时间就不会减少了,再添加一个”去掉倒计时”的复选框,同时关联一个`BOOL`类型的变量`m_ClearTime`.

```
HANDLE ProcH;
```
