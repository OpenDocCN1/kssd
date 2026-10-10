

# PE 程序自免疫技术程序设计一二

> 原文：[`www.pediy.com/kssd/pediy10/87396.html`](https://www.pediy.com/kssd/pediy10/87396.html)

学习《Windows 信息安全原理与实践》书过程中相关介绍，书中给的 VC 代码，闲暇将其改成汇编，供汇编初学者参考，大虾们飘过。  
PE 程序自免疫技术程序设计一：改进程序节表结构  
原理：  
插入方式的 PE 文件型病毒利用了每个字节对齐原因，程序实际的代码或数据可能没有占满该节，还有不少空余字节空间，病毒在空白区插入病毒。每节都存在空余字节空间，剩余空间大小由对应节表决定。节表结构中的联合 Misc 字段 VirtualSize 为该节的实际大小，该节实际占用空间有节表结构中字段 SizeOfRawData 得到。  
SizeOfRawData 减去 VirtualSize 的大小就是剩余空间的大小。病毒也要计算剩余空间的大小以判断是否可以插入病毒代码，如果空间不够，该种病毒就会放弃感染。这就可以启发我们，如果有意将字段 VirtualSize 修改的和 SizeOfRawData 一样大，而且在空白区填上随机数据，就可以欺骗以插入方式感染的病毒了。  
  
代码实现：  
（1）  得到节个数  
由 DOS 头结构字段 e\_lfanew 定位 PE 头结构位置，读取 PE 头，在其包含的文件头中得到节个数字段 NumberOfSections。  
（2）  读节表  
依据节个数分配节表缓冲区，读取所有节表到缓冲区。  
（3）  计算节空白区文件偏移和大小  
遍历每个节，空白区的文件偏移等于该节文件偏移（字段 PointerToRawData）加上该节大小（字段 VirtualSize）。空白区大小为该节实际占用磁盘空间（字段 SizeOfRawData）减去该节实际大小（字段 VirtualSize）得到。  
（4）  生成随机数据填充空白区  
（5）  修改节表结构，让缓冲区中每个节表的字段 VirtualSize 等于字段 SizeOfRawData，将缓冲区数据写入原节表位置。  
  
下面是我自己讲《Windows 信息安全原理与实践》中书上的 VC 程序改写成汇编语言的。   
.586p  
.model flat, stdcall  
option casemap :none   ; case sensitive  
include c:\\masm32\\include\\windows.inc  
include c:\\masm32\\include\\kernel32.inc  
includelib c:\\masm32\\lib\\kernel32.lib  
include c:\\masm32\\include\\user32.inc  
includelib c:\\masm32\\lib\\user32.lib   
include  c:\\masm32\\include\\comdlg32.inc  
includelib c:\\masm32\\lib\\comdlg32.lib  
include c:\\masm32\\include\\masm32.inc   ;这是 Alloc 函数所需的文件  
includelib c:\\masm32\\lib\\masm32.lib    ;这是 Alloc 函数所需的 Lib 文件  
  
  
  
.data  
  stOFAddr        OPENFILENAME  <>  
  szTitle         db            "ChangeSection",0  
  szFileName  db        MAX\_PATH dup (?)  
        szExtPe    db        'PE Files',0,'\*.exe;\*.dll;\*.scr;\*.fon;\*.drv',0  
            db        'All Files(\*.\*)',0,'\*.\*',0,0  
        hFile           dd            ?  
        dos\_header      IMAGE\_DOS\_HEADER <>  
  nt\_header       IMAGE\_NT\_HEADERS <>  
  section\_header  IMAGE\_SECTION\_HEADER <>  
  nBytesRead      dd            ?  
  pos             dd            ?  
  len             dd            ?  
  arrayWrite      dd            ?  
  sectionHeaderPos dd           ?  
  successTip      db            "    ChangeSection Succeded!   ",0  
.code   
  
Start:  
  invoke  RtlZeroMemory,offset stOFAddr,sizeof OPENFILENAME  
  mov  stOFAddr.lStructSize,sizeof OPENFILENAME  
        push  NULL  
  pop  stOFAddr.hwndOwner  
  mov  stOFAddr.lpstrFilter,offset szExtPe  
  mov  stOFAddr.lpstrFile,offset szFileName  
  mov  stOFAddr.nMaxFile,MAX\_PATH  
  mov  stOFAddr.Flags,OFN\_PATHMUSTEXIST or OFN\_FILEMUSTEXIST  
  invoke  GetOpenFileName,offset stOFAddr  
  .if  ! eax  
    jmp  @F  
  .endif  
  invoke  CreateFile,offset szFileName,GENERIC\_ALL ,FILE\_SHARE\_READ or \\  
      FILE\_SHARE\_WRITE,NULL,OPEN\_EXISTING,FILE\_ATTRIBUTE\_ARCHIVE,NULL  
        .if  eax !=  INVALID\_HANDLE\_VALUE  
      mov  hFile,eax  
      invoke ReadFile, hFile, offset dos\_header, sizeof IMAGE\_DOS\_HEADER, offset nBytesRead, NULL    
      movzx     eax,  dos\_header.e\_magic  
      ;检查 PE 文件是否有效  
      .if     eax == 5a4dh  
             invoke SetFilePointer, hFile, dos\_header.e\_lfanew, NULL, FILE\_BEGIN  
             invoke ReadFile, hFile, offset nt\_header, sizeof IMAGE\_NT\_HEADERS, offset nBytesRead, NULL  
             .if    nt\_header.Signature == 4550h  
                    movzx   esi,nt\_header.FileHeader.NumberOfSections  
                    cmp     esi,1  
                    jb      NoSection  
                    xor     ebx, ebx  
              LoopSections:  
                    mov     eax, dos\_header.e\_lfanew  
                    add     eax, sizeof IMAGE\_NT\_HEADERS  
                    mov     edx, ebx  
                    imul    edx, sizeof IMAGE\_SECTION\_HEADER  
                    add     eax, edx  
                    mov     sectionHeaderPos, eax  
                    ; 读取节表  
                    invoke SetFilePointer, hFile, sectionHeaderPos, NULL, FILE\_BEGIN  
                    invoke ReadFile, hFile, offset section\_header, sizeof IMAGE\_SECTION\_HEADER, offset nBytesRead, NULL  
                    ; 填充节中空白数据  
                    mov  eax, section\_header.PointerToRawData  
                    add  eax, section\_header.Misc.VirtualSize  
                    mov  pos,eax  
                    mov  eax, section\_header.SizeOfRawData  
                    sub  eax, section\_header.Misc.VirtualSize  
                    mov  len, eax  
                    invoke  Alloc,len  
              mov     dword ptr \[arrayWrite\], eax  
              xor     ecx,ecx  
        Goon:  
              mov eax, dword ptr \[arrayWrite\]  
              add eax, ecx  
              mov byte ptr\[eax\], 41h  
              inc ecx  
              cmp ecx, len  
                    jb Goon     
              invoke SetFilePointer, hFile, pos, NULL, FILE\_BEGIN  
                    invoke WriteFile, hFile, arrayWrite, len, offset nBytesRead, NULL     
                    ; 修改节表   
                    mov    eax, section\_header.SizeOfRawData  
                    mov    section\_header.Misc.VirtualSize, eax   
                    invoke SetFilePointer, hFile, sectionHeaderPos, NULL, FILE\_BEGIN  
                    invoke WriteFile, hFile, offset section\_header, sizeof IMAGE\_SECTION\_HEADER, offset nBytesRead, NULL                    
                    invoke Free, dword ptr \[arrayWrite\]  
              inc  ebx  
              dec  esi  
              cmp  esi,0  
              ja   LoopSections  
             NoSection:   
                     invoke CloseHandle, hFile  
                     invoke MessageBox, NULL, offset successTip, offset szTitle, MB\_OK       
             .endif  
      .endif  
  .endif  
@@:  
  invoke ExitProcess,0  
end Start 

PE 程序自免疫技术程序设计二：禁止插入新节  
原理：  
以附加方式感染的病毒，要添加新节就必须同时插入一个对应的新节表。能够插入新节表的前提是最后一个节表与第一个节区之间足以放下一个节表结构。而很多情况下编译器生成的可执行文件在上述之间还余有足够放多个节表的空间。  
如果把文件头和节表下移，且修改定位文件头位置的指针（DOS 头结构字段 e\_lfanew），那么病毒就无法插入新节了。  
  
代码实现：  
（1） 得到节区个数  
由 DOS 头结构字段 e\_lfanew 定位 PE 头结构位置，读取 PE 头，在其包含的文件头中得到节个数字段 NumberOfSections。  
(2)   分配缓冲区，将 PE 头结构和所有节表读到缓冲区。  
(3)   计算移动到的文件偏移，移动数据的长度为 PE 头结构大小（sizeof(IMAGE\_NT\_HEADERS)）加上所有节表结构的大小为（sizeof(IMAGE\_SECTION\_HEADER)\*nt\_header.FileHeader.NumberOfSections）。移动到的位置为第一节区的文件偏移（psection\_header\[0\].PointerToRawData），减去移动数据的长度。将缓冲区数据写入文件该位置。  
(4)  修改 DOS 头结构 e\_lfanew，并将该结构重新写入原位置。  
  
下面是我自己讲《Windows 信息安全原理与实践》中书上的 VC 程序改写成汇编语言的。  
.586p  
.model flat, stdcall  
option casemap :none   ; case sensitive  
include c:\\masm32\\include\\windows.inc  
include c:\\masm32\\include\\kernel32.inc  
includelib c:\\masm32\\lib\\kernel32.lib  
include c:\\masm32\\include\\user32.inc  
includelib c:\\masm32\\lib\\user32.lib   
include  c:\\masm32\\include\\comdlg32.inc  
includelib c:\\masm32\\lib\\comdlg32.lib  
include c:\\masm32\\include\\masm32.inc   ;这是 Alloc 函数所需的文件  
includelib c:\\masm32\\lib\\masm32.lib    ;这是 Alloc 函数所需的 Lib 文件  
  
  
  
.data  
  stOFAddr        OPENFILENAME  <>  
  szTitle         db            "ChangeSection",0  
  szFileName  db        MAX\_PATH dup (?)  
        szExtPe    db        'PE Files',0,'\*.exe;\*.dll;\*.scr;\*.fon;\*.drv',0  
            db        'All Files(\*.\*)',0,'\*.\*',0,0  
        hFile           dd            ?  
        dos\_header      IMAGE\_DOS\_HEADER <>  
  nt\_header       IMAGE\_NT\_HEADERS <>  
  section\_header  IMAGE\_SECTION\_HEADER <>  
  nBytesRead      dd            ?  
  pos             dd            ?  
  len             dd            ?  
  endPoint        dd            ?  
  arrayWrite      dd            ?  
  sectionHeaderPos dd           ?  
  successTip      db            "    ProtectSection Succeded!   ",0  
.code   
  
Start:  
  invoke  RtlZeroMemory,offset stOFAddr,sizeof OPENFILENAME  
  mov  stOFAddr.lStructSize,sizeof OPENFILENAME  
        push  NULL  
  pop  stOFAddr.hwndOwner  
  mov  stOFAddr.lpstrFilter,offset szExtPe  
  mov  stOFAddr.lpstrFile,offset szFileName  
  mov  stOFAddr.nMaxFile,MAX\_PATH  
  mov  stOFAddr.Flags,OFN\_PATHMUSTEXIST or OFN\_FILEMUSTEXIST  
  invoke  GetOpenFileName,offset stOFAddr  
  .if  ! eax  
    jmp  @F  
  .endif  
  invoke  CreateFile,offset szFileName,GENERIC\_ALL ,FILE\_SHARE\_READ or \\  
      FILE\_SHARE\_WRITE,NULL,OPEN\_EXISTING,FILE\_ATTRIBUTE\_ARCHIVE,NULL  
        .if  eax !=  INVALID\_HANDLE\_VALUE  
      mov  hFile,eax  
      invoke ReadFile, hFile, offset dos\_header, sizeof IMAGE\_DOS\_HEADER, offset nBytesRead, NULL    
      movzx     eax,  dos\_header.e\_magic  
      ;检查 PE 文件是否有效  
      .if     eax == 5a4dh  
             invoke SetFilePointer, hFile, dos\_header.e\_lfanew, NULL, FILE\_BEGIN  
             invoke ReadFile, hFile, offset nt\_header, sizeof IMAGE\_NT\_HEADERS, offset nBytesRead, NULL  
             .if    nt\_header.Signature == 4550h  
                    movzx   esi,nt\_header.FileHeader.NumberOfSections  
                    cmp     esi,1  
                    jb      NoSection  
                    xor     ebx, ebx  
                    ;读取第一个节表  
                    invoke ReadFile, hFile, offset section\_header, sizeof IMAGE\_SECTION\_HEADER, offset nBytesRead, NULL  
                      
                    mov  eax, section\_header.PointerToRawData  
                    mov  endPoint, eax  
                    mov  eax, sizeof IMAGE\_NT\_HEADERS  
                    mov  ebx, sizeof IMAGE\_SECTION\_HEADER  
                    imul ebx, esi  
                    add  eax, ebx  
                    mov  len, eax  
                    mov  eax, endPoint  
                    sub  eax, len  
                    mov  pos, eax  
                    invoke  Alloc,len  
              mov     dword ptr \[arrayWrite\], eax  
                
                    invoke SetFilePointer, hFile, dos\_header.e\_lfanew, NULL, FILE\_BEGIN  
                    invoke ReadFile, hFile, arrayWrite, len, offset nBytesRead, NULL  
                    invoke SetFilePointer, hFile, pos, NULL, FILE\_BEGIN  
                    invoke WriteFile, hFile, arrayWrite, len, offset nBytesRead, NULL  
                    mov    eax, pos  
              mov    dos\_header.e\_lfanew,eax  
              invoke SetFilePointer, hFile, 0, NULL, FILE\_BEGIN  
              invoke WriteFile, hFile, offset dos\_header, sizeof IMAGE\_DOS\_HEADER, offset nBytesRead, NULL  
             NoSection:   
                     invoke CloseHandle, hFile  
                     invoke MessageBox, NULL, offset successTip, offset szTitle, MB\_OK       
             .endif  
      .endif  
  .endif  
@@:  
  invoke ExitProcess,0  
end Start