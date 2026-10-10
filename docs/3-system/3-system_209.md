

# 去除程序 SEH 处理校检

> 原文：[`www.pediy.com/kssd/pediy10/70298.html`](https://www.pediy.com/kssd/pediy10/70298.html)

;程 序 名:  RemoveSEHVerification.asm  
;功    能:  去掉程序的 SEH 处理程序的校检  
;工作方式:  枚举当前目录下所有的 EXE 文件,修改 SEH 相关数据,还算优雅了,没有直接清掉 Load Config Table  
;作    者:  UPlusPlus  
;时    间:  2008/08/08   
  
    .386  
    .model flat,stdcall  
      option casemap:none  
      include windows.inc  
      include kernel32.inc  
     include user32.inc  
     include uplusplus.inc  
     includelib user32.lib  
      includelib kernel32.lib  
      
        
    .data  
    FileFilter db "\*.exe",0  
    FindData   WIN32\_FIND\_DATA <>  
  
    CurPath db 256 dup(0)  
    hFile dd 0  
    hFind dd 0  
    PE\_head\_addr dd 0  
    byte\_read  dd 0  
    Link dw 0808h  
    Msg db "Well done",0  
    Clr dd 0  
         dd 0  
      
    PE\_head             IMAGE\_NT\_HEADERS    <0\>  
    Section\_table       db        280h dup (0)    
      
    .code  
    start:  
      
        invoke GetCurrentDirectory,256,offset CurPath  
        invoke FindFirstFile,offset FileFilter,offset FindData  
        cmp eax,INVALID\_HANDLE\_VALUE  
        jz FindEnds  
        mov hFind,eax  
    GoOnFind:   
        invoke CreateFile,offset FindData.cFileName,GENERIC\_READ+GENERIC\_WRITE,FILE\_SHARE\_READ+FILE\_SHARE\_WRITE,0,OPEN\_EXISTING,FILE\_ATTRIBUTE\_NORMAL,0  
        cmp eax,INVALID\_HANDLE\_VALUE  
        jz createfail  
        mov hFile,eax  
        invoke SetFilePointer,hFile,3ch,0,FILE\_BEGIN  
        invoke ReadFile,hFile,offset PE\_head\_addr,4,offset byte\_read,0             ;从 3ch 读 PE 头地址  
        cmp eax,0  
        jz readfail  
        invoke SetFilePointer,hFile,PE\_head\_addr,0,FILE\_BEGIN                      ;指针移到 PE 头  
        invoke ReadFile,hFile,offset PE\_head,sizeof PE\_head+sizeof Section\_table,offset byte\_read,0      ;读出 PE 头  
          
        cmp DWORD  ptr PE\_head.Signature,IMAGE\_NT\_SIGNATURE  
        jnz exitwrite  
  
        lea edx,PE\_head  
        lea edx,(IMAGE\_NT\_HEADERS ptr \[edx\]).OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_LOAD\_CONFIG \* sizeof IMAGE\_DATA\_DIRECTORY\]  
        mov edx,\[edx\]  
        test edx,edx  
        je NoLCT  
          
        lea esi,\[Section\_table\]  
        @@:  
        mov ecx,esi  
        mov eax,(IMAGE\_SECTION\_HEADER ptr \[esi\]).VirtualAddress  
        add esi,sizeof IMAGE\_SECTION\_HEADER  
        cmp edx,eax  
        ja @B  
        sub edx,(IMAGE\_SECTION\_HEADER ptr \[ecx\]).VirtualAddress  
        add edx,(IMAGE\_SECTION\_HEADER ptr \[ecx\]).PointerToRawData  
        lea edx,(IMAGE\_LOAD\_CONFIG\_DIRECTORY32 ptr \[edx\]).SEHandlerTable  
        invoke SetFilePointer,hFile,edx,0,FILE\_BEGIN  
        invoke WriteFile,hFile,offset Clr,8,offset byte\_read,0  
NoLCT:  
        cmp WORD  ptr PE\_head\[1ah\],0808h              
        jz exitwrite  
        mov eax,DWORD  ptr PE\_head\_addr  
        add eax,1ah  
        invoke SetFilePointer,hFile,eax,0,FILE\_BEGIN  
        invoke WriteFile,hFile,offset Link,2,offset byte\_read,0  
    exitwrite:  
    readfail:  
         invoke CloseHandle,hFile  
    createfail:  
          
        invoke FindNextFile,hFind,offset FindData  
        test eax,eax  
       jnz GoOnFind   
                
        FindEnds:  
        invoke FindClose,hFile  
        invoke MessageBox,NULL,offset Msg,offset Msg,64  
        invoke ExitProcess,0  
  
    end    start      
  
![](img/8e0d77ee22c3d8438aad7d42c7829e53.jpg "使眼色") 五楼更新了下 

;程 序 名:  RemoveSEHVerification.asm  
;功    能:  去掉程序的 SEH 处理程序的校检  
;工作方式:  枚举当前目录下所有的 EXE 文件,修改 SEH 相关数据  
;作    者:  UPlusPlus  
;时    间:  2008/08/08  
;更    正:  修正一个地方,能让非 SEH 程序也能跑 SEH  
    .386  
    .model flat,stdcall  
      option casemap:none  
      include windows.inc  
      include kernel32.inc  
     include user32.inc  
     include uplusplus.inc  
     includelib user32.lib  
      includelib kernel32.lib  
      
        
    .data  
    FileFilter db "\*.exe",0  
    FindData   WIN32\_FIND\_DATA <>  
  
    CurPath db 256 dup(0)  
    hFile dd 0  
    hFind dd 0  
    PE\_head\_addr dd 0  
    byte\_read  dd 0  
    Link dw 0808h  
    Msg db "Well done",0  
    Clr dd 0  
         dd 0  
    SEH\_off dw 0  
    PE\_head             IMAGE\_NT\_HEADERS    <0\>  
    Section\_table       db        280h dup (0)    
      
    .code  
    start:  
      
        invoke GetCurrentDirectory,256,offset CurPath  
        invoke FindFirstFile,offset FileFilter,offset FindData  
        cmp eax,INVALID\_HANDLE\_VALUE  
        jz FindEnds  
        mov hFind,eax  
    GoOnFind:   
        invoke CreateFile,offset FindData.cFileName,GENERIC\_READ+GENERIC\_WRITE,FILE\_SHARE\_READ+FILE\_SHARE\_WRITE,0,OPEN\_EXISTING,FILE\_ATTRIBUTE\_NORMAL,0  
        cmp eax,INVALID\_HANDLE\_VALUE  
        jz createfail  
        mov hFile,eax  
        invoke SetFilePointer,hFile,3ch,0,FILE\_BEGIN  
        invoke ReadFile,hFile,offset PE\_head\_addr,4,offset byte\_read,0             ;从 3ch 读 PE 头地址  
        cmp eax,0  
        jz readfail  
        invoke SetFilePointer,hFile,PE\_head\_addr,0,FILE\_BEGIN                      ;指针移到 PE 头  
        invoke ReadFile,hFile,offset PE\_head,sizeof PE\_head+sizeof Section\_table,offset byte\_read,0      ;读出 PE 头  
          
        cmp DWORD  ptr PE\_head.Signature,IMAGE\_NT\_SIGNATURE  
        jnz exitwrite  
  
        lea edx,PE\_head  
        lea edx,(IMAGE\_NT\_HEADERS ptr \[edx\]).OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_LOAD\_CONFIG \* sizeof IMAGE\_DATA\_DIRECTORY\]  
        mov edx,\[edx\]  
        test edx,edx  
        je NoLCT  
          
        lea esi,\[Section\_table\]  
        @@:  
        mov ecx,esi  
        mov eax,(IMAGE\_SECTION\_HEADER ptr \[esi\]).VirtualAddress  
        add esi,sizeof IMAGE\_SECTION\_HEADER  
        cmp edx,eax  
        ja @B  
        sub edx,(IMAGE\_SECTION\_HEADER ptr \[ecx\]).VirtualAddress  
        add edx,(IMAGE\_SECTION\_HEADER ptr \[ecx\]).PointerToRawData  
        lea edx,(IMAGE\_LOAD\_CONFIG\_DIRECTORY32 ptr \[edx\]).SEHandlerTable  
        invoke SetFilePointer,hFile,edx,0,FILE\_BEGIN  
        invoke WriteFile,hFile,offset Clr,8,offset byte\_read,0  
NoLCT:  
        IMAGE\_DLLCHARACTERISTICS\_NO\_SEH equ 0400h  
        lea edx,PE\_head  
        mov dx,(IMAGE\_NT\_HEADERS ptr \[edx\]).OptionalHeader.DllCharacteristics  
        mov SEH\_off,dx  
        and dx,IMAGE\_DLLCHARACTERISTICS\_NO\_SEH  
        je @F  
        mov edx,PE\_head\_addr  
        lea edx,(IMAGE\_NT\_HEADERS ptr \[edx\]).OptionalHeader.DllCharacteristics  
        and SEH\_off,not IMAGE\_DLLCHARACTERISTICS\_NO\_SEH  
        invoke SetFilePointer,hFile,edx,0,FILE\_BEGIN  
        invoke WriteFile,hFile,offset SEH\_off,2,offset byte\_read,0  
        @@:  
          
        cmp WORD  ptr PE\_head\[1ah\],0808h              
        jz exitwrite  
        mov eax,DWORD  ptr PE\_head\_addr  
        add eax,1ah  
        invoke SetFilePointer,hFile,eax,0,FILE\_BEGIN  
        invoke WriteFile,hFile,offset Link,2,offset byte\_read,0  
    exitwrite:  
    readfail:  
         invoke CloseHandle,hFile  
    createfail:  
          
        invoke FindNextFile,hFind,offset FindData  
        test eax,eax  
       jnz GoOnFind   
                
        FindEnds:  
        invoke FindClose,hFile  
        invoke MessageBox,NULL,offset Msg,offset Msg,64  
        invoke ExitProcess,0  
  
    end    start