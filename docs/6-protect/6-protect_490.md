

# cyclotron's Win32 PE Library

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-550.htm`](https://www.pediy.com/kssd/pediy07/pediy7-550.htm)

Pamqara 写过一个 PELibrary,但自己写的东西毕竟用起来比较顺手,所以自己建了一个简单的 library,给写壳提供了一些特别的方便性.  
时间关系,注释不是很详细,让代码来说话吧,欢迎报告 bug 和扩充库:)  
  
  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
  
;>>>>>>>>             Win32 PE Library            >>>>>>>>>>>>>>>  
  
;>>>>>>>>               by cyclotron              >>>>>>>>>>>>>>>  
  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
  
.486  
.model flat, stdcall  
option casemap:none  
  
include windows.inc  
include user32.inc  
include kernel32.inc  
include comdlg32.inc  
includelib user32.lib  
includelib kernel32.lib  
includelib comdlg32.lib  
  
InitFileMap  proto  
CheckPEValidity  proto  
CloseFileMap  proto  
GetPeInfo  proto  :dword  
RVA2Offset  proto  :dword  
Offset2RVA  proto  :dword  
GetImportsInfo  proto  :dword  
GetCodeScnInfo  proto  :dword  
ObfuscateOEP  proto  :dword  
UpdateEntryPoint  proto  :dword  
AlignFile  proto  :dword  
AlignScn  proto  :dword  
AddSection  proto  :dword,:dword,:dword  
GetScnInfo  proto  :dword,:dword  
GetExtraDataInfo  proto  :dword  
MergeScns  proto  :dword,:dword,:dword  
OpenFileDialog  proto  
CreateOutFile  proto  
AppendScnCode  proto  :dword,:dword,:dword,:dword  
UpdatePackedScnHeader  proto  :dword,:dword,:dword  
CheckPackingFlag  proto  :dword  
SetPackingFlag  proto  :dword  
UpdateImportRVA  proto  :dword  
GetStrLen  proto  :dword  
tEHash    proto  :dword,:dword,:dword  
crc32    proto  :dword,:dword  
  
.data  
hInFile    dd  0  
hOutFile  dd  0  
dwFileSize  dd  0  
lpBytesRead  dd  0  
lpPeHeader  dd  0  
lpFileHeader  dd  0  
szPeFileName  db  MAX\_PATH dup(0)  
szFilter  db  'eXeFiles',0,'\*.exe',0,'All Files',0,'\*.\*',0,0  
szCurrentDir  db  '.',0  
szTitle    db  'Win32 PE Library - Proudly Presented by cyclotron',0  
CurrentHashOffset  dd  0  
  
;STRUCTURES  
PE\_INFO  struct  
  dwEntryPointRVA  dd  ?  
  dwImageBase  dd  ?  
  dwSectionAlignment  dd  ?  
  dwFileAlignment  dd  ?  
  dwSizeOfImage  dd  ?  
  dwCheckSum  dd  ?  
  dwNumberOfSections  dd  ?  
PE\_INFO  ends  
  
IMPORTS\_INFO  struct  
  dwImportDirRVA  dd  ?  
  dwImportDirSize  dd  ?  
  lpImportDir  dd  ?  
  dwIATDirRVA  dd  ?  
  dwIATDirSize  dd  ?  
  lpIATDir  dd  ?  
IMPORTS\_INFO  ends  
  
SCN\_INFO  struct  
  szName    dq  0  
  dwVirtualSize  dd  ?  
  dwVirtualAddress  dd  ?  
  dwRawSize  dd  ?  
  dwRawAddress  dd  ?  
  dwCharacteristics  dd  ?  
  lpScn    dd  ?  
SCN\_INFO  ends  
  
EXTRADATA\_INFO  struct  
  dwRawAddress  dd  ?  
  dwSize    dd  ?  
  lpExtraData  dd  ?  
EXTRADATA\_INFO  ends  
  
.code  
InitFileMap  proc  
  invoke  CreateFile,offset szPeFileName,\\  
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
  
CheckPEValidity  proc  uses esi  
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
  
CloseFileMap  proc  
  invoke  VirtualFree,lpFileHeader,0,MEM\_RELEASE  
  .if  eax==0  
                mov  eax,FALSE  
                 ret   
  .endif  
  mov  lpFileHeader,0  
        invoke  CloseHandle,hInFile  
        mov  hInFile,0  
        mov  eax,TRUE  
        ret  
CloseFileMap  endp  
  
GetPeInfo  proc  uses esi edi @lpPeInfo:dword  
  mov  esi,lpPeHeader  
  assume  esi:ptr IMAGE\_NT\_HEADERS  
  mov  edi,@lpPeInfo  
  assume  edi:ptr PE\_INFO  
  push  \[esi\].OptionalHeader.AddressOfEntryPoint  
  pop  \[edi\].dwEntryPointRVA  
  push  \[esi\].OptionalHeader.ImageBase  
  pop  \[edi\].dwImageBase  
  push  \[esi\].OptionalHeader.SectionAlignment  
  pop  \[edi\].dwSectionAlignment  
  push  \[esi\].OptionalHeader.FileAlignment  
  pop  \[edi\].dwFileAlignment  
  push  \[esi\].OptionalHeader.SizeOfImage  
  pop  \[edi\].dwSizeOfImage  
  push  \[esi\].OptionalHeader.CheckSum  
  pop  \[edi\].dwCheckSum  
  mov  ax,\[esi\].FileHeader.NumberOfSections  
  cwd  
  mov  \[edi\].dwNumberOfSections,eax  
  assume  esi:nothing  
  assume  edi:nothing  
  ret  
GetPeInfo  endp  
  
RVA2Offset  proc  uses ebx ecx edx edi esi @dwRVA:dword  
  mov  esi,lpFileHeader  
  assume  esi:ptr IMAGE\_DOS\_HEADER  
  add  esi,\[esi\].e\_lfanew  
  assume  esi:ptr IMAGE\_NT\_HEADERS  
  mov  edi,@dwRVA  
  mov  edx,esi  
  add  edx,sizeof IMAGE\_NT\_HEADERS  
  mov  cx,\[esi\].FileHeader.NumberOfSections  
  movzx  ecx,cx  
  assume  edx:ptr IMAGE\_SECTION\_HEADER  
  .while  ecx > 0  
    .if  edi >= \[edx\].VirtualAddress  
      mov  eax,\[edx\].VirtualAddress  
      add  eax,\[edx\].SizeOfRawData  
      .if  edi < eax  
        mov  eax,\[edx\].VirtualAddress  
        sub  edi,eax  
        mov  eax,\[edx\].PointerToRawData  
        add  eax,edi  
        ret  
      .endif  
    .endif  
    add  edx,sizeof IMAGE\_SECTION\_HEADER  
    dec  ecx  
  .endw  
  assume  edx:nothing  
  assume  esi:nothing  
  mov  eax,FALSE  
  ret  
RVA2Offset  endp  
  
Offset2RVA  proc  uses ebx ecx edx edi esi @dwOffset:dword  
  mov  esi,lpFileHeader  
  assume  esi:ptr IMAGE\_DOS\_HEADER  
  add  esi,\[esi\].e\_lfanew  
  assume  esi:ptr IMAGE\_NT\_HEADERS  
  mov  edi,@dwOffset  
  mov  edx,esi  
  add  edx,sizeof IMAGE\_NT\_HEADERS  
  mov  cx,\[esi\].FileHeader.NumberOfSections  
  movzx  ecx,cx  
  assume  edx:ptr IMAGE\_SECTION\_HEADER  
  .while  ecx > 0  
    .if  edi >= \[edx\].PointerToRawData  
      mov  eax,\[edx\].PointerToRawData  
      add  eax,\[edx\].SizeOfRawData  
      .if  edi < eax  
        mov  eax,\[edx\].PointerToRawData  
        sub  edi,eax  
        mov  eax,\[edx\].VirtualAddress  
        add  eax,edi  
        ret  
      .endif  
    .endif  
    add edx,sizeof IMAGE\_SECTION\_HEADER  
    dec ecx  
  .endw  
  assume  edx:nothing  
  assume  esi:nothing  
  mov  eax,FALSE  
  ret  
Offset2RVA  endp  
  
GetImportsInfo  proc  uses ebx esi edi @lpImportsInfo:dword  
  local  @dwIATMin:dword  
  local  @dwIATMax:dword  
    
  mov  esi,lpPeHeader  
  assume  esi:ptr IMAGE\_NT\_HEADERS  
  lea  eax,\[esi\].OptionalHeader.DataDirectory\\  
      \[IMAGE\_DIRECTORY\_ENTRY\_IMPORT\*sizeof IMAGE\_DATA\_DIRECTORY\]  
  mov  esi,eax  
  assume  esi:ptr IMAGE\_DATA\_DIRECTORY  
  mov  edi,@lpImportsInfo  
  assume  edi:ptr IMPORTS\_INFO  
  mov  eax,\[esi\].VirtualAddress  
  mov  \[edi\].dwImportDirRVA,eax  
  invoke  RVA2Offset,eax  
  add  eax,lpFileHeader  
  mov  \[edi\].lpImportDir,eax  
  push  \[esi\].isize  
  pop  \[edi\].dwImportDirSize  
    
  xor  eax,eax  
  mov  @dwIATMax,eax  
  or  eax,7FFFFFFFh  
  mov  @dwIATMin,eax  
    
  mov  eax,\[esi\].VirtualAddress  
  invoke  RVA2Offset,eax  
  add  eax,lpFileHeader  
  mov  esi,eax  
  assume  esi:ptr IMAGE\_IMPORT\_DESCRIPTOR  
  .while  \[esi\].OriginalFirstThunk || \[esi\].TimeDateStamp || \\  
      \[esi\].ForwarderChain || \[esi\].Name1 || \[esi\].FirstThunk  
    .if  \[esi\].OriginalFirstThunk  
      mov  ebx,\[esi\].OriginalFirstThunk  
    .else  
      mov  ebx,\[esi\].FirstThunk  
    .endif  
    invoke  RVA2Offset,ebx  
    mov  ebx,eax  
    add  ebx,lpFileHeader  
    mov  eax,\[esi\].FirstThunk  
    .while  dword ptr \[ebx\]  
      .if  eax < @dwIATMin  
        mov  @dwIATMin,eax  
      .endif  
      .if  eax > @dwIATMax  
        mov  @dwIATMax,eax  
      .endif  
      add  eax,4  
      add  ebx,4  
    .endw  
    add  esi,sizeof IMAGE\_IMPORT\_DESCRIPTOR  
  .endw  
  push  @dwIATMin  
  pop  \[edi\].dwIATDirRVA  
  mov  eax,@dwIATMax  
  sub  eax,@dwIATMin  
  add  eax,4  
  mov  \[edi\].dwIATDirSize,eax  
  invoke  RVA2Offset,@dwIATMin  
  add  eax,lpFileHeader  
  mov  \[edi\].lpIATDir,eax  
  assume  esi:nothing  
  assume  edi:nothing  
  ret  
GetImportsInfo  endp  
  
GetCodeScnInfo  proc  uses ebx ecx edx esi edi @lpCodeScnInfo:dword  
  mov  esi,lpPeHeader  
  assume  esi:ptr IMAGE\_NT\_HEADERS  
  mov  ebx,\[esi\].OptionalHeader.AddressOfEntryPoint  
    
  mov  edx,esi  
  add  edx,sizeof IMAGE\_NT\_HEADERS  
  mov  cx,\[esi\].FileHeader.NumberOfSections  
  movzx  ecx,cx  
  assume  edx:ptr IMAGE\_SECTION\_HEADER  
  .while  ecx > 0  
    .if  ebx >= \[edx\].VirtualAddress  
      mov  eax,\[edx\].VirtualAddress  
      add  eax,\[edx\].SizeOfRawData  
      .if  ebx < eax  
        mov  edi,@lpCodeScnInfo  
        assume  edi:ptr SCN\_INFO  
        push  \[edx\].VirtualAddress  
        pop  \[edi\].dwVirtualAddress  
        push  \[edx\].Misc.VirtualSize  
        pop  \[edi\].dwVirtualSize  
        push  \[edx\].SizeOfRawData  
        pop  \[edi\].dwRawSize  
        push  \[edx\].Characteristics  
        pop  \[edi\].dwCharacteristics  
        mov  eax,\[edx\].PointerToRawData  
        mov  \[edi\].dwRawAddress,eax  
        add  eax,lpFileHeader  
        mov  \[edi\].lpScn,eax  
        mov  esi,edx  
        mov  ecx,8  
        rep  movsb  
        mov  eax,TRUE  
        ret  
      .endif  
    .endif  
    add  edx,sizeof IMAGE\_SECTION\_HEADER  
    dec  ecx  
  .endw  
  assume  esi:nothing  
  assume  edi:nothing  
  assume  edx:nothing  
  mov  eax,FALSE  
  ret  
GetCodeScnInfo  endp  
  
; 偷取原始入口点的部分指令，经变形后存储在指定区域，清除原来的指令，返回值为结束偷取后第一条指令的 RVA  
ObfuscateOEP  proc  uses ecx esi edi @lpSubstitution:dword  
  mov  esi,lpPeHeader  
  mov  edi,@lpSubstitution  
  assume  esi:ptr IMAGE\_NT\_HEADERS  
  mov  eax,\[esi\].OptionalHeader.AddressOfEntryPoint  
  assume  esi:nothing  
  invoke  RVA2Offset,eax  
  add  eax,lpFileHeader  
  mov  esi,eax  
  mov  eax,\[esi\]  
  .while  al != 0E8h && ax != 15FFh && ax != 25FFh &&\\  
    al != 74h && al != 75h && al != 76h && al != 77h && al != 0E9h && al != 0EBh  
    .if  al == 55h    ;push ebp  
      mov  eax,4EC8303h  ;sub esp,4  //3 bytes  
      stosd  
      mov  eax,242C8903h  ;mov \[esp\],ebp  //3 bytes  
      stosd  
      xor  al,al  
      mov  \[esi\],al  
      inc  esi  
    .elseif  ax == 0EC8Bh    ;mov ebp,esp  
      mov  ax,5401h  ;push esp  //1 byte  
      stosw  
      mov  ax,5D01h  ;pop ebp  //1 byte  
      stosw  
      xor  ax,ax  
      mov  \[esi\],ax  
      add  esi,2  
    .elseif  al == 6Ah    ;push byte  //2 bytes  
      mov  byte ptr \[edi\],2  
      inc  edi  
      movsw  
      xor  ax,ax  
      mov  \[esi-2\],ax  
    .elseif  al == 68h    ;push dword  
      mov  ax,5001h  ;push eax  //1 byte  
      stosw  
      mov  eax,2404C707h  ;mov dword ptr \[esp\],XXXXXXXXX  //7 bytes  
      stosd  
      xor  eax,eax  
      mov  \[esi\],al  
      inc  esi  
      movsd  
      sub  esi,4  
      mov  \[esi\],eax  
      add  esi,4  
    .elseif  al == 64h  
      inc  esi  
      mov  ax,\[esi\]  
      .if  ax == 0A1h  ;mov eax,fs:\[0\]  
        mov  byte ptr \[edi\],2  
        inc  edi  
        mov  ax,0C033h  ;xor eax,eax  //2 bytes  
        stosw  
        mov  eax,30FF6403h  ;push dword ptr fs:\[eax\]  //3 bytes  
        stosd  
        mov  ax,5801h  ;pop eax  //1 byte  
        stosw  
        inc  esi  
        xor  eax,eax  
        mov  \[esi-2\],ax  
        mov  \[esi\],eax  
        add  esi,4  
      .elseif  ax == 2589h  ;mov fs:\[0\],esp  
        mov  ax,5401h  ;push esp  //1 byte  
        stosw  
        mov  eax,58F6407h  ;pop dword ptr fs:\[0\]  //7 bytes  
        stosd  
        xor  eax,eax  
        stosd  
        add  esi,2  
        mov  \[esi-4\],eax  
        mov  \[esi\],eax  
        add  esi,4  
      .endif  
    .elseif  al == 50h  ;push eax  
      mov  eax,4EC8303h  ;sub esp,4  //3 bytes  
      stosd  
      mov  eax,24048903h  ;mov \[esp\],eax  //3 bytes  
      stosd  
      xor  al,al  
      mov  \[esi\],al  
      inc  esi  
    .elseif  al == 53h  ;push ebx  
      mov  eax,4EC8303h  ;sub esp,4  //3 bytes  
      stosd  
      mov  eax,241C8903h  ;mov \[esp\],ebx  //3 bytes  
      stosd  
      xor  al,al  
      mov  \[esi\],al  
      inc  esi  
    .elseif  al == 56h  ;push esi  
      mov  eax,4EC8303h  ;sub esp,4  //3 bytes  
      stosd  
      mov  eax,24348903h  ;mov \[esp\],edi  //3 bytes  
      stosd  
      xor  al,al  
      mov  \[esi\],al  
      inc  esi  
    .elseif  al == 57h  ;push edi  
      mov  eax,4EC8303h  ;sub esp,4  //3 bytes  
      stosd  
      mov  eax,243C8903h  ;mov \[esp\],esi  //3 bytes  
      stosd  
      xor  al,al  
      mov  \[esi\],al  
      inc  esi  
    .elseif  ax == 0C483h  ;add esp,XX  
      mov  eax,\[esi\]  
      shl  eax,8  
      mov  al,3  
      stosd  
      dec  esi  
      xor  eax,eax  
      mov  \[esi\],eax  
      add  esi,4  
    .elseif  ax == 0EC83h  ;sub esp,XX  
      mov  ax,5101h  ;push ecx  //1 byte  
      stosw  
      lodsd  
      shl  eax,8    ;sub esp,(XX-4)  //3 bytes  
      mov  al,3  
      sub  eax,4000000h  
      stosd  
      sub  esi,5  
      xor  eax,eax  
      mov  \[esi\],eax  
      add  esi,4  
    .elseif  ax == 6589h  ;mov \[ebp+XX\],esp  
      mov  eax,\[esi\]  
      shl  eax,8  
      mov  al,3  
      stosd  
      dec  esi  
      xor  eax,eax  
      mov  \[esi\],eax  
      add  esi,4  
    .elseif  ax == 45C7h  ;mov \[ebp+XX\],XXXXXXXX  
      mov  ax,6805h  ;push XXXXXXXX  //5 bytes  
      stosw  
      add  esi,3  
      movsd  
      sub  esi,5  
      lodsb  
      shl  eax,16  
      mov  ax,458Fh  ;pop \[ebp+XX\]  //3 bytes  
      shl  eax,8  
      mov  al,3  
      stosd  
      xor  eax,eax  
      mov  \[esi-4\],eax  
      mov  \[esi\],eax  
      add  esi,4  
    .elseif  ax == 0DB33h  ;xor ebx,ebx  
      mov  byte ptr \[edi\],2  
      mov  ax,0DB2Bh  
      stosw  
      xor  ax,ax  
      mov  \[esi\],ax  
      add  esi,2  
    .elseif  ax == 5D89h  ;mov \[ebp+XX\],ebx  
      mov  eax,\[esi\]  
      shl  eax,8  
      mov  al,3  
      stosd  
      dec  esi  
      xor  eax,eax  
      mov  \[esi\],eax  
      add  esi,4  
    .else  
      .break  
    .endif  
    mov  eax,\[esi\]  
  .endw  
  xor  al,al  
  stosb  
  sub  esi,lpFileHeader  
  invoke  Offset2RVA,esi  
  ret  
ObfuscateOEP  endp  
  
; 更新入口点 RVA，返回值为原入口点 RVA  
UpdateEntryPoint  proc  uses esi @dwNewEntryPoint:dword  ;Return OEP  
  mov  esi,lpPeHeader  
  assume  esi:ptr IMAGE\_NT\_HEADERS  
  mov  eax,\[esi\].OptionalHeader.AddressOfEntryPoint  
  push  @dwNewEntryPoint  
  pop  \[esi\].OptionalHeader.AddressOfEntryPoint  
  assume  esi:nothing  
  ret  
UpdateEntryPoint  endp  
  
; 将输入双子按磁盘文件对齐值对齐  
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
  
; 将输入双子按区块对齐值对齐  
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
  
; 增加一个区块的块头信息，可以指定区块名称、大小和属性，同时更新 NumberOfSections 字段和 ImageSize 字段，调用了 GetScnInfo  
AddSection  proc  uses ebx ecx edx esi edi @szScnName:dword,@dwSize:dword,@dwCharacteristics:dword  
  local  @stNewScnInfo:SCN\_INFO  
  local  @dwImageSize  
    
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
    invoke  GetScnInfo,0,addr @stNewScnInfo  
    push  @dwSize  
    pop  \[esi\].Misc.VirtualSize  
    mov  eax,@stNewScnInfo.dwVirtualSize  
    invoke  AlignScn,eax  
    add  eax,@stNewScnInfo.dwVirtualAddress  
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
    pop  \[esi\].OptionalHeader.SizeOfImage  
    mov  eax,TRUE  
  .else  
    mov  eax,FALSE  
  .endif  
  assume  esi:nothing  
  assume  edi:nothing  
  ret  
AddSection  endp  
  
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
  add  eax,lpFileHeader  
  mov  \[edi\].lpScn,eax  
  push  \[esi\].Characteristics  
  pop  \[edi\].dwCharacteristics  
  mov  ecx,8  
  rep  movsb  
  .if  @nScn == 0  
    mov  eax,FALSE    ;got the last section  
  .else  
    mov  eax,TRUE  
  .endif  
  assume  esi:nothing  
  assume  edi:nothing  
  ret  
GetScnInfo  endp  
  
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
  
; 合并区块，输入参数包括起始区块序号（从 1 开始）和终止区块序号，并可以指定合并后的区块名称  
MergeScns  proc  uses ebx ecx edx esi edi @nStart:dword,@nEnd:dword,@szScnName:dword  
  mov  esi,lpPeHeader  
  assume  esi:ptr IMAGE\_NT\_HEADERS  
  xor  ecx,ecx  
  mov  cx,\[esi\].FileHeader.NumberOfSections  
  mov  ebx,@nStart  
  .if  ecx > ebx || ecx > @nEnd || ebx >= @nEnd  
    mov  eax,FALSE  
    ret  
  .endif  
    
  xor  eax,eax  
  mov  ax,\[esi\].FileHeader.NumberOfSections  
  add  eax,@nStart  
  sub  eax,@nEnd  
  mov  \[esi\].FileHeader.NumberOfSections,ax  
  add  esi,sizeof IMAGE\_NT\_HEADERS  
  assume  esi:ptr IMAGE\_SECTION\_HEADER  
  mov  eax,sizeof IMAGE\_SECTION\_HEADER  
  mov  ecx,ebx  
  dec  ecx  
  mul  ecx  
  add  esi,eax  
  mov  edi,esi  
  add  edi,sizeof IMAGE\_SECTION\_HEADER  
  assume  edi:ptr IMAGE\_SECTION\_HEADER  
  .while  ebx < @nEnd  
    mov  eax,\[edi\].VirtualAddress  
    add  eax,\[edi\].Misc.VirtualSize  
    sub  eax,\[esi\].VirtualAddress  
    mov  \[esi\].Misc.VirtualSize,eax  
    mov  eax,\[edi\].SizeOfRawData  
    add  \[esi\].SizeOfRawData,eax  
    mov  eax,\[edi\].Characteristics  
    or  \[esi\].Characteristics,eax  
    add  edi,sizeof IMAGE\_SECTION\_HEADER  
    inc  ebx  
  .endw  
  mov  ebx,@nEnd  
  sub  ebx,@nStart  
  push  esi  
  add  esi,sizeof IMAGE\_SECTION\_HEADER  
  .while  ebx > 0  
    xchg  esi,edi  
    mov  ecx,sizeof IMAGE\_SECTION\_HEADER  
    rep  movsb  
    xor  al,al  
    xchg  esi,edi  
    sub  edi,sizeof IMAGE\_SECTION\_HEADER  
    rep  stosb  
    dec  ebx  
  .endw  
  pop  esi  
  mov  edi,@szScnName  
  mov  ecx,8  
  xchg  esi,edi  
  rep  movsb  
  assume  esi:nothing  
  assume  edi:nothing  
  mov  eax,TRUE  
  ret  
MergeScns  endp  
  
; 调用通用对话框打开 PE 文件  
OpenFileDialog  proc  uses ecx edi  
  local  stofn:OPENFILENAME  
    
  xor  al,al  
  lea  edi,stofn  
  mov  ecx,sizeof OPENFILENAME  
  rep  stosb  
  mov  stofn.lStructSize,sizeof OPENFILENAME  
  mov  stofn.lpstrFilter,offset szFilter  
  mov  stofn.lpstrFile,offset szPeFileName  
  mov  stofn.nMaxFile,sizeof szPeFileName  
  mov  stofn.lpstrInitialDir,offset szCurrentDir  
  mov  stofn.lpstrTitle,offset szTitle  
  mov  stofn.Flags,OFN\_FILEMUSTEXIST or OFN\_LONGNAMES or OFN\_PATHMUSTEXIST  
  invoke  GetOpenFileName,addr stofn  
  .if  eax == 0  
    mov  eax,FALSE  
  .else  
    mov  eax,TRUE  
  .endif  
  ret  
OpenFileDialog  endp  
  
; 输出文件命名为 xxx\_.exe，并创建该文件，返回值为该文件的句柄  
CreateOutFile  proc  uses ecx edi  
  lea  edi,szPeFileName  
  xor  ecx,ecx  
  dec  ecx  
  mov  al,'.'  
  repnz  scasb  
  dec  edi  
  mov  al,'\_'  
  stosb  
  mov  eax,'exe.'  
  stosd  
  xor  al,al  
  stosb  
  invoke  CreateFile,offset szPeFileName,GENERIC\_READ or GENERIC\_WRITE,\\  
          FILE\_SHARE\_READ,\\  
          NULL,CREATE\_ALWAYS,\\  
          FILE\_ATTRIBUTE\_NORMAL,NULL  
  .if  eax == INVALID\_HANDLE\_VALUE  
    mov  eax,FALSE  
  .endif  
  ret  
CreateOutFile  endp  
  
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
  
; 当区块被压缩以后，其参数要作相应的调整  
UpdatePackedScnHeader  proc  uses ecx edx esi edi @nScn:dword,@dwSize:dword,@dwCharacteristics:dword  
  mov  esi,lpPeHeader  
  assume  esi:ptr IMAGE\_NT\_HEADERS  
  xor  ecx,ecx  
  mov  cx,\[esi\].FileHeader.NumberOfSections  
  .if  @nScn > ecx  
    mov  eax,FALSE  
    ret  
  .endif  
  add  esi,sizeof IMAGE\_NT\_HEADERS  
  assume  esi:ptr IMAGE\_SECTION\_HEADER  
  .if  @nScn == 1  
    push  @dwSize  
    pop  \[esi\].Misc.VirtualSize  
    invoke  AlignFile,@dwSize  
    mov  \[esi\].SizeOfRawData,eax  
    push  @dwCharacteristics  
    pop  \[esi\].Characteristics  
  .else  
    mov  ecx,@nScn  
    dec  ecx  
    mov  eax,sizeof IMAGE\_SECTION\_HEADER  
    mul  ecx  
    add  esi,eax  
    mov  edi,esi  
    sub  esi,sizeof IMAGE\_SECTION\_HEADER  
    assume  edi:ptr IMAGE\_SECTION\_HEADER  
    push  @dwSize  
    pop  \[edi\].Misc.VirtualSize  
    invoke  AlignScn,@dwSize  
    add  eax,\[esi\].VirtualAddress  
    mov  \[edi\].VirtualAddress,eax  
    invoke  AlignFile,@dwSize  
    mov  \[edi\].SizeOfRawData,eax  
    add  eax,\[esi\].PointerToRawData  
    mov  \[edi\].PointerToRawData,eax  
    push  @dwCharacteristics  
    pop  \[edi\].Characteristics      
  .endif  
  assume  esi:nothing  
  assume  edi:nothing  
  mov  eax,TRUE  
  ret  
UpdatePackedScnHeader  endp  
  
CheckPackingFlag  proc  uses esi @dwFlag:dword  
  mov  esi,lpPeHeader  
  assume  esi:ptr IMAGE\_NT\_HEADERS  
  mov  eax,\[esi\].FileHeader.NumberOfSymbols  
  .if  eax == @dwFlag  
    mov  eax,TRUE  
  .else  
    mov  eax,FALSE  
  .endif  
  assume  esi:nothing  
  ret  
CheckPackingFlag  endp  
  
SetPackingFlag  proc  uses esi @dwFlag:dword  
  mov  esi,lpPeHeader  
  assume  esi:ptr IMAGE\_NT\_HEADERS  
  push  @dwFlag  
  pop  \[esi\].FileHeader.NumberOfSymbols  
  assume  esi:nothing  
  ret  
SetPackingFlag  endp  
  
; 更新输入表 RVA，返回值为原来的 RVA  
UpdateImportRVA  proc  uses ebx esi @NewImportRVA  
  mov  esi,lpPeHeader  
  assume  esi:ptr IMAGE\_NT\_HEADERS  
  mov  eax,@NewImportRVA  
  lea  ebx,\[esi\].OptionalHeader.DataDirectory\\  
      \[IMAGE\_DIRECTORY\_ENTRY\_IMPORT\*sizeof IMAGE\_DATA\_DIRECTORY\]  
  xchg  eax,\[ebx\]  
  assume  esi:nothing  
  ret  
UpdateImportRVA  endp  
  
GetStrLen  proc  uses ecx edi @lpString:dword  
  or  ecx,0FFFFFFFFh  
  xor  al,al  
  mov  edi,@lpString  
  repnz  scasb  
  not  ecx  
  dec  ecx  
  mov  eax,ecx  
  ret  
GetStrLen  endp  
  
; tElock 中提取出来的校验算法修改版,@dwKey 一般可以传入循环计数值  
tEHash  proc  uses ebx ecx edx esi edi @SrcStart:dword,@SrcSize:dword,@dwKey:dword  
  mov  esi,@SrcStart  
  mov  ebx,@SrcSize  
  xor  ecx,ecx  
  lea  eax,\[ecx-1\]  
  mov  edi,@dwKey  
HasheLoop:  
  xor  edx,edx  
  mov  dl,\[esi\]  
  xor  dl,al  
HashiLoop:  
  shr  edx,1  
  jnb  @F  
  add  edx,edi  
@@:  
  inc  ecx  
  and  cl,7  
  jnz  HashiLoop  
  shr  eax,8  
  xor  eax,edx  
  inc  esi  
  dec  ebx  
  jg  HasheLoop  
  not  eax  
  ret  
tEHash  endp  
  
crc32  proc  @lData:dword,@ptrData:dword  
  push  esi  
  push  ecx  
  push  edx  
  
  mov  esi, @ptrData  
  xor  edx, edx  
  or  eax, -1  
  mov  ecx, @lData  
  
crc32\_loop:  
  mov  dl, byte ptr \[esi\]  
  xor  dl, al  
  shr  eax, 8  
  xor  eax, dword ptr \[crc32\_table + 4\*edx\]  
  inc  esi  
  dec  ecx  
  jnz  crc32\_loop  
  
  not  eax  
  
  pop  edx  
  pop  ecx  
  pop  esi  
  ret  
  
crc32\_table  dd  000000000h, 077073096h, 0EE0E612Ch, 0990951BAh, 0076DC419h  
    dd  0706AF48Fh, 0E963A535h, 09E6495A3h, 00EDB8832h, 079DCB8A4h  
    dd  0E0D5E91Eh, 097D2D988h, 009B64C2Bh, 07EB17CBDh, 0E7B82D07h  
    dd  090BF1D91h, 01DB71064h, 06AB020F2h, 0F3B97148h, 084BE41DEh  
    dd  01ADAD47Dh, 06DDDE4EBh, 0F4D4B551h, 083D385C7h, 0136C9856h  
    dd  0646BA8C0h, 0FD62F97Ah, 08A65C9ECh, 014015C4Fh, 063066CD9h  
    dd  0FA0F3D63h, 08D080DF5h, 03B6E20C8h, 04C69105Eh, 0D56041E4h  
    dd  0A2677172h, 03C03E4D1h, 04B04D447h, 0D20D85FDh, 0A50AB56Bh  
    dd  035B5A8FAh, 042B2986Ch, 0DBBBC9D6h, 0ACBCF940h, 032D86CE3h  
    dd  045DF5C75h, 0DCD60DCFh, 0ABD13D59h, 026D930ACh, 051DE003Ah  
    dd  0C8D75180h, 0BFD06116h, 021B4F4B5h, 056B3C423h, 0CFBA9599h  
    dd  0B8BDA50Fh, 02802B89Eh, 05F058808h, 0C60CD9B2h, 0B10BE924h  
    dd  02F6F7C87h, 058684C11h, 0C1611DABh, 0B6662D3Dh, 076DC4190h  
    dd  001DB7106h, 098D220BCh, 0EFD5102Ah, 071B18589h, 006B6B51Fh  
    dd  09FBFE4A5h, 0E8B8D433h, 07807C9A2h, 00F00F934h, 09609A88Eh  
    dd  0E10E9818h, 07F6A0DBBh, 0086D3D2Dh, 091646C97h, 0E6635C01h  
    dd  06B6B51F4h, 01C6C6162h, 0856530D8h, 0F262004Eh, 06C0695EDh  
    dd  01B01A57Bh, 08208F4C1h, 0F50FC457h, 065B0D9C6h, 012B7E950h  
    dd  08BBEB8EAh, 0FCB9887Ch, 062DD1DDFh, 015DA2D49h, 08CD37CF3h  
    dd  0FBD44C65h, 04DB26158h, 03AB551CEh, 0A3BC0074h, 0D4BB30E2h  
    dd  04ADFA541h, 03DD895D7h, 0A4D1C46Dh, 0D3D6F4FBh, 04369E96Ah  
    dd  0346ED9FCh, 0AD678846h, 0DA60B8D0h, 044042D73h, 033031DE5h  
    dd  0AA0A4C5Fh, 0DD0D7CC9h, 05005713Ch, 0270241AAh, 0BE0B1010h  
    dd  0C90C2086h, 05768B525h, 0206F85B3h, 0B966D409h, 0CE61E49Fh  
    dd  05EDEF90Eh, 029D9C998h, 0B0D09822h, 0C7D7A8B4h, 059B33D17h  
    dd  02EB40D81h, 0B7BD5C3Bh, 0C0BA6CADh, 0EDB88320h, 09ABFB3B6h  
    dd  003B6E20Ch, 074B1D29Ah, 0EAD54739h, 09DD277AFh, 004DB2615h  
    dd  073DC1683h, 0E3630B12h, 094643B84h, 00D6D6A3Eh, 07A6A5AA8h  
    dd  0E40ECF0Bh, 09309FF9Dh, 00A00AE27h, 07D079EB1h, 0F00F9344h  
    dd  08708A3D2h, 01E01F268h, 06906C2FEh, 0F762575Dh, 0806567CBh  
    dd  0196C3671h, 06E6B06E7h, 0FED41B76h, 089D32BE0h, 010DA7A5Ah  
    dd  067DD4ACCh, 0F9B9DF6Fh, 08EBEEFF9h, 017B7BE43h, 060B08ED5h  
    dd  0D6D6A3E8h, 0A1D1937Eh, 038D8C2C4h, 04FDFF252h, 0D1BB67F1h  
    dd  0A6BC5767h, 03FB506DDh, 048B2364Bh, 0D80D2BDAh, 0AF0A1B4Ch  
    dd  036034AF6h, 041047A60h, 0DF60EFC3h, 0A867DF55h, 0316E8EEFh  
    dd  04669BE79h, 0CB61B38Ch, 0BC66831Ah, 0256FD2A0h, 05268E236h  
    dd  0CC0C7795h, 0BB0B4703h, 0220216B9h, 05505262Fh, 0C5BA3BBEh  
    dd  0B2BD0B28h, 02BB45A92h, 05CB36A04h, 0C2D7FFA7h, 0B5D0CF31h  
    dd  02CD99E8Bh, 05BDEAE1Dh, 09B64C2B0h, 0EC63F226h, 0756AA39Ch  
    dd  0026D930Ah, 09C0906A9h, 0EB0E363Fh, 072076785h, 005005713h  
    dd  095BF4A82h, 0E2B87A14h, 07BB12BAEh, 00CB61B38h, 092D28E9Bh  
    dd  0E5D5BE0Dh, 07CDCEFB7h, 00BDBDF21h, 086D3D2D4h, 0F1D4E242h  
    dd  068DDB3F8h, 01FDA836Eh, 081BE16CDh, 0F6B9265Bh, 06FB077E1h  
    dd  018B74777h, 088085AE6h, 0FF0F6A70h, 066063BCAh, 011010B5Ch  
    dd  08F659EFFh, 0F862AE69h, 0616BFFD3h, 0166CCF45h, 0A00AE278h  
    dd  0D70DD2EEh, 04E048354h, 03903B3C2h, 0A7672661h, 0D06016F7h  
    dd  04969474Dh, 03E6E77DBh, 0AED16A4Ah, 0D9D65ADCh, 040DF0B66h  
    dd  037D83BF0h, 0A9BCAE53h, 0DEBB9EC5h, 047B2CF7Fh, 030B5FFE9h  
    dd  0BDBDF21Ch, 0CABAC28Ah, 053B39330h, 024B4A3A6h, 0BAD03605h  
    dd  0CDD70693h, 054DE5729h, 023D967BFh, 0B3667A2Eh, 0C4614AB8h  
    dd  05D681B02h, 02A6F2B94h, 0B40BBE37h, 0C30C8EA1h, 05A05DF1Bh  
    dd  02D02EF8Dh  
  
crc32    endp  

 4dd466d0e1e0cc123afbe504da2462ca