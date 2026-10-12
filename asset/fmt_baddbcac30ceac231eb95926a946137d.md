
# Telock 脱壳程序代码与说明

```asm
invoke RtlMoveMemory, edi, eax, dword ptr[esi].SizeOfRawData
add esi, sizeof IMAGE_SECTION_HEADER ; 定位到下个区块表
pop ecx
.endw
assume esi:nothing
invoke VirtualFree, pTempMem, 0, MEM_RELEASE

;---------------------------------------------------
; 创建一个文件名:原文件名(OK).exe
invoke lstrcpy, offset szDumpedFileName, offset szfilename
invoke lstrlen, offset szDumpedFileName
lea esi, szDumpedFileName
add esi, eax
sub esi, 4
mov eax, dword ptr[esi]
mov dword ptr [esi], "KO("
add esi, 4
mov dword ptr [esi], eax
add esi, 4
and byte ptr[esi], 0

;---------------------------------------------------
invoke CreateFile, ADDR szDumpedFileName, GENERIC_READ+GENERIC_WRITE, FILE_SHARE_READ+FILE_SHARE_WRITE, 0, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL
mov edi, eax
invoke WriteFile, edi, pDstMem, dwFileSize, addr dwBytesRW, NULL
invoke CloseHandle, edi
invoke VirtualFree, pDstMem, 0, MEM_RELEASE
invoke _AddLine, addr szUnPackedOK
invoke GetExitCodeProcess, hProcess, addr ExitCode
invoke TerminateProcess, hProcess, ExitCode
ret
.else
push DBG_EXCEPTION_NOT_HANDLED
.endif
inc DebugStep ; 增加异常计数
; 继续调试
push DebugEvent.dwThreadId
push DebugEvent.dwProcessId
call ContinueDebugEvent
.else ; 其他消息
invoke ContinueDebugEvent, DebugEvent.dwProcessId, DebugEvent.dwThreadId, DBG_CONTINUE
.endif
.endw
; 循环体结束
; ================================================================================
ret

_UnPackFileForTelock endp

;------------------------------------------------------------
DlgProc proc hWin:HWND, uMsg:UINT, wParam:WPARAM, lParam:LPARAM
mov eax, uMsg
.if eax == WM_INITDIALOG
push hWin
pop hWinMain
invoke GetDlgItem, hWin, IDC_EDT_INFO
mov hEditInfo, eax
invoke SendMessage, hEditInfo, WM_CLEAR, 0, 0
.elseif eax == WM_COMMAND
mov eax, wParam
.if ax == IDC_BTN_SEL_FILE ; 打开并加载 PE 文件
invoke _OpenFile, addr szfilename, offset szfilefilter, offset szfileext
invoke SendDlgItemMessage, hWin, IDC_EDT_FILENAME, WM_SETTEXT, MAX_PATH, addr szfilename
invoke SendMessage, hEditInfo, WM_CLEAR, 0, 0
invoke _UnPackFileForTelock, addr szfilename
.elseif ax == IDC_ABOUTBTN ; 关于 dlg
invoke DialogBoxParam, hInstance, IDD_ABOUTBOX, hWin, offset _ProcDlgAbout, NULL
.endif
.elseif eax == WM_CLOSE
invoke EndDialog, hWin, 0
.else
mov eax, FALSE
ret
.endif
mov eax, TRUE
ret
DlgProc endp

;===========================================================================
start:
invoke GetModuleHandle, NULL
mov hInstance, eax
invoke InitCommonControls
invoke DialogBoxParam, hInstance, IDD_DIALOG_MAIN, NULL, addr DlgProc, NULL
invoke ExitProcess, 0
end start
```

## 程序逻辑说明

程序开始创建一个对话框，在对话框回调函数里重要的是处理 `[打开文件]` 按钮的单击事件。子过程 `_OpenFile` 负责打开文件，`_UnPackFileForTelock` 是核心的函数，负责对打开的文件脱壳处理。`_SetBpx` 与 `_ClsBpx` 分别是设置断点和取消断点的，`_AddLine` 将信息添加到文本框里显示。下面主要看 `UnPackFileForTelock` 函数。

1.  打开文件，创建内存映像
2.  检测文件是否是 PE 文件
3.  获取文件基本信息，如区块数，映像大小，OEP 等。
4.  以 `DEBUG_PROCESS+DEBUG_ONLY_THIS_PROCESS` 标志创建调试进程，进入调试循环体
5.  调试事件的 `dwDebugEventCode` 为 `CREATE_PROCESS_DEBUG_EVENT`，也即调试进程刚被创建，保存相关信息，如进程 ID，映像基址，进程句柄，主线程 ID。然后初始化单步异常次数计数器 `DebugStep` 为 0。
6.  主要处理调试事件的 `dwDebugEventCode` 为 `EXCEPTION_DEBUG_EVENT` 的情况，根据上表的分析，应该关注 `DebugStep` 为 3, 15, 17, 18 (18 号是我们认为设定的断点), 21 时的情况。因为 3 号与 15 号异常地址固定，可以作为判断 `tElock` 是否为 0.98 版本的依据；原 18 号异常地址固定，但在此之前的地址 `40D21CH` 处要破坏输入表，所以要在 17 号异常时要在 `40D21CH` 处设断，到 18 号异常时(所以说这时 18 号异常就是我们人为设定的了)取消断点。由于我们多设了一个断点，因此程序里的异常次数要多 1，我们在最后一个异常处(21 号)可以 DUMP 程序了。因此便是：3, 15, 17, 18, 21。这一点我搞了好久才弄清。
7.  3, 15, 17, 18 号处的代码处理情况就不详细说了，主要是 21 号时 DUMP 程序的处理，比较繁琐。因为 PE 文件的 `FileAlignment` 是 `200h`，而在内存里的 `SectionAlignment` 是 `1000h`，因此如果将处于内存里的程序 DUMP 出来并保存为文件的话，必须要考虑对齐粒度的转换，否则 DUMP 出的文件会较大，因为区块之间会存在较大的空隙。因此我想到要“重新组织一下文件结构”，这一点书中也没有讲，我感觉还是有必要的。思路是这样：“通常区块是连续存放的，上个区块的 `[文件偏移]` 加上上个区块的 `[文件大小]` 就是下个区块的 `[文件偏移]`。而第一个区块的 `[文件偏移]` 便是区块表后按 `200h` 对齐后的大小。”，按照这个方法 DUMP 出的程序就比较紧凑了：57KB，而附书程序 DUMP 出的是 60KB。

至此，代码讲解完毕，其实我也是得益于《软件加密技术内幕》一书，只不过将自己的心得写出来而已，程序写得不完善之处，还请指正啊。永远支持看雪！

另外我是一个新兵，本来想发在“外壳技术”板块呢，结果：

`帖子数不到 20 的会员，『软件调试论坛』与『软件保护与分析』 不能发主题帖，但可跟帖（如有精华帖则无此限制）。你可以将问题发到『新兵论坛』，一些简单入门问题，都可在这版块交流。如果问题有深度，版主会将帖子转到相应版块。O(∩_∩)o…`

By Sing
`asmcvc@163.com`
2008-03-17
