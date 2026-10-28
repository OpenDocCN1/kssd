
```asm
invoke lstrlen,offset offset path1                         ;本段遍历磁盘文件，不知写的有没有问题，请指教

lea esi,path1
add esi,eax
.if byte ptr [esi-1]!="\"
        mov byte ptr [esi],"\"
        mov byte ptr [esi+1],0
.endif
invoke RtlZeroMemory,offset path2,sizeof path2
invoke lstrcpy,offset path2,offset path1
invoke lstrcat,offset path2,offset szFilter
invoke FindFirstFile,offset path2,addr @stFindData
.if eax!=INVALID\_HANDLE\_VALUE
        mov @hFind,eax
        .repeat
                invoke RtlZeroMemory,offset path3,sizeof path3
                invoke lstrcpy,offset path3,offset path1
                invoke lstrcat,offset path3,addr @stFindData.cFileName
                .if @stFindData.dwFileAttributes & FILE\_ATTRIBUTE\_DIRECTORY
                        .if byte ptr [@stFindData.cFileName]!="."
                                invoke \_Scan,offset path3
                                lea esi,path1
                                invoke lstrlen,offset path1
                                add esi,eax
                                invoke lstrlen,addr @stFindData.cFileName
                                sub esi,eax
                                mov byte ptr [esi-1],0
                        .endif
                .else
                        invoke \_unLock,offset path3
                .endif
                invoke FindNextFile,@hFind,addr @stFindData
        .until eax==FALSE
        invoke FindClose,@hFind
.endif
popad
ret
\_Scan endp

\_rePair proc
        LOCAL @szBuffer[108]:byte  ;用于存放磁盘盘符
        invoke RtlZeroMemory,addr @szBuffer,sizeof @szBuffer  ;缓冲区清0
        invoke GetLogicalDriveStrings,sizeof @szBuffer,addr @szBuffer ;有多少磁盘全部取出放入缓冲区
        lea esi,@szBuffer
        .repeat
                push esi
                invoke \_Scan,esi
                pop esi
                add esi,4
        .until dword ptr [esi]==0
        invoke wsprintf,offset szOut,offset szFmt,offset szFinish
        invoke SendMessage,hEdit,EM\_REPLACESEL,FALSE,offset szOut
        ret
\_rePair endp

\_DlgProc proc uses ebx esi edi hWnd,uMsg,wParam,lParam
        LOCAL @stRect:RECT
        mov eax,uMsg
        .if eax==WM\_CLOSE
                invoke EndDialog,hWinMain,NULL
        .elseif eax==WM\_SIZE
                invoke GetClientRect,hWinMain,addr @stRect
                mov esi,@stRect.right
                sub esi,@stRect.left
                mov edi,@stRect.bottom
                sub edi,@stRect.top
                sub edi,60
                mov ebx,edi
                shr edi,1
                sub edi,5
                invoke MoveWindow,hEdit,0,0,esi,edi,TRUE
                add edi,0ah
                invoke MoveWindow,hEdit2,0,edi,esi,160,TRUE
        .elseif eax==WM\_INITDIALOG
                push hWnd
                pop hWinMain
                invoke LoadIcon,hInstance,ICO\_1
                INVOKE SendMessage,hWinMain,WM\_SETICON,ICON\_BIG ,eax
                invoke  CreateWindowEx,WS\_EX\_CLIENTEDGE,offset szClassEdit,NULL,WS\_CHILD OR WS\_VISIBLE OR WS\_VSCROLL OR WS\_HSCROLL OR ES\_MULTILINE or ES\_NOHIDESEL,0,0,0,0,hWinMain,0,hInstance,NULL
                mov hEdit,eax
                invoke  CreateWindowEx,WS\_EX\_CLIENTEDGE,offset szClassEdit,NULL,WS\_CHILD OR WS\_VISIBLE OR WS\_VSCROLL OR WS\_HSCROLL OR ES\_MULTILINE or ES\_NOHIDESEL,0,0,0,0,hWinMain,0,hInstance,NULL
                mov hEdit2,eax
                invoke SendMessage,hEdit,WM\_SETTEXT,0,offset szWait
        .elseif eax==WM\_COMMAND
                mov eax,wParam
                .if ax==IDC\_BTN1
                        call \_rePair
                .endif
        .else
                mov eax,FALSE
                ret
        .endif
        mov eax,TRUE
        ret
\_DlgProc endp

start:
invoke LoadLibrary,offset szDllEdit
mov hDll,eax
invoke GetModuleHandle,NULL
mov hInstance,eax
invoke DialogBoxParam,hInstance,DLG\_MAIN,NULL,\_DlgProc,NULL
invoke FreeLibrary,hDll
invoke ExitProcess,NULL
end start
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [标准pe格式带毒样本.rar](javascript:void(0);)

  （39.50kb，48次下载）

- [非标准pe格式带毒样本.rar](javascript:void(0);)

  （31.79kb，48次下载）

- [修复.rar](javascript:void(0);)

  （6.37kb，53次下载）
