
# 在 PE 中添加,删除 SECTION

> 原文：[`www.pediy.com/kssd/pediy10/103092.html`](https://www.pediy.com/kssd/pediy10/103092.html)

在 PE 中添加,删除 SECTION

这两天空闲，自己写了个添加，删除 SECTION 的程序，发现里面的技巧很多，需要注意的地方也很多，现把心得写上，希望能给正在学习 PE 文件的朋友提供方便。

关于 PE 的格式，PE 的基本知识，请参看其他相关文章。

## 添加 SECTION

添加 SECTION 的前提是在最后一个 SECTION 与第一个区块之间有足够的空间)添加一个新 SECTION 40 字节。通常情况下，是有 40 字节的空间的，但是如果没有就需要想办法腾出 40 字节空间来。

通常有 2 种方法腾空间

### 重新编排 PE 头

通常 `IMAGE_DOS_HEADER` 和 `IMAGE_NT_HEADER` 不是紧挨着，中间有一些其他数据，比如 DOS 占位程序，这些数据基本上没用，可以将 NT 头以及 SECTION 数据前移覆盖这些数据达到扩充 PE 头空余空间添加新 SECTION。

`IMAGE_DOS_HEADER` 的 `e_flanew` 指向 NT 头相对于文件头的偏移，将 NT 头前移，只需修改 `e_flanew` 的值即可。

优点：相对于扩充 PE 头，不需要修改所有 SECTION 的 `PointerToRawData` 指向，不用移动所有区块。

缺点：腾出来的尺寸只能加入有限的新 SECTION.

### 扩充 PE 头

这种方法通常将 `SizeOfHeader` 的尺寸扩大文件对齐字节，0x200 字节，然后把所有 SECTION 的 `PointerToRawData` 值得加上 0x200 字节，然后再将所有区块数据向后移动 0x200 字节。当然 `SizeOfHeader` 也需要加 0x200 字节。

优点:使用这种方法可以实现无限的添加新 SECTION。

缺点:需要改动的地方比较多。

### 注意绑定输入表

有了空间不一定就能添加新 SECTION,还需要查看绑定输入表,绑定输入表和其他节表不同，绑定输入表紧挨着最后的一个 SECTION 存放,绑定输入表而且必须在 PE 头区内，由 `IMAGE_DATA_DIRECTORY[11]` 来指向。

我手上的 PE 编辑工具,LoardPE,PETools,StudPE 都忽视了这个问题，添加 SECTION 后都覆盖了绑定输入表数据，而且也没有 `IMAGE_DATA_DIRECTORY[11]`，导致添加节区后 PE 文件不能运行。

如果不想保留绑定输入，那么添加 SECTION 的时候直接把 `IMAGE_DATA_DIRECTORY[11]` 内的数据置 0.

如果想保留绑定输入表，那么就先把绑定数据拷贝出来，添加完 SECTION 再把数据复制到这个 SECTION 的后面，然后修改 `IMAGE_DATA_DIRECTORY[11]` 的指向就可以。

### `IMAGE_SECTION_HEADER` 结构内容

`Name` 新区块的名字，注意不能超过 8 字节大小。

`VirtualAddress` 虚拟地址；这个地址就是 PE 文件装载后，这个区块所在的内存地址。地址必须是"内存页对齐"，必须紧挨着上一个区块。计算公式就是上一个区 块的 `VirtualAddress` + `VirtualSize` 内存页对齐后的值。

`VirtualSize` 区块内数据的实际大小；通过数据的实际大小，就可以计算出文件对齐后的数据大小。

`SizeOfRawData` 区块数据文件对齐后的数据大小。根据 `VirtualSize` 计算可计算出文件对齐后的尺寸。

`PointerToRawData` 区块相对于文件头的文件偏移, 在文件中两区块数据之间不必紧挨着，但起始地址必须是"文件对齐"的。

`Characteristics` 区块的属性,可根据添加区块的实际目的设置属性，拿不准就设置 E0000060 代表可读，可写，可执行，包含代码，包含初始化数据。

添加 SECTION 后还需要添加这个 SCTION 的区块数据，首先需要把文件尺寸扩大，扩大的尺寸就是区块数据文件对齐后的大小。

最后将 `IMAGE_OPTIONAL_HEADER32` 的 `SizeOfImage` 加上新区块数据的 `VirtualAddress`。

将 `IMAGE_FILE_HEADER` 的 `NumberOfSection` 加 1。

## 删除 SECTION

删除一个 SECTION，需要考虑的比较多；

删除 SECTION 不能根据名称删除，应该根据序号删除，因为允许存在同名的 Section。

如果删除的 Section 位于中间位置，后面 Section 的 `PointerToRawData` 文件偏移可以向上移动，但是 `VirtualAddress` 虚拟地址就不能简单的向上移动了,因为如果某个节区的 VA 发生变化，那么这个节区里的所有 RVA 也会发生变化，而要修复这些 RVA 工作量就太大了。处理办法就是，把要删除的这个 Section 的上一个 Section 的 `VirtualSize` 扩大，也就是加上要删除的这个 Section 的 `VirtualSize`,这样，后面的所有 Section 的 `VirtualAddress` 不变，只修正 `PointerToRawData` 就可以。当然 `SizeOfImage` 也就不用调整了，PE 映像大小没变，只改变了 PE 文件大小。

如果删除的是最后一个 Section，那么删除后将 `SizeOfImage` 减小就可以，不存在 VA 问题。

如果删除的 Section 和 `IMAGE_NT_HEADER` 的 `DATA_DIRECTORY` 有关，比如重定位表，TLS 表,那么必须要清除数据目录的 `VirtualAddress`,和 `VirtualSize`。

最后，如果有绑定数据，需要在文件中上移一个 `IMAGE_SECTION_HEADER` 尺寸，然后修正数据目录表的指向。

附上一段小代码，一个完整的添加删除 SECTION 例子:

代码:
```



```asm
.686
.model flat,stdcall
option casemap:none

include     windows.inc
include     kernel32.inc
includelib  kernel32.lib
include     user32.inc
includelib  user32.lib
include    comdlg32.inc
includelib  comdlg32.lib
include     Shell32.inc
includelib  Shell32.lib

IDD_DIALOG         equ                100
IDC_RICHEDIT       equ                1006
IDC_EDIT_NAME      equ               1001
ID_OK              equ                1008
ID_DEL             equ             1009         
IDC_EDIT_SIZE     equ             1010

_OpenFile          proto       

    .data
hInstancd      dd ?
hWinMain       dd ?
hRichHandle    dd ?
lpMem          dd ?
dwFileSize     dd ?
hMapFile       dd ?
hFile          dd ?
szFileName     db MAX_PATH dup(?)


    .const
szMsg          db '节名称，取前 8 个字符',0
szMsg2         db  '添加时填入节尺寸，删除时填入节编号',0
szDllName      db  'RichEd20.dll',0
szFilter       db 'PE Files',0,'*.exe;*.dll;*.scr;*.fon;*.drv',0
szErrOpen      db '打开文件失败',0
szErrMap       db '创建文件映射失败',0
szErrView      db  '创建文件视图失败',0
szErrPE        db  '不是 PE 文件',01
szFmtSection   db  '%-4d  %-8s  %08X  %08X  %08X  %08X  %08X',0dh,0ah,0
szMsgSection   db  '----------------------------------------------------------------',0dh,0ah
           db  '序号  节区名称  节区大小  虚拟地址  Raw_ 尺寸  Raw_ 偏移  节区属性',0dh,0ah
           db  '----------------------------------------------------------------',0dh,0ah,0

    .code
   
_AppendInfo  proc  _lpsz
    local  @stCR:CHARRANGE
    pushad
    invoke  GetWindowTextLength,hRichHandle
    mov  @stCR.cpMin,eax
    mov  @stCR.cpMax,eax
    invoke  SendMessage,hRichHandle,EM_EXSETSEL,0,addr @stCR
    invoke  SendMessage,hRichHandle,EM_REPLACESEL,FALSE,_lpsz
    popad
    ret

_AppendInfo  endp

;计算对齐
_Align proc uses edx dwSize,dwAlign
  mov eax,dwSize
  xor edx,edx
  div dwAlign
  test edx,edx   
  jz @f   
  inc eax
@@:
  mul dwAlign
  ret
_Align endp

;删除 Section
;参数:dwNo Section 序号
_DelSection proc dwNo
  local @dwDataSize,@dwSNum,@dwWriteSize,@dwMemAlign
  local @hMap,@lpMem
  mov esi,lpMem
  add esi,[esi+3CH]
  assume esi:ptr IMAGE_NT_HEADERS
  push [esi].OptionalHeader.SectionAlignment
  pop @dwMemAlign
;Section 序号是否正确
  movzx ecx,[esi].FileHeader.NumberOfSections
  dec ecx
  mov @dwSNum,ecx
  cmp ecx,dwNo
  jb Err   
  invoke CreateFileMapping,INVALID_HANDLE_VALUE,NULL,PAGE_READWRITE,0,dwFileSize,NULL
  cmp eax,NULL
  je Err
  mov @hMap,eax
  invoke MapViewOfFile,@hMap,FILE_MAP_WRITE,0,0,0
  cmp eax,NULL
  je Err
;拷贝文件头
  mov @lpMem,eax
  add si,[esi].FileHeader.SizeOfOptionalHeader
  add esi,sizeof IMAGE_FILE_HEADER
  add esi,4
  mov ecx,esi
  sub ecx,lpMem
  mov esi,lpMem
  mov edi,@lpMem
  rep movsb
  assume esi:ptr IMAGE_SECTION_HEADER
  assume edi:ptr IMAGE_SECTION_HEADER
  xor edx,edx
  mov @dwDataSize,edx
  mov @dwWriteSize,edx  
;循环拷贝 Section,和 Section 节数据 要删除的跳过。
loopsec:
  cmp edx,dwNo
  je delsec
;拷贝 Section
  mov ecx,sizeof IMAGE_SECTION_HEADER   
  rep movsb
;修正 Section PointerOfRawData
  mov eax,@dwDataSize
  sub [edi-40].PointerToRawData,eax
;拷贝节区数据
  push esi
  push edi
  mov eax,[esi-40].PointerToRawData
  mov ecx,[esi-40].SizeOfRawData
  mov esi,lpMem
  add esi,eax
  mov edi,@lpMem
  add edi,eax
  sub edi,@dwDataSize
  rep movsb   
  pop edi
  pop esi
  jmp nextSection
delsec:
  push [esi].SizeOfRawData
  pop @dwDataSize
;清除相关的 DATA_DIRECTORY 数据
  push edi
  mov edi,@lpMem
  add edi,DWORD ptr [edi+3CH]
  assume edi:ptr IMAGE_NT_HEADERS
  add edi,4
  add edi,sizeof IMAGE_FILE_HEADER
  add edi,60H
  assume edi:ptr IMAGE_DATA_DIRECTORY
  mov ecx,16
loopDD:
  mov eax,[esi].VirtualAddress 
  cmp eax,[edi].VirtualAddress
  jne @f
  mov [edi].VirtualAddress,0
  mov [edi].isize,0
@@:
  add edi,sizeof IMAGE_DATA_DIRECTORY
  loop loopDD   
  assume edi:nothing
  pop edi
;删除的是最后一个 Section
  cmp edx,@dwSNum
  jne secmid
  push edi
  mov edi,@lpMem
  add edi,DWORD ptr [edi+3CH]
  assume edi:ptr IMAGE_NT_HEADERS
  invoke _Align,[esi].Misc.VirtualSize,@dwMemAlign
  sub [edi].OptionalHeader.SizeOfImage,eax   
  pop edi
  assume edi:ptr IMAGE_SECTION_HEADER
  jmp nextSection
;删除的是中间的 Section,修正上一个 Section 虚拟大小,需要计算对齐
secmid:
  mov eax,[esi].Misc.VirtualSize
  invoke _Align,eax,@dwMemAlign
  add [edi-40].Misc.VirtualSize,eax
  add esi,sizeof IMAGE_SECTION_HEADER
nextSection:
  inc edx
  cmp edx,@dwSNum
  jle loopsec   
;修正 NT_HEADER  NumberOfSections，复制绑定输入
  mov edi,@lpMem
  add edi,DWORD ptr[edi+3CH]
  assume edi:ptr IMAGE_NT_HEADERS
  dec [edi].FileHeader.NumberOfSections
  mov ecx,[edi].OptionalHeader.DataDirectory[88].isize
  test ecx,ecx
  jz @f
  mov edx,[edi].OptionalHeader.DataDirectory[88].VirtualAddress
  mov ebx,edx
  sub ebx,sizeof IMAGE_SECTION_HEADER
  mov [edi].OptionalHeader.DataDirectory[88].VirtualAddress,ebx
  movzx eax,[edi].FileHeader.NumberOfSections   
  add di,[edi].FileHeader.SizeOfOptionalHeader
  add edi,sizeof IMAGE_FILE_HEADER
  add edi,4
  imul eax,40
  add edi,eax
  mov esi,lpMem
  add esi,edx
  rep movsb   
;更新文件
@@:
  invoke UnmapViewOfFile,lpMem
  invoke CloseHandle,hMapFile   
  invoke WriteFile,hFile,@lpMem,dwFileSize,addr @dwWriteSize,NULL
  mov ecx,dwFileSize
  sub ecx,@dwDataSize
  invoke SetFilePointer,hFile,ecx,NULL,FILE_BEGIN
  invoke SetEndOfFile,hFile
  
  invoke CloseHandle,hFile     
  invoke UnmapViewOfFile,@lpMem
  invoke CloseHandle,@hMap   
  invoke _OpenFile
Err:
  ret   
_DelSection endp

;添加 Section
_AddSection proc scName,dwSize
  local @dwSNum,@dwHeadSize,@dwDataSize,@dwSizeOfHeader,@dwWriteSize,@dwBindSize,@dwBindAdd
  local @dwFileAlign,@dwMemAlign
  local @lpMem,@hMap
  lea eax,dwSize
  mov esi,lpMem
  assume esi:ptr IMAGE_DOS_HEADER   
  add esi,[esi].e_lfanew
  assume esi:ptr IMAGE_NT_HEADERS
  push [esi].OptionalHeader.FileAlignment
  pop  @dwFileAlign
  push [esi].OptionalHeader.SectionAlignment
  pop   @dwMemAlign
  push [esi].OptionalHeader.SizeOfHeaders
  pop  @dwSizeOfHeader
  movzx ecx,[esi].FileHeader.NumberOfSections
  mov @dwSNum,ecx
  push DWORD ptr[esi+0D4H]
  pop @dwBindSize
  push DWORD ptr[esi+0D0H]
  pop  @dwBindAdd
;计算 PE 头剩余空间，最后一个 Section  到第一个节数据直接的空间，包括绑定输入数据的尺寸
  add si,[esi].FileHeader.SizeOfOptionalHeader
  add esi,4
  add esi,sizeof IMAGE_FILE_HEADER
  assume esi:ptr IMAGE_SECTION_HEADER
  mov eax,[esi].PointerToRawData
  add eax,lpMem
  @@:
  add esi,sizeof IMAGE_SECTION_HEADER
  loop @b
  mov ebx,esi
  add ebx,@dwBindSize
  sub eax,ebx
  cmp eax,sizeof IMAGE_SECTION_HEADER
  mov @dwHeadSize,0
  ja @f
;计算 PE 头需要添加的尺寸
  push @dwFileAlign
  pop   @dwHeadSize
  @@:
;计算文件尾需要添加的尺寸
  invoke _Align,dwSize,@dwFileAlign
  mov @dwDataSize,eax
;创建新内存映射对象，尺寸为原文件大小加扩充大小
  mov ecx,dwFileSize
  add ecx,@dwHeadSize
  add ecx,@dwDataSize
  invoke CreateFileMapping,INVALID_HANDLE_VALUE,NULL,PAGE_READWRITE,0,ecx,NULL
  test eax,eax
  jz errMap
  mov @hMap,eax
  invoke MapViewOfFile,eax,FILE_MAP_WRITE,0,0,0
  test eax,eax
  jz errView
  mov @lpMem,eax
;拷贝 PE 头
  mov esi,lpMem
  mov edi,@lpMem
  mov ecx,@dwSizeOfHeader
  rep movsb
;拷贝节区数据
  add edi,@dwHeadSize
  mov ecx,dwFileSize
  sub ecx,@dwSizeOfHeader
  rep movsb   
;修正新映像文件头
  mov esi,@lpMem
  assume esi:ptr IMAGE_DOS_HEADER
  add esi,[esi].e_lfanew
  assume esi:ptr IMAGE_NT_HEADERS
  ;如果有绑定表修正
  mov ecx,@dwBindSize
  test ecx,ecx
  jz @f
  mov eax,sizeof IMAGE_SECTION_HEADER
  add DWORD ptr[esi+0D0H],eax
  ;节数目修正   
  @@:
  inc [esi].FileHeader.NumberOfSections   
  ;PE 头大小修正
  mov ecx,@dwSizeOfHeader
  add ecx,@dwHeadSize
  mov [esi].OptionalHeader.SizeOfHeaders,ecx
  ;PE 映像大小修正
  invoke _Align,dwSize,@dwMemAlign
  add [esi].OptionalHeader.SizeOfImage,eax
  ;定位第一个 Section
  add si,[esi].FileHeader.SizeOfOptionalHeader
  add esi,4
  add esi,sizeof IMAGE_FILE_HEADER
  assume esi:ptr IMAGE_SECTION_HEADER
;循环修正所有 Section 数值 PointerOfRawData;
  mov ecx,@dwSNum
  mov edx,@dwHeadSize
  @@:   
  add [esi].PointerToRawData,edx   
  add esi,40
  loop @b   
;添加新 Section
  mov ecx,[esi-40].VirtualAddress
  add ecx,[esi-40].Misc.VirtualSize
  invoke _Align,ecx,@dwMemAlign
  mov [esi].VirtualAddress,eax
  mov eax,dwSize   
  mov [esi].Misc.VirtualSize,eax   
  mov ecx,[esi-40].PointerToRawData
  add ecx,[esi-40].SizeOfRawData
  mov [esi].PointerToRawData,ecx
  push @dwDataSize
  pop [esi].SizeOfRawData
  mov [esi].Characteristics,0E0000060H
  lea edi,[esi].Name1
  mov esi,scName
  mov ecx,8
  rep movsb
;复制绑定表数据
  mov ecx,@dwBindSize
  test ecx,ecx
  jz @f
  mov edi,@lpMem
  add edi,@dwBindAdd
  add edi,40
  mov esi,lpMem
  add esi,@dwBindAdd
  mov ecx,@dwBindSize
  rep movsb     
;更新文件
@@:
  invoke UnmapViewOfFile,lpMem
  invoke CloseHandle,hMapFile   
  mov ecx,dwFileSize
  add ecx,@dwHeadSize
  add ecx,@dwDataSize   
  invoke WriteFile,hFile,@lpMem,ecx,addr @dwWriteSize,NULL   
  invoke CloseHandle,hFile     
  invoke UnmapViewOfFile,@lpMem
  invoke CloseHandle,@hMap   
  invoke _OpenFile
  ret   
errMap:
  invoke MessageBox,NULL,offset szErrMap,offset szErrMap,MB_OK
  ret
errView:
  invoke MessageBox,NULL,offset szErrView,offset szErrView,MB_OK   
  ret
_AddSection endp

;显示 Section 信息
_PESectionInfo proc
  local @szBuf[1024]:byte
  mov esi,lpMem
  add esi,DWORD ptr [esi+3CH]
  assume esi:ptr IMAGE_NT_HEADERS
  movzx ecx,[esi].FileHeader.NumberOfSections
  add si,WORD ptr [esi].FileHeader.SizeOfOptionalHeader
  add esi,4
  add esi,sizeof IMAGE_FILE_HEADER
  assume esi:ptr IMAGE_SECTION_HEADER
  invoke _AppendInfo,offset szMsgSection
  cld
  xor ebx,ebx
;循环显示 SECTION
@@:   
  push ecx
  invoke wsprintf,addr @szBuf,offset szFmtSection,ebx,addr [esi].Name1,[esi].Misc.VirtualSize,[esi].VirtualAddress,[esi].SizeOfRawData,[esi].PointerToRawData,[esi].Characteristics
  invoke _AppendInfo,addr @szBuf
  add esi,sizeof IMAGE_SECTION_HEADER
  pop ecx
  inc ebx
  loop @b   
  ret
_PESectionInfo endp

;获取文件名
_OpenFile proc
  invoke CreateFile,offset szFileName,GENERIC_READ or GENERIC_WRITE,FILE_SHARE_READ,NULL,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,NULL
  cmp  eax,INVALID_HANDLE_VALUE
  je  ErrOpen   
  mov hFile,eax
  invoke GetFileSize,eax,0
  mov dwFileSize,eax
  invoke CreateFileMapping,hFile,NULL,PAGE_READONLY,0,0,NULL
  cmp eax,NULL
  je   ErrMap
  mov hMapFile,eax
  invoke MapViewOfFile,eax,FILE_MAP_READ,0,0,0
  cmp eax,NULL   
  je  ErrView
;判断是否为 pe 文件   
  mov lpMem,eax
  ;  cmp word ptr [eax],5A4DH
  cmp word ptr [eax],"ZM"
  jne ErrPe
  add eax,[eax+03CH]
  ;  cmp word ptr [eax],4550H
  cmp word ptr [eax],"EP"
  jne ErrPe
  invoke _PESectionInfo
  ret
ErrOpen:
  lea eax,offset szErrOpen
  invoke CloseHandle,hFile
  jmp @f
ErrMap:
  lea eax,offset szErrMap   
  invoke CloseHandle,hMapFile
  jmp @f
ErrView:
  lea eax,offset szErrView
  invoke UnmapViewOfFile,lpMem   
  jmp @f   
ErrPe:
  mov eax,offset szErrPE   
  jmp @f   
@@:
  invoke SendMessage,hWinMain,WM_SETTEXT,NULL,eax   
  ret
_OpenFile endp

_DlgProc proc hWnd,wMsg,wParam,lParam   
  local @szNameBuf[8]:BYTE
  mov eax,wMsg
  cmp eax,WM_CLOSE
  je  wmclose
  cmp eax,WM_INITDIALOG
  je   wminit
  cmp eax,WM_COMMAND
  je   wmcommand
  cmp eax,WM_DROPFILES
  je  wmdrop
  xor eax,eax
  jmp finish   
wminit:
  push hWnd
  pop  hWinMain
  invoke SetDlgItemText,hWnd,IDC_EDIT_NAME,offset szMsg
  invoke SetDlgItemText,hWnd,IDC_EDIT_SIZE,offset szMsg2
  invoke GetDlgItem,hWnd,IDC_RICHEDIT
  mov hRichHandle, eax
  jmp process
wmcommand:
  mov eax,wParam   
  cmp ax,ID_OK
  jne @f
  lea edi,@szNameBuf
  xor eax,eax
  mov ecx,8
  rep stosb 
  invoke GetDlgItemText,hWnd,IDC_EDIT_NAME,addr @szNameBuf,8
  invoke GetDlgItemInt,hWnd,IDC_EDIT_SIZE,NULL,FALSE
  test eax,eax
  jz   process
  invoke _AddSection,addr @szNameBuf,eax   
  jmp process
  wmdrop:
  invoke DragQueryFile,wParam,0,offset szFileName,MAX_PATH
  invoke SendMessage,hWnd,WM_SETTEXT,NULL,offset szFileName
  invoke _OpenFile
  jmp process
@@:   
  cmp ax,ID_DEL
  jne @f
  invoke GetDlgItemInt,hWnd,IDC_EDIT_SIZE,NULL,FALSE
  cmp lpMem,0
  jz process
  invoke _DelSection,eax   
  jmp process   
@@:   
  jmp finish
wmclose:
  invoke EndDialog,hWnd,0
process:
  mov eax,1
finish:
  ret
_DlgProc endp

start:
  invoke LoadLibrary,offset szDllName
  mov hRichHandle,eax
  invoke GetModuleHandle,NULL
  mov hInstancd,eax
  invoke DialogBoxParam,hInstancd,IDD_DIALOG,NULL,offset _DlgProc,NULL   
  invoke  ExitProcess,NULL     
  end start
```



上传的附件

| `![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg)` | | `AddSection.rar` |
|---|---|---|
