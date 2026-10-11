
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
