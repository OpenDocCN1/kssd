
# 去除程序 SEH 处理校检

> 原文：[`www.pediy.com/kssd/pediy10/70298.html`](https://www.pediy.com/kssd/pediy10/70298.html)

```asm
;程 序 名:  RemoveSEHVerification.asm  
;功    能:  去掉程序的 SEH 处理程序的校检  
;工作方式:  枚举当前目录下所有的 EXE 文件,修改 SEH 相关数据,还算优雅了,没有直接清掉 Load Config Table  
;作    者:  UPlusPlus  
;时    间:  2008/08/08  
    .386  
    .model flat,stdcall  
      option casemap:none  
      include windows.inc  
      include kernel32.inc  
     include user32.inc  
     include uplusplus.inc  
     includelib user32.lib  
      includelib kernel32.lib  
    .data  
    FileFilter db "\*.exe",0  
    FindData   WIN32_FIND_DATA <>  
    CurPath db 256 dup(0)  
    hFile dd 0  
    hFind dd 0  
    PE_head_addr dd 0  
    byte_read  dd 0  
    Link dw 0808h  
    Msg db "Well done",0  
    Clr dd 0  
         dd 0  
    PE_head         IMAGE_NT_HEADERS    <0\>  
    Section_table     db        280h dup (0)   
    .code  
    start:  
        invoke GetCurrentDirectory,256,offset CurPath  
        invoke FindFirstFile,offset FileFilter,offset FindData  
        cmp eax,INVALID_HANDLE_VALUE  
        jz FindEnds  
        mov hFind,eax  
    GoOnFind:  
        invoke CreateFile,offset FindData.cFileName,GENERIC_READ+GENERIC_WRITE,FILE_SHARE_READ+FILE_SHARE_WRITE,0,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,0  
        cmp eax,INVALID_HANDLE_VALUE  
        jz createfail  
        mov hFile,eax  
        invoke SetFilePointer,hFile,3ch,0,FILE_BEGIN  
        invoke ReadFile,hFile,offset PE_head_addr,4,offset byte_read,0            ;从 3ch 读 PE 头地址  
        cmp eax,0  
        jz readfail  
        invoke SetFilePointer,hFile,PE_head_addr,0,FILE_BEGIN                       ;指针移到 PE 头  
        invoke ReadFile,hFile,offset PE_head,sizeof PE_head+sizeof Section_table,offset byte_read,0     ;读出 PE 头  
        cmp DWORD  ptr PE_head.Signature,IMAGE_NT_SIGNATURE  
        jnz exitwrite  
        lea edx,PE_head  
        lea edx,(IMAGE_NT_HEADERS ptr [edx]).OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG * sizeof IMAGE_DATA_DIRECTORY]  
        mov edx,[edx]  
        test edx,edx  
        je NoLCT  
        lea esi,[Section_table]  
        @@:  
        mov ecx,esi  
        mov eax,(IMAGE_SECTION_HEADER ptr [esi]).VirtualAddress  
        add esi,sizeof IMAGE_SECTION_HEADER  
        cmp edx,eax  
        ja @B  
        sub edx,(IMAGE_SECTION_HEADER ptr [ecx]).VirtualAddress  
        add edx,(IMAGE_SECTION_HEADER ptr [ecx]).PointerToRawData  
        lea edx,(IMAGE_LOAD_CONFIG_DIRECTORY32 ptr [edx]).SEHandlerTable  
        invoke SetFilePointer,hFile,edx,0,FILE_BEGIN  
        invoke WriteFile,hFile,offset Clr,8,offset byte_read,0  
NoLCT:  
        cmp WORD  ptr PE_head[1ah],0808h          
        jz exitwrite  
        mov eax,DWORD  ptr PE_head_addr  
        add eax,1ah  
        invoke SetFilePointer,hFile,eax,0,FILE_BEGIN  
        invoke WriteFile,hFile,offset Link,2,offset byte_read,0  
    exitwrite:  
    readfail:  
         invoke CloseHandle,hFile  
    createfail:  
        invoke FindNextFile,hFind,offset FindData  
        test eax,eax  
       jnz GoOnFind  
        FindEnds:  
        invoke FindClose,hFile  
        invoke MessageBox,NULL,offset Msg,offset Msg,64  
        invoke ExitProcess,0  
    end    start     
```

![](img/8e0d77ee22c3d8438aad7d42c7829e53.jpg "使眼色") 五楼更新了下 

```asm
;程 序 名:  RemoveSEHVerification.asm  
;功    能:  去掉程序的 SEH 处理程序的校检  
;工作方式:  枚举当前目录下所有的 EXE 文件,修改 SEH 相关数据  
;作    者:  UPlusPlus  
;时    间:  2008/08/08  
;更    正:  修正一个地方,能让非 SEH 程序也能跑 SEH  
    .386  
    .model flat,stdcall  
      option casemap:none  
      include windows.inc  
      include kernel32.inc  
     include user32.inc  
     include uplusplus.inc  
     includelib user32.lib  
      includelib kernel32.lib  
    .data  
    FileFilter db "\*.exe",0  
    FindData   WIN32_FIND_DATA <>  
    CurPath db 256 dup(0)  
    hFile dd 0  
    hFind dd 0  
    PE_head_addr dd 0  
    byte_read  dd 0  
    Link dw 0808h  
    Msg db "Well done",0  
    Clr dd 0  
         dd 0  
    SEH_off dw 0  
    PE_head         IMAGE_NT_HEADERS    <0\>  
    Section_table     db        280h dup (0)   
    .code  
    start:  
        invoke GetCurrentDirectory,256,offset CurPath  
        invoke FindFirstFile,offset FileFilter,offset FindData  
        cmp eax,INVALID_HANDLE_VALUE  
        jz FindEnds  
        mov hFind,eax  
    GoOnFind:  
        invoke CreateFile,offset FindData.cFileName,GENERIC_READ+GENERIC_WRITE,FILE_SHARE_READ+FILE_SHARE_WRITE,0,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,0  
        cmp eax,INVALID_HANDLE_VALUE  
        jz createfail  
        mov hFile,eax  
        invoke SetFilePointer,hFile,3ch,0,FILE_BEGIN  
        invoke ReadFile,hFile,offset PE_head_addr,4,offset byte_read,0            ;从 3ch 读 PE 头地址  
        cmp eax,0  
        jz readfail  
        invoke SetFilePointer,hFile,PE_head_addr,0,FILE_BEGIN                       ;指针移到 PE 头  
        invoke ReadFile,hFile,offset PE_head,sizeof PE_head+sizeof Section_table,offset byte_read,0     ;读出 PE 头  
        cmp DWORD  ptr PE_head.Signature,IMAGE_NT_SIGNATURE  
        jnz exitwrite  
        lea edx,PE_head  
        lea edx,(IMAGE_NT_HEADERS ptr [edx]).OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG * sizeof IMAGE_DATA_DIRECTORY]  
        mov edx,[edx]  
        test edx,edx  
        je NoLCT  
        lea esi,[Section_table]  
        @@:  
        mov ecx,esi  
        mov eax,(IMAGE_SECTION_HEADER ptr [esi]).VirtualAddress  
        add esi,sizeof IMAGE_SECTION_HEADER  
        cmp edx,eax  
        ja @B  
        sub edx,(IMAGE_SECTION_HEADER ptr [ecx]).VirtualAddress  
        add edx,(IMAGE_SECTION_HEADER ptr [ecx]).PointerToRawData  
        lea edx,(IMAGE_LOAD_CONFIG_DIRECTORY32 ptr [edx]).SEHandlerTable  
        invoke SetFilePointer,hFile,edx,0,FILE_BEGIN  
        invoke WriteFile,hFile,offset Clr,8,offset byte_read,0  
NoLCT:  
        IMAGE_DLLCHARACTERISTICS_NO_SEH equ 0400h  
        lea edx,PE_head  
        mov dx,(IMAGE_NT_HEADERS ptr [edx]).OptionalHeader.DllCharacteristics  
        mov SEH_off,dx  
        and dx,IMAGE_DLLCHARACTERISTICS_NO_SEH  
        je @F  
        mov edx,PE_head_addr  
        lea edx,(IMAGE_NT_HEADERS ptr [edx]).OptionalHeader.DllCharacteristics  
        and SEH_off,not IMAGE_DLLCHARACTERISTICS_NO_SEH  
        invoke SetFilePointer,hFile,edx,0,FILE_BEGIN  
        invoke WriteFile,hFile,offset SEH_off,2,offset byte_read,0  
        @@:  
        cmp WORD  ptr PE_head[1ah],0808h          
        jz exitwrite  
        mov eax,DWORD  ptr PE_head_addr  
        add eax,1ah  
        invoke SetFilePointer,hFile,eax,0,FILE_BEGIN  
        invoke WriteFile,hFile,offset Link,2,offset byte_read,0  
    exitwrite:  
    readfail:  
         invoke CloseHandle,hFile  
    createfail:  
        invoke FindNextFile,hFind,offset FindData  
        test eax,eax  
       jnz GoOnFind  
        FindEnds:  
        invoke FindClose,hFile  
        invoke MessageBox,NULL,offset Msg,offset Msg,64  
        invoke ExitProcess,0  
    end    start
```
