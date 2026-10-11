
# 在 MFC 程序中添加功能按钮

> 原文：[`www.pediy.com/kssd/pediy06/pediy6035.htm`](https://www.pediy.com/kssd/pediy06/pediy6035.htm)

## 步骤详解

### 1. 检查保护壳
先检查目标程序是不是有保护壳，如果有就先进行脱壳处理。

### 2. 修改资源
脱壳后修改目标程序的资源，在对话框中添加一个自己的按钮，并赋予一个新的控件 ID。可以使用 `ResHacker` 或者 `VC++` 对资源进行修改；这样就有一个我们自己的功能按钮了，接下来要添加一个消息映射和响应函数。

### 3. 寻找断点位置
通常我们会在一个已经存在的对话框上添加一个按钮，因此我们先摸索一下原来对话框的一些功能按钮，最好有 `MessageBox` 调用的函数，这样方便设断点。

### 4. 定位消息匹配函数
根据对 MFC 的代码，知道 MFC 程序有个消息响应函数 `CCmdTarget::OnCmdMsg()`，继而定位到消息查找匹配函数 `_AfxFindMessageEntry` 被调用的地方。

### 5. 设置条件断点
假如找到一个功能按钮并能弹出 `MessageBox` 的。在按下该按钮前我们先在 `_AfxFindMessageEntry` 被调用的地方设上断点，然后按下按钮。可能会有很多消息进入断点，因此我们可以设置一个条件断点，只拦截 `WM_COMMAND` (0x111h) 消息。

### 6. 分析汇编代码
如果一切顺利的话，我们已经在断点处拦截到功能按钮的调用了。下面是 `CCmdTarget::OnCmdMsg()` 的代码片段：

```asm
CCmdTarget::OnCmdMsg()

PUSH    EBP
MOV     EBP,ESP
MOV     EAX,DWORD PTR [EBP+C]
PUSH    EBX
PUSH    ESI                   ;  Sample.00402308
PUSH    EDI
CMP     EAX,-2                ;  Switch (cases FFFFFFFD..FFFFFFFF)
MOV     EDI,ECX
JE      6BC9E571              ;  MFC42.6BC9E571
CMP     EAX,-3
JE      6BC9E592              ;  MFC42.6BC9E592
CMP     EAX,-1
JNZ     SHORT 6BC42287        ;  MFC42.6BC42287
MOV     EBX,111               ;  Case FFFFFFFF of switch 6BC42245
MOV     EAX,DWORD PTR [EDI]   ;  Sample.00402310
MOV     ECX,EDI
CALL    NEAR DWORD PTR [EAX+30] ;  MFC42.GetMessageMap()
MOV     ESI,EAX               ;  Sample.00402308
TEST    ESI,ESI               ;  Sample.00402308
JE      SHORT 6BC422B0        ;  MFC42.6BC422B0
PUSH    DWORD PTR [EBP+8]     ; /Arg4 = 000003E8
PUSH    DWORD PTR [EBP+C]     ; |Arg3 = 00000000
PUSH    EBX                   ; |Arg2 = 00000111
PUSH    DWORD PTR [ESI+4]     ; |ESI + 4 == lpEntries, You Can Modify it to Chang MsgMap
CALL    6BC42088              ; #1145_AfxFindMessageEntry <----- 断点停在这里
TEST    EAX,EAX               ;  Sample.00402308
```

### 7. 数据结构介绍
现在有必要介绍一下比较重要的两个数据结构。

第一个是“消息映射结构”：

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

在这个结构中，我们只关心 `lpEntries` 这个值，因为这个值指向“消息处理映射入口”数组的首地址。

第二个是“消息处理映射入口”数据结构：

```c
AFX_MSGMAP_ENTRY
{
  nMessage;
  nCode    ;
  nID  ;
  nLastID ;
  nSig   ;
  lpEntry ;
}
```

### 8. 源代码分析
查看一下 MFC 关于调用 `_AfxFindMessageEntry` 的源代码。这个函数作用是，轮询查找和 `nMsg`、`nCode`、`nID` 匹配的 `AFX_MSGMAP_ENTRY` 结构，并从中提取出处理函数的入口地址。MS 为了提高效能，所以这个函数是用汇编写程序的。

```c
AFX_MSGMAP *pMessageMap;
pMessageMap = GetMessageMap()
lpEntry = _AfxFindMessageEntry(pMessageMap->lpEntries, nMsg, nCode, nID);
```

我们可以发现 `pMessageMap` 指向我们需要的 `AFX_MSGMAP` 数据结构，因此在汇编代码中“ESI + 4”就是指向 `lpEntries` 的地址。

### 9. 修改数组入口
我们现在所要做的就是，找到数据段空白的地方，然后将原来的“消息处理映射入口”数组复制到这里来，然后用新的数组入口地址改写“ESI + 4”所指向的数据。（因为没有空间，所以不在原来的数组后面添加）

### 10. 添加新入口
现在我们可以在新的数组中添加一个我们自己的“消息处理映射入口”结构了。下面是处理按钮消息的例子：

```c
AFX_MSGMAP_ENTRY
{
  nMessage = 0x111 WM_COMMAND
  nCode    = 0
  nID    = (ID) 按钮控件的 ID
  nLastID  = (ID) 按钮控件的 ID
  nSig     = 0xC
  lpEntry (消息响应函数入口)
}
```

### 11. 写入响应函数
在代码段的空白地方写入自己的消息响应函数，然后把这个函数的入口地址写入到我们自己构造的消息映射入口结构的成员 `lpEntry` 中。

### 12. 总结
基本上大致的流程就是这样了。
