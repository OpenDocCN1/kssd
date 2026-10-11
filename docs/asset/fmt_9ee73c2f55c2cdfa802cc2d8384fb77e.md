
```
  .if   edx != 0  

    inc   eax  

  .endif  

  mul   ecx  

  ret  

AlignFile  endp  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

; 将输入双字按区块对齐值对齐  

AlignScn  proc  uses ecx edx esi @OrgSize:dword  

  mov  esi,lpPeHeader  

  assume  esi:ptr IMAGE_NT_HEADERS  

  mov  ecx,[esi].OptionalHeader.SectionAlignment  

  assume  esi:nothing  

  mov  eax,@OrgSize  

  cdq  

  div  ecx  

  .if  edx != 0  

    inc  eax  

  .endif  

  mul  ecx  

  ret  

AlignScn  endp  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

; 取得指定区块的信息，0 表示最后一个区块，同时如果输入区块序号大于区块数，也取得最后一个区块的信息  

GetScnInfo  proc  uses ecx edx esi edi @nScn:dword,@lpScnInfo:dword   


  mov  esi,lpPeHeader  

  assume  esi:ptr IMAGE_NT_HEADERS  

  xor  ecx,ecx  

  mov  cx,[esi].FileHeader.NumberOfSections  

  add  esi,sizeof IMAGE_NT_HEADERS  

  .if  @nScn == 0 || @nScn > ecx  

    dec  ecx  

    mov  @nScn,0  

  .else  

    mov  ecx,@nScn  

    dec  ecx  

  .endif  

  mov  eax,sizeof IMAGE_SECTION_HEADER  

  mul  ecx  

  add  esi,eax  

  assume  esi:ptr IMAGE_SECTION_HEADER  

  mov  edi,@lpScnInfo  

  assume  edi:ptr SCN_INFO  

  push  [esi].Misc.VirtualSize  

  pop  [edi].dwVirtualSize  

  push  [esi].VirtualAddress  

  pop  [edi].dwVirtualAddress  

  push  [esi].SizeOfRawData  

  pop  [edi].dwRawSize  

  mov  eax,[esi].PointerToRawData  

  mov  [edi].dwRawAddress,eax  

  push  [esi].Characteristics  

  pop  [edi].dwCharacteristics  

  .if  @nScn == 0  

    mov  eax,FALSE    

  .else  

    mov  eax,TRUE  

  .endif  

  assume  esi:nothing  

  assume  edi:nothing  

  ret  

GetScnInfo  endp  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

; 增加一个区块的块头信息，可以指定区块名称、大小和属性，同时更新 NumberOfSections 字段和 ImageSize 字段  

AddSection  proc  uses ebx ecx edx esi edi @szScnName:dword,@dwSize:dword,@dwCharacteristics:dword  

  local  @stNewScnInfo:SCN_INFO  

  local  @dwImageSize  

  xor eax,eax  

  mov  esi,lpPeHeader  

  assume  esi:ptr IMAGE_NT_HEADERS  

  lea  ebx,[esi].OptionalHeader.DataDirectory\  

      [IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT*sizeof IMAGE_DATA_DIRECTORY]  

  mov  edi,[ebx]  

  .if  edi != 0  

    add  edi,lpFileHeader  

    mov  ecx,[ebx+4]  

    rep  stosb  

    mov  edi,ebx  

    mov  ecx,sizeof IMAGE_DATA_DIRECTORY  

    rep  stosb  

  .endif  

  xor  ecx,ecx  

  mov  cx,[esi].FileHeader.NumberOfSections  

  add  esi,sizeof IMAGE_NT_HEADERS  

  assume  esi:ptr IMAGE_SECTION_HEADER  

  mov  edi,[esi].PointerToRawData  

  add  edi,lpFileHeader  

  mov  eax,sizeof IMAGE_SECTION_HEADER  

  mul  ecx  

  add  esi,eax  

  add  esi,sizeof IMAGE_SECTION_HEADER  

  .if  esi <= edi  

    sub  esi,sizeof IMAGE_SECTION_HEADER  

    invoke  GetScnInfo,0, addr @stNewScnInfo  

    push  @dwSize  

    pop  [esi].Misc.VirtualSize  

    mov  eax,@stNewScnInfo.dwVirtualSize  

    ;mov  eax,@dwSize  

    invoke  AlignScn,eax  

    add  eax,@stNewScnInfo.dwVirtualAddress  

    mov dwNewEntry,eax  

    mov  [esi].VirtualAddress,eax  

    add  eax,@dwSize  

    mov  @dwImageSize,eax  

    invoke  AlignFile,@dwSize  

    mov  [esi].SizeOfRawData,eax  

    mov  eax,@stNewScnInfo.dwRawSize  

    invoke  AlignFile,eax  

    add  eax,@stNewScnInfo.dwRawAddress  

    mov  [esi].PointerToRawData,eax  

    push  @dwCharacteristics  

    pop  [esi].Characteristics  

    mov  edi,@szScnName  

    xchg  esi,edi  

    mov  ecx,8  

    rep  movsb  

    mov  esi,lpPeHeader  

    assume  esi:ptr IMAGE_NT_HEADERS  

    inc  [esi].FileHeader.NumberOfSections  

    push  @dwImageSize  

    ;push   0C000h  

    pop  [esi].OptionalHeader.SizeOfImage  

    mov  eax,TRUE  

  .else  

    mov  eax,FALSE  

  .endif  

  assume  esi:nothing  

  assume  edi:nothing  

  ret  

AddSection  endp  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

; 取得附加数据的信息  

GetExtraDataInfo  proc  uses ecx edx esi @lpExtraDataInfo:dword  

  mov  esi,lpPeHeader  

  assume  esi:ptr IMAGE_NT_HEADERS  

  xor  ecx,ecx  

  mov  cx,[esi].FileHeader.NumberOfSections  

  add  esi,sizeof IMAGE_NT_HEADERS  

  mov  eax,sizeof IMAGE_SECTION_HEADER  

  dec  ecx  

  mul  ecx  

  add  esi,eax  

  assume  esi:ptr IMAGE_SECTION_HEADER  

  mov  edx,[esi].PointerToRawData  

  add  edx,[esi].SizeOfRawData  

  push  edx  

  invoke  GetFileSize,hInFile,NULL  

  pop  edx  

  mov  esi,@lpExtraDataInfo  

  assume  esi:ptr EXTRADATA_INFO  

  .if  eax > edx  

    mov  [esi].dwRawAddress,edx  

    sub  eax,edx  

    mov  [esi].dwSize,eax  

    add  edx,lpFileHeader  

    mov  [esi].lpExtraData,edx  

    mov  eax,TRUE  

  .else  

    mov  [esi].dwRawAddress,0  

    mov  eax,FALSE  

  .endif  

  assume  esi:nothing  

  ret  

GetExtraDataInfo  endp  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

; 输出文件命名为 xxx@.exe，并创建该文件，返回值为该文件的句柄  

CreateOutFile  proc  uses ecx edi  

  lea  edi,szDesFileName  

  xor  ecx,ecx  

  dec  ecx  

  mov  al,'.'  

  repnz  scasb  

  dec  edi  

  mov  al,'@'  

  stosb  

  mov  eax,'exe.'  

  stosd  

  xor  al,al  

  stosb  

  invoke  CreateFile,offset szDesFileName,GENERIC_READ or GENERIC_WRITE,\  

          FILE_SHARE_READ,\  

          NULL,CREATE_ALWAYS,\  

          FILE_ATTRIBUTE_NORMAL,NULL  

  .if  eax == INVALID_HANDLE_VALUE  

    mov  eax,FALSE  

   .else  

   mov  hOutFile,eax  

  .endif  

  ret  

CreateOutFile  endp  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

; 在新文件中写入新增区块并补上附加数据  

AppendScnCode  proc  uses ebx esi edi @lpOrgLastScnInfo:dword,@lpExtraDataInfo:dword,\  

          @lpNewScnData:dword,@dwNewScnSize:dword  

  local  lpBytesWritten:dword  

  mov  esi,@lpOrgLastScnInfo  

  assume  esi:ptr SCN_INFO  

  mov  ebx,[esi].dwRawSize  

  add  ebx,[esi].dwRawAddress  

  invoke  WriteFile,hOutFile,lpFileHeader,ebx,addr lpBytesWritten,NULL  

  invoke  AlignFile,@dwNewScnSize  

  mov  ebx,eax  

  invoke  WriteFile,hOutFile,@lpNewScnData,ebx,addr lpBytesWritten,NULL  

  mov  edi,@lpExtraDataInfo  

  assume  edi:ptr EXTRADATA_INFO  

  .if  [edi].dwRawAddress != 0  

    invoke  WriteFile,hOutFile,[edi].lpExtraData,[edi].dwSize,addr lpBytesWritten,NULL  

  .endif  

  invoke  SetEndOfFile,hOutFile  

  assume  esi:nothing  

  assume  edi:nothing  

  ret  

AppendScnCode  endp  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

_windowcenter proc uses ebx edi esi hWnd  

local @DlgHeight,@DlgWidth,@x,@y  

 local @DlgRect:RECT,@DesktopRect:RECT  

 local @hwindow  

 invoke   GetWindowRect,hWnd,addr @DlgRect  

     invoke   GetDesktopWindow  

     mov @hwindow,eax  

     invoke   GetWindowRect,@hwindow,addr @DesktopRect  

     xor eax,eax  

     mov eax,@DlgRect.bottom  

     sub eax,@DlgRect.top  

     mov @DlgHeight,eax  

     xor eax,eax  

     mov eax,@DlgRect.right  

     sub eax,@DlgRect.left  

     mov @DlgWidth,eax  

     xor eax,eax  

     mov eax,@DesktopRect.left  

     add eax,@DesktopRect.right  

     sub eax,@DlgWidth  

     sar eax,1  

     mov @x,eax  

     xor eax,eax  

     mov eax,@DesktopRect.top  

     add eax,@DesktopRect.bottom  

     sub eax,@DlgHeight  

     sar eax,1  

     mov @y,eax  

     invoke MoveWindow,hWnd,@x,@y,@DlgWidth,@DlgHeight,FALSE  

     mov eax,TRUE  

     ret  

     _windowcenter endp  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

_ProcDlgMain  proc  uses ebx edi esi hWnd,wMsg,wParam,lParam  

    mov  eax,wMsg  

    .if  eax == WM_CLOSE  

      invoke  EndDialog,hWnd,NULL  

    .elseif  eax == WM_INITDIALOG  

      invoke  LoadIcon,hInstance,ICO_ICON  

      invoke  SendMessage,hWnd,WM_SETICON,ICON_BIG,eax  

      invoke   _windowcenter,hWnd  

      push hWnd  

      pop  hWinMain  

    .elseif  eax == WM_COMMAND  

      mov  eax,wParam  

      .if  ax == IDC_BExit  

        invoke CloseFileMap,lpFileHeader,hInFile  

            invoke CloseFileMap,lpDataHead,hInDataFile  
```
