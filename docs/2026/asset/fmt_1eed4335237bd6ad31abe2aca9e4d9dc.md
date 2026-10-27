
```
PAGE:0058BB1B ; __stdcall DbgkpConvertKernelToUserStateChange(x, x)
PAGE:0058BB1B _DbgkpConvertKernelToUserStateChange@8 proc near
PAGE:0058BB1B                                         ; CODE XREF: NtWaitForDebugEvent(x,x,x,x)+182↓p
PAGE:0058BB1B  
PAGE:0058BB1B arg_StateChange = dword ptr  8
PAGE:0058BB1B arg_DbgkmDebugEvent = dword ptr  0Ch
PAGE:0058BB1B  
PAGE:0058BB1B                 mov     edi, edi
PAGE:0058BB1D                 push    ebp
PAGE:0058BB1E                 mov     ebp, esp
PAGE:0058BB20                 mov     edx, [ebp+arg_DbgkmDebugEvent] ; 将内核调试消息赋给edx
PAGE:0058BB23                 mov     ecx, [edx+_DBGKM_DEBUG_EVENT.ClientId.UniqueProcess]
PAGE:0058BB26                 mov     eax, [ebp+arg_StateChange] ; 将StateChange地址赋给eax
PAGE:0058BB29                 mov     [eax+DBGUI_WAIT_STATE_CHANGE.AppClientId.UniqueProcess], ecx
PAGE:0058BB2C                 mov     ecx, [edx+_DBGKM_DEBUG_EVENT.ClientId.UniqueThread]
PAGE:0058BB2F                 mov     [eax+DBGUI_WAIT_STATE_CHANGE.AppClientId.UniqueThread], ecx
```

由于结构中存在联合体，所以需要根据内核调试消息的`ReturnedStatus`来选择如何为剩下成员赋值并设置`NewState`的值

```
PAGE:0058BB32                 mov     ecx, [edx+_DBGKM_DEBUG_EVENT.ApiMsg.ReturnedStatus]
PAGE:0058BB35                 sub     ecx, 0
PAGE:0058BB38                 push    esi
PAGE:0058BB39                 push    edi
PAGE:0058BB3A                 jz      short loc_58BBB2
PAGE:0058BB3C                 dec     ecx
PAGE:0058BB3D                 jz      short loc_58BB9E
PAGE:0058BB3F                 dec     ecx
PAGE:0058BB40                 jz      short loc_58BB87
PAGE:0058BB42                 dec     ecx
PAGE:0058BB43                 jz      short loc_58BB79
PAGE:0058BB45                 dec     ecx
PAGE:0058BB46                 jz      short loc_58BB71
PAGE:0058BB48                 dec     ecx
PAGE:0058BB49                 jz      short loc_58BB5A
PAGE:0058BB4B                 dec     ecx
PAGE:0058BB4C                 jnz     loc_58BBE0
PAGE:0058BB52                 mov     [eax+DBGUI_WAIT_STATE_CHANGE.NewState], 0Ah
PAGE:0058BB58                 jmp     short loc_58BB7F
```

当函数返回的时候，此时局部变量`var_StateChange`中就保存了转换以后的`DBGUI_WAIT_STATE_CHANGE`结构，接下来就需要将其赋值到参数`StateChange`指定的区域中，该区域是用户层`DBGUI_WAIT_STATE_CHANGE`结构的地址

```
PAGE:0058BE39                 lea     esi, [ebp+var_StateChange]
PAGE:0058BE3F                 mov     edi, [ebp+StateChange]
PAGE:0058BE42                 rep movsd
```

当函数返回到用户层的时候，也就是`DbgUiWaitStateChange`函数返回的时候，此时已经成功从内核中将调试对象的调试消息队列中保存的消息转换成`DBGUI_WAIT_STATE_CHANGE`结构。

`DbgUiConvertStateChangeStructure`就可以根据`DBGUI_WAIT_STATE_CHANGE`结构来生成用户层使用的`DEBUG_EVENT`结构，由于这两个结构都在用户层，并不需要进入内核。所以转换过程直接就在`ntdll.dll`中的`DbgUiConvertStateChangeStructure`中完成。

## 3.回复调试事件

`ContinueDebugEvent`函数被用来向调试子系统回复调试器处理结果，该函数通过调用`ntdll`中的`DbgUiContinue`来实现

```
.text:7C85B53D ; BOOL __stdcall ContinueDebugEvent(DWORD dwProcessId, DWORD dwThreadId, DWORD dwContinueStatus)
.text:7C85B53D                 public _ContinueDebugEvent@12
.text:7C85B53D _ContinueDebugEvent@12 proc near        ; DATA XREF: .text:off_7C802654↑o
.text:7C85B53D  
.text:7C85B53D ClientId = _CLIENT_ID ptr -8
.text:7C85B53D dwProcessId = dword ptr 8
.text:7C85B53D dwThreadId = dword ptr 0Ch
.text:7C85B53D dwContinueStatus = dword ptr 10h
.text:7C85B53D  
.text:7C85B53D                 mov     edi, edi
.text:7C85B53F                 push    ebp
.text:7C85B540                 mov     ebp, esp
.text:7C85B542                 push    ecx
.text:7C85B543                 push    ecx
.text:7C85B544                 push    esi
.text:7C85B545                 mov     esi, [ebp+dwProcessId]
.text:7C85B548                 push    edi
.text:7C85B549                 push    [ebp+dwContinueStatus] ; ContinueStatus
.text:7C85B54C                 mov     edi, [ebp+dwThreadId]
.text:7C85B54F                 lea     eax, [ebp+ClientId]
.text:7C85B552                 push    eax             ; ClientId
.text:7C85B553                 mov     [ebp+ClientId.UniqueProcess], esi
.text:7C85B556                 mov     [ebp+ClientId.UniqueThread], edi
.text:7C85B559                 call    _DbgUiContinue@8 ; DbgUiContinue(x,x)
.text:7C85B55E                 test    eax, eax
.text:7C85B560                 jge     short loc_7C85B56C
```

而`DbgUiContinue`则是通过调用内核函数`ZwDebugContinue`来实现

```
.text:7C96FF9C                 public DbgUiContinue
.text:7C96FF9C DbgUiContinue   proc near               ; DATA XREF: .text:off_7C923428↑o
.text:7C96FF9C  
.text:7C96FF9C arg_0 = dword ptr 8
.text:7C96FF9C arg_4 = dword ptr 0Ch
.text:7C96FF9C  
.text:7C96FF9C                 mov     edi, edi
.text:7C96FF9E                 push    ebp
.text:7C96FF9F                 mov     ebp, esp
.text:7C96FFA1                 mov     eax, large fs:18h ; 获取TEB地址
.text:7C96FFA7                 push    [ebp+arg_4]
.text:7C96FFAA                 push    [ebp+arg_0]
.text:7C96FFAD                 push    dword ptr [eax+0F24h] ; 将调试对象句柄入栈
.text:7C96FFB3                 call    ZwDebugContinue
.text:7C96FFB8                 pop     ebp
.text:7C96FFB9                 retn 8
.text:7C96FFB9 DbgUiContinue   endp
```

在内核函数中，会根据`ContinueStatus`来选择相应的操作

```
PAGE:0058C5CC                 mov     eax, [ebp+ContinueStatus]
PAGE:0058C5CF                 cmp     eax, DBG_EXCEPTION_NOT_HANDLED
PAGE:0058C5D4                 jz      short loc_58C5FC
PAGE:0058C5D6                 cmp     eax, 10000h
PAGE:0058C5DB                 jle     short loc_58C5F2
PAGE:0058C5DD                 cmp     eax, DBG_CONTINUE
PAGE:0058C5E2                 jle     short loc_58C5FC
PAGE:0058C5E4                 cmp     eax, DBG_UNABLE_TO_PROVIDE_HANDLE
PAGE:0058C5E9                 jle     short loc_58C5F2
PAGE:0058C5EB                 cmp     eax, DBG_TERMINATE_PROCESS
PAGE:0058C5F0                 jle     short loc_58C5FC
PAGE:0058C5F2 
PAGE:0058C5F2 loc_58C5F2:                             ; CODE XREF: NtDebugContinue(x,x,x)+50↑j
PAGE:0058C5F2                                         ; NtDebugContinue(x,x,x)+5E↑j
PAGE:0058C5F2                 mov     eax, STATUS_INVALID_PARAMETER
PAGE:0058C5F7                 jmp     loc_58C6D2
```

如果`ContinueStatus`是`DBG_CONTINUE`，则会调用函数获取调试对象
