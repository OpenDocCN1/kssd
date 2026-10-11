
```asm
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

;把外壳写入文件,把修改的文件头写入文件

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

mov ecx,ShellSize
invoke SetFilePointer,hFile, ecx,0,FILE_END
invoke SetEndOfFile,hFile
mov eax,FileSize
invoke SetFilePointer,hFile,eax,0,FILE_BEGIN
invoke WriteFile,hFile,peshell,ecx,addr ReadNumber,0
invoke SetFilePointer,hFile,0,0,FILE_BEGIN
invoke WriteFile,hFile,HeaderBuffer,dword ptr [esi+54h],addr ReadNumber,0
invoke CloseHandle,hFile
invoke VirtualFree,peshell,0,MEM_RELEASE
invoke VirtualFree,HeaderBuffer,0,MEM_RELEASE
ret

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>pe 外壳程序>>>>>>>>>>>>>>>>>>>>>>>>

shellstart:  ;shell 开始
call @@1
ImportTable    dd THUNK_DATA1-ImportTable
       dd 0,0
Import_Name    dd  DllName-ImportTable
Import_FirstThunk dd  THUNK_DATA1-ImportTable
       dd 0,0,0,0,0
THUNK_DATA1     dd  IMPORT_BY_NAME1-ImportTable
THUNK_DATA2     dd  IMPORT_BY_NAME2-ImportTable
THUNK_DATA3     dd  IMPORT_BY_NAME3-ImportTable
               dd 0
DllName         db 'Kernel32.dll',0

IMPORT_BY_NAME1 dw 0
     db 'GetProcAddress',0
IMPORT_BY_NAME2 dw 0
     db 'GetModuleHandleA',0
IMPORT_BY_NAME3  dw 0
             db 'LoadLibraryA',0
ImportTableend:

;>>>>>>>>>>>>>>>>>>>>>>>>>以上为输入表结构>>>>>>>>>>>>>>>>>>>>>>>>>>
;在把壳写入文件时，需要重新定位以上信息>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;ebp 中保存着要加壳文件的 shellstart 的地址（不是相对虚拟地址，而是绝对）
@@1:
    pop ebp;此时 ebp 中为 ImportTable 的地址
    sub ebp,(ImportTable-shellstart)
    mov edi, dword ptr [ebp+(RVA_jiami-shellstart)]
    push 0
    call dword ptr [ebp+(THUNK_DATA2-shellstart)]
    add edi,eax
    mov dword ptr [ebp+(handle-shellstart)],eax
    mov ecx, dword ptr [ebp+(SIZE_jiami-shellstart)]
jiemi:
   xor byte ptr [edi],13h
   inc edi
   loop jiemi
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>初始化输入表>>>>>>>>>>>>>>>>>>>>>>>>>>>>
   mov edi,dword ptr [ebp+(Import-shellstart)]
   push 0
   call dword ptr [ebp+(THUNK_DATA2-shellstart)]
   mov dword ptr [ebp+(handle-shellstart)],eax
   add edi,dword ptr [ebp+(handle-shellstart)]

GetDllAddr:
   mov esi,dword ptr [edi+0ch];esi--->dllname 的 rva
   .if esi==0
      jmp DllFinish
   .endif
   add esi,dword ptr [ebp+(handle-shellstart)]
   push esi
   call dword ptr [ebp+(THUNK_DATA2-shellstart)];获取 dll 句柄
   .if eax==0
      push esi
      call dword ptr [ebp+(THUNK_DATA3-shellstart)]
   .endif
    mov esi,eax
    mov edx,dword ptr [edi]
    .if edx==0
      mov edx,dword ptr [edi+10h]
    .endif
    add edx,dword ptr [ebp+(handle-shellstart)]
    mov ebx,dword ptr [edi+10h]
    add ebx,dword ptr [ebp+(handle-shellstart)]
GetFuncAddr:
   mov eax,dword ptr [edx] ;IMAGE_THUNK_DATA
   cmp eax,0
   jz FunctionFinish
   push ebx
   push edx
   cdq
   .if edx==0
     add eax,2h
     add eax,dword ptr [ebp+(handle-shellstart)]
   .else
     and eax,7fffffffh
   .endif
   push eax
   push esi;dll 句柄
   call dword ptr [ebp+(THUNK_DATA1-shellstart)]
   mov dword ptr [ebx],eax
   pop edx
   pop ebx
   add edx,4h
   add ebx,4h
   jmp GetFuncAddr
FunctionFinish:
   add edi,14h
   jmp GetDllAddr
DllFinish:
   mov eax,dword ptr [ebp+(OEP-shellstart)]
   add eax,dword ptr [ebp+(handle-shellstart)]
   mov ebp,esp
   push ebp
   add esp,1
   inc ecx
   nop
   dec ecx
   pop ebp
   add esp,-1
   nop
   push eax
   retn
   RVA_jiami    DD 0
   SIZE_jiami   DD 0
   Import       DD 0
   handle       DD 0
   OEP          DD 0
shellend:
AddShell endp

_DlgProcMain proc uses ebx edi esi hWnd,wMsg,wParam,lParam
   local @lpfn:OPENFILENAME
   local @dwBytesRead:dword
   invoke RtlZeroMemory,addr @lpfn,sizeof @lpfn
   mov eax,wMsg
   .if eax==WM_CLOSE
     invoke EndDialog,hWnd,NULL
   .elseif eax==WM_INITDIALOG
     invoke LoadString,hInstance,szErr101,addr szErr,sizeof szErr
     invoke LoadString,hInstance,szPass102,addr szPass,sizeof szPass
     invoke LoadString,hInstance,szStop103,addr szStop,sizeof szStop
     invoke LoadIcon,hInstance,ICO_ISCC
      invoke SendMessage,hWnd,WM_SETICON,ICO_ISCC,eax
     invoke GetDlgItem,hWnd,IDC_PASS
     invoke EnableWindow,eax,FALSE
   .elseif eax==WM_COMMAND
     mov eax,wParam
     .if ax==IDOK
       mov @lpfn.lStructSize,sizeof @lpfn
       push hWnd
       pop @lpfn.hWndOwner
       mov @lpfn.lpstrFilter,offset lpstrFilter
       mov @lpfn.lpstrFile,offset szFileName
       mov @lpfn.nMaxFile,MAX_PATH
       mov @lpfn.Flags,OFN_FILEMUSTEXIST or OFN_PATHMUSTEXIST
       invoke GetOpenFileName,addr @lpfn
       .if eax
         invoke SetDlgItemText,hWnd,IDC_EDIT1,addr szFileName
         invoke GetDlgItem,hWnd,IDC_PASS
         invoke EnableWindow,eax,TRUE
       .endif
      .elseif ax==IDC_PASS
       invoke GetDlgItemText,hWnd,IDC_EDIT1,addr szFileName,MAX_PATH
       call AddShell
      .endif
    .else
      mov eax,FALSE
       ret
      .endif
     mov eax,TRUE
     ret

_DlgProcMain endp

start:
   invoke GetModuleHandle,NULL
   mov hInstance,eax
   invoke DialogBoxParam,hInstance, addr szdlg,NULL,addr _DlgProcMain,NULL
   invoke ExitProcess,eax
   end start

--------------------------------------------------------------------------------
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 05 月 13 日 下午 06:23:02
```

`4dd466d0e1e0cc123afbe504da2462ca`
