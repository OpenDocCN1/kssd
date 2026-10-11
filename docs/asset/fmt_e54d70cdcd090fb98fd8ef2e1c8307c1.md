
```
.386
.model flat, stdcall
option casemap:none
 
;; ----------------------------------------
;; header file and lib file
;; ----------------------------------------
include kernel32.inc
include user32.inc
include comdlg32.inc
includelib kernel32.lib
includelib user32.lib
includelib comdlg32.lib
include windows.inc
PEAlign proto dwTarNum : DWORD, dwAlignTo : DWORD
AddSection proto pMem : LPVOID, 
                pSectionName : LPVOID, 
                dwSectionSize : DWORD
;DLGproc proto dlghwnd:HWND,uMsg:UINT,wParam:WPARAM,lParam:LPARAM
;\_CenterWindow    proto    :DWORD        ;将窗口移动到屏幕中间 by 罗云彬@公用子程序部分：窗口部分
APPEND\_SIZE                 equ 2000h
.data
ofn OPENFILENAME <>
lpstrFilter db "eXe Files(\*.eXe)",0
lpstrFile db 255 dup(0)
lpstrBAKFile db 255 dup(0)
lpBAK db ".bak",0
lpsectionName db 8 dup(0)
lpsectionSize db 8 dup(0)
 
;////////////////////////////////////////////////////////////////
;以下变量定义摘自"玩命"大侠 【成果 6.1】软件保护壳技术专题 - 添加新节
;////////////////////////////////////////////////////////////////
g\_szErr                   db "错误",0
g\_szDone                  db "文件加密成功!",0
g\_szDoneCap               db "^\_^",0
g\_szOpenFileFailed        db "打不开文件",0
g\_szGetFileSizeFailed     db "获取文件大小失败",0
g\_szCreateMapFailed       db "创建文件映射失败",0
g\_szMapFileFailed         db "映射文件到内存失败",0
g\_szInvalidPE             db "无效的 PE 文件",0
g\_bError            db 0
g\_dwNewSectionSize  dd 0
 
.code
;////////////////////////////////////////////////////////////////
;以下函数修改自"玩命"大侠 【成果 6.1】软件保护壳技术专题 - 添加新节
;http://bbs.pediy.com/showthread.php?p=467116
;一文，稍作改动
;////////////////////////////////////////////////////////////////
CryptFile proc szFname : LPSTR,szSectionName:LPSTR,dwSectionSize:DWORD
    LOCAL hFile : HANDLE
    LOCAL hMap : HANDLE
    LOCAL pMem : LPVOID
    LOCAL dwOrigFileSize : DWORD
    LOCAL dwNTHeaderAddr : DWORD
 
    ;; init data
    xor eax, eax
    mov g\_bError, al
    mov eax, dwSectionSize
    mov g\_dwNewSectionSize, eax
 
    ;; open file
    invoke CreateFile, szFname,\
                     GENERIC\_WRITE + GENERIC\_READ,\
                     FILE\_SHARE\_WRITE + FILE\_SHARE\_READ,\
                     NULL,\
                     OPEN\_EXISTING,\
                     FILE\_ATTRIBUTE\_NORMAL,\
                     0
    .IF eax == INVALID\_HANDLE\_VALUE
        jmp OpenFileFailed                
    .ENDIF
    mov hFile, eax 
    invoke GetFileSize, hFile, NULL
    .IF eax == 0
        invoke CloseHandle, hFile   
        jmp GetFileSizeFailed
    .ENDIF
    mov dwOrigFileSize, eax    
    
    add eax, APPEND\_SIZE
 
    xchg eax, ecx
    ;; create memory map
    xor ebx, ebx     
    invoke CreateFileMapping, hFile, ebx, PAGE\_READWRITE, ebx, ecx, ebx
    .IF eax == 0
        invoke CloseHandle, hFile
        jmp CreateMapFailed               
    .ENDIF
    mov hMap, eax
    ;; map file to memory
    invoke MapViewOfFile, hMap,
                       FILE\_MAP\_WRITE+FILE\_MAP\_READ+FILE\_MAP\_COPY, 
                       ebx, ebx, ebx
    .IF eax == 0
        invoke CloseHandle, hMap
        invoke CloseHandle, hFile
        jmp MapFileFailed
    .ENDIF
    mov pMem, eax                                  
    ;; check it's PE file or not ?
    xchg eax, esi
    assume esi : ptr IMAGE\_DOS\_HEADER
    .IF [esi].e\_magic != 'ZM'
        invoke UnmapViewOfFile, pMem
        invoke CloseHandle, hMap
        invoke CloseHandle, hFile
        jmp InvalidPE        
    .ENDIF       
    add esi, [esi].e\_lfanew
    assume esi : ptr IMAGE\_NT\_HEADERS    
    .IF word ptr [esi].Signature != 'EP'
        invoke UnmapViewOfFile, pMem
        invoke CloseHandle, hMap
        invoke CloseHandle, hFile
        jmp InvalidPE        
    .ENDIF
    mov dwNTHeaderAddr, esi
 
 
    invoke AddSection, pMem,szSectionName, g\_dwNewSectionSize
    push eax
    mov esi, dwNTHeaderAddr
    assume esi : ptr IMAGE\_NT\_HEADERS
 
 
LogicShellExit:
    ;; close handle & write it
    invoke UnmapViewOfFile, pMem
    invoke CloseHandle, hMap
    invoke CloseHandle, hFile
    .IF g\_bError == 0
        ;; show success message   
        invoke MessageBox, NULL, offset g\_szDone, offset g\_szDoneCap, MB\_ICONINFORMATION
    .ENDIF         
    ret
;; ----- Show error message ----- 
OpenFileFailed:
    lea eax, g\_szOpenFileFailed
    jmp ShowErr
GetFileSizeFailed:
    lea eax, g\_szGetFileSizeFailed
    jmp ShowErr     
CreateMapFailed:
    lea eax, g\_szCreateMapFailed
    jmp ShowErr
MapFileFailed:
    lea eax, g\_szMapFileFailed
    jmp ShowErr        
InvalidPE:           
    lea eax, g\_szInvalidPE
    jmp ShowErr    
ShowErr:
    invoke MessageBox, NULL, eax, offset g\_szErr, MB\_ICONERROR
    mov al, 1
    mov g\_bError, al
    jmp LogicShellExit
 
CryptFile endp 
 
AddSection proc uses ebx ecx edx esi edi, pMem : LPVOID,
                          pSectionName : LPVOID,
                          dwSectionSize : DWORD
;; add a new section
;; ret: eax =  new section table file offset
    LOCAL dwNTHeader : LPVOID
    LOCAL dwLastSecTbl : LPVOID     
    LOCAL dwFileAlig : DWORD
    LOCAL dwSecAlig : DWORD
 
    ;; move to section table
    mov esi, pMem
 
    ;; assume esi : ptr IMAGE\_DOS\_HEADER
    ;; add esi, dword ptr [esi].e\_lfanew
    add esi, dword ptr [esi+3ch]
    mov dwNTHeader, esi   
    assume esi : ptr IMAGE\_NT\_HEADERS
    ;; update the number of section
    mov cx, word ptr [esi].FileHeader.NumberOfSections
    movzx ecx, cx
 
    inc word ptr [esi].FileHeader.NumberOfSections
    push dword ptr [esi].OptionalHeader.FileAlignment
    pop dwFileAlig
    push dword ptr [esi].OptionalHeader.SectionAlignment
    pop dwSecAlig         
    ;; move esi point to section table
 
    add esi, sizeof IMAGE\_NT\_HEADERS
    ;; store the last section table
 
    mov eax, sizeof IMAGE\_SECTION\_HEADER
    mov ebx, ecx
    imul ebx
 
    add esi, eax                        ; esi = the end of orig last section fva
    push esi
 
    sub esi, sizeof IMAGE\_SECTION\_HEADER    ; esi = the orig last section fva
    mov dwLastSecTbl, esi
    pop esi
    ;; set new section table
 
    assume esi : ptr IMAGE\_SECTION\_HEADER
    ;; set section name
    push esi
    lea edi, [esi].Name1
    mov esi, pSectionName
CopySectionNameLoop:    
    lodsb
    test al, al
    jz EndCopySectionNameLoop
    stosb
    jmp CopySectionNameLoop
EndCopySectionNameLoop:   
    pop esi   
 
    push 0E00000E0h
    pop dword ptr [esi].Characteristics
 
    push dwSectionSize
    pop dword ptr [esi].Misc.VirtualSize
 
    invoke PEAlign, dwSectionSize, dwFileAlig
    mov dword ptr [esi].SizeOfRawData, eax
 
    mov eax, dwLastSecTbl                 ; eax = orig last section table fva
    assume eax : ptr IMAGE\_SECTION\_HEADER
    mov ecx, dword ptr [eax].VirtualAddress
    add ecx, dword ptr [eax].Misc.VirtualSize        ; ecx = new section rva
    mov ebx, dword ptr [eax].PointerToRawData
    add ebx, dword ptr [eax].SizeOfRawData           ; ebx = new section fva
    invoke PEAlign, ecx, dwSecAlig
    mov dword ptr [esi].VirtualAddress, eax
    ;; set section pointertorawdata
    invoke PEAlign, ebx, dwFileAlig
    mov dword ptr [esi].PointerToRawData, eax
 
    mov eax, dword ptr [esi].VirtualAddress
    add eax, dword ptr [esi].Misc.VirtualSize
    invoke PEAlign, eax, dwSecAlig
    mov edx, dwNTHeader
    assume edx : ptr IMAGE\_NT\_HEADERS
    mov dword ptr [edx].OptionalHeader.SizeOfImage, eax
    push dword ptr [esi].PointerToRawData
    pop edi
    add edi, pMem
    ;; clear the new sec
 
    mov ecx, dwSectionSize
    xor eax, eax
    cld
    rep stosb
 
 
    mov eax, esi
    assume esi : nothing
    assume eax : nothing
    assume edx : nothing
    ret
AddSection endp
 
PEAlign proc uses ecx edx, dwTarNum : DWORD, dwAlignTo : DWORD
;; returns aligned value     
    ;; Algorithms:
    ;; $1 = dwTarNum / dwAlignTo
    ;; if remain != 0
    ;; $r = $1 + 1 \* dwAlignTo
    ;; return $r
    mov ecx, dwAlignTo
    mov eax, dwTarNum
    xor edx, edx
    div ecx
    cmp edx, 0
    jz AlreadyAligned
    inc eax
AlreadyAligned:
    mul ecx       
    ret
 
PEAlign endp    
 
;////////////////////////////////////////////////////////////////////////
;引用代码结束
;////////////////////////////////////////////////////////////////////////
\_CenterWindow    proc    hWnd:DWORD
        local    @stRectDeskTop:RECT,@stRectWin:RECT
        local    @dwWidth:DWORD,@dwHeight:DWORD
 
        invoke    GetWindowRect,hWnd,addr @stRectWin
        invoke    GetDesktopWindow
        mov    ebx,eax
        invoke    GetWindowRect,ebx,addr @stRectDeskTop
 
        mov    eax,@stRectWin.bottom
        sub    eax,@stRectWin.top
        mov    @dwHeight,eax
        mov    eax,@stRectWin.right
        sub    eax,@stRectWin.left
        mov    @dwWidth,eax
 
        mov    ebx,@stRectDeskTop.bottom
        sub    ebx,@dwHeight
        shr    ebx,1
        mov    ecx,@stRectDeskTop.right
        sub    ecx,@dwWidth
        shr    ecx,1
 
        invoke    MoveWindow,hWnd,ecx,ebx,@dwWidth,@dwHeight,FALSE
        ret
 
\_CenterWindow    endp
 
LpstrToHex proc uses esi edi ecx edx ebx, lpstr:LPSTR
 
    LOCAL dwM:DWORD
    mov ebx,10h
    mov edi,lpstr
    mov dwM,0
    invoke lstrlen,lpstr
    mov esi,eax
looop3:   
        .if esi\>0
        mov al,byte ptr [edi]
                .if al\>=30h
              .if al<=39h
            sub al,30h
                    jmp looop
              .elseif al\>=61h
                   .if al<=66h
                    sub al,61h
                    add al,0ah
                    jmp looop
 
                   .elseif al\>=41h
                        .if al<=46h
                    sub al,41h
                    add al,0ah
                    jmp looop
                    .endif
                   .endif
              .endif
        looop:   
                movzx eax,al
            mov ecx,esi
            dec ecx
            looop2: 
                    .if ecx\>0
                mul ebx
                dec ecx
                jmp looop2
            .endif
            add dwM,eax
            inc edi
            dec esi
            jmp looop3
        .endif
    .endif
    mov eax,dwM
    ret
 
LpstrToHex endp
DLGproc proc dlghwnd:HWND,uMsg:UINT,wParam:WPARAM,lParam:LPARAM
        mov    eax,uMsg
        .if    eax == WM\_CLOSE
            invoke    EndDialog,dlghwnd,NULL
;*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
        .elseif    eax == WM\_INITDIALOG
            invoke    \_CenterWindow,dlghwnd
;*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
        .elseif    eax == WM\_COMMAND
            mov    eax,wParam
            movzx    eax,ax
            .if    eax ==    1001
                   mov ofn.lStructSize,sizeof ofn
                              mov eax,offset lpstrFilter  ;初始化 ofn 结构
              mov ofn.lpstrFilter,eax
              mov eax,offset lpstrFile
              mov ofn.lpstrFile,eax
              mov ofn.nMaxFile,sizeof lpstrFile
              mov ofn.Flags,OFN\_FILEMUSTEXIST or OFN\_PATHMUSTEXIST
            invoke GetOpenFileName,addr ofn
            .if eax\==1
                invoke SetDlgItemText,dlghwnd,1003,ofn.lpstrFile
            .endif
        .elseif    eax\==1008
                   invoke IsDlgButtonChecked,dlghwnd,1002
                   .if eax\==1
                     invoke lstrcpy,offset lpstrBAKFile,offset lpstrFile
                     invoke lstrcat,offset lpstrBAKFile,offset lpBAK
                     invoke CopyFile,offset lpstrFile,offset lpstrBAKFile,0
                 .endif
              invoke GetDlgItemText,dlghwnd,1003,offset lpstrFile,255
              invoke GetDlgItemText,dlghwnd,1005,offset lpsectionName,8
              invoke GetDlgItemText,dlghwnd,1007,offset lpsectionSize,8
              invoke LpstrToHex,offset lpsectionSize
              invoke CryptFile,offset lpstrFile,offset lpsectionName,eax
            .endif
;*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;    注意：对话框的消息处理后，要返回 TRUE,对没有处理的消息
;    要返回 FALSE
;*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
        .else
            mov    eax,FALSE
            ret
        .endif
        mov    eax,TRUE
        ret
 
DLGproc endp
start:
invoke GetModuleHandle,0
invoke DialogBoxParam,eax,1000,0,addr DLGproc,0
invoke ExitProcess,0
end start
 
 
```
