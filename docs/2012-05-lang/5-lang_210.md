
# MFC 逆向初级研究（1）

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-87.htm`](https://www.pediy.com/kssd/pediy09/pediy09-87.htm)

> 【文章标题】: MFC 逆向初级研究(1)  
> 【文章作者】: 北斗之摇光  
> 【作者邮箱】: hardlywhen@hotmail.com  
> 【下载地址】: 自己搜索下载  
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  

---

## 【详细过程】

### 引言

本文主要针对微软的 `VC++6.0` 中使用 `MFC` 产生的 `EXE` 文件的逆向研究，我曾经使用微软的 `Visual Studio 2005` 编译了一个 `EXE` 文件，通过 `IDA` 反汇编以后发现该文件与 `VC++6.0` 产生的文件还是有所区别，因此特别在此声明一下。文中主要使用了 `IDA pro 5.0` 和在看雪 (`www.pediy.com`) 下载的 `OllyICE` 作为工具对目标文件进行反汇编。在此也感谢看雪论坛的各位的无私奉献，在研究过程的中的困难多通过各位的帖子得到了帮助。

### 逆向的关键

我认为逆向的关键主要是要弄明白目标文件的算法和实现过程，在 `Window` 操作系统下，软件的实现过程就体现在其对 `Window` 消息的处理，而软件的算法则包含在处理的具体过程中。对于通过 `SDK` 编写的"传统"的 `Windows` 应用程序基本都具备几个共同的特征：`WinMain` 函数、`WinProc` 函数、窗口注册、消息循环。对于这类目标文件的分析主要集中的 `WinProc` 的分析上，`WinProc` 的函数地址获得一般是通过窗口注册函数中的参数获得。（由于我对于这类文件没有具体逆向过，所以只是大概的说说，有不对的地方请各位不要客气，尽管拍砖）

而使用 `MFC`（Microsoft Function Class）顾名思义，该类库主要封装了大部分的 `Windows API` 函数所以在代码中看不到原本的 `SDK` 编程中的消息循环、窗口过程函数等等东西，所有这些封装在相应的 `mfcxx.dll` 中，让程序员能够专著与处理过程与算法。这种做法于逆向而言有好处也有坏处：

*   **坏处**就是加大了对于 `MFC` 产生的 `EXE` 文件的逆向难度，让许多的和我一样的菜鸟迷失在汇编代码中找不找北了，基本主要就靠猜测实现过程中用到了那些函数，然后对文件导入表的函数下断点来寻找我们所需要的处理过程；
*   **好处**就是这样的做法使得 `EXE` 文件中主要都是目标程序的 `Window` 消息处理流程以及算法，而且 `dll` 中的大部分函数的功能都能在 `MSDN` 中查到。如果能够通过对目标文件的分析得到这个 `Window` 消息处理流程和算法架构，基本上我们就可以重写整个软件；

要做到上面的目标，首先我们要对 `MFC` 有所了解，推荐没有基础的兄弟们读读候俊杰的《深入浅出 MFC》。该书在逆向过程中完全可以作为一本参考书，让你能通过源代码了解实现过程，网上有很多该书的电子版下载。

### 一个逆向 MFC 产生的 EXE 文件的例子

下面我们就通过一个具体的例子来学习一下如何从目标文件中挖到我们需要的东西。首先我们来产生一个需要的 `EXE` 文件。在此我假定各位对 `MFC` 有过一定的使用经验，毕竟逆向分析才是本文的重点。

#### 1．产生例子所需要的目标文件

我们通过 `VC++6.0` 的向导来产生一个名为 `ReverseMFC` 的工程，这个工程的设置情况如下：

*   Application type of fff:
*   Dialog-Based Application targeting:
    *   Win32
*   Classes to be created:
    *   Application: `CFffApp` in `ReverseMFC.h` and `ReverseMFC.cpp`
    *   Dialog: `CFffDlg` in `ReverseMFCDlg.h` and `ReverseMFCDlg.cpp`
*   Features:
    *   Uses shared DLL implementation (`MFC42.DLL`)
    *   Localizable text in: 中文\[中国\]

直接编译以后就能够运行，为了确定我们是否正确的分析的整个目标文件，在该对话框中加入一个我们自定义的按钮如下，对于该按钮的处理函数如下设定为：`AfxMessageBox("I find it!", MB_OK)`；编译后就得到了我们需要的目标文件。

现在我们得到了所需要的目标文件，在 `IDA` 中载入该文件。在此我们最好是产生 `Release` 版本的 `EXE` 文件，毕竟所有的发布软件都是 `Release` 版本的。

#### 2．具体分析

在 `IDA` 中按 `Ctrl＋S` 找到 `.rdata` 段，该段主要存储了目标文件的类运行时创建信息、`MessageMap` 信息、`MessageEntry` 信息、虚函数表、`RTTI` 数据（如果编译选项中选择了支持 `RTTI` 的话）。

在到达 `.rdata` 段后我们可以看到这样的代码，对数据进行格式转换后可以得到如下图所示的数据。

```asm
.rdata:004021C0 ; 屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯?
.rdata:004021C0
.rdata:004021C0 ; Segment type: Pure data
.rdata:004021C0 ; Segment permissions: Read
.rdata:004021C0 _rdata           segment para public 'DATA' use32
.rdata:004021C0                 assume cs:_rdata
.rdata:004021C0                 ;org 4021C0h
.rdata:004021C0 off_4021C0       dd offset sub_401000    ; DATA XREF: sub_401010o
.rdata:004021C4                 dd offset dword_4021C8
.rdata:004021C8 dword_4021C8    dd 111h                  ; DATA XREF: .rdata:004021C4o
.rdata:004021CC                 dd 0
.rdata:004021D0                 dd 0E146h
.rdata:004021D4                 dd 0E146h
.rdata:004021D8                 dd 0Ch
.rdata:004021DC                 dd offset CWinApp::OnHelp(void)
.rdata:004021E0                 dd 0
.rdata:004021E4                 dd 0
.rdata:004021E8                 dd 0
.rdata:004021EC                 dd 0
.rdata:004021F0                 dd 0
.rdata:004021F4                 dd 0
.rdata:004021F8 off_4021F8      dd offset CWinApp::GetRuntimeClass(void)
.rdata:004021F8                                           ; DATA XREF: unknown_libname_1-56o
.rdata:004021FC                 dd offset sub_401040
.rdata:00402200                 dd offset nullsub_2
.rdata:00402204                 dd offset nullsub_3
.rdata:00402208                 dd offset nullsub_2
.rdata:0040220C                 dd offset CCmdTarget::OnCmdMsg(uint,int,void *,AFX_CMDHANDLERINFO *)
```

其中的 `off_4021C0` 就是一个 `MessageMap` 数据；`dword_4021C8` 就是 `MessageMap` 所指的 `MessageEntry` 数据；`off_4021F8` 就是一个类的虚函数表的开始位置。那么具体这些数据时那个类的相关数据呢？如此判断的依据是什么？

首先我们知道 `MessageEntry` 是的数据结构定义如下，而且以 6 个 0 表示整个数组的结束。

```c
struct AFX_MSGMAP_ENTRY
{
  UINT nMessage;   // windows message
  UINT nCode;      // control code or WM_NOTIFY code
  UINT nID;        // control ID (or 0 for windows messages)
  UINT nLastID;    // used for entries specifying a range of control id's
  UINT nSig;       // signature type (action) or pointer to message #
  AFX_PMSG pfn;    // routine to call (or special value)
};
```

因此我们有理由假设"`dword_4021C8` 就是 `MessageMap` 所指的 `MessageEntry` 数据"。

而 `MessageMap` 数据结构定义如下：

```c
struct AFX_MSGMAP
{
#ifdef _AFXDLL
  const AFX_MSGMAP* (PASCAL* pfnGetBaseMap)();
#else
  const AFX_MSGMAP* pBaseMap;
#endif
  const AFX_MSGMAP_ENTRY* lpEntries;
};
```

`off_4021C0` 的两个数据中第二个数据恰恰就是我们前面假设为 `MessageEntry` 的指针，跟入其第一个数据，我们看到如下的代码：

```asm
.text:00401000 ; ************ S U B R O U T I N E ************
.text:00401000
.text:00401000
.text:00401000 sub_401000       proc near                 ; DATA XREF: .rdata:off_4021C0o
.text:00401000                   mov     eax, ds:AFX_MSGMAP const CWinApp::messageMap
.text:00401005                   retn
.text:00401005
.text:00401005 sub_401000       endp
```

恰恰是一个返回基类的 `MessageMap` 的函数。因此我们也同样有理由假设"`off_4021C0` 就是一个 `MessageMap` 数据"。

对于虚函数表的假设是如何被证明呢？首先我们要知道关于虚函数表的一点知识：虚函数表由虚函数的地址组成，表中函数地址的顺序和它们第一次出现的顺序（即在类定义的顺序）一致。若有重载的函数，则替换掉基类函数的地址。通过这个我们可以知道 `MFC` 中虚函数表中的函数顺序必然是先按照 `CObject->CCmdtarget->...` 这个类继承顺序中的虚函数顺序来处理虚函数表中的函数顺序的。只要证明这个我们"假设的虚函数"中的函数顺序与上面提到的知识相符合则有理由说明我们的假设成立。

首先来看 `CObject` 中虚函数的顺序，在查看 `CObject` 的声明文件后得到了这个类的虚函数顺序：

```cpp
virtual CRuntimeClass* GetRuntimeClass() const;
virtual ~CObject();  // virtual destructors are necessary
virtual void Serialize(CArchive& ar);
#if defined(_DEBUG) || defined(_AFXDLL)
  // Diagnostic Support
  virtual void AssertValid() const;
  virtual void Dump(CDumpContext& dc) const;
```

再来查看 `CCmdtarget` 的虚函数顺序，在查看 `CObject` 的声明文件后得到了这个类的虚函数顺序：

```cpp
DECLARE_DYNAMIC(CCmdTarget)；
virtual BOOL OnCmdMsg(UINT nID, int nCode, void* pExtra, 
  AFX_CMDHANDLERINFO* pHandlerInfo);
#ifndef _AFX_NO_OLE_SUPPORT
  // called when last OLE reference is released
  virtual void OnFinalRelease();
#endif
#ifndef _AFX_NO_OLE_SUPPORT
  // called before dispatching to an automation handler function
  virtual BOOL IsInvokeAllowed(DISPID dispid);
  virtual BOOL GetDispatchIID(IID* pIID);
  virtual UINT GetTypeInfoCount();
  virtual CTypeLibCache* GetTypeLibCache();
  virtual HRESULT GetTypeLib(LCID lcid, LPTYPELIB* ppTypeLib);
```

之所以还要列出"`DECLARE_DYNAMIC(CCmdTarget)；`"是因为这个宏的定义如下：

```cpp
#define DECLARE_DYNAMIC(class_name) \\
protected: \\
  static CRuntimeClass* PASCAL _GetBaseClass(); \\
public: \\
  static const AFX_DATA CRuntimeClass class##class_name; \\
  virtual CRuntimeClass* GetRuntimeClass() const; \\
```

这个 `virtual CRuntimeClass* GetRuntimeClass() const;` 覆盖掉了一开始的 `CObject` 的相对应函数。依次按照类的顺序对照下来，就可以知道该表确实是虚函数表。同时，对应的 `GetMessageMap` 虚函数的位置上跟入后，可以得到如下代码：



## 汇编代码分析

```assembly
.text:00401010 ; ******************************************************* SUBROUTINE *******************************************************
.text:00401010
.text:00401010
.text:00401010 sub_401010      proc near            ; DATA XREF: .rdata:00402228
.text:00401010                mov     eax, offset off_4021C0
.text:00401015                retn
.text:00401015
.text:00401015 sub_401010      endp
```

恰恰是返回了我们之前假设的 `MessageMap` 的地址。

---

**【版权声明】** 本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢！

2007 年 03 月 15 日 17:15:29

**【文章标题】** MFC 逆向初级研究（2）
**【文章作者】** 北斗之摇光
**【作者邮箱】** hardlywhen@hotmail.com
**【下载地址】** 自己搜索下载
**【作者声明】** 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

### 详细过程

在《MFC 逆向初级研究(1)》中我们初步证明了我们的假设，在经过对几个文件的反汇编以后，现在我们可以总结出一条经过证明的推断：

"MFC 中的类信息的存储可能是按照如下的顺序来存储的：
1. 该类的 `MessageMap`
2. 该类的 `MessageEntry`
3. 该类的 RTTI Complete Object Locator(如果该类有 RTTI 信息）
4. 该类的虚函数表"

按照这样的顺序看下来，我们在 `.rdata` 区域开始一点一点往下，会得到几个类的信息。下面是通过 `IDA` 对我们的例子程序反编译以后我整理好的 `.rdata` 的内容，仔细查看虚函数表，你会发现有部分虚函数没有像其他的虚函数那样指向其父类的相关函数，这是因为在我们的例子程序中对这些函数进行了重载。那么到底重载的是哪些函数呢？你可以根据虚函数在其父类中的顺序确定，在注释中我已经标出。

```assembly
.rdata:004021C0 ; 屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯?
.rdata:004021C0
.rdata:004021C0 ; Segment type: Pure data
.rdata:004021C0 ; Segment permissions: Read
.rdata:004021C0 _rdata          segment para public 'DATA' use32
.rdata:004021C0                 assume cs:_rdata
.rdata:004021C0                 ;org 4021C0h
.rdata:004021C0 off_4021C0      dd offset sub_401000     ; DATA XREF: sub_401010
.rdata:004021C4                 dd offset dword_4021C8
.rdata:004021C8 dword_4021C8    dd 111h                 ; DATA XREF: .rdata:004021C4
.rdata:004021CC                 dd 0
.rdata:004021D0                 dd 0E146h
.rdata:004021D4                 dd 0E146h
.rdata:004021D8                 dd 0Ch
.rdata:004021DC                 dd offset CWinApp::OnHelp(void)
.rdata:004021E0                 dd 0
.rdata:004021E4                 dd 0
.rdata:004021E8                 dd 0
.rdata:004021EC                 dd 0
.rdata:004021F0                 dd 0
.rdata:004021F4                 dd 0
.rdata:004021F8 off_4021F8      dd offset CWinApp::GetRuntimeClass(void)         \虚函数表的开始
.rdata:004021F8                                ; DATA XREF: unknown_libname_1-56    \这个引用可能在类构造函数中
.rdata:004021FC                 dd offset sub_401040                       \类析构函数
.rdata:00402200                 dd offset nullsub_2
.rdata:00402204                 dd offset nullsub_3
.rdata:00402208                 dd offset nullsub_2
.rdata:0040220C                 dd offset CCmdTarget::OnCmdMsg(uint,int,void *,AFX_CMDHANDLERINFO *)
.rdata:00402210                 dd offset CCmdTarget::OnFinalRelease(void)
.rdata:00402214                 dd offset CCmdTarget::IsInvokeAllowed(long)
.rdata:00402218                 dd offset CCmdTarget::GetDispatchIID(_GUID *)
.rdata:0040221C                 dd offset CCmdTarget::GetTypeInfoCount(void)
.rdata:00402220                 dd offset CCmdTarget::GetTypeLibCache(void)
.rdata:00402224                 dd offset CCmdTarget::GetTypeLib(ulong,ITypeLib * *)
.rdata:00402228                 dd offset sub_401010                     \GetMessageMap
.rdata:0040222C                 dd offset CCmdTarget::GetCommandMap(void)
.rdata:00402230                 dd offset CCmdTarget::GetDispatchMap(void)
.rdata:00402234                 dd offset CCmdTarget::GetConnectionMap(void)
.rdata:00402238                 dd offset CCmdTarget::GetInterfaceMap(void)
.rdata:0040223C                 dd offset CCmdTarget::GetEventSinkMap(void)
.rdata:00402240                 dd offset CCmdTarget::OnCreateAggregates(void)
.rdata:00402244                 dd offset CCmdTarget::GetInterfaceHook(void const *)
.rdata:00402248                 dd offset CCmdTarget::GetExtraConnectionPoints(CPtrArray *)
.rdata:0040224C                 dd offset CCmdTarget::GetConnectionHook(_GUID const &)
.rdata:00402250                 dd offset sub_4010B0                     \InitInstance
.rdata:00402254                 dd offset CWinApp::Run(void)
.rdata:00402258                 dd offset CWinThread::PreTranslateMessage(tagMSG *)
.rdata:0040225C                 dd offset CWinThread::PumpMessage(void)
.rdata:00402260                 dd offset CWinApp::OnIdle(long)
.rdata:00402264                 dd offset CWinThread::IsIdleMessage(tagMSG *)
.rdata:00402268                 dd offset CWinApp::ExitInstance(void)
.rdata:0040226C                 dd offset CWinApp::ProcessWndProcException(CException *,tagMSG const *)
.rdata:00402270                 dd offset CWinThread::ProcessMessageFilter(int,tagMSG *)
.rdata:00402274                 dd offset CWinThread::GetMainWnd(void)
.rdata:00402278                 dd offset CWinThread::Delete(void)
.rdata:0040227C                 dd offset CWinApp::OpenDocumentFile(char const *)
.rdata:00402280                 dd offset CWinApp::AddToRecentFileList(char const *)
.rdata:00402284                 dd offset CWinApp::InitApplication(void)
.rdata:00402288                 dd offset CWinApp::SaveAllModified(void)
.rdata:0040228C                 dd offset CWinApp::DoMessageBox(char const *,uint,uint)
.rdata:00402290                 dd offset CWinApp::DoWaitCursor(int)
.rdata:00402294                 dd offset CWinApp::OnDDECommand(char *)
.rdata:00402298                 dd offset CWinApp::WinHelpA(ulong,uint)
.rdata:0040229C                 align 10h
.rdata:004022A0 off_4022A0      dd offset sub_4011F0     ; DATA XREF: sub_401200
.rdata:004022A4                 dd offset dword_4022A8
.rdata:004022A8 dword_4022A8    dd 0Fh                  ; DATA XREF: .rdata:004022A4
.rdata:004022AC                 dd 0
.rdata:004022B0                 dd 0
.rdata:004022B4                 dd 0
.rdata:004022B8                 dd 0Ch
.rdata:004022BC                 dd offset sub_401250
.rdata:004022C0                 dd 37h
.rdata:004022C4                 dd 0
.rdata:004022C8                 dd 0
.rdata:004022CC                 dd 0
.rdata:004022D0                 dd 23h
.rdata:004022D4                 dd offset sub_401310
.rdata:004022D8                 dd 111h
.rdata:004022DC                 dd 0
.rdata:004022E0                 dd 3E8h
.rdata:004022E4                 dd 3E8h
.rdata:004022E8                 dd 0Ch
.rdata:004022EC                 dd offset sub_401320
.rdata:004022F0                 dd 0
.rdata:004022F4                 dd 0
.rdata:004022F8                 dd 0
.rdata:004022FC                 dd 0
.rdata:00402300                 dd 0
.rdata:00402304                 dd 0
.rdata:00402308 off_402308      dd offset CDialog::GetRuntimeClass(void)
.rdata:00402308                                ; DATA XREF: sub_401150+31
.rdata:0040230C                 dd offset sub_4011C0                \类析构函数
.rdata:00402310                 dd offset nullsub_2
```



```
.rdata:00402314             dd offset nullsub_3
.rdata:00402318             dd offset nullsub_2
.rdata:0040231C             dd offset CDialog::OnCmdMsg(uint,int,void*,AFX_CMDHANDLERINFO*)
.rdata:00402320             dd offset CWnd::OnFinalRelease(void)
.rdata:00402324             dd offset CCmdTarget::IsInvokeAllowed(long)
.rdata:00402328             dd offset CCmdTarget::GetDispatchIID(_GUID*)
.rdata:0040232C             dd offset CCmdTarget::GetTypeInfoCount(void)
.rdata:00402330             dd offset CCmdTarget::GetTypeLibCache(void)
.rdata:00402334             dd offset CCmdTarget::GetTypeLib(ulong,ITypeLib**)
.rdata:00402338             dd offset sub_401200                     \\GetMessageMap
.rdata:0040233C             dd offset CCmdTarget::GetCommandMap(void)
.rdata:00402340             dd offset CCmdTarget::GetDispatchMap(void)
.rdata:00402344             dd offset CCmdTarget::GetConnectionMap(void)
.rdata:00402348             dd offset CCmdTarget::GetInterfaceMap(void)
.rdata:0040234C             dd offset CCmdTarget::GetEventSinkMap(void)
.rdata:00402350             dd offset CCmdTarget::OnCreateAggregates(void)
.rdata:00402354             dd offset CCmdTarget::GetInterfaceHook(void const*)
.rdata:00402358             dd offset CCmdTarget::GetExtraConnectionPoints(CPtrArray*)
.rdata:0040235C             dd offset CCmdTarget::GetConnectionHook(_GUID const&)
.rdata:00402360             dd offset CWnd::PreSubclassWindow(void)
.rdata:00402364             dd offset CWnd::Create(char const*,char const*,ulong,tagRECT const&,CWnd*,uint,CCreateContext*)
.rdata:00402368             dd offset CWnd::DestroyWindow(void)
.rdata:0040236C             dd offset CWnd::PreCreateWindow(tagCREATESTRUCTA&)
.rdata:00402370             dd offset CWnd::CalcWindowRect(tagRECT*,uint)
.rdata:00402374             dd offset CWnd::OnToolHitTest(CPoint,tagTOOLINFOA*)
.rdata:00402378             dd offset CWnd::GetScrollBarCtrl(int)
.rdata:0040237C             dd offset CWnd::WinHelpA(ulong,uint)
.rdata:00402380             dd offset CWnd::ContinueModal(void)
.rdata:00402384             dd offset CWnd::EndModalLoop(int)
.rdata:00402388             dd offset CWnd::OnCommand(uint,long)
.rdata:0040238C             dd offset CWnd::OnNotify(uint,long,long*)
.rdata:00402390             dd offset CWnd::GetSuperWndProcAddr(void)
.rdata:00402394             dd offset nullsub_4                    \\DoDataExchange
.rdata:00402398             dd offset sub_401330                   \\BeginModalState
.rdata:0040239C             dd offset sub_401340                   \\EndModalState
.rdata:004023A0             dd offset CDialog::PreTranslateMessage(tagMSG*)
.rdata:004023A4             dd offset CWnd::OnAmbientProperty(COleControlSite*,long,tagVARIANT*)
.rdata:004023A8             dd offset CWnd::WindowProc(uint,uint,long)
.rdata:004023AC             dd offset CWnd::OnWndMsg(uint,uint,long,long*)
.rdata:004023B0             dd offset CWnd::DefWindowProcA(uint,uint,long)
.rdata:004023B4             dd offset CWnd::PostNcDestroy(void)
.rdata:004023B8             dd offset CWnd::OnChildNotify(uint,uint,long,long*)
.rdata:004023BC             dd offset CDialog::CheckAutoCenter(void)
.rdata:004023C0             dd offset CWnd::IsFrameWnd(void)
.rdata:004023C4             dd offset CDialog::SetOccDialogInfo(_AFX_OCC_DIALOG_INFO*)
.rdata:004023C8             dd offset CDialog::DoModal(void)
.rdata:004023CC             dd offset sub_401210                   \\OnInitDialog
.rdata:004023D0             dd offset CDialog::OnSetFont(CFont*)
.rdata:004023D4             dd offset CDialog::OnOK(void)
.rdata:004023D8             dd offset CDialog::OnCancel(void)
.rdata:004023DC             dd offset CDialog::PreInitDialog(void)
.rdata:004023E0 dword_4023E0 dd 0FFFFFFFFh         ; DATA XREF: start+5o     \\再以下就是其他数据
.rdata:004023E4             dd offset sub_4016AE
.rdata:004023E8             dd offset sub_4016C2
.rdata:004023EC             dd 0
.rdata:004023F0 stru_4023F0  dd 19930520h            ; Magic
.rdata:004023F0                              ; DATA XREF: .text:loc_401768o
.rdata:004023F0             dd 1                 ; Count
.rdata:004023F0             dd offset stru_4023F0.Info; InfoPtr
.rdata:004023F0             dd 0                 ; CountDtr
.rdata:004023F0             dd 0                 ; DtrPtr
.rdata:004023F0             dd 3 dup(0)           ; _unk
.rdata:004023F0             dd -1                 ; Info.Id
.rdata:004023F0             dd offset sub_401760   ; Info.Proc
.rdata:00402418 stru_402418  dd 19930520h            ; Magic
```

经过以上步骤以后我们对于`MessageMap`、`MessageEntry`、虚函数表三部分的处理已经完成，接下来的任务就是确定我们已经找好相关数据的类到底是哪个类？根据 MFC 的了解，我们知道 MFC 在建立之处会让你选择建立哪个类型的工程，其中有：多文档、单文档、对话框三种可以选择。

对于我们这个工程中选择的 Dialog 类型，其建立之初就有 2 个类（如果选择包含 About Dialog 的话则有 3 个类）:`CReverseMFC` 和 `CReverseMFCDlg` 类。

对于这几个类的作用在《深入浅出 MFC》中有过介绍，我们现在需要确定的是我们刚才找到的数据具体是那个类呢？

我们可以根据虚函数表的继承关系来找，`CReverseMFCDlg` 类继承自 `CDialog`<-`CWnd`<-`CCmdTarget`；
```
CReverseMFC 类继承自 CWinApp<-CWinthread<-CCmdtarget
```
这样基本确定了两个类以及他们的`MeeeageMap`。在确定`MessageMap`后，我们就可以根据这个消息处理流程来寻找我们需要的算法和软件处理过程了。

那么如何来完成在本文之初我们所设立的目标呢－"找到按钮的相关处理过程"？

首先我们要知道按钮的资源代码是多少，这一点我们可以通过相关的资源处理软件来处理（我用的是`ResHacker`)可以查到按钮的的资源 ID 为 1000，也就是 0X3E8。MFC 消息处理中将 Button 消息作为 Command 类消息，在`windows.h`头文件中对`WM_COMMAND`消息的定义为 0X111。回顾对`MessageEntry`的数据定义，第一个为 windows 消息代码，第 3 个为控件 ID。而按钮类的处理过程一般是放在对话框类中实现，因此查看类`CReverseMFCDlg`类的`MessageEntry`可以找到了如下的代码：

```
.rdata:004022D8             dd 111h                    \\windows message id
.rdata:004022DC             dd 0                        \\nCode
.rdata:004022E0             dd 3E8h                    \\nID
.rdata:004022E4             dd 3E8h                    \\nLastID
.rdata:004022E8             dd 0Ch                      \\nSig
.rdata:004022EC             dd offset sub_401320         \\pfn
```

跟入`sub_401320`：

```
.text:00401320 ; *************** S U B R O U T I N E ************
.text:00401320

.text:00401320 sub_401320      proc near               ; DATA XREF: .rdata:004022ECo
.text:00401320                 push    0
.text:00401322                 push    0
.text:00401324                 push    offset s_IFindIt ; "I find it!"
.text:00401329                 call    AfxMessageBox(char const*,uint,uint)
.text:0040132E                 retn
.text:0040132E sub_401320      endp
```

恰恰是我们所定义的按钮行为！如此，我们便跟随着 EXE 文件给我们留下的“藤”摸到了我们需要的“瓜”，这些藤为我们编织了一张软件架构的大网，需要我们仔细的分析才能达成目的。

总结以上，对于 VC++6.0 MFC 产生的 EXE 文件的逆向我们可以归结以下几个要点（当然，这个文件必须是没加壳同时确定是 VC++6.0 产生的）：

1. `IDA`反汇编后直接跳到`.rdata`段，此段必然是相关的用于存储相关的虚函数表、`MessageMap`、`MessageEntry`等信息；
2. 查找虚函数表时可以直接搜索`GetRunTimeClass`函数，该函数必然时各个虚函数表的开始；



通过完善虚函数表，我们可以找到重载的函数地址，在完善函数表时要注意确定正确的函数顺序；

### 结语

在此，这篇文章只是简单的对 `MFC` 逆向做了一个肤浅的分析，希望能起到抛砖引玉的作用的同时也能让刚刚接触 `MFC` 逆向的兄弟们有点头绪，知道该如何去找到自己需要的函数处理过程。毕竟本文的例子只是一个非常简单的工程，在正式的商业软件中，往往有着十几个基本类以及自定义类，还有大量的自定义消息和数据结构，类的成员变量和成员函数等等。这些都需要经验和时间的积累，小菜由于也是初次接触 `MFC` 的逆向，还没有经验，等俺逆向过几个软件后再发吧，免的贻笑大方。

另：让笨笨熊版主失望了，俺只写了 2，没写 3、4、5、6 那么多，嘿嘿，经验不足

--------------------------------------------------------------------------------
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 03 月 16 日 11:14:49
`4dd466d0e1e0cc123afbe504da2462ca`
