
# BitComet 软件中附带广告的去除

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-509.htm`](https://www.pediy.com/kssd/pediy08/pediy8-509.htm)

- **【文章标题】**: BitComet 软件中附带广告的去除
- **【文章作者】**: 火翼[CCG]
- **【作者邮箱】**: `wyyy@263.net`
- **【作者主页】**: 无
- **【作者 QQ 号】**: 网上能查到
- **【软件名称】**: `Bitcomet`
- **【软件大小】**: 2.5M
- **【下载地址】**: 自己搜索下载
- **【加壳方式】**: 无
- **【保护方式】**: N 多广告
- **【编写语言】**: `MFC`
- **【使用工具】**: `OllyDbg`
- **【操作平台】**: `XP`
- **【软件介绍】**: 使用很广泛的 BT 下载软件
- **【作者声明】**: 不知道为什么,很讨厌共享软件里面的广告

---

### 详细过程

`BitComet` 是目前使用最广泛的 Bt 下载软件，但它最近的几个版本不仅功能没有什么进步，还加入了大量的网页式广告。由于目前最新的 `0.72` 版并不稳定，在很多机器上出现了 CPU 占用 100% 等其他一些问题，所以本文针对的是 `BitComet` 的最近的一个稳定版本--`0.70`。

`BitComet` 的广告嵌入在软件的很多地方，所以对调试者的耐心是一个挑战。

首先，查找广告的位置和类型，`BitComet` 软件的广告包括主窗体的广告和制作 Torrent 文件窗口，任务属性页面，视频预览页面的广告。是用 `Visual C++` 附带的 `Spy++` 工具就能发现这些页面中的广告除了主窗体中间的广告是图片外，其他的都是嵌入软件内部的 IE 内核的浏览器对象。

然后正式开始调试过程，还是老规矩，使用 `Peid 0.94` 检测程序是否加壳，发现程序没有加壳。直接使用 `IDA 5.0` 载入，开始反汇编，`IDA` 自动加载了 `VC` 和 `MFC` 的函数库进行分析，结束后可以看到大部分 `MFC` 封装的类方法已经被自动识别了出来。下面就针对不同页面里的广告进行动态调试，使用的软件是 `OllyDbg 1.10`。

#### 主页面图片广告

首先是和其他地方都不一样的主页面中间的图片广告，此处图片在软件刚开始时显示的是 `Paypal` 的捐款连接，之后才会变为广告图片，那么就从 `Paypal` 的图片入手，使用 `eXeScope v6.5` 查看可执行文件的资源后，发现 `Paypal` 的图片在资源中的名字为 `PAYPAL.GIF`，在 `IDA` 中按 `ALT+T` 查找 `PAYPAL`，可以找到如下地址

```
.text:00410F48 push    offset aPaypal_gif ; "PAYPAL.GIF"
.text:00410F4D mov     [edi], eax
.text:00410F4F call    sub_401670
```

这部分代码属于函数 `sub_410E70`,向上查找调用这个函数的地址，发现只有一处调用

```
.text:0042AD4E jz      short loc_42AD6B
.text:0042AD50 push    eax
.text:0042AD51 call    sub_410E70       ; 创建捐款对象
```

试着把上面的调用上面的 `jz short loc_42AD6B`（`74 1B`）改为 `jmp loc_42AD6B`（`EB 1B`）后运行，发现此处广告已经不会出现，第一处广告处理结束。

#### 左侧工具条搜索页面

接下来就要处理左侧工具条里面的的搜索页面，使用 `Spy++` 软件可以查出主工具条下方的频道/搜索/IE 小工具条使用了 `ReBar` 和 `ToolBar` 两种控件，熟悉 `MFC` 的程序员肯定马上就能想到他的实现方式，不熟悉的也没关系，只要去搜索引擎上查找 `Rebar` 或者 `ToolBar` 就能查到你想要的全部东西。查完资料后，我们知道了 `CToolBar` 的按钮要使用 `CToolBar::SetButtonText` 对按钮显示的文字进行设置，所以我们在 `IDA` 的 `Name` 窗口中找到 `CToolBar::SetButtonText`，并在记下函数开始的地方，然后运行 `OllyDbg`，在刚才记下的地址下断点，断下后，观察堆栈中的参数指向的字符串，找到设置我们要寻找的按钮的文字的地方，这里有两点需要注意，一个是本程序使用 `Unicode` 编码，所以内存中的字符串格式基本都是 `Unicode`，另一个是由几个地方都把按钮设置成了我们要查找的按钮上的文字，不过通过修改堆栈中的字符串参数指针，并观察程序的变化，我们最后可以找到真正的设置地点是：

```
.text:0042EC4D push    821Bh//按钮的 Command ID
.text:0042EC52 mov     ecx, esi
.text:0042EC54 call    CToolBar::CommandToIndex(uint)

//由 Command ID 返回按钮的编号
.text:0042EC59 push    eax
.text:0042EC5A mov     ecx, esi
.text:0042EC5C call    CToolBar::SetButtonText(int,wchar_t const *)

//设置按钮文字
.text:0042EC61 push    offset aIe       ; "IE"
.text:0042EC66 push    821Ah
.text:0042EC6B mov     ecx, esi
.text:0042EC6D call    CToolBar::CommandToIndex(uint)
.text:0042EC72 push    eax
.text:0042EC73 mov     ecx, esi
.text:0042EC75 call    CToolBar::SetButtonText(int,wchar_t const *)
.text:0042EC7A cmp     dword_6C1B2C, 8
.text:0042EC81 mov     eax, dword_6C1B18
.text:0042EC86 jnb     short loc_42EC8D
.text:0042EC88 mov     eax, offset dword_6C1B18
.text:0042EC8D loc_42EC8D:  ; CODE XREF: SetToolBarButtonText+56 j
.text:0042EC8D push    eax
.text:0042EC8E push    8229h
.text:0042EC93 mov     ecx, esi
.text:0042EC95 call    CToolBar::CommandToIndex(uint)
.text:0042EC9A push    eax
.text:0042EC9B mov     ecx, esi
.text:0042EC9D call    CToolBar::SetButtonText(int,wchar_t const *)
```

这个函数根据选择的语言对 3 个按钮上显示的文字进行设置，在 `IDA` 中自定义此函数的名字为 `SetToolBarButtonText`,然后在 `OllyDbg` 中继续执行，可以找到调用这个函数的地方 `42E6D1`，此处代码如下：

```
.text:0042E68E push    3
.text:0042E690 push    0
.text:0042E692 mov     ecx, edi
.text:0042E694 call    CToolBar::SetButtons(uint const *,int)

//创建 3 个空按钮
.text:0042E699 push    4
.text:0042E69B push    10h
.text:0042E69D push    821Bh//Command ID
.text:0042E6A2 push    0
.text:0042E6A4 mov     ecx, edi
.text:0042E6A6 call    CToolBar::SetButtonInfo(int,uint,uint,int)

//设置第 1 个按钮的属性
.text:0042E6AB push    11h
.text:0042E6AD push    10h
.text:0042E6AF push    8229h//Command ID
.text:0042E6B4 push    1
.text:0042E6B6 mov     ecx, edi
.text:0042E6B8 call    CToolBar::SetButtonInfo(int,uint,uint,int)

//设置第 2 个按钮的属性，全部改为 NOP
.text:0042E6BD push    0Fh
.text:0042E6BF push    10h
.text:0042E6C1 push    821Ah//Command ID
.text:0042E6C6 push    2     ;此处改为 1
.text:0042E6C8 mov     ecx, edi
.text:0042E6CA call    CToolBar::SetButtonInfo(int,uint,uint,int)

//设置第 3 个按钮的属性
.text:0042E6CF mov     ecx, esi
.text:0042E6D1 call    SetToolBarButtonText

//设置按钮文字
```

由于我们要去掉第二个按钮搜索，所以我们把创建按钮的参数改为 2，并把对应第 2 个按钮的设置部分改为 `NOP`,由于修改后只有 2 个按钮所以原来的设置第 3 个按钮属性的 `SetButtonInfo` 函数的指示按钮编号的参数要改为 1，修改后测试完全正常，继续进行下一步——主页面内容简介按钮和对应页面的去除。

#### 内容简介按钮去除

内容简介按钮所属的控件为 `TreeView`，在 `IDA` 中找到 `CTreeCtrl::InsertItem` 函数的位置，并在 `OllyDby` 中下断，根据堆栈中第 2 个参数（项目名称）找到调用的地方为 `sub_431D90` 函数的 `4320DC`，继续向上跟踪找到调用 `sub_431D90` 的地方为 `sub_433D50` 函数内的 `433E21`，此处代码为：

```
.text:00433E10 mov     eax, [esp+28h]
.text:00433E14 cmp     eax, edi
.text:00433E16 jz      short loc_433E63 ; 

//判断是否已经全部完成，此处跳过将完全不在此 TrewView 中创建任何按钮
.text:00433E18 mov     ecx, [esp+20h]
.text:00433E1C push    ebp
.text:00433E1D push    ebx
.text:00433E1E push    eax
.text:00433E1F mov     edx, esi
.text:00433E21 call    sub_431D90       ; //创建 Item
.text:00433E26 mov     eax, [esp+28h]
.text:00433E2A cmp     eax, edi
.text:00433E2C jz      short loc_433E3A
.text:00433E2E mov     ecx, [eax]
.text:00433E30 mov     edx, [ecx+8]
.text:00433E33 push    eax
.text:00433E34 call    edx
.text:00433E36 mov     [esp+28h], edi
.text:00433E3A loc_433E3A: CODE XREF: sub_433D50+DC j
.text:00433E3A mov     eax, [esp+0Ch]
.text:00433E3E mov     ecx, [eax]
.text:00433E40 lea     edx, [esp+28h]
.text:00433E44 push    edx
.text:00433E45 push    eax
.text:00433E46 mov     eax, [ecx+24h]
.text:00433E49 call    eax
.text:00433E4B test    eax, eax
.text:00433E4D jge     short loc_433E10//继续创建下一个 Item
```

这里不太好改，继续看看下面的代码，从 `sub_433D50` 返回 `sub_46DE20` 函数的 `46DF1F` 处，向下继续执行到

```
.text:0046E091 push    esi            ; lParam
.text:0046E092 push    9              ; wParam
.text:0046E094 push    TVM_SELECTITEM ; Msg
.text:0046E099 push    eax            ; hWnd
.text:0046E09A call    ds:SendMessageW
```

此处是设置 `TreeView` 的当前选择的 `Item`,我们可以利用这里去掉 `TreView` 里面的内容简介，只需要把 `TVM_SELECTITEM`（`0x1110B`）改为 `TVM_DELETEITEM`（`0x1101`）,并把 `wParam` 设为 1 就可以删除 `TreeView` 中的第一项，但这样只是去掉了 `TreeView` 中的内容简介，程序执行时还是会默认显示内容简介页。由于点击其它项的时候主页面搜索页会被隐藏，所以在 `OllyDby` 中的 `Cmd` 工具条中输入 `bpx ShowWindow`，对主程序中所有调用 `ShowWindow` 函数的地方设置断点，设置后点击 `TreeView` 中的任务列表后断在 `56A29D` 处，此处属于 `MFC` 标准类函数 `CWnd::ShowWindow`，继续跟踪，返回 `4C28B6` 处

```
.text:004C289C mov     [esp+14h+arg_4], eax
.text:004C28A0 jz      short loc_4C28B6
.text:004C28A2 push     0             ; dwNewLong
.text:004C28A4 mov     ecx, eax
.text:004C28A6 call     CWnd::SetDlgCtrlID(int)
.text:004C28AB mov    ecx, [esp+14h+arg_4]
.text:004C28AF push    0             ; nCmdShow
.text:004C28B1 call     CWnd::ShowWindow(int)//隐藏
.text:004C28B6 loc_4C28B6; CODE XREF: SwitchWindowVisiable+B0 j
.text:004C28B6 push    ebp
.text:004C28B7 push    ebx
.text:004C28B8 mov     ecx, esi
.text:004C28BA call     CSplitterWnd::IdFromRowCol(int,int)
.text:004C28BF mov     ebx, [esp+14h+var_4]
.text:004C28C3 push    eax           ; dwNewLong
.text:004C28C4 mov     ecx, ebx
.text:004C28C6 call      CWnd::SetDlgCtrlID(int)
.text:004C28CB push    5             ; nCmdShow
.text:004C28CD mov     ecx, ebx
.text:004C28CF call     CWnd::ShowWindow(int) ; 显示
```

这个函数是根据传入的行和列设定当前显示的页面，在函数开始的地方设置断点，然后在 `OllyDbg` 中重新加载程序，可以发现当调用参数为（3，1）的时候为显示搜索页面，代码如下：

```
.text:0044551E push    3;改为 push 1
.text:00445520 push    1;改为 push 1
.text:00445522 Call    CSplitterWnd::SetActivePane
```

修改传入的参数为（1,1）即可让程序执行时默认访问任务摘要页面。



然后就是 3 个子窗口上的广告了，这个 3 个窗口上广告的调用方式都差不多，我们先来看制作 Torrent 文件窗口上的广告，使用`OllyDbg`加载程序运行后，点开制作 Torrent 文件窗口，然后在在`IDA`中查出的`CDialog::OnInitDialog(void)`地址处设断点，点击内容简介，程序中断，函数返回地址为`sub_43EC50`函数的`43EC86`，这个函数很长，包括很多控件的初始化过程，继续跟踪，直到这个函数快要结束的地方,

```asm
.text:0043F0BE call    sub_443B60//创建对象

.text:0043F0C3 push    0

.text:0043F0C5 push    0

.text:0043F0C7 push    300000h

.text:0043F0CC mov     ecx, esi

.text:0043F0CE call     CWnd::ModifyStyle(ulong,ulong,uint)

.text:0043F0D3 cmp     dword_6C2590, 8

.text:0043F0DA mov     eax, dword_6C257C

.text:0043F0DF jnb     short loc_43F0E6

.text:0043F0E1 mov     eax, offset dword_6C257C

.text:0043F0E6 loc_43F0E6:CODE XREF: .text:0043F0DF j

.text:0043F0E6 push    0

.text:0043F0E8 push    0

.text:0043F0EA push    0

.text:0043F0EC push    0

.text:0043F0EE push    0

.text:0043F0F0 push    eax

.text:0043F0F1 mov     ecx, esi//网址字符串指针

.text:0043F0F3 call     sub_5715C6//访问指定网址

.text:0043F0F8 mov     eax, [ebp+224h]

.text:0043F0FE push    0

.text:0043F100 push    0

.text:0043F102 push    CB_SETCURSEL

.text:0043F107 push    eax

.text:0043F108 call    ds:SendMessageW
```

由于执行到`43F0F3`时堆栈中的参数为`http://bcsearch.mdbchina.com/cache/index.html`，很明显这个函数就是用来访问网页的，向上浏览代码，看到`43F0BE`处的对`sub_443B60`的调用十分可疑，试着把`43F0BE`到`43F0FE`之前的全部代码都改成`NOP`后重新打开制作 Torrent 文件页，广告果然没有出现，说明猜想是正确的，`sub_443B60`就是 IE 内核浏览器对象的创建函数，在这个函数的入口点上设置断点后在点击任务属性和预览视频文件分别能找到类似的代码，都改成`NOP`就可以了，具体代码如下：

### 任务属性窗口

```asm
.text:0043CC40 call    CreateIECoreObject

// 此处已把函数 sub_443B60 名称改为 CreateIECoreObject

.text:0043CC45 push    0

.text:0043CC47 push    0

.text:0043CC49 push    300000h

.text:0043CC4E mov     ecx, ebp

.text:0043CC50 call     CWnd::ModifyStyle(ulong,ulong,uint)

.text:0043CC55 cmp     dword_6C2590, 8

.text:0043CC5C mov     eax, dword_6C257C

.text:0043CC61 jnb     short loc_43CC68

.text:0043CC63 mov     eax, offset dword_6C257C

.text:0043CC68 loc_43CC68: ; CODE XREF: .text:0043CC61 j

.text:0043CC68 push    0

.text:0043CC6A push    0

.text:0043CC6C push    0

.text:0043CC6E push    0

.text:0043CC70 push    0

.text:0043CC72 push    eax

.text:0043CC73 mov     ecx, ebp

.text:0043CC75 call    VisitURL
```

### 视频预览窗口

```asm
.text:0043F879 lea     ebx, [esi+0CCh]

.text:0043F87F push    ebx

.text:0043F880 mov     ecx, esi

.text:0043F882 mov     edi, 468h

.text:0043F887 call    CreateIECoreObject

.text:0043F88C push    1                 ; bEnable

.text:0043F88E mov     ecx, ebx

.text:0043F890 call       CWnd::EnableWindow(int)

.text:0043F895 push    1                 ; nCmdShow

.text:0043F897 mov     ecx, ebx

.text:0043F899 call     CWnd::ShowWindow(int)

.text:0043F89E push    2                 ; uFlags

.text:0043F8A0 push    3Eh               ; cy

.text:0043F8A2 push    1D5h              ; int

.text:0043F8A7 push    0                 ; Y

.text:0043F8A9 push    0                 ; X

.text:0043F8AB push    offset dword_7D3B40 ; int

.text:0043F8B0 mov     ecx, ebx

.text:0043F8B2 call    $LN4

.text:0043F8B7 push    0

.text:0043F8B9 push    0

.text:0043F8BB push    0B00000h

.text:0043F8C0 mov     ecx, ebx

.text:0043F8C2 call      CWnd::ModifyStyle(ulong,ulong,uint)

.text:0043F8C7 push    0

.text:0043F8C9 push    0

.text:0043F8CB push    20300h

.text:0043F8D0 mov     ecx, ebx

.text:0043F8D2 call      CWnd::ModifyStyleEx(ulong,ulong,uint)

.text:0043F8D7 mov     dword ptr [esi+1C8h], 1

.text:0043F8E1 call    sub_43FD40
```

去掉了视频预览页的广告后，为了美观还要用资源编辑软件把文件列表的窗体改大，挡住原来广告的位置，显示部分的修改就到此结束了，下面说一下三个和广告显示无关的优化。

第一个是`BitComet`软件运行后会自动试图向他的服务器发送ＵＤＰ数据，对软件中包含的广告进行更新，只要去掉下面的代码就可以禁止这个部分数据的发送和接收。

### 接收部分

```asm
.text:005632F7 call    esi ; WaitForMultipleObjects

.text:005632F9 test    eax, eax

.text:005632FB jz      short loc_563322 //改为 jmp  short loc_563322 即可

.text:005632FD lea     ecx, [ecx+0]

.text:00563300 loc_563300:  ; CODE XREF: sub_5632C0+60 j

.text:00563300 cmp     eax, 1

.text:00563303 jz      short loc_56330C

.text:00563305 cmp     eax, WAIT_TIMEOUT

.text:0056330A jnz     short loc_563311

.text:0056330C loc_56330C:   ; CODE XREF: sub_5632C0+43 j

.text:0056330C call    sub_563160//接收 UDP 数据

.text:00563311 loc_563311:  ; CODE XREF: sub_5632C0+4A j

.text:00563311 push    0FFFFFFFFh      ; dwMilliseconds

.text:00563313 push    0               ; bWaitAll

.text:00563315 lea     ecx, [esp+20h+Handles]

.text:00563319 push    ecx             ; lpHandles

.text:0056331A push    2               ; nCount

.text:0056331C call    esi ; WaitForMultipleObjects

.text:0056331E test    eax, eax

.text:00563320 jnz     short loc_563300

.text:00563322 push    offset stru_7D22B8 ; lpCriticalSection
```

### 发送部分

```asm
.text:0056359D call    ebx ; EnterCriticalSection

.text:0056359F mov     eax, dword_6C8338

.text:005635A4 xor     ebp, ebp

.text:005635A6 cmp     [eax+14h], ebp

.text:005635A9 jz      loc_5637BF//改为 jmp loc_5637BF
```

第二个是`BitComet`针对５q 校园网的下载限制的去除，在`BitComet`的中`IDA`分析页面中搜索`bt.5qzone.net`，就可以找到判断 tracker 服务器网址中是否包含`bt.5qzone.net`的部分，跳过这个判断就可以避开对校园网下载的速度限制，具体代码如下：

```asm
.text:005610B9 mov     eax, offset aBt_5qzone_net ; "bt.5qzone.net"

.text:005610BE mov     [esp+0B4h+var_C], 22h

.text:005610C9 lea     edx, [eax+2]

.text:005610CC lea     esp, [esp+0]

.text:005610D0 loc_5610D0: ; CODE XREF: sub_560610+AC9 j

.text:005610D0 mov     cx, [eax]

.text:005610D3 add     eax, 2

.text:005610D6 cmp     cx, bx

.text:005610D9 jnz     short loc_5610D0

.text:005610DB sub     eax, edx

.text:005610DD sar     eax, 1

.text:005610DF push    eax             ; 长度

.text:005610E0 push    ebx

.text:005610E1 push    offset aBt_5qzone_net ; "bt.5qzone.net"

.text:005610E6 lea     ecx, [esp+0C0h+var_34]

.text:005610ED call    sub_420150      ; 判断函数

.text:005610F2 cmp     eax, 0FFFFFFFFh

.text:005610F5 jz      loc_56132A;改为 jmp  loc_56132A
```

最后是禁止`BitComet`软件自动连接升级服务器进行自动更新检测：

在`IDA`中按`Alt+T`打开 Text search，在 string 框中输入`http://update.bitcomet.com/bitcomet/`后按 OK，就可以找到引用升级网页的地址，检查字符串的交叉引用发现只有函数`sub_419E40`种的`419FBE`一个地方引用了此升级网址，代码如下：

```asm
.text:00419FBE   push    offset aHttpUpdate_bit 

// "http://update.bitcomet.com/bitcomet/?ve"...

.text:00419FC3   push    edx

.text:00419FC4 mov     [esp+124h+var_84], ebp

.text:00419FCB mov     [esp+124h+var_88], ebx

.text:00419FD2 mov     word ptr [esp+124h+var_98], bx

.text:00419FDA call    sub_4125F0
```

向后看一下函数的代码就可以发现这个函数作用是生成访问判断是否更新网页的网址链接字符串，向上来到函数`sub_419E40`的开始处，可以找到如下代码：

```asm
.text:00419E75 mov     large fs:0, eax

.text:00419E7B mov     eax, [esp+118h+arg_0]

.text:00419E82 xor     ebx, ebx

.text:00419E84 cmp     dword_7D581C, ebx

.text:00419E8A mov     [esp+118h+var_E4], eax

.text:00419E8E jz      loc_41A44F
```

只要把`419E8E`处的`jz loc_41A44F`（`0F 84 BB 05 00 00`）改为`jmp loc_41A44F`（`90 E9 BB 05 00 00`）即可阻止生成访问链接，也就禁止了`BitComet`的自动更新检测。

到此，`BitComet`软件就差不多真的成为绿色软件了。

***

**【版权声明】**: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2006 年 12 月 18 日 23:26:37

`4dd466d0e1e0cc123afbe504da2462ca`
