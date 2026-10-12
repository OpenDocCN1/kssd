
# 自己动手给被艾尼(ANI)病毒感染的 EXE 除毒

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-603.htm`](https://www.pediy.com/kssd/pediy09/pediy09-603.htm)

> 上个月正是艾尼病毒的高发期，`ANI` 病毒利用了 `MS` 系统的高危漏洞进行传播，而我的系统一直是用精简版的，而且自动更新从来都是关着的，更要命的还是裸奔！

> 终于有一天（4 月 16 号）,被这病毒感染了，但当时不知道他会感染 `EXE`，重装系统后，昨天终于发现我多年收集的软件盘（`60G`）中的 `EXE` 全被感染了，而我新装的 `NOD32` 在发现病毒后只是一味的把 `EXE` 给隔离起来，在杀毒时也只能删除不能清除，晕，多年来的心血，我 `N` 多的工具、程序、电子书都在这里呢！可不能被他这么给糟蹋了！

> 在用 `OD` 试了一下程序后，并用 `16` 进制查看工具看了一下被感染的 `EXE`，发现病毒只是很简单的把原文件放到他自己后面，并且在文件最后加了 `8` 字节，前 `4` 字节指向的是源可执行文件在总文件的偏移，后 `4` 字节是感染日期，我也是从这里才知道是我 4 月 16 号全部感染的。

> 我还以为病毒会把可执行文件压缩成资源或更改 `OEP` 等方法，那样的话可能还得用到动态解压、调试 `API` 等麻烦许多，我用我自己写的文件切割工具把文件搞出来再后正常文件一比，丝毫不差！既然现在源文件全部还在被感染的文件中并且没有丝毫损坏，那就方便了，自己动手写一个，全部复制出来就 `OK` 了。

> 因为我用 `winmasm` 多年了，所以习惯小工具都用汇编写，从 2002 年至今，开发一直都用 `RadASM`，如果有需要的朋友可以用这个直接编译

程序主要涉及到内容：

多线程、公共对话框、文件操作、目录遍历、文件映射等

清除方法：

可以选择是清除目录还是单个文件，程序自动判断，在遍历目录时只查找 `exe`，因为这病毒只把我的 `exe` 感染了，直接映射文件，并取倒数第 `4` 个字节，再在文件中查看以这个偏移的内容是不是一个真正的可执行文件头，是的话，直接把中间内容复制出来，并把病毒文件重命名为一个特殊的扩展名，这样，在完成后，我用 `TC` 很容易就把病毒全找出来（不删除病毒以防万一）

> 结果花了半天，折腾出了这东东，程序很简单，并没有什么高深之处，可以给初学汇编的人做个参考，其中也可能有少许没有考虑全的情况，如磁盘空间不足等，因为对我来说，没必要，我的磁盘空间大着呢，不可能出个种情况。

代码:

```asm
//关键代码：分离出源文件
invoke FindFirstFile,pszFileName,addr fn
.if eax!=INVALID_HANDLE_VALUE
  invoke CreateFile,pszFileName,GENERIC_READ,0,NULL,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,NULL
  .if eax!=INVALID_HANDLE_VALUE
    mov hfile,eax
    invoke CreateFileMapping,eax,NULL,PAGE_READONLY,0,0,NULL
    .if eax
      mov hmap,eax
      invoke MapViewOfFile,eax,FILE_MAP_READ,0,0,0
      .if eax
        mov pview,eax
        ;///////////////////////////////////
        add eax,fn.nFileSizeLow
        sub eax,8
        mov eax,[eax]
        push fn.nFileSizeLow
        pop Len
        .if eax && eax<Len
          sub Len,eax
          sub Len,8
          mov ebx,pview
          add ebx,eax
          .if byte ptr [ebx] == "M" && byte ptr [ebx + 1] == "Z"
            invoke ListAddItem,pszFileName
            invoke lstrcpy,addr pszNewFile,pszFileName
            invoke lstrcat,addr pszNewFile,offset g_pszNewFileExt
            invoke CreateFile,addr pszNewFile,GENERIC_WRITE,0,NULL,CREATE_NEW,FILE_ATTRIBUTE_NORMAL,NULL
            .if eax==INVALID_HANDLE_VALUE
              invoke MessageBox,0,offset g_pszFileExist,offset g_pszError,MB_OK
              invoke CreateFile,pszNewFile,GENERIC_WRITE,0,NULL,CREATE_ALWAYS,FILE_ATTRIBUTE_NORMAL,NULL
            .endif
            
            .if eax!=INVALID_HANDLE_VALUE
              mov hnewfile,eax
              lea ecx,wLen
              invoke WriteFile,eax,ebx,Len,ecx,0
              mov eax,wLen
              .if Len==eax
                mov bOK,1
              .endif
              invoke CloseHandle,hnewfile
            .endif
          .endif
        .endif
      .endif
      ;    .if eax==20070416h
      ;    .endif
        ;/////////////////////////////////////////////////////////////////////////////////////////
        invoke UnmapViewOfFile,pview
      .endif
      invoke CloseHandle,hmap
    .endif
    invoke CloseHandle,hfile
  .endif
.endif
.if bOK
  invoke lstrcpy,addr pszVirusFile,pszFileName
  invoke lstrcat,addr pszVirusFile,offset g_pszVirusFileExt
  invoke MoveFile,pszFileName,addr pszVirusFile
  invoke MoveFile,addr pszNewFile,pszFileName
.endif
```

代码:

```asm
//文件与文件夹遍历
invoke GetFileAttributes,pszFileName
.if eax!=-1
  .if eax & FILE_ATTRIBUTE_DIRECTORY
    invoke lstrcpy,addr pszFind,pszFileName
    invoke lstrlen,addr pszFind
    lea ecx,pszFind
    add ecx,eax
    dec ecx
    .if byte ptr[ecx] != "\"
      inc ecx
      mov byte ptr [ecx],"\""
      mov byte ptr [ecx+1],0
    .endif
    invoke lstrcpy,addr pszPath,addr pszFind
    invoke lstrcat,addr pszFind,addr g_pszFindExt
  ;  invoke RtlZeroMemory,addr fn,sizeof WIN32_FIND_DATA
    invoke FindFirstFile,addr pszFind,addr fn
    mov FHandle,eax
    .if eax!=INVALID_HANDLE_VALUE
      .WHILE EAX
        test fn.dwFileAttributes,FILE_ATTRIBUTE_DIRECTORY 
        .if ZERO? && fn.nFileSizeLow > 512
          invoke lstrcpy,addr pszFind,addr pszPath
          invoke lstrcat,addr pszFind,addr fn.cFileName
          invoke KillFileVirus,addr pszFind
        .endif
        .if g_State==S_Stop
          jmp FindOut
        .else
          .while g_State==S_Pause
            invoke Sleep,1
          .endw
        
        .endif
        invoke FindNextFile,FHandle,addr fn
      .endw
    .endif
    invoke FindClose,FHandle
    
    invoke lstrcpy,addr pszFind,addr pszPath
    invoke lstrcat,addr pszFind,addr g_pszFindAll
    invoke FindFirstFile,addr pszFind,addr fn
    mov FHandle,eax
    .if eax!=INVALID_HANDLE_VALUE
      .WHILE EAX
        test fn.dwFileAttributes,FILE_ATTRIBUTE_DIRECTORY 
        .if !ZERO?
          invoke lstrcmp,addr fn.cFileName,addr g_pszFolder1
          .if eax
            invoke lstrcmp,addr fn.cFileName,addr g_pszFolder2
            .if eax
              invoke lstrcpy,addr pszFind,addr pszPath
              invoke lstrcat,addr pszFind,addr fn.cFileName
              invoke KillVirus,addr pszFind
            .endif
          .endif
        .endif
        .if g_State==S_Stop
          jmp FindOut
        .else
          .while g_State==S_Pause
            invoke Sleep,1
          .endw
        
        .endif
        invoke FindNextFile,FHandle,addr fn
      .endw
    .endif   
FindOut:  invoke FindClose,FHandle
  .else
    invoke KillFileVirus,pszFileName
  .endif
.endif
```

`4dd466d0e1e0cc123afbe504da2462ca`
