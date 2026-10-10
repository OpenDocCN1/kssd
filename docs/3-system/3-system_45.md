

# 我写的 AddSection 源码，希望对各位学习 PE 格式有点用

> 原文：[`www.pediy.com/kssd/pediy10/85711.html`](https://www.pediy.com/kssd/pediy10/85711.html)

主要功能是给 pe 文件增加一个节（像记事本有绑定输入结构的先清 0，在增加节，ZeroAdd，topo 对这些程序的处理会出错），然后把数据文件写入新节，例子见我写的另一篇“PEDIY 技术之新思路（一）”文章，希望各位指出不足之处，并加以完善，谢谢了！部分代码来自 cyclotron 大虾写的 pelibrary，特别感谢！  
\----------------------------------------------------AddSection.asm------------------------------------------------------------------------  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
.586  
.model flat,stdcall  
option casemap:none  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
include windows.inc  
include user32.inc  
include kernel32.inc  
includelib user32.lib  
includelib kernel32.lib  
include  Comdlg32.inc  
includelib  Comdlg32.lib  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
IDD\_DLGMain equ 1000  
IDC\_BDesFile equ 1001  
IDC\_EDesFile equ 1003  
IDC\_EScnName equ 1005  
IDC\_BData equ 1007  
IDC\_EData equ 1008  
IDC\_EScnSize equ 1009  
IDC\_CHKWrite equ 1012  
IDC\_CHKUpEntry equ  1013  
IDC\_EEntry equ 1017  
IDC\_EBase equ 1019  
IDC\_EScnNum equ 1021  
IDC\_EVOffset equ 1023  
IDC\_EFOffset equ 1025  
IDC\_ERSize equ 1027  
IDC\_BAbout equ 1029  
IDC\_BMake equ 1030  
IDC\_BExit equ 1031  
ICO\_ICON  equ 1045  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
.data?  
hInstance   dd ?  
hWinMain   dd ?  
hInFile    dd  ?  
hInDataFile dd ?  
hOutFile  dd  ?  
dwFileSize  dd  ?  
dwDataFileSize dd ?  
lpDataHead  dd ?  
lpBytesRead  dd  ?  
dwNewEntry  dd   ?  
lpPeHeader  dd  ?  
lpFileHeader  dd  ?  
szDesFileName  db  MAX\_PATH dup (?)  
szDataFileName db MAX\_PATH dup (?)  
szBuffer  db 100 dup(?)  
dwNewScnSize dd ?  
UpEntryFlag  dd ?  
szNewSectionName  db  8 dup(?)  
WriteDataFlag dd ?  
IsPe  dd ?  
  
PE\_INFO  struct  
  dwEntryPointRVA  dd  ?  
  dwImageBase  dd  ?  
  dwNumberOfSections  dd  ?  
PE\_INFO  ends  
  
SCN\_INFO  struct  
  dwVirtualSize  dd  ?  
  dwVirtualAddress  dd  ?  
  dwRawSize  dd  ?  
  dwRawAddress  dd  ?  
  dwCharacteristics  dd  ?  
SCN\_INFO  ends  
  
EXTRADATA\_INFO  struct  
  dwRawAddress  dd  ?  
  dwSize    dd  ?  
  lpExtraData  dd  ?  
EXTRADATA\_INFO  ends  
  
OrglastScainfo  SCN\_INFO  <>  
NewScnInfo   SCN\_INFO   <>  
PeInfo   PE\_INFO   <>  
ExtraData     EXTRADATA\_INFO  <>  
.data  
szCaption db  'Info',0  
szText   db  '成功完成',0  
ScnFlagsDefault      dd   0E0000020h  
szAbout db '    AddSection 1.0',0dh,0ah  
        db '      by:moonife',0dh,0ah  
        db '    QQ:765496322',0dh,0ah  
        db 'E-mail:moonife@163.com',0  
szAboutCaption db 'About',0        
szFilter  db 'All Files(\*.\*)',0,'\*.\*',0,0  
szNotPe  db '不是 PE 文件，请重新选择！',0  
szNotPeCap db 'Error',0  
szNoFile  db '请先选择 PE 文件',0  
szScnSize db  '请填写字节大小',0  
szScnName db  '请填写节区名字',0  
szData  db  '请先选择数据文件',0  
szFormat  db '%lXh',0  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
.code  
;>>>>>>>>>>>>>>>  
ErrorReport  proc   
invoke  MessageBox,NULL,offset szText,offset szCaption,MB\_ICONERROR or MB\_OK  
ret  
ErrorReport  endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
\_OpenFile  proc  @lpFileName:dword  
    local  @stOF:OPENFILENAME  
  
    invoke  RtlZeroMemory,addr @stOF,sizeof @stOF  
    mov  @stOF.lStructSize,sizeof @stOF  
    push  hWinMain  
    pop  @stOF.hwndOwner  
    mov  @stOF.lpstrFilter,offset szFilter  
    push @lpFileName  
    pop  @stOF.lpstrFile  
    mov  @stOF.nMaxFile,MAX\_PATH  
    mov  @stOF.Flags,OFN\_FILEMUSTEXIST or OFN\_PATHMUSTEXIST  
    invoke  GetOpenFileName,addr @stOF  
    ret  
  
\_OpenFile  endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
InitFileMap  proc   ;把文件映射到内存，然后进行处理 return 0 or 1  
  invoke  CreateFile,addr szDesFileName,\\  
                       GENERIC\_READ,\\  
                       FILE\_SHARE\_READ,\\  
                       NULL,OPEN\_EXISTING,FILE\_ATTRIBUTE\_ARCHIVE,\\  
                       NULL    
  .if  eax == INVALID\_HANDLE\_VALUE  
    mov  eax,FALSE  
    ret  
  .else  
    mov  hInFile,eax  
  .endif  
  invoke  GetFileSize,hInFile,NULL  
  mov  dwFileSize,eax  
  invoke  VirtualAlloc,NULL,eax,MEM\_COMMIT,PAGE\_READWRITE  
  mov  lpFileHeader,eax  
  invoke  ReadFile,hInFile,eax,dwFileSize,offset lpBytesRead,NULL  
  .if  eax != 0  
    mov  eax,TRUE  
  .else  
    mov  eax,FALSE  
  .endif  
  ret  
InitFileMap  endp  
;>>>>>>>>>>>>>>>>>>>  
InitDataFile proc  
invoke  CreateFile,addr szDataFileName,\\  
                       GENERIC\_READ,\\  
                       FILE\_SHARE\_READ,\\  
                       NULL,OPEN\_EXISTING,FILE\_ATTRIBUTE\_ARCHIVE,\\  
                       NULL    
  .if  eax == INVALID\_HANDLE\_VALUE  
    mov  eax,FALSE  
    ret  
  .else  
    mov  hInDataFile,eax  
  .endif  
  invoke  GetFileSize,hInDataFile,NULL  
  mov  dwDataFileSize,eax  
  invoke  VirtualAlloc,NULL,eax,MEM\_COMMIT,PAGE\_READWRITE  
  mov  lpDataHead,eax  
  invoke  ReadFile,hInDataFile,eax,dwDataFileSize,offset lpBytesRead,NULL  
  .if  eax != 0  
    mov  eax,TRUE  
  .else  
    mov  eax,FALSE  
  .endif  
  ret  
 InitDataFile  endp  
;>>>>>>>>>>>>>>>>>>>>>  
CheckPEValidity  proc  uses esi     ;判断是否为 pe 文件 return  1 or 0  
  mov  esi,lpFileHeader  
  .if  word ptr \[esi\] == IMAGE\_DOS\_SIGNATURE  
    assume  esi:ptr IMAGE\_DOS\_HEADER  
    add  esi,\[esi\].e\_lfanew  
    assume  esi:nothing  
    .if  word ptr \[esi\] == IMAGE\_NT\_SIGNATURE  
      mov  lpPeHeader,esi  
    .else  
      mov  eax,FALSE  
      ret  
    .endif  
  .else  
    mov  eax,FALSE  
    ret  
  .endif  
  mov  eax,TRUE  
  ret  
CheckPEValidity  endp  
;>>>>>>>>>>>>>>>>>>>>>>  
GetPeInfo  proc  uses esi edi @lpPeInfo:DWORD   ;获取 pe 基本信息  
  mov  esi,lpPeHeader  
  assume  esi:ptr IMAGE\_NT\_HEADERS  
  mov  edi,@lpPeInfo  
  assume  edi:ptr PE\_INFO  
  push  \[esi\].OptionalHeader.AddressOfEntryPoint  
  pop  \[edi\].dwEntryPointRVA  
  push  \[esi\].OptionalHeader.ImageBase  
  pop  \[edi\].dwImageBase  
  mov  ax,\[esi\].FileHeader.NumberOfSections  
  cwd  
  mov  \[edi\].dwNumberOfSections,eax  
  assume  esi:nothing  
  assume  edi:nothing  
  ret  
GetPeInfo  endp  
;>>>>>>>>>>>>>>>>>>>>>>>>  
CloseFileMap  proc  @lpFileHeader:DWORD ,@hInFile:dword   ;释放文件影像 return 1 or 0  
  invoke  VirtualFree,@lpFileHeader,0,MEM\_RELEASE  
  .if  eax==0  
                mov  eax,FALSE  
                 ret   
  .endif  
  mov  @lpFileHeader,0  
        invoke  CloseHandle,@hInFile  
        mov  @hInFile,0  
        mov  eax,TRUE  
        ret  
CloseFileMap  endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
; 更新入口点 RVA，返回值为原入口点 RVA  
UpdateEntryPoint  proc  uses esi @dwNewEntryPoint:dword  ;Return OEP  
  mov  esi,lpPeHeader  
  assume  esi:ptr IMAGE\_NT\_HEADERS  
  mov  eax,\[esi\].OptionalHeader.AddressOfEntryPoint  
  add eax,\[esi\].OptionalHeader.ImageBase  
  push  @dwNewEntryPoint  
  pop  \[esi\].OptionalHeader.AddressOfEntryPoint  
  assume  esi:nothing  
  ret  
UpdateEntryPoint  endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
; 将输入双字按磁盘文件对齐值对齐  
AlignFile  proc  uses ecx edx esi @OrgSize:dword  
  mov  esi,lpPeHeader  
  assume  esi:ptr IMAGE\_NT\_HEADERS  
  mov  ecx,\[esi\].OptionalHeader.FileAlignment  
  assume  esi:nothing  
  mov  eax,@OrgSize  
  cdq  
  div  ecx  
  .if  edx != 0  
    inc  eax  
  .endif  
  mul  ecx  
  ret  
AlignFile  endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
; 将输入双字按区块对齐值对齐  
AlignScn  proc  uses ecx edx esi @OrgSize:dword  
  mov  esi,lpPeHeader  
  assume  esi:ptr IMAGE\_NT\_HEADERS  
  mov  ecx,\[esi\].OptionalHeader.SectionAlignment  
  assume  esi:nothing  
  mov  eax,@OrgSize  
  cdq  
  div  ecx  
  .if  edx != 0  
    inc  eax  
  .endif  
  mul  ecx  
  ret  
AlignScn  endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
; 取得指定区块的信息，0 表示最后一个区块，同时如果输入区块序号大于区块数，也取得最后一个区块的信息  
GetScnInfo  proc  uses ecx edx esi edi @nScn:dword,@lpScnInfo:dword    
  mov  esi,lpPeHeader  
  assume  esi:ptr IMAGE\_NT\_HEADERS  
  xor  ecx,ecx  
  mov  cx,\[esi\].FileHeader.NumberOfSections  
  add  esi,sizeof IMAGE\_NT\_HEADERS  
  .if  @nScn == 0 || @nScn > ecx  
    dec  ecx  
    mov  @nScn,0  
  .else  
    mov  ecx,@nScn  
    dec  ecx  
  .endif  
  mov  eax,sizeof IMAGE\_SECTION\_HEADER  
  mul  ecx  
  add  esi,eax  
  assume  esi:ptr IMAGE\_SECTION\_HEADER  
  mov  edi,@lpScnInfo  
  assume  edi:ptr SCN\_INFO  
  push  \[esi\].Misc.VirtualSize  
  pop  \[edi\].dwVirtualSize  
  push  \[esi\].VirtualAddress  
  pop  \[edi\].dwVirtualAddress  
  push  \[esi\].SizeOfRawData  
  pop  \[edi\].dwRawSize  
  mov  eax,\[esi\].PointerToRawData  
  mov  \[edi\].dwRawAddress,eax  
  push  \[esi\].Characteristics  
  pop  \[edi\].dwCharacteristics  
  .if  @nScn == 0  
    mov  eax,FALSE      
  .else  
    mov  eax,TRUE  
  .endif  
  assume  esi:nothing  
  assume  edi:nothing  
  ret  
GetScnInfo  endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
; 增加一个区块的块头信息，可以指定区块名称、大小和属性，同时更新 NumberOfSections 字段和 ImageSize 字段  
AddSection  proc  uses ebx ecx edx esi edi @szScnName:dword,@dwSize:dword,@dwCharacteristics:dword  
  local  @stNewScnInfo:SCN\_INFO  
  local  @dwImageSize  
  xor eax,eax  
  mov  esi,lpPeHeader  
  assume  esi:ptr IMAGE\_NT\_HEADERS  
  lea  ebx,\[esi\].OptionalHeader.DataDirectory\\  
      \[IMAGE\_DIRECTORY\_ENTRY\_BOUND\_IMPORT\*sizeof IMAGE\_DATA\_DIRECTORY\]  
  mov  edi,\[ebx\]  
  .if  edi != 0  
    add  edi,lpFileHeader  
    mov  ecx,\[ebx+4\]  
    rep  stosb  
    mov  edi,ebx  
    mov  ecx,sizeof IMAGE\_DATA\_DIRECTORY  
    rep  stosb  
  .endif  
  xor  ecx,ecx  
  mov  cx,\[esi\].FileHeader.NumberOfSections  
  add  esi,sizeof IMAGE\_NT\_HEADERS  
  assume  esi:ptr IMAGE\_SECTION\_HEADER  
  mov  edi,\[esi\].PointerToRawData  
  add  edi,lpFileHeader  
  mov  eax,sizeof IMAGE\_SECTION\_HEADER  
  mul  ecx  
  add  esi,eax  
  add  esi,sizeof IMAGE\_SECTION\_HEADER  
  .if  esi <= edi  
    sub  esi,sizeof IMAGE\_SECTION\_HEADER  
    invoke  GetScnInfo,0, addr @stNewScnInfo  
    push  @dwSize  
    pop  \[esi\].Misc.VirtualSize  
    mov  eax,@stNewScnInfo.dwVirtualSize  
    ;mov  eax,@dwSize  
    invoke  AlignScn,eax  
    add  eax,@stNewScnInfo.dwVirtualAddress  
    mov dwNewEntry,eax  
    mov  \[esi\].VirtualAddress,eax  
    add  eax,@dwSize  
    mov  @dwImageSize,eax  
    invoke  AlignFile,@dwSize  
    mov  \[esi\].SizeOfRawData,eax  
    mov  eax,@stNewScnInfo.dwRawSize  
    invoke  AlignFile,eax  
    add  eax,@stNewScnInfo.dwRawAddress  
    mov  \[esi\].PointerToRawData,eax  
    push  @dwCharacteristics  
    pop  \[esi\].Characteristics  
    mov  edi,@szScnName  
    xchg  esi,edi  
    mov  ecx,8  
    rep  movsb  
    mov  esi,lpPeHeader  
    assume  esi:ptr IMAGE\_NT\_HEADERS  
    inc  \[esi\].FileHeader.NumberOfSections  
    push  @dwImageSize  
    ;push   0C000h  
    pop  \[esi\].OptionalHeader.SizeOfImage  
    mov  eax,TRUE  
  .else  
    mov  eax,FALSE  
  .endif  
  assume  esi:nothing  
  assume  edi:nothing  
  ret  
AddSection  endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
; 取得附加数据的信息  
GetExtraDataInfo  proc  uses ecx edx esi @lpExtraDataInfo:dword  
  mov  esi,lpPeHeader  
  assume  esi:ptr IMAGE\_NT\_HEADERS  
  xor  ecx,ecx  
  mov  cx,\[esi\].FileHeader.NumberOfSections  
  add  esi,sizeof IMAGE\_NT\_HEADERS  
  mov  eax,sizeof IMAGE\_SECTION\_HEADER  
  dec  ecx  
  mul  ecx  
  add  esi,eax  
  assume  esi:ptr IMAGE\_SECTION\_HEADER  
  mov  edx,\[esi\].PointerToRawData  
  add  edx,\[esi\].SizeOfRawData  
  push  edx  
  invoke  GetFileSize,hInFile,NULL  
  pop  edx  
  mov  esi,@lpExtraDataInfo  
  assume  esi:ptr EXTRADATA\_INFO  
  .if  eax > edx  
    mov  \[esi\].dwRawAddress,edx  
    sub  eax,edx  
    mov  \[esi\].dwSize,eax  
    add  edx,lpFileHeader  
    mov  \[esi\].lpExtraData,edx  
    mov  eax,TRUE  
  .else  
    mov  \[esi\].dwRawAddress,0  
    mov  eax,FALSE  
  .endif  
  assume  esi:nothing  
  ret  
GetExtraDataInfo  endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
; 输出文件命名为 xxx@.exe，并创建该文件，返回值为该文件的句柄  
CreateOutFile  proc  uses ecx edi  
  lea  edi,szDesFileName  
  xor  ecx,ecx  
  dec  ecx  
  mov  al,'.'  
  repnz  scasb  
  dec  edi  
  mov  al,'@'  
  stosb  
  mov  eax,'exe.'  
  stosd  
  xor  al,al  
  stosb  
  invoke  CreateFile,offset szDesFileName,GENERIC\_READ or GENERIC\_WRITE,\\  
          FILE\_SHARE\_READ,\\  
          NULL,CREATE\_ALWAYS,\\  
          FILE\_ATTRIBUTE\_NORMAL,NULL  
  .if  eax == INVALID\_HANDLE\_VALUE  
    mov  eax,FALSE  
   .else  
   mov  hOutFile,eax  
  .endif  
  ret  
CreateOutFile  endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
; 在新文件中写入新增区块并补上附加数据  
AppendScnCode  proc  uses ebx esi edi @lpOrgLastScnInfo:dword,@lpExtraDataInfo:dword,\\  
          @lpNewScnData:dword,@dwNewScnSize:dword  
  local  lpBytesWritten:dword  
    
  mov  esi,@lpOrgLastScnInfo  
  assume  esi:ptr SCN\_INFO  
  mov  ebx,\[esi\].dwRawSize  
  add  ebx,\[esi\].dwRawAddress  
  invoke  WriteFile,hOutFile,lpFileHeader,ebx,addr lpBytesWritten,NULL  
  invoke  AlignFile,@dwNewScnSize  
  mov  ebx,eax  
  invoke  WriteFile,hOutFile,@lpNewScnData,ebx,addr lpBytesWritten,NULL  
  mov  edi,@lpExtraDataInfo  
  assume  edi:ptr EXTRADATA\_INFO  
  .if  \[edi\].dwRawAddress != 0  
    invoke  WriteFile,hOutFile,\[edi\].lpExtraData,\[edi\].dwSize,addr lpBytesWritten,NULL  
  .endif  
  invoke  SetEndOfFile,hOutFile  
  assume  esi:nothing  
  assume  edi:nothing  
  ret  
AppendScnCode  endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
\_windowcenter proc uses ebx edi esi hWnd  
local @DlgHeight,@DlgWidth,@x,@y  
 local @DlgRect:RECT,@DesktopRect:RECT  
 local @hwindow  
 invoke   GetWindowRect,hWnd,addr @DlgRect  
      invoke   GetDesktopWindow  
      mov @hwindow,eax  
      invoke   GetWindowRect,@hwindow,addr @DesktopRect  
      xor eax,eax  
      mov eax,@DlgRect.bottom  
      sub eax,@DlgRect.top  
      mov @DlgHeight,eax  
      xor eax,eax  
      mov eax,@DlgRect.right  
      sub eax,@DlgRect.left  
      mov @DlgWidth,eax  
      xor eax,eax  
      mov eax,@DesktopRect.left  
      add eax,@DesktopRect.right  
      sub eax,@DlgWidth  
      sar eax,1  
      mov @x,eax  
      xor eax,eax  
      mov eax,@DesktopRect.top  
      add eax,@DesktopRect.bottom  
      sub eax,@DlgHeight  
      sar eax,1  
      mov @y,eax  
      invoke MoveWindow,hWnd,@x,@y,@DlgWidth,@DlgHeight,FALSE  
      mov eax,TRUE  
      ret  
      \_windowcenter endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
\_ProcDlgMain  proc  uses ebx edi esi hWnd,wMsg,wParam,lParam  
    mov  eax,wMsg  
    .if  eax == WM\_CLOSE  
      invoke  EndDialog,hWnd,NULL  
    .elseif  eax == WM\_INITDIALOG  
      invoke  LoadIcon,hInstance,ICO\_ICON  
      invoke  SendMessage,hWnd,WM\_SETICON,ICON\_BIG,eax  
      invoke   \_windowcenter,hWnd  
      push hWnd  
      pop  hWinMain  
    .elseif  eax == WM\_COMMAND  
      mov  eax,wParam  
      .if  ax == IDC\_BExit  
        invoke CloseFileMap,lpFileHeader,hInFile  
            invoke CloseFileMap,lpDataHead,hInDataFile  
        invoke  EndDialog,hWnd,NULL  
      .elseif ax==IDC\_BDesFile  
        invoke \_OpenFile,addr szDesFileName  
        mov eax,DWORD ptr szDesFileName  
        .if eax     
        invoke SetDlgItemText,hWnd,IDC\_EDesFile,addr szDesFileName  
        invoke InitFileMap  
        invoke  CheckPEValidity  
        .if !eax  
          invoke MessageBox,hWnd,addr szNotPe,addr szNotPeCap,MB\_OK or MB\_ICONERROR  
        .else    
        mov IsPe,1  
        invoke  GetPeInfo,addr PeInfo  
        invoke  wsprintf,addr szBuffer,addr szFormat,PeInfo.dwEntryPointRVA  
        invoke  SetDlgItemText,hWnd,IDC\_EEntry,addr szBuffer  
        invoke  wsprintf,addr szBuffer,addr szFormat,PeInfo.dwImageBase  
        invoke  SetDlgItemText,hWnd,IDC\_EBase,addr szBuffer  
        invoke  SetDlgItemInt,hWnd,IDC\_EScnNum,PeInfo.dwNumberOfSections,FALSE  
        .endif  
        .endif  
      .elseif ax==IDC\_BData    
        invoke \_OpenFile,addr szDataFileName  
        invoke SetDlgItemText,hWnd,IDC\_EData,addr szDataFileName  
      .elseif ax==IDC\_CHKWrite  
      invoke  IsDlgButtonChecked,hWnd,IDC\_CHKWrite  
        .if  eax == BST\_CHECKED  
          mov WriteDataFlag,TRUE  
          mov ebx,0  
        .else  
          mov WriteDataFlag,FALSE  
          mov ebx,1  
        .endif  
        invoke  GetDlgItem,hWnd,IDC\_EScnSize  
        ;not WriteDataFlag    
        invoke  EnableWindow,eax, ebx  
      .elseif  ax==IDC\_CHKUpEntry  
        invoke  IsDlgButtonChecked,hWnd,IDC\_CHKUpEntry  
        .if  eax == BST\_CHECKED  
          mov UpEntryFlag,TRUE  
        .else  
          mov UpEntryFlag,FALSE  
        .endif  
      .elseif ax==IDC\_BMake  
        mov eax,DWORD ptr szDesFileName  
        .if !eax || !IsPe             invoke MessageBox,hWnd,addr szNoFile,addr szNotPeCap,MB\_OK or MB\_ICONERROR  
        .else    
        invoke  GetScnInfo,0, addr OrglastScainfo  
        invoke  GetDlgItemText,hWnd,IDC\_EScnName,addr szNewSectionName,8  
        .if !eax  
            invoke MessageBox,hWnd,addr szScnName,addr szNotPeCap,MB\_OK or MB\_ICONERROR  
            mov  eax,TRUE  
            ret  
          .endif    
        .if !WriteDataFlag            
          invoke GetDlgItemInt,hWnd,IDC\_EScnSize,NULL,FALSE  
          .if !eax  
            invoke MessageBox,hWnd,addr szScnSize,addr szNotPeCap,MB\_OK or MB\_ICONERROR  
            mov  eax,TRUE  
            ret  
          .endif    
          mov dwNewScnSize,eax  
        .else   
          mov eax,DWORD ptr szDataFileName  
          .if !eax  
            invoke MessageBox,hWnd,addr szData,addr szNotPeCap,MB\_OK or MB\_ICONERROR  
            mov  eax,TRUE  
            ret  
          .endif    
          invoke  InitDataFile  
          push  dwDataFileSize  
          pop   dwNewScnSize  
        .endif  
            invoke AddSection,offset szNewSectionName,dwNewScnSize,ScnFlagsDefault  
            invoke   GetExtraDataInfo,offset ExtraData  
            .if UpEntryFlag  
              invoke  UpdateEntryPoint,dwNewEntry  
            .endif   
            invoke  CreateOutFile  
            invoke AppendScnCode,offset OrglastScainfo,offset ExtraData,lpDataHead,dwNewScnSize  
            invoke GetScnInfo,0,addr NewScnInfo  
            invoke  wsprintf,addr szBuffer,addr szFormat,NewScnInfo.dwVirtualAddress  
        invoke  SetDlgItemText,hWnd,IDC\_EVOffset,addr szBuffer  
        invoke  wsprintf,addr szBuffer,addr szFormat,NewScnInfo.dwRawAddress  
        invoke  SetDlgItemText,hWnd,IDC\_EFOffset,addr szBuffer  
        invoke  wsprintf,addr szBuffer,addr szFormat,NewScnInfo.dwVirtualSize  
        invoke  SetDlgItemText,hWnd,IDC\_ERSize,addr szBuffer  
            invoke MessageBox,hWnd,addr szText,addr szCaption,MB\_OK  
            .endif  
      .elseif ax==IDC\_BAbout  
        invoke MessageBox,hWnd,addr szAbout,addr szAboutCaption,MB\_OK or MB\_ICONINFORMATION  
      .endif  
    .else  
      mov  eax,FALSE  
      ret  
    .endif  
    mov  eax,TRUE  
    ret  
  
\_ProcDlgMain  endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
start:  
    invoke  GetModuleHandle,NULL  
    mov  hInstance,eax  
    invoke  DialogBoxParam,hInstance,IDD\_DLGMain,NULL,offset \_ProcDlgMain,NULL  
    invoke  ExitProcess,NULL  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
    end  start  
\------------------------------------------------------the end-------------------------------------------------------------------------------------  
ps：附件中文件有：AddSection.asm ，AddSection.rc ，AddSection.exe，Main.ico

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || AddSection.rar  
 |