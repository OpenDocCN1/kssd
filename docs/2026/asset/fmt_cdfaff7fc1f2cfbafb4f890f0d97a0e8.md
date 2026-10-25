
# [原创]新人入手第一个游戏外挂,附上详细制作过程

此教程为原创,但本人是个菜鸟,刚入手写程序,因此本帖有部分内容是参照其他一些教程,以及本论坛里有关连连看的帖子,再经过一些修改,再附上制作过程,以及本人的一些经验.

经过这次的练习,才知道很多东西要通过自己动手去做,才能领悟```其中有不对之处,希望有高人能够指点一二,希望此贴能够对像我一样的纯菜鸟有帮助.(发帖子,不懂技巧,附件都被吞了,可以下载二楼的[复件 QQ连连看游戏外挂详解教程.rar]文件.![](https://bbs.kanxue.com/view/img/face/16.gif)
本教程分为几大部分
一、 先创建外挂对话框,定义外挂所需要的基本功能
二、 找出棋盘数组的地址,读出棋盘数据
三、 编写实现消去棋子的代码框架
四、 实现消去棋子功能
五、 修补与完善外挂

## 一、 先创建外挂对话框,定义外挂所需要的基本功能

1. 创建对话框: 创建一个MFC工程对话框
打开VC++软件 选择¬¬------>(File)文件------>New(新建)
如图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_9q5zj3gqmy3iw74.png)

选择--->Project(工程文件)---->MFC AppWizard[exe],填写好工程名字,选择好文件路径,点击OK
2. 选择---->Dialog based(基本对话框)------->Finish(完成)------>OK,如下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_fdk86zr4ywgq5t8.png)

3. 先删除了原来的按钮(选中按delete),再添加我们所按钮.(点一下想要添加的按钮或控件,再到对话框内点击一下,即可添加)如图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_r2u8mxi853gj54j.png)

4. 再把按钮的属性先改好(ID与标题)
在所需要改的按钮上右键---->Properties(属性),

![](https://bbs.kanxue.com/upload/attach/201301/480923_ghsgjgbc1cqgkvj.png)

图上添加了两个按钮,两个复选框,跟一个滑块.
一些基本功能都差不多了,后面如果还需要,再按以上方法添加.

## 二、 找出棋盘数组的地址,读出棋盘数据

1. 找出棋盘数组地址，我们需要用到一个Cheat Engine（CE）软件，(这些工具大多都是英文的，对于外语我可是一点也不懂，所以，工具的单词我都去网上翻译一下，如果有不对的地方，请理解，明白意思就行了)。

![](https://bbs.kanxue.com/upload/attach/201301/480923_f55ap0u8qa8mi16.png)

2. 这CE工具主要是访问进程地址空间，然后通过枚举的方法找出想要找的地址。
在枚举地址之前，还要用到另外一个工具XueTr.exe，因为QQ游戏大厅如果检测到CE工具在运行的话，就会弹出一个警告的对话框，然后退出游戏。

![](https://bbs.kanxue.com/upload/attach/201301/480923_qr0mpknanmw15e0.png)

用XueTr.exe工具能解决弹出警告,步骤:
打开QQ游戏大厅(要先关掉CE,免得又弹出警告),再打开XueTr.exe--->点击 进程 ---->右击QQ游戏大厅进程----->选择 查看进程模块;

![](https://bbs.kanxue.com/upload/attach/201301/480923_o9nd5b0dkj5n59b.png)

找到 Tersafe.dll,右键点击 Tersafe.dll ----->删除模块文件.
然后再返回进程右键点击 QQ游戏大厅----->选择 查看进程线程---->点击一下 模块 让他排序好---->右键点击 TenSLX.dll----->选择 结束线程(不管有几个TenSLX.dll,都结束掉).如下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_98fjurnpmcibmkf.png)

好了,现在可以关掉XueTr.exe了,打开CE工具就不会再弹出警告了,当然QQ游戏大厅每次重新启动都需要用XueTr.exe来结束那几个线程,才能正常使用CE.(那个Tersafe.dll模块已经删掉了,就没有了,也就不用再次删除了)
3.

![](https://bbs.kanxue.com/upload/attach/201301/480923_arh53ysl2qwo5x7.png)

如上图,我们知道棋盘是一个11\*19的二维数组的结构,我们只要找到数组的基地址就能算出整个数组每个棋子的位置,就是说只要枚举出上图做记号的那个位置的地址就行了.每一个棋子都是用一个数值来表示的,不同的棋子,用不同的数表示,空的地方当然就是0了,假如说1代表一种棋子,2又代表另一种棋子,依此类推,那么整个棋盘的最大范围都不会超过255(一个字节范围)所以我们枚举棋盘地址的时候选择数据类型为字节就足够了,如图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_vsr9trkzwosh7kg.png)

4. 现在进入QQ连连看游戏房间,找位置坐下,点击练习,再打开CE工具---->点击 选择进程打开----->选择 连连看游戏进程---->Open(打开)
如图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_ta2zw7znlo3uvzx.png)

![](https://bbs.kanxue.com/upload/attach/201301/480923_jyladx88l016hnr.png)

如上图,我们要找的地址没有棋子,就是为0,那么用CE工具就选择Exact Value(精确值)再点New Scan(新扫描)…如下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_rc7qpki78g70jlw.png)

我们看到上图左边Found(扫描结果)处:有一个很大的数值,再下来的Address(偏移地址)与Value(值)的地方是表示符合条件的结果;(数值都为0)
然后我们再点游戏的 练习,换一盘棋,

![](https://bbs.kanxue.com/upload/attach/201301/480923_guckgx4flmb2s9s.png)

请看左上角的地方还是没有棋子,就再选择Exact Value(精确值)再点Next Scan(再次扫描)…如下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_ksdp8uaxuz4axie.png)

我们会发现扫描结果已经在减少;
再次点击 练习,更换棋子.

![](https://bbs.kanxue.com/upload/attach/201301/480923_70x42hboshu67of.png)

上图左上角已经有一个棋子了,那么CE就要选择Bigger than(大于)------->Next Scan(再次扫描)

![](https://bbs.kanxue.com/upload/attach/201301/480923_s3pqo9790skvcxw.png)

现在发现扫描结果明显少了很多,然后再点击练习,换棋子,如果左上角没有棋子,就选择Exact Value(精确值)为0, ------->Next Scan(再次扫描); 如果有棋子就选择Bigger than(大于)------->Next Scan(再次扫描).直到扫描结果剩下一个的时候,就是棋盘数组的基地址了(在扫描到后面剩下几个扫描结果的时候,可能遇到再怎么换棋子来扫描,结果都不会减少的情况,这时可以退出游戏,然后换个座位或者换个房间,CE工具再重新选择进程打开,接着扫描)

![](https://bbs.kanxue.com/upload/attach/201301/480923_5v1xen9dikz58kl.png)

现在找到了棋盘地址,我们验证一下这地址正不正确,双击该地址,该地址就会在如上图出现在CE工具的下面,右键该地址----->Browse this memory region(浏览此内存区域)

![](https://bbs.kanxue.com/upload/attach/201301/480923_k8827b5k1721m24.png)

此时，就能看到棋盘地址`0x0012A1F8`处的内容,再去跟棋盘的棋子对应,就会发现,都是可以吻合的,(当然,如果不吻合,肯定是地址不正确了,)如图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_nsllqgk7kgnuto7.png)

看上图，第一排是空的，第二排的第一格也为空，加起来就一共是20个空的，再看下图，`0x0012A1F8`处，前面一开始也是有20字节为0的，紧接着两个字节为0C跟08，再看棋盘也刚好有两个棋子.(当你在棋盘上消去一对棋子的时候,会看到地址表中对应的两个字节,也会跟着被设为零)
现在基本上可以肯定这棋盘地址是正确的了:`0x0012A1F8` //棋盘基地址

![](https://bbs.kanxue.com/upload/attach/201301/480923_mpw7zmgxbcft43d.png)

5. 在读出棋盘数据之前,我们还需要知道窗口的标题,可以用VC++的Spy++,打开VC++选择Tools(工具)------>Spy++,然后点击“小望远镜”的图标.如下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_er4bcia27w4wrgx.png)

在弹出来的对话框中选定Properties(属性),打开连连看游戏,用鼠标按住Finder Tool(查找工具)的图标,把它拖动到连连看的游戏窗口中

![](https://bbs.kanxue.com/upload/attach/201301/480923_2q45ipp6qp575g5.png)
![](https://bbs.kanxue.com/upload/attach/201301/480923_j8jmp69cty51jb6.png)

然后点击OK,

![](https://bbs.kanxue.com/upload/attach/201301/480923_s7z637kkzdkkh24.png)

如上图中,在General(常规)中的Window Caption(窗口标题中)就可以直接把窗口标题复制下来.

首先我们回到VC++,在对话框中多添加一个按钮与编辑框,用来读出棋盘数据.

![](https://bbs.kanxue.com/upload/attach/201301/480923_o02d8cgann70sco.png)

用来读出棋盘数据的编辑框,大小要调整合适,然后再修改编辑框的属性
选定该编辑框----->右键------> Properties(属性)--->Styles(样式),然后把Multiline(多行)与Want return(这里的意思应该是换行)前面钩上,如下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_dk791jxrp8ybrh1.png)

(这个属性更改好了,直接关掉属性对话框,或者按回车,就会回到编辑外挂对话框了)
然后再给编辑框关联一个类变量,右键编辑框----->Class Wizard(类向导)---->Member Variables(成员变量)------>选定编辑框ID(我没有更改该编辑框的ID,所以它是IDC_EDIT1)----->Add Variables(添加变量).如下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_j1143ao1x1j8ask.png)

在弹出的对话框中设置好Member variable name(变量类型)跟Variable type(变量名,因为棋盘数据是一个字符串数组,所以我们要选择字符串类型)---->OK.
如下图:
![](https://bbs.kanxue.com/upload/attach/201301/480923_l2wdv8mjnazxffm.png)
返回外挂编辑图,双击”读取棋盘数据”按钮就可以转到实现触发该按钮消息的成员函数.我们在这里编写代码实现读出棋盘数据.如下图:
![](https://bbs.kanxue.com/upload/attach/201301/480923_m28q1nbgmq8jfx4.png)

先为棋盘地址与窗口标题定义一个宏(定义在函数外面,要全局的,)

```c
#define CHESS_DATA_ADDRESS 0x0012A1F8
#define GAME_CAPTION "QQ游戏 - 连连看角色版"
byte chessdata[11][19];  //存放棋盘数据的数组,这个数组我们定义为全局变量,因为别的函数还要用到此数组.
void CQQllkWgDlg::OnButton1()
{
  // TODO: Add your control notification handler code here

  DWORD prcesssid;      //返回窗口进程ID
  LPVOID chessaddress = (LPVOID)CHESS_DATA_ADDRESS;
  DWORD byread;

        HWND hwnd = ::FindWindow(NULL,GEAM_CAPTION);  //获取窗口句                       柄
        if (hwnd == NULL)//如果打开窗口不成功
        {
            GetWindowThreadProcessId(hwnd,&prcesssid); //获取窗口进程ID
            HANDLE processsH = OpenProcess(PROCESS_ALL_ACCESS,FALSE, prcesssid) ;//打开指定进程
            ReadProcessMemory(processsH,chessaddress,chessdata,19*11,&byread); //读取进程指定地址、大小的数据(读出数据到chessaddress数组)
            CloseHandle(processsH);//关闭进程
         }
}
```

//(编译代码,看是否有错误,我们往后每完成一小部分代码都应该编译一下,看是否有错,有错误就要先改过来,再继续写下去)
现在已经读出棋盘数据,那么我们再把chessaddress数组的数据显示到之前定义变量为`m_ChessData`的编辑框,这样我们能更清楚地验证棋盘数据.

```c
 CString chessdata1;
 CString tempStr;
 for(int x = 0;x<11;x++)
 {
   for (int y = 0;y<19;y++)
   {
     tempStr.Format("%2x",chessdata[x][y]);//把数据进行排版
     chessdata1 += tempStr;
     }
   chessdata1 += "\r\n";
 }
 m_ChessData = chessdata1;
 UpdateData(FALSE);//更新数据到编辑框
```

把上面代码加入到`void CQQllkWgDlg::OnButton1()`函数读取棋盘数据的代码后面,编译通过后,运行程序,就能显示棋盘数据到编辑框.如下图:
![](https://bbs.kanxue.com/upload/attach/201301/480923_8vucgle3znqsrsb.png)
![](https://bbs.kanxue.com/upload/attach/201301/480923_gjvzehuzj178662.png)
现在已经能显示棋盘数据了,不过看起来,不是很好看,再修改一下:
把`tempStr.Format("%2x",chessdata[x][y]);` 里面的`%2x`改成`%2.0x`;在
`chessdata1 += tempStr;`后面再上`chessdata1 += " ";`这样看起来会更加好看.
![](https://bbs.kanxue.com/upload/attach/201301/480923_0nspd4ucumkibau.png)
![](https://bbs.kanxue.com/upload/attach/201301/480923_s8ml99luwa44js6.png)
这样看起来更加贴切

既然读出了棋盘数据,那么接下来就是准备实现消去一对棋子代码框架.

在写代码前,我们先建立一个头文件,然后我们就在头文件中编写实现的代码,这样会比较整洁,清晰

点击菜单File(文件)--->New(新建)--->Files--->C/C++ Header File,写上头文件名,如下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_jw3kbxztjky5mbw.png)

在VC++应用于软件的左边,FileView版块的Header Files文件夹中我们就能看到刚才新建的头文件了,后面我们的一些函数的实现,都写在这头文件中,如下图:

![](https://bbs.kanxue.com/upload/attach/201301/480923_85m0xzgy9psvpgn.png)

## 三、 编写实现消去棋子的代码框架
