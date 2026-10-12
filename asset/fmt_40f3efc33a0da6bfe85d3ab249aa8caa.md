
```
.rdata:0040349C off_40349C      dd offset ?GetRuntimeClass@CWinApp@@UBEPAUCRuntimeClass@@XZ ;CWinApp::GetRuntimeClass(void)
.rdata:004034A0             dd offset sub_401010
.rdata:004034A4             dd offset nullsub_1
.rdata:004034A8             dd offset nullsub_2
.rdata:004034AC             dd offset nullsub_1
.rdata:004034B0             dd offset ?OnCmdMsg@CCmdTarget@@UAEHIHPAXPAUAFX_CMDHANDLERINFO@@@Z ; CCmdTarget::OnCmdMsg(uint,int,void *,AFX_CMDHANDLERINFO *)
.rdata:004034B4             dd offset ?OnFinalRelease@CCmdTarget@@UAEXXZ ; CCmdTarget::OnFinalRelease(void)
.rdata:004034B8             dd offset ?IsInvokeAllowed@CCmdTarget@@UAEHJ@Z ; CCmdTarget::IsInvokeAllowed(long)
.rdata:004034BC             dd offset ?GetDispatchIID@CCmdTarget@@UAEHPAU_GUID@@@Z ; CCmdTarget::GetDispatchIID(_GUID *)
.rdata:004034C0             dd offset ?GetTypeInfoCount@CCmdTarget@@UAEIXZ ; CCmdTarget::GetTypeInfoCount(void)
.rdata:004034C4             dd offset ?GetTypeLibCache@CCmdTarget@@UAEPAVCTypeLibCache@@XZ ; CCmdTarget::GetTypeLibCache(void)
.rdata:004034C8             dd offset ?GetTypeLib@CCmdTarget@@UAEJKPAPAUITypeLib@@@Z ; CCmdTarget::GetTypeLib(ulong,ITypeLib * *)
.rdata:004034CC             dd offset sub_401000
;.......................................................
```

现在您应该会有个疑问，MFC 在哪里得到的地址？让我们快来看一下它的引用…

代码:

```
.text:004023B0 sub_4023B0     proc near
.text:004023B0             push    0
.text:004023B2             mov     ecx, offset dword_405498
.text:004023B7             call    ??0CWinApp@@QAE@PB_W@Z ; CWinApp::CWinApp(wchar_t const *)
.text:004023BC             push    offset sub_4023F0 ; void (__cdecl *)()
.text:004023C1             mov     dword_405498, offset off_40349C ;<-- this is our offset
.text:004023CB             call    _atexit
.text:004023D0             pop     ecx
.text:004023D1             retn
```

虚拟地址(VA)`004023B0` 包含在一个结构中。

代码:

```
.rdata:00403304 unk_403304     db    0
.rdata:00403305             db    0
.rdata:00403306             db    0
.rdata:00403307             db    0
.rdata:00403308             dd offset _pre_cpp_init
.rdata:0040330C             dd offset ??__E_afxInitAppState@@YAXXZ ; `dynamic initializer for '_afxInitAppState''(void)
.rdata:00403310             dd offset sub_4023B0
```

在`WinMain`运行之前由`__initterm`压入。

代码:

```
.text:00401AAC             push    offset unk_403314
.text:00401AB1             push    offset unk_403304
.text:00401AB6             call    _initterm
```

现在，让我们回到`AfxWinMain`中：

`call dword ptr [eax+98h]` (`40349C` + `98` = `00403534`) 调用...

代码:

```
.text:00403534             dd offset ?InitApplication@CWinApp@@UAEHXZ ; CWinApp::InitApplication(void)
```

...而 `call dword ptr [eax+58h]`, 就是 `pThread->InitInstance`, 调用了这个函数：

代码:

```
.rdata:004034F4             dd offset sub_401030
```

这个函数显示对话框，代码的主要部分都在这里了。

代码:

```
text:00401030 sub_401030     proc near
.text:00401030             push    ebp
.text:00401031             mov     ebp, esp
;..........................................................................
.text:0040109F             call    sub_401130
;--------------------------------------------------------------------------
;entrato nella call
.text:00401155             push    0               ; lpIconName
.text:00401157             push    66h             ; Dialog ID
.text:00401159             mov     ecx, esi
.text:0040115B             call    ??0CDialog@@QAE@IPAVCWnd@@@Z ; CDialog::CDialog(uint,CWnd *)
.text:00401160             mov     [esp+14h+var_4], 0
.text:00401168             mov     dword ptr [esi], offset off_403744 ;virtual functions table offset which is store
; in CDialog.DoModal -> CDialog__PreModal -> AfxHookWindowCreate
.text:0040116E             call    ?AfxGetModuleState@@YGPAVAFX_MODULE_STATE@@XZ ; AfxGetModuleState(void)
;exit the call
;---------------------------------------------------------------------------
.text:004010A4             lea     edx, [esp+8+arg_4]
.text:004010A8             mov     [esp+8+arg_88], 0
.text:004010B3             mov     ecx, edx
.text:004010B5             mov     [esi+20h], edx
.text:004010B8             call    ?DoModal@CDialog@@UAEHXZ ; CDialog::DoModal(void)
.text:004010BD             lea     ecx, [esp+8+arg_4]
.text:004010C1             mov     [esp+8+arg_88], 0FFFFFFFFh
.text:004010CC             call    ??1CDialog@@UAE@XZ ; CDialog::~CDialog(void)
;..........................................................................
.text:004010E3             mov     esp, ebp
.text:004010E5             pop     ebp
.text:004010E6             retn
```

### 获取 MESSAGE_MAP

但`MESSAGE_MAP`在哪呢？ Message Map 可以再这里找到：

代码:

```
BOOL CWnd::OnWndMsg(UINT message, WPARAM wParam, LPARAM lParam, LRESULT* pResult)
{
    // ....
    const AFX_MSGMAP* pMessageMap;
    pMessageMap = GetMessageMap();
    // ....
      if ((lpEntry = AfxFindMessageEntry(pMessageMap->lpEntries, message, 0, 0)) != NULL)
    // ...

}
```

这是反汇编后：

代码:

```
.text:78312E91             mov     eax, [edi] ; eax = 403744
.text:78312E93             mov     ecx, edi
.text:78312E95             call    dword ptr [eax+30h] ; eax+30h = 00403774 = GetMessageMap()
;.rdata:00403774             dd offset sub_4011E0
;...................................................................
.text:78312F1B             push    0
.text:78312F1D             push    0
.text:78312F1F             jnb     short loc_78312F67
.text:78312F21             push    [ebp+arg_0] ;messagge
.text:78312F24             push    dword ptr [esi+4] ; lpEntries (0040362C)
.text:78312F27             call    AfxFindMessageEntry
```

从 `78312E95` 的调用进入到了这里:

代码:

```
;GetMessageMap()
.text:004011E0             mov     eax, offset off_403628 ;eax = pMessageMap
.text:004011E5             retn
;----------------------------------------------------------------
;pMessageMap
.rdata:00403628 off_403628     dd offset ?GetThisMessageMap@CDialog@@KGPBUAFX_MSGMAP@@XZ
.rdata:00403628                             ; CDialog::GetThisMessageMap(void)
.rdata:0040362C             dd offset unk_403580 ;pMessageMap->lpEntries
```

在 `403580` 处就是这个对话框的 `MESSAGE_MAP`。

我们可以通过下面的方式快速获取 `MessageMap`:

1.  在 `CDialog:DoModal` 调用之前找一条指令，类似于： `mov dword ptr [esi], offset off_XXXXXX` (通常是用来定位虚函数表的)。
2.  在指令中的 `offset` 地址上加 `0x30` 就得到了 `GetMessageMap` 函数： 在函数中找到一条指令 `mov eax, offset off_XXXXXX`, `eax` 为 `pMessageMap`。
3.  在 `pMessageMap` 上`+4` 就得到了对话框的 `MessageMap`。

现在来看一个例子。 这是软件资源信息：

代码:

```
CONTROL "Register", 1006, BUTTON,   //1006 = 0x3ee
CONTROL "About", 1007, BUTTON,      //1007 = 0x3ef
CONTROL "Cancel", 1008, BUTTON,     //1008 = 0x3f0
```

下面是该资源所对应的`MESSAGE_MAP`结构体

代码:
